* TOC
{:toc}



## 一、概述



Harbor，是一个英文单词，意思是港湾，港湾是干什么的呢，就是停放货物的，而货物呢，是装在集装箱中的，说到集装箱，就不得不提到Docker容器，因为docker容器的技术正是借鉴了集装箱的原理。所以，Harbor正是一个用于存储Docker镜像的企业级Registry服务。

Docker容器应用的开发和运行离不开可靠的镜像管理，虽然Docker官方也提供了公共的镜像仓库，但是从安全和效率等方面考虑，部署我们私有环境内的Registry也是非常必要的。Harbor是由VMware公司开源的企业级的Docker Registry管理项目，它包括权限管理(RBAC)、LDAP、日志审核、管理界面、自我注册、镜像复制和中文支持等功能。

```shell
# 官方下载地址：
https://goharbor.io/
https://github.com/goharbor/harbor
https://github.com/goharbor/harbor/releases


# 下载Harbor的压缩包
链接：https://pan.baidu.com/s/1W0eawaqMmq3ijx-jvrQqXQ  
提取码：acby
```





## 二、安装部署



### 1.安装 docker-compose

```shell
#下载源码
curl -L https://github.com/docker/compose/releases/download/1.25.0/docker-compose-Linux-x86_64 -o /usr/local/bin/docker-compose


#给docker-compose添加执行权限
sudo chmod +x /usr/local/bin/docker-compose

#查看docker-compose是否安装成功
docker-compose -version
```

```shell
# 方法二

去github手动下载文件：https://github.com/docker/compose/releases/tag/1.25.0-rc4

将文件上传到/usr/local/bin/ 目录下，重命名为docker-compose，修改文件权限：

chmod +x /usr/local/bin/docker-compose
```

![](/images/devops/deploy/deploy-harbor/harbor-14.png)



### 2.安装Harbor

```shell
# 1.下载Harbor的压缩包
链接：https://pan.baidu.com/s/1W0eawaqMmq3ijx-jvrQqXQ  提取码：acby


# 2.解压
/devops/harbor
 
# tar -xzf harbor-offline-installer-v2.1.0.tgz
# cd harbor
# cp harbor.yml.tmpl harbor.yml
 
 
# 3.修改配置文件
vi harbor.yml
 
hostname: 172.18.249.213 
port: 16888 
harbor_admin_password：默认admin 密码  Harbor12345  修改为：admin
 
注释https
#https:
  # https port for harbor, default is 443
  #port: 443
  # The path of cert and key files for nginx
  #certificate: /your/certificate/path
  #private_key: /your/private/key/path
 
 
# 4.安装Harbor
./prepare
# 安装，并开启 hlem charts 功能
./install.sh  --with-chartmuseum


# 5.启动Harbor
docker-compose up -d 启动
docker-compose stop 停止
docker-compose restart 重新启动
```

devops节点配置docker私有镜像源

```shell
# 6. 配置docker私有镜像源（worker节点）
#把Harbor地址加入到Docker信任列表， 注意：worker节点都需要修改配置

vi /etc/docker/daemon.json
"insecure-registries": ["http://IP:端口"]

{
   "registry-mirrors": ["https://gcctk8ld.mirror.aliyuncs.com"],
   "insecure-registries": ["http://39.96.178.134:8888"]
}


#重启Docker
systemctl daemon-reload
systemctl restart docker
```

登录

```shell
# 7.登陆harbor

# docker login -u 用户名 -p 密码 http://IP:端口
docker login -u admin -p admin http://39.96.178.134:16888
 
# 登出：
docker logout http://39.96.178.134:16888


# 外网地址：
http://39.96.178.134:16888
admin
admin
```

![](/images/devops/deploy/deploy-harbor/harbor-1.png)



### 3. harbor 作为 charts 仓库

- **用 Harbor 管理 Helm Charts，并开启 hlem charts 功能**

```shell
# 安装，并开启 hlem charts 功能
./install.sh  --with-chartmuseum

[root@dev harbor]# ./install.sh  --with-chartmuseum
......
[Step 5]: starting Harbor ...
Creating network "harbor_harbor" with the default driver
Creating network "harbor_harbor-chartmuseum" with the default driver
Creating harbor-log ... done
Creating redis         ... done
Creating registry      ... done
Creating harbor-db     ... done
Creating registryctl   ... done
Creating chartmuseum   ... done
Creating harbor-portal ... done
Creating harbor-core   ... done
Creating nginx             ... done
Creating harbor-jobservice ... done
✔ ----Harbor has been installed and started successfully.----
```



