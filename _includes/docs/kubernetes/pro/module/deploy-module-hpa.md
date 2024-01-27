* TOC
{:toc}


### 1.HPA

HPA（Horizontal Pod Autoscaler）在k8s集群中用于POD水平自动伸缩，它是基于CPU和内存利用率对Deployment和Replicaset控制器中的pod数量进行自动扩缩容（除了CPU和内存利用率之外，也可以基于其他应程序提供的度量指标custom metrics进行自动扩缩容）。pod自动缩放不适用于无法缩放的对象，比如DaemonSets。HPA由Kubernetes API资源和控制器实现。资源决定了控制器的行为，控制器会周期性的获取CPU和内存利用率，并与目标值相比较后来调整replication controller或deployment中的副本数量。

 

**HPA使用前提条件：**

- 集群中部署了Metrics Server插件, 可以获取到Pod的CPU和内存利用率。
- Pod部署的Yaml文件中必须设置资源限制和资源请求。



#### 1.1.安装metrics-server

metrics-server可以用来收集集群中的资源使用情况

https://github.com/kubernetes-sigs/metrics-server



**参考**

```shell
# 安装git
[root@master ~]# yum install git -y
# 获取metrics-server, 注意使用的版本
[root@master ~]# git clone -b v0.3.6 https://github.com/kubernetes-incubator/metrics-server
# 修改deployment, 注意修改的是镜像和初始化参数
[root@master ~]# cd /root/metrics-server/deploy/1.8+/
[root@master 1.8+]# vim metrics-server-deployment.yaml
按图中添加下面选项
hostNetwork: true
image: registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server-amd64:v0.3.6
args:
- --kubelet-insecure-tls
- --kubelet-preferred-address-types=InternalIP,Hostname,InternalDNS,ExternalDNS,ExternalIP
```

![](/images/kubernetes/pro/module/hpa-3.png)

**安装**

```shell
# 获取metrics-server, 注意使用的版本
[root@k8s-master hpa]# git clone -b v0.3.6 https://github.com/kubernetes-incubator/metrics-server


# 修改deployment, 注意修改的是镜像和初始化参数
[root@k8s-master1 1.8+]# cd /k8s/module/hpa/metrics-server/deploy/1.8+
[root@k8s-master1 1.8+]# ll
total 28
-rw-r--r--. 1 root root 393 Dec 17 09:22 aggregated-metrics-reader.yaml
-rw-r--r--. 1 root root 308 Dec 17 09:22 auth-delegator.yaml
-rw-r--r--. 1 root root 329 Dec 17 09:22 auth-reader.yaml
-rw-r--r--. 1 root root 298 Dec 17 09:22 metrics-apiservice.yaml
-rw-r--r--. 1 root root 804 Dec 17 09:22 metrics-server-deployment.yaml
-rw-r--r--. 1 root root 291 Dec 17 09:22 metrics-server-service.yaml
-rw-r--r--. 1 root root 517 Dec 17 09:22 resource-reader.yaml


[root@master 1.8+]# vim metrics-server-deployment.yaml
# 按图中添加下面选项
hostNetwork: true
image: registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server-amd64:v0.3.6
args:
- --kubelet-insecure-tls
- --kubelet-preferred-address-types=InternalIP,Hostname,InternalDNS,ExternalDNS,ExternalIP
```

```powershell
# 安装metrics-server

[root@k8s-master1 1.8+]# kubectl apply -f ./
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
Warning: rbac.authorization.k8s.io/v1beta1 ClusterRoleBinding is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 ClusterRoleBinding
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
Warning: rbac.authorization.k8s.io/v1beta1 RoleBinding is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 RoleBinding
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
Warning: apiregistration.k8s.io/v1beta1 APIService is deprecated in v1.19+, unavailable in v1.22+; use apiregistration.k8s.io/v1 APIService
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
serviceaccount/metrics-server created
deployment.apps/metrics-server created
service/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created


# 查看pod运行情况
[root@k8s-master 1.8+]# kubectl get pod -n kube-system
NAME                                       READY   STATUS    RESTARTS   AGE
...
metrics-server-9c7dc6fdc-mgl6r             1/1     Running   0          30s


# 使用kubectl top node 查看资源使用情况
[root@k8s-master1 1.8+]# kubectl top node
NAME          CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
k8s-master1   344m         8%     3791Mi          24%       
k8s-master2   283m         7%     2373Mi          15%       
k8s-master3   332m         8%     2202Mi          13%       
k8s-node1     250m         6%     1189Mi          7%        
k8s-node2     258m         6%     1270Mi          8%        
k8s-node3     267m         6%     1173Mi          7%        
k8s-node4     240m         6%     1197Mi          7% 


[root@k8s-master1 1.8+]# kubectl top pod -n kube-system
NAME                                       CPU(cores)   MEMORY(bytes)   
calico-kube-controllers-558995777d-l8lpr   3m           27Mi            
calico-node-6l5w2                          21m          150Mi           
calico-node-g68dm                          22m          142Mi           
calico-node-qt82k                          29m          150Mi           
calico-node-rvtbg                          34m          146Mi           
calico-node-tn2xj                          19m          146Mi           
calico-node-wnjhd                          24m          149Mi           
calico-node-wt5cb                          23m          149Mi           
coredns-7f89b7bc75-h2htb                   2m           16Mi            
coredns-7f89b7bc75-k5phr                   2m           16Mi            
etcd-k8s-master1                           19m          192Mi           
etcd-k8s-master2                           19m          193Mi           
etcd-k8s-master3                           21m          192Mi           
kube-apiserver-k8s-master1                 35m          453Mi           
kube-apiserver-k8s-master2                 35m          427Mi           
kube-apiserver-k8s-master3                 34m          438Mi           
kube-controller-manager-k8s-master1        1m           29Mi            
kube-controller-manager-k8s-master2        1m           27Mi            
kube-controller-manager-k8s-master3        7m           61Mi            
kube-proxy-5pjpd                           1m           20Mi            
kube-proxy-86vds                           1m           21Mi            
kube-proxy-cv28h                           1m           18Mi            
kube-proxy-mmbj6                           1m           17Mi            
kube-proxy-s9lhm                           1m           19Mi            
kube-proxy-tvt2z                           1m           18Mi            
kube-proxy-w7j6l                           1m           21Mi            
kube-scheduler-k8s-master1                 2m           23Mi            
kube-scheduler-k8s-master2                 2m           23Mi            
kube-scheduler-k8s-master3                 2m           27Mi            
metrics-server-67bd88dc86-dk2w9            1m           14Mi  


# 至此,metrics-server安装完成
```



#### 1.2.常用命令

```shell
# 使用kubectl top node 查看资源使用情况
[root@master 1.8+]# kubectl top node


[root@k8s-master hpa]# kubectl describe node k8s-node1


# 查看pod资源
[root@master 1.8+]# kubectl top pod -n kube-system


# 查看hpa
[root@k8s-master hpa]# kubectl get hpa -n dev


[root@k8s-master ~]# kubectl get deploy -n dev -w
```



