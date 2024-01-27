* TOC
{:toc}


### 1.MySQL安装方式

```shell
# 安装
# MySQL #3	presslabs/mysql-operator
https://github.com/bitpoke/mysql-operator


https://github.com/bitpoke/mysql-operator/blob/master/docs/_index.md
https://github.com/bitpoke/mysql-operator/blob/master/docs/deploy-mysql-cluster.md
```

```shell
# 部署集群

# 1.安装operator
helm repo add presslabs https://presslabs.github.io/charts
helm install presslabs/mysql-operator --name mysql-operator


# 2.安装mysql集群
kubectl apply -f https://raw.githubusercontent.com/bitpoke/mysql-operator/master/examples/example-cluster-secret.yaml
kubectl apply -f https://raw.githubusercontent.com/bitpoke/mysql-operator/master/examples/example-cluster.yaml
```

**下载rabbitmq/cluster-operator**

```shell
# git clone https://github.com/bitpoke/mysql-operator.git


# 下载上传
https://github.com/bitpoke/mysql-operator/archive/refs/tags/v0.5.2.tar.gz
```



### 2.部署MySQL集群

#### 2.1.Helm安装Operator

**注意：直接安装PVC会出问题，需要下载到本地，修改配置。**



**1.下载Helm安装包**

```shell
[root@k8s-master1 helm]# helm repo add presslabs https://presslabs.github.io/charts
"presslabs" has been added to your repositories

[root@k8s-master1 helm]# helm search repo mysql-operator
NAME                    	CHART VERSION	APP VERSION	DESCRIPTION                    
presslabs/mysql-operator	0.4.0        	v0.4.0     	A Helm chart for mysql operator

[root@k8s-master1 helm]# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "chartmuseum" chart repository
...Successfully got an update from the "harbor" chart repository
...Successfully got an update from the "elastic" chart repository
...Successfully got an update from the "ingress-nginx" chart repository
...Successfully got an update from the "presslabs" chart repository
...Successfully got an update from the "aliyun" chart repository
...Successfully got an update from the "gitlab" chart repository
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈


[root@k8s-master helm]# helm fetch presslabs/mysql-operator
[root@k8s-master helm]# tar -zxf mysql-operator-0.4.0.tgz
```



**2.修改配置**

```shell
[root@k8s-master1 mysql-operator]# pwd
/k8s/middleware/mysql/helm/mysql-operator
[root@k8s-master1 mysql-operator]# ll
total 24
-rwxr-xr-x 1 root root  279 Jun 17  2020 Chart.yaml
drwxr-xr-x 2 root root   85 Dec  1 17:26 crds
-rwxr-xr-x 1 root root 4545 Jun 17  2020 README.md
drwxr-xr-x 2 root root 4096 Dec  1 17:30 templates
-rwxr-xr-x 1 root root 7212 Dec  1 17:32 values.yaml


# 修改配置文件
[root@k8s-master mysql-operator]# vim values.yaml 
......
  persistence:
    enabled: true
    storageClass: "rook-ceph-block"
    accessMode: "ReadWriteOnce"
    size: 5Gi
......
```



**3.创建Operator**

```shell
# 安装
[root@k8s-master helm]# helm install mysql-operator mysql-operator
W1202 11:18:07.258634   61198 warnings.go:70] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1202 11:18:07.269227   61198 warnings.go:70] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1202 11:18:07.880685   61198 warnings.go:70] rbac.authorization.k8s.io/v1beta1 ClusterRoleBinding is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 ClusterRoleBinding
W1202 11:18:07.918011   61198 warnings.go:70] rbac.authorization.k8s.io/v1beta1 ClusterRoleBinding is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 ClusterRoleBinding
NAME: mysql-operator
LAST DEPLOYED: Thu Dec  2 11:18:07 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
You can create a new cluster by issuing:

cat <<EOF | kubectl apply -f-
apiVersion: mysql.presslabs.org/v1alpha1
kind: MysqlCluster
metadata:
  name: my-cluster
spec:
  replicas: 1
  secretName: my-cluster-secret
---
apiVersion: v1
kind: Secret
metadata:
  name: my-cluster-secret
type: Opaque
data:
  ROOT_PASSWORD: $(echo -n "not-so-secure" | base64)
EOF



[root@k8s-master1 helm]# helm list
NAME          	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART                 APP VERSION
mysql-operator	default  	1       	2021-12-21 16:14:27.996829056 +0800 CST	deployed	mysql-operator-0.4.0  v0.4.0  


NAME                           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)            AGE
service/kubernetes             ClusterIP   10.1.0.1       <none>        443/TCP            5d2h
service/mysql-operator         ClusterIP   10.1.249.252   <none>        80/TCP             109s
service/mysql-operator-0-svc   ClusterIP   10.1.239.169   <none>        80/TCP,10008/TCP   109s

NAME                              READY   AGE
statefulset.apps/mysql-operator   1/1     109s


[root@k8s-master1 helm]# kubectl get pvc
NAME                    STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
data-mysql-operator-0   Bound    pvc-ddbfb37a-6d9b-4a2d-ab40-7fc5c0676f1b   5Gi        RWO            rook-ceph-block   4m11s
```



