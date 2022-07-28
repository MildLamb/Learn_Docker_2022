# 帮助启动类命令
- 启动docker ：systemctl start docker
- 停止docker ：systemctl stop docker
- 重启docker ：systemctl restart docker
- 查看docker状态 ：systemctl status docker
- 开机启动 ：systemctl enable docker
- 查看docker概要信息 ：docker info
- 查看docker总体帮助文档 ：docker --help
- 查看docker命令帮助文档 ：docker 具体命令 --help

# 镜像命令
- docker images ：列出本地主机上的镜像
  - OPTIONS说明 ：
    - REPOSITORY ：表示镜像的仓库源
    - TAG ：镜像的标签（版本号）
    - IMAGE ID ：镜像的ID
    - CREATED ：镜像的创建时间
    - SIZE ：镜像的大小

```bash
[root@VM-4-16-centos ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
hello-world   latest    feb5d9fea6a5   10 months ago   13.3kB
```
- docker search 某个镜像的名字 ：远程仓库查询某个镜像
  - OPTIONS说明 ：
    - NAME ：镜像名称
    - DESCRIPTION ：镜像说明
    - STARS ：点赞数量
    - OFFICIAL ：是否是官方的
    - AUTOMATED ：是否是自动构建的
    - --list num : 显示排名前num个镜像

```bash
[root@VM-4-16-centos ~]# docker search hello-world
NAME                                       DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
kitematic/hello-world-nginx                A light-weight nginx container that demonstr…   152                  
tutum/hello-world                          Image to test docker deployments. Has Apache…   89                   [OK]
dockercloud/hello-world                    Hello World!                                    19                   [OK]
crccheck/hello-world                       Hello World web server in under 2.5 MB          15                   [OK]
... ...
```
- docker pull 某个镜像名字 ：拉取镜像
  - docker pull 镜像名称[:TAG]

```bash
[root@VM-4-16-centos ~]# docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
7b1a6ab2e44d: Pull complete 
Digest: sha256:626ffe58f6e7566e00254b638eb7e0f3b11d4da9675088f4781a50ae288f3322
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```

- docker system df ：查看镜像/容器/数据卷所占空间
```bash
[root@VM-4-16-centos ~]# docker system df
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          2         1         72.79MB   72.78MB (99%)
Containers      1         0         0B        0B
Local Volumes   0         0         0B        0B
Build Cache     0         0         0B        0B
```
- docker rmi [-f] 镜像ID : 删除某个镜像, 删除全部 docker rmi -f $(docker images -aq)

# 容器命令
- docker run [OPTIONS] IMAGE [COMMAND] [ARG...] : 启动交互式容器（前台命令行）
  - OPTIONS说明：
    - --name="容器名称" ：为容器指定一个名称
    - -d ：后台运行容器并返回容器ID，也即启动守护式容器(后台运行)
    
    - -i ：以交互模式运行容器，通常与-t一起使用
    - -t ：为容器重新分配一个伪输入终端，通常与-i同时使用
    
    - -P ：随机端口映射，大写P
    - -p ：指定端口映射，小写p

```bash
[root@VM-4-16-centos ~]# docker run -it ubuntu /bin/bash
root@cf275bef7c34:/#

[root@VM-4-16-centos ~]# docker run -it --name=mildlamb ubuntu /bin/bash
root@5838872347bf:/# 

[root@VM-4-16-centos ~]# docker run -d redis:6.0.8 
66e06d1c8c6d263171f58fd458d3bacc7dbd1d5eda0523f15cec3510f21940db
[root@VM-4-16-centos ~]# docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS      NAMES
66e06d1c8c6d   redis:6.0.8   "docker-entrypoint.s…"   3 seconds ago   Up 3 seconds   6379/tcp   determined_margulis
```


- docker ps ：查看正在运行的容器实例
  - OPTIONS说明：
    - -a ：列出当前所有正在运行的容器 + 历史运行过的
    - -l ：显示最近创建的容器
    - -n ：显示最近n个创建的容器
    - -q ：静默模式，只显示容器编号
```bash
[root@VM-4-16-centos /]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS         PORTS     NAMES
cf275bef7c34   ubuntu    "/bin/bash"   6 minutes ago   Up 6 minutes             beautiful_shannon
```
- 退出容器的两种方式
  - exit ：run进去容器，exit退出，容器停止
  - ctrl+p+q ：run进去容器，ctrl+p+q退出，容器不停止
- docker start ：容器ID或容器名 ：启动已经停止的容器
- docker restart ：容器ID或容器名 ：重新启动容器
- docker stop 容器ID或容器名 ：停止容器
- docker kill 容器ID或容器名 ：强制停止容器
- docker rm 容器ID ：删除已经停止的容器，一次删除多个实例，docker rm -f $(docker ps -aq) 或者 docker ps -aq | xargs docker rm -f 

