* TOC
{:toc}



## 一、告警规划



### 1.告警规划

1. **每种设备定义设备配置，在设备配置中定义报警规则**
2. **ThingsBoard通过规则引擎整合元数据**
3. **ThingsBoard通过规则引擎把消息路由到消息队列（RabbitMQ或Kafka、EMQX）**
4. **消息队列（RabbitMQ）消费数据，保存到数据库**
5. **应用平台用一套报警规则**





## 二、准备工作



### 1.创建设备

#### 1.1.创建设备配置

<img src="/images/iot/dev/dev-alarm/alarm-1.png" style="zoom:50%;" />



#### 1.2.创建设备

<img src="/images/iot/dev/dev-alarm/alarm-2.png" style="zoom:50%;" />



### 2.配置设备告警

#### 2.1.创建告警规则

![](/images/iot/dev/dev-alarm/alarm-3.png)

![](/images/iot/dev/dev-alarm/alarm-4.png)

![](/images/iot/dev/dev-alarm/alarm-5.png)

![](/images/iot/dev/dev-alarm/alarm-6.png)

![](/images/iot/dev/dev-alarm/alarm-7.png)



#### 2.2.清除告警规则

![](/images/iot/dev/dev-alarm/alarm-8.png)

![](/images/iot/dev/dev-alarm/alarm-9.png)

![](/images/iot/dev/dev-alarm/alarm-10.png)

![](/images/iot/dev/dev-alarm/alarm-11.png)

![](/images/iot/dev/dev-alarm/alarm-12.png)



### 3.MQTTX

![](/images/iot/dev/dev-alarm/alarm-13.png)

```json
{
	"temperature": 62.2,
	"humidity": 79
}
```



### 4.RabbitMQ

![](/images/iot/dev/dev-alarm/alarm-18.png)

![](/images/iot/dev/dev-alarm/alarm-19.png)



### 5.规则链

#### 5.1.创建报警规则链

![](/images/iot/dev/dev-alarm/alarm-16.png)

![](/images/iot/dev/dev-alarm/alarm-15.png)

![](/images/iot/dev/dev-alarm/alarm-29.png)



#### 5.2.报警数据

**1.创建报警**

```json
{
    "deviceName": "test-alarm",
    "deviceType": "test-alarm",
    "isNewAlarm": "true",
    "ts": "1696840521020"
}


{
    "id": {
        "entityType": "ALARM",
        "id": "309cfa0c-6689-4fcc-9dd9-63c552c13518"
    },
    "createdTime": 1696840521024,
    "tenantId": {
        "entityType": "TENANT",
        "id": "088202c0-64f4-11ee-b6d5-bdc7c43c6c8f"
    },
    "customerId": null,
    "type": "高温报警",
    "originator": {
        "entityType": "DEVICE",
        "id": "23beb960-6676-11ee-afb9-c790163a721a"
    },
    "severity": "CRITICAL",
    "acknowledged": false,
    "cleared": false,
    "assigneeId": null,
    "startTs": 1696840521020,
    "endTs": 1696840521020,
    "ackTs": 0,
    "clearTs": 0,
    "assignTs": 0,
    "details": {
        "data": "温度：62.2"
    },
    "propagate": false,
    "propagateToOwner": false,
    "propagateToTenant": false,
    "propagateRelationTypes": [],
    "originatorName": "test-alarm",
    "originatorLabel": "test-alarm",
    "assignee": null,
    "name": "高温报警",
    "status": "ACTIVE_UNACK"
}
```



**2.更新报警**

```json
{
    "deviceName": "test-alarm",
    "deviceType": "test-alarm",
    "isExistingAlarm": "true",
    "ts": "1696840526453"
}


{
    "id": {
        "entityType": "ALARM",
        "id": "309cfa0c-6689-4fcc-9dd9-63c552c13518"
    },
    "createdTime": 1696840521024,
    "tenantId": {
        "entityType": "TENANT",
        "id": "088202c0-64f4-11ee-b6d5-bdc7c43c6c8f"
    },
    "customerId": null,
    "type": "高温报警",
    "originator": {
        "entityType": "DEVICE",
        "id": "23beb960-6676-11ee-afb9-c790163a721a"
    },
    "severity": "CRITICAL",
    "acknowledged": false,
    "cleared": false,
    "assigneeId": null,
    "startTs": 1696840521020,
    "endTs": 1696840526456,
    "ackTs": 0,
    "clearTs": 0,
    "assignTs": 0,
    "details": {
        "data": "温度：62.2"
    },
    "propagate": false,
    "propagateToOwner": false,
    "propagateToTenant": false,
    "propagateRelationTypes": [],
    "originatorName": "test-alarm",
    "originatorLabel": "test-alarm",
    "assignee": null,
    "name": "高温报警",
    "status": "ACTIVE_UNACK"
}
```



