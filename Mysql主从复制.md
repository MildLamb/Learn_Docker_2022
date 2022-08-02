# 配置一主一从mysql
1. 新建主服务器容器实例3307
```bash
[root@VM-4-16-centos ~]# docker run -p 3307:3306 --name mysql-master \
> -v /mildlamb/mydata/mysql-master/log:/var/log/mysql \
> -v /mildlamb/mydata/mysql-master/data:/var/lib/mysql \
> -v /mildlamb/mydata/mysql-master/conf:/etc/mysql \
> -e MYSQL_ROOT_PASSWORD=mysqlpwd \
> -d mysql:5.7
```
2. 编写主机的my.cnf , 在宿主机对应master的conf目录下
```bash
[mysqld]
## 设置server_id,同一局域网中需要唯一
server_id=101
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql
## 开启二进制日志功能
log-bin=mall-mysql-bin
## 设置二进制日志使用内存大小(事务)
binlog_cache_size=1M
## 设置使用的二进制日志格式(mixed,statement,row)
binlog_format=mixed
## 二进制日志过期清理时间.默认值为0，表示不清理
expire_logs_days=7
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断
## 如: 1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
```
3. 重启mysql-master容器
```bash
[root@VM-4-16-centos conf]# docker restart mysql-master
mysql-master
```
4. 创建用户允许复制主机数据
```bash
# 创建用户
create user 'slave'@'%' identified by '123456';
# 授权
grant replication slave,replication client on *.* to 'slave'@'%';
```
5. 新建从服务器3308
```bash
[root@VM-4-16-centos ~]# docker run -p 3308:3306 --name mysql-slave \
> -v /mildlamb/mydata/mysql-slave/log:/var/log/mysql \
> -v /mildlamb/mydata/mysql-slave/data:/var/lib/mysql \
> -v /mildlamb/mydata/mysql-slave/conf:/etc/mysql \
> -e MYSQL_ROOT_PASSWORD=mysqlpwd \
> -d mysql:5.7
```
6. 编写从机的my.cnf , 在宿主机对应slave的conf目录下
```bash
[mysqld]
## 设置server_id,同一局域网中需要唯一
server_id=102
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql
## 开启二进制日志功能
log-bin=mall-mysql-slave1-bin
## 设置二进制日志使用内存大小(事务)
binlog_cache_size=1M
## 设置使用的二进制日志格式(mixed,statement,row)
binlog_format=mixed
## 二进制日志过期清理时间.默认值为0，表示不清理
expire_logs_days=7
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断
## 如: 1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
## relay_log配置中继日志
relay_log=mall-mysql-relay-bin
## log_slave_updates表示slave将复制事件写进自己的二进制日志
log_slave_updates=1
## slave设置为只读(具有super权限的用户除外)
read_only=1
```
7. 重启slave实例
```bash
docker restart mysql-slave
```
8. 在主数据库中查看主从同步状态
```bash
show master status;
```
9. 进入从机mysql-slave容器
10. 在从数据库中配置主从复制
```bash
change master to master_host='10.0.4.16',master_user='slave',master_password='123456',
master_port=3307,master_log_file='mall-mysql-bin.000001',master_log_pos=617,master_connect_retry=30
```
  - master_host：主数据库的ip地址
  - master_port：主数据库的运行端口
  - master_user：在主数据库创建的用于同步数据的用户名称
  - master_password：在主数据库创建的用于同步数据的用户的密码
  - master_log_file：指定从数据库要复制数据的日志文件，通过查看主数据库的状态，获取file参数
  - master_log_pos：指定从数据库从哪个位置开始复制数据，通过查看主数据库的状态，获取position参数
  - master_connect_retry：连接失败重试的时间间隔，单位为秒

11. 在从数据库总查看主从同步状态
```bash
show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 10.0.4.16
                  Master_User: slave
                  Master_Port: 3307
                Connect_Retry: 30
              Master_Log_File: mall-mysql-bin.000001
          Read_Master_Log_Pos: 617
               Relay_Log_File: mall-mysql-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mall-mysql-bin.000001
             Slave_IO_Running: No
            Slave_SQL_Running: No
            ...     ...
```
12. 在从数据库中开启主从同步
```bash
start slave;
```
13. 在从数据库重新查看主从同步状态
```bash
mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 10.0.4.16
                  Master_User: slave
                  Master_Port: 3307
                Connect_Retry: 30
              Master_Log_File: mall-mysql-bin.000001
          Read_Master_Log_Pos: 617
               Relay_Log_File: mall-mysql-relay-bin.000002
                Relay_Log_Pos: 325
        Relay_Master_Log_File: mall-mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
            ...   ...
```
14. 主从复制测试（主库创建表插入数据，从库查看）