- 安装 push 插件

```shell
# master(172.51.216.81)，安装helm服务器
# 下载太慢，如果有这个文件我们也可以直接拷贝到如下目录里：
/root/.cache/helm/plugins/https-github.com-chartmuseum-helm-push


# 安装
helm plugin install https://github.com/chartmuseum/helm-push 
# 查看已成功
helm plugin list


# 安装
[root@k8s-master ~]# helm plugin install https://github.com/chartmuseum/helm-push

# 查看已成功
[root@k8s-master ~]# helm plugin list
NAME   	VERSION	DESCRIPTION                      
cm-push	0.10.0 	Push chart package to ChartMuseum
```

![](/images/devops/deploy/deploy-harbor/harbor-2.png)



### 4.控制harbor服务

启动和重启

```shell
Harbor 的日常运维管理是通过docker-compose来完成的，Harbor本身有多个服务进程，都放在docker容器之中运行，我们可以通过docker ps命令查看。 

# 暂停Harbor
docker-compose pause

# 停止Harbor
docker-compose stop
docker-compose down -v

# 开启harbor服务
docker-compose start
 
# 重启Harbor
docker-compose up -d
```

```shell
# 进入目录
[root@dev harbor]# cd /k8s/harbor/harbor
[root@dev harbor]# pwd
/k8s/harbor/harbor


# 停止Harbor
[root@dev harbor]# docker-compose stop
Stopping nginx             ... done
Stopping harbor-jobservice ... done
Stopping harbor-core       ... done
Stopping harbor-portal     ... done
Stopping registry          ... done
Stopping chartmuseum       ... done
Stopping harbor-db         ... done
Stopping registryctl       ... done
Stopping redis             ... done
Stopping harbor-log        ... done


# 开启harbor服务
[root@dev harbor]# docker-compose start
Starting log         ... done
Starting registry    ... done
Starting registryctl ... done
Starting postgresql  ... done
Starting portal      ... done
Starting redis       ... done
Starting core        ... done
Starting jobservice  ... done
Starting proxy       ... done
Starting chartmuseum ... done
```



## 三、基本操作



### 1.镜像仓库

#### 1.1.上传镜像

**1.新建项目k8s**

![](/images/devops/deploy/deploy-harbor/harbor-3.png)

![](/images/devops/deploy/deploy-harbor/harbor-4.png)

![](/images/devops/deploy/deploy-harbor/harbor-5.png)

**注意：访问级别：公开，设成公开后不用登陆也能拉取镜像，存在安全问题**



**2.登陆Harbor**

在master服务器（172.51.216.81）登陆Harbor

```shell
# docker login -u 用户名 -p 密码 http://IP:端口
# docker login -u admin -p admin http://39.96.178.134:16888

[root@k8s-master ~]# docker login -u admin -p admin http://39.96.178.134:16888
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

# 登出：
# docker logout http://39.96.178.134:16888
```



**3.从DockerHub拉取Nginx镜像**

```shell
# 拉取nginx三个不同版本

[root@k8s-master ~]# docker pull nginx:1.15
[root@k8s-master ~]# docker pull nginx:1.16
[root@k8s-master ~]# docker pull nginx:1.17


[root@k8s-master ~]# docker images
REPOSITORY                                                        TAG        IMAGE ID       CREATED         SIZE
nginx                                                             1.17       9beeba249f3e   17 months ago   127MB
nginx                                                             1.16       dfcfd8e9a5d3   17 months ago   127MB
nginx                                                             1.15       53f3fd8007f7   2 years ago     109MB
```



**4.推送镜像到私有镜像仓库**

推送镜像到私有镜像仓库

```shell
#对原镜像打tag
docker tag 原镜像名称:版本号 私有镜像仓库IP:端口/项目名称/镜像名称:版本号
 
#推送镜像
docker push 私有镜像仓库IP:端口/项目名称/镜像名称:版本号
```