**3.清除报警**

```jso
{
    "deviceName": "test-alarm",
    "deviceType": "test-alarm",
    "isClearedAlarm": "true",
    "ts": "1696840531189"
}


{
    "id": {
        "entityType": "ALARM",
        "id": "309cfa0c-6689-4fcc-9dd9-63c552c13518"
    },
    "createdTime": 1696840521024,
    "tenantId": {
        "entityType": "TENANT",
        "id": "088202c0-64f4-11ee-b6d5-bdc7c43c6c8f"
    },
    "customerId": null,
    "type": "高温报警",
    "originator": {
        "entityType": "DEVICE",
        "id": "23beb960-6676-11ee-afb9-c790163a721a"
    },
    "severity": "CRITICAL",
    "acknowledged": false,
    "cleared": true,
    "assigneeId": null,
    "startTs": 1696840521020,
    "endTs": 1696840526769,
    "ackTs": 0,
    "clearTs": 1696840531190,
    "assignTs": 0,
    "details": {
        "data": "温度：42.2"
    },
    "propagate": false,
    "propagateToOwner": false,
    "propagateToTenant": false,
    "propagateRelationTypes": [],
    "originatorName": "test-alarm",
    "originatorLabel": "test-alarm",
    "assignee": null,
    "name": "高温报警",
    "status": "CLEARED_UNACK"
}
```



#### 5.3.整合元数据

```json
{
    "deviceName": "test-alarm",
    "deviceType": "test-alarm",
    "isNewAlarm": "true",
    "ts": "1696840521020"
}


{
    "deviceName": "test-alarm",
    "deviceType": "test-alarm",
    "isExistingAlarm": "true",
    "ts": "1696840526453"
}


{
    "deviceName": "test-alarm",
    "deviceType": "test-alarm",
    "isClearedAlarm": "true",
    "ts": "1696840531189"
}
```



![](/images/iot/dev/dev-alarm/alarm-17.png)



#### 5.4.配置RabbitMQ

![](/images/iot/dev/dev-alarm/alarm-20.png)





## 三、设备告警



### 1.创建设备

<img src="/images/iot/dev/dev-alarm/alarm-21.png" style="zoom:50%;" />



### 2.配置设备告警

![](/images/iot/dev/dev-alarm/alarm-22.png)



### 3.配置RabbitMQ

配置直连交换机：alarm-exchange、alarm-queue



### 4.配置规则引擎

#### 4.1.整体配置

![](/images/iot/dev/dev-alarm/alarm-23.png)

![](/images/iot/dev/dev-alarm/alarm-24.png)



#### 4.2.整个元数据

```json
{
    "deviceName": "test-alarm",
    "deviceType": "test-alarm",
    "isNewAlarm": "true",
    "ts": "1696840521020"
}


{
    "deviceName": "test-alarm",
    "deviceType": "test-alarm",
    "isExistingAlarm": "true",
    "ts": "1696840526453"
}


{
    "deviceName": "test-alarm",
    "deviceType": "test-alarm",
    "isClearedAlarm": "true",
    "ts": "1696840531189"
}
```



![](/images/iot/dev/dev-alarm/alarm-17.png)

#### 4.3.配置RabbitMQ

![](/images/iot/dev/dev-alarm/alarm-20.png)



### 5.测试

#### 5.1.MQTTX

```shell
v1/devices/me/telemetry


{
	"temperature": 22.2,
	"humidity": 79
}
```



![](/images/iot/dev/dev-alarm/alarm-25.png)



#### 5.2.告警数据

![](/images/iot/dev/dev-alarm/alarm-26.png)



