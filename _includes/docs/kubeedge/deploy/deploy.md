* TOC
{:toc}



## 一、概述



### 1.安装说明

```shell
https://kubeedge.io/zh/

https://github.com/kubeedge/kubeedge

https://github.com/kubeedge/kubeedge/releases/tag/v1.13.4
```



![](/images/kubeedge/deploy/deploy/deploy-1.png)



### 2.安装keadm

![](/images/kubeedge/deploy/deploy/deploy-2.png)

cloudcore、edgecore同样操作

```shell
wget https://github.com/kubeedge/kubeedge/releases/download/v1.13.4/keadm-v1.13.4-linux-amd64.tar.gz

tar -zxvf keadm-v1.13.4-linux-amd64.tar.gz

mv keadm-v1.13.4-linux-amd64/keadm/keadm /usr/local/bin/

keadm version
keadm
```



### 3.安装cloudcore

```shell
keadm init --advertise-address=192.168.202.201 --set iptablesManager.mode="external" --profile version=v1.13.4

kubectl get ns
kubectl get pods -n kubeedge
kubectl get svc -n kubeedge


# 修改SVC
[root@k8s-master01 ~]# kubectl edit svc cloudcore -n kubeedge
service/cloudcore edited

# 修改位置：
  selector:
    k8s-app: kubeedge
    kubeedge: cloudcore
  sessionAffinity: None
  type: LoadBalancer   此处由clusterIP修改为NodePort
```



### 4.安装edgecore

```shell
1.安装keadm
2.安装docker
3.加入Edge节点
```

```shell
TOKEN=bec7e346f2b62b87bb01cb8082111b05759aa16ba298d7b97442581c3bccee52.eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2NzMzMjI1Mjd9. BzQiyUFBp1dax9NC7BOssRe4PgjwOE24w2jE7S8Hp-0

SERVER=192.168.202.201:10000


keadm join --token=$TOKEN --cloudcore-ipport=$SERVER --kubeedge-version=v1.13.4 --runtimetype=docker --edgenode-name=edge-1

kubectl get nodes
systemctl status edgecore
```



### 5.启用 kubectl logs 功能

#### 5.1.官方文档

```shell
https://release-1-13.docs.kubeedge.io/docs/setup/install-with-keadm/
```

![](/images/kubeedge/deploy/deploy/deploy-14.png)



#### 5.2.设置iptables

1.获取tunnelport

```shell
# 获取 configmap 
[root@k8s-master kubeedge]# kubectl get cm tunnelport -nkubeedge -oyaml
apiVersion: v1
kind: ConfigMap
metadata:
  annotations:
    tunnelportrecord.kubeedge.io: '{"ipTunnelPort":{"192.168.202.202":10351},"port":{"10351":true}}'
  creationTimestamp: "2023-12-17T09:00:02Z"
  name: tunnelport
  namespace: kubeedge
  resourceVersion: "4756"
  uid: 80afe5e2-94e0-4d72-b0e6-32aab67535e8
```

![](/images/kubeedge/deploy/deploy/deploy-15.png)



2.设置iptables

```shell
# 清除 iptables
[root@k8s-master kubeedge]# iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X


# 设置iptables
[root@k8s-master kubeedge]# iptables -t nat -A OUTPUT -p tcp --dport 10351 -j DNAT --to 192.168.202.202:10003
```



#### 5.3.修改edgecore配置

```shell
[root@edge-1 kubeedge]# vim /etc/kubeedge/config/edgecore.yaml

  edgeStream:
  
    enable: true
    
    handshakeTimeout: 30
    readDeadline: 15
    server: 192.168.202.201:30480
    tlsTunnelCAFile: /etc/kubeedge/ca/rootCA.crt
    tlsTunnelCertFile: /etc/kubeedge/certs/server.crt
    tlsTunnelPrivateKeyFile: /etc/kubeedge/certs/server.key
    writeDeadline: 15


# systemctl restart edgecore
# systemctl status edgecore
```

![](/images/kubeedge/deploy/deploy/deploy-16.png)



```shell
# 查看日志

# kubectl logs nginx-deployment-6fb75cb954-2cw59
# kubectl exec -it nginx-deployment-6fb75cb954-2cw59 -- /bin/bash 
```





## 二、cloudcore安装部署



### 1.安装K8S集群

**安装准备：**

- 安装k8s集群
- 安装metrics-server



**软件版本：**

- Kubernetes：v1.23.12
- KubeEdge：v1.13.4

![](/images/kubeedge/deploy/deploy/deploy-4.png)



### 2.安装keadm

**master节点安装keadm**

```shell
# 1.下载安装文件
wget https://github.com/kubeedge/kubeedge/releases/download/v1.13.4/keadm-v1.13.4-linux-amd64.tar.gz

# 2.解压
tar -zxvf keadm-v1.13.4-linux-amd64.tar.gz

# 3.移动文件
mv keadm-v1.13.4-linux-amd64/keadm/keadm /usr/local/bin/

keadm
keadm version
```



