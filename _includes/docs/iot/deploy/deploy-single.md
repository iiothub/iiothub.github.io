* TOC
{:toc}



## 一、环境部署方案



### 1.部署方式

- **操作系统：CentOS 7.8**
- **数据库：TimescaleDB （推荐）或PostgreSQL**
- **消息队列：RabbitMQ**
- **内存数据库：Redis**



```shell
# 安装文档

# CentOS安装
https://thingsboard.io/docs/user-guide/install/rhel/
http://www.ithingsboard.com/docs/user-guide/install/rhel/?rhelThingsboardDatabase=timescale
```



### 2.部署方案

- **方案一（推荐）：TimescaleDB + RabbitMQ + Redis**
- **方案二：PostgreSQL + RabbitMQ + Redis**
- **方案三（轻量）：TimescaleDB（推荐）或PostgreSQL** 



### 3.环境准备

#### 3.1. Docker

- **安装版本19.03.***

```shell
$ wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
$ yum -y install docker-ce-19.03.15-3.el7
$ systemctl enable docker && systemctl start docker
$ docker --version
```



- **安装Docker（所有节点）**

安装版本19.03.*

```shell
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
   "registry-mirrors": ["https://*******.aliyuncs.com"],
   "exec-opts": ["native.cgroupdriver=systemd"]
} 
EOF
```

- **重启docker**

```shell
#重启docker

systemctl restart docker
```



#### 3.2.PostgreSQL

```shell
1.创建目录
# mkdir -p /ntms/pg/data/psql
 
 
2.运行容器
docker run -d --network host --name pg12 --restart=always \
-e LANG="C.UTF-8" \
-e 'TZ=Asia/Shanghai' \
-e "POSTGRES_DB=postgres" \
-e "POSTGRES_USER=postgres" \
-e "POSTGRES_PASSWORD=postgres" \
-v /ntms/pg/data/psql:/var/lib/postgresql/data \
postgres:12
  
 
3.进入容器
# docker exec -it pg12 /bin/sh
 
# 切换用户
# su - postgres
$ psql 
# \l
```

```shell
# 访问地址

192.168.202.188
5432
postgres/postgres
```



#### 3.3.TimescaleDB

```shell
1.创建目录
# mkdir -p /k8s/pg/data/psql
 
 
2.运行容器
docker run -d --name timescaledb --network host --restart=always \
-e LANG="C.UTF-8" \
-e 'TZ=Asia/Shanghai' \
-e "POSTGRES_DB=postgres" \
-e "POSTGRES_USER=postgres" \
-e "POSTGRES_PASSWORD=postgres" \
-v /k8s/pg/data/psql:/var/lib/postgresql/data \
timescale/timescaledb:2.1.0-pg12
 
 
3.进入容器
# docker exec -it timescaledb /bin/sh
 
 
# 切换用户
# su - postgres
$ psql 
# \l
 
 
# 查看版本信息
postgres=# select version();
 
postgres=# show server_version;
 
postgres=# SHOW server_version_num;
 
postgres=# SELECT current_setting('server_version_num');
 
# 客户端版本
7a7a34fb363c:~$ psql --version
psql (PostgreSQL) 11.11
```

```shell
# 访问地址

192.168.202.189
5432
postgres/postgres
```



#### 3.4.RabbitMQ

```shell
# 运行容器
docker run -d  --network host --restart=always --name rabbitmq rabbitmq:management
```

```shell
# 访问地址
http://192.168.202.188:15672/ 
http://192.168.202.189:15672/ 

# 账户/密码：
guest/guest 
```



#### 3.5.Redis

```shell
***************************************************************
# 安装环境准备
 

创建目录
mkdir -p /k8s/redis/master/log
mkdir -p /k8s/redis/master/data
对log目录进行读写授权
chmod 777 /k8s/redis/master/log


***************************************************************
# 修改配置文件


# 获取配置文件
 
1.获取redis配置文件
redis官方提供了一个配置文件样例，通过wget工具下载下来。我用的root用户，就直接下载到/root目录里了
 
wget http://download.redis.io/redis-stable/redis.conf


注意：下载6.0的安装包获取配置文件


#配置文件路径
/k8s/redis/master/redis.conf
 
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
 
# 运行容器
docker run -d --network host --restart=always \
--name redis \
--privileged=true \
-v /k8s/redis/master/redis.conf:/usr/local/etc/redis/redis.conf \
-v /k8s/redis/master/data:/data \
-v /k8s/redis/master/log:/var/log/redis \
redis:6.0 \
redis-server /usr/local/etc/redis/redis.conf
```

