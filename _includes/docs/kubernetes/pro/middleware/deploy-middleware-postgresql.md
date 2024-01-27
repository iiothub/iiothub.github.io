* TOC
{:toc}


### 1.安装方式

```shell
# 安装
PostgreSQL #2	zalando-incubator/postgres-operator
https://github.com/zalando/postgres-operator

https://postgres-operator.readthedocs.io/en/latest/

Quickstart
https://github.com/zalando/postgres-operator/blob/master/docs/quickstart.md#deployment-options
```

```shell
# 1.下载源码

git clone https://github.com/zalando/postgres-operator.git
cd postgres-operator
```



### 2.安装Operator

```shell
# 安装步骤

# apply the manifests in the following order
kubectl create -f manifests/configmap.yaml  # configuration
kubectl create -f manifests/operator-service-account-rbac.yaml  # identity and permissions
kubectl create -f manifests/postgres-operator.yaml  # deployment
kubectl create -f manifests/api-service.yaml  # operator API to be used by UI
```

```shell
# 创建Operator

[root@k8s-master postgres-operator]# kubectl create -f manifests/configmap.yaml
configmap/postgres-operator created
[root@k8s-master postgres-operator]# kubectl create -f manifests/operator-service-account-rbac.yaml
serviceaccount/postgres-operator created
clusterrole.rbac.authorization.k8s.io/postgres-operator created
clusterrolebinding.rbac.authorization.k8s.io/postgres-operator created
clusterrole.rbac.authorization.k8s.io/postgres-pod created
[root@k8s-master postgres-operator]# kubectl create -f manifests/postgres-operator.yaml
deployment.apps/postgres-operator created
[root@k8s-master postgres-operator]# kubectl create -f manifests/api-service.yaml
service/postgres-operator created


[root@k8s-master postgres-operator]# kubectl get all
NAME                                     READY   STATUS    RESTARTS   AGE
pod/postgres-operator-569b58b8c6-6qdlw   1/1     Running   0          53s

NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/kubernetes          ClusterIP   10.96.0.1        <none>        443/TCP          114d
service/pg-svc-master       NodePort    10.104.70.22     <none>        5432:31669/TCP   39m
service/pg-svc-replica      NodePort    10.105.246.108   <none>        5432:30871/TCP   36m
service/postgres-operator   ClusterIP   10.98.87.50      <none>        8080/TCP         42s

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/postgres-operator   1/1     1            1           53s

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/postgres-operator-569b58b8c6   1         1         1       53s
```

```shell
# 无法下载镜像  registry.opensource.zalan.do/acid/postgres-operator:v1.7.1
# 通过阿里云镜像服务拉取镜像
# 详细信息参考拉取墙外镜像文档

# 登录阿里云Docker Registry
# 公有仓库可以不登录

$ docker login --username=hollysyshs registry.cn-beijing.aliyuncs.com
# 用于登录的用户名为阿里云账号全名，密码为开通服务时设置的密码。
# 您可以在访问凭证页面修改凭证密码。



# 拉取镜像
# docker pull registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-postgres-operator:[镜像版本号]
docker pull registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-postgres-operator:v1.7.1


# 打标签
docker tag registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-postgres-operator:v1.7.1 registry.opensource.zalan.do/acid/postgres-operator:v1.7.1
```



### 3.修改配置

```shell
# vim complete-postgres-manifest.yaml 

  volume:
    size: 20Gi
    storageClass: "rook-ceph-block"
```



### 4.安装Postgres集群

```shell
# create a Postgres cluster
kubectl create -f manifests/complete-postgres-manifest.yaml

# check the deployed cluster
kubectl get postgresql

# check created database pods
kubectl get pods -l application=spilo -L spilo-role

# check created service resources
kubectl get svc -l application=spilo -L spilo-role
```

