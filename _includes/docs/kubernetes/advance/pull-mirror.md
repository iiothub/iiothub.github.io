* TOC
{:toc}



## 一、概述



由于国内的网络原因，在搭建环境时，经常无法pull到这些镜像。那我们可以考虑曲线救国，利用aliyun搭建自己的镜像仓库（当然你也可以在Docker Hub搭建）

**前提条件：**已注册GitHub及aliyun账户

**基本原理：**

1. 关联GitHub，配置aliyun自动构建镜像
2. 在Docker主机pull构建的镜像，重新docker tag为指定的镜像名




### 1.GitHub（Windows）

#### 1.1.创建代码仓库

登录GitHub，创建代码仓库，比如 docker-image-lib

![](/images/kubernetes/advance/images-8.png)

![](/images/kubernetes/advance/images-9.png)



#### 1.2.安装GitHub客户端

```shell
#客户端下载：
https://desktop.github.com/


# 本地仓库地址
D:\dev-tct\GitHub

# clone项目
docker-image-lib


# 创建Dockerfile
FROM registry.opensource.zalan.do/acid/postgres-operator:v1.7.1
MAINTAINER test
```

![](/images/kubernetes/advance/images-10.png)

![](/images/kubernetes/advance/images-11.png)



### 2.阿里云镜像服务

#### 2.1.访问镜像服务

![](/images/kubernetes/advance/images-1.png)

![](/images/kubernetes/advance/images-2.png)



#### 2.2.创建命名空间

创建一个命名空间，"默认仓库类型" 公有 或 私有 都可以。我这里是 私有。 比如我这里就叫 k8s-middleware，如图：

![](/images/kubernetes/advance/images-3.png)

![](/images/kubernetes/advance/images-4.png)

![](/images/kubernetes/advance/images-5.png)



#### 2.3.创建镜像仓库

创建一个镜像仓库，地域选择离自己近的，命名空间选择上面创建的 k8s-middleware，仓库名称为 image-lib，仓库类型任意（我这选择公开），如图：

![](/images/kubernetes/advance/images-6.png)

点击『下一步』，开始关联代码仓库，这里选择 GitHub，关联第一步在 GitHub 上创建的代码仓库：

![](/images/kubernetes/advance/images-7.png)

![](/images/kubernetes/advance/images-12.png)

![](/images/kubernetes/advance/images-13.png)

可以看到，除了 GitHub ，还有很多其它的选项，如果 GitHub 访问也吃力的话，可以选择其它的类型。

勾选『海外机器构建』，第一个自动构建镜像勾不勾选都可以，后面配置构建策略的时候可以更改。

然后点击『创建镜像仓库』



#### 2.4.添加构建规则

这一步是配置构建规则，比如分支名、Dockerfile 路径等信息，如图：

![](/images/kubernetes/advance/images-14.png)



进入构建页面，点击『添加规则』，如图：

![](/images/kubernetes/advance/images-15.png)



添加一条构建规则，如图：

![](/images/kubernetes/advance/images-16.png)





#### 2.5.构建镜像

创建完构建规则后，点击『立即构建』，如图：

![](/images/kubernetes/advance/images-17.png)

稍等会儿，看到构建状态成功，就表示构建完成，如图：

![](/images/kubernetes/advance/images-18.png)



#### 2.6.获取镜像

镜像构建完成了，开始获取镜像，如图：

![](/images/kubernetes/advance/images-19.png)

按照如上指南操作即可，比如我这里可以这样获取：

```shell
# docker pull registry.cn-beijing.aliyuncs.com/k8s-middleware/image-lib:[镜像版本号]


docker pull registry.cn-beijing.aliyuncs.com/k8s-middleware/image-lib:latest
```



#### 2.7.设置访问凭证

注意，第一步 docker login 的时候，需要输入的密码并不是阿里云的登录密码，而是这里单独设置的：

**密码： 111111**

![](/images/kubernetes/advance/images-20.png)

![](/images/kubernetes/advance/images-21.png)



### 3.拉取墙外镜像

