- Create a root ca, refer the [link](https://github.com/sumitmaji/kubernetes/blob/oldKubeInstall/certificates/install_ca.sh)
- Create client certificate and sign them with ca
```shell
# CN is the name of the user and O is group
openssl genrsa -out dashboard.key 4096
openssl req -new -key dashboard.key -out dashboard.csr -subj "/CN=dashboard/O=dashboard:masters"
openssl x509 -req -in dashboard.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out dashboard.crt -days 7200
```
- Create certificates for vm(host)
```shell
openssl genrsa -out ${APP_HOST}.key 4096
openssl req -new -key ${APP_HOST}.key -out ${APP_HOST}.csr -subj "/CN=${APP_HOST}" \
-addext "subjectAltName = DNS:${APP_HOST}"
openssl x509 -req -in ${APP_HOST}.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out ${APP_HOST}.crt -days 7200
```

- Checking content of csr
```shell
openssl req  -noout -text -in lancelot.csr
```

- Checking the context crt
```shell
openssl x509 -noout -text -in /etc/kubernetes/pki/ca.crt
```

- Adding certificate for browser
```shell
openssl pkcs12 -export -clcerts -inkey ${USERNAME}.key -in temp.crt -out ${USERNAME}.p12 -name "kubernetes-client"
# Add the p12 file in browser certificate
```