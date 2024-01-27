* TOC
{:toc}



## 一、概述



### 1.Operator

 ```shell
 # awesome-operators
 https://github.com/operator-framework/awesome-operators
 
 
 # 安装
 #Redis Cluster #2	ucloud/redis-cluster-operator
 https://github.com/ucloud/redis-cluster-operator
 ```



| Redis #1         | [spotahome/redis-operator](https://github.com/spotahome/redis-operator) | Redis Operator creates/configures/manages redis clusters atop Kubernetes. |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Redis #2         | [jw-s/redis-operator](https://github.com/jw-s/redis-operator) | Redis operator for Kubernetes                                |
| Redis #3         | [amaizfinance/redis-operator](https://github.com/amaizfinance/redis-operator) | Redis operator for Kubernetes. Provides high availability for Redis. Designed to resist to most kinds of failures without human intervention. |
| Redis #4         | [kube-incubator/redis-operator](https://github.com/kube-incubator/redis-operator) | Redis operator for Kubernetes based on operator-sdk.         |
| Redis #5         | [ucloud/redis-operator](https://github.com/ucloud/redis-operator) | Redis operator build a Highly Available Redis cluster with Sentinel atop Kubernetes |
| Redis Cluster #1 | [AmadeusITGroup/Redis-Operator](https://github.com/AmadeusITGroup/Redis-Operator) | A Kubernetes operator for running Redis in Cluster mode      |
| Redis Cluster #2 | [ucloud/redis-cluster-operator](https://github.com/ucloud/redis-cluster-operator) | Redis Cluster Operator creates and manages Redis in Cluster mode atop Kubernetes |



### 2.Helm

**Bitnami**

```shell
https://bitnami.com/
https://github.com/bitnami
https://bitnami.com/stacks
```



```shell
[root@k8s-master ~]# helm search repo redis
NAME                 	CHART VERSION	APP VERSION	DESCRIPTION                                       
bitnami/redis        	15.4.1       	6.2.6      	Open source, advanced key-value store. It is of...
bitnami/redis-cluster	7.0.1        	6.2.6      	Open source, advanced key-value store. It is of...
......
```



## 二、基础



### 1.Operator

```shell
# 安装
#Redis Cluster #2	ucloud/redis-cluster-operator
https://github.com/ucloud/redis-cluster-operator
```



Redis Cluster Operator用于管理基于k8s的Redis Cluster
该operator基于Operator framework之上（https://github.com/operator-framework/operator-sdk）

![](/images/kubernetes/middleware/redis-1.png)

每个master node和slave node由一个statefulset管理，每个statefulset创建一个headless svc， 所有的node创建一个clusterIP service。

每个statefulset使用PodAntiAffinity来确保master和slave部署在不同的node上。同时，当operator在一个statefulset上选择master时，他优先选择不同k8s nodes上的pod作为master。



#### 1.1.部署集群



**1.下载redis-cluster-operator**

```shell
git  clone  https://github.com/ucloud/redis-cluster-operator.git
```



**2.创建自定义资源（CRD）**

```bash
$ kubectl create -f deploy/crds/redis.kun_distributedredisclusters_crd.yaml
$ kubectl create -f deploy/crds/redis.kun_redisclusterbackups_crd.yaml
```



**3.创建operator**

```shell
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
```



**4.部署redis cluster**

注意：**只有使用持久性存储（pvc）的redis集群在意外删除或滚动更新后才能恢复。即使您不使用持久性（如rdb或aof），也需要将pvc设置为redis。**

```shell
$ kubectl apply -f deploy/example/redis.kun_v1alpha1_distributedrediscluster_cr.yaml
```

```shell
# 测试


$ kubectl get distributedrediscluster
NAME                              MASTERSIZE   STATUS    AGE
example-distributedrediscluster   3            Scaling   11s

$ kubectl get all -l redis.kun/name=example-distributedrediscluster
NAME                                          READY   STATUS    RESTARTS   AGE
pod/drc-example-distributedrediscluster-0-0   1/1     Running   0          2m48s
pod/drc-example-distributedrediscluster-0-1   1/1     Running   0          2m8s
pod/drc-example-distributedrediscluster-1-0   1/1     Running   0          2m48s
pod/drc-example-distributedrediscluster-1-1   1/1     Running   0          2m13s
pod/drc-example-distributedrediscluster-2-0   1/1     Running   0          2m48s
pod/drc-example-distributedrediscluster-2-1   1/1     Running   0          2m15s

NAME                                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)              AGE
service/example-distributedrediscluster     ClusterIP   172.17.132.71   <none>        6379/TCP,16379/TCP   2m48s
service/example-distributedrediscluster-0   ClusterIP   None            <none>        6379/TCP,16379/TCP   2m48s
service/example-distributedrediscluster-1   ClusterIP   None            <none>        6379/TCP,16379/TCP   2m48s
service/example-distributedrediscluster-2   ClusterIP   None            <none>        6379/TCP,16379/TCP   2m48s

NAME                                                     READY   AGE
statefulset.apps/drc-example-distributedrediscluster-0   2/2     2m48s
statefulset.apps/drc-example-distributedrediscluster-1   2/2     2m48s
statefulset.apps/drc-example-distributedrediscluster-2   2/2     2m48s

$ kubectl get distributedrediscluster
NAME                              MASTERSIZE   STATUS    AGE
example-distributedrediscluster   3            Healthy   4m
```



#### 1.2.持久卷部署

创建整个集群

```shell
$ kubectl create -f deploy/example/persistent.yaml
```



#### 1.3.扩容Redis Cluster

```yaml
# custom-service.yaml
# kubectl apply -f deploy/example/custom-service.yaml 


apiVersion: redis.kun/v1alpha1
kind: DistributedRedisCluster
metadata:
  annotations:
    # if your operator run as cluster-scoped, add this annotations
    redis.kun/scope: cluster-scoped
  name: example-distributedrediscluster
spec:
  # Increase the masterSize to trigger the scaling.
  masterSize: 4
  ClusterReplicas: 1
  image: redis:5.0.4-alpine
```



#### 1.4.缩容Redis Cluster

```yaml
# custom-service.yaml
# kubectl apply -f deploy/example/custom-service.yaml 


apiVersion: redis.kun/v1alpha1
kind: DistributedRedisCluster
metadata:
  annotations:
    # if your operator run as cluster-scoped, add this annotations
    redis.kun/scope: cluster-scoped
  name: example-distributedrediscluster
spec:
  # Increase the masterSize to trigger the scaling.
  masterSize: 3
  ClusterReplicas: 1
  image: redis:5.0.4-alpine
```



#### 1.5.删除集群

```shell
# 删除集群

# 根据创建选择
kubectl delete -f deploy/example/redis.kun_v1alpha1_distributedrediscluster_cr.yaml


kubectl delete -f deploy/cluster/operator.yaml
kubectl delete -f deploy/cluster/cluster_role_binding.yaml
kubectl delete -f deploy/cluster/cluster_role.yaml
kubectl delete -f deploy/service_account.yaml
kubectl delete -f deploy/crds/redis.kun_redisclusterbackups_crd.yaml
kubectl delete -f deploy/crds/redis.kun_distributedredisclusters_crd.yaml
```



