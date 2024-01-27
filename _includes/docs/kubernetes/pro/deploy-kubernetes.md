

* TOC
{:toc}



## 一、安装说明



### 1. 部署方式介绍

**目前生产部署 Kubernetes 集群主要有两种方式：**

- **kubeadm** 
  Kubeadm 是一个 K8s 部署工具， 提供 kubeadm init 和 kubeadm join， 用于快速部
  署 Kubernetes 集群。
  官方地址： https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/

- **二进制包**
  从 github 下载发行版的二进制包， 手动部署每个组件， 组成 Kubernetes 集群。
  Kubeadm 降低部署门槛， 但屏蔽了很多细节， 遇到问题很难排查。 如果想更容易可
  控， 推荐使用二进制包部署 Kubernetes 集群， 虽然手动部署麻烦点， 期间可以学习很
  多工作原理， 也利于后期维护。  



### 2. kubeadm 部署方式介绍  

kubeadm 是官方社区推出的一个用于快速部署 kubernetes 集群的工具， 这个工具能通
过两条指令完成一个 kubernetes 集群的部署：
第一、 创建一个 Master 节点 kubeadm init
第二， 将 Node 节点加入到当前集群中 $ kubeadm join <Master 节点的 IP 和端口 >  

这个工具能通过两条指令完成一个kubernetes集群的部署：

```shell
# 创建一个 Master 节点
$ kubeadm init

# 将一个 Node 节点加入到当前集群中
$ kubeadm join <Master节点的IP和端口 >
```





## 二、k8s集群部署



### 1. 部署环境

**节点配置**

| 主机名     | IP地址        | 角色   |
| ---------- | ------------- | ------ |
| k8s-master | 172.51.216.81 | master |
| k8s-node1  | 172.51.216.82 | node   |
| k8s-node2  | 172.51.216.83 | node   |
| k8s-node3  | 172.51.216.84 | node   |

**版本信息**

| 信息   | 版本     | 备注                       |
| ------ | -------- | -------------------------- |
| K8s    | 1.20.6   |                            |
| centos | 7.9.2009 | # cat  /etc/redhat-release |
| Docker | 19.03.15 | 建议 19.03.x               |



### 2. 环境准备

- **关闭防火墙**

```shell
#关闭防火墙

systemctl stop firewalld
systemctl disable firewalld
```

- **关闭selinux**

```shell
# 关闭selinux

# 永久
sed -i 's/enforcing/disabled/' /etc/selinux/config
# 临时
setenforce 0
```

- **关闭swap**

```shell
# 关闭swap

# 临时
swapoff -a
# 永久
sed -ri 's/.*swap.*/#&/' /etc/fstab
```

- **设置主机名**

```shell
# 根据规划设置主机名
hostnamectl set-hostname <hostname>

#分别在每台主机上设置
hostnamectl set-hostname k8s-master
hostnamectl set-hostname k8s-node1
hostnamectl set-hostname k8s-node2
hostnamectl set-hostname k8s-node3
```

- **在所有节点添加hosts**

```shell
# 在所有节点添加hosts

cat >> /etc/hosts << EOF
172.51.216.81 k8s-master
172.51.216.82 k8s-node1
172.51.216.83 k8s-node2
172.51.216.84 k8s-node3
EOF
```

- **将桥接的IPv4流量传递到iptables的链**

```shell
# 将桥接的IPv4流量传递到iptables的链

cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

# 生效
sysctl --system  
```

- **时间同步**

```shell
# 时间同步
yum install ntpdate -y
ntpdate time.windows.com


------
# 同步公网NTP服务时间

# 安装
yum install chrony
# 启用
systemctl start chronyd
systemctl enable chronyd
# 设置亚洲时区
timedatectl set-timezone Asia/Shanghai
# 启用NTP同步
timedatectl set-ntp yes
```



### 3. 安装Docker

- **安装Docker（所有节点）**

安装版本19.03.*

```shell
$ wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
$ yum -y install docker-ce-19.03.15-3.el7
$ systemctl enable docker && systemctl start docker
$ docker --version


# 参考  本次安装版本有问题
yum -y install docker-ce-19.03.*
```

