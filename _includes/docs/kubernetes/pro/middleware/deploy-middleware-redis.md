* TOC
{:toc}


### 1.安装方式

```shell
# 安装

#Redis Cluster #2	ucloud/redis-cluster-operator
https://github.com/ucloud/redis-cluster-operator
```



**下载redis-cluster-operator**

```shell
# git  clone  https://github.com/ucloud/redis-cluster-operator.git


[root@k8s-master1 redis]# git  clone  https://github.com/ucloud/redis-cluster-operator.git
Cloning into 'redis-cluster-operator'...
remote: Enumerating objects: 2024, done.
remote: Counting objects: 100% (51/51), done.
remote: Compressing objects: 100% (30/30), done.
remote: Total 2024 (delta 15), reused 38 (delta 11), pack-reused 1973
Receiving objects: 100% (2024/2024), 944.24 KiB | 0 bytes/s, done.
Resolving deltas: 100% (1288/1288), done.
```



### 2.创建Operator

**1.创建自定义资源（CRD）**

```bash
$ kubectl create -f deploy/crds/redis.kun_distributedredisclusters_crd.yaml
$ kubectl create -f deploy/crds/redis.kun_redisclusterbackups_crd.yaml



[root@k8s-master1 redis-cluster-operator]# pwd
/k8s/middleware/redis/redis-cluster-operator
[root@k8s-master1 redis-cluster-operator]# ll
total 144
drwxr-xr-x.  3 root root     35 Dec 22 14:45 build
drwxr-xr-x.  3 root root     36 Dec 22 14:45 charts
drwxr-xr-x.  3 root root     21 Dec 22 14:45 cmd
drwxr-xr-x.  6 root root    108 Dec 22 14:45 deploy
drwxr-xr-x.  3 root root     20 Dec 22 14:45 doc
-rw-r--r--.  1 root root    993 Dec 22 14:45 Dockerfile
-rw-r--r--.  1 root root   2939 Dec 22 14:45 go.mod
-rw-r--r--.  1 root root 107647 Dec 22 14:45 go.sum
drwxr-xr-x.  5 root root     60 Dec 22 14:45 hack
-rw-r--r--.  1 root root  11333 Dec 22 14:45 LICENSE
-rw-r--r--.  1 root root   1797 Dec 22 14:45 Makefile
drwxr-xr-x. 12 root root    148 Dec 22 14:45 pkg
-rw-r--r--.  1 root root   7871 Dec 22 14:45 README.md
drwxr-xr-x.  2 root root     31 Dec 22 14:45 static
drwxr-xr-x.  4 root root     35 Dec 22 14:45 test
-rw-r--r--.  1 root root    149 Dec 22 14:45 tools.go
drwxr-xr-x.  2 root root     24 Dec 22 14:45 version


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



### 3.持久卷部署

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
[root@k8s-master1 redis-cluster-operator]# kubectl get all -l redis.kun/name=example-distributedrediscluster
NAME                                          READY   STATUS    RESTARTS   AGE
pod/drc-example-distributedrediscluster-0-0   1/1     Running   0          3m55s
pod/drc-example-distributedrediscluster-0-1   1/1     Running   0          2m37s
pod/drc-example-distributedrediscluster-1-0   1/1     Running   0          3m55s
pod/drc-example-distributedrediscluster-1-1   1/1     Running   0          2m39s
pod/drc-example-distributedrediscluster-2-0   1/1     Running   0          3m55s
pod/drc-example-distributedrediscluster-2-1   1/1     Running   0          86s

NAME                                        TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)              AGE
service/example-distributedrediscluster     ClusterIP   10.1.123.77   <none>        6379/TCP,16379/TCP   3m54s
service/example-distributedrediscluster-0   ClusterIP   None          <none>        6379/TCP,16379/TCP   3m54s
service/example-distributedrediscluster-1   ClusterIP   None          <none>        6379/TCP,16379/TCP   3m54s
service/example-distributedrediscluster-2   ClusterIP   None          <none>        6379/TCP,16379/TCP   3m54s

NAME                                                     READY   AGE
statefulset.apps/drc-example-distributedrediscluster-0   2/2     3m55s
statefulset.apps/drc-example-distributedrediscluster-1   2/2     3m55s
statefulset.apps/drc-example-distributedrediscluster-2   2/2     3m55s


[root@k8s-master redis-cluster-operator]# kubectl get distributedrediscluster
NAME                              MASTERSIZE   STATUS    AGE
example-distributedrediscluster   3            Healthy   3m7s
```

```shell
# 查看存储

[root@k8s-master example]# kubectl get pvc
NAME                                                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
redis-data-drc-example-distributedrediscluster-0-0   Bound    pvc-689ff6e5-7756-4a56-ba77-db1b6951af49   3Gi        RWO            rook-ceph-block   4m39s
redis-data-drc-example-distributedrediscluster-0-1   Bound    pvc-d1ef97dc-2e7e-41c2-b6aa-b9429c7add79   3Gi        RWO            rook-ceph-block   3m21s
redis-data-drc-example-distributedrediscluster-1-0   Bound    pvc-e441d5ba-5a64-4680-b52f-c22f5d23ca18   3Gi        RWO            rook-ceph-block   4m39s
redis-data-drc-example-distributedrediscluster-1-1   Bound    pvc-847a7eee-2cb8-4ae3-bd29-ae23275f3550   3Gi        RWO            rook-ceph-block   3m23s
redis-data-drc-example-distributedrediscluster-2-0   Bound    pvc-4cccf90e-63da-4dc5-9f19-be8e942bbd01   3Gi        RWO            rook-ceph-block   4m39s
redis-data-drc-example-distributedrediscluster-2-1   Bound    pvc-4a580424-4041-4491-a869-e386ceb903b6   3Gi        RWO            rook-ceph-block   2m10s


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



### 4.新建Service

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

```yaml
# Redis集群配置


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



### 5.客户端

![](/images/kubernetes/pro/middleware/redis-3.png)

![](/images/kubernetes/pro/middleware/redis-4.png)