#### 1.6.配置说明

```shell
#redis-cluster-operator\deploy\example


[root@k8s-master example]# pwd
/k8s/middleware/redis/operator/redis-cluster-operator/deploy/example
[root@k8s-master example]# ll
total 32
drwxr-xr-x 2 root root 125 Nov 29 11:09 backup-restore
-rw-r--r-- 1 root root 889 Nov 29 11:09 custom-config.yaml
-rw-r--r-- 1 root root 557 Nov 29 11:09 custom-password.yaml
-rw-r--r-- 1 root root 410 Nov 29 11:09 custom-resources.yaml
-rw-r--r-- 1 root root 325 Nov 29 14:31 custom-service.yaml
-rw-r--r-- 1 root root 401 Nov 29 11:55 persistent.yaml
-rw-r--r-- 1 root root 346 Nov 29 11:09 prometheus-exporter.yaml
-rw-r--r-- 1 root root 320 Nov 29 11:09 redis.kun_v1alpha1_distributedrediscluster_cr.yaml
-rw-r--r-- 1 root root 517 Nov 29 11:09 securitycontext.yaml



# 说明

# 配置存储
persistent.yaml

# 扩容、缩容
custom-service.yaml

# 配置密码
custom-password.yaml

# 不配置存储
redis.kun_v1alpha1_distributedrediscluster_cr.yaml
```





## 三、实践



### 1.部署Redis集群

**1.下载redis-cluster-operator**

```shell
# git  clone  https://github.com/ucloud/redis-cluster-operator.git


[root@k8s-master operator]# git  clone  https://github.com/ucloud/redis-cluster-operator.git
Cloning into 'redis-cluster-operator'...
remote: Enumerating objects: 2024, done.
remote: Counting objects: 100% (51/51), done.
remote: Compressing objects: 100% (30/30), done.
remote: Total 2024 (delta 15), reused 38 (delta 11), pack-reused 1973
Receiving objects: 100% (2024/2024), 944.24 KiB | 0 bytes/s, done.
Resolving deltas: 100% (1288/1288), done.
```



**2.创建自定义资源（CRD）**

```shell
$ kubectl create -f deploy/crds/redis.kun_distributedredisclusters_crd.yaml
$ kubectl create -f deploy/crds/redis.kun_redisclusterbackups_crd.yaml



[root@k8s-master redis-cluster-operator]# pwd
/k8s/middleware/redis/operator/redis-cluster-operator


[root@k8s-master redis-cluster-operator]# kubectl create -f deploy/crds/redis.kun_distributedredisclusters_crd.yaml
Warning: apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
customresourcedefinition.apiextensions.k8s.io/distributedredisclusters.redis.kun created


[root@k8s-master redis-cluster-operator]# kubectl create -f deploy/crds/redis.kun_redisclusterbackups_crd.yaml
Warning: apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
customresourcedefinition.apiextensions.k8s.io/redisclusterbackups.redis.kun created
```



**3.创建operator**

```shell
// cluster-scoped
$ kubectl create -f deploy/service_account.yaml
$ kubectl create -f deploy/cluster/cluster_role.yaml
$ kubectl create -f deploy/cluster/cluster_role_binding.yaml
$ kubectl create -f deploy/cluster/operator.yaml


# 创建
[root@k8s-master redis-cluster-operator]# kubectl create -f deploy/service_account.yaml
serviceaccount/redis-cluster-operator created

[root@k8s-master redis-cluster-operator]# kubectl create -f deploy/cluster/cluster_role.yaml
clusterrole.rbac.authorization.k8s.io/redis-cluster-operator created

[root@k8s-master redis-cluster-operator]# kubectl create -f deploy/cluster/cluster_role_binding.yaml
clusterrolebinding.rbac.authorization.k8s.io/redis-cluster-operator created

[root@k8s-master redis-cluster-operator]# kubectl create -f deploy/cluster/operator.yaml
deployment.apps/redis-cluster-operator created
configmap/redis-admin created


[root@k8s-master redis-cluster-operator]# kubectl get all
NAME                                          READY   STATUS    RESTARTS   AGE
pod/redis-cluster-operator-6669898858-775vj   1/1     Running   0          11s

NAME                                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
service/kubernetes                       ClusterIP   10.96.0.1        <none>        443/TCP             103d
service/redis-cluster-operator-metrics   ClusterIP   10.100.160.214   <none>        8383/TCP,8686/TCP   1s

NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/redis-cluster-operator   1/1     1            1           11s

NAME                                                DESIRED   CURRENT   READY   AGE
replicaset.apps/redis-cluster-operator-6669898858   1         1         1       11s
```



**4.部署redis cluster**

注意：**没有使用持久性存储**

```shell
$ kubectl apply -f deploy/example/redis.kun_v1alpha1_distributedrediscluster_cr.yaml


[root@k8s-master redis-cluster-operator]# kubectl apply -f deploy/example/redis.kun_v1alpha1_distributedrediscluster_cr.yaml
distributedrediscluster.redis.kun/example-distributedrediscluster created
```

```shell
# 测试

[root@k8s-master redis-cluster-operator]# kubectl get distributedrediscluster
NAME                              MASTERSIZE   STATUS    AGE
example-distributedrediscluster   3            Scaling   27s


[root@k8s-master redis-cluster-operator]# kubectl get all -l redis.kun/name=example-distributedrediscluster
NAME                                          READY   STATUS    RESTARTS   AGE
pod/drc-example-distributedrediscluster-0-0   1/1     Running   0          3m4s
pod/drc-example-distributedrediscluster-0-1   1/1     Running   0          2m31s
pod/drc-example-distributedrediscluster-1-0   1/1     Running   0          3m4s
pod/drc-example-distributedrediscluster-1-1   1/1     Running   0          2m25s
pod/drc-example-distributedrediscluster-2-0   1/1     Running   0          3m4s
pod/drc-example-distributedrediscluster-2-1   1/1     Running   0          2m31s

NAME                                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)              AGE
service/example-distributedrediscluster     ClusterIP   10.105.168.17   <none>        6379/TCP,16379/TCP   3m3s
service/example-distributedrediscluster-0   ClusterIP   None            <none>        6379/TCP,16379/TCP   3m3s
service/example-distributedrediscluster-1   ClusterIP   None            <none>        6379/TCP,16379/TCP   3m3s
service/example-distributedrediscluster-2   ClusterIP   None            <none>        6379/TCP,16379/TCP   3m3s

NAME                                                     READY   AGE
statefulset.apps/drc-example-distributedrediscluster-0   2/2     3m4s
statefulset.apps/drc-example-distributedrediscluster-1   2/2     3m4s
statefulset.apps/drc-example-distributedrediscluster-2   2/2     3m4s


[root@k8s-master redis-cluster-operator]# kubectl get distributedrediscluster
NAME                              MASTERSIZE   STATUS    AGE
example-distributedrediscluster   3            Healthy   111s
```



