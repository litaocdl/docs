## useful curl command samples 


### use customer ca file to verify https 

instead of `-k` options to ignore the security the public key verification, we can use customer CA file `--cacert` to communicates 
with https for self-signed ca.

```
 curl https://<ip>:6443/version -H "Authorization: Bearer qpafj9.i2g5lucthim4wkgc"  --cacert ./ca.cert
```
