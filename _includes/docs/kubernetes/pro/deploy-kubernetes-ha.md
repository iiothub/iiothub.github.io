

* TOC
{:toc}



## 一、安装说明





### 1.高可用架构



![](/images/kubernetes/pro/deploy-kubernetes-ha/k8s-3.png)





## 二、k8s集群部署



### 1. 部署环境

**节点配置**

| 主机名        | IP地址        | 角色   |
| ------------- | ------------- | ------ |
| k8s-master1   | 172.51.216.81 | master |
| k8s-master2   | 172.51.216.82 | master |
| k8s-master3   | 172.51.216.83 | master |
| k8s-node1     | 172.51.216.84 | node   |
| k8s-node2     | 172.51.216.85 | node   |
| k8s-node3     | 172.51.216.86 | node   |
| k8s-node4     | 172.51.216.87 | node   |
| VIP（虚拟ip） | 172.51.216.90 | vip    |

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
hostnamectl set-hostname k8s-master1
hostnamectl set-hostname k8s-master2
hostnamectl set-hostname k8s-master3
hostnamectl set-hostname k8s-node1
hostnamectl set-hostname k8s-node2
hostnamectl set-hostname k8s-node3
hostnamectl set-hostname k8s-node4
```

- **在所有节点添加hosts**

```shell
# 在所有节点添加hosts

cat >> /etc/hosts << EOF
172.51.216.90   master.k8s.io    k8s-vip
172.51.216.81   master1.k8s.io   k8s-master1
172.51.216.82   master2.k8s.io   k8s-master2
172.51.216.83   master3.k8s.io   k8s-master3
172.51.216.84   node1.k8s.io     k8s-node1
172.51.216.85   node2.k8s.io     k8s-node2
172.51.216.86   node3.k8s.io     k8s-node3
172.51.216.87   node3.k8s.io     k8s-node4
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
yum install chrony -y
# 启用
systemctl start chronyd
systemctl enable chronyd
# 设置亚洲时区
timedatectl set-timezone Asia/Shanghai
# 启用NTP同步
timedatectl set-ntp yes
```



### 3.部署keepalived

**所有master节点部署keepalived（在三台master操作）**



#### 3.1.安装

```shell
yum install -y conntrack-tools libseccomp libtool-ltdl


yum install -y keepalived
```



#### 3.2.配置

默认的`keepalived`配置较复杂，这里用更为简明的方式进行配置，另外的两台master配置和上面类似，只需要修改对应的state配置为BACKUP，priority权重值不同即可，配置中的其他字段这里不做说明。

**k8s-master1节点配置**

```shell
cat > /etc/keepalived/keepalived.conf <<EOF 
! Configuration File for keepalived

global_defs {
   router_id k8s
}

vrrp_script check_haproxy {
    script "killall -0 haproxy"
    interval 3
    weight -2
    fall 10
    rise 2
}

vrrp_instance VI_1 {
    state MASTER 
    interface eth0 
    virtual_router_id 51
    priority 250
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass ceb1b3ec013d66163d6ab
    }
    virtual_ipaddress {
        172.51.216.90
    }
    track_script {
        check_haproxy
    }

}
EOF
```



**k8s-master2节点配置**

```shell
cat > /etc/keepalived/keepalived.conf <<EOF 
! Configuration File for keepalived

global_defs {
   router_id k8s
}

vrrp_script check_haproxy {
    script "killall -0 haproxy"
    interval 3
    weight -2
    fall 10
    rise 2
}

vrrp_instance VI_1 {
    state BACKUP 
    interface eth0 
    virtual_router_id 51
    priority 200
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass ceb1b3ec013d66163d6ab
    }
    virtual_ipaddress {
        172.51.216.90
    }
    track_script {
        check_haproxy
    }

}
EOF
```



**k8s-master3节点配置**

```shell
cat > /etc/keepalived/keepalived.conf <<EOF 
! Configuration File for keepalived

global_defs {
   router_id k8s
}

vrrp_script check_haproxy {
    script "killall -0 haproxy"
    interval 3
    weight -2
    fall 10
    rise 2
}

vrrp_instance VI_1 {
    state BACKUP 
    interface eth0 
    virtual_router_id 51
    priority 150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass ceb1b3ec013d66163d6ab
    }
    virtual_ipaddress {
        172.51.216.90
    }
    track_script {
        check_haproxy
    }

}
EOF
```



#### 3.3.启动和检查

3台master都启动

```shell
# 启动keepalived
$ systemctl start keepalived.service

# 设置开机启动
$ systemctl enable keepalived.service

