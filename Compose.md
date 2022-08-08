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

## Compose核心概念
- 一文件 ：docker-compose.yml
- 两要素 ：
  - 服务：一个个应用容器实例，比如订单微服务，库存微服务等
  - 工程：由一组关联的应用容器组成的一个完整业务单元，在docker-compose.yml文件中定义

## Compose使用的三个步骤
- 编写Dockerfile定义各个微服务应用并构建出对应的镜像文件
- 使用docker-compose.yml定义一个完整业务单元，安排好整体应用中的各个容器服务
- 最后，执行docker-compose up命令来启动并运行整个应用程序，完成一键部署上线

## Compose常用命令
- docker-compose -h : 查看帮助
- docker-compose up ：启动所有docker-compose服务
- docker-compose up -d ：启动所有docker-compose服务并后台运行
- docker-compose down ：停止并删除容器，网络，卷，镜像
- docker-compose exec yml中的服务id ：进入容器实例内部
- docker-compose ps ：展示当前docker-compose编排过的运行的所有容器
- docker-compose top ： 展示当前docker-compose编排过的容器进程

- docker-compose logs yml里面的服务id ：查看容器输出日志
- docker-compose config ：查看配置
- docker-compose config -q ：查看配置，有问题才有输出
- docker-compose restart ：重启服务
- docker-compose start ：启动服务
- docker-compose stop ：停止服务

## docker-compose.yml
```bash
vresion: "3"

services:
  microService:
    image: mildlamb_docker:1.6
    container_name: ms01
    ports:
      - "6001:6001"
    volumes:
      - /app/microService:/data
    networks:
      - mildlamb_net
    depends_on:
      - redis
      - mysql
  
  redis:
    image: redis:6.0.8
    ports:
      - "6379:6379"
    volumes:
      - /app/redis/redis.conf:/etc/redis/redis.conf
      - /app/redis/data:/data
    networks:
      - mildlamb_net
    command: redis-server /etc/redis/redis.conf
    
  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: "123456"
      MYSQL_ALLOW_EMPTY_PASSWORD: "no"
      MYSQL_DATABASE: "db2022"
      MYSQL_USER: "root"
      MYSQL_PASSWORD: "123456"
    ports:
      - "3306:3306"
    volumes:
      - /app/mysql/db:/var/lib/mysql
      - /app/mysql/conf/my.cnf:/etc/my.cnf
      - /app/mysql/init:/docker-entrypoint-initdb.d
    networks:
      - mildlamb_net
    command: --default-authentication-plugin=mysql_native_password # 解决外部无法访问
    
networks:
  mildlamb_net:
      
```
