* TOC
{:toc}



## 一、概述



### 1.CD介绍

- **持续交付（英语：Continuous delivery，缩写为 CD），部署到测试环境（test）、预生产环境（pre）**
- **持续部署（英语：Continuous Deployment，缩写为 CD），将最终产品发布到生成环境（pro），给用户使用**



**持续部署到Docker环境：**

![](/images/devops/cicd/cicd-cd/cd-1.png)

**持续部署到kubernetes环境：**

![](/images/devops/cicd/cicd-cd/cd-2.png)



### 2.持续部署（CD）

#### 2.1.持续部署规划

**持续部署规划：**

- **部署环境：** 生产环境（pro）、灰度环境（pre）
- **部署脚本：**部署shell脚本文件使用GitLab进行版本控制，尽量保持与微服务版本关联
- **容器环境：** 可以部署到Kubernetes或Docker
- **K8S环境：**使用Helm部署微服务



#### 2.2.部署方案

- **方案一： Jenkins + GitLab +  Harbor + Docker**

![](/images/devops/cicd/cicd-cd/cd-3.png)



- **方案二： Jenkins + GitLab +  Harbor + Kubernetes**

![](/images/devops/cicd/cicd-cd/cd-4.png)



### 3.持续交付（CD）

#### 3.1.持续交付规划

**持续交付规划：**

- **部署环境：** 开发环境（dev）、测试环境（test）
- **部署脚本：**部署shell脚本文件使用GitLab进行版本控制，尽量保持与微服务版本关联
- **容器环境：** 可以部署到Kubernetes或Docker
- **K8S环境：**使用Helm部署微服务



#### 3.2.部署方案

**与持续部署的部署方案相同。**



### 4.持续部署流水线

- **部署脚本：**部署shell脚本文件使用GitLab进行版本控制，尽量保持与微服务版本关联
- **容器环境：** 可以部署到Kubernetes或Docker
- **K8S环境：**使用Helm部署微服务



**持续部署与持续交付流水线基本相同，只是选择的镜像版本和部署的环境不同，本文档只介绍持续部署流水线，持续交付流水线参考持续部署流水线。**





## 二、持续部署中间件



### 1.Jenkins

#### 1.1.Jenkins凭证管理  

##### 1.1.1.凭据管理介绍

**凭据可以用来存储需要密文保护的数据库密码、Gitlab密码信息、Docker私有仓库密码等，以便Jenkins可以和这些第三方的应用进行交互。**  



**1.安装Credentials Binding插件**
要在Jenkins使用凭证管理功能，需要安装Credentials Binding插件



**2.安装插件后，在这里管理所有凭证**  

![](/images/devops/cicd/cicd-cd/cd-5.png)



![](/images/devops/cicd/cicd-cd/cd-6.png)



##### 1.1.2.配置部署服务器凭证（Docker）

![](/images/devops/cicd/cicd-cd/cd-9.png)



##### 1.1.3.配置部署服务器凭证（k8s）

![](/images/devops/cicd/cicd-cd/cd-10.png)



#### 1.2.配置远程服务器

##### 1.2.1.Docker

**1.安装SSH插件**

![](/images/devops/cicd/cicd-cd/cd-7.png)

**2.添加凭证**

![](/images/devops/cicd/cicd-cd/cd-8.png)

![](/images/devops/cicd/cicd-cd/cd-11.png)



##### 1.2.2.Kubernetes

![](/images/devops/cicd/cicd-cd/cd-12.png)





### 2.Kubernetes

#### 2.1.创建名字空间

```shell
[root@k8s-master ~]# kubectl create ns devops
namespace/devops created

[root@k8s-master ~]# kubectl get ns
NAME                   STATUS   AGE
...
devops                 Active   6s
```



#### 2.2.创建私有镜像密钥Secret

K8S所有工作节点登陆Harbor镜像仓库

```shell
# 登陆 admin/admin
# docker login http://172.51.216.89:16888

[root@k8s-master ~]# docker login -u admin -p admin http://172.51.216.89:16888
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
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


[root@k8s-master ~]# cat ~/.docker/config.json |base64 -w 0
ewoJImF1dGhzIjogewoJCSIxNzIuNTEuMjE2Ljg5OjE2ODg4IjogewoJCQkiYXV0aCI6ICJZV1J0YVc0NllXUnRhVzQ9IgoJCX0KCX0KfQ==
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
  namespace: devops
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: ewoJImF1dGhzIjogewoJCSIxNzIuNTEuMjE2Ljg5OjE2ODg4IjogewoJCQkiYXV0aCI6ICJZV1J0YVc0NllXUnRhVzQ9IgoJCX0KCX0KfQ==


[root@k8s-master cicd]# kubectl apply -f harbor-secret.yaml 
secret/harborsecret created


[root@k8s-master cicd]# kubectl get secret -n devops
NAME                  TYPE                                  DATA   AGE
default-token-lc987   kubernetes.io/service-account-token   3      17m
harborsecret          kubernetes.io/dockerconfigjson        1      34s
```



