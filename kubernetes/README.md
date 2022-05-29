
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