### 2.持久卷部署（首选）

**使用持久性存储方式部署**

**配置：class: rook-ceph-block  # Rook的块存储**

```yaml
# deploy\example\persistent.yaml


[root@k8s-master example]# vim persistent.yaml 

apiVersion: redis.kun/v1alpha1
kind: DistributedRedisCluster
metadata:
  annotations:
    # if your operator run as cluster-scoped, add this annotations
    redis.kun/scope: cluster-scoped
  name: example-distributedrediscluster
spec:
  image: redis:5.0.4-alpine
  masterSize: 3
  clusterReplicas: 1
  storage:
    type: persistent-claim
    size: 2Gi
    class: rook-ceph-block  # Rook的块存储
    deleteClaim: true
```

```shell
$ kubectl create -f deploy/example/persistent.yaml


[root@k8s-master redis-cluster-operator]# kubectl create -f deploy/example/persistent.yaml
distributedrediscluster.redis.kun/example-distributedrediscluster created



[root@k8s-master redis-cluster-operator]# kubectl get distributedrediscluster
NAME                              MASTERSIZE   STATUS    AGE
example-distributedrediscluster   3            Scaling   103s


[root@k8s-master redis-cluster-operator]# kubectl get all -l redis.kun/name=example-distributedrediscluster
NAME                                          READY   STATUS    RESTARTS   AGE
pod/drc-example-distributedrediscluster-0-0   1/1     Running   0          119s
pod/drc-example-distributedrediscluster-0-1   1/1     Running   0          74s
pod/drc-example-distributedrediscluster-1-0   1/1     Running   0          119s
pod/drc-example-distributedrediscluster-1-1   1/1     Running   0          66s
pod/drc-example-distributedrediscluster-2-0   1/1     Running   0          119s
pod/drc-example-distributedrediscluster-2-1   1/1     Running   0          75s

NAME                                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE
service/example-distributedrediscluster     ClusterIP   10.102.195.186   <none>        6379/TCP,16379/TCP   118s
service/example-distributedrediscluster-0   ClusterIP   None             <none>        6379/TCP,16379/TCP   119s
service/example-distributedrediscluster-1   ClusterIP   None             <none>        6379/TCP,16379/TCP   119s
service/example-distributedrediscluster-2   ClusterIP   None             <none>        6379/TCP,16379/TCP   119s

NAME                                                     READY   AGE
statefulset.apps/drc-example-distributedrediscluster-0   2/2     119s
statefulset.apps/drc-example-distributedrediscluster-1   2/2     119s
statefulset.apps/drc-example-distributedrediscluster-2   2/2     119s


[root@k8s-master redis-cluster-operator]# kubectl get distributedrediscluster
NAME                              MASTERSIZE   STATUS    AGE
example-distributedrediscluster   3            Healthy   3m7s
```

```shell
# 查看存储

[root@k8s-master example]# kubectl get pvc
NAME                                                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
redis-data-drc-example-distributedrediscluster-0-0   Bound    pvc-54d02c21-bacb-469a-8937-f42e694a1beb   2Gi        RWO            rook-ceph-block   7m22s
redis-data-drc-example-distributedrediscluster-0-1   Bound    pvc-d876ae49-0d43-4c27-8b61-ef65f9985555   2Gi        RWO            rook-ceph-block   6m37s
redis-data-drc-example-distributedrediscluster-1-0   Bound    pvc-06248e41-f5c7-4a93-b842-d32424e439a5   2Gi        RWO            rook-ceph-block   7m22s
redis-data-drc-example-distributedrediscluster-1-1   Bound    pvc-2062915a-9008-4940-ab7c-ce025f923d19   2Gi        RWO            rook-ceph-block   6m29s
redis-data-drc-example-distributedrediscluster-2-0   Bound    pvc-43b60e7c-b526-401d-b016-912b18d0d531   2Gi        RWO            rook-ceph-block   7m22s
redis-data-drc-example-distributedrediscluster-2-1   Bound    pvc-88b9530b-8ecc-4445-a2d2-66c602c972af   2Gi        RWO            rook-ceph-block   6m38s


[root@k8s-master example]# kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                                        STORAGECLASS      REASON   AGE
pvc-06248e41-f5c7-4a93-b842-d32424e439a5   2Gi        RWO            Delete           Bound    default/redis-data-drc-example-distributedrediscluster-1-0   rook-ceph-block            8m22s
pvc-2062915a-9008-4940-ab7c-ce025f923d19   2Gi        RWO            Delete           Bound    default/redis-data-drc-example-distributedrediscluster-1-1   rook-ceph-block            7m29s
pvc-43b60e7c-b526-401d-b016-912b18d0d531   2Gi        RWO            Delete           Bound    default/redis-data-drc-example-distributedrediscluster-2-0   rook-ceph-block            8m22s
pvc-54d02c21-bacb-469a-8937-f42e694a1beb   2Gi        RWO            Delete           Bound    default/redis-data-drc-example-distributedrediscluster-0-0   rook-ceph-block            8m22s
pvc-88b9530b-8ecc-4445-a2d2-66c602c972af   2Gi        RWO            Delete           Bound    default/redis-data-drc-example-distributedrediscluster-2-1   rook-ceph-block            7m39s
pvc-d876ae49-0d43-4c27-8b61-ef65f9985555   2Gi        RWO            Delete           Bound    default/redis-data-drc-example-distributedrediscluster-0-1   rook-ceph-block            7m38s


[root@k8s-master example]# kubectl exec -it drc-example-distributedrediscluster-0-0 -- sh
/data # ls
dump.rdb        lost+found      nodes.conf      redis_password
/data # 
/data # 
/data # df -Th
Filesystem           Type            Size      Used Available Use% Mounted on
overlay              overlay        50.0G     17.1G     32.8G  34% /
tmpfs                tmpfs          64.0M         0     64.0M   0% /dev
tmpfs                tmpfs           7.7G         0      7.7G   0% /sys/fs/cgroup


/dev/rbd0            ext4            1.9G      6.0M      1.9G   0% /data


/dev/mapper/centos-root
                     xfs            50.0G     17.1G     32.8G  34% /conf
/dev/mapper/centos-root
                     xfs            50.0G     17.1G     32.8G  34% /dev/termination-log
/dev/mapper/centos-root
                     xfs            50.0G     17.1G     32.8G  34% /etc/resolv.conf
/dev/mapper/centos-root
                     xfs            50.0G     17.1G     32.8G  34% /etc/hostname
/dev/mapper/centos-root
                     xfs            50.0G     17.1G     32.8G  34% /etc/hosts
shm                  tmpfs          64.0M         0     64.0M   0% /dev/shm
tmpfs                tmpfs           7.7G     12.0K      7.7G   0% /run/secrets/kubernetes.io/serviceaccount
tmpfs                tmpfs           7.7G         0      7.7G   0% /proc/acpi
tmpfs                tmpfs          64.0M         0     64.0M   0% /proc/kcore
tmpfs                tmpfs          64.0M         0     64.0M   0% /proc/keys
tmpfs                tmpfs          64.0M         0     64.0M   0% /proc/timer_list
tmpfs                tmpfs          64.0M         0     64.0M   0% /proc/timer_stats
tmpfs                tmpfs          64.0M         0     64.0M   0% /proc/sched_debug
tmpfs                tmpfs           7.7G         0      7.7G   0% /proc/scsi
tmpfs                tmpfs           7.7G         0      7.7G   0% /sys/firmware
/data # 

```