```shell
# 访问地址

redis
192.168.202.188
192.168.202.189
6379
```



<img src="/images/iot/deploy/deploy-single/single-1.png" style="zoom: 50%;" />



#### 3.6.关闭防火墙

```shell
1.启动防火墙
systemctl start firewalld

2.关闭防火墙
systemctl stop firewalld

3.查看状态
systemctl status firewalld

4.开机启用防火墙
systemctl enable firewalld

5.开机禁用防火墙
systemctl disable firewalld
```





## 二、单机环境部署（推荐）



**方案一（推荐）：TimescaleDB + RabbitMQ + Redis**

**注意：要保证TimescaleDB + RabbitMQ + Redis先启动，建议中间件跟tb分机器安装，后启动tb**



### 1.安装工具

```shell
# Install wget
sudo yum install -y nano wget

# Add latest EPEL release for CentOS 7
sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
```



### 2.安装Java 11(OpenJDK)

```shell
# ThingsBoard服务运行在Java 11请按照以下说明安装OpenJDK 11：
sudo yum install java-11-openjdk

# 可以使用以下命令检查安装：
java -version

# 命令输出结果：
[root@192 thingsboard]# java -version
openjdk version "11.0.19" 2023-04-18 LTS
OpenJDK Runtime Environment (Red_Hat-11.0.19.0.7-1.el7_9) (build 11.0.19+7-LTS)
OpenJDK 64-Bit Server VM (Red_Hat-11.0.19.0.7-1.el7_9) (build 11.0.19+7-LTS, mixed mode, sharing)
```



### 3.安装服务

```shell
# 下载安装包。
wget https://github.com/thingsboard/thingsboard/releases/download/v3.5.1/thingsboard-3.5.1.rpm

# 安装服务
sudo rpm -Uvh thingsboard-3.5.1.rpm
```

```shell
[root@192 ~]# cd /thingsboard/
[root@192 thingsboard]# ll
total 188944
-rw-r--r--. 1 root root 193478652 Aug  9 01:21 thingsboard-3.5.1.rpm


[root@192 thingsboard]# rpm -Uvh thingsboard-3.5.1.rpm
Preparing...                          ################################# [100%]
Updating / installing...
   1:thingsboard-0:3.5.1-1            ################################# [100%]
```



### 4.配置数据库

#### 4.1.TimescaleDB（推荐）

![](/images/iot/deploy/deploy-single/single-2.png)

TimescaleDB配置

编辑配置文件

```
sudo nano /etc/thingsboard/conf/thingsboard.conf
```

将下面内容添加到配置文件中并**替换**“PUT_YOUR_POSTGRESQL_PASSWORD_HERE”为**postgres帐户密码**：

```shell
# TimescaleDB 
export DATABASE_TS_TYPE=timescale
export SPRING_DATASOURCE_URL=jdbc:postgresql://192.168.202.189:5432/thingsboard
export SPRING_DATASOURCE_USERNAME=postgres
export SPRING_DATASOURCE_PASSWORD=postgres
# Specify Interval size for data chunks storage. Please note that this value can be set only once.
export SQL_TIMESCALE_CHUNK_TIME_INTERVAL=604800000 # Number of miliseconds. The current value corresponds to one week.
```



#### 4.2.PostgreSQL

![](/images/iot/deploy/deploy-single/single-3.png)

PostgreSQL配置

编辑配置文件

```
sudo nano /etc/thingsboard/conf/thingsboard.conf
```

将下面内容添加到配置文件中并**替换**“PUT_YOUR_POSTGRESQL_PASSWORD_HERE”为**postgres帐户密码**：

