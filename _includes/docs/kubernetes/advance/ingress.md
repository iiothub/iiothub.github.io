* TOC
{:toc}



## 一、Ingress部署



### 1.安装说明

```shell
# 参考

# K8S---v1.23.5版本部署Ingress
https://kubernetes.github.io/ingress-nginx/deploy/#quick-start
https://blog.csdn.net/m0_57776598/article/details/123978634
https://blog.csdn.net/zfw_666666/article/details/126466270
https://blog.csdn.net/shuaihj/article/details/122932940
```



```shell
# 下载安装文件
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.1/deploy/static/provider/cloud/deploy.yaml


#需要自己在阿里云上拉取
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/nginx-ingress-controller:v1.1.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.1.1


# 修改文件deploy.yaml的镜像
registry.cn-hangzhou.aliyuncs.com/google_containers/nginx-ingress-controller:v1.1.1
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.1.1
```



### 2.安装准备

**1.下载文件**

```shell
[root@k8s-master ingress]# wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.1/deploy/static/provider/cloud/deploy.yaml
--2023-08-07 17:29:20--  https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.1/deploy/static/provider/cloud/deploy.yaml
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.109.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 19299 (19K) [text/plain]
Saving to: ‘deploy.yaml’

100%[============================================================================================>] 19,299      60.6KB/s   in 0.3s   

2023-08-07 17:29:22 (60.6 KB/s) - ‘deploy.yaml’ saved [19299/19299]


[root@k8s-master ingress]# ll
total 20
-rw-r--r-- 1 root root 19299 Aug  7 17:29 deploy.yaml
```



**2.修改deploy.yaml**

```shell
[root@k8s-master ingress]# vim  deploy.yaml

  template:
    metadata:
      name: ingress-nginx-admission-create
      labels:
        helm.sh/chart: ingress-nginx-4.0.15
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 1.1.1
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/component: admission-webhook
    spec:
      containers:
        - name: create          image: registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.1.1
          imagePullPolicy: IfNotPresent
          args:
            - create
            - --host=ingress-nginx-controller-admission,ingress-nginx-controller-admission.$(POD_NAMESPACE).svc
            - --namespace=$(POD_NAMESPACE)
            - --secret-name=ingress-nginx-admission
          env:
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          securityContext:
            allowPrivilegeEscalation: false
      restartPolicy: OnFailure
      serviceAccountName: ingress-nginx-admission
      nodeSelector:
        kubernetes.io/os: linux
      securityContext:
        runAsNonRoot: true
        runAsUser: 2000
---
# Source: ingress-nginx/templates/admission-webhooks/job-patch/job-patchWebhook.yaml
apiVersion: batch/v1
  name: ingress-nginx-admission-patch
  namespace: ingress-nginx
  annotations:
    helm.sh/hook: post-install,post-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    helm.sh/chart: ingress-nginx-4.0.15
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 1.1.1
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: admission-webhook
spec:
  template:
    metadata:
      name: ingress-nginx-admission-patch
      labels:
        helm.sh/chart: ingress-nginx-4.0.15
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 1.1.1
                  fieldPath: metadata.namespace
          securityContext:
            allowPrivilegeEscalation: false
      restartPolicy: OnFailure
      serviceAccountName: ingress-nginx-admission
      nodeSelector:
        kubernetes.io/os: linux
      securityContext:
        runAsNonRoot: true
        runAsUser: 2000
---
# Source: ingress-nginx/templates/admission-webhooks/job-patch/job-patchWebhook.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: ingress-nginx-admission-patch
  namespace: ingress-nginx
  annotations:
    helm.sh/hook: post-install,post-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    helm.sh/chart: ingress-nginx-4.0.15
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 1.1.1
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: admission-webhook
spec:
  template:
    metadata:
      name: ingress-nginx-admission-patch
      labels:
        helm.sh/chart: ingress-nginx-4.0.15
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 1.1.1
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/component: admission-webhook
    spec:
      containers:
        - name: patch
          image: registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.1.1
          imagePullPolicy: IfNotPresent
          args:
            - patch
            - --webhook-name=ingress-nginx-admission
            - --namespace=$(POD_NAMESPACE)
            - --patch-mutating=false
            - --secret-name=ingress-nginx-admission
            - --patch-failure-policy=Fail
          env:
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          securityContext:
            allowPrivilegeEscalation: false
      restartPolicy: OnFailure
      serviceAccountName: ingress-nginx-admission
      nodeSelector:
        kubernetes.io/os: linux
      securityContext:
        runAsNonRoot: true
        runAsUser: 2000
```



