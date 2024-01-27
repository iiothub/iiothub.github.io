* TOC
{:toc}



## 一、集群方案



### 1.部署方式

- **kubernetes方式部署ThingsBoard集群**



### 2.安装环境准备

- **部署k8s集群**



### 3.安装ThingsBoard

```shell
# 安装服务

# 执行以下命令以运行安装：
./k8s-install-tb.sh --loadDemo

# 执行以下命令部署第三方资源：
./k8s-deploy-thirdparty.sh

# 执行以下命令以部署ThingsBoard资源：
./k8s-deploy-resources.sh
```





## 二、ThingsBoard集群部署



### 1.克隆脚本

```shell
git clone -b release-3.5.1 https://github.com/thingsboard/thingsboard-ce-k8s.git

cd thingsboard-ce-k8s/minikube
```

```shell
# tar -zxvf thingsboard-ce-k8s-3.5.1.tar.gz 

[root@192 thingsboard]# ll
total 19344
-rw-r--r--. 1 root root 19727671 Jul 20 17:27 minikube-latest.x86_64.rpm
drwxrwxr-x. 9 root root      161 May 31 20:44 thingsboard-ce-k8s-3.5.1
-rw-r--r--. 1 root root    75254 Jul 20 18:26 thingsboard-ce-k8s-3.5.1.tar.gz

[root@192 thingsboard]# cd thingsboard-ce-k8s-3.5.1
[root@192 thingsboard-ce-k8s-3.5.1]# ll
total 24
drwxrwxr-x. 4 root root    60 May 31 20:44 aws
drwxrwxr-x. 4 root root    60 May 31 20:44 azure
drwxrwxr-x. 4 root root    60 May 31 20:44 gcp
drwxrwxr-x. 3 root root    42 May 31 20:44 helm
drwxrwxr-x. 2 root root    29 May 31 20:44 kafka
-rw-rw-r--. 1 root root 11357 May 31 20:44 LICENSE
drwxrwxr-x. 4 root root  4096 May 31 20:44 minikube
drwxrwxr-x. 4 root root  4096 May 31 20:44 openshift
-rw-rw-r--. 1 root root   129 May 31 20:44 README.md

[root@192 thingsboard-ce-k8s-3.5.1]# cd minikube/
[root@192 minikube]# ll
total 92
-rw-rw-r--. 1 root root  3667 May 31 20:44 cassandra.yml
-rw-rw-r--. 1 root root  1441 May 31 20:44 database-setup.yml
drwxrwxr-x. 2 root root    38 May 31 20:44 hybrid
-rwxrwxr-x. 1 root root   822 May 31 20:44 k8s-delete-all.sh
-rwxrwxr-x. 1 root root   770 May 31 20:44 k8s-delete-resources.sh
-rwxrwxr-x. 1 root root   751 May 31 20:44 k8s-delete-thirdparty.sh
-rwxrwxr-x. 1 root root   894 May 31 20:44 k8s-deploy-resources.sh
-rwxrwxr-x. 1 root root   750 May 31 20:44 k8s-deploy-thirdparty.sh
-rwxrwxr-x. 1 root root  2627 May 31 20:44 k8s-install-tb.sh
-rwxrwxr-x. 1 root root  1262 May 31 20:44 k8s-upgrade-tb.sh
drwxrwxr-x. 2 root root    38 May 31 20:44 postgres
-rw-rw-r--. 1 root root  2338 May 31 20:44 postgres.yml
-rw-rw-r--. 1 root root   311 May 31 20:44 README.md
-rw-rw-r--. 1 root root  3355 May 31 20:44 routes.yml
-rw-rw-r--. 1 root root   693 May 31 20:44 tb-namespace.yml
-rw-rw-r--. 1 root root  3113 May 31 20:44 tb-node-configmap.yml
-rw-rw-r--. 1 root root  2762 May 31 20:44 tb-node.yml
-rw-rw-r--. 1 root root  5031 May 31 20:44 tb-transport-configmap.yml
-rw-rw-r--. 1 root root  8489 May 31 20:44 thingsboard.yml
-rw-rw-r--. 1 root root 10350 May 31 20:44 thirdparty.yml
```