#### 2.2.安装MySQL集群

**1.修改配置**

```shell
[root@k8s-master1 mysql]# tar -zxf mysql-operator-0.5.2.tar.gz 
[root@k8s-master1 mysql]# ll
total 328
drwxr-xr-x.  3 root root     60 Dec 21 16:11 helm
drwxrwxr-x. 12 root root   4096 Nov 23 17:53 mysql-operator-0.5.2
-rw-r--r--.  1 root root 329533 Dec 21 16:05 mysql-operator-0.5.2.tar.gz
[root@k8s-master1 mysql]# cd mysql-operator-0.5.2
[root@k8s-master1 mysql-operator-0.5.2]# cd examples/
[root@k8s-master1 examples]# ll
total 32
-rw-rw-r--. 1 root root  656 Nov 23 17:53 example-backup-secret.yaml
-rw-rw-r--. 1 root root  608 Nov 23 17:53 example-backup.yaml
-rw-rw-r--. 1 root root  331 Nov 23 17:53 example-cluster-init.yaml
-rw-rw-r--. 1 root root  256 Nov 23 17:53 example-cluster-secret.yaml
-rw-rw-r--. 1 root root 4103 Nov 23 17:53 example-cluster.yaml
-rw-rw-r--. 1 root root  184 Nov 23 17:53 example-database.yaml
-rw-rw-r--. 1 root root  533 Nov 23 17:53 example-user.yaml
```

```shell
[root@k8s-master1 examples]# pwd
/k8s/middleware/mysql/mysql-operator-0.5.2/examples
[root@k8s-master examples]# ll
total 32
-rw-r--r-- 1 root root  656 Dec  1 17:12 example-backup-secret.yaml
-rw-r--r-- 1 root root  608 Dec  1 17:12 example-backup.yaml
-rw-r--r-- 1 root root  331 Dec  1 17:12 example-cluster-init.yaml
-rw-r--r-- 1 root root  256 Dec  1 20:49 example-cluster-secret.yaml
-rw-r--r-- 1 root root 4131 Dec  1 20:53 example-cluster.yaml
-rw-r--r-- 1 root root  184 Dec  1 17:12 example-database.yaml
-rw-r--r-- 1 root root  533 Dec  1 17:12 example-user.yaml
```

```yaml
# example-cluster-secret.yaml


[root@k8s-master1 examples]# vim example-cluster-secret.yaml 

apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  # root password is required to be specified
  ROOT_PASSWORD: bXlwYXNz
  ## application credentials that will be created at cluster bootstrap
  # DATABASE:
  # USER:
  # PASSWORD:
  


# 修改root密码 ROOT_PASSWORD
[root@k8s-master examples]# echo -n root |base64
cm9vdA==
```

```yaml
# example-cluster.yam


[root@k8s-master1 examples]# vim example-cluster.yaml 

apiVersion: mysql.presslabs.org/v1alpha1
kind: MysqlCluster
metadata:
  name: my-cluster
spec:
  replicas: 2
  secretName: my-secret
......


# Rook: rook-ceph-block
# 修改配置
   volumeSpec:
     persistentVolumeClaim:
       accessModes: [ "ReadWriteOnce" ]
       resources:
         requests:
           storage: 25Gi
       storageClassName: rook-ceph-block
```



**2.安装MySQL**