#### 2.3.Helm构建微服务

##### 2.3.1.创建Helm Chart

```shell
# 创建 msa
[root@k8s-master devops]# helm create cloud -n devops
Creating cloud

[root@k8s-master devops]# ll
drwxr-xr-x 4 root root 93 Jan 23 17:38 cloud


[root@k8s-master devops]# tree
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

[root@k8s-master devops]# tree
.
└── cloud
    ├── charts
    ├── Chart.yaml
    ├── templates
    │   ├── msa-deploy-consumer.yaml
    │   ├── msa-deploy-producer.yaml
    │   ├── msa-eureka.yaml
    │   └── msa-gateway.yaml
    └── values.yaml

3 directories, 6 files
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



##### 2.3.2.注册中心（eureka-server）

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
      nodePort: {{ .Values.eureka.service.nodePort }} #对外暴露30011端口
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
              value: {{ .Values.eureka.env.profiles }}
  podManagementPolicy: "Parallel"
```



##### 2.3.3.网关（msa-gateway）

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
              value: {{ .Values.gateway.env.profiles }}
```



##### 2.3.4.生产者（msa-deploy-producer）

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
              value: {{ .Values.producer.env.profiles }}
            - name: DASHBOARD
              value: {{ .Values.producer.env.dashboard }}
```



##### 2.3.5.消费者（msa-deploy-consumer）

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
              value: {{ .Values.consumer.env.profiles }}
            - name: DASHBOARD
              value: {{ .Values.consumer.env.dashboard }}
```



##### 2.3.6.values

values.yaml 

```yaml
global:
  namespace: "devops"
  imagePullSecrets: "harborsecret"
  repository: "172.51.216.89:16888/springcloud/"
  env:
    eureka: "http://msa-eureka-0.msa-eureka.devops:10001/eureka/,http://msa-eureka-1.msa-eureka.devops:10001/eureka/,http://msa-eureka-2.msa-eureka.devops:10001/eureka/"


eureka:
  name: "msa-eureka"
  labels: "msa-eureka"
  service:
    type: "NodePort"
    port: 10001
    targetPort: 10001
    nodePort: 30011
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
    dashboard: "sentinel-server.devops.svc.cluster.local:8858"


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
    dashboard: "sentinel-server.devops.svc.cluster.local:8858"
```



#### 2.4.Helm Chart

##### 2.4.1.安装 push 插件

Helm服务器安装helm-push插件

```shell
# master(172.51.216.81)，安装helm服务器

# 安装
helm plugin install https://github.com/chartmuseum/helm-push 
# 查看已成功
helm plugin list


# 安装
[root@k8s-master ~]# helm plugin install https://github.com/chartmuseum/helm-push

# 查看已成功
[root@k8s-master helm]# helm plugin list
NAME   	VERSION	DESCRIPTION                      
cm-push	0.10.1 	Push chart package to ChartMuseum
```



##### 2.4.2.添加Helm仓库

```shell
# 添加 helm repo
helm repo add --username admin --password admin harborcloud http://172.51.216.89:16888/chartrepo/springcloud
helm repo list


# 添加 helm repo
[root@k8s-master ~]# helm repo add --username admin --password admin harborcloud http://172.51.216.89:16888/chartrepo/springcloud
"harborcloud" has been added to your repositories


# 查看
[root@k8s-master ~]# helm repo list
NAME         	URL                                                   
bitnami      	https://charts.bitnami.com/bitnami                    
ingress-nginx	https://kubernetes.github.io/ingress-nginx            
ali-stable   	https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
stable       	http://mirror.azure.cn/kubernetes/charts              
aliyun       	https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
elastic      	https://helm.elastic.co                               
harborcloud  	http://172.51.216.89:16888/chartrepo/springcloud


# 更新
[root@k8s-master ~]# helm repo update
```



##### 2.4.3.上传Helm仓库

```shell
$ helm cm-push mychart/ chartmuseum
Pushing mychart-0.3.2.tgz to chartmuseum...
Done.


$ helm cm-push mychart/ --version="1.2.3" chartmuseum
Pushing mychart-1.2.3.tgz to chartmuseum...
Done.


