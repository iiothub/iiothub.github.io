* TOC
{:toc}



## 一、概述



### 1.微服务部署方案

**1.架构图**



![](/images/kubernetes/microservice/dep-2.png)



**2.部署图**

![](/images/kubernetes/microservice/dep-3.png)



### 2.微服务部署规划

**1.微服务部署方式**

| 微服务              | 部署方式    | 实例数 | 备注            |
| ------------------- | ----------- | ------ | --------------- |
| Ingress             | NodePort    |        |                 |
| Eureka              | StatefulSet | 3      | 有状态服务      |
| Gateway             | Deployment  | 3      |                 |
| Zipkin              | Deployment  | 3      |                 |
| XXL-JOB             | Deployment  | 3      |                 |
| Sentinel            | Deployment  | 1      | 只能部署1个实例 |
| msa-deploy-producer | Deployment  | 3      |                 |
| msa-deploy-consumer | Deployment  | 3      |                 |
| msa-deploy-job      | Deployment  | 3      |                 |



**2.微服务功能**

| 微服务              | 功能                                    | 备注 |
| ------------------- | --------------------------------------- | ---- |
| Gateway             | 服务接口通过网关调用                    |      |
| msa-deploy-producer | 实现日志、调用链、Prometheus            |      |
| msa-deploy-consumer | 实现日志、调用链、Prometheus、openfeign |      |
| msa-deploy-job      | 分布式任务调度平台（XXL-JOB）、日志     |      |



**3.微服务访问信息**

| 微服务             | 内部端口 | 外部端口               | 访问地址                                                     |
| ------------------ | -------- | ---------------------- | ------------------------------------------------------------ |
| 注册中心           | 10001    | 30001                  | http://172.51.216.81:30001/                                  |
| 网关               | 8888     |                        | curl 10.99.231.140:8888/dconsumer/hello<br/>curl 10.99.231.140:8888/dconsumer/say<br/>curl 10.99.231.140:8888/dproducer/hello<br/>curl 10.99.231.140:8888/dproducer/say |
| 流量控制           | 8858     | 30858                  | 访问地址：http://172.51.216.81:30858/#/login<br/>账户密码：sentinel/sentinel |
| 调用链监控         | 9411     | 30411                  | http://172.51.216.81:30411/zipkin                            |
| 分布式任务调度平台 | 8080     | 30880                  | 访问地址：http://172.51.216.81:30880/xxl-job-admin<br/>账户密码：admin/123456 |
| 生产者             | 8911     |                        | curl 10.110.15.159:8911/hello<br/>curl 10.110.15.159:8911/say |
| 消费者             | 8912     |                        | curl 10.108.141.96:8912/hello<br/>curl 10.108.141.96:8912/say |
| Ingress            |          | 80:31208<br/>443:32099 | zipkin<br/>http://zipkin.k8s.com:31208/zipkin<br/><br/>sentinel<br/>http://sentinel.k8s.com:31208/#/login<br/>账户密码：sentinel/sentinel<br/><br/>xxl-job<br/>http://xxljob.k8s.com:31208/xxl-job-admin<br/>账户密码：admin/123456<br/><br/># eureka<br/>http://eureka.k8s.com:31208/<br/><br/># gateway<br/>http://gateway.k8s.com:31208/dconsumer/hello<br/>http://gateway.k8s.com:31208/dconsumer/say<br/>http://gateway.k8s.com:31208/dproducer/hello<br/>http://gateway.k8s.com:31208/dproducer/say |
| Prometheus         |          |                        | 消费者 msa-deploy-consumer<br/>http://gateway.k8s.com:31208/dconsumer/actuator/prometheus<br/><br/>生产者 msa-deploy-producer<br/>http://gateway.k8s.com:31208/dproducer/actuator/prometheus |



### 3.准备工作

#### 3.1.创建名字空间

```shell
[root@k8s-master ~]# kubectl create ns dev
namespace/dev created

[root@k8s-master ~]# kubectl get ns
...
dev                    Active   4s
```



#### 3.2.Harbor创建项目

在私有镜像仓库创建springcloud项目，存储微服务镜像



#### 3.3.创建私有镜像密钥Secret

K8S所有工作节点登陆Harbor镜像仓库

```shell
# 登陆 admin/admin
# docker login http://172.51.216.85:8888


[root@k8s-node1 ~]# docker login http://172.51.216.85:8888
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
{
	"auths": {
		"172.51.216.85:8888": {
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
ewoJImF1dGhzIjogewoJCSIxNzIuNTEuMjE2Ljg1Ojg4ODgiOiB7CgkJCSJhdXRoIjogIllXUnRhVzQ2WVdSdGFXND0iCgkJfQoJfQp9
```



**k8s创建secret秘钥**

**注意：必须在相同的命名空间，默认在default**

```shell
# secret
harbor-secret.yaml

apiVersion: v1
kind: Secret
metadata:
  name: harborsecret
  namespace: dev
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: ewoJImF1dGhzIjogewoJCSIxNzIuNTEuMjE2Ljg1Ojg4ODgiOiB7CgkJCSJhdXRoIjogIllXUnRhVzQ2WVdSdGFXND0iCgkJfQoJfQp9


[root@k8s-master harbor]# kubectl apply -f harbor-secret.yaml 
secret/harborsecret created

[root@k8s-master springcloud]# kubectl get secret -n dev
NAME                  TYPE                                  DATA   AGE
default-token-kjvf5   kubernetes.io/service-account-token   3      51m
harborsecret          kubernetes.io/dockerconfigjson        1      37s
```



#### 3.4.Dockerfile标准配置

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