![](/images/kubernetes/middleware/redis-2.png)



### 3.扩容Redis Cluster

```yaml
# custom-service.yaml


apiVersion: redis.kun/v1alpha1
kind: DistributedRedisCluster
metadata:
  annotations:
    # if your operator run as cluster-scoped, add this annotations
    redis.kun/scope: cluster-scoped
  name: example-distributedrediscluster
spec:
  # Increase the masterSize to trigger the scaling.
  masterSize: 4
  ClusterReplicas: 1
  image: redis:5.0.4-alpine
```

```shell
[root@k8s-master example]# kubectl apply -f custom-service.yaml 
Warning: resource distributedredisclusters/example-distributedrediscluster is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
distributedrediscluster.redis.kun/example-distributedrediscluster configured


# 查看
[root@k8s-master example]# kubectl get distributedrediscluster
NAME                              MASTERSIZE   STATUS    AGE
example-distributedrediscluster   4            Healthy   137m


[root@k8s-master example]# kubectl get all -l redis.kun/name=example-distributedrediscluster
NAME                                          READY   STATUS    RESTARTS   AGE
pod/drc-example-distributedrediscluster-0-0   1/1     Running   0          137m
pod/drc-example-distributedrediscluster-0-1   1/1     Running   0          136m
pod/drc-example-distributedrediscluster-1-0   1/1     Running   0          137m
pod/drc-example-distributedrediscluster-1-1   1/1     Running   0          136m
pod/drc-example-distributedrediscluster-2-0   1/1     Running   0          137m
pod/drc-example-distributedrediscluster-2-1   1/1     Running   0          136m
pod/drc-example-distributedrediscluster-3-0   1/1     Running   0          13m
pod/drc-example-distributedrediscluster-3-1   1/1     Running   0          13m

NAME                                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE
service/example-distributedrediscluster     ClusterIP   10.102.195.186   <none>        6379/TCP,16379/TCP   137m
service/example-distributedrediscluster-0   ClusterIP   None             <none>        6379/TCP,16379/TCP   137m
service/example-distributedrediscluster-1   ClusterIP   None             <none>        6379/TCP,16379/TCP   137m
service/example-distributedrediscluster-2   ClusterIP   None             <none>        6379/TCP,16379/TCP   137m
service/redis-svc                           ClusterIP   10.108.255.163   <none>        6379/TCP,16379/TCP   13m
service/redis-svc-0                         ClusterIP   None             <none>        6379/TCP,16379/TCP   13m
service/redis-svc-1                         ClusterIP   None             <none>        6379/TCP,16379/TCP   13m
service/redis-svc-2                         ClusterIP   None             <none>        6379/TCP,16379/TCP   13m
service/redis-svc-3                         ClusterIP   None             <none>        6379/TCP,16379/TCP   13m

NAME                                                     READY   AGE
statefulset.apps/drc-example-distributedrediscluster-0   2/2     137m
statefulset.apps/drc-example-distributedrediscluster-1   2/2     137m
statefulset.apps/drc-example-distributedrediscluster-2   2/2     137m
statefulset.apps/drc-example-distributedrediscluster-3   2/2     13m


[root@k8s-master example]# kubectl get pvc
NAME                                                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
redis-data-drc-example-distributedrediscluster-0-0   Bound    pvc-54d02c21-bacb-469a-8937-f42e694a1beb   2Gi        RWO            rook-ceph-block   137m
redis-data-drc-example-distributedrediscluster-0-1   Bound    pvc-d876ae49-0d43-4c27-8b61-ef65f9985555   2Gi        RWO            rook-ceph-block   136m
redis-data-drc-example-distributedrediscluster-1-0   Bound    pvc-06248e41-f5c7-4a93-b842-d32424e439a5   2Gi        RWO            rook-ceph-block   137m
redis-data-drc-example-distributedrediscluster-1-1   Bound    pvc-2062915a-9008-4940-ab7c-ce025f923d19   2Gi        RWO            rook-ceph-block   136m
redis-data-drc-example-distributedrediscluster-2-0   Bound    pvc-43b60e7c-b526-401d-b016-912b18d0d531   2Gi        RWO            rook-ceph-block   137m
redis-data-drc-example-distributedrediscluster-2-1   Bound    pvc-88b9530b-8ecc-4445-a2d2-66c602c972af   2Gi        RWO            rook-ceph-block   136m
redis-data-drc-example-distributedrediscluster-3-0   Bound    pvc-f649e6d2-29c1-4d98-96d1-b6727cac0ad3   2Gi        RWO            rook-ceph-block   14m
redis-data-drc-example-distributedrediscluster-3-1   Bound    pvc-aa510fd6-b254-4e12-8be4-7985108161bc   2Gi        RWO            rook-ceph-block   13m
```



### 4.缩容Redis Cluster

```yaml
# custom-service.yaml


apiVersion: redis.kun/v1alpha1
kind: DistributedRedisCluster
metadata:
  annotations:
    # if your operator run as cluster-scoped, add this annotations
    redis.kun/scope: cluster-scoped
  name: example-distributedrediscluster
spec:
  # Increase the masterSize to trigger the scaling.
  masterSize: 3
  ClusterReplicas: 1
  image: redis:5.0.4-alpine
```

