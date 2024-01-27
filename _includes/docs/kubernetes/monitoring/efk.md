* TOC
{:toc}



## 一、概述



### 1.统一日志管理的整体方案

Kubernetes本身并没有为日志数据提供原生的存储解决方案，但可以将许多现有的日志记录解决方案集成到Kubernetes集群中。在Kubernetes中，有三个层次的日志：

- 基础日志
- Node级别的日志
- 群集级别的日志架构



#### 1.1.基础日志

kubernetes基础日志即将日志数据输出到标准输出流，可以使用kubectl logs命令获取容器日志信息。

如果Pod中有多个容器，可以通过将容器名称附加到命令来指定要访问哪个容器的日志。

例如，在Kubernetes集群中的devops命名空间下有一个名称为nexus3-f5b7fc55c-hq5v7的Pod，就可以通过如下的命令获取日志：

```
$ kubectl logs nexus3-f5b7fc55c-hq5v7 --namespace=devops
```



#### 1.2.Node级别的日志

容器化应用写入到stdout和stderr的所有内容都是由容器引擎处理和重定向的。例如，docker容器引擎会将这两个流重定向到日志记录驱动，在Kubernetes中该日志驱动被配置为以json格式写入文件。docker json日志记录驱动将每一行视为单独的消息。当使用docker日志记录驱动时，并不支持多行消息，因此需要在日志代理级别或更高级别上处理多行消息。
默认情况下，如果容器重新启动，kubectl将会保留一个已终止的容器及其日志。如果从Node中驱逐Pod，那么Pod中所有相应的容器也会连同它们的日志一起被驱逐。Node级别的日志中的一个重要考虑是实现日志旋转，这样日志不会消耗Node上的所有可用存储。Kubernetes目前不负责旋转日志，部署工具应该建立一个解决方案来解决这个问题。

![](/images/kubernetes/monitoring/efk-1.png)



#### 1.3.集群级别的日志架构

Kubernetes本身没有为群集级别日志记录提供原生解决方案，但有几种常见的方法可以采用：

- 使用运行在每个Node上的Node级别的日志记录代理
- 在应用Pod中包含一个用于日志记录的sidecar
- 将日志直接从应用内推到后端

![](/images/kubernetes/monitoring/efk-2.png)



### 2.kubernetes日志收集方案

#### 2.1.在Node上部署logging agent

这种架构的核心点就在logging-agent，通常情况下它会以DS的方式运行在节点上，然后将宿主机的容器日志目录挂载进去，然后由logging-agent将日志收集转发出去。常用的方案有EFK，即用fluentd作为上面的logging-agent，然后将日志转发到远端的ElasticSearch中，然后由kibana进行展示。

这种方案的优点是只需要在节点部署一个logging-agent，而且不会对应用和Pod有任何侵入性，但是其缺点就是必须要应用的日志输出必须输出到stdout和stderr中去。

![](/images/kubernetes/monitoring/efk-2.png)



#### 2.2.sidecar收集日志

由于第一种方式无法收集一些没有标准输出到stdout和stderr中，所以就有了第二种方式对这种特殊情况进行处理：当容器的日志只能输出到某些文件的时候，就可以通过一个sidecar容器把这些日志重新输出到sidecar的stdout和stderr中去，这样就可以继续使用第一种方案进行日志再处理。

由于sidecar跟主容器是共享volume的，所以这里的sidecar方案其实并不会占用太多的资源。但是这种办法实际上是会存在两份文件，一份是系统自己写的文件，第二份是通过sidecar输出到stdout和stderr中所对应的JSON文件，多以对磁盘来说是一个大的开销。

![](/images/kubernetes/monitoring/efk-3.png)



#### 2.3.以sidecar运行logging-agent

这种方案就是直接在sidecar中运行logging-agent，直接将日志转发到后端存储，也就是相当于在方案一中将logging-agent放到一个Pod中了。

在这个方案中，应用可以直接把日志输出到一个文件中，我们的logging-agent可以使用fluentd收集日志转发到后端ElasticSearch中，不过这里的输入源变成了日志文件。

这样虽然更加灵活，但是在 sidecar 容器中运行日志采集代理程序会导致大量资源消耗，因为你有多少个要采集的 Pod，就需要运行多少个采集代理程序，另外还无法使用 kubectl logs 命令来访问这些日志，因为它们不受 kubelet 控制。

![](/images/kubernetes/monitoring/efk-4.png)



#### 2.4.日志采集方式

**K8S日志采集方式：**

- 原生方式：使用 `kubectl logs` 直接在查看本地保留的日志，或者通过docker engine的 `log driver` 把日志重定向到文件、syslog、fluentd等系统中。
- DaemonSet方式：在K8S的每个node上部署日志agent，由agent采集所有容器的日志到服务端
- Sidecar方式：一个POD中运行一个sidecar的日志agent容器，用于采集该POD主容器产生的日志

