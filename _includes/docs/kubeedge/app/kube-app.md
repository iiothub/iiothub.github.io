* TOC
{:toc}


## 一、概述



![](/images/kubeedge/app/kube-app/ap-1.png)



### 1.Kubernetes 对 Pod 调度规则

**Kubernetes 四种调度方式：**

- 自动调度：运行在哪个节点上完全由Scheduler经过一系列的算法计算得出
- 定向调度：NodeName、NodeSelector
- 亲和性调度：NodeAffinity、PodAffinity、PodAntiAffinity
- 污点（容忍）调度：Taints、Toleration



#### 1.1.自动调度

在默认情况下，一个Pod在哪个Node节点上运行，是由Scheduler组件采用相应的算法计算出来的，这个过程是不受人工控制的。



#### 1.2.定向调度

利用在pod上声明nodeName或者nodeSelector，以此将Pod调度到期望的node节点上。注意，这里的调度是强制的，这就意味着即使要调度的目标Node不存在，也会向上面进行调度，只不过pod运行失败而已。



- **nodeName**

NodeName用于强制约束将Pod调度到指定的Name的Node节点上。这种方式，其实是直接跳过Scheduler的调度逻辑，直接将Pod调度到指定名称的节点。

示例：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodename
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
  nodeName: node1 # 指定调度到node1节点上
```



- **nodeSelector**

NodeSelector用于将pod调度到添加了指定标签的node节点上。它是通过kubernetes的label-selector机制实现的，也就是说，在pod创建之前，会由scheduler使用MatchNodeSelector调度策略进行label匹配，找出目标node，然后将pod调度到目标节点，该匹配规则是强制约束。示例：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodeselector
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
  nodeSelector: 
    nodeenv: pro # 指定调度到具有nodeenv=pro标签的节点上
```



#### 1.3.亲和性调度

它在NodeSelector的基础之上的进行了扩展，可以通过配置的形式，实现优先选择满足条件的Node进行调度，如果没有，也可以调度到不满足条件的节点上，使调度更加灵活。

**Affinity主要分为三类：**

- nodeAffinity（node亲和性）：以Node为目标，解决Pod可以调度到那些Node的问题
- podAffinity（pod亲和性）：以Pod为目标，解决Pod可以和那些已存在的Pod部署在同一个拓扑域中的问题
- podAntiAffinity（pod反亲和性）：以Pod为目标，解决Pod不能和那些已经存在的Pod部署在同一拓扑域中的问题



**关于亲和性和反亲和性的使用场景的说明：**

- 亲和性：如果两个应用频繁交互，那么就有必要利用亲和性让两个应用尽可能的靠近，这样可以较少因网络通信而带来的性能损耗
- 反亲和性：当应用采用多副本部署的时候，那么就有必要利用反亲和性让各个应用实例打散分布在各个Node上，这样可以提高服务的高可用性



#### 1.4.污点和容忍

在 Kubernetes 中，节点亲和性 `NodeAffinity` 是 Pod 上定义的一种属性，能够使 `Pod` 按我们的要求调度到某个节点上，而 `Taints`(污点) 则恰恰相反，它是 `Node` 上的一个属性，可以让 Pod 不能调度到带污点的节点上，甚至会对带污点节点上已有的 Pod 进行驱逐。

当然，对应的 `Kubernetes` 可以给 `Pod` 设置 `Tolerations`(容忍) 属性来让 `Pod` 能够容忍节点上设置的污点，这样在调度时就会忽略节点上设置的污点，将 `Pod` 调度到该节点。一般时候 `Taints` 通常与 `Tolerations` 配合使用。



- 一个 node 可以有多个污点；
- 一个 pod 可以有多个容忍；
- kubernetes 执行多个污点和容忍方法类似于过滤器

如果一个 node 有多个污点，且 pod 上也有多个容忍，只要 pod 中容忍能包含 node 上设置的全部污点，就可以将 pod 调度到该 node 上。如果 pod 上设置的容忍不能够包含 node 上设置的全部污点，且 node 上剩下不能被包含的污点 effect 为 PreferNoSchedule，那么也可能会被调度到该节点。



**注意：**

当 pod 存在容忍，首先 pod 会选择没有污点的节点，然后再次选择容忍污点的节点。

- 如果 node 上带有污点 effect 为 NoSchedule，而 pod 上不带响应的容忍，kubernetes 就不会调度 pod 到这台 node 上。
- 如果 Node 上带有污点 effect 为 PreferNoShedule，这时候 Kubernetes 会努力不要调度这个 Pod 到这个 Node 上。
- 如果 Node 上带有污点 effect 为 NoExecute，这个已经在 Node 上运行的 Pod 会从 Node 上驱逐掉。没有运行在 Node 的 Pod 不能被调度到这个 Node 上。一般使用与当某个节点处于 NotReady 状态下，pod 迅速在其他正常节点启动。