```shell
# PostgreSQL
export DATABASE_TS_TYPE=sql
export SPRING_DATASOURCE_URL=jdbc:postgresql://192.168.202.188:5432/thingsboard
export SPRING_DATASOURCE_USERNAME=postgres
export SPRING_DATASOURCE_PASSWORD=postgres
# Specify partitioning size for timestamp key-value storage. Allowed values: DAYS, MONTHS, YEARS, INDEFINITE.
export SQL_POSTGRES_TS_KV_PARTITIONING=MONTHS
```



#### 4.3.创建数据库

**注意：需要手动创建数据库 thingsboard，否则启动会失败。**

![](/images/iot/deploy/deploy-single/single-4.png)



### 5.配置RabbitMQ

ThingsBoard配置

编辑配置文件

```shell
sudo nano /etc/thingsboard/conf/thingsboard.conf
```

将以下行添加到配置文件将“YOUR_USERNAME”和“YOUR_PASSWORD”替换为**真实的信息**将“localhost”和“5672”替换为真实的**RabbitMQ主机和端口**：

```shell
export TB_QUEUE_TYPE=rabbitmq
export TB_QUEUE_RABBIT_MQ_USERNAME=guest
export TB_QUEUE_RABBIT_MQ_PASSWORD=guest
export TB_QUEUE_RABBIT_MQ_HOST=192.168.202.189
export TB_QUEUE_RABBIT_MQ_PORT=5672
```



### 6.配置Redis

**Redis需要修改的配置**

```shell
cache:
  # caffeine or redis
  type: "${CACHE_TYPE:caffeine}"
  
redis:
  # standalone or cluster
  connection:
    type: "${REDIS_CONNECTION_TYPE:standalone}"
  standalone:
    host: "${REDIS_HOST:localhost}"
    port: "${REDIS_PORT:6379}"  
```



ThingsBoard配置

编辑配置文件

```shell
sudo nano /etc/thingsboard/conf/thingsboard.conf
```

```shell
#Redis
export CACHE_TYPE=redis
export REDIS_CONNECTION_TYPE=standalone
export REDIS_HOST=192.168.202.189
export REDIS_PORT=6379
```



### 7. [可选]低性能配置(1GB内存)

编辑配置文件

```
sudo nano /etc/thingsboard/conf/thingsboard.conf
```

将以下行添加到配置文件

```shell
# Update ThingsBoard memory usage and restrict it to 256MB in/etc/thingsboard/conf/thingsboard.conf

export JAVA_OPTS="$JAVA_OPTS -Xms256M -Xmx256M"
```



### 8. 运行安装脚本

执行以下脚本安装ThingsBoard服务并初始化演示数据：

```
# --loadDemo option will load demo data: users, devices, assets, rules, widgets.

sudo /usr/share/thingsboard/bin/install/install.sh --loadDemo
```



**注意：需要手动创建数据库**

```shell
[root@192 thingsboard]# /usr/share/thingsboard/bin/install/install.sh --loadDemo
  ______    __      _                              ____                               __
 /_  __/   / /_    (_)   ____    ____ _   _____   / __ )  ____   ____ _   _____  ____/ /
  / /     / __ \  / /   / __ \  / __ `/  / ___/  / __  | / __ \ / __ `/  / ___/ / __  /
 / /     / / / / / /   / / / / / /_/ /  (__  )  / /_/ / / /_/ // /_/ /  / /    / /_/ /
/_/     /_/ /_/ /_/   /_/ /_/  \__, /  /____/  /_____/  \____/ \__,_/  /_/     \__,_/
                              /____/

 ===================================================
 :: ThingsBoard ::       (v3.5.1)
 ===================================================

Starting ThingsBoard Installation...
Installing DataBase schema for entities...
Installing SQL DataBase schema part: schema-entities.sql
Installing SQL DataBase schema indexes part: schema-entities-idx.sql
Installing SQL DataBase schema PostgreSQL specific indexes part: schema-entities-idx-psql-addon.sql
Installing SQL DataBase schema views and functions: schema-views-and-functions.sql
Successfully executed query: DROP VIEW IF EXISTS device_info_view CASCADE;
Successfully executed query: CREATE OR REPLACE VIEW device_info_view AS SELECT * FROM device_info_active_attribute_view;
Installing DataBase schema for timeseries...
Installing SQL DataBase schema part: schema-ts-psql.sql
Successfully executed query: CREATE TABLE IF NOT EXISTS ts_kv_indefinite PARTITION OF ts_kv DEFAULT;
Loading system data...
Creating default notification configs for system admin
Creating default notification configs for all tenants
Loading demo data...
Installation finished successfully!
ThingsBoard installed successfully!
```