```shell
# 解压
[root@k8s-master kubeedge]# tar -xf keadm-v1.13.4-linux-amd64.tar.gz

# 移动文件
[root@k8s-master kubeedge]# mv keadm-v1.13.4-linux-amd64/keadm/keadm /usr/local/bin/

[root@k8s-master kubeedge]# keadm version
version: version.Info{Major:"1", Minor:"13", GitVersion:"v1.13.4", GitCommit:"043bd224ff34c44c10990e1ecbe50dd33f80b52b", GitTreeState:"clean", BuildDate:"2023-11-30T12:53:38Z", GoVersion:"go1.17.13", Compiler:"gc", Platform:"linux/amd64"}

[root@k8s-master kubeedge]# keadm

    +----------------------------------------------------------+
    | KEADM                                                    |
    | Easily bootstrap a KubeEdge cluster                      |
    |                                                          |
    | Please give us feedback at:                              |
    | https://github.com/kubeedge/kubeedge/issues              |
    +----------------------------------------------------------+

    Create a cluster with cloud node
    (which controls the edge node cluster), and edge nodes
    (where native containerized application, in the form of
    pods and deployments run), connects to devices.
......
```



### 3.安装cloudcore

```shell
keadm init --advertise-address=192.168.202.201 --set iptablesManager.mode="external" --profile version=v1.13.4

kubectl get ns
kubectl get pods -n kubeedge
kubectl get svc -n kubeedge
```



```shell
#192.168.202.201为 master 的IP

[root@k8s-master kubeedge]# keadm init --advertise-address=192.168.202.201 --set iptablesManager.mode="external" --profile version=v1.13.4
Kubernetes version verification passed, KubeEdge installation will start...
CLOUDCORE started
=========CHART DETAILS=======
NAME: cloudcore
LAST DEPLOYED: Sun Jan  7 12:34:45 2024
NAMESPACE: kubeedge
STATUS: deployed
REVISION: 1


[root@k8s-master kubeedge]# kubectl get ns
NAME              STATUS   AGE
default           Active   2d2h
kube-node-lease   Active   2d2h
kube-public       Active   2d2h
kube-system       Active   2d2h
kubeedge          Active   2m39s


[root@k8s-master kubeedge]# kubectl get all -n kubeedge
NAME                               READY   STATUS    RESTARTS   AGE
pod/cloud-iptables-manager-592m5   1/1     Running   0          66s
pod/cloud-iptables-manager-pg4pl   1/1     Running   0          66s
pod/cloudcore-5959c5986f-8hsc4     1/1     Running   0          66s

NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                                             AGE
service/cloudcore   ClusterIP   10.110.71.216   <none>        10000/TCP,10001/TCP,10002/TCP,10003/TCP,10004/TCP   66s

NAME                                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/cloud-iptables-manager   2         2         2       2            2           <none>          66s

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cloudcore   1/1     1            1           66s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/cloudcore-5959c5986f   1         1         1       66s


#生成的token
[root@k8s-master kubeedge]# keadm gettoken
f06b43005c8eb23508c4c95a756606f7ed4db588b8fd3461b078947a5f57a487.eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MDQ2ODg0OTN9.vSJ-YXHevJ8QX3qb_8vTzjpI1HCk2Wkai0tUmFkJQSY
```

![](/images/kubeedge/deploy/deploy/deploy-5.png)



### 4.修改SVC

```shell
# 修改cloudcore的svc类型为NodePort


[root@k8s-master kubeedge]# kubectl edit svc cloudcore -n kubeedge

# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
kind: Service
.....
  sessionAffinity: None
  
  type: NodePort
  
status:
  loadBalancer: {}
  
  
[root@k8s-master kubeedge]# kubectl get svc -n kubeedge
NAME        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                           AGE
cloudcore   NodePort   10.110.71.216   <none>        10000:30976/TCP,10001:31372/TCP,10002:31922/TCP,10003:30163/TCP,10004:31927/TCP   5m23s
```

![](/images/kubeedge/deploy/deploy/deploy-6.png)

![](/images/kubeedge/deploy/deploy/deploy-7.png)



### 5.打标签

```shell
# 因为边缘计算的硬件条件都不好，这里我们需要打上标签，让一些应用不扩展到edge节点上去

kubectl get daemonset -n kube-system |grep -v NAME |awk '{print $1}' | xargs -n 1 kubectl patch daemonset -n kube-system --type='json' -p='[{"op": "replace","path": "/spec/template/spec/affinity","value":{"nodeAffinity":{"requiredDuringSchedulingIgnoredDuringExecution":{"nodeSelectorTerms":[{"matchExpressions":[{"key":"node-role.kubernetes.io/edge","operator":"DoesNotExist"}]}]}}}}]'


# 暂时不用
kubectl get daemonset -n metallb-system |grep -v NAME |awk '{print $1}' | xargs -n 1 kubectl patch daemonset -n metallb-system --type='json' -p='[{"op": "replace","path": "/spec/template/spec/affinity","value":{"nodeAffinity":{"requiredDuringSchedulingIgnoredDuringExecution":{"nodeSelectorTerms":[{"matchExpressions":[{"key":"node-role.kubernetes.io/edge","operator":"DoesNotExist"}]}]}}}}]'
```