# 查看启动状态
$ systemctl status keepalived.service
```

启动后查看k8s-master1的网卡信息

```shell
ip a s eth0
```

```shell
# k8s-master1  有虚拟IP
[root@localhost ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether fe:fc:fe:bc:83:e7 brd ff:ff:ff:ff:ff:ff
    inet 172.51.216.81/24 brd 172.51.216.255 scope global eth0
       valid_lft forever preferred_lft forever
       
    inet 172.51.216.90/32 scope global eth0
       valid_lft forever preferred_lft forever
       
    inet6 fe80::fcfc:feff:febc:83e7/64 scope link 
       valid_lft forever preferred_lft forever
```



### 4.部署haproxy

**所有master节点部署keepalived（在三台master操作）**



#### 4.1 安装

```shell
# yum install -y haproxy
```



#### 4.2 配置

3台master节点的配置均相同，配置中声明了后端代理的两个master节点服务器，指定了haproxy运行的端口为16443等，因此16443端口为集群的入口

```shell
cat > /etc/haproxy/haproxy.cfg << EOF
#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2
    
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon 
       
    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats
#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------  
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000
#---------------------------------------------------------------------
# kubernetes apiserver frontend which proxys to the backends
#--------------------------------------------------------------------- 
frontend kubernetes-apiserver
    mode                 tcp
    bind                 *:16443
    option               tcplog
    default_backend      kubernetes-apiserver    
#---------------------------------------------------------------------
# round robin balancing between the various backends
#---------------------------------------------------------------------
backend kubernetes-apiserver
    mode        tcp
    balance     roundrobin
    server      master1.k8s.io   172.51.216.81:6443 check
    server      master2.k8s.io   172.51.216.82:6443 check
    server      master3.k8s.io   172.51.216.83:6443 check
#---------------------------------------------------------------------
# collection haproxy statistics message
#---------------------------------------------------------------------
listen stats
    bind                 *:1080
    stats auth           admin:awesomePassword
    stats refresh        5s
    stats realm          HAProxy\ Statistics
    stats uri            /admin?stats
EOF
```



#### 4.3 启动和检查

3台master都启动

```shell
# 设置开机启动
$ systemctl enable haproxy

# 开启haproxy
$ systemctl start haproxy

# 查看启动状态
$ systemctl status haproxy
```

检查端口

```shell
$ netstat -lntup|grep haproxy
```

```shell
[root@localhost ~]# netstat -lntup|grep haproxy
tcp        0      0 0.0.0.0:1080            0.0.0.0:*               LISTEN      94885/haproxy
tcp        0      0 0.0.0.0:16443           0.0.0.0:*               LISTEN      94885/haproxy
udp        0      0 0.0.0.0:49230           0.0.0.0:*                           94884/haproxy
```



### 5. 安装Docker

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



### 6. 安装kubeadm、kubelet和kubectl

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



### 7. 部署Kubernetes Master

**在具有vip的master上操作，这里为k8s-master1。**



#### 7.1.创建kubeadm配置文件

在具有vip的master上操作，这里为k8s-master1

```
$ mkdir /usr/local/kubernetes/manifests -p

$ cd /usr/local/kubernetes/manifests/

$ vi kubeadm-config.yaml

apiServer:
  certSANs:
    - master1.k8s.io
    - master2.k8s.io
    - master3.k8s.io
    - master.k8s.io
    - 172.51.216.90
    - 172.51.216.81
    - 172.51.216.82
    - 172.51.216.83
    - 127.0.0.1
  extraArgs:
    authorization-mode: Node,RBAC
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta1
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: "master.k8s.io:16443"
controllerManager: {}
dns: 
  type: CoreDNS
etcd:
  local:    
    dataDir: /var/lib/etcd
imageRepository: registry.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: v1.20.6
networking: 
  dnsDomain: cluster.local  
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.1.0.0/16
scheduler: {}
```

```shell
# 说明

# vip: master.k8s.io
# 端口16443
controlPlaneEndpoint: "master.k8s.io:16443"

# 版本
kubernetesVersion: v1.20.6
```



#### 7.2 在k8s-master1节点执行

```
$ kubeadm init --config kubeadm-config.yaml
```

```shell
# 实际执行结果

[root@localhost manifests]# kubeadm init --config kubeadm-config.yaml
W1215 10:10:56.843561     614 common.go:77] your configuration file uses a deprecated API spec: "kubeadm.k8s.io/v1beta1". Please use 'kubeadm config migrate --old-config old.yaml --new-config new.yaml', which will write the new, similar spec using a newer API version.
[init] Using Kubernetes version: v1.20.6
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-master1 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local master.k8s.io master1.k8s.io master2.k8s.io master3.k8s.io] and IPs [10.1.0.1 172.51.216.81 172.51.216.90 172.51.216.82 172.51.216.83 127.0.0.1]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-master1 localhost] and IPs [172.51.216.81 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-master1 localhost] and IPs [172.51.216.81 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[endpoint] WARNING: port specified in controlPlaneEndpoint overrides bindPort in the controlplane address
[kubeconfig] Writing "admin.conf" kubeconfig file
[endpoint] WARNING: port specified in controlPlaneEndpoint overrides bindPort in the controlplane address
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[endpoint] WARNING: port specified in controlPlaneEndpoint overrides bindPort in the controlplane address
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[endpoint] WARNING: port specified in controlPlaneEndpoint overrides bindPort in the controlplane address
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
[apiclient] All control plane components are healthy after 73.517804 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.20" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node k8s-master1 as control-plane by adding the labels "node-role.kubernetes.io/master=''" and "node-role.kubernetes.io/control-plane='' (deprecated)"
[mark-control-plane] Marking the node k8s-master1 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: efurkk.sc40m6h2bcxrw0bx
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[endpoint] WARNING: port specified in controlPlaneEndpoint overrides bindPort in the controlplane address
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

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join master.k8s.io:16443 --token efurkk.sc40m6h2bcxrw0bx \
    --discovery-token-ca-cert-hash sha256:f6fff482e44e4235a6cf2ee9fcd4993b5498ec3a11fa0337cbd405e790236a26 \
    --control-plane 

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join master.k8s.io:16443 --token efurkk.sc40m6h2bcxrw0bx \
    --discovery-token-ca-cert-hash sha256:f6fff482e44e4235a6cf2ee9fcd4993b5498ec3a11fa0337cbd405e790236a26
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

# 操作2
  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

# 操作 加入master
  kubeadm join master.k8s.io:16443 --token efurkk.sc40m6h2bcxrw0bx \
    --discovery-token-ca-cert-hash sha256:f6fff482e44e4235a6cf2ee9fcd4993b5498ec3a11fa0337cbd405e790236a26 \
    --control-plane 

Then you can join any number of worker nodes by running the following on each as root:

# 操作 加入node
kubeadm join master.k8s.io:16443 --token efurkk.sc40m6h2bcxrw0bx \
    --discovery-token-ca-cert-hash sha256:f6fff482e44e4235a6cf2ee9fcd4993b5498ec3a11fa0337cbd405e790236a26
```



**k8s-master1执行**

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config


# 执行
[root@localhost manifests]# mkdir -p $HOME/.kube
[root@localhost manifests]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@localhost manifests]# sudo chown $(id -u):$(id -g) $HOME/.kube/config
```



**查看集群状态**

```bash
$ kubectl get cs
$ kubectl get nodes
$ kubectl get pods -n kube-system
```

```shell
[root@localhost manifests]# kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS      MESSAGE                                                                                       ERROR
controller-manager   Unhealthy   Get "http://127.0.0.1:10252/healthz": dial tcp 127.0.0.1:10252: connect: connection refused   
scheduler            Unhealthy   Get "http://127.0.0.1:10251/healthz": dial tcp 127.0.0.1:10251: connect: connection refused   
etcd-0               Healthy     {"health":"true"}                                                                             
[root@localhost manifests]# kubectl get nodes
NAME          STATUS     ROLES                  AGE   VERSION
k8s-master1   NotReady   control-plane,master   13m   v1.20.6
[root@localhost manifests]# kubectl get pods -n kube-system
NAME                                  READY   STATUS    RESTARTS   AGE
coredns-7f89b7bc75-ddr6j              0/1     Pending   0          13m
coredns-7f89b7bc75-kvfrw              0/1     Pending   0          13m
etcd-k8s-master1                      1/1     Running   0          13m
kube-apiserver-k8s-master1            1/1     Running   0          13m
kube-controller-manager-k8s-master1   1/1     Running   0          13m
kube-proxy-cn7wb                      1/1     Running   0          13m
kube-scheduler-k8s-master1            1/1     Running   0          13m
```



**master节点加入集群**

```shell
# master节点加入集群

kubeadm join master.k8s.io:16443 --token efurkk.sc40m6h2bcxrw0bx \
    --discovery-token-ca-cert-hash sha256:f6fff482e44e4235a6cf2ee9fcd4993b5498ec3a11fa0337cbd405e790236a26 \
    --control-plane


#注：token有效期24小时，过去失效，如获取新的使用命令
sudo kubeadm token create --print-join-command
```



**node节点加入集群**

```shell
# node节点加入集群

kubeadm join master.k8s.io:16443 --token efurkk.sc40m6h2bcxrw0bx \
    --discovery-token-ca-cert-hash sha256:f6fff482e44e4235a6cf2ee9fcd4993b5498ec3a11fa0337cbd405e790236a26
    
#注：token有效期24小时，过去失效，如获取新的使用命令
sudo kubeadm token create --print-join-command
```



### 8.master节点加入集群

#### 8.1 复制密钥及相关文件

从k8s-master1复制密钥及相关文件到k8s-master2、k8s-master3



**k8s-master2**

```bash
# ssh root@172.51.216.82 mkdir -p /etc/kubernetes/pki/etcd

# scp /etc/kubernetes/admin.conf root@172.51.216.82:/etc/kubernetes
   
# scp /etc/kubernetes/pki/{ca.*,sa.*,front-proxy-ca.*} root@172.51.216.82:/etc/kubernetes/pki
   
# scp /etc/kubernetes/pki/etcd/ca.* root@172.51.216.82:/etc/kubernetes/pki/etcd
```



**k8s-master3**

```bash
# ssh root@172.51.216.83 mkdir -p /etc/kubernetes/pki/etcd

# scp /etc/kubernetes/admin.conf root@172.51.216.83:/etc/kubernetes
   
# scp /etc/kubernetes/pki/{ca.*,sa.*,front-proxy-ca.*} root@172.51.216.83:/etc/kubernetes/pki
   
# scp /etc/kubernetes/pki/etcd/ca.* root@172.51.216.83:/etc/kubernetes/pki/etcd
```



#### 8.2 master节点加入集群

执行在k8s-master1上init后输出的join命令,需要带上参数`--control-plane`表示把master控制节点加入集群



**在k8s-master2、k8s-master3上分别执行**

```shell
# master节点加入集群

kubeadm join master.k8s.io:16443 --token efurkk.sc40m6h2bcxrw0bx \
    --discovery-token-ca-cert-hash sha256:f6fff482e44e4235a6cf2ee9fcd4993b5498ec3a11fa0337cbd405e790236a26 \
    --control-plane
```

```shell
# 执行结果


[root@localhost ~]# kubeadm join master.k8s.io:16443 --token efurkk.sc40m6h2bcxrw0bx \
>     --discovery-token-ca-cert-hash sha256:f6fff482e44e4235a6cf2ee9fcd4993b5498ec3a11fa0337cbd405e790236a26 \
>     --control-plane
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks before initializing the new control plane instance
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-master3 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local master.k8s.io master1.k8s.io master2.k8s.io master3.k8s.io] and IPs [10.1.0.1 172.51.216.83 172.51.216.90 172.51.216.81 172.51.216.82 127.0.0.1]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-master3 localhost] and IPs [172.51.216.83 127.0.0.1 ::1]
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-master3 localhost] and IPs [172.51.216.83 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Valid certificates and keys now exist in "/etc/kubernetes/pki"
[certs] Using the existing "sa" key
[kubeconfig] Generating kubeconfig files
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[endpoint] WARNING: port specified in controlPlaneEndpoint overrides bindPort in the controlplane address
[kubeconfig] Using existing kubeconfig file: "/etc/kubernetes/admin.conf"
[endpoint] WARNING: port specified in controlPlaneEndpoint overrides bindPort in the controlplane address
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[endpoint] WARNING: port specified in controlPlaneEndpoint overrides bindPort in the controlplane address
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[check-etcd] Checking that the etcd cluster is healthy
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...
[etcd] Announced new etcd member joining to the existing etcd cluster
[etcd] Creating static Pod manifest for "etcd"
[etcd] Waiting for the new etcd member to join the cluster. This can take up to 40s
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[mark-control-plane] Marking the node k8s-master3 as control-plane by adding the labels "node-role.kubernetes.io/master=''" and "node-role.kubernetes.io/control-plane='' (deprecated)"
[mark-control-plane] Marking the node k8s-master3 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]

This node has joined the cluster and a new control plane instance was created:

* Certificate signing request was sent to apiserver and approval was received.
* The Kubelet was informed of the new secure connection details.
* Control plane (master) label and taint were applied to the new node.
* The Kubernetes control plane instances scaled up.
* A new etcd member was added to the local/stacked etcd cluster.

To start administering your cluster from this node, you need to run the following as a regular user:

	mkdir -p $HOME/.kube
	sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
	sudo chown $(id -u):$(id -g) $HOME/.kube/config

Run 'kubectl get nodes' to see this node join the cluster.

```



**按照提示配置环境变量，使用kubectl工具：**

**k8s-master2、k8s-master2分别执行**

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config


# 执行
[root@localhost ~]# mkdir -p $HOME/.kube
[root@localhost ~]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@localhost ~]# sudo chown $(id -u):$(id -g) $HOME/.kube/config
```



**查看集群状态**

```bash
$ kubectl get cs
$ kubectl get nodes
$ kubectl get pods -n kube-system
$ kubectl get pods --all-namespaces
```

```shell
[root@k8s-master1 ~]# kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS      MESSAGE                                                                                       ERROR
scheduler            Unhealthy   Get "http://127.0.0.1:10251/healthz": dial tcp 127.0.0.1:10251: connect: connection refused   
controller-manager   Unhealthy   Get "http://127.0.0.1:10252/healthz": dial tcp 127.0.0.1:10252: connect: connection refused   
etcd-0               Healthy     {"health":"true"}                                                                             

