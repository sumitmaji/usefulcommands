# Curl Commands

`GET Calls`
- Normal curl call
```console
curl -si https://master.cloud.com:32028/app1 | head -n 
```

- To disable certificate check
```console
curl -sk -i https://master.cloud.com:32028/app1 | head -n 
```

- To increase the verbosity, this would show you request headers.
```console
curl -kv auth.cub.marchenko.net.ua/check | head -n
```

- To add values in header
```console
curl -si -H "Cookie: authorization=345" https://master.cloud.com:32028/app1
```

- Providing ca certificate
```console
curl --cacert ${path to ca certificate ca.crt} https://master.cloud.com:32028/app1 
```

- To send bearer token
```console
curl -k https://master.cloud.com/api
   -H "Accept: application/json"
   -H "Authorization: Bearer {token}"
```

`POST Calls`

- To send json data in body of the request
```console
curl --header "Content-Type: application/json" \
  --request POST \
  --data '{"login":"my_login","password":"my_password"}' \
  http://master.cloud.com:32028/localpayload
```

- To send json data as in the body of the request present in file, tr.json is a file containing json data
```console
curl --header "Content-Type: application/json" \
  --request POST \
  --data @tr.json \
  http://master.cloud.com:32028/localpayload
```

- To pretty print data coming from curl
```console
curl .... | jq 
```