```shell
# 下载镜像之后，修改 Tag ；
# 理论上通过这种方式可以构建任意国外被墙的镜像了


# 拉取镜像
docker pull registry.cn-beijing.aliyuncs.com/k8s-middleware/image-lib/postgres-operator:latest

# 打标签
docker tag registry.cn-beijing.aliyuncs.com/k8s-middleware/image-lib/postgres-operator:latest registry.opensource.zalan.do/acid/postgres-operator:v1.7.1
```

```shell
# 登录阿里云Docker Registry
# 公有仓库可以不登录

$ docker login --username=xxxxxx registry.cn-beijing.aliyuncs.com
# 用于登录的用户名为阿里云账号全名，密码为开通服务时设置的密码。
# 您可以在访问凭证页面修改凭证密码。


# 密码：******
[root@k8s-node1 ~]# docker login --username=xxxxxx registry.cn-beijing.aliyuncs.com
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```



**拉取镜像打标签**

```shell
# 公有仓库可以不登录


# 拉取镜像
docker pull registry.cn-beijing.aliyuncs.com/k8s-middleware/image-lib/postgres-operator:latest

docker pull registry.cn-beijing.aliyuncs.com/k8s-middleware/image-lib:latest


# 打标签
docker tag registry.cn-beijing.aliyuncs.com/k8s-middleware/image-lib:latest registry.opensource.zalan.do/acid/postgres-operator:v1.7.1


---------------------------------------------
# 拉取镜像
[root@k8s-master postgres-operator]# docker pull registry.cn-beijing.aliyuncs.com/k8s-middleware/image-lib:latest
latest: Pulling from k8s-middleware/image-lib
e519532ddf75: Pull complete 
7dd595794cc3: Pull complete 
06ea1f0d228f: Pull complete 
9a2998dd079a: Pull complete 
7137e6464013: Pull complete 
9687ee2d0503: Pull complete 
6a11152d7c6d: Pull complete 
Digest: sha256:84d4bc9b0d7c3d5d06dcd43e55cedd011acf4d2810daf4c7e6d02428e6c706a3
Status: Downloaded newer image for registry.cn-beijing.aliyuncs.com/k8s-middleware/image-lib:latest
registry.cn-beijing.aliyuncs.com/k8s-middleware/image-lib:latest


[root@k8s-master postgres-operator]# docker images
REPOSITORY                                                        TAG            IMAGE ID       CREATED         SIZE
registry.cn-beijing.aliyuncs.com/k8s-middleware/image-lib         latest         3b16e3baa01e   5 weeks ago     70.4MB


# 打标签
[root@k8s-master postgres-operator]# docker tag registry.cn-beijing.aliyuncs.com/k8s-middleware/image-lib:latest registry.opensource.zalan.do/acid/postgres-operator:v1.7.1

[root@k8s-master postgres-operator]# docker images
REPOSITORY                                                        TAG            IMAGE ID       CREATED         SIZE
registry.cn-beijing.aliyuncs.com/k8s-middleware/image-lib         latest         3b16e3baa01e   5 weeks ago     70.4MB
registry.opensource.zalan.do/acid/postgres-operator               v1.7.1         3b16e3baa01e   5 weeks ago     70.4MB
```



理论上通过这种方式可以构建任意国外被墙的镜像了 ，类似：

![](/images/kubernetes/advance/images-22.png)





## 二、实践



### 1.postgres-operator（zalando）

#### 1.1.创建代码仓库

**1.创建代码仓库：zalando-postgres-operator**

![](/images/kubernetes/advance/images-23.png)

**2.推送到GitHub**

![](/images/kubernetes/advance/images-24.png)

![](/images/kubernetes/advance/images-25.png)

![](/images/kubernetes/advance/images-26.png)



#### 1.2.postgres-operator

```shell
# 拉取镜像

registry.opensource.zalan.do/acid/postgres-operator:v1.7.1
```



##### 1.2.1.创建镜像仓库

**创建镜像仓库： zalando-postgres-operator**

![](/images/kubernetes/advance/images-27.png)

![](/images/kubernetes/advance/images-28.png)



##### 1.2.2.创建Dockerfile

**1.Dockerfile**

```shell
# 本地仓库地址
D:\dev-tct\GitHub


# 在此路径下创建Dockerfile
D:\dev-tct\GitHub\zalando-postgres-operator\postgres-operator

# Dockerfile
FROM registry.opensource.zalan.do/acid/postgres-operator:v1.7.1
MAINTAINER test
```



