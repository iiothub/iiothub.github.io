* TOC
{:toc}


### 1.环境规划

- **微服务：**k8s部署微服务，名字空间：pro；
- **中间件：**中间件部署在k8s外部，PostgreSQL、MySQL、Redis、RabbitMQ，名字空间：ext；
- **系统监控：**k8s部署Prometheus，Elastic Stack部署在k8s外部；
- **镜像仓库：**Harbor作为私有镜像仓库，创建springcloud项目；



### 2.准备工作

#### 2.1.创建名字空间

```shell
[root@k8s-master1 pro]# kubectl create ns pro
namespace/pro created

[root@k8s-master1 pro]# kubectl create ns ext
namespace/ext created


[root@k8s-master1 pro]# kubectl get ns
NAME                   STATUS   AGE
......
ext                    Active   17s
pro                    Active   56m
```



#### 2.2.创建私有镜像密钥Secret

K8S所有工作节点登陆Harbor镜像仓库

```shell
# 登陆 admin/admin
# docker login http://172.51.216.88:8888


[root@k8s-node1 ~]# docker login http://172.51.216.88:8888
Username: admin
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

提示登录成功。

登录过程创建或更新一个包含授权令牌的config.json文件。
查看config.json文件：

```shell
cat ~/.docker/config.json
```

输出包含类似以下内容的部分：

```shell
[root@k8s-master1 ~]# cat ~/.docker/config.json
{
	"auths": {
		"172.51.216.88:8888": {
			"auth": "YWRtaW46YWRtaW4="
		}
	}
}
```

注意：如果您使用Docker凭据存储，您将看不到该auth条目，而是看到一个以存储名称为值的credsstore条目。

**在master节点操作**

```shell
# 对秘钥文件进行base64加密
cat ~/.docker/config.json |base64 -w 0


[root@k8s-master harbor]# cat ~/.docker/config.json |base64 -w 0
ewoJImF1dGhzIjogewoJCSIxNzIuNTEuMjE2Ljg4Ojg4ODgiOiB7CgkJCSJhdXRoIjogIllXUnRhVzQ2WVdSdGFXND0iCgkJfQoJfQp9
```



**k8s创建secret秘钥**

**注意：必须在相同的命名空间，默认在default**

```yaml
# secret
harbor-secret.yaml

apiVersion: v1
kind: Secret
metadata:
  name: harborsecret
  namespace: pro
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: ewoJImF1dGhzIjogewoJCSIxNzIuNTEuMjE2Ljg4Ojg4ODgiOiB7CgkJCSJhdXRoIjogIllXUnRhVzQ2WVdSdGFXND0iCgkJfQoJfQp9



[root@k8s-master1 pro]# kubectl apply -f harbor-secret.yaml 
secret/harborsecret created

[root@k8s-master1 pro]# kubectl get secret -n pro
NAME                  TYPE                                  DATA   AGE
default-token-8ddb4   kubernetes.io/service-account-token   3      19m
harborsecret          kubernetes.io/dockerconfigjson        1      48s
```



### 3.微服务配置

#### 3.1.注册中心（eureka-server）

 msa-eureka.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.eureka.name }}
  labels:
    app: {{ .Values.eureka.labels }}
spec:
  type: {{ .Values.eureka.service.type }}
  ports:
    - port: {{ .Values.eureka.service.port }}
      name: {{ .Values.eureka.name }}
      targetPort: {{ .Values.eureka.service.targetPort }}
  selector:
    app: {{ .Values.eureka.labels }}

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.eureka.name }}
spec:
  serviceName: {{ .Values.eureka.name }}
  replicas: {{ .Values.eureka.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.eureka.labels }}
  template:
    metadata:
      labels:
        app: {{ .Values.eureka.labels }}
    spec:
      imagePullSecrets:
        - name: {{ .Values.global.imagePullSecrets }}
      containers:
        - name: {{ .Values.eureka.image.name }}
          imagePullPolicy: {{ .Values.eureka.image.pullPolicy }}
          image: {{ .Values.global.repository }}{{ .Values.eureka.image.name }}:{{ .Values.eureka.image.tag }}
          ports:
          - containerPort: {{ .Values.eureka.image.containerPort }}
          env:
            - name: MY_POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: EUREKA_SERVER
              value: {{ .Values.global.env.eureka }}
            - name: EUREKA_INSTANCE_HOSTNAME
              value: ${MY_POD_NAME}.msa-eureka.dev
            - name: ACTIVE
              value: {{ .Values.global.env.profiles }}
  podManagementPolicy: "Parallel"
```



#### 3.2.网关（msa-gateway）

 msa-gateway.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.gateway.name }}
  labels:
    app: {{ .Values.gateway.labels }}
spec:
  type: {{ .Values.gateway.service.type }}
  ports:
    - port: {{ .Values.gateway.service.port }}
      targetPort: {{ .Values.gateway.service.targetPort }}
  selector:
    app: {{ .Values.gateway.labels }}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.gateway.name }}
spec:
  replicas: {{ .Values.gateway.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.gateway.labels }}
  template:
    metadata:
      labels:
        app: {{ .Values.gateway.labels }}
    spec:
      imagePullSecrets:
        - name: {{ .Values.global.imagePullSecrets }}
      containers:
        - name: {{ .Values.gateway.image.name }}
          image: {{ .Values.global.repository }}{{ .Values.gateway.image.name }}:{{ .Values.gateway.image.tag }}
          imagePullPolicy: {{ .Values.gateway.image.pullPolicy }}
          ports:
          - containerPort: {{ .Values.gateway.image.containerPort }}
          env:
            - name: EUREKA_SERVER
              value:  {{ .Values.global.env.eureka }}
            - name: ACTIVE
              value: {{ .Values.global.env.profiles }}