每种采集方式都有一定的优劣势，这里我们进行简单的对比：

|              | 原生方式                                          | DaemonSet方式                    | Sidecar方式                                      |
| ------------ | ------------------------------------------------- | -------------------------------- | ------------------------------------------------ |
| 采集日志类型 | 标准输出                                          | 标准输出+部分文件                | 文件                                             |
| 部署运维     | 低，原生支持                                      | 一般，需维护DaemonSet            | 较高，每个需要采集日志的POD都需要部署sidecar容器 |
| 日志分类存储 | 无法实现                                          | 一般，可通过容器/路径等映射      | 每个POD可单独配置，灵活性高                      |
| 多租户隔离   | 弱                                                | 一般，只能通过配置间隔离         | 强，通过容器进行隔离，可单独分配资源             |
| 支持集群规模 | 本地存储无限制，若使用syslog、fluentd会有单点限制 | 中小型规模，业务数最多支持百级别 | 无限制                                           |
| 资源占用     | 低，docker engine提供                             | 较低，每个节点运行一个容器       | 较高，每个POD运行一个容器                        |
| 查询便捷性   | 低                                                | 较高，可进行自定义的查询、统计   | 高，可根据业务特点进行定制                       |
| 可定制性     | 低                                                | 低                               | 高，每个POD单独配置                              |
| 适用场景     | 测试、POC等非生产场景                             | 功能单一型的集群                 | 大型、混合型、PAAS型集群                         |

从上述表格中可以看出：

- 原生方式相对功能太弱，一般不建议在生产系统中使用，否则问题调查、数据统计等工作很难完成
- DaemonSet方式在每个节点只允许一个日志agent，相对资源占用要小很多，但扩展性、租户隔离性受限，比较适用于功能单一或业务不是很多的集群
- Sidecar方式为每个POD单独部署日志agent，相对资源占用较多，但灵活性以及多租户隔离性较强，建议大型的K8S集群或作为PAAS平台为多个业务方服务的集群使用该方式



### 3.官方推荐方案-EFK

采用使用Node日志记录代理的方面进行Kubernetes的统一日志管理，相关的工具采用：

- 日志记录代理（logging-agent）：日志记录代理用于从容器中获取日志信息，使用**Fluentd**
- 日志记录后台（Logging-Backend）：日志记录后台用于处理日志记录代理推送过来的日志，使用**Elasticsearch**
- 日志记录展示：日志记录展示用于向用户显示统一的日志信息，使用**Kibana**

在Kubernetes中通过了Elasticsearch 附加组件，此组件包括Elasticsearch、Fluentd和Kibana。Elasticsearch是一种负责存储日志并允许查询的搜索引擎。Fluentd从Kubernetes中获取日志消息，并发送到Elasticsearch；而Kibana是一个图形界面，用于查看和查询存储在Elasticsearch中的日志。

```shell
# 官方参考
https://github.com/kubernetes/kubernetes/tree/master/cluster/addons
https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/fluentd-elasticsearch
https://codeload.github.com/kubernetes/kubernetes/tar.gz/refs/tags/v1.20.6


# ES
https://www.elastic.co/cn/

# fluentd
https://www.fluentd.org/
https://github.com/fluent/fluentd
```



### 4.EFK组件介绍

Kubernetes中比较流行的日志收集解决方案是Elasticsearch、Fluentd和Kibana（EFK）技术栈，也是官方推荐的一种方案。

- **Elasticsearch**

Elasticsearch是一个实时的，分布式的，可扩展的搜索引擎，它允许进行全文本和结构化搜索以及对日志进行分析。它通常用于索引和搜索大量日志数据，也可以用于搜索许多不同种类的文档。

- **Kibana**

Elasticsearch通常与Kibana一起部署，kibana是Elasticsearch 的功能强大的数据可视化的dashboard（仪表板）。Kibana允许你通过Web界面浏览Elasticsearch日志数据，也可自定义查询条件快速检索出elasticccsearch中的日志数据。

- **Fluentd**

Fluentd是一个流行的开源数据收集器，我们将在Kubernetes 集群节点上安装 Fluentd，通过获取容器日志文件、过滤和转换日志数据，然后将数据传递到 Elasticsearch 集群，在该集群中对其进行索引和存储。



### 5.限制日志

```shell
# 限制docker日志大小
https://docs.docker.com/config/containers/logging/json-file/
https://docs.docker.com/config/containers/logging/configure/


# 日志路径
[root@k8s-node1 ~]# cd /var/log/containers/

# 新版本路径
[root@k8s-master containers]# cd /var/log/pods/
```






