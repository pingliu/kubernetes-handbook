## Node

Node是kubernetes的工作节点，可以是物理机器和虚拟机。每个节点需要运行的程序是：docker，kubelet，kube-proxy。

## Node Status

node有如下状态信息：

- Addresses
    - HostName：节点的主机名，可以被kubelet中的`--hostname-override`参数替代。
    - ExternalIP：节点的ip地址，可以被集群外部访问的地址，一般为空。
    - InternalIP：节点的ip地址，可以被集群内部访问的地址，一般就是局域网地址。
- Condition
    - OutOfDisk：为True表示节点磁盘不足，一般为False。
    - Ready：为True表示节点健康允许被调度，一般为True。
    - MemoryPressure：为True表示节点内存有压力，一般为False。
    - DiskPressure：为True表示节点磁盘空间有压力，一般为False。
- Capacity
    - 节点可用的cpu个数
    - 节点可用的内存大小
    - 节点可运行的最大pod数量，可以被kubelet中的`--max-pods`参数替代。
- Info
    - 节点的版本信息，比如kernel，kubelet，kube-proxy,docker等。

## Node Controller

Node Controller是kube-controller-manager一部分，主要管理node的生命周期，维护内部节点列表与云提供商的可用机器列表的更新，监控节点的健康状态。

Node Controller默认每5s去同步一次节点的状态，这个时间可以通过`--node-monitor-period`参数替代。

Node Controller默认40s超时则定义节点不健康，这个时间可以通过`--node-monitor-grace-period`参数替代。

Node Controller认定节点不健康后，默认5min后开始驱逐节点上的pod信息，这个时间可以通过`--pod-eviction-timeout`参数替代。

Node Controller认定节点不健康后，驱逐速度默认是每10s不超过1个，这个值可以通过`--node-eviction-rate`参数替代。

## Node Admin

```bash
# 标记node为不可调度状态，不会影响节点已经存在的pod
$ kubectl cordon $NODENAME

# 驱逐该节点上的所有pod，DaemonSet除外
$ kubectl drain $NODENAME

# 标记node为可调度状态
$ kubectl uncordon $NODENAME
```
