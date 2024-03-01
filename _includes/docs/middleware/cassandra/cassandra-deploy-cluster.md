* TOC
{:toc}



## 一、概述



### 1.官方文档

```shell
https://cassandra.apache.org/_/index.html
https://cassandra.apache.org/_/download.html


# 下载 cassandra-4.0.1
https://archive.apache.org/dist/cassandra/
https://archive.apache.org/dist/cassandra/4.0.1/
```



![](/images/middleware/cassandra/cassandra-deploy-cluster/deploy-1.png)

### 2. 克隆服务器

```shell
# 克隆机器

# 修改IP地址
cd /etc/sysconfig/network-scripts
vim ifcfg-ens33
192.168.202.156

# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

# 设置主机名
hostnamectl set-hostname cassandra
```



### 3.安装说明

采用3台CentOS x64系统（虚拟机）

为每台虚拟机设置静态IP

- 192.168.202.151 (seed)
- 192.168.202.152 (seed)
- 192.168.202.153

选择 151,、152 两台机器作为集群的种子节点（seed）。



| 服务器名称 | IP地址          | 备注 |
| ---------- | --------------- | ---- |
| Cassan-1   | 192.168.202.151 | seed |
| Cassan-2   | 192.168.202.152 | seed |
| Cassan-3   | 192.168.202.153 |      |



### 4.安装准备

安装准备工作 3 台服务器都要操作。



#### 4.1.安装 JDK 11

注意：Cassandra 使用 JAVA 语言开发，首先保证当前机器中已经安装 JDK 11

```shell
# 安装JDK 11 

# yum install java-11-openjdk -y

# java -version

# cd /usr/lib/jvm
```



```shell
[root@cassandra cassandra]# yum install java-11-openjdk -y


[root@cassandra cassandra]# java -version
openjdk version "11.0.22" 2024-01-16 LTS
OpenJDK Runtime Environment (Red_Hat-11.0.22.0.7-1.el7_9) (build 11.0.22+7-LTS)
OpenJDK 64-Bit Server VM (Red_Hat-11.0.22.0.7-1.el7_9) (build 11.0.22+7-LTS, mixed mode, sharing)


[root@cassandra cassandra]# cd /usr/lib/jvm
[root@cassandra jvm]# ll
total 0
drwxr-xr-x. 6 root root 68 Feb 28 19:22 java-11-openjdk-11.0.22.0.7-1.el7_9.x86_64
lrwxrwxrwx. 1 root root 21 Feb 28 19:22 jre -> /etc/alternatives/jre
lrwxrwxrwx. 1 root root 24 Feb 28 19:22 jre-11 -> /etc/alternatives/jre_11
lrwxrwxrwx. 1 root root 32 Feb 28 19:22 jre-11-openjdk -> /etc/alternatives/jre_11_openjdk
lrwxrwxrwx. 1 root root 42 Feb 28 19:22 jre-11-openjdk-11.0.22.0.7-1.el7_9.x86_64 -> java-11-openjdk-11.0.22.0.7-1.el7_9.x86_64
lrwxrwxrwx. 1 root root 29 Feb 28 19:22 jre-openjdk -> /etc/alternatives/jre_openjdk
```

![](/images/middleware/cassandra/cassandra-deploy-cluster/deploy-2.png)



#### 4.2.安装 Python

注意：Cassandra的客户端的使用需要用的Python2.X版本。需要先安装Python2.X

```shell
[root@cassandra cassandra]# python -V
Python 2.7.5
```

![](/images/middleware/cassandra/cassandra-deploy-cluster/deploy-3.png)



#### 4.3.下载文件

```shell
# 下载 4.0.1
# wget https://archive.apache.org/dist/cassandra/4.0.1/apache-cassandra-4.0.1-bin.tar.gz


# 解压
# tar -zxvf apache-cassandra-4.0.1-bin.tar.gz

[root@cassandra cassandra]# ll
total 48248
drwxr-xr-x. 8 root root      176 Feb 28 19:09 apache-cassandra-4.0.1
-rw-r--r--. 1 root root 49404559 Feb 28 19:08 apache-cassandra-4.0.1-bin.tar.gz


# 移动文件
[root@cassandra cassandra]# mv apache-cassandra-4.0.1 /usr/local/

[root@cassandra apache-cassandra-4.0.1]# ll
total 600
drwxr-xr-x. 2 root root    230 Feb 28 19:09 bin
-rw-r--r--. 1 root root   4832 Aug 30  2021 CASSANDRA-14092.txt
-rw-r--r--. 1 root root 434601 Aug 30  2021 CHANGES.txt
drwxr-xr-x. 3 root root   4096 Feb 28 19:09 conf
drwxr-xr-x. 3 root root     33 Feb 28 19:09 doc
drwxr-xr-x. 3 root root   4096 Feb 28 19:09 lib
-rw-r--r--. 1 root root  12960 Aug 30  2021 LICENSE.txt
-rw-r--r--. 1 root root 135759 Aug 30  2021 NEWS.txt
-rw-r--r--. 1 root root    349 Aug 30  2021 NOTICE.txt
drwxr-xr-x. 3 root root    230 Feb 28 19:09 pylib
drwxr-xr-x. 4 root root    169 Feb 28 19:09 tools
```