- **添加阿里云加速镜像**

```shell
# 添加阿里云加速镜像

cat > /etc/docker/daemon.json << EOF
{
   "registry-mirrors": ["https://gcctk8ld.mirror.aliyuncs.com"]
} 
EOF
```

- **重启docker**

```shell
#重启docker

systemctl restart docker
```



查看版本

```shell
# 用下面的命令可以查看可以安装的版本

yum list docker-ce --showduplicates | sort -r
```



- **添加阿里云YUM软件源（所有服务器）**

```shell
# 添加阿里云YUM软件源（所有服务器）
 
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```



### 4. 安装kubeadm、kubelet和kubectl

安装kubeadm，kubelet和kubectl（所有服务器）

```shell
#安装组件
yum install -y kubelet-1.20.6 kubeadm-1.20.6 kubectl-1.20.6
 
#设置开机启动
systemctl enable kubelet
```



查看版本

```shell
# 可以用下面的命令查看可以安装的版本

yum list kubeadm --showduplicates | sort -r
```



### 5. 部署Kubernetes Master（k8s-master）

部署Kubernetes Master（k8s-master 172.51.216.81），此命令只在主节点（Master）执行。

```shell
# 部署Kubernetes Master（k8s-master 172.51.216.81）
 
#在172.51.216.81（Master）执行

kubeadm init \
--apiserver-advertise-address=172.51.216.81 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.20.6 \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16
```

说明

```shell
# 说明

kubeadm init \
--apiserver-advertise-address=172.17.88.27 \ #master节点IP
--image-repository registry.aliyuncs.com/google_containers \ #指定拉取仓库
--kubernetes-version v1.18.0 \ #指定k8s版本
--service-cidr=10.96.0.0/12 \ #指明pod网络可以使用的IP地址段
--pod-network-cidr=10.244.0.0/16 #为service的虚拟IP地址另外指定IP地址段
```

执行结果

```shell
# 执行成功

[root@k8s-master ~]# kubeadm init \
> --apiserver-advertise-address=172.51.216.81 \
> --image-repository registry.aliyuncs.com/google_containers \
> --kubernetes-version v1.20.6 \
> --service-cidr=10.96.0.0/12 \
> --pod-network-cidr=10.244.0.0/16
[init] Using Kubernetes version: v1.20.6
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 172.51.216.81]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-master localhost] and IPs [172.51.216.81 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-master localhost] and IPs [172.51.216.81 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[kubelet-check] Initial timeout of 40s passed.
[apiclient] All control plane components are healthy after 73.002096 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.20" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node k8s-master as control-plane by adding the labels "node-role.kubernetes.io/master=''" and "node-role.kubernetes.io/control-plane='' (deprecated)"
[mark-control-plane] Marking the node k8s-master as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: kx8q8v.fg193i70v4hgzeh0
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.51.216.81:6443 --token kx8q8v.fg193i70v4hgzeh0 \
    --discovery-token-ca-cert-hash sha256:ea6e05a86b0b8d0d5c3cd9d7a0f920dd4b382de23c28a1ac7325f2105186f88c
```

```shell
# 说明


# 执行成功
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

# 操作1
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

# 操作 加入node
kubeadm join 172.51.216.81:6443 --token kx8q8v.fg193i70v4hgzeh0 \
    --discovery-token-ca-cert-hash sha256:ea6e05a86b0b8d0d5c3cd9d7a0f920dd4b382de23c28a1ac7325f2105186f88c
```



**k8s-master执行**

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config


# 执行
[root@k8s-master ~]# mkdir -p $HOME/.kube
[root@k8s-master ~]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@k8s-master ~]# sudo chown $(id -u):$(id -g) $HOME/.kube/config
```



**查看集群状态**

```bash
$ kubectl get cs
$ kubectl get nodes
$ kubectl get pods -n kube-system
```

```shell
[root@k8s-master ~]# kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS      MESSAGE                                                                                       ERROR
scheduler            Unhealthy   Get "http://127.0.0.1:10251/healthz": dial tcp 127.0.0.1:10251: connect: connection refused   
controller-manager   Unhealthy   Get "http://127.0.0.1:10252/healthz": dial tcp 127.0.0.1:10252: connect: connection refused   
etcd-0               Healthy     {"health":"true"}                                                                             