```



#### 3.3.流量控制（Sentinel）

sentinel-server.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.sentinel.name }}
  labels:
    app: {{ .Values.sentinel.labels }}
spec:
  type: {{ .Values.sentinel.service.type }}
  ports:
    - port: {{ .Values.sentinel.service.port }}
      targetPort: {{ .Values.sentinel.service.targetPort }}
  selector:
    app: {{ .Values.sentinel.labels }}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.sentinel.name }}
spec:
  replicas: {{ .Values.sentinel.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.sentinel.labels }}
  template:
    metadata:
      labels:
        app: {{ .Values.sentinel.labels }}
    spec:
      containers:
        - name: {{ .Values.sentinel.image.name }}
          image: bladex/sentinel-dashboard:{{ .Values.zipkin.image.tag }}
          imagePullPolicy: {{ .Values.sentinel.image.pullPolicy }}
          ports:
          - containerPort: {{ .Values.sentinel.image.containerPort }}
```



#### 3.4.调用链监控（Zipkin）

zipkin-server.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.zipkin.name }}
  labels:
    app: {{ .Values.zipkin.labels }}
spec:
  type: {{ .Values.zipkin.service.type }}
  ports:
    - port: {{ .Values.zipkin.service.port }}
      targetPort: {{ .Values.zipkin.service.targetPort }}
  selector:
    app: {{ .Values.zipkin.labels }}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.zipkin.name }}
spec:
  replicas: {{ .Values.zipkin.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.zipkin.labels }}
  template:
    metadata:
      labels:
        app: {{ .Values.zipkin.labels }}
    spec:
      containers:
        - name: {{ .Values.zipkin.image.name }}
          image: openzipkin/zipkin:{{ .Values.zipkin.image.tag }}
          imagePullPolicy: {{ .Values.zipkin.image.pullPolicy }}
          ports:
          - containerPort: {{ .Values.zipkin.image.containerPort }}
          env:
            - name: RABBIT_ADDRESSES
              value: {{ .Values.zipkin.env.addresses }}
            - name: RABBIT_USER
              value: {{ .Values.zipkin.env.user }}
            - name: RABBIT_PASSWORD
              value: {{ .Values.zipkin.env.password }}
            - name: STORAGE_TYPE
              value: {{ .Values.zipkin.env.storage }}
            - name: ES_HOSTS
              value: {{ .Values.zipkin.env.eshosts }}
```



#### 3.5.分布式任务调度平台（XXL-JOB）

xxl-job.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.xxljob.name }}
  labels:
    app: {{ .Values.xxljob.labels }}
spec:
  type: {{ .Values.xxljob.service.type }}
  ports:
    - port: {{ .Values.xxljob.service.port }}
      targetPort: {{ .Values.xxljob.service.targetPort }}
  selector:
    app: {{ .Values.xxljob.labels }}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.xxljob.name }}
spec:
  replicas: {{ .Values.xxljob.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.xxljob.labels }}
  template:
    metadata:
      labels:
        app: {{ .Values.xxljob.labels }}
    spec:
      containers:
        - name: {{ .Values.xxljob.image.name }}
          image: xuxueli/xxl-job-admin:{{ .Values.xxljob.image.tag }}
          imagePullPolicy: {{ .Values.xxljob.image.pullPolicy }}
          ports:
          - containerPort: {{ .Values.xxljob.image.containerPort }}
          env:
            - name: PARAMS
              value:  {{ .Values.xxljob.env.params }}
```



#### 3.6.XXL-JOB执行器（msa-deploy-job）

msa-deploy-job.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.job.name }}
  labels:
    app: {{ .Values.job.labels }}
spec:
  type: {{ .Values.job.service.type }}
  ports:
    - port: {{ .Values.job.service.port }}
      targetPort: {{ .Values.job.service.targetPort }}
  selector:
    app: {{ .Values.job.labels }}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.job.name }}
spec:
  replicas: {{ .Values.job.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.job.labels }}
  template:
    metadata:
      labels:
        app: {{ .Values.job.labels }}
    spec:
      imagePullSecrets:
        - name: {{ .Values.global.imagePullSecrets }}
      containers:
        - name: {{ .Values.job.image.name }}
          image: {{ .Values.global.repository }}{{ .Values.job.image.name }}:{{ .Values.job.image.tag }}
          imagePullPolicy: {{ .Values.job.image.pullPolicy }}
          ports:
          - containerPort: {{ .Values.job.image.containerPort }}
          env:
            - name: ACTIVE
              value: {{ .Values.global.env.profiles }}
```



#### 3.7.生产者（msa-deploy-producer）

msa-deploy-producer.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.producer.name }}
  labels:
    app: {{ .Values.producer.labels }}
spec:
  type: {{ .Values.producer.service.type }}
  ports:
    - port: {{ .Values.producer.service.port }}
      targetPort: {{ .Values.producer.service.targetPort }}
  selector:
    app: {{ .Values.producer.labels }}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.producer.name }}
spec:
  replicas: {{ .Values.producer.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.producer.labels }}
  template:
    metadata:
      labels:
        app: {{ .Values.producer.labels }}
    spec:
      imagePullSecrets:
        - name: {{ .Values.global.imagePullSecrets }}
      containers:
        - name: {{ .Values.producer.image.name }}
          image: {{ .Values.global.repository }}{{ .Values.producer.image.name }}:{{ .Values.producer.image.tag }}
          imagePullPolicy: {{ .Values.producer.image.pullPolicy }}
          ports:
          - containerPort: {{ .Values.producer.image.containerPort }}
          env:
            - name: EUREKA_SERVER
              value:  {{ .Values.global.env.eureka }}
            - name: ACTIVE
              value: {{ .Values.global.env.profiles }}
            - name: DASHBOARD
              value: {{ .Values.producer.env.dashboard }}
```



