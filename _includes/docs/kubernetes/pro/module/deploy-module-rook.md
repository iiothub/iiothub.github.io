* TOC
{:toc}


### 1.Rook

#### 1.1.磁盘规划

为4台服务器（k8s-node1、k8s-node2、k8s-node3、k8s-node4）分别增加200G裸盘

查看磁盘

```shell
[root@k8s-node1 ~]# lsblk -f
NAME            FSTYPE      LABEL           UUID                                   MOUNTPOINT
sr0             iso9660     CentOS 7 x86_64 2020-11-04-11-36-43-00                 
vda                                                                                
├─vda1          xfs                         80853249-f7a8-43f7-91dc-820d70c300cc   /boot
└─vda2          LVM2_member                 Jth3iR-YpQG-cwff-2pBY-goGN-yl26-b9AZ2n 
  ├─centos-root xfs                         a1c8d6f7-61ba-4af5-bde9-175afd2093d0   /
  ├─centos-swap swap                        ff17871a-3e49-4d75-8c33-f252532cde1b   
  └─centos-home xfs                         5262049d-e810-4073-b542-55639c43158e   /home
vdb                                                                                


[root@k8s-node1 ~]# lsblk
NAME            MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sr0              11:0    1   4.4G  0 rom  
vda             252:0    0   300G  0 disk 
├─vda1          252:1    0     1G  0 part /boot
└─vda2          252:2    0   299G  0 part 
  ├─centos-root 253:0    0    50G  0 lvm  /
  ├─centos-swap 253:1    0   7.9G  0 lvm  
  └─centos-home 253:2    0 241.1G  0 lvm  /home
vdb             252:16   0   200G  0 disk 


# vdb是200G裸盘
```



#### 1.2.安装步骤

```shell
git clone --single-branch --branch release-1.7 https://github.com/rook/rook.git

cd rook/cluster/examples/kubernetes/ceph

kubectl create -f crds.yaml -f common.yaml -f operator.yaml

kubectl create -f cluster.yaml
```



#### 1.2.下载源码