[root@k8s-master ~]# kubectl get nodes
NAME         STATUS     ROLES                  AGE     VERSION
k8s-master   NotReady   control-plane,master   5m15s   v1.20.6


[root@k8s-master ~]# kubectl get pods -n kube-system
NAME                                 READY   STATUS    RESTARTS   AGE
coredns-7f89b7bc75-bw8ms             0/1     Pending   0          5m12s
coredns-7f89b7bc75-k692f             0/1     Pending   0          5m12s
etcd-k8s-master                      1/1     Running   0          5m27s
kube-apiserver-k8s-master            1/1     Running   0          5m27s
kube-controller-manager-k8s-master   1/1     Running   0          5m27s
kube-proxy-g4mm5                     1/1     Running   0          5m12s
kube-scheduler-k8s-master            1/1     Running   0          5m27s
```



**node节点加入集群**

```shell
# node节点加入集群

kubeadm join 172.51.216.81:6443 --token kx8q8v.fg193i70v4hgzeh0 \
    --discovery-token-ca-cert-hash sha256:ea6e05a86b0b8d0d5c3cd9d7a0f920dd4b382de23c28a1ac7325f2105186f88c


#注：token有效期24小时，过去失效，如获取新的使用命令
sudo kubeadm token create --print-join-command
```



### 6. 加入Kubernetes Node

此命令在3个子节点（Node）执行。

向集群添加新节点，执行在kubeadm init输出的kubeadm join命令：
**注意：如下命令要复制你根据kubeadm init生成的命令**

```shell
# node节点加入集群

kubeadm join 172.51.216.81:6443 --token kx8q8v.fg193i70v4hgzeh0 \
    --discovery-token-ca-cert-hash sha256:ea6e05a86b0b8d0d5c3cd9d7a0f920dd4b382de23c28a1ac7325f2105186f88c


#注：token有效期24小时，过去失效，如获取新的使用命令
sudo kubeadm token create --print-join-command
```

查看集群状态

```shell
# 查看集群状态
kubectl get nodes

[root@k8s-master ~]# kubectl get nodes
NAME         STATUS     ROLES                  AGE     VERSION
k8s-master   NotReady   control-plane,master   9m17s   v1.20.6
k8s-node1    NotReady   <none>                 24s     v1.20.6
k8s-node2    NotReady   <none>                 17s     v1.20.6
k8s-node3    NotReady   <none>                 11s     v1.20.6

# 由于k8s集群网络不同，status状态为NotReady,需使用flannel网络组件是k8s网络畅通 
```



### 7. 部署CNI网络插件

#### 7.1. Calico

安装calico

```shell
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

运行完成之后，等待calico pod 变成Running

```shell
kubectl get pod --all-namespaces


[root@k8s-master ~]# kubectl get pod --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-558995777d-pp6vr   1/1     Running   0          3m52s
kube-system   calico-node-6pw6w                          1/1     Running   0          3m53s
kube-system   calico-node-74c5m                          1/1     Running   0          3m53s
kube-system   calico-node-rqh92                          1/1     Running   0          3m53s
kube-system   calico-node-wk4lv                          1/1     Running   0          3m53s
kube-system   coredns-7f89b7bc75-bw8ms                   1/1     Running   0          14m
kube-system   coredns-7f89b7bc75-k692f                   1/1     Running   0          14m
kube-system   etcd-k8s-master                            1/1     Running   0          14m
kube-system   kube-apiserver-k8s-master                  1/1     Running   0          14m
kube-system   kube-controller-manager-k8s-master         1/1     Running   0          14m
kube-system   kube-proxy-42hcw                           1/1     Running   0          5m31s
kube-system   kube-proxy-8bnzz                           1/1     Running   0          5m37s
kube-system   kube-proxy-g4mm5                           1/1     Running   0          14m
kube-system   kube-proxy-wsv29                           1/1     Running   0          5m44s
kube-system   kube-scheduler-k8s-master                  1/1     Running   0          14m
```