#编译命令
# docker build -t 172.51.216.85:8888/springcloud/msa-app:1.0.0 .
```





## 二、微服务部署



### 1.微服务配置

#### 1.1.注册中心（eureka-server）

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
      nodePort: {{ .Values.eureka.service.nodePort }} #对外暴露30001端口
  selector:
    app: {{ .Values.eureka.labels }}

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  namespace: dev
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
        - name: {{ .Values.global.imagePullSecrets }}   #对应创建私有镜像密钥Secret
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
              value: {{ .Values.eureka.env.profiles }}
  podManagementPolicy: "Parallel"
```



```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-eureka
  labels:
    app: msa-eureka
spec:
  type: NodePort
  ports:
    - port: 10001
      name: msa-eureka
      targetPort: 10001
      nodePort: 30001 #对外暴露30001端口
  selector:
    app: msa-eureka

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  namespace: dev
  name: msa-eureka
spec:
  serviceName: msa-eureka
  replicas: 3
  selector:
    matchLabels:
      app: msa-eureka
  template:
    metadata:
      labels:
        app: msa-eureka
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-eureka
          image: 172.51.216.85:8888/springcloud/msa-eureka:2.0.0
          ports:
          - containerPort: 10001
          env:
            - name: MY_POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: EUREKA_INSTANCE_HOSTNAME
              value: ${MY_POD_NAME}.msa-eureka.dev
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
  podManagementPolicy: "Parallel"
```



#### 1.2.网关（msa-gateway）

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
        - name: {{ .Values.global.imagePullSecrets }}   #对应创建私有镜像密钥Secret
      containers:
        - name: {{ .Values.gateway.image.name }}
          image: {{ .Values.global.repository }}{{ .Values.gateway.image.name }}:{{ .Values.gateway.image.tag }}
          imagePullPolicy: {{ .Values.gateway.image.pullPolicy }} #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: {{ .Values.gateway.image.containerPort }}
          env:
            - name: EUREKA_SERVER
              value:  {{ .Values.global.env.eureka }}
            - name: ACTIVE
              value: {{ .Values.gateway.env.profiles }}
```



```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-gateway
  labels:
    app: msa-gateway
spec:
  type: ClusterIP
  ports:
    - port: 8888
      targetPort: 8888
  selector:
    app: msa-gateway

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-gateway
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-gateway
  template:
    metadata:
      labels:
        app: msa-gateway
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-gateway
          image: 172.51.216.85:8888/springcloud/msa-gateway:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8888
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
```



#### 1.3.流量控制（Sentinel）

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
      nodePort: {{ .Values.sentinel.service.nodePort }}
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
          imagePullPolicy: {{ .Values.sentinel.image.pullPolicy }} #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: {{ .Values.sentinel.image.containerPort }}
```



```shell
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: sentinel-server
  labels:
    app: sentinel-server
spec:
  type: NodePort
  ports:
    - port: 8858
      targetPort: 8858
      nodePort: 30858 #对外暴露30858端口
  selector:
    app: sentinel-server

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: sentinel-server
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sentinel-server
  template:
    metadata:
      labels:
        app: sentinel-server
    spec:
      containers:
        - name: sentinel-server
          image: bladex/sentinel-dashboard:1.7.2
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8858
```



#### 1.4.调用链监控（Zipkin）

**1.RabbitMQ**

```yaml
# rabbitmq.yml


apiVersion: v1
kind: Service
metadata:
  name: rabbitmq-svc
  namespace: dev
spec:
  ports:
  - port: 5672
    targetPort: 5672
    protocol: TCP

---
apiVersion: v1
kind: Endpoints
metadata:
  name: rabbitmq-svc
  namespace: dev
subsets:
  - addresses:
    - ip: 172.51.216.98
    ports:
    - port: 5672
```



**2.Elasticsearch**

```yaml
# elasticsearch.yml


apiVersion: v1
kind: Service
metadata:
  name: elasticsearch-svc
  namespace: dev
spec:
  ports:
  - port: 9200
    targetPort: 9200
    protocol: TCP

---
apiVersion: v1
kind: Endpoints
metadata:
  name: elasticsearch-svc
  namespace: dev
subsets:
  - addresses:
    - ip: 172.51.216.98
    ports:
    - port: 9200
```



**3.部署Zipkin服务**

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
      nodePort: {{ .Values.zipkin.service.nodePort }}
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
          imagePullPolicy: {{ .Values.zipkin.image.pullPolicy }} #如果省略imagePullPolicy,策略为IfNotPresent
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



```shell
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: zipkin-server
  labels:
    app: zipkin-server
spec:
  type: NodePort
  ports:
    - port: 9411
      targetPort: 9411
      nodePort: 30411 #对外暴露30411端口
  selector:
    app: zipkin-server

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: zipkin-server
spec:
  replicas: 2
  selector:
    matchLabels:
      app: zipkin-server
  template:
    metadata:
      labels:
        app: zipkin-server
    spec:
      containers:
        - name: zipkin-server
          image: openzipkin/zipkin:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 9411
          env:
            - name: RABBIT_ADDRESSES
              value: "rabbitmq-svc.dev.svc.cluster.local:5672"
            - name: RABBIT_USER
              value: "guest"
            - name: RABBIT_PASSWORD
              value: "guest"
            - name: STORAGE_TYPE
              value: "elasticsearch"
            - name: ES_HOSTS
              value: "elasticsearch-svc.dev.svc.cluster.local:9200"
```



#### 1.5.分布式任务调度平台（XXL-JOB）

**1.mysql**

```yaml
# mysql.yml


apiVersion: v1
kind: Service
metadata:
  name: mysql-svc
  namespace: dev
spec:
  ports:
  - port: 3306
    targetPort: 3306
    protocol: TCP

---
apiVersion: v1
kind: Endpoints
metadata:
  name: mysql-svc
  namespace: dev
subsets:
  - addresses:
    - ip: 172.51.216.98
    ports:
    - port: 3306
```



