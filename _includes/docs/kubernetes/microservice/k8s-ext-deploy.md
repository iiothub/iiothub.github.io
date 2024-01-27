* TOC
{:toc}



## 一、概述



kubernetes部署无状态服务，有状态服务部署到k8s外部。

1.微服务基本都是无状态服务，部署到k8s内部

2.中间件基本都是有状态服务，部署到k8s外部



### 1.K8S访问外部服务

k8s集群内的pod需要访问mysql，由于mysql的性质，不适合部署在k8s集群内，故k8s集群内的应用需要链接mysql时，需要配置链接外网的mysql,本次测试 k8s集群ip段为`192.168.23.xx`。以下提供两种方式，`Endpoint`和`ExternalName`方式。



Service可以抽象访问Pod集群，同时 Service也可以抽象其他后端

- 在生产环境中使用外部数据库，在测试环境中使用自己的数据库
- 将自己的Service指向其他集群或者其他命名空间的Service
- 迁移应用到k8s，但是还是有些应用运行在k8s之



#### 1.1.ExternalName方式

通过完全限定域名（FQDN）访问外部服务——创建ExternalName类型的服务。

![](/images/kubernetes/microservice/service-18.png)

ExternalName类型的服务创建后，pod可以通过external-service.default.svc.cluster.local域名连接到外部服务，或者通过externale-service。当需要指向其他外部服务时，只需要修改spec.externalName的值即可。

```tex
说明： ExternalName 接受 IPv4 地址字符串，但作为包含数字的 DNS 名称，而不是 IP 地址。 类似于 IPv4 地址的外部名称不能由 CoreDNS 或 ingress-nginx 解析，因为外部名称旨在指定规范的 DNS 名称。 要对 IP 地址进行硬编码，请考虑使用 [headless Services](#headless-services)。
```



**参考1：**

```shell
> mkdir -p  ~/mysql-endpoint
> cd ~/mysql-endpoin
> cat <<EOF > my-mysql-external.yaml


apiVersion: v1
kind: Service
metadata:
  name: my-mysql-external #此名字随便起
  namespace: my-first-app  #在固定的命名空间下
spec:
  type: ExternalName 
  externalName: www.baidu.com ##提供方的服务完全限定域名,如rds域名等。
  ports:
    - port: 80
    
#  ExternalName类型的服务创建后，pod可以通过my-mysql-external.default.svc.cluster.local域名连接到外部服务，
#  或者通过my-mysql-external。当需要指向其他外部服务时，只需要修改spec.externalName的值即可。

EOF
```



```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```



#### 1.2.Endpoint方式

​       **endpoint介绍**
  服务和pod不是直接连接，而是通过Endpoint资源进行连通。endpoint资源是暴露一个服务的ip地址和port的列表。
  选择器用于构建ip和port列表，然后存储在endpoint资源中。当客户端连接到服务时，服务代理选择这些列表中的ip和port对中的一个，并将传入连接重定向到在该位置监听的服务器。
  endpoint是一个单独的资源并不是服务的属性，endpoint的名称必须和服务的名称相匹配。

​        **创建**
  为没有选择器的服务创建endpoint资源：$ kubectl create -f endpoint.yml
  endpoint对象需要与服务相同的名称，并包含该服务的目标ip和port列表，服务和endpoint资源都发布到服务器后，这样服务就可以像具有pod选择器那样的服务正常使用。

![](/images/kubernetes/microservice/service-19.png)



```shell
# 查看服务的Endpoint
$ kubectl describe svc service_name

# 通过kubectl describe可以看到Endpoints字段。
# 查看endpoints信息
$ kubectl get endpoints endpoint_name
```



**通过endpoint配置外部服务**

将pod关联到外部具有两个endpoint的服务上。

![](/images/kubernetes/microservice/service-20.png)



**实例1：**

创建 endpoints

创建`my-mysql-endpoints.yaml`

```shell
> mkdir -p  ~/mysql-endpoint
> cd ~/mysql-endpoint

> cat <<EOF > my-mysql-endpoints.yaml

apiVersion: v1
kind: Endpoints
apiVersion: v1
metadata:
  name: my-mysql-endpoint #此名字需与 my-mysql-service.yaml文件中的 metadata.name 的值一致
  namespace: my-first-app #在固定的命名空间下
subsets:
  - addresses:
      - ip: 192.168.23.1 ## 宿主机，由于我的虚拟机ping不通我本机的mysql（安装mysql时候禁用了未打开）
      - ip: 220.181.38.148 ## 随便取的一个 公网Ip
    ports:
      - port: 3306
EOF
```

创建service

创建`my-mysql-service.yaml`

```shell
> cat <<EOF > my-mysql-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-mysql-endpoint #此名字需与 my-mysql-endpoints.yaml文件中的 metadata.name 的值一致
  namespace: my-first-app  #在固定的命名空间下
spec:
  ports:
    - port: 3306
    
###  验证，进入，ping一下配置的Ip地址。
# kubectl exec -it my-first-springcloud-94cdd7487-xxxxx -n  my-first-app -- /bin/sh
# ping 220.181.38.148

EOF 
```

部署pod 进行验证

部署自己的服务到 `my-first-app`命名空间下，并且进入容器，进行ping测试

```shell
> kubectl exec -it my-first-springcloud-94cdd7487-xxxxx -n  my-first-app -- /bin/sh
> ping 220.181.38.148  ## 也可以在容器中直接调用对应的端口测试，此处只演示ping通，就代表能访问了
PING 220.181.38.148 (220.181.38.148): 56 data bytes
64 bytes from 220.181.38.148: seq=0 ttl=127 time=5.356 ms
64 bytes from 220.181.38.148: seq=1 ttl=127 time=4.946 ms
64 bytes from 220.181.38.148: seq=2 ttl=127 time=18.165 ms
。。。。


> 也可以查看service的 Endpoints 信息 
> kubectl describe svc my-mysql-endpoint -n my-first-app

Name:              my-mysql-endpoint
Namespace:         my-first-app
Labels:            <none>
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP:                10.102.160.141
Port:              <unset>  3306/TCP
TargetPort:        3306/TCP
Endpoints:         192.168.23.1:3306,220.181.38.148:3306
Session Affinity:  None
Events:            <none>
```

**注意**

此种方式只适合Ip访问，对于像阿里云rds等数据库的。需要用域名。则需要用`ExternalName`方式不而不是`Endpoints`方式。



**实例2：**

1.创建mysql-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-production
  namespace: ms
spec:
  ports:
    - port: 3306
      targetPort: 3306
      protocol: TCP
```

2.创建mysql-endpoints.yaml

```yaml
kind: Endpoints
apiVersion: v1
metadata:
  name: mysql-production
  namespace: ms
subsets:
  - addresses:
      - ip: 10.255.20.176
    ports:
      - port: 3306
```



**实例3：**

**访问 ES**

外部的 es ip为 192.168.0.200 端口为 9400；这里创建一个 Endpoints 和 Service。

```yaml
cat es.yaml 

apiVersion: v1
kind: Service
metadata:
  name: es-svc
  namespace: klvchen
spec:
  ports:
  - port: 9400
    targetPort: 9400
    protocol: TCP
    name: tcp
---
apiVersion: v1
kind: Endpoints
metadata:
  name: es-svc
  namespace: klvchen
subsets:
  - addresses:
    - ip: 192.168.0.200
    ports:
    - port: 9400
      name: tcp

# 在 K8S 中的容器使用 es-svc.klvchen.svc.cluster.local:9400 就可以访问到 es 了。
```

**访问 Kibana**

通过 ingress 管理 Kibana，让用户通过 kibana.klvchen.com 这个域名来访问 192.168.0.200 上的 kibana。
注意：kibana.klvchen.com 域名需要解析到 ingress 上

```yaml
cat kibana.yaml 

apiVersion: v1
kind: Service
metadata:
  name: kibana-svc
  namespace: klvchen
spec:
  ports:
  - port: 5601
    targetPort: 5601
    protocol: TCP
    name: http
---
apiVersion: v1
kind: Endpoints
metadata:
  name: kibana-svc
  namespace: klvchen
subsets:
  - addresses:
    - ip: 192.168.0.200
    ports:
    - port: 5601
      name: http
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kibana
  namespace: klvchen
spec:
  rules:
    - host: kibana.klvchen.com
      http:
        paths:
          - path: /
            backend:
              serviceName: kibana-svc
              servicePort: http
```



### 2.ExternalIP

如果外部的 IP 路由到集群中一个或多个 Node 上，Kubernetes Service 会被暴露给这些 externalIPs。通过外部 IP（作为目的 IP 地址）进入到集群，打到 Service 端口上的流量，将会被路由到 Service 的 Endpoint 上。

externalIPs 不会被 Kubernetes 管理，它属于集群管理员的职责范畴。

根据 Service 的规定，externalIPs 可以同任意的 ServiceType 来一起指定。在下面的例子中，my-service 可以在【模拟外网IP】“10.0.0.240”(externalIP:port) 上被客户端访问。





## 二、基础中间件



### 1.PostgreSQL

#### 1.1.创建Endpoint

 postgresql.yml

```yaml
# postgresql.yml


apiVersion: v1
kind: Service
metadata:
  name: postgresql-svc
  namespace: dev
spec:
  ports:
  - port: 5432
    targetPort: 5432
    protocol: TCP

---
apiVersion: v1
kind: Endpoints
metadata:
  name: postgresql-svc
  namespace: dev
subsets:
  - addresses:
    - ip: 182.92.210.65
    ports:
    - port: 5432
```

```shell
[root@k8s-master postgresql]# vim postgresql.yml


[root@k8s-master postgresql]# kubectl apply -f postgresql.yml 
service/postgresql-svc created
endpoints/postgresql-svc created


[root@k8s-master postgresql]# kubectl describe ep postgresql-svc -n dev
Name:         postgresql-svc
Namespace:    dev
Labels:       <none>
Annotations:  <none>
Subsets:
  Addresses:          182.92.210.65
  NotReadyAddresses:  <none>
  Ports:
    Name     Port  Protocol
    ----     ----  --------
    <unset>  5432  TCP

Events:  <none>


[root@k8s-master postgresql]# kubectl describe svc  postgresql-svc -n dev
Name:              postgresql-svc
Namespace:         dev
Labels:            <none>
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Families:       <none>
IP:                10.98.186.25
IPs:               10.98.186.25
Port:              <unset>  5432/TCP
TargetPort:        5432/TCP
Endpoints:         182.92.210.65:5432
Session Affinity:  None
Events:            <none>


# K8S 中的容器使用访问
svcname.namespace.svc.cluster.local:port
postgresql-svc.dev.svc.cluster.local:5432
```



#### 1.2.创建镜像

Dockerfile

```dockerfile
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
# docker build -t 172.51.216.85:8888/springcloud/msa-ext-postgresql:1.0.0 .
```

```shell
# 1.创建镜像
# docker build -t 172.51.216.85:8888/springcloud/msa-ext-postgresql:1.0.0 .
[root@k8s-master postgresql]# docker build -t 172.51.216.85:8888/springcloud/msa-ext-postgresql:1.0.0 .
Sending build context to Docker daemon  38.65MB
Step 1/9 : FROM openjdk:8-jdk-alpine
 ---> a3562aa0b991
Step 2/9 : VOLUME /temp
 ---> Using cache
 ---> ac1307e171b0
Step 3/9 : ENV JVM_OPS="-Xms256m -Xmx256m -XX:PermSize=512M -XX:MaxPermSize=512m"
 ---> Using cache
 ---> 26a5ca6d54c4
Step 4/9 : ENV JAVA_POS=""
 ---> Using cache
 ---> bf77f79bc401
Step 5/9 : ENV ACTIVE="-Dspring.profiles.active=test"
 ---> Using cache
 ---> 994f1ecfad1f
Step 6/9 : ADD *.jar app.jar
 ---> 1d58ae821a58
Step 7/9 : ENTRYPOINT java -Djava.security.egd=file:/dev/./urandom -jar ${JVM_OPS} ${ACTIVE} app.jar ${JAVA_OPS}
 ---> Running in c522080a58f8
Removing intermediate container c522080a58f8
 ---> 5a108972ce58
Step 8/9 : ENV TZ=Asia/Shanghai
 ---> Running in b67877aab643
Removing intermediate container b67877aab643
 ---> 2e81b08253e8
Step 9/9 : RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
 ---> Running in 3d8cfa22bd5b
Removing intermediate container 3d8cfa22bd5b
 ---> 83d3a622e5c1
Successfully built 83d3a622e5c1
Successfully tagged 172.51.216.85:8888/springcloud/msa-ext-postgresql:1.0.0


# 8-jdk-alpine 镜像小了很多
[root@k8s-master postgresql]# docker images
REPOSITORY                                                        TAG            IMAGE ID       CREATED          SIZE
172.51.216.85:8888/springcloud/msa-ext-postgresql                 1.0.0          83d3a622e5c1   34 seconds ago   143MB
...