![](/images/iot/deploy/deploy-single/single-5.png)



### 9.启动ThingsBoard

执行以下命令以启动ThingsBoard：

```shell
service thingsboard start

systemctl start thingsboard
systemctl enable thingsboard
systemctl status thingsboard
```

启动后使用以下链接打开Web UI：

```shell
http://192.168.202.189:8080/
```



```shell
[root@192 thingsboard]# systemctl start thingsboard
[root@192 thingsboard]# systemctl status thingsboard
● thingsboard.service - thingsboard
   Loaded: loaded (/usr/lib/systemd/system/thingsboard.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2023-08-09 01:58:38 CST; 9s ago
 Main PID: 14924 (thingsboard.jar)
   CGroup: /system.slice/thingsboard.service
           ├─14924 /bin/bash /usr/share/thingsboard/bin/thingsboard.jar
           └─14940 /usr/bin/java -Dsun.misc.URLClassPath.disableJarChecking=true -Dplatform=rpm -Dinstall.data_dir=/usr/share/thingsboard/data...

Aug 09 01:58:38 192.168.202.188 systemd[1]: Started thingsboard.
Aug 09 01:58:39 192.168.202.188 thingsboard.jar[14924]: ______    __      _                              ____                               __
Aug 09 01:58:39 192.168.202.188 thingsboard.jar[14924]: /_  __/   / /_    (_)   ____    ____ _   _____   / __ )  ____   ____ _   _____  ____/ /
Aug 09 01:58:39 192.168.202.188 thingsboard.jar[14924]: / /     / __ \  / /   / __ \  / __ `/  / ___/  / __  | / __ \ / __ `/  / ___/ / __  /
Aug 09 01:58:39 192.168.202.188 thingsboard.jar[14924]: / /     / / / / / /   / / / / / /_/ /  (__  )  / /_/ / / /_/ // /_/ /  / /    / /_/ /
Aug 09 01:58:39 192.168.202.188 thingsboard.jar[14924]: /_/     /_/ /_/ /_/   /_/ /_/  \__, /  /____/  /_____/  \____/ \__,_/  /_/     \__,_/
Aug 09 01:58:39 192.168.202.188 thingsboard.jar[14924]: /____/
Aug 09 01:58:39 192.168.202.188 thingsboard.jar[14924]: ===================================================
Aug 09 01:58:39 192.168.202.188 thingsboard.jar[14924]: :: ThingsBoard ::       (v3.5.1)
Aug 09 01:58:39 192.168.202.188 thingsboard.jar[14924]: ===================================================
[root@192 thingsboard]# systemctl enable firewalld
Created symlink from /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service to /usr/lib/systemd/system/firewalld.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/firewalld.service to /usr/lib/systemd/system/firewalld.service.
```



### 10.访问ThingsBoard

```shell
# 如果在安装脚本的执行过程中指定了-loadDemo则可以使用以下默认帐号：

System Administrator: sysadmin@thingsboard.org / sysadmin
Tenant Administrator: tenant@thingsboard.org / tenant
Customer User: customer@thingsboard.org / customer
```



```shell
http://192.168.202.189:8080/login
tenant@thingsboard.org
tenant
```



<img src="/images/iot/deploy/deploy-single/single-6.png" style="zoom:80%;" />

<img src="/images/iot/deploy/deploy-single/single-7.png" style="zoom:67%;" />



### 11.安装说明

#### 11.1.PostgreSQL

```shell
192.168.202.189
5432
postgres
postgres
```

<img src="/images/iot/deploy/deploy-single/single-8.png" style="zoom: 50%;" />

![](/images/iot/deploy/deploy-single/single-9.png)





#### 11.2.RabbitMQ

```shell
http://192.168.202.189:15672/#/
guest
guest
```



![](/images/iot/deploy/deploy-single/single-10.png)

