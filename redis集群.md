# Redis集群
- 新建6个docker容器实例
```bash
docker run -d --name redis-node-1 --net host --privileged=true -v /mildlamb/redis/share/redis-node-1:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6381
docker run -d --name redis-node-2 --net host --privileged=true -v /mildlamb/redis/share/redis-node-2:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6382
docker run -d --name redis-node-3 --net host --privileged=true -v /mildlamb/redis/share/redis-node-3:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6383
docker run -d --name redis-node-4 --net host --privileged=true -v /mildlamb/redis/share/redis-node-4:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6384
docker run -d --name redis-node-5 --net host --privileged=true -v /mildlamb/redis/share/redis-node-5:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6385
docker run -d --name redis-node-6 --net host --privileged=true -v /mildlamb/redis/share/redis-node-6:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6386
```
- 进入docker容器后执行以下命令，使用真实IP,--cluster-replicas 1 表示一个master对应一个slave
```bash
redis-cli --cluster create 10.0.4.16:6381 10.0.4.16:6382 10.0.4.16:6383 10.0.4.16:6384 10.0.4.16:6385 10.0.4.16:6386 --cluster-replicas 1
```
- 上面的命令没问题就是下面这样
```bash
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 10.0.4.16:6385 to 10.0.4.16:6381
Adding replica 10.0.4.16:6386 to 10.0.4.16:6382
Adding replica 10.0.4.16:6384 to 10.0.4.16:6383
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: 57d6d265fa9b2b2234ad776e2c18fd232c025238 10.0.4.16:6381
   slots:[0-5460] (5461 slots) master
M: df80201f0587c5007ee4de8b82d3256cf3c032a9 10.0.4.16:6382
   slots:[5461-10922] (5462 slots) master
M: 596b0428fabef6c1d6705055c9abd6b464afb9a0 10.0.4.16:6383
   slots:[10923-16383] (5461 slots) master
S: 9aa6c43a89a421e0fbd0648396a28aadb661b714 10.0.4.16:6384
   replicates df80201f0587c5007ee4de8b82d3256cf3c032a9
S: 031a52b41e7a73780fdb012761d5f1df762f0205 10.0.4.16:6385
   replicates 596b0428fabef6c1d6705055c9abd6b464afb9a0
S: 99adbc860d347596ff6d46b4869500fa3c2a3c1b 10.0.4.16:6386
   replicates 57d6d265fa9b2b2234ad776e2c18fd232c025238
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
.
>>> Performing Cluster Check (using node 10.0.4.16:6381)
M: 57d6d265fa9b2b2234ad776e2c18fd232c025238 10.0.4.16:6381
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: 596b0428fabef6c1d6705055c9abd6b464afb9a0 10.0.4.16:6383
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
M: df80201f0587c5007ee4de8b82d3256cf3c032a9 10.0.4.16:6382
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 9aa6c43a89a421e0fbd0648396a28aadb661b714 10.0.4.16:6384
   slots: (0 slots) slave
   replicates df80201f0587c5007ee4de8b82d3256cf3c032a9
S: 99adbc860d347596ff6d46b4869500fa3c2a3c1b 10.0.4.16:6386
   slots: (0 slots) slave
   replicates 57d6d265fa9b2b2234ad776e2c18fd232c025238
S: 031a52b41e7a73780fdb012761d5f1df762f0205 10.0.4.16:6385
   slots: (0 slots) slave
   replicates 596b0428fabef6c1d6705055c9abd6b464afb9a0
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```
- 进入容器中的redis,查看集群信息
```bash
cluster info
```
- 主从节点信息
```bash
cluster nodes
```

