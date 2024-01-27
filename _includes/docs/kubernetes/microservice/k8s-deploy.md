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



### 1.注册中心（eureka-server）

#### 1.1.配置文件

**1.微服务配置文件**

```yaml
# application-k8s.yml

server:
  port: 10001

spring:
  application:
    name: msa-eureka

eureka:
  server:
    #关闭自我保护
    enable-self-preservation: false
    use-read-only-response-cache: false
    #设置自动清理时间
    eviction-interval-timer-in-ms: 5000
  client:
    registry-fetch-interval-seconds: 5
    #注册中心职责是维护服务实例，false：不检索服务。
    fetch-registry: true
    #此应用为注册中心，false：不向注册中心注册自己。
    register-with-eureka: true
    service-url:
      defaultZone: ${EUREKA_SERVER:http://127.0.0.1:${server.port}/eureka/}
  instance:
    hostname: ${EUREKA_INSTANCE_HOSTNAME:${spring.application.name}}
    lease-renewal-interval-in-seconds: 5
    lease-expiration-duration-in-seconds: 10
    instance-id: ${EUREKA_INSTANCE_HOSTNAME:${spring.application.name}}:${server.port}@${random.l ong(1000000,9999999)}
```



**2.k8s-yaml**

 msa-eureka.yaml

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



#### 1.2.创建镜像

```shell
# 1.上传JAR
# 2.创建Dockerfile

# 3.创建镜像
docker build -t 172.51.216.85:8888/springcloud/msa-eureka:2.0.0 .

# 4.推送镜像
docker push 172.51.216.85:8888/springcloud/msa-eureka:2.0.0
```



#### 1.3.创建服务

```shell
# 创建
kubectl apply -f msa-eureka.yaml 


# 查看
[root@k8s-master msa-eureka]# kubectl get all -n dev
NAME               READY   STATUS    RESTARTS   AGE
pod/msa-eureka-0   1/1     Running   0          36s
pod/msa-eureka-1   1/1     Running   0          36s
pod/msa-eureka-2   1/1     Running   0          36s

NAME                 TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
service/msa-eureka   NodePort   10.103.51.174   <none>        10001:30001/TCP   36s

NAME                          READY   AGE
statefulset.apps/msa-eureka   3/3     36s
```



#### 1.4.测试

访问地址：http://172.51.216.81:30001/



### 2.网关（msa-gateway）

#### 2.1.配置文件

**1.微服务配置文件**

```yaml
# application-k8s.yml


server:
  port: 8888

spring:
  application:
    name: msa-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: MSA-PRODUCER    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://MSA-PRODUCER # 服务名称匹配后提供服务的路由地址
          predicates:
            - Path= /producer/**
          filters:
            - StripPrefix=1
        - id: MSA-CONSUMER    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://MSA-CONSUMER # 服务名称匹配后提供服务的路由地址
          predicates:
            - Path= /consumer/**
          filters:
            - StripPrefix=1
        - id: msa-deploy-producer    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://msa-deploy-producer # 服务名称匹配后提供服务的路由地址
          predicates:
            - Path= /dproducer/**
          filters:
            - StripPrefix=1
        - id: msa-deploy-consumer    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://msa-deploy-consumer # 服务名称匹配后提供服务的路由地址
          predicates:
            - Path= /dconsumer/**
          filters:
            - StripPrefix=1

eureka:
  client:
    #此客户端是否获取eureka服务器注册表上的注册信息，默认为true
    fetch-registry: true
    #实例是否在eureka服务器上注册自己的信息以供其他服务发现，默认为true,即自己注册自己。
    register-with-eureka: true
    service-url:
      defaultZone: ${EUREKA_SERVER:http://127.0.0.1:10001/eureka/}
  #服务注册中心实例的主机名
  instance:
    hostname: ${EUREKA_INSTANCE_HOSTNAME:${spring.application.name}}
    prefer-ip-address: true
    instance-id: ${spring.cloud.client.ip-address}:${server.port}
    ip-address: ${spring.cloud.client.ip-address}
    lease-expiration-duration-in-seconds: 15
    lease-renewal-interval-in-seconds: 3
```



