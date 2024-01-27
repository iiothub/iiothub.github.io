* TOC
{:toc}



### 1.流复制介绍

流复制其原理为：备库不断的从主库同步相应的数据，并在备库apply每个WAL record，这里的流复制每次传输单位是WAL日志的record。

**PostgreSQL物理流复制按照同步方式分为两类：**

- 异步流复制
- 同步流复制



**物理流复制具有以下特点：**

1. 延迟极低，不怕大事务
2. 支持断点续传
3. 支持多副本
4. 配置简单
5. 备库与主库物理完全一致，并支持只读



![](/images/middleware/postgresql/postgres-stream/rep-1.png)



### 2.异步流复制

#### 2.1.主库部署

- 安装postgresql

```shell
# yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
 
# yum install -y postgresql11-server 
 
 
# 关闭防火墙
firewall-cmd --state                 #查看默认防火墙状态（关闭后显示notrunning，开启后显示running）
systemctl stop firewalld.service     #停止firewall
systemctl start firewalld.service    #开启防火墙
systemctl disable firewalld.service  #禁止firewall开机启动
systemctl enable firewalld.service   #开启firewall开机启动
```



- 初始化数据库

```shell
主库初始化
# /usr/pgsql-11/bin/postgresql-11-setup initdb
 
启动服务 
# systemctl start postgresql-11 
 
服务自启动
# systemctl enable postgresql-11
 
切换用户，设置数据库密码 
# su - postgres
$ psql
# ALTER USER postgres with encrypted password 'postgres';
创建用于主从同步的用户， 用户名replica, 密码replica：
# CREATE ROLE replica login replication encrypted password 'replica';
 
postgres=# \q
#-bash-4.2$ exit
```



- 修改配置文件


```shell
1.修改连接权限
# vim /var/lib/pgsql/11/data/pg_hba.conf
 
# 客户端访问
host    all             all             all                     md5
# replica是用来做备份的用户，172.51.216.82/32是备的IP地址
host    replication     replica         172.51.216.82/32        md5
 
# 完整配置
# TYPE  DATABASE        USER            ADDRESS                 METHOD
 
# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
# IPv6 local connections:
host    all             all             ::1/128                 ident
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            ident
host    replication     all             ::1/128                 ident
 
host    all             all             all                     md5
host    replication     replica         172.51.216.82/32        md5
 
 
2.修改数据库配置：
# vim /var/lib/pgsql/11/data/postgresql.conf
 
同步增加配置：
synchronous_commit = on         # synchronization level;
synchronous_standby_names = 'msa'
 
listen_addresses = '*'            # what IP address(es) to listen on;
port = 5432                # (change requires restart)
max_connections = 512            # (change requires restart)
shared_buffers = 128MB            # min 128kB
dynamic_shared_memory_type = posix    # the default is the first option
wal_level = hot_standby        # minimal, replica, or logical
archive_mode = on        # enables archiving; off, on, or always
archive_command = 'cp %p /var/lib/pgsql/11/data/pg_archive/%f'        # command to use to archive a logfile segment
max_wal_senders = 6        # max number of walsender processes
wal_keep_segments = 256    # in logfile segments, 16MB each; 0 disables
wal_sender_timeout = 60s    # in milliseconds; 0 disables
log_directory = 'log'    # directory where log files are written 
 
 
修改完，要创建刚刚配置的一些目录结构：
# mkdir /var/lib/pgsql/11/data/pg_archive/
# chown -R postgres.postgres /var/lib/pgsql/11/data
```



- 重启主库服务

```shell
# systemctl restart postgresql-11
# systemctl status postgresql-11 
```



#### 2.2.备库部署

- 安装postgresql

```shell
# yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
 
# yum install -y postgresql11-server 
 
 
# 关闭防火墙
firewall-cmd --state                 #查看默认防火墙状态（关闭后显示notrunning，开启后显示running）
systemctl stop firewalld.service     #停止firewall
systemctl start firewalld.service    #开启防火墙
systemctl disable firewalld.service  #禁止firewall开机启动
systemctl enable firewalld.service   #开启firewall开机启动
```



- 初始化数据库

