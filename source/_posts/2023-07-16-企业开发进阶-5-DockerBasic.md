---
title: 企业开发进阶-5-DockerBasic
date: 2023-07-16 18:07:25
tags: 
  - Docker
categories: 
  - Technology
swiper_index: 
---

# Docker 简介

[docker官网（官方文档）](http://www.docker.com)

[Docker Hub官网（镜像仓库）](https://hub.docker.com/)

## Docker概念

> **解决了运行环境和配置问题的软件容器，方便做持续集成并有助于整体发布的容器虚拟化技术。**

Docker是基于Go语言实现的云开源项目。Docker的主要目标是通过对应用组件的封装、分发、部署、运行等生命周期的管理，使用户的APP（可以是一个WEB应用或数据库应用等等）及其运行环境能够做到“一次镜像，处处运行”。

Linux容器技术的出现就解决了这样一个问题，而 Docker 就是在它的基础上发展过来的。将应用打成镜像，通过镜像成为运行在Docker容器上面的实例，而 Docker容器在任何操作系统上都是一致的，这就实现了跨平台、跨服务器。只需要一次配置好环境，换到别的机器上就可以一键部署好，大大简化了操作。

## 虚拟机和容器

> 传统虚拟机技术

虚拟机就是带环境安装的一种解决方案。可以在一种操作系统里面运行另一种操作系统。应用程序对此毫无感知，因为虚拟机看上去跟真实系统一模一样，而对于底层系统来说，虚拟机就是一个普通文件，不需要了就删掉，对其他部分毫无影响。这类虚拟机完美的运行了另一套系统，能够使应用程序，操作系统和硬件三者之间的逻辑不变。  

虚拟机的缺点：

* 资源占用多 
* 冗余步骤多
* 启动慢

> 容器虚拟化技术

Linux容器是与系统其他部分隔离开的一系列进程，从另一个镜像运行，并由该镜像提供支持进程所需的全部文件。容器提供的镜像包含了应用的所有依赖项，因而在从开发到测试再到生产的整个过程中，它都具有可移植性和一致性。

Linux 容器不是模拟一个完整的操作系统而是**对进程进行隔离**。有了容器，就可以将软件运行所需的所有资源打包到一个隔离的容器中。容器与虚拟机不同，不需要捆绑一整套操作系统，只需要软件工作所需的库资源和设置。系统因此而变得高效轻量并保证部署在任何环境中的软件都能始终如一地运行。

> 比较了 Docker 和传统虚拟化方式的不同之处：

* 传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；
* 容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。
* 每个容器之间互相隔离，每个容器有自己的文件系统 ，容器之间进程不会相互影响，能区分计算资源。

> Docker为什么比虚拟机快

* **docker有着比虚拟机更少的抽象层**
  由于docker不需要Hypervisor(虚拟机)实现硬件资源虚拟化，运行在docker容器上的程序直接使用的都是实际物理机的硬件资源。因此在CPU、内存利用率上docker将会在效率上有明显优势。
* **docker利用的是宿主机的内核，而不需要加载操作系统OS内核**
  当新建一个容器时，docker不需要和虚拟机一样重新加载一个操作系统内核。进而避免引寻、加载操作系统内核返回等比较费时费资源的过程，当新建一个虚拟机时,虚拟机软件需要加载OS，返回新建过程是分钟级别的。而docker由于直接利用宿主机的操作系统，则省略了返回过程

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-06.png)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-07.png)

## Docker 能做什么

> 更快速的应用交付和部署

传统的应用开发完成后，需要提供一堆安装程序和配置说明文档，安装部署后需根据配置文档进行繁杂的配置才能正常运行。Docker化之后只需要交付少量容器镜像文件，在正式生产环境加载镜像并运行即可，应用安装配置在镜像里已经内置好，大大节省部署配置和测试验证时间。

> 更便捷的升级和扩缩容

随着微服务架构和Docker的发展，大量的应用会通过微服务方式架构，应用的开发构建将变成搭乐高积木一样，每个Docker容器将变成一块“积木”，应用的升级将变得非常容易。当现有的容器不足以支撑业务处理时，可通过镜像运行新的容器进行快速扩容，使应用系统的扩容从原先的天级变成分钟级甚至秒级。

> 更简单的系统运维

应用容器化运行后，生产环境运行的应用可与开发、测试环境的应用高度一致，容器会将应用程序相关的环境和状态完全封装起来，不会因为底层基础架构和操作系统的不一致性给应用带来影响，产生新的BUG。当出现程序异常时，也可以通过测试环境的相同容器进行快速定位和修复。

> 更高效的计算资源利用

