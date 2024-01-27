* TOC
{:toc}



## 一、部署方案



### 1.微服务部署方案

**1.架构图**



![](/images/kubernetes/pro/microservice/dep-2.png)



**2.部署图**

![](/images/kubernetes/pro/microservice/dep-3.png)



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

```yaml
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





## 二、环境部署规划



### 1.环境介绍

开发过程中四个环境分别是：pro、pre、test、dev环境，中文名字：生产环境、灰度环境、测试环境、开发环境。

- **pro环境：**生产环境，面向外部用户的环境，连接上互联网即可访问的正式环境
- **pre环境：**灰度环境，外部用户可以访问，但是服务器配置相对低，其它和生产一样
- **test环境：**测试环境，外部用户无法访问，专门给测试人员使用的，版本相对稳定
- **dev环境：**开发环境，外部用户无法访问，开发人员使用，版本变动很大

![](/images/kubernetes/pro/microservice/sc-1.png)



### 2.微服务环境规划

**微服务部署原则：**

1. 微服务采用Helm部署
2. pro、pre、test、dev环境，分别部署在名字空间：pro、pre、test、dev
3. 通过修改 **values.yaml** 进行部署升级
4. Harbor作为私有镜像仓库，存储微服务镜像
4. Harbor作为Helm仓库