# 主从容错，切换迁移
- 向6381中新增两个key
- 为防止路由失效，可以添加-c以集群方式进入redis
- 集群检查
```bash
redis-cli --cluster check 10.0.4.16:6381
```
```bash
root@VM-4-16-centos:/data# redis-cli --cluster check 10.0.4.16:6381
10.0.4.16:6381 (57d6d265...) -> 0 keys | 5461 slots | 1 slaves.
10.0.4.16:6383 (596b0428...) -> 0 keys | 5461 slots | 1 slaves.
10.0.4.16:6382 (df80201f...) -> 1 keys | 5462 slots | 1 slaves.
[OK] 1 keys in 3 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 10.0.4.16:6381)
M: 57d6d265fa9b2b2234ad776e2c18fd232c025238 10.0.4.16:6381
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: 596b0428fabef6c1d6705055c9abd6b464afb9a0 10.0.4.16:6383
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
M: df80201f0587c5007ee4de8b82d3256cf3c032a9 10.0.4.16:6382
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 9aa6c43a89a421e0fbd0648396a28aadb661b714 10.0.4.16:6384
   slots: (0 slots) slave
   replicates df80201f0587c5007ee4de8b82d3256cf3c032a9
S: 99adbc860d347596ff6d46b4869500fa3c2a3c1b 10.0.4.16:6386
   slots: (0 slots) slave
   replicates 57d6d265fa9b2b2234ad776e2c18fd232c025238
S: 031a52b41e7a73780fdb012761d5f1df762f0205 10.0.4.16:6385
   slots: (0 slots) slave
   replicates 596b0428fabef6c1d6705055c9abd6b464afb9a0
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```
- 停止6381容器实例，发现原来的从机变成了主机
```bash
127.0.0.1:6382> cluster nodes
9aa6c43a89a421e0fbd0648396a28aadb661b714 10.0.4.16:6384@16384 slave df80201f0587c5007ee4de8b82d3256cf3c032a9 0 1659491643806 2 connected
031a52b41e7a73780fdb012761d5f1df762f0205 10.0.4.16:6385@16385 slave 596b0428fabef6c1d6705055c9abd6b464afb9a0 0 1659491642000 3 connected
99adbc860d347596ff6d46b4869500fa3c2a3c1b 10.0.4.16:6386@16386 master - 0 1659491641000 7 connected 0-5460
df80201f0587c5007ee4de8b82d3256cf3c032a9 10.0.4.16:6382@16382 myself,master - 0 1659491643000 2 connected 5461-10922
596b0428fabef6c1d6705055c9abd6b464afb9a0 10.0.4.16:6383@16383 master - 0 1659491642803 3 connected 10923-16383
57d6d265fa9b2b2234ad776e2c18fd232c025238 10.0.4.16:6381@16381 master,fail - 1659491571572 1659491567563 1 disconnected
```
- 如果6381重新启动，6381会是什么情况（6381最终沦为了从节点）