![](/images/iot/deploy/deploy-single/single-11.png)



#### 11.3.Redis

```shell
192.168.202.189
6379
```



![](/images/iot/deploy/deploy-single/single-12.png)



#### 11.4.安装目录

```shell
# 安装目录
/usr/share/thingsboard


[root@192 thingsboard]# pwd
/usr/share/thingsboard


[root@192 thingsboard]# ll
total 0
drwxr-xr-x. 3 thingsboard thingsboard  87 Aug  9 01:59 bin
drwxr-xr-x. 4 thingsboard thingsboard 119 Aug  9 02:43 conf
drwxr-xr-x. 8 thingsboard thingsboard  96 Aug  9 01:21 data


# 执行文件路径
/usr/share/thingsboard/bin
[root@192 bin]# ll
total 205116
drwxr-xr-x. 2 thingsboard thingsboard        84 Aug  9 02:32 install
-r-x------. 1 thingsboard thingsboard 210037773 May 31 18:24 thingsboard.jar
lrwxrwxrwx. 1 thingsboard thingsboard        43 Aug  9 01:21 thingsboard.yml -> /usr/share/thingsboard/conf/thingsboard.yml


# 配置文件路径
[root@192 conf]# pwd
/usr/share/thingsboard/conf
[root@192 conf]# ll
total 84
-rwxr-xr--. 1 thingsboard thingsboard   649 May 31 18:24 banner.txt
drwxr-xr-x. 2 thingsboard thingsboard    33 Aug  9 01:21 i18n
-rwxr-xr--. 1 thingsboard thingsboard  1799 May 31 18:24 logback.xml
drwxr-xr-x. 2 thingsboard thingsboard   265 Aug  9 01:21 templates
-rwxr-xr--. 1 thingsboard thingsboard  1966 Aug  9 01:44 thingsboard.conf
-rwxr-xr--. 1 thingsboard thingsboard 73110 May 31 18:24 thingsboard.yml
```



![](/images/iot/deploy/deploy-single/single-13.png)

![](/images/iot/deploy/deploy-single/single-14.png)



#### 11.5.配置文件

```shell
# 配置文件
thingsboard.conf
thingsboard.yml


[root@192 conf]# pwd
/usr/share/thingsboard/conf
[root@192 conf]# ll
total 84
-rwxr-xr--. 1 thingsboard thingsboard   649 May 31 18:24 banner.txt
drwxr-xr-x. 2 thingsboard thingsboard    33 Aug  9 01:21 i18n
-rwxr-xr--. 1 thingsboard thingsboard  1799 May 31 18:24 logback.xml
drwxr-xr-x. 2 thingsboard thingsboard   265 Aug  9 01:21 templates
-rwxr-xr--. 1 thingsboard thingsboard  1966 Aug  9 01:44 thingsboard.conf
-rwxr-xr--. 1 thingsboard thingsboard 73110 May 31 18:24 thingsboard.yml
```



##### 11.5.1.thingsboard.conf

**TimescaleDB**

```shell
[root@thingsboard ~]# vim /usr/share/thingsboard/conf/thingsboard.conf 

#
# Copyright © 2016-2023 The Thingsboard Authors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

export JAVA_OPTS="$JAVA_OPTS -Dplatform=rpm -Dinstall.data_dir=/usr/share/thingsboard/data"
export JAVA_OPTS="$JAVA_OPTS -XX:+IgnoreUnrecognizedVMOptions -XX:+HeapDumpOnOutOfMemoryError"
export JAVA_OPTS="$JAVA_OPTS -XX:-UseBiasedLocking -XX:+UseTLAB -XX:+ResizeTLAB -XX:+PerfDisableSharedMem -XX:+UseCondCardMark"
export LOG_FILENAME=thingsboard.out
export LOADER_PATH=/usr/share/thingsboard/conf,/usr/share/thingsboard/extensions
export SQL_DATA_FOLDER=/usr/share/thingsboard/data/sql


# TimescaleDB 
export DATABASE_TS_TYPE=timescale
export SPRING_DATASOURCE_URL=jdbc:postgresql://192.168.202.189:5432/thingsboard
export SPRING_DATASOURCE_USERNAME=postgres
export SPRING_DATASOURCE_PASSWORD=postgres
# Specify Interval size for data chunks storage. Please note that this value can be set only once.
export SQL_TIMESCALE_CHUNK_TIME_INTERVAL=604800000 # Number of miliseconds. The current value corresponds to one week.


# RabbitMQ
export TB_QUEUE_TYPE=rabbitmq
export TB_QUEUE_RABBIT_MQ_USERNAME=guest
export TB_QUEUE_RABBIT_MQ_PASSWORD=guest
export TB_QUEUE_RABBIT_MQ_HOST=192.168.202.189
export TB_QUEUE_RABBIT_MQ_PORT=5672


# Redis
export CACHE_TYPE=redis
export REDIS_CONNECTION_TYPE=standalone
export REDIS_HOST=192.168.202.189
export REDIS_PORT=6379
```