### 2.配置数据库

![](/images/iot/deploy/deploy-cluster/k8s-1.png)



```shell
[root@192 minikube]# vim .env

# Database used by ThingsBoard, can be either postgres (PostgreSQL) or hybrid (PostgreSQL for entities database and Cassandra for timeseries database).
# According to the database type corresponding kubernetes resources will be deployed (see postgres.yml, cassandra.yml for details).
DATABASE=hybrid

# Replication factor for Cassandra database (will be ignored if PostgreSQL was selected as the database).
# Must be less or equals to the number of Cassandra nodes which can be configured in ./common/cassandra.yml ('StatefulSet.spec.replicas' property)
CASSANDRA_REPLICATION_FACTOR=1
```



### 3.修改配置文件

​          . . . . . .



### 4.安装

#### 4.1.运行安装

```shell
# 执行以下命令以运行安装：

./k8s-install-tb.sh --loadDemo

# 说明:
--loadDemo` -可选参数用于是否加载演示数据。
```



```shell
[root@k8s-master minikube]# ./k8s-install-tb.sh --loadDemo
namespace/thingsboard unchanged
Context "kubernetes-admin@kubernetes" modified.
persistentvolumeclaim/postgres-pv-claim created
deployment.apps/postgres created
service/tb-database created
Waiting for deployment "postgres" rollout to finish: 0 of 1 updated replicas are available...
deployment "postgres" successfully rolled out
configmap/cassandra-probe-config created
statefulset.apps/cassandra created
service/cassandra created
Waiting for 1 pods to be ready...
partitioned roll out complete: 1 new pods have been updated...
configmap/tb-node-db-config created
configmap/tb-node-config created
pod/tb-db-setup created
pod/tb-db-setup condition met
Starting ThingsBoard installation ...
09:22:25,805 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Could NOT find resource [logback-test.xml]
09:22:25,805 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Found resource [logback.xml] at [file:/config/logback.xml]
09:22:25,806 |-WARN in ch.qos.logback.classic.LoggerContext[default] - Resource [logback.xml] occurs multiple times on the classpath.
09:22:25,806 |-WARN in ch.qos.logback.classic.LoggerContext[default] - Resource [logback.xml] occurs at [file:/usr/share/thingsboard/conf/logback.xml]
09:22:25,806 |-WARN in ch.qos.logback.classic.LoggerContext[default] - Resource [logback.xml] occurs at [file:/config/logback.xml]
09:22:25,960 |-INFO in ch.qos.logback.classic.joran.action.ConfigurationAction - debug attribute not set
09:22:25,974 |-INFO in ch.qos.logback.classic.joran.action.ConfigurationAction - Will scan for changes in [file:/config/logback.xml] 
09:22:25,974 |-INFO in ch.qos.logback.classic.joran.action.ConfigurationAction - Setting ReconfigureOnChangeTask scanning period to 10 seconds
09:22:25,978 |-INFO in ch.qos.logback.core.joran.action.AppenderAction - About to instantiate appender of type [ch.qos.logback.core.rolling.RollingFileAppender]
09:22:25,984 |-INFO in ch.qos.logback.core.joran.action.AppenderAction - Naming appender as [fileLogAppender]
09:22:25,997 |-INFO in c.q.l.core.rolling.SizeAndTimeBasedRollingPolicy@1182320432 - setting totalSizeCap to 3 GB
09:22:26,004 |-INFO in c.q.l.core.rolling.SizeAndTimeBasedRollingPolicy@1182320432 - Archive files will be limited to [100 MB] each.
09:22:26,033 |-INFO in c.q.l.core.rolling.SizeAndTimeBasedRollingPolicy@1182320432 - No compression will be used
09:22:26,034 |-INFO in c.q.l.core.rolling.SizeAndTimeBasedRollingPolicy@1182320432 - Will use the pattern /var/log/thingsboard/tb-db-setup/thingsboard.%d{yyyy-MM-dd}.%i.log for the active file
09:22:26,037 |-INFO in ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP@6767c1fc - The date pattern is 'yyyy-MM-dd' from file name pattern '/var/log/thingsboard/tb-db-setup/thingsboard.%d{yyyy-MM-dd}.%i.log'.
09:22:26,037 |-INFO in ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP@6767c1fc - Roll-over at midnight.
09:22:26,043 |-INFO in ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP@6767c1fc - Setting initial period to Tue Aug 08 09:22:26 UTC 2023
09:22:26,045 |-INFO in ch.qos.logback.core.joran.action.NestedComplexPropertyIA - Assuming default type [ch.qos.logback.classic.encoder.PatternLayoutEncoder] for [encoder] property
09:22:26,073 |-INFO in ch.qos.logback.core.rolling.RollingFileAppender[fileLogAppender] - Active log file name: /var/log/thingsboard/tb-db-setup/thingsboard.log
09:22:26,073 |-INFO in ch.qos.logback.core.rolling.RollingFileAppender[fileLogAppender] - File property is set to [/var/log/thingsboard/tb-db-setup/thingsboard.log]
09:22:26,074 |-INFO in ch.qos.logback.core.joran.action.AppenderAction - About to instantiate appender of type [ch.qos.logback.core.ConsoleAppender]
09:22:26,076 |-INFO in ch.qos.logback.core.joran.action.AppenderAction - Naming appender as [STDOUT]
09:22:26,076 |-INFO in ch.qos.logback.core.joran.action.NestedComplexPropertyIA - Assuming default type [ch.qos.logback.classic.encoder.PatternLayoutEncoder] for [encoder] property
09:22:26,077 |-INFO in ch.qos.logback.classic.joran.action.LoggerAction - Setting level of logger [org.thingsboard.server] to INFO
09:22:26,077 |-INFO in ch.qos.logback.classic.joran.action.LoggerAction - Setting level of logger [com.google.common.util.concurrent.AggregateFuture] to OFF
09:22:26,077 |-INFO in ch.qos.logback.classic.joran.action.RootLoggerAction - Setting level of ROOT logger to INFO
09:22:26,078 |-INFO in ch.qos.logback.core.joran.action.AppenderRefAction - Attaching appender named [fileLogAppender] to Logger[ROOT]
09:22:26,078 |-INFO in ch.qos.logback.core.joran.action.AppenderRefAction - Attaching appender named [STDOUT] to Logger[ROOT]
09:22:26,078 |-INFO in ch.qos.logback.classic.joran.action.ConfigurationAction - End of configuration.
09:22:26,079 |-INFO in ch.qos.logback.classic.joran.JoranConfigurator@29ee9faa - Registering current configuration as safe fallback point

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
Installing Cassandra DataBase schema part: schema-keyspace.cql
Installing Cassandra DataBase schema part: schema-ts.cql
Loading system data...
Creating default notification configs for system admin
Creating default notification configs for all tenants
Loading demo data...
Installation finished successfully!
pod "tb-db-setup" deleted
```



![](/images/iot/deploy/deploy-cluster/k8s-2.png)



```shell
[root@k8s-master minikube]# kubectl get all
NAME                            READY   STATUS    RESTARTS   AGE
pod/cassandra-0                 1/1     Running   0          4m22s
pod/postgres-65f44bf565-ffkrs   1/1     Running   0          4m45s

NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/cassandra     ClusterIP   None             <none>        9042/TCP   4m22s
service/tb-database   ClusterIP   10.111.184.252   <none>        5432/TCP   4m45s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/postgres   1/1     1            1           4m45s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/postgres-65f44bf565   1         1         1       4m45s

NAME                         READY   AGE
statefulset.apps/cassandra   1/1     4m22s
```



#### 4.2.部署第三方资源

```shell
# 执行以下命令部署第三方资源：

./k8s-deploy-thirdparty.sh
```



```shell
[root@k8s-master minikube]# ./k8s-deploy-thirdparty.sh
Context "kubernetes-admin@kubernetes" modified.
configmap/tb-zookeeper created
statefulset.apps/zookeeper created
service/zookeeper created
service/zookeeper-headless created
configmap/tb-kafka created
statefulset.apps/tb-kafka created
service/tb-kafka created
deployment.apps/tb-redis created
service/tb-redis created
```



```shell
[root@k8s-master minikube]# kubectl get all
NAME                            READY   STATUS    RESTARTS   AGE
pod/cassandra-0                 1/1     Running   0          10m
pod/postgres-65f44bf565-ffkrs   1/1     Running   0          10m
pod/tb-kafka-0                  1/1     Running   0          88s
pod/tb-redis-b8964c499-4rlxb    1/1     Running   0          88s
pod/zookeeper-0                 1/1     Running   0          88s
pod/zookeeper-1                 1/1     Running   0          88s
pod/zookeeper-2                 1/1     Running   0          88s