```shell
# git clone --single-branch --branch v1.6.5 https://github.com/rook/rook.git


[root@k8s-master rook]# git clone --single-branch --branch v1.6.5 https://github.com/rook/rook.git
Cloning into 'rook'...
remote: Enumerating objects: 64346, done.
remote: Counting objects: 100% (17/17), done.
remote: Compressing objects: 100% (17/17), done.
remote: Total 64346 (delta 0), reused 17 (delta 0), pack-reused 64329
Receiving objects: 100% (64346/64346), 37.89 MiB | 1.76 MiB/s, done.
Resolving deltas: 100% (45134/45134), done.
Note: checking out '2ee204712b854921ea6dc018ffb151e58bc714e7'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b new_branch_name



# cd rook/cluster/examples/kubernetes/ceph
[root@k8s-master rook]# cd rook/cluster/examples/kubernetes/ceph
[root@k8s-master ceph]# ll
total 896
-rw-r--r-- 1 root root    436 Nov 16 15:47 ceph-client.yaml
-rw-r--r-- 1 root root   1059 Nov 16 15:47 cluster-external-management.yaml
-rw-r--r-- 1 root root   1242 Nov 16 15:47 cluster-external.yaml
-rw-r--r-- 1 root root   8151 Nov 16 15:47 cluster-on-pvc.yaml
-rw-r--r-- 1 root root   3230 Nov 16 15:47 cluster-stretched.yaml
-rw-r--r-- 1 root root   1411 Nov 16 15:47 cluster-test.yaml
-rw-r--r-- 1 root root  14105 Nov 16 15:47 cluster.yaml
-rw-r--r-- 1 root root   2341 Nov 16 15:47 common-external.yaml
-rw-r--r-- 1 root root   2723 Nov 16 15:47 common-second-cluster.yaml
-rw-r--r-- 1 root root  30170 Nov 16 15:47 common.yaml
-rw-r--r-- 1 root root 561795 Nov 16 15:47 crds.yaml
-rw-r--r-- 1 root root  55487 Nov 16 15:47 create-external-cluster-resources.py
-rw-r--r-- 1 root root   3615 Nov 16 15:47 create-external-cluster-resources.sh
drwxr-xr-x 5 root root     47 Nov 16 15:47 csi
-rw-r--r-- 1 root root    411 Nov 16 15:47 dashboard-external-https.yaml
-rw-r--r-- 1 root root    410 Nov 16 15:47 dashboard-external-http.yaml
-rw-r--r-- 1 root root    954 Nov 16 15:47 dashboard-ingress-https.yaml
-rw-r--r-- 1 root root    413 Nov 16 15:47 dashboard-loadbalancer.yaml
-rw-r--r-- 1 root root   1789 Nov 16 15:47 direct-mount.yaml
-rw-r--r-- 1 root root   3432 Nov 16 15:47 filesystem-ec.yaml
-rw-r--r-- 1 root root   1215 Nov 16 15:47 filesystem-mirror.yaml
-rw-r--r-- 1 root root    777 Nov 16 15:47 filesystem-test.yaml
-rw-r--r-- 1 root root   4463 Nov 16 15:47 filesystem.yaml
drwxr-xr-x 2 root root    115 Nov 16 15:47 flex
-rw-r--r-- 1 root root   4977 Nov 16 15:47 import-external-cluster.sh
drwxr-xr-x 2 root root   4096 Nov 16 15:47 monitoring
-rw-r--r-- 1 root root    745 Nov 16 15:47 nfs-test.yaml
-rw-r--r-- 1 root root   1832 Nov 16 15:47 nfs.yaml
-rw-r--r-- 1 root root    587 Nov 16 15:47 object-bucket-claim-delete.yaml
-rw-r--r-- 1 root root    587 Nov 16 15:47 object-bucket-claim-retain.yaml
-rw-r--r-- 1 root root   3669 Nov 16 15:47 object-ec.yaml
-rw-r--r-- 1 root root    814 Nov 16 15:47 object-external.yaml
-rw-r--r-- 1 root root   1804 Nov 16 15:47 object-multisite-pull-realm.yaml
-rw-r--r-- 1 root root   1282 Nov 16 15:47 object-multisite.yaml
-rw-r--r-- 1 root root   5917 Nov 16 15:47 object-openshift.yaml
-rw-r--r-- 1 root root    685 Nov 16 15:47 object-test.yaml
-rw-r--r-- 1 root root    508 Nov 16 15:47 object-user.yaml
-rw-r--r-- 1 root root   5521 Nov 16 15:47 object.yaml
-rw-r--r-- 1 root root  23928 Nov 16 15:47 operator-openshift.yaml
-rw-r--r-- 1 root root  22587 Nov 16 15:47 operator.yaml
-rw-r--r-- 1 root root   2652 Nov 16 15:47 osd-purge.yaml
-rw-r--r-- 1 root root   1100 Nov 16 15:47 pool-ec.yaml
-rw-r--r-- 1 root root    504 Nov 16 15:47 pool-test.yaml
-rw-r--r-- 1 root root   3313 Nov 16 15:47 pool.yaml
drwxr-xr-x 2 root root     23 Nov 16 15:47 pre-k8s-1.16
-rw-r--r-- 1 root root   1456 Nov 16 15:47 rbdmirror.yaml
-rw-r--r-- 1 root root    525 Nov 16 15:47 rgw-external.yaml
-rw-r--r-- 1 root root   2558 Nov 16 15:47 scc.yaml
-rw-r--r-- 1 root root    737 Nov 16 15:47 storageclass-bucket-delete.yaml
-rw-r--r-- 1 root root    736 Nov 16 15:47 storageclass-bucket-retain.yaml
drwxr-xr-x 2 root root     29 Nov 16 15:47 test-data
-rw-r--r-- 1 root root   1738 Nov 16 15:47 toolbox-job.yaml
-rw-r--r-- 1 root root   1474 Nov 16 15:47 toolbox.yaml
```



#### 1.3.部署Rook Operator

**1.修改配置**

由于国内环境无法Pull官方镜像，所以要修改默认镜像地址，改为阿里云镜像仓库(registry.aliyuncs.com/it00021hot是我的镜像地址，利用github action每天定时同步官方镜像，所有版本的镜像都有)

**operator.yaml**

