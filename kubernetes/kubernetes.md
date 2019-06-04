### kubernetes简介
Kubernetes是谷歌开源的容器集群管理系统，是Google多年大规模容器管理技术Borg的开源版本，主要功能包括：
- 基于容器的应用部署、维护和滚动升级
- 负载均衡和服务发现
- 跨机器和跨地区的集群调度
- 自动伸缩
- 无状态服务和有状态服务
- 广泛的Volume支持
- 插件机制保证扩展性 

Kubernetes发展非常迅速，已经成为容器编排领域的领导者。
Kubernetes通过声明式的API和一系列独立、可组合的
控制器保证了应用总是在期望的状态，而用户并不需要关心中间状态是如何转换的。这使得整个系统更容易使用，而且更强
大、更可靠、更具弹性和可扩展性。

### 核心组件
Kubernetes主要由以下几个核心组件组成：
- etcd保存了整个集群的状态；
- apiserver提供了资源操作的唯一入口，并提供认证、授权、访问控制、API注册和发现等机制；
- controller manager负责维护集群的状态，比如故障检测、自动扩展、滚动更新等；
- scheduler负责资源的调度，按照预定的调度策略将Pod调度到相应的机器上；
- kubelet负责维护容器的生命周期，同时也负责Volume（CVI） 和网络（CNI） 的管理；
- Container runtime负责镜像管理以及Pod和容器的真正运行（CRI） ；
- kube-proxy负责为Service提供cluster内部的服务发现和负载均衡
![image](https://www.hi-linux.com/img/linux/k8s-arch1.png)


- kubectl是 Kubernetes 的命令行工具（CLI），是 Kubernetes 用户和管理员必备的管理工具

### 资源对象
#### Pod
Pod 是一组紧密关联的容器集合，它们共享 IPC、Network 和 UTS 命名空间，是 Kubernetes 调度的基本单位。Pod 的设
计理念是支持多个容器在一个 Pod 中共享网络和文件系统，可以通过进程间通信和文件共享这种简单高效的方式组合完成服
务。

**Pod的特征** 
- 包含多个共享 IPC、Network 和 UTC 命名空间 的容器，可直接通过 localhost 通信
- 所有 Pod 内容器都可以访问共享的 Volume，可以访问共享数据
- 无容错性：直接创建的 Pod 一旦被调度后就跟 Node 绑定，即使 Node 挂掉也不会被重新调度（而是被自动删除） 
- 优雅终止：Pod删除的时候先给其内的进程发送SIGTERM，等待一段时间（grace period）后才强制停止依然还在运行的进程

**pod定义**

通过 yaml 或 json 描述 Pod 和其内容器的运行环境以及期望状态,示例：
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: harbor.fonsview.com:31883/openplatform/nginx
    ports:
    - containerPort: 80
```
#### ReplicaSet
用来确保容器应用的副本数始终保持在用户定义的副本数，即如果有容器异常退出，会自动创建新的Pod来替代；而异常多出来的容器也会自动回收。典型应用场景包括确保健康Pod的数量、弹性伸缩、滚动升级以及应用多版本发布跟踪等。
示例：
```
apiVersion: extensions/v1beta1
kind: ReplicationSet
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: harbor.fonsview.com:31883/openplatform/nginx
        ports:
        - containerPort: 80
```
#### Deployment
支持滚动更新、版本记录、回滚、暂停升级等高级特性
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: harbor.fonsview.com:31883/openplatform/nginx
        ports:
        - containerPort: 80
```
#### DaemonSet
DaemonSet 保证在每个Node上都运行一个容器副本，常用来部署一些集群的日志、监控或者其他系统管理应用。
- 支持滚动更新，可以通过.spec.updateStrategy.type 设置更新策略。目前支持两种策略: </br>
`OnDelete`：默认策略，更新模板后，只有手动删除了旧的 Pod 后才会创建新的 Pod </br>
`RollingUpdate`：更新 DaemonSet 模版后，自动删除旧的 Pod 并创建新的 Pod，在使用RollingUpdate策略时，还可以设置.spec.updateStrategy.rollingUpdate.maxUnavailable , 默认 1 
- 支持回滚

#### 将pod调度至指定Node节点
- nodeSelector：只调度到匹配指定 label 的 Node 上
- nodeAffinity：功能更丰富的 Node 选择器，比如支持集合操作
- podAffinity：调度到满足条件的 Pod 所在的 Node 上

#### Namespace
Namespace 是对一组资源和对象的抽象集合，比如可以用来将系统内部的对象划分为不同的项目组或用户组。

**创建**
```
kubectl create namespace new-namespace
```
或通过yaml文件创建
```
apiVersion: v1
kind: Namespace
metadata:
  name: new-namespace
```

#### Service
Kubernetes 在设计之初就充分考虑了针对容器的服务发现与负载均衡机制，提供了 Service 资源。

Service 是对一组提供相同功能的Pods的抽象，并为它们提供一个统一的入口。借助Service，应用可以方便的实现服务发现与负载均衡，并实现应用的零宕机升级。Service通过标签来选取服务后端，一般配合 Replicationset或者Deployment来保证后端容器的正常运行。这些匹配标签的 Pod IP和端口列表组成endpoints，由 kube-proxy 负责将服务 IP负载均衡到这些 endpoints 上。
![image](https://static.oschina.net/uploads/img/201702/16004423_IjJy.png)

示例：
```
apiVersion: v1
kind: Service
metadata:
  labels:
    run: nginx
    name: nginx
  namespace: default
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: ClusterIP
```
**服务发现**

Kubernetes集群提供了DNS插件，DNS 服务器监视着创建新 Service 的 Kubernetes API，从而为每个 Service 创建一组 DNS 记录。DNS会为每一个service指定一个`A`记录，名为`my-svc.my-namespace.svc.cluster.local`,指向service的ClusterIP。

**service类型**
- ClusterIP
- NodePort
- LoadBalancer
- ExternalName

### hpa
Horizontal Pod Autoscaling，根据pod 的CPU使用率或应用自定义metrics，让service中的Pod个数自动调整。hpa由API server和controller共同实现。
![image](https://jimmysong.io/kubernetes-handbook/images/horizontal-pod-autoscaler.png)
Controller manager 会周期性的从 resource metric API（每个 pod 的 resource metric）或者自定义 metric API（所有的metric）中获取 metric。

示例：
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: harbor.fonsview.com:31883/openplatform/nginx
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 1
            memory: 1Gi
          requests:
            cpu: 1
            memory: 1Gi
            
# kubectl autoscale deployment nginx-deployment --min=2 --max=5 --cpu-percent=80
```

### Rolling Update 
当有镜像发布新版本，新版本服务上线时如何实现服务的滚动和平滑升级？

如果你使用ReplicationSet创建的pod可以使用kubectl rollingupdate命令滚动升级，如果使用的是Deployment创建的Pod可以直接修改yaml文件后执行kubectl apply即可。

Deployment已经内置了RollingUpdate strategy，因此不用再调用kubectl rollingupdate命令，升级的过程是先创建新版的pod将流量导入到新pod上后销毁原来的旧的pod。
```
$ kubectl rolling-update NAME -f FILE

如果只是更新镜像，则可以使用：
$ kubectl rolling-update NAME [NEW_NAME] --image=IMAGE:TAG

如果滚动更新是出现问题，还可以回滚：
$ kubectl rolling-update NAME --rollback
``` 
**回滚**
```
# 查询历史版本
$ kubectl rollout history daemonset <daemonset-name>
# 查询某个历史版本的详细信息
$ kubectl rollout history daemonset <daemonset-name> --revision=1
# 回滚
$ kubectl rollout undo daemonset <daemonset-name> --to-revision=<revision>
# 查询回滚状态
$ kubectl rollout status ds/<daemonset-name>
```

### volume
我们知道默认情况下容器的数据都是非持久化的，在容器消亡以后数据也跟着丢失，所以Docker提供了Volume机制以便将数据持久化存储。类似的，Kubernetes提供了更强大的Volume机制和丰富的插件，解决了容器数据持久化和容器间共享数据的问题。</br>
与 Docker 不同，Kubernetes Volume 的生命周期与 Pod 绑定 </br>
- 容器挂掉后 Kubelet 再次重启容器时，Volume 的数据依然还在
而 Pod 删除时，Volume 才会清理
- 数据是否丢失取决于具体的 Volume 类型，比如 emptyDir 的数据会丢失，而 PV 的数据则不会丢

**Volume类型**
- emptyDir
- hostPath
- nfs
- configMap
- persistentVolumeClaim
- rbd
- cephfs
- glusterfs
- gcePersistentDisk
- awsElasticBlockStore
- ...

**emptyDir**

如果 Pod 设置了 emptyDir 类型 Volume， Pod 被分配到 Node 上时候，会创建 emptyDir，只要 Pod 运行在 Node 上，
emptyDir 都会存在（容器挂掉不会导致 emptyDir 丢失数据） ，但是如果 Pod 从 Node 上被删除（Pod 被删除，或者 Pod 发
生迁移） ，emptyDir 也会被删除，并且永久丢失。
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: harbor.fonsview.com:31883/openplatform/nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

**hostPath**

hostPath 允许挂载 Node 上的文件系统到 Pod 里面去。如果 Pod 需要使用 Node 上的文件，可以使用 hostPath。
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: harbor.fonsview.com:31883/openplatform/nginx
    ports:
    - containerPort: 80
  volumeMounts:
  - mountPath: /test-pd
    name: test-volume
  volumes:
  - name: test-volume
    hostPath:
    path: /data
```

**NFS**

Kubernetes 中通过简单地配置就可以挂载 NFS 到 Pod 中，而 NFS中的数据是可以永久保存的，同时 NFS 支持同时写操作。
```
...
volumes:
- name: nfs
  nfs:
    server: 10.254.234.223
    path: "/"
```

**configMap**

许多应用程序会从配置文件、命令行参数或环境变量中读取配置信息。 这些配置信息需要与docker image解耦，不能每修改一个配置就重做一个image，ConfigMap API给我们提供了向容器中注入配置信息的机制，ConfigMap可以被用来保存单个属性，也可以用来保存整个配置文件或者JSON二进制大对象。
ConfigMaps可以被用来：
- 设置环境变量的值
- 在容器里设置命令行参数
- 在数据卷里面创建config文件
```
# kubectl create configmap my-config --from-file=path/to/bar

apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-cm
data:
  config； |
    user nginx;    
    worker_processes  1;
    
    error_log  /var/log/nginx/error.log;
    pid        /var/run/nginx.pid;
    
    events {
      use   epoll; 
      worker_connections  1024;
    }
    
    http {
      include       /etc/nginx/mime.types;
      default_type  application/octet-stream;
      access_log    /var/log/nginx/access.log;
    
      sendfile        on;
    
      keepalive_timeout  65;
      tcp_nodelay        on;
    
      include /etc/nginx/conf.d/*.conf;
      }
    
     server {
          listen       80;
          server_name  www.fosnview.com;
    
          access_log  logs/access.log  main;
    
          location / {
            root   /root;
            index index.php index.html index.htm;   
          }
    
          error_page   500 502 503 504 /50x.html;  
          location = /50x.html {
          root   /root;
      }

---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: harbor.fonsview.com:31883/openplatform/nginx
    ports:
    - containerPort: 80
  volumeMounts:
  - mountPath: /usr/share/nginx/html
    name: config
  volumes:
  - name: config
    configMap:
      name: nginx-cm
      items:
      - key: config
        path: nginx.conf
```

**PersistentVolume (PV) & PersistentVolumeClaim (PVC)**

参考： https://www.jianshu.com/p/fda9de00ba5f

PersistentVolume（pv）和PersistentVolumeClaim（pvc）是k8s提供的两种API资源，用于抽象存储细节。管理员关注于如何通过pv提供存储功能而无需关注用户如何使用，同样的用户只需要挂载pvc到容器中而不需要关注存储卷采用何种技术实现。</br>
pvc可以向pv申请指定大小的存储资源并设置访问模式,这就可以通过Provision -> Claim 的方式，来对存储资源进行控制。

**Provisioning**

PV资源供给方式：
- static：通过集群管理者创建多个PV，为集群“使用者”提供存储能力而隐藏真实存储的细节。
- dynamic：动态卷供给是kubernetes独有的功能，这一功能允许按需创建存储建。在此之前，集群管理员需要事先在集群外由存储提供者或者云提供商创建

**PV类型**
- GCEPersistentDisk
- AWSElasticBlockStore
- NFS
- iSCSI
- RBD (Ceph Block Device)
- Glusterfs
- AzureFile
- AzureDisk
- CephFS
- cinder
- FC
- FlexVolume
- Flocker
- PhotonPersistentDisk
- Quobyte
- VsphereVolume
- HostPath (single node testing only – local storage is not supported in any way and WILL NOT WORK in a multi-node cluster)

示例：
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2
    
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow
  selector:
    matchLabels:
      release: "stable"
    matchExpressions:
      - {key: environment, operator: In, values: [dev]}
      
---
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```
**pv & pvc生命周期**

pv和pvc遵循以下生命周期：

**供应准备** 通过集群外的存储系统或者云平台来提供存储持久化支持。
- 静态提供：管理员手动创建多个PV，供PVC使用。
- 动态提供：动态创建PVC特定的PV，并绑定。

**绑定** 用户创建pvc并指定需要的资源和访问模式。在找到可用pv之前，pvc会保持未绑定状态。

**使用** 用户可在pod中像volume一样使用pvc。

**释放** 用户删除pvc来回收存储资源，pv将变成“released”状态。由于还保留着之前的数据，这些数据需要根据不同的策略来处理，否则这些存储资源无法被其他pvc使用。

**回收(Reclaiming)** pv可以设置三种回收策略：保留（Retain），回收（Recycle）和删除（Delete）。
- 保留策略：允许人工处理保留的数据。
- 删除策略：将删除pv和外部关联的存储资源，需要插件支持。
- 回收策略：将执行清除操作，之后可以被新的pvc使用，需要插件支持。

**PV卷状态：**
- Available – 资源尚未被claim使用
- Bound – 卷已经被绑定到claim了
- Released – claim被删除，卷处于释放状态，但未被集群回收。
- Failed – 卷自动回收失败


参考：
- https://jimmysong.io/kubernetes-handbook/
- https://kubernetes.io/docs/
- https://kubernetes.feisky.xyz/