```shell
[root@k8s-master1 examples]# kubectl apply -f example-cluster-secret.yaml 
secret/my-secret created
[root@k8s-master1 examples]# kubectl apply -f example-cluster.yaml 
mysqlcluster.mysql.presslabs.org/my-cluster created


# 查看
[root@k8s-master1 examples]# kubectl get all
NAME                     READY   STATUS    RESTARTS   AGE
pod/my-cluster-mysql-0   4/4     Running   0          23m
pod/my-cluster-mysql-1   4/4     Running   0          19m
pod/mysql-operator-0     2/2     Running   0          41m

NAME                                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE
service/kubernetes                  ClusterIP   10.1.0.1       <none>        443/TCP             5d3h
service/my-cluster-mysql            ClusterIP   10.1.163.112   <none>        3306/TCP            23m
service/my-cluster-mysql-master     ClusterIP   10.1.32.174    <none>        3306/TCP,8080/TCP   23m
service/my-cluster-mysql-replicas   ClusterIP   10.1.199.160   <none>        3306/TCP,8080/TCP   23m
service/mysql                       ClusterIP   None           <none>        3306/TCP,9125/TCP   23m
service/mysql-operator              ClusterIP   10.1.249.252   <none>        80/TCP              41m
service/mysql-operator-0-svc        ClusterIP   10.1.239.169   <none>        80/TCP,10008/TCP    41m

NAME                                READY   AGE
statefulset.apps/my-cluster-mysql   2/2     23m
statefulset.apps/mysql-operator     1/1     41m


[root@k8s-master1 examples]# kubectl get pvc
NAME                      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
data-my-cluster-mysql-0   Bound    pvc-0119f938-3c89-4f1b-b2df-8b5189c1525c   25Gi       RWO            rook-ceph-block   23m
data-my-cluster-mysql-1   Bound    pvc-87fbd15b-bfb7-443a-9289-177385eb5e3f   25Gi       RWO            rook-ceph-block   19m
data-mysql-operator-0     Bound    pvc-ddbfb37a-6d9b-4a2d-ab40-7fc5c0676f1b   5Gi        RWO            rook-ceph-block   42m
```



### 3.MySQL访问

#### 3.1.修改service类型

```shell
[root@k8s-master1 examples]# kubectl get svc
NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE
kubernetes                  ClusterIP   10.1.0.1       <none>        443/TCP             5d3h
my-cluster-mysql            ClusterIP   10.1.163.112   <none>        3306/TCP            37m
my-cluster-mysql-master     ClusterIP   10.1.32.174    <none>        3306/TCP,8080/TCP   37m
my-cluster-mysql-replicas   ClusterIP   10.1.199.160   <none>        3306/TCP,8080/TCP   37m
mysql                       ClusterIP   None           <none>        3306/TCP,9125/TCP   37m
mysql-operator              ClusterIP   10.1.249.252   <none>        80/TCP              55m
mysql-operator-0-svc        ClusterIP   10.1.239.169   <none>        80/TCP,10008/TCP    55m
```

```shell
# 修改service类型 type: NodePort

[root@k8s-master1 examples]# kubectl edit svc my-cluster-mysql-master
service/my-cluster-mysql-master edited

[root@k8s-master1 examples]# kubectl edit svc my-cluster-mysql-replicas
service/my-cluster-mysql-replicas edited

spec:
......
  type: NodePort


# 查看
[root@k8s-master1 examples]# kubectl get svc
NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                         AGE
kubernetes                  ClusterIP   10.1.0.1       <none>        443/TCP                         5d3h
my-cluster-mysql            ClusterIP   10.1.163.112   <none>        3306/TCP                        39m


my-cluster-mysql-master     NodePort    10.1.32.174    <none>        3306:30001/TCP,8080:32568/TCP   39m
my-cluster-mysql-replicas   NodePort    10.1.199.160   <none>        3306:32393/TCP,8080:30392/TCP   39m


mysql                       ClusterIP   None           <none>        3306/TCP,9125/TCP               39m
mysql-operator              ClusterIP   10.1.249.252   <none>        80/TCP                          58m
mysql-operator-0-svc        ClusterIP   10.1.239.169   <none>        80/TCP,10008/TCP                58m

```



#### 3.2.访问MySQL

```shell
# 访问地址


my-cluster-mysql-master     NodePort    10.1.32.174    <none>        3306:30001/TCP,8080:32568/TCP   39m
my-cluster-mysql-replicas   NodePort    10.1.199.160   <none>        3306:32393/TCP,8080:30392/TCP   39m

# 外部访问地址
my-cluster-mysql-master
172.51.216.81
30001
root/root

my-cluster-mysql-replicas
172.51.216.81
32393
root/root


# k8s内部访问地址
# K8s 中的容器使用访问
svcname.namespace.svc.cluster.local:port
my-cluster-mysql-master.default.svc.cluster.local:3306
```

![](/images/kubernetes/pro/middleware/mysql-1.png)

![](/images/kubernetes/pro/middleware/mysql-2.png)



#### 3.3.测试

主节点：k8s-mysql-master     my-cluster-mysql-master

从节点：k8s-mysql-slaver       my-cluster-mysql-replicas

**主节点可以读写，从节点只能读。**

在主节点创建数据库db1，从节点自动创建db1.

![](/images/kubernetes/pro/middleware/mysql-3.png)



