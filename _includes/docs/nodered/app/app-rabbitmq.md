* TOC
{:toc}



## 一、概述



### 1.RabbitMQ

```shell
# 运行容器
docker run -d  --network host --restart=always --name rabbitmq rabbitmq:management


# 访问地址
http://82.157.188.11:15672/
# 账户/密码：
guest/guest 
```



### 2.node-red-contrib-amqp 

```shell
https://flows.nodered.org/node/node-red-contrib-amqp
```



#### 2.1.安装节点

![](/images/nodered/app/app-rabbitmq/rabbitmq-1.png)

![](/images/nodered/app/app-rabbitmq/rabbitmq-2.png)

![](/images/nodered/app/app-rabbitmq/rabbitmq-3.png)



