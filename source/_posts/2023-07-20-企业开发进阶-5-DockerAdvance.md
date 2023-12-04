---
title: 企业开发进阶-5-DockerAdvance
date: 2023-07-20 17:35:24
tags: 
  - Docker
categories: 
  - Technology
---


# Dockerfile

Dockerfile 是用来构建 Docker 镜像的文本文件，是由一条条构建镜像所需的指令和参数构成的脚本。

> 三部曲

1. 编写 Dockerfile 文件
2. docker build 命令构建镜像
3. docker run 依镜像运行容器实例

## 基础知识

1. 每条保留字指令都必须为大写字母且后面要跟随至少一个参数
2. 指令按照从上到下，顺序执行
3. `#` 表示注释
4. 每条指令都会创建一个新的镜像层并对镜像进行提交

## 执行 Dockerfile 大致流程

1. docker 从基础镜像运行一个容器
2. 执行一条指令并对容器作出修改
3. 执行类似 docker commit 的操作提交一个新的镜像层
4. docker 再基于刚提交的镜像运行一个新容器
5. 执行 dockerfile 中的下一条指令直到所有指令都执行完成

> 小结

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-34.png)

* **Dockerfile** 定义了进程需要的一切东西。Dockerfile 涉及的内容包括执行代码或者是文件、环境变量、依赖包、运行时环境、动态链接库、操作系统的发行版、服务进程和内核进程

* **Docker 镜像**，在用 Dockerfile 定义一个文件之后，docker build 时会产生一个 Docker 镜像，当运行 Docker 镜像时会真正开始提供服务;

* Docker 容器，容器是直接提供服务的

## 常用保留字指令

[Tomcat 8 Dockerfile](https://github.com/docker-library/tomcat)

### FROM

基础镜像，当前新镜像是基于哪个镜像的，指定一个已经存在的镜像作为模板，第一条必须是 from

### MAINTAINER

镜像维护者的姓名和邮箱地址

### RUN

容器构建时需要运行的命令，RUN 是在 `docker build` 时运行

> shell 格式

```shell
RUN <命令行命令>

RUN yum -y install vim
```

> exec 格式

```text
RUN ["可执行文件", "参数1", "参数2"]
```

### EXPOSE

当前容器对外暴露出的端口

### WORKDIR

指定在创建容器后，终端默认登陆的进来工作目录，一个落脚点

### USER

指定该镜像以什么样的用户去执行，如果都不指定，默认是 root

### ENV

用来在构建镜像过程中设置环境变量

```shell
ENV MY_PATH /usr/mytest
这个环境变量可以在后续的任何RUN指令中使用，这就如同在命令前面指定了环境变量前缀一样；
也可以在其它指令中直接使用这些环境变量，
 
比如：WORKDIR $MY_PATH
```

### ADD

将宿主机目录下的文件拷贝进镜像**且会自动处理 URL 和解压 tar 压缩包**

### COPY

类似 ADD，拷贝文件和目录到镜像中。
将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置

```bash
COPY src dest
```

* `<src源路径>`：源文件或者源目录
* `<dest目标路径>`：容器内的指定路径，该路径不用事先建好，路径不存在的话，会自动创建。

### VOLUME

容器数据卷，用于数据保存和持久化工作

### CMD

指定容器启动后的要干的事情

> shell 格式

```shell
CMD <命令>
```

> exec 格式

```text
CMD ["可执行文件", "参数1", "参数2" ...]
```

> 参数列表格式

`CMD ["参数1", "参数2" ...]`，在指定了 `ENTRYPOINT` 指令后，用 `CMD` 指定具体的参数

Dockerfile 中可以有多个 CMD 指令，**但只有最后一个生效，CMD 会被 docker run 之后的参数替换**

> CMD 和 RUN 区别

* CMD 是在 docker run 时运行
* RUN 是在 docker build 时运行

### ENTRYPOINT

用来指定一个容器启动时要运行的命令，类似于 CMD 指令，但是 **ENTRYPOINT 不会被 docker run 后面的命令覆盖，而且这些命令行参数会被当作参数送给 ENTRYPOINT 指令指定的程序**

> 命令格式

```bash
ENTRYPOINT ["<executable>", "<param1>", "<param2>"]
```

* ENTRYPOINT 可以和 CMD 一起用，一般是变参才会使用 CMD ，这里的 CMD 等于是在给 ENTRYPOINT 传参。

* 当指定了 ENTRYPOINT 后，CMD 的含义就发生了变化，不再是直接运行其命令而是将 CMD 的内容作为参数传递给 ENTRYPOINT 指令，他两个组合会变成 

  ```bash
  <ENTRYPOINT>"<CMD>"
  ```

> 假设已通过 Dockerfile 构建了 nginx:test 镜像

```bash
FROM nginx

ENTRYPOINT ["nginx". "-c"] # 定参
CMD ["/etc/nginx/nginx.conf"] # 变参
```

| 是否传参         | 按照dockerfile编写执行         | 传参运行                                                  |
| ---------------- | ------------------------------ | --------------------------------------------------------- |
| Docker命令       | docker run nginx:test          | docker run nginx:test -c nginx -c **/etc/nginx/new.conf** |
| 衍生出的实际命令 | ngnix -c /etc/nginx/nginx.conf | nginx -c **/etc/nginx/new.conf**                          |

## 案例说明

> 制作centos镜像
>
> * 具备vim
> * 具备ifconfig
> * 具备java8

**1.编写Dockerfile文件**

```bash
mkdir myfile
vim Dockerfile
```

```bash
FROM centos
MAINTAINER CyanChau<https://www.cyanzzy.github.io>

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN cd /etc/yum.repos.d/
RUN sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
RUN sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*

# 安装vim编辑器
RUN yum -y install vim
# 安装ifconfig命令查看网络IP
RUN yum -y install net-tools
#安装java8及lib库
RUN yum -y install glibc.i686
RUN mkdir /usr/local/java
# ADD 是相对路径jar,把jdk-8u171-linux-x64.tar.gz添加到容器中,安装包必须要和Dockerfile文件在同一位置
ADD jdk-8u171-linux-x64.tar.gz /usr/local/java/
# 配置java环境变量
ENV JAVA_HOME /usr/local/java/jdk1.8.0_171
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH $JAVA_HOME/bin:$PATH

EXPOSE 80 

CMD echo $MYPATH
CMD echo "success--------------ok"
CMD /bin/bash
```

**2.构建** 注意，上面 TAG 后面有个空格，有个点

```bash
docker build -t 新镜像名字:TAG .

docker build -t centosjava8:1.5 .	
```

**3.运行**

```bash
docker run -it 新镜像名字:TAG 
docker run -it centosjava8:1.5 /bin/bash
```

# 虚悬镜像

仓库名、标签都是 `<none>` 的镜像，俗称 dangling image

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-35.png)

