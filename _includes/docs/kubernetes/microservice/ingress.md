* TOC
{:toc}



## 一、概述



### 1.Ingress

通过Ingress访问的组件（HTTP、HTTPS），Ingress访问方式NodePort

- 注册中心（eureka-server）
- 网关（msa-gateway）
- 调用链监控（Zipkin）
- 流量控制（Sentinel）
- 分布式任务调度平台（XXL-JOB）



### 2.Ingress安装

（1）在gitlab上下载yaml文件，并创建部署

gitlab ingress-nginx项目：https://github.com/kubernetes/ingress-nginx

ingress安装指南：https://kubernetes.github.io/ingress-nginx/deploy/



```shell
# 创建文件夹
[root@k8s-master ingress-controller]# mkdir ingress-controller
[root@k8s-master ingress-controller]# cd ingress-controller/
[root@k8s-master ingress-controller]# pwd
/k8s/ingress/ingress-controller


[root@k8s-master ingress-controller]# wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.0/deploy/static/provider/baremetal/deploy.yaml


[root@k8s-master ingress-controller]# kubectl apply -f deploy.yaml 
namespace/ingress-nginx configured
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
[root@k8s-master ingress-controller]# kubectl get pod -n ingress-nginx -o wide
NAME                                        READY   STATUS              RESTARTS   AGE     IP               NODE        NOMINATED NODE   READINESS GATES
ingress-nginx-admission-create-nzs4l        0/1     ImagePullBackOff    0          11m     10.244.36.77     k8s-node1   <none>           <none>
ingress-nginx-admission-patch-x4gg6         0/1     ImagePullBackOff    0          11m     10.244.107.215   k8s-node3   <none>           <none>
ingress-nginx-controller-57ffff5864-b22m4   0/1     ContainerCreating   0          11m     <none>           k8s-node2   <none>           <none>
nginx-ingress-controller-54b86f8f7b-jxzst   1/1     Running             0          7h53m   10.244.107.212   k8s-node3   <none>           <none>


# 查看service规则
[root@k8s-master ingress-controller]#  kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx                        NodePort    10.97.245.122    <none>        80:31208/TCP,443:32099/TCP   7h47m
ingress-nginx-controller             NodePort    10.101.238.213   <none>        80:30743/TCP,443:31410/TCP   5m21s
ingress-nginx-controller-admission   ClusterIP   10.100.142.101   <none>        443/TCP                      5m21s
```



### 3.Ingress测试

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

```shell
# 创建
[root@k8s-master ingress]# kubectl apply -f tomcat-nginx.yaml 
deployment.apps/nginx-deployment created
deployment.apps/tomcat-deploymeny created
service/nginx-service created
service/tomcat-service created


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



**Http代理**

> **创建ingress-http.yaml**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-http
  namespace: dev
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
    - host: nginx.nana.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
    - host: tomcat.nana.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: tomcat-service
                port:
                  number: 8080
```

```shell
# 创建
[root@k8s-master ingress]# kubectl apply -f ingress-http.yaml 
ingress.networking.k8s.io/ingress-http created


# 查看
[root@k8s-master ingress]# kubectl get ing -n dev
NAME           CLASS    HOSTS                            ADDRESS         PORTS   AGE
ingress-http   <none>   nginx.nana.com,tomcat.nana.com   10.97.245.122   80      49s


# 查看详情
[root@k8s-master ingress]# kubectl get ing -n dev
NAME           CLASS    HOSTS                            ADDRESS         PORTS   AGE
ingress-http   <none>   nginx.nana.com,tomcat.nana.com   10.97.245.122   80      49s
[root@k8s-master ingress]# 
[root@k8s-master ingress]# 
[root@k8s-master ingress]# kubectl describe ing ingress-http -n dev
Name:             ingress-http
Namespace:        dev
Address:          10.97.245.122
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host             Path  Backends
  ----             ----  --------
  nginx.nana.com   
                   /   nginx-service:80 (10.244.107.193:80,10.244.169.188:80,10.244.36.93:80)
  tomcat.nana.com  
                   /   tomcat-service:8080 (10.244.169.175:8080,10.244.169.184:8080,10.244.36.94:8080)
Annotations:       kubernetes.io/ingress.class: nginx
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  CREATE  83s   nginx-ingress-controller  Ingress dev/ingress-http
  Normal  UPDATE  65s   nginx-ingress-controller  Ingress dev/ingress-http


# 测试
在本机添加域名解析
C:\Windows\System32\drivers\etc
在hosts文件添加域名解析(master主机的iP地址)
172.51.216.81 		nginx.nana.com,tomcat.nana.com 	


[root@k8s-master ingress-controller]#  kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx                        NodePort    10.97.245.122    <none>        80:31208/TCP,443:32099/TCP   8h
ingress-nginx-controller             NodePort    10.101.238.213   <none>        80:30743/TCP,443:31410/TCP   45m
ingress-nginx-controller-admission   ClusterIP   10.100.142.101   <none>        443/TCP    


# 在本机浏览器输入:
http://nginx.nana.com:31208/
http://tomcat.nana.com:31208/
```



