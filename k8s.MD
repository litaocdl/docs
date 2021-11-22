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

---

## Kubernetes overview 

### Structure 

  ![](https://github.com/litaocdl/docs/blob/master/pics/k8s_stuct.png)
  
  Master Node 3 key components:
  
  * API Server: statefulless, heart. can have multi api-server if there is multiple master node. 
  * Controller Manager: statefulless, manage pod replication. has various controller, like node controller, endpoint controller, namespace controller etc.
  * Scheduler: statefulless, pod scheduler, decide the pod scheduled to which nodes. 
  * ETCD: storage, only work with API Server

  Worker:
  
  * kubelet: running in worker node. register itself to master node. collects worker node status to master node.
  * kube-proxy: redirect request to pod for services, use Round Robin algorithm. 
  * Container Runtime: docker, etc
  
  Addons
  
  * CoreDNS: Service Discovery in k8s, available since k8s 1.11. https://coredns.io/
      KubeDNS and CoreDNS are used for k8s services discovery, they are mapping the services/pod name with to the correct ip address. 
      [KubeDNS vs CoreDNS](https://www.digitalocean.com/community/tutorials/an-introduction-to-the-kubernetes-dns-service)

      debug coreDNS
        * kubectl create -f https://raw.githubusercontent.com/litaocdl/docs/master/template/k8s/nslookup.yaml
        * check dns `kubectl exec -it busybox -- nslookup <servicesname>`

  * Web UI (Dashboard):
  * CNI: Container network interface. 
      [More about CNI](https://github.com/litaocdl/docs/blob/master/CNI.md)


### How k8s create a pod 

  ![](https://github.com/litaocdl/docs/blob/master/pics/k8s_create_pod.png)

---

## K8s Objects 

Check the object spec `kubectl explain pod.spec`

### Secrets

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

secrets type: `Opaque` means secrets is unstructured, could contains any kinds of key/value. in contrast, there are secrets types which k8s understanding. 


### POD

* static pod: controlled by kubelet, not apiserver. Running in a node for long time. kubelet scan the `--pod-manifest-path` and create the static pods. `type=pod`


* Pod life cycle 

Pending --> Running (at least one container is in running status) --> Succeed (all container stopped and not restart)
                    --> Failed (at least one container failed)
                    --> Unknown


#### Expose pod services to external from pod level

pod definitation as below, using `hostNetwork` or `hostPort`

```
spec:
 hostNetwork: true
 containers: 
 - name: sample
   ports:
    - containerPort: 8080
      hostPort: 18080

```

With `hostNetwork=true`, the all the port in the pod will map to the node network. if `hostPort` is specific, must equals to `containerPort`.
hostNetwork means docker is using `--net=host`. 

With `hostNetwork=false` (default), we can set the ports level `containerPort` and `hostPort`, to map one port to the node network. `hostPort`
is the actually port in host.  

Using `<nodeip>:<hostPort>` to access the pod services. make sure check which node the pod is running and using that specific node ip address. 

  * stop pod 
   `--grace-period` the pod will be delete anyway in 60 seconds, default 30 seconds. 
   kubectl delete deployment test --grace-period=60

#### Prob in pod.
  
  if `readness` failed, server offline. if `liveness` failed, restart container. 

  * Liveness: container level health check. Based on restartPolicy (Always `default` (container exit),OnFailure (container exit not 0),Never) to restart container. when container in exit status, kubelet will restart the container in a delay way, 10s, 20s, 40s ... 300s ? when container in running status
  longer than 10 minutes, reset the delay value. 
  * Readness: service level health check. remove the service from load balance. 

  three type:
  
  `ExecAction` (check if shell return 0), `TCPSocketAction` (check if port is open), `HTTPGetAction` (check response code between [200,400))
  


  Best Practice: liveness and readness use same way, and readness set longer internal than liveness. 

#### Deploy Pod

   * Using Job. for the pod expected to stop.
   * Using `Deployment`, ReplicaSet, ReplicationController. for the pod not expected stop.
   * Using DaemonSet

#### Init container

   Handle the dependency of service/pod. if multi `initContainers` exists, will execute in sequence per defined in yaml. `initContainers` execute before services container starts. if the `initContainers` failed, the pods will failed. 


#### Hooks provided for pod lifecycle

  * postStart: execute after the pod starts. but we are not sure which one is first compared with `ENTRYPOINT`
  * preStop:   execute before pod stop. but pod will still stop if the pod `terminationGracePeriodSeconds` is reach. 


#### Label 

  Label could be applied to any object in k8s, like deployment, pod, node, service etc. 
   `kubectl get all --show-labels`
   `kubectl get pod -l a=b`
  * Label selector 
  1. equal based.  `matchLabels`
   
  2. expression based. `matchExpressions`

   ```
     selector:
       matchLabels:
        - app: nginx
        - name: tao
       matchExpressions
         {key: tier, operator: in,values: [cache]}
   ```


#### RC and RS and Deployment. 

   RS support `expression based label selector` while RC does not. this is the only difference.
   `Deployments` (k8s v1.2) wrap the RS, use deployment, we can see the pod deployment progress. and deployment support rollback,stop and resume.

  pause deployment
  `kubectl rollout pause`
  resume deployment
  `kubectl rollout resume`
  check upgrade status
  `kubectl rollout status`
  check upgrade history, to see the history, need add `--record=true`
  `kubectl rollout history <> --to-revision=2`
  rollback
  `kubectl rollout undo <> --to-revision=2`
  scale up
  `kubectl scale <> --replicas=10`
  auto scale with hpa
  `kubectl autoscale <> --min=10 --max=15 --cpu-percent=80`

   Relationship: Deployment --> Replica Sets --> Pods
   Naming: <deployment> --> <deployment>-<random number> --> <deployment>-<random number>-<random string>


## Service, Deployment, Pod
  
  [reference](https://www.cnblogs.com/rancherlabs/p/14860697.html)
  
  ![image](https://img-blog.csdnimg.cn/20210607214602153.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JhbmNoZXJMYWJz,size_16,color_FFFFFF,t_70)

  
  
### Services - ClusterIP vs NodePort vs LoadBalancer vs Ingress

  ![](https://github.com/litaocdl/docs/blob/master/pics/k8s_services.png)

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

when a services is created, endpoints will be created for each pod and each port in the pod. 

`ClusterIP` and `NodePort` `LoadBalancer` are all services type. Ingress is not a type of service

`ClusterIP` means the services is exposed to cluster internal ip. To reach the ClusterIp from an external source, you can open a Kubernetes proxy between the external source and the cluster. This is usually only used for development. we can create dns mapping for services name and clusterip to solve the service discovery. 

`NodePort` means services exposed to each node level which can be access from external using `<nodeIP>:<nodePort>`
  
  ![image](https://user-images.githubusercontent.com/42050086/131457450-3d08e72e-8292-4476-9913-22b2104606e0.png)

  
`LoadBalancer` means services exposed to public cloud load balance in external. using a cloud provider’s load balancer solution. and load balance will map to automatically created nodePort.
  
   Expensive: One load Balancer per service

  ![image](https://user-images.githubusercontent.com/42050086/131457603-d2bb8d50-c3f7-436a-a4db-72d448349dfd.png)

  
For both `<clusterip>:<port>` or `<nodeip>:<nodeport>` access, the kube-proxy will create a access rule in iptables to reroute the access. 
  
`Ingress` Ingress acts like reverse proxy, which exposed it self as nodeport, clusterip or loadbalancer, and redirect the rule based on the configuration rules to clusterips. 

  Cheap: One load balance for multiple services
  
  ![image](https://user-images.githubusercontent.com/42050086/131457768-24c1f576-06a3-4fc2-ac69-f5540d55b3a0.png)

Reference [clusterip vs nodeport and ingress](https://www.ovh.com/blog/getting-external-traffic-into-kubernetes-clusterip-nodeport-loadbalancer-and-ingress/)
  

How services routed to pod, iptables vs ipvs

![](https://github.com/litaocdl/docs/blob/master/pics/services-proxy-mode.png)


Services Discovery

* clusterip: use iptables route
* NodePort: use iptables route
* LoadBalancer: different cloud provider use different impl. will auto assign an external ip. tencent use iptables.
* Ingress nginx/traefik, 7 layer
* Istio: Ingress gateway, 7 layer
* hostnetwork: `hostNetWork`docker use hostnetwork, use another services to register and manage the port. serivces is mapped to port. 

Expose service

Service could be exposed on deployment, pod , service, rc, rs level 

```
kubectl -n xxx expose [pod/deployment/service/rc/rs] xxx --name=<service name>. --port=<port service will expose>. --target-port=<port service port will redirect to> --protocol=<tcp or udp, tcp is default> --type=[ClusterIP(default), NodePort, or LoadBalancer] --external-ip=<external ip which could routed to the svc>
```

if `--port` is not specific, will use the default port the resource exposed. 

Using `ClusterIP` can also expose a port on `a` node.  but `NodePort` will expose port on all nodes.
`kubectl -n xxx expose deployment xxx --name=xxx --type=ClusterIP --external-ip=<a node ip>`

`Expose Logic`

`ClusterIP` will expose `spec.clusterIp`:`spec.ports[*].port`, so service could only be accessed from virtual clusterip and port. clusterIP svc looks like below. if externalIP is set `--external-ip=xxxx`, then `ClusterIP` will expose
`spec.clusterIp`:`spec.ports[*].port` and `spec.externalIPs[*]`:`spec.ports[*].port`

```
NAME                     TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)          AGE
my-service-clusterip     ClusterIP      10.111.204.192   <none>          8080/TCP         41m
```

`NodePort` will expose the following:

`<NodeIP>`:`spec.ports[*].nodePort`  exposed on all nodes ip with static port `nodePort`.     
`spec.clusterIp`:`spec.ports[*].port` exposed on clusterIP with port

when request comes to `NodeIP:nodePort`, request will be redirect to `clusterIP:port` and then redirect to `targetPort`
NodePort looks like this, when create NodePort, nodeport can not be assigned, which will generated automatically.  

```
NAME                     TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)  (port/nodePort)        AGE
my-service-nodeport2     NodePort       10.104.140.15    <none>          8080:38717/TCP                  15m
```

`LoadBalancer` will expose the following:  
`spec.loadBalancerIp`:`spec.ports[*].port` expose the `port` to balanceIP.       
`<NodeIP>`:`spec.ports[*].nodePort`     expose the nodePort to node ips. 
`spec.clusterIp`:`spec.ports[*].port`    expose the port on clusterIP. 

we can access the `loadBalanceIP:port` , request will routed to `nodeIP:nodePort`, then to `clusterIP:port`, then to `targetPort`. 

LoadBalancer looks like:

```
NAME                     TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)  (port/nodePort)        AGE
my-service               LoadBalancer   10.100.230.21    <pending>       8080:4123/TCP                   55m

```

A diagram from web for those three

![](https://i.stack.imgur.com/1owA5.jpg)

### ingress

![](https://github.com/litaocdl/docs/blob/master/pics/ingress.png)



  
### Port forwarding
  kubectl port-forward [service/pod/deployment/rs]. local:remote
  kubectl port-forward service/xxx 5454:5432 
  
  localhost:5454:service/xxx:5432

### Job

   type=Job, restartPolicy=Never

### ConfigMap

`kubectl create configmap <> --from-file=<>` # key is filename, value is filecontent
`kubectl create configmap <> --from-env-file=<>` key/value are defined in file

 Reference config map in Pod. 
 1. Use as env , when configure map is recreate, the env will not update. 
 2. Use as volumn. when configure map is recreate, the volumn will update.

Etcd has size limitation (1M) for the values under one entry. so the configure map has a size limitation. 

### DaemonSet

`DaemonSet`. similar with Deployment, for deploy a list of pod, but makesure there is one pod running in each node. and make sure the pod can not be delete. new node added to cluster will auto create the DaemonSet pod. 

 Classic scenario: need a process run in every node for storage, log collecting, monitoring etc. different with static pod
, DaemonSet is controlled by apiserver and kubectl.

`kubectl delete daemonSet <> --cascade=false`  
`--cascade=false` only delete daemonSet, not delete pod. default will delete all pod under this DaemonSet. 

### StatefulSet

   Can ensure the pod deploy in sequence and give each pod a unique pod name. 

### Secret 

### Service Account
### Annotation

   User defined message. 


### Label
  add label
  `kubectl label node <> key=value`
  remove label
  `kubectl label node <> key-`
  
  
  
### Operators   
  [reference](https://www.cnblogs.com/rancherlabs/p/13595060.html?share_token=23C0D1D5-B6BA-4633-AC25-7277AA769A12&tt_from=weixin&utm_source=weixin&utm_medium=toutiao_ios&utm_campaign=client_share&wxshare_count=1)
  
  
  Operators is a special controller which extends the k8s api and hook the change of a series of CR, and based on the changes of the CR and do some actions. 
  
  `kube-controller-manager` could be recognized as the 'first' operator and all other operator is created after that. 
  
  ![image](https://user-images.githubusercontent.com/42050086/131469028-1b46480c-256e-482b-aa02-5e640200fab9.png)

  To create a operator 
  
  1. create the CRD
  2. impl the Resource Controlled CRD points to
  3. create CR for the CRD.
  4. debug. 
  
  reference: https://www.qikqiak.com/post/k8s-operator-101/
  
 two ways to add CR
  1. use crd
  2. use AA (Aggregation api)
 
#### CRD 
  
  The steps to create a CRD 
  
  1. create the CRD definitation. the crd definiation may includes following
     a. if using openAPI/v3, include the validation.
     b. define the crd name, names, shortname, type, category, api endpoints etc. 
     c. use kubectl apply xxx.yaml to create the crd.
  2. according to the crd definiation, create the cr. 
     a. cr follow the crd defined endpoint, name,type etc. 
     b. use kubectl apply xxx.yaml
  
  
  
---
---

## K8s QoS

Classification by priority 

  * Guaranteed: ensure. first priority. limits=request or only has limits.  m=1/1000 cpu

     ```
       resources:
         # limits is the resource limitation 
         limits:
           cpu: 10m
           memory: 1Gi
         # requests is how much we request. 
          requests:
           cpu: 10m
           memory: 1Gi
     ```
  * Burstable: 
    Set resource but not the above. 
  * Best-Effort: share resource, try best. lowest priority. 
    Not set resource

  Kubelet resource recyle policy. if no available resource (memory,not cpu), will first kill Best_Effort, then Burstable then Guaranteed, to make sure kubelet can 
  run successfully. 

 Classification by resource type

  CPU: if cpu usage in pod exceed the limits, cpu resource will reduced in pod.

  Memory: if memory used in pod exceed the limits, pod will be killed. 

---

## k8s scheduler 

   scheduler the pod running in a certain node. 
   Two phase 
   1. pre-select. Predicates 
      filter out the un-satisfied node. 
      request(pod) = Max(Max(request(initContainer1),...), SUM(request(container)))
      success = request(pod) < node.allocatable - node.requested

      ```
      kubectl describe node

      Capacity:
       cpu:                2
       ephemeral-storage:  244198912Ki
       hugepages-1Gi:      0
       hugepages-2Mi:      0
       memory:             8009304Ki
       pods:               110
      Allocatable:
       cpu:                2
       ephemeral-storage:  225053716927
       hugepages-1Gi:      0
       hugepages-2Mi:      0
       memory:             7906904Ki
       pods:               110
      ```

   2. best-select. Priority 
      find out the best node to schedule. 

      a. `nodeSelector`  exactly match the node label

        ```
           nodeSelector:
              diskType: ssd
              key: value
        ```

      b. `nodeAffinity` upgrade version of nodeSelector. check the node and used for pod looking for node.
          * Support in, NotIn. support enum of label, like zone in [label1,label2...]
          * Support required(hard condition) and preferred(soft condition) condition 
             `requiredDuringSchedulingIgnoreDuringException`: must match, but after pod deployed if condition change, pod will not removed from node.
              multi condition is or.
             `preferredDuringSchedulingIgnoreDuringException`: better to match, but after pod deployed if condition change, pod will not removed from node.
      c. `podAffinity` `podAntiAffinity` checking the existed pod running in node used for pod looking for node.
             `requiredDuringSchedulingIgnoreDuringException`: must match, but after pod deployed if condition change, pod will not removed from node.
             multi condition is and.
             `preferredDuringSchedulingIgnoreDuringException`: better to match, but after pod deployed if condition change, pod will not removed from node.
            
             `podAntiAffinity` use same process with `podAffinity`, just get `!{result}`

            the scope of the topolog search
             ` topologyKey:kubernetes.io/zone ` 
             ` topologyKey:kubernetes.io/hostname ` 

      d. `Taints` and `Tolerations`
         Taints: Added on node. a special label with `effect`, avoid pod deploy to a special node. 

         ```
            taints:
             - effect: NoSchedule/PreferNoSchedule
               key: key1
               value: value1
         ```
         Tolerations: Added on pod. ignore a Taints
         * exactly match: key=value:effect
         * match any value: if value is null and operator is Exists
         * match any effect: effect is null. 

         ```
            tolerations:
            - key: key1
              operator: Equal
              value: value1
              effect: NoSchedule
         ```
    

---

## k8s network module

   The mini schedule item in k8s is pod. each pod has its own ip (`hostNetwork` pod has same ip with host), each container in the pod share same network (using docker `container` network model).

   Two kinds of network

   * CNM from docker
       Container Network Model. Calico, Weave, Docker Swarm overlay, Macvlan, only support docker
   * CNI from google, CoreOS, k8s
       Container Network Interface. Calico, Weave, Flannel, kubernetes, Macvlan, support other containers like rkt.

       pod -> pod
       

---

## k8s storage
  
  `volumns`
   k8s is using volumns to store the data, volumns does not have QoS restriction.

  * configmap: start container
  * secret: start container
  * emptyDir: temp, emptydir will create a random location in host and will delete when pod delete.

        ```
        containers:
          ...
          volumeMounts:
           - mountPath: /containerlocation
            name: test
        volumes:
         - name test
           emptyDir: {}
        ```
  * hostPath: stored in host machine, will not delete while pod delete. can share data between pod.

     ```
        containers:
          ...
          volumeMounts:
           - mountPath: /containerlocation
            name: test
        volumes:
         - name test
           hostPath: 
             #host directory 
             path: /data
     ```
  * nfs: persistent 
  * cephfs: persistent
  * GlusterFS: persistent
  * Cloud: persistent


  `PV` PersistentVolume and `PVC` PersistentVolumnClaim

     The storage object in k8s, claim a storage with a certain type and capacity. 

     * static mode: Create PV and create `PersistentVolumnClaim`, use PVC to bind with pod. 
     * dynamic mode: Create `storageClass`, then create `PersistentVolumnClaim` and use PVC to bind with pod. 

---

## k8s ops

`kubectl cluster-info`
`journalctl -u kubelet`
isolate a node, once cordon, no new pod will be scheduled on this node. 
`kubectl cordon/uncordon <node>`

---
## K8s tips

### use bootstrap token auth

bootstrap token can retrieved using `kubeadm token list`

apiserver need enable `-enable-bootstrap-token-auth=true`
```
curl -H "Authorization: Bearer qpafj9.i2g5lucthim4wkgc" https://9.30.146.114:6443/version -k
```

[bootstrap token reference](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-token/)

### logs

check kubelet log
`journalctl -u kubelet`


### restart

both master and work nodes
```
swapoff -a
systemctl start docker
systemctl start kubelet
```


### Generate x509 certificate for k8s 
Use openssl to generate self certification for x509 client authentication. 
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

### Use services account to access k8s

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


### Trouble shooting 

#### can not pull image from redhat, missing redhat certification

openssl s_client -showcerts -servername registry.access.redhat.com -connect registry.access.redhat.com:443  2>/dev/null | openssl x509 -text > /etc/rhsm/ca/redhat-uep.pem

#### node port can access from worker node but not from master node

  cat /proc/sys/net/ipv4/ip_forward
  0
  
  change it to 1
  sysctl -w net.ipv4.ip_forward=1
 http://www.ducea.com/2006/08/01/how-to-enable-ip-forwarding-in-linux/