> 查看虚悬镜像

```bash
docker image ls -f dangling=true
```

> 删除虚悬镜像

```bash
docker image prune
```

# Docker && 微服务

> **新建微服务模块**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.6</version>
        <relativePath/>
    </parent>

    <groupId>com.cyan</groupId>
    <artifactId>docker-boot</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <junit.version>4.12</junit.version>
        <log4j.version>1.2.17</log4j.version>
        <lombok.version>1.16.18</lombok.version>
        <mysql.version>5.1.47</mysql.version>
        <druid.version>1.1.16</druid.version>
        <mapper.version>4.1.5</mapper.version>
        <mybatis.spring.boot.version>1.3.0</mybatis.spring.boot.version>
    </properties>

    <dependencies>
        <!--SpringBoot通用依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--监控-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!--打包插件-->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <!--责处理项目资源文件并拷贝到输出目录，如果有额外的资源文件目录则需要配置-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <version>3.3.0</version>
            </plugin>
        </plugins>
    </build>
</project>
```

```yml
server:
  port: 6001
```

```java
@SpringBootApplication
public class DockerApplication {

    public static void main(String[] args) {
        SpringApplication.run(DockerApplication.class ,args);
    }
}
```

```java
@RestController
@RequestMapping("/order")
public class OrderController {

    @Value("${server.port}")
    private String port;

    @RequestMapping("/docker")
    public String docker() {
        return "Hello Docker" + "\t" + port + "\t" + UUID.randomUUID().toString();
    }

