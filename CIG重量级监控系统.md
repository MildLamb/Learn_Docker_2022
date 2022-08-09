# CIG
- CAdvisor监控收集 + influxDB存储数据 + Granfana展示图表

## CAdvisor
CAdvisor是一个容器资源监控工具，包括容器的内存，CPU，网络IO，磁盘IO等监控，同时提供了一个WEB页面  
用于查看容器的实时运行状态，CAdvisor默认存储2分钟的数据，而且只针对单物理机。不过，CAdvisor提供了  
很多数据集成接口，支持InfluxDB，Redis，Kafka，ElasticSearch等集成，可以加上对应配置将监控数据  
发往这些数据库存储起来

- CAdvisor功能主要有两点
  - 展示Host和容器两个层次的监控数据
  - 展示历史变化数据

## CIG结合compose一键搭建监控平台
- compose.yml
```bash
version: '3.1'

volumes:
  grafana_data: {}

services:
  influxdb:
	# tutum/influxdb 相比influxdb多了web可视化视图。但是该镜像已被标记为已过时
    image: tutum/influxdb:0.9
    restart: always
    environment:
      - PRE_CREATE_DB=cadvisor
    ports:
      - "8083:8083"         # 数据库web可视化页面端口
      - "8086:8086"         # 数据库端口
    volumes:
      - ./data/influxdb:/data

  cadvisor:
    image: google/cadvisor:v0.32.0
    links:
      - influxdb:influxsrv
    command:
      - -storage_driver=influxdb
      - -storage_driver_db=cadvisor
      - -storage_driver_host=influxsrv:8086
    restart: always
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro

  grafana:
    image: grafana/grafana:8.5.2
    user: '104'
    restart: always
    links:
      - influxdb:influxsrv
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - HTTP_USER=admin
      - HTTP_PASS=admin
      - INFLUXDB_HOST=influxsrv
      - INFLUXDB_PORT=8086
      - INFLUXDB_NAME=cadvisor
      - INFLUXDB_USER=root
      - INFLUXDB_PASS=root
```