NAME                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/cassandra            ClusterIP   None             <none>        9042/TCP                     10m
service/tb-database          ClusterIP   10.111.184.252   <none>        5432/TCP                     10m
service/tb-kafka             ClusterIP   None             <none>        9092/TCP                     88s
service/tb-redis             ClusterIP   10.109.130.94    <none>        6379/TCP                     88s
service/zookeeper            ClusterIP   10.101.92.147    <none>        2181/TCP,2888/TCP,3888/TCP   88s
service/zookeeper-headless   ClusterIP   None             <none>        2181/TCP,2888/TCP,3888/TCP   88s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/postgres   1/1     1            1           10m
deployment.apps/tb-redis   1/1     1            1           88s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/postgres-65f44bf565   1         1         1       10m
replicaset.apps/tb-redis-b8964c499    1         1         1       88s

NAME                         READY   AGE
statefulset.apps/cassandra   1/1     10m
statefulset.apps/tb-kafka    1/1     88s
statefulset.apps/zookeeper   3/3     88s
```



![](/images/iot/deploy/deploy-cluster/k8s-3.png)



#### 4.3.部署ThingsBoard资源

```shell
# 执行以下命令以部署ThingsBoard资源：

./k8s-deploy-resources.sh
```



```shell
[root@k8s-master minikube]# ./k8s-deploy-resources.sh
Context "kubernetes-admin@kubernetes" modified.
configmap/tb-node-config unchanged
configmap/tb-mqtt-transport-config created
configmap/tb-http-transport-config created
configmap/tb-coap-transport-config created
deployment.apps/tb-js-executor created
statefulset.apps/tb-mqtt-transport created
service/tb-mqtt-transport created
statefulset.apps/tb-http-transport created
service/tb-http-transport created
statefulset.apps/tb-coap-transport created
service/tb-coap-transport created
deployment.apps/tb-web-ui created
service/tb-web-ui created
statefulset.apps/tb-node created
service/tb-node created
ingress.networking.k8s.io/tb-ingress created
```



```shell
[root@k8s-master minikube]# kubectl get all
NAME                                  READY   STATUS    RESTARTS        AGE
pod/cassandra-0                       1/1     Running   0               26m
pod/postgres-65f44bf565-ffkrs         1/1     Running   0               26m
pod/tb-coap-transport-0               1/1     Running   2 (112s ago)    10m
pod/tb-http-transport-0               1/1     Running   1 (4m1s ago)    10m
pod/tb-js-executor-5bd7cb7bdd-2l78m   1/1     Running   0               10m
pod/tb-js-executor-5bd7cb7bdd-8mjdq   1/1     Running   0               10m
pod/tb-js-executor-5bd7cb7bdd-shs84   1/1     Running   0               10m
pod/tb-js-executor-5bd7cb7bdd-tphdn   1/1     Running   0               10m
pod/tb-js-executor-5bd7cb7bdd-wjp9r   1/1     Running   0               10m
pod/tb-kafka-0                        1/1     Running   0               17m
pod/tb-mqtt-transport-0               1/1     Running   1 (3m40s ago)   10m
pod/tb-node-0                         1/1     Running   1 (106s ago)    10m
pod/tb-redis-b8964c499-4rlxb          1/1     Running   0               17m
pod/tb-web-ui-bf5c5785b-nhb7q         1/1     Running   0               10m
pod/zookeeper-0                       1/1     Running   0               17m
pod/zookeeper-1                       1/1     Running   0               17m
pod/zookeeper-2                       1/1     Running   0               17m