# 主从扩容案例
- 新建6387，6388两个节点+新建后启动+查看是否8节点
```bash
docker run -d --name redis-node-7 --net host --privileged=true -v /data/redis/share/redis-node-7:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6387
docker run -d --name redis-node-8 --net host --privileged=true -v /data/redis/share/redis-node-8:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6388
```
- 进入6387容器实例内部
```bash
docker exec -it redis-node-7 /bin/bash
```
- 将新增的6387节点(空槽号)作为master节点加入原集群
```bash
# redis-cli --cluster add-node 新加入的节点 集群中的原主机节点
redis-cli --cluster add-node 10.0.4.16:6387 10.0.4.16:6381
```
```bash
>>> Adding node 10.0.4.16:6387 to cluster 10.0.4.16:6381
>>> Performing Cluster Check (using node 10.0.4.16:6381)
S: 57d6d265fa9b2b2234ad776e2c18fd232c025238 10.0.4.16:6381
   slots: (0 slots) slave
   replicates 99adbc860d347596ff6d46b4869500fa3c2a3c1b
S: 031a52b41e7a73780fdb012761d5f1df762f0205 10.0.4.16:6385
   slots: (0 slots) slave
   replicates 596b0428fabef6c1d6705055c9abd6b464afb9a0
M: 596b0428fabef6c1d6705055c9abd6b464afb9a0 10.0.4.16:6383
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 9aa6c43a89a421e0fbd0648396a28aadb661b714 10.0.4.16:6384
   slots: (0 slots) slave
   replicates df80201f0587c5007ee4de8b82d3256cf3c032a9
M: 99adbc860d347596ff6d46b4869500fa3c2a3c1b 10.0.4.16:6386
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: df80201f0587c5007ee4de8b82d3256cf3c032a9 10.0.4.16:6382
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
>>> Send CLUSTER MEET to node 10.0.4.16:6387 to make it join the cluster.
[OK] New node added correctly.
```
- 检查节点状态(第一次)
```bash
redis-cli --cluster check 10.0.4.16:6381
```
```bash
# 此时暂时没有槽位
10.0.4.16:6383 (596b0428...) -> 0 keys | 5461 slots | 1 slaves.
10.0.4.16:6386 (99adbc86...) -> 0 keys | 5461 slots | 1 slaves.
10.0.4.16:6382 (df80201f...) -> 1 keys | 5462 slots | 1 slaves.
10.0.4.16:6387 (2a3eaf47...) -> 0 keys | 0 slots | 0 slaves.
[OK] 1 keys in 4 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 10.0.4.16:6381)
S: 57d6d265fa9b2b2234ad776e2c18fd232c025238 10.0.4.16:6381
   slots: (0 slots) slave
   replicates 99adbc860d347596ff6d46b4869500fa3c2a3c1b
S: 031a52b41e7a73780fdb012761d5f1df762f0205 10.0.4.16:6385
   slots: (0 slots) slave
   replicates 596b0428fabef6c1d6705055c9abd6b464afb9a0
M: 596b0428fabef6c1d6705055c9abd6b464afb9a0 10.0.4.16:6383
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 9aa6c43a89a421e0fbd0648396a28aadb661b714 10.0.4.16:6384
   slots: (0 slots) slave
   replicates df80201f0587c5007ee4de8b82d3256cf3c032a9
M: 99adbc860d347596ff6d46b4869500fa3c2a3c1b 10.0.4.16:6386
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: df80201f0587c5007ee4de8b82d3256cf3c032a9 10.0.4.16:6382
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
M: 2a3eaf47c5ec7f4e8f2c65b19a24831ac30eeb39 10.0.4.16:6387
   slots: (0 slots) master
```
- 重新分配槽号
```bash
# redis-cli --cluster reshard 集群IP地址:端口号

root@VM-4-16-centos:/data# redis-cli --cluster reshard 10.0.4.16:6381
>>> Performing Cluster Check (using node 10.0.4.16:6381)
S: 57d6d265fa9b2b2234ad776e2c18fd232c025238 10.0.4.16:6381
   slots: (0 slots) slave
   replicates 99adbc860d347596ff6d46b4869500fa3c2a3c1b
S: 031a52b41e7a73780fdb012761d5f1df762f0205 10.0.4.16:6385
   slots: (0 slots) slave
   replicates 596b0428fabef6c1d6705055c9abd6b464afb9a0
M: 596b0428fabef6c1d6705055c9abd6b464afb9a0 10.0.4.16:6383
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 9aa6c43a89a421e0fbd0648396a28aadb661b714 10.0.4.16:6384
   slots: (0 slots) slave
   replicates df80201f0587c5007ee4de8b82d3256cf3c032a9
M: 99adbc860d347596ff6d46b4869500fa3c2a3c1b 10.0.4.16:6386
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: df80201f0587c5007ee4de8b82d3256cf3c032a9 10.0.4.16:6382
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
M: 2a3eaf47c5ec7f4e8f2c65b19a24831ac30eeb39 10.0.4.16:6387
   slots: (0 slots) master
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
# 16384/4(主机数量) = 4096
How many slots do you want to move (from 1 to 16384)? 4096
# 分配给谁 ，选择新加入的主机的ID
What is the receiving node ID? 2a3eaf47c5ec7f4e8f2c65b19a24831ac30eeb39
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
Source node #1: all
```
- 检查节点分配情况(第二次)(新节点的槽位，由原来的节点各自分一部分出来)
```bash
root@VM-4-16-centos:/data# redis-cli --cluster check 10.0.4.16:6381
10.0.4.16:6383 (596b0428...) -> 0 keys | 4096 slots | 1 slaves.
10.0.4.16:6386 (99adbc86...) -> 0 keys | 4096 slots | 1 slaves.
10.0.4.16:6382 (df80201f...) -> 0 keys | 4096 slots | 1 slaves.
10.0.4.16:6387 (2a3eaf47...) -> 1 keys | 4096 slots | 0 slaves.
[OK] 1 keys in 4 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 10.0.4.16:6381)
S: 57d6d265fa9b2b2234ad776e2c18fd232c025238 10.0.4.16:6381
   slots: (0 slots) slave
   replicates 99adbc860d347596ff6d46b4869500fa3c2a3c1b
S: 031a52b41e7a73780fdb012761d5f1df762f0205 10.0.4.16:6385
   slots: (0 slots) slave
   replicates 596b0428fabef6c1d6705055c9abd6b464afb9a0
M: 596b0428fabef6c1d6705055c9abd6b464afb9a0 10.0.4.16:6383
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
S: 9aa6c43a89a421e0fbd0648396a28aadb661b714 10.0.4.16:6384
   slots: (0 slots) slave
   replicates df80201f0587c5007ee4de8b82d3256cf3c032a9
M: 99adbc860d347596ff6d46b4869500fa3c2a3c1b 10.0.4.16:6386
   slots:[1365-5460] (4096 slots) master
   1 additional replica(s)
M: df80201f0587c5007ee4de8b82d3256cf3c032a9 10.0.4.16:6382
   slots:[6827-10922] (4096 slots) master
   1 additional replica(s)
M: 2a3eaf47c5ec7f4e8f2c65b19a24831ac30eeb39 10.0.4.16:6387
   slots:[0-1364],[5461-6826],[10923-12287] (4096 slots) master
```
- 为主节点6387分配从节点6388
```bash
# redis-cli --cluster add-node IP:新slave端口 IP:新master端口 --cluster-slave --cluster-master-id 新主机节点ID
redis-cli --cluster add-node 10.0.4.16:6388 10.0.4.16:6387 --cluster-slave --cluster-master-id 2a3eaf47c5ec7f4e8f2c65b19a24831ac30eeb39
```
- 检查节点分配情况(第三次)
```bash
redis-cli --cluster check 10.0.4.16:6381
Could not connect to Redis at 127.0.0.1:6379: Connection refused
root@VM-4-16-centos:/data# redis-cli --cluster check 10.0.4.16:6381
10.0.4.16:6383 (596b0428...) -> 0 keys | 4096 slots | 1 slaves.
10.0.4.16:6386 (99adbc86...) -> 0 keys | 4096 slots | 1 slaves.
10.0.4.16:6382 (df80201f...) -> 0 keys | 4096 slots | 1 slaves.
10.0.4.16:6387 (2a3eaf47...) -> 1 keys | 4096 slots | 1 slaves.
[OK] 1 keys in 4 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 10.0.4.16:6381)
S: 57d6d265fa9b2b2234ad776e2c18fd232c025238 10.0.4.16:6381
   slots: (0 slots) slave
   replicates 99adbc860d347596ff6d46b4869500fa3c2a3c1b
S: 6b20fa709ff85c23ca084767472b7043f8d08423 10.0.4.16:6388
   slots: (0 slots) slave
   replicates 2a3eaf47c5ec7f4e8f2c65b19a24831ac30eeb39
S: 031a52b41e7a73780fdb012761d5f1df762f0205 10.0.4.16:6385
   slots: (0 slots) slave
   replicates 596b0428fabef6c1d6705055c9abd6b464afb9a0
M: 596b0428fabef6c1d6705055c9abd6b464afb9a0 10.0.4.16:6383
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
S: 9aa6c43a89a421e0fbd0648396a28aadb661b714 10.0.4.16:6384
   slots: (0 slots) slave
   replicates df80201f0587c5007ee4de8b82d3256cf3c032a9
M: 99adbc860d347596ff6d46b4869500fa3c2a3c1b 10.0.4.16:6386
   slots:[1365-5460] (4096 slots) master
   1 additional replica(s)
M: df80201f0587c5007ee4de8b82d3256cf3c032a9 10.0.4.16:6382
   slots:[6827-10922] (4096 slots) master
   1 additional replica(s)
M: 2a3eaf47c5ec7f4e8f2c65b19a24831ac30eeb39 10.0.4.16:6387
   slots:[0-1364],[5461-6826],[10923-12287] (4096 slots) master
   1 additional replica(s)
```

# 主从缩容案例
- 先清除从节点6388
```bash
# redis-cli --cluster del-node IP:从机端口 从机6388节点ID
redis-cli --cluster del-node 10.0.4.16:6388 6b20fa709ff85c23ca084767472b7043f8d08423
```
- 将6387的槽号清空，重新分配，本例将清出来的槽号都给6381
```bash
redis-cli --cluster reshard 10.0.4.16:6381
[OK] All 16384 slots covered.
# 选择移动槽号数量
How many slots do you want to move (from 1 to 16384)? 1365
# 选择哪个主机来接收槽号
What is the receiving node ID? 690b2e1f604a0227068388d3e5b1f1940524c565
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
# 从哪里移除槽号
Source node #1:724a8a15f4efe5a01454cb971d7471d6e84279f3
Source node #2:done
```
- 删除主机6387
```bash
redis-cli --cluster del-node 10.0.4.16:6387 2a3eaf47c5ec7f4e8f2c65b19a24831ac30eeb39
```
- 检查节点
```bash
redis-cli --cluster check 10.0.4.16:6381
```
