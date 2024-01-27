* TOC
{:toc}


### 1.Ingress

Ingress 是对集群中服务的外部访问进行管理的 API 对象，典型的访问方式是 HTTP和HTTPS。

Ingress 可以提供负载均衡、SSL 和基于名称的虚拟托管。

必须具有 ingress 控制器【例如 ingress-nginx】才能满足 Ingress 的要求。仅创建 Ingress 资源无效。



#### 1.1.搭建Ingress

（1）在gitlab上下载yaml文件，并创建部署

gitlab ingress-nginx项目：https://github.com/kubernetes/ingress-nginx

ingress安装指南：https://kubernetes.github.io/ingress-nginx/deploy/



```shell
https://kubernetes.github.io/ingress-nginx/deploy/

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.0/deploy/static/provider/baremetal/deploy.yaml


# 安装说明
Bare metal clusters¶
This section is applicable to Kubernetes clusters deployed on bare metal servers, as well as "raw" VMs where Kubernetes was installed manually, using generic Linux distros (like CentOS, Ubuntu...)

For quick testing, you can use a NodePort. This should work on almost every cluster, but it will typically use a port in the range 30000-32767.


kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.0/deploy/static/provider/baremetal/deploy.yaml


For more information about bare metal deployments (and how to use port 80 instead of a random port in the 30000-32767 range), see bare-metal considerations.
```



```shell
# 创建文件夹
[root@k8s-master1 ingress]# pwd
/k8s/module/ingress


[root@k8s-master1 ingress]# wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.0/deploy/static/provider/baremetal/deploy.yaml
Connecting to raw.githubusercontent.com (185.199.108.133:443)
deploy.yaml          100% |******************************************************************************| 19299   0:00:00 ETA


# kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.0/deploy/static/provider/baremetal/deploy.yaml
[root@k8s-master1 ingress]# kubectl apply -f deploy.yaml
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
configmap/ingress-nginx-controller created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
service/ingress-nginx-controller-admission created
service/ingress-nginx-controller created
deployment.apps/ingress-nginx-controller created
ingressclass.networking.k8s.io/nginx created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
serviceaccount/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created


# 查看nginx-ingress-controller控制器
[root@k8s-master1 ingress]# kubectl get pod -n ingress-nginx -o wide
NAME                                        READY   STATUS              RESTARTS   AGE   IP               NODE        NOMINATED NODE   READINESS GATES
ingress-nginx-admission-create-d77f7        0/1     ErrImagePull        0          23m   10.244.36.67     k8s-node1   <none>           <none>
ingress-nginx-admission-patch-245h9         0/1     ErrImagePull        0          23m   10.244.169.131   k8s-node2   <none>           <none>
ingress-nginx-controller-69db7f75b4-ws8x9   0/1     ContainerCreating   0          23m   <none>           k8s-node3   <none>           <none>


# 此种方式无法拉取镜像
```

```shell
# 问题解决


# 无法拉取镜像
[root@k8s-master1 ingress]#  kubectl get pod -n ingress-nginx -o wide
NAME                                        READY   STATUS              RESTARTS   AGE   IP               NODE        NOMINATED NODE   READINESS GATES
ingress-nginx-admission-create-bcd7x        0/1     ImagePullBackOff    0          18m   10.244.122.95    k8s-node4   <none>           <none>
ingress-nginx-admission-patch-hkp57         0/1     ErrImagePull        0          18m   10.244.107.228   k8s-node3   <none>           <none>
ingress-nginx-controller-65c4f84996-crknk   0/1     ContainerCreating   0          18m   <none>           k8s-node2   <none>           <none>


# 需要拉取的镜像
k8s.gcr.io/ingress-nginx/controller:v1.0.0@sha256:0851b34f69f69352bf168e6ccf30e1e20714a264ab1ecd1933e4d8c0fc3215c6

k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.0@sha256:f3b6b39a6062328c095337b4cadcefd1612348fdd5190b1dcbcb9b9e90bd8068


#修改镜像地址
	image: k8s.gcr.io/ingress-nginx/controller:v1.0.0  -> https://hub.docker.com/r/willdockerhub/ingress-nginx-controller
	docker pull willdockerhub/ingress-nginx-controller:v1.0.0
	
	image: k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.0 -> https://hub.docker.com/r/jettech/kube-webhook-certgen/tags
	docker pull jettech/kube-webhook-certgen:v1.0.0
	
	
# 修改文件
# 修改镜像地址：
# willdockerhub/ingress-nginx-controller:v1.0.0
# jettech/kube-webhook-certgen:v1.0.0
[root@k8s-master1 ingress]# vim deploy.yaml
```

```shell
# 创新创建

[root@k8s-master1 ingress]# kubectl apply -f deploy.yaml
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
configmap/ingress-nginx-controller created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
service/ingress-nginx-controller-admission created
service/ingress-nginx-controller created
deployment.apps/ingress-nginx-controller created
ingressclass.networking.k8s.io/nginx created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
serviceaccount/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created


[root@k8s-master1 ingress]# kubectl get all -n ingress-nginx
NAME                                            READY   STATUS      RESTARTS   AGE
pod/ingress-nginx-admission-create-hhhbp        0/1     Completed   0          57s
pod/ingress-nginx-admission-patch-nd458         0/1     Completed   0          57s
pod/ingress-nginx-controller-7d4df87d89-7gjmc   1/1     Running     0          57s

NAME                                         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx-controller             NodePort    10.1.253.108   <none>        80:31100/TCP,443:31826/TCP   57s
service/ingress-nginx-controller-admission   ClusterIP   10.1.236.3     <none>        443/TCP                      57s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-nginx-controller   1/1     1            1           57s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/ingress-nginx-controller-7d4df87d89   1         1         1       57s

NAME                                       COMPLETIONS   DURATION   AGE
job.batch/ingress-nginx-admission-create   1/1           2s         57s
job.batch/ingress-nginx-admission-patch    1/1           2s         57s



# 查看service规则
[root@k8s-master1 ingress]# kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.1.253.108   <none>        80:31100/TCP,443:31826/TCP   83s
ingress-nginx-controller-admission   ClusterIP   10.1.236.3     <none>        443/TCP                      83s
```



```shell
# 问题
[root@k8s-master ingress]#  kubectl apply -f ingress-http.yaml
Error from server (InternalError): error when creating "ingress-http.yaml": Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io": Post "https://ingress-nginx-controller-admission.ingress-nginx.svc:443/networking/v1/ingresses?timeout=10s": dial tcp 10.100.142.101:443: connect: connection refused

# 解决方式
解决方案：
最后参考下面的文章解决此问题
使用下面的命令查看 webhook
kubectl get validatingwebhookconfigurations
ingress-nginx-admission

删除ingress-nginx-admission
kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission

https://blog.csdn.net/qq_39218530/article/details/115372879
```