#### 3.8.消费者（msa-deploy-consumer）

msa-deploy-consumer.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.consumer.name }}
  labels:
    app: {{ .Values.consumer.labels }}
spec:
  type: {{ .Values.consumer.service.type }}
  ports:
    - port: {{ .Values.consumer.service.port }}
      targetPort: {{ .Values.consumer.service.targetPort }}
  selector:
    app: {{ .Values.consumer.labels }}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.consumer.name }}
spec:
  replicas: {{ .Values.consumer.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.consumer.labels }}
  template:
    metadata:
      labels:
        app: {{ .Values.consumer.labels }}
    spec:
      imagePullSecrets:
        - name: {{ .Values.global.imagePullSecrets }}
      containers:
        - name: {{ .Values.consumer.image.name }}
          image: {{ .Values.global.repository }}{{ .Values.consumer.image.name }}:{{ .Values.consumer.image.tag }}
          imagePullPolicy: {{ .Values.consumer.image.pullPolicy }}
          ports:
          - containerPort: {{ .Values.consumer.image.containerPort }}
          env:
            - name: EUREKA_SERVER
              value:  {{ .Values.global.env.eureka }}
            - name: ACTIVE
              value: {{ .Values.global.env.profiles }}
            - name: DASHBOARD
              value: {{ .Values.consumer.env.dashboard }}
```



#### 3.9.Ingress

**调用链监控（Zipkin）、流量控制（Sentinel）、分布式任务调度平台（XXL-JOB）、注册中心（eureka-server）、网关（msa-gateway）。**



**Http代理**

> **创建 msa-pro-http.yaml**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: msa-http
  namespace: pro
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
    - host: zipkin.pro.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: zipkin-server
                port:
                  number: 9411
    - host: sentinel.pro.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: sentinel-server
                port:
                  number: 8858
    - host: xxljob.pro.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: xxl-job
                port:
                  number: 8080
    - host: eureka.pro.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: msa-eureka
                port:
                  number: 10001
    - host: gateway.pro.com
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



#### 3.10.values

values.yaml 

```yaml
global:
  namespace: "pro"
  imagePullSecrets: "harborsecret"
  repository: "172.51.216.88:8888/springcloud/"
  env:
    eureka: "http://msa-eureka-0.msa-eureka.pro:10001/eureka/,http://msa-eureka-1.msa-eureka.pro:10001/eureka/,http://msa-eureka-2.msa-eureka.pro:10001/eureka/"
    profiles: "-Dspring.profiles.active=k8s-pro"


eureka:
  name: "msa-eureka"
  labels: "msa-eureka"
  service:
    type: "ClusterIP"
    port: 10001
    targetPort: 10001
  replicaCount: 3
  image:
    name: "msa-eureka"
    pullPolicy: "IfNotPresent"
    tag: "1.0.0"
    containerPort: 10001


gateway:
  name: "msa-gateway"
  labels: "msa-gateway"
  service:
    type: "ClusterIP"
    port: 8888
    targetPort: 8888
  replicaCount: 3
  image:
    name: "msa-gateway"
    pullPolicy: "IfNotPresent"
    tag: "1.0.0"
    containerPort: 8888


sentinel:
  name: "sentinel-server"
  labels: "sentinel-server"
  service:
    type: "ClusterIP"
    port: 8858
    targetPort: 8858
  replicaCount: 1
  image:
    name: "sentinel-server"
    pullPolicy: "IfNotPresent"
    tag: "1.7.2"
    containerPort: 8858


zipkin:
  name: "zipkin-server"
  labels: "zipkin-server"
  service:
    type: "ClusterIP"
    port: 9411
    targetPort: 9411
  replicaCount: 3
  image:
    name: "zipkin-server"
    pullPolicy: "IfNotPresent"
    tag: "latest"
    containerPort: 9411
  env:
    addresses: "ext-rabbitmq.ext.svc.cluster.local:5672"
    user: "guest"
    password: "guest"
    storage: "elasticsearch"
    eshosts: "ext-elasticsearch.ext.svc.cluster.local:9200"


xxljob:
  name: "xxl-job"
  labels: "xxl-job"
  service:
    type: "ClusterIP"
    port: 8080
    targetPort: 8080
  replicaCount: 3
  image:
    name: "xxl-job"
    pullPolicy: "IfNotPresent"
    tag: "2.3.0"
    containerPort: 8080
  env:
    params: "--spring.datasource.username=root --spring.datasource.password=root --spring.datasource.url=jdbc:mysql://ext-mysql.ext.svc.cluster.local:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai"


job:
  name: "msa-deploy-job"
  labels: "msa-deploy-job"
  service:
    type: "ClusterIP"
    port: 8913
    targetPort: 8913
  replicaCount: 3
  image:
    name: "msa-deploy-job"
    pullPolicy: "IfNotPresent"
    tag: "1.0.0"
    containerPort: 8913


