* TOC
{:toc}



## 一、事件处理规划



### 1.事件规划

1. **处理事件类型：CONNECT_EVENT、DISCONNECT_EVENT、ACTIVITY_EVENT、INACTIVITY_EVENT**
2. **ThingsBoard通过规则引擎把事件路由到消息队列（RabbitMQ或Kafka、EMQX）**





## 二、准备工作



### 1.RabbitMQ

队列：event-queue

![](/images/iot/dev/dev-event/event-1.png)

![](/images/iot/dev/dev-event/event-2.png)



### 2.规则引擎

#### 2.1.创建规则链

![](/images/iot/dev/dev-event/event-3.png)

![](/images/iot/dev/dev-event/event-4.png)

![](/images/iot/dev/dev-event/event-5.png)



#### 2.2.数据整合

```javascript
msg.msgType = msgType;
msg.deviceName = metadata.deviceName;
msg.deviceType = metadata.deviceType;
msg.deviceLabel = metadata.deviceLabel;

return {msg: msg, metadata: metadata, msgType: msgType};
```



```json
{
    "lastConnectTime": 1697017421141,
    "lastActivityTime": 1697016501369,
    "lastDisconnectTime": 1697016501383,
    "lastInactivityAlarmTime": 1697017103552,
    "inactivityTimeout": 600000,
    "msgType": "CONNECT_EVENT",
    "deviceName": "Test-event",
    "deviceType": "default",
    "deviceLabel": ""
}

{
    "deviceLabel": "",
    "deviceName": "Test-event",
    "deviceType": "default",
    "scope": "SERVER_SCOPE"
}
```



![](/images/iot/dev/dev-event/event-6.png)



#### 2.3.RabbitMQ

![](/images/iot/dev/dev-event/event-7.png)



### 3.数据

#### 1.CONNECT_EVENT

```shell
{
    "lastConnectTime": 1697015317349,
    "lastActivityTime": 0,
    "lastDisconnectTime": 0,
    "lastInactivityAlarmTime": 0,
    "inactivityTimeout": 600000
}

# 元数据
{
    "deviceLabel": "",
    "deviceName": "Test-event",
    "deviceType": "default",
    "scope": "SERVER_SCOPE"
}
```



#### 2.DISCONNECT_EVENT

```shell
{
    "active": true,
    "lastConnectTime": 1697015317349,
    "lastActivityTime": 1697015317350,
    "lastDisconnectTime": 1697015348610,
    "lastInactivityAlarmTime": 0,
    "inactivityTimeout": 600000
}

# 元数据
{
    "deviceLabel": "",
    "deviceName": "Test-event",
    "deviceType": "default",
    "scope": "SERVER_SCOPE"
}
```



#### 3.ACTIVITY_EVENT

```shell
{
    "active": true,
    "lastConnectTime": 1697015317349,
    "lastActivityTime": 1697015317350,
    "lastDisconnectTime": 0,
    "lastInactivityAlarmTime": 0,
    "inactivityTimeout": 600000
}

# 元数据
{
    "deviceLabel": "",
    "deviceName": "Test-event",
    "deviceType": "default",
    "scope": "SERVER_SCOPE"
}
```



#### 4.INACTIVITY_EVENT

```shell
{
    "active": false,
    "lastConnectTime": 1697016478869,
    "lastActivityTime": 1697016501369,
    "lastDisconnectTime": 1697016501383,
    "lastInactivityAlarmTime": 1697017103552,
    "inactivityTimeout": 600000
}

# 元数据
{
    "deviceLabel": "",
    "deviceName": "Test-event",
    "deviceType": "default",
    "scope": "SERVER_SCOPE"
}
```



### 4.服务端属性

![](/images/iot/dev/dev-event/event-8.png)



### 5.MQTTX

![](/images/iot/dev/dev-event/event-10.png)





## 三、事件处理



### 1.创建设备

![](/images/iot/dev/dev-event/event-9.png)



### 2.创建规则引擎

![](/images/iot/dev/dev-event/event-3.png)

![](/images/iot/dev/dev-event/event-5.png)



### 3.配置RabbitMQ

![](/images/iot/dev/dev-event/event-7.png)



### 4.测试

![](/images/iot/dev/dev-event/event-11.png)

```json
{
	"lastConnectTime": 1697016478869,
	"lastActivityTime": 1697016109278,
	"lastDisconnectTime": 1697016109289,
	"lastInactivityAlarmTime": 0,
	"inactivityTimeout": 600000,
	"msgType": "CONNECT_EVENT",
	"deviceName": "Test-event",
	"deviceType": "default",
	"deviceLabel": ""
}


{
	"active": true,
	"lastConnectTime": 1697016478869,
	"lastActivityTime": 1697016478870,
	"lastDisconnectTime": 1697016501383,
	"lastInactivityAlarmTime": 0,
	"inactivityTimeout": 600000,
	"msgType": "DISCONNECT_EVENT",
	"deviceName": "Test-event",
	"deviceType": "default",
	"deviceLabel": ""
}


{
	"active": true,
	"lastConnectTime": 1697017421141,
	"lastActivityTime": 1697017421142,
	"lastDisconnectTime": 1697016501383,
	"lastInactivityAlarmTime": 1697017103552,
	"inactivityTimeout": 600000,
	"msgType": "ACTIVITY_EVENT",
	"deviceName": "Test-event",
	"deviceType": "default",
	"deviceLabel": ""
}


{
	"active": false,
	"lastConnectTime": 1697016478869,
	"lastActivityTime": 1697016501369,
	"lastDisconnectTime": 1697016501383,
	"lastInactivityAlarmTime": 1697017103552,
	"inactivityTimeout": 600000,
	"msgType": "INACTIVITY_EVENT",
	"deviceName": "Test-event",
	"deviceType": "default",
	"deviceLabel": ""
}
```





## 四、REST API



### 1.事件 API

![](/images/iot/dev/dev-event/event-12.png)



