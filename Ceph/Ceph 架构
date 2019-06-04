# CEPH ARCHITECTURE
[译自官方文档](http://docs.ceph.com/docs/master/architecture/)  

![image](http://docs.ceph.com/docs/master/_images/stack.png)

## THE CEPH STORAGE CLUSTER
ceph 通过 ***RADOS*** 提供了一个可以无限扩展的集群存储系统，可以通过  [RADOS - A Scalable, Reliable Storage Service for Petabyte-scale Storage Clusters](https://ceph.com/wp-content/uploads/2016/08/weil-rados-pdsw07.pdf) 这篇文章了解更多细节。

ceph storage cluster 主要由两种daemon 组成：

- ***Ceph Monitor***
- ***Ceph OSD Daemon***

```
graph TB
    A(OSDs)   
    B(Monitors)
```

Ceph Monitor 保存了 `cluster map` 的原版拷贝. Ceph monitor cluster 为 monitor damon 提供高可用，避免单点故障. 集群存储客户端（OSD）可通过 Ceph Monitor 来恢复 cluster map 拷贝。

Ceph OSD daemon 会检测自己和其他节点的状态（通过端口间的心跳消息）并上报 monitor

### STORAING DATA
---

ceph 集群存储从客户端接受数据( 可能通过 Ceph Black Device, Ceph Object Storage 或者 Ceph Filesystem 以及通过 `librados` 工具库写入)。数据被作为对象存储到ceph，每个对象都会对应文件系统中的某个文件(存储在OSD中). ceph OSD Daemon 会负责磁盘的读写操作.

```
graph LR
    A(Object)-->B[File]
    B --> C((Disk))
```

Ceph OSD daemon 将所有的数据存在同一个目录
> Ceph OSD Daemons store all data as objects in a flat namespace (e.g., no hierarchy of directories).

一个 Object 包含一个id, 二进制数据和元数据组成的key/value对.
![image](http://docs.ceph.com/docs/master/_images/ditaa-ae8b394e1d31afd181408bab946ca4a216ca44b7.png)

> **Note :** An object ID is unique across the entire cluster, not just the local filesystem.


### SCALABILITY AND HIGH AVAILABILITY
---
在传统架构中，一个复杂的系统往往会有一个集中式的访问入口(e.g: gateway, broker, facade, etc.) 这种模型在某种程度上限制了系统的性能和可扩展性,并且存在单点故障的隐患(e.g: 当中心组件挂掉,会导致整个系统不可用)

Ceph 实现了去中心化,客户端直接与 Ceph OSD Daemon 交互. Ceph OSD Daemons 会从其他节点复制数据拷贝来保障数据安全和高可用.Ceph 同时使用 Ceph Monitor cluster 来保障高可用. 为了实现去中心化, Ceph 使用了 ***CRUSH*** 算法.


### CRUSH INTRODUCTION
---
Ceph client 和 Ceph OSD Daemons 都通过 ***CRUSH*** 算法来定位 Object 存储位置, 而不是依赖于某个表来查询. 相比传统方案, ***CRUSH*** 提供了一个更好的数据管理机制, ***CRUSH*** 使用智能数据复制来确保弹性(可以定义Object的副本数量),更适用于超大规模的存储集群. 关于 ***CRUSH*** 的详述,参考:  
[ CRUSH - Controlled, Scalable, Decentralized Placement of Replicated Data](https://ceph.com/wp-content/uploads/2016/08/weil-crush-sc06.pdf)


### CLUSTER MAP
---
>Ceph depends upon Ceph Clients and Ceph OSD Daemons having knowledge of the cluster topology(拓扑学), which is inclusive of 5 maps collectively referred to as the “Cluster Map”:
>
> 1. ***The Monitor Map:*** Contains the cluster `fsid`, the position, name address and port of each monitor. It also indicates the current epoch, when the map was created, and the last time it changed. To view a monitor map, execute ceph `mon dump`.  
> 2. ***The OSD Map:*** Contains the cluster `fsid`, when the map was created and last modified, a list of pools, replica sizes, PG numbers, a list of OSDs and their status (e.g., `up`, `in`). To view an OSD map, execute `ceph osd dump`.
> 3. ***The PG Map:*** Contains the PG version, its time stamp, the last OSD map epoch, the full ratios, and details on each placement group such as the PG ID, the Up Set, the Acting Set, the state of the PG (e.g., `active + clean`), and data usage statistics for each pool.
> 4. ***The CRUSH Map***: Contains a list of storage devices, the failure domain hierarchy (e.g., device, host, rack, row, room, etc.), and rules for traversing the hierarchy when storing data. To view a CRUSH map, execute `ceph osd getcrushmap -o {filename}`; then, decompile it by executing `crushtool -d {comp-crushmap-filename} -o {decomp-crushmap-filename}`. You can view the decompiled map in a text editor or with `cat`.
> 5. ***The MDS Map:*** Contains the current MDS map epoch, when the map was created, and the last time it changed. It also contains the pool for storing metadata, a list of metadata servers, and which metadata servers are `up` and `in`. To view an MDS map, execute `ceph fs dump`.
>
>Each map maintains an iterative(迭代的) history of its operating state changes. Ceph Monitors maintain a master copy of the cluster map including the cluster members, state, changes, and the overall health of the Ceph Storage Cluster.

简言之: `cluster map` 描述集群中所有 OSD, Monitor 信息,状态及其逻辑关系.


### HIGH AVAILABILITY MONITORS
---
Ceph Clients 需要先从 Ceph Monitor 处获取最新的 `cluster map`, 然后才能进行读写.为了避免 Ceph Monitor 的单点故障,需要为Ceph Monitor 添加集群.

在 Ceph Monitor 集群中, 潜在的bug或者其他原因(如服务器故障,网络不通等)导致部分Monitor 在当前集群状态中失效. 由于这个原因, Ceph Monitor cluster 中必须有超过半数的节点可用才能决定 cluster 是否工作.(e.g: 1, 2:3, 3:5, 4:6, etc.) 各个Monitor间通过 Paxos 算法来达成共识.


### HIGH AVAILABILITY AUTHENTICATION
---
为了预防 `man-in-the-middle attacks` (中间人攻击,ARP欺骗), Ceph 提供了 `cephx` 来鉴定 user 和 daemon.
> ***NOTE :*** The cephx protocol does not address data encryption in transport (e.g., SSL/TLS) or encryption at rest.

`cephx` 使用对称密钥来认证,这就意味着客户端和Monitor都需要一份密钥拷贝.在部署集群的时候,要确保 `client` 和 `Monitor` 都拥有相同的key文件.

由于 ceph 的去中心化设计, 这就意味着客户端需要直接和OSDs产生交互.为了保护数据, Ceph提供其cephx认证系统，用于认证用户操作Ceph客户端。

首先 Ceph 客户端需要和 monitor 通信(默认情况下会指定一个名为admin的用户来进行认证). Monitor 会返回一个认证的数据结构,里面包含了用于获取Ceph服务的会话密钥. 会话密钥本身由用户密钥加密,因此只有该用户能够从Ceph monitor 请求服务. 客户端使用会话密钥想monitor请求服务,随后monitor提供一个用于和OSDs直接交互的 ticket. Ceph Monitor 和 OSDs 使用同一个密码,所以客户端能于 Ceph Monitor 管辖下的每一个 OSD 交互. 想大多数认证方法, ticket 存在过期时间, 攻击者无法使用一个过期的ticket来请求.因此只要密钥未在过期前被泄露, 中间的攻击者就无计可施.

为了使用 `cephx`, 需要先设置系统管理账号. 一般通过 `ceph auth get-or-create-key` 命令来生成 `client.admin` 用户的用户名和密钥. 另外需要将生成的密钥copy 一份至 Ceph 客户端上去. 如下图所示:
> ***Note :*** The `client.admin` user must provide the user ID and secret key to the user in a secure manner
 
 ![img](http://docs.ceph.com/docs/master/_images/ditaa-6b1dafb6d8f177ab2beb3325857f1e98e4593ec6.png)

为了达成认证, 分如下步骤:

1. 客户端需要向 monitor 传递用户名,如admin  
2. 随后monitor生成一个会话密钥,并用与该用户名相关联的秘密密钥进行加密。  
3. 将加密的会话密钥传送至客户端之后,客户端使用共享密钥解密,得到解密后会话密钥.(会话密钥定义了当前会话的用户)  
4. 在获得会话密钥之后,客户端代表会话密钥签名的用户(即admin)请求ticket.  
5. Monitor 生成ticket, 使用admin对应的密钥加密后发送给客户端.
6. 客户端解密,并利用该ticket向集群中的OSDs或者MDS发送请求.

![image](http://docs.ceph.com/docs/master/_images/ditaa-56e3a72e085f9070289331d64453b84ab1e9510b.png)

总体流程如下所示:

![image](http://docs.ceph.com/docs/master/_images/ditaa-f97566f2e17ba6de07951872d259d25ae061027f.png)

疑问: 若存在多个OSD, ceph Client 与哪个通信? 

### SMART DAEMONS ENABLE HYPERSCALE
---

在许多集群架构中, 一个最基本作用的就是核心组件要知道哪些节点是能够访问的.然后集群的入口(如apigw等的组件)向客户端提供服务--这样的做法在面对PB级别数据流量的时候会成为瓶颈.

为了消除瓶颈,ceph 的 OSD Daemon 和 Ceph Client 之间是相互感知的.像Ceph客户端，每个Ceph OSD守护进程都知道集群中的其他Ceph OSD守护进程。 这使得Ceph OSD守护进程能够与其他Ceph OSD守护进程和Ceph Monitor直接交互。 此外，它使Ceph客户端可以直接与Ceph OSD守护进程进行交互。

Ceph客户端，Ceph监视器和Ceph OSD守护进程互相交流的能力意味着Ceph OSD守护进程可以利用Ceph节点的CPU和RAM轻松执行会使中央服务器恶化的任务。 利用这种计算能力的能力有以下几个主要优点：
1. OSDs 直接向客户端提供服务:由于任何网络设备支持的并发连接数有限制,因此集中式的访问在高并发下会受物理条件限制. ceph 客户端直接访问OSDs, 在提高性能和系统容量的同时,还能消除单点故障.
2. OSD 成员关系和状态: OSD 进程加入集群后会汇报其状态.最基础的,OSD 进程的 `up` 和 `down` 反映了OSD 是否在运行以及能否为客户端提供服务. `down` &`in` 意味着OSD守护进程失败. 如果OSD 守护进程未运行(e.g., 服务器故障), OSD 无法通知Monitor自己的处于 `down` 状态. OSD会间歇性的像Monitor 发送心跳消息. 如果Monitor 在特定时间间隔内未收到消息,就会标记该OSD 为 `down` 状态. 这是一个故障安全的机制. 然而,一般来说: 当一个OSD 失败,邻近的OSD 守护进程会发现并上报至Monitor. 这就减轻了 Ceph Monitor 的复杂度.
3. 数据清洗:  作为数据一致性和数据清洁的一部分, OSD 能够通过 `PG` 来清洗 `object` .具体来说就是,Ceph OSD守护进程可以将一个`PG`中的对象元数据与其他OSD中存储的`PG`中的副本进行比较, 以此捕获bug和文件系统错误(通常每天执行一次). OSD也执行通过对object数据逐个字节的对比来实现更深层次的清洗.深度清洗可以发现轻度清洗中捕获不到的磁盘坏道(通常每周执行一次),不过期间会影响性能.
4. 复制: 像Ceph客户端一样，Ceph OSD守护进程使用CRUSH算法，但是Ceph OSD Daemon使用它来计算应该存储对象的副本（并重新平衡）,在典型的写操作场景下,客户端使用CRUSH 算法来计算Object存储的位置,并将对象映射到 `pool` 和 `PG`, 然后查看CRUSH映射以标识 `PG` 的 `Primary OSD`.


客户端将object 写入到 `PG` 对应的 `Primary OSD`, 随后 `Primary OSD` 会将object 复制到第二个和第三个 OSD(默认Object的拷贝数为3), 当所有副本存储成功后,会向客户端返回响应.过程如下图所示:

![image](http://docs.ceph.com/docs/master/_images/ditaa-54719cc959473e68a317f6578f9a2f0f3a8345ee.png)

凭借执行数据复制的能力，Ceph OSD守护进程可以减轻Ceph客户的负担，同时确保高数据可用性和数据安全性。


### DYNAMIC CLUSTER MANAGEMENT
---
Ceph设计的关键是自主，自我修复和智能Ceph OSD守护进程。 我们来深入了解CRUSH如何使现代云存储基础架构能够放置数据，重新平衡群集并动态地从故障中恢复

#### ABOUT POOLS
Ceph 存储系统支持"pool"的概念,作为存储对象的逻辑分区.
Ceph客户端从Ceph Monitor检索 `cluster map`，并将对象写入`pool`。 `pool` 的大小或副本数，CRUSH规则集和 `PG` 数决定了Ceph如何放置数据。
如下图所示:  
![image](http://docs.ceph.com/docs/jewel/_images/ditaa-65961c2ab9771b66c8c73e6d5fd648b0ea83c2da.png)

`pool` 至少需要如下配置:  
-   object 的 owner和访问权限
-   `PG` 个数
-   使用的 CRUSH 规则集 

    
#### MAPPING PGs TO OSDs

每个 `pool` 包含若干 `PG`. CRUSH 将 OSD 动态的关联到 `PG`. 当客户端要存储一个object, CRUSH 会将object映射到某个`PG` 上.

`PG` 想当与 OSD Daemon 和 Ceph client 之间的一个逻辑层.Ceph存储集群必须能够在动态存储对象的情况下增长（或缩小）并重新平衡（再平衡）. 如果Ceph client 知道对象和object的对应关系,这将在Ceph客户端和Ceph OSD守护进程之间建立紧密的耦合.相反，CRUSH算法将每个对象映射到一个 `PG`，然后将每个 `PG` 映射到一个或多个Ceph OSD守护进程。 这种逻辑层允许Ceph在新的Ceph OSD守护进程和底层OSD设备上线时动态重新平衡。 下图描绘CRUSH如何将对象映射到 `PG`，将 `PG` 映射到OSD。(概念有些类似lvm)

![image](http://docs.ceph.com/docs/master/_images/ditaa-c7fd5a4042a21364a7bef1c09e6b019deb4e4feb.png)

使用 `cluster map` 的副本和CRUSH算法，客户端可以准确计算在读取或写入特定对象时要使用的OSD。

**疑问：** 一个PG能关联多少个OSD？


#### CALCULATING PG IDS

当客户端连接到了Ceph Monitor， 就会获得一份最新的 cluster map 拷贝。通过 cluster map 客户端可以知道集群中所有组件的信息。然而，却无法得知object的存储位置。

>Object locations get computed

客户端只需要输入 object ID和pool信息。通过object id, a hash code, pool的PG数量和pool name就能计算出object位于哪个PG中。Ceph 客户端计算PG IDs 的步骤如下：
>  1. The client inputs the pool ID and the object ID. (e.g., pool = “liverpool” and object-id = “john”)  
> 2. Ceph takes the object ID and hashes it.  
> 3. Ceph calculates the hash modulo(以…为模) the number of PGs. (e.g., 58) to get a PG ID.
> 4. Ceph gets the pool ID given the pool name (e.g., “liverpool” = 4)
> 5. Ceph prepends(预先考虑) the pool ID to the PG ID (e.g., 4.58).

在本地计算object位置要比通过session查询快得多。CRUSH 算法允许客户端计算出object的存储位置，并且可以让客户端直接向 primary OSD 中存储/读取数据。


#### PEERING AND SETS

Ceph OSD Daemons 除了检测彼此间心跳之外，还会将所有OSD中PG存储的objects(以及object metadata)状态相一致。这个过程称之为 “peering”(OSD 还负责数据一致性检查).

>Another thing Ceph OSD daemons do is called ‘peering’, which is the process of bringing all of the OSDs that store a Placement Group (PG) into agreement about the state of all of the objects (and their metadata) in that PG

> ***Note :*** Agreeing on the state does not mean that the PGs have the latest contents.

Ceph 存储集群中每个Object至少会有两份拷贝（i.e.,`size=2`）,这是为保障数据安全性的最低要求。为了实现高可用，应该存储2份以上拷贝（e.g.,`size=3` & `min size=2`）,只有这样集群才能在降级状态下继续保障数据安全。

一般osd守护进程以osd.0, osd.1等命名，而不是osd.primary, osd.secondary。方便起见， primary osd 即第一个 `Acting Set` 的osd， primary osd 负责 `peering` 过程，并且只有primary osd能接收客户端发起的写请求.

当一系列的OSD负责一个 `PG` 时，这一系列OSD，我们把它们称为一个 `Acting Set`。

Ceph OSD Daemon 可能不总是处于 `up`. 当位于`Acting Set` 中的OSD状态为 `up`时，那么这个OSD就是 `Up Set` 中的一员。这两者有很大区别，因为当位于 `Up Set` 中的OSD Daemon失败后，Ceph重新映射PGs和OSD Daemons.

> ***Note :*** In an Acting Set for a PG containing osd.25, osd.32 and osd.61, the first OSD, osd.25, is the Primary. If that OSD fails, the Secondary, osd.32, becomes the Primary, and osd.25 will be removed from the Up Set.


#### REBALANCING

当你向Ceph存储集群中新增了一个OSD Daemon时，`cluster map`会被更新（`PG`对应的OSD会发生改变）。  
[http://docs.ceph.com/docs/master/architecture/#rebalancing](http://docs.ceph.com/docs/master/architecture/#rebalancing)

> Referring back to Calculating PG IDs, this changes the cluster map. Consequently, it changes object placement, because it changes an input for the calculations.

这里提到了增加OSD会改变object的存储位置，因为它改变了计算的输入。
但根据 [Caculating PG IDs](http://docs.ceph.com/docs/master/architecture/#caculating-pg-ids)中的描述，客户端在计算位置时，只需要输入`object id + pool name + pg num`. 其中`object id`和`pool name` 应该是不变的，`pg num` 在pool创建的时候就指定了，因此也不会随着新OSD的加入而发生变化。因此个人感觉这段描述有误。

下图展示了 rebalanceing 过程：
![img](http://docs.ceph.com/docs/master/_images/ditaa-b31e1f646135b9706000fa0799d572563dffac81.png)

>***Note :***  
>1.  当Ceph 存储集群具有一定规模的时候，加入新的OSD时，只有少部分PG和OSD的映射关系会发生改变。
>2.  即使处于 `rebalancing` 状态，CRUSH也是稳定的（因为大部分PG和OSD映射不会变化）。因此可以不必担心 `rebalancing` 期间会导致负载波动。  



### DATA CONSISTENCY
---
作为保持数据一致性和清洁度的一部分，Ceph OSD还可以擦除 `PG` 内的对象。 也就是说，Ceph OSD可以将一个 `PG` 中的对象元数据与其他OSD中存储的  `PG` 中的副本进行比较。 清洗（通常每天执行）会捕获OSD错误或文件系统错误。 OSD也可以通过比较比特中的数据进行更深的清洗。 深度清洗（通常每周执行一次）可以发现在清洗过程察觉不到的磁盘坏道。

有关配置清洗的详细信息，请参阅[Data Scrubbing](http://docs.ceph.com/docs/master/rados/configuration/osd-config-ref#scrubbing)。


### ERASURE CODING
---

纠错码池将 object 存储为 `k+m` 块。 `k` 为数据块， `m` 为编码块。 为了将每个块存储到同一 `Acting Set` 的OSD中（及同一个PG中），需要设置 `pool size=k+m`, 块的顺序被作为object的属性存储在metadata中。

例如， 消除编码池中有5个OSD（K+M=5）,其中两个块丢失（M=2）

#### READING AND WRITING ENCODED CHUNKS

一个名为 **NYAN** 的object, 数据部分为 `ABCDEFGHI` 写入纠错码池中，纠错码函数将数据分为3个数据块。分别是`ABC`, `DEF`, `GHI`. 如果内容长度不为`k` 的倍数，将会被填充。此外还会产生`m` 个编码块，分别为`YXY` 和`GQC`. 每个块都存储在同一`Acting Set`的OSD中。这些块都有相同的object name (**NAYN**)，但位于不同的OSD中。数据块之间的顺序被保存为object的一个属性（shard_t）。 如下图所示：  
![img](http://docs.ceph.com/docs/master/_images/ditaa-96fe8c3c73e5e54cf27fa8a4d64ed08d17679ba3.png)

其中`QGC`位于的OSD4，处于 `OUT` 状态无法被读取，`DEF` 位于的OSD2，读取速度太慢，即：在解码过程中，`EFG` 和 `GQC`数据块缺失（它们被称为擦除）。当object **NYAN** 从纠错码的pool中读取出来时，纠错码函数读取3个数据块：`ABC`, `GHI`, `YXY`. 然后重新构成原始内容`ABCDEFGHI`. 

![img](http://docs.ceph.com/docs/master/_images/ditaa-1f3acf28921568db86bb22bb748cbf42c9db7059.png)



#### INTERRUPTED FULL WRITES

在纠错码池中，在`Up set` 中的 primary OSD 接受所有的写操作，并负责将有效数据转化成`k+m` 个数据块，随后传送至OSD。它还负责维护`PG`日志的权威版本。

下图中，一个有纠错功能的`PG`, `k=2 + m=1`,并由三个OSD支持。分别为OSD1，OSD2，OSD3。一个经过编码的对象存储在OSD中： 块`D1v1`（第一个数据块，版本为1）位于OSD1，块`D2v1` 位于OSD2以及块`C1v1`（第一个纠错码块，版本1）位于OSD3. 每个OSD上的`PG`日志是相同的。（i.e.1,1 对应着`epoch 1, version 1`）

![img](http://docs.ceph.com/docs/master/_images/ditaa-a60e808835cf8860e19b9f2a9c83691c2a4f0218.png)

OSD1 作为primay从客户端接受了一个 **WRITE FULL** 请求，即将object全部替换，而不是重写其中某个部分。版本2创建的object会覆盖版本1. OSD1将v2 object 编码成3个数据块：D1v2, D2v2, C1v2,并分别分布到OSD1，OSD2，OSD3。OSD1除了要负责存储快的写入以外，也会维护一份`PG`日志的权威版本。

当一个OSD收到消息写入数据块时，同时也会在`PG`的log中创建一个新的条目，来反映此次修改。例如：只要OSD3存储C1v2,就会在日志中添加一条`1,2` (i.e. `epoch 1,version 2`)。

由于OSDs是异步的，当OSD1已经写入D1v2时，D2v2可能还在传送当中。即OSD2中存储的仍是D2v1.如下图所示：

![img](http://docs.ceph.com/docs/master/_images/ditaa-513e0558c5877884d43ffc9e7b792a5f77466831.png)

如果一切顺利,日志的last_complete指针可以从1,1移动到1,2。  

![img](http://docs.ceph.com/docs/master/_images/ditaa-8db474f2d1f9a795067c4aef26c0530072cfa77f.png)

最后，可以删除用于存储对象先前版本的块的文件：OSD 1上的D1v1，OSD 2上的D2v1和OSD 3上的C1v1。  

![img](http://docs.ceph.com/docs/master/_images/ditaa-4f9c14e4c28edb4b34e0eaa33253231a4d96b11f.png)

但是，如果D2v2仍在传输过程中，OSD1挂了，版本2的object只有一部分被写入了：OSD3 有一部分纠错码但并不足以恢复数据。这样一来就丢失了两个数据块：`D1v2` 和 `D2v2`.

OSD 4成为新的primary OSD，并发现last_complete日志条目（即，在此条目之前的所有对象在之前的`Acting Set`中都是可用的）为1,1，并且将成为新的权威日志。

![img](http://docs.ceph.com/docs/master/_images/ditaa-e211bf856cdbbf5d055980e95d39a4b60113c954.png)

OSD 3中发现的日志条目1,2是从OSD 4提供的新权威日志中相异：它被丢，并且包含C1v2块的文件被删除。 在刷新过程中，将通过纠错码重构D1v1数据块，并将其存储在新的主OSD 4上。 

![img](http://docs.ceph.com/docs/master/_images/ditaa-77b8a9b262ce5e9cbd7030c5da9ed7ab0edffc8a.png)



### CACHE TIERING
---

高速缓存层为Ceph客户端存储在后端存储层中的数据子集提供了更好的I / O性能。高速缓存层需要创建一个`pool` 来关联一个快速存储设备（e.g. ssd）,并设置为高速缓存层。一些便宜的低速设备可配置为后端存储层。`Ceph Objecter` 会决定object的存储位置，分层代理决定何时将缓存层中的数据刷新到后端存储层。因此，缓存层和后备存储层对Ceph客户端是完全透明的。

![img](http://docs.ceph.com/docs/master/_images/ditaa-2982c5ed3031cac4f9e40545139e51fdb0b33897.png)
