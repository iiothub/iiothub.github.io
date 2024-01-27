* TOC
{:toc}


### 1.环境规划

- **微服务：**k8s部署微服务，名字空间：test；
- **中间件：**k8s部署中间件，PostgreSQL、MySQL、Redis、RabbitMQ；
- **系统监控：**k8s部署Prometheus，Elastic Stack；
- **镜像仓库：**Harbor作为私有镜像仓库，创建springcloud项目；



### 2.准备工作

#### 2.1.创建名字空间

```shell
[root@k8s-master1 k8s]# kubectl create ns test
namespace/test created


[root@k8s-master1 k8s]# kubectl get ns
NAME                   STATUS   AGE
ext                    Active   26h
pro                    Active   27h
test                   Active   4s
......
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
  namespace: test
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: ewoJImF1dGhzIjogewoJCSIxNzIuNTEuMjE2Ljg4Ojg4ODgiOiB7CgkJCSJhdXRoIjogIllXUnRhVzQ2WVdSdGFXND0iCgkJfQoJfQp9


[root@k8s-master1 test]# kubectl apply -f harbor-secret.yaml 
secret/harborsecret created


[root@k8s-master1 test]# kubectl get secret -n test
NAME                  TYPE                                  DATA   AGE
default-token-frr8h   kubernetes.io/service-account-token   3      5m4s
harborsecret          kubernetes.io/dockerconfigjson        1      26s
```



### 3.微服务配置

#### 3.9.Ingress

**调用链监控（Zipkin）、流量控制（Sentinel）、分布式任务调度平台（XXL-JOB）、注册中心（eureka-server）、网关（msa-gateway）。**



**Http代理**

> **创建 msa-test-http.yaml**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: msa-http
  namespace: test
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
    - host: zipkin.test.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: zipkin-server
                port:
                  number: 9411
    - host: sentinel.test.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: sentinel-server
                port:
                  number: 8858
    - host: xxljob.test.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: xxl-job
                port:
                  number: 8080
    - host: eureka.test.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: msa-eureka
                port:
                  number: 10001
    - host: gateway.test.com
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
  namespace: "test"
  imagePullSecrets: "harborsecret"
  repository: "172.51.216.88:8888/springcloud/"
  env:
    eureka: "http://msa-eureka-0.msa-eureka.test:10001/eureka/,http://msa-eureka-1.msa-eureka.test:10001/eureka/,http://msa-eureka-2.msa-eureka.test:10001/eureka/"
    profiles: "-Dspring.profiles.active=k8s-test"


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
    addresses: "production-ready.default.svc.cluster.local:5672"
    user: "default_user_VcSb2rx0RLFPBfw3kcv"
    password: "apexs6u5gj2wAu29O1YxGtXuQnINzEvs"
    storage: "elasticsearch"
    eshosts: "elasticsearch-logging.kube-system.svc.cluster.local:9200"


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
    dashboard: "sentinel-server.test.svc.cluster.local:8858"


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
    dashboard: "sentinel-server.test.svc.cluster.local:8858"
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
[root@k8s-master1 test]# helm create cloud -n pro
Creating cloud


# Chart.yaml
[root@k8s-master1 cloud]# vim Chart.yaml

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
msa-test-http.yaml
values.yaml



# 创建后的文件结构
[root@k8s-master1 cloud]# pwd
/k8s/springcloud/test/cloud
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
│   ├── msa-test-http.yaml
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
[root@k8s-master1 test]# helm install msa cloud --dry-run --debug -n test



# 安装部署
[root@k8s-master1 test]# helm install msa cloud -n test
NAME: msa
LAST DEPLOYED: Tue Dec 21 15:11:21 2021
NAMESPACE: test
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

