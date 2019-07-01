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