```shell
[root@k8s-master ceph]# vim operator.yaml


#  原来默认配置

 75   # ROOK_CSI_CEPH_IMAGE: "quay.io/cephcsi/cephcsi:v3.3.1"
 76   # ROOK_CSI_REGISTRAR_IMAGE: "k8s.gcr.io/sig-storage/csi-node-driver-registrar:v2.0.1"
 77   # ROOK_CSI_RESIZER_IMAGE: "k8s.gcr.io/sig-storage/csi-resizer:v1.0.1"
 78   # ROOK_CSI_PROVISIONER_IMAGE: "k8s.gcr.io/sig-storage/csi-provisioner:v2.0.4"
 79   # ROOK_CSI_SNAPSHOTTER_IMAGE: "k8s.gcr.io/sig-storage/csi-snapshotter:v4.0.0"
 80   # ROOK_CSI_ATTACHER_IMAGE: "k8s.gcr.io/sig-storage/csi-attacher:v3.0.2"


# 修改后

  ROOK_CSI_CEPH_IMAGE: "registry.aliyuncs.com/it00021hot/cephcsi:v3.3.1"
  ROOK_CSI_REGISTRAR_IMAGE: "registry.aliyuncs.com/it00021hot/csi-node-driver-registrar:v2.0.1"
  ROOK_CSI_RESIZER_IMAGE: "registry.aliyuncs.com/it00021hot/csi-resizer:v1.0.1"
  ROOK_CSI_PROVISIONER_IMAGE: "registry.aliyuncs.com/it00021hot/csi-provisioner:v2.0.4"
  ROOK_CSI_SNAPSHOTTER_IMAGE: "registry.aliyuncs.com/it00021hot/csi-snapshotter:v4.0.0"
  ROOK_CSI_ATTACHER_IMAGE: "registry.aliyuncs.com/it00021hot/csi-attacher:v3.0.2"



# 开启自动发现磁盘（用于后期扩展）
  ROOK_ENABLE_DISCOVERY_DAEMON: "true"
```



**2.创建Operator**

