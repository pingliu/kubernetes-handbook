# 快速安装

## 安装前的准备

1. 一台装有centos 7.3以上并且能够访问外网的虚拟机。

2. 关闭selinux。

   **永久方法 – 需要重启服务器**

   修改`/etc/selinux/config`文件中设置SELINUX=disabled ，然后重启服务器。

   **临时方法 – 设置系统参数**

   使用命令`setenforce 0`

## 部署

```bash
$ wget http://103.210.237.214/download/v1.9.5/kubernetes-install.tar.gz
$ tar zxf kubernetes-install.tar.gz
$ cd kubernetes-install
$ ./install.sh 172.16.129.100
```

提示：在部署时需要一点时间，因为需要下载kubernetes的二进制包。172.16.129.100是虚拟机的ip。

## 部署验证
       
```bash
$ kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
kvm-1     Ready     <none>    4h        v1.9.5
$ kubectl run nginx --replicas=2 --labels="run=load-balancer-example" --image=hub.c.163.com/library/nginx:latest  --port=80
deployment "nginx" created
$ kubectl expose deployment nginx --type=NodePort --name=example-service
service "example-service" exposed
$ kubectl describe svc example-service
Name:                     example-service
Namespace:                default
Labels:                   run=load-balancer-example
Annotations:              <none>
Selector:                 run=load-balancer-example
Type:                     NodePort
IP:                       10.254.211.15
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30175/TCP
Endpoints:                172.30.42.2:80,172.30.42.3:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
$ curl 10.254.211.15:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## 接下来阅读