**PostgreSQL**

```shell
[root@thingsboard ~]# vim /usr/share/thingsboard/conf/thingsboard.conf

#
# Copyright © 2016-2023 The Thingsboard Authors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

export JAVA_OPTS="$JAVA_OPTS -Dplatform=rpm -Dinstall.data_dir=/usr/share/thingsboard/data"
export JAVA_OPTS="$JAVA_OPTS -XX:+IgnoreUnrecognizedVMOptions -XX:+HeapDumpOnOutOfMemoryError"
export LOG_FILENAME=thingsboard.out
export LOADER_PATH=/usr/share/thingsboard/conf,/usr/share/thingsboard/extensions
export SQL_DATA_FOLDER=/usr/share/thingsboard/data/sql


# PostgreSQL 
export DATABASE_TS_TYPE=sql
export SPRING_DATASOURCE_URL=jdbc:postgresql://192.168.202.188:5432/thingsboard
export SPRING_DATASOURCE_USERNAME=postgres
export SPRING_DATASOURCE_PASSWORD=postgres
export SQL_POSTGRES_TS_KV_PARTITIONING=MONTHS


# RabbitMQ
export TB_QUEUE_TYPE=rabbitmq
export TB_QUEUE_RABBIT_MQ_USERNAME=guest
export TB_QUEUE_RABBIT_MQ_PASSWORD=guest
export TB_QUEUE_RABBIT_MQ_HOST=192.168.202.188
export TB_QUEUE_RABBIT_MQ_PORT=5672


# Redis
export CACHE_TYPE=redis
export REDIS_CONNECTION_TYPE=standalone
export REDIS_HOST=192.168.202.188
export REDIS_PORT=6379
```



##### 11.5.2.thingsboard.yml