**2.上传仓库**

**提交到仓库后，推送**

![](/images/kubernetes/advance/images-29.png)

![](/images/kubernetes/advance/images-30.png)



##### 1.2.3.构建镜像

**1.添加构建规则**

![](/images/kubernetes/advance/images-31.png)

**2.构建镜像**

![](/images/kubernetes/advance/images-32.png)



##### 1.2.4.拉取镜像

```shell
# 拉取镜像
# docker pull registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-postgres-operator:[镜像版本号]
docker pull registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-postgres-operator:v1.7.1


# 打标签
docker tag registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-postgres-operator:v1.7.1 registry.opensource.zalan.do/acid/postgres-operator:v1.7.1


# 登录
[root@k8s-node1 ~]# docker login --username=xxxxxx registry.cn-beijing.aliyuncs.com
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded


# 拉取镜像
[root@k8s-node2 ~]# docker pull registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-postgres-operator:v1.7.1
v1.7.1: Pulling from k8s-middleware/zalando-postgres-operator
e519532ddf75: Pull complete 
7dd595794cc3: Pull complete 
06ea1f0d228f: Pull complete 
9a2998dd079a: Pull complete 
7137e6464013: Pull complete 
9687ee2d0503: Pull complete 
6a11152d7c6d: Pull complete 
Digest: sha256:84d4bc9b0d7c3d5d06dcd43e55cedd011acf4d2810daf4c7e6d02428e6c706a3
Status: Downloaded newer image for registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-postgres-operator:v1.7.1
registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-postgres-operator:v1.7.1


[root@k8s-node2 ~]# docker images
REPOSITORY                                                                  TAG                    IMAGE ID       CREATED         SIZE
registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-postgres-operator   v1.7.1                 3b16e3baa01e   5 weeks ago     70.4MB


# 打标签
[root@k8s-node2 ~]# docker tag registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-postgres-operator:v1.7.1 registry.opensource.zalan.do/acid/postgres-operator:v1.7.1


[root@k8s-node2 ~]# docker images
REPOSITORY                                                                  TAG                    IMAGE ID       CREATED         SIZE
registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-postgres-operator   v1.7.1                 3b16e3baa01e   5 weeks ago     70.4MB
registry.opensource.zalan.do/acid/postgres-operator                         v1.7.1                 3b16e3baa01e   5 weeks ago     70.4MB
```



**导出、导入镜像**

```shell
# 导出镜像
docker save : 将指定镜像保存成 tar 归档文件。

语法：
docker save [OPTIONS] IMAGE [IMAGE...]
OPTIONS 说明：
-o :输出到的文件。
例如，将镜像 nginx 生成 my_nginx.tar 文档，命令如下：
$docker save -o my_nginx.tar nginx:latest


# 导入镜像
docker load : 导入使用 docker save 命令导出的镜像。

语法：
docker load [OPTIONS]
OPTIONS 说明：
input , -i : 指定导入的文件，代替 STDIN。
quiet , -q : 精简输出信息。
例如：导入镜像：
$ docker load
$ docker load input my_nginx.tar
```

```shell
# 导出
docker save -o postgres-operator.tar registry.opensource.zalan.do/acid/postgres-operator:v1.7.1


[root@k8s-node2 ~]# docker save -o postgres-operator.tar registry.opensource.zalan.do/acid/postgres-operator:v1.7.1

[root@k8s-node2 ~]# ll
total 69312
-rw-------  1 root root 70969856 Dec  9 16:19 postgres-operator.tar
```

```shell
# 导入
docker load -i postgres-operator.tar  

[root@k8s-node1 ~]# docker load -i postgres-operator.tar
e6688e911f15: Loading layer [==================================================>]  5.852MB/5.852MB
111535f040c4: Loading layer [==================================================>]  2.048kB/2.048kB
ed23a174c545: Loading layer [==================================================>]  1.644MB/1.644MB
38ad34eef427: Loading layer [==================================================>]  44.03kB/44.03kB
f805c035c380: Loading layer [==================================================>]  63.38MB/63.38MB
126f5700352e: Loading layer [==================================================>]  4.608kB/4.608kB
221feb7bce76: Loading layer [==================================================>]  11.78kB/11.78kB
Loaded image: registry.opensource.zalan.do/acid/postgres-operator:v1.7.1


[root@k8s-node1 ~]# docker images
REPOSITORY                                                                 TAG                    IMAGE ID       CREATED         SIZE
registry.opensource.zalan.do/acid/postgres-operator                        v1.7.1                 3b16e3baa01e   5 weeks ago     70.4MB
```



