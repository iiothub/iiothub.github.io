* TOC
{:toc}



## 一、概述



### 1.Ceph

Ceph 是一个分布式存储系统，具备大规模、高性能、无单点失败的特点。Ceph 是一个软件定义的系统，也就是说他可以运行在任何符合其要求的硬件之上。

Ceph 是一个开源的分布式存储系统，包括[对象存储](https://cloud.tencent.com/product/cos?from=10680)、块设备、文件系统。它可靠性高、管理方便、伸缩性强，能够轻松应对 PB、EB 级别数据。

Ceph是一种高度可扩展的分布式存储解决方案，提供对象、文件和块存储。在每个存储节点上，您将找到Ceph存储对象的文件系统和Ceph OSD（对象存储守护程序）进程。在Ceph集群上，您还可以找到Ceph MON（监控）守护程序，它们确保Ceph集群保持高可用性。

![](/images/kubernetes/storage/rook-4.png)

![](/images/kubernetes/storage/rook-5.png)



![](/images/kubernetes/storage/rook-6.png)





**Ceph 包括多个组件：**

- **Ceph Monitors(MON)**：负责生成集群票选机制。所有的集群节点都会向 Mon 进行汇报，并在每次状态变更时进行共享信息。
- **Ceph Object Store Devices(OSD)**：负责在本地文件系统保存对象，并通过网络提供访问。通常 OSD 守护进程会绑定在集群的一个物理盘上，Ceph 客户端直接和 OSD 打交道。
- **Ceph Manager(MGR)**：提供额外的监控和界面给外部的监管系统使用。
- **Reliable Autonomic Distributed Object Stores**：Ceph 存储集群的核心。这一层用于为存储数据提供一致性保障，执行数据复制、故障检测以及恢复等任务。

为了在 Ceph 上进行读写，客户端首先要联系 MON，获取最新的集群地图，其中包含了集群拓扑以及数据存储位置的信息。Ceph 客户端使用集群地图来获知需要交互的 OSD，从而和特定 OSD 建立联系。



### 2.Rook

Rook是一个自我管理的分布式存储编排系统，它本身并不是存储系统，在存储和k8s之间搭建了一个桥梁，使存储系统的搭建或者维护变得特别简单，Rook将分布式存储系统转变为自我管理、自我扩展、自我修复的存储服务。它让一些存储的操作，比如部署、配置、扩容、升级、迁移、灾难恢复、监视和资源管理变得自动化，无需人工处理。并且Rook支持cSsl，可以利用CSI做一些Pvc的快照、扩容等操作。

Rook 是一个可以提供 Ceph 集群管理能力的 Operator。Rook 使用 CRD 一个控制器来对 Ceph 之类的资源进行部署和管理。

Rook 是专用于 Cloud-Native 环境的文件、块、对象存储服务。它实现了一个自动管理的、自动扩容的、自动修复的分布式存储服务。Rook 支持 Ceph 存储，基于 Kubernetes 使用 Rook 可以大大简化 Ceph 存储集群的搭建以及使用。

**Rook 如何集成在kubernetes 如图：**

![](/images/kubernetes/storage/rook-2.png)

**Rook部署Ceph集群的架构图**

![](/images/kubernetes/storage/rook-1.png)

各组件说明

- Operator：Rook控制端，监控存储守护进程，确保存储集群的健康

- Agent：在每个存储节点创建，配置了FlexVolume插件和Kubernetes 的存储卷控制框架（CSI）进行集成

- OSD：提供存储，每块硬盘可以看做一个osd

- Mon：监控ceph集群的存储情况，记录集群的拓扑，数据存储的位置信息

- MDS：负责跟踪**文件存储**的层次结构

- RGW：Rest API结构，提供**对象存储**接口

- MGR：为外界提供统一入口

  

![](/images/kubernetes/storage/rook-3.png)



**Rook 包含多个组件：**

- **Rook Operator**：Rook 的核心组件，Rook Operator 是一个简单的容器，自动启动存储集群，并监控存储守护进程，来确保存储集群的健康
- **Rook Agent**：在每个存储节点上运行，并配置一个 FlexVolume 插件，和 Kubernetes 的存储卷控制框架进行集成。Agent 处理所有的存储操作，例如挂接网络存储设备、在主机上加载存储卷以及格式化文件系统等。
- **Rook Discovers**：检测挂接到存储节点上的存储设备
  - Rook 还会用 Kubernetes Pod 的形式，部署 Ceph 的 MON、OSD 以及 MGR 守护进程
  - Rook Operator 让用户可以通过 CRD 的是用来创建和管理存储集群。每种资源都定义了自己的 CRD
- **Rook Cluster**：提供了对存储机群的配置能力，用来提供块存储、[对象存储](https://cloud.tencent.com/product/cos?from=10680)以及共享文件系统。每个集群都有多个 Pool
- **Pool**：为块存储提供支持。Pool 也是给文件和对象存储提供内部支持
- **Object Store**：用 S3 兼容接口开放存储服务
- **File System**：为多个 Kubernetes Pod 提供共享存储





## 二、基础



### 1.常用操作

```shell
# 实时查看pod创建进度
kubectl get pod -n rook-ceph -w
# 实时查看集群创建进度
kubectl get cephcluster -n rook-ceph rook-ceph -w
# 详细描述
kubectl describe cephcluster -n rook-ceph rook-ceph

kubectl get all -n rook-ceph



------------------------------------------------
[root@k8s-master ~]# kubectl get cephcluster -n rook-ceph rook-ceph -w
NAME        DATADIRHOSTPATH   MONCOUNT   AGE   PHASE   MESSAGE                        HEALTH      EXTERNAL
rook-ceph   /var/lib/rook     3          22h   Ready   Cluster created successfully   HEALTH_OK 


[root@k8s-master ~]# kubectl get all -n rook-ceph
NAME                                                      READY   STATUS      RESTARTS   AGE
pod/csi-cephfsplugin-26gzf                                3/3     Running     0          22h
pod/csi-cephfsplugin-bn9f2                                3/3     Running     0          21h
pod/csi-cephfsplugin-hhjsz                                3/3     Running     0          22h
pod/csi-cephfsplugin-provisioner-575f74897d-j69qc         6/6     Running     0          22h
pod/csi-cephfsplugin-provisioner-575f74897d-lqw4d         6/6     Running     0          21h
pod/csi-rbdplugin-gfsgm                                   3/3     Running     0          21h
pod/csi-rbdplugin-l24pz                                   3/3     Running     0          22h
pod/csi-rbdplugin-pgh7d                                   3/3     Running     0          22h
pod/csi-rbdplugin-provisioner-8576bbbbc7-86rg5            6/6     Running     0          22h
pod/csi-rbdplugin-provisioner-8576bbbbc7-vb24x            6/6     Running     0          21h
pod/rook-ceph-crashcollector-k8s-node1-7b85799995-vrkzc   1/1     Running     0          21h
pod/rook-ceph-crashcollector-k8s-node2-5c6dd78c8-z49cj    1/1     Running     0          21h
pod/rook-ceph-crashcollector-k8s-node3-864f9745b9-4bg8w   1/1     Running     0          21h
pod/rook-ceph-mgr-a-54ffd8fb84-4nxqx                      1/1     Running     0          21h
pod/rook-ceph-mon-a-7d87565c6c-bmptg                      1/1     Running     0          22h
pod/rook-ceph-mon-b-75d97b58dd-2k5b4                      1/1     Running     0          22h
pod/rook-ceph-mon-c-57856c8589-kdxcc                      1/1     Running     0          21h
pod/rook-ceph-operator-5b94b79f5-72jw9                    1/1     Running     0          22h
pod/rook-ceph-osd-0-855b4fb47-xc8tx                       1/1     Running     0          21h
pod/rook-ceph-osd-1-796c4f55f9-lws4t                      1/1     Running     0          21h
pod/rook-ceph-osd-2-555574f875-fjnbd                      1/1     Running     0          21h
pod/rook-ceph-osd-3-7b7474b4cb-56gm4                      1/1     Running     0          119m
pod/rook-ceph-osd-prepare-k8s-node1-s8wfj                 0/1     Completed   0          118m
pod/rook-ceph-osd-prepare-k8s-node2-cd228                 0/1     Completed   0          118m
pod/rook-ceph-osd-prepare-k8s-node3-wffgm                 0/1     Completed   0          118m
pod/rook-ceph-tools-6bc7c4f9fc-p5j59                      1/1     Running     0          22h
pod/rook-discover-2n5j5                                   1/1     Running     0          22h
pod/rook-discover-8kgz6                                   1/1     Running     0          22h
pod/rook-discover-fs2lc                                   1/1     Running     0          22h

NAME                                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
service/csi-cephfsplugin-metrics        ClusterIP   10.108.51.169    <none>        8080/TCP,8081/TCP   22h
service/csi-rbdplugin-metrics           ClusterIP   10.96.25.112     <none>        8080/TCP,8081/TCP   22h
service/rook-ceph-mgr                   ClusterIP   10.100.194.156   <none>        9283/TCP            21h
service/rook-ceph-mgr-dashboard         ClusterIP   10.100.100.126   <none>        8443/TCP            21h
service/rook-ceph-mgr-dashboard-http    NodePort    10.111.92.246    <none>        7000:31700/TCP      3h43m
service/rook-ceph-mgr-dashboard-https   NodePort    10.109.20.210    <none>        8443:32700/TCP      3h45m
service/rook-ceph-mon-a                 ClusterIP   10.97.103.24     <none>        6789/TCP,3300/TCP   22h
service/rook-ceph-mon-b                 ClusterIP   10.104.142.159   <none>        6789/TCP,3300/TCP   22h
service/rook-ceph-mon-c                 ClusterIP   10.109.133.209   <none>        6789/TCP,3300/TCP   21h

NAME                              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/csi-cephfsplugin   3         3         3       3            3           <none>          22h
daemonset.apps/csi-rbdplugin      3         3         3       3            3           <none>          22h
daemonset.apps/rook-discover      3         3         3       3            3           <none>          22h

NAME                                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/csi-cephfsplugin-provisioner         2/2     2            2           22h
deployment.apps/csi-rbdplugin-provisioner            2/2     2            2           22h
deployment.apps/rook-ceph-crashcollector-k8s-node1   1/1     1            1           21h
deployment.apps/rook-ceph-crashcollector-k8s-node2   1/1     1            1           21h
deployment.apps/rook-ceph-crashcollector-k8s-node3   1/1     1            1           21h
deployment.apps/rook-ceph-mgr-a                      1/1     1            1           21h
deployment.apps/rook-ceph-mon-a                      1/1     1            1           22h
deployment.apps/rook-ceph-mon-b                      1/1     1            1           22h
deployment.apps/rook-ceph-mon-c                      1/1     1            1           21h
deployment.apps/rook-ceph-operator                   1/1     1            1           22h
deployment.apps/rook-ceph-osd-0                      1/1     1            1           21h
deployment.apps/rook-ceph-osd-1                      1/1     1            1           21h
deployment.apps/rook-ceph-osd-2                      1/1     1            1           21h
deployment.apps/rook-ceph-osd-3                      1/1     1            1           119m
deployment.apps/rook-ceph-tools                      1/1     1            1           22h

NAME                                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/csi-cephfsplugin-provisioner-575f74897d         2         2         2       22h
replicaset.apps/csi-rbdplugin-provisioner-8576bbbbc7            2         2         2       22h
replicaset.apps/rook-ceph-crashcollector-k8s-node1-7b85799995   1         1         1       21h
replicaset.apps/rook-ceph-crashcollector-k8s-node2-5c6dd78c8    1         1         1       21h
replicaset.apps/rook-ceph-crashcollector-k8s-node3-864f9745b9   1         1         1       21h
replicaset.apps/rook-ceph-crashcollector-k8s-node3-fc65d5787    0         0         0       21h
replicaset.apps/rook-ceph-mgr-a-54ffd8fb84                      1         1         1       21h
replicaset.apps/rook-ceph-mon-a-7d87565c6c                      1         1         1       22h
replicaset.apps/rook-ceph-mon-b-75d97b58dd                      1         1         1       22h
replicaset.apps/rook-ceph-mon-c-57856c8589                      1         1         1       21h
replicaset.apps/rook-ceph-operator-5b94b79f5                    1         1         1       22h
replicaset.apps/rook-ceph-osd-0-855b4fb47                       1         1         1       21h
replicaset.apps/rook-ceph-osd-1-796c4f55f9                      1         1         1       21h
replicaset.apps/rook-ceph-osd-2-555574f875                      1         1         1       21h
replicaset.apps/rook-ceph-osd-3-7b7474b4cb                      1         1         1       119m
replicaset.apps/rook-ceph-tools-6bc7c4f9fc                      1         1         1       22h

NAME                                        COMPLETIONS   DURATION   AGE
job.batch/rook-ceph-osd-prepare-k8s-node1   1/1           4s         118m
job.batch/rook-ceph-osd-prepare-k8s-node2   1/1           4s         118m
job.batch/rook-ceph-osd-prepare-k8s-node3   1/1           4s         118m
```



### 2.Toolbox

```shell
# 进入工作目录
cd /root/rook/cluster/examples/kubernetes/ceph
# 创建toolbox
kubectl  create -f toolbox.yaml -n rook-ceph


# 查看pod
kubectl  get pod -n rook-ceph -l app=rook-ceph-tools
-# 进入pod
kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash
-# 查看集群状态
ceph status
# 查看osd状态
ceph osd status
# 集群空间用量
ceph df
rados df
# 健康情况
ceph health detail



-------------------------------------------------
[root@k8s-master ~]# kubectl  get pod -n rook-ceph -l app=rook-ceph-tools
NAME                               READY   STATUS    RESTARTS   AGE
rook-ceph-tools-6bc7c4f9fc-p5j59   1/1     Running   0          81s


[root@k8s-master ~]# kubectl -n rook-ceph exec -it rook-ceph-tools-6bc7c4f9fc-p5j59 -- bash
[root@k8s-master ~]# kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash
[root@rook-ceph-tools-6bc7c4f9fc-p5j59 /]# ceph status
  cluster:
    id:     b5824376-be37-4d71-aeb2-b34ef718bc17
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum a,c,b (age 3h)
    mgr: a(active, since 22h)
    osd: 4 osds: 4 up (since 2h), 4 in (since 2h)
 
  data:
    pools:   1 pools, 1 pgs
    objects: 0 objects, 0 B
    usage:   4.0 GiB used, 396 GiB / 400 GiB avail
    pgs:     1 active+clean


[root@rook-ceph-tools-6bc7c4f9fc-p5j59 /]# ceph osd status
ID  HOST        USED  AVAIL  WR OPS  WR DATA  RD OPS  RD DATA  STATE      
 0  k8s-node3  1027M  98.9G      0        0       0        0   exists,up  
 1  k8s-node2  1027M  98.9G      0        0       0        0   exists,up  
 2  k8s-node1  1027M  98.9G      0        0       0        0   exists,up  
 3  k8s-node1  1027M  98.9G      0        0       0        0   exists,up   

 
[root@rook-ceph-tools-6bc7c4f9fc-p5j59 /]# ceph df
--- RAW STORAGE ---
CLASS  SIZE     AVAIL    USED    RAW USED  %RAW USED
hdd    400 GiB  396 GiB  14 MiB   4.0 GiB       1.00
TOTAL  400 GiB  396 GiB  14 MiB   4.0 GiB       1.00
 
--- POOLS ---
POOL                   ID  PGS  STORED  OBJECTS  USED  %USED  MAX AVAIL
device_health_metrics   1    1     0 B        0   0 B      0    125 GiB


[root@rook-ceph-tools-6bc7c4f9fc-p5j59 /]# rados df
POOL_NAME              USED  OBJECTS  CLONES  COPIES  MISSING_ON_PRIMARY  UNFOUND  DEGRADED  RD_OPS   RD  WR_OPS   WR  USED COMPR  UNDER COMPR
device_health_metrics   0 B        0       0       0                   0        0         0       0  0 B       0  0 B         0 B          0 B

total_objects    0
total_used       4.0 GiB
total_avail      396 GiB
total_space      400 GiB


[root@rook-ceph-tools-6bc7c4f9fc-p5j59 /]# ceph health detail
HEALTH_OK
```



### 3.Dashboard

```shell
#访问地址：
https://172.51.216.81:32700/
admin
1qaz2wsx


# 获取密码
kubectl -n rook-ceph get secret rook-ceph-dashboard-password -o jsonpath="{['data']['password']}" | base64 --decode && echo
```

![](/images/kubernetes/storage/rook-7.png)

![](/images/kubernetes/storage/rook-8.png)

![](/images/kubernetes/storage/rook-9.png)

![](/images/kubernetes/storage/rook-10.png)



### 4.Ceph存储使用

#### 4.1. 三种存储类型

| 存储类型           | 特征                                                         | 应用场景          | 典型设备  |
| :----------------- | :----------------------------------------------------------- | :---------------- | :-------- |
| 块存储（RBD）      | 存储速度较快 不支持共享存储 [**ReadWriteOnce**]              | 虚拟机硬盘        | 硬盘 Raid |
| 文件存储（CephFS） | 存储速度慢（需经操作系统处理再转为块存储） 支持共享存储 [**ReadWriteMany**] | 文件共享          | FTP NFS   |
| 对象存储（Object） | 具备块存储的读写性能和文件存储的共享特性 操作系统不能直接访问，只能通过应用程序级别的API访问 | 图片存储 视频存储 | OSS       |

**使用方式：**

- **块存储（RBD）：**适用StatefulSet，每个Pod有自己的存储；
- **文件存储（CephFS）：**适用Deployment，多个Pod文件共享；



#### 4.2. 块存储

**1.创建CephBlockPool和StorageClass**

- 文件路径：`/k8s/rook/rook/cluster/examples/kubernetes/ceph/csi/rbd/storageclass.yaml`
- CephBlockPool和StorageClass都位于storageclass.yaml 文件
- 官网参考：https://www.rook.io/docs/rook/v1.6/ceph-block.html



```yaml
vim storageclass.yaml
-------------------------------------------
 
apiVersion: ceph.rook.io/v1
kind: CephBlockPool
metadata:
  name: replicapool
  namespace: rook-ceph
spec:
  failureDomain: host # host级容灾
  replicated:
    size: 3           # 默认三个副本
    requireSafeReplicaSize: true  # 强制高可用，如果size为1则需改为false
 
---
apiVersion: storage.k8s.io/v1
kind: StorageClass                       # sc无需指定命名空间
metadata:
  name: rook-ceph-block
provisioner: rook-ceph.rbd.csi.ceph.com   # 存储驱动
parameters:
  clusterID: rook-ceph # namespace:cluster
  pool: replicapool                       # 关联到CephBlockPool
  imageFormat: "2"
  imageFeatures: layering
 
  csi.storage.k8s.io/provisioner-secret-name: rook-csi-rbd-provisioner
  csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph # namespace:cluster
  csi.storage.k8s.io/controller-expand-secret-name: rook-csi-rbd-provisioner
  csi.storage.k8s.io/controller-expand-secret-namespace: rook-ceph # namespace:cluster
  csi.storage.k8s.io/node-stage-secret-name: rook-csi-rbd-node
  csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph # namespace:cluster
  
  csi.storage.k8s.io/fstype: ext4   
 
allowVolumeExpansion: true               # 是否允许扩容
reclaimPolicy: Delete                    # PV回收策略
```



创建CephBlockPool和StorageClass

```shell
[root@k8s-master test]# kubectl apply -f storageclass.yaml 
cephblockpool.ceph.rook.io/replicapool created
storageclass.storage.k8s.io/rook-ceph-block created



# 查看sc
[root@k8s-master test]# kubectl get sc
NAME              PROVISIONER                  RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
rook-ceph-block   rook-ceph.rbd.csi.ceph.com   Delete          Immediate           true                   28s

 
# 查看CephBlockPool（也可在dashboard中查看）
[root@k8s-master test]# kubectl get cephblockpools -n rook-ceph
NAME          AGE
replicapool   2m41s
```

![](/images/kubernetes/storage/rook-11.png)



**2.Deployment单副本+PersistentVolumeClaim**

```yaml
vim nginx-deploy-rbd.yaml
-------------------------------
 
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deploy-rbd
  name: nginx-deploy-rbd
  namespace: dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-deploy-rbd
  template:
    metadata:
      labels:
        app: nginx-deploy-rbd
    spec:
      containers:
      - image: nginx
        name: nginx
        volumeMounts:
        - name: data
          mountPath: /usr/share/nginx/html
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: nginx-rbd-pvc
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-rbd-pvc
  namespace: dev
spec:
  storageClassName: "rook-ceph-block"
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

```shell
# 创建
[root@k8s-master test]# kubectl apply -f nginx-deploy-rbd.yaml 
deployment.apps/nginx-deploy-rbd created
persistentvolumeclaim/nginx-rbd-pvc created



[root@k8s-master test]# kubectl get all -n dev
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deploy-rbd-7f468884cf-j6v4b   1/1     Running   0          110s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy-rbd   1/1     1            1           110s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deploy-rbd-7f468884cf   1         1         1       110s



[root@k8s-master test]# kubectl get pvc -n dev
NAME            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
nginx-rbd-pvc   Bound    pvc-32b9ebfe-1646-4a8f-bb03-e9496c3f9dc8   1Gi        RWO            rook-ceph-block   2m12s

[root@k8s-master ~]# kubectl get pv -n dev
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS      REASON   AGE
pvc-e1a18fe9-cc52-4a5a-881e-1746ff84f601   1Gi        RWO            Delete           Bound    dev/nginx-rbd-pvc        rook-ceph-block            33m



# Toolbox
[root@rook-ceph-tools-6bc7c4f9fc-p5j59 /]# ceph df
--- RAW STORAGE ---
CLASS  SIZE     AVAIL    USED    RAW USED  %RAW USED
hdd    400 GiB  396 GiB  70 MiB   4.1 GiB       1.02
TOTAL  400 GiB  396 GiB  70 MiB   4.1 GiB       1.02
 
--- POOLS ---
POOL                   ID  PGS  STORED  OBJECTS  USED    %USED  MAX AVAIL
device_health_metrics   1    1     0 B        0     0 B      0    125 GiB
replicapool             2   32  17 MiB       17  51 MiB   0.01    125 GiB


[root@rook-ceph-tools-6bc7c4f9fc-p5j59 /]# rados df
POOL_NAME                USED  OBJECTS  CLONES  COPIES  MISSING_ON_PRIMARY  UNFOUND  DEGRADED  RD_OPS       RD  WR_OPS      WR  USED COMPR  UNDER COMPR
device_health_metrics     0 B        0       0       0                   0        0         0       0      0 B       0     0 B         0 B          0 B
replicapool            51 MiB       17       0      51                   0        0         0    3639  7.0 MiB      46  17 MiB         0 B          0 B

total_objects    17
total_used       4.1 GiB
total_avail      396 GiB
total_space      400 GiB
```

![](/images/kubernetes/storage/rook-12.png)

```shell
# 测试
[root@k8s-master ~]# kubectl -n dev exec -it nginx-deploy-rbd-7f468884cf-j6v4b -- bash
root@nginx-deploy-rbd-7f468884cf-j6v4b:/# echo "Hello  Nginx Rook-block" > /usr/share/nginx/html/index.html
root@nginx-deploy-rbd-7f468884cf-j6v4b:/# cat /usr/share/nginx/html/index.html
Hello  Nginx Rook-block


[root@k8s-master ~]# kubectl get pod -n dev -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP              NODE        NOMINATED NODE   READINESS GATES
nginx-deploy-rbd-7f468884cf-j6v4b   1/1     Running   0          18m   10.244.36.105   k8s-node1   <none>           <none>

[root@k8s-master ~]# curl 10.244.36.105:80
Hello  Nginx Rook-block


# 删除Pod,Pod重建后数据还在
[root@k8s-master ~]# kubectl delete pod nginx-deploy-rbd-7f468884cf-j6v4b -n dev
pod "nginx-deploy-rbd-7f468884cf-j6v4b" deleted
 
[root@k8s-master ~]# kubectl get pod -n dev -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP              NODE        NOMINATED NODE   READINESS GATES
nginx-deploy-rbd-7f468884cf-jnk8n   1/1     Running   0          27s   10.244.36.107   k8s-node1   <none>           <none>

[root@k8s-master ~]# curl 10.244.36.107:80
Hello  Nginx Rook-block



# 删除后，块消失，pvc、pv删除
[root@k8s-master test]# kubectl delete -f nginx-deploy-rbd.yaml 
deployment.apps "nginx-deploy-rbd" deleted
persistentvolumeclaim "nginx-rbd-pvc" deleted

# 重新创建
[root@k8s-master test]# kubectl apply -f nginx-deploy-rbd.yaml 
deployment.apps/nginx-deploy-rbd created
persistentvolumeclaim/nginx-rbd-pvc created

[root@k8s-master test]# kubectl get pod -n dev -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP              NODE        NOMINATED NODE   READINESS GATES
nginx-deploy-rbd-7f468884cf-rfzpk   1/1     Running   0          25s   10.244.36.100   k8s-node1   <none>           <none>

# 原来数据不存在
[root@k8s-master test]# curl 10.244.36.100:80
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.21.4</center>
</body>
</html>
```



**3.StatefulSet多副本+volumeClaimTemplates**

```yaml
vim nginx-ss-rbd.yaml
-------------------------------
 
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-ss-rbd
  namespace: dev
spec:
  selector:
    matchLabels:
      app: nginx-ss-rbd 
  serviceName: "nginx"
  replicas: 3 
  template:
    metadata:
      labels:
        app: nginx-ss-rbd 
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "rook-ceph-block"
      resources:
        requests:
          storage: 2Gi
```

```shell
# 创建
[root@k8s-master test]# kubectl apply -f nginx-ss-rbd.yaml 
statefulset.apps/nginx-ss-rbd created


# 查看
[root@k8s-master test]# kubectl get all -n dev
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deploy-rbd-7f468884cf-rfzpk   1/1     Running   0          14m
pod/nginx-ss-rbd-0                      1/1     Running   0          112s
pod/nginx-ss-rbd-1                      1/1     Running   0          89s
pod/nginx-ss-rbd-2                      1/1     Running   0          60s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy-rbd   1/1     1            1           14m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deploy-rbd-7f468884cf   1         1         1       14m

NAME                            READY   AGE
statefulset.apps/nginx-ss-rbd   3/3     112s


[root@k8s-master ~]# kubectl get pvc -n dev
NAME                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
www-nginx-ss-rbd-0   Bound    pvc-2f1a8b03-9bff-4769-8f48-16a91ccfcaa8   2Gi        RWO            rook-ceph-block   19m
www-nginx-ss-rbd-1   Bound    pvc-2db380dc-d52c-470c-807f-5829eb5694ab   2Gi        RWO            rook-ceph-block   19m
www-nginx-ss-rbd-2   Bound    pvc-cbd3478d-9bf0-4607-9212-04ef5391feb7   2Gi        RWO            rook-ceph-block   18m

[root@k8s-master ~]# kubectl get pv -n dev
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS      REASON   AGE
pvc-2db380dc-d52c-470c-807f-5829eb5694ab   2Gi        RWO            Delete           Bound    dev/www-nginx-ss-rbd-1   rook-ceph-block            20m
pvc-2f1a8b03-9bff-4769-8f48-16a91ccfcaa8   2Gi        RWO            Delete           Bound    dev/www-nginx-ss-rbd-0   rook-ceph-block            20m
pvc-cbd3478d-9bf0-4607-9212-04ef5391feb7   2Gi        RWO            Delete           Bound    dev/www-nginx-ss-rbd-2   rook-ceph-block            19m
```

![](/images/kubernetes/storage/rook-13.png)

```shell
# 测试
[root@k8s-master test]# kubectl get pod -n dev -owide | grep ss
nginx-ss-rbd-0                      1/1     Running   0          8m44s   10.244.36.109   k8s-node1   <none>           <none>
nginx-ss-rbd-1                      1/1     Running   0          8m21s   10.244.36.77    k8s-node1   <none>           <none>
nginx-ss-rbd-2                      1/1     Running   0          7m52s   10.244.36.108   k8s-node1   <none>           <none>


[root@k8s-master test]# kubectl -n dev exec -it nginx-ss-rbd-0 -- bash
root@nginx-ss-rbd-0:/# echo "Hello  Nginx Rook-block-0" > /usr/share/nginx/html/index.html
root@nginx-ss-rbd-0:/# cat /usr/share/nginx/html/index.html
Hello  Nginx Rook-block-0

[root@k8s-master test]# kubectl -n dev exec -it nginx-ss-rbd-1 -- bash
root@nginx-ss-rbd-1:/# echo "Hello  Nginx Rook-block-1" > /usr/share/nginx/html/index.html
root@nginx-ss-rbd-1:/#  cat /usr/share/nginx/html/index.html
Hello  Nginx Rook-block-1

[root@k8s-master test]# kubectl -n dev exec -it nginx-ss-rbd-2 -- bash
root@nginx-ss-rbd-2:/# echo "Hello  Nginx Rook-block-2" > /usr/share/nginx/html/index.html
root@nginx-ss-rbd-2:/# cat /usr/share/nginx/html/index.html
Hello  Nginx Rook-block-2


[root@k8s-master test]# curl 10.244.36.109
Hello  Nginx Rook-block-0
[root@k8s-master test]# curl 10.244.36.77
Hello  Nginx Rook-block-1
[root@k8s-master test]# curl 10.244.36.108
Hello  Nginx Rook-block-2


# 删除Pod，Pod重建后，数据还在
[root@k8s-master test]# kubectl get pod -n dev -owide | grep ss
nginx-ss-rbd-0                      1/1     Running   0          15m   10.244.36.109   k8s-node1   <none>           <none>
nginx-ss-rbd-1                      1/1     Running   0          14m   10.244.36.77    k8s-node1   <none>           <none>
nginx-ss-rbd-2                      1/1     Running   0          14m   10.244.36.108   k8s-node1   <none>           <none>

[root@k8s-master test]# kubectl delete pod nginx-ss-rbd-0 -n dev
pod "nginx-ss-rbd-0" deleted
[root@k8s-master test]# kubectl delete pod nginx-ss-rbd-1 -n dev
pod "nginx-ss-rbd-1" deleted
[root@k8s-master test]# kubectl delete pod nginx-ss-rbd-2 -n dev
pod "nginx-ss-rbd-2" deleted

[root@k8s-master test]# kubectl get pod -n dev -owide | grep ss
nginx-ss-rbd-0                      1/1     Running   0          56s   10.244.36.111   k8s-node1   <none>           <none>
nginx-ss-rbd-1                      1/1     Running   0          45s   10.244.36.112   k8s-node1   <none>           <none>
nginx-ss-rbd-2                      1/1     Running   0          20s   10.244.36.114   k8s-node1   <none>           <none>

[root@k8s-master test]# curl 10.244.36.111
Hello  Nginx Rook-block-0
[root@k8s-master test]# curl 10.244.36.112
Hello  Nginx Rook-block-1
[root@k8s-master test]# curl 10.244.36.114
Hello  Nginx Rook-block-2

# 删除之后，pvc、pv都存在
[root@k8s-master test]# kubectl delete -f nginx-deploy-rbd.yaml 
deployment.apps "nginx-deploy-rbd" deleted
persistentvolumeclaim "nginx-rbd-pvc" deleted

[root@k8s-master ~]# kubectl get pv -n dev
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS      REASON   AGE
pvc-2db380dc-d52c-470c-807f-5829eb5694ab   2Gi        RWO            Delete           Bound    dev/www-nginx-ss-rbd-1   rook-ceph-block            34m
pvc-2f1a8b03-9bff-4769-8f48-16a91ccfcaa8   2Gi        RWO            Delete           Bound    dev/www-nginx-ss-rbd-0   rook-ceph-block            34m
pvc-cbd3478d-9bf0-4607-9212-04ef5391feb7   2Gi        RWO            Delete           Bound    dev/www-nginx-ss-rbd-2   rook-ceph-block            34m

[root@k8s-master ~]# kubectl get pvc -n dev
NAME                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
www-nginx-ss-rbd-0   Bound    pvc-2f1a8b03-9bff-4769-8f48-16a91ccfcaa8   2Gi        RWO            rook-ceph-block   35m
www-nginx-ss-rbd-1   Bound    pvc-2db380dc-d52c-470c-807f-5829eb5694ab   2Gi        RWO            rook-ceph-block   34m
www-nginx-ss-rbd-2   Bound    pvc-cbd3478d-9bf0-4607-9212-04ef5391feb7   2Gi        RWO            rook-ceph-block   34m


# 再次重建数据还在
[root@k8s-master test]# kubectl apply -f nginx-ss-rbd.yaml 
statefulset.apps/nginx-ss-rbd created

[root@k8s-master test]#  kubectl get pod -n dev -owide | grep ss
nginx-ss-rbd-0   1/1     Running   0          89s   10.244.36.116   k8s-node1   <none>           <none>
nginx-ss-rbd-1   1/1     Running   0          63s   10.244.36.106   k8s-node1   <none>           <none>
nginx-ss-rbd-2   1/1     Running   0          30s   10.244.36.122   k8s-node1   <none>           <none>
 
[root@k8s-master test]# curl 10.244.36.116
Hello  Nginx Rook-block-0
[root@k8s-master test]# curl 10.244.36.106
Hello  Nginx Rook-block-1
[root@k8s-master test]# curl 10.244.36.122
Hello  Nginx Rook-block-2


# 删除StatefulSet，需要手动删除pvc
```



#### 4.3.共享文件存储

**1.部署MDS**

- 创建Cephfs文件系统需要先部署MDS服务，该服务负责处理文件系统中的元数据。
- 文件路径：`/k8s/rook/rook/cluster/examples/kubernetes/ceph/filesystem.yaml`
- 官方参考： https://www.rook.io/docs/rook/v1.7/ceph-filesystem.html



```yaml
vim filesystem.yaml
---------------------------------------


apiVersion: ceph.rook.io/v1
kind: CephFilesystem
metadata:
  name: myfs
  namespace: rook-ceph 
spec:
  metadataPool:
    replicated:
      size: 3                         # 元数据副本数
      requireSafeReplicaSize: true
    parameters:
      compression_mode:
        none
  dataPools:
    - failureDomain: host
      replicated:
        size: 3                     # 存储数据的副本数
        requireSafeReplicaSize: true
      parameters:
        compression_mode:
          none
  preserveFilesystemOnDelete: true
  metadataServer:
    activeCount: 3                # MDS实例的副本数，默认1，生产环境建议设置为3
```

```shell
[root@k8s-master test]# kubectl apply -f filesystem.yaml 
cephfilesystem.ceph.rook.io/myfs created


[root@k8s-master ~]# kubectl get CephFilesystem -n rook-ceph
NAME   ACTIVEMDS   AGE    PHASE
myfs   3           150m   Ready

[root@k8s-master ~]# kubectl -n rook-ceph get pod -l app=rook-ceph-mds
NAME                                    READY   STATUS    RESTARTS   AGE
rook-ceph-mds-myfs-a-7f65bc58fc-fk8mr   1/1     Running   0          4h3m
rook-ceph-mds-myfs-b-57c67856c5-95mgk   1/1     Running   0          4h3m
rook-ceph-mds-myfs-c-6bdf97c5d9-t2qm6   1/1     Running   0          4h3m
rook-ceph-mds-myfs-d-65fc96f959-l6zmh   1/1     Running   0          4h3m
rook-ceph-mds-myfs-e-799998bcd9-fhmtf   1/1     Running   0          4h3m
rook-ceph-mds-myfs-f-5d964dcb7f-qnfz2   1/1     Running   0          4h3m


[root@rook-ceph-tools-6bc7c4f9fc-p5j59 /]# ceph status
...
  services:
    mds: myfs:3 {0=myfs-b=up:active,1=myfs-a=up:active,2=myfs-e=up:active} 3 up:standby
    ...
```



**2.创建StorageClass**

- 文件路径：`/k8s/rook/rook/cluster/examples/kubernetes/ceph/csi/cephfs/storageclass.yaml`

```yaml
vim storageclass.yaml 
----------------------------------


apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rook-cephfs
provisioner: rook-ceph.cephfs.csi.ceph.com # driver:namespace:operator
parameters:
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rook-cephfs
provisioner: rook-ceph.cephfs.csi.ceph.com # driver:namespace:operator
parameters:
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rook-cephfs
provisioner: rook-ceph.cephfs.csi.ceph.com # driver:namespace:operator
parameters:
  # clusterID is the namespace where operator is deployed.
  clusterID: rook-ceph # namespace:cluster

  # CephFS filesystem name into which the volume shall be created
  fsName: myfs

  # Ceph pool into which the volume shall be created
  # Required for provisionVolume: "true"
  pool: myfs-data0

  # The secrets contain Ceph admin credentials. These are generated automatically by the operator
  # in the same namespace as the cluster.
  csi.storage.k8s.io/provisioner-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph # namespace:cluster
  csi.storage.k8s.io/controller-expand-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/controller-expand-secret-namespace: rook-ceph # namespace:cluster
  csi.storage.k8s.io/node-stage-secret-name: rook-csi-cephfs-node
  csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph # namespace:cluster

  # (optional) The driver can use either ceph-fuse (fuse) or ceph kernel client (kernel)
  # If omitted, default volume mounter will be used - this is determined by probing for ceph-fuse
  # or by setting the default mounter explicitly via --volumemounter command-line argument.
  # mounter: kernel
reclaimPolicy: Delete
allowVolumeExpansion: true
mountOptions:
  # uncomment the following line for debugging
  #- debug
```



```shell
[root@k8s-master cephfs]# cd /k8s/rook/rook/cluster/examples/kubernetes/ceph/csi/cephfs
[root@k8s-master cephfs]# pwd
/k8s/rook/rook/cluster/examples/kubernetes/ceph/csi/cephfs



[root@k8s-master cephfs]# kubectl apply -f storageclass.yaml
storageclass.storage.k8s.io/rook-cephfs created


[root@k8s-master cephfs]# kubectl get sc
NAME              PROVISIONER                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
rook-ceph-block   rook-ceph.rbd.csi.ceph.com      Delete          Immediate           true                   4h31m
rook-cephfs       rook-ceph.cephfs.csi.ceph.com   Delete          Immediate           true                   2m44s
```



**3.Deployment多副本+ PersistentVolumeClaim**

```yaml
vim nginx-deploy-cephfs.yaml
-------------------------------


apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deploy-cephfs
  name: nginx-deploy-cephfs
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deploy-cephfs
  template:
    metadata:
      labels:
        app: nginx-deploy-cephfs
    spec:
      containers:
      - image: nginx
        name: nginx
        volumeMounts:
        - name: data
          mountPath: /usr/share/nginx/html
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: nginx-cephfs-pvc
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-cephfs-pvc
  namespace: dev
spec:
  storageClassName: "rook-cephfs"
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```

```shell
[root@k8s-master test]# kubectl apply -f nginx-deploy-cephfs.yaml 
deployment.apps/nginx-deploy-cephfs created
persistentvolumeclaim/nginx-cephfs-pvc created


[root@k8s-master ~]# kubectl get pvc -n dev
NAME               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nginx-cephfs-pvc   Bound    pvc-d3f0307a-5d70-4d7e-aa3f-34897f1f61ac   1Gi        RWX            rook-cephfs    3m30s

[root@k8s-master ~]# kubectl get pv -n dev
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   REASON   AGE
pvc-d3f0307a-5d70-4d7e-aa3f-34897f1f61ac   1Gi        RWX            Delete           Bound    dev/nginx-cephfs-pvc   rook-cephfs             3m33s


[root@k8s-master test]# kubectl get all -n dev
NAME                                       READY   STATUS    RESTARTS   AGE
pod/nginx-deploy-cephfs-7f58cd5447-642s2   1/1     Running   0          3m17s
pod/nginx-deploy-cephfs-7f58cd5447-qmpkb   1/1     Running   0          3m17s
pod/nginx-deploy-cephfs-7f58cd5447-rhj8h   1/1     Running   0          3m17s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy-cephfs   3/3     3            3           3m17s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deploy-cephfs-7f58cd5447   3         3         3       3m17s


# 查看挂载
[root@k8s-master test]# kubectl exec -it nginx-deploy-cephfs-7f58cd5447-bl69d -n dev -- sh
# df -Th
Filesystem                                                                                                                                               Type     Size  Used Avail Use% Mounted on
overlay                                                                                                                                                  overlay   50G  9.7G   41G  20% /
tmpfs                                                                                                                                                    tmpfs     64M     0   64M   0% /dev
tmpfs                                                                                                                                                    tmpfs    7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/mapper/centos-root                                                                                                                                  xfs       50G  9.7G   41G  20% /etc/hosts
shm                                                                                                                                                      tmpfs     64M     0   64M   0% /dev/shm

10.97.103.24:6789,10.109.133.209:6789,10.104.142.159:6789:/volumes/csi/csi-vol-481bb8b1-480d-11ec-9b57-4ecbcb966fa5/dce62825-dbc6-4e38-a136-ec6312eea49b ceph     1.0G     0  1.0G   0% /usr/share/nginx/html
tmpfs                                                                                                                                                    tmpfs    7.8G   12K  7.8G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs                                                                                                                                                    tmpfs    7.8G     0  7.8G   0% /proc/acpi
tmpfs                                                                                                                                                    tmpfs    7.8G     0  7.8G   0% /proc/scsi
tmpfs                                                                                                                                                    tmpfs    7.8G     0  7.8G   0% /sys/firmware


# df
Filesystem                                                                                                                                               1K-blocks     Used Available Use% Mounted on
overlay                                                                                                                                                   52403200 10077548  42325652  20% /
tmpfs                                                                                                                                                        65536        0     65536   0% /dev
tmpfs                                                                                                                                                      8123648        0   8123648   0% /sys/fs/cgroup
/dev/mapper/centos-root                                                                                                                                   52403200 10077548  42325652  20% /etc/hosts
shm                                                                                                                                                          65536        0     65536   0% /dev/shm
10.97.103.24:6789,10.109.133.209:6789,10.104.142.159:6789:/volumes/csi/csi-vol-481bb8b1-480d-11ec-9b57-4ecbcb966fa5/dce62825-dbc6-4e38-a136-ec6312eea49b   1048576        0   1048576   0% /usr/share/nginx/html
tmpfs                                                                                                                                                      8123648       12   8123636   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs                                                                                                                                                      8123648        0   8123648   0% /proc/acpi
tmpfs                                                                                                                                                      8123648        0   8123648   0% /proc/scsi
tmpfs                                                                                                                                                      8123648        0   8123648   0% /sys/firmware
```

![](/images/kubernetes/storage/rook-14.png)

```shell
# 测试
[root@k8s-master test]# kubectl get pod -n dev -owide
NAME                                   READY   STATUS    RESTARTS   AGE     IP             NODE        NOMINATED NODE   READINESS GATES
nginx-deploy-cephfs-7f58cd5447-642s2   1/1     Running   0          4m53s   10.244.36.67   k8s-node1   <none>           <none>
nginx-deploy-cephfs-7f58cd5447-qmpkb   1/1     Running   0          4m53s   10.244.36.81   k8s-node1   <none>           <none>
nginx-deploy-cephfs-7f58cd5447-rhj8h   1/1     Running   0          4m53s   10.244.36.66   k8s-node1   <none>           <none>


[root@k8s-master test]# kubectl -n dev exec -it nginx-deploy-cephfs-7f58cd5447-642s2 -- bash 
root@nginx-deploy-cephfs-7f58cd5447-642s2:/# echo "Hello  Nginx Rook-cephfs" > /usr/share/nginx/html/index.html
root@nginx-deploy-cephfs-7f58cd5447-642s2:/# cat /usr/share/nginx/html/index.html
Hello  Nginx Rook-cephfs


root@nginx-deploy-cephfs-7f58cd5447-642s2:/# curl 10.244.36.66
Hello  Nginx Rook-cephfs
root@nginx-deploy-cephfs-7f58cd5447-642s2:/# curl 10.244.36.81
Hello  Nginx Rook-cephfs
```

```shell
[root@k8s-master ~]# kubectl get pvc -n dev
NAME               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nginx-cephfs-pvc   Bound    pvc-d3f0307a-5d70-4d7e-aa3f-34897f1f61ac   1Gi        RWX            rook-cephfs    9m11s
[root@k8s-master ~]# kubectl get pvc -n dev
NAME               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nginx-cephfs-pvc   Bound    pvc-d3f0307a-5d70-4d7e-aa3f-34897f1f61ac   1Gi        RWX            rook-cephfs    9m14s


# 删除
[root@k8s-master test]# kubectl delete -f nginx-deploy-cephfs.yaml 
deployment.apps "nginx-deploy-cephfs" deleted
persistentvolumeclaim "nginx-cephfs-pvc" deleted

# pvc、pv自动删除
[root@k8s-master ~]# kubectl get pvc -n dev
No resources found in dev namespace.
[root@k8s-master ~]# kubectl get pv -n dev
No resources found
```



### 5.块存储（RBD）

RBD即RADOS Block Device的简称，RBD块存储是最稳定且最常用的存储类型。RBD块设备类似磁盘可以被挂载。 RBD块设备具有快照、多副本、克隆和一致性等特性，数据以条带化的方式存储在Ceph集群的多个OSD中。如下是对Ceph RBD的理解。

RBD 就是 Ceph 里的块设备，一个 4T 的块设备的功能和一个 4T 的 SATA 类似，挂载的 RBD 就可以当磁盘用；
**resizable：**这个块可大可小；
**data striped：**这个块在Ceph里面是被切割成若干小块来保存，不然 1PB 的块怎么存的下；
**thin-provisioned：**精简置备，1TB 的集群是能创建无数 1PB 的块的。其实就是块的大小和在 Ceph 中实际占用大小是没有关系的，刚创建出来的块是不占空间，今后用多大空间，才会在 Ceph 中占用多大空间。举例：你有一个 32G 的 U盘，存了一个2G的电影，那么 RBD 大小就类似于 32G，而 2G 就相当于在 Ceph 中占用的空间 ；

块存储本质就是将裸磁盘或类似裸磁盘(lvm)设备映射给主机使用，主机可以对其进行格式化并存储和读取数据，块设备读取速度快但是不支持共享。
ceph可以通过内核模块和librbd库提供块设备支持。客户端可以通过内核模块挂在rbd使用，客户端使用rbd块设备就像使用普通硬盘一样，可以对其就行格式化然后使用；客户应用也可以通过librbd使用ceph块，典型的是云平台的块存储服务（如下图），云平台可以使用rbd作为云的存储后端提供镜像存储、volume块或者客户的系统引导盘等。

```shell
使用场景： 云平台（OpenStack做为云的存储后端提供镜像存储）
K8s容器
map成块设备直接使用 ISCIS
安装Ceph客户端
```





### 6.文件存储（CephFS）

ceph文件系统提供了任何大小的符合posix标准的分布式文件系统，它使用Ceph RADOS存储数据。要实现ceph文件系统，需要一个正在运行的ceph存储集群和至少一个ceph元数据服务器(MDS)来管理其元数据并使其与数据分离，这有助于降低复杂性和提高可靠性。

libcephfs库在支持其多个客户机实现方面发挥重要作用。它具有本机linux内核驱动程序支持，因此客户机可以使用本机文件系统安装，例如使用mount命令。她与samba紧密集成，支持CIFS和SMB。Ceph FS使用cephfuse模块扩展到用户空间(FUSE)中的文件系统。它还允许使用libcephfs库与RADOS集群进行直接的应用程序交互。
只有Ceph FS才需要Ceph MDS，其他存储方法的块和基于对象的存储不需要MDS。Ceph MDS作为一个守护进程运行，它允许客户机挂载任意大小的POSIX文件系统。MDS不直接向客户端提供任何数据，数据服务仅OSD完成。



**CephFS用于为RADOS存储集群提供一个POSIX兼容的文件系统接口**

- 基于RADOS存储集群将数据与元数据IO进行解耦
- 动态探测和迁移元数据负载到其它MDS，实现了对元数据IO的扩展
- 第一个稳定版随Jewel版本释出
- 自Luminous版本起支持多活MDS（Multiple Active MDS）

**特性：**

- 目录分片
- 动态子树分区和子树绑定（静态子树分区）
- 支持内核及FUSE客户端
- 其它尚未稳定特性还包括内联数据（INLINEDATA）、快照和多文件系统等





## 三、实践



### 1.块存储（RBD）

**块存储（RBD）：**适用StatefulSet，每个Pod有自己的存储，相当于一块磁盘；



#### 1.1.官网参考

Ceph Storage -> Block Storage

地址：https://www.rook.io/docs/rook/v1.6/ceph-block.html



**1.StorageClass定义**

官网 storageclass.yaml

```yaml
apiVersion: ceph.rook.io/v1
kind: CephBlockPool
metadata:
  name: replicapool
  namespace: rook-ceph
spec:
  failureDomain: host
  replicated:
    size: 3
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: rook-ceph-block
# Change "rook-ceph" provisioner prefix to match the operator namespace if needed
provisioner: rook-ceph.rbd.csi.ceph.com
parameters:
    # clusterID is the namespace where the rook cluster is running
    clusterID: rook-ceph
    # Ceph pool into which the RBD image shall be created
    pool: replicapool

    # (optional) mapOptions is a comma-separated list of map options.
    # For krbd options refer
    # https://docs.ceph.com/docs/master/man/8/rbd/#kernel-rbd-krbd-options
    # For nbd options refer
    # https://docs.ceph.com/docs/master/man/8/rbd-nbd/#options
    # mapOptions: lock_on_read,queue_depth=1024

    # (optional) unmapOptions is a comma-separated list of unmap options.
    # For krbd options refer
    # https://docs.ceph.com/docs/master/man/8/rbd/#kernel-rbd-krbd-options
    # For nbd options refer
    # https://docs.ceph.com/docs/master/man/8/rbd-nbd/#options
    # unmapOptions: force

    # RBD image format. Defaults to "2".
    imageFormat: "2"

    # RBD image features. Available for imageFormat: "2". CSI RBD currently supports only `layering` feature.
    imageFeatures: layering

    # The secrets contain Ceph admin credentials.
    csi.storage.k8s.io/provisioner-secret-name: rook-csi-rbd-provisioner
    csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph
    csi.storage.k8s.io/controller-expand-secret-name: rook-csi-rbd-provisioner
    csi.storage.k8s.io/controller-expand-secret-namespace: rook-ceph
    csi.storage.k8s.io/node-stage-secret-name: rook-csi-rbd-node
    csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph

    # Specify the filesystem type of the volume. If not specified, csi-provisioner
    # will set default as `ext4`. Note that `xfs` is not recommended due to potential deadlock
    # in hyperconverged settings where the volume is mounted on the same node as the osds.
    csi.storage.k8s.io/fstype: ext4

# Delete the rbd volume when a PVC is deleted
reclaimPolicy: Delete
```



下载源码：/k8s/rook/rook/cluster/examples/kubernetes/ceph/csi/rbd/storageclass.yaml

```yaml
apiVersion: ceph.rook.io/v1
kind: CephBlockPool
metadata:
  name: replicapool
  namespace: rook-ceph
spec:
  failureDomain: host
  replicated:
    size: 3
    # Disallow setting pool with replica 1, this could lead to data loss without recovery.
    # Make sure you're *ABSOLUTELY CERTAIN* that is what you want
    requireSafeReplicaSize: true
    # gives a hint (%) to Ceph in terms of expected consumption of the total cluster capacity of a given pool
    # for more info: https://docs.ceph.com/docs/master/rados/operations/placement-groups/#specifying-expected-pool-size
    #targetSizeRatio: .5
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rook-ceph-block
# Change "rook-ceph" provisioner prefix to match the operator namespace if needed
provisioner: rook-ceph.rbd.csi.ceph.com
parameters:
  # clusterID is the namespace where the rook cluster is running
  # If you change this namespace, also change the namespace below where the secret namespaces are defined
  clusterID: rook-ceph # namespace:cluster

  # If you want to use erasure coded pool with RBD, you need to create
  # two pools. one erasure coded and one replicated.
  # You need to specify the replicated pool here in the `pool` parameter, it is
  # used for the metadata of the images.
  # The erasure coded pool must be set as the `dataPool` parameter below.
  #dataPool: ec-data-pool
  pool: replicapool

  # (optional) mapOptions is a comma-separated list of map options.
  # For krbd options refer
  # https://docs.ceph.com/docs/master/man/8/rbd/#kernel-rbd-krbd-options
  # For nbd options refer
  # https://docs.ceph.com/docs/master/man/8/rbd-nbd/#options
  # mapOptions: lock_on_read,queue_depth=1024

  # (optional) unmapOptions is a comma-separated list of unmap options.
  # For krbd options refer
  # https://docs.ceph.com/docs/master/man/8/rbd/#kernel-rbd-krbd-options
  # For nbd options refer
  # https://docs.ceph.com/docs/master/man/8/rbd-nbd/#options
  # unmapOptions: force

  # RBD image format. Defaults to "2".
  imageFormat: "2"

  # RBD image features. Available for imageFormat: "2". CSI RBD currently supports only `layering` feature.
  imageFeatures: layering

  # The secrets contain Ceph admin credentials. These are generated automatically by the operator
  # in the same namespace as the cluster.
  csi.storage.k8s.io/provisioner-secret-name: rook-csi-rbd-provisioner
  csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph # namespace:cluster
  csi.storage.k8s.io/controller-expand-secret-name: rook-csi-rbd-provisioner
  csi.storage.k8s.io/controller-expand-secret-namespace: rook-ceph # namespace:cluster
  csi.storage.k8s.io/node-stage-secret-name: rook-csi-rbd-node
  csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph # namespace:cluster
  # Specify the filesystem type of the volume. If not specified, csi-provisioner
  # will set default as `ext4`. Note that `xfs` is not recommended due to potential deadlock
  # in hyperconverged settings where the volume is mounted on the same node as the osds.
  csi.storage.k8s.io/fstype: ext4
# uncomment the following to use rbd-nbd as mounter on supported nodes
# **IMPORTANT**: If you are using rbd-nbd as the mounter, during upgrade you will be hit a ceph-csi
# issue that causes the mount to be disconnected. You will need to follow special upgrade steps
# to restart your application pods. Therefore, this option is not recommended.
#mounter: rbd-nbd
allowVolumeExpansion: true
reclaimPolicy: Delete
```

```shell
# 创建
kubectl create -f cluster/examples/kubernetes/ceph/csi/rbd/storageclass.yaml
```



**2.创建CephBlockPool和StorageClass**

```yaml
apiVersion: ceph.rook.io/v1
kind: CephBlockPool
metadata:
  name: replicapool
  namespace: rook-ceph
spec:
  failureDomain: host
  replicated:
    size: 3
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: rook-ceph-block
provisioner: ceph.rook.io/block
parameters:
  blockPool: replicapool
  # The value of "clusterNamespace" MUST be the same as the one in which your rook cluster exist
  clusterNamespace: rook-ceph
  # Specify the filesystem type of the volume. If not specified, it will use `ext4`.
  fstype: ext4
# Optional, default reclaimPolicy is "Delete". Other options are: "Retain", "Recycle" as documented in https://kubernetes.io/docs/concepts/storage/storage-classes/
reclaimPolicy: Retain
# Optional, if you want to add dynamic resize for PVC. Works for Kubernetes 1.14+
# For now only ext3, ext4, xfs resize support provided, like in Kubernetes itself.
allowVolumeExpansion: true
```

```shell
# 创建
kubectl create -f cluster/examples/kubernetes/ceph/flex/storageclass.yaml
```



#### 1.2.创建CephBlockPool和StorageClass

```yaml
vim storageclass.yaml
-------------------------------------------
 
apiVersion: ceph.rook.io/v1
kind: CephBlockPool
metadata:
  name: replicapool
  namespace: rook-ceph
spec:
  failureDomain: host # host级容灾
  replicated:
    size: 3           # 默认三个副本
    requireSafeReplicaSize: true  # 强制高可用，如果size为1则需改为false
 
---
apiVersion: storage.k8s.io/v1
kind: StorageClass                       # sc无需指定命名空间
metadata:
  name: rook-ceph-block
provisioner: rook-ceph.rbd.csi.ceph.com   # 存储驱动
parameters:
  clusterID: rook-ceph # namespace:cluster
  pool: replicapool                       # 关联到CephBlockPool
  imageFormat: "2"
  imageFeatures: layering
 
  csi.storage.k8s.io/provisioner-secret-name: rook-csi-rbd-provisioner
  csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph # namespace:cluster
  csi.storage.k8s.io/controller-expand-secret-name: rook-csi-rbd-provisioner
  csi.storage.k8s.io/controller-expand-secret-namespace: rook-ceph # namespace:cluster
  csi.storage.k8s.io/node-stage-secret-name: rook-csi-rbd-node
  csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph # namespace:cluster
  
  csi.storage.k8s.io/fstype: ext4   
 
allowVolumeExpansion: true               # 是否允许扩容
reclaimPolicy: Delete                    # PV回收策略
```



创建CephBlockPool和StorageClass

```shell
[root@k8s-master test]# kubectl apply -f storageclass.yaml 
cephblockpool.ceph.rook.io/replicapool created
storageclass.storage.k8s.io/rook-ceph-block created



# 查看sc
[root@k8s-master test]# kubectl get sc
NAME              PROVISIONER                  RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
rook-ceph-block   rook-ceph.rbd.csi.ceph.com   Delete          Immediate           true                   28s

 
# 查看CephBlockPool（也可在dashboard中查看）
[root@k8s-master test]# kubectl get cephblockpools -n rook-ceph
NAME          AGE
replicapool   2m41s
```

![](/images/kubernetes/storage/rook-15.png)



#### 1.3.StatefulSet

**StatefulSet，每个Pod有自己的存储，相当于一块磁盘，需要使用块存储。块存储不支持共享存储**

StatefulSet删除，需要手动删除PVC.

```yaml
vim nginx-ss-rbd.yaml
-------------------------------


apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-ss-rbd
  namespace: dev
spec:
  selector:
    matchLabels:
      app: nginx-ss-rbd 
  serviceName: "nginx"
  replicas: 3 
  template:
    metadata:
      labels:
        app: nginx-ss-rbd 
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "rook-ceph-block"
      resources:
        requests:
          storage: 2Gi
```

```shell
# 创建
[root@k8s-master test]# kubectl apply -f nginx-ss-rbd.yaml 
statefulset.apps/nginx-ss-rbd created


# 查看
[root@k8s-master test]# kubectl get all -n dev
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deploy-rbd-7f468884cf-rfzpk   1/1     Running   0          14m
pod/nginx-ss-rbd-0                      1/1     Running   0          112s
pod/nginx-ss-rbd-1                      1/1     Running   0          89s
pod/nginx-ss-rbd-2                      1/1     Running   0          60s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy-rbd   1/1     1            1           14m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deploy-rbd-7f468884cf   1         1         1       14m

NAME                            READY   AGE
statefulset.apps/nginx-ss-rbd   3/3     112s


[root@k8s-master ~]# kubectl get pvc -n dev
NAME                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
www-nginx-ss-rbd-0   Bound    pvc-2f1a8b03-9bff-4769-8f48-16a91ccfcaa8   2Gi        RWO            rook-ceph-block   19m
www-nginx-ss-rbd-1   Bound    pvc-2db380dc-d52c-470c-807f-5829eb5694ab   2Gi        RWO            rook-ceph-block   19m
www-nginx-ss-rbd-2   Bound    pvc-cbd3478d-9bf0-4607-9212-04ef5391feb7   2Gi        RWO            rook-ceph-block   18m

[root@k8s-master ~]# kubectl get pv -n dev
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS      REASON   AGE
pvc-2db380dc-d52c-470c-807f-5829eb5694ab   2Gi        RWO            Delete           Bound    dev/www-nginx-ss-rbd-1   rook-ceph-block            20m
pvc-2f1a8b03-9bff-4769-8f48-16a91ccfcaa8   2Gi        RWO            Delete           Bound    dev/www-nginx-ss-rbd-0   rook-ceph-block            20m
pvc-cbd3478d-9bf0-4607-9212-04ef5391feb7   2Gi        RWO            Delete           Bound    dev/www-nginx-ss-rbd-2   rook-ceph-block            19m


# 查看磁盘挂载
[root@k8s-master test]# kubectl exec -it nginx-ss-rbd-0 -n dev -- bash
root@nginx-ss-rbd-0:/# 
root@nginx-ss-rbd-0:/# df
Filesystem              1K-blocks     Used Available Use% Mounted on
overlay                  52403200 10093996  42309204  20% /
tmpfs                       65536        0     65536   0% /dev
tmpfs                     8123648        0   8123648   0% /sys/fs/cgroup
/dev/mapper/centos-root  52403200 10093996  42309204  20% /etc/hosts
shm                         65536        0     65536   0% /dev/shm
/dev/rbd0                 1998672     6144   1976144   1% /usr/share/nginx/html
tmpfs                     8123648       12   8123636   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs                     8123648        0   8123648   0% /proc/acpi
tmpfs                     8123648        0   8123648   0% /proc/scsi
tmpfs                     8123648        0   8123648   0% /sys/firmware
root@nginx-ss-rbd-0:/# 
root@nginx-ss-rbd-0:/# df -Th
Filesystem              Type     Size  Used Avail Use% Mounted on
overlay                 overlay   50G  9.7G   41G  20% /
tmpfs                   tmpfs     64M     0   64M   0% /dev
tmpfs                   tmpfs    7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/mapper/centos-root xfs       50G  9.7G   41G  20% /etc/hosts
shm                     tmpfs     64M     0   64M   0% /dev/shm
/dev/rbd0               ext4     2.0G  6.0M  1.9G   1% /usr/share/nginx/html
tmpfs                   tmpfs    7.8G   12K  7.8G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs                   tmpfs    7.8G     0  7.8G   0% /proc/acpi
tmpfs                   tmpfs    7.8G     0  7.8G   0% /proc/scsi
tmpfs                   tmpfs    7.8G     0  7.8G   0% /sys/firmware
```

![](/images/kubernetes/storage/rook-13.png)

```shell
# 测试
[root@k8s-master test]# kubectl get pod -n dev -owide | grep ss
nginx-ss-rbd-0                      1/1     Running   0          8m44s   10.244.36.109   k8s-node1   <none>           <none>
nginx-ss-rbd-1                      1/1     Running   0          8m21s   10.244.36.77    k8s-node1   <none>           <none>
nginx-ss-rbd-2                      1/1     Running   0          7m52s   10.244.36.108   k8s-node1   <none>           <none>


[root@k8s-master test]# kubectl -n dev exec -it nginx-ss-rbd-0 -- bash
root@nginx-ss-rbd-0:/# echo "Hello  Nginx Rook-block-0" > /usr/share/nginx/html/index.html
root@nginx-ss-rbd-0:/# cat /usr/share/nginx/html/index.html
Hello  Nginx Rook-block-0

[root@k8s-master test]# kubectl -n dev exec -it nginx-ss-rbd-1 -- bash
root@nginx-ss-rbd-1:/# echo "Hello  Nginx Rook-block-1" > /usr/share/nginx/html/index.html
root@nginx-ss-rbd-1:/#  cat /usr/share/nginx/html/index.html
Hello  Nginx Rook-block-1

[root@k8s-master test]# kubectl -n dev exec -it nginx-ss-rbd-2 -- bash
root@nginx-ss-rbd-2:/# echo "Hello  Nginx Rook-block-2" > /usr/share/nginx/html/index.html
root@nginx-ss-rbd-2:/# cat /usr/share/nginx/html/index.html
Hello  Nginx Rook-block-2


[root@k8s-master test]# curl 10.244.36.109
Hello  Nginx Rook-block-0
[root@k8s-master test]# curl 10.244.36.77
Hello  Nginx Rook-block-1
[root@k8s-master test]# curl 10.244.36.108
Hello  Nginx Rook-block-2


# 删除Pod，Pod重建后，数据还在
[root@k8s-master test]# kubectl get pod -n dev -owide | grep ss
nginx-ss-rbd-0                      1/1     Running   0          15m   10.244.36.109   k8s-node1   <none>           <none>
nginx-ss-rbd-1                      1/1     Running   0          14m   10.244.36.77    k8s-node1   <none>           <none>
nginx-ss-rbd-2                      1/1     Running   0          14m   10.244.36.108   k8s-node1   <none>           <none>

[root@k8s-master test]# kubectl delete pod nginx-ss-rbd-0 -n dev
pod "nginx-ss-rbd-0" deleted
[root@k8s-master test]# kubectl delete pod nginx-ss-rbd-1 -n dev
pod "nginx-ss-rbd-1" deleted
[root@k8s-master test]# kubectl delete pod nginx-ss-rbd-2 -n dev
pod "nginx-ss-rbd-2" deleted

[root@k8s-master test]# kubectl get pod -n dev -owide | grep ss
nginx-ss-rbd-0                      1/1     Running   0          56s   10.244.36.111   k8s-node1   <none>           <none>
nginx-ss-rbd-1                      1/1     Running   0          45s   10.244.36.112   k8s-node1   <none>           <none>
nginx-ss-rbd-2                      1/1     Running   0          20s   10.244.36.114   k8s-node1   <none>           <none>

[root@k8s-master test]# curl 10.244.36.111
Hello  Nginx Rook-block-0
[root@k8s-master test]# curl 10.244.36.112
Hello  Nginx Rook-block-1
[root@k8s-master test]# curl 10.244.36.114
Hello  Nginx Rook-block-2

# 删除之后，pvc、pv都存在
[root@k8s-master test]# kubectl delete -f nginx-deploy-rbd.yaml 
deployment.apps "nginx-deploy-rbd" deleted
persistentvolumeclaim "nginx-rbd-pvc" deleted

[root@k8s-master ~]# kubectl get pv -n dev
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS      REASON   AGE
pvc-2db380dc-d52c-470c-807f-5829eb5694ab   2Gi        RWO            Delete           Bound    dev/www-nginx-ss-rbd-1   rook-ceph-block            34m
pvc-2f1a8b03-9bff-4769-8f48-16a91ccfcaa8   2Gi        RWO            Delete           Bound    dev/www-nginx-ss-rbd-0   rook-ceph-block            34m
pvc-cbd3478d-9bf0-4607-9212-04ef5391feb7   2Gi        RWO            Delete           Bound    dev/www-nginx-ss-rbd-2   rook-ceph-block            34m

[root@k8s-master ~]# kubectl get pvc -n dev
NAME                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
www-nginx-ss-rbd-0   Bound    pvc-2f1a8b03-9bff-4769-8f48-16a91ccfcaa8   2Gi        RWO            rook-ceph-block   35m
www-nginx-ss-rbd-1   Bound    pvc-2db380dc-d52c-470c-807f-5829eb5694ab   2Gi        RWO            rook-ceph-block   34m
www-nginx-ss-rbd-2   Bound    pvc-cbd3478d-9bf0-4607-9212-04ef5391feb7   2Gi        RWO            rook-ceph-block   34m


# 再次重建数据还在
[root@k8s-master test]# kubectl apply -f nginx-ss-rbd.yaml 
statefulset.apps/nginx-ss-rbd created

[root@k8s-master test]#  kubectl get pod -n dev -owide | grep ss
nginx-ss-rbd-0   1/1     Running   0          89s   10.244.36.116   k8s-node1   <none>           <none>
nginx-ss-rbd-1   1/1     Running   0          63s   10.244.36.106   k8s-node1   <none>           <none>
nginx-ss-rbd-2   1/1     Running   0          30s   10.244.36.122   k8s-node1   <none>           <none>
 
[root@k8s-master test]# curl 10.244.36.116
Hello  Nginx Rook-block-0
[root@k8s-master test]# curl 10.244.36.106
Hello  Nginx Rook-block-1
[root@k8s-master test]# curl 10.244.36.122
Hello  Nginx Rook-block-2


# 删除StatefulSet，需要手动删除pvc
```



#### 1.4.PVC动态申请存储

pvc是存储卷类型的资源、它通过申请占用某个pv而创建，它于pv是一对一的关系、用户无需关心底层实现细节。申请时、用户只需指定目标空间的大小、访问模式、PV标签选择器和STORAGECLASS等相关信息即可。

```shell
# pvc的spec字段的可嵌套字段具体如下：


[root@master chapter7]# kubectl explain pvc.spec
KIND:     PersistentVolumeClaim
VERSION:  v1

RESOURCE: spec <Object>

DESCRIPTION:
     Spec defines the desired characteristics of a volume requested by a pod
     author. More info:
     https://kubernetes.io/docs/concepts/storage/persistent-volumes#persistentvolumeclaims

     PersistentVolumeClaimSpec describes the common attributes of storage
     devices and allows a Source for provider-specific attributes

FIELDS:
   accessModes	<[]string>
   #PVC也可以设置访问模式，用于描述用户应用对存储资源的访问权限。其三种访问模式的设置与PV的设置相同。
     AccessModes contains the desired access modes the volume should have. More
     info:
     https://kubernetes.io/docs/concepts/storage/persistent-volumes#access-modes-1

   dataSource	<Object>
     This field can be used to specify either: * An existing VolumeSnapshot
     object (snapshot.storage.k8s.io/VolumeSnapshot - Beta) * An existing PVC
     (PersistentVolumeClaim) * An existing custom resource/object that
     implements data population (Alpha) In order to use VolumeSnapshot object
     types, the appropriate feature gate must be enabled
     (VolumeSnapshotDataSource or AnyVolumeDataSource) If the provisioner or an
     external controller can support the specified data source, it will create a
     new volume based on the contents of the specified data source. If the
     specified data source is not supported, the volume will not be created and
     the failure will be reported as an event. In the future, we plan to support
     more data source types and the behavior of the provisioner may change.

   resources	<Object>
   #描述对存储资源的请求，目前仅支持request.storage的设置，即存储空间大小
     Resources represents the minimum resources the volume should have. More
     info:
     https://kubernetes.io/docs/concepts/storage/persistent-volumes#resources

   selector	<Object>
   #通过对Label  Selector的设置，可使PVC对于系统中已存在的各种PV进行筛选。系统将根据标签选出合适的PV与该PVC进行绑定。选择条件可以使用matchLabels和matchExpressions进行设置，如果两个字段都设置了，则Selector的逻辑将是两组条件同时满足才能完成匹配。
     A label query over volumes to consider for binding.

   storageClassName	<string>
     Name of the StorageClass required by the claim. More info:
     https://kubernetes.io/docs/concepts/storage/persistent-volumes#class-1

   volumeMode	<string>
   #PVC也可以设置存储卷模式，用于描述希望使用的PV存储卷模式，包括文件系统和块设备。
     volumeMode defines what type of volume is required by the claim. Value of
     Filesystem is implied when not included in claim spec.

   volumeName	<string>
   #用于直接指定要绑定的pv的卷名
     VolumeName is the binding reference to the PersistentVolume backing this
     claim.
```



**1.PVC动态申请存储**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-rdb
  namespace: dev
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: rook-ceph-block
  resources:
    requests:
      storage: 1Gi
```

```shell
# 创建pvc，自动创建pv
[root@k8s-master test]# kubectl apply -f pvc-rdb.yml 
persistentvolumeclaim/pvc-rdb created


[root@k8s-master flex]# kubectl get pvc -n dev
NAME                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
pvc-rdb              Bound    pvc-feb4b298-ccf3-4a0c-b0c7-26dda1fa6f56   1Gi        RWO            rook-ceph-block   8s

[root@k8s-master flex]# kubectl get pv -n dev
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS      REASON   AGE
pvc-feb4b298-ccf3-4a0c-b0c7-26dda1fa6f56   1Gi        RWO            Delete           Bound    dev/pvc-rdb              rook-ceph-block            14s
```



**2.使用PVC**

在POD资源中调用PVC资源、只需要在定义volumes时使用persistentVolumeClaim字段嵌套指定两个字段即可、具体如下

```yaml
[root@master chapter7]# kubectl explain pod.spec.volumes.persistentVolumeClaim
KIND:     Pod
VERSION:  v1

RESOURCE: persistentVolumeClaim <Object>

DESCRIPTION:
     PersistentVolumeClaimVolumeSource represents a reference to a
     PersistentVolumeClaim in the same namespace. More info:
     https://kubernetes.io/docs/concepts/storage/persistent-volumes#persistentvolumeclaims

     PersistentVolumeClaimVolumeSource references the user's PVC in the same
     namespace. This volume finds the bound PV and mounts that volume for the
     pod. A PersistentVolumeClaimVolumeSource is, essentially, a wrapper around
     another type of volume that is owned by someone else (the system).

FIELDS:
   claimName	<string> -required-
   #需要调用PVC存储卷的名称、PVC卷要与pod在通一个名称空间中
     ClaimName is the name of a PersistentVolumeClaim in the same namespace as
     the pod using this volume. More info:
     https://kubernetes.io/docs/concepts/storage/persistent-volumes#persistentvolumeclaims

   readOnly	<boolean>
   #是否将存储卷强制挂在为指读模式、默认为false
     Will force the ReadOnly setting in VolumeMounts. Default false.
```



```shell
apiVersion: v1
kind: Pod
metadata:
  name: pod-pvc-vol
  namespace: default
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html/
  volumes:
  - name: html
    persistentVoumeClaim:
      claimName: pvc-rdb
```



### 2.共享文件存储（CephFS）

**文件存储（CephFS）：**适用Deployment，多个Pod文件共享；



#### 2.1.官网参考

Ceph Storage -> Shared Filesystem

地址：https://www.rook.io/docs/rook/v1.6/ceph-filesystem.html



创建Cephfs文件系统需要先部署MDS服务，该服务负责处理文件系统中的元数据。

**1.filesystem定义**

官网 filesystem.yaml

```yaml
apiVersion: ceph.rook.io/v1
kind: CephFilesystem
metadata:
  name: myfs
  namespace: rook-ceph
spec:
  metadataPool:
    replicated:
      size: 3
  dataPools:
    - replicated:
        size: 3
  preserveFilesystemOnDelete: true
  metadataServer:
    activeCount: 1
    activeStandby: true
```



下载源码：/k8s/rook//rook/cluster/examples/kubernetes/ceph/filesystem.yaml

```yaml
#################################################################################################################
# Create a filesystem with settings with replication enabled for a production environment.
# A minimum of 3 OSDs on different nodes are required in this example.
#  kubectl create -f filesystem.yaml
#################################################################################################################

apiVersion: ceph.rook.io/v1
kind: CephFilesystem
metadata:
  name: myfs
  namespace: rook-ceph # namespace:cluster
spec:
  # The metadata pool spec. Must use replication.
  metadataPool:
    replicated:
      size: 3
      requireSafeReplicaSize: true
    parameters:
      # Inline compression mode for the data pool
      # Further reference: https://docs.ceph.com/docs/nautilus/rados/configuration/bluestore-config-ref/#inline-compression
      compression_mode:
        none
        # gives a hint (%) to Ceph in terms of expected consumption of the total cluster capacity of a given pool
      # for more info: https://docs.ceph.com/docs/master/rados/operations/placement-groups/#specifying-expected-pool-size
      #target_size_ratio: ".5"
  # The list of data pool specs. Can use replication or erasure coding.
  dataPools:
    - failureDomain: host
      replicated:
        size: 3
        # Disallow setting pool with replica 1, this could lead to data loss without recovery.
        # Make sure you're *ABSOLUTELY CERTAIN* that is what you want
        requireSafeReplicaSize: true
      parameters:
        # Inline compression mode for the data pool
        # Further reference: https://docs.ceph.com/docs/nautilus/rados/configuration/bluestore-config-ref/#inline-compression
        compression_mode:
          none
          # gives a hint (%) to Ceph in terms of expected consumption of the total cluster capacity of a given pool
        # for more info: https://docs.ceph.com/docs/master/rados/operations/placement-groups/#specifying-expected-pool-size
        #target_size_ratio: ".5"
  # Whether to preserve filesystem after CephFilesystem CRD deletion
  preserveFilesystemOnDelete: true
  # The metadata service (mds) configuration
  metadataServer:
    # The number of active MDS instances
    activeCount: 1
    # Whether each active MDS instance will have an active standby with a warm metadata cache for faster failover.
    # If false, standbys will be available, but will not have a warm cache.
    activeStandby: true
    # The affinity rules to apply to the mds deployment
    placement:
      #  nodeAffinity:
      #    requiredDuringSchedulingIgnoredDuringExecution:
      #      nodeSelectorTerms:
      #      - matchExpressions:
      #        - key: role
      #          operator: In
      #          values:
      #          - mds-node
      #  topologySpreadConstraints:
      #  tolerations:
      #  - key: mds-node
      #    operator: Exists
      #  podAffinity:
      podAntiAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
                - key: app
                  operator: In
                  values:
                    - rook-ceph-mds
            # topologyKey: kubernetes.io/hostname will place MDS across different hosts
            topologyKey: kubernetes.io/hostname
        preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - rook-ceph-mds
              # topologyKey: */zone can be used to spread MDS across different AZ
              # Use <topologyKey: failure-domain.beta.kubernetes.io/zone> in k8s cluster if your cluster is v1.16 or lower
              # Use <topologyKey: topology.kubernetes.io/zone>  in k8s cluster is v1.17 or upper
              topologyKey: topology.kubernetes.io/zone
    # A key/value list of annotations
    annotations:
    #  key: value
    # A key/value list of labels
    labels:
    #  key: value
    resources:
    # The requests and limits set here, allow the filesystem MDS Pod(s) to use half of one CPU core and 1 gigabyte of memory
    #  limits:
    #    cpu: "500m"
    #    memory: "1024Mi"
    #  requests:
    #    cpu: "500m"
    #    memory: "1024Mi"
    # priorityClassName: my-priority-class
  mirroring:
    enabled: false
```

```shell
# 创建
kubectl create -f filesystem.yaml
```



**2.创建StorageClass**

官网 storageclass.yaml 

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rook-cephfs
# Change "rook-ceph" provisioner prefix to match the operator namespace if needed
provisioner: rook-ceph.cephfs.csi.ceph.com
parameters:
  # clusterID is the namespace where operator is deployed.
  clusterID: rook-ceph

  # CephFS filesystem name into which the volume shall be created
  fsName: myfs

  # Ceph pool into which the volume shall be created
  # Required for provisionVolume: "true"
  pool: myfs-data0

  # The secrets contain Ceph admin credentials. These are generated automatically by the operator
  # in the same namespace as the cluster.
  csi.storage.k8s.io/provisioner-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph
  csi.storage.k8s.io/controller-expand-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/controller-expand-secret-namespace: rook-ceph
  csi.storage.k8s.io/node-stage-secret-name: rook-csi-cephfs-node
  csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph

reclaimPolicy: Delete
```



下载源码：/k8s/rook/rook/cluster/examples/kubernetes/ceph/csi/cephfs/storageclass.yaml

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rook-cephfs
provisioner: rook-ceph.cephfs.csi.ceph.com # driver:namespace:operator
parameters:
  # clusterID is the namespace where operator is deployed.
  clusterID: rook-ceph # namespace:cluster

  # CephFS filesystem name into which the volume shall be created
  fsName: myfs

  # Ceph pool into which the volume shall be created
  # Required for provisionVolume: "true"
  pool: myfs-data0

  # The secrets contain Ceph admin credentials. These are generated automatically by the operator
  # in the same namespace as the cluster.
  csi.storage.k8s.io/provisioner-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph # namespace:cluster
  csi.storage.k8s.io/controller-expand-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/controller-expand-secret-namespace: rook-ceph # namespace:cluster
  csi.storage.k8s.io/node-stage-secret-name: rook-csi-cephfs-node
  csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph # namespace:cluster

  # (optional) The driver can use either ceph-fuse (fuse) or ceph kernel client (kernel)
  # If omitted, default volume mounter will be used - this is determined by probing for ceph-fuse
  # or by setting the default mounter explicitly via --volumemounter command-line argument.
  # mounter: kernel
reclaimPolicy: Delete
allowVolumeExpansion: true
mountOptions:
  # uncomment the following line for debugging
  #- debug
```

```shell
# 创建
kubectl create -f cluster/examples/kubernetes/ceph/csi/cephfs/storageclass.yaml
```



#### 2.2.创建filesystem和StorageClass

创建Cephfs文件系统需要先部署MDS服务，该服务负责处理文件系统中的元数据。



**1.部署MDS**

```yaml
vim filesystem.yaml
---------------------------------------


apiVersion: ceph.rook.io/v1
kind: CephFilesystem
metadata:
  name: myfs
  namespace: rook-ceph 
spec:
  metadataPool:
    replicated:
      size: 3                         # 元数据副本数
      requireSafeReplicaSize: true
    parameters:
      compression_mode:
        none
  dataPools:
    - failureDomain: host
      replicated:
        size: 3                     # 存储数据的副本数
        requireSafeReplicaSize: true
      parameters:
        compression_mode:
          none
  preserveFilesystemOnDelete: true
  metadataServer:
    activeCount: 3                # MDS实例的副本数，默认1，生产环境建议设置为3
```

```shell
[root@k8s-master test]# kubectl apply -f filesystem.yaml 
cephfilesystem.ceph.rook.io/myfs created


[root@k8s-master ~]# kubectl get CephFilesystem -n rook-ceph
NAME   ACTIVEMDS   AGE    PHASE
myfs   3           150m   Ready

[root@k8s-master ~]# kubectl -n rook-ceph get pod -l app=rook-ceph-mds
NAME                                    READY   STATUS    RESTARTS   AGE
rook-ceph-mds-myfs-a-7f65bc58fc-fk8mr   1/1     Running   0          4h3m
rook-ceph-mds-myfs-b-57c67856c5-95mgk   1/1     Running   0          4h3m
rook-ceph-mds-myfs-c-6bdf97c5d9-t2qm6   1/1     Running   0          4h3m
rook-ceph-mds-myfs-d-65fc96f959-l6zmh   1/1     Running   0          4h3m
rook-ceph-mds-myfs-e-799998bcd9-fhmtf   1/1     Running   0          4h3m
rook-ceph-mds-myfs-f-5d964dcb7f-qnfz2   1/1     Running   0          4h3m


[root@rook-ceph-tools-6bc7c4f9fc-p5j59 /]# ceph status
...
  services:
    mds: myfs:3 {0=myfs-b=up:active,1=myfs-a=up:active,2=myfs-e=up:active} 3 up:standby
    ...
```



**2.创建StorageClass**

**直接部署`/k8s/rook/rook/cluster/examples/kubernetes/ceph/csi/cephfs/storageclass.yaml`**

```yaml
vim storageclass.yaml 
----------------------------------


apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rook-cephfs
provisioner: rook-ceph.cephfs.csi.ceph.com # driver:namespace:operator
parameters:
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rook-cephfs
provisioner: rook-ceph.cephfs.csi.ceph.com # driver:namespace:operator
parameters:
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rook-cephfs
provisioner: rook-ceph.cephfs.csi.ceph.com # driver:namespace:operator
parameters:
  # clusterID is the namespace where operator is deployed.
  clusterID: rook-ceph # namespace:cluster

  # CephFS filesystem name into which the volume shall be created
  fsName: myfs

  # Ceph pool into which the volume shall be created
  # Required for provisionVolume: "true"
  pool: myfs-data0

  # The secrets contain Ceph admin credentials. These are generated automatically by the operator
  # in the same namespace as the cluster.
  csi.storage.k8s.io/provisioner-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph # namespace:cluster
  csi.storage.k8s.io/controller-expand-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/controller-expand-secret-namespace: rook-ceph # namespace:cluster
  csi.storage.k8s.io/node-stage-secret-name: rook-csi-cephfs-node
  csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph # namespace:cluster

  # (optional) The driver can use either ceph-fuse (fuse) or ceph kernel client (kernel)
  # If omitted, default volume mounter will be used - this is determined by probing for ceph-fuse
  # or by setting the default mounter explicitly via --volumemounter command-line argument.
  # mounter: kernel
reclaimPolicy: Delete
allowVolumeExpansion: true
mountOptions:
  # uncomment the following line for debugging
  #- debug
```



```shell
[root@k8s-master cephfs]# cd /k8s/rook/rook/cluster/examples/kubernetes/ceph/csi/cephfs
[root@k8s-master cephfs]# pwd
/k8s/rook/rook/cluster/examples/kubernetes/ceph/csi/cephfs



[root@k8s-master cephfs]# kubectl apply -f storageclass.yaml
storageclass.storage.k8s.io/rook-cephfs created


[root@k8s-master cephfs]# kubectl get sc
NAME              PROVISIONER                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
rook-ceph-block   rook-ceph.rbd.csi.ceph.com      Delete          Immediate           true                   4h31m
rook-cephfs       rook-ceph.cephfs.csi.ceph.com   Delete          Immediate           true                   2m44s
```



#### 2.3.Deployment

**Deployment，多个Pod文件共享，不能使用块存储。**

Deployment删除，PVC、PV自动删除.



```yaml
vim nginx-deploy-cephfs.yaml
-------------------------------


apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deploy-cephfs
  name: nginx-deploy-cephfs
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deploy-cephfs
  template:
    metadata:
      labels:
        app: nginx-deploy-cephfs
    spec:
      containers:
      - image: nginx
        name: nginx
        volumeMounts:
        - name: data
          mountPath: /usr/share/nginx/html
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: nginx-cephfs-pvc
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-cephfs-pvc
  namespace: dev
spec:
  storageClassName: "rook-cephfs"
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```

```shell
[root@k8s-master test]# kubectl apply -f nginx-deploy-cephfs.yaml 
deployment.apps/nginx-deploy-cephfs created
persistentvolumeclaim/nginx-cephfs-pvc created


[root@k8s-master ~]# kubectl get pvc -n dev
NAME               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nginx-cephfs-pvc   Bound    pvc-d3f0307a-5d70-4d7e-aa3f-34897f1f61ac   1Gi        RWX            rook-cephfs    3m30s

[root@k8s-master ~]# kubectl get pv -n dev
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   REASON   AGE
pvc-d3f0307a-5d70-4d7e-aa3f-34897f1f61ac   1Gi        RWX            Delete           Bound    dev/nginx-cephfs-pvc   rook-cephfs             3m33s


[root@k8s-master test]# kubectl get all -n dev
NAME                                       READY   STATUS    RESTARTS   AGE
pod/nginx-deploy-cephfs-7f58cd5447-642s2   1/1     Running   0          3m17s
pod/nginx-deploy-cephfs-7f58cd5447-qmpkb   1/1     Running   0          3m17s
pod/nginx-deploy-cephfs-7f58cd5447-rhj8h   1/1     Running   0          3m17s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy-cephfs   3/3     3            3           3m17s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deploy-cephfs-7f58cd5447   3         3         3       3m17s


# 查看挂载
[root@k8s-master test]# kubectl exec -it nginx-deploy-cephfs-7f58cd5447-bl69d -n dev -- sh
# df -Th
Filesystem                                                                                                                                               Type     Size  Used Avail Use% Mounted on
overlay                                                                                                                                                  overlay   50G  9.7G   41G  20% /
tmpfs                                                                                                                                                    tmpfs     64M     0   64M   0% /dev
tmpfs                                                                                                                                                    tmpfs    7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/mapper/centos-root                                                                                                                                  xfs       50G  9.7G   41G  20% /etc/hosts
shm                                                                                                                                                      tmpfs     64M     0   64M   0% /dev/shm

10.97.103.24:6789,10.109.133.209:6789,10.104.142.159:6789:/volumes/csi/csi-vol-481bb8b1-480d-11ec-9b57-4ecbcb966fa5/dce62825-dbc6-4e38-a136-ec6312eea49b ceph     1.0G     0  1.0G   0% /usr/share/nginx/html
tmpfs                                                                                                                                                    tmpfs    7.8G   12K  7.8G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs                                                                                                                                                    tmpfs    7.8G     0  7.8G   0% /proc/acpi
tmpfs                                                                                                                                                    tmpfs    7.8G     0  7.8G   0% /proc/scsi
tmpfs                                                                                                                                                    tmpfs    7.8G     0  7.8G   0% /sys/firmware


# df
Filesystem                                                                                                                                               1K-blocks     Used Available Use% Mounted on
overlay                                                                                                                                                   52403200 10077548  42325652  20% /
tmpfs                                                                                                                                                        65536        0     65536   0% /dev
tmpfs                                                                                                                                                      8123648        0   8123648   0% /sys/fs/cgroup
/dev/mapper/centos-root                                                                                                                                   52403200 10077548  42325652  20% /etc/hosts
shm                                                                                                                                                          65536        0     65536   0% /dev/shm
10.97.103.24:6789,10.109.133.209:6789,10.104.142.159:6789:/volumes/csi/csi-vol-481bb8b1-480d-11ec-9b57-4ecbcb966fa5/dce62825-dbc6-4e38-a136-ec6312eea49b   1048576        0   1048576   0% /usr/share/nginx/html
tmpfs                                                                                                                                                      8123648       12   8123636   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs                                                                                                                                                      8123648        0   8123648   0% /proc/acpi
tmpfs                                                                                                                                                      8123648        0   8123648   0% /proc/scsi
tmpfs                                                                                                                                                      8123648        0   8123648   0% /sys/firmware
```

![](/images/kubernetes/storage/rook-14.png)

```shell
# 测试
[root@k8s-master test]# kubectl get pod -n dev -owide
NAME                                   READY   STATUS    RESTARTS   AGE     IP             NODE        NOMINATED NODE   READINESS GATES
nginx-deploy-cephfs-7f58cd5447-642s2   1/1     Running   0          4m53s   10.244.36.67   k8s-node1   <none>           <none>
nginx-deploy-cephfs-7f58cd5447-qmpkb   1/1     Running   0          4m53s   10.244.36.81   k8s-node1   <none>           <none>
nginx-deploy-cephfs-7f58cd5447-rhj8h   1/1     Running   0          4m53s   10.244.36.66   k8s-node1   <none>           <none>


[root@k8s-master test]# kubectl -n dev exec -it nginx-deploy-cephfs-7f58cd5447-642s2 -- bash 
root@nginx-deploy-cephfs-7f58cd5447-642s2:/# echo "Hello  Nginx Rook-cephfs" > /usr/share/nginx/html/index.html
root@nginx-deploy-cephfs-7f58cd5447-642s2:/# cat /usr/share/nginx/html/index.html
Hello  Nginx Rook-cephfs


root@nginx-deploy-cephfs-7f58cd5447-642s2:/# curl 10.244.36.66
Hello  Nginx Rook-cephfs
root@nginx-deploy-cephfs-7f58cd5447-642s2:/# curl 10.244.36.81
Hello  Nginx Rook-cephfs
```

```shell
[root@k8s-master ~]# kubectl get pvc -n dev
NAME               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nginx-cephfs-pvc   Bound    pvc-d3f0307a-5d70-4d7e-aa3f-34897f1f61ac   1Gi        RWX            rook-cephfs    9m11s
[root@k8s-master ~]# kubectl get pvc -n dev
NAME               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nginx-cephfs-pvc   Bound    pvc-d3f0307a-5d70-4d7e-aa3f-34897f1f61ac   1Gi        RWX            rook-cephfs    9m14s


# 删除
[root@k8s-master test]# kubectl delete -f nginx-deploy-cephfs.yaml 
deployment.apps "nginx-deploy-cephfs" deleted
persistentvolumeclaim "nginx-cephfs-pvc" deleted

# pvc、pv自动删除
[root@k8s-master ~]# kubectl get pvc -n dev
No resources found in dev namespace.
[root@k8s-master ~]# kubectl get pv -n dev
No resources found
```



#### 2.4.PVC动态申请存储

pvc是存储卷类型的资源、它通过申请占用某个pv而创建，它于pv是一对一的关系、用户无需关心底层实现细节。申请时、用户只需指定目标空间的大小、访问模式、PV标签选择器和STORAGECLASS等相关信息即可。



**1.PVC动态申请存储**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-cephfs
  namespace: dev
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: rook-cephfs
  resources:
    requests:
      storage: 3Gi
```

```shell
# 创建pvc，自动创建pv
[root@k8s-master test]# kubectl apply -f pvc-cephfs.yaml 
persistentvolumeclaim/pvc-cephfs created


[root@k8s-master cephfs]# kubectl get pvc -n dev
NAME               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
pvc-cephfs         Bound    pvc-1a045966-4b26-4596-ba6e-c73a5de69b72   3Gi        RWX            rook-cephfs       22s

[root@k8s-master cephfs]# kubectl get pv -n dev
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS      REASON   AGE
pvc-1a045966-4b26-4596-ba6e-c73a5de69b72   3Gi        RWX            Delete           Bound    dev/pvc-cephfs         rook-cephfs                30s
```



**2.使用PVC**

在POD资源中调用PVC资源、只需要在定义volumes时使用persistentVolumeClaim字段嵌套指定两个字段即可、具体如下

```shell
apiVersion: v1
kind: Pod
metadata:
  name: pod-pvc-vol
  namespace: default
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html/
  volumes:
  - name: html
    persistentVoumeClaim:
      claimName: pvc-cephfs
```



### 3.**块存储（RBD）PVC在线扩容**

#### 3.1.实现方式

**RBD扩容原则：**

- storage class 必须支持在线扩容
- 只能扩容，不能收缩
- 根据扩容大小，卷扩容需要一定时间



**1.查看storageclass是否支持动态扩容**

```shell
[root@k8s-master03 ~]# kubectl  get storageclass 
NAME            PROVISIONER         AGE
cephfs          ceph.com/cephfs     289d
rbd (default)   kubernetes.io/rbd   289d

[root@k8s-master03 ceph]# kubectl edit storageclasses.storage.k8s.io rbd


# 查看是否有如下字段
allowVolumeExpansion: true   #增加该字段表示允许动态扩容
```

**2.编辑pvc，修改存储大小，保存退出**

```shell
kubectl edit pvc/grafana-pvc -n kube-system

spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 11Gi


#查看pvc大小是否更新完成，或者登陆容器检查挂载分区是否扩容成功
kubectl get pvc/grafana-pvc -n kube-system
```



#### 3.2.查看storageclass是否支持动态扩容

```shell
[root@k8s-master test]# kubectl get sc
NAME              PROVISIONER                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
rook-ceph-block   rook-ceph.rbd.csi.ceph.com      Delete          Immediate           true                   3h49m


[root@k8s-master test]# kubectl describe sc rook-ceph-block
Name:            rook-ceph-block
...
AllowVolumeExpansion:  True
...
```



#### 3.3.PVC直接扩容

```shell
[root@k8s-master cephfs]# kubectl get pv -n dev
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM         STORAGECLASS      REASON   AGE
pvc-feb4b298-ccf3-4a0c-b0c7-26dda1fa6f56   1Gi        RWO            Delete           Bound    dev/pvc-rdb   rook-ceph-block            109m

[root@k8s-master cephfs]# kubectl get pv -n dev
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM         STORAGECLASS      REASON   AGE
pvc-feb4b298-ccf3-4a0c-b0c7-26dda1fa6f56   1Gi        RWO            Delete           Bound    dev/pvc-rdb   rook-ceph-block            109m



[root@k8s-master cephfs]# kubectl edit pvc pvc-rdb -n dev
persistentvolumeclaim/pvc-rdb edited

spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
  storageClassName: rook-ceph-block
  volumeMode: Filesystem
  volumeName: pvc-feb4b298-ccf3-4a0c-b0c7-26dda1fa6f56


[root@k8s-master cephfs]# kubectl get pvc -n dev
NAME      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
pvc-rdb   Bound    pvc-feb4b298-ccf3-4a0c-b0c7-26dda1fa6f56   1Gi        RWO            rook-ceph-block   117m
[root@k8s-master cephfs]# kubectl get pv -n dev
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM         STORAGECLASS      REASON   AGE
pvc-feb4b298-ccf3-4a0c-b0c7-26dda1fa6f56   3Gi        RWO            Delete           Bound    dev/pvc-rdb   rook-ceph-block            117m
```



#### 3.4.PVC在线扩容

```yaml
vim nginx-ss-rbd.yaml
-------------------------------


apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-ss-rbd
  namespace: dev
spec:
  selector:
    matchLabels:
      app: nginx-ss-rbd 
  serviceName: "nginx"
  replicas: 3 
  template:
    metadata:
      labels:
        app: nginx-ss-rbd 
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "rook-ceph-block"
      resources:
        requests:
          storage: 2Gi
```

```shell
# 创建
[root@k8s-master test]# kubectl apply -f nginx-ss-rbd.yaml 
statefulset.apps/nginx-ss-rbd created


# 查看
[root@k8s-master cephfs]# kubectl get pvc -n dev
NAME                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
www-nginx-ss-rbd-0   Bound    pvc-7bdf4ea2-a198-486b-be9c-6641fd1b69c7   2Gi        RWO            rook-ceph-block   4m55s
www-nginx-ss-rbd-1   Bound    pvc-a5950eed-4c53-4295-a6fa-1c4d55061d5c   2Gi        RWO            rook-ceph-block   4m17s
www-nginx-ss-rbd-2   Bound    pvc-a1d12e1f-7a33-4032-bcdb-f171de69706b   2Gi        RWO            rook-ceph-block   3m31s
[root@k8s-master cephfs]# kubectl get pv -n dev
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS      REASON   AGE
pvc-7bdf4ea2-a198-486b-be9c-6641fd1b69c7   2Gi        RWO            Delete           Bound    dev/www-nginx-ss-rbd-0   rook-ceph-block            4m57s
pvc-a1d12e1f-7a33-4032-bcdb-f171de69706b   2Gi        RWO            Delete           Bound    dev/www-nginx-ss-rbd-2   rook-ceph-block            3m33s
pvc-a5950eed-4c53-4295-a6fa-1c4d55061d5c   2Gi        RWO            Delete           Bound    dev/www-nginx-ss-rbd-1   rook-ceph-block            4m18s


[root@k8s-master test]# kubectl get pod -n dev
NAME             READY   STATUS    RESTARTS   AGE
nginx-ss-rbd-0   1/1     Running   0          6m11s
nginx-ss-rbd-1   1/1     Running   0          5m33s
nginx-ss-rbd-2   1/1     Running   0          4m47s



# 查看磁盘挂载
[root@k8s-master test]#  kubectl exec -it nginx-ss-rbd-0 -n dev -- bash
root@nginx-ss-rbd-0:/# df -Th
Filesystem              Type     Size  Used Avail Use% Mounted on
overlay                 overlay   50G  9.7G   41G  20% /
tmpfs                   tmpfs     64M     0   64M   0% /dev
tmpfs                   tmpfs    7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/mapper/centos-root xfs       50G  9.7G   41G  20% /etc/hosts
shm                     tmpfs     64M     0   64M   0% /dev/shm


/dev/rbd0               ext4     2.0G  6.0M  1.9G   1% /usr/share/nginx/html

tmpfs                   tmpfs    7.8G   12K  7.8G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs                   tmpfs    7.8G     0  7.8G   0% /proc/acpi
tmpfs                   tmpfs    7.8G     0  7.8G   0% /proc/scsi
tmpfs                   tmpfs    7.8G     0  7.8G   0% /sys/firmware
```

```shell
[root@k8s-master cephfs]# kubectl get pvc -n dev
NAME                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
www-nginx-ss-rbd-0   Bound    pvc-7bdf4ea2-a198-486b-be9c-6641fd1b69c7   2Gi        RWO            rook-ceph-block   7m57s
www-nginx-ss-rbd-1   Bound    pvc-a5950eed-4c53-4295-a6fa-1c4d55061d5c   2Gi        RWO            rook-ceph-block   7m19s
www-nginx-ss-rbd-2   Bound    pvc-a1d12e1f-7a33-4032-bcdb-f171de69706b   2Gi        RWO            rook-ceph-block   6m33s


# 修改存储容量
[root@k8s-master cephfs]# kubectl edit pvc www-nginx-ss-rbd-0 -n dev
persistentvolumeclaim/www-nginx-ss-rbd-0 edited

spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
  storageClassName: rook-ceph-block
  volumeMode: Filesystem
  volumeName: pvc-a5950eed-4c53-4295-a6fa-1c4d55061d5c


[root@k8s-master cephfs]# 
[root@k8s-master cephfs]# kubectl edit pvc www-nginx-ss-rbd-1 -n dev
persistentvolumeclaim/www-nginx-ss-rbd-1 edited

spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 4Gi
  storageClassName: rook-ceph-block
  volumeMode: Filesystem
  volumeName: pvc-a5950eed-4c53-4295-a6fa-1c4d55061d5c


# 查看
[root@k8s-master cephfs]# kubectl get pvc -n dev
NAME                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
www-nginx-ss-rbd-0   Bound    pvc-7bdf4ea2-a198-486b-be9c-6641fd1b69c7   3Gi        RWO            rook-ceph-block   11m
www-nginx-ss-rbd-1   Bound    pvc-a5950eed-4c53-4295-a6fa-1c4d55061d5c   4Gi        RWO            rook-ceph-block   10m
www-nginx-ss-rbd-2   Bound    pvc-a1d12e1f-7a33-4032-bcdb-f171de69706b   2Gi        RWO            rook-ceph-block   9m54s

[root@k8s-master cephfs]# kubectl get pv -n dev
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS      REASON   AGE
pvc-7bdf4ea2-a198-486b-be9c-6641fd1b69c7   3Gi        RWO            Delete           Bound    dev/www-nginx-ss-rbd-0   rook-ceph-block            11m
pvc-a1d12e1f-7a33-4032-bcdb-f171de69706b   2Gi        RWO            Delete           Bound    dev/www-nginx-ss-rbd-2   rook-ceph-block            10m
pvc-a5950eed-4c53-4295-a6fa-1c4d55061d5c   4Gi        RWO            Delete           Bound    dev/www-nginx-ss-rbd-1   rook-ceph-block            10m


# 查看磁盘挂载
[root@k8s-master test]#  kubectl exec -it nginx-ss-rbd-0 -n dev -- bash
Filesystem              Type     Size  Used Avail Use% Mounted on
overlay                 overlay   50G  9.7G   41G  20% /
tmpfs                   tmpfs     64M     0   64M   0% /dev
tmpfs                   tmpfs    7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/mapper/centos-root xfs       50G  9.7G   41G  20% /etc/hosts
shm                     tmpfs     64M     0   64M   0% /dev/shm


/dev/rbd0               ext4     2.9G  6.0M  2.9G   1% /usr/share/nginx/html

tmpfs                   tmpfs    7.8G   12K  7.8G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs                   tmpfs    7.8G     0  7.8G   0% /proc/acpi
tmpfs                   tmpfs    7.8G     0  7.8G   0% /proc/scsi
tmpfs                   tmpfs    7.8G     0  7.8G   0% /sys/firmware


[root@k8s-master test]#  kubectl exec -it nginx-ss-rbd-1 -n dev -- bash
root@nginx-ss-rbd-1:/# df -Th
Filesystem              Type     Size  Used Avail Use% Mounted on
overlay                 overlay   50G  9.7G   41G  20% /
tmpfs                   tmpfs     64M     0   64M   0% /dev
tmpfs                   tmpfs    7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/mapper/centos-root xfs       50G  9.7G   41G  20% /etc/hosts
shm                     tmpfs     64M     0   64M   0% /dev/shm


/dev/rbd1               ext4     3.9G  8.0M  3.9G   1% /usr/share/nginx/html

tmpfs                   tmpfs    7.8G   12K  7.8G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs                   tmpfs    7.8G     0  7.8G   0% /proc/acpi
tmpfs                   tmpfs    7.8G     0  7.8G   0% /proc/scsi
tmpfs                   tmpfs    7.8G     0  7.8G   0% /sys/firmware
```

![](/images/kubernetes/storage/rook-16.png)