**2.部署XXL-JOB服务**

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
      nodePort: {{ .Values.xxljob.service.nodePort }}
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
          imagePullPolicy: {{ .Values.xxljob.image.pullPolicy }} #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: {{ .Values.xxljob.image.containerPort }}
          env:
            - name: PARAMS
              value:  {{ .Values.xxljob.env.params }}
```



```shell
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: xxl-job
  labels:
    app: xxl-job
spec:
  type: NodePort
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30880 #对外暴露30880端口
  selector:
    app: xxl-job

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: xxl-job
spec:
  replicas: 1
  selector:
    matchLabels:
      app: xxl-job
  template:
    metadata:
      labels:
        app: xxl-job
    spec:
      containers:
        - name: xxl-job
          image: xuxueli/xxl-job-admin:2.3.0
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080
          env:
            - name: PARAMS
              value:  "--spring.datasource.username=root --spring.datasource.password=root --spring.datasource.url=jdbc:mysql://mysql-svc.dev.svc.cluster.local:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai"
```



#### 1.6.部署XXL-JOB执行器（msa-deploy-job）

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
        - name: {{ .Values.global.imagePullSecrets }}   #对应创建私有镜像密钥Secret
      containers:
        - name: {{ .Values.job.image.name }}
          image: {{ .Values.global.repository }}{{ .Values.job.image.name }}:{{ .Values.job.image.tag }}
          imagePullPolicy: {{ .Values.job.image.pullPolicy }} #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: {{ .Values.job.image.containerPort }}
          env:
            - name: ACTIVE
              value: {{ .Values.job.env.profiles }}
```



```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-deploy-job
  labels:
    app: msa-deploy-job
spec:
  type: ClusterIP
  ports:
    - port: 8913
      targetPort: 8913
  selector:
    app: msa-deploy-job

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-deploy-job
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-deploy-job
  template:
    metadata:
      labels:
        app: msa-deploy-job
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-deploy-job
          image: 172.51.216.85:8888/springcloud/msa-deploy-job:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8913
          env:
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
```



#### 1.7.生产者（msa-deploy-producer）

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
        - name: {{ .Values.global.imagePullSecrets }}   #对应创建私有镜像密钥Secret
      containers:
        - name: {{ .Values.producer.image.name }}
          image: {{ .Values.global.repository }}{{ .Values.producer.image.name }}:{{ .Values.producer.image.tag }}
          imagePullPolicy: {{ .Values.producer.image.pullPolicy }} #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: {{ .Values.producer.image.containerPort }}
          env:
            - name: EUREKA_SERVER
              value:  {{ .Values.global.env.eureka }}
            - name: ACTIVE
              value: {{ .Values.producer.env.profiles }}
            - name: DASHBOARD
              value: {{ .Values.producer.env.dashboard }}
```



```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-deploy-producer
  labels:
    app: msa-deploy-producer
spec:
  type: ClusterIP
  ports:
    - port: 8911
      targetPort: 8911
  selector:
    app: msa-deploy-producer

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-deploy-producer
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-deploy-producer
  template:
    metadata:
      labels:
        app: msa-deploy-producer
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-deploy-producer
          image: 172.51.216.85:8888/springcloud/msa-deploy-producer:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8911
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"
```



#### 1.8.消费者（msa-deploy-consumer）

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
        - name: {{ .Values.global.imagePullSecrets }}   #对应创建私有镜像密钥Secret
      containers:
        - name: {{ .Values.consumer.image.name }}
          image: {{ .Values.global.repository }}{{ .Values.consumer.image.name }}:{{ .Values.consumer.image.tag }}
          imagePullPolicy: {{ .Values.consumer.image.pullPolicy }} #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: {{ .Values.consumer.image.containerPort }}
          env:
            - name: EUREKA_SERVER
              value:  {{ .Values.global.env.eureka }}
            - name: ACTIVE
              value: {{ .Values.consumer.env.profiles }}
            - name: DASHBOARD
              value: {{ .Values.consumer.env.dashboard }}
```



```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-deploy-consumer
  labels:
    app: msa-deploy-consumer
spec:
  type: ClusterIP
  ports:
    - port: 8912
      targetPort: 8912
  selector:
    app: msa-deploy-consumer

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-deploy-consumer
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-deploy-consumer
  template:
    metadata:
      labels:
        app: msa-deploy-consumer
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-deploy-consumer
          image: 172.51.216.85:8888/springcloud/msa-deploy-consumer:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8912
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"
```



#### 1.9.Ingress

**调用链监控（Zipkin）、流量控制（Sentinel）、分布式任务调度平台（XXL-JOB）、注册中心（eureka-server）、网关（msa-gateway）。**



**Http代理**

> **创建 msa-http.yaml**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: msa-http
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



#### 1.10.values

values.yaml 