### 3.安装

```shell
[root@k8s-master ingress]# kubectl apply -f deploy.yaml
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


# 发现namespace为ingress-nginx的三个pod已经成功完成，status为completed的两个pod为job类型资源，completed表示job已经成功执行无需管它。
[root@k8s-master ingress]# kubectl get pod -n ingress-nginx
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-lsrmq        0/1     Completed   0          50s
ingress-nginx-admission-patch-bftjz         0/1     Completed   0          50s
ingress-nginx-controller-74c6bcdc65-t77nt   1/1     Running     0          50s


[root@k8s-master ingress]# kubectl get all -n ingress-nginx
NAME                                            READY   STATUS      RESTARTS   AGE
pod/ingress-nginx-admission-create-lsrmq        0/1     Completed   0          112s
pod/ingress-nginx-admission-patch-bftjz         0/1     Completed   0          112s
pod/ingress-nginx-controller-74c6bcdc65-t77nt   1/1     Running     0          112s

NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx-controller             LoadBalancer   10.98.187.46   <pending>     80:30831/TCP,443:31019/TCP   112s
service/ingress-nginx-controller-admission   ClusterIP      10.110.81.40   <none>        443/TCP                      112s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-nginx-controller   1/1     1            1           112s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/ingress-nginx-controller-74c6bcdc65   1         1         1       112s

NAME                                       COMPLETIONS   DURATION   AGE
job.batch/ingress-nginx-admission-create   1/1           7s         112s
job.batch/ingress-nginx-admission-patch    1/1           7s         112s
```



### 4.编辑Ingress的svc

编辑Ingress的svc，改为NodePort

```shell
kubectl edit svc ingress-nginx-controller -n ingress-nginx
#将type：LoadBalancer   -------->   改为type：NodePort


[root@k8s-master ingress]# kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.98.187.46   <none>        80:30831/TCP,443:31019/TCP   11m
ingress-nginx-controller-admission   ClusterIP   10.110.81.40   <none>        443/TCP                      11m
```





### 5.测试

#### 5.1.配置文件

**创建tomcat-nginx.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
        - name: nginx
          image: nginx:1.17.1
          ports:
            - containerPort: 80

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deploymeny
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      app: tomcat-pod
  template:
    metadata:
      labels:
        app: tomcat-pod
    spec:
      containers:
        - name: tomcat
          image: tomcat:8.5-jre10-slim
          ports:
            - containerPort: 8080
            
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: dev
spec:
  selector:
    app: nginx-pod
  clusterIP: None
  type: ClusterIP
  ports:
    - port: 80      # service端口
      targetPort: 80    # pod端口

---
apiVersion: v1
kind: Service
metadata:
  name: tomcat-service
  namespace: dev
spec:
  selector:
    app: tomcat-pod
  clusterIP: None
  type: ClusterIP
  ports:
    - port: 8080    # service端口
      targetPort: 8080   # pod端口
```



**Http代理**

> **创建ingress-http.yaml**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-http
  namespace: dev
spec:
  ingressClassName: nginx
  rules:
  - host: nginx.hollysys.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
             name: nginx-service
             port: 
               number: 80
  - host: tomcat.hollysys.com
    http:
      paths:
      - pathType: Prefix 
        path: "/"
        backend:
          service:
             name: tomcat-service
             port: 
               number: 8080
```



#### 5.2.创建

**1.创建资源**

