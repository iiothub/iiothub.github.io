* TOC
{:toc}



## 一、概述



![](/images/iot/tb-edge/edge-deploy-docker/deploy-10.png)



### 1.官方文档

```shell
# ThingsBoard Edge安装
https://thingsboard.io/docs/user-guide/install/edge/installation-options/

# 使用 Docker 安装（Linux 或 Mac OS）
https://thingsboard.io/docs/user-guide/install/edge/docker/
```



### 2.部署说明

**部署环境：**

- 操作系统：CentOS 7.8
- 软件版本：3.5.1
- 数据库：PostgreSQL 12



**部署方式：**

- Docker  Compose 方式部署
- Docker 直接部署



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



#### 3.3.安装 docker-compose

**安装 docker-compose**

```shell
#下载源码
curl -L https://github.com/docker/compose/releases/download/1.25.0/docker-compose-Linux-x86_64 -o /usr/local/bin/docker-compose


#给docker-compose添加执行权限
sudo chmod +x /usr/local/bin/docker-compose

#查看docker-compose是否安装成功
docker-compose version
```



**docker-compose 基本操作**

```shell
# 安装并启动EdgeX
sudo docker-compose up -d     # -d 后台运行容器
 
# 查看所有容器运行状况
sudo docker-compose ps
 
# 显示容器日志
docker-compose logs -f [compose-contatainer-name]
 
# 停止容器
sudo docker-compose stop
 
# 启动容器
sudo docker-compose start
 
# 停止和删除所有容器
sudo docker-compose down


# 常用命令：
启动：docker-compose up -d 注意这里需要在yml配置文件路径执行，其他路径执行需要-f指定配置文件地址。
查看日志：docker-compose logs -f ${compose-contatainer-name}
停止：docker-compose stop
停止并删除容器：docker-compose down
其他命令帮助：docker-compose --help
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



#### 3.5.创建 Edge 实例

在 ThingsBoard 服务端上配置 Edge

![](/images/iot/tb-edge/edge-deploy-docker/deploy-1.png)

![](/images/iot/tb-edge/edge-deploy-docker/deploy-2.png)

![](/images/iot/tb-edge/edge-deploy-docker/deploy-3.png)



```shell
# ThingsBoard 服务器地址
82.157.166.86

# 边缘键
672b5ad6-cf07-c8af-e7cf-ac8a85114902

# 边缘密钥
tuhk87tqb4l1463revxf
```





## 二、Docker  Compose 方式部署



### 1.创建 docker-compose.yml

**1.创建数据和日志文件夹**

```shell
mkdir -p ~/.mytb-edge-data && sudo chown -R 799:799 ~/.mytb-edge-data
mkdir -p ~/.mytb-edge-logs && sudo chown -R 799:799 ~/.mytb-edge-logs
```



**2.docker-compose.yml**

```shell
version: '3.0'
services:
  mytbedge:
    restart: always
    image: "thingsboard/tb-edge:3.5.1EDGE"
    ports:
      - "8080:8080"
      - "1883:1883"
      - "5683-5688:5683-5688/udp"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/tb-edge
      CLOUD_ROUTING_KEY: 672b5ad6-cf07-c8af-e7cf-ac8a85114902
      CLOUD_ROUTING_SECRET: tuhk87tqb4l1463revxf
      CLOUD_RPC_HOST: 82.157.166.86
      CLOUD_RPC_PORT: 7070
      CLOUD_RPC_SSL_ENABLED: false
    volumes:
      - ~/.mytb-edge-data:/data
      - ~/.mytb-edge-logs:/var/log/tb-edge
  postgres:
    restart: always
    image: "postgres:12"
    ports:
      - "5432"
    environment:
      POSTGRES_DB: tb-edge
      POSTGRES_PASSWORD: postgres
    volumes:
      - ~/.mytb-edge-data/db:/var/lib/postgresql/data
```



### 2.运行容器

```shell
docker compose up -d
docker compose logs -f mytbedge
```

![](/images/iot/tb-edge/edge-deploy-docker/deploy-4.png)

![](/images/iot/tb-edge/edge-deploy-docker/deploy-5.png)



### 3.访问 Edge

```shell
http://192.168.202.166:8080/login
tenant@thingsboard.org
tenant
```



![](/images/iot/tb-edge/edge-deploy-docker/deploy-6.png)

![](/images/iot/tb-edge/edge-deploy-docker/deploy-7.png)





## 三、Docker 直接部署



### 1.创建数据库

![](/images/iot/tb-edge/edge-deploy-docker/deploy-8.png)



### 2.运行容器

**1.创建数据和日志文件夹**

```shell
mkdir -p ~/.mytb-edge-data && sudo chown -R 799:799 ~/.mytb-edge-data
mkdir -p ~/.mytb-edge-logs && sudo chown -R 799:799 ~/.mytb-edge-logs
```



**2.运行容器**

```shell
docker run -d --network host --name tb-edge --restart=always \
-e "SPRING_DATASOURCE_URL=jdbc:postgresql://192.168.202.166:5432/tb-edge" \
-e "SPRING_DATASOURCE_USERNAME=postgres" \
-e "SPRING_DATASOURCE_PASSWORD=postgres" \
-e "CLOUD_ROUTING_KEY=672b5ad6-cf07-c8af-e7cf-ac8a85114902" \
-e "CLOUD_ROUTING_SECRET=tuhk87tqb4l1463revxf" \
-e "CLOUD_RPC_HOST=82.157.166.86" \
-e "CLOUD_RPC_PORT=7070" \
-e "CLOUD_RPC_SSL_ENABLED=false" \
-v ~/.mytb-edge-data:/data \
-v ~/.mytb-edge-logs:/var/log/tb-edge \
thingsboard/tb-edge:3.5.1EDGE
```



### 3.访问 Edge

```shell
http://192.168.202.166:8080/login
tenant@thingsboard.org
tenant
```