```yaml
global:
  namespace: "dev"
  imagePullSecrets: "harborsecret"
  repository: "172.51.216.85:8888/springcloud/"
  env:
    eureka: "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"


eureka:
  name: "msa-eureka"
  labels: "msa-eureka"
  service:
    type: "NodePort"
    port: 10001
    targetPort: 10001
    nodePort: 30001
  replicaCount: 3
  image:
    name: "msa-eureka"
    pullPolicy: "IfNotPresent"
    tag: "2.0.0"
    containerPort: 10001
  env:
    profiles: "-Dspring.profiles.active=k8s"


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
    tag: "2.0.0"
    containerPort: 8888
  env:
    profiles: "-Dspring.profiles.active=k8s"


sentinel:
  name: "sentinel-server"
  labels: "sentinel-server"
  service:
    type: "NodePort"
    port: 8858
    targetPort: 8858
    nodePort: 30858
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
    type: "NodePort"
    port: 9411
    targetPort: 9411
    nodePort: 30411
  replicaCount: 3
  image:
    name: "zipkin-server"
    pullPolicy: "IfNotPresent"
    tag: "latest"
    containerPort: 9411
  env:
    addresses: "rabbitmq-svc.dev.svc.cluster.local:5672"
    user: "guest"
    password: "guest"
    storage: "elasticsearch"
    eshosts: "elasticsearch-svc.dev.svc.cluster.local:9200"


xxljob:
  name: "xxl-job"
  labels: "xxl-job"
  service:
    type: "NodePort"
    port: 8080
    targetPort: 8080
    nodePort: 30880
  replicaCount: 3
  image:
    name: "xxl-job"
    pullPolicy: "IfNotPresent"
    tag: "2.3.0"
    containerPort: 8080
  env:
    params: "--spring.datasource.username=root --spring.datasource.password=root --spring.datasource.url=jdbc:mysql://mysql-svc.dev.svc.cluster.local:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai"


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
    tag: "2.0.0"
    containerPort: 8913
  env:
    profiles: "-Dspring.profiles.active=k8s"


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
    tag: "2.0.0"
    containerPort: 8911
  env:
    profiles: "-Dspring.profiles.active=k8s"
    dashboard: "sentinel-server.dev.svc.cluster.local:8858"


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
    tag: "2.0.0"
    containerPort: 8912
  env:
    profiles: "-Dspring.profiles.active=k8s"
    dashboard: "sentinel-server.dev.svc.cluster.local:8858"
```



### 2.部署服务

**创建**

```shell
# 创建 msa
[root@k8s-master springcloud-helm]# helm create cloud -n dev
Creating cloud


[root@k8s-master test]# ll
drwxr-xr-x 4 root root 93 Nov 11 13:50 cloud


[root@k8s-master test]# tree
.
└── cloud
    ├── charts
    ├── Chart.yaml
    ├── templates
    │   ├── deployment.yaml
    │   ├── _helpers.tpl
    │   ├── hpa.yaml
    │   ├── ingress.yaml
    │   ├── NOTES.txt
    │   ├── serviceaccount.yaml
    │   ├── service.yaml
    │   └── tests
    │       └── test-connection.yaml
    └── values.yaml

4 directories, 10 files


--------------------------------------------
# templates删除所有文件
# templates创建配置文件

[root@k8s-master cloud]# tree
.
├── charts
├── Chart.yaml
├── templates
│   ├── elasticsearch.yml
│   ├── msa-deploy-consumer.yaml
│   ├── msa-deploy-job.yaml
│   ├── msa-deploy-producer.yaml
│   ├── msa-eureka.yaml
│   ├── msa-gateway.yaml
│   ├── msa-http.yaml
│   ├── mysql.yml
│   ├── NOTES.txt
│   ├── rabbitmq.yml
│   ├── sentinel-server.yaml
│   ├── xxl-job.yaml
│   └── zipkin-server.yaml
└── values.yaml

2 directories, 15 files
```

```shell
#模板文件

# Chart.yaml
[root@k8s-master cloud]# vim Chart.yaml

apiVersion: v2
name: cloud
description: A Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: "1.0.0"


# values.yaml 暂时不用
[root@k8s-master cloud]# vim values.yaml


# NOTES.txt
[root@k8s-master cloud]# vim NOTES.txt

Spring Cloud!!!
```



**安装**

```shell
# 查看实际的模板被渲染过后的资源文件
# helm get manifest web
# helm install web nginx/

# 生成的最终的资源清单文件打印出来，而不会真正的去部署一个release实例
# helm install --name mychart --dry-run --debug -f global.yaml ./mychart/
# helm install --name mychart --dry-run --debug --set course="k8s" ./mychart/


# 调试
# Helm也提供了--dry-run --debug调试参数，帮助你验证模板正确性。在执行helm install时候带上这两个参数就可以把对应的values值和渲染的资源清单打印出来，而不会真正的去部署一个release。

[root@k8s-master springcloud-helm]#  helm install msa cloud --dry-run --debug -n dev


[root@k8s-master springcloud-helm]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION

----------------------------------------------------------------------
# 安装
[root@k8s-master test]# helm install msa cloud -n dev
NAME: msa
LAST DEPLOYED: Thu Nov 11 14:25:55 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Spring Cloud!!!


[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	dev      	1       	2021-11-11 14:25:55.000327298 +0800 CST	deployed	cloud-0.1.0	1.0.0   


[root@k8s-master springcloud-helm]# helm install msa cloud -n dev
NAME: msa
LAST DEPLOYED: Thu Nov 11 22:56:20 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Spring Cloud!!!


# 检索已经发布的 release 的资源文件
[root@k8s-master springcloud-helm]# helm get manifest msa -n dev


# 打包
# 修改Chart.yaml中的helm chart配置信息，然后使用下列命令将chart打包成一个压缩文件。
# 打包出cloud-0.1.0.tgz文件。

[root@k8s-master springcloud-helm]# helm package cloud
Successfully packaged chart and saved it to: /k8s/springcloud-helm/cloud-0.1.0.tgz

[root@k8s-master springcloud-helm]# ll
total 8
drwxr-xr-x 4 root root   93 Nov 11 22:50 cloud
-rw-r--r-- 1 root root 2733 Nov 11 23:00 cloud-0.1.0.tgz
```