```shell
# 创建
[root@k8s-master-01 ~]# kubectl apply -f tomcat-nginx.yaml 


# 查看
[root@k8s-master ingress]# kubectl get svc -n dev
NAME             TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
nginx-service    ClusterIP   None         <none>        80/TCP     11s
tomcat-service   ClusterIP   None         <none>        8080/TCP   11s


[root@k8s-master ingress]# kubectl get pod -n dev
NAME                                 READY   STATUS    RESTARTS   AGE
nginx-deployment-5ffc5bf56c-tp2nc    1/1     Running   0          116s
nginx-deployment-5ffc5bf56c-w5fjv    1/1     Running   0          116s
nginx-deployment-5ffc5bf56c-x8mg8    1/1     Running   0          116s
tomcat-deploymeny-7db86c59b7-7fhfn   1/1     Running   0          116s
tomcat-deploymeny-7db86c59b7-hf278   1/1     Running   0          116s
tomcat-deploymeny-7db86c59b7-kdwht   1/1     Running   0          116s
```



**2.创建ing**

```shell
# 创建
[root@k8s-master ingress]#  kubectl apply -f ingress-http.yaml
ingress.networking.k8s.io/ingress-http created


# 查看
[root@k8s-master ingress]# kubectl get ing -n dev
NAME           CLASS   HOSTS                                    ADDRESS   PORTS   AGE
ingress-http   nginx   nginx.hollysys.com,tomcat.hollysys.com             80      9s


# 查看详情
[root@k8s-master ingress]# kubectl describe ing ingress-http -n dev
Name:             ingress-http
Labels:           <none>
Namespace:        dev
Address:          10.98.187.46
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host                 Path  Backends
  ----                 ----  --------
  nginx.hollysys.com   
                       /   nginx-service:80 (10.244.169.166:80,10.244.169.177:80,10.244.36.84:80)
  tomcat.hollysys.com  
                       /   tomcat-service:8080 (10.244.169.130:8080,10.244.36.102:8080,10.244.36.122:8080)
Annotations:           <none>
Events:
  Type    Reason  Age                 From                      Message
  ----    ------  ----                ----                      -------
  Normal  Sync    66s (x2 over 114s)  nginx-ingress-controller  Scheduled for sync


# 查看ingress-nginx-controller的pod所在节点ip 
[root@k8s-master ingress]# kubectl get pod -n ingress-nginx -o wide
NAME                                        READY   STATUS      RESTARTS   AGE     IP               NODE        NOMINATED NODE   READINESS GATES
ingress-nginx-admission-create-lsrmq        0/1     Completed   0          3h48m   10.244.36.127    k8s-node1   <none>           <none>
ingress-nginx-admission-patch-bftjz         0/1     Completed   0          3h48m   10.244.169.168   k8s-node2   <none>           <none>
ingress-nginx-controller-74c6bcdc65-t77nt   1/1     Running     0          3h48m   10.244.169.161   k8s-node2   <none>           <none>


# 192.168.202.203为ingress-nginx-controller所在节点ip
192.168.202.203  nginx.hollysys.com
192.168.202.203  tomcat.hollysys.com


# 查看ingress端口30831
[root@k8s-master ingress]# kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.98.187.46   <none>        80:30831/TCP,443:31019/TCP   3h50m
ingress-nginx-controller-admission   ClusterIP   10.110.81.40   <none>        443/TCP                      3h50m
```



#### 5.3.访问

```shell
# linux配置hosts解析
#vim /etc/hosts
10.0.0.18 nginx.hollysys.com
10.0.0.18 tomcat.hollysys.com


# 测试
########## 192.168.202.203为ingress-nginx-controller所在节点ip
在本机添加域名解析
C:\Windows\System32\drivers\etc
在hosts文件添加域名解析(master主机的iP地址)
192.168.202.203  nginx.hollysys.com
192.168.202.203  tomcat.hollysys.com
  

# 在本机浏览器输入:
http://nginx.hollysys.com:30831/
http://tomcat.hollysys.com:30831/
```