### 2.KubeEdge 应用部署

#### 2.1.KubeEdge应用部署方式

**KubeEdge应用部署方式：**

- 使用 deployment  和 daemonset 控制器
- Pod调度首先选择定向调度 nodeName 或 nodeSelector
- 根据需要选择亲和性调度、污点和容忍



#### 2.2.标签操作

**1.Node添加label**

```shell
# 查看所有node 标签
# kubectl get nodes --show-labels

#添加标签
# kubectl label nodes node01 disktype=ssdnode

# 删除标签
# kubectl label nodes uat-k8s-node1 disktype-

# 修改标签
# kubectl label nodes node01 disktype=ssdnode --overwrite
```





## 二、KubeEdge应用部署



### 1.Node添加标签

```shell
# 查看所有node 标签
# kubectl get nodes --show-labels

#添加标签
# kubectl label nodes node01 disktype=ssdnode

# 删除标签
# kubectl label nodes uat-k8s-node1 disktype-

# 修改标签
# kubectl label nodes node01 disktype=ssdnode --overwrite
```



```shell
[root@k8s-master kubeedge]# kubectl get nodes
NAME         STATUS   ROLES                  AGE    VERSION
edge-1       Ready    agent,edge             2d9h   v1.23.17-kubeedge-v1.13.4
edge-2       Ready    agent,edge             2d9h   v1.23.17-kubeedge-v1.13.4
k8s-master   Ready    control-plane,master   9d     v1.23.12
k8s-node1    Ready    <none>                 9d     v1.23.12


#添加标签
# kubectl label nodes k8s-node1 node-type=node
# kubectl label nodes edge-1 node-type=edge
# kubectl label nodes edge-1 node-type=edge


[root@k8s-master kubeedge]# kubectl label nodes k8s-node1 node-type=node
node/k8s-node1 labeled

[root@k8s-master kubeedge]# kubectl get nodes k8s-node1 --show-labels
NAME        STATUS   ROLES    AGE   VERSION    LABELS
k8s-node1   Ready    <none>   9d    v1.23.12   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node1,kubernetes.io/os=linux,node-type=node


[root@k8s-master kubeedge]# kubectl describe node k8s-node1
Name:               k8s-node1
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=k8s-node1
                    kubernetes.io/os=linux
                    
                    node-type=node
                    
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    projectcalico.org/IPv4Address: 192.168.202.202/24
                    projectcalico.org/IPv4IPIPTunnelAddr: 10.244.36.64
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Fri, 15 Dec 2023 14:43:20 +0800
Taints:             <none>
Unschedulable:      false
```

```shell
#添加标签
[root@k8s-master kubeedge]# kubectl label nodes k8s-node1 node-type=node
node/k8s-node1 labeled

[root@k8s-master kubeedge]# kubectl label nodes edge-1 node-type=edge
node/edge-1 labeled

[root@k8s-master kubeedge]# kubectl label nodes edge-2 node-type=edge
node/edge-2 labeled

```

![](/images/kubeedge/app/kube-app/ap-2.png)



### 2.DaemonSet部署

#### 2.1.部署到所有节点

```shell
[root@k8s-master kubeedge]# vim nginx-set-all.yaml

apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-set-all
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      hostNetwork: true   # 使用主机网络
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

```shell
[root@k8s-master kubeedge]# kubectl apply -f nginx-set-all.yaml 
deployment.apps/nginx-set-all created


[root@k8s-master kubeedge]# kubectl get pod -owide
NAME                  READY   STATUS    RESTARTS   AGE   IP                NODE        NOMINATED NODE   READINESS GATES
nginx-set-all-79gkd   1/1     Running   0          76s   192.168.202.212   edge-2      <none>           <none>
nginx-set-all-7mprj   1/1     Running   0          76s   192.168.202.211   edge-1      <none>           <none>
nginx-set-all-nn9z4   1/1     Running   0          76s   192.168.202.202   k8s-node1   <none>           <none>
```

![](/images/kubeedge/app/kube-app/ap-3.png)

```shell
# curl 192.168.202.212
# curl 192.168.202.211
# curl 192.168.202.202


[root@k8s-master kubeedge]# curl 192.168.202.212
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



#### 2.2.部署到边缘节点

```shell
[root@k8s-master kubeedge]# vim nginx-set-edge.yaml

apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-set-all
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      hostNetwork: true   # 使用主机网络
      nodeSelector:
        node-type: edge
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

```shell
[root@k8s-master kubeedge]# kubectl apply -f nginx-set-edge.yaml 
daemonset.apps/nginx-set-all created


