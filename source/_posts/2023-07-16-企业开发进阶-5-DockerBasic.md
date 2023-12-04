---
title: 企业开发进阶-5-DockerBasic
date: 2023-07-16 18:07:25
tags: 
  - Docker
categories: 
  - Technology
---

# Docker 简介

[Docker 官网（官方文档）](http://www.docker.com)  
[Docker Hub 官网（镜像仓库）](https://hub.docker.com/)

## Docker概念

> **解决了运行环境和配置问题的软件容器，方便做持续集成并有助于整体发布的容器虚拟化技术。**

Docker 是基于 Go 语言实现的云开源项目。Docker 的主要目标是通过对应用组件的封装、分发、部署、运行等生命周期的管理，使用户的 APP（可以是一个WEB应用或数据库应用等等）及其运行环境能够做到“一次镜像，处处运行”。

Linux 容器技术的出现就解决了这样一个问题，而 Docker 就是在它的基础上发展过来的。将应用打成镜像，通过镜像成为运行在Docker 容器上面的实例，而 Docker 容器在任何操作系统上都是一致的，这就实现了跨平台、跨服务器。只需要一次配置好环境，换到别的机器上就可以一键部署好，大大简化了操作。

## 虚拟机和容器

> 传统虚拟机技术

虚拟机就是带环境安装的一种解决方案。可以在一种操作系统里面运行另一种操作系统。应用程序对此毫无感知，因为虚拟机看上去跟真实系统一模一样，而对于底层系统来说，虚拟机就是一个普通文件，不需要了就删掉，对其他部分毫无影响。这类虚拟机完美的运行了另一套系统，能够使应用程序，操作系统和硬件三者之间的逻辑不变。  

虚拟机的缺点：

* 资源占用多 
* 冗余步骤多
* 启动慢

> 容器虚拟化技术

Linux 容器是与系统其他部分隔离开的一系列进程，从另一个镜像运行，并由该镜像提供支持进程所需的全部文件。容器提供的镜像包含了应用的所有依赖项，因而在从开发到测试再到生产的整个过程中，它都具有可移植性和一致性。

Linux 容器不是模拟一个完整的操作系统而是 **对进程进行隔离**。有了容器，就可以将软件运行所需的所有资源打包到一个隔离的容器中。容器与虚拟机不同，不需要捆绑一整套操作系统，只需要软件工作所需的库资源和设置。系统因此而变得高效轻量并保证部署在任何环境中的软件都能始终如一地运行。

> 比较了 Docker 和传统虚拟化方式的不同之处：

* 传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；
* 容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。
* 每个容器之间互相隔离，每个容器有自己的文件系统 ，容器之间进程不会相互影响，能区分计算资源。

> Docker 为什么比虚拟机快

* **docker 有着比虚拟机更少的抽象层**
  由于 docker 不需要 Hypervisor (虚拟机)实现硬件资源虚拟化，运行在 docker 容器上的程序直接使用的都是实际物理机的硬件资源。因此在 CPU、内存利用率上 docker 将会在效率上有明显优势。
* **docker 利用的是宿主机的内核，而不需要加载操作系统OS内核**
  当新建一个容器时，docker 不需要和虚拟机一样重新加载一个操作系统内核。进而避免引寻、加载操作系统内核返回等比较费时费资源的过程，当新建一个虚拟机时，虚拟机软件需要加载 OS，返回新建过程是分钟级别的。而 docker 由于直接利用宿主机的操作系统，则省略了返回过程

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-06.png)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-07.png)

## Docker 能做什么

> 更快速的应用交付和部署

传统的应用开发完成后，需要提供一堆安装程序和配置说明文档，安装部署后需根据配置文档进行繁杂的配置才能正常运行。Docker 化之后只需要交付少量容器镜像文件，在正式生产环境加载镜像并运行即可，应用安装配置在镜像里已经内置好，大大节省部署配置和测试验证时间。

> 更便捷的升级和扩缩容

