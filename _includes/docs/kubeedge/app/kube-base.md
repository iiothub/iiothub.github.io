* TOC
{:toc}


![](/images/kubeedge/app/kube-base/base-1.png)

![](/images/kubeedge/app/kube-base/base-15.png)





### 1.基础操作

```shell
# 基本操作

[root@k8s-master kubeedge]# kubectl get node
NAME         STATUS   ROLES                  AGE     VERSION
edge-1       Ready    agent,edge             3h24m   v1.23.17-kubeedge-v1.13.4
edge-2       Ready    agent,edge             173m    v1.23.17-kubeedge-v1.13.4
k8s-master   Ready    control-plane,master   7d1h    v1.23.12
k8s-node1    Ready    <none>                 7d1h    v1.23.12


[root@k8s-master kubeedge]# kubectl top node
NAME         CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
edge-1       22m          1%     256Mi           6%        
edge-2       24m          1%     303Mi           8%        
k8s-master   91m          4%     1396Mi          36%       
k8s-node1    46m          2%     776Mi           20%       


[root@k8s-master kubeedge]# kubectl get all -n kubeedge
NAME                               READY   STATUS    RESTARTS       AGE
pod/cloud-iptables-manager-5ccvn   1/1     Running   1 (175m ago)   3h47m
pod/cloudcore-5959c5986f-bhc8w     1/1     Running   1 (175m ago)   3h47m

NAME                TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                           AGE
service/cloudcore   NodePort   10.101.191.217   <none>        10000:30032/TCP,10001:30501/TCP,10002:32021/TCP,10003:31580/TCP,10004:32330/TCP   3h47m

NAME                                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/cloud-iptables-manager   1         1         1       1            1           <none>          3h47m

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cloudcore   1/1     1            1           3h47m

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/cloudcore-5959c5986f   1         1         1       3h47m
```

![](/images/kubeedge/app/kube-base/base-2.png)

![](/images/kubeedge/app/kube-base/base-3.png)





### 2.部署Nginx

```yaml
[root@k8s-master kubeedge]# vim nginx.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
      nodeName: edge-2  #调度到指定机器
      hostNetwork: true   # 使用主机网络
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          hostPort: 8080
```

```shell
[root@k8s-master kubeedge]# kubectl apply -f nginx.yaml 
deployment.apps/nginx-deployment created


[root@k8s-master kubeedge]# kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-6bd49f646f-2wd69   1/1     Running   0          36s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   7d1h

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   1/1     1            1           36s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-6bd49f646f   1         1         1       36s
```

![](/images/kubeedge/app/kube-base/base-4.png)

```shell
# 访问
# curl 192.168.202.212:8080


[root@k8s-master kubeedge]# curl 192.168.202.212:8080
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





### 3.CloudCore

#### 3.1.查看日志

```shell
# 获取IP 192.168.202.202

# kubectl get cm tunnelport -nkubeedge -oyaml
```

![](/images/kubeedge/app/kube-base/base-5.png)



```shell
# 系统重新启动，需要重新设置iptables

# 清除 iptables
[root@k8s-master kubeedge]# iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X


# 设置iptables
[root@k8s-master kubeedge]# iptables -t nat -A OUTPUT -p tcp --dport 10351 -j DNAT --to 192.168.202.202:10003
```



```shell
# kubectl logs nginx-deployment-6bd49f646f-2wd69


[root@k8s-master kubeedge]# kubectl logs nginx-deployment-6bd49f646f-2wd69
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/12/22 16:14:56 [notice] 1#1: using the "epoll" event method
2023/12/22 16:14:56 [notice] 1#1: nginx/1.21.5
2023/12/22 16:14:56 [notice] 1#1: built by gcc 10.2.1 20210110 (Debian 10.2.1-6) 
2023/12/22 16:14:56 [notice] 1#1: OS: Linux 3.10.0-1127.el7.x86_64
2023/12/22 16:14:56 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2023/12/22 16:14:56 [notice] 1#1: start worker processes
2023/12/22 16:14:56 [notice] 1#1: start worker process 31
2023/12/22 16:14:56 [notice] 1#1: start worker process 32
192.168.202.201 - - [22/Dec/2023:16:18:09 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.29.0" "-"
```

![](/images/kubeedge/app/kube-base/base-6.png)



#### 3.2.kuebctl exec

```shell
# kubectl exec -it nginx-deployment-6fb75cb954-2cw59 -- /bin/bash
# kubectl exec -it -n test nginx-deployment-6748d54874-62jmm -- /bin/bash 
# kubectl exec -it nginx-deployment-5585cb6658-z2qdb bash
# kubectl exec -it nginx-deployment-5585cb6658-z2qdb /bin/bash
```

```shell
# kubectl exec -it nginx-deployment-6bd49f646f-2wd69 /bin/bash
# kubectl exec -it nginx-deployment-6bd49f646f-2wd69 -- /bin/bash


[root@k8s-master kubeedge]# kubectl exec -it nginx-deployment-6bd49f646f-2wd69 -- /bin/bash
root@nginx-deployment-6bd49f646f-2wd69:/# ls
bin  boot  dev	docker-entrypoint.d  docker-entrypoint.sh  etc	home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@nginx-deployment-6bd49f646f-2wd69:/# exit
exit
```

![](/images/kubeedge/app/kube-base/base-7.png)



#### 3.3.收集metrics 

```shell
[root@k8s-master kubeedge]# kubectl top nodes
NAME         CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
edge-1       21m          1%     257Mi           7%        
edge-2       24m          1%     350Mi           9%        
k8s-master   88m          4%     1383Mi          36%       
k8s-node1    48m          2%     777Mi           20%       


[root@k8s-master kubeedge]# kubectl top pod
NAME                                CPU(cores)   MEMORY(bytes)   
nginx-deployment-6bd49f646f-2wd69   0m           2Mi 
```

![](/images/kubeedge/app/kube-base/base-14.png)



#### 3.4.配置信息

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





### 4.EdgeCore

#### 4.1.edgecore服务

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

![](/images/kubeedge/app/kube-base/base-8.png)



#### 4.2.edgecore日志

```shell
# journalctl -u edgecore.service -xe


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

![](/images/kubeedge/app/kube-base/base-9.png)



#### 4.3.配置文件

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

![](/images/kubeedge/app/kube-base/base-10.png)



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

![](/images/kubeedge/app/kube-base/base-11.png)



#### 4.4.Docker

```shell
[root@edge-2 kubeedge]# docker ps -a


[root@edge-2 kubeedge]# docker logs c6adffadf37b
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/12/22 16:14:56 [notice] 1#1: using the "epoll" event method
2023/12/22 16:14:56 [notice] 1#1: nginx/1.21.5
2023/12/22 16:14:56 [notice] 1#1: built by gcc 10.2.1 20210110 (Debian 10.2.1-6) 
2023/12/22 16:14:56 [notice] 1#1: OS: Linux 3.10.0-1127.el7.x86_64
2023/12/22 16:14:56 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2023/12/22 16:14:56 [notice] 1#1: start worker processes
2023/12/22 16:14:56 [notice] 1#1: start worker process 31
2023/12/22 16:14:56 [notice] 1#1: start worker process 32
192.168.202.201 - - [22/Dec/2023:16:18:09 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.29.0" "-"
```

![](/images/kubeedge/app/kube-base/base-12.png)

![](/images/kubeedge/app/kube-base/base-13.png)



