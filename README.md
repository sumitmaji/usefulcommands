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