**Https代理**



**创建证书**

```shell
# 生成证书
[root@k8s-master ingress]# openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/C=CN/ST=BJ/L=BJ/O=nginx/CN=itheima.com"
Generating a 2048 bit RSA private key
...................+++
...........................................+++
writing new private key to 'tls.key'
-----


[root@k8s-master ingress]# ll
total 16
-rw-r--r-- 1 root root 1249 Nov  8 14:59 tls.crt
-rw-r--r-- 1 root root 1704 Nov  8 14:59 tls.key


# 创建密钥
[root@k8s-master ingress]# kubectl create secret tls tls-secret --key tls.key --cert tls.crt
secret/tls-secret created
```



**创建ingress-https.yaml**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-https
  namespace: dev
spec:
  tls:
    - hosts:
        - nginx.nana.com
        - tomcat.nana.com
      secretName: tls-secret    # 指定密钥
  rules:
    - host: nginx.nana.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
    - host: tomcat.nana.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: tomcat-service
                port:
                  number: 8080
```

```shell
# 创建
[root@k8s-master ingress]# kubectl create -f ingress-https.yaml 
ingress.networking.k8s.io/ingress-https created


# 查看
[root@k8s-master ingress]# kubectl get ing -n dev
NAME            CLASS    HOSTS                            ADDRESS         PORTS     AGE
ingress-http    <none>   nginx.nana.com,tomcat.nana.com   10.97.245.122   80        12m
ingress-https   <none>   nginx.nana.com,tomcat.nana.com                   80, 443   28s


# 查看详情
[root@k8s-master ingress]# kubectl describe ing ingress-https -n dev
Name:             ingress-https
Namespace:        dev
Address:          10.97.245.122
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
TLS:
  tls-secret terminates nginx.nana.com,tomcat.nana.com
Rules:
  Host             Path  Backends
  ----             ----  --------
  nginx.nana.com   
                   /   nginx-service:80 (10.244.107.193:80,10.244.169.188:80,10.244.36.93:80)
  tomcat.nana.com  
                   /   tomcat-service:8080 (10.244.169.175:8080,10.244.169.184:8080,10.244.36.94:8080)
Annotations:       <none>
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  CREATE  67s   nginx-ingress-controller  Ingress dev/ingress-https
  Normal  UPDATE  26s   nginx-ingress-controller  Ingress dev/ingress-https


[root@k8s-master ingress-controller]#  kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx                        NodePort    10.97.245.122    <none>        80:31208/TCP,443:32099/TCP   8h
ingress-nginx-controller             NodePort    10.101.238.213   <none>        80:30743/TCP,443:31410/TCP   45m
ingress-nginx-controller-admission   ClusterIP   10.100.142.101   <none>        443/TCP


# 在本机浏览器输入:
https://nginx.nana.com:32099/
https://tomcat.nana.com:32099/
```





## 二、SpringCloud集成



### 1.注册中心（eureka-server）

**Http代理**

> **创建 eureka-http.yaml**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: eureka-http
  namespace: dev
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
    - host: eureka.k8s.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: msa-eureka
                port:
                  number: 10001
```