```shell
[root@k8s-master postgres-operator]# kubectl create -f manifests/complete-postgres-manifest.yaml
postgresql.acid.zalan.do/acid-test-cluster created


# 查看
[root@k8s-master postgres-operator]# kubectl get all
NAME                                     READY   STATUS    RESTARTS   AGE
pod/acid-test-cluster-0                  1/1     Running   0          80s
pod/acid-test-cluster-1                  1/1     Running   0          41s
pod/postgres-operator-569b58b8c6-6qdlw   1/1     Running   0          10m

NAME                               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/acid-test-cluster          ClusterIP   10.105.111.206   <none>        5432/TCP         81s
service/acid-test-cluster-config   ClusterIP   None             <none>        <none>           38s
service/acid-test-cluster-repl     ClusterIP   10.103.24.96     <none>        5432/TCP         81s
service/postgres-operator          ClusterIP   10.98.87.50      <none>        8080/TCP         10m

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/postgres-operator   1/1     1            1           10m

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/postgres-operator-569b58b8c6   1         1         1       10m

NAME                                 READY   AGE
statefulset.apps/acid-test-cluster   2/2     81s

NAME                                         TEAM   VERSION   PODS   VOLUME   CPU-REQUEST   MEMORY-REQUEST   AGE   STATUS
postgresql.acid.zalan.do/acid-test-cluster   acid   14        2      20Gi     10m           100Mi            82s   Running


[root@k8s-master postgres-operator]# kubectl get pvc
NAME                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
pgdata-acid-test-cluster-0   Bound    pvc-6b5cdb31-d549-4bf7-9742-c7ba5a9f5c37   20Gi       RWO            rook-ceph-block   109s
pgdata-acid-test-cluster-1   Bound    pvc-4bb98bf7-b58e-4368-93f4-7fa2ac2c5f17   20Gi       RWO            rook-ceph-block   70s
```

```shell
# 无法下载镜像  registry.opensource.zalan.do/acid/spilo-14:2.1-p3
# 通过阿里云镜像服务拉取镜像


# 拉取镜像
# docker pull registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-spilo-14:[镜像版本号]
docker pull registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-spilo-14:2.1-p3


# 打标签
docker tag  registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-spilo-14:2.1-p3 registry.opensource.zalan.do/acid/spilo-14:2.1-p3
```



### 5.外部访问数据库

**1.新建service**

```shell
# 新建service


# pg-svc-master.yaml

apiVersion: v1
kind: Service
metadata:
  namespace: default
  name: pg-svc-master
  labels:
    spilo-role: master
spec:
  type: NodePort
  ports:
    - port: 5432
      name: pg-svc-master
      targetPort: 5432
  selector:
    spilo-role: master


# pg-svc-replica.yaml

apiVersion: v1
kind: Service
metadata:
  namespace: default
  name: pg-svc-replica
  labels:
    spilo-role: replica
spec:
  type: NodePort
  ports:
    - port: 5432
      name: pg-svc-replica
      targetPort: 5432
  selector:
    spilo-role: replica
    


[root@k8s-master postgres-operator]# kubectl get svc
NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
acid-test-cluster          ClusterIP   10.105.111.206   <none>        5432/TCP         7m54s
acid-test-cluster-config   ClusterIP   None             <none>        <none>           7m11s
acid-test-cluster-repl     ClusterIP   10.103.24.96     <none>        5432/TCP         7m54s

pg-svc-master              NodePort    10.104.70.22     <none>        5432:31669/TCP   55m
pg-svc-replica             NodePort    10.105.246.108   <none>        5432:30871/TCP   52m
```