```shell
[root@k8s-master example]# kubectl apply -f custom-service.yaml 
distributedrediscluster.redis.kun/example-distributedrediscluster configured


# 查看
[root@k8s-master example]# kubectl get distributedrediscluster
NAME                              MASTERSIZE   STATUS    AGE
example-distributedrediscluster   3            Healthy   142m


[root@k8s-master example]# kubectl get all -l redis.kun/name=example-distributedrediscluster
NAME                                          READY   STATUS    RESTARTS   AGE
pod/drc-example-distributedrediscluster-0-0   1/1     Running   0          142m
pod/drc-example-distributedrediscluster-0-1   1/1     Running   0          141m
pod/drc-example-distributedrediscluster-1-0   1/1     Running   0          142m
pod/drc-example-distributedrediscluster-1-1   1/1     Running   0          141m
pod/drc-example-distributedrediscluster-2-0   1/1     Running   0          142m
pod/drc-example-distributedrediscluster-2-1   1/1     Running   0          141m

NAME                                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE
service/example-distributedrediscluster     ClusterIP   10.102.195.186   <none>        6379/TCP,16379/TCP   142m
service/example-distributedrediscluster-0   ClusterIP   None             <none>        6379/TCP,16379/TCP   142m
service/example-distributedrediscluster-1   ClusterIP   None             <none>        6379/TCP,16379/TCP   142m
service/example-distributedrediscluster-2   ClusterIP   None             <none>        6379/TCP,16379/TCP   142m
service/redis-svc                           ClusterIP   10.108.255.163   <none>        6379/TCP,16379/TCP   19m
service/redis-svc-0                         ClusterIP   None             <none>        6379/TCP,16379/TCP   19m
service/redis-svc-1                         ClusterIP   None             <none>        6379/TCP,16379/TCP   19m
service/redis-svc-2                         ClusterIP   None             <none>        6379/TCP,16379/TCP   19m
service/redis-svc-3                         ClusterIP   None             <none>        6379/TCP,16379/TCP   19m

NAME                                                     READY   AGE
statefulset.apps/drc-example-distributedrediscluster-0   2/2     142m
statefulset.apps/drc-example-distributedrediscluster-1   2/2     142m
statefulset.apps/drc-example-distributedrediscluster-2   2/2     142m


[root@k8s-master example]# kubectl get pvc
NAME                                                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
redis-data-drc-example-distributedrediscluster-0-0   Bound    pvc-54d02c21-bacb-469a-8937-f42e694a1beb   2Gi        RWO            rook-ceph-block   142m
redis-data-drc-example-distributedrediscluster-0-1   Bound    pvc-d876ae49-0d43-4c27-8b61-ef65f9985555   2Gi        RWO            rook-ceph-block   141m
redis-data-drc-example-distributedrediscluster-1-0   Bound    pvc-06248e41-f5c7-4a93-b842-d32424e439a5   2Gi        RWO            rook-ceph-block   142m
redis-data-drc-example-distributedrediscluster-1-1   Bound    pvc-2062915a-9008-4940-ab7c-ce025f923d19   2Gi        RWO            rook-ceph-block   141m
redis-data-drc-example-distributedrediscluster-2-0   Bound    pvc-43b60e7c-b526-401d-b016-912b18d0d531   2Gi        RWO            rook-ceph-block   142m
redis-data-drc-example-distributedrediscluster-2-1   Bound    pvc-88b9530b-8ecc-4445-a2d2-66c602c972af   2Gi        RWO            rook-ceph-block   141m
```

**缩容时删除存储。**



### 5.测试

```shell
# 查看
[root@k8s-master example]# kubectl get distributedrediscluster
NAME                              MASTERSIZE   STATUS    AGE
example-distributedrediscluster   3            Healthy   142m


[root@k8s-master example]# kubectl get all -l redis.kun/name=example-distributedrediscluster
NAME                                          READY   STATUS    RESTARTS   AGE
pod/drc-example-distributedrediscluster-0-0   1/1     Running   0          2m6s
pod/drc-example-distributedrediscluster-0-1   1/1     Running   0          75s
pod/drc-example-distributedrediscluster-1-0   1/1     Running   0          2m6s
pod/drc-example-distributedrediscluster-1-1   1/1     Running   0          89s
pod/drc-example-distributedrediscluster-2-0   1/1     Running   0          2m6s
pod/drc-example-distributedrediscluster-2-1   1/1     Running   0          79s

NAME                                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)              AGE
service/example-distributedrediscluster     ClusterIP   10.99.154.50   <none>        6379/TCP,16379/TCP   2m6s
service/example-distributedrediscluster-0   ClusterIP   None           <none>        6379/TCP,16379/TCP   2m6s
service/example-distributedrediscluster-1   ClusterIP   None           <none>        6379/TCP,16379/TCP   2m6s
service/example-distributedrediscluster-2   ClusterIP   None           <none>        6379/TCP,16379/TCP   2m6s

NAME                                                     READY   AGE
statefulset.apps/drc-example-distributedrediscluster-0   2/2     2m6s
statefulset.apps/drc-example-distributedrediscluster-1   2/2     2m6s
statefulset.apps/drc-example-distributedrediscluster-2   2/2     2m6s


[root@k8s-master example]# kubectl get pvc
NAME                                                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
redis-data-drc-example-distributedrediscluster-0-0   Bound    pvc-5819bec5-c5be-4778-9458-64985b544a45   2Gi        RWO            rook-ceph-block   2m50s
redis-data-drc-example-distributedrediscluster-0-1   Bound    pvc-dfd22e43-4b69-4b6e-bb87-cb8c5d2444b7   2Gi        RWO            rook-ceph-block   119s
redis-data-drc-example-distributedrediscluster-1-0   Bound    pvc-f69b1d94-a79c-40ff-9312-a161768ecaed   2Gi        RWO            rook-ceph-block   2m50s
redis-data-drc-example-distributedrediscluster-1-1   Bound    pvc-a2f134df-ec93-42cc-873c-20bd506cf675   2Gi        RWO            rook-ceph-block   2m13s
redis-data-drc-example-distributedrediscluster-2-0   Bound    pvc-437975b5-71b5-489b-b229-def84abb4453   2Gi        RWO            rook-ceph-block   2m50s
redis-data-drc-example-distributedrediscluster-2-1   Bound    pvc-c7123d07-a5e9-4ac0-8dc0-ee62026f41e0   2Gi        RWO            rook-ceph-block   2m3s
```

```shell
# 进入容器链接集群
# example-distributedrediscluster 集群service


[root@k8s-master example]# kubectl exec -ti drc-example-distributedrediscluster-0-0 -- sh
/data # redis-cli -h example-distributedrediscluster
example-distributedrediscluster:6379> 


# 设置数据
example-distributedrediscluster:6379> set a b
OK
example-distributedrediscluster:6379> set c d
(error) MOVED 7365 10.244.36.117:6379
example-distributedrediscluster:6379> set c d
(error) MOVED 7365 10.244.36.117:6379
example-distributedrediscluster:6379> quit
/data # redis-cli -h 10.244.36.117
10.244.36.117:6379> set c d
OK
10.244.36.117:6379> get c
"d"
10.244.36.117:6379> get a
(error) MOVED 15495 10.244.107.230:6379
10.244.36.117:6379> exit
/data # redis-cli -h 10.244.107.230
10.244.107.230:6379> get a
"b"
10.244.107.230:6379> get c
(error) MOVED 7365 10.244.36.117:6379
10.244.107.230:6379> 
10.244.107.230:6379> exit
/data # exit


# 集群配置信息，需要持久化
[root@k8s-master example]# kubectl exec -ti drc-example-distributedrediscluster-0-0 -- sh
/data # ls
dump.rdb        lost+found      nodes.conf      redis_password
/data # 
/data # vi nodes.conf

c97b583c1431519aa7bc8816e2f417a486734cfe 10.244.36.117:6379@16379 master - 0 1638167534000 7 connected 0-1364 5461 6827-10922
8b1d97063d22f8ec8e5bf1afc22bdd5d73518424 10.244.169.172:6379@16379 master - 0 1638167536693 9 connected 1365-5460 10924-12287
114dcc4b04dcd349dc0555aeef71f2a12bb2f9b2 10.244.107.237:6379@16379 myself,slave 8b1d97063d22f8ec8e5bf1afc22bdd5d73518424 0 1638167535
79f1ee91773885cfea39eeb7a0df4013bf754377 10.244.36.126:6379@16379 slave a219535763238726f0ac529f18e5a65aa8262a14 0 1638167536000 8 co
a219535763238726f0ac529f18e5a65aa8262a14 10.244.107.230:6379@16379 master - 0 1638167536000 8 connected 5462-6826 10923 12288-16383
92ccfcf03c988a82030b3f12d009c13bafa5f062 10.244.169.150:6379@16379 slave c97b583c1431519aa7bc8816e2f417a486734cfe 0 1638167537680 7 c
vars currentEpoch 9 lastVoteEpoch 0
```