Docker是内核级虚拟化，其不像传统的虚拟化技术一样需要额外的Hypervisor支持，所以在一台物理机上可以运行很多个容器实例，可大大提升物理服务器的CPU和内存的利用率。

# Docker 安装

> 前提条件

目前，CentOS 仅发行版本中的内核支持 Docker。Docker 运行在CentOS 7 (64-bit)上

```shell
cat /etc/redhat-release
```

> 查看自己的内核

uname命令用于打印当前系统相关信息（内核版本号、硬件架构、主机名称和操作系统类型等）。

```shell
uname -r
```

## Docker 基本组成

> 镜像（Image）

Docker 镜像就是一个**只读的模板**。一个镜像可以创建很多容器。它也相当于是一个root文件系统。
docker镜像文件类似于Java的类模板，而docker容器实例类似于java中new出来的实例对象。

> 容器（Container）

* **从面向对象角度**
  Docker 利用容器（Container）独立运行的一个或一组应用，应用程序或服务运行在容器里面，容器就类似于一个虚拟化的运行环境，**容器是用镜像创建的运行实例**。就像是Java中的类和实例对象一样，**镜像是静态的定义，容器是镜像运行时的实体**。容器为镜像提供了一个标准的和隔离的运行环境，它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台

* **从镜像容器角度**
  可以把容器看做是一个简易版的 Linux 环境（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。

> 仓库（Repository）

仓库（Repository）是集中存放镜像文件的场所。Docker公司提供的官方registry被称为Docker Hub，存放各种镜像模板的地方。仓库分为公开仓库（Public）和私有仓库（Private）两种形式。最大的公开仓库是 Docker Hub(https://hub.docker.com/)，存放了数量庞大的镜像供用户下载。国内的公开仓库包括阿里云 、网易云等

> Docker平台

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-01.png)

> Docker工作原理

Docker是一个Client-Server结构的系统，Docker守护进程运行在主机上， 然后通过Socket连接从客户端访问，守护进程从客户端接受命令并管理运行在主机上的容器。 容器，是一个运行时环境，就是我们前面说到的集装箱

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-02.png)

> Docker整体架构

Docker是一个C/S模式的架构，后端是一个松耦合架构，众多模块各司其职。

Docker运行的基本流程：

1. 用户使用Docker Client 与 Docker Daemon建立通信，并发送请求给后者。
2. Docker Daemon作为Docker架构中的主体部分，首先提供Docker Server的功能使其可以接受Docker Client 的请求
3. Docker Engine执行Docker内部的一系列工作，每一项工作都是以一个Job的形式的存在。
4. Job的运行过程中，当需要容器镜像时，则从Docker Registry 中下载镜像，并通过镜像管理驱动Graph diver将下载镜像以Graph的形式存储
5. 当需要为Docker创建网络环境时，通过网络管理驱动Network driver创建并配置Docker容器网络环境。
6. 当需要限制Docker容器运行资源或执行用户指令等操作时，则通过 Exec driver来完成。
7. Libcontainer是一项独立的容器管理包，Network driver以及Exec driver都是通过Libcontainer来实现具体对容器进行的操作。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-03.png)

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

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-docker-20230715-05.png)

# Docker 常用命令

## 帮助启动类命令

| 命令                     | 说明                   |
| ------------------------ | ---------------------- |
| systemctl start docker   | 启动docker             |
| systemctl stop docker    | 停止docker             |
| systemctl restart docker | 重启docker             |
| systemctl status docker  | 查看docker状态         |
| systemctl enable docker  | 开机启动               |
| docker info              | 查看docker概要信息     |
| docker --help            | 查看docker总体帮助文档 |
| docker 具体命令 --help   | 查看docker命令帮助文档 |

## 镜像命令

| 命令                             | 选项                                                         | 说明                           |
| -------------------------------- | ------------------------------------------------------------ | ------------------------------ |
| docker images                    | a :列出本地所有的镜像（含历史映像层）；-q :只显示镜像ID      | 列出本地主机上的镜像；         |
| docker search [OPTIONS] 镜像名字 | --limit : 只列出N个镜像，默认25个；docker search --limit 5 redis | 查找某镜像                     |
| docker pull 镜像名字[:TAG]       | 没有TAG就是最新版；相当于docker pull 镜像名字:latest         | 下载镜像                       |
| docker system df                 |                                                              | 查看镜像/容器/数据卷所占的空间 |
| docker rmi 某个XXX镜像名字ID     | docker rmi  -f 镜像ID（删除单个）；docker rmi -f 镜像名1:TAG 镜像名2:TAG（删除多个）；docker rmi -f $(docker images -qa)（删除全部） | 删除镜像                       |

> 谈谈Docker虚悬镜像

仓库名、标签都是<none>的镜像，俗称虚悬镜像dangling image

## 容器命令