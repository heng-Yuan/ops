## 什么是docker
Docker最初是dotCloud公司的一个内部项目，于2013年3月以`Apache 2.0`授权协议开源。Docker自开源后就受到广泛的关注和讨论，甚至由于Docker项目的火爆，dotCloud公司在2013年底更名为Docker。

Docker使用Go语言开发实现，基于Linux内核的`cgroup`，`namespace`，以及`AUFS`类的`Union FS`等技术，对进程进行封装隔离，属于操作系统级虚拟化技术。容器技术最初由LXC实现，隔离进程独立于宿主机，Docker在容器的基础上，进行了进一步的封装，从文件系统、网络互连到进程隔离。极大的简化了容器的创建和维护。

> Docker is an open platform for developing, shipping, and running applications. Docker enables you to separate your applications from your infrastructure so you can deliver software quickly. With Docker, you can manage your infrastructure in the same ways you manage your applications. By taking advantage of Docker’s methodologies for shipping, testing, and deploying code quickly, you can significantly reduce the delay between writing code and running it in production.

**Docker VS virtual machine**

<center>
<img src="https://www.docker.com/sites/default/files/Container%402x.png" width=480 height=500 />
<img src="https://www.docker.com/sites/default/files/VM%402x.png" width=480 height=500 />
</center>

### 基本概念
##### 镜像
Docker镜像是一个特殊的文件系统，它包含了应用程序运行时所需要的代码、运行时、库、环境变量和配置文件。镜像不包含任何动态数据，其内容在构建之后就不会再改变。镜像利用Union FS，分层存储，通常，一个镜像都是基于另一个镜像构建而成。

<center>
<img src="https://coolshell.cn/wp-content/uploads/2015/04/docker-filesystems-multilayer.png" width=680 height=480 />
</center>

##### 容器
镜像（image）和容器（container）的关系就像面向对象程序设计中`类`和`对象`的关系，镜像是静态的定义，容器是镜像运行时的实体。

容器的实质是进程，但它与直接运行在系统上的进程又有区别，每个容器进程运行在一个独立的命名空间（namespace）。因此，它有独立的文件系统，网络、进程空间、用户ID空间。
容器也是分层存储的，每一个容器运行时，以镜像为基础层，在其上创建一个当前容器的存储层，容器运行时的所有文件写入操作，都写在这个存储层中。

容器的存储层生命周期和容器一样，容器消亡时，容器存储层也就消亡了。因此，任何保存在容器存储层中的数据也将消失。
<center>
<img src="https://docs.docker.com/storage/storagedriver/images/sharing-layers.jpg" width=650 height=420 />
</center>

##### Registry
Docker Registry是一个集中存储、分发镜像的服务。

一个Docker Registry中可以包含多个仓库(Repository)；每个仓库可以包含多个标签（tag）;每个标签对应一个镜像。

### 底层原理
##### 基本架构
Docker是一个C/S架构的程序。提供了一个命令行工具`docker`以及一整套RESTful API。

<center>
<img src="https://docs.docker.com/engine/images/engine-components-flow.png" width=480 height=380 />
</center>

<center>
<img src="https://docs.docker.com/engine/images/architecture.svg" width=600 height=480 />
</center>

##### namespace
Docker使用`命名空间`技术来提供容器间的隔离。每当你运行一个容器，Docker就给这个容器创建一系列`namespace`。
- The pid namespace: Process isolation (PID: Process ID).
- The net namespace: Managing network interfaces (NET: Networking).
- The ipc namespace: Managing access to IPC resources (IPC: InterProcess Communication).
- The mnt namespace: Managing filesystem mount points (MNT: Mount).
- The uts namespace: Isolating kernel and version identifiers. (UTS: Unix Timesharing System).

#####  cgroup
cgroup的全称是`Linux control groups`，它提供了应用程序访问资源的限制。如：限制容器使用的`cpu`或`memory`。

##### UnionFS
> https://docs.docker.com/v17.03/engine/userguide/storagedriver/imagesandcontainers/

