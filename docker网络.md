## Docker网络常用命令
- docker network ls : 展示docker网络模式，默认创建三个网络模式
```bash
NETWORK ID     NAME      DRIVER    SCOPE
4c90d14df80b   bridge    bridge    local
56d18e9ca301   host      host      local
1958877eeef4   none      null      local
```
- docker network create 网络名称 : 创建一个新的docker网络
```bash
docker network create mildlamb_network
```
- docker network rm 网络名称 ：删除一个网络
```bash
docker network rm mildlamb_network
```
- docker network inspect 网络名称 : 查看网络数据源
```bash
docker network inspect bridge
```

## docker网络能干什么?
- 容器间的互联和通信以及端口映射
- 容器IP变动时可以通过服务名直接网络通信而不受到影响

## docker的网络模式有几种?
| 网络模式 | 简介 |
|:--:|:--:|
| bridge | 为每一个容器分配，设置IP等，
并将容器连接到一个docker0，虚拟网桥，默认为该模式 |
| host | 容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口 |
| none | 容器有独立的Network namespace，但没有对其进行任何网络设置，如分配veth pair和网桥连接，IP等 |
| container | 新创建的容器不会创建自己的网卡和配置自己的IP，而是和一个指定的容器共享IP，端口范围 |


# Bridge
1. Docker使用Linux桥接，在宿主机虚拟一个Docker容器网桥(docker0),Docker启动一个容器时会根据Docker网桥内的网段分配给容器一个
IP地址，称为Container-IP，同时Docker网桥是每个容器的默认网关。因为在同一宿主机内的容器都接入同一个网桥，这样容器之间就能通过
容器的Container-IP直接通信
2. docker run的时候，没有指定network的话默认使用的网桥模式就是bridge，使用的就是docker0，在宿主机ifconfig，就可以看到docker0
和自己create的network，eth0，eth1，eth2... 代表网卡1，网卡2...，lo代表127.0.0.1，即localhost，inet addr用来表示网卡的IP地址
3. 网桥docker0创建对对等虚拟设备接口一个叫veth，另一个叫eth0，成对匹配

# Host
- 容器将不会获得一个独立的Network Namespace，而是和宿主机共用一个Network Namespace，容器将不会虚拟出自己的网卡而是使用宿主机的IP和端口

# None
- 在none模式下，并不为Docker容器进行任何网络配置
- 也就是说，这个Docker容器没有网卡，IP，路由等信息，只有一个lo
- 需要我们自己为Docker容器添加网卡，配置IP等

# Container
- 新建的容器和已经存在的一个容器共享一个网络IP配置而不是和宿主机共享，新创建的容器不会创建自己的网卡，配置自己的IP，而是和一个指定的容器共享
IP，端口范围等。同样，两个容器除了网络方面，其他的如文件系统，进程列表等还是隔离的。

# 自定义网络
- 如果不使用自定义网络，通过IP地址可以ping通，但无法使用服务名ping通

- 使用自定义网络
  - 自定义桥接网络，自定义网络默认使用的是桥接网络bridge
  - 新建自定义网络
```bash
[root@VM-4-16-centos ~]# docker network create mildlamb
```
  - 新建容器加入上一步新建的自定义网络
```bash
[root@VM-4-16-centos ~]# docker run -it --network mildlamb --name alpine1 alpine /bin/sh
[root@VM-4-16-centos ~]# docker run -it --network mildlamb --name alpine2 alpine /bin/sh
```
  - 通过服务名互相ping测试
```bash
/ # ping alpine1
PING alpine1 (172.19.0.2): 56 data bytes
64 bytes from 172.19.0.2: seq=0 ttl=64 time=0.149 ms
64 bytes from 172.19.0.2: seq=1 ttl=64 time=0.108 ms
64 bytes from 172.19.0.2: seq=2 ttl=64 time=0.107 ms
```