**2.k8s-yaml**

 msa-gateway.yaml

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



#### 2.2.创建镜像

```shell
# 1.上传JAR
# 2.创建Dockerfile

# 3.创建镜像
docker build -t 172.51.216.85:8888/springcloud/msa-gateway:2.0.0 .

# 4.推送镜像
docker push 172.51.216.85:8888/springcloud/msa-gateway:2.0.0
```



#### 2.3.创建服务

```shell
# 创建
kubectl apply -f msa-gateway.yaml 


# 查看
[root@k8s-master msa-gateway]# kubectl get all -n dev | grep msa-gateway
pod/msa-gateway-597494c7f4-2h4w4   1/1     Running   0          43s
pod/msa-gateway-597494c7f4-k45gv   1/1     Running   0          43s
pod/msa-gateway-597494c7f4-vvs9r   1/1     Running   0          43s

service/msa-gateway   ClusterIP   10.99.231.140   <none>        8888/TCP          43s

deployment.apps/msa-gateway   3/3     3            3           43s

replicaset.apps/msa-gateway-597494c7f4   3         3         3       43s
```



#### 2.4.测试

```shell
curl 10.99.231.140:8888/dconsumer/hello
curl 10.99.231.140:8888/dconsumer/say
curl 10.99.231.140:8888/dproducer/hello
curl 10.99.231.140:8888/dproducer/say
```



### 3.流量控制（Sentinel）

sentinel-server.yaml

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


# 参考容器配置
docker run --name sentinel --restart=always -d -p 8858:8858 -d bladex/sentinel-dashboard
```

```shell
# 创建
kubectl apply -f sentinel-server.yaml 


[root@k8s-master sentinel]#  kubectl get all -n dev | grep sentinel-server
pod/sentinel-server-5697bdc87b-8p2ft   1/1     Running   0          14s

service/sentinel-server   NodePort    10.104.240.19   <none>        8858:30858/TCP    14s

deployment.apps/sentinel-server   1/1     1            1           14s

replicaset.apps/sentinel-server-5697bdc87b   1         1         1       14s


# 访问dashboard
访问地址：http://172.51.216.81:30858/#/login
账户密码：sentinel/sentinel
```



### 4.调用链监控（Zipkin）

#### 4.1.部署方式

![](/images/kubernetes/microservice/dep-4.png)



#### 4.2.创建Endpoint

##### 4.2.1.RabbitMQ

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



```shell
# 创建
kubectl apply -f rabbitmq.yml 


[root@k8s-master zipkin]# kubectl get all -n dev | grep rabbitmq-svc 
service/rabbitmq-svc      ClusterIP   10.102.151.225   <none>        5672/TCP          63s
[root@k8s-master zipkin]# 
[root@k8s-master zipkin]# 
[root@k8s-master zipkin]# kubectl describe ep rabbitmq-svc -n dev
Name:         rabbitmq-svc
Namespace:    dev
Labels:       <none>
Annotations:  <none>
Subsets:
  Addresses:          172.51.216.98
  NotReadyAddresses:  <none>
  Ports:
    Name     Port  Protocol
    ----     ----  --------
    <unset>  5672  TCP

Events:  <none>


[root@k8s-master zipkin]# kubectl describe svc rabbitmq-svc -n dev
Name:              rabbitmq-svc
Namespace:         dev
Labels:            <none>
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Families:       <none>
IP:                10.102.151.225
IPs:               10.102.151.225
Port:              <unset>  5672/TCP
TargetPort:        5672/TCP
Endpoints:         172.51.216.98:5672
Session Affinity:  None
Events:            <none>


# K8S 中的容器使用访问
svcname.namespace.svc.cluster.local:port
rabbitmq-svc.dev.svc.cluster.local:5672
```



##### 4.2.2.Elasticsearch

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



```shell
# 创建
kubectl apply -f elasticsearch.yml 

kubectl describe ep elasticsearch-svc -n dev
kubectl describe svc elasticsearch-svc -n dev