    @GetMapping("/index")
    public String index() {
        return "Server Port:\t" + port + "\t" + UUID.randomUUID().toString();
    }

}
```

> **通过 dockerfile 发布微服务部署到 docker 容器**

**1.IDEA 工具里面搞定微服务 jar 包**

**2.编写 Dockerfile，将微服务 jar 包和 Dockerfile 文件上传到同一个目录下/mydocker**

```bash
# 基础镜像使用java
FROM java:8
# 作者
MAINTAINER CyanChau <https://www.cyanzzy.github.io>
# VOLUME 指定临时文件目录为/tmp，在主机/var/lib/docker目录下创建了一个临时文件并链接到容器的/tmp
VOLUME /tmp
# 将jar包添加到容器中并更名为docker_test.jar
ADD docker-boot-1.0-SNAPSHOT.jar docker_test.jar
# 运行jar包
RUN bash -c 'touch /docker_test.jar'
ENTRYPOINT ["java","-jar","/docker_test.jar"]
# 暴露6001端口作为微服务
EXPOSE 6001
```

**3.构建镜像**

```bash
docker build -t docker_test:1.6 .
```

**4.运行容器**

```bash
docker run -d -p 6001:6001 docker_test:1.6
```

# Docker 网络

> Docker 不启动，默认网络情况

* ens33
* lo
* virbr0

> Docker 启动后，网络情况

会产生一个名为 docker0 的虚拟网桥

## 常用基本命令

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-36.png)

| 命令                                | 说明           |
| ----------------------------------- | -------------- |
| `docker network ls`                   | 查看网络       |
| `docker network inspect  XXX网络名字` | 查看网络源数据 |
| `docker network rm XXX网络名字`       | 删除网络       |

## docker network 作用

* 容器间的互联和通信以及端口映射
* 容器 IP 变动时可以通过服务名直接网络通信而不受到影响

## docker network 模式

| 网络模式  | 指定                                       | 说明                                                         |
| --------- | ------------------------------------------ | ------------------------------------------------------------ |
| `bridge`    | 使用`--network  bridge`指定，默认使用docker0 | 为每个容器分配、设置IP，并将容器连接到一个`docker0`虚拟网桥，默认为该模式 |
| `host`      | 使用`--network host`指定                     | 容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口 |
| `none`      | 使用`--network none`指定                     | 容器有独立的network namespace，但没有对其进行任何网络设置，如分配veth pair和网桥连接、IP等 |
| `container` | 使用`--network container:NAME或者容器ID`指定 | 新创建的容器不会创建自己的网卡和配置自己的IP，而是和一个指定的容器共享IP、端口范围等 |

> 容器实例内默认网络 IP 生产规则

docker 容器内部的 ip 是有可能会发生改变的

### bridge

Docker 服务默认会创建一个 docker0 网桥（其上有一个 docker0 内部接口），该桥接网络的名称为 docker0，它在内核层连通了其他的物理或虚拟网卡，**这就将所有容器和本地主机都放到同一个物理网络**。Docker 默认指定了 docker0 接口 的 IP 地址和子网掩码，**让主机和容器之间可以通过网桥相互通信**。

```bash
# 查看 bridge 网络的详细信息，并通过 grep 获取名称项
docker network inspect bridge | grep name