```shell
[root@192 conf]# vim thingsboard.yml 

#
# Copyright © 2016-2023 The Thingsboard Authors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

server:
  # Server bind address
  address: "${HTTP_BIND_ADDRESS:0.0.0.0}"
  # Server bind port
  port: "${HTTP_BIND_PORT:8080}"
  # Server forward headers strategy
  forward_headers_strategy: "${HTTP_FORWARD_HEADERS_STRATEGY:NONE}"
  # Server SSL configuration
  ssl:
    # Enable/disable SSL support
    enabled: "${SSL_ENABLED:false}"
    # Server SSL credentials
    credentials:
      # Server credentials type (PEM - pem certificate file; KEYSTORE - java keystore)
      type: "${SSL_CREDENTIALS_TYPE:PEM}"
      # PEM server credentials
      pem:
        # Path to the server certificate file (holds server certificate or certificate chain, may include server private key)
        cert_file: "${SSL_PEM_CERT:server.pem}"
        key_file: "${SSL_PEM_KEY:server_key.pem}"
        # Server certificate private key password (optional)
        key_password: "${SSL_PEM_KEY_PASSWORD:server_key_password}"
      # Keystore server credentials
      keystore:
        # Type of the key store (JKS or PKCS12)
        type: "${SSL_KEY_STORE_TYPE:PKCS12}"
        # Path to the key store that holds the SSL certificate
        store_file: "${SSL_KEY_STORE:classpath:keystore/keystore.p12}"
        # Password used to access the key store
        store_password: "${SSL_KEY_STORE_PASSWORD:thingsboard}"
        # Key alias
        key_alias: "${SSL_KEY_ALIAS:tomcat}"
        # Password used to access the key
        key_password: "${SSL_KEY_PASSWORD:thingsboard}"
  # HTTP/2 support (takes effect only if server SSL is enabled)
  http2:
    # Enable/disable HTTP/2 support
    enabled: "${HTTP2_ENABLED:true}"
  log_controller_error_stack_trace: "${HTTP_LOG_CONTROLLER_ERROR_STACK_TRACE:false}"
  ws:
    send_timeout: "${TB_SERVER_WS_SEND_TIMEOUT:5000}"
    # recommended timeout >= 30 seconds. Platform will attempt to send 'ping' request 3 times within the timeout
    ping_timeout: "${TB_SERVER_WS_PING_TIMEOUT:30000}"
    dynamic_page_link:
      refresh_interval: "${TB_SERVER_WS_DYNAMIC_PAGE_LINK_REFRESH_INTERVAL_SEC:60}"
      refresh_pool_size: "${TB_SERVER_WS_DYNAMIC_PAGE_LINK_REFRESH_POOL_SIZE:1}"
      max_alarm_queries_per_refresh_interval: "${TB_SERVER_WS_MAX_ALARM_QUERIES_PER_REFRESH_INTERVAL:10}"
      max_per_user: "${TB_SERVER_WS_DYNAMIC_PAGE_LINK_MAX_PER_USER:10}"
    max_entities_per_data_subscription: "${TB_SERVER_WS_MAX_ENTITIES_PER_DATA_SUBSCRIPTION:10000}"
    max_entities_per_alarm_subscription: "${TB_SERVER_WS_MAX_ENTITIES_PER_ALARM_SUBSCRIPTION:10000}"
    max_queue_messages_per_session: "${TB_SERVER_WS_DEFAULT_QUEUE_MESSAGES_PER_SESSION:1000}"
  rest:
    server_side_rpc:
```



### 12.故障排查

**ThingsBoard日志存储在以下目录中：**

```shell
/var/log/thingsboard
```

**执行如下命令检查后面是否有错误：**

```shell
cat /var/log/thingsboard/thingsboard.log | grep ERROR
```



### 13.系统时间设置

```shell
# CentOS7 时间快8小时问题处理
https://www.cnblogs.com/java365/articles/17286540.html

#centos7 时间同步
https://blog.csdn.net/yangziqi098/article/details/129542096
```





## 三、单机环境部署（轻量）



**方案三（轻量）：TimescaleDB（推荐）或PostgreSQL** 



### 1.配置数据库

#### 4.1.TimescaleDB（推荐）

![](/images/iot/deploy/deploy-single/single-2.png)

TimescaleDB配置

编辑配置文件

```shell
sudo nano /etc/thingsboard/conf/thingsboard.conf
```

将下面内容添加到配置文件中并**替换**“PUT_YOUR_POSTGRESQL_PASSWORD_HERE”为**postgres帐户密码**：

```shell
# TimescaleDB 
export DATABASE_TS_TYPE=timescale
export SPRING_DATASOURCE_URL=jdbc:postgresql://192.168.202.189:5432/thingsboard
export SPRING_DATASOURCE_USERNAME=postgres
export SPRING_DATASOURCE_PASSWORD=postgres
# Specify Interval size for data chunks storage. Please note that this value can be set only once.
export SQL_TIMESCALE_CHUNK_TIME_INTERVAL=604800000 # Number of miliseconds. The current value corresponds to one week.
```



#### 4.2.PostgreSQL

![](/images/iot/deploy/deploy-single/single-3.png)

PostgreSQL配置

编辑配置文件

```shell
sudo nano /etc/thingsboard/conf/thingsboard.conf
```

将下面内容添加到配置文件中并**替换**“PUT_YOUR_POSTGRESQL_PASSWORD_HERE”为**postgres帐户密码**：

```shell
# PostgreSQL
export DATABASE_TS_TYPE=sql
export SPRING_DATASOURCE_URL=jdbc:postgresql://192.168.202.188:5432/thingsboard
export SPRING_DATASOURCE_USERNAME=postgres
export SPRING_DATASOURCE_PASSWORD=postgres
# Specify partitioning size for timestamp key-value storage. Allowed values: DAYS, MONTHS, YEARS, INDEFINITE.
export SQL_POSTGRES_TS_KV_PARTITIONING=MONTHS
```