```shell
# 创建
[root@k8s-master ingress]# kubectl apply -f eureka-http.yaml 
ingress.networking.k8s.io/eureka-http created


# 查看
[root@k8s-master ingress]# kubectl get ing -n dev
NAME          CLASS    HOSTS            ADDRESS         PORTS   AGE
eureka-http   <none>   eureka.k8s.com   10.97.245.122   80      63s


# 查看详情
[root@k8s-master ingress]# kubectl describe ing eureka-http -n dev
Name:             eureka-http
Namespace:        dev
Address:          10.97.245.122
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host            Path  Backends
  ----            ----  --------
  eureka.k8s.com  
                  /   msa-eureka:10001 (10.244.169.153:10001,10.244.169.155:10001,10.244.169.157:10001)
Annotations:      kubernetes.io/ingress.class: nginx
Events:
  Type    Reason  Age    From                      Message
  ----    ------  ----   ----                      -------
  Normal  CREATE  3m38s  nginx-ingress-controller  Ingress dev/eureka-http
  Normal  UPDATE  3m6s   nginx-ingress-controller  Ingress dev/eureka-http


# 测试
在本机添加域名解析
C:\Windows\System32\drivers\etc
在hosts文件添加域名解析(master主机的iP地址)
172.51.216.81 		eureka.k8s.com


[root@k8s-master ingress-controller]#  kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx                        NodePort    10.97.245.122    <none>        80:31208/TCP,443:32099/TCP   8h
ingress-nginx-controller             NodePort    10.101.238.213   <none>        80:30743/TCP,443:31410/TCP   45m
ingress-nginx-controller-admission   ClusterIP   10.100.142.101   <none>        443/TCP    


# 在本机浏览器输入:
http://eureka.k8s.com:31208/
http://172.51.216.81:30001/
```



### 2.网关（msa-gateway）

**Http代理**

> **创建 gateway-http.yaml**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: gateway-http
  namespace: dev
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
    - host: gateway.k8s.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: msa-gateway
                port:
                  number: 8888
```

```shell
# 创建
[root@k8s-master ingress]# kubectl apply -f gateway-http.yaml 
ingress.networking.k8s.io/gateway-http created


# 查看
[root@k8s-master ingress]# kubectl get ing -n dev
NAME           CLASS    HOSTS             ADDRESS         PORTS   AGE
eureka-http    <none>   eureka.k8s.com    10.97.245.122   80      4h46m
gateway-http   <none>   gateway.k8s.com   10.97.245.122   80      60s


# 查看详情
[root@k8s-master ingress]#  kubectl describe ing gateway-http -n dev
Name:             gateway-http
Namespace:        dev
Address:          10.97.245.122
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host             Path  Backends
  ----             ----  --------
  gateway.k8s.com  
                   /   msa-gateway:8888 (10.244.107.207:8888,10.244.107.208:8888,10.244.169.151:8888)
Annotations:       kubernetes.io/ingress.class: nginx
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  CREATE  103s  nginx-ingress-controller  Ingress dev/gateway-http
  Normal  UPDATE  66s   nginx-ingress-controller  Ingress dev/gateway-http


# 测试
在本机添加域名解析
C:\Windows\System32\drivers\etc
在hosts文件添加域名解析(master主机的iP地址)
172.51.216.81 		gateway.k8s.com


[root@k8s-master ingress]# kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx                        NodePort    10.97.245.122    <none>        80:31208/TCP,443:32099/TCP   56d
ingress-nginx-controller             NodePort    10.101.238.213   <none>        80:30743/TCP,443:31410/TCP   55d
ingress-nginx-controller-admission   ClusterIP   10.100.142.101   <none>        443/TCP                      55d   


# 网关地址
http://gateway.k8s.com:31208/


# 访问地址：
http://172.51.216.81:30008/consumer/hello
http://172.51.216.81:30008/consumer/say
http://172.51.216.81:30008/producer/hello
http://172.51.216.81:30008/producer/say


