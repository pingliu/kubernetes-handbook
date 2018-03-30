## Namespaces

一个kubernetes集群可以划分为多个虚拟集群，这个虚拟集群的名称就叫namespace。在实际使用中，会根据项目和团队对一个kubernetes集群划分成不同的namespace。

## Working with Namespaces

获取namespace
```
$ kubectl get ns
NAME          STATUS    AGE
default       Active    2d
kube-public   Active    2d
kube-system   Active    2d
```
kube-public：里面的对象所有用户可读。

default：当创建对象没有申明namespace，一般应用会在这个下面。

kube-system：kubernetes系统创建的对象的namespace，一般与集群管理相关的服务在这个下面。

另外不是所有的对象资源都在namespace下，比如nodes和persistentVolumes不属于任何namespace。
