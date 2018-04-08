## Deployments

Deployments控制器是为[Pods](pod.md)和[ReplicaSets]提供了声明式的更新。您只需要在Deployment中描述您想要的目标状态是什么，Deployment controller就会帮您将Pod和ReplicaSet的实际状态改变到您的目标状态。

## 用例

Deployments的经典用例：

- 使用Deployment部署ReplicaSet。ReplicaSet会在后台创建pods。
- 通过更新PodTemplateSpec来声明pods的新状态。Deployment以一定的控制速率删除旧的ReplicaSet并创建新的ReplicaSet。
- 回滚到旧版本。每次回滚会更新revision。
- 扩容来满足更高的负载。
- 暂停Deployment来修复问题，然后继续部署。
- 使用Deployment的状态来判断部署是否夯住。
- 删除ReplicaSet。

## 创建Deployment

```bash
$ kubectl create -f yaml/nginx.yaml
deployment "nginx" created

$ kubectl get deployments
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx     2         2         2            2           17s
```

会看到这个正是我们期望的状态。`NAME`对应`metadata.name`，`DESIRED`对应`spec.replicas`，`CURRENT`对应`status.replicas`，`UP-TO-DATE`对应`status.updatedReplicas`，`AVAILABLE`对应`status.availableReplicas`，`AGE`代表app运行了多长时间。使用`kubectl get deployments -o yaml`可以看到`status`的信息。

下面的结果显示部署成功完成。
```bash
$ kubectl rollout status deployment/nginx
deployment "nginx" successfully rolled out
```

```bash
$ kubectl get rs --show-labels
NAME               DESIRED   CURRENT   READY     AGE    LABELS
nginx-76dd6b6487   2         2         2         10m    app=nginx,pod-template-hash=3288262043
```

会发现ReplicaSet的名字是由`[deployment-name]-[hash-value]`构成。这个hash值是Deployment创建时自动生成的。

```bash
$ kubectl get pods --show-labels
NAME                     READY     STATUS    RESTARTS   AGE       LABELS
nginx-76dd6b6487-b597r   1/1       Running   0          16m       app=nginx,pod-template-hash=3288262043
nginx-76dd6b6487-sbgfh   1/1       Running   0          16m       app=nginx,pod-template-hash=3288262043
```

**注意**：在Deployment中必须定义一个唯一的标签。这个标签不要和其他控制器的标签重名，否则会有意外的冲突和行为。

## 更新Deployment

更新deployment有两种方式：

- 执行`kubectl edit deploy name`进行更改。
- 修改对应的yaml文件，然后执行`kubectl replace -f file.yaml`。

如果只想更新pods，需要更新`template`字段里的内容。Deployment会确保按照一定的速率来更新pods。默认是25%的比例，意思是先创建25%的新pods，然后销毁旧的pods再创建。

**注意**：不到万不得已不要去修改标签。

## 回滚Deployment

有时候需要回滚Deployment，比如它不稳定时。默认情况下会记录Deployment的revision记录，可以使用这个记录回滚。

```bash
$ kubectl rollout history deploy nginx
deployments "nginx"
REVISION  CHANGE-CAUSE
4         <none>
5         <none>

# 回滚到上个版本
$ kubectl rollout undo deployment/nginx
deployment "nginx" rolled back 

#回滚到指定版本
$ kubectl rollout undo deployment/nginx --to-revision=4
deployment "nginx" rolled back
```

## 扩展Deployment

```bash
# 手动扩容deployment
$ kubectl scale deployment nginx --replicas=5
deployment "nginx" scaled

# 基于cpu利用率来自动扩容
$ kubectl autoscale deployment nginx --min=5 --max=10 --cpu-percent=80
deployment "nginx" autoscaled
```

### 按照比例扩容

扩容时支持多个版本在同一时间运行。为了降低风险，Deployment controller 将会平衡已存在的活动中的 ReplicaSet（有 Pod 的 ReplicaSet）和新加入的 replica。这被称为比例扩容。
可以通过`maxSurge`和`maxUnavailable`指定比例。

## 暂停和恢复Deployment

在触发更新时可以暂停然后恢复。这个功能允许我们去修复一些问题的时候，不会去触发不必要的rollout操作。

```bash
# 暂停rollout
$ kubectl rollout pause deployment/nginx
deployment "nginx" paused

$ kubectl set image deploy/nginx nginx=hub.c.163.com/library/nginx:1.13.0
deployment "nginx" image updated

# 会发现没有启动新的rollout
$ kubectl rollout history deploy nginx
deployments "nginx"
REVISION  CHANGE-CAUSE
4         <none>
5         <none>

# 恢复rollout
$ kubectl rollout resume deploy/nginx
```

## Deployment状态
