producer:
  name: "msa-deploy-producer"
  labels: "msa-deploy-producer"
  service:
    type: "ClusterIP"
    port: 8911
    targetPort: 8911
  replicaCount: 3
  image:
    name: "msa-deploy-producer"
    pullPolicy: "IfNotPresent"
    tag: "1.0.0"
    containerPort: 8911
  env:
    dashboard: "sentinel-server.pro.svc.cluster.local:8858"


consumer:
  name: "msa-deploy-consumer"
  labels: "msa-deploy-consumer"
  service:
    type: "ClusterIP"
    port: 8912
    targetPort: 8912
  replicaCount: 3
  image:
    name: "msa-deploy-consumer"
    pullPolicy: "IfNotPresent"
    tag: "1.0.0"
    containerPort: 8912
  env:
    dashboard: "sentinel-server.pro.svc.cluster.local:8858"
```



### 4.部署服务

#### 4.1.上传微服务镜像

**上传镜像的微服务包括：**

- 注册中心（eureka-server）
- 网关（msa-gateway）
- XXL-JOB执行器（msa-deploy-job）
- 生产者（msa-deploy-producer）
- 消费者（msa-deploy-consumer）



**1.创建Dockerfile**

```shell
FROM openjdk:8-jdk-alpine

VOLUME /temp

ENV JVM_OPS="-Xms256m -Xmx256m -XX:PermSize=512M -XX:MaxPermSize=512m"

ENV JAVA_POS=""

ENV ACTIVE="-Dspring.profiles.active=test"

ADD *.jar app.jar

ENTRYPOINT java -Djava.security.egd=file:/dev/./urandom -jar ${JVM_OPS} ${ACTIVE} app.jar ${JAVA_OPS}

ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```



**2.创建、上传镜像**

```shell
# 1.创建镜像
docker build -t 172.51.216.88:8888/springcloud/msa-xxx:1.0.0 .


# 2.推送镜像
docker push 172.51.216.88:8888/springcloud/msa-xxx:1.0.0
```

```shell
# msa-eureka
docker build -t 172.51.216.88:8888/springcloud/msa-eureka:1.0.0 .
docker push 172.51.216.88:8888/springcloud/msa-eureka:1.0.0


# msa-gateway
docker build -t 172.51.216.88:8888/springcloud/msa-gateway:1.0.0 .
docker push 172.51.216.88:8888/springcloud/msa-gateway:1.0.0


# msa-deploy-job
docker build -t 172.51.216.88:8888/springcloud/msa-deploy-job:1.0.0 .
docker push 172.51.216.88:8888/springcloud/msa-deploy-job:1.0.0


# msa-deploy-producer
docker build -t 172.51.216.88:8888/springcloud/msa-deploy-producer:1.0.0 .
docker push 172.51.216.88:8888/springcloud/msa-deploy-producer:1.0.0


# msa-deploy-consumer
docker build -t 172.51.216.88:8888/springcloud/msa-deploy-consumer:1.0.0 .
docker push 172.51.216.88:8888/springcloud/msa-deploy-consumer:1.0.0
```



#### 4.2.构建Helm Chart

```shell
# 创建 cloud
[root@k8s-master1 pro]# helm create cloud -n pro
Creating cloud


# Chart.yaml
[root@k8s-master cloud]# vim Chart.yaml

apiVersion: v2
name: cloud
description: A Helm chart for Kubernetes
type: application
version: 1.0.0
appVersion: "1.0.1"


# 创建文件
msa-eureka.yaml
msa-gateway.yaml
sentinel-server.yaml
zipkin-server.yaml
xxl-job.yaml
msa-deploy-job.yaml
msa-deploy-producer.yaml
msa-deploy-consumer.yaml
msa-pro-http.yaml
values.yaml



# 创建后的文件结构
[root@k8s-master1 cloud]# pwd
/k8s/springcloud/pro/cloud
[root@k8s-master1 cloud]# tree
.
├── charts
├── Chart.yaml
├── templates
│   ├── msa-deploy-consumer.yaml
│   ├── msa-deploy-job.yaml
│   ├── msa-deploy-producer.yaml
│   ├── msa-eureka.yaml
│   ├── msa-gateway.yaml
│   ├── msa-pro-http.yaml
│   ├── sentinel-server.yaml
│   ├── xxl-job.yaml
│   └── zipkin-server.yaml
└── values.yaml

2 directories, 11 files
```



#### 4.3.Helm安装

```shell
# 调试
# Helm也提供了--dry-run --debug调试参数，帮助你验证模板正确性。在执行helm install时候带上这两个参数就可以把对应的values值和渲染的资源清单打印出来，而不会真正的去部署一个release。
[root@k8s-master1 pro]# helm install msa cloud --dry-run --debug -n pro



# 安装部署
[root@k8s-master1 pro]# helm install msa cloud -n pro
NAME: msa
LAST DEPLOYED: Mon Dec 20 15:52:15 2021
NAMESPACE: pro
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

