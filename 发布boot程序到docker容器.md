# 发布boot程序到docker容器
- 在IDEA中编写一个简单的程序
- 编写Dockerfile
```bash
# 基础镜像使用java
FROM java:8
# 作者
MAINTAINER mildlamb
# VOLUME 指定临时文件目录为/tmp，在主机/var/lib/docker目录下创建一个临时文件并链接到容器的/tmp
VOLUME /tmp
# 将jar包添加到容器中 并更名为 mildlamb.jar
ADD docker_test-0.0.1-SNAPSHOT.jar mildlamb.jar
# 运行jar包
RUN bash -c 'touch /mildlamb.jar'
ENTRYPOINT ["java","-jar","mildlamb.jar"]
# 暴露8080端口作为微服务
EXPOSE 8080
```
- 构建镜像
```bash
docker build -t mildlamb_docker:1.0 .
```