测试

```shell
kubectl get pods -n kube-system
kubectl get nodes
kubectl get pods --all-namespaces

[root@k8s-master ~]# kubectl get nodes
NAME         STATUS   ROLES                  AGE     VERSION
k8s-master   Ready    control-plane,master   15m     v1.20.6
k8s-node1    Ready    <none>                 6m19s   v1.20.6
k8s-node2    Ready    <none>                 6m12s   v1.20.6
k8s-node3    Ready    <none>                 6m6s    v1.20.6
```



### 8. 测试KUBERNETES集群

在Kubernetes集群中创建一个pod，验证是否正常运行：

```shell
$ kubectl create deployment nginx --image=nginx
$ kubectl expose deployment nginx --port=80 --type=NodePort
$ kubectl get pod,svc
```



```shell
[root@localhost ~]# kubectl create deployment nginx --image=nginx
deployment.apps/nginx created

[root@localhost ~]# kubectl expose deployment nginx --port=80 --type=NodePort
service/nginx exposed

[root@k8s-master ~]# kubectl get pod
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6799fc88d8-8w75p   1/1     Running   0          105s

[root@k8s-master ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        18m
nginx        NodePort    10.98.155.106   <none>        80:31449/TCP   2m50s


# 访问地址：http://NodeIP:Port
http://172.51.216.81:31449
```





## 三、安装dashboard



### 1. 根据  yaml部署dashboard

官网地址：https://kubernetes.io/zh/docs/tasks/access-application-cluster/web-ui-dashboard/

github地址：https://github.com/kubernetes/dashboard



在master上创建（根据k8s版本选择版本）

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml
```

```shell
[root@localhost ~]# kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created



[root@k8s-master ~]# kubectl get all -n kubernetes-dashboard
NAME                                             READY   STATUS    RESTARTS   AGE
pod/dashboard-metrics-scraper-79c5968bdc-9c9hf   1/1     Running   0          66s
pod/kubernetes-dashboard-658485d5c7-jlbsl        1/1     Running   0          66s

NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/dashboard-metrics-scraper   ClusterIP   10.96.233.196   <none>        8000/TCP   66s
service/kubernetes-dashboard        ClusterIP   10.97.0.33      <none>        443/TCP    66s

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dashboard-metrics-scraper   1/1     1            1           66s
deployment.apps/kubernetes-dashboard        1/1     1            1           66s

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/dashboard-metrics-scraper-79c5968bdc   1         1         1       66s
replicaset.apps/kubernetes-dashboard-658485d5c7        1         1         1       66s
```



### 2. 设置svc为NodePort

```shell
kubectl  patch svc kubernetes-dashboard -n kubernetes-dashboard -p '{"spec":{"type":"NodePort","ports": [{"port":443,"targetPort":8443,"nodePort":30443}]}}'
```



### 3. 配置认证方式

创建dashboard-adminuser.yaml

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard  
```

创建登录用户

```shell
kubectl apply -f dashboard-adminuser.yaml
```

查看admin-user账户的token

```shell
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetesdashboard get secret | grep admin-user | awk '{print $1}')
```

执行结果：

