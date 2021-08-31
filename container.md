## 关于容器和编排的一些名词

OCI : Open Container Initiative. 由Docker牵头，其他公司follow up的镜像和容器的规范。OCI有一个实现是runc.

CRI: Container Runtime Interface. 是k8s的API，定义了k8s如何与容器运行时交互。所以可以选择具体的CRI实现。实现有containerd (containerd实现了CRI，所有k8s可以调度docker的container)
,CRI-O

[reference](https://www.cnblogs.com/rancherlabs/p/14984052.html)
