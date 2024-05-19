# Ldap

- Please refer links for more commands. [link](http://techpubs.spinlocksolutions.com/dklar/ldap.html),
[link](https://www.lisenet.com/2014/install-and-configure-an-openldap-server-with-ssl-on-debian-wheezy/),

- Search for user in ldap using admin
```shell
ldapsearch -x -b "ou=users,dc=default,dc=svc,dc=cloud,dc=uat" "cn=smaji" -D "cn=admin,dc=default,dc=svc,dc=cloud,dc=uat" -w sumit -H ldap://ldap.default.svc.cloud.uat -LLL
```

- Add user to ldap
```shell
uid=$(< /var/userid)
export UNAME=smaji
export UPASSWORD=sumit
export BASE_DN=dc=default,dc=svc,dc=cloud,dc=uat
export LDAP_PASSWORD=sumit
export LDAP_HOSTNAME=ldap://ldap.default.svc.cloud.uat
gid=`ldapsearch -x -b "ou=groups,$BASE_DN" "cn=$2" -D "cn=admin,$BASE_DN" -w ${LDAP_PASSWORD} -H ${LDAP_HOSTNAME} -LLL gidNumber | grep 'gidNumber' | grep -Eo '[0-9]+'`
cat <<EOF > alice.ldif
dn: cn=$1,ou=users,$BASE_DN
cn: $UNAME
gidnumber: $gid
givenname: Sumit
homedirectory: /home/users/$UNAME
loginshell: /bin/bash
objectclass: inetOrgPerson
objectclass: posixAccount
objectclass: top
sn: hadoop
uid: smaji
uidnumber: $uid
userpassword: $UPASSWORD
EOF
echo $(($uid + 1)) > /var/userid
```

- Creating a group
```shell
uid=$(< /var/userid)
gid=$(< /var/groupid)

LDAP_PASSWORD:=sumit
BASE_DN:=dc=default,dc=svc,dc=cloud,dc=uat
GROUP_NAME=developers

echo "dn: cn=$GROUP_NAME,ou=groups,$BASE_DN
cn: $GROUP_NAME
gidnumber: $gid
objectclass: posixGroup
objectclass: top
" > /var/tmp/groups.ldif

ldapadd -x -D "cn=admin,$BASE_DN" -w ${LDAP_PASSWORD} -H ldapi:/// -f /var/tmp/groups.ldif
```


```shell
ldapsearch -x -b "dc=cloud,dc=com" -H ldap://ldap.cloud.com -LL
ldapdelete "cn=kdc-srv,ou=krb5,dc=cloud,dc=com" -x -D 'cn=admin,dc=cloud,dc=com' -w sumit -H ldapi:///
ldapsearch -LLLQY EXTERNAL -H ldapi:/// -b cn=config "(|(cn=config)(olcDatabase={1}hdb))"
ldapmodify -QY EXTERNAL -H ldapi:/// -f ~/olc-mod1.ldif
ldapsearch -x -H ldap:/// -LLL -D 'cn=admin,dc=default,dc=svc,dc=cloud,dc=uat' -w sumit -b "cn=schema,cn=config" "(objectClass=olcSchemaConfig)" dn -Z 
ldapsearch -x -H ldap:/// -L -D 'cn=admin,dc=default,dc=svc,dc=cloud,dc=uat' -w sumit -b "ou=users,dc=default,dc=svc,dc=cloud,dc=uat" "(uid=smaji)" dn -Z 
ldapsearch -x -H ldap://ldap.default.svc.cloud.uat -D 'cn=admin,dc=default,dc=svc,dc=cloud,dc=uat' -w sumit -b "ou=users,dc=default,dc=svc,dc=cloud,dc=uat" "(uid=smaji)"
```

- Run the following command to test if the OpenLDAP server is actually running:
```shell
nmap -p 389 localhost
```

- Perform a quick test by generating an LDAP Data Interchange Format (LDIF) dump of the contents of a the database:
```shell
slapcat
```

- To see all the schema
```shell
ldapsearch -LLLQY EXTERNAL -H ldapi:/// -b cn=config
```

- To filter a specific schema
```shell
ldapsearch -LLLQY EXTERNAL -H ldapi:/// -b cn=config cn={5}kerberos
or 
ldapsearch -LLLQY EXTERNAL -H ldapi:/// -b cn=config olcDatabase={1}mdb
or
ldapsearch -LLLQY EXTERNAL -H ldapi:/// -b cn=config "(|(cn=config)(olcDatabase={1}mdb))"
```

- To search for user under `ou=users,dc=cloud,dc=com`
```shell
ldapsearch -LLLQY EXTERNAL -H ldapi:/// -b "ou=users,dc=cloud,dc=com" "cn=smaji"
```

- To seach for specific attribute
```shell
ldapsearch -LLLQY EXTERNAL -H ldapi:/// -b "ou=users,dc=cloud,dc=com" "cn=smaji" givenName
```

- Add/Delete attribute of user
```shelll
# mail attribute is part of inetOrgPerson objectclass.
ldapmodify -x -D "cn=admin,dc=cloud,dc=com" -w sumit -H ldapi:/// <<EOF
dn: cn=smaji,ou=users,dc=cloud,dc=com
changetype: modify
add: mail
mail: skmaji@outlook.com
EOF

ldapmodify -x -D "cn=admin,dc=cloud,dc=com" -w sumit -H ldapi:/// <<EOF
dn: cn=smaji,ou=users,dc=cloud,dc=com
changetype: modify
delete: mail
EOF
```

- Search for an attribute
```shell
ldapsearch -LLLQY EXTERNAL -H ldapi:/// -b "cn=config" | less
```
