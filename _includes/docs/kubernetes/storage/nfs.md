* TOC
{:toc}



## 一、搭建NFS服务



### 1.NFS服务端

```shell
# 192.168.202.193


# 1.安装服务
[root@nfs ~]# yum install -y nfs-utils


# 查看系统是否已安装NFS
# nfs-utils:提供了NFS服务器程序和对应的管理工具
# rpcbind:获取nfs服务器端的端口等信息

[root@192 ~]# rpm -qa nfs-utils
nfs-utils-1.3.0-0.68.el7.2.x86_64
[root@192 ~]# rpm -qa rpcbind
rpcbind-0.2.0-49.el7.x86_64


# 2.创建共享目录/nfs/data
[root@192 ~]# mkdir /nfs/data -p
# 设置权限 chmod 666 /nfs/data
[root@192 ~]# chmod 666 /nfs/data


# 3.修改配置文件
[root@nfs etc]# vim /etc/exports
/nfs/data 192.168.202.0/24(rw,sync,no_root_squash)

# /nfs/data	#表示要共享文件的目录
# 192.168.202.0/2	#表示所有允许访问的客户端IP网段，也可以写成指定的ip，只允许当前客户机访问
# (rw,sync)		#rw:表示读写权限，sync:表示数据同步写入内存硬盘


# 4.启动服务
[root@192 ~]# systemctl start rpcbind
[root@192 ~]# systemctl start nfs

# 设置开机启动
[root@nfs etc]# systemctl enable rpcbind
[root@nfs etc]# systemctl enable nfs

# 查看服务状态
[root@nfs etc]# systemctl status rpcbind
[root@nfs etc]# systemctl status nfs


# 5.查看
[root@192 ~]# exportfs
/nfs/data     	192.168.202.0/24

# 检查 NFS 服务器端是否有目录共享：showmount -e nfs服务器的IP
[root@192 ~]# showmount -e 192.168.202.193
Export list for 192.168.202.193:
/nfs/data 192.168.202.0/24

[root@192 ~]# showmount -e
Export list for 192.168.202.193:
/nfs/data 192.168.202.0/24


# 6.关闭防火墙
systemctl stop firewalld
systemctl disable firewalld
systemctl status firewalld
```



```shell
# 查看磁盘空间

[root@192 ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 1.9G     0  1.9G   0% /dev
tmpfs                    1.9G     0  1.9G   0% /dev/shm
tmpfs                    1.9G   12M  1.9G   1% /run
tmpfs                    1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/mapper/centos-root   26G  1.6G   25G   6% /
/dev/sda1               1014M  150M  865M  15% /boot
tmpfs                    378M     0  378M   0% /run/user/0
```





### 2.NFS客户端

```shell
#服务器：192.168.202.201；192.168.202.202；192.168.202.203
#NFS服务器：192.168.202.193


# 1.安装
[root@k8s-node1 ~]# yum install nfs-utils -y


# 2.启动服务
# 注意：客户端不需要启动nfs服务
[root@k8s-node1 ~]# systemctl start rpcbind
# 设置开机启动
[root@k8s-node1 ~]# systemctl enable rpcbind
# 查看服务状态
[root@k8s-node1 ~]# systemctl status rpcbind


# 3.检查 NFS 服务器端是否有目录共享：showmount -e nfs服务器的IP
[root@k8s-master ~]# showmount -e 192.168.202.193
Export list for 192.168.202.193:
/nfs/data 192.168.202.0/24


# 4.手动挂载NFS共享目录
# mount -t nfs IP地址:共享目录名 挂载到的位置
[root@k8s-node1 ~]# mkdir -p /nfs/data
[root@k8s-master ~]# mount -t nfs 192.168.202.193:/nfs/data /nfs/data


# 5.查看挂载情况
[root@k8s-master ~]# df -h
Filesystem                 Size  Used Avail Use% Mounted on
devtmpfs                   1.9G     0  1.9G   0% /dev
tmpfs                      2.0G     0  2.0G   0% /dev/shm
tmpfs                      2.0G   12M  1.9G   1% /run
tmpfs                      2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/mapper/centos-root     26G  4.4G   22G  17% /
/dev/sda1                 1014M  186M  829M  19% /boot

......
192.168.202.193:/nfs/data   26G  1.6G   25G   6% /nfs/data
```



```shell
# 测试

# 客户端创建文件
[root@k8s-master ~]# cd /nfs/data/
[root@k8s-master data]# vim test.txt
[root@k8s-master data]# ll
total 4
-rw-r--r-- 1 root root 18 Aug  2  2023 test.txt


# nfs服务端查看
[root@192 ~]# cd /nfs/data/
[root@192 data]# ll
total 4
-rw-r--r--. 1 root root 18 Aug  2 01:13 test.txt
[root@192 data]# cat test.txt 
nfs client write!
```





## 二、创建存储



### 1.配置NFS存储  

