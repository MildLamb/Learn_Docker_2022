# Portainer
## 是什么?
- Portainer是一款轻量级的应用，它提供了图形化界面，用于方便的管理Docker环境，包括单机环境和集群环境

## 安装并登录
- 安装并启动
```bash
docker run -d -p 8000:8000 -p 9000:9000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```
- 第一次登录需创建admin，访问地址：xxx.xxx.xxx.xxx:9090
- 设置admin用户和密码后首次登录
- 选择local选项卡后docker详细信息展示