# 测试地址
http://gateway.k8s.com:31208/consumer/hello
http://gateway.k8s.com:31208/consumer/say
http://gateway.k8s.com:31208/producer/hello
http://gateway.k8s.com:31208/producer/say
```



### 3.中间件

**调用链监控（Zipkin）、流量控制（Sentinel）、分布式任务调度平台（XXL-JOB）。**

**Http代理**

> **创建 middleware-http.yaml**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: middleware-http
  namespace: dev
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
    - host: zipkin.k8s.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: zipkin-server
                port:
                  number: 9411
    - host: sentinel.k8s.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: sentinel-server
                port:
                  number: 8858
    - host: xxljob.k8s.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: xxl-job
                port:
                  number: 8080
```

```shell
# 创建
[root@k8s-master ingress]# kubectl apply -f middleware-http.yaml 
ingress.networking.k8s.io/middleware-http created


# 查看
[root@k8s-master ingress]# kubectl get ing -n dev
NAME              CLASS    HOSTS                                            ADDRESS         PORTS   AGE
eureka-http       <none>   eureka.k8s.com                                   10.97.245.122   80      5h7m
gateway-http      <none>   gateway.k8s.com                                  10.97.245.122   80      21m
middleware-http   <none>   zipkin.k8s.com,sentinel.k8s.com,xxljob.k8s.com                   80      20s


# 查看详情
[root@k8s-master ingress]# kubectl describe ing middleware-http -n dev
Name:             middleware-http
Namespace:        dev
Address:          
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host              Path  Backends
  ----              ----  --------
  zipkin.k8s.com    
                    /   zipkin-server:9411 (10.244.169.158:9411,10.244.36.127:9411)
  sentinel.k8s.com  
                    /   sentinel-server:8858 (10.244.169.183:8858)
  xxljob.k8s.com    
                    /   xxl-job:8080 (10.244.107.249:8080,10.244.36.88:8080,10.244.36.91:8080)
Annotations:        kubernetes.io/ingress.class: nginx
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  CREATE  61s   nginx-ingress-controller  Ingress dev/middleware-http


# 测试
在本机添加域名解析
C:\Windows\System32\drivers\etc
在hosts文件添加域名解析(master主机的iP地址)
172.51.216.81 		zipkin.k8s.com
172.51.216.81 		sentinel.k8s.com
172.51.216.81 		xxljob.k8s.com


[root@k8s-master ingress]# kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx                        NodePort    10.97.245.122    <none>        80:31208/TCP,443:32099/TCP   56d
ingress-nginx-controller             NodePort    10.101.238.213   <none>        80:30743/TCP,443:31410/TCP   55d
ingress-nginx-controller-admission   ClusterIP   10.100.142.101   <none>        443/TCP                      55d   


# ingress地址
http://zipkin.k8s.com:31208/
http://sentinel.k8s.com:31208/
http://xxljob.k8s.com:31208/


# 访问地址：
# zipkin
http://172.51.216.81:30411/zipkin
#sentinel
http://172.51.216.81:30858/#/login
账户密码：sentinel/sentinel
# xxl-job
http://172.51.216.81:30880/xxl-job-admin
账户密码：admin/123456


# 测试地址
# zipkin
http://zipkin.k8s.com:31208/zipkin
#sentinel
http://sentinel.k8s.com:31208/#/login
账户密码：sentinel/sentinel
# xxl-job
http://xxljob.k8s.com:31208/xxl-job-admin
账户密码：admin/123456
```



### 4.HTTPS

**调用链监控（Zipkin）、流量控制（Sentinel）、分布式任务调度平台（XXL-JOB）、注册中心（eureka-server）、网关（msa-gateway）。**

**Https代理**

> **创建 all-https.yaml**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: all-https
  namespace: dev
spec:
  tls:
    - hosts:
        - zipkin.k8s.com
        - sentinel.k8s.com
        - xxljob.k8s.com
        - eureka.k8s.com
        - gateway.k8s.com
      secretName: tls-secret    # 指定密钥
  rules:
    - host: zipkin.k8s.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: zipkin-server
                port:
                  number: 9411
    - host: sentinel.k8s.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: sentinel-server
                port:
                  number: 8858
    - host: xxljob.k8s.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: xxl-job
                port:
                  number: 8080
    - host: eureka.k8s.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: msa-eureka
                port:
                  number: 10001
    - host: gateway.k8s.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: msa-gateway
                port:
                  number: 8888