```shell
[root@k8s-master1 test]# helm list -n test
NAME	NAMESPACE	REVISION	UPDATED                               	STATUS  	CHART      	APP VERSION
msa 	test     	1       	2021-12-21 15:11:21.98047356 +0800 CST	deployed	cloud-1.0.0	1.0.1



# 检索已经发布的 release 的资源文件
[root@k8s-master1 test]# helm get manifest msa -n test


# 查看
[root@k8s-master1 test]# kubectl get all -n test
NAME                                       READY   STATUS    RESTARTS   AGE
pod/msa-deploy-consumer-6f8897bdb4-lv89v   1/1     Running   0          117s
pod/msa-deploy-consumer-6f8897bdb4-pc7kl   1/1     Running   0          117s
pod/msa-deploy-consumer-6f8897bdb4-rzhmx   1/1     Running   0          117s
pod/msa-deploy-job-85cd75b9fc-chxfq        1/1     Running   0          117s
pod/msa-deploy-job-85cd75b9fc-hkmjf        1/1     Running   0          117s
pod/msa-deploy-job-85cd75b9fc-x67k8        1/1     Running   0          117s
pod/msa-deploy-producer-5d9948bb7-4hkx6    1/1     Running   0          117s
pod/msa-deploy-producer-5d9948bb7-cn9wd    1/1     Running   0          117s
pod/msa-deploy-producer-5d9948bb7-r8q9b    1/1     Running   0          117s
pod/msa-eureka-0                           1/1     Running   0          117s
pod/msa-eureka-1                           1/1     Running   0          117s
pod/msa-eureka-2                           1/1     Running   0          117s
pod/msa-gateway-799b85ffd8-75ng8           1/1     Running   0          117s
pod/msa-gateway-799b85ffd8-c52wh           1/1     Running   0          117s
pod/msa-gateway-799b85ffd8-xcffm           1/1     Running   0          117s
pod/sentinel-server-8597dd6fcc-b6rxx       1/1     Running   0          117s
pod/xxl-job-7d8cdb957f-hr6jt               1/1     Running   0          117s
pod/xxl-job-7d8cdb957f-wdwl5               1/1     Running   0          117s
pod/xxl-job-7d8cdb957f-zctkh               1/1     Running   0          117s
pod/zipkin-server-c856888b8-64lpf          1/1     Running   0          117s
pod/zipkin-server-c856888b8-cswxp          1/1     Running   0          117s
pod/zipkin-server-c856888b8-mzvbq          1/1     Running   0          117s

NAME                          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
service/msa-deploy-consumer   ClusterIP   10.1.62.78     <none>        8912/TCP    117s
service/msa-deploy-job        ClusterIP   10.1.91.23     <none>        8913/TCP    117s
service/msa-deploy-producer   ClusterIP   10.1.69.228    <none>        8911/TCP    117s
service/msa-eureka            ClusterIP   10.1.209.34    <none>        10001/TCP   117s
service/msa-gateway           ClusterIP   10.1.229.185   <none>        8888/TCP    117s
service/sentinel-server       ClusterIP   10.1.138.238   <none>        8858/TCP    117s
service/xxl-job               ClusterIP   10.1.213.242   <none>        8080/TCP    117s
service/zipkin-server         ClusterIP   10.1.47.208    <none>        9411/TCP    117s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/msa-deploy-consumer   3/3     3            3           117s
deployment.apps/msa-deploy-job        3/3     3            3           117s
deployment.apps/msa-deploy-producer   3/3     3            3           117s
deployment.apps/msa-gateway           3/3     3            3           117s
deployment.apps/sentinel-server       1/1     1            1           117s
deployment.apps/xxl-job               3/3     3            3           117s
deployment.apps/zipkin-server         3/3     3            3           117s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/msa-deploy-consumer-6f8897bdb4   3         3         3       117s
replicaset.apps/msa-deploy-job-85cd75b9fc        3         3         3       117s
replicaset.apps/msa-deploy-producer-5d9948bb7    3         3         3       117s
replicaset.apps/msa-gateway-799b85ffd8           3         3         3       117s
replicaset.apps/sentinel-server-8597dd6fcc       1         1         1       117s
replicaset.apps/xxl-job-7d8cdb957f               3         3         3       117s
replicaset.apps/zipkin-server-c856888b8          3         3         3       117s

NAME                          READY   AGE
statefulset.apps/msa-eureka   3/3     117s
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
172.51.216.81 		zipkin.test.com
172.51.216.81 		sentinel.test.com
172.51.216.81 		xxljob.test.com
172.51.216.81       eureka.test.com
172.51.216.81       gateway.test.com


[root@k8s-master1 ext]# kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.1.253.108   <none>        80:31100/TCP,443:31826/TCP   2d20h
ingress-nginx-controller-admission   ClusterIP   10.1.236.3     <none>        443/TCP                      2d20h



# ingress地址
# 在本机浏览器输入:
http://zipkin.test.com:31100/
http://sentinel.test.com:31100/
http://xxljob.test.com:31100/
http://eureka.test.com:31100/
http://gateway.test.com:31100/



# 测试地址
# zipkin
http://zipkin.test.com:31100/zipkin
#sentinel
http://sentinel.test.com:31100/#/login
账户密码：sentinel/sentinel
# xxl-job
http://xxljob.test.com:31100/xxl-job-admin
账户密码：admin/123456
# eureka
http://eureka.test.com:31100/

# gateway
http://gateway.test.com:31100/dconsumer/hello
http://gateway.test.com:31100/dconsumer/say
http://gateway.test.com:31100/dproducer/hello
http://gateway.test.com:31100/dproducer/say
```

```shell
# 测试分布式任务调度平台（XXL-JOB）

# 配置服务端
# 参考文档：K8S-SpringCloud---外部服务.md    2.5.部署XXL-JOB执行器（msa-ext-job）


# 查看日志
[root@k8s-master1 test]# kubectl logs -f --tail=10 msa-deploy-job-85cd75b9fc-chxfq -n test
----job one 1---Tue Dec 21 15:20:44 CST 2021
----job one 1---Tue Dec 21 15:20:45 CST 2021
----job one 1---Tue Dec 21 15:20:46 CST 2021
----job one 1---Tue Dec 21 15:20:47 CST 2021
----job one 1---Tue Dec 21 15:20:48 CST 2021
----job one 1---Tue Dec 21 15:20:49 CST 2021
----job one 1---Tue Dec 21 15:20:50 CST 2021
----job one 1---Tue Dec 21 15:20:51 CST 2021
----job one 1---Tue Dec 21 15:20:52 CST 2021
----job one 1---Tue Dec 21 15:20:53 CST 2021
----job one 1---Tue Dec 21 15:20:54 CST 2021
----job one 1---Tue Dec 21 15:20:55 CST 2021
----job one 1---Tue Dec 21 15:20:56 CST 2021
```