![](/images/middleware/cassandra/cassandra-deploy-cluster/deploy-4.png)





## 二、安装部署



### 1.配置 Cassandra

**1.进入解压后的目录，创建3个 Cassandra 的数据文件夹**

```shell
[root@cassandra apache-cassandra-4.0.1]# mkdir data
[root@cassandra apache-cassandra-4.0.1]# mkdir commitlog
[root@cassandra apache-cassandra-4.0.1]# mkdir saved-caches
```



```shell
[root@cassandra apache-cassandra-4.0.1]# pwd
/usr/local/apache-cassandra-4.0.1
[root@cassandra apache-cassandra-4.0.1]# mkdir data
[root@cassandra apache-cassandra-4.0.1]# mkdir commitlog
[root@cassandra apache-cassandra-4.0.1]# mkdir saved-caches
```

![](/images/middleware/cassandra/cassandra-deploy-cluster/deploy-5.png)



**2.修改配置文件**

在 conf 目录中找到 cassandra.yaml 配置文件，配置上面创建的3个数据目录

- 配置 data_file_directories

```shell
data_file_directories:
    - /usr/local/apache-cassandra-4.0.1/data
```



- 配置 commitlog_directory

```shell
commitlog_directory: /usr/local/apache-cassandra-4.0.1/commitlog
```



- 配置 saved_caches_directory

```shell
saved_caches_directory: /usr/local/apache-cassandra-4.0.1/saved_caches
```



- 集群配置

需要在每台机器的配置文件cassandra.yml中进行一些修改，包括

cluster_name 集群名字，每个节点都要一样

seeds 填写2个节点的ip作为 种子节点，每个节点的内容都要一样

listen_address 填写当前节点所在机器的IP地址

rpc_address 填写当前节点所在机器的IP地址



具体修改如下：

192.168.202.151机器修改的内容：7000

```shell
cluster_name: 'Cassandra Cluster'
seed_provider:
  - class_name: org.apache.cassandra.locator.SimpleSeedProvider
    parameters:
         - seeds: "192.168.202.151,192.168.202.152"
listen_address: 192.168.202.151
rpc_address: 192.168.202.151
```



192.168.202.152 机器的修改内容

```shell
cluster_name: 'Cassandra Cluster'
seed_provider:
  - class_name: org.apache.cassandra.locator.SimpleSeedProvider
    parameters:
         - seeds: "192.168.202.151,192.168.202.152"
listen_address: 192.168.202.152
rpc_address: 192.168.202.152
```



192.168.202.153 机器的修改内容

```shell
cluster_name: 'Cassandra Cluster'
seed_provider:
  - class_name: org.apache.cassandra.locator.SimpleSeedProvider
    parameters:
         - seeds: "192.168.202.151,192.168.202.152"
listen_address: 192.168.202.153
rpc_address: 192.168.202.153
```

修改完成后，启动每个节点。可以在192.168.202.151机器上使用noodtools status 命令进行测试



### 2.启动 Cassandra

```shell
# cd /usr/local/apache-cassandra-4.0.1/bin

# ./cassandra -R
```

```shell
[root@cassandra /]# cd /usr/local/apache-cassandra-4.0.1/bin
[root@cassandra bin]# ll
total 152
-rwxr-xr-x. 1 root root 10542 Aug 30  2021 cassandra
-rw-r--r--. 1 root root  5667 Aug 30  2021 cassandra.in.sh
-rwxr-xr-x. 1 root root  2995 Aug 30  2021 cqlsh
-rwxr-xr-x. 1 root root 95408 Aug 30  2021 cqlsh.py
-rwxr-xr-x. 1 root root  1894 Aug 30  2021 debug-cql
-rwxr-xr-x. 1 root root  3491 Aug 30  2021 nodetool
-rwxr-xr-x. 1 root root  1770 Aug 30  2021 sstableloader
-rwxr-xr-x. 1 root root  1778 Aug 30  2021 sstablescrub
-rwxr-xr-x. 1 root root  1778 Aug 30  2021 sstableupgrade
-rwxr-xr-x. 1 root root  1781 Aug 30  2021 sstableutil
-rwxr-xr-x. 1 root root  1778 Aug 30  2021 sstableverify
-rwxr-xr-x. 1 root root  1175 Aug 30  2021 stop-server
```

![](/images/middleware/cassandra/cassandra-deploy-cluster/deploy-6.png)



```shell
[root@cassandra bin]# ./cassandra -R
[root@cassandra bin]# OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
CompileCommand: dontinline org/apache/cassandra/db/Columns$Serializer.deserializeLargeSubset(Lorg/apache/cassandra/io/util/DataInputPlus;Lorg/apache/cassandra/db/Columns;I)Lorg/apache/cassandra/db/Columns;
CompileCommand: dontinline org/apache/cassandra/db/Columns$Serializer.serializeLargeSubset(Ljava/util/Collection;ILorg/apache/cassandra/db/Columns;ILorg/apache/cassandra/io/util/DataOutputPlus;)V
CompileCommand: dontinline org/apache/cassandra/db/Columns$Serializer.serializeLargeSubsetSize(Ljava/util/Collection;ILorg/apache/cassandra/db/Columns;I)I
```