[root@k8s-master1 ~]# kubectl get nodes
NAME          STATUS     ROLES                  AGE   VERSION
k8s-master1   NotReady   control-plane,master   66m   v1.20.6
k8s-master2   NotReady   control-plane,master   2m    v1.20.6
k8s-master3   NotReady   control-plane,master   42m   v1.20.6


[root@k8s-master1 ~]# kubectl get pods -n kube-system
NAME                                  READY   STATUS    RESTARTS   AGE
coredns-7f89b7bc75-ddr6j              0/1     Pending   0          66m
coredns-7f89b7bc75-kvfrw              0/1     Pending   0          66m
etcd-k8s-master1                      1/1     Running   0          66m
etcd-k8s-master2                      1/1     Running   0          2m21s
etcd-k8s-master3                      1/1     Running   0          42m
kube-apiserver-k8s-master1            1/1     Running   0          66m
kube-apiserver-k8s-master2            1/1     Running   0          2m21s
kube-apiserver-k8s-master3            1/1     Running   0          42m
kube-controller-manager-k8s-master1   1/1     Running   1          66m
kube-controller-manager-k8s-master2   1/1     Running   0          2m21s
kube-controller-manager-k8s-master3   1/1     Running   0          42m
kube-proxy-cn7wb                      1/1     Running   0          66m
kube-proxy-jckcs                      1/1     Running   0          42m
kube-proxy-p6rcq                      1/1     Running   0          2m22s
kube-scheduler-k8s-master1            1/1     Running   1          66m
kube-scheduler-k8s-master2            1/1     Running   0          2m21s
kube-scheduler-k8s-master3            1/1     Running   0          42m


