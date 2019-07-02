## useful curl command samples 


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