- docker run -d 镜像id ：启动守护式容器
- docker logs 容器id ：查看容器日志
- docker top 容器id ：查看容器内运行的进程
- docker inspect 容器id ：查看容器内部细节
```bash
[root@VM-4-16-centos ~]# docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS      NAMES
66e06d1c8c6d   redis:6.0.8   "docker-entrypoint.s…"   12 minutes ago   Up 12 minutes   6379/tcp   determined_margulis
[root@VM-4-16-centos ~]# docker logs 66e06d1c8c6d
1:C 28 Jul 2022 00:39:01.472 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 28 Jul 2022 00:39:01.472 # Redis version=6.0.8, bits=64, commit=00000000, modified=0, pid=1, just started
1:C 28 Jul 2022 00:39:01.472 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
1:M 28 Jul 2022 00:39:01.473 * Running mode=standalone, port=6379.
1:M 28 Jul 2022 00:39:01.473 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
1:M 28 Jul 2022 00:39:01.473 # Server initialized
1:M 28 Jul 2022 00:39:01.473 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
1:M 28 Jul 2022 00:39:01.473 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo madvise > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled (set to 'madvise' or 'never').
1:M 28 Jul 2022 00:39:01.474 * Ready to accept connections
[root@VM-4-16-centos ~]# docker top 66e06d1c8c6d
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
polkitd             31439               31420               0                   08:39               ?                   00:00:01            redis-server *:6379
```

- 进入正在运行的容器并以命令行交互
  - docker exec -it 容器id bashShell
  - docker attach 容器id
  - exec 和 attach 区别
    - attach 直接进入容器启动命令的终端，不会启动新的进程，用exit退出，会导致容器停止
    - exec 是在容器中打开一个新的终端，并且可以启动新的进程，用exit退出，不会导致容器停止

```bash
[root@VM-4-16-centos ~]# docker exec -it 66e06d1c8c6d /bin/bash
root@66e06d1c8c6d:/data# cd ..
root@66e06d1c8c6d:/# 

[root@VM-4-16-centos ~]# docker attach c31b23a7679b
root@c31b23a7679b:/# exit
exit
[root@VM-4-16-centos ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```


- 容器内拷贝文件到主机
  - docker cp 容器id:容器内路径 目的主机路径

```bash
[root@VM-4-16-centos local]# docker cp 30164aaf29e5:/tmp/mildlamb.txt /usr/local
[root@VM-4-16-centos local]# ls
bin  etc  games  include  lib  lib64  libexec  mildlamb.txt  qcloud  sa  sbin  share  src
```

- 导入或导出容器
  - export : 导出容器的内容作为一个tar归档文件【对应import命令】
  - import ：从tar包中的内容创建一个新的文件系统再导入为镜像【对应export】
  - 命令
    - docker export 容器id > 文件名.tar
    - cat 文件名.tar | docker import - 镜像用户/镜像名:镜像版本号

```bash
# 容器备份
[root@VM-4-16-centos temp]# docker export 30164aaf29e5 > ubuntu.tar
[root@VM-4-16-centos temp]# ls
mildlamb.txt  ubuntu.tar

# 容器恢复
[root@VM-4-16-centos temp]# cat ubuntu.tar | docker import - wildwolf/ubuntu:1.0
sha256:556d27697771343c1d10491700427e513e5cc67be470b564805fb1e105207735
[root@VM-4-16-centos temp]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@VM-4-16-centos temp]# docker images
REPOSITORY        TAG       IMAGE ID       CREATED          SIZE
wildwolf/ubuntu   1.0       556d27697771   10 seconds ago   72.8MB
ubuntu            latest    ba6acccedd29   9 months ago     72.8MB
hello-world       latest    feb5d9fea6a5   10 months ago    13.3kB
redis             6.0.8     16ecd2772934   21 months ago    104MB
[root@VM-4-16-centos temp]# docker run -it 556d27697771 /bin/bash
root@97a71244aef3:/# cd tmp
root@97a71244aef3:/tmp# ls
mildlamb.txt
```




### 使用ubuntu 镜像，以交互模式启动一个容器，在容器内执行/bin/bash 命令
```bash
[root@VM-4-16-centos ~]# docker run -it ubuntu /bin/bash
root@cf275bef7c34:/# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 02:43 pts/0    00:00:00 /bin/bash
root         8     1  0 02:43 pts/0    00:00:00 ps -ef
root@cf275bef7c34:/# ls
bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```
- 参数说明：
  - -i ：交互式操作
  - -t ：终端
  - ubuntu ： ubuntu镜像
  - /bin/bash ：放在镜像名后的是命令，这里我们希望有个交互式Shell，因此用的是/bin/bash
  - 退出终端，exit