[root@k8s-master1 ~]# kubectl get pods -n kube-system -owide
NAME                                  READY   STATUS    RESTARTS   AGE     IP              NODE          NOMINATED NODE   READINESS GATES
coredns-7f89b7bc75-ddr6j              0/1     Pending   0          66m     <none>          <none>        <none>           <none>
coredns-7f89b7bc75-kvfrw              0/1     Pending   0          66m     <none>          <none>        <none>           <none>
etcd-k8s-master1                      1/1     Running   0          66m     172.51.216.81   k8s-master1   <none>           <none>
etcd-k8s-master2                      1/1     Running   0          2m35s   172.51.216.82   k8s-master2   <none>           <none>
etcd-k8s-master3                      1/1     Running   0          43m     172.51.216.83   k8s-master3   <none>           <none>
kube-apiserver-k8s-master1            1/1     Running   0          66m     172.51.216.81   k8s-master1   <none>           <none>
kube-apiserver-k8s-master2            1/1     Running   0          2m35s   172.51.216.82   k8s-master2   <none>           <none>
kube-apiserver-k8s-master3            1/1     Running   0          43m     172.51.216.83   k8s-master3   <none>           <none>
kube-controller-manager-k8s-master1   1/1     Running   1          66m     172.51.216.81   k8s-master1   <none>           <none>
kube-controller-manager-k8s-master2   1/1     Running   0          2m35s   172.51.216.82   k8s-master2   <none>           <none>
kube-controller-manager-k8s-master3   1/1     Running   0          43m     172.51.216.83   k8s-master3   <none>           <none>
kube-proxy-cn7wb                      1/1     Running   0          66m     172.51.216.81   k8s-master1   <none>           <none>
kube-proxy-jckcs                      1/1     Running   0          43m     172.51.216.83   k8s-master3   <none>           <none>
kube-proxy-p6rcq                      1/1     Running   0          2m36s   172.51.216.82   k8s-master2   <none>           <none>
kube-scheduler-k8s-master1            1/1     Running   1          66m     172.51.216.81   k8s-master1   <none>           <none>
kube-scheduler-k8s-master2            1/1     Running   0          2m35s   172.51.216.82   k8s-master2   <none>           <none>
kube-scheduler-k8s-master3            1/1     Running   0          43m     172.51.216.83   k8s-master3   <none>           <none>
```



### 9. 加入Kubernetes Node

此命令在3个子节点（Node）执行。

向集群添加新节点，执行在kubeadm init输出的kubeadm join命令：
**注意：如下命令要复制你根据kubeadm init生成的命令**

```shell
# node节点加入集群

