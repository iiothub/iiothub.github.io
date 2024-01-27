* TOC
{:toc}


## 一、概述



### 1.hostNetwork

在 k8s 中，若 **pod 使用主机网络**，也就是`hostNetwork=true`。则该pod会使用主机的dns以及所有网络配置，**默认情况下是无法使用 k8s 自带的 dns 解析服务**，但是可以修改 DNS 策略或者修改主机上的域名解析（`/etc/resolv.conf`），使主机可以用 k8s 自身的 dns 服务。一般通过 DNS 策略（`ClusterFirstWithHostNet`）来使用 k8s DNS 内部域名解析

**k8s DNS 策略如下：**

- `Default`：继承 Pod 所在宿主机的 DNS 设置，hostNetwork 的默认策略。
- `ClusterFirst（默认DNS策略）`：优先使用 kubernetes 环境的 dns 服务，将无法解析的域名转发到从宿主机继承的 dns 服务器。
- `ClusterFirstWithHostNet`：和 ClusterFirst 类似，对于以 `hostNetwork` 模式运行的 Pod 应明确知道使用该策略。也是可以同时解析内部和外部的域名。
- `None`：忽略 kubernetes 环境的 dns 配置，通过 spec.dnsConfig 自定义 DNS 配置。



一般使用主机网络就增加如下几行即可：

```shell
hostNetwork: true
dnsPolicy: "ClusterFirstWithHostNet"
```

**『示例』：`hostNetwork.yaml`**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      # 使用主机网络
      hostNetwork: true
      # 该设置是使POD使用k8s的dns，dns配置在/etc/resolv.conf文件中
      # 如果不加，pod默认使用所在宿主主机使用的DNS，这样会导致容器
      # 内不能通过service name访问k8s集群中其他POD
      dnsPolicy: ClusterFirstWithHostNet
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - name: metrics
          # 如果hostNetwork: true，hostPort必须跟containerPort一样，所以hostPort一般不写，端口也是占用宿主机上的端口。
          hostPort: 80
          containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  type: NodePort
  ports:
    - name: http
      port: 80
      targetPort: 80
      protocol: TCP
      nodePort: 31280
```



### 2.**hostPort** 

**hostPort 和 NodePort的区别：**

`hostPort` 只会在运行机器上开启端口， `NodePort` 是所有 Node 上都会开启端口。

- hostPort 是由 portmap 这个 cni 提供 portMapping 能力，同时如果想使用这个能力，在配置文件中一定需要开启 portmap。
- 使用 hostPort 后，会在 iptables 的 nat 链中插入相应的规则，而且这些规则是在 KUBE-SERVICES 规则之前插入的，也就是说会优先匹配 hostPort 的规则，我们常用的 NodePort 规则其实是在 KUBE-SERVICES 之中，也排在其后。
- hostport 可以通过 iptables 命令查看到， 但是无法在 ipvsadm 中查看到。
- 使用 lsof/netstat 也查看不到这个端口,这是因为 hostport 是通过 iptables 对请求中的目的端口进行转发的，并不是在主机上通过端口监听。
- **在生产环境中不建议使用 hostPort**。



### 3.Pod主机网络选择

**1.使用主机网络 - hostNetwork**

- 设置 hostNetwork: true

- hostPort必须跟containerPort一样，所以hostPort一般不写，端口也是占用宿主机上的端口

- Edge节点只能运行一个Pod，否则端口冲突

  

**2.使用hostPort**

- 可以改变默认端口，或开放部分端口
- 如果hostNetwork: true，hostPort必须跟containerPort一样，所以hostPort一般不写





## 二、EdgeCore



### 1.hostNetwork

```shell
    spec:
      # 使用主机网络
      hostNetwork: true
      # 该设置是使POD使用k8s的dns，dns配置在/etc/resolv.conf文件中
      # 如果不加，pod默认使用所在宿主主机使用的DNS，这样会导致容器
      # 内不能通过service name访问k8s集群中其他POD
      dnsPolicy: ClusterFirstWithHostNet
```



```shell
[root@k8s-master kubeedge]# vim nginx.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      nodeName: edge-2    #调度到指定机器
      hostNetwork: true   # 使用主机网络
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          hostPort: 8080
```

```shell
[root@k8s-master kubeedge]# kubectl apply -f nginx.yaml 
deployment.apps/nginx-deployment created


[root@k8s-master kubeedge]# kubectl get pod -owide
NAME                                READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
nginx-deployment-6bd49f646f-njpb2   1/1     Running   0          28s   172.17.0.3   edge-2   <none>           <none>
```

```shell
# 访问
curl 192.168.202.212
curl 192.168.202.212:80

[root@k8s-master kubeedge]# curl 192.168.202.212:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
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
</html
```



### 2.hostPort

```shell
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - name: metrics
          # 如果hostNetwork: true，hostPort必须跟containerPort一样，所以hostPort一般不写，端口也是占用宿主机上的端口。
          hostPort: 80
          containerPort: 80
```



```shell
[root@k8s-master kubeedge]# vim nginx-hostport.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      nodeName: edge-2  #调度到指定机器
      #hostNetwork: true
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          hostPort: 8080  #主机端口
```

```shell
[root@k8s-master kubeedge]# kubectl apply -f nginx-hostport.yaml 
deployment.apps/nginx-deployment created


[root@k8s-master kubeedge]# kubectl get pod -owide
NAME                                READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
nginx-deployment-6bd49f646f-njpb2   1/1     Running   0          28s   172.17.0.3   edge-2   <none>           <none>
```

```shell
# 访问
curl 192.168.202.212
curl 192.168.202.212:8080


[root@k8s-master kubeedge]# curl 192.168.202.212
curl: (7) Failed connect to 192.168.202.212:80; Connection refused


[root@k8s-master kubeedge]# curl 192.168.202.212:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
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




