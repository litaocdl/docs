## Deploy kubenetes using kubeadm
  
  need swapoff -a to disable swap. 
  [centOS guide](https://www.linuxtechi.com/install-kubernetes-1-7-centos7-rhel7/)
  
  [install weave network](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/)


## Configure kubeconfig for kubectl 
  1. login to master node and extra the kube config 
  kubectl config view --flatten --minify > cluster-cert.txt
cat cluster-cert.txt
  2. copy the kube configure to local
   kubectl --kubeconfig=./kubeconfig get nodes.


## Kubectl auth

### use bootstrap token auth

bootstrap token can retrieved using `kubeadm token list`

apiserver need enable `-enable-bootstrap-token-auth=true`
```
curl -H "Authorization: Bearer qpafj9.i2g5lucthim4wkgc" https://9.30.146.114:6443/version -k
```


## logs

check kubelet log
`journalctl -u kubelet`


## restart

both master and work nodes
```
swapoff -a
systemctl start docker
systemctl start kubelet
```


## Generate x509 certificate for k8s 
## Use openssl to generate self certification for x509 client authentication. 
1. Generate private key for CA, ca.key

```
openssl genrsa -out ca.key 2048
```

2. based on the ca.key to generate the x509 ca certificate 

`master_ip` is the ip address this ca.cert will locates. 

```
openssl req -x509 -new -nodes -key ca.key -subj "/CN=${master_ip}" -days 1000 -out ca.crt 
```

3. Generate the server private key server.key

```
openssl genrsa -out server.key 2048
```

4. Create configuration file `csr.conf` to generate csr (certification signing request).

```
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
req_extensions = req_ext
distinguished_name = dn

[ dn ]
C = <country>
ST = <state>
L = <city>
O = <organization>
OU = <organization unit>
CN = <MASTER_IP>

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster
DNS.5 = kubernetes.default.svc.cluster.local
IP.1 = <MASTER_IP>
IP.2 = <MASTER_CLUSTER_IP>

[ v3_ext ]
authorityKeyIdentifier=keyid,issuer:always
basicConstraints=CA:FALSE
keyUsage=keyEncipherment,dataEncipherment
extendedKeyUsage=serverAuth,clientAuth
subjectAltName=@alt_names

```

5. Generate teh certificate signing request based on the csr.conf

```
openssl req -new -key server.key -out server.csr -config csr.conf
```
6. Using server.csr to generate the certificate. server.crt is based on ca.key, ca.crt and server.csr

```
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key \
-CAcreateserial -out server.crt -days 10000 \
-extensions v3_ext -extfile csr.conf
```