```shell
# 查看资源
[root@k8s-master springcloud-helm]# kubectl get all -n dev
NAME                                       READY   STATUS    RESTARTS   AGE
pod/msa-deploy-consumer-6b75cf55d-dj4vm    1/1     Running   0          5m8s
pod/msa-deploy-consumer-6b75cf55d-dzgtc    1/1     Running   0          5m8s
pod/msa-deploy-consumer-6b75cf55d-rx6sm    1/1     Running   0          5m8s
pod/msa-deploy-job-7c999dd76-87cpr         1/1     Running   0          5m8s
pod/msa-deploy-job-7c999dd76-wgt5v         1/1     Running   0          5m8s
pod/msa-deploy-job-7c999dd76-z5hp2         1/1     Running   0          5m8s
pod/msa-deploy-producer-7965c98bbf-hz6nj   1/1     Running   0          5m8s
pod/msa-deploy-producer-7965c98bbf-r8rs4   1/1     Running   0          5m8s
pod/msa-deploy-producer-7965c98bbf-s65qh   1/1     Running   0          5m8s
pod/msa-deploy-producer-7965c98bbf-sbndc   1/1     Running   0          5m8s
pod/msa-deploy-producer-7965c98bbf-w4bkf   1/1     Running   0          5m8s
pod/msa-eureka-0                           1/1     Running   0          5m8s
pod/msa-eureka-1                           1/1     Running   0          5m8s
pod/msa-eureka-2                           1/1     Running   0          5m8s
pod/msa-gateway-597494c7f4-fq47k           1/1     Running   0          5m8s
pod/msa-gateway-597494c7f4-hvj2r           1/1     Running   0          5m8s
pod/msa-gateway-597494c7f4-ws8f7           1/1     Running   0          5m8s
pod/sentinel-server-5697bdc87b-xlgzg       1/1     Running   0          5m8s
pod/xxl-job-57649955bd-9lcpw               1/1     Running   0          5m8s
pod/zipkin-server-84c5fdcf5d-76vlw         1/1     Running   0          5m8s
pod/zipkin-server-84c5fdcf5d-t4jjh         1/1     Running   0          5m8s

NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
service/elasticsearch-svc     ClusterIP   10.100.36.157    <none>        9200/TCP          5m8s
service/msa-deploy-consumer   ClusterIP   10.108.247.253   <none>        8912/TCP          5m8s
service/msa-deploy-job        ClusterIP   10.97.206.223    <none>        8913/TCP          5m8s
service/msa-deploy-producer   ClusterIP   10.102.227.63    <none>        8911/TCP          5m8s
service/msa-eureka            NodePort    10.111.13.62     <none>        10001:30001/TCP   5m8s
service/msa-gateway           ClusterIP   10.103.80.16     <none>        8888/TCP          5m8s
service/mysql-svc             ClusterIP   10.102.78.175    <none>        3306/TCP          5m8s
service/rabbitmq-svc          ClusterIP   10.110.218.166   <none>        5672/TCP          5m8s
service/sentinel-server       NodePort    10.111.141.18    <none>        8858:30858/TCP    5m8s
service/xxl-job               NodePort    10.102.102.253   <none>        8080:30880/TCP    5m8s
service/zipkin-server         NodePort    10.102.220.51    <none>        9411:30411/TCP    5m8s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/msa-deploy-consumer   3/3     3            3           5m8s
deployment.apps/msa-deploy-job        3/3     3            3           5m8s
deployment.apps/msa-deploy-producer   5/5     5            5           5m8s
deployment.apps/msa-gateway           3/3     3            3           5m8s
deployment.apps/sentinel-server       1/1     1            1           5m8s
deployment.apps/xxl-job               1/1     1            1           5m8s
deployment.apps/zipkin-server         2/2     2            2           5m8s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/msa-deploy-consumer-6b75cf55d    3         3         3       5m8s
replicaset.apps/msa-deploy-job-7c999dd76         3         3         3       5m8s
replicaset.apps/msa-deploy-producer-7965c98bbf   5         5         5       5m8s
replicaset.apps/msa-gateway-597494c7f4           3         3         3       5m8s
replicaset.apps/sentinel-server-5697bdc87b       1         1         1       5m8s
replicaset.apps/xxl-job-57649955bd               1         1         1       5m8s
replicaset.apps/zipkin-server-84c5fdcf5d         2         2         2       5m8s

NAME                          READY   AGE
statefulset.apps/msa-eureka   3/3     5m8s
```



### 3.测试

```shell
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
ingress-nginx                        NodePort    10.97.245.122    <none>        80:31208/TCP,443:32099/TCP   57d
ingress-nginx-controller             NodePort    10.101.238.213   <none>        80:30743/TCP,443:31410/TCP   57d
ingress-nginx-controller-admission   ClusterIP   10.100.142.101   <none>        443/TCP                      57d


# ingress地址
# 在本机浏览器输入:
http://zipkin.k8s.com:31208/
http://sentinel.k8s.com:31208/
http://xxljob.k8s.com:31208/
http://eureka.k8s.com:31208/
http://gateway.k8s.com:31208/


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
# eureka
http://eureka.k8s.com:31208/

# gateway
http://gateway.k8s.com:31208/dconsumer/hello
http://gateway.k8s.com:31208/dconsumer/say
http://gateway.k8s.com:31208/dproducer/hello
http://gateway.k8s.com:31208/dproducer/say
```

