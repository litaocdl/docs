##
## Overlay network

   Overlay network is over the native network and make the connection between native network accessible. 
## CNI
CNI: container network interface, which is a lightweight network stands supported by k8s, mesos etc. there are multiple impl 
CNI plugin can choose, like flannel, calico, weave, ovs(open vSwitch) etc
[Comparasion](https://rancher.com/blog/2019/2019-03-21-comparing-kubernetes-cni-providers-flannel-calico-canal-and-weave/)

CNI is also a kind of overlay but make sure pod can be accessed by each other across node. 

## CNM
the docker standards network model, container network model, which is provided by docker company. using network sandbox, 
endpoint and network to build the network across containers. 