kubeadm join master.k8s.io:16443 --token efurkk.sc40m6h2bcxrw0bx \
    --discovery-token-ca-cert-hash sha256:f6fff482e44e4235a6cf2ee9fcd4993b5498ec3a11fa0337cbd405e790236a26
   
   
# 默认token有效期为24小时，当过期之后，该token就不可用了。这时就需要重新创建token，操作如下：
kubeadm token create --print-join-command
```

```shell
# 执行结果

[root@localhost ~]# kubeadm join master.k8s.io:16443 --token efurkk.sc40m6h2bcxrw0bx \
>     --discovery-token-ca-cert-hash sha256:f6fff482e44e4235a6cf2ee9fcd4993b5498ec3a11fa0337cbd405e790236a26
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```



查看集群状态

```shell
# 查看集群状态
kubectl get nodes

[root@k8s-master1 ~]# kubectl get nodes
NAME          STATUS     ROLES                  AGE     VERSION
k8s-master1   NotReady   control-plane,master   20m     v1.20.6
k8s-master2   NotReady   control-plane,master   2m36s   v1.20.6
k8s-master3   NotReady   control-plane,master   6m53s   v1.20.6
k8s-node1     NotReady   <none>                 41s     v1.20.6
k8s-node2     NotReady   <none>                 36s     v1.20.6
k8s-node3     NotReady   <none>                 30s     v1.20.6
k8s-node4     NotReady   <none>                 24s     v1.20.6


# 由于k8s集群网络不同，status状态为NotReady,需使用calico网络组件是k8s网络畅通 
```



### 10. 部署CNI网络插件

#### 10.1. Calico

安装calico

```shell
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

运行完成之后，等待calico pod 变成Running