ifconfig
```

* Docker 使用 Linux 桥接，在宿主机虚拟一个 Docker 容器网桥(docker0)，Docker 启动一个容器时会根据 Docker 网桥的网段分配给容器一个 IP 地址，称为 Container-IP，同时 Docker 网桥是每个容器的默认网关。因为在同一宿主机内的容器都接入同一个网桥，这样容器之间就能够通过容器的Container-IP直接通信

* docker run 时未指定 network 则默认使用的网桥模式就是 bridge，即 `docker0`

* 网桥 docker0 创建一对对等虚拟设备接口一个叫 `veth`，另一个叫 `eth0`，成对匹配
  * 整个宿主机的网桥模式都是 `docker0`，类似一个交换机有一堆接口，每个接口叫 `veth`，在本地主机和容器内分别创建一个虚拟接口，并让他们彼此联通（这样一对接口叫 veth pair）；

  * 每个容器实例内部也有一块网卡，每个接口叫 `eth0`；
  * `docker0` 上面的每个 `veth` 匹配某个容器实例内部的 `eth0`

通过上述，将宿主机上的所有容器都连接到这个内部网络上，两个容器在同一个网络下，会从这个网关下各自拿到分配的 ip，此时两个容器的网络是互通的。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-37.png)

```bash
docker run -d -p 8081:8080   --name tomcat81 billygoo/tomcat8-jdk8
docker run -d -p 8082:8080   --name tomcat82 billygoo/tomcat8-jdk8
```

> 两两匹配验证

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-38.png)

### host

* 该模式下直接使用宿主机的 IP 地址与外界进行通信，不再需要额外进行 NAT 转换
* 容器将不会获得一个独立的 Network Namespace， 而是和宿主机共用一个 Network Namespace
* 容器将不会虚拟出自己的网卡而是使用宿主机的 IP 和端口

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-39.png)

> 错误示范：

```bash
docker run -d -p 8083:8080 --network host --name tomcat83 billygoo/tomcat8-jdk8
```

* docker 启动时指定 `--network=hos`t 或 `-net=host`，如果还指定了 `-p` 映射端口，此时就会有警告，
  并且**通过-p 设置的参数将不会起到任何作用**，**端口号会以主机端口号为主，重复时则递增**。
* 解决方案：使用 docker 的其他网络模式，例如 `--network=bridge`

> 正确示范：

```bash
docker run -d --network host --name tomcat83 billygoo/tomcat8-jdk8
```

> 容器实例内部

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-40.png)

> 如果没有设置 `-p` 的端口映射，那么如何访问启动的 tomcat83 ?

Q：http://宿主机IP:8080/

在CentOS里面用默认的火狐浏览器访问容器内的tomcat83看到访问成功，因为此时容器的IP借用主机的，
所以容器共享宿主机网络IP，这样的好处是外部主机与容器可以直接通信。

### none

该模式下禁用网络功能，只有 lo 标识（即127.0.0.1，表示本地回环）。在 none 模式下，并不为 Docker 容器进行任何网络配置，该 Docker 容器没有网卡、IP、路由等信息，只有一个 lo，需要自行为 Docker 容器添加网卡、配置 IP 等。

```bash
docker run -d -p 8084:8080 --network none --name tomcat84 billygoo/tomcat8-jdk8
```

### container

新创建的容器不会创建自己的网卡，配置自己的 IP，而是和一个指定的容器共享 IP、端口范围等。两个容器除网络方面，其他的如文件系统、进程列表等还是隔离的。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-41.png)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-42.png)

### 自定义网络

自定义桥接网络，自定义网络默认使用的是桥接网络 bridge

```bash
docker network create es-net
docker run -d -p 8081:8080 --network es-net  --name tomcat81 billygoo/tomcat8-jdk8
docker run -d -p 8082:8080 --network es-net  --name tomcat81 billygoo/tomcat8-jdk8
```

自定义网络本身就维护好了主机名和 ip 的对应关系（ip 和域名都能 ping 通）

# Docker Compose 容器编排

**Compose 是 Docker 公司推出的一个工具软件，可以管理多个 Docker 容器组成一个应用**。你需要定义一个 YAML 格式的配置文件 docker-compose.yml，写好多个容器之间的调用关系。然后只要一个命令，就能同时启动/关闭这些容器

Compose 允许用户通过一个单独的 docker-compose.yml 模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目。可以很容易地用一个配置文件定义一个多容器的应用，然后使用一条指令安装这个应用的所有依赖、完成构建。Docker-Compose **解决了容器与容器之间管理编排的问题**。

[compose-file 官网文档](https://docs.docker.com/compose/compose-file/compose-file-v3/) 	

[compose 下载地址](https://docs.docker.com/compose/install/)

> 安装步骤

```bash
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

> 核心概念

* docker-compose.yml

* 服务（service）：一个个应用容器实例，比如订单微服务、库存微服务、mysql 容器、nginx 容器或者 redis 容器
* 工程（project）：由一组关联的应用容器组成的一个完整业务单元，在 docker-compose.yml 文件中定义。

> 三部曲

1. 编写 Dockerfile 定义各个微服务应用并构建出对应的镜像文件
2. 使用 docker-compose.yml 定义一个完整业务单元，安排好整体应用中的各个容器服务。
3. 最后，执行 `docker-compose up` 命令 来启动并运行整个应用程序，完成一键部署上线

## Compose 常用命令