**1.创建告警**

```json
{
	"id": {
		"entityType": "ALARM",
		"id": "7ff9f094-49fa-4b88-b73a-37ce92b33183"
	},
	"createdTime": 1696844052922,
	"tenantId": {
		"entityType": "TENANT",
		"id": "088202c0-64f4-11ee-b6d5-bdc7c43c6c8f"
	},
	"customerId": null,
	"type": "高温报警",
	"originator": {
		"entityType": "DEVICE",
		"id": "23beb960-6676-11ee-afb9-c790163a721a"
	},
	"severity": "CRITICAL",
	"acknowledged": false,
	"cleared": false,
	"assigneeId": null,
	"startTs": 1696844052913,
	"endTs": 1696844052913,
	"ackTs": 0,
	"clearTs": 0,
	"assignTs": 0,
	"details": {
		"data": "温度：62.2"
	},
	"propagate": false,
	"propagateToOwner": false,
	"propagateToTenant": false,
	"propagateRelationTypes": [],
	"originatorName": "test-alarm",
	"originatorLabel": "test-alarm",
	"assignee": null,
	"name": "高温报警",
	"status": "ACTIVE_UNACK",
	"deviceType": "test-alarm",
	"deviceName": "test-alarm",
	"isNewAlarm": "true",
	"ts": "1696844052913"
}
```



**2.更新告警**

```json
{
	"id": {
		"entityType": "ALARM",
		"id": "7ff9f094-49fa-4b88-b73a-37ce92b33183"
	},
	"createdTime": 1696844052922,
	"tenantId": {
		"entityType": "TENANT",
		"id": "088202c0-64f4-11ee-b6d5-bdc7c43c6c8f"
	},
	"customerId": null,
	"type": "高温报警",
	"originator": {
		"entityType": "DEVICE",
		"id": "23beb960-6676-11ee-afb9-c790163a721a"
	},
	"severity": "CRITICAL",
	"acknowledged": false,
	"cleared": false,
	"assigneeId": null,
	"startTs": 1696844052913,
	"endTs": 1696844057193,
	"ackTs": 0,
	"clearTs": 0,
	"assignTs": 0,
	"details": {
		"data": "温度：52.2"
	},
	"propagate": false,
	"propagateToOwner": false,
	"propagateToTenant": false,
	"propagateRelationTypes": [],
	"originatorName": "test-alarm",
	"originatorLabel": "test-alarm",
	"assignee": null,
	"name": "高温报警",
	"status": "ACTIVE_UNACK",
	"deviceType": "test-alarm",
	"isExistingAlarm": "true",
	"deviceName": "test-alarm",
	"ts": "1696844057173"
}
```



**3.清除告警**

```json
{
	"id": {
		"entityType": "ALARM",
		"id": "7ff9f094-49fa-4b88-b73a-37ce92b33183"
	},
	"createdTime": 1696844052922,
	"tenantId": {
		"entityType": "TENANT",
		"id": "088202c0-64f4-11ee-b6d5-bdc7c43c6c8f"
	},
	"customerId": null,
	"type": "高温报警",
	"originator": {
		"entityType": "DEVICE",
		"id": "23beb960-6676-11ee-afb9-c790163a721a"
	},
	"severity": "CRITICAL",
	"acknowledged": false,
	"cleared": true,
	"assigneeId": null,
	"startTs": 1696844052913,
	"endTs": 1696844057193,
	"ackTs": 0,
	"clearTs": 1696844061917,
	"assignTs": 0,
	"details": {
		"data": "温度：22.2"
	},
	"propagate": false,
	"propagateToOwner": false,
	"propagateToTenant": false,
	"propagateRelationTypes": [],
	"originatorName": "test-alarm",
	"originatorLabel": "test-alarm",
	"assignee": null,
	"name": "高温报警",
	"status": "CLEARED_UNACK",
	"deviceType": "test-alarm",
	"isClearedAlarm": "true",
	"deviceName": "test-alarm",
	"ts": "1696844061906"
}
```





## 四、REST API



### 1.告警API

![](/images/iot/dev/dev-alarm/alarm-27.png)

![](/images/iot/dev/dev-alarm/alarm-28.png)