随着微服务架构和 Docker 的发展，大量的应用会通过微服务方式架构，应用的开发构建将变成搭乐高积木一样，每个 Docker 容器将变成一块“积木”，应用的升级将变得非常容易。当现有的容器不足以支撑业务处理时，可通过镜像运行新的容器进行快速扩容，使应用系统的扩容从原先的天级变成分钟级甚至秒级。

> 更简单的系统运维

应用容器化运行后，生产环境运行的应用可与开发、测试环境的应用高度一致，容器会将应用程序相关的环境和状态完全封装起来，不会因为底层基础架构和操作系统的不一致性给应用带来影响，产生新的BUG。当出现程序异常时，也可以通过测试环境的相同容器进行快速定位和修复。

> 更高效的计算资源利用

Docker 是内核级虚拟化，其不像传统的虚拟化技术一样需要额外的 Hypervisor 支持，所以在一台物理机上可以运行很多个容器实例，可大大提升物理服务器的CPU和内存的利用率。


## Docker 基本组成

> 镜像（Image）

Docker 镜像就是一个 **只读的模板**。一个镜像可以创建很多容器。它也相当于是一个 root 文件系统。
docker 镜像文件类似于 Java 的类模板，而 docker 容器实例类似于 java 中 new 出来的实例对象。

> 容器（Container）

* **从面向对象角度**
  Docker 利用容器（Container）独立运行的一个或一组应用，应用程序或服务运行在容器里面，容器就类似于一个虚拟化的运行环境，**容器是用镜像创建的运行实例**。就像是 Java 中的类和实例对象一样，**镜像是静态的定义，容器是镜像运行时的实体**。容器为镜像提供了一个标准的和隔离的运行环境，它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台

* **从镜像容器角度**
  可以把容器看做是一个简易版的 Linux 环境（包括 root 用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。

> 仓库（Repository）

仓库（Repository）是集中存放镜像文件的场所。Docker 公司提供的官方 registry 被称为 Docker Hub，存放各种镜像模板的地方。仓库分为公开仓库（Public）和私有仓库（Private）两种形式。最大的公开仓库是 Docker Hub(https://hub.docker.com/)，存放了数量庞大的镜像供用户下载。国内的公开仓库包括阿里云 、网易云等

> Docker平台

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-01.png)

> Docker工作原理

Docker 是一个 Client-Server 结构的系统，Docker 守护进程运行在主机上， 然后通过 Socket 连接从客户端访问，守护进程从客户端接受命令并管理运行在主机上的容器。 容器，是一个运行时环境，就是我们前面说到的集装箱

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-02.png)

> Docker整体架构

Docker 是一个 C/S 模式的架构，后端是一个松耦合架构，众多模块各司其职。

Docker 运行的基本流程：

1. 用户使用 Docker Client 与 Docker Daemon 建立通信，并发送请求给后者。
2. Docker Daemon 作为 Docker 架构中的主体部分，首先提供 Docker Server 的功能使其可以接受 Docker Client 的请求
3. Docker Engine 执行 Docker 内部的一系列工作，每一项工作都是以一个 Job 的形式的存在。
4. Job 的运行过程中，当需要容器镜像时，则从 Docker Registry 中下载镜像，并通过镜像管理驱动 Graph diver 将下载镜像以 Graph 的形式存储
5. 当需要为 Docker 创建网络环境时，通过网络管理驱动 Network driver 创建并配置 Docker 容器网络环境。
6. 当需要限制 Docker 容器运行资源或执行用户指令等操作时，则通过 Exec driver 来完成。
7. Libcontainer 是一项独立的容器管理包，Network driver 以及 Exec driver 都是通过 Libcontainer 来实现具体对容器进行的操作。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-03.png)



# Docker 常用命令

## 帮助启动类命令

| 命令                     | 说明                   |
| ------------------------ | ---------------------- |
| `systemctl start docker`   | 启动docker             |
| `systemctl stop docker`    | 停止docker             |
| `systemctl restart docker` | 重启docker             |
| `systemctl status docker`  | 查看docker状态         |
| `systemctl enable docker`  | 开机启动               |
| `docker info`              | 查看docker概要信息     |
| `docker --help`            | 查看docker总体帮助文档 |
| `docker 具体命令 --help`   | 查看docker命令帮助文档 |