```shell
[root@k8s-master kubeedge]# kubectl get daemonset -n kube-system |grep -v NAME |awk '{print $1}' | xargs -n 1 kubectl patch daemonset -n kube-system --type='json' -p='[{"op": "replace","path": "/spec/template/spec/affinity","value":{"nodeAffinity":{"requiredDuringSchedulingIgnoredDuringExecution":{"nodeSelectorTerms":[{"matchExpressions":[{"key":"node-role.kubernetes.io/edge","operator":"DoesNotExist"}]}]}}}}]'
daemonset.apps/calico-node patched
daemonset.apps/kube-proxy patched
```



### 6.查看端口

```shell
# 查看cloudcore端口
netstat -nplt


[root@k8s-node2 ~]# netstat -nplt
```

![](/images/kubeedge/deploy/deploy/deploy-8.png)



## 三、edgecore安装部署



### 1. 安装准备

```shell
#克隆机器

cd /etc/sysconfig/network-scripts

vim ifcfg-ens33 

#关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

hostnamectl set-hostname edge-1
```



### 2.安装keadm

```shell
# 解压
[root@edge-1 kubeedge]# tar -xf keadm-v1.13.4-linux-amd64.tar.gz


# 移动文件
[root@edge-1 kubeedge]# mv keadm-v1.13.4-linux-amd64/keadm/keadm /usr/local/bin/


[root@edge-1 kubeedge]# keadm version
version: version.Info{Major:"1", Minor:"13", GitVersion:"v1.13.4", GitCommit:"043bd224ff34c44c10990e1ecbe50dd33f80b52b", GitTreeState:"clean", BuildDate:"2023-11-30T12:53:38Z", GoVersion:"go1.17.13", Compiler:"gc", Platform:"linux/amd64"}
```



### 3.安装Docker

安装版本19.03.*

```shell
$ wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo

$ yum -y install docker-ce

$ systemctl enable docker && systemctl start docker

$ docker --version
```

- **添加阿里云加速镜像**

```shell
# 添加阿里云加速镜像

cat > /etc/docker/daemon.json << EOF
{
   "registry-mirrors": ["https://gcctk8ld.mirror.aliyuncs.com"],
   "exec-opts": ["native.cgroupdriver=cgroupfs"]
} 
EOF
```

- **重启docker**

```shell
#重启docker
systemctl restart docker
```



### 4.加入Edge节点

```shell
# 在master节点上获取token

[root@k8s-master kubeedge]# keadm gettoken
f06b43005c8eb23508c4c95a756606f7ed4db588b8fd3461b078947a5f57a487.eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MDQ2ODg0OTN9.vSJ-YXHevJ8QX3qb_8vTzjpI1HCk2Wkai0tUmFkJQSY


[root@k8s-master kubeedge]# kubectl get svc -n kubeedge
NAME        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                           AGE
cloudcore   NodePort   10.110.71.216   <none>        10000:30976/TCP,10001:31372/TCP,10002:31922/TCP,10003:30163/TCP,10004:31927/TCP   22m


10000:30976
10001:31372
10002:31922
10003:30163
10004:31927
```



```shell
# 加入edge节点

[root@edge-1 kubeedge]# TOKEN=f06b43005c8eb23508c4c95a756606f7ed4db588b8fd3461b078947a5f57a487.eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MDQ2ODg0OTN9.vSJ-YXHevJ8QX3qb_8vTzjpI1HCk2Wkai0tUmFkJQSY
[root@edge-1 kubeedge]# echo $TOKEN
58af829fff435f1363b8084b475f44c90d54f959af3f5837032c4ce6095939b3.eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MDQ2ODA5NDJ9.0Xcx0Bxwdu44XSZ3YTSY1GTAJeEs74Dlk5pJGJmiVco


[root@edge-1 kubeedge]# SERVER=192.168.202.201:30976
[root@edge-1 kubeedge]# echo $SERVER
192.168.202.201:30976



# keadm join --token=$TOKEN --cloudcore-ipport=$SERVER --kubeedge-version=v1.13.4 --runtimetype=docker --edgenode-name=edge-1

[root@edge-1 kubeedge]# keadm join --token=$TOKEN --cloudcore-ipport=$SERVER --kubeedge-version=v1.13.4 --runtimetype=docker --edgenode-name=edge-1
I0107 21:00:16.935240    1919 command.go:845] 1. Check KubeEdge edgecore process status
I0107 21:00:16.951529    1919 command.go:845] 2. Check if the management directory is clean
I0107 21:00:16.951652    1919 join.go:107] 3. Create the necessary directories
I0107 21:00:16.953321    1919 join.go:184] 4. Pull Images
Pulling kubeedge/installation-package:v1.13.4 ...
Successfully pulled kubeedge/installation-package:v1.13.4
Pulling eclipse-mosquitto:1.6.15 ...
Successfully pulled eclipse-mosquitto:1.6.15
Pulling kubeedge/pause:3.6 ...
Successfully pulled kubeedge/pause:3.6
I0107 21:02:10.245112    1919 join.go:184] 5. Copy resources from the image to the management directory
I0107 21:02:10.730869    1919 join.go:184] 6. Start the default mqtt service
I0107 21:02:10.731301    1919 join.go:107] 7. Generate systemd service file
I0107 21:02:10.731783    1919 join.go:107] 8. Generate EdgeCore default configuration
I0107 21:02:10.731795    1919 join.go:270] The configuration does not exist or the parsing fails, and the default configuration is generated
W0107 21:02:11.978368    1919 validation.go:71] NodeIP is empty , use default ip which can connect to cloud.
I0107 21:02:11.980278    1919 join.go:107] 9. Run EdgeCore daemon
I0107 21:02:12.108286    1919 join.go:435] 
I0107 21:02:12.108298    1919 join.go:436] KubeEdge edgecore is running, For logs visit: journalctl -u edgecore.service -xe
```



