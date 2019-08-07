## Deploy kubenetes using kubeadm
  
  need swapoff -a to disable swap. 
  [centOS guide](https://www.linuxtechi.com/install-kubernetes-1-7-centos7-rhel7/)
  
  [install weave network](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/)

  when the machine restart, need swapoff -a to disable swap, then api and controller will auto start. 

## Configure kubeconfig for kubectl 
  1. login to master node and extra the kube config 
  kubectl config view --flatten --minify > cluster-cert.txt
cat cluster-cert.txt
  2. copy the kube configure to local
   kubectl --kubeconfig=./kubeconfig get nodes.

## Secrets

Auto generate password using helm function 
`derivePassword` function reference
[create helm chart](https://banzaicloud.com/blog/creating-helm-charts/)
[crypto](http://masterminds.github.io/sprig/crypto.html)

```
{{ .Values.secret.abc.password | default (derivePassword 1 "long" uuidv4 .Release.Name .Chart.Name) | b64enc }}
```
or use uuidv4 directly 
```
{{ .Values.secret.abc.password | default uuidv4 | b64enc }}
```

auto generate secrets values

```
kubectl create secret generic <name> --from-literal=consoleApiKey=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 32) --namespace <your namespace>
```

## Kubectl auth

### use bootstrap token auth

bootstrap token can retrieved using `kubeadm token list`

apiserver need enable `-enable-bootstrap-token-auth=true`
```
curl -H "Authorization: Bearer qpafj9.i2g5lucthim4wkgc" https://9.30.146.114:6443/version -k
```

[bootstrap token reference](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-token/)

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

## Use services account to access k8s

1. create a service account

```
kubectl create serviceaccount api-service-account
```

2. grant privilege for this service account 

```
kubectl create -f https://raw.githubusercontent.com/litaocdl/docs/master/sample/authorization.yaml
```

3. Get token from the services account 

```
kubectl get secrets $(kubectl get serviceaccount api-service-account  -o json | jq -Mr '.secrets[].name') -o json | jq -Mr '.data.token' | base64 -D
```
4. get k8s endpoints 

```
kubectl get endpoints | grep kubernetes
```

5. this token could be used to access k8s api

```
curl -k  https://<ip>/api/v1/namespaces -H "Authorization: Bearer $token"
```

## ClusterIP vs NodePort vs LoadBalancer vs Ingress

A services yaml

```
apiVersion: v1
kind: Service
Spec:
 ports:
   - name:
     port:        # the port on clusterIP
     targetPort:  # the target pod ports to map, the routing will finally reach this port inner the pod.
     nodePort:    # the port on node ip, if services type is NodePort, use this value to access the services from outside
 type: NodePort/ClusterIP/LoadBalancer
```

`ClusterIP` and `NodePort` `LoadBalancer` are all services type. Ingress is not a type of service

`ClusterIP` means the services is exposed to cluster internal ip. To reach the ClusterIp from an external source, you can open a Kubernetes proxy between the external source and the cluster. This is usually only used for development.

`NodePort` means services exposed to each node level which can be access from external using <nodeIP>:<nodePort>
  
`LoadBalancer` means services exposed to public cloud load balance in external. using a cloud provider’s load balancer solution. and load balance will map to automatically created nodePort.

For both <clusterip>:<port> or <nodeip>:<nodeport> access, the kube-proxy will create a access rule in iptables to reroute the access. 
  
`Ingress` Ingress acts like reverse proxy, which exposed it self as nodeport, clusterip or loadbalancer, and redirect the rule based on the configuration rules to clusterips. 

Reference [clusterip vs nodeport and ingress](https://www.ovh.com/blog/getting-external-traffic-into-kubernetes-clusterip-nodeport-loadbalancer-and-ingress/)