$ helm cm-push mychart-0.3.2.tgz chartmuseum
Pushing mychart-0.3.2.tgz to chartmuseum...
Done.
```



```shell
# 推送

[root@k8s-master1 test]# helm cm-push cloud/ harborcloud
Pushing cloud-1.0.0.tgz to harborcloud...
Done.


[root@k8s-master1 test]# helm cm-push cloud-1.0.0.tgz harborcloud
Pushing cloud-1.0.0.tgz to harborcloud...
Done.


# 版本号维护（Helm Chart的版本号）
# vim Chart.yaml
version: 2.0.0
```



### 3.Docker

#### 3.1.配置docker私有镜像源

devops节点配置docker私有镜像源

```shell
# 配置docker私有镜像源（172.51.216.81，172.51.216.82，172.51.216.83，172.51.216.84，172.51.216.89）
 
#把Harbor地址加入到Docker信任列表

vi /etc/docker/daemon.json
"insecure-registries": ["http://IP:端口"]

{
   "registry-mirrors": ["https://gcctk8ld.mirror.aliyuncs.com"],
   "insecure-registries": ["http://172.51.216.89:16888"]
}


#重启Docker
systemctl daemon-reload
systemctl restart docker
```



### 4.Harbor

```shell
# 登陆harbor

# docker login -u 用户名 -p 密码 http://IP:端口
docker login -u admin -p admin http://172.51.216.89:16888
 
# 登出：
docker logout http://172.51.216.89:16888


# 外网地址：
http://172.51.216.89:16888
admin
admin
```

![](/images/devops/cicd/cicd-cd/cd-25.png)



### 5.GitLab

#### 5.1.GitLab创建部署项目

**1.GitLab创建项目**

![](/images/devops/cicd/cicd-cd/cd-13.png)

```shell
# 项目地址

git@172.51.216.89:cicd-group/k8s-deploy.git
```



#### 5.2.创建Docker部署脚本

**1.编写部署脚本**

```shell
# 克隆
# git clone git@172.51.216.89:cicd-group/k8s-deploy.git


Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/cicd
$ git clone git@172.51.216.89:cicd-group/k8s-deploy.git
Cloning into 'k8s-deploy'...
warning: You appear to have cloned an empty repository.

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/cicd
$ ll
total 4
drwxr-xr-x 1 Administrator 197121 0 Feb 10 11:44 k8s-deploy/
```

```shell
# 创建脚本


$ cd k8s-deploy/
$ mkdir docker
$ cd docker/

$ vim deploy.sh
```

```shell
# 脚本deploy.sh


#! /bin/sh

echo "登录Harbor ---"
docker login -u admin -p admin http://172.51.216.89:16888

echo "运行注册中心 ---"
docker pull 172.51.216.89:16888/springcloud/msa-eureka:2.0.0
docker run -d --network host --restart=always --name msa-eureka 172.51.216.89:16888/springcloud/msa-eureka:2.0.0

echo "运行网关 ---"
docker pull 172.51.216.89:16888/springcloud/msa-gateway:2.0.0
docker run -d --network host --restart=always --name msa-gateway 172.51.216.89:16888/springcloud/msa-gateway:2.0.0

echo "运行生产者 ---"
docker pull 172.51.216.89:16888/springcloud/msa-deploy-producer:2.0.0
docker run -d --network host --restart=always --name msa-deploy-producer 172.51.216.89:16888/springcloud/msa-deploy-producer:2.0.0

echo "运行消费者 ---"
docker pull 172.51.216.89:16888/springcloud/msa-deploy-consumer:2.0.0
docker run -d --network host --restart=always --name msa-deploy-consumer 172.51.216.89:16888/springcloud/msa-deploy-consumer:2.0.0

echo "完成 ---"
```



**2.上传GitLab仓库**

```shell
$ git add .
$ git commit -m "创建部署脚本"

# 推送远程仓库
$ git push origin master
```

![](/images/devops/cicd/cicd-cd/cd-14.png)



#### 5.3.创建Kubernetes部署脚本

**Helm源码上传GitLab仓库**

```shell
# 克隆
[root@k8s-master devops]# git clone git@172.51.216.89:cicd-group/k8s-deploy.git


[root@k8s-master devops]# cd k8s-deploy/
[root@k8s-master k8s-deploy]# mkdir k8s
[root@k8s-master k8s-deploy]# ll
total 0
drwxr-xr-x 2 root root 23 Jan 23 19:09 docker
drwxr-xr-x 2 root root  6 Jan 23 19:15 k8s