```shell
# 192.168.202.193


# 1.创建目录
mkdir -p /nfs/data/thingsboard-postgresql
chmod -R 777 thingsboard-*


[root@192 data]# pwd
/nfs/data
[root@192 data]# ll
total 0
[root@192 data]# mkdir thingsboard-postgresql
[root@192 data]# ll
total 0
drwxr-xr-x. 2 root root 6 Aug  2 01:26 thingsboard-postgresql
[root@192 data]# chmod -R 777 thingsboard-*
[root@192 data]# ll
total 0
drwxrwxrwx. 2 root root 6 Aug  2 01:26 thingsboard-postgresql


# 2.修改配置文件
[root@192 data]# vim /etc/exports
/nfs/data *(rw,no_root_squash,no_all_squash,sync)
/nfs/data/thingsboard-postgresql *(rw,no_root_squash,no_all_squash,sync)


# 3.重启服务
[root@192 ~]# systemctl restart nfs-server
[root@192 ~]# systemctl restart nfs
```





### 2.创建pv、pvc

postgresql-pv.yaml

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  namespace: thingsboard  
  name: data-thingsboard-pg-postgresql-0-pv  
  labels:
    type: data-thingsboard-pg-postgresql-0-pv
spec:  
  capacity:
    storage: 8Gi  
  accessModes:
    - ReadWriteMany  
  persistentVolumeReclaimPolicy: Recycle  
  storageClassName: nfs              
  nfs:
    path: "/nfs/data/thingsboard-postgresql"     
    server: 192.168.202.193 #k8s-nfs     
    readOnly: false
```



postgresql-pvc.yaml

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  namespace: thingsboard
  name: data-thingsboard-pg-postgresql-0
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 8Gi
  storageClassName: nfs
```



```shell
[root@k8s-master nfs]# pwd
/thingsboard/nfs
[root@k8s-master nfs]# ll
total 8
-rw-r--r-- 1 root root 230 Aug  2 01:01 postgresql-pvc.yaml
-rw-r--r-- 1 root root 453 Aug  2 01:00 postgresql-pv.yaml


# 创建PV
[root@k8s-master nfs]# kubectl apply -f postgresql-pv.yaml 
persistentvolume/data-thingsboard-pg-postgresql-0-pv created

[root@k8s-master nfs]# kubectl get pv -n thingsboard
NAME                                  CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
data-thingsboard-pg-postgresql-0-pv   8Gi        RWX            Recycle          Available           nfs                     26s


# 创建PVC
[root@k8s-master nfs]# kubectl apply -f postgresql-pvc.yaml 
persistentvolumeclaim/data-thingsboard-pg-postgresql-0 created

[root@k8s-master nfs]# kubectl get pvc -n thingsboard
NAME                               STATUS    VOLUME                                CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-thingsboard-pg-postgresql-0   Bound     data-thingsboard-pg-postgresql-0-pv   8Gi        RWX            nfs            50s


# 查看pv、pvc
[root@k8s-master nfs]# kubectl get pv -n thingsboard
NAME                                  CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                          STORAGECLASS   REASON   AGE
data-thingsboard-pg-postgresql-0-pv   8Gi        RWX            Recycle          Bound    thingsboard/data-thingsboard-pg-postgresql-0   nfs                     12m
[root@k8s-master nfs]# 
[root@k8s-master nfs]# kubectl get pvc -n thingsboard
NAME                               STATUS    VOLUME                                CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-thingsboard-pg-postgresql-0   Bound     data-thingsboard-pg-postgresql-0-pv   8Gi        RWX            nfs            3m3s
```





## 三、StorageClass



### 1.配置NFS存储  

```shell
# 192.168.202.193

# 使用这个
mkdir -p /nfs/data/thingsboard-sc


# 1.创建目录
mkdir -p /nfs/data/thingsboard-postgresql
chmod -R 777 thingsboard-*


# 2.修改配置文件
[root@192 data]# vim /etc/exports
/nfs/data *(rw,no_root_squash,no_all_squash,sync)
/nfs/data/thingsboard-postgresql *(rw,no_root_squash,no_all_squash,sync)


# 3.重启服务
[root@192 ~]# systemctl restart nfs-server
[root@192 ~]# systemctl restart nfs


# showmount -e 192.168.202.193
[root@k8s-master ~]# showmount -e 192.168.202.193
Export list for 192.168.202.193:
/nfs/data/thingsboard-postgresql *
/nfs/data
```



### 2.配置account及相关权限

rbac.yaml    #唯一需要修改的地方只有namespace,根据实际情况定义

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
  namespace: default
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: default
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  namespace: default
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: default
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io
```



### 3.创建NFS资源的StorageClass

nfs-sc.yaml

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-nfs-storage
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"  # 是否设置为默认的storageclass
provisioner: k8s/nfs-subdir-external-provisioner # or choose another name, must match deployment's env PROVISIONER_NAME'
allowVolumeExpansion: true
parameters:
  archiveOnDelete: "false" # 设置为"false"时删除PVC不会保留数据,"true"则保留数据

```



