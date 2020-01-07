
### OSI  七层模型

*Layer#7* 应用层 Application Layer
*Layer#6* 表示层 Presentation Layer
*Layer#5* 会话层 Session Layer
### *Layer#4* 传输层 Transport Layer TCP, UDP is used in this layer,防火墙在这一层通过端口号进行过滤
TCP：三次握手，面向连接，可靠
UDP：连接不可靠，不需要握手反馈，速度快

TCP或者UDP数据包中添加额外source port和destination port用来定义源和目标端口号，占用16位。所以操作系统的端口号范围是
0~65535（2的16次方）
小于1024的端口号为特权端口号，需要有root的权限才能开启。发送数据包的时候源端口号是随机分配一个大于1024的端口号。

### Layer#3* 网络层 Network Layer  IP, arp ICMP is used in this layer.防火墙在这一层通过ip地址，数据包内容进行拦截
网络层相关的内容
IP数据包最大可以达到65535B,包含源和目标ip地址

IPv4地址分类以及预留局域网ip  
```
ClassA: 0.x.x.x   ~ 127.x.x.x 预留内网 10.x.x.x
ClassB: 128.x.x.x ~ 191.x.x.x 预留内网 172.16.x.x ~ 172.31.x.x
ClassC: 192.x.x.x ~ 223.x.x.x 预留内网 192.168.x.x 
ClassD: 224.x.x.x ~ 239.x.x.x 组播地址
ClassE: 240.x.x.x ~ 255.x.x.x 保留IP
```

CIDR 无类别域间路由  
网络地址=network/netmask 组成，由netmask决定netid,netid不相同就不在一个网段，需要通过路由器转发
Q:192.168.10.100/25 vs 192.168.10.200/25 在不在同一个网络？
A: 子网借用第四部分(25-24=1)位，所以第四部分256个网段被分成两部分共(2的一次方)，位0~127,128~255 所以不在一个网段。  
192.168.10.100/25 network=192.168.10.0 netmask=255.255.255.128 broadcast: 192.168.10.128
192.168.10.200/25 network=192.168.10.128 netmask=255.255.255.128 broadcast: 192.168.10.255  

ARP:地址解析协议 解析ip地址和MAC地址之间的对应，动态更新  
```
arp -n 192.168.100.200
? (192.168.100.200) at 0:11:32:89:69:b2 on en0 ifscope [ethernet]
taos-MacBook-Pro:~ tao$ arp -n 192.168.100.1
? (192.168.100.1) at 50:d2:f5:22:43:6e on en0 ifscope [ethernet]

arp  -a
xiaoqiang (192.168.100.1) at 50:d2:f5:22:43:6e on en0 ifscope [ethernet]
? (192.168.100.140) at f4:a9:97:54:e8:71 on en0 ifscope [ethernet]
? (192.168.100.200) at 0:11:32:89:69:b2 on en0 ifscope [ethernet]
? (192.168.100.255) at ff:ff:ff:ff:ff:ff on en0 ifscope [ethernet]
? (224.0.0.251) at 1:0:5e:0:0:fb on en0 ifscope permanent [ethernet]
? (239.255.255.250) at 1:0:5e:7f:ff:fa on en0 ifscope permanent [ethernet]

ICMP协议 测试网络连接状态的协议，ICMP数据包也是通过IP数据包传送 ping, traceroute命令用到ICMP协议测试网络状态


```
### *Layer#2* 数据链路层 Data-Link Layer 防火墙在这一层通过MAC地址进行隔离

MAC 数据帧，包含源和目的MAC地址 

MTU表示数据帧的最大容量(Max Transmission Unit) MTU=1500B,如果自定义MTU,并不能确定网络上所有网络设备都支持大于1500的MTU,不推荐在
internet上使用。 
Check MTU of a network device  
```  
ifconfig en0
en0: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
	ether ac:bc:32:bf:67:53 
	inet6 fe80::4db:bdd8:3386:be91%en0 prefixlen 64 secured scopeid 0x5 
	inet 192.168.100.134 netmask 0xffffff00 broadcast 192.168.100.255
	nd6 options=201<PERFORMNUD,DAD>
	media: autoselect
	status: active

```  

Hub vs Switch  
Switch不是共享设备，每个port可以有独有的内从传递MAC数据帧。 
### *Layer#1* 物理层 Physical-Layer  
RJ-45  
568A 绿白绿蓝白橙橙白蓝棕白棕  
568B 橙白橙蓝白绿绿白蓝棕白棕  

Auto MDI/MDIX  
switch支持自动识别直线或者交叉线
