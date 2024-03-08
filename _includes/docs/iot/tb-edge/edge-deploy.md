* TOC
{:toc}



## 一、概述



![](/images/iot/tb-edge/edge-deploy/deploy-10.png)



### 1.官方文档

```shell
# ThingsBoard Edge安装
https://thingsboard.io/docs/user-guide/install/edge/installation-options/

# 在 CentOS/RHEL 服务器上安装
https://thingsboard.io/docs/user-guide/install/edge/rhel/
```



### 2.部署说明

**部署环境：**

- 操作系统：CentOS 7.8
- 软件版本：3.5.1
- 数据库：PostgreSQL 12



**安装步骤：**

1. 在 ThingsBoard 服务器上配置新的 Edge 实例
2. 安装 Java 11 (OpenJDK)
3. 安装数据库 PostgreSQL 12
4. ThingsBoard Edge 服务安装



### 3.安装准备

#### 3.1. 克隆服务器

```shell
# 克隆机器

# 修改IP地址
cd /etc/sysconfig/network-scripts
vim ifcfg-ens33
192.168.202.166

# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

# 设置主机名
hostnamectl set-hostname tb-edge
```



#### 3.2.安装 Docker

安装版本19.03.*

```shell
$ yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

$ wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo

$ yum -y install docker-ce

$ systemctl enable docker && systemctl start docker

$ docker --version
```

- **添加阿里云加速镜像**

```shell
# 添加阿里云加速镜像

cat > /etc/docker/daemon.json << EOF
{
   "registry-mirrors": ["https://gcctk8ld.mirror.aliyuncs.com"],
   "exec-opts": ["native.cgroupdriver=cgroupfs"]
} 
EOF
```

- **重启docker**

```shell
#重启docker
systemctl restart docker
```



#### 3.3.安装 Java 11

ThingsBoard 服务在 Java 11 上运行。按照以下说明安装 OpenJDK 11

```shell
yum install java-11-openjdk

java -version
```



#### 3.4.安装 PostgreSQL

```shell
1.创建目录
# mkdir -p /pg/data/psql
 
 
2.运行容器
docker run -d --network host --name pg12 --restart=always \
-e LANG="C.UTF-8" \
-e 'TZ=Asia/Shanghai' \
-e "POSTGRES_DB=postgres" \
-e "POSTGRES_USER=postgres" \
-e "POSTGRES_PASSWORD=postgres" \
-v /pg/data/psql:/var/lib/postgresql/data \
postgres:12
 

3.进入容器
# docker exec -it pg12 /bin/sh
 
切换用户
# su - postgres
$ psql 
# \l


CREATE DATABASE tb_edge;
\q
退出: \q
列出所有库 \l
列出所有用户 \du
列出库下所有表\d
```

```shell
# 访问地址

192.168.202.166
5432
postgres/postgres
```



#### 3.5.下载安装包

```shell
# tb-edge-3.5.1.rpm

wget https://github.com/thingsboard/thingsboard-edge/releases/download/v3.5.1/tb-edge-3.5.1.rpm
```





## 二、安装部署



### 1.创建 Edge 实例

在 ThingsBoard 服务器上配置 Edge

![](/images/iot/tb-edge/edge-deploy/deploy-1.png)

![](/images/iot/tb-edge/edge-deploy/deploy-2.png)

![](/images/iot/tb-edge/edge-deploy/deploy-3.png)



```shell
# ThingsBoard 服务器地址
82.157.166.86

# 边缘键
672b5ad6-cf07-c8af-e7cf-ac8a85114902

# 边缘密钥
tuhk87tqb4l1463revxf
```



### 2.创建数据库

![](/images/iot/tb-edge/edge-deploy/deploy-4.png)



### 3.Edge 服务安装

```shell
# 1.安装 ThingsBoard Edge 服务
rpm -Uvh tb-edge-3.5.1.rpm

# 2.修改 ThingsBoard Edge 配置文件
vim /etc/tb-edge/conf/tb-edge.conf

# 3.运行安装脚本
/usr/share/tb-edge/bin/install/install.sh
```



#### 3.1.安装服务

```shell
[root@tb-edge edge]# rpm -Uvh tb-edge-3.5.1.rpm
Preparing...                          ################################# [100%]
Updating / installing...
   1:tb-edge-0:3.5.1EDGE-1            ################################# [100%]
```



#### 3.2.配置 Edge

```shell
[root@tb-edge edge]# vim /etc/tb-edge/conf/tb-edge.conf


# UNCOMMENT NEXT LINES AND PUT YOUR CLOUD CONNECTION SETTINGS:
export CLOUD_ROUTING_KEY=672b5ad6-cf07-c8af-e7cf-ac8a85114902
export CLOUD_ROUTING_SECRET=tuhk87tqb4l1463revxf

# UNCOMMENT NEXT LINES IF YOU CHANGED DEFAULT CLOUD RPC HOST/PORT SETTINGS:
export CLOUD_RPC_HOST=82.157.166.86
export CLOUD_RPC_PORT=7070

# UNCOMMENT NEXT LINES IF YOU HAVE CHANGED DEFAULT POSTGRESQL DATASOURCE SETTINGS:
export SPRING_DATASOURCE_URL=jdbc:postgresql://192.168.202.166:5432/tb_edge
export SPRING_DATASOURCE_USERNAME=postgres
export SPRING_DATASOURCE_PASSWORD=postgres
```

![](/images/iot/tb-edge/edge-deploy/deploy-5.png)



#### 3.3.运行安装脚本

```shell
# /usr/share/tb-edge/bin/install/install.sh
```

![](/images/iot/tb-edge/edge-deploy/deploy-6.png)



#### 3.4.重新启动服务

```shell
systemctl restart tb-edge


systemctl start tb-edge
systemctl enable tb-edge
systemctl status tb-edge
```

![](/images/iot/tb-edge/edge-deploy/deploy-7.png)



### 4.访问 Edge

```shell
http://192.168.202.166:8080/login
tenant@thingsboard.org
tenant
```



![](/images/iot/tb-edge/edge-deploy/deploy-8.png)

![](/images/iot/tb-edge/edge-deploy/deploy-9.png)



### 5.故障排查

ThingsBoard Edge 日志存储在以下目录中：

```
/var/log/tb-edge
```

您可以发出以下命令来检查服务端是否有任何错误：

```
cat /var/log/tb-edge/tb-edge.log | grep ERROR
```