## 二、基础



### 1.EFK

```shell
# 官方地址

# kubernetes
https://github.com/kubernetes/kubernetes/tree/master/cluster/addons

# fluentd-elasticsearch
https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/fluentd-elasticsearch

# 下载源码（与k8s版本对应）
https://codeload.github.com/kubernetes/kubernetes/tar.gz/refs/tags/v1.20.6


https://kubernetes.io/docs/concepts/cluster-administration/logging/
```



### 2.EFK部署方案

Elasticsearch、Kibana可以部署在K8S内部或外部，部署在K8S内部需要提供分布式是存储，生产环境建议部署在K8S外部。

**两种部署方案：**

- **方案一：**Fluentd以DaemonSet方式部署，Elasticsearch、Kibana部署在K8S外部
- **方案一：**Fluentd以DaemonSet方式部署，Elasticsearch、Kibana部署在K8S内部



### 3.下载源码

**注意：EFK版本要与Kubernetes版本一致**

```shell
# fluentd-elasticsearch
https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/fluentd-elasticsearch


# 下载源码（与k8s版本对应）
https://codeload.github.com/kubernetes/kubernetes/tar.gz/refs/tags/v1.20.6
```

```shell
[root@k8s-master efk]# ll
total 48
drwxr-xr-x 3 root root   169 Nov 19 16:41 es-image
-rw-r--r-- 1 root root   580 Nov 19 16:41 es-service.yaml
-rw-r--r-- 1 root root  3186 Nov 19 16:41 es-statefulset.yaml
-rw-r--r-- 1 root root 16125 Nov 19 16:41 fluentd-es-configmap.yaml
-rw-r--r-- 1 root root  2581 Nov 19 16:41 fluentd-es-ds.yaml
drwxr-xr-x 2 root root   112 Nov 19 16:41 fluentd-es-image
-rw-r--r-- 1 root root  1519 Nov 19 16:41 kibana-deployment.yaml
-rw-r--r-- 1 root root   354 Nov 19 16:41 kibana-service.yaml
-rw-r--r-- 1 root root   189 Nov 19 16:41 OWNERS
drwxr-xr-x 2 root root    33 Nov 19 16:41 podsecuritypolicies
-rw-r--r-- 1 root root  4550 Nov 19 16:41 README.md


[root@k8s-master efk]# ll
total 48
# es
-rw-r--r-- 1 root root   580 Nov 19 16:41 es-service.yaml
-rw-r--r-- 1 root root  3186 Nov 19 16:41 es-statefulset.yaml

# fluentd
-rw-r--r-- 1 root root 16125 Nov 19 16:41 fluentd-es-configmap.yaml
-rw-r--r-- 1 root root  2581 Nov 19 16:41 fluentd-es-ds.yaml

# kibana
-rw-r--r-- 1 root root  1519 Nov 19 16:41 kibana-deployment.yaml
-rw-r--r-- 1 root root   354 Nov 19 16:41 kibana-service.yaml
```



### 4.Docker部署Elasticsearch

```shell
***************************************************************
# elasticsearch
 
 
# 下载镜像 查看镜像
docker pull elasticsearch:7.4.2


# 运行 elasticsearch
docker run -d --name elasticsearch --network host --restart=always \
-e "discovery.type=single-node" elasticsearch:7.4.2
 

# 检测 elasticsearch 是否启动成功
curl 127.0.0.1:9200


***************************************************************
# kibana
 
 
# 下载镜像 查看镜像
docker pull kibana:7.4.2
 

# 运行 Kibana
docker run -d --name kibana --network host --restart=always kibana:7.4.2


# 修改配置文件
docker exec -it kibana /bin/bash


/usr/share/kibana/config/kibana.yml

server.name: kibana
server.host: "0"
elasticsearch.hosts: [ "http://172.51.216.98:9200" ]
xpack.monitoring.ui.container.elasticsearch.enabled: true


# 重启docker



# Kibana地址：
http://172.51.216.98:5601
# ES地址：
http://172.51.216.98:9200
```





## 三、实践



### 1.EFK安装部署

#### 1.1.安装Elasticsearch

```shell
# 直接安装，不修改配置，不配置存储


kubectl apply -f es-statefulset.yaml
kubectl apply -f es-service.yaml
```

```shell
# 查看

[root@k8s-master efk]# kubectl get all -n kube-system
NAME                                           READY   STATUS    RESTARTS   AGE
......
pod/elasticsearch-logging-0                    1/1     Running   0          2d16h
pod/elasticsearch-logging-1                    1/1     Running   0          2d16h
......

NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
service/elasticsearch-logging   ClusterIP   None             <none>        9200/TCP,9300/TCP        2d17h

NAME                                     READY   AGE
statefulset.apps/elasticsearch-logging   2/2     2d16h
```



