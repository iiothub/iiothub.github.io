* TOC
{:toc}



### 1.Yum安装TimescaleDB

#### 1.1.安装PostgreSQL

```shell
1.安装EPEL仓库(安装数据库有些包依赖EPEL仓库)
# yum install epel-release
 
2.安装yum源 
# yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
 
3.安装postgresql
# yum install -y postgresql11-server
 
4.数据库初始化
# /usr/pgsql-11/bin/postgresql-11-setup initdb
 
配置文件
# cd /var/lib/pgsql/11/data/
 
修改配置文件
vim /var/lib/pgsql/11/data/postgresql.conf
 
# 修改为如下：
listen_addresses = '*'
port = 5432 
 
# 修改配置文件
vim /var/lib/pgsql/11/data/pg_hba.conf
 
# 修改为如下：
host    all             all             0.0.0.0/0               md5
 
或
host    all             all             all                          md5
 
5.postgresql 安装目录授权（不做） 
# chown postgres:root -R /usr/pgsql-11/
 
6.启动服务 
# systemctl start postgresql-11 
# systemctl enable postgresql-11
# systemctl status postgresql-11
 
查看端口
# netstat -lntp
# netstat -nat
 
7.切换用户，设置数据库密码 
# su - postgres
$ psql -U postgres
# ALTER USER postgres with encrypted password 'postgres';
 
退出: \q
列出所有库 \l
列出所有用户 \du
列出库下所有表\d
```



#### 1.2.安装Timescaledb插件

- 安装Timescaledb插件（v2.0）

```shell
cat > /etc/yum.repos.d/timescale_timescaledb.repo <<EOL
[timescale_timescaledb]
name=timescale_timescaledb
baseurl=https://packagecloud.io/timescale/timescaledb/el/$(rpm -E %{rhel})/\$basearch
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://packagecloud.io/timescale/timescaledb/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
EOL
 
 
# yum update -y 
# yum install -y timescaledb-2-postgresql-11
```



- 配置Timescaledb 


```shell
#运行以下命令以启动配置向导：
 
 
# timescaledb-tune --pg-config=/usr/pgsql-11/bin/pg_config --quiet --yes 
 
[root@iZ2ze72jggg737pb9vp1g6Z ~]# timescaledb-tune --pg-config=/usr/pgsql-11/bin/pg_config --quiet --yes
Using postgresql.conf at this path:
/var/lib/pgsql/11/data/postgresql.conf
 
Writing backup to:
/tmp/timescaledb_tune.backup202103291649
 
Recommendations based on 7.64 GB of available memory and 2 CPUs for PostgreSQL 11
shared_preload_libraries = 'timescaledb'# (change requires restart)
shared_buffers = 1955MB
effective_cache_size = 5866MB
maintenance_work_mem = 1001211kB
work_mem = 10012kB
timescaledb.max_background_workers = 8
max_worker_processes = 13
max_parallel_workers_per_gather = 1
max_parallel_workers = 2
wal_buffers = 16MB
min_wal_size = 512MB
default_statistics_target = 500
random_page_cost = 1.1
checkpoint_completion_target = 0.9
max_locks_per_transaction = 64
autovacuum_max_workers = 10
autovacuum_naptime = 10
effective_io_concurrency = 200
timescaledb.last_tuned = '2021-03-29T16:49:56+08:00'
timescaledb.last_tuned_version = '0.11.0'
Saving changes to: /var/lib/pgsql/11/data/postgresql.conf
```



- 重启服务

```shell
# systemctl restart postgresql-11 
```



#### 1.3.创建Timescaledb扩展

- postgres创建扩展


```shell
# su - postgres
-bash-4.2$ psql
postgres=# CREATE EXTENSION timescaledb;
 
postgres=# CREATE EXTENSION timescaledb;
WARNING:  
WELCOME TO
 _____ _                               _     ____________  
|_   _(_)                             | |    |  _  \ ___ \ 
  | |  _ _ __ ___   ___  ___  ___ __ _| | ___| | | | |_/ / 
  | | | |  _ ` _ \ / _ \/ __|/ __/ _` | |/ _ \ | | | ___ \ 
  | | | | | | | | |  __/\__ \ (_| (_| | |  __/ |/ /| |_/ /
  |_| |_|_| |_| |_|\___||___/\___\__,_|_|\___|___/ \____/
               Running version 2.1.0
For more information on TimescaleDB, please visit the following links:
 
 1. Getting started: https://docs.timescale.com/getting-started
 2. API reference documentation: https://docs.timescale.com/api
 3. How TimescaleDB is designed: https://docs.timescale.com/introduction/architecture
 
Note: TimescaleDB collects anonymous reports to better understand and assist our users.
For more information and how to disable, please see our docs https://docs.timescaledb.com/using-timescaledb/telemetry.
 
CREATE EXTENSION
```

![](/images/middleware/timescale/deploy/deploy-1.png)

![](/images/middleware/timescale/deploy/deploy-2.png)



- 创建新数据库

```shell
# 现在创建一个新的空数据库
postgres=# CREATE database tutorial;
 
# 进入tutorial库
\c tutorial
 
postgres=# \c tutorial
psql (9.2.24, server 11.11)
WARNING: psql version 9.2, server version 11.0.
         Some psql features might not work.
You are now connected to database "tutorial" as user "postgres".
tutorial=# 
 
# 把tutorial库转换为使用TimescaleDB扩展数据库
tutorial=# CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;

# 链接数据库
psql -U postgres -d tutorial 
-bash-4.2$ psql -U postgres -d tutorial 
```



### 2.Docker安装Timescaledb

```shell
1.创建目录
# mkdir -p /data/psql
 
2.运行容器
docker run -d --name timescaledb -p 5432:5432 \
-e LANG="C.UTF-8" \
-e 'TZ=Asia/Shanghai' \
-e "POSTGRES_DB=postgres" \
-e "POSTGRES_USER=postgres" \
-e "POSTGRES_PASSWORD=postgres" \
-v /data/psql:/var/lib/postgresql/data \
timescale/timescaledb:2.1.0-pg11
 
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