```shell
[root@k8s-master1 pro]# helm list -n pro
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	pro      	1       	2021-12-20 16:23:59.606220862 +0800 CST	deployed	cloud-1.0.0	1.0.1 



# 检索已经发布的 release 的资源文件
[root@k8s-master1 pro]# helm get manifest msa -n pro


# 查看
[root@k8s-master1 pro]# kubectl get all -n pro
NAME                                       READY   STATUS    RESTARTS   AGE
pod/msa-deploy-consumer-b854cf678-4r56d    1/1     Running   0          3m52s
pod/msa-deploy-consumer-b854cf678-mlm6t    1/1     Running   0          3m52s
pod/msa-deploy-consumer-b854cf678-rjv9s    1/1     Running   0          3m52s
pod/msa-deploy-job-d5574c7db-hfh8q         1/1     Running   0          3m52s
pod/msa-deploy-job-d5574c7db-hfm8v         1/1     Running   0          3m52s
pod/msa-deploy-job-d5574c7db-hr5n6         1/1     Running   0          3m52s
pod/msa-deploy-producer-6d7bc67c89-6698w   1/1     Running   0          3m52s
pod/msa-deploy-producer-6d7bc67c89-j97dq   1/1     Running   0          3m52s
pod/msa-deploy-producer-6d7bc67c89-p8l5t   1/1     Running   0          3m52s
pod/msa-eureka-0                           1/1     Running   0          3m52s
pod/msa-eureka-1                           1/1     Running   0          3m52s
pod/msa-eureka-2                           1/1     Running   0          3m52s
pod/msa-gateway-8444759846-6txqr           1/1     Running   0          3m52s
pod/msa-gateway-8444759846-gxxjl           1/1     Running   0          3m52s
pod/msa-gateway-8444759846-qxnwc           1/1     Running   0          3m52s
pod/sentinel-server-8597dd6fcc-nbhnl       1/1     Running   0          3m52s
pod/xxl-job-7d8cdb957f-gtphp               1/1     Running   0          3m52s
pod/xxl-job-7d8cdb957f-t6hxd               1/1     Running   0          3m52s
pod/xxl-job-7d8cdb957f-t6qzh               1/1     Running   0          3m52s
pod/zipkin-server-c856888b8-gv7n7          1/1     Running   0          3m52s
pod/zipkin-server-c856888b8-phplw          1/1     Running   0          3m52s
pod/zipkin-server-c856888b8-qncrq          1/1     Running   0          3m52s

NAME                          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
service/msa-deploy-consumer   ClusterIP   10.1.113.152   <none>        8912/TCP    3m53s
service/msa-deploy-job        ClusterIP   10.1.112.110   <none>        8913/TCP    3m53s
service/msa-deploy-producer   ClusterIP   10.1.181.87    <none>        8911/TCP    3m53s
service/msa-eureka            ClusterIP   10.1.98.13     <none>        10001/TCP   3m53s
service/msa-gateway           ClusterIP   10.1.106.62    <none>        8888/TCP    3m53s
service/sentinel-server       ClusterIP   10.1.153.24    <none>        8858/TCP    3m53s
service/xxl-job               ClusterIP   10.1.40.100    <none>        8080/TCP    3m53s
service/zipkin-server         ClusterIP   10.1.151.230   <none>        9411/TCP    3m53s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/msa-deploy-consumer   3/3     3            3           3m53s
deployment.apps/msa-deploy-job        3/3     3            3           3m53s
deployment.apps/msa-deploy-producer   3/3     3            3           3m53s
deployment.apps/msa-gateway           3/3     3            3           3m53s
deployment.apps/sentinel-server       1/1     1            1           3m53s
deployment.apps/xxl-job               3/3     3            3           3m53s
deployment.apps/zipkin-server         3/3     3            3           3m53s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/msa-deploy-consumer-b854cf678    3         3         3       3m53s
replicaset.apps/msa-deploy-job-d5574c7db         3         3         3       3m52s
replicaset.apps/msa-deploy-producer-6d7bc67c89   3         3         3       3m53s
replicaset.apps/msa-gateway-8444759846           3         3         3       3m53s
replicaset.apps/sentinel-server-8597dd6fcc       1         1         1       3m53s
replicaset.apps/xxl-job-7d8cdb957f               3         3         3       3m52s
replicaset.apps/zipkin-server-c856888b8          3         3         3       3m53s

NAME                          READY   AGE
statefulset.apps/msa-eureka   3/3     3m53s
```



#### 4.4.安装helm-push插件

Helm服务器安装helm-push插件

```shell
# master(172.51.216.81，172.51.216.82，172.51.216.83)，安装helm服务器

# 安装
helm plugin install https://github.com/chartmuseum/helm-push 
# 查看已成功
helm plugin list



# 安装
[root@k8s-master ~]# helm plugin install https://github.com/chartmuseum/helm-push
Downloading and installing helm-push v0.10.1 ...
https://github.com/chartmuseum/helm-push/releases/download/v0.10.1/helm-push_0.10.1_linux_amd64.tar.gz
Installed plugin: cm-push


# 查看已成功
[root@k8s-master helm]# helm plugin list
NAME   	VERSION	DESCRIPTION                      
cm-push	0.10.1 	Push chart package to ChartMuseum
```

**本地安装**