## 镜像命令

| 命令                             | 选项                                                         | 说明                           |
| -------------------------------- | ------------------------------------------------------------ | ------------------------------ |
| `docker images`                    | `a` :列出本地所有的镜像（含历史映像层）；`-q` :只显示镜像ID      | 列出本地主机上的镜像；         |
| `docker search [OPTIONS] 镜像名字` | `--limit` : 只列出N个镜像，默认25个；docker search --limit 5 redis | 查找某镜像                     |
| `docker pull 镜像名字[:TAG]`       | 没有TAG就是最新版；相当于docker pull 镜像名字:latest         | 下载镜像                       |
| `docker system df`                 |                                                              | 查看镜像/容器/数据卷所占的空间 |
| `docker rmi 某个XXX镜像名字ID`     | docker rmi  -f 镜像ID（删除单个）；docker rmi -f 镜像名1:TAG 镜像名2:TAG（删除多个）；docker rmi -f $(docker images -qa)（删除全部） | 删除镜像                       |

> 谈谈 Docker 虚悬镜像

仓库名、标签都是 `<none>` 的镜像，俗称虚悬镜像 dangling image

## 容器命令

> 新建 + 启动容器

```bash
# 帮助命令
docker run --help

docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

 OPTIONS说明（常用）：

| 选项                  | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| `--name="容器新名字"` | 为容器指定一个名称                                           |
| `-d`                  | 后台运行容器并返回容器ID，也即启动守护式容器(后台运行)       |
| `-i`                  | 以交互模式运行容器，通常与 -t 同时使用；                     |
| `-t`                  | 为容器重新分配一个伪输入终端，通常与 -i 同时使用；<br/>也即启动交互式容器(前台有伪终端，等待交互) |
| `-P`                  | 随机端口映射，大写P                                          |
| `-p`                  | 指定端口映射，小写p                                          |

| 参数                          | 说明                               |
| ----------------------------- | ---------------------------------- |
| `-p hostPort:containerPort`     | 端口映射 -p 8080:80                |
| `-p ip:hostPort:containerPort`  | 配置监听地址 -p 10.0.0.100:8080:80 |
| `-p ip::containerPort`          | 随机分配端口 -p 10.0.0.100::80     |
| `-p hostPort:containerPort:udp` | 指定协议 -p 8080:80:tcp            |
| `-p 81:80 -p 443:443`           | 指定多个                           |



```bash
# 使用镜像centos:latest以交互模式启动一个容器,在容器内执行/bin/bash命令。
docker run -it centos /bin/bash 
```

参数说明：

* `-i`: 交互式操作
* `-t`: 终端
* `centos` : centos 镜像
* `/bin/bash`：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。要退出终端，直接输入 exit:

> 列出当前所有正在运行的容器

```bash
docker ps [OPTIONS]
```

OPTIONS说明（常用）：

| 选项 | 说明                                      |
| ---- | ----------------------------------------- |
| `-a`   | 列出当前所有正在运行的容器+历史上运行过的 |
| `-l`   | 显示最近创建的容器                        |
| `-n`   | 显示最近n个创建的容器。                   |
| `-q`   | 静默模式，只显示容器编号                  |

> 退出容器

| 命令     | 说明                                  |
| -------- | ------------------------------------- |
| `exit`     | run进去容器，exit退出，容器停止       |
| ctrl+p+q | run进去容器，ctrl+p+q退出，容器不停止 |

> 启动已停止运行的容器

| 命令                          | 说明                 |
| ----------------------------- | -------------------- |
| `docker start 容器ID或者容器名` | 启动已停止运行的容器 |

> 重启容器

| 命令                            | 说明     |
| ------------------------------- | -------- |
| `docker restart 容器ID或者容器名` | 重启容器 |

> 停止容器

| 命令                         | 说明         |
| ---------------------------- | ------------ |
| `docker stop 容器ID或者容器名` | 停止容器     |
| `docker kill 容器ID或容器名`   | 强制停止容器 |

> 删除已停止的容器

| 命令             | 说明             |
| ---------------- | ---------------- |
| `docker rm 容器ID` | 删除已停止的容器 |

```bash
# 一次性删除多个容器示例
docker rm -f $(docker ps -a -q)