```shell
主库初始化
# /usr/pgsql-11/bin/postgresql-11-setup initdb
 
启动服务 
# systemctl start postgresql-11 
 
服务自启动
# systemctl enable postgresql-11
```



- 拷贝主库数据

```shell
1.进入data目录,清空从节点数据
# su - postgres
$ cd /var/lib/pgsql/11/data/
$ rm -rf *
 
 
2.把主节点所有的数据文件都会拷贝过来
-bash-4.2$ pg_basebackup -h 172.51.216.81 -U replica -D /var/lib/pgsql/11/data/ -X stream -P
Password: replica 
```



-  修改配置文件 

```shell
1、修改从库配置文件
-bash-4.2$ vim /var/lib/pgsql/11/data/postgresql.conf 
 
listen_addresses = '*'            # what IP address(es) to listen on;
port = 5432                # (change requires restart)
max_connections = 1000            # (change requires restart)
shared_buffers = 128MB            # min 128kB
dynamic_shared_memory_type = posix    # the default is the first option
wal_level = replica        # minimal, replica, or logical
archive_mode = on        # enables archiving; off, on, or always
archive_command = 'cp %p /var/lib/pgsql/11/data/pg_archive/%f'        # command to use to archive a logfile segment
wal_sender_timeout = 60s    # in milliseconds; 0 disables
hot_standby = on            # "on" allows queries during recovery
max_standby_streaming_delay = 30s    # max delay before canceling queries
wal_receiver_status_interval = 10s    # send replies at least this often
hot_standby_feedback = on        # send info from standby to prevent
log_directory = 'log'    # directory where log files are written,
 
 
2.创建恢复文件recovery.conf
-bash-4.2$ cp /usr/pgsql-11/share/recovery.conf.sample /var/lib/pgsql/11/data/recovery.conf
-bash-4.2$ vim /var/lib/pgsql/11/data/recovery.conf
 
# 修改参数：
recovery_target_timeline = 'latest'   #同步到最新数据
standby_mode = on                     #指明从库身份
trigger_file = 'failover.now'
primary_conninfo = 'host=172.51.216.81 port=5432 user=replica password=replica'  #连接到主库信息 
 
切换到root用户
$ exit
```



- 重启从库服务

```shell
# systemctl restart postgresql-11
# systemctl start postgresql-11
# systemctl status postgresql-11
 
# netstat -lntp
# netstat -nat
```



#### 2.3.测试

```shell
进入主节点：
su - postgres
psql
 
在主库上运行以下命令
postgres=# select client_addr,sync_state from pg_stat_replication;
 
postgres=# select client_addr,sync_state from pg_stat_replication;
  client_addr  | sync_state 
---------------+------------
 172.51.216.82 | async
(1 row)
 
 
postgres=# \x
Expanded display is on.
postgres=# select * from pg_stat_replication;
-[ RECORD 1 ]----+------------------------------
pid              | 8437
usesysid         | 16384
usename          | replica
application_name | walreceiver
client_addr      | 172.51.216.82
client_hostname  | 
client_port      | 37542
backend_start    | 2021-03-12 13:28:55.818239+08
backend_xmin     | 572
state            | streaming
sent_lsn         | 0/C01DE30
write_lsn        | 0/C01DE30
flush_lsn        | 0/C01DE30
replay_lsn       | 0/C01DE30
write_lag        | 
flush_lag        | 
replay_lag       | 
sync_priority    | 0
sync_state       | async
 
 
 
# 方法-1
在主库端检查，说明89服务器是从节点，在接收流，而且是异步流复制：
postgres=# select usename , application_name , client_addr,sync_state from pg_stat_replication;
-[ RECORD 1 ]----+--------------
usename          | replica
application_name | walreceiver
client_addr      | 172.51.216.89
sync_state       | async
 
 
# 方法-2
在主、从节点分别执行如下命令：
 
# 主
postgres  34833  11712  0 14:14 ?        00:00:00 postgres: walsender replica 172.51.216.89(51848) streaming 0/9024250
 
# 从
postgres  77147  77128  0 14:14 ?        00:00:03 postgres: walreceiver   streaming 0/9024250
```



### 3.同步复制

#### 3.1.主库部署

- 安装postgresql

