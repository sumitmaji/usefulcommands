
- Kubernetes secrets are mounted on `/var/run/secrets/kubernetes.io/serviceaccount` directory

- By putting `CURL_CA_BUNDLE` environment variable, you dont need to specify `--cacert` in the curl command
```console
export CURL_CA_BUNDLE=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

- Calling kubernetes service api from container
```console
export CURL_CA_BUNDLE=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl -H "Athorization: Bearer $TOKEN" https://kubernetes
```

- To use dnsutil pod:
    - Install dnutil pod
    ```console
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
    ```console
    kubectl exec -i -t dnsutils -- nslookup kubernetes.default.svc.cloud.uat 
    ```

- To use curl in pod

  - Install curl image
    ```console
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