```shell
[root@edge-1 kubeedge]# systemctl status edgecore
● edgecore.service
   Loaded: loaded (/etc/systemd/system/edgecore.service; enabled; vendor preset: disabled)
   Active: activating (auto-restart) (Result: exit-code) since Sun 2024-01-07 21:02:43 CST; 9s ago
  Process: 2098 ExecStart=/usr/local/bin/edgecore (code=exited, status=1/FAILURE)
 Main PID: 2098 (code=exited, status=1/FAILURE)

Jan 07 21:02:43 edge-1 systemd[1]: edgecore.service: main process exited, code=exited, status=1/FAILURE
Jan 07 21:02:43 edge-1 systemd[1]: Unit edgecore.service entered failed state.
Jan 07 21:02:43 edge-1 systemd[1]: edgecore.service failed.


[root@k8s-master ha]# kubectl get node
NAME         STATUS   ROLES                  AGE   VERSION
edge-1       Ready    agent,edge             15m   v1.23.17-kubeedge-v1.13.4
k8s-master   Ready    control-plane,master   16d   v1.23.12
k8s-node1    Ready    <none>                 16d   v1.23.12
k8s-node2    Ready    <none>                 19h   v1.23.12
```

![](/images/kubeedge/deploy/deploy/deploy-9.png)



### 5.问题解决

