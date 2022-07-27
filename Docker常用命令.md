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