```shell
#对原镜像打tag，镜像仓库地址：172.51.216.85，端口8888，项目名称：k8s

# 打标签
[root@k8s-master ~]# docker tag nginx:1.15 172.51.216.85:8888/k8s/mnginx:1.15
[root@k8s-master ~]# docker tag nginx:1.16 172.51.216.85:8888/k8s/mnginx:1.16
[root@k8s-master ~]# docker tag nginx:1.17 172.51.216.85:8888/k8s/mnginx:1.17

# 查看本地镜像
[root@k8s-master ~]# docker images
REPOSITORY                                                        TAG        IMAGE ID       CREATED         SIZE
172.51.216.85:8888/k8s/mnginx                                     1.17       9beeba249f3e   17 months ago   127MB
nginx                                                             1.17       9beeba249f3e   17 months ago   127MB
172.51.216.85:8888/k8s/mnginx                                     1.16       dfcfd8e9a5d3   17 months ago   127MB
nginx                                                             1.16       dfcfd8e9a5d3   17 months ago   127MB
172.51.216.85:8888/k8s/mnginx                                     1.15       53f3fd8007f7   2 years ago     109MB
nginx    
```

```shell
#推送镜像
docker push 私有镜像仓库IP:端口/项目名称/镜像名称:版本号

# 推送镜像
[root@k8s-master ~]# docker push 172.51.216.85:8888/k8s/mnginx:1.15
The push refers to repository [172.51.216.85:8888/k8s/mnginx]
332fa54c5886: Pushed 
6ba094226eea: Pushed 
6270adb5794c: Pushed 
1.15: digest: sha256:e770165fef9e36b990882a4083d8ccf5e29e469a8609bb6b2e3b47d9510e2c8d size: 948

[root@k8s-master ~]# docker push 172.51.216.85:8888/k8s/mnginx:1.16
The push refers to repository [172.51.216.85:8888/k8s/mnginx]
c23548ea0b99: Pushed 
82068c842707: Pushed 
c2adabaecedb: Pushed 
1.16: digest: sha256:2963fc49cc50883ba9af25f977a9997ff9af06b45c12d968b7985dc1e9254e4b size: 948

[root@k8s-master ~]# docker push 172.51.216.85:8888/k8s/mnginx:1.17
The push refers to repository [172.51.216.85:8888/k8s/mnginx]
6c7de695ede3: Pushed 
2f4accd375d9: Pushed 
ffc9b21953f4: Pushed 
1.17: digest: sha256:8269a7352a7dad1f8b3dc83284f195bac72027dd50279422d363d49311ab7d9b size: 948
```



![](/images/devops/deploy/deploy-harbor/harbor-6.png)

![](/images/devops/deploy/deploy-harbor/harbor-7.png)



#### 1.2.拉取镜像

```shell
# 删除镜像
[root@k8s-master ~]# docker rmi 172.51.216.85:8888/k8s/mnginx:1.15
Untagged: 172.51.216.85:8888/k8s/mnginx:1.15
Untagged: 172.51.216.85:8888/k8s/mnginx@sha256:e770165fef9e36b990882a4083d8ccf5e29e469a8609bb6b2e3b47d9510e2c8d


# 拉取镜像
[root@k8s-master ~]# docker pull 172.51.216.85:8888/k8s/mnginx:1.15
1.15: Pulling from k8s/mnginx
Digest: sha256:e770165fef9e36b990882a4083d8ccf5e29e469a8609bb6b2e3b47d9510e2c8d
Status: Downloaded newer image for 172.51.216.85:8888/k8s/mnginx:1.15
172.51.216.85:8888/k8s/mnginx:1.15
```



#### 1.3.登陆登出

```shell
# docker login -u 用户名 -p 密码 http://IP:端口
docker login -u admin -p admin http://172.51.216.85:8888
# 登出：
docker logout http://172.51.216.85:8888

--------------------------------------
# 登出
[root@k8s-master ~]# docker logout http://172.51.216.85:8888
Removing login credentials for 172.51.216.85:8888

[root@k8s-master ~]# docker pull 172.51.216.85:8888/k8s/mnginx:1.15
Error response from daemon: unauthorized: unauthorized to access repository: k8s/mnginx, action: pull: unauthorized to access repository: k8s/mnginx, action: pull


# 登陆
# docker login http://172.51.216.85:8888
[root@k8s-master ~]# docker login http://172.51.216.85:8888
Username: admin
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

[root@k8s-master ~]# docker pull 172.51.216.85:8888/k8s/mnginx:1.15
1.15: Pulling from k8s/mnginx
Digest: sha256:e770165fef9e36b990882a4083d8ccf5e29e469a8609bb6b2e3b47d9510e2c8d
Status: Image is up to date for 172.51.216.85:8888/k8s/mnginx:1.15
172.51.216.85:8888/k8s/mnginx:1.15
```