```shell
# 下载 helm-push_0.10.1_linux_amd64.tar.gz
https://github.com/chartmuseum/helm-push/releases/download/v0.10.1/helm-push_0.10.1_linux_amd64.tar.gz


# 解压
[root@k8s-master2 helm]# tar -zxf helm-push_0.10.1_linux_amd64.tar.gz 
[root@k8s-master2 helm]# ll
total 23636
drwxr-xr-x. 2 root root         26 Dec 23 10:51 bin
-rw-r--r--. 1 root root   10481411 Dec 23 10:51 helm-push_0.10.1_linux_amd64.tar.gz
-rw-r--r--. 1 1001 docker    11357 Oct 12 22:09 LICENSE
-rw-r--r--. 1 1001 docker      407 Oct 12 22:09 plugin.yaml


# 安装
[root@k8s-master2 helm]# helm plugin install .
sh: scripts/install_plugin.sh: No such file or directory
Error: plugin install hook for "cm-push" exited with error


# 测试
[root@k8s-master2 helm]# helm cm-push --help
Helm plugin to push chart package to ChartMuseum

Examples:

  $ helm cm-push mychart-0.1.0.tgz chartmuseum       # push .tgz from "helm package"
  $ helm cm-push . chartmuseum                       # package and push chart directory
  $ helm cm-push . --version="1.2.3" chartmuseum     # override version in Chart.yaml
  $ helm cm-push . https://my.chart.repo.com         # push directly to chart repo URL

Usage:
  helm cm-push [flags]

Flags:
      --access-token string             Send token in Authorization header [$HELM_REPO_ACCESS_TOKEN]
  -a, --app-version string              Override app version pre-push
      --auth-header string              Alternative header to use for token auth [$HELM_REPO_AUTH_HEADER]
      --ca-file string                  Verify certificates of HTTPS-enabled servers using this CA bundle [$HELM_REPO_CA_FILE]
      --cert-file string                Identify HTTPS client using this SSL certificate file [$HELM_REPO_CERT_FILE]
      --check-helm-version              outputs either "2" or "3" indicating the current Helm major version
      --context-path string             ChartMuseum context path [$HELM_REPO_CONTEXT_PATH]
      --debug                           Enable verbose output
  -d, --dependency-update               update dependencies from "requirements.yaml" to dir "charts/" before packaging
  -f, --force                           Force upload even if chart version exists
  -h, --help                            help for helm
      --home string                     Location of your Helm config. Overrides $HELM_HOME (default "/root/.helm")
      --host string                     Address of Tiller. Overrides $HELM_HOST
      --insecure                        Connect to server with an insecure way by skipping certificate verification [$HELM_REPO_INSECURE]
      --key-file string                 Identify HTTPS client using this SSL key file [$HELM_REPO_KEY_FILE]
      --keyring string                  location of a public keyring (default "/root/.gnupg/pubring.gpg")
      --kube-context string             Name of the kubeconfig context to use
      --kubeconfig string               Absolute path of the kubeconfig file to be used
  -p, --password string                 Override HTTP basic auth password [$HELM_REPO_PASSWORD]
      --tiller-connection-timeout int   The duration (in seconds) Helm will wait to establish a connection to Tiller (default 300)
      --tiller-namespace string         Namespace of Tiller (default "kube-system")
  -t, --timeout int                     The duration (in seconds) Helm will wait to get response from chartmuseum (default 30)
  -u, --username string                 Override HTTP basic auth username [$HELM_REPO_USERNAME]
  -v, --version string                  Override chart version pre-push
```



### 5.系统测试

```shell
# 测试
在本机添加域名解析
C:\Windows\System32\drivers\etc
在hosts文件添加域名解析(master主机的iP地址)
172.51.216.81 		zipkin.pro.com
172.51.216.81 		sentinel.pro.com
172.51.216.81 		xxljob.pro.com
172.51.216.81       eureka.pro.com
172.51.216.81       gateway.pro.com


[root@k8s-master pro]#  kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.106.196.185   <none>        80:31356/TCP,443:31221/TCP   2d21h
ingress-nginx-controller-admission   ClusterIP   10.103.157.181   <none>        443/TCP                      2d21h



# ingress地址
# 在本机浏览器输入:
http://zipkin.pro.com:31356/
http://sentinel.pro.com:31356/
http://xxljob.pro.com:31356/
http://eureka.pro.com:31356/
http://gateway.pro.com:31356/



# 测试地址
# zipkin
http://zipkin.pro.com:31356/zipkin
#sentinel
http://sentinel.pro.com:31356/#/login
账户密码：sentinel/sentinel
# xxl-job
http://xxljob.pro.com:31356/xxl-job-admin
账户密码：admin/123456
# eureka
http://eureka.pro.com:31356/

# gateway
http://gateway.pro.com:31356/dconsumer/hello
http://gateway.pro.com:31356/dconsumer/say
http://gateway.pro.com:31356/dproducer/hello
http://gateway.pro.com:31356/dproducer/say
```

```shell
# 测试分布式任务调度平台（XXL-JOB）

# 配置服务端
# 参考文档：K8S-SpringCloud---外部服务.md    2.5.部署XXL-JOB执行器（msa-ext-job）


# 查看日志
[root@k8s-master1 pro]# kubectl logs -f --tail=10 msa-deploy-job-d5574c7db-hfh8q -n pro
----job one 1---Mon Dec 20 16:44:07 CST 2021
----job one 1---Mon Dec 20 16:44:08 CST 2021
----job one 1---Mon Dec 20 16:44:09 CST 2021
----job one 1---Mon Dec 20 16:44:10 CST 2021
----job one 1---Mon Dec 20 16:44:11 CST 2021
----job one 1---Mon Dec 20 16:44:12 CST 2021
----job one 1---Mon Dec 20 16:44:13 CST 2021
----job one 1---Mon Dec 20 16:44:14 CST 2021
----job one 1---Mon Dec 20 16:44:15 CST 2021
----job one 1---Mon Dec 20 16:44:16 CST 2021
----job one 1---Mon Dec 20 16:44:17 CST 2021
----job one 1---Mon Dec 20 16:44:18 CST 2021
----job one 1---Mon Dec 20 16:44:19 CST 2021
```



### 6.Helm测试

#### 6.1.添加仓库

