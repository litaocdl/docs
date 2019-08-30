
##Install

Mac: 
https://docs.docker.com/docker-for-mac/install/

Desktop vs Tookbox
https://docs.docker.com/docker-for-mac/docker-toolbox/

### Docker storage module

### Network module 


* Bridge
  1. Use docker0 bridge (created by docker daemon), in each container, create a veth (eth0 - veth) from container to docker0 bridge. 
  
* Host
  1. Share ip address with host, share port with host, no port mapping. 

* None
  1. No network, Can only use loopback network. handle batch work.
  
* Container
  1. Share network module with another existed container. in k8s, inner one pod, the `pause` container will start first
  ,all the other containers in one pad use `--net=Container:pause` with this pause container. 
  
* Overlay
  latest network module docker created to support NFS. besides the overlay docker created, there are others like flannel, weave and calico etc. 
  Docker is using CNM (container network module) to use those network modules 
  CNM contains:
     * Endpoints
     * Network
     * Sandbox
