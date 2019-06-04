### 简介
Ceph是一个高可靠、高可扩展和高性能的统一的分布式存储系统；
- 软件定义存储（Software Defined Storage, SDS）：几乎可以运行在所有主流Linux发行版；
- 分布式存储：去中心化，基于计算寻址使得Ceph客户端可以直接与服务端任意节点通信，从而避免热点访问而产出的性能瓶颈；
- 高可靠：支持多副本和纠删码的方式提供数据冗余，保证数据可靠性，并提供自动数据恢复、动态数据平衡；
- 统一存储：支持块存储、文件系统存储和对象存储多种存储方式；

### 核心组件
#### RADOS概述
传统的存储系统基于中心控制器或查表法dispatch客户端请求，因此存在性能瓶颈，也无法大规模扩展；而Ceph部署的存储集群容量可以达到PB甚至EB级别，当存储规模到达一定程度时，存储设备的故障就成为必然的了，如何保障设备故障后存储数据的可用性及数据恢复。另外，随着存储需求的不断增长，也会有新的设备被加入进来，同样地，如何保障集群新加入设备后数据存储的负载均衡。这样看来，规模庞大的存储系统其集群拓扑必然会一直处于变化中，集群中存储的数据必然也需要动态调整以实现各节点的负载均衡。

Ceph基于RADOS(reliable,autonomous,distributed object storage)在大规模异构存储集群中实现动态数据与负载均衡的。RADOS基于计算实时分布数据，允许自定义故障域，把任意类型的应用数据抽象成为对象这个抽象数据类型，从而提供可靠、高性能和全分布式的存储服务。

一个RADOS集群由大量**OSD**(Object Storage Device/Daemon)和少数几个**Monitor**组成。

#### OSD 
OSD是个抽象概念，一般对应于一块本地磁盘，在其工作周期内会占用一些CPU、内存、网络等物理资源，并依赖于某个具体的本地对象存储引擎（FileStore/BlueStore），来访问磁盘上的数据。主要功能是存储数据，处理数据的复制、恢复，检查其他OSD的心跳并向monitor报告OSD的状态。具体的数据存取则是由本地对象存储引擎实现。

#### Monitor
Monitor负责维护和分发集群的关键元数据，是客户端访问RADOS集群的桥梁。客户端通过访问monitor获取集群的Cluster map，从而直接与后端的OSD通信进行数据存取。

#### Manager
Manager：12.X版本后引入的新组件，提供了一些插件，如：
- Dashboard plugin
- RESTful plugin
- Zabbix plugin
- Prometheus plugin
- Iostat plugin
- Balancer plugin
- ...

负责收集集群的一些统计信息和当前集群的状态信息，如存储利用率、系统负载等监控信息。另外，还提供Dashboard和REST API暴露prometheus格式的监控数据。

#### MDS
MetaData Server：为Ceph文件系统存储元数据，使得POSIX文件系统用户可以在不对Ceph存储集群造成负担的前提下，执行诸如ls、find等基本命令。

### 架构
![image](http://s15.sinaimg.cn/mw690/004ex8oRzy73ZkzI1gq0e)

### 数据存储
Ceph OSD通过多副本和纠删码的方式，并将多个副本分布在不同的故障域，实现数据的安全、可靠。Ceph客户端和OSD使用**CRUSH**(Controlled Replication Under Scalable Hashing)算法计算数据位置。

Ceph提供了三种类型的客户端：Ceph Block Device, Ceph Filesystem, Ceph Object Storage，不论哪一种客户端最终都会将数据转换成对象存入Ceph存储集群。

![image](http://www.wendangwang.com/pic/2bcd8b0d0a13daedff03e6bc/3-357-png_6_0_0_0_0_0_0_892.979_1262.879-893-0-993-893.jpg)

1. Ceph client 向 Ceph monitor 请求 Cluster Map。获取Ceph 存储集群内所有信息（OSDs, pool, pg）；
2. Ceph client 将要读写的数据转化成object；
3. Ceph client 映射object到PG上；
4. Ceph client 根据CRUSH算法为每个PG挑选出指定副本数量的OSD;并将PG中的object写入primary osd;
5. Primary OSD 将object复制到其它OSD。

#### Object & PG & Pool
**Object**：Ceph内部存储数据的数据结构

![image](http://tianshan.github.io/images/2015-03-08-storage-2.png)

**PG**：RADOS并不是将任何应用程序数据一步到位地写入OSD的本地存储设备，而是引入了一个中间结构，PG(Placement Group)。PG是一些对象的集合。

一个普通规模的集群，可能存储数百万个对象，使得直接以对象为力度进行资源管理和任务管理代价过于昂贵。

> 一个100TB的集群，可以存储100TB/(3*4M) = 8738133个对象

**Pool:** 虚拟概念，表示一组约束条件，如针对存储池设计一组CRUSH规则，限制其只能使用某些规格相同的OSD，或者尽量将所有数据副本分布在物理上隔离的、不同的故障域；也可以针对不同用途的存储池设置不同的副本策略；甚至为每个存储池分别指定独立的Scrub、压缩、校验策略等。

#### 数据条带化
![image](http://tianshan.github.io/images/2015-03-08-striping-1.png)

#### obj --> PG
1. 客户端输入pool name和object ID（e.g., pool="liverpool" and object-id="john"）;
2. hash(oid);
3. hash(oid) 对PG（e.g., 64）数量取模;
4. 根据pool name获得pool ID(e.g., "liverpool" = 4);
5. 将pool ID附加在PG ID 之前(e.g., 4.17)；

#### PG --> OSD

### 核心应用
#### RBD
上层应用访问RBD块设备有两种途径：librbd和krbd。librbd是一个基于librados的用户态接口库，而krbd是集成在Linux内核中的一个内核模块，通过用户态rbd命令行工具，可以将RBD块设备映射成为本地的一个块设备文件。

![image](https://www.suse.com/documentation/suse-enterprise-storage-5/singlehtml/book_storage_admin/images/ceph_rbd_schema.png)
#### RGW

![image](http://docs.ceph.com/docs/master/_images/ditaa-50d12451eb76c5c72c4574b08f0320b39a42e5f1.png)

通常情况下，一个对象存储系统的基础数据实体包括用户、存储桶和对象，三种之间是一种层级关系。
- **用户**：对象存储应用的使用者。一个用户拥有一个或多个存储桶。
- **存储桶**：对象的容器，是为了方便管理和操作具有同一属性的一类对象引入的一层管理单元。
- **对象**：

#### CephFS
![image](https://tse3.mm.bing.net/th?id=OIP.6LrR_wuuKaPaln7iopo60gHaDo&pid=Api)

> http://docs.ceph.com/docs/master/