### 4.创建NFS provisioner

nfs-provisioner.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
  labels:
    app: nfs-client-provisioner
  namespace: default
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: registry.cn-beijing.aliyuncs.com/mydlq/nfs-subdir-external-provisioner:v4.0.0
          imagePullPolicy: IfNotPresent
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: k8s/nfs-subdir-external-provisioner
            - name: NFS_SERVER
              value: 192.168.202.193
            - name: NFS_PATH
              value: /nfs/data/thingsboard-sc
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.202.193  # NFS SERVER_IP
            path: /nfs/data/thingsboard-sc
```



```shell
# 查看创建情况

[root@k8s-master storageclass]# kubectl get storageclass
NAME                  PROVISIONER                           RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
managed-nfs-storage   k8s/nfs-subdir-external-provisioner   Delete          Immediate           true                   31m


# 查看pod
[root@k8s-master storageclass]# kubectl get pod --all-namespaces
NAMESPACE       NAME                                       READY   STATUS              RESTARTS       AGE
default         nfs-client-provisioner-58f9746c8f-p45nz    1/1     Running             0              12m
```



### 5.测试

创建测试pod,检查是否部署成功

创建PVC

```yaml
[root@k8s-master-1 storageClass]# cat pvc.yaml 

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-claim
spec:
  storageClassName: managed-nfs-storage
  accessModes: ["ReadWriteMany","ReadOnlyMany"]
  resources:
    requests:
      storage: 100Mi
```





## 四、使用NFS存储



### 1.手动创建PV、PVC方式

postgresql-pv.yaml

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  namespace: thingsboard  
  name: data-thingsboard-pg-postgresql-0-pv  
  labels:
    type: data-thingsboard-pg-postgresql-0-pv
spec:  
  capacity:
    storage: 8Gi  
  accessModes:
    - ReadWriteMany  
  persistentVolumeReclaimPolicy: Recycle  
  storageClassName: nfs              
  nfs:
    path: "/nfs/data/thingsboard-postgresql"     
    server: 192.168.202.193 #k8s-nfs     
    readOnly: false
```



postgresql-pvc.yaml

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  namespace: thingsboard
  name: data-thingsboard-pg-postgresql-0
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 8Gi
  storageClassName: nfs
```



```shell
[root@k8s-master nfs]# pwd
/thingsboard/nfs
[root@k8s-master nfs]# ll
total 8
-rw-r--r-- 1 root root 230 Aug  2 01:01 postgresql-pvc.yaml
-rw-r--r-- 1 root root 453 Aug  2 01:00 postgresql-pv.yaml


# 创建PV
[root@k8s-master nfs]# kubectl apply -f postgresql-pv.yaml 
persistentvolume/data-thingsboard-pg-postgresql-0-pv created

[root@k8s-master nfs]# kubectl get pv -n thingsboard
NAME                                  CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
data-thingsboard-pg-postgresql-0-pv   8Gi        RWX            Recycle          Available           nfs                     26s


# 创建PVC
[root@k8s-master nfs]# kubectl apply -f postgresql-pvc.yaml 
persistentvolumeclaim/data-thingsboard-pg-postgresql-0 created

[root@k8s-master nfs]# kubectl get pvc -n thingsboard
NAME                               STATUS    VOLUME                                CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-thingsboard-pg-postgresql-0   Bound     data-thingsboard-pg-postgresql-0-pv   8Gi        RWX            nfs            50s



# 查看pv、pvc
[root@k8s-master nfs]# kubectl get pv -n thingsboard
NAME                                  CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                          STORAGECLASS   REASON   AGE
data-thingsboard-pg-postgresql-0-pv   8Gi        RWX            Recycle          Bound    thingsboard/data-thingsboard-pg-postgresql-0   nfs                     12m


[root@k8s-master nfs]# kubectl get pvc -n thingsboard
NAME                               STATUS    VOLUME                                CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-thingsboard-pg-postgresql-0   Bound     data-thingsboard-pg-postgresql-0-pv   8Gi        RWX            nfs            3m3s
```



### 2.StorageClass自动分配

#### 2.1.Deployment创建PVC

**Deployment要手动创建PVC，storageClassName: managed-nfs-storage**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pv-claim
  namespace: thingsboard
  labels:
    app: postgres
spec:

  # 分配存储
  storageClassName: managed-nfs-storage
  
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```



#### 2.2.StatefulSet自动分配

**StatefulSet自动创建PV、PVC，storageClassName: managed-nfs-storage**

```yaml
  volumeClaimTemplates:
  - metadata:
      name: cassandra-data
    spec:
    
      # 自动分配存储
      storageClassName: managed-nfs-storage
    
    
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```




