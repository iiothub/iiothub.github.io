* TOC
{:toc}



## 一、概述



### 1.Operator

 ```shell
 # awesome-operators
 https://github.com/operator-framework/awesome-operators
 
 
 # 安装
 # 搭建主从集群
 # MySQL #3	presslabs/mysql-operator
 https://github.com/bitpoke/mysql-operator
 
 
 # 官方
 # MySQL #2	oracle/mysql-operator
 https://github.com/oracle/mysql-operator
 https://github.com/mysql/mysql-operator
 ```



| App Name | Github                                                       | Description                                                  |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| MySQL #1 | [grtl/mysql-operator](https://github.com/grtl/mysql-operator) | This operator creates a Kubernetes Custom Resource for MySQL. |
| MySQL #2 | [oracle/mysql-operator](https://github.com/oracle/mysql-operator) | This operator creates, operates, and scales self-healing MySQL clusters in Kubernetes |
| MySQL #3 | [presslabs/mysql-operator](https://github.com/presslabs/mysql-operator) | This operator manages all the necessary resources for deploying and managing a highly available MySQL cluster. It provides efortless backups, while keeping the cluster highly-available. |
| MySQL #4 | [banzaicloud/mysql-operator](https://github.com/banzaicloud/mysql-operator) | Create, operate and scale self-healing MySQL clusters in Kubernetes. |
| MySQL #5 | [Percona-Lab/percona-xtradb-cluster-operator](https://github.com/Percona-Lab/percona-xtradb-cluster-operator) | A Kubernetes operator for [Percona XtraDB Cluster](https://www.percona.com/software/mysql-database/percona-xtradb-cluster). Multi-master MySQL cluster with ProxySQL ingress, native backups, scaling, monitoring, reliable automatic self-healing. |



### 2.Helm

**Bitnami**

```shell
https://bitnami.com/
https://github.com/bitnami
https://bitnami.com/stacks
```



```shell
[root@k8s-master production-ready]# helm search repo mysql
NAME                         	CHART VERSION	APP VERSION	DESCRIPTION                                       
aliyun/mysql                 	0.3.5        	           	Fast, reliable, scalable, and easy to use open-...
bitnami/mysql                	8.8.8        	8.0.26     	Chart to create a Highly available MySQL cluster  
aliyun/percona               	0.3.0        	           	free, fully compatible, enhanced, open source d...
aliyun/percona-xtradb-cluster	0.0.2        	5.7.19     	free, fully compatible, enhanced, open source d...
bitnami/phpmyadmin           	8.2.16       	5.1.1      	phpMyAdmin is an mysql administration frontend    
aliyun/gcloud-sqlproxy       	0.2.3        	           	Google Cloud SQL Proxy                            
aliyun/mariadb               	2.1.6        	10.1.31    	Fast, reliable, scalable, and easy to use open-...
bitnami/mariadb              	9.6.2        	10.5.12    	Fast, reliable, scalable, and easy to use open-...
bitnami/mariadb-cluster      	1.0.2        	10.2.14    	DEPRECATED Chart to create a Highly available M...
bitnami/mariadb-galera       	6.0.1        	10.6.4     	MariaDB Galera is a multi-master database clust...
```





## 二、基础



### 1.Operator

```shell
# 安装
# MySQL #3	presslabs/mysql-operator
https://github.com/bitpoke/mysql-operator


https://github.com/bitpoke/mysql-operator/blob/master/docs/_index.md
https://github.com/bitpoke/mysql-operator/blob/master/docs/deploy-mysql-cluster.md


Goals and status
The main goals of this operator are:
Easily deploy MySQL clusters in Kubernetes (cluster-per-service model)
Friendly to devops (monitoring, availability, scalability and backup stories solved)
Out-of-the-box backups (scheduled and on demand) and point-in-time recovery
Support for cloning in cluster and across clusters.
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



#### 1.1.配置

```yaml
# example-cluster-secret.yaml


[root@k8s-master examples]# vim example-cluster-secret.yaml 

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
  

# 修改root密码
[root@k8s-master examples]# echo -n root |base64
cm9vdA==
```

```yaml
# example-cluster.yam


[root@k8s-master examples]# vim example-cluster.yaml 

apiVersion: mysql.presslabs.org/v1alpha1
kind: MysqlCluster
metadata:
  name: my-cluster