# K8S 中的容器使用访问
svcname.namespace.svc.cluster.local:port
elasticsearch-svc.dev.svc.cluster.local:9200
```



#### 4.3.部署Zipkin服务

zipkin-server.yaml

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


--------------------------------------------
# 参考容器配置
docker run -d --name zipkin -p 9411:9411 --restart=always \
-e RABBIT_ADDRESSES=172.51.216.98:5672 \
-e RABBIT_USER=guest \
-e RABBIT_PASSWORD=guest \
-e STORAGE_TYPE=elasticsearch \
-e ES_HOSTS=http://172.51.216.98:9200 \
openzipkin/zipkin:latest
```

```shell
# 创建
kubectl apply -f zipkin-server.yaml 


[root@k8s-master zipkin]#  kubectl get all -n dev | grep zipkin-server
pod/zipkin-server-84c5fdcf5d-fcdlj     1/1     Running   0          13s
pod/zipkin-server-84c5fdcf5d-qd9rx     1/1     Running   0          13s

service/zipkin-server       NodePort    10.106.137.33    <none>        9411:30411/TCP    13s

deployment.apps/zipkin-server     2/2     2            2           13s

replicaset.apps/zipkin-server-84c5fdcf5d     2         2         2       13s


# 访问地址
http://172.51.216.81:30411/zipkin
```



### 5.分布式任务调度平台（XXL-JOB）

#### 5.1.创建Endpoint

 mysql.yml

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

```shell
# 创建
kubectl apply -f mysql.yml 

kubectl describe ep mysql-svc -n dev
kubectl describe svc  mysql-svc -n dev

# K8S 中的容器使用访问
svcname.namespace.svc.cluster.local:port
mysql-svc.dev.svc.cluster.local:3306
```



#### 5.2.部署XXL-JOB服务

xxl-job.yaml

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



# 参考容器配置
docker run --network host -d --restart=always --name xxl-job-admin \
-e PARAMS="--spring.datasource.username=root \
--spring.datasource.password=root \
--spring.datasource.url=jdbc:mysql://172.51.216.98:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai" \
-v /tmp:/data/applogs \
xuxueli/xxl-job-admin:2.3.0
```

```shell
# 创建
kubectl apply -f xxl-job.yaml 


[root@k8s-master xxl-job]# kubectl get all -n dev | grep xxl-job
pod/xxl-job-57649955bd-n849s           1/1     Running   0          11s

service/xxl-job             NodePort    10.101.116.148   <none>        8080:30880/TCP    11s

deployment.apps/xxl-job           1/1     1            1           11s

replicaset.apps/xxl-job-57649955bd           1         1         1       11s


# 访问dashboard
访问地址：http://172.51.216.81:30880/xxl-job-admin
账户密码：admin/123456
```



#### 5.3.配置XXL-JOB

**1.配置执行器**

![](/images/kubernetes/microservice/dep-5.png)



**2.配置任务**

![](/images/kubernetes/microservice/dep-6.png)



### 6.部署XXL-JOB执行器（msa-deploy-job）

#### 6.1.配置文件

**1.微服务配置文件**

```yaml
# application-k8s.properties


# web port
server.port=8913
# no web
#spring.main.web-environment=false
# log config
#logging.config=classpath:logback.xml

### xxl-job admin address list, such as "http://address" or "http://address01,http://address02"
xxl.job.admin.addresses=http://xxl-job.dev.svc.cluster.local:8080/xxl-job-admin
### xxl-job, access token
xxl.job.accessToken=
### xxl-job executor appname
xxl.job.executor.appname=msa-ext-job
### xxl-job executor registry-address: default use address to registry , otherwise use ip:port if address is null
xxl.job.executor.address=
### xxl-job executor server-info
xxl.job.executor.ip=
xxl.job.executor.port=9999
### xxl-job executor log-path
xxl.job.executor.logpath=/data/applogs/xxl-job/jobhandler
### xxl-job executor log-retention-days
xxl.job.executor.logretentiondays=30
```



**2.k8s-yaml**

msa-deploy-job.yaml

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



#### 6.2.创建镜像

```shell
# 1.上传JAR
# 2.创建Dockerfile

