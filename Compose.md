# Compose
## Compose是什么?
- Docker-Compose是官方的开源项目，负责实现对Docker容器集群的快速编排
- Compose是Docker公司推出的一个工具软件，可以管理多个Docker容器组成的一个应用，你需要
  定义一个YAML格式的配置文件docker-compose.yml，写好多个容器之间的协调关系。然后，只要
  一个命令，就能同时启动/关闭这些容器
- Compose允许用户通过单独的docker-compose.yml模板文件(YAML格式)来定义一组相关联的应用容器为一个项目(project)
- Docker-Compose解决了容器与容器之间如何管理编排的问题

## Compose下载和安装
- 官网 ：[https://docs.docker.com/compose/compose-file/compose-file-v3/](https://docs.docker.com/compose/compose-file/compose-file-v3/)
- 官网下载 : [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)
- 安装步骤：
  - curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  - sudo chmod +x /usr/local/bin/docker-compose
  - docker-compose --version
- 卸载：
  - sudo rm /usr/local/bin/docker-compose