NAME                         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/cassandra            ClusterIP      None             <none>        9042/TCP                     26m
service/tb-coap-transport    LoadBalancer   10.107.239.252   <pending>     5683:30423/UDP               10m
service/tb-database          ClusterIP      10.111.184.252   <none>        5432/TCP                     26m
service/tb-http-transport    ClusterIP      10.103.68.158    <none>        8080/TCP                     10m
service/tb-kafka             ClusterIP      None             <none>        9092/TCP                     17m
service/tb-mqtt-transport    ClusterIP      10.111.148.215   <none>        1883/TCP                     10m
service/tb-node              ClusterIP      10.100.119.68    <none>        8080/TCP                     10m
service/tb-redis             ClusterIP      10.109.130.94    <none>        6379/TCP                     17m
service/tb-web-ui            ClusterIP      10.98.137.65     <none>        8080/TCP                     10m
service/zookeeper            ClusterIP      10.101.92.147    <none>        2181/TCP,2888/TCP,3888/TCP   17m
service/zookeeper-headless   ClusterIP      None             <none>        2181/TCP,2888/TCP,3888/TCP   17m

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/postgres         1/1     1            1           26m
deployment.apps/tb-js-executor   5/5     5            5           10m
deployment.apps/tb-redis         1/1     1            1           17m
deployment.apps/tb-web-ui        1/1     1            1           10m

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/postgres-65f44bf565         1         1         1       26m
replicaset.apps/tb-js-executor-5bd7cb7bdd   5         5         5       10m
replicaset.apps/tb-redis-b8964c499          1         1         1       17m
replicaset.apps/tb-web-ui-bf5c5785b         1         1         1       10m

NAME                                 READY   AGE
statefulset.apps/cassandra           1/1     26m
statefulset.apps/tb-coap-transport   1/1     10m
statefulset.apps/tb-http-transport   1/1     10m
statefulset.apps/tb-kafka            1/1     17m
statefulset.apps/tb-mqtt-transport   1/1     10m
statefulset.apps/tb-node             1/1     10m
statefulset.apps/zookeeper           3/3     17m
```



![](/images/iot/deploy/deploy-cluster/k8s-6.png)



### 5.访问ThingsBoard

```shell
# 你应该看到ThingsBoard登录页面。
# 使用以下默认凭据：
System Administrator: sysadmin@thingsboard.org / sysadmin
System Administrator: sysadmin@thingsboard.org / sysadmin

# 如果使用演示数据（使用--loadDemo标志）安装了数据库则还可以使用以下凭据：
Tenant Administrator: tenant@thingsboard.org / tenant
Customer User: customer@thingsboard.org / customer
```



```shell
http://192.168.202.201:30088/login
tenant@thingsboard.org
tenant
```



<img src="/images/iot/deploy/deploy-cluster/k8s-9.png" style="zoom:50%;" />

![](/images/iot/deploy/deploy-cluster/k8s-10.png)