```shell
# 查看日志
[root@k8s-master msa-deploy-job]# kubectl logs -f --tail=10 msa-deploy-job-7c999dd76-87cpr -n dev
----job one 1---Wed Nov 10 09:30:50 CST 2021
----job one 1---Wed Nov 10 09:30:53 CST 2021
----job one 1---Wed Nov 10 09:30:56 CST 2021
----job one 1---Wed Nov 10 09:30:59 CST 2021
----job one 1---Wed Nov 10 09:31:02 CST 2021
----job one 1---Wed Nov 10 09:31:05 CST 2021
----job one 1---Wed Nov 10 09:31:08 CST 2021
----job one 1---Wed Nov 10 09:31:11 CST 2021
----job one 1---Wed Nov 10 09:31:14 CST 2021
----job one 1---Wed Nov 10 09:31:17 CST 2021
----job one 1---Wed Nov 10 09:31:20 CST 2021
```



### 4.升级

通过修改模板yaml升级

```shell
# 升级方式
# 修改生产者服务（msa-deploy-producer）接口两次，形成两个新版本的docker镜像
# 分别升级两次版本


# 升级前
[root@k8s-master springcloud-helm]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	dev      	1       	2021-11-11 22:56:20.805228313 +0800 CST	deployed	cloud-0.1.0	1.0.0  


[root@k8s-master springcloud-helm]# helm status msa -n dev
NAME: msa
LAST DEPLOYED: Thu Nov 11 22:56:20 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Spring Cloud!!!


[root@k8s-master springcloud-helm]# helm history msa -n dev
REVISION	UPDATED                 	STATUS  	CHART      	APP VERSION	DESCRIPTION     
1       	Thu Nov 11 22:56:20 2021	deployed	cloud-0.1.0	1.0.0      	Install complete


[root@k8s-master templates]# kubectl get all -n dev
NAME                                       READY   STATUS    RESTARTS   AGE
pod/msa-deploy-consumer-6b75cf55d-dj4vm    1/1     Running   0          10h
pod/msa-deploy-consumer-6b75cf55d-dzgtc    1/1     Running   0          10h
pod/msa-deploy-consumer-6b75cf55d-rx6sm    1/1     Running   0          10h
pod/msa-deploy-job-7c999dd76-87cpr         1/1     Running   0          10h
pod/msa-deploy-job-7c999dd76-wgt5v         1/1     Running   0          10h
pod/msa-deploy-job-7c999dd76-z5hp2         1/1     Running   0          10h
pod/msa-deploy-producer-7965c98bbf-hz6nj   1/1     Running   0          10h
pod/msa-deploy-producer-7965c98bbf-r8rs4   1/1     Running   0          10h
pod/msa-deploy-producer-7965c98bbf-s65qh   1/1     Running   0          10h
pod/msa-deploy-producer-7965c98bbf-sbndc   1/1     Running   0          10h
pod/msa-deploy-producer-7965c98bbf-w4bkf   1/1     Running   0          10h
pod/msa-eureka-0                           1/1     Running   0          10h
pod/msa-eureka-1                           1/1     Running   0          10h
pod/msa-eureka-2                           1/1     Running   0          10h
pod/msa-gateway-597494c7f4-fq47k           1/1     Running   0          10h
pod/msa-gateway-597494c7f4-hvj2r           1/1     Running   0          10h
pod/msa-gateway-597494c7f4-ws8f7           1/1     Running   0          10h
pod/sentinel-server-5697bdc87b-xlgzg       1/1     Running   0          10h
pod/xxl-job-57649955bd-9lcpw               1/1     Running   0          10h
pod/zipkin-server-84c5fdcf5d-76vlw         1/1     Running   0          10h
pod/zipkin-server-84c5fdcf5d-t4jjh         1/1     Running   0          10h

NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
service/elasticsearch-svc     ClusterIP   10.100.36.157    <none>        9200/TCP          10h
service/msa-deploy-consumer   ClusterIP   10.108.247.253   <none>        8912/TCP          10h
service/msa-deploy-job        ClusterIP   10.97.206.223    <none>        8913/TCP          10h
service/msa-deploy-producer   ClusterIP   10.102.227.63    <none>        8911/TCP          10h
service/msa-eureka            NodePort    10.111.13.62     <none>        10001:30001/TCP   10h
service/msa-gateway           ClusterIP   10.103.80.16     <none>        8888/TCP          10h
service/mysql-svc             ClusterIP   10.102.78.175    <none>        3306/TCP          10h
service/rabbitmq-svc          ClusterIP   10.110.218.166   <none>        5672/TCP          10h
service/sentinel-server       NodePort    10.111.141.18    <none>        8858:30858/TCP    10h
service/xxl-job               NodePort    10.102.102.253   <none>        8080:30880/TCP    10h
service/zipkin-server         NodePort    10.102.220.51    <none>        9411:30411/TCP    10h

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/msa-deploy-consumer   3/3     3            3           10h
deployment.apps/msa-deploy-job        3/3     3            3           10h
deployment.apps/msa-deploy-producer   5/5     5            5           10h
deployment.apps/msa-gateway           3/3     3            3           10h
deployment.apps/sentinel-server       1/1     1            1           10h
deployment.apps/xxl-job               1/1     1            1           10h
deployment.apps/zipkin-server         2/2     2            2           10h

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/msa-deploy-consumer-6b75cf55d    3         3         3       10h
replicaset.apps/msa-deploy-job-7c999dd76         3         3         3       10h
replicaset.apps/msa-deploy-producer-7965c98bbf   5         5         5       10h
replicaset.apps/msa-gateway-597494c7f4           3         3         3       10h
replicaset.apps/sentinel-server-5697bdc87b       1         1         1       10h
replicaset.apps/xxl-job-57649955bd               1         1         1       10h
replicaset.apps/zipkin-server-84c5fdcf5d         2         2         2       10h

NAME                          READY   AGE
statefulset.apps/msa-eureka   3/3     10h


# 升级1
[root@k8s-master springcloud-helm]# helm upgrade msa cloud -n dev
Release "msa" has been upgraded. Happy Helming!
NAME: msa
LAST DEPLOYED: Fri Nov 12 09:31:55 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
Spring Cloud!!!


[root@k8s-master springcloud-helm]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	dev      	2       	2021-11-12 09:31:55.408522116 +0800 CST	deployed	cloud-0.1.0	1.0.0  


# 升级2
[root@k8s-master springcloud-helm]# helm upgrade msa cloud -n dev
Release "msa" has been upgraded. Happy Helming!
NAME: msa
LAST DEPLOYED: Fri Nov 12 09:35:03 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 3
TEST SUITE: None
NOTES:
Spring Cloud!!!


[root@k8s-master springcloud-helm]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	dev      	3       	2021-11-12 09:35:03.053485256 +0800 CST	deployed	cloud-0.1.0	1.0.0  


[root@k8s-master springcloud-helm]# helm history msa -n dev
REVISION	UPDATED                 	STATUS    	CHART      	APP VERSION	DESCRIPTION     
1       	Thu Nov 11 22:56:20 2021	superseded	cloud-0.1.0	1.0.0      	Install complete
2       	Fri Nov 12 09:31:55 2021	superseded	cloud-0.1.0	1.0.0      	Upgrade complete
3       	Fri Nov 12 09:35:03 2021	deployed  	cloud-0.1.0	1.0.0      	Upgrade complete


[root@k8s-master springcloud-helm]# helm get manifest msa -n dev
```