```shell
# 添加仓库


helm repo add --ca-file <ca file> --cert-file <cert file> --key-file <key file>     --username <username> --password <password> <repo name> http://172.51.216.88:8888/chartrepo/springcloud
```

```shell
[root@k8s-master pro]# helm repo add --username admin --password admin harborcloud http://172.51.216.88:8888/chartrepo/springcloud
"harborcloud" has been added to your repositories


[root@k8s-master2 harbor]# helm repo list
NAME         	URL                                                   
stable       	http://mirror.azure.cn/kubernetes/charts              
aliyun       	https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
bitnami      	https://charts.bitnami.com/bitnami                    
ingress-nginx	https://kubernetes.github.io/ingress-nginx            
elastic      	https://helm.elastic.co                               
harborcloud  	http://172.51.216.88:8888/chartrepo/springcloud


[root@k8s-master2 harbor]# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "harborcloud" chart repository
...Successfully got an update from the "ingress-nginx" chart repository
...Successfully got an update from the "aliyun" chart repository
...Successfully got an update from the "elastic" chart repository
...Successfully got an update from the "bitnami" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈
```



#### 6.2.Helm Chart上传

```shell
# 参考 
 
  $ helm cm-push mychart-0.1.0.tgz chartmuseum       # push .tgz from "helm package"
  $ helm cm-push . chartmuseum                       # package and push chart directory
  $ helm cm-push . --version="1.2.3" chartmuseum     # override version in Chart.yaml
  $ helm cm-push . https://my.chart.repo.com         # push directly to chart repo URL
```

```shell
# 查看基本信息

[root@k8s-master2 harbor]# vim cloud/Chart.yaml 

apiVersion: v2
# 名称
name: cloud

description: A Helm chart for Kubernetes
type: application

# 版本
version: 1.0.0

appVersion: "1.0.1"



[root@k8s-master2 harbor]# tree
.
└── cloud
    ├── charts
    ├── Chart.yaml
    ├── templates
    │   ├── msa-deploy-consumer.yaml
    │   ├── msa-deploy-job.yaml
    │   ├── msa-deploy-producer.yaml
    │   ├── msa-eureka.yaml
    │   ├── msa-gateway.yaml
    │   ├── msa-test-http.yaml
    │   ├── sentinel-server.yaml
    │   ├── xxl-job.yaml
    │   └── zipkin-server.yaml
    └── values.yaml

3 directories, 11 files
```

```shell
# 上传
[root@k8s-master2 harbor]# helm cm-push cloud/ harborcloud
Pushing cloud-1.0.0.tgz to harborcloud...
Done.


# 更新
[root@k8s-master pro]# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "harborcloud" chart repository
...Successfully got an update from the "elastic" chart repository
...Successfully got an update from the "aliyun" chart repository
...Successfully got an update from the "ingress-nginx" chart repository
...Successfully got an update from the "presslabs" chart repository
...Successfully got an update from the "stable" chart repository
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈


# 查询
[root@k8s-master pro]# helm search repo harborcloud/cloud
NAME             	CHART VERSION	APP VERSION	DESCRIPTION                
harborcloud/cloud	1.0.0        	1.0.1      	A Helm chart for Kubernetes


# 获取
[root@k8s-master pro]# helm fetch harborcloud/cloud
[root@k8s-master pro]# ll
total 8
drwxr-xr-x 4 root root   93 Dec 27 10:56 cloud
-rw-r--r-- 1 root root 2952 Dec 27 11:07 cloud-1.0.0.tgz
```



#### 6.3.Helm Chart安装

```shell
# 安装chart


helm install --ca-file <ca file> --cert-file <cert file> --key-file <key file>     --username=<username> --password=<password> --version 1.0.0 <repo name>/cloud
```