```shell
# kubectl create -f crds.yaml -f common.yaml -f operator.yaml

[root@k8s-master ceph]# kubectl create -f crds.yaml -f common.yaml -f operator.yaml
customresourcedefinition.apiextensions.k8s.io/cephblockpools.ceph.rook.io created
customresourcedefinition.apiextensions.k8s.io/cephclients.ceph.rook.io created
customresourcedefinition.apiextensions.k8s.io/cephclusters.ceph.rook.io created
customresourcedefinition.apiextensions.k8s.io/cephfilesystemmirrors.ceph.rook.io created
customresourcedefinition.apiextensions.k8s.io/cephfilesystems.ceph.rook.io created
customresourcedefinition.apiextensions.k8s.io/cephnfses.ceph.rook.io created
customresourcedefinition.apiextensions.k8s.io/cephobjectrealms.ceph.rook.io created
customresourcedefinition.apiextensions.k8s.io/cephobjectstores.ceph.rook.io created
customresourcedefinition.apiextensions.k8s.io/cephobjectstoreusers.ceph.rook.io created
customresourcedefinition.apiextensions.k8s.io/cephobjectzonegroups.ceph.rook.io created
customresourcedefinition.apiextensions.k8s.io/cephobjectzones.ceph.rook.io created
customresourcedefinition.apiextensions.k8s.io/cephrbdmirrors.ceph.rook.io created
customresourcedefinition.apiextensions.k8s.io/objectbucketclaims.objectbucket.io created
customresourcedefinition.apiextensions.k8s.io/objectbuckets.objectbucket.io created
customresourcedefinition.apiextensions.k8s.io/volumereplicationclasses.replication.storage.openshift.io created
customresourcedefinition.apiextensions.k8s.io/volumereplications.replication.storage.openshift.io created
customresourcedefinition.apiextensions.k8s.io/volumes.rook.io created
namespace/rook-ceph created
clusterrolebinding.rbac.authorization.k8s.io/rook-ceph-object-bucket created
serviceaccount/rook-ceph-admission-controller created
clusterrole.rbac.authorization.k8s.io/rook-ceph-admission-controller-role created
clusterrolebinding.rbac.authorization.k8s.io/rook-ceph-admission-controller-rolebinding created
clusterrole.rbac.authorization.k8s.io/rook-ceph-cluster-mgmt created
role.rbac.authorization.k8s.io/rook-ceph-system created
clusterrole.rbac.authorization.k8s.io/rook-ceph-global created
clusterrole.rbac.authorization.k8s.io/rook-ceph-mgr-cluster created
clusterrole.rbac.authorization.k8s.io/rook-ceph-object-bucket created
serviceaccount/rook-ceph-system created
rolebinding.rbac.authorization.k8s.io/rook-ceph-system created
clusterrolebinding.rbac.authorization.k8s.io/rook-ceph-global created
serviceaccount/rook-ceph-osd created
serviceaccount/rook-ceph-mgr created
serviceaccount/rook-ceph-cmd-reporter created
role.rbac.authorization.k8s.io/rook-ceph-osd created
clusterrole.rbac.authorization.k8s.io/rook-ceph-osd created
clusterrole.rbac.authorization.k8s.io/rook-ceph-mgr-system created
role.rbac.authorization.k8s.io/rook-ceph-mgr created
role.rbac.authorization.k8s.io/rook-ceph-cmd-reporter created
rolebinding.rbac.authorization.k8s.io/rook-ceph-cluster-mgmt created
rolebinding.rbac.authorization.k8s.io/rook-ceph-osd created
rolebinding.rbac.authorization.k8s.io/rook-ceph-mgr created
rolebinding.rbac.authorization.k8s.io/rook-ceph-mgr-system created
clusterrolebinding.rbac.authorization.k8s.io/rook-ceph-mgr-cluster created
clusterrolebinding.rbac.authorization.k8s.io/rook-ceph-osd created
rolebinding.rbac.authorization.k8s.io/rook-ceph-cmd-reporter created
podsecuritypolicy.policy/00-rook-privileged created
clusterrole.rbac.authorization.k8s.io/psp:rook created
clusterrolebinding.rbac.authorization.k8s.io/rook-ceph-system-psp created
rolebinding.rbac.authorization.k8s.io/rook-ceph-default-psp created
rolebinding.rbac.authorization.k8s.io/rook-ceph-osd-psp created
rolebinding.rbac.authorization.k8s.io/rook-ceph-mgr-psp created
rolebinding.rbac.authorization.k8s.io/rook-ceph-cmd-reporter-psp created
serviceaccount/rook-csi-cephfs-plugin-sa created
serviceaccount/rook-csi-cephfs-provisioner-sa created
role.rbac.authorization.k8s.io/cephfs-external-provisioner-cfg created
rolebinding.rbac.authorization.k8s.io/cephfs-csi-provisioner-role-cfg created
clusterrole.rbac.authorization.k8s.io/cephfs-csi-nodeplugin created
clusterrole.rbac.authorization.k8s.io/cephfs-external-provisioner-runner created
clusterrolebinding.rbac.authorization.k8s.io/rook-csi-cephfs-plugin-sa-psp created
clusterrolebinding.rbac.authorization.k8s.io/rook-csi-cephfs-provisioner-sa-psp created
clusterrolebinding.rbac.authorization.k8s.io/cephfs-csi-nodeplugin created
clusterrolebinding.rbac.authorization.k8s.io/cephfs-csi-provisioner-role created
serviceaccount/rook-csi-rbd-plugin-sa created
serviceaccount/rook-csi-rbd-provisioner-sa created
role.rbac.authorization.k8s.io/rbd-external-provisioner-cfg created
rolebinding.rbac.authorization.k8s.io/rbd-csi-provisioner-role-cfg created
clusterrole.rbac.authorization.k8s.io/rbd-csi-nodeplugin created
clusterrole.rbac.authorization.k8s.io/rbd-external-provisioner-runner created
clusterrolebinding.rbac.authorization.k8s.io/rook-csi-rbd-plugin-sa-psp created
clusterrolebinding.rbac.authorization.k8s.io/rook-csi-rbd-provisioner-sa-psp created
clusterrolebinding.rbac.authorization.k8s.io/rbd-csi-nodeplugin created
clusterrolebinding.rbac.authorization.k8s.io/rbd-csi-provisioner-role created
configmap/rook-ceph-operator-config created
deployment.apps/rook-ceph-operator created



[root@k8s-master1 ceph]# kubectl get all -n rook-ceph
NAME                                     READY   STATUS    RESTARTS   AGE
pod/rook-ceph-operator-5b94b79f5-tvtf8   1/1     Running   0          10m
pod/rook-discover-2n9px                  1/1     Running   0          5m52s
pod/rook-discover-542w9                  1/1     Running   0          5m52s
pod/rook-discover-nzgtp                  1/1     Running   0          5m52s
pod/rook-discover-qm64s                  1/1     Running   0          5m52s

NAME                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/rook-discover   4         4         4       4            4           <none>          5m52s

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/rook-ceph-operator   1/1     1            1           10m

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/rook-ceph-operator-5b94b79f5   1         1         1       10m
```



#### 1.4.创建Ceph Cluster

**cluster.yaml默认配置不修改**

