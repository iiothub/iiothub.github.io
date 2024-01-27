



## 一、环境准备



### 1.测试环境

```shell
# ThingsBoard
http://192.168.202.188:8080/login

# EMQX
http://192.168.202.189:18083

# Node-RED
http://192.168.202.188:1880

设备名称：Node-RED-mqtt
访问令牌：5eUOggpjfPsvUPUUJ0U5

Nodered流程: TB-mqtt
```



### 2.创建设备

<img src="/images/iot/nodered/nodered-link-mqtt/mqtt-1.png" style="zoom:50%;" />



### 3.创建Node-RED流程

![](/images/iot/nodered/nodered-link-mqtt/mqtt-2.png)



### 4.MQTTX

![](/images/iot/nodered/nodered-link-mqtt/mqtt-3.png)





## 二、上传遥测



### 1.配置Node-RED流程

```shell
# 发布主题
v1/devices/me/telemetry

# 访问令牌
8Qgo3kgDRDVTXWtIcnQk
```



配置上传遥测

![](/images/iot/nodered/nodered-link-mqtt/mqtt-4.png)

![](/images/iot/nodered/nodered-link-mqtt/mqtt-5.png)



配置emqx

![](/images/iot/nodered/nodered-link-mqtt/mqtt-6.png)

![](/images/iot/nodered/nodered-link-mqtt/mqtt-7.png)

![](/images/iot/nodered/nodered-link-mqtt/mqtt-8.png)



### 2.EMQX发送数据

```shell
v1/devices/me/telemetry

{
	"stringKey": "value1",
	"booleanKey": true,
	"doubleKey": 42.0,
	"longKey": 73,
	"jsonKey": {
		"someNumber": 42,
		"someArray": [1, 2, 3],
		"someNestedObject": {
			"key": "value"
		}
	}
}
```



![](/images/iot/nodered/nodered-link-mqtt/mqtt-9.png)

![](/images/iot/nodered/nodered-link-mqtt/mqtt-10.png)

![](/images/iot/nodered/nodered-link-mqtt/mqtt-11.png)





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

![](/images/iot/nodered/nodered-link-mqtt/mqtt-12.png)

![](/images/iot/nodered/nodered-link-mqtt/mqtt-13.png)

![](/images/iot/nodered/nodered-link-mqtt/mqtt-14.png)



### 2.EMQX订阅共享属性

<img src="/images/iot/nodered/nodered-link-mqtt/mqtt-15.png" style="zoom: 67%;" />

![](/images/iot/nodered/nodered-link-mqtt/mqtt-16.png)

![](/images/iot/nodered/nodered-link-mqtt/mqtt-17.png)

![](/images/iot/nodered/nodered-link-mqtt/mqtt-18.png)





## 四、总结



![](/images/iot/nodered/nodered-link-mqtt/mqtt-19.png)