```shell
# 查看服务状态
[root@edge-1 kubeedge]# systemctl status edgecore
● edgecore.service
   Loaded: loaded (/etc/systemd/system/edgecore.service; enabled; vendor preset: disabled)
   Active: activating (auto-restart) (Result: exit-code) since Mon 2023-12-18 01:38:11 CST; 4s ago
  Process: 2184 ExecStart=/usr/local/bin/edgecore (code=exited, status=1/FAILURE)
 Main PID: 2184 (code=exited, status=1/FAILURE)

Dec 18 01:38:11 edge-1 systemd[1]: edgecore.service: main process exited, code=exited, status=1/FAILURE
Dec 18 01:38:11 edge-1 systemd[1]: Unit edgecore.service entered failed state.
Dec 18 01:38:11 edge-1 systemd[1]: edgecore.service failed.


# 查看日志
[root@edge-1 kubeedge]# journalctl -u edgecore.service -xe
Dec 18 01:39:23 edge-1 systemd[1]: Stopped edgecore.service.
-- Subject: Unit edgecore.service has finished shutting down
-- Defined-By: systemd
-- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
-- 
-- Unit edgecore.service has finished shutting down.
Dec 18 01:39:23 edge-1 systemd[1]: Started edgecore.service.
-- Subject: Unit edgecore.service has finished start-up
-- Defined-By: systemd
-- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
-- 
-- Unit edgecore.service has finished starting up.
-- 
-- The start-up result is done.
Dec 18 01:39:23 edge-1 edgecore[2255]: W1218 01:39:23.403517    2255 validation.go:71] NodeIP is empty , use default ip which can connect to cloud.
Dec 18 01:39:23 edge-1 edgecore[2255]: I1218 01:39:23.403574    2255 server.go:102] Version: v1.13.4
Dec 18 01:39:23 edge-1 edgecore[2255]: I1218 01:39:23.404592    2255 sql.go:21] Begin to register twin db model
Dec 18 01:39:23 edge-1 edgecore[2255]: I1218 01:39:23.404677    2255 module.go:52] Module twin registered successfully
Dec 18 01:39:23 edge-1 edgecore[2255]: I1218 01:39:23.410874    2255 module.go:52] Module edged registered successfully
Dec 18 01:39:23 edge-1 edgecore[2255]: I1218 01:39:23.410888    2255 module.go:52] Module websocket registered successfully
Dec 18 01:39:23 edge-1 edgecore[2255]: I1218 01:39:23.410892    2255 module.go:52] Module eventbus registered successfully
Dec 18 01:39:23 edge-1 edgecore[2255]: I1218 01:39:23.410911    2255 metamanager.go:41] Begin to register metamanager db model
Dec 18 01:39:23 edge-1 edgecore[2255]: I1218 01:39:23.410930    2255 module.go:52] Module metamanager registered successfully
Dec 18 01:39:23 edge-1 edgecore[2255]: W1218 01:39:23.410936    2255 module.go:55] Module servicebus is disabled, do not register
Dec 18 01:39:23 edge-1 edgecore[2255]: I1218 01:39:23.442777    2255 edgestream.go:55] Get node local IP address successfully: 192.168.202.211
Dec 18 01:39:23 edge-1 edgecore[2255]: W1218 01:39:23.442801    2255 module.go:55] Module edgestream is disabled, do not register
Dec 18 01:39:23 edge-1 edgecore[2255]: W1218 01:39:23.442806    2255 module.go:55] Module testManager is disabled, do not register
Dec 18 01:39:23 edge-1 edgecore[2255]: table `device` already exists, skip
Dec 18 01:39:23 edge-1 edgecore[2255]: table `device_attr` already exists, skip
Dec 18 01:39:23 edge-1 edgecore[2255]: table `device_twin` already exists, skip
Dec 18 01:39:23 edge-1 edgecore[2255]: table `sub_topics` already exists, skip
Dec 18 01:39:23 edge-1 edgecore[2255]: table `meta` already exists, skip
Dec 18 01:39:23 edge-1 edgecore[2255]: table `meta_v2` already exists, skip
Dec 18 01:39:23 edge-1 edgecore[2255]: table `target_urls` already exists, skip
Dec 18 01:39:23 edge-1 edgecore[2255]: I1218 01:39:23.443508    2255 core.go:46] starting module twin
Dec 18 01:39:23 edge-1 edgecore[2255]: I1218 01:39:23.443535    2255 core.go:46] starting module edged
Dec 18 01:39:23 edge-1 edgecore[2255]: I1218 01:39:23.443545    2255 core.go:46] starting module websocket
Dec 18 01:39:23 edge-1 edgecore[2255]: I1218 01:39:23.443556    2255 core.go:46] starting module eventbus
Dec 18 01:39:23 edge-1 edgecore[2255]: I1218 01:39:23.443563    2255 core.go:46] starting module metamanager
Dec 18 01:39:23 edge-1 edgecore[2255]: I1218 01:39:23.443614    2255 process.go:119] Begin to sync sqlite
Dec 18 01:39:23 edge-1 edgecore[2255]: I1218 01:39:23.443773    2255 edged.go:121] Starting edged...
Dec 18 01:39:23 edge-1 edgecore[2255]: I1218 01:39:23.444579    2255 http.go:40] tlsConfig InsecureSkipVerify true
Dec 18 01:39:23 edge-1 edgecore[2255]: I1218 01:39:23.444840    2255 common.go:97] start connect to mqtt server with client id: hub-client-sub-1702834763
Dec 18 01:39:23 edge-1 edgecore[2255]: I1218 01:39:23.444851    2255 common.go:99] client hub-client-sub-1702834763 isconnected: false
Dec 18 01:39:23 edge-1 edgecore[2255]: E1218 01:39:23.445074    2255 common.go:101] connect error: Network Error : dial tcp 127.0.0.1:1883: connect: connection refused
Dec 18 01:39:23 edge-1 edgecore[2255]: I1218 01:39:23.445137    2255 dmiworker.go:67] dmi worker start
Dec 18 01:39:23 edge-1 edgecore[2255]: F1218 01:39:23.445381    2255 certmanager.go:96] Error: failed to get CA certificate, err: Get "https://192.168.202.201:10002/ca.crt": dial tcp 192.168.202.201:10002: conne
Dec 18 01:39:23 edge-1 systemd[1]: edgecore.service: main process exited, code=exited, status=1/FAILURE
Dec 18 01:39:23 edge-1 systemd[1]: Unit edgecore.service entered failed state.
Dec 18 01:39:23 edge-1 systemd[1]: edgecore.service failed.
```

![](D:\KubeEdge\dev\KebeEdge-安装篇\md\install\deploy-9.png)

![](/images/kubeedge/deploy/deploy/deploy-10.png)



```shell
# 问题分析
需要修改cloudcore的外部端口

# 修改配置端口
vim /etc/kubeedge/config/edgecore.yaml
```



### 6.修改配置文件

```shell
# 修改配置文件
[root@edge-1 ~]# cd /etc/kubeedge/config/
[root@edge-1 config]# ll
total 8
-rw-r--r--. 1 root root 5106 Dec 12 20:33 edgecore.yaml


httpServer: https://192.168.202.201:10002
server: 192.168.202.201:10004


[root@k8s-master kubeedge]# kubectl get svc -n kubeedge
NAME        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                           AGE
cloudcore   NodePort   10.110.71.216   <none>        10000:30976/TCP,10001:31372/TCP,10002:31922/TCP,10003:30163/TCP,10004:31927/TCP   22m


#重启服务
systemctl start edgecore
systemctl status edgecore
```

![](/images/kubeedge/deploy/deploy/deploy-11.png)