```shell
# kubectl create -f cluster.yaml

[root@k8s-master ceph]# kubectl create -f cluster.yaml
cephcluster.ceph.rook.io/rook-ceph created


# 实时查看pod创建进度
kubectl get pod -n rook-ceph -w
# 实时查看集群创建进度
kubectl get cephcluster -n rook-ceph rook-ceph -w
# 详细描述
kubectl describe cephcluster -n rook-ceph rook-ceph




[root@k8s-master1 ceph]# kubectl get all -n rook-ceph
NAME                                                      READY   STATUS      RESTARTS   AGE
pod/csi-cephfsplugin-2gqgj                                3/3     Running     0          35m
pod/csi-cephfsplugin-6n5cr                                3/3     Running     0          35m
pod/csi-cephfsplugin-jsqxq                                3/3     Running     0          35m
pod/csi-cephfsplugin-mf5vz                                3/3     Running     0          35m
pod/csi-cephfsplugin-provisioner-575f74897d-mdsqg         6/6     Running     0          35m
pod/csi-cephfsplugin-provisioner-575f74897d-n5vxd         6/6     Running     0          35m
pod/csi-rbdplugin-8v984                                   3/3     Running     0          35m
pod/csi-rbdplugin-j868t                                   3/3     Running     0          35m
pod/csi-rbdplugin-ln5kf                                   3/3     Running     0          35m
pod/csi-rbdplugin-provisioner-8576bbbbc7-s8pmg            6/6     Running     0          35m
pod/csi-rbdplugin-provisioner-8576bbbbc7-xbd48            6/6     Running     0          35m
pod/csi-rbdplugin-wkg68                                   3/3     Running     0          35m
pod/rook-ceph-crashcollector-k8s-node1-7b85799995-9knfh   1/1     Running     0          24m
pod/rook-ceph-crashcollector-k8s-node2-5c6dd78c8-8rvdx    1/1     Running     0          24m
pod/rook-ceph-crashcollector-k8s-node3-864f9745b9-c8rwg   1/1     Running     0          24m
pod/rook-ceph-crashcollector-k8s-node4-64df8c7dc6-tpl5h   1/1     Running     0          24m
pod/rook-ceph-mgr-a-649b594c6b-8hkv9                      1/1     Running     0          24m
pod/rook-ceph-mon-a-6db6589c4b-gxlgl                      1/1     Running     0          31m
pod/rook-ceph-mon-b-84789c4d67-zqlnc                      1/1     Running     0          28m
pod/rook-ceph-mon-c-5765fd46d4-qhm7j                      1/1     Running     0          25m
pod/rook-ceph-operator-5b94b79f5-tvtf8                    1/1     Running     0          49m
pod/rook-ceph-osd-0-59cff4d448-5tmwd                      1/1     Running     0          24m
pod/rook-ceph-osd-1-c89d9c79-682lv                        1/1     Running     0          24m
pod/rook-ceph-osd-2-589c5c677c-7nddz                      1/1     Running     0          24m
pod/rook-ceph-osd-3-75457f8dd9-ht4sf                      1/1     Running     0          24m
pod/rook-ceph-osd-prepare-k8s-node1-zdwxl                 0/1     Completed   0          24m
pod/rook-ceph-osd-prepare-k8s-node2-kstlz                 0/1     Completed   0          24m
pod/rook-ceph-osd-prepare-k8s-node3-74khx                 0/1     Completed   0          24m
pod/rook-ceph-osd-prepare-k8s-node4-nxd5x                 0/1     Completed   0          24m
pod/rook-discover-2n9px                                   1/1     Running     0          45m
pod/rook-discover-542w9                                   1/1     Running     0          45m
pod/rook-discover-nzgtp                                   1/1     Running     0          45m
pod/rook-discover-qm64s                                   1/1     Running     0          45m

NAME                               TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE
service/csi-cephfsplugin-metrics   ClusterIP   10.1.251.118   <none>        8080/TCP,8081/TCP   35m
service/csi-rbdplugin-metrics      ClusterIP   10.1.181.21    <none>        8080/TCP,8081/TCP   35m
service/rook-ceph-mgr              ClusterIP   10.1.78.114    <none>        9283/TCP            24m
service/rook-ceph-mgr-dashboard    ClusterIP   10.1.61.67     <none>        8443/TCP            24m
service/rook-ceph-mon-a            ClusterIP   10.1.154.144   <none>        6789/TCP,3300/TCP   31m
service/rook-ceph-mon-b            ClusterIP   10.1.191.249   <none>        6789/TCP,3300/TCP   28m
service/rook-ceph-mon-c            ClusterIP   10.1.79.47     <none>        6789/TCP,3300/TCP   25m

NAME                              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/csi-cephfsplugin   4         4         4       4            4           <none>          35m
daemonset.apps/csi-rbdplugin      4         4         4       4            4           <none>          35m
daemonset.apps/rook-discover      4         4         4       4            4           <none>          45m

NAME                                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/csi-cephfsplugin-provisioner         2/2     2            2           35m
deployment.apps/csi-rbdplugin-provisioner            2/2     2            2           35m
deployment.apps/rook-ceph-crashcollector-k8s-node1   1/1     1            1           24m
deployment.apps/rook-ceph-crashcollector-k8s-node2   1/1     1            1           24m
deployment.apps/rook-ceph-crashcollector-k8s-node3   1/1     1            1           24m
deployment.apps/rook-ceph-crashcollector-k8s-node4   1/1     1            1           24m
deployment.apps/rook-ceph-mgr-a                      1/1     1            1           24m
deployment.apps/rook-ceph-mon-a                      1/1     1            1           31m
deployment.apps/rook-ceph-mon-b                      1/1     1            1           28m
deployment.apps/rook-ceph-mon-c                      1/1     1            1           25m
deployment.apps/rook-ceph-operator                   1/1     1            1           49m
deployment.apps/rook-ceph-osd-0                      1/1     1            1           24m
deployment.apps/rook-ceph-osd-1                      1/1     1            1           24m
deployment.apps/rook-ceph-osd-2                      1/1     1            1           24m
deployment.apps/rook-ceph-osd-3                      1/1     1            1           24m

NAME                                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/csi-cephfsplugin-provisioner-575f74897d         2         2         2       35m
replicaset.apps/csi-rbdplugin-provisioner-8576bbbbc7            2         2         2       35m
replicaset.apps/rook-ceph-crashcollector-k8s-node1-7b85799995   1         1         1       24m
replicaset.apps/rook-ceph-crashcollector-k8s-node2-5c6dd78c8    1         1         1       24m
replicaset.apps/rook-ceph-crashcollector-k8s-node3-864f9745b9   1         1         1       24m
replicaset.apps/rook-ceph-crashcollector-k8s-node3-fc65d5787    0         0         0       24m
replicaset.apps/rook-ceph-crashcollector-k8s-node4-64df8c7dc6   1         1         1       24m
replicaset.apps/rook-ceph-mgr-a-649b594c6b                      1         1         1       24m
replicaset.apps/rook-ceph-mon-a-6db6589c4b                      1         1         1       31m
replicaset.apps/rook-ceph-mon-b-84789c4d67                      1         1         1       28m
replicaset.apps/rook-ceph-mon-c-5765fd46d4                      1         1         1       25m
replicaset.apps/rook-ceph-operator-5b94b79f5                    1         1         1       49m
replicaset.apps/rook-ceph-osd-0-59cff4d448                      1         1         1       24m
replicaset.apps/rook-ceph-osd-1-c89d9c79                        1         1         1       24m
replicaset.apps/rook-ceph-osd-2-589c5c677c                      1         1         1       24m
replicaset.apps/rook-ceph-osd-3-75457f8dd9                      1         1         1       24m

NAME                                        COMPLETIONS   DURATION   AGE
job.batch/rook-ceph-osd-prepare-k8s-node1   1/1           4s         24m
job.batch/rook-ceph-osd-prepare-k8s-node2   1/1           4s         24m
job.batch/rook-ceph-osd-prepare-k8s-node3   1/1           4s         24m
job.batch/rook-ceph-osd-prepare-k8s-node4   1/1           4s         24m



# 查看日志
[root@k8s-master ~]# kubectl logs -f rook-ceph-operator-5b94b79f5-72jw9 -n rook-ceph

```