# 3.创建镜像
docker build -t 172.51.216.85:8888/springcloud/msa-deploy-job:2.0.0 .

# 4.推送镜像
docker push 172.51.216.85:8888/springcloud/msa-deploy-job:2.0.0
```



#### 6.3.创建服务

```shell
# 创建
kubectl apply -f msa-deploy-job.yaml 

# 查看
[root@k8s-master msa-deploy-job]# kubectl get all -n dev | grep msa-deploy-job
pod/msa-deploy-job-7c999dd76-jnmmv     1/1     Running   0          17s
pod/msa-deploy-job-7c999dd76-nfl86     1/1     Running   0          17s
pod/msa-deploy-job-7c999dd76-ns86z     1/1     Running   0          17s

service/msa-deploy-job      ClusterIP   10.101.54.214    <none>        8913/TCP          17s

deployment.apps/msa-deploy-job    3/3     3            3           17s

replicaset.apps/msa-deploy-job-7c999dd76     3         3         3       17s
```



#### 6.4.测试

```shell
# 查看日志
[root@k8s-master msa-deploy-job]# kubectl logs -f --tail=10 msa-deploy-job-7c999dd76-jnmmv -n dev
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



### 7.生产者（msa-deploy-producer）

#### 7.1.配置文件

**1.微服务配置文件**

```yaml
# application-k8s.yml


server:
  port: 8911

spring:
  application:
    name: msa-deploy-producer
  cloud:
    sentinel:
      transport:
        port: 8719
        dashboard: ${DASHBOARD:172.51.216.98:8858}
  sleuth:
    web:
      client:
        enabled: true
    sampler:
      probability: 1.0 # 将采样比例设置为 1.0，也就是全部都需要。默认是 0.1
  zipkin:
    #    rabbitmq:
    #      queue: msa-zipkin
    #    discovery-client-enabled: false
    sender:
      type: rabbit
  rabbitmq:
    host: rabbitmq-svc.dev.svc.cluster.local
    username: guest
    password: guest
    port: 5672

eureka:
  client:
    #此客户端是否获取eureka服务器注册表上的注册信息，默认为true
    fetch-registry: true
    #实例是否在eureka服务器上注册自己的信息以供其他服务发现，默认为true,即自己注册自己。
    register-with-eureka: true
    service-url:
      defaultZone: ${EUREKA_SERVER:http://127.0.0.1:10001/eureka/}
  #服务注册中心实例的主机名
  instance:
    hostname: ${EUREKA_INSTANCE_HOSTNAME:${spring.application.name}}
    prefer-ip-address: true
    instance-id: ${spring.cloud.client.ip-address}:${server.port}
    ip-address: ${spring.cloud.client.ip-address}
    lease-expiration-duration-in-seconds: 15
    lease-renewal-interval-in-seconds: 3
```



**2.k8s-yaml**

msa-deploy-producer.yaml

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



#### 7.2.创建镜像

```shell
# 1.上传JAR
# 2.创建Dockerfile

# 3.创建镜像
docker build -t 172.51.216.85:8888/springcloud/msa-deploy-producer:2.0.0 .

# 4.推送镜像
docker push 172.51.216.85:8888/springcloud/msa-deploy-producer:2.0.0
```



#### 7.3.创建服务

```shell
# 创建
kubectl apply -f msa-deploy-producer.yaml 

# 查看
[root@k8s-master msa-deploy-producer]# kubectl get all -n dev | grep msa-deploy-producer
pod/msa-deploy-producer-7965c98bbf-cgxg8   1/1     Running   0          60s
pod/msa-deploy-producer-7965c98bbf-s7d8d   1/1     Running   0          60s
pod/msa-deploy-producer-7965c98bbf-vk7c4   1/1     Running   0          60s

service/msa-deploy-producer   ClusterIP   10.110.15.159    <none>        8911/TCP          60s

deployment.apps/msa-deploy-producer   3/3     3            3           60s

replicaset.apps/msa-deploy-producer-7965c98bbf   3         3         3       60s
```