### 6.Spring Boot测试

#### 6.1.服务配置

msa-k8s-redis.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-k8s-redis
  labels:
    app: msa-k8s-redis
spec:
  type: ClusterIP
  ports:
    - port: 8133
      targetPort: 8133
  selector:
    app: msa-k8s-redis

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-k8s-redis
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-k8s-redis
  template:
    metadata:
      labels:
        app: msa-k8s-redis
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-k8s-redis
          image: 172.51.216.85:8888/springcloud/msa-k8s-redis:1.4.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8133
          env:
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
```



application-k8s.yml

```yaml
# Redis集群配置

server:
  port: 8133

spring:
  application:
    name: msa-k8s-redis
  #redis连接
  redis:
    cluster:
      #nodes: 10.244.107.228:6379,10.244.36.70:6379,10.244.169.155:6379,10.244.107.232:6379,10.244.36.80:6379,10.244.169.158:6379
      nodes: example-distributedrediscluster-0.default.svc.cluster.local:6379,example-distributedrediscluster-1.default.svc.cluster.local:6379,example-distributedrediscluster-2.default.svc.cluster.local:6379
    database: 0
    

# 说明
# 配置：
example-distributedrediscluster-0；
example-distributedrediscluster-1；
example-distributedrediscluster-2；
#完整配置
nodes: example-distributedrediscluster-0.default.svc.cluster.local:6379,example-distributedrediscluster-1.default.svc.cluster.local:6379,example-distributedrediscluster-2.default.svc.cluster.local:6379


[root@k8s-master operator]# kubectl get svc
NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE
example-distributedrediscluster     ClusterIP   10.99.154.50     <none>        6379/TCP,16379/TCP   134m
example-distributedrediscluster-0   ClusterIP   None             <none>        6379/TCP,16379/TCP   134m
example-distributedrediscluster-1   ClusterIP   None             <none>        6379/TCP,16379/TCP   134m
example-distributedrediscluster-2   ClusterIP   None             <none>        6379/TCP,16379/TCP   134m
```



#### 6.2.创建服务

```shell
docker build -t 172.51.216.85:8888/springcloud/msa-k8s-redis:1.4.0 .

docker push 172.51.216.85:8888/springcloud/msa-k8s-redis:1.4.0


[root@k8s-master operator]# kubectl apply -f msa-k8s-redis.yaml 
service/msa-k8s-redis created
deployment.apps/msa-k8s-redis created


[root@k8s-master operator]# kubectl get all -n dev | grep msa-k8s-redis
pod/msa-k8s-redis-5bb9675b94-btr5w         1/1     Running            0          45s
pod/msa-k8s-redis-5bb9675b94-gtlrd         1/1     Running            0          45s
pod/msa-k8s-redis-5bb9675b94-w7zvw         1/1     Running            0          45s

service/msa-k8s-redis            ClusterIP   10.101.202.206   <none>        8133/TCP          45s

deployment.apps/msa-k8s-redis         3/3     3            3           45s

replicaset.apps/msa-k8s-redis-5bb9675b94         3         3         3       45s
```

```shell
# 测试

curl 10.101.202.206:8133/hello
curl 10.101.202.206:8133/say


[root@k8s-master operator]# curl 10.98.49.197:8133/hello
hello，this is redis messge! ### Mon Nov 29 17:04:39 CST 2021
[root@k8s-master operator]# curl 10.98.49.197:8133/say
say，this is first messge！ $$$Mon Nov 29 17:04:49 CST 2021


[root@k8s-master ~]# kubectl exec -it drc-example-distributedrediscluster-0-0 -- sh
/data # redis-cli -h example-distributedrediscluster
example-distributedrediscluster:6379> get a
(error) MOVED 15495 10.244.36.70:6379
example-distributedrediscluster:6379> get b
(error) MOVED 3300 10.244.107.232:6379


/data # redis-cli -h 10.244.36.70
10.244.36.70:6379> get a
"hello\xef\xbc\x8cthis is redis messge! ### Mon Nov 29 16:50:31 CST 2021"


10.244.36.70:6379> exit
/data # redis-cli -h 10.244.107.232
10.244.107.232:6379> get b
"say\xef\xbc\x8cthis is first messge\xef\xbc\x81 $$$Mon Nov 29 16:50:38 CST 2021"
```



#### 6.3.新建Service

```yaml
# 查看pod标签
[root@k8s-master operator]# kubectl get pod --show-labels
NAME                                      READY   STATUS    RESTARTS   AGE     LABELS
drc-example-distributedrediscluster-0-0   1/1     Running   0          139m    controller-revision-hash=drc-example-distributedrediscluster-0-7fbbcc6f5d,managed-by=redis-cluster-operator,redis.kun/name=example-distributedrediscluster,statefulSet=drc-example-distributedrediscluster-0,statefulset.kubernetes.io/pod-name=drc-example-distributedrediscluster-0-0
drc-example-distributedrediscluster-0-1   1/1     Running   0          138m    controller-revision-hash=drc-example-distributedrediscluster-0-7fbbcc6f5d,managed-by=redis-cluster-operator,redis.kun/name=example-distributedrediscluster,statefulSet=drc-example-distributedrediscluster-0,statefulset.kubernetes.io/pod-name=drc-example-distributedrediscluster-0-1
drc-example-distributedrediscluster-1-0   1/1     Running   0          139m    controller-revision-hash=drc-example-distributedrediscluster-1-79c5bc577,managed-by=redis-cluster-operator,redis.kun/name=example-distributedrediscluster,statefulSet=drc-example-distributedrediscluster-1,statefulset.kubernetes.io/pod-name=drc-example-distributedrediscluster-1-0
drc-example-distributedrediscluster-1-1   1/1     Running   0          138m    controller-revision-hash=drc-example-distributedrediscluster-1-79c5bc577,managed-by=redis-cluster-operator,redis.kun/name=example-distributedrediscluster,statefulSet=drc-example-distributedrediscluster-1,statefulset.kubernetes.io/pod-name=drc-example-distributedrediscluster-1-1
drc-example-distributedrediscluster-2-0   1/1     Running   0          139m    controller-revision-hash=drc-example-distributedrediscluster-2-77f45c9f48,managed-by=redis-cluster-operator,redis.kun/name=example-distributedrediscluster,statefulSet=drc-example-distributedrediscluster-2,statefulset.kubernetes.io/pod-name=drc-example-distributedrediscluster-2-0
drc-example-distributedrediscluster-2-1   1/1     Running   0          138m    controller-revision-hash=drc-example-distributedrediscluster-2-77f45c9f48,managed-by=redis-cluster-operator,redis.kun/name=example-distributedrediscluster,statefulSet=drc-example-distributedrediscluster-2,statefulset.kubernetes.io/pod-name=drc-example-distributedrediscluster-2-1