# 2.推送镜像
[root@k8s-master postgresql]# docker push 172.51.216.85:8888/springcloud/msa-ext-postgresql:1.0.0
The push refers to repository [172.51.216.85:8888/springcloud/msa-ext-postgresql]
3a38d43ebd51: Pushed 
249d30d27f82: Pushed 
ceaf9e1ebef5: Mounted from springcloud/msa-zipkin-producer 
9b9b7f3d56a0: Mounted from springcloud/msa-zipkin-producer 
f1b5933fe4b5: Mounted from springcloud/msa-zipkin-producer 
1.0.0: digest: sha256:4df6d2fe5f8ac8aaac7c06023c18522a509eefbe8be990870d0279b4b8a681f6 size: 1366
```



#### 1.3.创建服务

msa-ext-postgresql.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-ext-postgresql
  labels:
    app: msa-ext-postgresql
spec:
  type: ClusterIP
  ports:
    - port: 8111
      targetPort: 8111
  selector:
    app: msa-ext-postgresql

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-ext-postgresql
spec:
  replicas: 2
  selector:
    matchLabels:
      app: msa-ext-postgresql
  template:
    metadata:
      labels:
        app: msa-ext-postgresql
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-ext-postgresql
          image: 172.51.216.85:8888/springcloud/msa-ext-postgresql:1.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8111
          env:
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
```



```shell
[root@k8s-master postgresql]# vim msa-ext-postgresql.yaml


# 创建
[root@k8s-master postgresql]# kubectl apply -f msa-ext-postgresql.yaml 
service/msa-ext-postgresql created
deployment.apps/msa-ext-postgresql created


# 查看
[root@k8s-master postgresql]# kubectl get all -n dev | grep msa-ext-postgresql
pod/msa-ext-postgresql-75f6bf657d-mngkd   1/1     Running   0          9m53s
pod/msa-ext-postgresql-75f6bf657d-tzf5x   1/1     Running   0          9m53s

service/msa-ext-postgresql    ClusterIP   10.106.82.105    <none>        8111/TCP          9m53s

deployment.apps/msa-ext-postgresql    2/2     2            2           9m53s

replicaset.apps/msa-ext-postgresql-75f6bf657d   2         2         2       9m53s


[root@k8s-master postgresql]# curl http://10.106.82.105:8111/hello
[UserInfo(id=3, name=CCC, age=30), UserInfo(id=4, name=DDD, age=40), UserInfo(id=5, name=EEE, age=50), UserInfo(id=6, name=FFF, age=60), UserInfo(id=7, name=GGG, age=70), UserInfo(id=8, name=HHH, age=80), UserInfo(id=9, name=III, age=90), UserInfo(id=10, name=JJJ, age=100), UserInfo(id=2, name=BBB, age=88), UserInfo(id=1, name=AAA, age=33)]
```



#### 1.4.测试

访问地址：

curl http://10.106.82.105:8111/hello

**注意：经过测试直接使用IP地址也可以访问，不用配置端点**



#### 1.5.搭建Postgresql数据库

```shell
docker run -d --name timescaledb --network host --restart=always \
-e LANG="C.UTF-8" \
-e 'TZ=Asia/Shanghai' \
-e "POSTGRES_DB=postgres" \
-e "POSTGRES_USER=postgres" \
-e "POSTGRES_PASSWORD=postgres" \
-v /k8s/pg/data/psql:/var/lib/postgresql/data \
timescale/timescaledb:2.1.0-pg12


182.92.210.65
5432
postgres/postgres
```



### 2.MySQL

#### 2.1.创建Endpoint

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
    - ip: 182.92.210.65
    ports:
    - port: 3306
```

```shell
[root@k8s-master mysql]# vim mysql.yml


[root@k8s-master mysql]# kubectl apply -f mysql.yml 
service/mysql-svc created
endpoints/mysql-svc created


[root@k8s-master mysql]# kubectl describe ep mysql-svc -n dev
Name:         mysql-svc
Namespace:    dev
Labels:       <none>
Annotations:  <none>
Subsets:
  Addresses:          182.92.210.65
  NotReadyAddresses:  <none>
  Ports:
    Name     Port  Protocol
    ----     ----  --------
    <unset>  3306  TCP

Events:  <none>


[root@k8s-master mysql]# kubectl describe svc  mysql-svc -n dev
Name:              mysql-svc
Namespace:         dev
Labels:            <none>
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Families:       <none>
IP:                10.99.194.252
IPs:               10.99.194.252
Port:              <unset>  3306/TCP
TargetPort:        3306/TCP
Endpoints:         182.92.210.65:3306
Session Affinity:  None
Events:            <none>


# K8S 中的容器使用访问
svcname.namespace.svc.cluster.local:port
mysql-svc.dev.svc.cluster.local:3306
```



#### 2.2.创建镜像

Dockerfile

```dockerfile
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
# docker build -t 172.51.216.85:8888/springcloud/msa-ext-mysql:1.0.0 .
```

```shell
# 1.创建镜像
# docker build -t 172.51.216.85:8888/springcloud/msa-ext-mysql:1.0.0 .
[root@k8s-master mysql]# docker build -t 172.51.216.85:8888/springcloud/msa-ext-mysql:1.0.0 .


# 8-jdk-alpine 镜像小了很多
[root@k8s-master mysql]# docker images
REPOSITORY                                                        TAG            IMAGE ID       CREATED          SIZE
172.51.216.85:8888/springcloud/msa-ext-mysql                      1.0.0          0619f0e88e25   28 seconds ago   145MB
...


# 2.推送镜像
[root@k8s-master mysql]# docker push 172.51.216.85:8888/springcloud/msa-ext-mysql:1.0.0
```



#### 2.3.创建服务

msa-ext-mysql.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-ext-mysql
  labels:
    app: msa-ext-mysql
spec:
  type: ClusterIP
  ports:
    - port: 8112
      targetPort: 8112
  selector:
    app: msa-ext-mysql

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-ext-mysql
spec:
  replicas: 2
  selector:
    matchLabels:
      app: msa-ext-mysql
  template:
    metadata:
      labels:
        app: msa-ext-mysql
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-ext-mysql
          image: 172.51.216.85:8888/springcloud/msa-ext-mysql:1.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8112
          env:
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
```



```shell
[root@k8s-master mysql]# vim msa-ext-mysql.yaml


# 创建
[root@k8s-master mysql]# kubectl apply -f msa-ext-mysql.yaml 
service/msa-ext-mysql created
deployment.apps/msa-ext-mysql created


# 查看
[root@k8s-master mysql]# kubectl get all -n dev | grep msa-ext-mysql
pod/msa-ext-mysql-79f5594b6b-2xq7r        1/1     Running   0          48s
pod/msa-ext-mysql-79f5594b6b-jcvwx        1/1     Running   0          48s

service/msa-ext-mysql         ClusterIP   10.111.50.177    <none>        8112/TCP          48s

deployment.apps/msa-ext-mysql         2/2     2            2           48s

replicaset.apps/msa-ext-mysql-79f5594b6b        2         2         2       48s


[root@k8s-master mysql]# curl http://10.111.50.177:8112/hello
[UserInfo(id=1, name=AAA, age=10), UserInfo(id=2, name=BBB, age=20), UserInfo(id=3, name=CCC, age=30), UserInfo(id=4, name=DDD, age=40), UserInfo(id=5, name=EEE, age=50), UserInfo(id=6, name=FFF, age=60), UserInfo(id=7, name=GGG, age=70), UserInfo(id=8, name=HHH, age=80), UserInfo(id=9, name=III, age=90), UserInfo(id=10, name=JJJ, age=100)]
```



#### 2.4.测试

访问地址：

curl http://10.111.50.177:8112/hello

**注意：经过测试直接使用IP地址也可以访问，不用配置端点**



#### 2.5.搭建MySQL数据库

```shell
#创建目录
mkdir -p /k8s/mysql/conf
mkdir -p /k8s/mysql/data
chmod 777 * -R /k8s/mysql/conf
chmod 777 * -R /k8s/mysql/data
 
#创建配置文件
vim /k8s/mysql/conf/my.cnf
 
#输入一下内容
[mysqld]
log-bin=mysql-bin #开启二进制日志
server-id=119 #服务id，取本机IP最后三位
sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION'
 
 
#启动容器
docker run -d --restart=always \
-p 3306:3306 \
-v /k8s/mysql/data:/var/lib/mysql \
-v /k8s/mysql/conf:/etc/mysql/ \
-e MYSQL_ROOT_PASSWORD=root \
--name mysql \
percona:5.7.23
 
 
#进入mysql容器内部
docker exec -it mysql bash  
#登陆mysql
mysql -u root -p
```



### 3.RabbitMQ

#### 3.1.创建Endpoint

 rabbitmq.yml

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
[root@k8s-master msa-zipkin-producer]# kubectl apply -f rabbitmq.yml 
service/rabbitmq-svc unchanged
endpoints/rabbitmq-svc created


[root@k8s-master msa-zipkin-producer]# kubectl describe ep rabbitmq-svc -n dev
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


[root@k8s-master msa-zipkin-producer]# kubectl describe ep rabbitmq-svc -n dev
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
[root@k8s-master msa-zipkin-producer]# 
[root@k8s-master msa-zipkin-producer]# 
[root@k8s-master msa-zipkin-producer]# kubectl describe svc rabbitmq-svc -n dev
Name:              rabbitmq-svc
Namespace:         dev
Labels:            <none>
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Families:       <none>
IP:                10.101.38.222
IPs:               10.101.38.222
Port:              <unset>  5672/TCP
TargetPort:        5672/TCP
Endpoints:         172.51.216.98:5672
Session Affinity:  None
Events:            <none>


# K8S 中的容器使用访问
svcname.namespace.svc.cluster.local:port
rabbitmq-svc.dev.svc.cluster.local:5672
```



#### 3.2.测试

在**调用链监控（Zipkin）**测试中已经测试通过，不再测试，原理与其他基础中间件相同。

**注意：经过测试直接使用IP地址也可以访问，不用配置端点**



### 4.Redis

#### 4.1.创建Endpoint

 redis.yml

```yaml
# redis.yml


apiVersion: v1
kind: Service
metadata:
  name: redis-svc
  namespace: dev
spec:
  ports:
  - port: 26379
    targetPort: 26379
    protocol: TCP

---
apiVersion: v1
kind: Endpoints
metadata:
  name: redis-svc
  namespace: dev
subsets:
  - addresses:
    - ip: 182.92.210.65
    ports:
    - port: 26379
```

```shell
[root@k8s-master redis]# vim redis.yml


[root@k8s-master redis]#  kubectl apply -f redis.yml 
service/redis-svc created
endpoints/redis-svc created


[root@k8s-master redis]# kubectl describe ep redis-svc -n dev
Name:         redis-svc
Namespace:    dev
Labels:       <none>
Annotations:  <none>
Subsets:
  Addresses:          182.92.210.65
  NotReadyAddresses:  <none>
  Ports:
    Name     Port   Protocol
    ----     ----   --------
    <unset>  26379  TCP

Events:  <none>


[root@k8s-master redis]# kubectl describe svc  redis-svc -n dev
Name:              redis-svc
Namespace:         dev
Labels:            <none>
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Families:       <none>
IP:                10.104.67.141
IPs:               10.104.67.141
Port:              <unset>  26379/TCP
TargetPort:        26379/TCP
Endpoints:         182.92.210.65:26379
Session Affinity:  None
Events:            <none>


# K8S 中的容器使用访问
svcname.namespace.svc.cluster.local:port
redis-svc.dev.svc.cluster.local:26379
```



#### 4.2.创建镜像

Dockerfile

```dockerfile
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
# docker build -t 172.51.216.85:8888/springcloud/msa-ext-redis:1.0.0 .
```

```shell
# 1.创建镜像
# docker build -t 172.51.216.85:8888/springcloud/msa-ext-redis:1.0.0 .
[root@k8s-master redis]# docker build -t 172.51.216.85:8888/springcloud/msa-ext-redis:1.0.0 .


# 8-jdk-alpine 镜像小了很多
[root@k8s-master redis]# docker images
REPOSITORY                                                        TAG            IMAGE ID       CREATED          SIZE
172.51.216.85:8888/springcloud/msa-ext-redis                      1.0.0          c5ba87a89010   24 seconds ago   132MB
...


# 2.推送镜像
[root@k8s-master mysql]# docker push 172.51.216.85:8888/springcloud/msa-ext-redis:1.0.0
```



#### 4.3.创建服务

msa-ext-redis.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-ext-redis
  labels:
    app: msa-ext-redis
spec:
  type: ClusterIP
  ports:
    - port: 8113
      targetPort: 8113
  selector:
    app: msa-ext-redis

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-ext-redis
spec:
  replicas: 2
  selector:
    matchLabels:
      app: msa-ext-redis
  template:
    metadata:
      labels:
        app: msa-ext-redis
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-ext-redis
          image: 172.51.216.85:8888/springcloud/msa-ext-redis:1.0.2
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8113
          env:
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
```



