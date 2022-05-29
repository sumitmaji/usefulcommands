
- Kubernetes secrets are mounted on `/var/run/secrets/kubernetes.io/serviceaccount` directory

- To use dnsutil pod:
    - Install dnutil pod
    ```shell
    cat <<EOF | kubectl apply -f -
    apiVersion: v1
    kind: Pod
    metadata:
    name: dnsutils
    namespace: default
    spec:
    containers:
    - name: dnsutils
      image: k8s.gcr.io/e2e-test-images/jessie-dnsutils:1.3
      command:
        - sleep
        - "3600"
          imagePullPolicy: IfNotPresent
          restartPolicy: Always
    EOF
    ```
    - Execute the command in pod
    ```shell
    kubectl exec -i -t dnsutils -- nslookup kubernetes.default.svc.cloud.uat 
    ```

- [ClusterConfiguration](https://pkg.go.dev/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2#ClusterConfiguration) file to be used in kubeadm

- To use curl in pod

    - Install curl image
    ```shell
    cat <<EOF | kubectl apply -f -
    apiVersion: v1
    kind: Pod
    metadata:
      name: curl
    spec:
      containers:
      - name: main
        image: tutum/curl
        command: ["sleep", "9999999"]
    EOF  
    ```
    - Execute command in pod
    ```shell
    kubectl exec -i -t curl -- curl -kv https://kubenetes 
    ```
    - By putting `CURL_CA_BUNDLE` environment variable, you dont need to specify `--cacert` in the curl command
    ```shell
    export CURL_CA_BUNDLE=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    ```

    - Calling kubernetes service api from container
    ```shell
    export CURL_CA_BUNDLE=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
    curl -H "Athorization: Bearer $TOKEN" https://kubernetes
    ```

- To generate a kubeconfig file having only server details. The below generates kubeconfig file /root/oauth.conf.
```shell
kubectl config set-cluster cloud.com --certificate-authority=/etc/kubernetes/pki/ca.crt --embed-certs=true --server=https://192.168.43.23:6443 --kubeconfig=/root/oauth.conf
kubectl config --kubeconfig=/root/oauth.conf set-context oauthuser@cloud.com --cluster=cloud.com --user=oauthuser
kubectl config --kubeconfig=/root/oauth.conf use-context oauthuser@cloud.com
```

- Creating roles for user/groups and granting them access
    - Granting access to a user
    ```shell
    TODO
    ```
    - Granting access to a group
    ```shell
    # This assign cluster-admin role to dashboard:master group, any user
    # Who belongs to this group will have cluster admin role. name under subjects tags 
    # can contain any group name
    cat <<EOF | kubectl create -f -
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: dashboard-cluster-admin
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
    - apiGroup: rbac.authorization.k8s.io
      kind: Group
      name: dashboard:masters
    EOF
    ```

  - Different ways of authenticating users and granting them access:
      1. Certificates
         1. Creating user certificates
         ```shell
         # here `CN` contains the username and `O` contains user group name, same group name should existing while creaing roles and assigning them access
         # this creates key and csr file which need to be approved by ca(Certificate Authority)
         export USERNAME=sumit
         openssl genrsa -out ${USERNAME}.key 4096
         openssl req -new -key ${USERNAME}.key -out ${USERNAME}.csr -subj "/CN=${USERNAME}/O=cloud:masters"
         ```
            1. Signing certificates using ca.crt
            ```shell
            export USERNAME=sumit
            openssl x509 -req -in ${USERNAME}.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out ${USERNAME}.crt -days 7200
            ```
            2. Signing certificates via kubectl
            ```shell
            # The certificate is signed by certificate authority and corresponding crt file is generated.
            export USERNAME=sumit
            cat <<EOF > ${USERNAME}-signing-request.yaml
            apiVersion: certificates.k8s.io/v1
            kind: CertificateSigningRequest
            metadata:
              name: __USERNAME__-csr
            spec:
              groups:
                - system:authenticated
                - cloud:masters
              request: __CSRREQUEST__
              signerName: kubernetes.io/kube-apiserver-client
              usages:
                - digital signature
                - key encipherment
                - client auth
            EOF
            export B64=`cat ${USERNAME}.csr | base64 | tr -d '\n'`
            sed -i "s@__USERNAME__@${USERNAME}@" ${USERNAME}-signing-request.yaml
            sed -i "s@__CSRREQUEST__@${B64}@" ${USERNAME}-signing-request.yaml
            kubectl create -f ${USERNAME}-signing-request.yaml
            kubectl certificate approve ${USERNAME}-csr
            kubectl get csr ${USERNAME}-csr -o jsonpath='{.status.certificate}' | base64 -d > ${USERNAME}.crt
            ```
         2. Using certificates to create kubeconfig file
         ```shell
         export CLUSTER=cloud.com
         export CA_CERTIFICATE=/etc/kubernetes/pki/ca.crt
         export API_SERVER=https://192.168.0.12:6443
         export USERNAME=sumit
         kubectl config set-cluster $CLUSTER --certificate-authority=$CA_CERTIFICATE --embed-certs=true --server=$API_SERVER --kubeconfig=${USERNAME}-kubeconfig
         kubectl config set-credentials ${USERNAME} --client-certificate=${USERNAME}.crt --client-key=${USERNAME}.key --embed-certs=true --kubeconfig=${USERNAME}-kubeconfig
         kubectl config set-context $CLUSTER --cluster=$CLUSTER --user=${USERNAME} --kubeconfig=${USERNAME}-kubeconfig
         kubectl config use-context cloud.com --kubeconfig=${USERNAME}-kubeconfig
         ```
         3. Using kubeconfig file to login.
         ```shell
         export USERNAME=sumit
         alias kctl='kubectl --kubeconfig=${USERNAME}-kubeconfig' 
         kctl get po
         ```
      2. Token based authentication - OpenId Connect
         1. Setup kubernetes to enable oidc. Refer the [`link`](https://github.com/sumitmaji/kubernetes/blob/master/install_k8s/cluster-config-master.yaml#L8) to for set up.
         2. Setup a [`proxy application`](https://github.com/sumitmaji/kubeauthentication/blob/main/src/main/java/com/sum/security/KubeController.java), that would authenticate user credentials with openId connect and returns id_token.
         3. A [`cli`](https://github.com/sumitmaji/kubernetes/tree/master/install_k8s/kube-login) that would call the proxy application and saves the token.
         4. Create kubeconf file containing server details, below command would create oauth.conf file.
         ```shell
         kubectl config set-cluster cloud.com --certificate-authority=/etc/kubernetes/pki/ca.crt --embed-certs=true --server=https://192.168.43.23:6443 --kubeconfig=oauth.conf
         kubectl config --kubeconfig=oauth.conf set-context oauthuser@cloud.com --cluster=cloud.com --user=oauthuser
         kubectl config --kubeconfig=oauth.conf use-context oauthuser@cloud.com
         ```
         5. Pass the token via kubectl command
         ```shell
         kubectl --kubeconfig=oauth.conf --token=__TOKEN__ get po
         ```
      3. [Webhook](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#webhook-token-authentication) based authentication.
      This is yet to be done, details present in the [link](https://itnext.io/implementing-ldap-authentication-for-kubernetes-732178ec2155)
         1. Setup Kubernetes to enable proxy based authentication.
         2. Setup a [`proxy application`](https://github.com/sumitmaji/kubeauthentication/blob/main/src/main/java/com/sum/security/KubeController.java), 
         that would authenticate user credentials return TokenReview object containing authenticated and group details. Please check the above
         url for more details.
         3. Pass the user authentication details to kubernetes
         ```shell
         kubectl --kubeconfig admin.conf config set-credentials alice --token alice:alicepassword
         # Create ClusterRole
         kubectl --kubeconfig admin.conf create clusterrole alice --resource nodes --verb list
         # Create ClusterRoleBinding
         kubectl --kubeconfig admin.conf create clusterrolebinding alice --clusterrole alice --user alice
         kubectl --kubeconfig admin.conf --user alice get nodes
         ```
