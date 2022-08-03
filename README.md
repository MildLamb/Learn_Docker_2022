# Learn_Docker_2022

# docker批量停止/开启命令
```bash
docker stop $(docker ps -a | grep "xxx" | awk '{print $1}')
```

# Linux查看防火墙，开放端口
1. 查看防火墙状态

systemctl status firewalld

2. 如果不是显示active状态，需要打开防火墙

systemctl start firewalld

3.  查看所有已开放的临时端口（默认为空）

firewall-cmd --list-ports

### 查看所有永久开放的端口（默认为空）
firewall-cmd --list-ports --permanent



 

### 添加临时开放端口（例如：比如我修改ssh远程连接端口是223，则需要开放这个端口）
firewall-cmd --add-port=223/tcp

### 添加永久开放的端口（例如：223端口）
firewall-cmd --add-port=223/tcp --permanent

### 关闭临时端口
firewall-cmd --remove-port=80/tcp

### 关闭永久端口
firewll-cmd --remove-port=80/tcp --permanent

### 配置结束后需要输入重载命令并重启防火墙以生效配置

```bash
 firewall-cmd --reload
 systemctl restart firewalld
```

<hr>

## Docker安装
- 卸载旧版本
```bash
 sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
- 安装需要的软件包
```bash
yum install -y yum-utils
```
- 设置镜像仓库
```bash
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
- 更新yum软件包索引
```bash
yum makecache fast
```
- 安装docker引擎
```bash
yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
- 启动docker
```bash
systemctl start docker
```
## 配置阿里云镜像加速
- 登录阿里云，在产品中找到容器镜像服务，创建一个个人版的镜像托管
- 在镜像工具中，找到镜像加速器
- 执行按照提示执行下面4个命令
```bash
sudo mkdir -p /etc/docker

sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://9ku8hjj8.mirror.aliyuncs.com"]
}
EOF

sudo systemctl daemon-reload

sudo systemctl restart docker
```
- 执行完命令就完成了阿里云的镜像加速

## Run干了什么
![image](https://user-images.githubusercontent.com/92672384/181137453-a4d6d8c4-c932-411e-8b4c-4ac936389c69.png)