spec:
  replicas: 2
  secretName: my-secret

  ## For setting custom docker image or specifying mysql version
  ## the image field has priority over mysqlVersion.
  # image: percona:5.7
  # mysqlVersion: "5.7"

  # initBucketURL: gs://bucket_name/backup.xtrabackup.gz
  # initBucketSecretName:

  ## PodDisruptionBudget
  # minAvailable: 1

  ## For recurrent backups set backupSchedule with a cronjob expression with seconds
  # backupSchedule:
  # backupURL: s3://bucket_name/
  # backupSecretName:
  # backupScheduleJobsHistoryLimit:
  # backupRemoteDeletePolicy:

  ## Custom Server ID Offset for replication
  # serverIDOffset: 100

  ## Configs that will be added to my.cnf for cluster
  mysqlConf:
  #   innodb-buffer-size: 128M


  ## Specify additional pod specification
  # podSpec:
  #   imagePullSecrets: []
  #   labels: {}
  #   annotations: {}
  #   affinity:
  #     podAntiAffinity:
  #       preferredDuringSchedulingIgnoredDuringExecution:
  #         weight: 100
  #         podAffinityTerm:
  #           topologyKey: "kubernetes.io/hostname"
  #           labelSelector:
  #             matchlabels: <cluster-labels>
  #   backupAffinity: {}
  #   backupNodeSelector: {}
  #   backupPriorityClassName:
  #   backupTolerations: []
  #   # Override the default preStop hook with a custom command/script
  #   mysqlLifecycle:
  #     preStop:
  #       exec:
  #         command:
  #           - /scripts/demote-if-master
  #   nodeSelector: {}
  #   resources:
  #     requests:
  #       memory: 1G
  #       cpu:    200m
  #   tolerations: []
  #   priorityClassName:
  #   serviceAccountName: default
  #   # Use a initContainer to fix the permissons of a hostPath volume.
  #   initContainers:
  #     - name: volume-permissions
  #       image: busybox
  #       securityContext:
  #         runAsUser: 0
  #       command:
  #         - sh
  #         - -c
  #         - chmod 750 /data/mysql; chown 999:999 /data/mysql
  #       volumeMounts:
  #         - name: data
  #           mountPath: /data/mysql

  ## Specify additional volume specification
  # volumeSpec:
  #   # https://godoc.org/k8s.io/api/core/v1#EmptyDirVolumeSource
  #   emptyDir: {}

  #   # https://godoc.org/k8s.io/api/core/v1#HostPathVolumeSource
  #   hostPath:
  #     path:
  #     type:

  #   # https://godoc.org/k8s.io/api/core/v1#PersistentVolumeClaimSpec
  #   persistentVolumeClaim:
  #     accessModes: [ "ReadWriteOnce" ]
  #     resources:
  #       requests:
  #         storage: 1Gi

  ## Specify service objectives
  ## If thoses SLO are not fulfilled by cluster node then that node is
  ## removed from scheme
  # targetSLO:
  #   maxSlaveLatency: 10s

  ## You can use custom volume for /tmp partition if needed.
  ## Is disabled by default
  # tmpfsSize: 1Gi

  ## Set cluster in read only
  # readOnly: false

  ## Use `pigz` for parallel compression/decompression of backups
  ## Or specify any arbitrary compress/decompress commands with args
  # backupCompressCommand:
  #   - pigz
  #   - --stdout
  #
  # backupDecompressCommand:
  #   - pigz
  #   - --decompress

  ## Add metrics exporter extra arguments
  # metricsExporterExtraArgs:
  #   - --collect.info_schema.userstats
  #   - --collect.perf_schema.file_events

  ## Add extra arguments to rclone
  # rcloneExtraArgs:
  #   - --buffer-size=1G
  #   - --multi-thread-streams=8
  #   - --retries-sleep=10s
  #   - --retries=10
  #   - --transfers=8
  #   - --s3-force-path-style=false # when use Alibaba OSS

  ## Add extra arguments to xbstream
  # xbstreamExtraArgs:
  #   - --parallel=8

  ## Add extra arguments to xtrabackup
  # xtrabackupExtraArgs:
  #   - --parallel=8

  ## Add extra arguments to xtrabackup during --prepare
  # xtrabackupPrepareExtraArgs:
  #   - --use-memory=5G

  ## Set xtrabackup target directory (the directory needs to exist)
  # xtrabackupTargetDir: /var/lib/mysql/.tmp/xtrabackup/

  # Add additional SQL commands to run during init of mysql
  # initFileExtraSQL:
  #   - "CREATE USER test@localhost"
  
  
# 修改配置
   volumeSpec:
     persistentVolumeClaim:
       accessModes: [ "ReadWriteOnce" ]
       resources:
         requests:
           storage: 20Gi
       storageClassName: rook-ceph-block
```



#### 1.2.部署集群

**注意：直接安装PVC会出问题，需要下载到本地，修改配置。**

```shell
# 安装Operator
[root@k8s-master examples]# helm repo add presslabs https://presslabs.github.io/charts
"presslabs" has been added to your repositories

[root@k8s-master examples]# helm search repo mysql-operator
NAME                    	CHART VERSION	APP VERSION	DESCRIPTION                    
presslabs/mysql-operator	0.4.0        	v0.4.0     	A Helm chart for mysql operator

[root@k8s-master examples]# helm repo update
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


[root@k8s-master mysql]# helm install mysql-operator mysql-operator
```

```shell
# 安装MySQL集群

[root@k8s-master examples]# kubectl apply -f example-cluster-secret.yaml 
secret/my-secret created
[root@k8s-master examples]# kubectl apply -f example-cluster.yaml 
mysqlcluster.mysql.presslabs.org/my-cluster created
```



#### 1.3.删除集群

```shell
[root@k8s-master examples]# pwd
/k8s/middleware/mysql/operator/mysql-operator/examples
[root@k8s-master examples]# ll
total 32
-rw-r--r-- 1 root root  656 Dec  1 17:12 example-backup-secret.yaml
-rw-r--r-- 1 root root  608 Dec  1 17:12 example-backup.yaml
-rw-r--r-- 1 root root  331 Dec  1 17:12 example-cluster-init.yaml
-rw-r--r-- 1 root root  256 Dec  1 20:49 example-cluster-secret.yaml
-rw-r--r-- 1 root root 4131 Dec  1 20:53 example-cluster.yaml
-rw-r--r-- 1 root root  184 Dec  1 17:12 example-database.yaml
-rw-r--r-- 1 root root  533 Dec  1 17:12 example-user.yaml


