## 理解Pods

Pod是kubernetes最小的业务部署单元，一个pod代表一个进程。一个Pod将容器、存储资源、ip等资源封装在一起，一般一个pod由一个容器或者多个小的容器组成。

> [Docker](https://www.docker.com)是kubernetes中最常用的容器运行时，但是Pod也支持其他容器运行时。

Pods在kubernetes中有两种使用方式：

- **一个Pod中运行一个容器**。这个模式是最通用的使用场景，在这种模式下，你可以把pod想象成一个容器的封装，kubernetes直接管理的是pod而不是容器。
- **一个Pod中运行多个容器**。一个pod可以封装多个紧密耦合并需要共享资源的容器。这个模式在使用场景中比较少见。

每个pod对应着一个应用的实例，如果你想水平扩展你的应用，可以使用多个pods。在kubernetes里一般叫做replication，这些副本通过[Pods and Controllers](#1)控制。

## Pod如何管理多个容器

Pods被设计成支持多个进程协同工作，属于同一个pod的容器会被调度到同一个节点上，这些容器可以共享资源，依赖关系，互相通信。

一个pod使用多个容器是一种高级用法，实际场景很少见，除非是紧密耦合的。比如一个容器作为web服务需要共享文件，一个“sidecar”容器从远端更新这些文件，如下图所示：

![pod diagram](../images/pod-overview.png)

Pod中可以共享两种资源：网络和存储。

#### 网络

每个Pod都会被分配一个唯一的IP地址。Pod中的所有容器共享网络空间，包括IP地址和端口。Pod内部的容器可以使用`localhost`互相通信。Pod中的容器与外界通信时，必须分配共享网络资源（例
如使用宿主机的端口映射）。

#### 存储

可以Pod指定多个共享的Volume。Pod中的所有容器都可以访问共享的volume。Volume也可以用来持久化Pod中的存储资源，以防容器重启后文件丢失。

## 使用Pod

你很少会直接在kubernetes中创建单个Pod。因为Pod的生命周期是短暂的，用后即焚的实体。当Pod被创建后（不论是由你直接创建还是被其他Controller），都会被Kuberentes调度到集群的Node上。直到Pod的进程终止、被删掉、因为缺少资源而被驱逐、或者Node故障之前这个Pod都会一直保持在那个Node上。

> 注意：重启Pod中的容器跟重启Pod不是一回事。Pod只提供容器的运行环境并保持容器的运行状态，重启容器不会造成Pod重启。

Pod不会自愈。如果Pod运行的Node故障，或者是调度器本身故障，这个Pod就会被删除。同样的，如果Pod所在Node缺少资源或者Pod处于维护状态，Pod也会被驱逐。Kubernetes使用更高级的称为Controller的抽象层，来管理Pod实例。虽然可以直接使用Pod，但是在Kubernetes中通常是使用Controller来管理Pod的。

### <h3 id="1">Pods and Controllers</h3>

Controller可以创建和管理多个Pod，提供副本管理、滚动升级和集群级别的自愈能力。例如，如果一个Node故障，Controller就能自动将该节点上的Pod调度到其他健康的Node上。

包含一个或者多个Pod的Controller示例：

- [Deployment](deployment.md)
- [StatefulSet](statefulset.md)
- [DaemonSet](daemonset.md)

通常，Controller会用你提供的Pod Template来创建相应的Pod。

## Pod Templates

Pod模版是包含了其他object的Pod定义，例如[Replication Controllers](replicaset.md)，[Jobs](job.md)和
[DaemonSets](daemonset.md)。Controller根据Pod模板来创建实际的Pod。

Pod模版的定义除了不需要声明apiVersion和kind，其他字段和pod的字段一样。