#### 7.4.测试

```shell
curl 10.110.15.159:8911/hello
curl 10.110.15.159:8911/say
```



### 8.消费者（msa-deploy-consumer）

#### 8.1.配置文件

**1.微服务配置文件**

```yaml
# application-k8s.yml


server:
  port: 8912

spring:
  application:
    name: msa-deploy-consumer
  cloud:
    sentinel:
      transport:
        port: 8719
        dashboard: ${DASHBOARD:172.51.216.98:8858}
  sleuth:
    web:
      client:
        enabled: true
    sampler:
      probability: 1.0 # 将采样比例设置为 1.0，也就是全部都需要。默认是 0.1
  zipkin:
    #    rabbitmq:
    #      queue: msa-zipkin
    #    discovery-client-enabled: false
    sender:
      type: rabbit
  rabbitmq:
    host: rabbitmq-svc.dev.svc.cluster.local
    username: guest
    password: guest
    port: 5672

eureka:
  client:
    #此客户端是否获取eureka服务器注册表上的注册信息，默认为true
    fetch-registry: true
    #实例是否在eureka服务器上注册自己的信息以供其他服务发现，默认为true,即自己注册自己。
    register-with-eureka: true
    service-url:
      defaultZone: ${EUREKA_SERVER:http://127.0.0.1:10001/eureka/}
  #服务注册中心实例的主机名
  instance:
    hostname: ${EUREKA_INSTANCE_HOSTNAME:${spring.application.name}}
    prefer-ip-address: true
    instance-id: ${spring.cloud.client.ip-address}:${server.port}
    ip-address: ${spring.cloud.client.ip-address}
    lease-expiration-duration-in-seconds: 15
    lease-renewal-interval-in-seconds: 3
```



**2.k8s-yaml**

msa-deploy-consumer.yaml

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



#### 8.2.创建镜像

```shell
# 1.上传JAR
# 2.创建Dockerfile

# 3.创建镜像
docker build -t 172.51.216.85:8888/springcloud/msa-deploy-consumer:2.0.0 .

# 4.推送镜像
docker push 172.51.216.85:8888/springcloud/msa-deploy-consumer:2.0.0
```



#### 8.3.创建服务

```shell
# 创建
kubectl apply -f msa-deploy-consumer.yaml 


# 查看
[root@k8s-master msa-deploy-consumer]# kubectl get all -n dev | grep msa-deploy-consumer
pod/msa-deploy-consumer-6b75cf55d-829pf    1/1     Running   0          13s
pod/msa-deploy-consumer-6b75cf55d-bzpvk    1/1     Running   0          13s
pod/msa-deploy-consumer-6b75cf55d-cwdgj    1/1     Running   0          13s

service/msa-deploy-consumer   ClusterIP   10.108.141.96    <none>        8912/TCP          13s

deployment.apps/msa-deploy-consumer   3/3     3            3           13s

replicaset.apps/msa-deploy-consumer-6b75cf55d    3         3         3       13s
```



#### 8.4.测试

```shell
curl 10.108.141.96:8912/hello
curl 10.108.141.96:8912/say
```





## 三、Ingress



### 1.HTTP代理

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

