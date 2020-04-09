

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
#### Search for entries specific by `-b`, binding using `-D` and `-w` (-W to input password from standard input)
```
ldapsearch -x -LLL -D cn=xx,dc=xxx,dc=com -w xxx -H ldap://<ip> -b ou=People,dc=xxx,dc=com
```

`-LLL` let talk about it

 Search results are display in LDAP Data Interchange Format  detailed  in ldif(5).   
 A  single  -L restricts the output to LDIFv1.
 A second -L disables comments.  
 A third -L disables printing of the LDIF version.  
 The default is to use an extended version of LDIF.
 
default: see extended LDIF
 
```
bash-4.2$ ldapsearch -H ldapi://   -s base -b "" supportedSASLMechanisms
SASL/EXTERNAL authentication started
SASL username: gidNumber=55+uidNumber=55,cn=peercred,cn=external,cn=auth
SASL SSF: 0
# extended LDIF
#
# LDAPv3
# base <> with scope baseObject
# filter: (objectclass=*)
# requesting: supportedSASLMechanisms
#

#
dn:
supportedSASLMechanisms: EXTERNAL

```

`-L`  see version 1

```
bash-4.2$ ldapsearch -H ldapi://  -L  -s base -b "" supportedSASLMechanisms
SASL/EXTERNAL authentication started
SASL username: gidNumber=55+uidNumber=55,cn=peercred,cn=external,cn=auth
SASL SSF: 0
version: 1

#
# LDAPv3
# base <> with scope baseObject
# filter: (objectclass=*)
# requesting: supportedSASLMechanisms
#

#
dn:
supportedSASLMechanisms: EXTERNAL
```

`-LL`: disable comments

```
bash-4.2$ ldapsearch -H ldapi://  -LL  -s base -b "" supportedSASLMechanisms
SASL/EXTERNAL authentication started
SASL username: gidNumber=55+uidNumber=55,cn=peercred,cn=external,cn=auth
SASL SSF: 0
version: 1

dn:
supportedSASLMechanisms: EXTERNAL

```

`-LLL` disable the version print 

```
bash-4.2$ ldapsearch -H ldapi://  -LLL  -s base -b "" supportedSASLMechanisms
SASL/EXTERNAL authentication started
SASL username: gidNumber=55+uidNumber=55,cn=peercred,cn=external,cn=auth
SASL SSF: 0
dn:
supportedSASLMechanisms: EXTERNAL
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

`-Y EXTERNAL` means use EXTERNAL SASL mechanism, if -Y is not specificed, will search one mechanism

to get all supported Mechanism

```
ldapsearch -H ldapi://  -LLL -s base -b "" supportedSASLMechanisms
```

Since the ldapi scheme requires a local connection, we never will have to specify a server name here.

### LDAP Authentication 
LDAP need client identify them self so sever can grant level of access, there are three common kinds of authentication, this is called binding. 

#### anonymous binding

use `-x`, as most ldap server enable the SASL authentication, use -x means we do not want to use SASL

#### Simple Authentication
A simple bind uses an entry within the LDAP server to authenticate the request. The DN (distinguished name) of the entry functions as a username for the authentication. Inside of the entry, an attribute defines a password which must be provided during the request.

use `-D -w/W`

#### SASL authentication 

SASL stands for simple authentication and security layer. It is a framework for hooking up authentication methods with protocols in order to provide a flexible authentication system that is not tied to a specific implementation. 

SASL authentication provides various mechanism, to find all the mechanism we can issue 

```
ldapsearch -H ldap:// -x -LLL -s base -b "" supportedSASLMechanisms
```

one common SASL mechanism is EXTERNAL

```
The EXTERNAL mechanism indicates that authentication and security is handled by some other means associated with the connection. For instance, it can be used with SSL to provide encryption and authentication.
```


### LDAP server 

[reference for ubnuntu](https://help.ubuntu.com/lts/serverguide/openldap-server.html)

install open ldap `sudo apt install slapd`

reconfig the basedn and rootdn

```
sudo dpkg-reconfigure slapd
```

after ubuntu 18, the slapd supports dynamically configure the ldap without restart the server, the ldap is configure by slapd itself using ldif. the configuraiton file is under `/etc/ldap/slapd.d/` folder in DIT format.

Get all slapd-config database 

```
ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config dn
===
Result and explain
cn=config: global settings

cn=module{0},cn=config: a dynamically loaded module

cn=schema,cn=config: contains hard-coded system-level schema

cn={0}core,cn=schema,cn=config: the hard-coded core schema

cn={1}cosine,cn=schema,cn=config: the cosine schema

cn={2}nis,cn=schema,cn=config: the nis schema

cn={3}inetorgperson,cn=schema,cn=config: the inetorgperson schema

olcBackend={0}mdb,cn=config: the 'mdb' backend storage type

olcDatabase={-1}frontend,cn=config: frontend database, default settings for other databases

olcDatabase={0}config,cn=config: slapd configuration database (cn=config)

olcDatabase={1}mdb,cn=config: your database instance (dc=example,dc=com)

```


Get the base dn and root dn 

```
ldapsearch -x -LLL -H ldap:/// -b dc=fyre,dc=ibm,dc=com dn
===
dn: dc=fyre,dc=ibm,dc=com : base dn

dn: cn=admin,dc=fyre,dc=ibm,dc=com : root dn

# if there are entries, it is listed here. 
```


