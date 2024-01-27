* TOC
{:toc}



## 一、概述



**Kubernetes Operator**

Operator 是由 CoreOS 开发的，用来扩展 Kubernetes API，特定的应用程序控制器，它用来创建、配置和管理复杂的有状态应用，如数据库、缓存和监控系统。Operator 基于 Kubernetes 的资源和控制器概念之上构建，但同时又包含了应用程序特定的领域知识。创建Operator 的关键是CRD（自定义资源）的设计。

Kubernetes 1.7 版本以来就引入了[自定义控制器](https://links.jianshu.com/go?to=https%3A%2F%2Fkubernetes.io%2Fdocs%2Fconcepts%2Fapi-extension%2Fcustom-resources%2F)的概念，该功能可以让开发人员扩展添加新功能，更新现有的功能，并且可以自动执行一些管理任务，这些自定义的控制器就像 Kubernetes 原生的组件一样，Operator 直接使用 Kubernetes API进行开发，也就是说他们可以根据这些控制器内部编写的自定义规则来监控集群、更改 Pods/Services、对正在运行的应用进行扩缩容。



**Operator Framework**

Operator Framework 同样也是 CoreOS 开源的一个用于快速开发 Operator 的工具包，该框架包含两个主要的部分：

- Operator SDK: 无需了解复杂的 Kubernetes API 特性，即可让你根据你自己的专业知识构建一个 Operator 应用
- Operator Lifecycle Manager OLM: 帮助你安装、更新和管理跨集群的运行中的所有 Operator（以及他们的相关服务）



**Workflow**

Operator SDK 提供以下工作流来开发一个新的 Operator：

- 使用 SDK 创建一个新的 Operator 项目
- 通过添加自定义资源（CRD）定义新的资源 API
- 指定使用 SDK API 来 watch 的资源
- 定义 Operator 的协调（reconcile）逻辑
- 使用 Operator SDK 构建并生成 Operator 部署清单文件



Kubernetes Operator 这个新生事物，就成了开发和部署分布式应用的一项事实标准。时至今日，无论是 etcd、TiDB、Redis，还是 Kafka、RocketMQ、Spark、TensorFlow，几乎每一个你能叫上名字来的分布式项目，都由官方维护着各自的 Kubernetes Operator。而 Operator 官方库里，也一直维护着一个知名分布式项目的 Operator 汇总。

https://github.com/operator-framework/awesome-operators



**Helm和Operator的对比**

Operator本质上是针对特定的场景去做有状态服务，或者说针对拥有复杂应用的应用场景去简化其运维管理的工具。Helm的话，它其实是一个比较普适的工具，想法也很简单，就是把你的K8S资源模板化，方便共享，然后在不同的配置中重用。

其实Operator做的东西Helm大部分也可以做。用Operator去监控更新etcd的集群状态，也可以用定制的Chart做同样的事情。只不过你可能需要一些更复杂的处理而已，例如在etcd没有建立起来时候，你可能需要一些init Container去做配置的更新，去检查状态，然后把这个节点用对应的信息给拉起来。删除的时候，则加一些PostHook去做一些处理。所以说Helm是一个更加普适的工具。两者甚至可以结合使用，比如stable仓库里就有etcd-operator chart。

就个人理解来说，在K8S这个庞然大物之上，他们两者都诞生于简单但自然的想法，helm是为了配置分离，operator则是针对复杂应用的自动化管理。



为了能够管理和搜索Operator，Red hat、Google等公司联合社区发起了[operatorhub](https://www.operatorhub.io/)项目，可以直接访问相关的Operator仓库。

- Kubernetes操作器仓库-OperatorHub，https://www.operatorhub.io/
- Kubernetes应用安装仓库-Helm Charts，https://github.com/helm/charts
- 容器镜像仓库-DockerHub，[https://hub.docker.com](https://hub.docker.com/)



```shell
# 官网

https://operatorframework.io/
https://github.com/operator-framework
https://github.com/operator-framework/awesome-operators
https://operatorhub.io/
```



-------------------------------------------------------------------------------------------------------------



**Helm vs Operator**

Kubernetes提供声明式API对标准对象进行生命周期管理，例如Deployment，Pod，Service以及ConfigMap等。基于这种原子能力，开发人员可以自由组合各种对象，以Yaml，Json等格式文件进行定义，构建自己的业务应用。

随着应用自身的复杂度增加，依赖条件变多，部署环境的多样性，这种原始的声明式API使用方式显得低效且不够灵活；另外，在应用与声明式API之间缺少一个构建标准，也不利于应用的共享，分发，以及应用市场的发展。Helm和Operator应运而生，在声明式API之上进行了一层抽象和封装，成为解决这种困境的两个主要流派。



**Helm Chart**

Helm是基于Kubernetes的应用包管理工具，可实现应用程序封装、版本管理、依赖检查、应用分发，是对容器应用所需资源组件进行集中管理，并通过模板化和配置分离提高声明式API的开发和使用效率。Chart包则是应用的载体，实现了应用的分发和共享，支撑了应用市场的发展。



**Operator**

Operator主要包含CRD和Controller两个部分：

CRD是在Kubernetes的标准对象之上构建一层直接面向应用的扩展对象，例如Mysql Operator提供的InnoDBCluster和MySQLBackup这两个扩展对象。
Controller是指自定义控制器，以Deployment的形式在Kubernetes中运行，监听CRD的配置变化，并转换成标准对象进行实现。控制器自动化智能化的灵活实现应用生命周期管理能力以及运维能力，是Site Reliability Engineering (SRE)思想在容器集群领域的一个落地实践。



**Helm与Operator的比较**

Helm和Operator从不同角度衔接了应用和声明式API，两者之间有着明显的差异，下面从不同角度对Helm以及Operator进行了一个比较：

![](/images/kubernetes/advance/operator-1.png)



关于应用场景方面，其实不太好区分，因为这两项技术本身就致力于解决不同的问题：

- Helm本质是包管理工具，适合于安装标准的通用的应用程序
- Operator本质是一种设计模式，强项在于支持有状态应用的复杂生命周期管理，将运维过程中的知识和方法固化在控制器中，实现自动化运维
  



## 二、基础



### 1.Redis集群

#### 1.1.文档

```shell
# operator
https://github.com/operator-framework/awesome-operators


#Redis Cluster #2	ucloud/redis-cluster-operator
https://github.com/ucloud/redis-cluster-operator


# 安装步骤
1.Register the DistributedRedisCluster and RedisClusterBackup custom resource definition (CRD).
$ kubectl create -f deploy/crds/redis.kun_distributedredisclusters_crd.yaml
$ kubectl create -f deploy/crds/redis.kun_redisclusterbackups_crd.yaml


2.
// cluster-scoped
$ kubectl create -f deploy/service_account.yaml
$ kubectl create -f deploy/cluster/cluster_role.yaml
$ kubectl create -f deploy/cluster/cluster_role_binding.yaml
$ kubectl create -f deploy/cluster/operator.yaml

// namespace-scoped
$ kubectl create -f deploy/service_account.yaml
$ kubectl create -f deploy/namespace/role.yaml
$ kubectl create -f deploy/namespace/role_binding.yaml
$ kubectl create -f deploy/namespace/operator.yaml


3.
kubectl apply -f deploy/example/redis.kun_v1alpha1_distributedrediscluster_cr.yaml
```



#### 1.2.安装

```shell
# 拉取文件
[root@k8s-master redis]# git clone https://github.com/ucloud/redis-cluster-operator.git
Cloning into 'redis-cluster-operator'...
remote: Enumerating objects: 2024, done.
remote: Counting objects: 100% (51/51), done.
remote: Compressing objects: 100% (30/30), done.
remote: Total 2024 (delta 15), reused 38 (delta 11), pack-reused 1973
Receiving objects: 100% (2024/2024), 944.24 KiB | 850.00 KiB/s, done.
Resolving deltas: 100% (1288/1288), done.

[root@k8s-master redis]# ll
drwxr-xr-x 14 root root 310 Oct 15 09:51 redis-cluster-operator


[root@k8s-master redis]# cd redis-cluster-operator/
[root@k8s-master redis-cluster-operator]# ll
total 144
drwxr-xr-x  3 root root     35 Oct 15 09:51 build
drwxr-xr-x  3 root root     36 Oct 15 09:51 charts
drwxr-xr-x  3 root root     21 Oct 15 09:51 cmd
drwxr-xr-x  6 root root    108 Oct 15 09:51 deploy
drwxr-xr-x  3 root root     20 Oct 15 09:51 doc
-rw-r--r--  1 root root    993 Oct 15 09:51 Dockerfile
-rw-r--r--  1 root root   2939 Oct 15 09:51 go.mod
-rw-r--r--  1 root root 107647 Oct 15 09:51 go.sum
drwxr-xr-x  5 root root     60 Oct 15 09:51 hack
-rw-r--r--  1 root root  11333 Oct 15 09:51 LICENSE
-rw-r--r--  1 root root   1797 Oct 15 09:51 Makefile
drwxr-xr-x 12 root root    148 Oct 15 09:51 pkg
-rw-r--r--  1 root root   7871 Oct 15 09:51 README.md
drwxr-xr-x  2 root root     31 Oct 15 09:51 static
drwxr-xr-x  4 root root     35 Oct 15 09:51 test
-rw-r--r--  1 root root    149 Oct 15 09:51 tools.go
drwxr-xr-x  2 root root     24 Oct 15 09:51 version


# 安装
$ kubectl create -f deploy/crds/redis.kun_distributedredisclusters_crd.yaml
$ kubectl create -f deploy/crds/redis.kun_redisclusterbackups_crd.yaml


// cluster-scoped
$ kubectl create -f deploy/service_account.yaml
$ kubectl create -f deploy/cluster/cluster_role.yaml
$ kubectl create -f deploy/cluster/cluster_role_binding.yaml
$ kubectl create -f deploy/cluster/operator.yaml


kubectl apply -f deploy/example/redis.kun_v1alpha1_distributedrediscluster_cr.yaml


# 查询
[root@k8s-master redis-cluster-operator]# kubectl get distributedrediscluster
NAME                              MASTERSIZE   STATUS    AGE
example-distributedrediscluster   3            Healthy   29m


[root@k8s-master redis-cluster-operator]# kubectl get all -l redis.kun/name=example-distributedrediscluster
NAME                                          READY   STATUS    RESTARTS   AGE
pod/drc-example-distributedrediscluster-0-0   1/1     Running   0          30m
pod/drc-example-distributedrediscluster-0-1   1/1     Running   0          29m
pod/drc-example-distributedrediscluster-1-0   1/1     Running   0          30m
pod/drc-example-distributedrediscluster-1-1   1/1     Running   0          28m
pod/drc-example-distributedrediscluster-2-0   1/1     Running   0          30m
pod/drc-example-distributedrediscluster-2-1   1/1     Running   0          28m

NAME                                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)              AGE
service/example-distributedrediscluster     ClusterIP   10.101.65.88   <none>        6379/TCP,16379/TCP   30m
service/example-distributedrediscluster-0   ClusterIP   None           <none>        6379/TCP,16379/TCP   30m
service/example-distributedrediscluster-1   ClusterIP   None           <none>        6379/TCP,16379/TCP   30m
service/example-distributedrediscluster-2   ClusterIP   None           <none>        6379/TCP,16379/TCP   30m

NAME                                                     READY   AGE
statefulset.apps/drc-example-distributedrediscluster-0   2/2     30m
statefulset.apps/drc-example-distributedrediscluster-1   2/2     30m
statefulset.apps/drc-example-distributedrediscluster-2   2/2     30m


[root@k8s-master redis-cluster-operator]# kubectl get distributedrediscluster
NAME                              MASTERSIZE   STATUS    AGE
example-distributedrediscluster   3            Healthy   30m
```



#### 1.3.删除集群

```shell
# 清理集群
kubectl delete -f deploy/example/redis.kun_v1alpha1_distributedrediscluster_cr.yaml
kubectl delete -f deploy/cluster/operator.yaml
kubectl delete -f deploy/cluster/cluster_role_binding.yaml
kubectl delete -f deploy/cluster/cluster_role.yaml
kubectl delete -f deploy/service_account.yaml
kubectl delete -f deploy/crds/redis.kun_redisclusterbackups_crd.yaml
kubectl delete -f deploy/crds/redis.kun_distributedredisclusters_crd.yaml
```



#### 1.4.测试

```shell
# 查询
[root@k8s-master redis-cluster-operator]# kubectl get svc
NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)              AGE
example-distributedrediscluster     ClusterIP   10.101.65.88    <none>        6379/TCP,16379/TCP   32m
example-distributedrediscluster-0   ClusterIP   None            <none>        6379/TCP,16379/TCP   32m
example-distributedrediscluster-1   ClusterIP   None            <none>        6379/TCP,16379/TCP   32m
example-distributedrediscluster-2   ClusterIP   None            <none>        6379/TCP,16379/TCP   32m
kubernetes                          ClusterIP   10.96.0.1       <none>        443/TCP              58d
redis-cluster-operator-metrics      ClusterIP   10.100.104.38   <none>        8383/TCP,8686/TCP    32m

root@k8s-master redis-cluster-operator]# kubectl get pod
NAME                                      READY   STATUS    RESTARTS   AGE
drc-example-distributedrediscluster-0-0   1/1     Running   0          34m
drc-example-distributedrediscluster-0-1   1/1     Running   0          33m
drc-example-distributedrediscluster-1-0   1/1     Running   0          34m
drc-example-distributedrediscluster-1-1   1/1     Running   0          33m
drc-example-distributedrediscluster-2-0   1/1     Running   0          34m
drc-example-distributedrediscluster-2-1   1/1     Running   0          33m
redis-cluster-operator-6669898858-9b7kh   1/1     Running   0          35m


# 进入容器链接集群
[root@k8s-master redis-cluster-operator]# kubectl exec -ti drc-example-distributedrediscluster-0-0 -- sh
/data # redis-cli -h example-distributedrediscluster
example-distributedrediscluster:6379> 
example-distributedrediscluster:6379> 


# 设置值
example-distributedrediscluster:6379> set a b
(error) MOVED 15495 10.244.107.194:6379
example-distributedrediscluster:6379> quit
/data # redis-cli -h 10.244.107.194
10.244.107.194:6379> set a b
OK
10.244.107.194:6379> get a
"b"


10.244.107.194:6379> quit
/data # 
/data # redis-cli -h example-distributedrediscluster
example-distributedrediscluster:6379> get a
(error) MOVED 15495 10.244.107.194:6379


# 集群配置信息，需要持久化
[root@k8s-master redis-cluster-operator]#  kubectl exec -ti drc-example-distributedrediscluster-0-0 -- sh
/data # ls
dump.rdb        nodes.conf      redis_password
/data # cat nodes.conf 
12f521bd8cf6ab3d2740d987441f683ff85c2a8f 10.244.169.135:6379@16379 master - 0 1634269083335 8 connected 5462-10923
1df34b30f1aa2d8a06895e08b776b26037347542 10.244.107.194:6379@16379 master - 0 1634269079329 9 connected 10924-16383
80dc04b4ae99d1cfb43139936552d1fed009a303 10.244.36.106:6379@16379 master - 0 1634269082332 7 connected 0-5461
f67b02d2891aeb89d72af3e77e91a86e74589e4a 10.244.169.133:6379@16379 slave 1df34b30f1aa2d8a06895e08b776b26037347542 0 1634269078000 9 connected
2e05c610d696f67fc115c6b6c4435e5d0a52d3a8 10.244.107.253:6379@16379 myself,slave 80dc04b4ae99d1cfb43139936552d1fed009a303 0 1634269075000 2 connected
858a82acb6a1a50288ac410af6a8866481c86fc4 10.244.36.109:6379@16379 slave 12f521bd8cf6ab3d2740d987441f683ff85c2a8f 0 1634269081331 8 connected
vars currentEpoch 9 lastVoteEpoch 0
```



#### 1.5.扩缩容

```shell
# 查询
[root@k8s-master redis-cluster-operator]# kubectl get all -l redis.kun/name=example-distributedrediscluster
NAME                                          READY   STATUS    RESTARTS   AGE
pod/drc-example-distributedrediscluster-0-0   1/1     Running   0          50m
pod/drc-example-distributedrediscluster-0-1   1/1     Running   0          49m
pod/drc-example-distributedrediscluster-1-0   1/1     Running   0          50m
pod/drc-example-distributedrediscluster-1-1   1/1     Running   0          49m
pod/drc-example-distributedrediscluster-2-0   1/1     Running   0          50m
pod/drc-example-distributedrediscluster-2-1   1/1     Running   0          49m

NAME                                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)              AGE
service/example-distributedrediscluster     ClusterIP   10.101.65.88   <none>        6379/TCP,16379/TCP   50m
service/example-distributedrediscluster-0   ClusterIP   None           <none>        6379/TCP,16379/TCP   50m
service/example-distributedrediscluster-1   ClusterIP   None           <none>        6379/TCP,16379/TCP   50m
service/example-distributedrediscluster-2   ClusterIP   None           <none>        6379/TCP,16379/TCP   50m

NAME                                                     READY   AGE
statefulset.apps/drc-example-distributedrediscluster-0   2/2     50m
statefulset.apps/drc-example-distributedrediscluster-1   2/2     50m
statefulset.apps/drc-example-distributedrediscluster-2   2/2     50m

[root@k8s-master redis-cluster-operator]# kubectl get DistributedRedisCluster
NAME                              MASTERSIZE   STATUS    AGE
example-distributedrediscluster   3            Healthy   51m


# 修改副本数 masterSize: 4
[root@k8s-master redis-cluster-operator]# vim deploy/example/redis.kun_v1alpha1_distributedrediscluster_cr.yaml
apiVersion: redis.kun/v1alpha1
kind: DistributedRedisCluster
metadata:
  annotations:
    # if your operator run as cluster-scoped, add this annotations
    redis.kun/scope: cluster-scoped
  name: example-distributedrediscluster
spec:
  # Add fields here
  masterSize: 3
  clusterReplicas: 1
  image: redis:5.0.4-alpine


[root@k8s-master redis-cluster-operator]# kubectl get DistributedRedisCluster
NAME                              MASTERSIZE   STATUS    AGE
example-distributedrediscluster   3            Healthy   55m
# 扩容到
[root@k8s-master redis-cluster-operator]# kubectl edit DistributedRedisCluster example-distributedrediscluster
distributedrediscluster.redis.kun/example-distributedrediscluster edited


# 查看，生成8个pod
[root@k8s-master redis-cluster-operator]# kubectl get all -l redis.kun/name=example-distributedrediscluster
NAME                                          READY   STATUS    RESTARTS   AGE
pod/drc-example-distributedrediscluster-0-0   1/1     Running   0          57m
pod/drc-example-distributedrediscluster-0-1   1/1     Running   0          56m
pod/drc-example-distributedrediscluster-1-0   1/1     Running   0          57m
pod/drc-example-distributedrediscluster-1-1   1/1     Running   0          56m
pod/drc-example-distributedrediscluster-2-0   1/1     Running   0          57m
pod/drc-example-distributedrediscluster-2-1   1/1     Running   0          56m
pod/drc-example-distributedrediscluster-3-0   1/1     Running   0          80s
pod/drc-example-distributedrediscluster-3-1   1/1     Running   0          41s

NAME                                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)              AGE
service/example-distributedrediscluster     ClusterIP   10.101.65.88   <none>        6379/TCP,16379/TCP   57m
service/example-distributedrediscluster-0   ClusterIP   None           <none>        6379/TCP,16379/TCP   57m
service/example-distributedrediscluster-1   ClusterIP   None           <none>        6379/TCP,16379/TCP   57m
service/example-distributedrediscluster-2   ClusterIP   None           <none>        6379/TCP,16379/TCP   57m
service/example-distributedrediscluster-3   ClusterIP   None           <none>        6379/TCP,16379/TCP   80s

NAME                                                     READY   AGE
statefulset.apps/drc-example-distributedrediscluster-0   2/2     57m
statefulset.apps/drc-example-distributedrediscluster-1   2/2     57m
statefulset.apps/drc-example-distributedrediscluster-2   2/2     57m
statefulset.apps/drc-example-distributedrediscluster-3   2/2     80s


# 缩容 masterSize: 3
[root@k8s-master redis-cluster-operator]# kubectl edit DistributedRedisCluster example-distributedrediscluster
distributedrediscluster.redis.kun/example-distributedrediscluster edited


# 查看，生成6个pod
[root@k8s-master redis-cluster-operator]# kubectl get all -l redis.kun/name=example-distributedrediscluster
NAME                                          READY   STATUS    RESTARTS   AGE
pod/drc-example-distributedrediscluster-0-0   1/1     Running   0          74m
pod/drc-example-distributedrediscluster-0-1   1/1     Running   0          73m
pod/drc-example-distributedrediscluster-1-0   1/1     Running   0          74m
pod/drc-example-distributedrediscluster-1-1   1/1     Running   0          73m
pod/drc-example-distributedrediscluster-2-0   1/1     Running   0          74m
pod/drc-example-distributedrediscluster-2-1   1/1     Running   0          73m

NAME                                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)              AGE
service/example-distributedrediscluster     ClusterIP   10.101.65.88   <none>        6379/TCP,16379/TCP   74m
service/example-distributedrediscluster-0   ClusterIP   None           <none>        6379/TCP,16379/TCP   74m
service/example-distributedrediscluster-1   ClusterIP   None           <none>        6379/TCP,16379/TCP   74m
service/example-distributedrediscluster-2   ClusterIP   None           <none>        6379/TCP,16379/TCP   74m

NAME                                                     READY   AGE
statefulset.apps/drc-example-distributedrediscluster-0   2/2     74m
statefulset.apps/drc-example-distributedrediscluster-1   2/2     74m
statefulset.apps/drc-example-distributedrediscluster-2   2/2     74m
```



### 2.RabbitMQ集群

#### 2.1.文档

```shell
# operator
https://github.com/operator-framework/awesome-operators

#RabbitMQ #1 (Official)	  rabbitmq/cluster-operator
https://github.com/rabbitmq/cluster-operator


# 安装步骤
kubectl apply -f https://github.com/rabbitmq/cluster-operator/releases/latest/download/cluster-operator.yml

kubectl apply -f https://raw.githubusercontent.com/rabbitmq/cluster-operator/main/docs/examples/hello-world/rabbitmq.yaml
```



#### 2.2.安装

```shell
# 拉取文件
[root@k8s-master rabbitmq]# git clone https://github.com/rabbitmq/cluster-operator.git
Cloning into 'cluster-operator'...
remote: Enumerating objects: 11681, done.
remote: Counting objects: 100% (2068/2068), done.
remote: Compressing objects: 100% (1060/1060), done.
remote: Total 11681 (delta 1267), reused 1604 (delta 946), pack-reused 9613
Receiving objects: 100% (11681/11681), 11.95 MiB | 1.63 MiB/s, done.
Resolving deltas: 100% (7569/7569), done.

[root@k8s-master rabbitmq]# ll
total 4
drwxr-xr-x 14 root root 4096 Oct 15 16:04 cluster-operator

[root@k8s-master rabbitmq]# cd cluster-operator/

[root@k8s-master cluster-operator]# ll
total 252
drwxr-xr-x  3 root root     21 Oct 15 16:04 api
drwxr-xr-x  2 root root     59 Oct 15 16:04 bin
-rw-r--r--  1 root root   3368 Oct 15 16:04 CODE_OF_CONDUCT.md
drwxr-xr-x 10 root root    126 Oct 15 16:04 config
-rw-r--r--  1 root root   5471 Oct 15 16:04 CONTRIBUTING.md
drwxr-xr-x  2 root root   4096 Oct 15 16:04 controllers
-rw-r--r--  1 root root   1060 Oct 15 16:04 Dockerfile
drwxr-xr-x  7 root root     77 Oct 15 16:04 docs
-rw-r--r--  1 root root   9696 Oct 15 16:04 go.mod
-rw-r--r--  1 root root 169157 Oct 15 16:04 go.sum
drwxr-xr-x  2 root root    185 Oct 15 16:04 hack
drwxr-xr-x  6 root root     67 Oct 15 16:04 internal
-rw-r--r--  1 root root  17096 Oct 15 16:04 LICENSE.txt
-rw-r--r--  1 root root   5061 Oct 15 16:04 main.go
-rw-r--r--  1 root root   9325 Oct 15 16:04 Makefile
drwxr-xr-x  4 root root     77 Oct 15 16:04 observability
-rw-r--r--  1 root root    360 Oct 15 16:04 PROJECT
-rw-r--r--  1 root root   2946 Oct 15 16:04 README.md
drwxr-xr-x  2 root root     78 Oct 15 16:04 system_tests
drwxr-xr-x  2 root root     22 Oct 15 16:04 tools
-rw-r--r--  1 root root   2701 Oct 15 16:04 version_guidelines.md


# 安装
# 安装operator
[root@k8s-master cluster-operator]# kubectl apply -f https://github.com/rabbitmq/cluster-operator/releases/latest/download/cluster-operator.yml
namespace/rabbitmq-system created
customresourcedefinition.apiextensions.k8s.io/rabbitmqclusters.rabbitmq.com created
serviceaccount/rabbitmq-cluster-operator created
role.rbac.authorization.k8s.io/rabbitmq-cluster-leader-election-role created
clusterrole.rbac.authorization.k8s.io/rabbitmq-cluster-operator-role created
rolebinding.rbac.authorization.k8s.io/rabbitmq-cluster-leader-election-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/rabbitmq-cluster-operator-rolebinding created
deployment.apps/rabbitmq-cluster-operator created


# 查看operator部署
[root@k8s-master cluster-operator]# kubectl get all -n rabbitmq-system
NAME                                             READY   STATUS    RESTARTS   AGE
pod/rabbitmq-cluster-operator-7cbf865f89-gqwgh   1/1     Running   0          2m25s

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/rabbitmq-cluster-operator   1/1     1            1           2m26s

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/rabbitmq-cluster-operator-7cbf865f89   1         1         1       2m25s


# 部署rabbtimq集群
[root@k8s-master cluster-operator]# kubectl apply -f https://raw.githubusercontent.com/rabbitmq/cluster-operator/main/docs/examples/hello-world/rabbitmq.yaml
rabbitmqcluster.rabbitmq.com/hello-world created


[root@k8s-master cluster-operator]# kubectl get all
NAME                                          READY   STATUS    RESTARTS   AGE
pod/hello-world-server-0                      0/1     Pending   0          11m

NAME                                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                          AGE
service/hello-world                         ClusterIP   10.109.130.168   <none>        5672/TCP,15672/TCP,15692/TCP     11m
service/hello-world-nodes                   ClusterIP   None             <none>        4369/TCP,25672/TCP               11m

NAME                                                     READY   AGE
statefulset.apps/hello-world-server                      0/1     11m

NAME                                       ALLREPLICASREADY   RECONCILESUCCESS   AGE
rabbitmqcluster.rabbitmq.com/hello-world   False              Unknown            11m
```