```shell
kubectl get pod --all-namespaces


[root@k8s-master1 ~]# kubectl get pod --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-558995777d-l8lpr   1/1     Running   0          3m58s
kube-system   calico-node-6l5w2                          1/1     Running   0          3m58s
kube-system   calico-node-g68dm                          1/1     Running   0          3m58s
kube-system   calico-node-qt82k                          1/1     Running   0          3m58s
kube-system   calico-node-rvtbg                          1/1     Running   0          3m58s
kube-system   calico-node-tn2xj                          1/1     Running   0          3m58s
kube-system   calico-node-wnjhd                          1/1     Running   0          3m58s
kube-system   calico-node-wt5cb                          1/1     Running   0          3m58s
kube-system   coredns-7f89b7bc75-h2htb                   1/1     Running   0          24m
kube-system   coredns-7f89b7bc75-k5phr                   1/1     Running   0          24m
kube-system   etcd-k8s-master1                           1/1     Running   0          25m
kube-system   etcd-k8s-master2                           1/1     Running   0          7m12s
kube-system   etcd-k8s-master3                           1/1     Running   0          11m
kube-system   kube-apiserver-k8s-master1                 1/1     Running   0          25m
kube-system   kube-apiserver-k8s-master2                 1/1     Running   0          7m13s
kube-system   kube-apiserver-k8s-master3                 1/1     Running   0          11m
kube-system   kube-controller-manager-k8s-master1        1/1     Running   1          25m
kube-system   kube-controller-manager-k8s-master2        1/1     Running   0          7m13s
kube-system   kube-controller-manager-k8s-master3        1/1     Running   0          11m
kube-system   kube-proxy-5pjpd                           1/1     Running   0          5m18s
kube-system   kube-proxy-86vds                           1/1     Running   0          5m1s
kube-system   kube-proxy-cv28h                           1/1     Running   0          24m
kube-system   kube-proxy-mmbj6                           1/1     Running   0          11m
kube-system   kube-proxy-s9lhm                           1/1     Running   0          5m7s
kube-system   kube-proxy-tvt2z                           1/1     Running   0          7m13s
kube-system   kube-proxy-w7j6l                           1/1     Running   0          5m13s
kube-system   kube-scheduler-k8s-master1                 1/1     Running   1          25m
kube-system   kube-scheduler-k8s-master2                 1/1     Running   0          7m12s
kube-system   kube-scheduler-k8s-master3                 1/1     Running   0          11m
```



测试

```shell
kubectl get pods -n kube-system
kubectl get nodes
kubectl get pods --all-namespaces


[root@k8s-master1 manifests]# kubectl get nodes
NAME          STATUS   ROLES                  AGE     VERSION
k8s-master1   Ready    control-plane,master   24m     v1.20.6
k8s-master2   Ready    control-plane,master   6m41s   v1.20.6
k8s-master3   Ready    control-plane,master   10m     v1.20.6
k8s-node1     Ready    <none>                 4m46s   v1.20.6
k8s-node2     Ready    <none>                 4m41s   v1.20.6
k8s-node3     Ready    <none>                 4m35s   v1.20.6
k8s-node4     Ready    <none>                 4m29s   v1.20.6
```



### 11. 测试KUBERNETES集群

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

[root@k8s-master1 ~]# kubectl get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-6799fc88d8-d5v72   1/1     Running   0          66s

NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.1.0.1      <none>        443/TCP        85m
service/nginx        NodePort    10.1.61.243   <none>        80:31879/TCP   57s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           66s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-6799fc88d8   1         1         1       66s



# 访问地址：http://NodeIP:Port
http://172.51.216.81:31879
```



### 12.高可用测试

**1.通过VIP（172.51.216.90）进行终端操作；**

**2.关闭服务器k8s-master1，VIP切换到k8s-master2；**

**3.挂掉两台master主机，kubectl不能正常使用；**

**4.三台master都可以kubectl操作；**





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



[root@localhost ~]# kubectl get all -n kubernetes-dashboard
NAME                                             READY   STATUS    RESTARTS   AGE
pod/dashboard-metrics-scraper-79c5968bdc-lrcnl   1/1     Running   0          112s
pod/kubernetes-dashboard-658485d5c7-z4nf7        1/1     Running   0          112s

NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/dashboard-metrics-scraper   ClusterIP   10.107.74.214    <none>        8000/TCP   112s
service/kubernetes-dashboard        ClusterIP   10.108.176.179   <none>        443/TCP    112s

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dashboard-metrics-scraper   1/1     1            1           112s
deployment.apps/kubernetes-dashboard        1/1     1            1           112s

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/dashboard-metrics-scraper-79c5968bdc   1         1         1       112s
replicaset.apps/kubernetes-dashboard-658485d5c7        1         1         1       112s

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
[root@k8s-master3 ~]# kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetesdashboard get secret | grep admin-user | awk '{print $1}')
No resources found in kubernetesdashboard namespace.
Name:         admin-user-token-jnn2c
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: 050b4445-99d3-4ab8-8899-2886c0da6d29

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  20 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IkotVkw3dkRGN1ZsNE04WnhhOVpaUENsMktjUDM2ZWhrRi1ndDZyN3g5c2MifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLWpubjJjIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwNTBiNDQ0NS05OWQzLTRhYjgtODg5OS0yODg2YzBkYTZkMjkiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.Hqqroo0-KP06ttqUeDGyxQHL8OzTiye5bHte1sNu6cKAfxht9033NQ_mHdnGWBs9L9mC32LHAQJMHSgfmU8SziOu387i6zCeMTeylQq2NSnmpZgjcvrCXKUlYfAdVWoe38QG7mJptiQlS8CEFZoDfm4iWVYt73bOQZSOZp8tLwnjhwy7SnHUgjl6Wtdf0XRX_1JZ9gd7p-c1xhMaXmRyRf-EcQVrtV9CrwepTVPiWwlvVNHxp_w8y5G0JRZ8y80guhjU3Nzn5xchLLrl9UtqAGpdcYSc8lHzcPuxqErsuTSToNYBHeL6rhOD9ZVzBZ1x1pbWtwJMY2IP54eX5zqmIw



# admin-user-token-xxxx账号密码
```

