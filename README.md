# Useful Commands

- Stop the kubelet service running in vm
```console
systemctl stop kubelet
```

- Start the kubelet service running in vm
```console
systemctl start kubelet
```

- Check the status of kubelet service
```console
systemctl status kubelet
```

- Debugging logs in kubelet
```console
journalctl -u kubelet 
```

- To free ram memory
```console
su -c "echo 3 >'/proc/sys/vm/drop_caches' && swapoff -a && swapon -a && printf '\n%s\n' 'Ram-cache and Swap Cleared'" root 
```

- To check ram memory
```console
free -h 
```

- To add content to file
```console
cat <<EOF > output.txt
test
EOF 
```

- To print all commands executed
```shell
#!/bin/bash

[[ "TRACE" ]] && set -x
```

- To provide input to commands
```shell
kdb5_ldap_util -D cn=admin,$BASE_DN -w $LDAP_PASSWORD stashsrvpw \
-f /etc/krb5kdc/service.keyfile cn=kdc-srv,ou=krb5,$BASE_DN << EOF
$KDC_PASSWORD
$KDC_PASSWORD
EOF
```

- To send input to script
```shell
# e.g. ./execute.sh -h 192.168.0.23 -e LOCAL
while [ $# -gt 0 ]; do
    case "$1" in
        -h | --host)
        shift
        CLOUD_HOST_IP=$1
        ;;
        -e | --env)
        shift
        ENV=$1
        ;;
    esac
shift
done
```

- To set environment variable in script
```shell
: ${USERNAME:=sumit}
: ${USERNAME:=$USER_NAME}
: ${USERNAME_PASS:=$(</etc/secret/ldap/password)}
```

- Reading content of file and pass into variable
```shell
export PASS=$(</etc/secret/ldap/password)
```

- To execute command in shell script and assign to a variable
```shell
MY_ECHO=$(echo "abc")
echo $MY_ECHO
```

- Applying shell script in cat command, also using variable values
```shell
cat <<EOF | sudo tee openssl.cnf
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
HOSTNAME=$(hostname -s)
$keyUsage
`
if [ $TYPE == 'server' ]
then 
echo "subjectAltName = IP:$NODE_IP, DNS:$HOSTNAME"
fi`
EOF
```

- Taking output from method call and assign it to variable
```shell
get_data(){
  echo "My data"
}

MY_VARIABLE=$(get_data)
```

- Check if file exists or vise versa
```shell
if [ ! -f kubernetes-server-linux-amd64.tar.gz ]
then
  #TODO
fi
```

- Check if directory exists or vise versa
```shell
if [ ! -d /opt/kubernetes ]
then
 #TODO
fi
```

- Check if command is executed successfully
```shell
ar -xf kubernetes-server-linux-amd64.tar.gz -C /opt/
 if [ $? -ne 0 ]
 then
   exit 1
 fi
```

- Inorder to put replace string having `/` using sed, follow below commands
```shell
APISERVER_HOST="$(echo $APISERVER_HOST | sed 's/\//\\\//g')"
sed -i "s/\$APISERVER_HOST/$APISERVER_HOST/" $WORKDIR/v1.6.3.yaml
```

- Append at the end of the file
```shell
sed -i '$a\##rndc-key copy end' ./rndc-key
```

- Appending after certain line
```shell
sed '393 a \            - --default-backend-service=$(POD_NAMESPACE)/default-backend' ./rndc-key
```

- Looping through string using split 
```shell
get_etcd_cluster(){
# String has format: ETCD_CLUSTERS=IP1:HOST1,IP2:HOST2...
 IFS=','
 counter=0
 cluster=""
 for worker in $ETCD_CLUSTERS; do
  oifs=$IFS
  IFS=':'
  read -r ip node <<< "$worker"
  if [ -z "$cluster" ]
  then
   cluster="$node=${proto}://$ip:2380"
  else
   cluster="$cluster,$node=${proto}://$ip:2380"
  fi
  counter=$((counter+1))
  IFS=$oifs
 done
 unset IFS
 echo "$cluster"

}
```