```shell
# yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
 
# yum install -y postgresql11-server 
 
 
# 关闭防火墙
firewall-cmd --state                 #查看默认防火墙状态（关闭后显示notrunning，开启后显示running）
systemctl stop firewalld.service     #停止firewall
systemctl start firewalld.service    #开启防火墙
systemctl disable firewalld.service  #禁止firewall开机启动
systemctl enable firewalld.service   #开启firewall开机启动
```



- 初始化数据库

```shell
主库初始化
# /usr/pgsql-11/bin/postgresql-11-setup initdb
 
启动服务 
# systemctl start postgresql-11 
 
服务自启动
# systemctl enable postgresql-11
 
切换用户，设置数据库密码 
# su - postgres
$ psql
# ALTER USER postgres with encrypted password 'postgres';
创建用于主从同步的用户， 用户名replica, 密码replica：
# CREATE ROLE replica login replication encrypted password 'replica';
 
postgres=# \q
#-bash-4.2$ exit
```



- 修改配置文件

```shell
1.修改连接权限
# vim /var/lib/pgsql/11/data/pg_hba.conf
 
# 客户端访问
host    all             all             all                     md5
# replica是用来做备份的用户，172.51.216.82/32是备的IP地址
host    replication     replica         172.51.216.82/32        md5
 
#完成配置
# TYPE  DATABASE        USER            ADDRESS                 METHOD
 
# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
# IPv6 local connections:
host    all             all             ::1/128                 ident
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            ident
host    replication     all             ::1/128                 ident
 
host    all             all             all                     md5
host    replication     replica         172.51.216.82/32        md5
 
 
2.修改数据库配置：
# vim /var/lib/pgsql/11/data/postgresql.conf
 
同步增加配置：
synchronous_commit = on         # synchronization level;
synchronous_standby_names = 'msa'
 
 
listen_addresses = '*'            # what IP address(es) to listen on;
port = 5432                # (change requires restart)
max_connections = 512            # (change requires restart)
shared_buffers = 128MB            # min 128kB
dynamic_shared_memory_type = posix    # the default is the first option
wal_level = hot_standby        # minimal, replica, or logical
archive_mode = on        # enables archiving; off, on, or always
archive_command = 'cp %p /var/lib/pgsql/11/data/pg_archive/%f'        # command to use to archive a logfile segment
max_wal_senders = 6        # max number of walsender processes
wal_keep_segments = 256    # in logfile segments, 16MB each; 0 disables
wal_sender_timeout = 60s    # in milliseconds; 0 disables
log_directory = 'log'    # directory where log files are written 
 
 
修改完，要创建刚刚配置的一些目录结构：
# mkdir /var/lib/pgsql/11/data/pg_archive/
# chown -R postgres.postgres /var/lib/pgsql/11/data
```



- 重启主库服务

```shell
# systemctl restart postgresql-11
# systemctl status postgresql-11 
```



#### 3.2.备库部署

- 安装postgresql

```shell
# yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
 
# yum install -y postgresql11-server 
 
 
# 关闭防火墙
firewall-cmd --state                 #查看默认防火墙状态（关闭后显示notrunning，开启后显示running）
systemctl stop firewalld.service     #停止firewall
systemctl start firewalld.service    #开启防火墙
systemctl disable firewalld.service  #禁止firewall开机启动
systemctl enable firewalld.service   #开启firewall开机启动
```



- 初始化数据库

```shell
主库初始化
# /usr/pgsql-11/bin/postgresql-11-setup initdb
 
启动服务 
# systemctl start postgresql-11 
 
服务自启动
# systemctl enable postgresql-11
```



- 拷贝主库数据

```shell
1.进入data目录,清空从节点数据
# su - postgres
$ cd /var/lib/pgsql/11/data/
$ rm -rf *
 
 
2.把主节点所有的数据文件都会拷贝过来
-bash-4.2$ pg_basebackup -h 172.51.216.81 -U replica -D /var/lib/pgsql/11/data/ -X stream -P
Password: replica
```



-  修改配置文件 