#### 1.3.spilo-14

```shell
# 拉取镜像

registry.opensource.zalan.do/acid/spilo-14:2.1-p3
```



##### 1.2.1.创建镜像仓库

**创建镜像仓库： zalando-spilo-14**

![](/images/kubernetes/advance/images-27.png)

![](/images/kubernetes/advance/images-28.png)



##### 1.2.2.创建Dockerfile

**1.Dockerfile**

```shell
# 本地仓库地址
D:\dev-tct\GitHub


# 在此路径下创建Dockerfile
D:\dev-tct\GitHub\zalando-postgres-operator\spilo-14

# Dockerfile
FROM registry.opensource.zalan.do/acid/spilo-14:2.1-p3
MAINTAINER test
```



**2.上传仓库**

**提交到仓库后，推送**

![](/images/kubernetes/advance/images-33.png)



##### 1.2.3.构建镜像

**1.添加构建规则**

![](/images/kubernetes/advance/images-34.png)

**2.构建镜像**

![](/images/kubernetes/advance/images-35.png)



##### 1.2.4.拉取镜像

```shell
# 拉取镜像
# docker pull registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-spilo-14:[镜像版本号]
docker pull registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-spilo-14:2.1-p3


# 打标签
docker tag registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-postgres-operator:v1.7.1 registry.opensource.zalan.do/acid/postgres-operator:v1.7.1


# 登录
[root@k8s-node1 ~]# docker login --username=xxxxxx registry.cn-beijing.aliyuncs.com
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded


# 拉取镜像
[root@k8s-node1 ~]# docker pull registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-spilo-14:2.1-p3
2.1-p3: Pulling from k8s-middleware/zalando-spilo-14
e4ca327ec0e7: Pull complete 
4fe977156127: Pull complete 
5ae65d1957db: Pull complete 
5b405acfa5e2: Pull complete 
f32c5cef7756: Pull complete 
a35202eb0946: Pull complete 
a15269460c54: Pull complete 
2e3d6cc2d0c2: Pull complete 
8e19b9f27982: Pull complete 
0e8865a15e14: Pull complete 
84a84a38054e: Pull complete 
df6b38c9ee73: Pull complete 
5c5c8a035bd6: Pull complete 
99d087ad3f9a: Pull complete 
f8a5c080fc5a: Pull complete 
Digest: sha256:985a28e370959a50b6975570bf28c618e0d541b92508370e24ba6a09314137fc
Status: Downloaded newer image for registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-spilo-14:2.1-p3
registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-spilo-14:2.1-p3


[root@k8s-node1 ~]# docker images
REPOSITORY                                                                 TAG                    IMAGE ID       CREATED         SIZE
postgres                                                                   latest                 
registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-spilo-14           2.1-p3                 7c6d4ae25834   6 weeks ago     754MB


[root@k8s-node2 ~]# docker images
REPOSITORY                                                                  TAG                    IMAGE ID       CREATED         SIZE
registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-postgres-operator   v1.7.1                 3b16e3baa01e   5 weeks ago     70.4MB


# 打标签
[root@k8s-node1 ~]# docker tag registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-spilo-14:2.1-p3 registry.opensource.zalan.do/acid/spilo-14:2.1-p3


[root@k8s-node2 ~]# docker images
REPOSITORY                                                                  TAG                    IMAGE ID       CREATED         SIZE
registry.cn-beijing.aliyuncs.com/k8s-middleware/zalando-postgres-operator   v1.7.1                 3b16e3baa01e   5 weeks ago     70.4MB
registry.opensource.zalan.do/acid/postgres-operator                         v1.7.1                 3b16e3baa01e   5 weeks ago     70.4MB
```