#### 1.2.安装Fluentd

```shell
# 直接安装，不配置存储


kubectl apply -f fluentd-es-configmap.yaml
kubectl apply -f fluentd-es-ds.yaml 
```

```shell
# 查看


[root@k8s-master efk]# kubectl get all -n kube-system
NAME                                           READY   STATUS    RESTARTS   AGE
pod/elasticsearch-logging-0                    1/1     Running   0          2d16h
pod/elasticsearch-logging-1                    1/1     Running   0          2d16h
pod/fluentd-es-v3.1.0-8j9g8                    1/1     Running   6          2d17h
pod/fluentd-es-v3.1.0-dgkcc                    1/1     Running   8          2d17h
pod/fluentd-es-v3.1.0-ft7pn                    1/1     Running   7          2d17h

NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
service/elasticsearch-logging   ClusterIP   None             <none>        9200/TCP,9300/TCP        2d17h

NAME                               DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/fluentd-es-v3.1.0   3         3         3       3            3           <none>                   2d17h

NAME                                     READY   AGE
statefulset.apps/elasticsearch-logging   2/2     2d16h


# 每个Node一个DaemonSet
[root@k8s-master efk]# kubectl get pod -n kube-system -o wide
NAME                                       READY   STATUS    RESTARTS   AGE     IP               NODE         NOMINATED NODE   READINESS GATES
elasticsearch-logging-0                    1/1     Running   0          2d17h   10.244.169.180   k8s-node2    <none>           <none>
elasticsearch-logging-1                    1/1     Running   0          2d17h   10.244.169.166   k8s-node2    <none>           <none>
etcd-k8s-master                            1/1     Running   10         96d     172.51.216.81    k8s-master   <none>           <none>
fluentd-es-v3.1.0-8j9g8                    1/1     Running   6          2d17h   10.244.36.97     k8s-node1    <none>           <none>
fluentd-es-v3.1.0-dgkcc                    1/1     Running   8          2d17h   10.244.107.228   k8s-node3    <none>           <none>
fluentd-es-v3.1.0-ft7pn                    1/1     Running   7          2d17h   10.244.169.171   k8s-node2    <none>           <none>
```



#### 1.3.安装Kibana

```shell
# 修改配置安装

kubectl apply -f kibana-deployment.yaml 
kubectl apply -f kibana-service.yaml
```

```yaml
# kibana-deployment.yaml 
# 修改 replicas: 3 
# NodePort映射端口需要注释 
#- name: SERVER_BASEPATH
#  value: /api/v1/namespaces/kube-system/services/kibana-logging/proxy


[root@k8s-master efk]# vim kibana-deployment.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: kibana-logging
  namespace: kube-system
  labels:
    k8s-app: kibana-logging
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  replicas: 3
  selector:
    matchLabels:
      k8s-app: kibana-logging
  template:
    metadata:
      labels:
        k8s-app: kibana-logging
    spec:
      securityContext:
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: kibana-logging
          image: docker.elastic.co/kibana/kibana-oss:7.4.2
          resources:
            # need more cpu upon initialization, therefore burstable class
            limits:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kibana-logging
  namespace: kube-system
  labels:
    k8s-app: kibana-logging
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  replicas: 3
  selector:
    matchLabels:
      k8s-app: kibana-logging
  template:
    metadata:
      labels:
        k8s-app: kibana-logging
    spec:
      securityContext:
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: kibana-logging
          image: docker.elastic.co/kibana/kibana-oss:7.4.2
          resources:
            # need more cpu upon initialization, therefore burstable class
            limits:
              cpu: 1000m
            requests:
              cpu: 100m
          env:
            - name: ELASTICSEARCH_HOSTS
              value: http://elasticsearch-logging:9200
            - name: SERVER_NAME
              value: kibana-logging
            #- name: SERVER_BASEPATH
            #  value: /api/v1/namespaces/kube-system/services/kibana-logging/proxy
            - name: SERVER_REWRITEBASEPATH
              value: "false"
          ports:
            - containerPort: 5601
              name: ui
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /api/status
              port: ui
            initialDelaySeconds: 5
            timeoutSeconds: 10
          readinessProbe:
            httpGet:
              path: /api/status
              port: ui
            initialDelaySeconds: 5
            timeoutSeconds: 10
```

```yaml
# kibana-service.yaml
# 修改 type: NodePort
# 修改nodePort: 30601 


[root@k8s-master efk]# vim kibana-service.yaml

apiVersion: v1
kind: Service
metadata:
  name: kibana-logging
  namespace: kube-system
  labels:
    k8s-app: kibana-logging
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/name: "Kibana"
spec:
  type: NodePort
  ports:
  - port: 5601
    protocol: TCP
    targetPort: ui
    nodePort: 30601
  selector:
    k8s-app: kibana-logging
```

