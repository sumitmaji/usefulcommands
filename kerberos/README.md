
- Please refer the links for details about [kerberos](http://techpubs.spinlocksolutions.com/dklar/kerberos.html) 
and [kerberized ssh](http://jurjenbokma.com/ApprenticesNotes/kerberized_ssh.xhtml).

- Login the kerberos terminal as admin
```shell
kadmin -p root/admin
# Password: admin
```
- Exit from kerberos terminal
```shell
quit
```

- To list principals available
```shell
kadmin -p root/admin
listprincs
```

- Add user to kerberos
```shell
kadmin -p root/admin
addprinc -policy user smaji
```

- On ldap client, generate the kerberos ticket
```shell
su smaji
kinit smaji
# Provide password
klist -f
```

- Renewing kerberos ticket
```shell
krenew -v
# krenew: renewing credentials for apprentice@MYDOMAIN.COM
klist
```

- To destroy the ticket
```shell
su smaji
kinit smaji
kdestroy
```

### Note: In current dns server configuration in kubernetes, a ping to ldapclient2 is getting resolved to `192-168-255-22.ldapclient2.default.svc.cloud.uat`
rather than `ldapclient2.default.svc.cloud.uat`
Because of the above issue ssh from ldapcline1 to ldapclient2 is not happening using kerberos
when kerberos is trying to resolve the host, its getting resolved to `192-168-255-22.ldapclient2.default.svc.cloud.uat`
rather than `ldapclient2.default.svc.cloud.uat`. We have created principle for `ldapclient2.default.svc.cloud.uat`
and not for `192-168-255-22.ldapclient2.default.svc.cloud.uat`.

Add commands mentioned below to resolve the issue.
```shell
kadmin -p root/admin -w admin -q "addprinc -randkey host/192-168-255-22.ldapclient2.default.svc.cloud.uat@DEFAULT.SVC.CLOUD.UAT"
kadmin -p root/admin -w admin -q "xst -k /etc/krb5.keytab host/192-168-255-22.ldapclient2.default.svc.cloud.uat@DEFAULT.SVC.CLOUD.UAT"
```

- To add a new ssh host to kerberos
```shell
kadmin -p root/admin -w admin -q "addprinc -randkey host/192-168-255-22.ldapclient2.default.svc.cloud.uat@DEFAULT.SVC.CLOUD.UAT"
kadmin -p root/admin -w admin -q "xst -k /etc/krb5.keytab host/192-168-255-22.ldapclient2.default.svc.cloud.uat@DEFAULT.SVC.CLOUD.UAT"
```

- To stop ssh service
```shell
sudo service ssh stop
```

- To start ssh service
```shell
sudo service ssh start
```

- Run the SSH server in the foreground, with debugging on:
```shell
sudo service ssh stop
sudo /usr/sbin/sshd -D -ddd
```

- Run the client verbosely
```shell
ssh -vvv smaji@ssh-server
```