访问令牌

```shell
# Name:         admin-user-token-w9tl6

token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IkotVkw3dkRGN1ZsNE04WnhhOVpaUENsMktjUDM2ZWhrRi1ndDZyN3g5c2MifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLWpubjJjIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwNTBiNDQ0NS05OWQzLTRhYjgtODg5OS0yODg2YzBkYTZkMjkiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.Hqqroo0-KP06ttqUeDGyxQHL8OzTiye5bHte1sNu6cKAfxht9033NQ_mHdnGWBs9L9mC32LHAQJMHSgfmU8SziOu387i6zCeMTeylQq2NSnmpZgjcvrCXKUlYfAdVWoe38QG7mJptiQlS8CEFZoDfm4iWVYt73bOQZSOZp8tLwnjhwy7SnHUgjl6Wtdf0XRX_1JZ9gd7p-c1xhMaXmRyRf-EcQVrtV9CrwepTVPiWwlvVNHxp_w8y5G0JRZ8y80guhjU3Nzn5xchLLrl9UtqAGpdcYSc8lHzcPuxqErsuTSToNYBHeL6rhOD9ZVzBZ1x1pbWtwJMY2IP54eX5zqmIw
```



### 4. 浏览器访问

浏览器登陆任意node节点IP+30443,填入获取的token
https://172.51.216.81:30443

火狐浏览器、chrome、edge可以直接访问

![登录](/images/kubernetes/pro/deploy-kubernetes-ha/dashboard-login.png)

Dashboard

![dashboard首页](/images/kubernetes/pro/deploy-kubernetes-ha/dashboard-first.png)



在谷歌浏览器（Chrome）启动文件中加入启动参数，用于解决无法访问Dashboard的问题，参考图：

```shell
--test-type --ignore-certificate-errors
```

![chrome设置](/images/kubernetes/pro/deploy-kubernetes-ha/chrome.png)





## 四、k8s证书有效期



**使用kubeadm搭建K8s集群，默认自动生成的证书有效期只有 1 年，因此需要每年手动更新一次证书，这种形式显然对实际生产环境来说很不友好；因此下面教给大家修改这个过期时间的终极方法。**



https://blog.csdn.net/m0_37814112/article/details/115899660



### 1.查看证书有效期

```shell
# kubeadm alpha certs check-expiration


[root@k8s-master1 ~]# kubeadm alpha certs check-expiration
Command "check-expiration" is deprecated, please use the same command under "kubeadm certs"
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Dec 16, 2022 05:51 UTC   364d                                    no      
apiserver                  Dec 16, 2022 05:51 UTC   364d            ca                      no      
apiserver-etcd-client      Dec 16, 2022 05:51 UTC   364d            etcd-ca                 no      
apiserver-kubelet-client   Dec 16, 2022 05:51 UTC   364d            ca                      no      
controller-manager.conf    Dec 16, 2022 05:51 UTC   364d                                    no      
etcd-healthcheck-client    Dec 16, 2022 05:51 UTC   364d            etcd-ca                 no      
etcd-peer                  Dec 16, 2022 05:51 UTC   364d            etcd-ca                 no      
etcd-server                Dec 16, 2022 05:51 UTC   364d            etcd-ca                 no      
front-proxy-client         Dec 16, 2022 05:51 UTC   364d            front-proxy-ca          no      
scheduler.conf             Dec 16, 2022 05:51 UTC   364d                                    no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Dec 14, 2031 05:51 UTC   9y              no      
etcd-ca                 Dec 14, 2031 05:51 UTC   9y              no      
front-proxy-ca          Dec 14, 2031 05:51 UTC   9y              no
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

![](/images/kubernetes/pro/deploy-kubernetes-ha/k8s-4.png)



```shell
# 编译报错

yum install rsync -y