```shell
[root@edge-1 config]# systemctl start edgecore

[root@edge-1 config]# systemctl status edgecore
● edgecore.service
   Loaded: loaded (/etc/systemd/system/edgecore.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2023-12-18 01:49:22 CST; 2min 44s ago
 Main PID: 2891 (edgecore)
    Tasks: 13
   Memory: 35.8M
   CGroup: /system.slice/edgecore.service
           └─2891 /usr/local/bin/edgecore

Dec 18 01:49:41 edge-1 edgecore[2891]: I1218 01:49:41.964052    2891 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 01:49:56 edge-1 edgecore[2891]: I1218 01:49:56.947845    2891 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 01:50:11 edge-1 edgecore[2891]: I1218 01:50:11.951621    2891 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 01:50:26 edge-1 edgecore[2891]: I1218 01:50:26.959084    2891 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 01:50:41 edge-1 edgecore[2891]: I1218 01:50:41.949065    2891 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 01:50:56 edge-1 edgecore[2891]: I1218 01:50:56.959534    2891 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 01:51:11 edge-1 edgecore[2891]: I1218 01:51:11.944630    2891 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 01:51:26 edge-1 edgecore[2891]: I1218 01:51:26.955171    2891 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 01:51:41 edge-1 edgecore[2891]: I1218 01:51:41.947225    2891 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 01:51:56 edge-1 edgecore[2891]: I1218 01:51:56.956478    2891 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
```

![](/images/kubeedge/deploy/deploy/deploy-12.png)



### 7.安装检查

```shell
[root@k8s-master ha]# kubectl get node
NAME         STATUS   ROLES                  AGE     VERSION
edge-1       Ready    agent,edge             3m54s   v1.23.17-kubeedge-v1.13.4
k8s-master   Ready    control-plane,master   22d     v1.23.12
k8s-node1    Ready    <none>                 22d     v1.23.12
k8s-node2    Ready    <none>                 6d19h   v1.23.12
```

![](/images/kubeedge/deploy/deploy/deploy-13.png)



```shell
[root@edge-1 kubeedge]# docker ps -a
CONTAINER ID   IMAGE                COMMAND                  CREATED         STATUS         PORTS                                                                                  NAMES
08daf58a6cb9   3a05ba674344         "/docker-entrypoint.…"   6 minutes ago   Up 6 minutes                                                                                          k8s_mqtt_mqtt-kubeedge_default_fb2da009-0caf-48f8-92db-680a25c8cd59_0
43e207dcd886   kubeedge/pause:3.6   "/pause"                 6 minutes ago   Up 6 minutes   0.0.0.0:1883->1883/tcp, :::1883->1883/tcp, 0.0.0.0:9001->9001/tcp, :::9001->9001/tcp   k8s_POD_mqtt-kubeedge_default_fb2da009-0caf-48f8-92db-680a25c8cd59_0


[root@edge-1 kubeedge]# docker images
REPOSITORY                      TAG       IMAGE ID       CREATED       SIZE
kubeedge/installation-package   v1.13.4   2603294e1722   2 weeks ago   218MB
eclipse-mosquitto               1.6.15    3a05ba674344   2 years ago   11.6MB
kubeedge/pause                  3.6       6270bb605e12   2 years ago   683kB
```





## 四、常规操作



### 1.cloudcore

#### 1.1.基本操作

```shell
[root@k8s-master kubeedge]# kubectl get all -n kubeedge
NAME                               READY   STATUS    RESTARTS   AGE
pod/cloud-iptables-manager-592m5   1/1     Running   0          68m
pod/cloud-iptables-manager-pg4pl   1/1     Running   0          68m
pod/cloudcore-5959c5986f-8hsc4     1/1     Running   0          68m

NAME                TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                           AGE
service/cloudcore   NodePort   10.110.71.216   <none>        10000:30976/TCP,10001:31372/TCP,10002:31922/TCP,10003:30163/TCP,10004:31927/TCP   68m

NAME                                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/cloud-iptables-manager   2         2         2       2            2           <none>          68m

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cloudcore   1/1     1            1           68m

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/cloudcore-5959c5986f   1         1         1       68m


[root@k8s-master kubeedge]# kubectl get node
NAME         STATUS   ROLES                  AGE     VERSION
edge-1       Ready    agent,edge             19m     v1.23.17-kubeedge-v1.13.4
k8s-master   Ready    control-plane,master   22d     v1.23.12
k8s-node1    Ready    <none>                 22d     v1.23.12
k8s-node2    Ready    <none>                 6d19h   v1.23.12
```

![](/images/kubeedge/deploy/deploy/deploy-17.png)

![](/images/kubeedge/deploy/deploy/deploy-18.png)



#### 1.2.部署Nginx

```shell
[root@k8s-master kubeedge]# vim nginx-nodename.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-nodename
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      nodeName: edge-1    #调度到指定机器
      hostNetwork: true   # 使用主机网络
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

```shell
[root@k8s-master kubeedge]# kubectl apply -f nginx-nodename.yaml 
deployment.apps/nginx-nodename created


