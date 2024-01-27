* TOC
{:toc}



## 一、环境准备



### 1.测试环境

```shell
# ThingsBoard
http://192.168.202.188:8080/login

# Node-RED
http://192.168.202.188:1880

设备名称：Node-RED-tcp
访问令牌：ZeOyL33yncOg5XcubZuk

Nodered流程: TB-tcp

# 网络调试助手
192.168.202.1
8888
```



### 2.创建设备

<img src="/images/iot/nodered/nodered-link-tcp/tcp-1.png" style="zoom:50%;" />



### 3.创建Node-RED流程

![](/images/iot/nodered/nodered-link-tcp/tcp-2.png)



### 4.网络调试助手

![](/images/iot/nodered/nodered-link-tcp/tcp-3.png)





## 二、上传遥测



### 1.启动TCP服务

![](/images/iot/nodered/nodered-link-tcp/tcp-4.png)



### 2.配置流程

![](/images/iot/nodered/nodered-link-tcp/tcp-5.png)

![](/images/iot/nodered/nodered-link-tcp/tcp-6.png)

![](/images/iot/nodered/nodered-link-tcp/tcp-7.png)

![](/images/iot/nodered/nodered-link-tcp/tcp-8.png)



### 3.上传遥测

![](/images/iot/nodered/nodered-link-tcp/tcp-9.png)

![](/images/iot/nodered/nodered-link-tcp/tcp-10.png)

![](/images/iot/nodered/nodered-link-tcp/tcp-11.png)





## 三、订阅共享属性

```shell
//订阅一个主题
emqClient.subscribe("v1/devices/me/attributes", QosEnum.QoS1);


订阅者订阅到了消息,topic=v1/devices/me/attributes,messageid=1,qos=1,
payload=

{
	"shared2": 10001
}
```



### 1.配置Node-RED流程

![](/images/iot/nodered/nodered-link-tcp/tcp-12.png)

![](/images/iot/nodered/nodered-link-tcp/tcp-13.png)

![](/images/iot/nodered/nodered-link-tcp/tcp-14.png)



### 2.修改共享属性

![](/images/iot/nodered/nodered-link-tcp/tcp-15.png)

![](/images/iot/nodered/nodered-link-tcp/tcp-16.png)

![](/images/iot/nodered/nodered-link-tcp/tcp-17.png)





## 四、总结



![](/images/iot/nodered/nodered-link-tcp/tcp-18.png)