```shell
# 查看

[root@k8s-master efk]# kubectl get all -n kube-system
NAME                                           READY   STATUS    RESTARTS   AGE
pod/elasticsearch-logging-0                    1/1     Running   0          2d16h
pod/elasticsearch-logging-1                    1/1     Running   0          2d16h
pod/fluentd-es-v3.1.0-8j9g8                    1/1     Running   6          2d17h
pod/fluentd-es-v3.1.0-dgkcc                    1/1     Running   8          2d17h
pod/fluentd-es-v3.1.0-ft7pn                    1/1     Running   7          2d17h
pod/kibana-logging-7b5d6867cd-nvdk8            1/1     Running   0          14m
pod/kibana-logging-7b5d6867cd-vzs4h            1/1     Running   0          14m
pod/kibana-logging-7b5d6867cd-z7nqg            1/1     Running   0          76m

NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
service/elasticsearch-logging   ClusterIP   None             <none>        9200/TCP,9300/TCP        2d17h
service/kibana-logging          NodePort    10.103.188.206   <none>        5601:30601/TCP           2d17h

NAME                               DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/fluentd-es-v3.1.0   3         3         3       3            3           <none>                   2d17h

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/kibana-logging            3/3     3            3           2d17h

NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/kibana-logging-7b5d6867cd            3         3         3       76m

NAME                                     READY   AGE
statefulset.apps/elasticsearch-logging   2/2     2d16h
```



#### 1.4.测试

```shell
# 查看


[root@k8s-master efk]# kubectl get all -n kube-system
NAME                                           READY   STATUS    RESTARTS   AGE
pod/elasticsearch-logging-0                    1/1     Running   0          2d17h
pod/elasticsearch-logging-1                    1/1     Running   0          2d17h
pod/etcd-k8s-master                            1/1     Running   10         96d
pod/fluentd-es-v3.1.0-8j9g8                    1/1     Running   6          2d17h
pod/fluentd-es-v3.1.0-dgkcc                    1/1     Running   8          2d17h
pod/fluentd-es-v3.1.0-ft7pn                    1/1     Running   7          2d17h
pod/kibana-logging-7b5d6867cd-m7z5t            1/1     Running   0          6m41s
pod/kibana-logging-7b5d6867cd-mm5gl            1/1     Running   0          6m41s
pod/kibana-logging-7b5d6867cd-w92sf            1/1     Running   0          6m41s


NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
service/elasticsearch-logging   ClusterIP   None             <none>        9200/TCP,9300/TCP        2d17h
service/kibana-logging          NodePort    10.103.188.206   <none>        5601:30601/TCP           2d17h
service/kube-dns                ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   96d
service/metrics-server          ClusterIP   10.106.104.139   <none>        443/TCP                  31d


NAME                               DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/fluentd-es-v3.1.0   3         3         3       3            3           <none>                   2d17h


NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/kibana-logging            3/3     3            3           6m41s


NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/kibana-logging-7b5d6867cd            3         3         3       6m41s


NAME                                     READY   AGE
statefulset.apps/elasticsearch-logging   2/2     2d17h
```

```shell
# Kibana地址

http://172.51.216.81:30601/
```

![](/images/kubernetes/monitoring/efk-5.png)

![](/images/kubernetes/monitoring/efk-6.png)



![](/images/kubernetes/monitoring/efk-7.png)



### 2.K8S存储方式部署

**把安装的ES删除，重新安装，副本改成3个，配置ceph的存储方式。**

ES部署方式是StatefulSet，选择有状态服务的存储方式，块存储（RBD）



#### 2.1.修改配置