# redis pod 标识标签
statefulset.kubernetes.io/pod-name=drc-example-distributedrediscluster-0-0
statefulset.kubernetes.io/pod-name=drc-example-distributedrediscluster-0-1
statefulset.kubernetes.io/pod-name=drc-example-distributedrediscluster-1-0
statefulset.kubernetes.io/pod-name=drc-example-distributedrediscluster-1-1
statefulset.kubernetes.io/pod-name=drc-example-distributedrediscluster-2-0
statefulset.kubernetes.io/pod-name=drc-example-distributedrediscluster-2-1
```

redis-svc-1.yaml

```yaml
# redis-svc-1.yaml


apiVersion: v1
kind: Service
metadata:
  namespace: default
  name: redis-svc-1
  labels:
    statefulset.kubernetes.io/pod-name: drc-example-distributedrediscluster-0-0
spec:
  type: NodePort
  ports:
    - port: 6379
      name: redis-svc-1
      targetPort: 6379
      nodePort: 30901
  selector:
    statefulset.kubernetes.io/pod-name: drc-example-distributedrediscluster-0-0
```

redis-svc-2.yaml

```yaml
# redis-svc-2.yaml


apiVersion: v1
kind: Service
metadata:
  namespace: default
  name: redis-svc-2
  labels:
    statefulset.kubernetes.io/pod-name: drc-example-distributedrediscluster-0-1
spec:
  type: NodePort
  ports:
    - port: 6379
      name: redis-svc-2
      targetPort: 6379
      nodePort: 30902
  selector:
    statefulset.kubernetes.io/pod-name: drc-example-distributedrediscluster-0-1
```

redis-svc-3.yaml

```yaml
# redis-svc-3.yaml


apiVersion: v1
kind: Service
metadata:
  namespace: default
  name: redis-svc-3
  labels:
    statefulset.kubernetes.io/pod-name: drc-example-distributedrediscluster-1-0
spec:
  type: NodePort
  ports:
    - port: 6379
      name: redis-svc-3
      targetPort: 6379
      nodePort: 30903
  selector:
    statefulset.kubernetes.io/pod-name: drc-example-distributedrediscluster-1-0
```

redis-svc-4.yaml

```yaml
# redis-svc-4.yaml


apiVersion: v1
kind: Service
metadata:
  namespace: default
  name: redis-svc-4
  labels:
    statefulset.kubernetes.io/pod-name: drc-example-distributedrediscluster-1-1
spec:
  type: NodePort
  ports:
    - port: 6379
      name: redis-svc-4
      targetPort: 6379
      nodePort: 30904
  selector:
    statefulset.kubernetes.io/pod-name: drc-example-distributedrediscluster-1-1
```

redis-svc-5.yaml

```yaml
# redis-svc-5.yaml


apiVersion: v1
kind: Service
metadata:
  namespace: default
  name: redis-svc-5
  labels:
    statefulset.kubernetes.io/pod-name: drc-example-distributedrediscluster-2-0
spec:
  type: NodePort
  ports:
    - port: 6379
      name: redis-svc-5
      targetPort: 6379
      nodePort: 30905
  selector:
    statefulset.kubernetes.io/pod-name: drc-example-distributedrediscluster-2-0
```

redis-svc-6.yaml

```yaml
# redis-svc-6.yaml


apiVersion: v1
kind: Service
metadata:
  namespace: default
  name: redis-svc-6
  labels:
    statefulset.kubernetes.io/pod-name: drc-example-distributedrediscluster-2-1
spec:
  type: NodePort
  ports:
    - port: 6379
      name: redis-svc-6
      targetPort: 6379
      nodePort: 30906
  selector:
    statefulset.kubernetes.io/pod-name: drc-example-distributedrediscluster-2-1
```



创建服务

```shell
# kubectl apply -f redis-svc-1.yaml
# kubectl apply -f redis-svc-2.yaml
# kubectl apply -f redis-svc-3.yaml
# kubectl apply -f redis-svc-4.yaml
# kubectl apply -f redis-svc-5.yaml
# kubectl apply -f redis-svc-6.yaml


[root@k8s-master operator]# kubectl get svc
NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE
example-distributedrediscluster     ClusterIP   10.99.154.50     <none>        6379/TCP,16379/TCP   5h21m
example-distributedrediscluster-0   ClusterIP   None             <none>        6379/TCP,16379/TCP   5h21m
example-distributedrediscluster-1   ClusterIP   None             <none>        6379/TCP,16379/TCP   5h21m
example-distributedrediscluster-2   ClusterIP   None             <none>        6379/TCP,16379/TCP   5h21m
redis-svc-1                         NodePort    10.97.60.200     <none>        6379:30901/TCP       72s
redis-svc-2                         NodePort    10.96.116.98     <none>        6379:30902/TCP       29s
redis-svc-3                         NodePort    10.109.25.127    <none>        6379:30903/TCP       23s
redis-svc-4                         NodePort    10.108.160.149   <none>        6379:30904/TCP       18s
redis-svc-5                         NodePort    10.101.191.132   <none>        6379:30905/TCP       11s
redis-svc-6                         NodePort    10.103.173.119   <none>        6379:30906/TCP       4s
```

application-k8s.yml

```yaml
# Redis集群配置


server:
  port: 8133

spring:
  application:
    name: msa-k8s-redis
  #redis连接
  redis:
    cluster:
      #nodes: 10.244.107.228:6379,10.244.36.70:6379,10.244.169.155:6379,10.244.107.232:6379,10.244.36.80:6379,10.244.169.158:6379
      #nodes: example-distributedrediscluster-0.default.svc.cluster.local:6379,example-distributedrediscluster-1.default.svc.cluster.local:6379,example-distributedrediscluster-2.default.svc.cluster.local:6379
      nodes: redis-svc-1.default.svc.cluster.local:6379,redis-svc-2.default.svc.cluster.local:6379,redis-svc-3.default.svc.cluster.local:6379,redis-svc-4.default.svc.cluster.local:6379,redis-svc-5.default.svc.cluster.local:6379,redis-svc-6.default.svc.cluster.local:6379
    database: 0
    


