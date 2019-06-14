kubernetes 配置：

测试node宕机，pod需要等几分才会在其它的node上重新启动，这个明显不合理，对于大多数业务

kube-controller-manager配置：

  /etc/systemd/system/kube-controller-manager.service

--node-monitor-grace-period=10s \
--node-monitor-period=3s \
--node-startup-grace-period=20s \
--pod-eviction-timeout=10s \

 

kubernetes节点失效后pod的调度过程：

0、Master每隔一段时间和node联系一次，判定node是否失联，这个时间周期配置项为 node-monitor-period ，默认5s

1、当node失联后一段时间后，kubernetes判定node为notready状态，这段时长的配置项为 node-monitor-grace-period ，默认40s

2、当node失联后一段时间后，kubernetes判定node为unhealthy，这段时长的配置项为 node-startup-grace-period ，默认1m0s

3、当node失联后一段时间后，kubernetes开始删除原node上的pod，这段时长配置项为 pod-eviction-timeout ，默认5m0s

在应用中，想要缩短pod的重启时间，可以修改上述几个参数



解释 官方有：

 
--node-monitor-grace-period duration     Default: 40s
 	Amount of time which we allow running Node to be unresponsive before marking it unhealthy. Must be N times more than kubelet's nodeStatusUpdateFrequency, where N means number of retries allowed for kubelet to post node status.

--node-monitor-period duration     Default: 5s
 	The period for syncing NodeStatus in NodeController.

--node-startup-grace-period duration     Default: 1m0s
 	Amount of time which we allow starting Node to be unresponsive before marking it unhealthy.

--pod-eviction-timeout duration     Default: 5m0s
 	The grace period for deleting pods on failed nodes.