```shell
[root@k8s-master redis]# vim msa-ext-redis.yaml


# 创建
[root@k8s-master redis]# kubectl apply -f msa-ext-redis.yaml 
service/msa-ext-redis created
deployment.apps/msa-ext-redis created


# 查看
[root@k8s-master redis]# kubectl get all -n dev | grep msa-ext-redis
pod/msa-ext-redis-98c67b6b-kx8n8          1/1     Running   0          9s
pod/msa-ext-redis-98c67b6b-stsw6          1/1     Running   0          9s

service/msa-ext-redis         ClusterIP   10.108.244.13    <none>        8113/TCP          9s

deployment.apps/msa-ext-redis         2/2     2            2           9s

replicaset.apps/msa-ext-redis-98c67b6b          2         2         2       9s


[root@k8s-master redis]# curl http://10.108.244.13:8113/hello
hello，this is redis messge! ### Mon Nov 01 18:57:00 CST 2021
```



#### 4.4.测试

访问地址：

 curl http://10.108.244.13:8113/hello

**注意：经过测试直接使用IP地址也可以访问，不用配置端点**



#### 4.5.搭建Redis

```shell
# 创建目录
mkdir -p /k8s/redis/master/log
mkdir -p /k8s/redis/master/data
# 对log目录进行读写授权
chmod 777 /k8s/redis/master/log


# 获取配置文件
1.获取redis配置文件
redis官方提供了一个配置文件样例，通过wget工具下载下来。我用的root用户，就直接下载到/root目录里了
wget http://download.redis.io/redis-stable/redis.conf

注意：下载6.0的安装包获取配置文件
 
# 修改配置文件
 
 
# 注释这一行，表示Redis可以接受任意ip的连接
# bind 127.0.0.1
 
# 端口
port 6379 
 
# 关闭保护模式
protected-mode no 
 
# 让redis服务后台运行
daemonize no
 
# 开启数据持久化
appendonly yes
 
# 设定密码(可选，如果这里开启了密码要求，slave的配置里就要加这个密码. 只是练习配置，就不使用密码认证了)
# requirepass Mypwd@123456
 
# 从库时需要增加主库配置
# 主库密码（一个集群密码需要保持一致）
masterauth Mypwd@123456
 
# 配置日志路径，为了便于排查问题，指定redis的日志文件目录
logfile "/var/log/redis/redis.log"

 
***************************************************************
# 拉取镜像
 
docker pull redis:6.0


# 运行master容器

docker run -d --network host --restart=always \
--name redis \
--privileged=true \
-v /k8s/redis/master/redis.conf:/usr/local/etc/redis/redis.conf \
-v /k8s/redis/master/data:/data \
-v /k8s/redis/master/log:/var/log/redis \
redis:6.0 \
redis-server /usr/local/etc/redis/redis.conf
```



### 5.Elasticsearch

#### 5.1.创建Endpoint

 elasticsearch.yml

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
[root@k8s-master msa-zipkin-producer]# kubectl apply -f elasticsearch.yml 
service/elasticsearch-svc created
endpoints/elasticsearch-svc created


[root@k8s-master msa-zipkin-producer]# kubectl describe ep elasticsearch-svc -n dev
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


[root@k8s-master msa-zipkin-producer]# kubectl describe ep elasticsearch-svc -n dev
Name:         elasticsearch-svc
Namespace:    dev
Labels:       <none>
Annotations:  <none>
Subsets:
  Addresses:          172.51.216.98
  NotReadyAddresses:  <none>
  Ports:
    Name     Port  Protocol
    ----     ----  --------
    <unset>  9200  TCP

Events:  <none>


[root@k8s-master msa-zipkin-producer]#  kubectl describe svc elasticsearch-svc -n dev
Name:              elasticsearch-svc
Namespace:         dev
Labels:            <none>
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Families:       <none>
IP:                10.105.9.231
IPs:               10.105.9.231
Port:              <unset>  9200/TCP
TargetPort:        9200/TCP
Endpoints:         172.51.216.98:9200
Session Affinity:  None
Events:            <none>



# K8S 中的容器使用访问
svcname.namespace.svc.cluster.local:port
elasticsearch-svc.dev.svc.cluster.local:9200
```



#### 5.2.测试

在**调用链监控（Zipkin）**测试中已经测试通过，不再测试，原理与其他基础中间件相同。

**注意：经过测试直接使用IP地址也可以访问，不用配置端点。**





## 三、系统中间件



### 1.调用链监控（Zipkin）

#### 1.1.Zipkin

**1.Zipkin 服务端**

```shell
关于 Zipkin 的服务端，在使用 Spring Boot 2.x 版本后，官方就不推荐自行定制编译了，反而是直接提供了编译好的 jar 包来给我们使用，
详情请看 https://github.com/openzipkin/zipkin/issues/1962 
并且以前的@EnableZipkinServer也已经被打上了@Deprecated

If you decide to make a custom server, you accept responsibility for troubleshooting your build or configuration problems, even if such problems are a reaction to a change made by the OpenZipkin maintainers. In other words, custom servers are possible, but not supported.
EnableZipkinServer.javagithub.com/openzipkin/zipkin/blob/master/zipkin-server/src/main/java/zipkin/server/EnableZipkinServer.java
简而言之就是：私自改包，后果自负。

所以官方提供了一键脚本 

curl -sSL https://zipkin.io/quickstart.sh | bash -s
java -jar zipkin.jar

如果用 Docker 的话，直接
docker run -d -p 9411:9411 openzipkin/zipkin 

访问 http://8.131.70.152:9411/zipkin/
```



**2.通信方式**

zipkin客户端与服务端通信方式：
HTTP方式
RabbitMQ方式



**Rabbitmq方式**

```shell
依赖加上rabbitmq的依赖，依赖如下：
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
</dependency>

在配置文件上需要配置rabbitmq的配置，配置信息如下：
spring:
  rabbitmq:
    host: localhost
    username: guest
    password: guest
    port: 5672
```



**3.存储方式**

```shell
# zipkin是支持将链路数据存储在mysql、cassandra、elasticsearch中的。
https://github.com/openzipkin/zipkin/tree/master/zipkin-storage


# masql
https://github.com/openzipkin/zipkin/tree/master/zipkin-storage/mysql-v1 
https://github.com/openzipkin/zipkin/blob/master/zipkin-storage/mysql-v1/src/main/resources/mysql.sql
```



![](/images/kubernetes/microservice/ext-1.png)

```shell
STORAGE_TYPE=mysql MYSQL_HOST=localhost MYSQL_TCP_PORT=3306 MYSQL_USER=root MYSQL_PASS=123456 MYSQL_DB=zipkin java -jar zipkin.jar
 
java -jar zipkin.jar --zipkin.torage.type=mysql --zipkin.torage.mysql.host=localhost --zipkin.torage.mysql.port=3306 --zipkin.torage.mysql.username=root --zipkin.torage.mysql.password=123456
```



**Elasticsearch**

```shell
https://github.com/openzipkin/zipkin/tree/master/zipkin-storage/elasticsearch


zipkin-server支持将链路数据存储在ElasticSearch中。需要安装ElasticSearch和Kibana，
下载地址为https://www. elastic.co/products/elasticsearch。
安装完成后启动，其中ElasticSearch的默认端口号为9200，Kibana的默认端口号为5601。
同理，zipkin连接elasticsearch也是从环境变量中读取的，elasticsearch相关的环境变量和对应的属性如下：
```

![](/images/kubernetes/microservice/ext-2.png)

```shell
STORAGE_TYPE=elasticsearch ES_HOSTS=http://localhost:9200 ES_INDEX=zipkin java -jar zipkin.jar
 
java -jar zipkin.jar --STORAGE_TYPE=elasticsearch --ES_HOSTS=http://localhost:9200 --ES_INDEX=zipkin
 
java -jar zipkin.jar --STORAGE_TYPE=elasticsearch --ES_HOSTS=http://localhost:9200 --ES_INDEX=zipkin
 
java -jar zipkin.jar --zipkin.torage.type=elasticsearch --zipkin.torage.elasticsearch.hosts=http://localhost:9200 --zipkin.torage.elasticsearch.index=zipkin
```



**4.部署方式**

部署方式有：MQ方式、HTTP方式。推荐MQ方式。



**MQ方式**

![image-20211029134317548](/images/kubernetes/microservice/ext-7.png)



**HTTP方式**

![image-20211029134357670](/images/kubernetes/microservice/ext-8.png)



#### 1.2.部署Elasticsearch

```shell
# 下载镜像 查看镜像
docker pull elasticsearch:7.1.1


# 运行 elasticsearch
docker run -d --name elasticsearch --network host --restart=always \
-e "discovery.type=single-node" elasticsearch:7.1.1
 

# 检测 elasticsearch 是否启动成功
curl 127.0.0.1:9200