# 复制文件
[root@k8s-master devops]# cp -r cloud k8s-deploy/k8s/cloud
```

```shell
# 提交本地仓库
[root@k8s-master k8s-deploy]# git add .
[root@k8s-master k8s-deploy]# git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#	new file:   k8s/cloud/.helmignore
#	new file:   k8s/cloud/Chart.yaml
#	new file:   k8s/cloud/templates/msa-deploy-consumer.yaml
#	new file:   k8s/cloud/templates/msa-deploy-producer.yaml
#	new file:   k8s/cloud/templates/msa-eureka.yaml
#	new file:   k8s/cloud/templates/msa-gateway.yaml
#	new file:   k8s/cloud/values.yaml
#

[root@k8s-master k8s-deploy]# git commit -m "增加helm文件"
[master 8f4919f] 增加helm文件
 7 files changed, 287 insertions(+)
 create mode 100644 k8s/cloud/.helmignore
 create mode 100644 k8s/cloud/Chart.yaml
 create mode 100644 k8s/cloud/templates/msa-deploy-consumer.yaml
 create mode 100644 k8s/cloud/templates/msa-deploy-producer.yaml
 create mode 100644 k8s/cloud/templates/msa-eureka.yaml
 create mode 100644 k8s/cloud/templates/msa-gateway.yaml
 create mode 100644 k8s/cloud/values.yaml


# 上传仓库
[root@k8s-master k8s-deploy]# git remote -v
origin  git@172.51.216.89:cicd-group/k8s-deploy.git (fetch)
origin  git@172.51.216.89:cicd-group/k8s-deploy.git (push)


[root@k8s-master k8s-deploy]# git push origin master
Counting objects: 13, done.
Delta compression using up to 6 threads.
Compressing objects: 100% (11/11), done.
Writing objects: 100% (12/12), 2.51 KiB | 0 bytes/s, done.
Total 12 (delta 3), reused 0 (delta 0)
To git@39.96.178.134:test-group/k8s-deploy.git
   42d20b1..8f4919f  master -> master
```

![](/images/devops/cicd/cicd-cd/cd-26.png)



#### 5.4.配置SSH免密登陆GitLab

##### 5.4.1.配置服务器说明

**需要配置SSH免密登陆的服务器：**

- Dokcer部署服务器：172.51.216.89
- Kubernetes部署服务器：172.51.216.81（master）



##### 5.4.2.配置SSH免密登陆

**1.设置用户签名**

```shell
git config --global user.name hollysyshs
git config --global user.email hollysysapple@126.com


[root@k8s-master ~]# git config --list
user.name=hollysyshs
user.email=hollysysapple@126.com


[root@k8s-master ~]#  vim ~/.gitconfig 
```



**2.生成秘钥**

```shell
$ ssh-keygen
$ ssh-keygen -C hollysysapple@126.com


[root@k8s-master ~]# ssh-keygen -C hollysysapple@126.com
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:9oykcfXcFFKHYOn+ZvM/qDbB9+T02rXAm2C0GdkXG0M hollysysapple@126.com
The key's randomart image is:
+---[RSA 2048]----+
|            ++oE.|
|           ...oo |
|          ..  .+ |
|         . o+o  =|
|      . S .=o..o |
|       * +.o*..o |
|      . . o=o+* o|
|          .o..O=+|
|          ..o=.=*|
+----[SHA256]-----+


[root@k8s-master ~]# cat .ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC2LEijB+gA5s4tU2Flk1PIk69p4RBNi1c6JW2nkxGUwB9i9t+DfpcqOO2KBP1pc7pyqYUoWyASLTKjQ+94w5wpiCObai747ZtaWP3EuotR/sCsdFOlYEBZQ+S74aptNl3WLjve40prmQL5iBdX8ROHdK9gS2aIC9Gdun9+6Wz8Dw5fTGkWqMbKr8O9881x2nZ1cq6Y1eZ5Eut/G7DxPWwEVgz7vHXiGNEO8VYmweRHA+OQpDYMfklAsZO40AhNkVz6q4t+jaruPJWggaNd+4iD+3mwAq7CDt4RJIf2EbEFOg8kfZbwEwsrLgNF3P9B8rWCY1MnRXDQVsW9Ygn8Qhg9 hollysysapple@126.com
```



**3.GitLab配置SSH**

**查看你生成的公钥：**

vim id_rsa.pub

就可以查看到你的公钥



**一定要root登陆！！！**

**root登陆GitLab账号，点击用户图像，然后 Settings -> 左栏点击 SSH keys**

![](/images/devops/cicd/cicd-cd/cd-15.png)

**复制公钥内容，粘贴进“Key”文本区域内，取名字**

**点击Add Key**

![](/images/devops/cicd/cicd-cd/cd-16.png)

![](/images/devops/cicd/cicd-cd/cd-17.png)



**测试**

```shell
[root@k8s-master ~]# git clone git@172.51.216.89:cicd-group/k8s-deploy.git
Cloning into 'k8s-deploy'...
The authenticity of host '172.51.216.89 (172.51.216.89)' can't be established.
ECDSA key fingerprint is SHA256:ctbaLG/l0ErH2WKma4Gx/20ewRcyoCwPHoyKej/GTnc.
ECDSA key fingerprint is MD5:b8:9a:a7:cd:72:26:43:32:80:ef:6e:57:67:d0:bf:bf.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.51.216.89' (ECDSA) to the list of known hosts.
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 4 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (4/4), done.