```

```shell
# 创建
[root@k8s-master ingress]# kubectl apply -f all-https.yaml 
ingress.networking.k8s.io/all-https created


# 查看
[root@k8s-master ingress]# kubectl get ing -n dev
NAME              CLASS    HOSTS                                                        ADDRESS         PORTS     AGE
all-https         <none>   zipkin.k8s.com,sentinel.k8s.com,xxljob.k8s.com + 2 more...   10.97.245.122   80, 443   37s
eureka-http       <none>   eureka.k8s.com                                               10.97.245.122   80        17h
gateway-http      <none>   gateway.k8s.com                                              10.97.245.122   80        12h
middleware-http   <none>   zipkin.k8s.com,sentinel.k8s.com,xxljob.k8s.com               10.97.245.122   80        12h


# 查看详情
[root@k8s-master ingress]# kubectl describe ing all-https -n dev
Name:             all-https
Namespace:        dev
Address:          10.97.245.122
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
TLS:
  tls-secret terminates zipkin.k8s.com,sentinel.k8s.com,xxljob.k8s.com,eureka.k8s.com,gateway.k8s.com
Rules:
  Host              Path  Backends
  ----              ----  --------
  zipkin.k8s.com    
                    /   zipkin-server:9411 (10.244.169.158:9411,10.244.36.127:9411)
  sentinel.k8s.com  
                    /   sentinel-server:8858 (10.244.169.183:8858)
  xxljob.k8s.com    
                    /   xxl-job:8080 (10.244.107.249:8080,10.244.36.88:8080,10.244.36.91:8080)
  eureka.k8s.com    
                    /   msa-eureka:10001 (10.244.169.153:10001,10.244.169.155:10001,10.244.169.157:10001)
  gateway.k8s.com   
                    /   msa-gateway:8888 (10.244.107.207:8888,10.244.107.208:8888,10.244.169.151:8888)
Annotations:        <none>
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  CREATE  87s   nginx-ingress-controller  Ingress dev/all-https
  Normal  UPDATE  78s   nginx-ingress-controller  Ingress dev/all-https


# 测试
在本机添加域名解析
C:\Windows\System32\drivers\etc
在hosts文件添加域名解析(master主机的iP地址)
172.51.216.81 		zipkin.k8s.com
172.51.216.81 		sentinel.k8s.com
172.51.216.81 		xxljob.k8s.com
172.51.216.81       eureka.k8s.com
172.51.216.81       gateway.k8s.com


[root@k8s-master ingress]# kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx                        NodePort    10.97.245.122    <none>        80:31208/TCP,443:32099/TCP   56d
ingress-nginx-controller             NodePort    10.101.238.213   <none>        80:30743/TCP,443:31410/TCP   55d
ingress-nginx-controller-admission   ClusterIP   10.100.142.101   <none>        443/TCP                      55d   


# ingress地址
# 在本机浏览器输入:
https://zipkin.k8s.com:32099/
https://sentinel.k8s.com:32099/
https://xxljob.k8s.com:32099/
https://eureka.k8s.com:32099/
https://gateway.k8s.com:32099/


# 访问地址：
# zipkin
http://172.51.216.81:30411/zipkin
#sentinel
http://172.51.216.81:30858/#/login
账户密码：sentinel/sentinel
# xxl-job
http://172.51.216.81:30880/xxl-job-admin
账户密码：admin/123456


# 测试地址
# zipkin
https://zipkin.k8s.com:32099/zipkin
#sentinel
https://sentinel.k8s.com:32099/#/login
账户密码：sentinel/sentinel
# xxl-job
https://xxljob.k8s.com:32099/xxl-job-admin
账户密码：admin/123456
# eureka
https://eureka.k8s.com:32099/

# gateway
https://gateway.k8s.com:32099/consumer/hello
https://gateway.k8s.com:32099/consumer/say
https://gateway.k8s.com:32099/producer/hello
https://gateway.k8s.com:32099/producer/say
```