docker ps -a -q | xargs docker rm
```

> 注意：有镜像才能创建容器（根本前提）

> 启动守护式容器（后台服务器）

* 在大部分的场景下，我们希望 docker 的服务是在后台运行的，我们可以过 `-d` 指定容器的后台运行模式
* `docker run -d 容器名`
* **Docker 容器后台运行，就必须有一个前台进程。容器运行的命令如果不是那些一直挂起的命令（比如运行 top，tail），就会自动退出的**，最佳的解决方案是将要运行的程序以前台进程的形式运行，常见就是命令行模式，表示还有交互操作，别中断

> 查看容器日志：

```bash
docker logs 容器ID
```

> 查看容器内运行的进程：

```bash
docker top 容器ID
```

>查看容器内部细节：

```bash
docker inspect 容器ID
```

> 进入正在运行的容器并以命令行交互

```bash
docker exec -it 容器ID bashShell
```

```bash
重新进入docker attach 容器ID
```

* attach 直接进入容器启动命令的终端，不会启动新的进程
  用 exit 退出，会导致容器的停止
* exec 是在容器中打开新的终端，并且可以启动新的进程
  **用 exit 退出，不会导致容器的停止**

```bash
# example
进入redis服务

docker exec -it 容器ID /bin/bash

docker exec -it 容器ID redis-cli