```shell
# 创建
kubectl apply -f msa-http.yaml


# 查看
[root@k8s-master ingress]# kubectl get ing -n dev
NAME       CLASS    HOSTS                                                        ADDRESS         PORTS   AGE
msa-http   <none>   zipkin.k8s.com,sentinel.k8s.com,xxljob.k8s.com + 2 more...   10.97.245.122   80      45s


# 查看详情
[root@k8s-master ingress]# kubectl describe ing msa-http -n dev
Name:             msa-http
Namespace:        dev
Address:          10.97.245.122
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host              Path  Backends
  ----              ----  --------
  zipkin.k8s.com    
                    /   zipkin-server:9411 (10.244.169.129:9411,10.244.36.98:9411)
  sentinel.k8s.com  
                    /   sentinel-server:8858 (10.244.169.130:8858)
  xxljob.k8s.com    
                    /   xxl-job:8080 (10.244.107.253:8080)
  eureka.k8s.com    
                    /   msa-eureka:10001 (10.244.169.189:10001,10.244.169.191:10001,10.244.36.99:10001)
  gateway.k8s.com   
                    /   msa-gateway:8888 (10.244.107.195:8888,10.244.107.254:8888,10.244.36.97:8888)
Annotations:        kubernetes.io/ingress.class: nginx
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  CREATE  94s   nginx-ingress-controller  Ingress dev/msa-http
  Normal  UPDATE  69s   nginx-ingress-controller  Ingress dev/msa-http


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



### 2.HTTPS代理

**调用链监控（Zipkin）、流量控制（Sentinel）、分布式任务调度平台（XXL-JOB）、注册中心（eureka-server）、网关（msa-gateway）。**



**Https代理**

> **创建 msa-https.yaml**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: msa-https
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


# 查看详情
[root@k8s-master ingress]# kubectl describe ing all-https -n dev


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
https://gateway.k8s.com:32099/dconsumer/hello
https://gateway.k8s.com:32099/dconsumer/say
https://gateway.k8s.com:32099/dproducer/hello
https://gateway.k8s.com:32099/dproducer/say
```



### 3.Prometheus监控

#### 3.1.配置文件

**1.Metrics端点**

```shell
# 消费者 msa-deploy-consumer
http://gateway.k8s.com:31208/dconsumer/actuator/prometheus

# 生产者 msa-deploy-producer
http://gateway.k8s.com:31208/dproducer/actuator/prometheus
```



2.Prometheus配置文件

```yaml
# Ingress方式配置

  - job_name: msa-deploy-producer
    metrics_path: '/dproducer/actuator/prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['gateway.k8s.com:31208']
        labels:
          instance: producer

  - job_name: msa-deploy-consumer
    metrics_path: '/dconsumer/actuator/prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['gateway.k8s.com:31208']
        labels:
          instance: consumer
```

```yaml
# 完整配置文件


# prometheus.yml：
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - 172.17.88.22:9093
   
rule_files:
  - "rules/*.yml"
 
scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']
        labels:
          instance: prometheus
     
  - job_name: alertmanager
    scrape_interval: 5s
    static_configs:
      - targets: ['172.17.88.22:9093']
        labels:
          instance: alert

  - job_name: msa-deploy-producer
    metrics_path: '/dproducer/actuator/prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['gateway.k8s.com:31208']
        labels:
          instance: producer

  - job_name: msa-deploy-consumer
    metrics_path: '/dconsumer/actuator/prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['gateway.k8s.com:31208']
        labels:
          instance: consumer
```



#### 3.2.安装Prometheus

```shell
### 安装prometheus
 
# 创建目录
mkdir -p /ntms/prometheus
mkdir -p /ntms/prometheus/data
mkdir -p /ntms/prometheus/rules
chmod 777 -R /ntms/prometheus/data
 
 
# 创建配置文件
/ntms/prometheus/prometheus.yml
touch prometheus.yml
 
 
# 修改配置文件
/prometheus/prometheus/prometheus.yml
 

***************************************************************
### 运行docker 
docker run -d --network host --name prometheus --restart=always \
-v /ntms/prometheus:/etc/prometheus \
-v /ntms/prometheus/data:/prometheus \
-e TZ=Asia/Shanghai \
prom/prometheus
 

### 访问地址
http://172.51.216.98:9090
```



#### 3.3.配置Grafana

```shell
### 配置Grafana模板
 
# 搜索官网模板
https://grafana.com/
https://grafana.com/grafana/dashboards
 
# 根据模板ID查找模板
ID: XXXX
https://grafana.com/grafana/dashboards/XXXX
 
### 选择模板
# 4701
https://grafana.com/grafana/dashboards/4701
```



#### 3.4.配置本地hosts

 ```shell
 # 配置本地hosts
 
 [root@hadoop101 mort]# vi /etc/hosts
 
 127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
 ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
 
 172.51.216.81 gateway.k8s.com
 ```