#### 1.5.安装Toolbox

```shell
# 安装toolbox工具
kubectl create -f toolbox.yaml

[root@k8s-master ceph]# kubectl create -f toolbox.yaml
deployment.apps/rook-ceph-tools created


- 进入工作目录
cd /root/rook/cluster/examples/kubernetes/ceph
- 创建toolbox
kubectl  create -f toolbox.yaml -n rook-ceph
- 查看pod
kubectl  get pod -n rook-ceph -l app=rook-ceph-tools
- 进入pod
kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash
- 查看集群状态
ceph status
- 查看osd状态
ceph osd status
- 集群空间用量
ceph df

ceph status
ceph osd status
ceph df
rados df



[root@k8s-master1 ceph]# kubectl  get pod -n rook-ceph -l app=rook-ceph-tools
NAME                               READY   STATUS    RESTARTS   AGE
rook-ceph-tools-6bc7c4f9fc-s5k87   1/1     Running   0          42s



[root@k8s-master ~]# kubectl -n rook-ceph exec -it rook-ceph-tools-6bc7c4f9fc-p5j59 -- bash
[root@k8s-master ~]# kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash
[root@rook-ceph-tools-6bc7c4f9fc-s5k87 /]# ceph status
  cluster:
    id:     0baad0d4-c9b0-4be3-a3be-e0c779394658
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum a,c,b (age 27m)
    mgr: a(active, since 27m)
    osd: 4 osds: 4 up (since 27m), 4 in (since 27m)
 
  data:
    pools:   1 pools, 1 pgs
    objects: 0 objects, 0 B
    usage:   4.0 GiB used, 796 GiB / 800 GiB avail
    pgs:     1 active+clean
 


[root@rook-ceph-tools-6bc7c4f9fc-p5j59 /]# ceph osd status
ID  HOST        USED  AVAIL  WR OPS  WR DATA  RD OPS  RD DATA  STATE      
 0  k8s-node3  1026M  98.9G      0        0       0        0   exists,up  
 1  k8s-node2  1026M  98.9G      0        0       0        0   exists,up  
 2  k8s-node1  1026M  98.9G      0        0       0        0   exists,up 

 
[root@rook-ceph-tools-6bc7c4f9fc-s5k87 /]# ceph df
--- RAW STORAGE ---
CLASS  SIZE     AVAIL    USED     RAW USED  %RAW USED
hdd    800 GiB  796 GiB  9.8 MiB   4.0 GiB       0.50
TOTAL  800 GiB  796 GiB  9.8 MiB   4.0 GiB       0.50
 
--- POOLS ---
POOL                   ID  PGS  STORED  OBJECTS  USED  %USED  MAX AVAIL
device_health_metrics   1    1     0 B        0   0 B      0    252 GiB


[root@rook-ceph-tools-6bc7c4f9fc-s5k87 /]# ceph df
--- RAW STORAGE ---
CLASS  SIZE     AVAIL    USED     RAW USED  %RAW USED
hdd    800 GiB  796 GiB  9.8 MiB   4.0 GiB       0.50
TOTAL  800 GiB  796 GiB  9.8 MiB   4.0 GiB       0.50
 
--- POOLS ---
POOL                   ID  PGS  STORED  OBJECTS  USED  %USED  MAX AVAIL
device_health_metrics   1    1     0 B        0   0 B      0    252 GiB


[root@rook-ceph-tools-6bc7c4f9fc-s5k87 /]# rados df
POOL_NAME              USED  OBJECTS  CLONES  COPIES  MISSING_ON_PRIMARY  UNFOUND  DEGRADED  RD_OPS   RD  WR_OPS   WR  USED COMPR  UNDER COMPR
device_health_metrics   0 B        0       0       0                   0        0         0       0  0 B       0  0 B         0 B          0 B

total_objects    0
total_used       4.0 GiB
total_avail      796 GiB
total_space      800 GiB


[root@rook-ceph-tools-6bc7c4f9fc-s5k87 /]# ceph health detail
HEALTH_OK
```



