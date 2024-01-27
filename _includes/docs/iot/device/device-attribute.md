* TOC
{:toc}



## 一、概述



### 1.IoT设备属性

ThingsBoard基于实体提供了自定义属性的功能并保存在数据中库用于可视化和数据处理。

属性是一种key-value格式由于key-value的灵活性可以与IoT设备无缝，键始终是一个字符串而值可以是string, boolean, double, integer或JSON。

**键值数据类型：**

- string 
- boolean
- double
- integer
- JSON



**例始：**

```json
{
	"firmwareVersion": "v2.3.1",
	"booleanParameter": true,
	"doubleParameter": 42.0,
	"longParameter": 73,
	"configuration": {
		"someNumber": 42,
		"someArray": [1, 2, 3],
		"someNestedObject": {
			"key": "value"
		}
	}
}
```



**名称**

建议使用[骆峰](https://en.wikipedia.org/wiki/Camel_case)命名方式做为属性名称这样对数据处理和可视化的自定义JS函数十分方便。



### 2.属性类型

**属性主要分为三种:**

- 服务端属性：服务端定义，服务端使用，设备端不能使用
- 共享属性：服务端定义，设备端可以使用，不能修改
- 客户端属性：设备端定义属性，服务端可以使用，不能修改



#### 2.1.服务端属性

几乎所有平台实体都支持这种类型的属性：Device, Asset, Customer, Tenant, User等；服务器端属性可以通过管理界面或REST API配置的属性，设备固件无法访问服务器端属性。

![](/images/iot/device/device-attribute/a-1.svg)



**设想要构建一个楼宇监控解决方案请查看下面示例：**

1. *latitude*, *longitude*和*address*代表建筑物或其他资产的服务器端属性可以在仪表板的地图部件上使用此属性来可视化建筑物的位置
2. *floorPlanImage*是图片的URL使用此属性在图片地图部件上可视化平面图
3. *maxTemperatureThreshold*和*temperatureAlarmEnabled*用于配置和启用/禁用设备或资产的警报



#### 2.2.共享属性

共享属性仅适用于设备类似于服务器端属性但区别在于能过订阅更新属性值，通过MQTT或其他双向通信协议可以订阅属性更新并实时接收通知；通过HTTP或其他请求-响应通信协议进行通信的设备可能会定期请求共享属性的值。

![](/images/iot/device/device-attribute/a-2.svg)

**设想要构建一个楼宇监控解决方案请查看下面示例：**

1. *targetFirmwareVersion*属性可用于存储设备的固件版本
2. *maxTemperature*属性可用于在房间温过过高时自动启用HVAC

可以通过UI、脚本或其他服务器端应用程序通过REST API更改属性值。



#### 2.3.客户端属性

客户端属性仅适用于设备向服务器上报各种半静态数据与[共享属性](http://www.ithingsboard.com/docs/user-guide/attributes/#shared-attributes)类似但是区别在设备应用程序将属性值从设备发送平台。

![](/images/iot/device/device-attribute/a-3.svg)

**设想要构建一个楼宇监控解决方案请查看下面示例：**

1. *currentFirmwareVersion*属性可用于向平台报告设备的应用程序版本
2. *currentConfiguration*属性可用于向平台报告设备的应用程序配置
3. *currentState*可用于通过网络持久保存和恢复设备的应用程序状态

用户和服务器端应用程序可以通过UI/REST API浏览客户端属性但无法更改它们，因为客户端属性对于UI/REST API是只读状态。



### 3.属性API

ThingsBoard属性API能够使设备具备如下功能

- 将[客户端](http://www.ithingsboard.com/docs/user-guide/attributes/#attribute-types)设备属性上传到服务端
- 从服务端请求[客户端和共享](http://www.ithingsboard.com/docs/user-guide/attributes/#attribute-types)属性
- 从服务端订阅[共享](http://www.ithingsboard.com/docs/user-guide/attributes/#attribute-types)属性



#### 3.1.发布客户端属性

**通过服务端发布客户端属性**

通过ThingsBoard服务端发布客户端属性必须PUBLISH消息到下面主题:

```shell
v1/devices/me/attributes
```



**new-attributes-values.json**

```json
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



#### 3.2.获取服务端属性

**通过服务端获取属性值**

通过ThingsBoard服务端获取**客户端属性**或**共享属性**必须PUBLISH消息到下面主题:

```shell
v1/devices/me/attributes/request/$request_id
```

其中**$request_id**表示整数的请求标识符。

在发送带有请求的PUBLISH消息之前客户端需要订阅。

```shell
v1/devices/me/attributes/response/+
```



**mqtt-js-attributes-request.js**

```javascript
var mqtt = require('mqtt')
var client  = mqtt.connect('mqtt://demo.thingsboard.io',{
    username: process.env.TOKEN
})

client.on('connect', function () {
    console.log('connected')
    client.subscribe('v1/devices/me/attributes/response/+')
    client.publish('v1/devices/me/attributes/request/1', '{"clientKeys":"attribute1,attribute2", "sharedKeys":"shared1,shared2"}')
})

client.on('message', function (topic, message) {
    console.log('response.topic: ' + topic)
    console.log('response.body: ' + message.toString())
    client.end()
})
```

**Result**

```json
{"client":{"attribute1":"value1","attribute2":true}}
```



#### 3.3.订阅服务端属性

**通过服务端订阅属性**

通过服务端订阅共享属性必须SUBSCRIBE消息到下面主题：

```shell
v1/devices/me/attributes
```

如果服务端组件（例如REST API或规则链）更改了共享属性时客户端就会收到对应属性值：

```shell
{"key1":"value1"}
```



### 4.数据库

**attribute_kv**

![](/images/iot/device/device-attribute/a-9.png)

```sql
SELECT * from attribute_kv WHERE attribute_key like 'attribute%'

SELECT * from attribute_kv WHERE attribute_key like 'shared%'
```





## 二、MQTT操作属性



### 1.环境准备

1. 创建测试设备 test-3
2. 创建测试工程 tb-attribute



**1.程序配置**

```yaml
mqtt:
  broker-url: tcp://192.168.202.188:1883
  client-id: emq-client-xxxxxx
  username: FO3JsWBgdukOr9pZjhab
  password:
```



**2.程序源码**

**上传属性（Upload）**

```java
package com.iiotos.attribute;

import com.iiotos.mqtt.EmqClient;
import com.iiotos.mqtt.MqttProperties;
import com.iiotos.mqtt.QosEnum;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;


@Component
public class Upload {

    @Autowired
    private EmqClient emqClient;

    @Autowired
    private MqttProperties properties;

    @PostConstruct
    public void init(){
        //连接服务端
        emqClient.connect(properties.getUsername(),properties.getPassword());
        //订阅一个主题
        //emqClient.subscribe("testtopic/#", QosEnum.QoS1);
    }

    @Scheduled(fixedRate = 2000)
    public void publish(){

        String data = getData();

        emqClient.publish("v1/devices/me/attributes",data,
                QosEnum.QoS1,false);
    }

    private String getData(){

        String data = "{\n" +
                "\t\"attribute1\": \"value1\",\n" +
                "\t\"attribute2\": true,\n" +
                "\t\"attribute3\": 42.0,\n" +
                "\t\"attribute4\": 73,\n" +
                "\t\"attribute5\": {\n" +
                "\t\t\"someNumber\": 42,\n" +
                "\t\t\"someArray\": [1, 2, 3],\n" +
                "\t\t\"someNestedObject\": {\n" +
                "\t\t\t\"key\": \"value\"\n" +
                "\t\t}\n" +
                "\t}\n" +
                "}";

        return data;

    }
}
```



**下载属性（Download）**

```java
package com.iiotos.attribute;

import com.iiotos.mqtt.EmqClient;
import com.iiotos.mqtt.MqttProperties;
import com.iiotos.mqtt.QosEnum;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;

@Component
public class Download {


    @Autowired
    private EmqClient emqClient;

    @Autowired
    private MqttProperties properties;

    @PostConstruct
    public void init(){
        //连接服务端
        emqClient.connect(properties.getUsername(),properties.getPassword());
        //订阅一个主题
        emqClient.subscribe("v1/devices/me/attributes/response/+", QosEnum.QoS1);
    }

    @Scheduled(fixedRate = 2000)
    public void publish(){

        String data = getData();

        emqClient.publish("v1/devices/me/attributes/request/1",data, QosEnum.QoS1,false);
    }

    private String getData(){

        String data = "{\n" +
                "\t\"clientKeys\": \"attribute1,attribute2\",\n" +
                "\t\"sharedKeys\": \"shared1,shared2\"\n" +
                "}";

        return data;
    }
}
```



**订阅（Subscribe）**

```java
package com.iiotos.attribute;

import com.iiotos.mqtt.EmqClient;
import com.iiotos.mqtt.MqttProperties;
import com.iiotos.mqtt.QosEnum;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;

@Component
public class Subscribe {


    @Autowired
    private EmqClient emqClient;

    @Autowired
    private MqttProperties properties;

    @PostConstruct
    public void init(){
        //连接服务端
        emqClient.connect(properties.getUsername(),properties.getPassword());
        //订阅一个主题
        emqClient.subscribe("v1/devices/me/attributes", QosEnum.QoS1);
    }

    @Scheduled(fixedRate = 2000)
    public void publish(){

        String data = getData();

        //emqClient.publish("v1/devices/me/attributes/request/1",data, QosEnum.QoS1,false);
    }

    private String getData(){

        String data = "{\n" +
                "\t\"clientKeys\": \"attribute1,attribute2\",\n" +
                "\t\"sharedKeys\": \"shared1,shared2\"\n" +
                "}";

        return data;

    }
}
```



### 2.上传客户端属性

**通过服务端发布客户端属性**

通过ThingsBoard服务端发布客户端属性必须PUBLISH消息到下面主题:

```shell
v1/devices/me/attributes
```



**new-attributes-values.json**

```json
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



![](/images/iot/device/device-attribute/a-4.png)

```sql
SELECT * from attribute_kv where attribute_key like 'attribute%'
```

![](/images/iot/device/device-attribute/a-5.png)



### 3.下载服务端属性

**通过服务端获取属性值**

通过ThingsBoard服务端获取**客户端属性**或**共享属性**必须PUBLISH消息到下面主题:

```shell
v1/devices/me/attributes/request/$request_id
```

其中**$request_id**表示整数的请求标识符。

在发送带有请求的PUBLISH消息之前客户端需要订阅。

```shell
v1/devices/me/attributes/response/+
```



```java
//订阅一个主题
emqClient.subscribe("v1/devices/me/attributes/response/+", QosEnum.QoS1);

//发布消息
emqClient.publish("v1/devices/me/attributes/request/1",data, QosEnum.QoS1,false);
```



**发布消息请求的属性**

```json
{
	"clientKeys": "attribute1,attribute2",
	"sharedKeys": "shared1,shared2"
}
```



```shell
消息发布完成,messageid=6,topics=[v1/devices/me/attributes/request/1]

订阅者订阅到了消息,topic=v1/devices/me/attributes/response/2,messageid=5,qos=1,
payload=

{
	"client": {
		"attribute1": "value1",
		"attribute2": true
	},
	"shared": {
		"shared1": "shared1value",
		"shared2": 1000
	}
}
```



**客户端属性**

![](/images/iot/device/device-attribute/a-6.png)



**共享属性**

![](/images/iot/device/device-attribute/a-7.png)



### 4.订阅共享属性

通过服务端订阅共享属性必须SUBSCRIBE消息到下面主题：

```shell
v1/devices/me/attributes
```

如果服务端组件（例如REST API或规则链）更改了共享属性时客户端就会收到对应属性值：

```shell
{"key1":"value1"}
```



```java
//订阅一个主题
emqClient.subscribe("v1/devices/me/attributes", QosEnum.QoS1);
```



```shell
订阅者订阅到了消息,topic=v1/devices/me/attributes,messageid=1,qos=1,
payload=

{
	"shared2": 10001
}
```



![](/images/iot/device/device-attribute/a-8.png)



