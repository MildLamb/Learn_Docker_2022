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