```yaml
# es-statefulset.yaml

# 1.修改：replicas: 3
# 2.注释：      
#volumes:
#  - name: elasticsearch-logging
#    emptyDir: {}
# 3.增加配置
  volumeClaimTemplates:
  - metadata:
      name: elasticsearch-logging
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "rook-ceph-block"
      resources:
        requests:
          storage: 4Gi
          
          


# ES完整配置
[root@k8s-master efk]# vim es-statefulset.yaml
......
---
# Elasticsearch deployment itself
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: elasticsearch-logging
  namespace: kube-system
  labels:
    k8s-app: elasticsearch-logging
    version: v7.4.3
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  serviceName: elasticsearch-logging
  replicas: 2
  selector:
    matchLabels:
      k8s-app: elasticsearch-logging
      version: v7.4.3
  template:
    metadata:
      labels:
        k8s-app: elasticsearch-logging
        version: v7.4.3
    spec:
      serviceAccountName: elasticsearch-logging
      containers:
        - image: quay.io/fluentd_elasticsearch/elasticsearch:v7.4.3
          name: elasticsearch-logging
          imagePullPolicy: Always
          resources:
            # need more cpu upon initialization, therefore burstable class
            limits:
              cpu: 1000m
              memory: 3Gi
            requests:
              cpu: 100m
              memory: 3Gi
          ports:
            - containerPort: 9200
              name: db
              protocol: TCP
            - containerPort: 9300
              name: transport
              protocol: TCP
          livenessProbe:
            tcpSocket:
              port: transport
            initialDelaySeconds: 5
            timeoutSeconds: 10
          readinessProbe:
            tcpSocket:
              port: transport
            initialDelaySeconds: 5
            timeoutSeconds: 10
          volumeMounts:
            - name: elasticsearch-logging
              mountPath: /data
          env:
            - name: "NAMESPACE"
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
            - name: "MINIMUM_MASTER_NODES"
              value: "1"
      #volumes:
      #  - name: elasticsearch-logging
      #    emptyDir: {}
      # Elasticsearch requires vm.max_map_count to be at least 262144.
      # If your OS already sets up this number to a higher value, feel free
      # to remove this init container.
      initContainers:
        - image: alpine:3.6
          command: ["/sbin/sysctl", "-w", "vm.max_map_count=262144"]
          name: elasticsearch-logging-init
          securityContext:
            privileged: true
  volumeClaimTemplates:
  - metadata:
      name: elasticsearch-logging
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "rook-ceph-block"
      resources:
        requests:
          storage: 4Gi
```



#### 2.2.重新部署

```shell
# 删除es、kibana

[root@k8s-master efk]# kubectl delete -f kibana-deployment.yaml 

[root@k8s-master efk]# kubectl delete es-statefulset.yaml 



# 创建ES
[root@k8s-master efk]# kubectl apply -f es-statefulset.yaml 


# ES安装成功后再安装Kibana
[root@k8s-master efk]# kubectl apply -f kibana-deployment.yaml 



# 查看
[root@k8s-master test]# kubectl get pvc -n kube-system
NAME                                            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
elasticsearch-logging-elasticsearch-logging-0   Bound    pvc-ef784e9d-39d8-45d5-b5b3-ef52b6a18b70   4Gi        RWO            rook-ceph-block   12m
elasticsearch-logging-elasticsearch-logging-1   Bound    pvc-4c55fa22-a7db-4693-af15-23fa5ee9b328   4Gi        RWO            rook-ceph-block   11m
elasticsearch-logging-elasticsearch-logging-2   Bound    pvc-e26a2ccf-dd36-4ab3-acfd-332fe7b3c4a9   4Gi        RWO            rook-ceph-block   10m


[root@k8s-master test]# kubectl get pv -n kube-system
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                                       STORAGECLASS      REASON   AGE
pvc-4c55fa22-a7db-4693-af15-23fa5ee9b328   4Gi        RWO            Delete           Bound    kube-system/elasticsearch-logging-elasticsearch-logging-1   rook-ceph-block            11m
pvc-e26a2ccf-dd36-4ab3-acfd-332fe7b3c4a9   4Gi        RWO            Delete           Bound    kube-system/elasticsearch-logging-elasticsearch-logging-2   rook-ceph-block            10m
pvc-ef784e9d-39d8-45d5-b5b3-ef52b6a18b70   4Gi        RWO            Delete           Bound    kube-system/elasticsearch-logging-elasticsearch-logging-0   rook-ceph-block            12m
```



### 3.外部存储方式部署

#### 3.1.部署方式

在K8S内部部署Fluentd，Elasticsearch、Kibana部署在K8S集群外部。

Fluentd配置连接外部集群。



#### 3.2.部署Elasticsearch - Kibana

参考：二.4.Docker部署Elasticsearch

版本：elasticsearch:7.4.2



#### 3.3.创建Elasticsearch的Endpoint

```yaml
# elasticsearch-ext.yml


apiVersion: v1
kind: Service
metadata:
  name: elasticsearch-ext
  namespace: kube-system
spec:
  ports:
  - port: 9200
    targetPort: 9200
    protocol: TCP

---
apiVersion: v1
kind: Endpoints
metadata:
  name: elasticsearch-ext
  namespace: kube-system
subsets:
  - addresses:
    - ip: 172.51.216.98
    ports:
    - port: 9200
```



```shell
# 创建
kubectl apply -f elasticsearch-ext.yml 


kubectl describe ep elasticsearch-ext -n kube-system
kubectl describe svc elasticsearch-ext -n kube-system


# K8S 中的容器使用访问
svcname.namespace.svc.cluster.local:port
elasticsearch-ext.kube-system.svc.cluster.local:9200
```