#### 4.3.创建数据库

**注意：需要手动创建数据库 thingsboard，否则启动会失败。**

![](/images/iot/deploy/deploy-single/single-4.png)



### 2.不配置RabbitMQ



### 3.不配置Redis





## 四、卸载软件



### 1.CentOS卸载软件

```shell
1. 通过rpm -q <关键字>查到rpm包的名字。
2. 调用rpm -e <包名>删除特定的rpm包


# 安装
 rpm -ivh  rpm安装包的名字
 
# 卸载
 rpm -e   rpm的名字
 rpm -e   rpm的名字 --nodeps  (强制卸载)


[root@192 thingsboard]# rpm -qa thingsboard
thingsboard-3.4.4-1.noarch


#rpm -e thingsboard-3.4.4-1.noarch
```



### 2.卸载ThingsBoard

```shell
# 1.停止服务
[root@192 thingsboard]# systemctl stop thingsboard
[root@192 thingsboard]# systemctl status thingsboard
● thingsboard.service - thingsboard
   Loaded: loaded (/usr/lib/systemd/system/thingsboard.service; disabled; vendor preset: disabled)
   Active: inactive (dead)


# 2.通过rpm -q <关键字>查到rpm包的名字
[root@192 thingsboard]# rpm -qa thingsboard
thingsboard-3.4.4-1.noarch


# 3.卸载
[root@192 thingsboard]# rpm -e thingsboard-3.4.4-1.noarch
warning: /usr/share/thingsboard/conf/thingsboard.conf saved as /usr/share/thingsboard/conf/thingsboard.conf.rpmsave


# 4.删除安装目录

# 安装目录
[root@192 thingsboard]# pwd
/usr/share/thingsboard
[root@192 thingsboard]# ll
total 0
drwxr-xr-x. 3 thingsboard thingsboard 41 Oct  8 00:50 bin
drwxr-xr-x. 4 thingsboard thingsboard 67 Oct  8 00:50 conf
drwxr-xr-x. 7 thingsboard thingsboard 74 Sep  8 00:26 data

# 日志目录
[root@192 thingsboard]# pwd
/var/log/thingsboard
[root@192 thingsboard]# ll
total 10636
-rw-r--r--. 1 thingsboard thingsboard  348247 Oct  8 00:49 gc.log
-rw-r--r--. 1 thingsboard thingsboard  170872 Sep  8 00:33 gc.log.0
-rw-r--r--. 1 thingsboard thingsboard 2256074 Sep  7 18:35 gc.log.1
-rw-r--r--. 1 thingsboard thingsboard 6587768 Sep  7 21:31 gc.log.2
-rw-r--r--. 1 thingsboard thingsboard  640558 Sep 19 06:13 gc.log.3
-rw-r--r--. 1 thingsboard thingsboard  311432 Sep 19 17:44 gc.log.4
-rw-r--r--. 1 thingsboard thingsboard   12809 Sep  8 00:33 install.log
-rw-r--r--. 1 thingsboard thingsboard  427992 Sep 19 17:44 thingsboard.2023-09-19.0.log
-rw-r--r--. 1 thingsboard thingsboard  114042 Oct  8 00:49 thingsboard.log

# 配置文件
[root@192 conf]# pwd
/etc/thingsboard/conf
[root@192 conf]# ll
total 80
-rwxr-xr--. 1 thingsboard thingsboard   649 Feb  7  2023 banner.txt
drwxr-xr-x. 2 thingsboard thingsboard    33 Oct  8 00:59 i18n
-rwxr-xr--. 1 thingsboard thingsboard  1799 Feb  7  2023 logback.xml
drwxr-xr-x. 2 thingsboard thingsboard   265 Oct  8 00:59 templates
-rwxr-xr--. 1 thingsboard thingsboard  1388 Feb  7  2023 thingsboard.conf
-rwxr-xr--. 1 thingsboard thingsboard 68856 Feb  7  2023 thingsboard.yml



# 5.删除数据库

```