```shell
[root@k8s-master dashboard]# kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetesdashboard get secret | grep admin-user | awk '{print $1}')
No resources found in kubernetesdashboard namespace.
Name:         admin-user-token-px45h
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: 7c17b305-6f06-4073-b9bd-8980d2f7bb1e

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  20 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6ImVjQTZKRUd2U1BBZTRDcm1jd0ZzMHpvTGQxNVZSb3AtcmFOQkRPTi14N1kifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXB4NDVoIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI3YzE3YjMwNS02ZjA2LTQwNzMtYjliZC04OTgwZDJmN2JiMWUiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.iigCoOapTAY34INm-XK5luJuBto_a0SLB3yAI2kdKoChGK6XcJUWxQ_kHUVbmoTQeewZAPt_2TPdYun46OkD17gkMgaklT74gudMctcmcU_uoeb9MR2jQR_nxTO0vG72YkI3mmftkB9U8IA5QINl4eE9rUwnCP7YUEZ8DBqraiQ5Fn2vJL-IBSRgdUgcevU2W44khXg5AG1_EXLkMle0_8qkr-cJgp-mNvUVkRUt_PiWGF4ty14V2R4QyTysbDmu-CDFVSibDDzOgc1KQK0r9vtQf-GAXzFJ-GMzmm71GrjrularONiLbby9OvmFjh5PQ9b0Iv3BKoS-tEAi4dh89g



# admin-user-token-xxxx账号密码
```

访问令牌

```shell
# Name:         admin-user-token-hpg8g

token:      eyJhbGciOiJSUzI1NiIsImtpZCI6ImVjQTZKRUd2U1BBZTRDcm1jd0ZzMHpvTGQxNVZSb3AtcmFOQkRPTi14N1kifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXB4NDVoIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI3YzE3YjMwNS02ZjA2LTQwNzMtYjliZC04OTgwZDJmN2JiMWUiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.iigCoOapTAY34INm-XK5luJuBto_a0SLB3yAI2kdKoChGK6XcJUWxQ_kHUVbmoTQeewZAPt_2TPdYun46OkD17gkMgaklT74gudMctcmcU_uoeb9MR2jQR_nxTO0vG72YkI3mmftkB9U8IA5QINl4eE9rUwnCP7YUEZ8DBqraiQ5Fn2vJL-IBSRgdUgcevU2W44khXg5AG1_EXLkMle0_8qkr-cJgp-mNvUVkRUt_PiWGF4ty14V2R4QyTysbDmu-CDFVSibDDzOgc1KQK0r9vtQf-GAXzFJ-GMzmm71GrjrularONiLbby9OvmFjh5PQ9b0Iv3BKoS-tEAi4dh89g
```



### 4. 浏览器访问

浏览器登陆任意node节点IP+30443,填入获取的token
https://172.51.216.81:30443

火狐浏览器、chrome、edge可以直接访问

![](/images/kubernetes/pro/deploy-kubernetes/dashboard-login.png)

Dashboard

![](/images/kubernetes/pro/deploy-kubernetes/dashboard-first.png)



在谷歌浏览器（Chrome）启动文件中加入启动参数，用于解决无法访问Dashboard的问题，参考图：

```shell
--test-type --ignore-certificate-errors
```

![chrome设置](/images/kubernetes/pro/deploy-kubernetes/chrome.png)





## 四、k8s证书有效期



**使用kubeadm搭建K8s集群，默认自动生成的证书有效期只有 1 年，因此需要每年手动更新一次证书，这种形式显然对实际生产环境来说很不友好；因此下面教给大家修改这个过期时间的终极方法。**



https://blog.csdn.net/m0_37814112/article/details/115899660



### 1.查看证书有效期

```shell
# kubeadm alpha certs check-expiration


[root@k8s-master ~]# kubeadm alpha certs check-expiration
Command "check-expiration" is deprecated, please use the same command under "kubeadm certs"
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Dec 24, 2022 01:05 UTC   364d                                    no      
apiserver                  Dec 24, 2022 01:05 UTC   364d            ca                      no      
apiserver-etcd-client      Dec 24, 2022 01:05 UTC   364d            etcd-ca                 no      
apiserver-kubelet-client   Dec 24, 2022 01:05 UTC   364d            ca                      no      
controller-manager.conf    Dec 24, 2022 01:05 UTC   364d                                    no      
etcd-healthcheck-client    Dec 24, 2022 01:05 UTC   364d            etcd-ca                 no      
etcd-peer                  Dec 24, 2022 01:05 UTC   364d            etcd-ca                 no      
etcd-server                Dec 24, 2022 01:05 UTC   364d            etcd-ca                 no      
front-proxy-client         Dec 24, 2022 01:05 UTC   364d            front-proxy-ca          no      
scheduler.conf             Dec 24, 2022 01:05 UTC   364d                                    no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Dec 22, 2031 01:05 UTC   9y              no      
etcd-ca                 Dec 22, 2031 01:05 UTC   9y              no      
front-proxy-ca          Dec 22, 2031 01:05 UTC   9y              no
```