```shell
1、修改从库配置文件
-bash-4.2$ vim /var/lib/pgsql/11/data/postgresql.conf 
 
listen_addresses = '*'            # what IP address(es) to listen on;
port = 5432                # (change requires restart)
max_connections = 1000            # (change requires restart)
shared_buffers = 128MB            # min 128kB
dynamic_shared_memory_type = posix    # the default is the first option
wal_level = replica        # minimal, replica, or logical
archive_mode = on        # enables archiving; off, on, or always
archive_command = 'cp %p /var/lib/pgsql/11/data/pg_archive/%f'        # command to use to archive a logfile segment
wal_sender_timeout = 60s    # in milliseconds; 0 disables
hot_standby = on            # "on" allows queries during recovery
max_standby_streaming_delay = 30s    # max delay before canceling queries
wal_receiver_status_interval = 10s    # send replies at least this often
hot_standby_feedback = on        # send info from standby to prevent
log_directory = 'log'    # directory where log files are written,
 
 
2.创建恢复文件recovery.conf
-bash-4.2$ cp /usr/pgsql-11/share/recovery.conf.sample /var/lib/pgsql/11/data/recovery.conf
-bash-4.2$ vim /var/lib/pgsql/11/data/recovery.conf
 
# 修改参数：
recovery_target_timeline = 'latest'   #同步到最新数据
standby_mode = on                     #指明从库身份
trigger_file = 'failover.now'
primary_conninfo = 'host=172.51.216.81 port=5432 user=replica password=replica application_name=msa'  #连接到主库信息 
 
同步primary_conninfo增加：
application_name=msa
 
切换到root用户
$ exit
```



- 重启从库服务

```shell
# systemctl restart postgresql-11
# systemctl start postgresql-11
# systemctl status postgresql-11
 
# netstat -lntp
# netstat -nat
```



#### 3.3.测试

```shell
# 进入主节点：
su - postgres
psql
 
在主库上运行以下命令
postgres=# select client_addr,sync_state from pg_stat_replication;
 
postgres=# select client_addr,sync_state from pg_stat_replication;
  client_addr  | sync_state 
---------------+------------
 172.51.216.82 | sync
(1 row)
 
 
postgres=# \x
Expanded display is on.
postgres=# select * from pg_stat_replication;
-[ RECORD 1 ]----+------------------------------
pid              | 61092
usesysid         | 16384
usename          | replica
application_name | msa
client_addr      | 172.51.216.82
client_hostname  | 
client_port      | 48152
backend_start    | 2021-03-12 14:30:54.523831+08
backend_xmin     | 573
state            | streaming
sent_lsn         | 0/100000D0
write_lsn        | 0/100000D0
flush_lsn        | 0/100000D0
replay_lsn       | 0/100000D0
write_lag        | 
flush_lag        | 
replay_lag       | 
sync_priority    | 1
sync_state       | sync
 
 
 
# 方法-1
在主库端检查，说明89服务器是从节点，在接收流，而且是异步流复制：
postgres=# select usename , application_name , client_addr,sync_state from pg_stat_replication;
-[ RECORD 1 ]----+--------------
usename          | replica
application_name | walreceiver
client_addr      | 172.51.216.89
sync_state       | sync
 
 
# 方法-2
在主、从节点分别执行如下命令：
 
# 主
postgres  34833  11712  0 14:14 ?        00:00:00 postgres: walsender replica 172.51.216.89(51848) streaming 0/9024250
 
# 从
postgres  77147  77128  0 14:14 ?        00:00:03 postgres: walreceiver   streaming 0/9024250
```



### 4.主备切换

- 关闭主库

```shell
在主库执行 pg_ctl stop 模拟主库宕机。 
pg_ctl stop
 
-bash-4.2$  pg_ctl stop
waiting for server to shut down.... done
server stopped
 
 
这时备库日志会报错，提示 primary 主库连接不上
2021-03-15 13:22:57.311 CST [66145] FATAL:  could not connect to the primary server: could not connect to server: Connection refused
   Is the server running on host "172.51.216.81" and accepting
   TCP/IP connections on port 5432?
```



- 激活备库

```shell
在备库执行 pg_ctl promote 激活备库 
 
-bash-4.2$ pg_ctl promote
waiting for server to promote.... done
server promoted
 
 
备库激活后可以插入数据，变为可读写。这时配置文件 recovery.conf 变为 recovery.done。 

postgres=#  SELECT pg_is_in_recovery();
 pg_is_in_recovery 
-------------------
 f
(1 row)
```