![](/images/middleware/cassandra/cassandra-deploy-cluster/deploy-7.png)

 

输入命令来查看正在运行的cassandra的 pid

```shell
ps -ef|grep cassandra
```

显示如图，pid 是 1746：

![](/images/middleware/cassandra/cassandra-deploy-cluster/deploy-8.png)



### 3.关闭Cassandra

刚才已经查到了 pid，现在可以使用命令杀掉这个pid对应的进程

```shell
kill -9 1746
```



### 4.查看状态

运行bin 目录下的 nodetool

```shell
[root@localhost bin]# ./nodetool status

# ./nodetool -Dcom.sun.jndi.rmiURLParsing=legacy status
# ./nodetool -h ::FFFF:127.0.0.1 status
```

![](/images/middleware/cassandra/cassandra-deploy-cluster/deploy-9.png)



```shell
# 问题
[root@cassandra bin]# ./nodetool status
nodetool: Failed to connect to '127.0.0.1:7199' - URISyntaxException: 'Malformed IPv6 address at index 7: rmi://[127.0.0.1]:7199'.


# 解决办法
[root@cassandra bin]# ./nodetool -Dcom.sun.jndi.rmiURLParsing=legacy status
[root@cassandra bin]# ./nodetool -h ::FFFF:127.0.0.1 status
```



### 5.客户端连接服务器

进入Cassandra的 bin 目录，输入

```shell
./cqlsh 192.168.202.151 9042


[root@cassandra-1 bin]# ./cqlsh 192.168.202.151 9042
Python 2.7 support is deprecated. Install Python 3.6+ or set CQLSH_NO_WARN_PY2 to suppress this message.

Connected to Cassandra Cluster at 192.168.202.151:9042
[cqlsh 6.0.0 | Cassandra 4.0.1 | CQL spec 3.4.5 | Native protocol v5]
Use HELP for help.
cqlsh> 
```

![](/images/middleware/cassandra/cassandra-deploy-cluster/deploy-10.png)



### 6.服务运行脚本

为了方便管理，可以编写脚本来管理，在 /usr/local/apache-cassandra-4.0.1 下创建一个 startme.sh，输入一下内容：

```shell
#!/bin/sh
CASSANDRA_DIR="/usr/local/apache-cassandra-4.0.1"
 echo "************cassandra***************"
case "$1" in
        start)

                echo "*                                  *"
                echo "*            starting              *"
                nohup $CASSANDRA_DIR/bin/cassandra -R >> $CASSANDRA_DIR/logs/system.log 2>&1 &
                echo "*            started               *"
                echo "*                                  *"
                echo "************************************"
                ;;
        stop)

                echo "*                                  *"
                echo "*           stopping               *"
                PID_COUNT=`ps aux |grep CassandraDaemon |grep -v grep | wc -l`
                PID=`ps aux |grep CassandraDaemon |grep -v grep | awk {'print $2'}`
                if [ $PID_COUNT -gt 0 ];then
                        echo "*           try stop               *"
                        kill -9 $PID
                        echo "*          kill  SUCCESS!          *"
                else
                        echo "*          there is no !           *"
                echo "*                                  *"
                echo "************************************"
                fi
                ;;
        restart)

                echo "*                                  *"
                echo "*********     restarting      ******"
                $0 stop
                $0 start
                echo "*                                  *"
                echo "************************************"
                ;;
        status)
                $CASSANDRA_DIR/bin/nodetool status
                ;;

        *)
        echo "Usage:$0 {start|stop|restart|status}"

        exit 1
esac
```


接下来就可以使用这个脚本进行 启动，重启，关闭 的操作

```shell
[root@cassandra-1 apache-cassandra-4.0.1]# ./startme.sh start
[root@cassandra-1 apache-cassandra-4.0.1]# ./startme.sh restart
[root@cassandra-1 apache-cassandra-4.0.1]# ./startme.sh stop
```



```shell
# chmod +x startme.sh


[root@cassandra-1 apache-cassandra-4.0.1]# ./startme.sh start
************cassandra***************
*                                  *
*            starting              *
*            started               *
*                                  *
************************************
[root@cassandra-1 apache-cassandra-4.0.1]# ./startme.sh restart
************cassandra***************
*                                  *
*********     restarting      ******
************cassandra***************
*                                  *
*           stopping               *
*           try stop               *
*          kill  SUCCESS!          *
************cassandra***************
*                                  *
*            starting              *
*            started               *
*                                  *
************************************
*                                  *
************************************
[root@cassandra-1 apache-cassandra-4.0.1]# ./startme.sh stop
************cassandra***************
*                                  *
*           stopping               *
*           try stop               *
*          kill  SUCCESS!          *
```



