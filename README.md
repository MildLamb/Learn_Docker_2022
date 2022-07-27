# Learn_Docker_2022
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