```shell
# 修改主数据库密码 postgres

[root@k8s-master ~]#  kubectl get pod
NAME                                 READY   STATUS    RESTARTS   AGE
acid-minimal-cluster-0               1/1     Running   0          17h
acid-minimal-cluster-1               1/1     Running   0          16h
postgres-operator-569b58b8c6-dqfj2   1/1     Running   0          19h


[root@k8s-master operator]# kubectl exec -it acid-test-cluster-0 -- bash

 ____        _ _
/ ___| _ __ (_) | ___
\___ \| '_ \| | |/ _ \
 ___) | |_) | | | (_) |
|____/| .__/|_|_|\___/
      |_|

This container is managed by runit, when stopping/starting services use sv

Examples:

sv stop cron
sv restart patroni

Current status: (sv status /etc/service/*)

run: /etc/service/patroni: (pid 33) 552s
run: /etc/service/pgqd: (pid 34) 552s
root@acid-test-cluster-0:/home/postgres# su - postgres
postgres@acid-minimal-cluster-0:~$ psql
psql (14.0 (Ubuntu 14.0-1.pgdg18.04+1))
Type "help" for help.

postgres=# ALTER USER postgres with encrypted password 'postgres';
ALTER ROLE
postgres=# \d
                      List of relations
 Schema |          Name           |     Type      |  Owner   
--------+-------------------------+---------------+----------
 public | failed_authentication_0 | view          | postgres
 public | failed_authentication_1 | view          | postgres
 public | failed_authentication_2 | view          | postgres
 public | failed_authentication_3 | view          | postgres
 public | failed_authentication_4 | view          | postgres
 public | failed_authentication_5 | view          | postgres
 public | failed_authentication_6 | view          | postgres
 public | failed_authentication_7 | view          | postgres
 public | pg_auth_mon             | view          | postgres
 public | pg_stat_kcache          | view          | postgres
 public | pg_stat_kcache_detail   | view          | postgres
 public | pg_stat_statements      | view          | postgres
 public | pg_stat_statements_info | view          | postgres
 public | postgres_log            | table         | postgres
 public | postgres_log_0          | foreign table | postgres
 public | postgres_log_1          | foreign table | postgres
 public | postgres_log_2          | foreign table | postgres
 public | postgres_log_3          | foreign table | postgres
 public | postgres_log_4          | foreign table | postgres
 public | postgres_log_5          | foreign table | postgres
 public | postgres_log_6          | foreign table | postgres
 public | postgres_log_7          | foreign table | postgres
(22 rows)

postgres=# \du
                                                     List of roles
    Role name    |                         Attributes                         |               Member of                
-----------------+------------------------------------------------------------+----------------------------------------
 admin           | Create DB, Cannot login                                    | {bar_owner,zalando,foo_user}
 bar_data_owner  | Cannot login                                               | {bar_data_reader,bar_data_writer}
 bar_data_reader | Cannot login                                               | {}
 bar_data_writer | Cannot login                                               | {bar_data_reader}
 bar_owner       | Cannot login                                               | {bar_reader,bar_writer,bar_data_owner}
 bar_reader      | Cannot login                                               | {}
 bar_writer      | Cannot login                                               | {bar_reader}
 foo_user        |                                                            | {}
 postgres        | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 robot_zmon      | Cannot login                                               | {}
 standby         | Replication                                                | {}
 zalando         | Superuser, Create DB                                       | {}
 zalandos        | Create DB, Cannot login                                    | {}

postgres=# \l
                                  List of databases
   Name    |   Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+-----------+----------+-------------+-------------+-----------------------
 bar       | bar_owner | UTF8     | en_US.utf-8 | en_US.utf-8 | 
 foo       | zalando   | UTF8     | en_US.utf-8 | en_US.utf-8 | 
 postgres  | postgres  | UTF8     | en_US.utf-8 | en_US.utf-8 | 
 template0 | postgres  | UTF8     | en_US.utf-8 | en_US.utf-8 | =c/postgres          +
           |           |          |             |             | postgres=CTc/postgres
 template1 | postgres  | UTF8     | en_US.utf-8 | en_US.utf-8 | =c/postgres          +
           |           |          |             |             | postgres=CTc/postgres
(5 rows)

postgres=# \q
postgres@acid-minimal-cluster-0:~$ 
```



**2.访问数据库**

```shell
# 访问地址

# 主
172.51.216.81
5432:31162
postgres
postgres

# 从
172.51.216.81
5432:31363
postgres
postgres
```