**查看kubeadm版本号**

```shell
# 查看kubeadm版本号


[root@k8s-master1 ~]# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"20", GitVersion:"v1.20.6", GitCommit:"8a62859e515889f07e3e3be6a1080413f17cf2c3", GitTreeState:"clean", BuildDate:"2021-04-15T03:26:21Z", GoVersion:"go1.15.10", Compiler:"gc", Platform:"linux/amd64"}
```



### 2.Go 环境部署

```shell
[root@k8s-master ~]# wget https://dl.google.com/go/go1.17.4.linux-amd64.tar.gz
Connecting to dl.google.com (120.253.255.33:443)
go1.17.4.linux-amd64 100% |************************************************************************************|   128M  0:00:00 ETA


[root@localhost ~]# tar -zxvf go1.17.4.linux-amd64.tar.gz -C /usr/local


[root@localhost ~]# vim /etc/profile
export PATH=$PATH:/usr/local/go/bin


[root@localhost ~]# source /etc/profile
```



### 3.下载 K8S 源码

```shell
# 也可用此种方式


wget https://github.com/kubernetes/kubernetes/archive/refs/tags/v1.20.6.tar.gz

tar -zxf kubernetes-1.20.6.tar.gz
```



### 4.修改Kubeadm源码编译

```shell
# 1、添加内容
[root@k8s-master1 ~]# cd kubernetes-1.20.6

vim cmd/kubeadm/app/util/pkiutil/pki_helpers.go # kubeadm1.14版本之后
const duration3650days = time.Hour * 24 * 365 * 10
NotAfter:     time.Now().Add(duration3650days).UTC(),

# 注意：vim staging/src/k8s.io/client-go/util/cert/cert.go  #kubeadm1.14版本之前


# 2、注释内容（如下图所示）
// NotAfter:     time.Now().Add(kubeadmconstants.CertificateValidity).UTC(),



-------------------------------------------------------------------
# 修改
        const duration3650days = time.Hour * 24 * 365 * 10
        NotAfter := time.Now().Add(duration3650days).UTC()
        //NotAfter := time.Now().Add(kubeadmconstants.CertificateValidity).UTC()


# 修改后内容
# 用tab键对齐不要用空格
[root@localhost kubernetes]# vim cmd/kubeadm/app/util/pkiutil/pki_helpers.go


// NewSignedCert creates a signed certificate using the given CA certificate and key
func NewSignedCert(cfg *CertConfig, key crypto.Signer, caCert *x509.Certificate, caKey crypto.Signer) (*x509.Certificate, error) {
        const duration3650days = time.Hour * 24 * 365 * 10
        serial, err := cryptorand.Int(cryptorand.Reader, new(big.Int).SetInt64(math.MaxInt64))
        if err != nil {
                return nil, err
        }
        if len(cfg.CommonName) == 0 {
                return nil, errors.New("must specify a CommonName")
        }
        if len(cfg.Usages) == 0 {
                return nil, errors.New("must specify at least one ExtKeyUsage")
        }

        RemoveDuplicateAltNames(&cfg.AltNames)

        certTmpl := x509.Certificate{
                Subject: pkix.Name{
                        CommonName:   cfg.CommonName,
                        Organization: cfg.Organization,
                },
                DNSNames:     cfg.AltNames.DNSNames,
                IPAddresses:  cfg.AltNames.IPs,
                SerialNumber: serial,
                NotBefore:    caCert.NotBefore,
                NotAfter:     time.Now().Add(duration3650days).UTC(),
                //NotAfter:     time.Now().Add(kubeadmconstants.CertificateValidity).UTC(),
                KeyUsage:     x509.KeyUsageKeyEncipherment | x509.KeyUsageDigitalSignature,
                ExtKeyUsage:  cfg.Usages,
        }
        certDERBytes, err := x509.CreateCertificate(cryptorand.Reader, &certTmpl, caCert, key.Public(), caKey)
        if err != nil {
                return nil, err
        }
        return x509.ParseCertificate(certDERBytes)
}
```

