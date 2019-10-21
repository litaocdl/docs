
##Install

Mac: 
https://docs.docker.com/docker-for-mac/install/

Desktop vs Tookbox
https://docs.docker.com/docker-for-mac/docker-toolbox/

### Docker storage module

   To support varias linux file system, like ext4,xfs. docker need to use network driver. there are many network driver docker 
   support, like aufs, device mapper and overlay2. 
   
   Docker supports the following storage module
   1. Using host machine volumn 
      like a random host machine volumn is mount to docker `/tmp/myfolder`
      
     `docker run -itd -v /tmp/myfolder -name data-volumn busybox sleep 1000`
    
    2. Using a fix folder in host
     
       `docker run -itd -v /home/litaocdl:/tmp/myfolder ... `
       
    3. Use a volumn container
    
       `docker run -d --volumns-from data-volumn --name busy-box2 sleep 1000`
       
    
### Network module 


 ![](https://github.com/litaocdl/docs/blob/master/pics/docker_network_driver.png)
     

* Bridge
  1. Use docker0 bridge (created by docker daemon), in each container, create a veth (eth0 - veth) from container to docker0 bridge. 
     when docker visit the outside network, will use NAT to change the source or target ip address. when access outside, use SNAT (Source NAT), when outside visit docker, use DNAT (Dest NAT).  by default, outside can visit the docker network, it is controlled by ip forward. set `net.ipv4.ip_forward=1`
      `sysctl -w net.ipv4.ip_forward=1`
      
      ![](https://github.com/litaocdl/docs/blob/master/pics/docker_net_bridge1.png)
      
      NAT to change the target ip address 
      
      ![](https://github.com/litaocdl/docs/blob/master/pics/iptables_dnat.png)
      
      NAT to change the source ip address
      
      ![](https://github.com/litaocdl/docs/blob/master/pics/iptables_snat.png)
  
* Host
  1. Share ip address with host, share port with host, no port mapping. 

      ![](https://github.com/litaocdl/docs/blob/master/pics/docker_net_host.png)
      
* None
  1. No network, Can only use loopback network. handle batch work.
  
* Container
  1. Share network module with another existed container. in k8s, inner one pod, the `pause` container will start first
  ,all the other containers in one pad use `--net=Container:pause` with this pause container. 
  
    ![](https://github.com/litaocdl/docs/blob/master/pics/docker_net_container.png)
    
* Overlay
  latest network module docker created to support NFS. besides the overlay docker created, there are others like flannel, weave and calico etc. 
  Docker is using CNM (container network module) to use those network modules 
  CNM contains:
     * Endpoints
     * Network
     * Sandbox
     
     
     
 ### Create docker registry 
 
 reference: https://docs.docker.com/registry/deploying/
 
   1. Prepare CA certification 
   2. Install docker registry image 
   
   ```
   docker run -d \
  --restart=always \
  --name registry \
  -v "$(pwd)"/certs:/certs \
  -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  -p 443:443 \
  registry:2
   
   ```
   
   

