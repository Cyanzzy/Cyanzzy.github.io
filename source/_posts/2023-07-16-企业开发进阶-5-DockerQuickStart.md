---
title: 企业开发进阶-5-DockerQuickStart
date: 2023-07-16 18:06:33
tags: 
  - Docker
categories: 
  - Technology
swiper_index: 
---

# Docker 简介

[Docker官网](https://www.docker.com/)

[Docker官方文档](https://docs.docker.com/)

[Docker源码](https://github.com/docker/docker-ce         )

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-08.png)

## 概述

Docker 是一个开源的应用容器引擎，基于 [Go 语言](https://www.runoob.com/go/go-tutorial.html) 并遵从 Apache2.0 协议开源。

Docker 可以让开发者**打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化**。

容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app），更重要的是容器性能开销极低。

## 应用场景

- Web 应用的自动化打包和发布。
- 自动化测试和持续集成、发布。
- 在服务型环境中部署和调整数据库或其他的后台应用。
- 从头编译或者扩展现有的 OpenShift 或 Cloud Foundry 平台来搭建自己的 PaaS 环境。

## 优点

Docker 是一个用于开发，交付和运行应用程序的开放平台。Docker 使您能够将应用程序与基础架构分开，从而可以**快速交付软件**。借助 Docker，您可以与管理应用程序相同的方式来管理基础架构。通过利用 Docker 的方法来**快速交付，测试和部署代码**，您可以大大减少编写代码和在生产环境中运行代码之间的延迟。

> 1、快速，一致地交付您的应用程序

Docker 允许开发人员使用您提供的应用程序或服务的本地容器在标准化环境中工作，从而简化了开发的生命周期。

容器非常适合持续集成和持续交付（CI / CD）工作流程，请考虑以下示例方案：

- 您的开发人员在本地编写代码，并使用 Docker 容器与同事共享他们的工作。
- 他们使用 Docker 将其应用程序推送到测试环境中，并执行自动或手动测试。
- 当开发人员发现错误时，他们可以在开发环境中对其进行修复，然后将其重新部署到测试环境中，以进行测试和验证。
- 测试完成后，将修补程序推送给生产环境，就像将更新的镜像推送到生产环境一样简单。

> 2、响应式部署和扩展

Docker 是基于容器的平台，允许高度可移植的工作负载。Docker 容器可以在开发人员的本机上，数据中心的物理或虚拟机上，云服务上或混合环境中运行。

Docker 的可移植性和轻量级的特性，还可以使您轻松地完成动态管理的工作负担，并根据业务需求指示，实时扩展或拆除应用程序和服务。

> 3、在同一硬件上运行更多工作负载

Docker 轻巧快速。它为基于虚拟机管理程序的虚拟机提供了可行、经济、高效的替代方案，因此您可以利用更多的计算能力来实现业务目标。Docker 非常适合于高密度环境以及中小型部署，而您可以用更少的资源做更多的事情。

## 能做什么

比较Docker和虚拟机技术

* 传统虚拟机，虚拟出一条硬件，运行一个完整的操作系统，在该系统上运行和安装软件
* 容器内的应用直接运行在宿主机的内核，容器本身没有内核
* 每个容器间相互隔离，每个容器内部有属于自己的文件系统，互不影响

> DevOps（开发、运维）

**应用更快速的交付和部署**

传统：一堆帮助文档和安装程序

Docker：打包镜像发布测试，一键运行

**更便捷的升级和扩缩容**

使用Docker后，容易部署项目

项目打包为一个镜像，扩展服务器A 服务器B

**更简单的系统运维**

在容器化之后，开发和测试环境高度一致

**更高效的计算资源利用**

Docker是内核级别的虚拟化，可以在物理机上运行多个容器实例

## 架构

Docker 包括三个基本概念:

- **镜像（Image）**：Docker 镜像是用于创建 Docker 容器的模板，通过模板创建容器服务，容器可以创建多个
- **容器（Container）**：镜像和容器的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，**容器是镜像运行时的实体**。容器可以被创建、启动、停止、删除、暂停等。
- **仓库（Repository）**：仓库可看成一个代码控制中心，**用来保存镜像**。

Docker 使用客户端-服务器 (C/S) 架构模式，使用远程API来管理和创建Docker容器。

Docker 容器通过 Docker 镜像来创建。

> 容器与镜像的关系类似于面向对象编程中的对象与类。

| Docker | 面向对象 |
| :----- | :------- |
| 容器   | 对象     |
| 镜像   | 类       |

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-09.png)

| 概念                   | 说明                                                         |
| :--------------------- | :----------------------------------------------------------- |
| Docker 镜像(Images)    | Docker 镜像是用于创建 Docker 容器的模板，比如 Ubuntu 系统。  |
| Docker 容器(Container) | 容器是独立运行的一个或一组应用，是镜像运行时的实体。         |
| Docker 客户端(Client)  | Docker 客户端通过命令行或者其他工具使用 Docker SDK (https://docs.docker.com/develop/sdk/) 与 Docker 的守护进程通信。 |
| Docker 主机(Host)      | 一个物理或者虚拟的机器用于执行 Docker 守护进程和容器。       |
| Docker Registry        | Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。Docker Hub([https://hub.docker.com](https://hub.docker.com/)) 提供了庞大的镜像集合供使用。一个 Docker Registry 中可以包含多个仓库（Repository）；每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 **<仓库名>:<标签>** 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 **latest** 作为默认标签。 |
| Docker Machine         | Docker Machine是一个简化Docker安装的命令行工具，通过一个简单的命令行即可在相应的平台上安装Docker，比如VirtualBox、 Digital Ocean、Microsoft Azure。 |

## 安装 Docker

[Windows环境安装Docker](https://www.runoob.com/docker/windows-docker-install.html)

[Linux环境安装Docker](https://www.runoob.com/docker/centos-docker-install.html)

[Docker 镜像加速](https://www.runoob.com/docker/docker-mirror-acceleration.html)

## 底层原理

Docker是一个Client-Server结构的系统，Docker的守护进程运行在主机上，通过Socket从客户端访问，DockerServer接受到Docker-Client的指令，就会执行该命令

> Run的运行流程

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-10.png)

> Docker为什么比虚拟机快？

1. Docker的抽象层比虚拟机更少
2. Docker利用宿主机的内核，VM需要Guest OS

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-11.png)

# Docker 常用命令

## 帮助命令

```shell
docker version # 显示docker的版本信息
docker info # 显示docker的系统信息，包括镜像和容器的数量
docker 命令 --help # 帮助命令
```



##  镜像命令

**docker images** 查看所有本地的主机上的镜像

```shell
docker images [OPTIONS] [REPOSITORY[:TAG]]
```

```shell
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
<none>                    <none>              77af4d6b9913        19 hours ago        1.089 GB
committ                   latest              b6fa739cedf5        19 hours ago        1.089 GB
<none>                    <none>              78a85c484f71        19 hours ago        1.089 GB
docker                    latest              30557a29d5ab        20 hours ago        1.089 GB
<none>                    <none>              5ed6274db6ce        24 hours ago        1.089 GB
postgres                  9                   746b819f315e        4 days ago          213.4 MB
postgres                  9.3                 746b819f315e        4 days ago          213.4 MB
postgres                  9.3.5               746b819f315e        4 days ago          213.4 MB
postgres                  latest              746b819f315e        4 days ago          213.4 MB
```

> 解释

| 名称       | 说明         |
| ---------- | ------------ |
| REPOSITORY | 镜像仓库源   |
| TAG        | 镜像标签     |
| IMAGE ID   | 镜像ID       |
| CREATED    | 镜像创建时间 |
| SIZE       | 镜像大小     |

> 可选项

| Name, shorthand   | Description                                             |
| ----------------- | ------------------------------------------------------- |
| `--all` , `-a`    | **Show all images** (default hides intermediate images) |
| `--digests`       | Show digests                                            |
| `--filter` , `-f` | Filter output based on conditions provided              |
| `--format`        | Pretty-print images using a Go template                 |
| `--no-trunc`      | Don't truncate output                                   |
| `--quiet` , `-q`  | **Only show image IDs**                                 |

**docker search**  搜索镜像

```shell
docker search [OPTIONS] TERM
```

> 可选项

| Name, shorthand   | Default | Description                                |
| ----------------- | ------- | ------------------------------------------ |
| `--filter` , `-f` |         | Filter output based on conditions provided |
| `--format`        |         | Pretty-print search using a Go template    |
| `--limit`         | `25`    | Max number of search results               |
| `--no-trunc`      |         | Don't truncate output                      |

**docker pull** 下载镜像

```shell
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

```shell
docker pull debian

Using default tag: latest # 若不写tag 默认 latest
latest: Pulling from library/debian
fdd5d7827f33: Pull complete
a3ed95caeb02: Pull complete
Digest: sha256:e7d38b3517548a1c71e41bffe9c8ae6d6d29546ce46bf62159837aad072c90aa
Status: Downloaded newer image for debian:latest
```

**docker rmi** 删除镜像

```shell
docker rmi [OPTIONS] IMAGE [IMAGE...]
```

> 可选项

| Name, shorthand  | Description                    |
| ---------------- | ------------------------------ |
| `--force` , `-f` | Force removal of the image     |
| `--no-prune`     | Do not delete untagged parents |

## 容器命令

**docker run** 新建并启动容器

```shell
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

> 参数说明 

| 参数            | 说明                               |
| --------------- | ---------------------------------- |
| `--name="Name"` | 容器名称                           |
| `-d`            | 后台方式运行                       |
| `-it`           | 使用交互方式运行，进入容器查看内容 |
| `-p`            | 指定容器的端口                     |
| `-P`            | 随机指定端口                       |

> 实例

```shell
# 获取centos镜像
docker pull centos
# 启动容器
docker run -it centos /bin/bash
# 查看容器内的centos
ls
# 从容器中退回主机
exit
```

**docker ps**列出所有运行的容器

```shell
docker ps [OPTIONS]
```

> 可选项

| 可选项  | 说明                                             |
| ------- | ------------------------------------------------ |
| ``      | 列出当前正在运行的容器                           |
| `-a`    | Show all containers (default shows just running) |
| `-n=？` | 显示最近创建的容器                               |
| `-q`    | 只显示容器的编号                                 |

退出容器

| 命令     | 说明             |
| -------- | ---------------- |
| exit     | 停止容器，并退出 |
| Ctrl+P+Q | 不停止容器，退出 |

**docker rm** 删除容器

```shell
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

```shell
docker rm 容器id # 删除指定的容器，不能删除正在运行的容器，如果强制删除 rm -f
docker rm -f $(docker ps -aq) # 删除所有的容器
docker ps -a -q|xargs docker rm # 删除所有的容器
```

启动和停止容器

```shell
docker start 容器id # 启动容器
docker restart 容器id # 重启容器
docker stop 容器id # 停止当前正在运行的容器
docker kill 容器id # 强制停止当前容器
```

## 其他命令

**后台启动容器**

```shell
# 命令 docker run -d 镜像名;
# 问题 docker ps 发现centos停止了
# 常见的坑 docker容器使用后台运行，就必须要有一个前台进程，若docker发现没有应用，就会自动停止
# 如 nginx 容器启动后，发现自己没有提供服务，就会立即停止
```

**查看日志**

```shell
docker logs -f -t --tail
```

查看容器中的进程信息

```shell
docker top 容器ID
```

**查看镜像的元数据**

```shell
docker inspect 容器id
```

**进入当前正在运行的容器**

```shell
# 容器通常都是后台方式运行的，需要进入容器，修改些配置
# 命令
# 方式1
docker exec -it 容器id bashShell
# 方式2
docker attach 容器id
```

> 两种方式区别

* `docker exec` 表示 进入容器后开启一个新的终端，可以在里面操作（常用） 
* `docker attach`  表示进入容器正在执行的终端，不会启动新的进程

**从容器内拷贝文件到主机上**

```shell
docker cp 容器id:容器内路径 目的主机路径
```

> 小结

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-12.png)

### Docker安装Nginx

```shell
# 1. 搜索镜像 search
# 2. 下载镜像 pull
# 3. 运行测试
docker images

# -d 后台运行
# --name 容器命名
# -p 宿主机端口:容器内部端口
docker run -d --name nginx01 -p 3344:80 nginx
# 测试本机
curl localhost:3344
```

> 端口暴露

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-13.png)

### Docker安装Tomcat

```shell
# 官方用法 （用完即删）
docker run -it --rm tomcat:9.0
```



```shell
# 下载镜像
docker pull tomcat:9.0
# 查看镜像
docker images
# 运行容器
docker run -d -p 3355:8080 --name tomcat01 tomcat
```



# Docker 镜像

## 镜像的概念

> 镜像概念

镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需的所有内容，包括代码、运行时，库、环境变量和配置文件。
所有的应用，直接打包docker撓像，就可以直接跑起来!

> 得到镜像

* 从远程仓库下载
* 拷贝
* 自制镜像DockerFile

## Docker 镜像加载原理

> UnionFS (联合文件系统)  

Union文件系统是一种**分层、轻量级并且高性能**的文件系统，它**支持对文件系统的修改作为一次提交来一层层的叠加**，同时可以将不同目录挂载到同一个虚拟文件系统下。 Union 文件系统是**Docker镜像的基础**。镜像可以通过分层来进行继承，**基于基础镜像(没有父镜像)，可以制作各种具体的应用镜像**。
特性：

一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系統，联合加载会把各层文件系统叠加起来，最终的文件系统会**包含所有底层的文件和目录**

> Docker镜像加载原理

**docker的镜像实际上由一层一层的文件系统组成**，bootfs(boot file system)主要包含bootloader和kernel, bootloader主要是引导加载kernel，Linux刚启动时会加载bootfs文件系统，**在Docker镜像的最底层是bootfs**。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸款bootfs。
rootfs (root file system) ， 在bootfs之上包含的就是典型Linux系統中的/dev，/proc，/bin， /etc等标准目录和文件。rootfs就是各种不同的操作系统发行版,比如Ubuntu , Centos等等。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-14.png)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-15.png)

对于一个精简的OS，rootfs可以很小，只需要包含最基本的命令，工具和程序，因为底层直接用主机的kernel，自己只需要提供rootfs，由此可见对于不同的Linux发行版，bootfs基本一致，rootfs会有差距，rootfs会有差别，因此不同的发行版可以共用bootfs

## 分层理解

> 分层的镜像

下载Redis镜像，观察日志输出，发现是分层下载

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-16.png)

> 镜像分层

所有的Docker镜像最开始起于基础镜像层，当进行修改或增加新的内容，便会在当前镜像层上，创建新的镜像层

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-17.png)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-18.png)

下图展示与系统显示相同的三层镜像，所有镜像层堆叠合并，对外提供统一的视图

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-19.png)

> 特点

Docker镜像都是只读的，当容器启动时，一个新的可写层加载到镜像的顶部，该层就是容器层，容器层下面称镜像层

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-20.png)

## Commit镜像

```shell
docker commit # 提交容器成为一个副本
# 命令和git原理类似
docker commit -m="提交的描述信息" -a="作者" 容器id 目标镜像名:[TAG]
```

# 容器数据卷

## 概念

容器间的数据共享，将Docker容器产生的数据，同步到本地。容器的持久化和同步操作，容器间可以数据共享

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-21.png)

## 使用数据卷

> 方式1 使用命令挂载 `-v`

```shell
docker run -it -v 主机目录:容器内目录

docker run -it -v /home/ceshi:/home centos /bin/bash
```



![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-22.png)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-23.png)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-24.png)

### Docker安装MySQL

```shell
# 获取mysql镜像
docker pull mysql:5.7
# 查看镜像
docker images
# 启动容器，进行挂载
# -d 后台运行
# -p 端口映射
# -v 卷挂载
# -e 环境配置
# --name 容器名字
docker run -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root --name mysql01 mysql:5.7
```

### 具名挂载和匿名挂载

> 匿名挂载

```shell
# 匿名挂载, -v只写容器内的路径，没有写容器外的路径
-v 容器内路径
docker run -d -P --name nginx01 -v /etc/nginx nginx

# 查看所有volume情况
docker volume
```

> 具名挂载

```shell
# 具名挂载, -v 卷名:容器内路径
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx nginx
```

所有docker容器内的卷，没有指定目录的情况下都是在`/var/lib/docker/volumes/xxx/_data`

> 拓展

通过 `-v 容器内路径` `ro` `rw` 改变读写权限

```shell
docker run -d -P --name nginx01 -v juming-nginx:/etc/nginx:ro nginx
docker run -d -P --name nginx01 -v juming-nginx:/etc/nginx:rw nginx
```

**注意** `ro`说明该路径只能通过宿主机操作，容器内部无法操作

### 数据卷容器

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-25.png)



容器之间的配置消息的传递，数据卷容器的生命周期一直持续到没有容器使用为止，但是一但持久化到本地，本地的数据是不会删除的



## Dockerfile

Dockerfile是用来构建docker镜像的文件，命令参数脚本

> 构建步骤

1. 编写一个dockerfile文件
2. docker build 构建成为一个镜像
3. docker run运行镜像
4. docker push 发布镜像（DockerHub、阿里云镜像仓库）

> 构建过程

**基础知识**

1. 每个保留关键字（指令）都是必须大写字母
2. 从上到下顺序执行
3. `#`表示注释
4. 每一个指令都会创建提交一个新的镜像层，并提交

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-26.png)

### Dockerfile指令

| 指令       | 说明                                                     |
| ---------- | -------------------------------------------------------- |
| FROM       | 指定基础镜像                                             |
| MAINTAINER | 指定维护者信息                                           |
| RUN        | 镜像构建时要运行的命令                                   |
| ADD        | 添加内容                                                 |
| WORKDIR    | 设置当前工作目录                                         |
| VOLUME     | 设置卷，挂载主机目录                                     |
| EXPOSE     | 指定对外的端口                                           |
| CMD        | 指定容器启动时运行的命令（只有最后一个会生效，可被替代） |
| ENTRYPOINT | 指定容器启动时运行的命令（可以追加命令）                 |
| ONBUILD    | 触发指令                                                 |
| COPY       | 文件拷贝到镜像                                           |
| ENV        | 构建时设置环境变量                                       |

### Dockerfile实战

#### 创建个性化的centos

**1.编写Dockerfile文件**

```shell
FROM centos
MAINTAINER Cyan-Chau

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yun -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "----end----"
CMD /bin/bash
```

**2.通过文件构建镜像**

```shell
# docker build -f dockerfile文件路径 -t 镜像名:[tag]
docker build -f myfile -t mycnetos:0.1
```

**补充命令**

```shell
# 镜像的历史变更记录
docker history 镜像ID
```

#### 创建个性化的Tomcat

1. 准备镜像文件tomcat压缩包，jdk的压缩包

   ![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-27.png)

2. 编写Dockerfile文件

   ```shell
   vim Dockerfile
   ```

   ```shell
   FROM centos
   MAINTAINER Cyan-Chau
   
   COPY readme.txt /usr/local/readme.txt
   
   ADD jdk-8u11-linux-x64.tar.gz /usr/local
   ADD apache-tomcat-9.0.22.tar.gz /usr/local
   
   RUN yum -y install vim
   
   ENV MYPATH /usr/local
   WORKDIR $MYPATH
   
   ENV JAVA_HOME /usr/local/jdk1.8.0_11
   ENV CLASSPATH $JAVA_HOME/lib/dt.jar;$JAVA_HOME/lib/tools.jar
   ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.22
   ENV CATALINA_BASE /usr/local/apache-tomcat-9.0.22
   ENV PATH $PATH:$JAVA_HOME/bin;$CATALINA_HOME/lib;CATALINA_HOME/bin
   
   EXPOSE 8080
   
   CMD /usr/local/apache-tomcat-9.0.22/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.22/bin/logs/catalina.out
   ```

   ```shell
   # 构建镜像
   docker build -t mytomcat .
   ```

### 发布镜像

> DockerHub

1. [注册账号](https://hub.docker.com)

2. 登录账号

   ```shell
   docker login --help
   
   docker login -u xxx
   ```

3. 提交镜像（带上版本号）

   ```shell
   docker push xxx:version
   ```