# 说明
# 配置：
redis-svc-1；
redis-svc-2；
redis-svc-3；
redis-svc-4；
redis-svc-5；
redis-svc-6；
#完整配置
nodes: redis-svc-1.default.svc.cluster.local:6379,redis-svc-2.default.svc.cluster.local:6379,redis-svc-3.default.svc.cluster.local:6379,redis-svc-4.default.svc.cluster.local:6379,redis-svc-5.default.svc.cluster.local:6379,redis-svc-6.default.svc.cluster.local:6379



redis-svc-1.default.svc.cluster.local:6379
redis-svc-2.default.svc.cluster.local:6379
redis-svc-3.default.svc.cluster.local:6379
redis-svc-4.default.svc.cluster.local:6379
redis-svc-5.default.svc.cluster.local:6379
redis-svc-6.default.svc.cluster.local:6379
```



#### 6.4.客户端

![](/images/kubernetes/middleware/redis-3.png)

![](/images/kubernetes/middleware/redis-4.png)



#### 6.5.总结

**1.开发测试：**

每个Redis实例单独配置service，以NodePort方式暴露端口；

```shell
# Spring Boot配置


nodes: redis-svc-1.default.svc.cluster.local:6379,redis-svc-2.default.svc.cluster.local:6379,redis-svc-3.default.svc.cluster.local:6379,redis-svc-4.default.svc.cluster.local:6379,redis-svc-5.default.svc.cluster.local:6379,redis-svc-6.default.svc.cluster.local:6379


redis-svc-1.default.svc.cluster.local:6379
redis-svc-2.default.svc.cluster.local:6379
redis-svc-3.default.svc.cluster.local:6379
redis-svc-4.default.svc.cluster.local:6379
redis-svc-5.default.svc.cluster.local:6379
redis-svc-6.default.svc.cluster.local:6379
```



**2.集群内服访问**

```shell
# Spring Boot

# 方式一
nodes: redis-svc-1.default.svc.cluster.local:6379,redis-svc-2.default.svc.cluster.local:6379,redis-svc-3.default.svc.cluster.local:6379,redis-svc-4.default.svc.cluster.local:6379,redis-svc-5.default.svc.cluster.local:6379,redis-svc-6.default.svc.cluster.local:6379


# 方式二
nodes: example-distributedrediscluster-0.default.svc.cluster.local:6379,example-distributedrediscluster-1.default.svc.cluster.local:6379,example-distributedrediscluster-2.default.svc.cluster.local:6379
```



### 7.删除集群

 ```shell
 # 删除集群
 
 # 根据创建选择
 kubectl delete -f deploy/example/persistent.yaml
 
 
 kubectl delete -f deploy/cluster/operator.yaml
 kubectl delete -f deploy/cluster/cluster_role_binding.yaml
 kubectl delete -f deploy/cluster/cluster_role.yaml
 kubectl delete -f deploy/service_account.yaml
 kubectl delete -f deploy/crds/redis.kun_redisclusterbackups_crd.yaml
 kubectl delete -f deploy/crds/redis.kun_distributedredisclusters_crd.yaml
 ```

```shell
# 查看
[root@k8s-master operator]# kubectl get distributedrediscluster
NAME                              MASTERSIZE   STATUS    AGE
example-distributedrediscluster   3            Healthy   19h
[root@k8s-master operator]# kubectl get all -l redis.kun/name=example-distributedrediscluster
NAME                                          READY   STATUS    RESTARTS   AGE
pod/drc-example-distributedrediscluster-0-0   1/1     Running   0          19h
pod/drc-example-distributedrediscluster-0-1   1/1     Running   0          19h
pod/drc-example-distributedrediscluster-1-0   1/1     Running   0          19h
pod/drc-example-distributedrediscluster-1-1   1/1     Running   0          19h
pod/drc-example-distributedrediscluster-2-0   1/1     Running   0          19h
pod/drc-example-distributedrediscluster-2-1   1/1     Running   0          19h

NAME                                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)              AGE
service/example-distributedrediscluster     ClusterIP   10.99.154.50   <none>        6379/TCP,16379/TCP   19h
service/example-distributedrediscluster-0   ClusterIP   None           <none>        6379/TCP,16379/TCP   19h
service/example-distributedrediscluster-1   ClusterIP   None           <none>        6379/TCP,16379/TCP   19h
service/example-distributedrediscluster-2   ClusterIP   None           <none>        6379/TCP,16379/TCP   19h

NAME                                                     READY   AGE
statefulset.apps/drc-example-distributedrediscluster-0   2/2     19h
statefulset.apps/drc-example-distributedrediscluster-1   2/2     19h
statefulset.apps/drc-example-distributedrediscluster-2   2/2     19h
```

```shell
# 删除集群
[root@k8s-master redis-cluster-operator]# kubectl delete -f deploy/example/persistent.yaml
distributedrediscluster.redis.kun "example-distributedrediscluster" deleted

# 查看
[root@k8s-master redis-cluster-operator]# kubectl get distributedrediscluster
No resources found in default namespace.
[root@k8s-master redis-cluster-operator]# kubectl get all -l redis.kun/name=example-distributedrediscluster
No resources found in default namespace.
[root@k8s-master redis-cluster-operator]# kubectl get pvc
No resources found in default namespace.


# 删除operator
[root@k8s-master redis-cluster-operator]# kubectl get pod
NAME                                      READY   STATUS    RESTARTS   AGE
redis-cluster-operator-6669898858-775vj   1/1     Running   0          23h

[root@k8s-master redis-cluster-operator]# kubectl delete -f deploy/cluster/operator.yaml
deployment.apps "redis-cluster-operator" deleted
configmap "redis-admin" deleted


# 删除剩余资源
[root@k8s-master redis-cluster-operator]# kubectl delete -f deploy/cluster/cluster_role_binding.yaml
clusterrolebinding.rbac.authorization.k8s.io "redis-cluster-operator" deleted
[root@k8s-master redis-cluster-operator]# kubectl delete -f deploy/cluster/cluster_role.yaml
clusterrole.rbac.authorization.k8s.io "redis-cluster-operator" deleted
[root@k8s-master redis-cluster-operator]# kubectl delete -f deploy/service_account.yaml
serviceaccount "redis-cluster-operator" deleted
[root@k8s-master redis-cluster-operator]# kubectl delete -f deploy/crds/redis.kun_redisclusterbackups_crd.yaml
Warning: apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
customresourcedefinition.apiextensions.k8s.io "redisclusterbackups.redis.kun" deleted
[root@k8s-master redis-cluster-operator]# kubectl delete -f deploy/crds/redis.kun_distributedredisclusters_crd.yaml
Warning: apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
customresourcedefinition.apiextensions.k8s.io "distributedredisclusters.redis.kun" deleted
```