[root@k8s-master kubeedge]# kubectl get pod -owide
NAME                              READY   STATUS    RESTARTS   AGE   IP                NODE     NOMINATED NODE   READINESS GATES
nginx-nodename-846f55748c-x8xqx   1/1     Running   0          63s   192.168.202.211   edge-1   <none>           <none>
```

```shell
[root@k8s-master kubeedge]# curl 192.168.202.211
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```



#### 1.3.查看日志

```shell
# 获取IP 192.168.202.203
# kubectl get cm tunnelport -nkubeedge -oyaml
```

![](/images/kubeedge/deploy/deploy/deploy-19.png)



```shell
# 系统重新启动，需要重新设置iptables
iptables -t nat -A OUTPUT -p tcp --dport $YOUR-TUNNEL-PORT -j DNAT --to $YOUR-CLOUDCORE-IP:10003

# 清除 iptables
[root@k8s-master kubeedge]# iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X


# 设置iptables
[root@k8s-master kubeedge]# iptables -t nat -A OUTPUT -p tcp --dport 10351 -j DNAT --to 192.168.202.203:10003
```



```shell
# kubectl logs nginx-nodename-6fb75cb954-xgtf4
# kubectl exec -it nginx-nodename-6fb75cb954-xgtf4 -- /bin/bash
```

![](/images/kubeedge/deploy/deploy/deploy-25.png)



```shell
# kubectl logs nginx-deployment-6fb75cb954-2cw59
# kubectl exec -it nginx-deployment-6fb75cb954-2cw59 -- /bin/bash
# kubectl exec -it -n test nginx-deployment-6748d54874-62jmm -- /bin/bash 
# kubectl exec -it nginx-deployment-5585cb6658-z2qdb bash
# kubectl exec -it nginx-deployment-5585cb6658-z2qdb /bin/bash
```



#### 1.4.配置信息

```shell
kubectl edit configmap cloudcore -n kubeedge


[root@k8s-master kubeedge]# kubectl edit configmap cloudcore -n kubeedge

# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  cloudcore.yaml: |
    apiVersion: cloudcore.config.kubeedge.io/v1alpha2
    commonConfig:
      monitorServer:
        bindAddress: 127.0.0.1:9091
      tunnelPort: 10351
    kind: CloudCore
    kubeAPIConfig:
      burst: 10000
      contentType: application/vnd.kubernetes.protobuf
      kubeConfig: ""
      master: ""
      qps: 5000
    modules:
      cloudHub:
        advertiseAddress:
        - 192.168.202.201
        dnsNames:
        - ""
        edgeCertSigningDuration: 365
        enable: true
        https:
          address: 0.0.0.0
          enable: true
          port: 10002
        keepaliveInterval: 30
        nodeLimit: 1000
        quic:
          address: 0.0.0.0
          enable: false
          maxIncomingStreams: 10000
          port: 10001
        tlsCAFile: /etc/kubeedge/ca/rootCA.crt
        tlsCAKeyFile: /etc/kubeedge/ca/rootCA.key
        tlsCertFile: /etc/kubeedge/certs/edge.crt
        tlsPrivateKeyFile: /etc/kubeedge/certs/edge.key
        tokenRefreshDuration: 12
        unixsocket:
          address: unix:///var/lib/kubeedge/kubeedge.sock
          enable: true
        websocket:
          address: 0.0.0.0
          enable: true
          port: 10000
        writeTimeout: 30
      cloudStream:
        enable: true
        streamPort: 10003
        tlsStreamCAFile: /etc/kubeedge/ca/streamCA.crt
        tlsStreamCertFile: /etc/kubeedge/certs/stream.crt
        tlsStreamPrivateKeyFile: /etc/kubeedge/certs/stream.key
        tlsTunnelCAFile: /etc/kubeedge/ca/rootCA.crt
        tlsTunnelCertFile: /etc/kubeedge/certs/server.crt
        tlsTunnelPrivateKeyFile: /etc/kubeedge/certs/server.key
        tunnelPort: 10004
......
```



### 2.edgecore

#### 2.1.edgecore服务

```shell
systemctl status edgecore
systemctl restart edgecore


[root@edge-1 ~]# systemctl status edgecore
● edgecore.service
   Loaded: loaded (/etc/systemd/system/edgecore.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2023-12-18 02:55:02 CST; 3h 3min ago
 Main PID: 686 (edgecore)
    Tasks: 13
   Memory: 105.7M
   CGroup: /system.slice/edgecore.service
           └─686 /usr/local/bin/edgecore

Dec 18 05:56:01 edge-1 edgecore[686]: I1218 05:56:01.314120     686 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 05:56:16 edge-1 edgecore[686]: I1218 05:56:16.318657     686 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 05:56:31 edge-1 edgecore[686]: I1218 05:56:31.315051     686 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 05:56:46 edge-1 edgecore[686]: I1218 05:56:46.325380     686 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 05:57:01 edge-1 edgecore[686]: I1218 05:57:01.307275     686 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 05:57:15 edge-1 edgecore[686]: E1218 05:57:15.239956     686 serviceaccount.go:111] resource "default"/"default"/[]string(nil)/3607/v1.BoundObjectReference{Kind:"Pod", APIVersion:"v1", N...} token expired
Dec 18 05:57:16 edge-1 edgecore[686]: I1218 05:57:16.315064     686 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 05:57:31 edge-1 edgecore[686]: I1218 05:57:31.306502     686 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 05:57:46 edge-1 edgecore[686]: I1218 05:57:46.321487     686 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 05:58:01 edge-1 edgecore[686]: I1218 05:58:01.306622     686 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Hint: Some lines were ellipsized, use -l to show in full.
```

![](/images/kubeedge/deploy/deploy/deploy-20.png)



#### 2.2.edgecore日志

```shell
journalctl -u edgecore.service -xe