# 删除mysql集群
[root@k8s-master examples]# kubectl delete -f example-cluster.yaml 
mysqlcluster.mysql.presslabs.org "my-cluster" deleted

[root@k8s-master examples]# kubectl delete -f example-cluster-secret.yaml 
secret "my-secret" deleted


# 删除operator
[root@k8s-master examples]# helm delete mysql-operator
W1202 10:50:18.598111   80752 warnings.go:70] rbac.authorization.k8s.io/v1beta1 ClusterRoleBinding is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 ClusterRoleBinding
release "mysql-operator" uninstalled
```





## 三、实践



### 1.部署说明

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
[root@k8s-master helm]# helm repo add presslabs https://presslabs.github.io/charts
"presslabs" has been added to your repositories

[root@k8s-master helm]# helm search repo mysql-operator
NAME                    	CHART VERSION	APP VERSION	DESCRIPTION                    
presslabs/mysql-operator	0.4.0        	v0.4.0     	A Helm chart for mysql operator

[root@k8s-master helm]# helm repo update
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
[root@k8s-master mysql-operator]# pwd
/k8s/middleware/mysql/helm/mysql-operator
[root@k8s-master mysql-operator]# ll
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
    size: 8Gi
......
```



**3.手动创建PVC（此步不需要）**

data-mysql-operator-pvc.yaml

```yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-mysql-operator-0
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: rook-ceph-block
  resources:
    requests:
      storage: 8Gi
```

```shell
[root@k8s-master helm]# kubectl apply -f data-mysql-operator-pvc.yaml 
persistentvolumeclaim/data-mysql-operator-0 created

[root@k8s-master helm]# kubectl get pvc
NAME                    STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
data-mysql-operator-0   Bound    pvc-640cb015-b792-4d44-b35c-7d7b636aa053   8Gi        RWO            rook-ceph-block   3s
```



**4.创建Operator**

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


[root@k8s-master helm]# helm list
NAME          	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART               	APP VERSION
mysql-operator	default  	1       	2021-12-02 11:18:07.662672576 +0800 CST	deployed	mysql-operator-0.4.0	v0.4.0


[root@k8s-master helm]# kubectl get all
NAME                   READY   STATUS    RESTARTS   AGE
pod/mysql-operator-0   2/2     Running   0          25s

NAME                           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)            AGE
service/kubernetes             ClusterIP   10.96.0.1      <none>        443/TCP            106d
service/mysql-operator         ClusterIP   10.101.35.96   <none>        80/TCP             25s
service/mysql-operator-0-svc   ClusterIP   10.99.58.186   <none>        80/TCP,10008/TCP   25s

NAME                              READY   AGE
statefulset.apps/mysql-operator   1/1     25s
```



#### 2.2.安装MySQL集群

**1.修改配置**

```shell
[root@k8s-master examples]# pwd
/k8s/middleware/mysql/operator/mysql-operator/examples
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


[root@k8s-master examples]# vim example-cluster-secret.yaml 

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


[root@k8s-master examples]# vim example-cluster.yaml 

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
           storage: 20Gi
       storageClassName: rook-ceph-block
```



**2.安装MySQL**

```shell
[root@k8s-master examples]# kubectl apply -f example-cluster-secret.yaml 
secret/my-secret created
[root@k8s-master examples]# kubectl apply -f example-cluster.yaml 
mysqlcluster.mysql.presslabs.org/my-cluster created


# 查看
[root@k8s-master examples]# kubectl get all
NAME                     READY   STATUS    RESTARTS   AGE
pod/my-cluster-mysql-0   4/4     Running   0          2m35s
pod/my-cluster-mysql-1   4/4     Running   0          93s
pod/mysql-operator-0     2/2     Running   0          5m28s

NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
service/my-cluster-mysql            ClusterIP   10.101.55.122   <none>        3306/TCP            2m35s
service/my-cluster-mysql-master     ClusterIP   10.98.74.201    <none>        3306/TCP,8080/TCP   2m35s
service/my-cluster-mysql-replicas   ClusterIP   10.105.202.27   <none>        3306/TCP,8080/TCP   2m35s
service/mysql                       ClusterIP   None            <none>        3306/TCP,9125/TCP   2m35s
service/mysql-operator              ClusterIP   10.101.35.96    <none>        80/TCP              5m28s
service/mysql-operator-0-svc        ClusterIP   10.99.58.186    <none>        80/TCP,10008/TCP    5m28s

NAME                                READY   AGE
statefulset.apps/my-cluster-mysql   2/2     2m35s
statefulset.apps/mysql-operator     1/1     5m28s

[root@k8s-master examples]# kubectl get pvc
NAME                      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
data-my-cluster-mysql-0   Bound    pvc-12f44ebb-d2b9-47bc-91fd-388c7a517884   20Gi       RWO            rook-ceph-block   2m41s
data-my-cluster-mysql-1   Bound    pvc-7af0e187-41c1-4821-aa18-ccfde8f5d975   20Gi       RWO            rook-ceph-block   99s
data-mysql-operator-0     Bound    pvc-640cb015-b792-4d44-b35c-7d7b636aa053   8Gi        RWO            rook-ceph-block   6m25s
```



### 3.MySQL集群说明

```shell
[root@k8s-master examples]# kubectl get all
NAME                     READY   STATUS    RESTARTS   AGE
pod/my-cluster-mysql-0   4/4     Running   0          2m35s
pod/my-cluster-mysql-1   4/4     Running   0          93s
pod/mysql-operator-0     2/2     Running   0          5m28s

NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
service/kubernetes                  ClusterIP   10.96.0.1       <none>        443/TCP             106d
service/my-cluster-mysql            ClusterIP   10.101.55.122   <none>        3306/TCP            2m35s
service/my-cluster-mysql-master     ClusterIP   10.98.74.201    <none>        3306/TCP,8080/TCP   2m35s
service/my-cluster-mysql-replicas   ClusterIP   10.105.202.27   <none>        3306/TCP,8080/TCP   2m35s
service/mysql                       ClusterIP   None            <none>        3306/TCP,9125/TCP   2m35s
service/mysql-operator              ClusterIP   10.101.35.96    <none>        80/TCP              5m28s
service/mysql-operator-0-svc        ClusterIP   10.99.58.186    <none>        80/TCP,10008/TCP    5m28s

NAME                                READY   AGE
statefulset.apps/my-cluster-mysql   2/2     2m35s
statefulset.apps/mysql-operator     1/1     5m28s


# 查看Pod
[root@k8s-master examples]# kubectl get pod -owide
NAME                 READY   STATUS    RESTARTS   AGE     IP               NODE        NOMINATED NODE   READINESS GATES
my-cluster-mysql-0   4/4     Running   0          5m32s   10.244.36.101    k8s-node1   <none>           <none>
my-cluster-mysql-1   4/4     Running   0          4m30s   10.244.169.175   k8s-node2   <none>           <none>
mysql-operator-0     2/2     Running   0          8m25s   10.244.36.105    k8s-node1   <none>           <none>


# 主节点
[root@k8s-master examples]# kubectl describe pod my-cluster-mysql-0
Name:         my-cluster-mysql-0
Namespace:    default
Priority:     0
Node:         k8s-node1/172.51.216.82
Start Time:   Thu, 02 Dec 2021 12:10:23 +0800
Labels:       app.kubernetes.io/component=database
              app.kubernetes.io/instance=my-cluster
              app.kubernetes.io/managed-by=mysql.presslabs.org
              app.kubernetes.io/name=mysql
              app.kubernetes.io/version=5.7.26
              controller-revision-hash=my-cluster-mysql-5ccf796768
              healthy=yes
              mysql.presslabs.org/cluster=my-cluster
              
              role=master
              
              statefulset.kubernetes.io/pod-name=my-cluster-mysql-0
......


# 从节点
[root@k8s-master examples]# kubectl describe pod my-cluster-mysql-1
Name:         my-cluster-mysql-1
Namespace:    default
Priority:     0
Node:         k8s-node2/172.51.216.83
Start Time:   Thu, 02 Dec 2021 12:11:25 +0800
Labels:       app.kubernetes.io/component=database
              app.kubernetes.io/instance=my-cluster
              app.kubernetes.io/managed-by=mysql.presslabs.org
              app.kubernetes.io/name=mysql
              app.kubernetes.io/version=5.7.26
              controller-revision-hash=my-cluster-mysql-5ccf796768
              healthy=yes
              mysql.presslabs.org/cluster=my-cluster
              
              role=replica
              
              statefulset.kubernetes.io/pod-name=my-cluster-mysql-1
......
```

```shell
[root@k8s-master examples]# kubectl get svc
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
my-cluster-mysql            ClusterIP   10.101.55.122   <none>        3306/TCP            41m
my-cluster-mysql-master     ClusterIP   10.98.74.201    <none>        3306/TCP,8080/TCP   41m
my-cluster-mysql-replicas   ClusterIP   10.105.202.27   <none>        3306/TCP,8080/TCP   41m
mysql                       ClusterIP   None            <none>        3306/TCP,9125/TCP   41m
mysql-operator              ClusterIP   10.101.35.96    <none>        80/TCP              44m
mysql-operator-0-svc        ClusterIP   10.99.58.186    <none>        80/TCP,10008/TCP    44m


# 主节点
[root@k8s-master examples]# kubectl describe svc my-cluster-mysql-master
Name:              my-cluster-mysql-master
Namespace:         default
Labels:            app.kubernetes.io/component=database
                   app.kubernetes.io/instance=my-cluster
                   app.kubernetes.io/managed-by=mysql.presslabs.org
                   app.kubernetes.io/name=mysql
                   app.kubernetes.io/version=5.7.26
                   mysql.presslabs.org/cluster=my-cluster
                   mysql.presslabs.org/service-type=master
Annotations:       <none>
Selector:          app.kubernetes.io/managed-by=mysql.presslabs.org,app.kubernetes.io/name=mysql,mysql.presslabs.org/cluster=my-cluster,role=master
Type:              ClusterIP
IP Families:       <none>
IP:                10.98.74.201
IPs:               10.98.74.201
Port:              mysql  3306/TCP
TargetPort:        3306/TCP
Endpoints:         10.244.36.101:3306
Port:              sidecar-http  8080/TCP
TargetPort:        8080/TCP
Endpoints:         10.244.36.101:8080
Session Affinity:  None
Events:            <none>


