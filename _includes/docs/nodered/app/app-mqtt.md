* TOC
{:toc}



## 一、概述



### 1.EMQX

```shell
# 4.4
docker pull emqx/emqx:4.4.17

docker run -d --name emqx -p 1883:1883 -p 8081:8081 -p 8083:8083 -p 8084:8084 -p 8883:8883 -p 18083:18083 emqx/emqx:4.4.17
```

```shell
账号：admin
初始密码：public
修改密码：1qaz2wsx

http://192.168.202.168:18083
账号：admin
修改密码：1qaz2wsx
```



![](/images/nodered/app/app-mqtt/mqtt-1.png)



### 2.MQTTX

![](/images/nodered/app/app-mqtt/mqtt-2.png)





## 二、发布数据到MQTT



### 1.Node-RED配置

#### 1.1.配置 mqtt out

![](/images/nodered/app/app-mqtt/mqtt-3.png)

![](/images/nodered/app/app-mqtt/mqtt-4.png)

![](/images/nodered/app/app-mqtt/mqtt-5.png)

![](/images/nodered/app/app-mqtt/mqtt-6.png)



#### 1.2.配置流程

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



![](/images/nodered/app/app-mqtt/mqtt-7.png)

![](/images/nodered/app/app-mqtt/mqtt-8.png)

![](/images/nodered/app/app-mqtt/mqtt-9.png)



### 2.MQTTX订阅数据

![](/images/nodered/app/app-mqtt/mqtt-10.png)

![](/images/nodered/app/app-mqtt/mqtt-11.png)



### 3.测试

![](/images/nodered/app/app-mqtt/mqtt-12.png)

![](/images/nodered/app/app-mqtt/mqtt-14.png)

![](/images/nodered/app/app-mqtt/mqtt-13.png)





## 三、从MQTT订阅数据



### 1.Node-RED配置

#### 1.1.配置 mqtt in

![](/images/nodered/app/app-mqtt/mqtt-16.png)

![](/images/nodered/app/app-mqtt/mqtt-15.png)



#### 1.2.配置流程

![](/images/nodered/app/app-mqtt/mqtt-17.png)



### 2.MQTT发布数据

```shell
v1/devices/me/attribute

{
	"temperature": 62.2,
	"humidity": 79
}
```



![](/images/nodered/app/app-mqtt/mqtt-18.png)



### 3.测试

![](/images/nodered/app/app-mqtt/mqtt-19.png)

![](/images/nodered/app/app-mqtt/mqtt-20.png)





## 四、MQTT-Broker



### 1.node-red-contrib-aedes

```shell
node-red-contrib-aedes

https://flows.nodered.org/node/node-red-contrib-aedes
```



### 2.安装节点

![](/images/nodered/app/app-mqtt/mqtt-21.png)

![](/images/nodered/app/app-mqtt/mqtt-22.png)

![](/images/nodered/app/app-mqtt/mqtt-23.png)

![](/images/nodered/app/app-mqtt/mqtt-24.png)



### 3.配置 MQTT  Broker

```shell
MQTT version: 3.1.1
```



![](/images/nodered/app/app-mqtt/mqtt-25.png)



### 4.配置流程

![](/images/nodered/app/app-mqtt/mqtt-27.png)

![](/images/nodered/app/app-mqtt/mqtt-28.png)

![](/images/nodered/app/app-mqtt/mqtt-29.png)



### 4.测试

![](/images/nodered/app/app-mqtt/mqtt-26.png)

![](/images/nodered/app/app-mqtt/mqtt-30.png)

![](/images/nodered/app/app-mqtt/mqtt-31.png)

![](/images/nodered/app/app-mqtt/mqtt-32.png)