[root@k8s-master ~]# ll
drwxr-xr-x   4 root root        32 Jan 23 19:04 k8s-deploy
```





## 三、持续部署流水线（Docker）



### 1.流水线设计

- **微服务部署脚本存放在GitLab服务器，进行版本维护**
- **Jenkins调用Docker服务器远程执行脚本deploy.sh**
- **Docker服务器远程执行脚本deploy.sh，从GitLab服务器拉取部署脚本，GitLab脚本部署微服务**



### 2.创建流水线项目

![](/images/devops/cicd/cicd-cd/cd-18.png)

![](/images/devops/cicd/cicd-cd/cd-19.png)

![](/images/devops/cicd/cicd-cd/cd-20.png)



```shell
# 执行远程脚本

sh /root/deploy.sh
```



### 3.Docker服务器脚本

```shell
# Docker服务器
IP：172.51.216.89


# 脚本deploy.sh
[root@tencent-22 ~]# vim deploy.sh

#! /bin/sh

echo "克隆项目 ---"
rm -rf k8s-deploy
git clone git@172.51.216.89:cicd-group/k8s-deploy.git

echo "部署服务 ---"
chmod +x k8s-deploy/docker/deploy.sh
./k8s-deploy/docker/deploy.sh


# 修改权限
[root@localhost ~]# chmod +x deploy.sh
```



### 4.构建项目

```shell
# 镜像
[root@localhost ~]# docker images
REPOSITORY                                            TAG             IMAGE ID       CREATED         SIZE
172.51.216.89:16888/springcloud/msa-deploy-consumer   2.0.0           f53258c0a16e   23 hours ago    164MB
172.51.216.89:16888/springcloud/msa-deploy-producer   2.0.0           b92630f3bbd2   23 hours ago    163MB
172.51.216.89:16888/springcloud/msa-gateway           2.0.0           ce4c7c546da7   23 hours ago    152MB
172.51.216.89:16888/springcloud/msa-eureka            2.0.0           841d2d7f323b   23 hours ago    155MB


# 容器
[root@localhost ~]# docker ps -a
CONTAINER ID   IMAGE                                                       COMMAND                  CREATED          STATUS                PORTS                       NAMES
c04e572bfbad   172.51.216.89:16888/springcloud/msa-deploy-consumer:2.0.0   "/bin/sh -c 'java -D…"   17 seconds ago   Up 17 seconds                                     msa-deploy-consumer
9b021ddaa2c5   172.51.216.89:16888/springcloud/msa-deploy-producer:2.0.0   "/bin/sh -c 'java -D…"   21 seconds ago   Up 20 seconds                                     msa-deploy-producer
c0869c0935b5   172.51.216.89:16888/springcloud/msa-gateway:2.0.0           "/bin/sh -c 'java -D…"   23 seconds ago   Up 23 seconds                                     msa-gateway
83193c11d557   172.51.216.89:16888/springcloud/msa-eureka:2.0.0            "/bin/sh -c 'java -D…"   26 seconds ago   Up 25 seconds



# 调用接口
[root@localhost ~]# curl http://localhost:8911/hello
hello，this is first messge! ### -----2.0.0Thu Feb 10 14:09:37 CST 2022

[root@localhost ~]# curl http://localhost:8912/hello
hello，this is first messge! ### -----2.0.0Thu Feb 10 14:09:55 CST 2022  openfeign + sentinel!!!