```shell
# 远程安装
[root@k8s-master pro]# helm install msa harborcloud/cloud --version 1.0.0 -n pro
NAME: msa
LAST DEPLOYED: Mon Dec 27 11:13:40 2021
NAMESPACE: pro
STATUS: deployed
REVISION: 1
TEST SUITE: None


# 查看
[root@k8s-master pro]# helm list -n pro
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	pro      	1       	2021-12-27 11:13:40.243818519 +0800 CST	deployed	cloud-1.0.0	1.0.1 


[root@k8s-master pro]# kubectl get all -n pro
NAME                                       READY   STATUS    RESTARTS   AGE
pod/msa-deploy-consumer-b854cf678-br2lv    1/1     Running   0          53s
pod/msa-deploy-consumer-b854cf678-wxwhw    1/1     Running   0          53s
pod/msa-deploy-consumer-b854cf678-xk62t    1/1     Running   0          53s
pod/msa-deploy-job-d5574c7db-9rqww         1/1     Running   0          53s
pod/msa-deploy-job-d5574c7db-l85dh         1/1     Running   0          53s
pod/msa-deploy-job-d5574c7db-rzr24         1/1     Running   0          53s
pod/msa-deploy-producer-6d7bc67c89-h9vhs   1/1     Running   0          53s
pod/msa-deploy-producer-6d7bc67c89-pr5t2   1/1     Running   0          53s
pod/msa-deploy-producer-6d7bc67c89-z58nn   1/1     Running   0          53s
pod/msa-eureka-0                           1/1     Running   0          53s
pod/msa-eureka-1                           1/1     Running   0          53s
pod/msa-eureka-2                           1/1     Running   0          53s
pod/msa-gateway-8444759846-6ncwc           1/1     Running   0          53s
pod/msa-gateway-8444759846-8pbzn           1/1     Running   0          53s
pod/msa-gateway-8444759846-mpx72           1/1     Running   0          53s
pod/sentinel-server-8597dd6fcc-5mt9z       1/1     Running   0          53s
pod/xxl-job-7d8cdb957f-nvdsx               1/1     Running   0          53s
pod/xxl-job-7d8cdb957f-w58wm               1/1     Running   0          53s
pod/xxl-job-7d8cdb957f-wr958               1/1     Running   0          53s
pod/zipkin-server-c856888b8-42ddq          1/1     Running   0          53s
pod/zipkin-server-c856888b8-z9g4s          1/1     Running   0          53s
pod/zipkin-server-c856888b8-zdbx4          1/1     Running   0          53s

NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
service/msa-deploy-consumer   ClusterIP   10.99.248.228    <none>        8912/TCP    53s
service/msa-deploy-job        ClusterIP   10.106.240.144   <none>        8913/TCP    53s
service/msa-deploy-producer   ClusterIP   10.108.38.186    <none>        8911/TCP    53s
service/msa-eureka            ClusterIP   10.109.185.118   <none>        10001/TCP   53s
service/msa-gateway           ClusterIP   10.111.119.122   <none>        8888/TCP    53s
service/sentinel-server       ClusterIP   10.101.245.25    <none>        8858/TCP    53s
service/xxl-job               ClusterIP   10.111.177.143   <none>        8080/TCP    53s
service/zipkin-server         ClusterIP   10.96.60.12      <none>        9411/TCP    53s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/msa-deploy-consumer   3/3     3            3           53s
deployment.apps/msa-deploy-job        3/3     3            3           53s
deployment.apps/msa-deploy-producer   3/3     3            3           53s
deployment.apps/msa-gateway           3/3     3            3           53s
deployment.apps/sentinel-server       1/1     1            1           53s
deployment.apps/xxl-job               3/3     3            3           53s
deployment.apps/zipkin-server         3/3     3            3           53s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/msa-deploy-consumer-b854cf678    3         3         3       53s
replicaset.apps/msa-deploy-job-d5574c7db         3         3         3       53s
replicaset.apps/msa-deploy-producer-6d7bc67c89   3         3         3       53s
replicaset.apps/msa-gateway-8444759846           3         3         3       53s
replicaset.apps/sentinel-server-8597dd6fcc       1         1         1       53s
replicaset.apps/xxl-job-7d8cdb957f               3         3         3       53s
replicaset.apps/zipkin-server-c856888b8          3         3         3       53s

NAME                          READY   AGE
statefulset.apps/msa-eureka   3/3     53s
```



#### 6.4.Helm Chart升级

```shell
# 修改
# 分布式调度访问k8s内部mysql

[root@k8s-master2 cloud]# vim Chart.yaml 

apiVersion: v2
name: cloud
description: A Helm chart for Kubernetes
type: application

# 修改版本号
version: 2.0.0
appVersion: "2.0.1"


# 修改mysql链接: my-cluster-mysql-master.default.svc.cluster.local:3306
[root@k8s-master cloud]# vim values.yaml
......
xxljob:
  name: "xxl-job"
  labels: "xxl-job"
  service:
    type: "ClusterIP"
    port: 8080
    targetPort: 8080
  replicaCount: 3
  image:
    name: "xxl-job"
    pullPolicy: "IfNotPresent"
    tag: "2.3.0"
    containerPort: 8080
  env:
    params: "--spring.datasource.username=root --spring.datasource.password=root --spring.datasource.url=jdbc:mysql://my-cluster-mysql-master.default.svc.cluster.local:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai"
......
```

```shell
# 上传
[root@k8s-master pro]# helm cm-push cloud/ harborcloud
Pushing cloud-2.0.0.tgz to harborcloud...
Done.


# 升级

[root@k8s-master pro]# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "harborcloud" chart repository
...Successfully got an update from the "aliyun" chart repository
...Successfully got an update from the "elastic" chart repository
...Successfully got an update from the "ingress-nginx" chart repository
...Successfully got an update from the "bitnami" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈



[root@k8s-master pro]# helm upgrade msa harborcloud/cloud --version 2.0.0 -n pro
Release "msa" has been upgraded. Happy Helming!
NAME: msa
LAST DEPLOYED: Mon Dec 27 11:26:38 2021
NAMESPACE: pro
STATUS: deployed
REVISION: 2
TEST SUITE: None


[root@k8s-master pro]# helm list -n pro
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	pro      	2       	2021-12-27 11:26:38.977959094 +0800 CST	deployed	cloud-2.0.0	2.0.1
```



#### 6.5.删除仓库

```shell
# Harbor中的文件从页面删除

[root@k8s-master pro]# helm repo list
NAME         	URL                                                   
stable       	http://mirror.azure.cn/kubernetes/charts              
aliyun       	https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
bitnami      	https://charts.bitnami.com/bitnami                    
ingress-nginx	https://kubernetes.github.io/ingress-nginx            
elastic      	https://helm.elastic.co                               
harborcloud  	http://172.51.216.88:8888/chartrepo/springcloud


[root@k8s-master pro]# helm repo remove harborcloud
"harborcloud" has been removed from your repositories


[root@k8s-master pro]# helm repo list
NAME         	URL                                                   
stable       	http://mirror.azure.cn/kubernetes/charts              
aliyun       	https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
bitnami      	https://charts.bitnami.com/bitnami                    
ingress-nginx	https://kubernetes.github.io/ingress-nginx            
elastic      	https://helm.elastic.co
```