###  5.回滚

```shell
# 如果在发布后没有达到预期的效果，则可以使用helm rollback回滚到之前的版本。
# 例如将应用回滚到第一个版本：
helm rollback web 1


# 回滚前
[root@k8s-master springcloud-helm]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	dev      	3       	2021-11-12 09:35:03.053485256 +0800 CST	deployed	cloud-0.1.0	1.0.0


[root@k8s-master springcloud-helm]# helm status msa -n dev
NAME: msa
LAST DEPLOYED: Fri Nov 12 09:35:03 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 3
TEST SUITE: None
NOTES:
Spring Cloud!!!


[root@k8s-master springcloud-helm]# helm history msa -n dev
REVISION	UPDATED                 	STATUS    	CHART      	APP VERSION	DESCRIPTION     
1       	Thu Nov 11 22:56:20 2021	superseded	cloud-0.1.0	1.0.0      	Install complete
2       	Fri Nov 12 09:31:55 2021	superseded	cloud-0.1.0	1.0.0      	Upgrade complete
3       	Fri Nov 12 09:35:03 2021	deployed  	cloud-0.1.0	1.0.0      	Upgrade complete


# 回滚到版本1
[root@k8s-master springcloud-helm]# helm rollback msa 1 -n dev
Rollback was a success! Happy Helming!


# 回滚后的状态
[root@k8s-master springcloud-helm]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	dev      	4       	2021-11-12 09:41:31.271565007 +0800 CST	deployed	cloud-0.1.0	1.0.0  

[root@k8s-master springcloud-helm]# helm status msa -n dev
NAME: msa
LAST DEPLOYED: Fri Nov 12 09:41:31 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 4
TEST SUITE: None
NOTES:
Spring Cloud!!!


[root@k8s-master springcloud-helm]# helm history msa -n dev
REVISION	UPDATED                 	STATUS    	CHART      	APP VERSION	DESCRIPTION     
1       	Thu Nov 11 22:56:20 2021	superseded	cloud-0.1.0	1.0.0      	Install complete
2       	Fri Nov 12 09:31:55 2021	superseded	cloud-0.1.0	1.0.0      	Upgrade complete
3       	Fri Nov 12 09:35:03 2021	superseded	cloud-0.1.0	1.0.0      	Upgrade complete
4       	Fri Nov 12 09:41:31 2021	deployed  	cloud-0.1.0	1.0.0      	Rollback to 1 
```



### 6.删除

```shell
卸载发行版，请使用以下helm uninstall命令：
# helm uninstall web -n dev
# helm delete web -n dev


[root@k8s-master springcloud-helm]# helm uninstall msa -n dev
release "msa" uninstalled


[root@k8s-master springcloud-helm]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
```



### 7.使用包安装

```shell
# 使用 cloud-0.1.0.tgz 安装

[root@k8s-master springcloud-helm]# helm install msa cloud-0.1.0.tgz -n dev
NAME: msa
LAST DEPLOYED: Fri Nov 12 09:51:09 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Spring Cloud!!!


[root@k8s-master springcloud-helm]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	dev      	1       	2021-11-12 09:51:09.530677806 +0800 CST	deployed	cloud-0.1.0	1.0.0
```



### 8.配置values

#### 8.1.修改配置信息

values.yaml 

```yaml
global:
  namespace: "dev"

producer:
  name: "msa-deploy-producer"
  serviceType: "ClusterIP"
  port: 8911
  targetPort: 8911
  replicas: 4
  imageVersion: "2.0.0"
```



**生产者**

msa-deploy-producer.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.producer.name }}
  labels:
    app: {{ .Values.producer.name }}
spec:
  type: {{ .Values.producer.serviceType }}
  ports:
    - port: {{ .Values.producer.port }}
      targetPort: {{ .Values.producer.targetPort }}
  selector:
    app: {{ .Values.producer.name }}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.producer.name }}