```shell
# 编译

[root@localhost kubernetes]# yum install rsync -y

[root@localhost kubernetes]# make WHAT=cmd/kubeadm GOFLAGS=-v
```

![](/images/kubernetes/pro/deploy-kubernetes/k8s-4.png)



### 5.证书更新

```shell
# 原证书备份
kubernetes-1.20.6
cp -arp /etc/kubernetes /etc/kubernetes_`date +%F`
mv /usr/bin/kubeadm /usr/bin/kubeadm_`date +%F`
cp -a _output/bin/kubeadm /usr/bin



[root@localhost kubernetes-1.20.6]# cp -arp /etc/kubernetes /etc/kubernetes_`date +%F`
[root@localhost kubernetes-1.20.6]# mv /usr/bin/kubeadm /usr/bin/kubeadm_`date +%F`
[root@localhost kubernetes-1.20.6]# cp -a _output/bin/kubeadm /usr/bin
```



```shell
# 更新证书

kubeadm config view > /root/kubeadm-config.yaml
kubeadm alpha certs renew all --config=/root/kubeadm-config.yaml



[root@localhost kubernetes-1.20.6]# kubeadm config view > /root/kubeadm-config.yaml
Command "view" is deprecated, This command is deprecated and will be removed in a future release, please use 'kubectl get cm -o yaml -n kube-system kubeadm-config' to get the kubeadm config directly.

[root@localhost kubernetes-1.20.6]# kubeadm alpha certs renew all --config=/root/kubeadm-config.yaml
Command "all" is deprecated, please use the same command under "kubeadm certs"
certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself renewed
certificate for serving the Kubernetes API renewed
certificate the apiserver uses to access etcd renewed
certificate for the API server to connect to kubelet renewed
certificate embedded in the kubeconfig file for the controller manager to use renewed
certificate for liveness probes to healthcheck etcd renewed
certificate for etcd nodes to communicate with each other renewed
certificate for serving etcd renewed
certificate for the front proxy client renewed
certificate embedded in the kubeconfig file for the scheduler manager to use renewed

Done renewing certificates. You must restart the kube-apiserver, kube-controller-manager, kube-scheduler and etcd, so that they can use the new certificates.
```



### 6.查看证书有效期

```shell
# kubeadm alpha certs check-expiration


[root@k8s-master kubernetes-1.20.6]# kubeadm alpha certs check-expiration
Command "check-expiration" is deprecated, please use the same command under "kubeadm certs"
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Dec 22, 2031 01:48 UTC   9y                                      no      
apiserver                  Dec 22, 2031 01:48 UTC   9y              ca                      no      
apiserver-etcd-client      Dec 22, 2031 01:48 UTC   9y              etcd-ca                 no      
apiserver-kubelet-client   Dec 22, 2031 01:48 UTC   9y              ca                      no      
controller-manager.conf    Dec 22, 2031 01:48 UTC   9y                                      no      
etcd-healthcheck-client    Dec 22, 2031 01:48 UTC   9y              etcd-ca                 no      
etcd-peer                  Dec 22, 2031 01:48 UTC   9y              etcd-ca                 no      
etcd-server                Dec 22, 2031 01:48 UTC   9y              etcd-ca                 no      
front-proxy-client         Dec 22, 2031 01:48 UTC   9y              front-proxy-ca          no      
scheduler.conf             Dec 22, 2031 01:48 UTC   9y                                      no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Dec 22, 2031 01:05 UTC   9y              no      
etcd-ca                 Dec 22, 2031 01:05 UTC   9y              no      
front-proxy-ca          Dec 22, 2031 01:05 UTC   9y              no
```