[root@edge-1 ~]# journalctl -u edgecore.service -xe
Dec 18 05:50:31 edge-1 edgecore[686]: I1218 05:50:31.312908     686 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 05:50:46 edge-1 edgecore[686]: I1218 05:50:46.321330     686 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 05:51:01 edge-1 edgecore[686]: I1218 05:51:01.311605     686 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 05:51:05 edge-1 edgecore[686]: E1218 05:51:05.178708     686 serviceaccount.go:111] resource "default"/"default"/[]string(nil)/3607/v1.BoundObjectReference{Kind:"Pod", APIVersion:"v1", Name:"nginx-deploym
Dec 18 05:51:16 edge-1 edgecore[686]: I1218 05:51:16.309213     686 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 05:51:31 edge-1 edgecore[686]: I1218 05:51:31.312873     686 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 05:51:46 edge-1 edgecore[686]: I1218 05:51:46.308663     686 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 05:52:01 edge-1 edgecore[686]: I1218 05:52:01.309796     686 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 05:52:06 edge-1 edgecore[686]: E1218 05:52:06.182333     686 serviceaccount.go:111] resource "default"/"default"/[]string(nil)/3607/v1.BoundObjectReference{Kind:"Pod", APIVersion:"v1", Name:"nginx-deploym
Dec 18 05:52:16 edge-1 edgecore[686]: I1218 05:52:16.323821     686 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 05:52:31 edge-1 edgecore[686]: I1218 05:52:31.323385     686 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 05:52:46 edge-1 edgecore[686]: I1218 05:52:46.310824     686 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 05:53:01 edge-1 edgecore[686]: I1218 05:53:01.321170     686 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
Dec 18 05:53:16 edge-1 edgecore[686]: I1218 05:53:16.317406     686 edgedmetricsconnection.go:136] receive stop signal, so stop metrics scan ...
```

![](/images/kubeedge/deploy/deploy/deploy-21.png)



#### 2.3.配置文件

```shell
# vim /etc/kubeedge/config/edgecore.yaml


[root@edge-1 ~]# vim /etc/kubeedge/config/edgecore.yaml

apiVersion: edgecore.config.kubeedge.io/v1alpha2
database:
  aliasName: default
  dataSource: /var/lib/kubeedge/edgecore.db
  driverName: sqlite3
kind: EdgeCore
modules:
  dbTest:
    enable: false
  deviceTwin:
    enable: true
  edgeHub:
    enable: true
    heartbeat: 15
    httpServer: https://192.168.202.201:32688
    messageBurst: 60
    messageQPS: 30
    projectID: e632aba927ea4ac2b575ec1603d56f10
    quic:
      enable: false
      handshakeTimeout: 30
      readDeadline: 15
      server: 192.168.202.211:10001
      writeDeadline: 15
    rotateCertificates: true
    tlsCaFile: /etc/kubeedge/ca/rootCA.crt
    tlsCertFile: /etc/kubeedge/certs/server.crt
    tlsPrivateKeyFile: /etc/kubeedge/certs/server.key
    token: ""
    websocket:
      enable: true
      handshakeTimeout: 30
      readDeadline: 15
      server: 192.168.202.201:31487
      writeDeadline: 15
  edgeStream:
    enable: true
    handshakeTimeout: 30
    readDeadline: 15
    server: 192.168.202.201:30480
    tlsTunnelCAFile: /etc/kubeedge/ca/rootCA.crt
    tlsTunnelCertFile: /etc/kubeedge/certs/server.crt
    tlsTunnelPrivateKeyFile: /etc/kubeedge/certs/server.key
    writeDeadline: 15
  edged:
    cniBinDir: /opt/cni/bin
    cniCacheDir: /var/lib/cni/cache
    cniConfDir: /etc/cni/net.d
    containerRuntime: docker
    enable: true
    hostnameOverride: edge-1
......
```

![](/images/kubeedge/deploy/deploy/deploy-22.png)



```shell
[root@edge-1 kubeedge]# pwd
/etc/kubeedge
[root@edge-1 kubeedge]# tree
.
├── ca
│?? └── rootCA.crt
├── certs
│?? ├── server.crt
│?? └── server.key
├── config
│?? ├── edgecore.yaml
│?? └── edgecore.yaml.bak
└── dmi.sock
```

![](/images/kubeedge/deploy/deploy/deploy-23.png)



