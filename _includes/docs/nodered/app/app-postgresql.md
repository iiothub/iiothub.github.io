* TOC
{:toc}



## 一、概述



### 1.PostgreSQL

![](/images/nodered/app/app-postgresql/pg-4.png)

![](/images/nodered/app/app-postgresql/pg-5.png)

```shell
82.157.166.86
33047
postgres
postgres
```



### 2.node-red-contrib-postgresql

```shell
https://flows.nodered.org/node/node-red-contrib-postgresql
```



![](/images/nodered/app/app-postgresql/pg-1.png)

![](/images/nodered/app/app-postgresql/pg-2.png)

![](/images/nodered/app/app-postgresql/pg-3.png)





## 二、Node-RED



### 1.查询数据

#### 1.1.配置流程

![](/images/nodered/app/app-postgresql/pg-6.png)

![](/images/nodered/app/app-postgresql/pg-7.png)

![](/images/nodered/app/app-postgresql/pg-8.png)

![](/images/nodered/app/app-postgresql/pg-9.png)



#### 1.2.测试

![](/images/nodered/app/app-postgresql/pg-10.png)



### 2.插入数据

#### 2.1.配置流程

```shell
INSERT INTO student(name,sex) VALUES('aaa', 'bbb');

INSERT INTO student VALUES('aaa', 'bbb');
```



![](/images/nodered/app/app-postgresql/pg-11.png)



#### 2.2.测试

![](/images/nodered/app/app-postgresql/pg-12.png)

![](/images/nodered/app/app-postgresql/pg-13.png)



### 3.更新数据

#### 3.1.配置流程

```shell
UPDATE student SET sex='45' where name = 'tom'
```



![](/images/nodered/app/app-postgresql/pg-14.png)



#### 3.2.测试

![](/images/nodered/app/app-postgresql/pg-15.png)

![](/images/nodered/app/app-postgresql/pg-16.png)



### 4.删除数据

略