# 从新编译报错
[root@k8s-master1 kubernetes-1.20.6]# make WHAT=cmd/kubeadm GOFLAGS=-v
./hack/run-in-gopath.sh: line 34: _output/bin/prerelease-lifecycle-gen: Permission denied
make[1]: *** [gen_prerelease_lifecycle] Error 1
make: *** [generated_files] Error 2

# 删除解压文件

# 从新解压修改编译通过
```



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


[root@k8s-master1 kubernetes-1.20.6]# kubeadm alpha certs check-expiration
Command "check-expiration" is deprecated, please use the same command under "kubeadm certs"
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Dec 14, 2031 06:44 UTC   9y                                      no      
apiserver                  Dec 14, 2031 06:44 UTC   9y              ca                      no      
apiserver-etcd-client      Dec 14, 2031 06:44 UTC   9y              etcd-ca                 no      
apiserver-kubelet-client   Dec 14, 2031 06:44 UTC   9y              ca                      no      
controller-manager.conf    Dec 14, 2031 06:44 UTC   9y                                      no      
etcd-healthcheck-client    Dec 14, 2031 06:44 UTC   9y              etcd-ca                 no      
etcd-peer                  Dec 14, 2031 06:44 UTC   9y              etcd-ca                 no      
etcd-server                Dec 14, 2031 06:44 UTC   9y              etcd-ca                 no      
front-proxy-client         Dec 14, 2031 06:44 UTC   9y              front-proxy-ca          no      
scheduler.conf             Dec 14, 2031 06:44 UTC   9y                                      no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Dec 14, 2031 05:51 UTC   9y              no      
etcd-ca                 Dec 14, 2031 05:51 UTC   9y              no      
front-proxy-ca          Dec 14, 2031 05:51 UTC   9y              no
```



### 7.其余 Master 节点证书更新

**其它master节点可按如下操作执行。**

```shell
# 1、备份另外两个matser节点证书文件及kubeadm工具
cp -arp /etc/kubernetes /etc/kubernetes_`date +%F`
mv /usr/bin/kubeadm /usr/bin/kubeadm_`date +%F`


# 2、将第一个master更新好的kubeadm工具拷贝到另外两个master节点的/usr/bin目录下
scp /usr/bin/kubeadm root@192.168.1.213:/usr/bin
scp /usr/bin/kubeadm root@192.168.1.214:/usr/bin


# 3、证书更新
kubeadm config view > /root/kubeadm-config.yaml
kubeadm alpha certs renew all --config=/root/kubeadm-config.yaml


# 4、查看证书有效期
kubeadm alpha certs check-expiration
```

```shell
# 1、备份另外两个matser节点证书文件及kubeadm工具
# k8s-master2、k8s-master3分别执行
cp -arp /etc/kubernetes /etc/kubernetes_`date +%F`
mv /usr/bin/kubeadm /usr/bin/kubeadm_`date +%F`



# 2、将第一个master更新好的kubeadm工具拷贝到另外两个master节点的/usr/bin目录下
# k8s-master2
scp /usr/bin/kubeadm root@172.51.216.82:/usr/bin
# k8s-master3
scp /usr/bin/kubeadm root@172.51.216.83:/usr/bin



# 3、证书更新
# k8s-master2、k8s-master3分别执行
kubeadm config view > /root/kubeadm-config.yaml
kubeadm alpha certs renew all --config=/root/kubeadm-config.yaml


[root@localhost ~]# kubeadm config view > /root/kubeadm-config.yaml
Command "view" is deprecated, This command is deprecated and will be removed in a future release, please use 'kubectl get cm -o yaml -n kube-system kubeadm-config' to get the kubeadm config directly.
[root@localhost ~]# kubeadm alpha certs renew all --config=/root/kubeadm-config.yaml
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


# 4、查看证书有效期
kubeadm alpha certs check-expiration

[root@k8s-master3 ~]# kubeadm alpha certs check-expiration
Command "check-expiration" is deprecated, please use the same command under "kubeadm certs"
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Dec 14, 2031 06:48 UTC   9y                                      no      
apiserver                  Dec 14, 2031 06:48 UTC   9y              ca                      no      
apiserver-etcd-client      Dec 14, 2031 06:48 UTC   9y              etcd-ca                 no      
apiserver-kubelet-client   Dec 14, 2031 06:48 UTC   9y              ca                      no      
controller-manager.conf    Dec 14, 2031 06:48 UTC   9y                                      no      
etcd-healthcheck-client    Dec 14, 2031 06:48 UTC   9y              etcd-ca                 no      
etcd-peer                  Dec 14, 2031 06:48 UTC   9y              etcd-ca                 no      
etcd-server                Dec 14, 2031 06:48 UTC   9y              etcd-ca                 no      
front-proxy-client         Dec 14, 2031 06:48 UTC   9y              front-proxy-ca          no      
scheduler.conf             Dec 14, 2031 06:48 UTC   9y                                      no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Dec 14, 2031 05:51 UTC   9y              no      
etcd-ca                 Dec 14, 2031 05:51 UTC   9y              no      
front-proxy-ca          Dec 14, 2031 05:51 UTC   9y              no
```