# 注册中心地址
http://172.51.216.89:10001/
```

![](/images/devops/cicd/cicd-cd/cd-21.png)







## 四、持续部署流水线（Kubernetes）



### 1.流水线设计

- **微服务部署脚本存放在GitLab服务器，进行版本维护；**
- **Jenkins调用Docker服务器远程执行脚本deploy.sh；**
- **Docker服务器远程执行脚本deploy.sh，从GitLab服务器拉取部署脚本，GitLab脚本部署微服务；**



### 2.创建流水线项目

![](/images/devops/cicd/cicd-cd/cd-22.png)

![](/images/devops/cicd/cicd-cd/cd-23.png)

![](/images/devops/cicd/cicd-cd/cd-24.png)



```shell
# 执行远程脚本

sh /root/deploy.sh
```



### 3.Kubernetes服务器脚本

```shell
# Kubernetes master服务器
IP：172.51.216.81



# 脚本deploy.sh
[root@k8s-master ~]# vim deploy.sh

#! /bin/sh

echo "克隆项目 ---"
rm -rf k8s-deploy
git clone git@172.51.216.89:cicd-group/k8s-deploy.git

echo "部署服务 ---"
helm install msa k8s-deploy/k8s/cloud -n devops



# 修改权限
[root@k8s-master ~]# chmod +x deploy.sh
```



### 4.构建项目

```shell
[root@k8s-master cicd]# helm list -n devops
NAME	NAMESPACE	REVISION	UPDATED                               	STATUS  	CHART      	APP VERSION
msa 	devops   	1       	2022-02-10 16:23:49.12918863 +0800 CST	deployed	cloud-0.1.0	1.0.0 



[root@k8s-master cicd]# kubectl get all -n devops
NAME                                       READY   STATUS    RESTARTS   AGE
pod/msa-deploy-consumer-5c959444cc-hgfht   1/1     Running   0          2m11s
pod/msa-deploy-consumer-5c959444cc-kzk6d   1/1     Running   0          2m11s
pod/msa-deploy-consumer-5c959444cc-wbxvh   1/1     Running   0          2m11s
pod/msa-deploy-producer-68c94df86b-pwc66   1/1     Running   0          2m11s
pod/msa-deploy-producer-68c94df86b-vz7tz   1/1     Running   0          2m11s
pod/msa-deploy-producer-68c94df86b-xqp6n   1/1     Running   0          2m11s
pod/msa-eureka-0                           1/1     Running   0          2m11s
pod/msa-eureka-1                           1/1     Running   0          2m11s
pod/msa-eureka-2                           1/1     Running   0          2m11s
pod/msa-gateway-5f96ccb9d5-8j7xz           1/1     Running   0          2m11s
pod/msa-gateway-5f96ccb9d5-rzx2w           1/1     Running   0          2m11s
pod/msa-gateway-5f96ccb9d5-x2tr4           1/1     Running   0          2m11s

NAME                          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
service/msa-deploy-consumer   ClusterIP   10.103.4.107    <none>        8912/TCP          2m11s
service/msa-deploy-producer   ClusterIP   10.110.94.225   <none>        8911/TCP          2m11s
service/msa-eureka            NodePort    10.96.12.117    <none>        10001:30011/TCP   2m11s
service/msa-gateway           ClusterIP   10.98.211.185   <none>        8888/TCP          2m11s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/msa-deploy-consumer   3/3     3            3           2m11s
deployment.apps/msa-deploy-producer   3/3     3            3           2m11s
deployment.apps/msa-gateway           3/3     3            3           2m11s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/msa-deploy-consumer-5c959444cc   3         3         3       2m11s
replicaset.apps/msa-deploy-producer-68c94df86b   3         3         3       2m11s
replicaset.apps/msa-gateway-5f96ccb9d5           3         3         3       2m11s

NAME                          READY   AGE
statefulset.apps/msa-eureka   3/3     2m11s
```



```shell
# 调用接口
[root@k8s-master cicd]# curl http://10.110.94.225:8911/hello
hello，this is first messge! ### -----2.0.0Thu Feb 10 16:34:44 CST 2022

[root@k8s-master cicd]# curl http://10.103.4.107:8912/hello
hello，this is first messge! ### -----2.0.0Thu Feb 10 16:35:16 CST 2022  openfeign + sentinel!!!



# 注册中心地址
http://172.51.216.81:30011/
```

![](/images/devops/cicd/cicd-cd/cd-27.png)



### 5.上传Helm Chart到Harbor

#### 5.1.Kubernetes服务器脚本

```shell
# Kubernetes master服务器
IP：172.51.216.81



# 脚本deploy.sh
[root@k8s-master ~]# vim deploy.sh

#! /bin/sh

echo "克隆项目 ---"
rm -rf k8s-deploy
git clone git@172.51.216.89:cicd-group/k8s-deploy.git

