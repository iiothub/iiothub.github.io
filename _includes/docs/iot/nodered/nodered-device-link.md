* TOC
{:toc}



## 一、环境准备



### 1.测试环境

```shell
# ThingsBoard
http://192.168.202.188:8080/login

# Node-RED
http://192.168.202.188:1880

设备名称：Node-RED
访问令牌：8Qgo3kgDRDVTXWtIcnQk

Nodered流程: TB-device
```



### 2.创建设备

<img src="/images/iot/nodered/nodered-device-link/device-1.png" style="zoom: 50%;" />



### 3.创建Node-RED流程

![](/images/iot/nodered/nodered-device-link/device-2.png)





## 二、上传遥测



```shell
# 发布主题
v1/devices/me/telemetry

# 访问令牌
8Qgo3kgDRDVTXWtIcnQk
```



### 1.配置MQTT

![](/images/iot/nodered/nodered-device-link/device-3.png)

![](/images/iot/nodered/nodered-device-link/device-4.png)

![](/images/iot/nodered/nodered-device-link/device-5.png)

![](/images/iot/nodered/nodered-device-link/device-6.png)

![](/images/iot/nodered/nodered-device-link/device-7.png)

![](/images/iot/nodered/nodered-device-link/device-8.png)



### 2.配置流程

![](/images/iot/nodered/nodered-device-link/device-9.png)

![](/images/iot/nodered/nodered-device-link/device-10.png)





## 三、属性



### 1.上传客户端属性

```shell
v1/devices/me/attributes

{
	"attribute1": "value1",
	"attribute2": true,
	"attribute3": 42.0,
	"attribute4": 73,
	"attribute5": {
		"someNumber": 42,
		"someArray": [1, 2, 3],
		"someNestedObject": {
			"key": "value"
		}
	}
}
```



![](/images/iot/nodered/nodered-device-link/device-11.png)

![](/images/iot/nodered/nodered-device-link/device-12.png)

![](/images/iot/nodered/nodered-device-link/device-13.png)

![](/images/iot/nodered/nodered-device-link/device-14.png)

![](/images/iot/nodered/nodered-device-link/device-15.png)



### 2.下载服务端属性

```shell
# 订阅一个主题
emqClient.subscribe("v1/devices/me/attributes/response/+", QosEnum.QoS1);

# 发布消息
emqClient.publish("v1/devices/me/attributes/request/1",data, QosEnum.QoS1,false);

# 发布消息请求的属性
{
	"clientKeys": "attribute1,attribute2",
	"sharedKeys": "shared1,shared2"
}
```



#### 2.1.订阅服务端属性

![](/images/iot/nodered/nodered-device-link/device-16.png)

![](/images/iot/nodered/nodered-device-link/device-17.png)

![](/images/iot/nodered/nodered-device-link/device-18.png)



#### 2.2.发布请求属性

![](/images/iot/nodered/nodered-device-link/device-19.png)

![](/images/iot/nodered/nodered-device-link/device-20.png)

![](/images/iot/nodered/nodered-device-link/device-21.png)

![](/images/iot/nodered/nodered-device-link/device-22.png)



### 3.订阅共享属性

```shell
//订阅一个主题
emqClient.subscribe("v1/devices/me/attributes", QosEnum.QoS1);

订阅者订阅到了消息,topic=v1/devices/me/attributes,messageid=1,qos=1,
payload=

{
	"shared2": 10001
}
```



![](/images/iot/nodered/nodered-device-link/device-23.png)

![](/images/iot/nodered/nodered-device-link/device-24.png)

![](/images/iot/nodered/nodered-device-link/device-25.png)

![](/images/iot/nodered/nodered-device-link/device-26.png)





## 四、服务端RPC



客户端订阅服务端RPC命令必须SUBSCRIBE消息发送下面主题：

```shell
v1/devices/me/rpc/request/+
```

订阅后客户端会收到一条命令作为对相应主题的PUBLISH命令：

```shell
v1/devices/me/rpc/request/$request_id
```

**$request_id**表示请求的整型标识符。

客户端PUBLISH下面主题进行响应：

```shell
v1/devices/me/rpc/response/$request_id
```



### 1.订阅服务端RPC

![](/images/iot/nodered/nodered-device-link/device-27.png)

![](/images/iot/nodered/nodered-device-link/device-28.png)



### 2.创建仪表板

![](/images/iot/nodered/nodered-device-link/device-29.png)

![](/images/iot/nodered/nodered-device-link/device-30.png)

![](/images/iot/nodered/nodered-device-link/device-31.png)





## 五、总结



![](/images/iot/nodered/nodered-device-link/device-32.png)