spec:
  replicas: {{ .Values.producer.replicas }}
  selector:
    matchLabels:
      app: {{ .Values.producer.name }}
  template:
    metadata:
      labels:
        app: {{ .Values.producer.name }}
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: {{ .Values.producer.name }}
          image: 172.51.216.85:8888/springcloud/{{ .Values.producer.name }}:{{ .Values.producer.imageVersion }}
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: {{ .Values.producer.targetPort }}
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"
```

原配置

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-deploy-producer
  labels:
    app: msa-deploy-producer
spec:
  type: ClusterIP
  ports:
    - port: 8911
      targetPort: 8911
  selector:
    app: msa-deploy-producer

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-deploy-producer
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-deploy-producer
  template:
    metadata:
      labels:
        app: msa-deploy-producer
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-deploy-producer
          image: 172.51.216.85:8888/springcloud/msa-deploy-producer:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8911
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"
```



#### 8.2.部署服务

```shell
[root@k8s-master test]# helm install msa cloud --dry-run --debug -n dev


# 安装
[root@k8s-master springcloud-helm]# 
[root@k8s-master springcloud-helm]# helm install msa cloud -n dev
NAME: msa
LAST DEPLOYED: Fri Nov 12 10:16:35 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Spring Cloud!!!


[root@k8s-master springcloud-helm]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	dev      	1       	2021-11-12 10:16:35.753166106 +0800 CST	deployed	cloud-0.2.0	2.0.0  


# 检索已经发布的 release 的资源文件
[root@k8s-master test]# helm get manifest msa -n dev

```



#### 8.3.升级

通过修改values.yaml升级

```shell
# 通过修改values.yaml升级，拉取不同镜像版本

# 升级前
[root@k8s-master springcloud-helm]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	dev      	1       	2021-11-12 10:16:35.753166106 +0800 CST	deployed	cloud-0.2.0	2.0.0  


[root@k8s-master springcloud-helm]# helm history msa -n dev
REVISION	UPDATED                 	STATUS  	CHART      	APP VERSION	DESCRIPTION     
1       	Fri Nov 12 10:16:35 2021	deployed	cloud-0.2.0	2.0.0      	Install complete


# 修改文件
# 修改成副本
[root@k8s-master cloud]# vim values.yaml
global:
  namespace: "dev"

producer:
  name: "msa-deploy-producer"
  serviceType: "ClusterIP"
  port: 8911
  targetPort: 8911
  replicas: 8
  imageVersion: "2.2.0"


[root@k8s-master springcloud-helm]# helm upgrade msa cloud -n dev
Release "msa" has been upgraded. Happy Helming!
NAME: msa
LAST DEPLOYED: Fri Nov 12 10:22:37 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
Spring Cloud!!!



[root@k8s-master springcloud-helm]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	dev      	2       	2021-11-12 10:22:37.210875583 +0800 CST	deployed	cloud-0.2.0	2.0.0  


[root@k8s-master springcloud-helm]# helm history msa -n dev
REVISION	UPDATED                 	STATUS    	CHART      	APP VERSION	DESCRIPTION     
1       	Fri Nov 12 10:16:35 2021	superseded	cloud-0.2.0	2.0.0      	Install complete
2       	Fri Nov 12 10:22:37 2021	deployed  	cloud-0.2.0	2.0.0      	Upgrade complete


[root@k8s-master test]# helm get manifest msa -n dev
```



### 9.Chart Hooks

#### 9.1.Hook配置

Hooks在资源清单中的metadata部分用annotations注解的方式进行声明：

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  annotations:
    "helm.sh/hook": post-install,post-upgrade
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": hook-succeeded
......
```



#### 9.2.生产者加Hook注解

msa-deploy-producer.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.producer.name }}
  labels:
    app: {{ .Values.producer.name }}
  annotations:
    "helm.sh/hook": post-install,post-upgrade
    "helm.sh/hook-weight": "-5"
spec:
  type: {{ .Values.producer.serviceType }}
  ports:
    - port: {{ .Values.producer.port }}
      targetPort: {{ .Values.producer.targetPort }}
  selector:
    app: {{ .Values.producer.name }}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.producer.name }}
  annotations:
    "helm.sh/hook": post-install,post-upgrade
    "helm.sh/hook-weight": "-5"
spec:
  replicas: {{ .Values.producer.replicas }}
  selector:
    matchLabels:
      app: {{ .Values.producer.name }}
  template:
    metadata:
      labels:
        app: {{ .Values.producer.name }}
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: {{ .Values.producer.name }}
          image: 172.51.216.85:8888/springcloud/{{ .Values.producer.name }}:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: {{ .Values.producer.targetPort }}
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"
```



```shell
[root@k8s-master test]# helm install msa cloud --dry-run --debug -n dev


# 安装
[root@k8s-master test]# helm install msa cloud -n dev
NAME: msa
LAST DEPLOYED: Fri Nov 12 11:36:58 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Spring Cloud!!!


[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	dev      	1       	2021-11-12 11:36:58.339808934 +0800 CST	deployed	cloud-0.5.0	1.0.5 


# 检索已经发布的 release 的资源文件
[root@k8s-master test]# helm get manifest msa -n dev


# 卸载
[root@k8s-master test]# helm uninstall msa -n dev
release "msa" uninstalled

# Hook资源需要自己删除
[root@k8s-master test]# kubectl get all -n dev
NAME                                       READY   STATUS             RESTARTS   AGE
pod/msa-deploy-producer-786f6dff44-4tn6h   0/1     ImagePullBackOff   0          5m24s
pod/msa-deploy-producer-786f6dff44-7wcbw   0/1     ImagePullBackOff   0          5m24s
pod/msa-deploy-producer-786f6dff44-b56r8   0/1     ImagePullBackOff   0          5m24s

NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/msa-deploy-producer   ClusterIP   10.101.112.240   <none>        8911/TCP   5m24s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/msa-deploy-producer   0/3     3            0           5m24s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/msa-deploy-producer-786f6dff44   3         3         0       5m24s
```