一般用-d后台启动的程序，再用exec进入对应容器实例
```

> 从容器内拷贝文件到主机上

```bash
docker cp  容器ID:容器内路径 目的主机路径
```

> 导入和导出容器

| 命令   | 说明                                                        |
| ------ | ----------------------------------------------------------- |
| `export` | 导出容器的内容留作为一个tar归档文件[对应import命令]         |
| `import` | 从tar包中的内容创建一个新的文件系统再导入为镜像[对应export] |

```bash
# example
docker export ContrainerId > 文件名.tar
cat 文件名.tar | docker import - 镜像用户/镜像名:镜像版本号
```

## 总结

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-12.png)

| 命令    | 英文解释                                                     | 中文解释                                                     |
| ------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `attach`  | Attach to a running container                                | 当前 shell 下 attach 连接指定运行镜像                        |
| `build`   | Build an image from a Dockerfile                             | 通过 Dockerfile 定制镜像                                     |
| `commit`  | Create a new image from a container changes                  | 提交当前容器为新的镜像                                       |
| `cp`      | Copy files/folders from the containers filesystem to the host path | 从容器中拷贝指定文件或者目录到宿主机中                       |
| `create`  | Create a new container                                       | 创建一个新的容器，同 run，但不启动容器                       |
| `diff`    | Inspect changes on a container's filesystem                  | 查看 docker 容器变化                                         |
| `events`  | Get real time events from the server                         | 从docker 服务获取容器实时事件                                |
| `exec`    | Run a command in an existing container                       | 在已存在的容器上运行命令                                     |
| `export`  | Stream the contents of a container as a tar archive          | 导出容器的内容流作为一个 tar 归档文件[对应 import ]          |
| `import`  | Create a new filesystem image from the contents of a tarball | 从tar包中的内容创建一个新的文件系统映像[对应export]          |
| `history` | Show the history of an image                                 | 展示一个镜像形成历史                                         |
| `images`  | List images                                                  | 列出系统当前镜像                                             |
| `info`    | Display system-wide information                              | 显示系统相关信息                                             |
| `inspect` | Return low-level information on a container                  | 查看容器详细信息                                             |
| `kill`    | Kill a running container                                     | kill 指定 docker 容器                                        |
| `load`    | Load an image from a tar archive                             | 从一个 tar 包中加载一个镜像[对应 save]                       |
| `login`   | Register or Login to the docker registry server              | 注册或者登陆一个 docker 源服务器                             |
| `logout`  | Log out from a Docker registry server                        | 从当前 Docker registry 退出                                  |
| `logs`    | Fetch the logs of a container                                | 输出当前容器日志信息                                         |
| `port`    | Lookup the public-facing port which is NAT-ed to PRIVATE_PORT | 查看映射端口对应的容器内部源端口                             |
| `pause`   | Pause all processes within a container                       | 暂停容器                                                     |
| `ps`      | List containers                                              | 列出容器列表                                                 |
| `pull`    | Pull an image or a repository from the docker registry server | 从docker镜像源服务器拉取指定镜像或者库镜像                   |
| `push`    | Push an image or a repository to the docker registry server  | 推送指定镜像或者库镜像至docker源服务器                       |
| `restart` | Restart a running container                                  | 重启运行的容器                                               |
| `rm`      | Remove one or more containers                                | 移除一个或者多个容器                                         |
| `rmi`     | Remove one or more images                                    | 移除一个或多个镜像[无容器使用该镜像才可删除，否则需删除相关容器才可继续或 -f 强制删除] |
| `run`     | Run a command in a new container                             | 创建一个新的容器并运行一个命令                               |
| `save`    | Save an image to a tar archive                               | 保存一个镜像为一个 tar 包[对应 load]                         |
| `search`  | Search for an image on the Docker Hub                        | 在 docker hub 中搜索镜像                                     |
| `start`   | Start a stopped containers                                   | 启动容器                                                     |
| `stop`    | Stop a running containers                                    | 停止容器                                                     |
| `tag`     | Tag an image into a repository                               | 给源中镜像打标签                                             |
| `top`     | Lookup the running processes of a container                  | 查看容器中运行的进程信息                                     |
| `unpause` | Unpause a paused container                                   | 取消暂停容器                                                 |
| `version` | Show the docker version information                          | 查看 docker 版本号                                           |
| `wait`    | Block until a container stops, then print its exit code      | 截取容器停止时的退出状态值                                   |

# Docker 镜像

## 镜像概念

镜像是一种轻量级、可执行的独立软件包，它包含运行某个软件所需的所有内容，我们把应用程序和配置依赖打包好形成一个可交付的运行环境(包括代码、运行时需要的库、环境变量和配置文件等)，这个打包好的运行环境就是 image 镜像文件。只有通过这个镜像文件才能生成 Docker 容器实例。

## UnionFS

> UnionFS（联合文件系统）：

Union文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下。Union 文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录

##  Docker 镜像加载原理

Docker 的镜像实际上由一层一层的文件系统组成，这种层级的文件系统 UnionFS。bootfs(boot file system) 主要包含bootloader和 kernel，bootloader 主要是引导加载 kernel，Linux 刚启动时会加载 bootfs 文件系统，在 Docker 镜像的最底层是引导文件系统 bootfs。这一层与我们典型的 Linux/Unix 系统是一样的，包含 boot 加载器和内核。当 boot 加载完成之后整个内核就都在内存中了，此时内存的使用权已由 bootfs 转交给内核，此时系统也会卸载 bootfs。

rootfs (root file system) ，在 bootfs 之上。包含的就是典型 Linux 系统中的 /dev, /proc, /bin, /etc 等标准目录和文件。rootfs 就是各种不同的操作系统发行版，比如 Ubuntu，Centos 等等。 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-28.png)

> 平时我们安装进虚拟机的 CentOS 都是好几个G，为什么docker这里才200M？

对于一个精简的 OS，rootfs 可以很小，只需要包括最基本的命令、工具和程序库就可以了，因为底层直接用 Host 的 kernel，自己只需要提供 rootfs 就行了。由此可见对于不同的 linux 发行版, bootfs 基本是一致的, rootfs 会有差别, 因此不同的发行版可以公用 bootfs。

> 为什么 Docker 镜像采用分层结构

镜像分层最大的一个好处就是共享资源，方便复制迁移，就是为了复用。

比如说有多个镜像都从相同的 base 镜像构建而来，那么 Docker Host 只需在磁盘上保存一份 base 镜像；
同时内存中也只需加载一份 base 镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-29.png)

Docker 镜像层都是只读的，容器层是可写的。当容器启动时，一个新的可写层被加载到镜像的顶部。
这一层通常被称作“容器层”，“容器层”之下的都叫“镜像层”。

## commit

| 命令                                                         | 说明                                          |
| ------------------------------------------------------------ | --------------------------------------------- |
| `docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[标签名]` | docker commit提交容器副本使之成为一个新的镜像 |



1. 从 Hub 上下载 ubuntu 镜像到本地并成功运行

2. 原始的默认 Ubuntu 镜像是不带着 vim 命令的

3. 外网连通的情况下，安装 vim

   ```bash
   # docker容器内执行上述两条命令：
   apt-get update
   apt-get -y install vim
   ```

4. 安装完成后，commit 我们自己的新镜像

5. 启动我们的新镜像并和原来的对比

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-30.png)

## 镜像发布

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-31.png)

> 发布到阿里云

1. 创建镜像仓库
2. 将镜像推送到阿里云 registry
3. 将阿里云镜像下载到本地

> 发布到 Docker Hub（私服）

# Docker 容器数据卷

> Docker 挂载主机目录访问如果出现 cannot open directory .: Permission denied

解决办法：在挂载目录后多加一个 `--privileged=true` 参数即可

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-32.png)

卷就是目录或文件，存在于一个或多个容器中，由 docker 挂载到容器，但不属于联合文件系统，因此能够绕过 Union File System 提供一些用于持续存储或共享数据的特性。卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷。一句话，将docker容器内的数据保存进宿主机的磁盘中

> 运行一个带有容器卷存储功能的容器实例

```bash
docker run -it --privileged=true -v /宿主机绝对路径目录:/容器内目录      镜像名
```

> 容器数据卷作用

将运用与运行的环境打包镜像，run 后形成容器实例运行 ，但是我们对数据的要求希望是持久化的。Docker 容器产生的数据，如果不备份，那么当容器实例删除后，容器内的数据自然也就没有了。
为了能保存数据在 docker 中我们使用卷。

* 数据卷可在容器之间共享或重用数据
* 卷中的更改可以直接实时生效 
* 数据卷中的更改不会包含在镜像的更新中
* 数据卷的生命周期一直持续到没有容器使用它为止

## 容器卷和主机互通互联

```bash
docker run -it --privileged=true -v /宿主机绝对路径目录:/容器内目录 镜像名

# 查看数据卷是否挂载成功
docker inspect 容器ID
```

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-33.png)

```bash
docker run -it --name myu3 --privileged=true -v /tmp/myHostData:/tmp/myDockerData ubuntu /bin/bash
```

> 容器和宿主机之间数据共享

```bash
1  docker修改，主机同步获得 
2 主机修改，docker同步获得
3 docker容器stop，主机修改，docker容器重启看数据是否同步。
```

## 读写规则映射添加说明

> 读写(默认)

```bash
docker run -it --privileged=true -v /宿主机绝对路径目录:/容器内目录:rw 镜像名
```

> 只读
>
> 容器实例内部被限制，只能读取不能写，此时如果宿主机写入内容，可以同步给容器内，容器可以读取到。

```bash
 docker run -it --privileged=true -v /宿主机绝对路径目录:/容器内目录:ro 镜像名
```

## 卷的继承和共享

> 容器 1 完成和宿主机的映射

```bash
 docker run -it  --privileged=true -v /mydocker/u:/tmp --name u1 ubuntu
```

> 容器 2 继承容器 1 的卷规则

```bash
docker run -it  --privileged=true --volumes-from 父类  --name u2 ubuntu
```