echo "上传Helm Chart ---"
helm cm-push k8s-deploy/k8s/cloud/ harborcloud

echo "升级服务 ---"
helm upgrade msa k8s-deploy/k8s/cloud -n devops



# 修改权限
[root@k8s-master ~]# chmod +x deploy.sh
```

```shell
# 注意：

# 第一次安装
echo "部署服务 ---"
helm install msa k8s-deploy/k8s/cloud -n devops

# 第二次升级
echo "升级服务 ---"
helm upgrade msa k8s-deploy/k8s/cloud -n devops
```



#### 5.2.测试

发布版本：1.0.0、2.0.0







## 五、微服务持续部署流水线（Kubernetes）



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



#### 1.4.调用链监控（Zipkin）

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



#### 1.5.分布式任务调度平台（XXL-JOB）

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



#### 1.6.XXL-JOB执行器（msa-deploy-job）

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



#### 1.9.Ingress

**调用链监控（Zipkin）、流量控制（Sentinel）、分布式任务调度平台（XXL-JOB）、注册中心（eureka-server）、网关（msa-gateway）。**



**Http代理**

**创建 msa-pro-http.yaml**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: msa-http
  namespace: devops
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



#### 1.10.values

values.yaml 

```yaml
global:
  namespace: "devops"
  imagePullSecrets: "harborsecret"
  repository: "172.51.216.89:16888/springcloud/"
  env:
    eureka: "http://msa-eureka-0.msa-eureka.devops:10001/eureka/,http://msa-eureka-1.msa-eureka.devops:10001/eureka/,http://msa-eureka-2.msa-eureka.devops:10001/eureka/"
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
    tag: "3.0.0"
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
    dashboard: "sentinel-server.devops.svc.cluster.local:8858"


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
    dashboard: "sentinel-server.devops.svc.cluster.local:8858"
```



### 2.创建流水线项目

CI流水线增加执行器项目。

![](/images/devops/cicd/cicd-cd/cd-28.png)



### 3.Kubernetes服务器脚本

```shell
# Kubernetes master服务器
IP：172.51.216.81


# 脚本deploy.sh
[root@k8s-master ~]# vim deploy.sh

#! /bin/sh

echo "克隆项目 ---"
rm -rf k8s-deploy
git clone git@172.51.216.89:cicd-group/k8s-deploy.git

echo "上传Helm Chart ---"
helm cm-push k8s-deploy/msa/cloud/ harborcloud

echo "升级服务 ---"
helm upgrade msa k8s-deploy/msa/cloud -n devops


# 修改权限
[root@k8s-master ~]# chmod +x deploy.sh
```

```shell
# 注意：

# 第一次安装
echo "部署服务 ---"
helm install msa k8s-deploy/msa/cloud -n devops

# 第二次升级
echo "升级服务 ---"
helm upgrade msa k8s-deploy/msa/cloud -n devops
```



### 4.构建项目

```shell
[root@k8s-master ~]# helm list -n devops
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	devops   	1       	2022-02-16 17:23:59.903302417 +0800 CST	deployed	cloud-1.0.0	1.0.0      


[root@k8s-master ~]# kubectl get all -n devops
NAME                                       READY   STATUS    RESTARTS   AGE
pod/msa-deploy-consumer-89999764-86zjs     1/1     Running   0          15h
pod/msa-deploy-consumer-89999764-pzmgt     1/1     Running   0          15h
pod/msa-deploy-consumer-89999764-zb699     1/1     Running   0          15h
pod/msa-deploy-job-6688679b54-9jhxg        1/1     Running   0          15h
pod/msa-deploy-job-6688679b54-ddjfl        1/1     Running   0          15h
pod/msa-deploy-job-6688679b54-wg76q        1/1     Running   0          15h
pod/msa-deploy-producer-548bb4696f-9p5hh   1/1     Running   0          15h
pod/msa-deploy-producer-548bb4696f-lrmwd   1/1     Running   0          15h
pod/msa-deploy-producer-548bb4696f-vlcxs   1/1     Running   0          15h
pod/msa-eureka-0                           1/1     Running   0          15h
pod/msa-eureka-1                           1/1     Running   0          15h
pod/msa-eureka-2                           1/1     Running   0          15h
pod/msa-gateway-5cb698b484-4rxgc           1/1     Running   0          15h
pod/msa-gateway-5cb698b484-7bhmz           1/1     Running   0          15h
pod/msa-gateway-5cb698b484-tn9kt           1/1     Running   0          15h
pod/sentinel-server-8597dd6fcc-lnd78       1/1     Running   0          15h
pod/xxl-job-7d8cdb957f-kn5qn               1/1     Running   0          15h
pod/xxl-job-7d8cdb957f-sklbk               1/1     Running   0          15h
pod/xxl-job-7d8cdb957f-x6wxt               1/1     Running   0          15h
pod/zipkin-server-c856888b8-chrsh          1/1     Running   0          15h
pod/zipkin-server-c856888b8-qzj4k          1/1     Running   0          15h
pod/zipkin-server-c856888b8-xnlqf          1/1     Running   0          15h

NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
service/msa-deploy-consumer   ClusterIP   10.96.182.128    <none>        8912/TCP    15h
service/msa-deploy-job        ClusterIP   10.103.160.62    <none>        8913/TCP    15h
service/msa-deploy-producer   ClusterIP   10.99.181.7      <none>        8911/TCP    15h
service/msa-eureka            ClusterIP   10.99.201.233    <none>        10001/TCP   15h
service/msa-gateway           ClusterIP   10.104.230.33    <none>        8888/TCP    15h
service/sentinel-server       ClusterIP   10.103.182.44    <none>        8858/TCP    15h
service/xxl-job               ClusterIP   10.103.205.158   <none>        8080/TCP    15h
service/zipkin-server         ClusterIP   10.101.99.107    <none>        9411/TCP    15h

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/msa-deploy-consumer   3/3     3            3           15h
deployment.apps/msa-deploy-job        3/3     3            3           15h
deployment.apps/msa-deploy-producer   3/3     3            3           15h
deployment.apps/msa-gateway           3/3     3            3           15h
deployment.apps/sentinel-server       1/1     1            1           15h
deployment.apps/xxl-job               3/3     3            3           15h
deployment.apps/zipkin-server         3/3     3            3           15h

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/msa-deploy-consumer-89999764     3         3         3       15h
replicaset.apps/msa-deploy-job-6688679b54        3         3         3       15h
replicaset.apps/msa-deploy-producer-548bb4696f   3         3         3       15h
replicaset.apps/msa-gateway-5cb698b484           3         3         3       15h
replicaset.apps/sentinel-server-8597dd6fcc       1         1         1       15h
replicaset.apps/xxl-job-7d8cdb957f               3         3         3       15h
replicaset.apps/zipkin-server-c856888b8          3         3         3       15h

NAME                          READY   AGE
statefulset.apps/msa-eureka   3/3     15h
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


[root@k8s-master ~]# kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.106.250.47   <none>        80:30002/TCP,443:30561/TCP   2d6h
ingress-nginx-controller-admission   ClusterIP   10.100.54.206   <none>        443/TCP                      2d6h


# ingress地址
# 在本机浏览器输入:
http://zipkin.pro.com:30002/
http://sentinel.pro.com:30002/
http://xxljob.pro.com:30002/
http://eureka.pro.com:30002/
http://gateway.pro.com:30002/


# 测试地址
# zipkin
http://zipkin.pro.com:30002/zipkin
#sentinel
http://sentinel.pro.com:30002/#/login
账户密码：sentinel/sentinel
# xxl-job
http://xxljob.pro.com:30002/xxl-job-admin
账户密码：admin/123456
# eureka
http://eureka.pro.com:30002/

# gateway
http://gateway.pro.com:30002/dconsumer/hello
http://gateway.pro.com:30002/dconsumer/say
http://gateway.pro.com:30002/dproducer/hello
http://gateway.pro.com:30002/dproducer/say
```

```shell
# 测试分布式任务调度平台（XXL-JOB）

# 配置服务端
# 参考文档：K8S-SpringCloud---外部服务.md    2.5.部署XXL-JOB执行器（msa-ext-job）


# 查看日志
[root@k8s-master ~]# kubectl logs -f --tail=10 msa-deploy-job-874f5db89-gpjxf -n devops
----job one 1---Thu Feb 17 09:47:10 CST 2022
----job one 1---Thu Feb 17 09:47:40 CST 2022
----job one 1---Thu Feb 17 09:48:09 CST 2022
----job one 1---Thu Feb 17 09:48:30 CST 2022
----job one 1---Thu Feb 17 09:49:00 CST 2022
----job one 1---Thu Feb 17 09:49:30 CST 2022
----job one 1---Thu Feb 17 09:50:00 CST 2022
----job one 1---Thu Feb 17 09:50:30 CST 2022
----job one 1---Thu Feb 17 09:51:00 CST 2022
----job one 1---Thu Feb 17 09:51:30 CST 2022
----job one 1---Thu Feb 17 09:52:00 CST 2022
```