> Ideally, very little data is written to a container’s writable layer, and you use Docker volumes to write data. However, some workloads require you to be able to write to the container’s writable layer. This is where storage drivers come in.

> Docker uses storage drivers to manage the contents of the image layers and the writable container layer. Each storage driver handles the implementation differently, but all drivers use stackable image layers and the copy-on-write (CoW) strategy.

> Docker supports several different storage drivers, using a pluggable architecture. The storage driver controls how images and containers are stored and managed on your Docker host.

<center>
<img src="https://coolshell.cn/wp-content/uploads/2015/08/docker-filesystems-busyboxrw.png" width=680 height=480 />
</center>
AUFS

AUFS是一种Union File System，所谓UnionFS就是把不同物理位置的目录合并mount到同一个目录中。

> https://docs.docker.com/v17.03/engine/userguide/storagedriver/aufs-driver/

devicemapper
> https://docs.docker.com/v17.03/engine/userguide/storagedriver/device-mapper-driver/

overlayFS
> https://docs.docker.com/v17.03/engine/userguide/storagedriver/overlayfs-driver/

BTRFS
> https://docs.docker.com/v17.03/engine/userguide/storagedriver/btrfs-driver/

##### networking
- bridge
- host
- overlay
- macvlan
<br/>Macvlan networks allow you to assign a MAC address to a container, making it appear as a physical device on your network. The Docker daemon routes traffic to containers by their MAC addresses.</br>
- none

> https://docs.docker.com/network/

## 为什么要用docker
1. 更高效的利用系统资源
2. 更快速的启动时间
3. 一致的运行环境
4. 更轻松的迁移和扩展

## 怎么用docker
##### 安装
- Community Edition (CE)
- Enterprise Edition (EE)
```
$ yum install docker-ce-<VERSION STRING>
```
> https://docs.docker.com/install/linux/docker-ce/centos/

#####  使用镜像
```
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]

eg:
docker pull alpine
```
使用dockerfile制作镜像
```
FROM ubuntu:14.04
CMD ["/bin/echo" , "Hi Docker !"]

$ docker build -t helloword:v1 .
```

使用docker save 和docker load
```
$ docker save alpine | gzip > alpine-latest.tar.gz

$ docker load -i alpine-latest.tar.gz
```

##### 操作容器
启动:
```
$ docker run ubuntu:14.04 /bin/echo 'Hello world'

$ docker run -t -i ubuntu:14.04 /bin/bash
```
其中，` -t `选项让Docker分配一个伪终端（pseudo-tty） 并绑定到容器的标准输入上， `-i`则让容器的标准输入保持打开。

终止:
```
docker container stop CONTAINER
```
删除：
```
docker container rm CONTAINER
```

#####  数据管理
- volumes
- bind mounts
- tmpfs mounts

<center>
<img src="https://docs.docker.com/storage/images/types-of-mounts-bind.png" width=500 height=280 />
</center>

创建volume:
```
docker volume create my-vol
```
挂载：
```
$ docker run -d \
  --name devtest \
  --mount source=my-vol,target=/app \
  nginx:latest
```
删除：
```
$ docker container stop devtest
$ docker container rm devtest
$ docker volume rm my-vol
```
bind mounts:
```
$ docker run -d \
  -it \
  --name devtest \
  -p 80:80 \
  --mount type=bind,source=index.html,target=/usr/share/nginx/html/index.html \
  nginx:latest
```
tmpfs mounts
```
$ docker run -d \
  -it \
  --name tmptest \
  --mount type=tmpfs,destination=/app \
  nginx:latest
```
> https://docs.docker.com/storage/

## 容器编排
-  docker swarm
-  kubernetes

---
> **参考：**
<br/> 《Docker——从入门到实践》 </br>
《第一本Docker书》
<br/>《Kubernetes 指南》</br>
https://docs.docker.com 
<br/>https://github.com/feiskyer/kubernetes-handbook</br>