#### 1.6.Dashboard

Ceph 有一个 Dashboard 工具，我们可以在上面查看集群的状态，包括总体运行状态，mgr、osd 和其他 Ceph 进程的状态，查看池和 PG 状态，以及显示守护进程的日志等等。

Dashboard 服务，如果在集群内部我们可以通过 DNS 名称 http://rook-ceph-mgr-dashboard.rook-ceph:7000 或者 CluterIP http://10.109.8.98:7000 来进行访问。

但是如果要在集群外部进行访问的话，我们就需要通过 Ingress 或者 NodePort 类型的 Service 来暴露了，为了方便测试我们这里创建一个新的 NodePort 类型的服务来访问 Dashboard，资源清单如下所示：（dashboard-external-http.yaml）

通过 `http://ip:port`就可以访问到 Dashboard 了，

Rook 创建了一个默认的用户 admin，并在运行 Rook 的命名空间中生成了一个名为 rook-ceph-dashboard-admin-password 的 Secret，要获取密码，可以运行以下命令：

```shell
kubectl -n rook-ceph get secret rook-ceph-dashboard-password -o jsonpath="{['data']['password']}" | base64 --decode && echo


[root@k8s-master ~]# kubectl -n rook-ceph get secret rook-ceph-dashboard-password -o jsonpath="{['data']['password']}" | base64 --decode && echo
Bl8c"Ab3+6-Q?N()~agu
```