[root@localhost ~]# curl 127.0.0.1:9200
{
  "name" : "localhost.localdomain",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "mXpk62HvT3mVlqEdFBEdQA",
  "version" : {
    "number" : "7.1.1",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "7a013de",
    "build_date" : "2019-05-23T14:04:00.380842Z",
    "build_snapshot" : false,
    "lucene_version" : "8.0.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

```shell
# ES地址

http://172.51.216.98:9200
```



#### 1.3.部署Kibana

```shell
# 下载镜像 查看镜像
docker pull kibana:7.1.1
 

# 运行 Kibana
docker run -d --name kibana --network host --restart=always kibana:7.1.1


# 修改配置文件
docker exec -it kibana /bin/bash


/usr/share/kibana/config/kibana.yml

server.name: kibana
server.host: "0"
elasticsearch.hosts: [ "http://172.51.216.98:9200" ]
xpack.monitoring.ui.container.elasticsearch.enabled: true
```

```shell
#Kibana地址：

http://172.51.216.98:5601
```



#### 1.4.部署RabbitMQ

```shell
***************************************************************
# 安装准备工作

# 拉取镜像
# docker pull rabbitmq:management

 
# 运行容器
docker run -d  --network host --restart=always --name rabbitmq rabbitmq:management
 
 
# 容器要挂载的目录
docker exec -it rabbitmq /bin/bash
配置文件目录：/etc/rabbitmq
数据存储目录：/var/lib/rabbitmq
日志目录：/var/log/rabbitmq 
 
 
# 宿主机创建目录
mkdir -p  /ntms/rabbitmq/lib/
mkdir -p  /ntms/rabbitmq/etc/
mkdir -p  /ntms/rabbitmq/log/
 
 
# 容器内容复制到宿主机
docker cp -a rabbitmq:/var/lib/rabbitmq  /ntms/rabbitmq/lib/
docker cp -a rabbitmq:/etc/rabbitmq  /ntms/rabbitmq/etc/
docker cp -a rabbitmq:/var/log/rabbitmq  /ntms/rabbitmq/log/
 
 
# 删除容器
docker rm -f rabbitmq
 
 
***************************************************************
# 运行容器
 
docker run -d  --network host --restart=always --name rabbitmq \
-v /ntms/rabbitmq/etc/rabbitmq:/etc/rabbitmq \
-v /ntms/rabbitmq/lib/rabbitmq:/var/lib/rabbitmq \
-v /ntms/rabbitmq/log/rabbitmq/:/var/log/rabbitmq \
rabbitmq:management 
 
 
# 访问地址

http://172.51.216.98:15672/ 
账户/密码：
guest/guest 

5672
```



#### 1.5.Docker方式部署Zipkin服务

**1.RabbitMQ通信方式**

```shell
# 此种方式也支持HTTP方式

# 拉取镜像 
docker pull openzipkin/zipkin:2
# 最新版本
docker pull openzipkin/zipkin:latest


# 运行容器
docker run -d --name zipkin -p 9411:9411 --restart=always \
-e RABBIT_ADDRESSES=172.51.216.98:5672 \
-e RABBIT_USER=guest \
-e RABBIT_PASSWORD=guest \
-e STORAGE_TYPE=elasticsearch \
-e ES_HOSTS=http://172.51.216.98:9200 \
openzipkin/zipkin:latest
```



**2.HTTP通信方式**

```shell
# 最新版本
docker pull openzipkin/zipkin:latest


# 运行容器
docker run -d --name zipkin -p 9411:9411 --restart=always \
-e STORAGE_TYPE=elasticsearch \
-e ES_HOSTS=http://172.17.88.25:9200 \
openzipkin/zipkin:latest
```



```shell
# zipkin地址

http://172.51.216.98:9411/zipkin/
```



#### 1.6.创建外部服务Endpoint

##### 1.6.1.RabbitMQ

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
[root@k8s-master msa-zipkin-producer]# kubectl apply -f rabbitmq.yml 
service/rabbitmq-svc unchanged
endpoints/rabbitmq-svc created


[root@k8s-master msa-zipkin-producer]# kubectl describe ep rabbitmq-svc -n dev
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


[root@k8s-master msa-zipkin-producer]# kubectl describe svc rabbitmq-svc -n dev
Name:              rabbitmq-svc
Namespace:         dev
Labels:            <none>
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Families:       <none>
IP:                10.101.38.222
IPs:               10.101.38.222
Port:              <unset>  5672/TCP
TargetPort:        5672/TCP
Endpoints:         172.51.216.98:5672
Session Affinity:  None
Events:            <none>


# K8S 中的容器使用访问
svcname.namespace.svc.cluster.local:port
rabbitmq-svc.dev.svc.cluster.local:5672
```



##### 1.6.2.Elasticsearch

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
[root@k8s-master msa-zipkin-producer]# kubectl apply -f elasticsearch.yml 
service/elasticsearch-svc created
endpoints/elasticsearch-svc created


[root@k8s-master msa-zipkin-producer]# kubectl describe ep elasticsearch-svc -n dev
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


[root@k8s-master msa-zipkin-producer]# kubectl describe ep elasticsearch-svc -n dev
Name:         elasticsearch-svc
Namespace:    dev
Labels:       <none>
Annotations:  <none>
Subsets:
  Addresses:          172.51.216.98
  NotReadyAddresses:  <none>
  Ports:
    Name     Port  Protocol
    ----     ----  --------
    <unset>  9200  TCP

Events:  <none>


[root@k8s-master msa-zipkin-producer]#  kubectl describe svc elasticsearch-svc -n dev
Name:              elasticsearch-svc
Namespace:         dev
Labels:            <none>
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Families:       <none>
IP:                10.105.9.231
IPs:               10.105.9.231
Port:              <unset>  9200/TCP
TargetPort:        9200/TCP
Endpoints:         172.51.216.98:9200
Session Affinity:  None
Events:            <none>


# K8S 中的容器使用访问
svcname.namespace.svc.cluster.local:port
elasticsearch-svc.dev.svc.cluster.local:9200
```



#### 1.7.K8S部署Zipkin服务

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
```

```shell
# 创建
[root@k8s-master msa-zipkin-producer]# kubectl apply -f zipkin-server.yaml 
service/zipkin-server created
deployment.apps/zipkin-server created


[root@k8s-master msa-zipkin-producer]# kubectl get all -n dev | grep zipkin-server
pod/zipkin-server-84c5fdcf5d-qf4zh         1/1     Running   0          2m51s
pod/zipkin-server-84c5fdcf5d-x57zf         1/1     Running   0          2m51s

service/zipkin-server         NodePort    10.97.79.39      <none>        9411:30411/TCP    2m51s

deployment.apps/zipkin-server         2/2     2            2           2m51s

replicaset.apps/zipkin-server-84c5fdcf5d         2         2         2       2m51s


# 访问地址
http://172.51.216.81:30411/zipkin
```



#### 1.8.HTTP方式部署Zipkin生产者（msa-zipkin-producer）

##### 1.8.1.配置文件

**1.微服务配置文件**

```yaml
# application-k8shttp.yml


server:
  port: 8006

spring:
  application:
    name: msa-zipkin-producer
  sleuth:
    web:
      client:
        enabled: true
    sampler:
      probability: 1.0 # 将采样比例设置为 1.0，也就是全部都需要。默认是 0.1
  zipkin:
    base-url: http://zipkin-server.dev.svc.cluster.local:9411/ # 指定了 Zipkin 服务器的地址：svcname.namespace.svc.cluster.local:port

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
    
    
# K8S 中的容器使用访问
svcname.namespace.svc.cluster.local:port
zipkin-server.dev.svc.cluster.local:9411
```



**2.Dockerfile**

```dockerfile
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
# docker build -t 172.51.216.85:8888/springcloud/msa-zipkin-producer:1.0.0 .
```



**3.k8s-yaml**

msa-zipkin-producer.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-zipkin-producer
  labels:
    app: msa-zipkin-producer
spec:
  type: ClusterIP
  ports:
    - port: 8006
      targetPort: 8006
  selector:
    app: msa-zipkin-producer

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-zipkin-producer
spec:
  replicas: 2
  selector:
    matchLabels:
      app: msa-zipkin-producer
  template:
    metadata:
      labels:
        app: msa-zipkin-producer
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-zipkin-producer
          image: 172.51.216.85:8888/springcloud/msa-zipkin-producer:1.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8006
          env:
            - name: EUREKA_SERVER
              value: "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8shttp"
```



##### 1.8.2.创建镜像

```shell
# 1.上传JAR
把 msa-zipkin-producer-1.0.0.jar 上传到服务器（172.51.216.81），位置（/k8s/springcloud/zipkin）


# 2.创建Dockerfile
[root@k8s-master zipkin]# vim Dockerfile


[root@k8s-master zipkin]# ll
total 52580
-rw-r--r-- 1 root root      396 Oct 28 16:29 Dockerfile
-rw-r--r-- 1 root root 53835607 Oct 28 16:27 msa-zipkin-producer-1.0.0.jar


# 3.创建镜像
# docker build -t 172.51.216.85:8888/springcloud/msa-zipkin-producer:1.0.0 .
[root@k8s-master zipkin]# docker build -t 172.51.216.85:8888/springcloud/msa-zipkin-producer:1.0.0 .
Sending build context to Docker daemon  53.84MB
Step 1/9 : FROM openjdk:8-jdk-alpine
 ---> a3562aa0b991
Step 2/9 : VOLUME /temp
 ---> Using cache
 ---> ac1307e171b0
Step 3/9 : ENV JVM_OPS="-Xms256m -Xmx256m -XX:PermSize=512M -XX:MaxPermSize=512m"
 ---> Using cache
 ---> 26a5ca6d54c4
Step 4/9 : ENV JAVA_POS=""
 ---> Using cache
 ---> bf77f79bc401
Step 5/9 : ENV ACTIVE="-Dspring.profiles.active=test"
 ---> Using cache
 ---> 994f1ecfad1f
Step 6/9 : ADD *.jar app.jar
 ---> 8614329d8c8a
Step 7/9 : ENTRYPOINT java -Djava.security.egd=file:/dev/./urandom -jar ${JVM_OPS} ${ACTIVE} app.jar ${JAVA_OPS}
 ---> Running in 40e580252308
Removing intermediate container 40e580252308
 ---> 4c47cc9a73a2
Step 8/9 : ENV TZ=Asia/Shanghai
 ---> Running in 54b1a5a7ffe7
Removing intermediate container 54b1a5a7ffe7
 ---> 0dfe3967db37
Step 9/9 : RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
 ---> Running in 745ce576d614
Removing intermediate container 745ce576d614
 ---> c21950e00550
Successfully built c21950e00550
Successfully tagged 172.51.216.85:8888/springcloud/msa-zipkin-producer:1.0.0


# 8-jdk-alpine 镜像小了很多
[root@k8s-master zipkin]# docker images
REPOSITORY                                                        TAG            IMAGE ID       CREATED          SIZE
172.51.216.85:8888/springcloud/msa-zipkin-producer                1.0.0          c21950e00550   34 seconds ago   159MB
...


# 4.推送镜像
[root@k8s-master zipkin]# docker push 172.51.216.85:8888/springcloud/msa-zipkin-producer:1.0.0
The push refers to repository [172.51.216.85:8888/springcloud/msa-zipkin-producer]
8fa0f3371377: Pushed 
d413b7bd33d8: Pushed 
ceaf9e1ebef5: Mounted from springcloud/msa-admin-client 
9b9b7f3d56a0: Mounted from springcloud/msa-admin-client 
f1b5933fe4b5: Mounted from springcloud/msa-admin-client 
1.0.0: digest: sha256:23704a6481b27ae2d4b78eea4974828764db5e94cf7ea0f41c08b33430e02f54 size: 1366
```



##### 1.8.3.创建服务

```shell
[root@k8s-master zipkin]# vim msa-zipkin-producer.yaml


# 创建
[root@k8s-master zipkin]# kubectl apply -f msa-zipkin-producer.yaml 
service/msa-zipkin-producer created
deployment.apps/msa-zipkin-producer created


# 查看
[root@k8s-master zipkin]# kubectl get all -n dev | grep msa-zipkin-producer
pod/msa-zipkin-producer-55bc6b6dcc-4mw6f   1/1     Running   0          43s
pod/msa-zipkin-producer-55bc6b6dcc-8jc7h   1/1     Running   0          43s

service/msa-zipkin-producer   ClusterIP   10.104.207.255   <none>        8006/TCP          43s

deployment.apps/msa-zipkin-producer   2/2     2            2           43s

replicaset.apps/msa-zipkin-producer-55bc6b6dcc   2         2         2       43s


[root@k8s-master zipkin]# kubectl get pod -n dev -o wide | grep msa-zipkin-producer
msa-zipkin-producer-55bc6b6dcc-4mw6f   1/1     Running   0          101s    10.244.107.200   k8s-node3   <none>           <none>
msa-zipkin-producer-55bc6b6dcc-8jc7h   1/1     Running   0          101s    10.244.107.223   k8s-node3   <none>           <none>


[root@k8s-master zipkin]# curl 10.104.207.255:8006/hello
hello，this is first messge! ### Fri Oct 29 11:21:08 CST 2021
[root@k8s-master zipkin]# curl 10.104.207.255:8006/say
say，this is first messge！ $$$Fri Oct 29 11:21:30 CST 2021

```



##### 1.8.4.测试

访问地址：

 curl 10.104.207.255:8006/hello

 curl 10.104.207.255:8006/say

zipkin地址：

http://172.51.216.81:30411/zipkin/

Kibana地址：

http://172.51.216.98:5601

![](/images/kubernetes/microservice/ext-5.png)

![](/images/kubernetes/microservice/ext-6.png)



#### 1.9.RabbitMQ方式部署Zipkin生产者（msa-zipkin-producer）

##### 1.9.1.配置文件

**1.微服务配置文件**

```yaml
# application-k8smq.yml


server:
  port: 8006

spring:
  application:
    name: msa-zipkin-producer
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
    

# K8S 中的容器使用访问
svcname.namespace.svc.cluster.local:port
rabbitmq-svc.dev.svc.cluster.local:9411
```



**2.Dockerfile**

```dockerfile
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
# docker build -t 172.51.216.85:8888/springcloud/msa-zipkin-producer:1.0.0 .
```



**3.k8s-yaml**

msa-zipkin-producer.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-zipkin-producer
  labels:
    app: msa-zipkin-producer
spec:
  type: ClusterIP
  ports:
    - port: 8006
      targetPort: 8006
  selector:
    app: msa-zipkin-producer

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-zipkin-producer
spec:
  replicas: 2
  selector:
    matchLabels:
      app: msa-zipkin-producer
  template:
    metadata:
      labels:
        app: msa-zipkin-producer
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-zipkin-producer
          image: 172.51.216.85:8888/springcloud/msa-zipkin-producer:1.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8006
          env:
            - name: EUREKA_SERVER
              value: "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8smq"
```



##### 1.9.2.删除msa-zipkin-producer服务

```shell
[root@k8s-master msa-zipkin-producer]# ll
total 52584
-rw-r--r-- 1 root root      396 Oct 28 16:29 Dockerfile
-rw-r--r-- 1 root root 53835607 Oct 28 16:27 msa-zipkin-producer-1.0.0.jar
-rw-r--r-- 1 root root     1155 Oct 28 16:33 msa-zipkin-producer.yaml


# 1.删除服务
[root@k8s-master msa-zipkin-producer]# kubectl delete -f msa-zipkin-producer.yaml 
service "msa-zipkin-producer" deleted
deployment.apps "msa-zipkin-producer" deleted

[root@k8s-master msa-zipkin-producer]# kubectl get all -n dev | grep msa-zipkin-producer


# 2.修改 msa-zipkin-producer.yaml

            - name: ACTIVE
              value: "-Dspring.profiles.active=k8smq"              
```



##### 1.9.3.重新创建服务

```shell
# 创建
[root@k8s-master msa-zipkin-producer]# kubectl apply -f msa-zipkin-producer.yaml 
service/msa-zipkin-producer created
deployment.apps/msa-zipkin-producer created


# 查看
[root@k8s-master zipkin]# kubectl get all -n dev | grep msa-zipkin-producer
pod/msa-zipkin-producer-7d5485b79-p8w4c   1/1     Running   0          12s
pod/msa-zipkin-producer-7d5485b79-vj7gj   1/1     Running   0          12s

service/msa-zipkin-producer   ClusterIP   10.111.116.227   <none>        8006/TCP          12s

deployment.apps/msa-zipkin-producer   2/2     2            2           12s

replicaset.apps/msa-zipkin-producer-7d5485b79   2         2         2       12s


[root@k8s-master zipkin]# curl 10.111.116.227:8006/hello
hello，this is first messge! ### Fri Oct 29 11:35:33 CST 2021

[root@k8s-master zipkin]# curl 10.111.116.227:8006/say
say，this is first messge！ $$$Fri Oct 29 11:36:16 CST 2021
```



##### 1.9.4.测试

```shell
# 访问地址
curl 10.111.116.227:8006/hello
curl 10.111.116.227:8006/hello
curl 10.111.116.227:8006/say

# zipkin地址
http://172.51.216.81:30411/zipkin/

# Kibana地址
http://172.51.216.98:5601
```



![](/images/kubernetes/microservice/ext-5.png)

![](/images/kubernetes/microservice/ext-6.png)



```shell
# 参考

https://hub.docker.com/r/openzipkin/zipkin
https://zipkin.io/
https://github.com/openzipkin/zipkin


kubectl exec -it msa-zipkin-producer-777975c454-mhp7x -n  dev -- /bin/sh
kubectl logs -f msa-zipkin-producer-777975c454-mhp7x -n dev
```



### 2.流量控制（Sentinel）

#### 2.1.Sentinel

**Sentinel: 分布式系统的流量防卫兵**



**Sentinel 是什么？**

随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

Sentinel 具有以下特征:

- **丰富的应用场景**：Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制、实时熔断下游不可用应用等
- **完备的实时监控**：Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况
- **广泛的开源生态**：Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Apache Dubbo、gRPC、Quarkus 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。同时 Sentinel 提供 Java/Go/C++ 等多语言的原生实现
- **完善的 SPI 扩展机制**：Sentinel 提供简单易用、完善的 SPI 扩展接口。您可以通过实现扩展接口来快速地定制逻辑。例如定制规则管理、适配动态数据源等

**Sentinel 的主要特性：**

![](/images/kubernetes/microservice/ext-13.png)

**Sentinel 的开源生态：**

![](/images/kubernetes/microservice/ext-14.png)



Sentinel 分为两个部分:

- 核心库（Java 客户端）不依赖任何框架/库，能够运行于所有 Java 运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持
- 控制台（Dashboard）基于 Spring Boot 开发，打包后可以直接运行，不需要额外的 Tomcat 等应用容器



#### 2.2.Docker部署Sentinel服务

```shell
# 拉取镜像
docker pull bladex/sentinel-dashboard


# 运行镜像
docker run --name sentinel --restart=always -d --network host bladex/sentinel-dashboard


# 运行镜像
docker run --name sentinel --restart=always -d -p 8858:8858 -d bladex/sentinel-dashboard


# 访问dashboard
访问地址：http://172.51.216.98:8858/#/login
账户密码：sentinel/sentinel


# Linux运行
nohup  java -Dserver.port=8858 -Dcsp.sentinel.dashboard.server=localhost:8718 -Dproject.name=sentinel-dashboard -jar /usr/local/sentinel-dashboard-1.8.0.jar &
```





#### 2.3.K8S部署Sentinel服务

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
[root@k8s-master sentinel]# kubectl apply -f sentinel-server.yaml 
service/sentinel-server unchanged
deployment.apps/sentinel-server created


[root@k8s-master sentinel]#  kubectl get all -n dev | grep sentinel-server
pod/sentinel-server-5697bdc87b-qfc2b        0/1     ContainerCreating   0          59s

service/sentinel-server         NodePort    10.96.6.193      <none>        8858:30858/TCP    2m42s

deployment.apps/sentinel-server         0/1     1            0           59s

replicaset.apps/sentinel-server-5697bdc87b        1         1         0       59s


# 访问dashboard
访问地址：http://172.51.216.81:30858/#/login
账户密码：sentinel/sentinel
```

![](/images/kubernetes/microservice/ext-9.png)



![](/images/kubernetes/microservice/ext-10.png)



#### 2.4.创建Endpoint

 sentinel.yml

```yaml
# rabbitmq.yml


apiVersion: v1
kind: Service
metadata:
  name: sentinel-svc
  namespace: dev
spec:
  ports:
  - port: 8858
    targetPort: 8858
    protocol: TCP

---
apiVersion: v1
kind: Endpoints
metadata:
  name: sentinel-svc
  namespace: dev
subsets:
  - addresses:
    - ip: 172.51.216.98
    ports:
    - port: 8858
```



```shell
[root@k8s-master sentinel]# kubectl apply -f sentinel.yml 
service/sentinel-svc created
endpoints/sentinel-svc created


[root@k8s-master sentinel]#  kubectl describe ep sentinel-svc -n dev
Name:         sentinel-svc
Namespace:    dev
Labels:       <none>
Annotations:  <none>
Subsets:
  Addresses:          172.51.216.98
  NotReadyAddresses:  <none>
  Ports:
    Name     Port  Protocol
    ----     ----  --------
    <unset>  8858  TCP

Events:  <none>


[root@k8s-master sentinel]# kubectl describe svc sentinel-svc -n dev
Name:              sentinel-svc
Namespace:         dev
Labels:            <none>
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Families:       <none>
IP:                10.96.106.50
IPs:               10.96.106.50
Port:              <unset>  8858/TCP
TargetPort:        8858/TCP
Endpoints:         172.51.216.98:8858
Session Affinity:  None
Events:            <none>


# K8S 中的容器使用访问
svcname.namespace.svc.cluster.local:port
sentinel-svc.dev.svc.cluster.local:8858
```



#### 2.5.部署sentinel生产者（msa-sentinel-producer）

##### 2.5.1.配置文件

**1.微服务配置文件**

```yaml
# application-k8s.yml


server:
  port: 8211

spring:
  application:
    name: msa-sentinel-producer
  cloud:
    sentinel:
      transport:
        port: 8719
        dashboard: ${DASHBOARD:172.51.216.98:8858}

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



**2.Dockerfile**

```dockerfile
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
# docker build -t 172.51.216.85:8888/springcloud/msa-sentinel-producer:1.0.0 .
```



**3.k8s-yaml**

msa-sentinel-producer.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-sentinel-producer
  labels:
    app: msa-sentinel-producer
spec:
  type: ClusterIP
  ports:
    - port: 8211
      targetPort: 8211
  selector:
    app: msa-sentinel-producer

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-sentinel-producer
spec:
  replicas: 2
  selector:
    matchLabels:
      app: msa-sentinel-producer
  template:
    metadata:
      labels:
        app: msa-sentinel-producer
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-sentinel-producer
          image: 172.51.216.85:8888/springcloud/msa-sentinel-producer:1.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8211
          env:
            - name: EUREKA_SERVER
              value: "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"
```



##### 2.5.2.创建镜像

```shell
# 1.上传JAR
把 msa-sentinel-producer-1.0.0.jar 上传到服务器（172.51.216.81），位置（/k8s/springcloud/sentinel/producer）


# 2.创建Dockerfile
[root@k8s-master producer]# vim Dockerfile


[root@k8s-master producer]# ll
total 49500
-rw-r--r-- 1 root root      396 Nov  3 12:17 Dockerfile
-rw-r--r-- 1 root root 50683517 Nov  3 11:43 msa-sentinel-producer-1.0.0.jar


# 3.创建镜像
# docker build -t 172.51.216.85:8888/springcloud/msa-sentinel-producer:1.0.0 .
[root@k8s-master producer]# docker build -t 172.51.216.85:8888/springcloud/msa-sentinel-producer:1.0.0 .


# 8-jdk-alpine 镜像小了很多
[root@k8s-master producer]# docker images
REPOSITORY                                                        TAG            IMAGE ID       CREATED          SIZE
172.51.216.85:8888/springcloud/msa-sentinel-producer              1.0.0          726635d821a9   29 seconds ago   156MB
...


# 4.推送镜像
[root@k8s-master producer]# docker push 172.51.216.85:8888/springcloud/msa-sentinel-producer:1.0.0
```



##### 2.5.3.创建服务

```shell
[root@k8s-master producer]# vim msa-sentinel-producer.yaml


# 创建
[root@k8s-master producer]# kubectl apply -f msa-sentinel-producer.yaml 
service/msa-sentinel-producer created
deployment.apps/msa-sentinel-producer created


# 查看
[root@k8s-master producer]# kubectl get all -n dev | grep msa-sentinel-producer
pod/msa-sentinel-producer-7748d85459-28k82   1/1     Running   0          62s
pod/msa-sentinel-producer-7748d85459-pmh2w   1/1     Running   0          62s

service/msa-sentinel-producer   ClusterIP   10.96.36.253     <none>        8211/TCP          62s

deployment.apps/msa-sentinel-producer   2/2     2            2           62s

replicaset.apps/msa-sentinel-producer-7748d85459   2         2         2       62s


[root@k8s-master producer]# curl 10.96.36.253:8211/hello
hello，this is first messge! ### Wed Nov 03 13:01:55 CST 2021
[root@k8s-master producer]# curl 10.96.36.253:8211/say
say，this is first messge！ $$$Wed Nov 03 13:02:04 CST 2021
```



##### 2.5.4.测试

```shell
# 访问地址

curl 10.96.36.253:8211/hello
curl 10.96.36.253:8211/say
```



#### 2.6.部署sentinel消费者（msa-sentinel-consumer）

##### 2.6.1.配置文件

**1.微服务配置文件**

```yaml
# application-k8s.yml


server:
  port: 8212

spring:
  application:
    name: msa-sentinel-consumer
  cloud:
    sentinel:
      transport:
        port: 8719
        dashboard: ${DASHBOARD:172.51.216.98:8858}

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



**2.Dockerfile**

```dockerfile
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
# docker build -t 172.51.216.85:8888/springcloud/msa-sentinel-consumer:1.0.0 .
```



**3.k8s-yaml**

msa-sentinel-consumer.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-sentinel-consumer
  labels:
    app: msa-sentinel-consumer
spec:
  type: ClusterIP
  ports:
    - port: 8212
      targetPort: 8212
  selector:
    app: msa-sentinel-consumer

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-sentinel-consumer
spec:
  replicas: 2
  selector:
    matchLabels:
      app: msa-sentinel-consumer
  template:
    metadata:
      labels:
        app: msa-sentinel-consumer
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-sentinel-consumer
          image: 172.51.216.85:8888/springcloud/msa-sentinel-consumer:1.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8212
          env:
            - name: EUREKA_SERVER
              value: "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"
```



##### 2.6.2.创建镜像

```shell
# 1.上传JAR
把 msa-sentinel-consumer-1.0.0.jar 上传到服务器（172.51.216.81），位置（/k8s/springcloud/sentinel/consumer）


# 2.创建Dockerfile
[root@k8s-master consumer]# vim Dockerfile


[root@k8s-master consumer]# ll
total 49504
-rw-r--r-- 1 root root      396 Nov  3 12:36 Dockerfile
-rw-r--r-- 1 root root 50686107 Nov  3 11:43 msa-sentinel-consumer-1.0.0.jar


# 3.创建镜像
# docker build -t 172.51.216.85:8888/springcloud/msa-sentinel-consumer:1.0.0 .
[root@k8s-master consumer]# docker build -t 172.51.216.85:8888/springcloud/msa-sentinel-consumer:1.0.0 .


# 8-jdk-alpine 镜像小了很多
[root@k8s-master consumer]# docker images
REPOSITORY                                                        TAG            IMAGE ID       CREATED          SIZE
172.51.216.85:8888/springcloud/msa-sentinel-consumer              1.0.0          1a2dbd89bad1   22 seconds ago   156MB
...


# 4.推送镜像
[root@k8s-master consumer]# docker push 172.51.216.85:8888/springcloud/msa-sentinel-consumer:1.0.0
```



##### 2.6.3.创建服务

```shell
[root@k8s-master consumer]#  vim msa-sentinel-consumer.yaml


# 创建
[root@k8s-master consumer]# kubectl apply -f msa-sentinel-consumer.yaml 
service/msa-sentinel-consumer created
deployment.apps/msa-sentinel-consumer created


# 查看
[root@k8s-master consumer]# kubectl get all -n dev | grep msa-sentinel-consumer
pod/msa-sentinel-consumer-79b4ff65df-7ttnj   1/1     Running   0          22s
pod/msa-sentinel-consumer-79b4ff65df-ps9s9   1/1     Running   0          22s

service/msa-sentinel-consumer   ClusterIP   10.99.163.236    <none>        8212/TCP          22s

deployment.apps/msa-sentinel-consumer   2/2     2            2           22s

replicaset.apps/msa-sentinel-consumer-79b4ff65df   2         2         2       22s


[root@k8s-master producer]# curl 10.99.163.236:8212/hello
hello，this is first messge! ### Wed Nov 03 13:04:10 CST 2021  openfeign + sentinel!!!
[root@k8s-master producer]# curl 10.99.163.236:8212/say
say，this is first messge！ $$$Wed Nov 03 13:04:15 CST 2021  openfeign + sentinel!!!
```



##### 2.6.4.测试

```shell
# 访问地址

curl 10.99.163.236:8212/hello
curl 10.99.163.236:8212/say
```



#### 2.7.K8S访问外部Sentinel

**1.Sentinel服务**

```shell
# K8S 中的容器使用访问

svcname.namespace.svc.cluster.local:port
sentinel-svc.dev.svc.cluster.local:8858
```



**2.修改配置，重新部署**

msa-sentinel-producer.yaml、msa-sentinel-producer.yaml

```yaml
...
          env:
...
            - name: DASHBOARD
              value: "sentinel-svc.dev.svc.cluster.local:8858"
```



**3.测试**

```shell
[root@k8s-master consumer]# kubectl get all -n dev | grep msa-sentinel-consumer
pod/msa-sentinel-consumer-75f855f78-bmzpm   1/1     Running   0          23s
pod/msa-sentinel-consumer-75f855f78-xlnpn   1/1     Running   0          23s

service/msa-sentinel-consumer   ClusterIP   10.108.135.162   <none>        8212/TCP          23s

deployment.apps/msa-sentinel-consumer   2/2     2            2           23s

replicaset.apps/msa-sentinel-consumer-75f855f78   2         2         2       23s


[root@k8s-master consumer]# curl 10.108.135.162:8212/hello
hello，this is first messge! ### Wed Nov 03 13:32:42 CST 2021  openfeign + sentinel!!!
[root@k8s-master consumer]# curl 10.108.135.162:8212/say
say，this is first messge！ $$$Wed Nov 03 13:32:49 CST 2021  openfeign + sentinel!!!
```

**测试失败！看不到调用结果。**



#### 2.8.K8S访问内部Sentinel

**1.创建Endpoint(参照2.4.创建Endpoint)**

```shell
[root@k8s-master sentinel]#  kubectl get all -n dev | grep sentinel-server
pod/sentinel-server-5697bdc87b-qfc2b        0/1     ContainerCreating   0          59s

service/sentinel-server         NodePort    10.96.6.193      <none>        8858:30858/TCP    2m42s

deployment.apps/sentinel-server         0/1     1            0           59s

replicaset.apps/sentinel-server-5697bdc87b        1         1         0       59s


# K8S 中的容器使用访问
svcname.namespace.svc.cluster.local:port
sentinel-server.dev.svc.cluster.local:8858
```



**2.修改配置，重新部署**

msa-sentinel-producer.yaml、msa-sentinel-producer.yaml

```yaml
...
          env:
...
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"
```



**3.测试**

```shell
[root@k8s-master producer]# kubectl get all -n dev | grep msa-sentinel-consumer
pod/msa-sentinel-consumer-9888fd67c-4sp9f    1/1     Running   0          57s
pod/msa-sentinel-consumer-9888fd67c-wjgdv    1/1     Running   0          57s

service/msa-sentinel-consumer   ClusterIP   10.111.103.164   <none>        8212/TCP          57s

deployment.apps/msa-sentinel-consumer   2/2     2            2           57s

replicaset.apps/msa-sentinel-consumer-9888fd67c    2         2         2       57s



[root@k8s-master consumer]# curl 10.111.103.164:8212/hello
hello，this is first messge! ### Wed Nov 03 13:32:42 CST 2021  openfeign + sentinel!!!
[root@k8s-master consumer]# curl 10.111.103.164:8212/say
say，this is first messge！ $$$Wed Nov 03 13:32:49 CST 2021  openfeign + sentinel!!!


# 单实例测试成功
```

![](/images/kubernetes/microservice/ext-12.png)

![](/images/kubernetes/microservice/ext-11.png)



**4.运行多个sentinel服务，页面无法正常登陆。**

```shell
[root@k8s-master sentinel]#  kubectl get all -n dev | grep sentinel-server
pod/sentinel-server-5697bdc87b-6c5jj         1/1     Running   0          73s
pod/sentinel-server-5697bdc87b-7czz6         1/1     Running   0          73s
pod/sentinel-server-5697bdc87b-vv45z         1/1     Running   0          73s
service/sentinel-server         NodePort    10.99.15.237     <none>        8858:30858/TCP    73s
deployment.apps/sentinel-server         3/3     3            3           73s
replicaset.apps/sentinel-server-5697bdc87b         3         3         3       73s

# 多实例页面无法正常访问
```



#### 2.9.总结

**Sentinel在K8S中以Deployment部署单实例，可以正常使用。多实例页面不能正常访问。**



### 3.分布式任务调度平台（XXL-JOB）

#### 3.1.XXL-JOB

XXL-JOB是一个分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展。现已开放源代码并接入多家公司线上产品线，开箱即用。

![](/images/kubernetes/microservice/ext-20.png)



```shell
# 参考

https://github.com/xuxueli/xxl-job

https://www.xuxueli.com/xxl-job/#%E3%80%8A%E5%88%86%E5%B8%83%E5%BC%8F%E4%BB%BB%E5%8A%A1%E8%B0%83%E5%BA%A6%E5%B9%B3%E5%8F%B0XXL-JOB%E3%80%8B
```



#### 2.2.Docker部署XXL-JOB服务

```shell
# 下载镜像
docker pull xuxueli/xxl-job-admin:2.3.0

docker pull xuxueli/xxl-job-admin:2.2.0


# 创建数据库
xxl-job-2.3.0\doc\db\tables_xxl_job.sql

root
root


# 运行容器

docker run --network host -d --restart=always --name xxl-job-admin \
-e PARAMS="--spring.datasource.username=root \
--spring.datasource.password=root \
--spring.datasource.url=jdbc:mysql://172.51.216.98:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai" \
-v /tmp:/data/applogs \
xuxueli/xxl-job-admin:2.3.0
```



#### 2.3.K8S部署XXL-JOB服务

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
              value:  "--spring.datasource.username=root --spring.datasource.password=root --spring.datasource.url=jdbc:mysql://172.51.216.98:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai"


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
[root@k8s-master xxljob]# kubectl apply -f xxl-job.yaml 
service/xxl-job created
deployment.apps/xxl-job unchanged


[root@k8s-master xxljob]# kubectl get all -n dev | grep xxl-job
pod/xxl-job-dbc6f75f5-62z4j                  1/1     Running   0          2m11s

service/xxl-job                 NodePort    10.105.29.133    <none>        8080:30880/TCP    64s

deployment.apps/xxl-job                 1/1     1            1           2m11s

replicaset.apps/xxl-job-dbc6f75f5                  1         1         1       2m11s


# 访问dashboard
访问地址：http://172.51.216.81:30880/xxl-job-admin
账户密码：admin/123456
```

![](/images/kubernetes/microservice/ext-15.png)



#### 2.4.配置XXL-JOB

**1.配置执行器**

![](/images/kubernetes/microservice/ext-18.png)



**2.配置任务**

![](/images/kubernetes/microservice/ext-19.png)



#### 2.5.部署XXL-JOB执行器（msa-ext-job）

##### 2.5.1.配置文件

**1.微服务配置文件**

```yaml
# application-k8s.yml


# web port
server.port=8116
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


# K8S 中的容器访问xxl-job
svcname.namespace.svc.cluster.local:port
xxl-job.dev.svc.cluster.local:8080
```



**2.Dockerfile**

```dockerfile
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
# docker build -t 172.51.216.85:8888/springcloud/msa-ext-job:1.0.0 .
```



**3.k8s-yaml**

msa-ext-job.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-ext-job
  labels:
    app: msa-ext-job
spec:
  type: ClusterIP
  ports:
    - port: 8116
      targetPort: 8116
  selector:
    app: msa-ext-job

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-ext-job
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-ext-job
  template:
    metadata:
      labels:
        app: msa-ext-job
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-ext-job
          image: 172.51.216.85:8888/springcloud/msa-ext-job:1.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8116
          env:
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
```



##### 2.5.2.创建镜像

```shell
# 1.创建镜像
# docker build -t 172.51.216.85:8888/springcloud/msa-ext-job:1.0.0 .
[root@k8s-master xxljob]# docker build -t 172.51.216.85:8888/springcloud/msa-ext-job:1.0.0 .


# 8-jdk-alpine 镜像小了很多
[root@k8s-master xxljob]# docker images
REPOSITORY                                                        TAG            IMAGE ID       CREATED          SIZE
172.51.216.85:8888/springcloud/msa-ext-job                        1.0.0          5c0c46f8bda5   23 seconds ago   133MB
...


# 4.推送镜像
[root@k8s-master xxljob]# docker push 172.51.216.85:8888/springcloud/msa-ext-job:1.0.0
```



##### 2.5.3.创建服务

```shell
# 创建
[root@k8s-master xxljob]# kubectl apply -f msa-ext-job.yaml 
service/msa-ext-job created
deployment.apps/msa-ext-job created


# 查看
[root@k8s-master xxljob]# kubectl get all -n dev | grep msa-ext-job
pod/msa-ext-job-5879fc4844-77p62             1/1     Running   0          45s
pod/msa-ext-job-5879fc4844-h2g52             1/1     Running   0          45s
pod/msa-ext-job-5879fc4844-hwhts             1/1     Running   0          45s

service/msa-ext-job             ClusterIP   10.101.225.7     <none>        8116/TCP          45s

deployment.apps/msa-ext-job             3/3     3            3           45s

replicaset.apps/msa-ext-job-5879fc4844             3         3         3       45s
```



##### 2.5.4.测试

访问dashboard

访问地址：http://172.51.216.81:30880/xxl-job-admin
账户密码：admin/123456



**1.查看管理界面**

![](/images/kubernetes/microservice/ext-16.png)

![](/images/kubernetes/microservice/ext-17.png)



**2.查看Pod日志**

```shell
# 查看pod
[root@k8s-master xxljob]# kubectl get pod -n dev -o wide | grep msa-ext-job
msa-ext-job-5879fc4844-77p62             1/1     Running   0          10m     10.244.107.248   k8s-node3   <none>           <none>
msa-ext-job-5879fc4844-h2g52             1/1     Running   0          10m     10.244.169.185   k8s-node2   <none>           <none>
msa-ext-job-5879fc4844-hwhts             1/1     Running   0          10m     10.244.107.250   k8s-node3   <none>           <none>


[root@k8s-master xxljob]# kubectl logs --tail=10 -f msa-ext-job-5879fc4844-77p62 -n dev
----job one 1---Thu Nov 04 13:44:05 CST 2021
----job one 1---Thu Nov 04 13:44:08 CST 2021
----job one 1---Thu Nov 04 13:44:11 CST 2021
----job one 1---Thu Nov 04 13:44:14 CST 2021
----job one 1---Thu Nov 04 13:44:17 CST 2021
----job one 1---Thu Nov 04 13:44:20 CST 2021
----job one 1---Thu Nov 04 13:44:23 CST 2021
----job one 1---Thu Nov 04 13:44:26 CST 2021
----job one 1---Thu Nov 04 13:44:29 CST 2021


[root@k8s-master xxljob]# kubectl logs --tail=10 -f msa-ext-job-5879fc4844-h2g52 -n dev
----job one 1---Thu Nov 04 13:45:08 CST 2021
----job one 1---Thu Nov 04 13:45:11 CST 2021
----job one 1---Thu Nov 04 13:45:14 CST 2021
----job one 1---Thu Nov 04 13:45:17 CST 2021
----job one 1---Thu Nov 04 13:45:20 CST 2021
----job one 1---Thu Nov 04 13:45:23 CST 2021
----job one 1---Thu Nov 04 13:45:26 CST 2021
----job one 1---Thu Nov 04 13:45:29 CST 2021
----job one 1---Thu Nov 04 13:45:32 CST 2021
----job one 1---Thu Nov 04 13:45:35 CST 2021


[root@k8s-master xxljob]# kubectl logs --tail=10 -f msa-ext-job-5879fc4844-hwhts -n dev
----job one 1---Thu Nov 04 13:46:21 CST 2021
----job one 1---Thu Nov 04 13:46:24 CST 2021
----job one 1---Thu Nov 04 13:46:27 CST 2021
----job one 1---Thu Nov 04 13:46:30 CST 2021
----job one 1---Thu Nov 04 13:46:33 CST 2021
----job one 1---Thu Nov 04 13:46:36 CST 2021
----job one 1---Thu Nov 04 13:46:39 CST 2021
----job one 1---Thu Nov 04 13:46:42 CST 2021
----job one 1---Thu Nov 04 13:46:45 CST 2021
----job one 1---Thu Nov 04 13:46:48 CST 2021
```



#### 2.6.测试XXL-JOB集群

**1.修改配置创建服务**

xxl-job.yaml

```shell
---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: xxl-job
spec:
  replicas: 3
  selector:
    matchLabels:
      app: xxl-job
  template:
...
```

```shell
# 创建
[root@k8s-master xxljob]# kubectl apply -f xxl-job.yaml 
service/xxl-job created
deployment.apps/xxl-job unchanged


[root@k8s-master xxljob]# kubectl get all -n dev | grep xxl-job
pod/xxl-job-dbc6f75f5-9hfjt                  1/1     Running   0          3m3s
pod/xxl-job-dbc6f75f5-lfftn                  1/1     Running   0          3m3s
pod/xxl-job-dbc6f75f5-xvgx9                  1/1     Running   0          3m3s

service/xxl-job                 NodePort    10.103.255.144   <none>        8080:30880/TCP    3m3s

deployment.apps/xxl-job                 3/3     3            3           3m3s
replicaset.apps/xxl-job-dbc6f75f5                  3         3         3       3m3s



# 访问dashboard
访问地址：http://172.51.216.81:30880/xxl-job-admin
账户密码：admin/123456
```



**测试3个xxl-job实例，可以正常工作。**



### 4.日志集合（ELK）

#### 4.1.ELASTIC  STACK

ELK是三个开源软件的缩写，分别表示：Elasticsearch , Logstash和Kibana。

ELK提供了一整套解决方案，并且都是开源软件，之间互相配合使用，完美衔接，高效的满足了很多场合的应用，是目前主流的一种日志分析平台。

一个完整的集中式日志系统，需要包含以下几个主要特点：
收集－能够采集多种来源的日志数据
传输－能够稳定的把日志数据传输到中央系统
存储－如何存储日志数据
分析－可以支持 UI 分析
警告－能够提供错误报告，监控机制

![](/images/kubernetes/microservice/elk-1.png)

**Elasticsearch**
Elasticsearch 基于java，是个开源分布式搜索引擎，它的特点有：分布式，零配置，自动发现，索引自动分片，索
引副本机制，restful风格接口，多数据源，自动搜索负载等。

**Logstash**
Logstash 基于java，是一个开源的用于收集,分析和存储日志的工具。

**Kibana**
Kibana 基于nodejs，也是一个开源和免费的工具，Kibana可以为 Logstash 和ElasticSearch 提供的日志分析友好的 Web 界面，可以汇总、分析和搜索重要数据日志。

**Beats**
Beats是elastic公司开源的一款采集系统监控数据的代理agent，是在被监控服务器上以客户端形式运行的数据收集器的统称，可以直接把数据发送给Elasticsearch或者通过Logstash发送给Elasticsearch，然后进行后续的数据分析活动。

**Beats由如下组成:**
**Packetbeat：**是一个网络数据包分析器，用于监控、收集网络流量信息，Packetbeat嗅探服务器之间的流量，解析应用层协议，并关联到消息的处理，其支 持ICMP (v4 and v6)、DNS、HTTP、Mysql、PostgreSQL、Redis、MongoDB、Memcache等协议；

**Filebeat：**用于监控、收集服务器日志文件，其已取代 logstash forwarder；

**Metricbeat：**可定期获取外部系统的监控指标信息，其可以监控、收集 Apache、HAProxy、MongoDB、MySQL、Nginx、PostgreSQL、Redis、System、Zookeeper等服务；

**Winlogbeat：**用于监控、收集Windows系统的日志信息；



**部署方式：**

![](/images/kubernetes/microservice/elk-8.png)



#### 4.2.环境搭建

需要部署RabbitMQ、Elasticsearch、Kibana，参考**调用链监控（Zipkin）**。

 **Logstash先不部署，日志发送到队列就可以。**



#### 4.3.配置RabbitMQ

**Rabbitmq配置**

建queue：msa-ext-elk-queue

建Exchange: msa-ext-elk-exchange  类型：direct  topic: log

Exchange binding queue



**1.创建队列**

![](/images/kubernetes/microservice/elk-2.png)



**2.创建交换机**

![](/images/kubernetes/microservice/elk-3.png)



**3.绑定交换机**

![](/images/kubernetes/microservice/elk-4.png)



![](/images/kubernetes/microservice/elk-5.png)



#### 4.4.创建Endpoint

 rabbitmq.yml

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
[root@k8s-master elk]# kubectl apply -f rabbitmq.yml 
service/rabbitmq-svc unchanged
endpoints/rabbitmq-svc created


[root@k8s-master elk]# kubectl describe ep rabbitmq-svc -n dev
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


[root@k8s-master elk]# kubectl describe ep rabbitmq-svc -n dev
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

[root@k8s-master elk]# kubectl describe svc rabbitmq-svc -n dev
Name:              rabbitmq-svc
Namespace:         dev
Labels:            <none>
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Families:       <none>
IP:                10.101.38.222
IPs:               10.101.38.222
Port:              <unset>  5672/TCP
TargetPort:        5672/TCP
Endpoints:         172.51.216.98:5672
Session Affinity:  None
Events:            <none>


# K8S 中的容器使用访问
svcname.namespace.svc.cluster.local:port
rabbitmq-svc.dev.svc.cluster.local:5672
```



#### 4.5.创建镜像

Dockerfile

```dockerfile
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
# docker build -t 172.51.216.85:8888/springcloud/msa-ext-elk:1.0.0 .
```

```shell
# 1.创建镜像
# docker build -t 172.51.216.85:8888/springcloud/msa-ext-elk:1.0.0 .
[root@k8s-master elk]# docker build -t 172.51.216.85:8888/springcloud/msa-ext-elk:1.0.0 .


# 8-jdk-alpine 镜像小了很多
[root@k8s-master elk]# docker images
REPOSITORY                                                        TAG            IMAGE ID       CREATED          SIZE
172.51.216.85:8888/springcloud/msa-ext-elk                        1.0.0          708012b77f65   19 seconds ago   126MB
...


# 2.推送镜像
[root@k8s-master elk]# docker push 172.51.216.85:8888/springcloud/msa-ext-elk:1.0.0
```



#### 4.6.创建服务

msa-ext-elk.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-ext-elk
  labels:
    app: msa-ext-elk
spec:
  type: ClusterIP
  ports:
    - port: 8114
      targetPort: 8114
  selector:
    app: msa-ext-elk

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-ext-elk
spec:
  replicas: 2
  selector:
    matchLabels:
      app: msa-ext-elk
  template:
    metadata:
      labels:
        app: msa-ext-elk
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-ext-elk
          image: 172.51.216.85:8888/springcloud/msa-ext-elk:1.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8114
          env:
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
```



```shell
[root@k8s-master elk]# vim msa-ext-elk.yaml


# 创建
[root@k8s-master elk]# kubectl apply -f msa-ext-elk.yaml 
service/msa-ext-elk created
deployment.apps/msa-ext-elk created


# 查看
[root@k8s-master elk]# kubectl get all -n dev | grep msa-ext-elk
pod/msa-ext-elk-6974d77cb7-kzncm          1/1     Running   0          4s
pod/msa-ext-elk-6974d77cb7-w4dmw          1/1     Running   0          4s

service/msa-ext-elk           ClusterIP   10.96.49.31      <none>        8114/TCP          4s

deployment.apps/msa-ext-elk           2/2     2            2           4s

replicaset.apps/msa-ext-elk-6974d77cb7          2         2         2       5s


[root@k8s-master elk]# curl http://10.96.49.31:8114/hello
hello，this is elk messge! ### Tue Nov 02 15:31:58 CST 2021
```



#### 4.4.测试

访问地址：

curl http://10.96.49.31:8114/hello

**注意：经过测试直接使用IP地址也可以访问，不用配置端点**

![](/images/kubernetes/microservice/elk-7.png)





### 5.系统监控（Prometheus）

#### 5.1.Prometheus

Prometheus 是一套开源的系统监控报警框架。它启发于 Google 的 borgmon 监控系统，由工作在 SoundCloud 的 google 前员工在 2012 年创建，作为社区开源项目进行开发，并于 2015 年正式发布。2016 年，Prometheus 正式加入 Cloud Native Computing Foundation，成为受欢迎度仅次于 Kubernetes 的项目。

![](/images/kubernetes/microservice/ext-26.png)

![](/images/kubernetes/microservice/ext-27.png)

**部署方式：**

K8S内部微服务把Metrics端点暴露出来，Prometheus从端点直接拉取指标。



#### 5.2.环境搭建

**1.安装Prometheus**

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

  - job_name: spring-boot-actuator
    metrics_path: '/actuator/prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['172.51.216.81:30158']
        labels:
          instance: spring-actuator

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

![](/images/kubernetes/microservice/ext-21.png)

**2.安装Grafana**

```shell
# 创建目录
mkdir -p /ntms/grafana/data
chmod 777 -R /ntms/grafana/data
 

### 运行docker 
 
docker run -d --network host --name=grafana --restart=always \
-v /ntms/grafana/data:/var/lib/grafana \
grafana/grafana
 

### 访问地址
# grafana
http://172.51.216.98:3000
 
# 默认账号密码
用户名：admin
密码：  admin
```

![](/images/kubernetes/microservice/ext-22.png)



**3.配置数据源**

![](/images/kubernetes/microservice/ext-23.png)



#### 5.3.部署Spring Boot程序（msa-ext-prometheus）

##### 2.5.1.配置文件

**1.微服务配置文件**

```yaml
# application-k8s.yml


management:
  metrics:
    tags:
      application: ${spring.application.name}
      region: iids
  endpoints:
    web:
      base-path: /actuator
      exposure:
        include: "*"
  endpoint:
    health:
      show-details: always
```



**2.Dockerfile**

```dockerfile
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
# docker build -t 172.51.216.85:8888/springcloud/msa-ext-prometheus:1.0.0 .
```



**3.k8s-yaml**

msa-ext-prometheus.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-ext-prometheus
  labels:
    app: msa-ext-prometheus
spec:
  type: NodePort
  ports:
    - port: 8158
      name: msa-ext-prometheus
      targetPort: 8158
      nodePort: 30158 #对外暴露30158端口
  selector:
    app: msa-ext-prometheus

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-ext-prometheus
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-ext-prometheus
  template:
    metadata:
      labels:
        app: msa-ext-prometheus
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-ext-prometheus
          image: 172.51.216.85:8888/springcloud/msa-ext-prometheus:1.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8158
          env:
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
```



##### 2.5.2.创建镜像

```shell
# 1.创建镜像
# docker build -t 172.51.216.85:8888/springcloud/msa-ext-prometheus:1.0.0 .
[root@k8s-master prometheus]# docker build -t 172.51.216.85:8888/springcloud/msa-ext-prometheus:1.0.0 .


# 8-jdk-alpine 镜像小了很多
[root@k8s-master prometheus]# docker images
REPOSITORY                                                        TAG            IMAGE ID       CREATED          SIZE
172.51.216.85:8888/springcloud/msa-ext-prometheus                 1.0.0          7107a79a6460   25 seconds ago   123MB
...


# 4.推送镜像
[root@k8s-master prometheus]# docker push 172.51.216.85:8888/springcloud/msa-ext-prometheus:1.0.0
```



##### 2.5.3.创建服务

```shell
# 创建
[root@k8s-master prometheus]# kubectl apply -f msa-ext-prometheus.yaml 
service/msa-ext-prometheus created
deployment.apps/msa-ext-prometheus created


# 查看
[root@k8s-master prometheus]# kubectl get all -n dev | grep msa-ext-prometheus
pod/msa-ext-prometheus-6599578495-cjqd9      1/1     Running   0          32s
pod/msa-ext-prometheus-6599578495-tvmdr      1/1     Running   0          32s
pod/msa-ext-prometheus-6599578495-wzlvb      1/1     Running   0          32s

service/msa-ext-prometheus      NodePort    10.105.77.203    <none>        8158:30158/TCP    32s

deployment.apps/msa-ext-prometheus      3/3     3            3           32s

replicaset.apps/msa-ext-prometheus-6599578495      3         3         3       32s


# 访问 http://172.51.216.81:30158/actuator
{"_links":{"self":{"href":"http://172.51.216.81:30158/actuator","templated":false},"beans":{"href":"http://172.51.216.81:30158/actuator/beans","templated":false},"caches":{"href":"http://172.51.216.81:30158/actuator/caches","templated":false},"caches-cache":{"href":"http://172.51.216.81:30158/actuator/caches/{cache}","templated":true},"health-path":{"href":"http://172.51.216.81:30158/actuator/health/{*path}","templated":true},"health":{"href":"http://172.51.216.81:30158/actuator/health","templated":false},"info":{"href":"http://172.51.216.81:30158/actuator/info","templated":false},"conditions":{"href":"http://172.51.216.81:30158/actuator/conditions","templated":false},"configprops":{"href":"http://172.51.216.81:30158/actuator/configprops","templated":false},"env-toMatch":{"href":"http://172.51.216.81:30158/actuator/env/{toMatch}","templated":true},"env":{"href":"http://172.51.216.81:30158/actuator/env","templated":false},"loggers":{"href":"http://172.51.216.81:30158/actuator/loggers","templated":false},"loggers-name":{"href":"http://172.51.216.81:30158/actuator/loggers/{name}","templated":true},"heapdump":{"href":"http://172.51.216.81:30158/actuator/heapdump","templated":false},"threaddump":{"href":"http://172.51.216.81:30158/actuator/threaddump","templated":false},"prometheus":{"href":"http://172.51.216.81:30158/actuator/prometheus","templated":false},"metrics":{"href":"http://172.51.216.81:30158/actuator/metrics","templated":false},"metrics-requiredMetricName":{"href":"http://172.51.216.81:30158/actuator/metrics/{requiredMetricName}","templated":true},"scheduledtasks":{"href":"http://172.51.216.81:30158/actuator/scheduledtasks","templated":false},"mappings":{"href":"http://172.51.216.81:30158/actuator/mappings","templated":false}}}



# 访问http://172.51.216.81:30158/actuator/prometheus

# HELP jvm_threads_daemon_threads The current number of live daemon threads
# TYPE jvm_threads_daemon_threads gauge
jvm_threads_daemon_threads{application="springboot-prometheus",region="iids",} 16.0
# HELP jvm_buffer_memory_used_bytes An estimate of the memory that the Java virtual machine is using for this buffer pool
# TYPE jvm_buffer_memory_used_bytes gauge
jvm_buffer_memory_used_bytes{application="springboot-prometheus",id="mapped",region="iids",} 0.0
jvm_buffer_memory_used_bytes{application="springboot-prometheus",id="direct",region="iids",} 8192.0
# HELP jvm_threads_peak_threads The peak live thread count since the Java virtual machine started or peak was reset
# TYPE jvm_threads_peak_threads gauge
jvm_threads_peak_threads{application="springboot-prometheus",region="iids",} 20.0
# HELP system_cpu_usage The "recent cpu usage" for the whole system
# TYPE system_cpu_usage gauge
system_cpu_usage{application="springboot-prometheus",region="iids",} 0.08191068754564687
# HELP jvm_gc_pause_seconds Time spent in GC pause
# TYPE jvm_gc_pause_seconds summary
jvm_gc_pause_seconds_count{action="end of major GC",application="springboot-prometheus",cause="Metadata GC Threshold",region="iids",} 1.0
jvm_gc_pause_seconds_sum{action="end of major GC",application="springboot-prometheus",cause="Metadata GC Threshold",region="iids",} 0.081
jvm_gc_pause_seconds_count{action="end of minor GC",application="springboot-prometheus",cause="Allocation Failure",region="iids",} 2.0
jvm_gc_pause_seconds_sum{action="end of minor GC",application="springboot-prometheus",cause="Allocation Failure",region="iids",} 0.055
# HELP jvm_gc_pause_seconds_max Time spent in GC pause
# TYPE jvm_gc_pause_seconds_max gauge
jvm_gc_pause_seconds_max{action="end of major GC",application="springboot-prometheus",cause="Metadata GC Threshold",region="iids",} 0.0
jvm_gc_pause_seconds_max{action="end of minor GC",application="springboot-prometheus",cause="Allocation Failure",region="iids",} 0.0
# HELP logback_events_total Number of error level events that made it to the logs
# TYPE logback_events_total counter
logback_events_total{application="springboot-prometheus",level="error",region="iids",} 0.0
logback_events_total{application="springboot-prometheus",level="debug",region="iids",} 0.0
logback_events_total{application="springboot-prometheus",level="info",region="iids",} 7.0
logback_events_total{application="springboot-prometheus",level="trace",region="iids",} 0.0
logback_events_total{application="springboot-prometheus",level="warn",region="iids",} 0.0
# HELP jvm_memory_committed_bytes The amount of memory in bytes that is committed for the Java virtual machine to use
# TYPE jvm_memory_committed_bytes gauge
jvm_memory_committed_bytes{application="springboot-prometheus",area="heap",id="Eden Space",region="iids",} 7.1630848E7
jvm_memory_committed_bytes{application="springboot-prometheus",area="heap",id="Tenured Gen",region="iids",} 1.78978816E8
jvm_memory_committed_bytes{application="springboot-prometheus",area="nonheap",id="Metaspace",region="iids",} 3.8535168E7
jvm_memory_committed_bytes{application="springboot-prometheus",area="nonheap",id="Compressed Class Space",region="iids",} 4980736.0
jvm_memory_committed_bytes{application="springboot-prometheus",area="heap",id="Survivor Space",region="iids",} 8912896.0
jvm_memory_committed_bytes{application="springboot-prometheus",area="nonheap",id="Code Cache",region="iids",} 1.1010048E7
# HELP jvm_buffer_count_buffers An estimate of the number of buffers in the pool
# TYPE jvm_buffer_count_buffers gauge
jvm_buffer_count_buffers{application="springboot-prometheus",id="mapped",region="iids",} 0.0
jvm_buffer_count_buffers{application="springboot-prometheus",id="direct",region="iids",} 1.0
# HELP process_start_time_seconds Start time of the process since unix epoch.
# TYPE process_start_time_seconds gauge
process_start_time_seconds{application="springboot-prometheus",region="iids",} 1.636345459824E9
# HELP process_files_open_files The open file descriptor count
# TYPE process_files_open_files gauge
process_files_open_files{application="springboot-prometheus",region="iids",} 26.0
# HELP jvm_gc_max_data_size_bytes Max size of old generation memory pool
# TYPE jvm_gc_max_data_size_bytes gauge
jvm_gc_max_data_size_bytes{application="springboot-prometheus",region="iids",} 1.78978816E8
# HELP jvm_gc_memory_allocated_bytes_total Incremented for an increase in the size of the young generation memory pool after one GC to before the next
# TYPE jvm_gc_memory_allocated_bytes_total counter
jvm_gc_memory_allocated_bytes_total{application="springboot-prometheus",region="iids",} 1.77294488E8
# HELP jvm_classes_unloaded_classes_total The total number of classes unloaded since the Java virtual machine has started execution
# TYPE jvm_classes_unloaded_classes_total counter
jvm_classes_unloaded_classes_total{application="springboot-prometheus",region="iids",} 0.0
# HELP jvm_memory_max_bytes The maximum amount of memory in bytes that can be used for memory management
# TYPE jvm_memory_max_bytes gauge
jvm_memory_max_bytes{application="springboot-prometheus",area="heap",id="Eden Space",region="iids",} 7.1630848E7
jvm_memory_max_bytes{application="springboot-prometheus",area="heap",id="Tenured Gen",region="iids",} 1.78978816E8
jvm_memory_max_bytes{application="springboot-prometheus",area="nonheap",id="Metaspace",region="iids",} -1.0
jvm_memory_max_bytes{application="springboot-prometheus",area="nonheap",id="Compressed Class Space",region="iids",} 1.073741824E9
jvm_memory_max_bytes{application="springboot-prometheus",area="heap",id="Survivor Space",region="iids",} 8912896.0
jvm_memory_max_bytes{application="springboot-prometheus",area="nonheap",id="Code Cache",region="iids",} 2.5165824E8
# HELP jvm_threads_states_threads The current number of threads having NEW state
# TYPE jvm_threads_states_threads gauge
jvm_threads_states_threads{application="springboot-prometheus",region="iids",state="blocked",} 0.0
jvm_threads_states_threads{application="springboot-prometheus",region="iids",state="terminated",} 0.0
jvm_threads_states_threads{application="springboot-prometheus",region="iids",state="waiting",} 12.0
jvm_threads_states_threads{application="springboot-prometheus",region="iids",state="timed-waiting",} 2.0
jvm_threads_states_threads{application="springboot-prometheus",region="iids",state="new",} 0.0
jvm_threads_states_threads{application="springboot-prometheus",region="iids",state="runnable",} 6.0
# HELP system_cpu_count The number of processors available to the Java virtual machine
# TYPE system_cpu_count gauge
system_cpu_count{application="springboot-prometheus",region="iids",} 1.0
# HELP jvm_buffer_total_capacity_bytes An estimate of the total capacity of the buffers in this pool
# TYPE jvm_buffer_total_capacity_bytes gauge
jvm_buffer_total_capacity_bytes{application="springboot-prometheus",id="mapped",region="iids",} 0.0
jvm_buffer_total_capacity_bytes{application="springboot-prometheus",id="direct",region="iids",} 8192.0
# HELP tomcat_sessions_created_sessions_total  
# TYPE tomcat_sessions_created_sessions_total counter
tomcat_sessions_created_sessions_total{application="springboot-prometheus",region="iids",} 0.0
# HELP jvm_memory_used_bytes The amount of used memory
# TYPE jvm_memory_used_bytes gauge
jvm_memory_used_bytes{application="springboot-prometheus",area="heap",id="Eden Space",region="iids",} 2.2972656E7
jvm_memory_used_bytes{application="springboot-prometheus",area="heap",id="Tenured Gen",region="iids",} 1.4420504E7
jvm_memory_used_bytes{application="springboot-prometheus",area="nonheap",id="Metaspace",region="iids",} 3.5914072E7
jvm_memory_used_bytes{application="springboot-prometheus",area="nonheap",id="Compressed Class Space",region="iids",} 4457376.0
jvm_memory_used_bytes{application="springboot-prometheus",area="heap",id="Survivor Space",region="iids",} 0.0
jvm_memory_used_bytes{application="springboot-prometheus",area="nonheap",id="Code Cache",region="iids",} 1.0979584E7
# HELP process_cpu_usage The "recent cpu usage" for the Java Virtual Machine process
# TYPE process_cpu_usage gauge
process_cpu_usage{application="springboot-prometheus",region="iids",} 3.5283607087356907E-7
# HELP jvm_threads_live_threads The current number of live threads including both daemon and non-daemon threads
# TYPE jvm_threads_live_threads gauge
jvm_threads_live_threads{application="springboot-prometheus",region="iids",} 20.0
# HELP tomcat_sessions_expired_sessions_total  
# TYPE tomcat_sessions_expired_sessions_total counter
tomcat_sessions_expired_sessions_total{application="springboot-prometheus",region="iids",} 0.0
# HELP tomcat_sessions_alive_max_seconds  
# TYPE tomcat_sessions_alive_max_seconds gauge
tomcat_sessions_alive_max_seconds{application="springboot-prometheus",region="iids",} 0.0
# HELP process_files_max_files The maximum file descriptor count
# TYPE process_files_max_files gauge
process_files_max_files{application="springboot-prometheus",region="iids",} 1048576.0
# HELP jvm_gc_live_data_size_bytes Size of old generation memory pool after a full GC
# TYPE jvm_gc_live_data_size_bytes gauge
jvm_gc_live_data_size_bytes{application="springboot-prometheus",region="iids",} 1.4420504E7
# HELP system_load_average_1m The sum of the number of runnable entities queued to available processors and the number of runnable entities running on the available processors averaged over a period of time
# TYPE system_load_average_1m gauge
system_load_average_1m{application="springboot-prometheus",region="iids",} 0.3134765625
# HELP tomcat_sessions_active_current_sessions  
# TYPE tomcat_sessions_active_current_sessions gauge
tomcat_sessions_active_current_sessions{application="springboot-prometheus",region="iids",} 0.0
# HELP tomcat_sessions_active_max_sessions  
# TYPE tomcat_sessions_active_max_sessions gauge
tomcat_sessions_active_max_sessions{application="springboot-prometheus",region="iids",} 0.0
# HELP process_uptime_seconds The uptime of the Java virtual machine
# TYPE process_uptime_seconds gauge
process_uptime_seconds{application="springboot-prometheus",region="iids",} 567.748
# HELP tomcat_sessions_rejected_sessions_total  
# TYPE tomcat_sessions_rejected_sessions_total counter
tomcat_sessions_rejected_sessions_total{application="springboot-prometheus",region="iids",} 0.0
# HELP jvm_classes_loaded_classes The number of classes that are currently loaded in the Java virtual machine
# TYPE jvm_classes_loaded_classes gauge
jvm_classes_loaded_classes{application="springboot-prometheus",region="iids",} 6770.0
# HELP jvm_gc_memory_promoted_bytes_total Count of positive increases in the size of the old generation memory pool before GC to after GC
# TYPE jvm_gc_memory_promoted_bytes_total counter
jvm_gc_memory_promoted_bytes_total{application="springboot-prometheus",region="iids",} 6692952.0
```

```shell
### 查看指标
 
http://localhost:8080/actuator
http://localhost:8080/actuator/prometheus
http://localhost:8080/actuator/metrics
http://localhost:8080/actuator/metrics/jvm.memory.max
http://localhost:8080/actuator/loggers
http://localhost:8080/actuator/prometheus
```



##### 2.5.4.测试

访问地址：http://172.51.216.81:30158/actuator
Prometheus地址：http://172.51.216.98:9090

![](/images/kubernetes/microservice/ext-24.png)



#### 5.4.Grafana配置

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



**在Grafana中配置4701如下**

![](/images/kubernetes/microservice/ext-25.png)



#### 5.5.Ingress

```yaml
# 配置方式，注意配置本地hosts
http://gateway.k8s.com:31208/dconsumer/actuator/prometheus
http://gateway.k8s.com:31208/dproducer/actuator/prometheus


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

```shell
# 配置本地hosts

[root@hadoop101 mort]# vi /etc/hosts

127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

172.51.216.81 gateway.k8s.com
```





## 四、总结



### 1.基础中间件

K8S内服服务访问外部服务可以配置Endpoint，也可以不配置。例如

```yaml
# postgresql.yml

apiVersion: v1
kind: Service
metadata:
  name: postgresql-svc
  namespace: dev
spec:
  ports:
  - port: 5432
    targetPort: 5432
    protocol: TCP

---
apiVersion: v1
kind: Endpoints
metadata:
  name: postgresql-svc
  namespace: dev
subsets:
  - addresses:
    - ip: 182.92.210.65
    ports:
    - port: 5432
```



### 2.系统中间件

#### 2.1.调用链监控（Zipkin）

Zipkin-Server部署在K8S内部，Deployment方式部署，可以部署多个实例，通过消息队列收集监控数据。

**MQ方式**

![image-20211029134317548](/images/kubernetes/microservice/ext-7.png)



#### 2.2.流量控制（Sentinel）

Sentinel在K8S中以Deployment部署单实例，可以正常使用。多实例页面不能正常访问。



#### 2.3.分布式任务调度平台（XXL-JOB）

XXL-JOB部署在K8S内部，Deployment方式部署，可以部署多个实例。



#### 2.4.日志集合（ELK）

ELK部署在集群外部，K8S内部微服务通过消息队列发送日志。



#### 2.5.系统监控（Prometheus）

Prometheus部署在集群外部，K8S内部微服务把Metrics端点暴露出来，Prometheus从端点直接拉取指标。

**通过Ingress暴露Metrics端点。**

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
          

# 实际地址
http://gateway.k8s.com:31208/dconsumer/actuator/prometheus
http://gateway.k8s.com:31208/dproducer/actuator/prometheus
```



**问题1：微服务多实例无法区分，后期解决。**



#### 2.6.分布式配置管理中心（Apollo-阿波罗）

后期测试。