[root@k8s-master kubeedge]# kubectl get pod -owide
NAME                  READY   STATUS    RESTARTS   AGE   IP                NODE     NOMINATED NODE   READINESS GATES
nginx-set-all-gkn6j   1/1     Running   0          30s   192.168.202.212   edge-2   <none>           <none>
nginx-set-all-zb4vv   1/1     Running   0          30s   192.168.202.211   edge-1   <none>           <none>
```

![](/images/kubeedge/app/kube-app/ap-4.png)



### 3.Deployment部署

#### 3.1.nodeName

```shell
[root@k8s-master kubeedge]# vim nginx-nodename.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-nodename
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
      nodeName: edge-1    #调度到指定机器
      hostNetwork: true   # 使用主机网络
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```



```shell
[root@k8s-master kubeedge]# kubectl apply -f nginx-nodename.yaml 
deployment.apps/nginx-nodename created


[root@k8s-master kubeedge]# kubectl get pod -owide
NAME                              READY   STATUS    RESTARTS   AGE   IP                NODE     NOMINATED NODE   READINESS GATES
nginx-nodename-846f55748c-fpgpv   1/1     Running   0          27s   192.168.202.211   edge-1   <none>           <none>
```

![](/images/kubeedge/app/kube-app/ap-5.png)



#### 3.2.nodeSelector

根据Node label调度Pod

```yaml
      #选择标签
      nodeSelector:
        #key1: val1
        #key2: val2
        node-type: node
```



1.部署到Node节点

```shell
[root@k8s-master kubeedge]# vim nginx-node.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment-node
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      nodeSelector:
        node-type: node
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

```shell
[root@k8s-master kubeedge]# kubectl apply -f nginx-node.yaml 
deployment.apps/nginx-deployment-node created


[root@k8s-master kubeedge]# kubectl get pod -owide
NAME                                     READY   STATUS    RESTARTS   AGE    IP             NODE        NOMINATED NODE   READINESS GATES
nginx-deployment-node-7457b6d59f-7j4fh   1/1     Running   0          3m9s   10.244.36.69   k8s-node1   <none>           <none>
nginx-deployment-node-7457b6d59f-jk9m4   1/1     Running   0          3m9s   10.244.36.70   k8s-node1   <none>           <none>
nginx-deployment-node-7457b6d59f-mj7gx   1/1     Running   0          3m9s   10.244.36.71   k8s-node1   <none>           <none>
```

![](/images/kubeedge/app/kube-app/ap-6.png)



2.部署到Edge节点

```shell
[root@k8s-master kubeedge]# vim nginx-edge.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment-edge
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      nodeSelector:
        node-type: edge
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

```shell
[root@k8s-master kubeedge]# kubectl apply -f nginx-edge.yaml 
deployment.apps/nginx-deployment-edge created


[root@k8s-master kubeedge]# kubectl get pod -owide
NAME                                     READY   STATUS    RESTARTS   AGE     IP             NODE        NOMINATED NODE   READINESS GATES
nginx-deployment-edge-7b8d874698-2g4h5   1/1     Running   0          50s     172.17.0.3     edge-2      <none>           <none>
nginx-deployment-edge-7b8d874698-2xhpn   1/1     Running   0          50s     172.17.0.3     edge-1      <none>           <none>
nginx-deployment-edge-7b8d874698-w6kkb   1/1     Running   0          50s     172.17.0.4     edge-1      <none>           <none>
```

![](/images/kubeedge/app/kube-app/ap-7.png)



#### 3.3.podAntiAffinity

edge节点部署应用

```shell
[root@k8s-master kubeedge]# vim nginx-edge-pod.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-edge-pod
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      hostNetwork: true   # 使用主机网络
      nodeSelector:
        node-type: edge
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
      affinity:  #亲和性设置
        podAntiAffinity: #设置pod亲和性
          requiredDuringSchedulingIgnoredDuringExecution: # 硬限制
          - labelSelector:
              matchExpressions: # 匹配app的值在["nginx"]中的标签
              - key: app
                operator: In
                values:
                - nginx
            topologyKey: kubernetes.io/hostname
```

```shell
[root@k8s-master kubeedge]# kubectl apply -f nginx-edge-pod.yaml 
deployment.apps/nginx-edge-pod created


[root@k8s-master kubeedge]# kubectl get pod -owide
NAME                              READY   STATUS    RESTARTS   AGE   IP                NODE     NOMINATED NODE   READINESS GATES
nginx-edge-pod-5487c6ff5f-6jjwg   1/1     Running   0          57s   192.168.202.212   edge-2   <none>           <none>
nginx-edge-pod-5487c6ff5f-qftsx   1/1     Running   0          57s   192.168.202.211   edge-1   <none>           <none>
```

![](/images/kubeedge/app/kube-app/ap-8.png)

```shell
# curl 192.168.202.211


[root@k8s-master kubeedge]# curl 192.168.202.211
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



