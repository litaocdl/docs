1. Deploy kubenetes using kubeadm
  need swapoff -a to disable swap. 
  https://www.linuxtechi.com/install-kubernetes-1-7-centos7-rhel7/
  
  install weave network https://www.weave.works/docs/net/latest/kubernetes/kube-addon/


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

`swapoff -a`
`systemctl start docker`
`systemctl start kubelet`
