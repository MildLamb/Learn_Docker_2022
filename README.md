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
yum-config-manager \
    --add-repo \
    http
```
