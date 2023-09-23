---
title: 企业开发进阶-5-DockerInstall
date: 2023-07-20 17:35:06
tags: 
  - Docker
categories: 
  - Technology
---

# 安装 Docker

## 安装 Docker

[安装文档](https://docs.docker.com/engine/install/centos/)

> 安装步骤

1. 确定机器是CentOS7及以上版本

   ```bash
   cat /etc/redhat-release
   ```

2. 卸载旧版本

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

3. yum安装gcc

   ```bash
   yum -y install gcc
   yum -y install gcc-c++
   ```

4. Set up the repository

   ```bash
   yum install -y yum-utils
   ```

   

   ![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-04.png)

5. 设置镜像仓库

   ```bash
   yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   ```

6. 更新yum软件包索引

   ```bash
   yum makecache fast
   ```

7. Install Docker Engine

   ```bash
   yum -y install docker-ce docker-ce-cli containerd.io
   ```

8. 启动docker

   ```bash
   systemctl start docker
   ps -ef | grep docker
   ```

9. 测试Docker

   ```bash
   docker version
   
   docker run hello-world
   ```

10. 卸载Docker

    ```bash
    systemctl stop docker 
    
    yum remove docker-ce docker-ce-cli containerd.io
    
    rm -rf /var/lib/docker
    
    rm -rf /var/lib/containerd
    ```

## 配置阿里云镜像加速

1. 登录阿里云

2. 进入控制台，选择 容器镜像服务，在镜像根据栏目下选择镜像加速器

   ```bash
   mkdir -p /etc/docker
   cd /etc/docker
   
   tee /etc/docker/daemon.json <<-'EOF'
   {
     "registry-mirrors": ["https://djimxgok.mirror.aliyuncs.com"]
   }
   EOF
   ```

3. 重启服务

   ```bash
   systemctl daemon-reload
   systemctl restart docker
   ```

4. 测试

   ```bash
   docker run hello-world
   ```

> run发生了什么

# Docker 常规安装

1. 搜索镜像
2. 拉取镜像
3. 查看镜像
4. 启动镜像：服务端口映射
5. 停止容器
6. 移除容器

## 安装 Tomcat

> docker hub上面查找tomcat镜像

```bash
docker search tomcat
```

> 从docker hub上拉取tomcat镜像到本地

```bash
docker pull tomcat
```

> docker images查看是否有拉取到的tomcat

```bash
docker images tomcat
```

> 使用tomcat镜像创建容器实例(也叫运行镜像)

```bash
# 交互运行
docker run -it -p 8080:8080 tomcat
# 或者后台运行
docker run -d -p 8080:8080 --name t1 tomcat

docker ps
```

* -p 小写，主机端口:docker容器端口
* -P 大写，随机分配端口
* i：交互
* t：终端
* d：后台

> 访问tomcat首页出现404

1. 可能没有映射端口或者没有关闭防火墙
2. 把webapps.dist目录换成webapps（启动tomcat后查看webapps为空）

```bash
[root@cyan ~]# docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED         SMES
0b15077c9db2   tomcat                 "catalina.sh run"        5 seconds ago   U
e3cf26fb0003   kibana:7.12.1          "/bin/tini -- /usr/l…"   46 hours ago    Ubana
057f6a0038ce   elasticsearch:7.12.1   "/bin/tini -- /usr/l…"   46 hours ago    U
[root@cyan ~]# docker exec -it 0b15077c9db2 /bin/bash
root@0b15077c9db2:/usr/local/tomcat# pwd
/usr/local/tomcat
root@0b15077c9db2:/usr/local/tomcat# ls -l
total 160
-rw-r--r-- 1 root root 18994 Dec  2  2021 BUILDING.txt
-rw-r--r-- 1 root root  6210 Dec  2  2021 CONTRIBUTING.md
-rw-r--r-- 1 root root 60269 Dec  2  2021 LICENSE
-rw-r--r-- 1 root root  2333 Dec  2  2021 NOTICE
-rw-r--r-- 1 root root  3378 Dec  2  2021 README.md
-rw-r--r-- 1 root root  6905 Dec  2  2021 RELEASE-NOTES
-rw-r--r-- 1 root root 16517 Dec  2  2021 RUNNING.txt
drwxr-xr-x 2 root root  4096 Dec 22  2021 bin
drwxr-xr-x 1 root root  4096 Jul 18 07:46 conf
drwxr-xr-x 2 root root  4096 Dec 22  2021 lib
drwxrwxrwx 1 root root  4096 Jul 18 07:46 logs
drwxr-xr-x 2 root root  4096 Dec 22  2021 native-jni-lib
drwxrwxrwx 2 root root  4096 Dec 22  2021 temp
drwxr-xr-x 2 root root  4096 Dec 22  2021 webapps
drwxr-xr-x 7 root root  4096 Dec  2  2021 webapps.dist
drwxrwxrwx 2 root root  4096 Dec  2  2021 work
root@0b15077c9db2:/usr/local/tomcat# cd webapps
root@0b15077c9db2:/usr/local/tomcat/webapps# ls -l
total 0
```

```bash
root@0b15077c9db2:/usr/local/tomcat# rm -r webapps
root@0b15077c9db2:/usr/local/tomcat# ls -l
total 156
-rw-r--r-- 1 root root 18994 Dec  2  2021 BUILDING.txt
-rw-r--r-- 1 root root  6210 Dec  2  2021 CONTRIBUTING.md
-rw-r--r-- 1 root root 60269 Dec  2  2021 LICENSE
-rw-r--r-- 1 root root  2333 Dec  2  2021 NOTICE
-rw-r--r-- 1 root root  3378 Dec  2  2021 README.md
-rw-r--r-- 1 root root  6905 Dec  2  2021 RELEASE-NOTES
-rw-r--r-- 1 root root 16517 Dec  2  2021 RUNNING.txt
drwxr-xr-x 2 root root  4096 Dec 22  2021 bin
drwxr-xr-x 1 root root  4096 Jul 18 07:46 conf
drwxr-xr-x 2 root root  4096 Dec 22  2021 lib
drwxrwxrwx 1 root root  4096 Jul 18 07:46 logs
drwxr-xr-x 2 root root  4096 Dec 22  2021 native-jni-lib
drwxrwxrwx 2 root root  4096 Dec 22  2021 temp
drwxr-xr-x 7 root root  4096 Dec  2  2021 webapps.dist
drwxrwxrwx 2 root root  4096 Dec  2  2021 work
root@0b15077c9db2:/usr/local/tomcat# mv webapps.dist webapps
```

最后访问http://47.113.217.195:8080/，成功访问Tomcat

> 免修改版说明

```bash
[root@cyan ~]# docker ps

[root@cyan ~]# docker stop t1

[root@cyan ~]# docker rm -f t1 

[root@cyan ~]# docker ps
```

```bash
docker pull billygoo/tomcat8-jdk8

docker run -d -p 8080:8080 --name mytomcat8 billygoo/tomcat8-jdk8
```

## 安装MySQL

```bash
# 先查看服务器有没有mysql
ps -ef | grep mysql
```



> docker hub 查找镜像

````bash
docker search mysql
````

> 拉取镜像

```bash
docker pull mysql:5.7
```

> 使用镜像创建容器

### 简单版本

```bash
docker run -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7

docker ps 
docker exec -it 容器id /bin/bash
mysql -uroot -p

[root@cyan ~]# docker exec -it f959a65e8c95 /bin/bash
root@f959a65e8c95:/# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.36 MySQL Community Server (GPL)

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

**问题描述：**

1. docker中字符编码全是latin
2. 容器删除后数据将消失，存在隐患

### 实战版本

> 新建mysql容器实例

```bash
docker run -d -p 3306:3306 --privileged=true -v /temp/mysql/log:/var/log/mysql -v /temp/mysql/data:/var/lib/mysql -v /temp/mysql/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=root  --name mysql mysql:5.7
```

> 新建my.cnf

通过容器卷同步给mysql容器实例

```bash
[client]
default_character_set=utf8
[mysqld]
collation_server = utf8_general_ci
character_set_server = utf8
```

> 重新启动mysql容器实例再重新进入并查看字符编码

```bash
dcoker restart mysql

docker exec -it mysql bash
```

## 安装Redis

> 从docker hub上(阿里云加速器)拉取redis镜像到本地标签为6.0.8

```bash
docker pull redis:6.0.8
docker images
```

> 入门命令

```bash
docker run -d -p 6379:6379 redis:6.0.8

docker exec -it containerId /bin/bash
redis -cli
```

> 命令提醒：容器卷记得加入--privileged=true

Docker挂载主机目录Docker访问出现cannot open directory .: Permission denied
解决办法：在挂载目录后多加一个--privileged=true参数即可

> 在CentOS宿主机下新建目录/app/redis

```bash
mkdir -p /app/redis
```

> 将一个redis.conf文件模板拷贝进/app/redis目录下

* /app/redis目录下修改redis.conf文件:开启redis验证    可选requirepass root

* 允许redis外地连接  必须注释掉 # bind 127.0.0.1
* daemonize no:将daemonize yes注释起来或者 daemonize no设置，因为该配置和docker run中-d参数冲突，会导致容器一直启动失败

* 开启redis数据持久化  appendonly yes  可选

> 使用redis6.0.8镜像创建容器(也叫运行镜像)

```bash
docker run  -p 6379:6379 --name myr3 --privileged=true -v /app/redis/redis.conf:/etc/redis/redis.conf -v /app/redis/data:/data -d redis:6.0.8 redis-server /etc/redis/redis.conf
```

> 测试redis-cli连接

```bash
docker exec -it 运行着Rediis服务的容器ID redis-cli
```

# Docker 复杂安装

## MySQL主从复制安装

## Redis集群安装

# Docker轻量级可视化工具Portainer

Portainer 是一款轻量级的应用，它提供了图形化界面，用于方便地管理Docker环境，包括单机环境和集群环境。

[Portainer官网](https://www.portainer.io/)   [官方文档](https://docs.portainer.io/v/ce-2.9/start/install/server/docker/linux)

```bash
docker run -d -p 8000:8000 -p 9000:9000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data     portainer/portainer
```

1. 第一次登录需创建admin，访问地址：xxx.xxx.xxx.xxx:9000
2. 设置admin用户和密码后首次登陆

3. 选择local选项卡后本地docker详细信息展示

# Docker容器监控 CAdvisor+InfluxDB+Granfana

> 原生命令

```bash
docker stats 
```


通过docker stats命令可以很方便的看到当前宿主机上所有容器的CPU,内存以及网络流量等数据，但是，docker stats统计结果只能是当前宿主机的全部容器，数据资料是实时的，没有地方存储、没有健康指标过线预警等功能

> CIG

CAdvisor监控收集+InfluxDB存储数据+Granfana展示图表

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-43.png)

## CAdvisor

CAdvisor是一个容器资源监控工具，包括容器的内存、CPU、网络IO、磁盘IO等监控，同时提供了一个WEB页面用于查看容器的实时运行状态。CAdvisor默认存储2分钟的数据，而且只是针对单物理机。不过，CAdvisor提供了很多数据集成接口，支InfluxDB，Redis，Kafka，Elasticsearch等集成，可以加上对应配置将监控数据发往这些数据库存储起来。

> CAdvisor功能主要有两点:

* 展示Host和容器两个层次的监控数据
* 展示历史变化数据。

## InfluxDB

InfluxDB是用Go语言编写的一个开源分布式时序、事件和指标数据库,无需外部依赖。
CAdvisor默认只在本机保存最近2分钟的数据，为了持久化存储数据和统一收集展示监控数据，需要将数据存储到InfluxDB中。InfluxDB是一个时序数据库，专门用于存储时序相关数据，很适合存储CAdvisor的数据。而且CAdvisor本身已经提供了InfluxDB的集成方法，丰启动容器时指定配置即可。

> lnfluxDB主要功能:

* 基于时间序列，支持与时间有关的相关函数(如最大、最小、求和等)
* 可度量性：你可以实时对大量数据进行计算;
* 基于事件：它支持任意的事件数据;

## Granfana

Grafana是一个开源的数据监控分析可视化平台，支持多种数据源配置(支持的数据源包括InfluxDB，MySQL，Elasticsearch，OpenTSDB，Graphite等)和丰富的插件及模板功能,支持图表权限控制和报警。

> Grafan主要特性

* 灵活丰富的图形化选项
* 可以混合多种风格
* 支持白天和夜间模式
* 多个数据源

## 实战

> **新建docker-compose.yml**

```yml
version: '3.1'
 
volumes:
  grafana_data: {}
 
services:
 influxdb:
  image: tutum/influxdb:0.9
  restart: always
  environment:
    - PRE_CREATE_DB=cadvisor
  ports:
    - "8083:8083"
    - "8086:8086"
  volumes:
    - ./data/influxdb:/data
 
 cadvisor:
  image: google/cadvisor
  links:
    - influxdb:influxsrv
  command: -storage_driver=influxdb -storage_driver_db=cadvisor -storage_driver_host=influxsrv:8086
  restart: always
  ports:
    - "8080:8080"
  volumes:
    - /:/rootfs:ro
    - /var/run:/var/run:rw
    - /sys:/sys:ro
    - /var/lib/docker/:/var/lib/docker:ro
 
 grafana:
  user: "104"
  image: grafana/grafana
  user: "104"
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

```bash
docker-compose config -q
```

> **启动docker-compose文件**

```bash
docker-compose up
```

> **查看服务是否启动**

```bash
docker ps
```

> **测试**

* 浏览cAdvisor收集服务，http://ip:8080/

* 浏览influxdb存储服务，http://ip:8083/
* 浏览grafana展现服务，http://ip:3000

