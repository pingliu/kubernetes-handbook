## DaemonSet

DaemonSet确保每个节点都会运行一个pod。经典的使用场景：

- 每个节点运行存储程序，比如glusterd，ceph。
- 每个节点运行日志收集程序，比如fluentd，logstash。
- 每个节点运行监控程序，比如prometheus，collectd。

## 编写Spec

参考例子`yaml/daemonset.yaml`。使用`kubectl create -f yaml/daemonset.yaml`创建DaemonSet。

### 需要的Fields

同其他资源一样，需要`apiVersion`，`kind`，`metadata`，`.spec`字段。

### Pod Template

这个和[pods](pod-overview.md)的pod template一样。

### Pod Selector

在1.8版本后，在`.spec.template`字段里必须声明匹配pod selector的标签。而且一旦创建后，`spec.selector`不能被改变。

`spec.selector`包含2个字段：

- `matchLabels`，和Deployments里的`spec.selector`同样的效果。
- `matchExpressions`，可以用来构建更加复杂的匹配规则。

两个字段同时出现时，结果是AND关系。

当`.spec.selector`定义时，必须匹配`.spec.template.metadata.labels`，否则会拒绝这个请求。

### 仅在某些节点运行Pods

可以通过`.spec.template.spec.nodeSelector`，然后DaemonSet会在匹配到的节点创建pods。默认情况下会在所有节点创建pods。

## 和Daemon Pods通信

和DaemonSet里的pods有以下几种方式：

- Push：一些pods被用来发送更新到其他服务，比如数据库状态。不需要客户端。
- NodeIP和Port：在DaemonSet里的pods可以使用`hostPort`，这样pods可以通过访问nodeIP和port访问。
- DNS：创建具有相同pod selector的`headless service`，这样通过使用`endpoints`或者dns查询发现DaemonSet。
- Service：创建具有相同 Pod Selector 的 Service，并使用该 Service 访问到某个随机 Node 上的 daemon。