# 从节点
[root@k8s-master examples]# kubectl describe svc my-cluster-mysql-replicas
Name:              my-cluster-mysql-replicas
Namespace:         default
Labels:            app.kubernetes.io/component=database
                   app.kubernetes.io/instance=my-cluster
                   app.kubernetes.io/managed-by=mysql.presslabs.org
                   app.kubernetes.io/name=mysql
                   app.kubernetes.io/version=5.7.26
                   mysql.presslabs.org/cluster=my-cluster
                   mysql.presslabs.org/service-type=ready-replicas
Annotations:       <none>
Selector:          app.kubernetes.io/managed-by=mysql.presslabs.org,app.kubernetes.io/name=mysql,healthy=yes,mysql.presslabs.org/cluster=my-cluster,role=replica
Type:              ClusterIP
IP Families:       <none>
IP:                10.105.202.27
IPs:               10.105.202.27
Port:              mysql  3306/TCP
TargetPort:        3306/TCP
Endpoints:         10.244.169.175:3306
Port:              sidecar-http  8080/TCP
TargetPort:        8080/TCP
Endpoints:         10.244.169.175:8080
Session Affinity:  None
Events:            <none>


# 主从节点 
[root@k8s-master examples]# kubectl describe svc my-cluster-mysql
Name:              my-cluster-mysql
Namespace:         default
Labels:            app.kubernetes.io/component=database
                   app.kubernetes.io/instance=my-cluster
                   app.kubernetes.io/managed-by=mysql.presslabs.org
                   app.kubernetes.io/name=mysql
                   app.kubernetes.io/version=5.7.26
                   mysql.presslabs.org/cluster=my-cluster
                   mysql.presslabs.org/service-type=ready-nodes
Annotations:       <none>
Selector:          app.kubernetes.io/managed-by=mysql.presslabs.org,app.kubernetes.io/name=mysql,healthy=yes,mysql.presslabs.org/cluster=my-cluster
Type:              ClusterIP
IP Families:       <none>
IP:                10.101.55.122
IPs:               10.101.55.122
Port:              mysql  3306/TCP
TargetPort:        3306/TCP
Endpoints:         10.244.169.175:3306,10.244.36.101:3306
Session Affinity:  None
Events:            <none>
```



### 4.MySQL访问

#### 4.1.修改service类型

```shell
[root@k8s-master examples]# kubectl get svc
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
kubernetes                  ClusterIP   10.96.0.1       <none>        443/TCP             107d
my-cluster-mysql            ClusterIP   10.101.55.122   <none>        3306/TCP            46m
my-cluster-mysql-master     ClusterIP   10.98.74.201    <none>        3306/TCP,8080/TCP   46m
my-cluster-mysql-replicas   ClusterIP   10.105.202.27   <none>        3306/TCP,8080/TCP   46m
mysql                       ClusterIP   None            <none>        3306/TCP,9125/TCP   46m
mysql-operator              ClusterIP   10.101.35.96    <none>        80/TCP              49m
mysql-operator-0-svc        ClusterIP   10.99.58.186    <none>        80/TCP,10008/TCP    49m
```

```shell
# 修改service类型 type: NodePort

[root@k8s-master examples]# kubectl edit svc my-cluster-mysql-master
service/my-cluster-mysql-master edited

[root@k8s-master examples]# kubectl edit svc my-cluster-mysql-replicas
service/my-cluster-mysql-replicas edited

spec:
......
  type: NodePort


# 查看
[root@k8s-master examples]# kubectl get svc
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                         AGE
kubernetes                  ClusterIP   10.96.0.1       <none>        443/TCP                         107d
my-cluster-mysql            ClusterIP   10.101.55.122   <none>        3306/TCP                        49m


my-cluster-mysql-master     NodePort    10.98.74.201    <none>        3306:30120/TCP,8080:31102/TCP   49m
my-cluster-mysql-replicas   NodePort    10.105.202.27   <none>        3306:30903/TCP,8080:31767/TCP   49m


mysql                       ClusterIP   None            <none>        3306/TCP,9125/TCP               49m
mysql-operator              ClusterIP   10.101.35.96    <none>        80/TCP                          52m
mysql-operator-0-svc        ClusterIP   10.99.58.186    <none>        80/TCP,10008/TCP                52m
```



#### 4.2.访问MySQL

```shell
# 访问地址


my-cluster-mysql-master     NodePort    10.98.74.201    <none>        3306:30120/TCP,8080:31102/TCP   49m
my-cluster-mysql-replicas   NodePort    10.105.202.27   <none>        3306:30903/TCP,8080:31767/TCP   49m

my-cluster-mysql-master
172.51.216.81
30120
root/root

my-cluster-mysql-replicas
172.51.216.81
30903
root/root
```

![](/images/kubernetes/middleware/mysql-1.png)

![](/images/kubernetes/middleware/mysql-2.png)



#### 4.3.测试

主节点：k8s-mysql-master     my-cluster-mysql-master

从节点：k8s-mysql-slaver       my-cluster-mysql-replicas

**主节点可以读写，从节点只能读。**

在主节点创建数据库db1，从节点自动创建db1

![](/images/kubernetes/middleware/mysql-3.png)



### 5.部署XXL-JOB

#### 5.1.创建数据库

tables_xxl_job.sql

```sql
#
# XXL-JOB v2.3.0
# Copyright (c) 2015-present, xuxueli.

CREATE database if NOT EXISTS `xxl_job` default character set utf8mb4 collate utf8mb4_unicode_ci;
use `xxl_job`;

