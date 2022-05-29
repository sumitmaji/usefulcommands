
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
    - Certificates
      - Creating user certificates
  ```shell
  Hi  
  ```
        - Signing certificates using ca.crt
        - Signing certificates via kubectl
    - Token based authentication