## Useful curl command samples 


### use customer ca file to verify https 

instead of `-k` options to ignore the security the public key verification, we can use customer CA file `--cacert` to communicates 
with https for self-signed ca, authenticate through basic authentication. 

```
 curl https://<ip>:6443/version -H "Authorization: Bearer qpafj9.i2g5lucthim4wkgc"  --cacert ./ca.cert
```

authenticate through client authenticate 

for client authentication, the server side deployed the server certification, like server.cert and server.key. the server.cert and client.cert need be sign using the same CA. `ca.cert`

```
curl https://9.30.146.114:6443/version   --cert ./client.cert --key ./client.key --cacert ./ca.cert

```

### use curl to query ldap

format see [reference](https://docs.oracle.com/cd/E19396-01/817-7616/ldurl.html)

<base dn>: start from which points to query
<attributes>: what attributes to query
<scope>: query scope base/one/sub
<filter>: default objectClass=*
 
 
```

curl -B "ldap://<ip>/<base dn>?<attributes>?<scope>/<filter>" -u "<bind DN>:<bind passwd>"

```
example 

```
curl -B "ldap://ip/dc=xxx,dc=com?*?sub?uid=xxx" -u "cn=xxx,dc=xxx,dc=com:xxx"

```