SET NAMES utf8mb4;

CREATE TABLE `xxl_job_info` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `job_group` int(11) NOT NULL COMMENT '执行器主键ID',
  `job_desc` varchar(255) NOT NULL,
  `add_time` datetime DEFAULT NULL,
  `update_time` datetime DEFAULT NULL,
  `author` varchar(64) DEFAULT NULL COMMENT '作者',
  `alarm_email` varchar(255) DEFAULT NULL COMMENT '报警邮件',
  `schedule_type` varchar(50) NOT NULL DEFAULT 'NONE' COMMENT '调度类型',
  `schedule_conf` varchar(128) DEFAULT NULL COMMENT '调度配置，值含义取决于调度类型',
  `misfire_strategy` varchar(50) NOT NULL DEFAULT 'DO_NOTHING' COMMENT '调度过期策略',
  `executor_route_strategy` varchar(50) DEFAULT NULL COMMENT '执行器路由策略',
  `executor_handler` varchar(255) DEFAULT NULL COMMENT '执行器任务handler',
  `executor_param` varchar(512) DEFAULT NULL COMMENT '执行器任务参数',
  `executor_block_strategy` varchar(50) DEFAULT NULL COMMENT '阻塞处理策略',
  `executor_timeout` int(11) NOT NULL DEFAULT '0' COMMENT '任务执行超时时间，单位秒',
  `executor_fail_retry_count` int(11) NOT NULL DEFAULT '0' COMMENT '失败重试次数',
  `glue_type` varchar(50) NOT NULL COMMENT 'GLUE类型',
  `glue_source` mediumtext COMMENT 'GLUE源代码',
  `glue_remark` varchar(128) DEFAULT NULL COMMENT 'GLUE备注',
  `glue_updatetime` datetime DEFAULT NULL COMMENT 'GLUE更新时间',
  `child_jobid` varchar(255) DEFAULT NULL COMMENT '子任务ID，多个逗号分隔',
  `trigger_status` tinyint(4) NOT NULL DEFAULT '0' COMMENT '调度状态：0-停止，1-运行',
  `trigger_last_time` bigint(13) NOT NULL DEFAULT '0' COMMENT '上次调度时间',
  `trigger_next_time` bigint(13) NOT NULL DEFAULT '0' COMMENT '下次调度时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `xxl_job_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `job_group` int(11) NOT NULL COMMENT '执行器主键ID',
  `job_id` int(11) NOT NULL COMMENT '任务，主键ID',
  `executor_address` varchar(255) DEFAULT NULL COMMENT '执行器地址，本次执行的地址',
  `executor_handler` varchar(255) DEFAULT NULL COMMENT '执行器任务handler',
  `executor_param` varchar(512) DEFAULT NULL COMMENT '执行器任务参数',
  `executor_sharding_param` varchar(20) DEFAULT NULL COMMENT '执行器任务分片参数，格式如 1/2',
  `executor_fail_retry_count` int(11) NOT NULL DEFAULT '0' COMMENT '失败重试次数',
  `trigger_time` datetime DEFAULT NULL COMMENT '调度-时间',
  `trigger_code` int(11) NOT NULL COMMENT '调度-结果',
  `trigger_msg` text COMMENT '调度-日志',
  `handle_time` datetime DEFAULT NULL COMMENT '执行-时间',
  `handle_code` int(11) NOT NULL COMMENT '执行-状态',
  `handle_msg` text COMMENT '执行-日志',
  `alarm_status` tinyint(4) NOT NULL DEFAULT '0' COMMENT '告警状态：0-默认、1-无需告警、2-告警成功、3-告警失败',
  PRIMARY KEY (`id`),
  KEY `I_trigger_time` (`trigger_time`),
  KEY `I_handle_code` (`handle_code`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `xxl_job_log_report` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `trigger_day` datetime DEFAULT NULL COMMENT '调度-时间',
  `running_count` int(11) NOT NULL DEFAULT '0' COMMENT '运行中-日志数量',
  `suc_count` int(11) NOT NULL DEFAULT '0' COMMENT '执行成功-日志数量',
  `fail_count` int(11) NOT NULL DEFAULT '0' COMMENT '执行失败-日志数量',
  `update_time` datetime DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `i_trigger_day` (`trigger_day`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `xxl_job_logglue` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `job_id` int(11) NOT NULL COMMENT '任务，主键ID',
  `glue_type` varchar(50) DEFAULT NULL COMMENT 'GLUE类型',
  `glue_source` mediumtext COMMENT 'GLUE源代码',
  `glue_remark` varchar(128) NOT NULL COMMENT 'GLUE备注',
  `add_time` datetime DEFAULT NULL,
  `update_time` datetime DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `xxl_job_registry` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `registry_group` varchar(50) NOT NULL,
  `registry_key` varchar(255) NOT NULL,
  `registry_value` varchar(255) NOT NULL,
  `update_time` datetime DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `i_g_k_v` (`registry_group`,`registry_key`,`registry_value`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `xxl_job_group` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `app_name` varchar(64) NOT NULL COMMENT '执行器AppName',
  `title` varchar(12) NOT NULL COMMENT '执行器名称',
  `address_type` tinyint(4) NOT NULL DEFAULT '0' COMMENT '执行器地址类型：0=自动注册、1=手动录入',
  `address_list` text COMMENT '执行器地址列表，多地址逗号分隔',
  `update_time` datetime DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `xxl_job_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(50) NOT NULL COMMENT '账号',
  `password` varchar(50) NOT NULL COMMENT '密码',
  `role` tinyint(4) NOT NULL COMMENT '角色：0-普通用户、1-管理员',
  `permission` varchar(255) DEFAULT NULL COMMENT '权限：执行器ID列表，多个逗号分割',
  PRIMARY KEY (`id`),
  UNIQUE KEY `i_username` (`username`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `xxl_job_lock` (
  `lock_name` varchar(50) NOT NULL COMMENT '锁名称',
  PRIMARY KEY (`lock_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

INSERT INTO `xxl_job_group`(`id`, `app_name`, `title`, `address_type`, `address_list`, `update_time`) VALUES (1, 'xxl-job-executor-sample', '示例执行器', 0, NULL, '2018-11-03 22:21:31' );
INSERT INTO `xxl_job_info`(`id`, `job_group`, `job_desc`, `add_time`, `update_time`, `author`, `alarm_email`, `schedule_type`, `schedule_conf`, `misfire_strategy`, `executor_route_strategy`, `executor_handler`, `executor_param`, `executor_block_strategy`, `executor_timeout`, `executor_fail_retry_count`, `glue_type`, `glue_source`, `glue_remark`, `glue_updatetime`, `child_jobid`) VALUES (1, 1, '测试任务1', '2018-11-03 22:21:31', '2018-11-03 22:21:31', 'XXL', '', 'CRON', '0 0 0 * * ? *', 'DO_NOTHING', 'FIRST', 'demoJobHandler', '', 'SERIAL_EXECUTION', 0, 0, 'BEAN', '', 'GLUE代码初始化', '2018-11-03 22:21:31', '');
INSERT INTO `xxl_job_user`(`id`, `username`, `password`, `role`, `permission`) VALUES (1, 'admin', 'e10adc3949ba59abbe56e057f20f883e', 1, NULL);
INSERT INTO `xxl_job_lock` ( `lock_name`) VALUES ( 'schedule_lock');

commit;
```

![](/images/kubernetes/middleware/mysql-4.png)



#### 5.2.K8S部署XXL-JOB服务

xxl-job.yaml

```shell
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: xxl-job
  labels:
    app: xxl-job
spec:
  type: NodePort
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30880 #对外暴露30880端口
  selector:
    app: xxl-job

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: xxl-job
spec:
  replicas: 1
  selector:
    matchLabels:
      app: xxl-job
  template:
    metadata:
      labels:
        app: xxl-job
    spec:
      containers:
        - name: xxl-job
          image: xuxueli/xxl-job-admin:2.3.0
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080
          env:
            - name: PARAMS
              value:  "--spring.datasource.username=root --spring.datasource.password=root --spring.datasource.url=jdbc:mysql://my-cluster-mysql-master.default.svc.cluster.local:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai"



# 参考容器配置
docker run --network host -d --restart=always --name xxl-job-admin \
-e PARAMS="--spring.datasource.username=root \
--spring.datasource.password=root \
--spring.datasource.url=jdbc:mysql://172.51.216.98:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai" \
-v /tmp:/data/applogs \
xuxueli/xxl-job-admin:2.3.0


# 服务地址
# 链接主节点
svcname.namespace.svc.cluster.local:port
my-cluster-mysql-master.default.svc.cluster.local:3306
```

```shell
# 创建
[root@k8s-master mysql]# kubectl apply -f xxl-job.yaml
service/xxl-job created
deployment.apps/xxl-job created


[root@k8s-master mysql]# kubectl get all -n dev | grep xxl-job
pod/xxl-job-77bc8f696d-s4kjc               1/1     Running            0          22s

service/xxl-job                  NodePort    10.103.211.124   <none>        8080:30880/TCP    22s

deployment.apps/xxl-job               1/1     1            1           22s

replicaset.apps/xxl-job-77bc8f696d               1         1         1       22s



# 访问dashboard
# 访问地址：http://172.51.216.81:30880/xxl-job-admin
# 账户密码：admin/123456
```



### 6.总结

```shell
# 安装
# MySQL #3	presslabs/mysql-operator
https://github.com/bitpoke/mysql-operator
```

```shell
[root@k8s-master mysql]# kubectl get svc
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                         AGE
my-cluster-mysql            ClusterIP   10.101.55.122   <none>        3306/TCP                        126m

my-cluster-mysql-master     NodePort    10.98.74.201    <none>        3306:30120/TCP,8080:31102/TCP   126m
my-cluster-mysql-replicas   NodePort    10.105.202.27   <none>        3306:30903/TCP,8080:31767/TCP   126m

mysql                       ClusterIP   None            <none>        3306/TCP,9125/TCP               126m
mysql-operator              ClusterIP   10.101.35.96    <none>        80/TCP                          129m
mysql-operator-0-svc        ClusterIP   10.99.58.186    <none>        80/TCP,10008/TCP                129m


# 主
my-cluster-mysql-master     NodePort    10.98.74.201    <none>        3306:30120/TCP,8080:31102/TCP   49m
my-cluster-mysql-replicas   NodePort    10.105.202.27   <none>        3306:30903/TCP,8080:31767/TCP   49m

my-cluster-mysql-master
172.51.216.81
30120
root/root

# 从
my-cluster-mysql-replicas
172.51.216.81
30903
root/root
```

```shell
# 使用方式

# 搭建的是主从集群
# 应用对主（my-cluster-mysql-master）进行读写
```



### 7.删除集群

```shell
# 查看

[root@k8s-master mysql]# kubectl get all
NAME                     READY   STATUS    RESTARTS   AGE
pod/my-cluster-mysql-0   4/4     Running   0          129m
pod/my-cluster-mysql-1   4/4     Running   0          128m
pod/mysql-operator-0     2/2     Running   0          132m

NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                         AGE
service/my-cluster-mysql            ClusterIP   10.101.55.122   <none>        3306/TCP                        129m
service/my-cluster-mysql-master     NodePort    10.98.74.201    <none>        3306:30120/TCP,8080:31102/TCP   129m
service/my-cluster-mysql-replicas   NodePort    10.105.202.27   <none>        3306:30903/TCP,8080:31767/TCP   129m
service/mysql                       ClusterIP   None            <none>        3306/TCP,9125/TCP               129m
service/mysql-operator              ClusterIP   10.101.35.96    <none>        80/TCP                          132m
service/mysql-operator-0-svc        ClusterIP   10.99.58.186    <none>        80/TCP,10008/TCP                132m

NAME                                READY   AGE
statefulset.apps/my-cluster-mysql   2/2     129m
statefulset.apps/mysql-operator     1/1     132m

[root@k8s-master mysql]# kubectl get pvc
NAME                      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
data-my-cluster-mysql-0   Bound    pvc-12f44ebb-d2b9-47bc-91fd-388c7a517884   20Gi       RWO            rook-ceph-block   129m
data-my-cluster-mysql-1   Bound    pvc-7af0e187-41c1-4821-aa18-ccfde8f5d975   20Gi       RWO            rook-ceph-block   128m
data-mysql-operator-0     Bound    pvc-640cb015-b792-4d44-b35c-7d7b636aa053   8Gi        RWO            rook-ceph-block   133m
```

```shell
# 删除MySQL集群

[root@k8s-master examples]# pwd
/k8s/middleware/mysql/operator/mysql-operator/examples
[root@k8s-master examples]# ll
total 32
-rw-r--r-- 1 root root  656 Dec  1 17:12 example-backup-secret.yaml
-rw-r--r-- 1 root root  608 Dec  1 17:12 example-backup.yaml
-rw-r--r-- 1 root root  331 Dec  1 17:12 example-cluster-init.yaml
-rw-r--r-- 1 root root  256 Dec  1 20:49 example-cluster-secret.yaml
-rw-r--r-- 1 root root 4132 Dec  2 11:42 example-cluster.yaml
-rw-r--r-- 1 root root  184 Dec  1 17:12 example-database.yaml
-rw-r--r-- 1 root root  533 Dec  1 17:12 example-user.yaml


# 删除
[root@k8s-master examples]# kubectl delete  -f example-cluster.yaml 
mysqlcluster.mysql.presslabs.org "my-cluster" deleted

[root@k8s-master examples]# kubectl delete -f example-cluster-secret.yaml 
secret "my-secret" deleted


# 查看
[root@k8s-master examples]# kubectl get all
NAME                   READY   STATUS    RESTARTS   AGE
pod/mysql-operator-0   2/2     Running   0          137m

NAME                           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE
service/mysql                  ClusterIP   None           <none>        3306/TCP,9125/TCP   134m
service/mysql-operator         ClusterIP   10.101.35.96   <none>        80/TCP              137m
service/mysql-operator-0-svc   ClusterIP   10.99.58.186   <none>        80/TCP,10008/TCP    137m

NAME                              READY   AGE
statefulset.apps/mysql-operator   1/1     137m

[root@k8s-master examples]# kubectl get pvc
NAME                    STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
data-mysql-operator-0   Bound    pvc-640cb015-b792-4d44-b35c-7d7b636aa053   8Gi        RWO            rook-ceph-block   138m
```

```shell
# 删除Operator


[root@k8s-master examples]# helm list
NAME          	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART               	APP VERSION
mysql-operator	default  	1       	2021-12-02 12:07:27.335025829 +0800 CST	deployed	mysql-operator-0.4.0	v0.4.0     
[root@k8s-master examples]# kubectl get pvc
NAME                    STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
data-mysql-operator-0   Bound    pvc-640cb015-b792-4d44-b35c-7d7b636aa053   8Gi        RWO            rook-ceph-block   139m


# 删除
[root@k8s-master examples]# helm delete mysql-operator
W1202 14:27:07.198677    3992 warnings.go:70] rbac.authorization.k8s.io/v1beta1 ClusterRoleBinding is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 ClusterRoleBinding
release "mysql-operator" uninstalled

[root@k8s-master examples]# helm list
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION


[root@k8s-master examples]# kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)             AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP             107d
service/mysql        ClusterIP   None         <none>        3306/TCP,9125/TCP   137m


[root@k8s-master examples]# kubectl delete svc mysql
service "mysql" deleted



[root@k8s-master mysql]# kubectl get pvc
NAME                    STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
data-mysql-operator-0   Bound    pvc-640cb015-b792-4d44-b35c-7d7b636aa053   8Gi        RWO            rook-ceph-block   3h26m

[root@k8s-master mysql]# kubectl delete pvc data-mysql-operator-0
persistentvolumeclaim "data-mysql-operator-0" deleted
```



