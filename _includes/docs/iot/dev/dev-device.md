* TOC
{:toc}



## 一、设备规划



### 1.设备规划

1. **ThingsBoard创建、删除实体等产生消息**
2. **实体更新消息通过规则引擎发送到消息队列（RabbitMQ或Kafka、EMQX）**
3. **通过REST API对设备进行管理（增、删、改、查）**





## 二、准备工作



### 1.RabbitMQ

device-queue、device-exchange

![](/images/iot/dev/dev-device/device-4.png)

![](/images/iot/dev/dev-device/device-5.png)



### 2.规则引擎

#### 2.1.创建规则链

![](/images/iot/dev/dev-device/device-6.png)

![](/images/iot/dev/dev-device/device-7.png)

![](/images/iot/dev/dev-device/device-10.png)



#### 2.2.数据整合

```javascript
msg.msgType = msgType;
msg.entityName = metadata.entityName;
msg.entityType = metadata.entityType;

return {msg: msg, metadata: metadata, msgType: msgType};
```



![](/images/iot/dev/dev-device/device-8.png)



#### 2.3.配置RabbitMQ

![](/images/iot/dev/dev-device/device-9.png)



#### 2.4.测试规则引擎

![](/images/iot/dev/dev-device/device-11.png)

```json
# 创建设备

{
    "id": {
        "entityType": "DEVICE",
        "id": "e4852e50-68a5-11ee-afb9-c790163a721a"
    },
    "createdTime": 1697077220789,
    "additionalInfo": {
        "gateway": false,
        "overwriteActivityTime": false,
        "description": ""
    },
    "tenantId": {
        "entityType": "TENANT",
        "id": "088202c0-64f4-11ee-b6d5-bdc7c43c6c8f"
    },
    "customerId": {
        "entityType": "CUSTOMER",
        "id": "13814000-1dd2-11b2-8080-808080808080"
    },
    "name": "device4",
    "type": "default",
    "label": "",
    "deviceProfileId": {
        "entityType": "DEVICE_PROFILE",
        "id": "088de9a0-64f4-11ee-b6d5-bdc7c43c6c8f"
    },
    "deviceData": {
        "configuration": {
            "type": "DEFAULT"
        },
        "transportConfiguration": {
            "type": "DEFAULT"
        }
    },
    "firmwareId": null,
    "softwareId": null,
    "externalId": null,
    "msgType": "ENTITY_CREATED",
    "entityName": "device4",
    "entityType": "DEVICE"
}
```



```json
# 删除设备

{
    "id": {
        "entityType": "DEVICE",
        "id": "e4852e50-68a5-11ee-afb9-c790163a721a"
    },
    "createdTime": 1697077220789,
    "additionalInfo": {
        "gateway": false,
        "overwriteActivityTime": false,
        "description": ""
    },
    "tenantId": {
        "entityType": "TENANT",
        "id": "088202c0-64f4-11ee-b6d5-bdc7c43c6c8f"
    },
    "customerId": {
        "entityType": "CUSTOMER",
        "id": "13814000-1dd2-11b2-8080-808080808080"
    },
    "name": "device4",
    "type": "default",
    "label": "",
    "deviceProfileId": {
        "entityType": "DEVICE_PROFILE",
        "id": "088de9a0-64f4-11ee-b6d5-bdc7c43c6c8f"
    },
    "deviceData": {
        "configuration": {
            "type": "DEFAULT"
        },
        "transportConfiguration": {
            "type": "DEFAULT"
        }
    },
    "firmwareId": null,
    "softwareId": null,
    "externalId": null,
    "msgType": "ENTITY_DELETED",
    "entityName": "device4",
    "entityType": "DEVICE"
}
```



### 3.数据

#### 3.1.创建设备

```json
# 数据

{
    "id": {
        "entityType": "DEVICE",
        "id": "f8c1d520-68a1-11ee-afb9-c790163a721a"
    },
    "createdTime": 1697075536754,
    "additionalInfo": {
        "gateway": false,
        "overwriteActivityTime": false,
        "description": ""
    },
    "tenantId": {
        "entityType": "TENANT",
        "id": "088202c0-64f4-11ee-b6d5-bdc7c43c6c8f"
    },
    "customerId": {
        "entityType": "CUSTOMER",
        "id": "13814000-1dd2-11b2-8080-808080808080"
    },
    "name": "device3",
    "type": "default",
    "label": "",
    "deviceProfileId": {
        "entityType": "DEVICE_PROFILE",
        "id": "088de9a0-64f4-11ee-b6d5-bdc7c43c6c8f"
    },
    "deviceData": {
        "configuration": {
            "type": "DEFAULT"
        },
        "transportConfiguration": {
            "type": "DEFAULT"
        }
    },
    "firmwareId": null,
    "softwareId": null,
    "externalId": null
}
```



```json
# 元数据

{
    "entityName": "device3",
    "entityType": "DEVICE",
    "userEmail": "tenant@thingsboard.org",
    "userId": "08f69680-64f4-11ee-b6d5-bdc7c43c6c8f",
    "userName": "tenant@thingsboard.org"
}
```



#### 3.2.删除数据

```json
# 数据

{
    "id": {
        "entityType": "DEVICE",
        "id": "f8c1d520-68a1-11ee-afb9-c790163a721a"
    },
    "createdTime": 1697075536754,
    "additionalInfo": {
        "gateway": false,
        "overwriteActivityTime": false,
        "description": ""
    },
    "tenantId": {
        "entityType": "TENANT",
        "id": "088202c0-64f4-11ee-b6d5-bdc7c43c6c8f"
    },
    "customerId": {
        "entityType": "CUSTOMER",
        "id": "13814000-1dd2-11b2-8080-808080808080"
    },
    "name": "device3",
    "type": "default",
    "label": "",
    "deviceProfileId": {
        "entityType": "DEVICE_PROFILE",
        "id": "088de9a0-64f4-11ee-b6d5-bdc7c43c6c8f"
    },
    "deviceData": {
        "configuration": {
            "type": "DEFAULT"
        },
        "transportConfiguration": {
            "type": "DEFAULT"
        }
    },
    "firmwareId": null,
    "softwareId": null,
    "externalId": null
}
```



```json
# 元数据

{
    "entityName": "device3",
    "entityType": "DEVICE",
    "userEmail": "tenant@thingsboard.org",
    "userId": "08f69680-64f4-11ee-b6d5-bdc7c43c6c8f",
    "userName": "tenant@thingsboard.org"
}
```





## 三、REST API



### 1.设备API 

#### 1.1.设备端API

![](/images/iot/dev/dev-device/device-1.png)



#### 1.2.设备服务端API

![](/images/iot/dev/dev-device/device-2.png)



#### 1.3.设备配置API

![](/images/iot/dev/dev-device/device-3.png)





## 四、REST API测试



### 1.getDeviceById

![](/images/iot/dev/dev-device/device-12.png)



### 2.deleteDvice

![](/images/iot/dev/dev-device/device-13.png)



### 3.getDeviceInfoById

![](/images/iot/dev/dev-device/device-14.png)



### 4.getDeviceTypes

![](/images/iot/dev/dev-device/device-15.png)



### 5.getDevicesByIds

![](/images/iot/dev/dev-device/device-16.png)