#### 3.4.安装Fluentd

**1.修改配置**

```yaml
# fluentd-es-configmap-ext.yaml 


[root@k8s-master efk]# vim fluentd-es-configmap-ext.yaml 

  output.conf: |-
    <match **>
      @id elasticsearch
      @type elasticsearch
      @log_level info
      type_name _doc
      include_tag_key true
      
      # 外部ES端点配置
      host elasticsearch-ext.kube-system.svc.cluster.local
      
      port 9200
      logstash_format true
      <buffer>
        @type file
        path /var/log/fluentd-buffers/kubernetes.system.buffer
        flush_mode interval
        retry_type exponential_backoff
        flush_thread_count 2
        flush_interval 5s
        retry_forever
        retry_max_interval 30
        chunk_limit_size 2M
        total_limit_size 500M
        overflow_action block
      </buffer>
    </match>
```



**2.安装Fluentd**

```shell
# 安装

[root@k8s-master efk]# kubectl apply -f fluentd-es-configmap-ext.yaml 
configmap/fluentd-es-config-v0.2.0 created


[root@k8s-master efk]# kubectl apply -f fluentd-es-ds.yaml 
serviceaccount/fluentd-es created
clusterrole.rbac.authorization.k8s.io/fluentd-es created
clusterrolebinding.rbac.authorization.k8s.io/fluentd-es created
daemonset.apps/fluentd-es-v3.1.0 created


# 查看
[root@k8s-master efk]# kubectl get pod -n kube-system -o wide
NAME                                       READY   STATUS    RESTARTS   AGE     IP               NODE         NOMINATED NODE   READINESS GATES
fluentd-es-v3.1.0-2vjrv                    1/1     Running   0          2m14s   10.244.169.131   k8s-node2    <none>           <none>
fluentd-es-v3.1.0-fq9pv                    1/1     Running   0          2m14s   10.244.107.231   k8s-node3    <none>           <none>
fluentd-es-v3.1.0-hlb2w                    1/1     Running   0          2m14s   10.244.36.107    k8s-node1    <none>           <none>
```



#### 3.5.测试

```shell
# 访问地址：

http://172.51.216.98:5601/
```

![](/images/kubernetes/monitoring/efk-8.png)

![](/images/kubernetes/monitoring/efk-9.png)





### 4. 微服务测试

#### 4.1.运行微服务

微服务：msa-ext-elk

创建镜像，上传镜像仓库，运行微服务。

**注意：配置私有镜像密钥Secret**



#### 4.2.测试

```shell
# 查看
[root@k8s-master ~]# kubectl get all -n dev
NAME                               READY   STATUS    RESTARTS   AGE
pod/msa-ext-elk-6974d77cb7-fqnrd   1/1     Running   0          5m19s
pod/msa-ext-elk-6974d77cb7-gshjr   1/1     Running   0          5m19s

NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/msa-ext-elk    ClusterIP   10.100.30.131    <none>        8114/TCP   5m19s
service/rabbitmq-svc   ClusterIP   10.101.192.176   <none>        5672/TCP   8m38s

NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/msa-ext-elk   2/2     2            2           5m19s

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/msa-ext-elk-6974d77cb7   2         2         2       5m19s



[root@k8s-master ~]# curl 10.100.30.131:8114/hello
hello，this is elk messge! ### Mon Nov 22 17:08:26 CST 2021
```



#### 4.3.总结

```shell
logback.xml配置同时向mq和控制台输出日志，在ES中会有两份日志，但格式不同：
1.K8S收集的Pod的标准输出日志（通过Pod名称过滤查看）
2.微服务发送到MQ写到的ES的日志
```



### 5.SideCar 模式部署 Filebeat 收集日志

使用 SideCar 模式将日志收集容器与业务容器部署在同一个 Pod 中，只收集对应容器的日志。

业务容器（Nginx）、SideCar（Filebeat）。



#### 5.1.Filebeat配置

**1.filebeat.yml** 

```yaml
[root@k8s-master ~/software]# cat filebeat.yml 


filebeat.inputs:
- type: log
  enabled: true 
  paths:
    - /log/access.log
  json.keys_under_root: true
  json.overwrite_keys: true
  tags: ["access"]

- type: log
  enabled: true 
  paths:
    - /log/error.log
  tags: ["error"]

output.elasticsearch:
  hosts: ["elasticsearch-logging.kube-system.svc.cluster.local:9200"]
  indices:
    - index: "nginx_access-%{[beat.version]}-%{+yyyy.MM}"
      when.contains:
        tags: "access"
    - index: "nginx_error-%{[beat.version]}-%{+yyyy.MM}"
      when.contains:
        tags: "error"

setup.template.name: "nginx"
setup.template.pattern: "nginx_*"
setup.template.enabled: false
setup.template.overwrite: true
```