| 命令                                 | 说明                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| `docker-compose -h`                    | 查看帮助                                                     |
| `docker-compose up`                    | 启动所有docker-compose服务                                   |
| `docker-compose up -d`                 | 启动所有docker-compose服务并后台运行                         |
| `docker-compose down`                  | 停止并删除容器、网络、卷、镜像。                             |
| `docker-compose exec  yml里面的服务id` | 进入容器实例内部  docker-compose exec docker-compose.yml文件中写的服务id /bin/bash |
| `docker-compose ps`                    | 展示当前docker-compose编排过的运行的所有容器                 |
| `docker-compose top`                   | 展示当前docker-compose编排过的容器进程                       |
| `docker-compose logs yml里面的服务id`  | 查看容器输出日志                                             |
| `docker-compose config`                | 检查配置                                                     |
| `docker-compose config -q`             | 检查配置，有问题才有输出                                     |
| `docker-compose restart`               | 重启服务                                                     |
| `docker-compose start`                 | 启动服务                                                     |
| `docker-compose stop`                  | 停止服务                                                     |

## Compose 编排微服务

**1.改造微服务，使用mvn package命令将微服务形成新的jar包并上传到Linux服务器/mydocker目录下**

（第一次改造：添加mysql和redis服务）

```yml
server.port=6001
# ========================alibaba.druid相关配置=====================
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://192.168.111.169:3306/db2021?useUnicode=true&characterEncoding=utf-8&useSSL=false
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.druid.test-while-idle=false
# ========================redis相关配置=====================
spring.redis.database=0
spring.redis.host=192.168.111.169
spring.redis.port=6379
spring.redis.password=
spring.redis.lettuce.pool.max-active=8
spring.redis.lettuce.pool.max-wait=-1ms
spring.redis.lettuce.pool.max-idle=8
spring.redis.lettuce.pool.min-idle=0
# ========================mybatis相关配置===================
mybatis.mapper-locations=classpath:mapper/*.xml
mybatis.type-aliases-package=com.atguigu.docker.entities
# ========================swagger=====================
spring.swagger2.enabled=true
```



**2.编写docker-compose.yml文件**

```bash
vim  docker-compose.yml
```



```yml
version: "3"
 
services:
  microService:
    image: zzyy_docker:1.6
    container_name: ms01
    ports:
      - "6001:6001"
    volumes:
      - /app/microService:/data
    networks: 
      - atguigu_net 
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
      - atguigu_net
    command: redis-server /etc/redis/redis.conf
 
  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: '123456'
      MYSQL_ALLOW_EMPTY_PASSWORD: 'no'
      MYSQL_DATABASE: 'db2021'
      MYSQL_USER: 'zzyy'
      MYSQL_PASSWORD: 'zzyy123'
    ports:
       - "3306:3306"
    volumes:
       - /app/mysql/db:/var/lib/mysql
       - /app/mysql/conf/my.cnf:/etc/my.cnf
       - /app/mysql/init:/docker-entrypoint-initdb.d
    networks:
      - atguigu_net
    command: --default-authentication-plugin=mysql_native_password #解决外部无法访问
 
networks: 
   atguigu_net: 
```

**改造微服务**

(第二次改造：通过服务名访问，IP无关)

```yml
server.port=6001

# ========================alibaba.druid相关配置=====================
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
#spring.datasource.url=jdbc:mysql://192.168.111.169:3306/db2021?useUnicode=true&characterEncoding=utf-8&useSSL=false
spring.datasource.url=jdbc:mysql://mysql:3306/db2021?useUnicode=true&characterEncoding=utf-8&useSSL=false
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.druid.test-while-idle=false

# ========================redis相关配置=====================
spring.redis.database=0
#spring.redis.host=192.168.111.169
spring.redis.host=redis
spring.redis.port=6379
spring.redis.password=
spring.redis.lettuce.pool.max-active=8
spring.redis.lettuce.pool.max-wait=-1ms
spring.redis.lettuce.pool.max-idle=8
spring.redis.lettuce.pool.min-idle=0

# ========================mybatis相关配置===================
mybatis.mapper-locations=classpath:mapper/*.xml
mybatis.type-aliases-package=com.atguigu.docker.entities

# ========================swagger=====================
spring.swagger2.enabled=true
```

**mvn package命令将微服务形成新的jar包并上传到Linux服务器/mydocker目录下，然后编写Dockerfile文件并构建镜像**

**3.执行 docker-compose up或者执行 docker-compose up -d**