

### LDAP commands

[ldap command reference](https://www.digitalocean.com/community/tutorials/how-to-manage-and-use-ldap-servers-with-openldap-utilities)

#### install ldap-utils

ubuntu or debian
```
 apt-get install ldap-utils
```

centos or Fedora
```
yum install openldap-clients
```
#### Search for entries specific by `-b`, binding using `-D` and `-w`
```
ldapsearch -x -LLL -D cn=xx,dc=xxx,dc=com -w xxx -H ldap://<ip> -b ou=People,dc=xxx,dc=com
```

#### Update entry password

use `-D` and `-w` to update the `-S` password to `-s`
```
ldappasswd -x  -H ldap://<ip> -D cn=xxx,dc=xxx,dc=com -w xxx -S uid=xxx,ou=People,dc=xxx,dc=com -s yyy

```

#### verify ldappasswd using ldapwhoami

`-d` is used to enable debug mode. 
```
ldapwhoami -x -D uid=xxx,ou=People,dc=xxx,dc=com -w xxx -H ldap://<ip> -d 1
```

#### use `ldapurl` to query

ldapurl is similar with `curl -B ldap://`

format

```
ldapurl ldap://host:port/base_dn?attr_to_return?search_scope?filter?extension
```

#### add ldap entry

1. use `ldapadd`

create ldif file contains the entry information, we need add all required attributes for objectClass
```
dn: uid=xxx,ou=People,dc=xxx,dc=com
uid: litaocdl4
gecos: litaocdl4
uidNumber: 5008
gidNumber: 3000
name: litaocdl4@xxx
homeDirectory: /home/litaocdl4
cn: litaocdl4
objectClass: account
objectClass: posixAccount
objectClass: top
objectClass: extensibleObject
loginShell: /bin/bash
email: litaocdl4@xxx

```

2. execute `ldapadd` to add the entry

`-n` could as dry-run.
```
ldapadd -H ldap://<ip> -D cn=xxx,dc=xxx,dc=com -w xxx -f ./newuser.ldif

```

or Use ldapModify to add.

`ldapmodify -a` == `ldapadd`

```
ldapmodify -a -H ldap://<ip> -D cn=xxx,dc=xxx,dc=com -w xxx -f ./newuser.ldif

```
3. use ldappasswd to set password 

```
ldappasswd -x -H ldap://<ip> -D cn=bluldap,dc=xxx,dc=com -w xxx -S uid=litaocdl4,ou=People,dc=xxx,dc=com -s password

```

and verify the result

```
ldapwhoami -x -H ldap://<ip> -D uid=litaocdl4,ou=People,dc=xxx,dc=com -w password
```

#### modify ldap entries

use ldapmodify can modify ldap entries, we need create ldif file with changetype

multiple action can separated by `-`
```
dn: uid=litaocdl5,ou=People,dc=xxx,dc=com
changetype: modify
add: description
description: addition
-
delete: email
-
replace: name
name: newname
```
use ldap modify to update
```
ldapmodify  -H ldap://<ip> -D cn=bluldap,dc=xxx,dc=com -w xxx -f ./change.ldif
```

#### delete ldap entries 

delete the entry `uid=litaocdl5,ou=People,dc=xxx,dc=com`

```
ldapdelete -H ldap://<ip> -D cn=bluldap,dc=xxx,dc=com -w xxx uid=litaocdl5,ou=People,dc=xxx,dc=com

```

### ldapi

ldapi means ldap over ipc, if the ldap client and ldap server are in the same machine, this could improve the performance. 

```
ldapsearch -H ldapi:// -D cn=bluldap,dc=blustratus,dc=com -w NGZjNmJhZWY1NzY0 -b ou=People,dc=blustratus,dc=com

```

### SASL

`SASL` is short for Simple Authentication Security Layer. `EXTERNAL` is one of mechanism SASL supports, which relay on TLS.
`ldapsearch` and `ldapmodify` support SASL.

```
ldapsearch -H ldapi:// -Y EXTERNAL -b ou=People,dc=blustratus,dc=com
```

query the ldap command using SASL
```
ldapsearch -H ldapi:// -Y EXTERNAL -b "cn=config" -LLL -Q "olcDatabase=*"
```