**2.创建configmap**

```shell
#kubectl create configmap filebeat-cm --from-file=filebeat.yml --namespace=dev

[root@k8s-master efk]# kubectl create configmap filebeat-cm --from-file=filebeat.yml --namespace=dev
configmap/filebeat-cm created


# 查看
[root@k8s-master efk]# kubectl get cm -n dev
NAME               DATA   AGE
filebeat-cm        1      52s


[root@k8s-master efk]# kubectl describe cm filebeat-cm -n dev
Name:         filebeat-cm
Namespace:    dev
Labels:       <none>
Annotations:  <none>

Data
====
filebeat.yml:
----
filebeat.inputs:
- type: log
  enabled: true 
  paths:
    - /log/access.log
  json.keys_under_root: true
  json.overwrite_keys: true
  tags: ["access"]

- type: log
  enabled: true 
  paths:
    - /log/error.log
  tags: ["error"]

output.elasticsearch:
  hosts: ["10.0.0.200:9200"]
  indices:
    - index: "nginx_access-%{[beat.version]}-%{+yyyy.MM}"
      when.contains:
        tags: "access"
    - index: "nginx_error-%{[beat.version]}-%{+yyyy.MM}"
      when.contains:
        tags: "error"

setup.template.name: "nginx"
setup.template.pattern: "nginx_*"
setup.template.enabled: false
setup.template.overwrite: true

Events:  <none>
```



#### 5.2.创建服务

**1.nginx-filebeat-sidecar.yaml**

```yaml
# nginx-filebeat-sidecar.yaml


[root@k8s-master ~/software]# cat nginx-filebeat-sidecar.yaml


apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-filebeat
  namespace: dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-filebeat
  template:
    metadata:
      namespace: dev
      labels:
        app: nginx-filebeat
    spec:
      containers:
      - image: nginx:alpine
        name: nginx
        ports: 
        - containerPort: 80
        volumeMounts:
        - name: applog
          mountPath: /var/log/nginx
      - image: elastic/filebeat:7.4.2
        name: filebeat
        name: filebeat
        volumeMounts:
        - name: applog
          mountPath: /log
        - name: filebeat-config
          mountPath: /usr/share/filebeat
      volumes:
      - name: applog
        emptyDir: {}
      - name: filebeat-config
        configMap:
          name: filebeat-cm
          


[root@k8s-master efk]# kubectl exec -it nginx-filebeat-8674b4d8d4-9xhs7 -c filebeat -n dev -- bash
bash-4.2$ ls
LICENSE.txt  README.md	fields.yml  filebeat.reference.yml  kibana  module
NOTICE.txt   data	filebeat    filebeat.yml	    logs    modules.d
bash-4.2$ pwd
/usr/share/filebeat

mountPath: /etc/filebeat/



# 参考
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-filebeat
  namespace: dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-filebeat
  template:
    metadata:
      namespace: dev
      labels:
        app: nginx-filebeat
    spec:
      containers:
      - image: nginx:alpine
        name: nginx
        ports: 
        - containerPort: 80
        volumeMounts:
        - name: applog
          mountPath: /var/log/nginx
      - image: elastic/filebeat:7.4.2
        name: filebeat
        volumeMounts:
        - name: applog
          mountPath: /log
        - name: filebeat-config
          mountPath: /usr/local/filebeat/config/
      volumes:
      - name: applog
        emptyDir: {}
      - name: filebeat-config
        configMap:
          name: filebeat-cm          
```

```shell
[root@k8s-master efk]# kubectl apply -f nginx-filebeat-sidecar.yaml 
deployment.apps/nginx-filebeat created



[root@k8s-master efk]# kubectl get all -n dev
NAME                                  READY   STATUS    RESTARTS   AGE
pod/nginx-filebeat-6d76d8cddd-zpxk5   2/2     Running   0          111s

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-filebeat   1/1     1            1           111s

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-filebeat-6d76d8cddd   1         1         1       111s
```



**2.service-nginx.yaml**

```yaml
# service-nginx.yaml


[root@k8s-master ~/software]# cat service-nginx.yaml 


apiVersion: v1
kind: Service
metadata:
  name: service-nginx
  namespace: dev
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30781
  selector:
    app: nginx-filebeat
```

```shell
[root@k8s-master efk]# kubectl apply -f service-nginx.yaml 
service/service-nginx created


[root@k8s-master efk]# kubectl get svc -n dev
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service-nginx   NodePort    10.106.197.74    <none>        80:30781/TCP   33s
```