**默认配置实ssl，只能通过HTTPS访问，不能通过HTTP访问。**



**1.HTTPS**

rook-dashboard-https.yaml

```shell
apiVersion: v1
kind: Service
metadata:
  name: rook-ceph-mgr-dashboard-https
  namespace: rook-ceph
  labels:
    app: rook-ceph-mgr
    rook_cluster: rook-ceph
spec:
  ports:
  - name: dashboard
    port: 8443
    nodePort: 32700
    protocol: TCP
    targetPort: 8443
  selector:
    app: rook-ceph-mgr
    rook_cluster: rook-ceph
  sessionAffinity: None
  type: NodePort



# 创建
kubectl apply -f rook-dashboard-https.yaml


[root@k8s-master1 rook]# kubectl get svc -n rook-ceph
NAME                            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE
rook-ceph-mgr-dashboard         ClusterIP   10.1.61.67     <none>        8443/TCP            35m
rook-ceph-mgr-dashboard-https   NodePort    10.1.236.73    <none>        8443:32700/TCP      11s



# 访问地址：
https://172.51.216.81:32700/
admin
Bl8c"Ab3+6-Q?N()~agu

# 修改密码： 1qaz2wsx
```



![](/images/kubernetes/pro/module/rook-1.png)

![](/images/kubernetes/pro/module/rook-2.png)



#### 1.7.块存储（RBD）

**创建CephBlockPool和StorageClass**

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



```shell
# 创建CephBlockPool和StorageClass

[root@k8s-master1 rook]# kubectl apply -f storageclass.yaml
cephblockpool.ceph.rook.io/replicapool created
storageclass.storage.k8s.io/rook-ceph-block created



# 查看sc
[root@k8s-master1 rook]# kubectl get sc
NAME              PROVISIONER                  RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
rook-ceph-block   rook-ceph.rbd.csi.ceph.com   Delete          Immediate           true                   30s


# 查看CephBlockPool（也可在dashboard中查看）
[root@k8s-master1 rook]# kubectl get cephblockpools -n rook-ceph
NAME          AGE
replicapool   58s
```

![](/images/kubernetes/pro/module/rook-3.png)



#### 1.8.共享文件存储（CephFS）

**创建filesystem和StorageClass**

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
[root@k8s-master1 rook]# kubectl apply -f filesystem.yaml
cephfilesystem.ceph.rook.io/myfs created


[root@k8s-master1 rook]# kubectl get CephFilesystem -n rook-ceph
NAME   ACTIVEMDS   AGE   PHASE
myfs   3           22s   Ready


[root@k8s-master1 rook]# kubectl -n rook-ceph get pod -l app=rook-ceph-mds
NAME                                    READY   STATUS    RESTARTS   AGE
rook-ceph-mds-myfs-a-65d54cb647-mvk9z   1/1     Running   0          29s
rook-ceph-mds-myfs-b-5d9db94db9-rzcwv   1/1     Running   0          28s
rook-ceph-mds-myfs-c-58fb4f6b6c-zq6sx   1/1     Running   0          27s
rook-ceph-mds-myfs-d-64f895bd7c-87dv4   1/1     Running   0          27s
rook-ceph-mds-myfs-e-78c85876c9-sjsfl   1/1     Running   0          26s
rook-ceph-mds-myfs-f-54749fbf49-xczts   1/1     Running   0          25s



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
[root@k8s-master1 rook]# pwd
/k8s/module/rook
[root@k8s-master1 rook]# ll
total 3420
drwxrwxr-x. 11 root root    4096 Jun 11  2021 rook



[root@k8s-master1 rook]# kubectl apply -f rook/cluster/examples/kubernetes/ceph/csi/cephfs/storageclass.yaml
storageclass.storage.k8s.io/rook-cephfs created


[root@k8s-master1 rook]# kubectl get sc
NAME              PROVISIONER                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
rook-ceph-block   rook-ceph.rbd.csi.ceph.com      Delete          Immediate           true                   12m
rook-cephfs       rook-ceph.cephfs.csi.ceph.com   Delete          Immediate           true                   71s
```

![](/images/kubernetes/pro/module/rook-4.png)



