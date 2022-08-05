# DockerFile是什么？
- DockerFile是用来构建Docker镜像的文本文件，是由一条条构建镜像所需的指令和参数构成的脚本

![23XJ{8S~(~EBN (CUUK9(BP](https://user-images.githubusercontent.com/92672384/182976375-16b9c439-aaea-43f7-8578-ef91940e6f80.png)

# DockerFile内容基础知识
1. 每条保留字指令都必须为大写字母且后面要跟随至少一个参数
2. 指令按照从上到下，顺序执行
3. #表示注释
4. 每条指令都会创建一个新的镜像层并对镜像进行提交

# Docker执行DockerFile的大致流程
1. docker从基础镜像运行一个容器
2. 执行一条指令并对容器作出修改
3. 执行类似docker commit的操作提交一个新的镜像层
4. docker再基于刚提交的镜像运行一个新容器
5. 执行dockerfile中的下一条指令直到所有指令都执行完成

# Dockerfile常用保留字指令
- FROM ：基础镜像，当前新镜像是基于哪个镜像的，指定一个已经存在的镜像作为模板，第一条必须是from
- MAINTAINER ：镜像维护者的姓名和邮件地址
- RUN ：容器构建时需要运行的命令，RUN在docker build时运行
  - 两种模式：
    - shell格式
    - exec格式
- EXPOSE ：当前容器对外暴露的端口
- WORKDIR ：指定在创建容器后，终端默认登录进来的工作目录
- USER ：指定该镜像是以什么样的用户去执行，如果不指定，默认是root
- ENV ：用来在构建镜像过程中设置环境变量
- VOLUME ：容器数据卷，用于数据保存和持久化工作
- ADD ：将宿主机目录下的文件拷贝进镜像且会自动处理URL和解压tar压缩包
- COPY ：类似ADD，拷贝文件和目录到镜像中。将从构建上下文目录中<源路径>的文件/目录复制到新的一层的镜像内的<目标路径>位置
- CMD ：容器启动后要运行的命令，可以有多个CMD指令，但只有最后一个生效，CMD会被docker run 之后的参数替换
- ENTRYPOINT ：也是指定一个容器启动时要运行的命令，ENTRYPOINT 和 CMD可以一起使用，当指定了ENTRYPOINT后，CMD的含义就发生了变化，不再是直接运行其命令而是将CMD
  内容作为参数传递给ENTRYPOINT指令，组合为 <ENTRYPOINT> "<CMD>"
  
# 实操：Dockerfile编写增强centos镜像
- 要求：centos7镜像具备 vim+ifconfig+jdk8
- 编写：准备编写Dockerfile文件，D要大写

```bash
FROM centos:7
MAINTAINER mildlamb<190@qq.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH

# 安装vim编辑器
RUN yum -y install vim
# 安装ifconfig命令查看网络IP
RUN yum -y install net-tools
# 安装 java8以及lib库
RUN yum -y install glibc.i686
RUN mkdir /usr/local/java
# 将宿主机中的jdk压缩文件拷贝到docker镜像中
ADD ./jdk-8u202-linux-x64.tar.gz /usr/local/java/
# 配置java环境变量
ENV JAVA_HOME /usr/local/java/jdk1.8.0_202
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH $JAVA_HOME/bin:$PATH

# EXPOSE 80

CMD echo $MYPATH
CMD echo "success ....... ok"
CMD /bin/bash
```
  
- 构建 ：docker build -t 新镜像名字:TAG .  , 注意最后面有个 "."
```bash
[root@VM-4-16-centos myfile]# docker build -t centos-java8:1.0 .
```
