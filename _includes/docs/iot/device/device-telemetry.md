* TOC
{:toc}



## 一、概述



### 1.遥测数据

ThingsBoard提供了一组与时序列数据相关的功能：

- **采集** 基于各种[协议和集成](http://www.ithingsboard.com/docs/getting-started-guides/connectivity/)采集数据
- **存储** 基于SQL(PostgreSQL)或NoSQL(Cassandra或Timescale)数据库存储时间序列数据
- **查询** 基于API查询最新时序数据或指定时间范围内的所有数据
- **订阅** 基于[WebSockets](http://www.ithingsboard.com/docs/user-guide/telemetry/#websocket-api)订阅数据进行可视化或实时分析
- **可视化** 基于[仪表板](http://www.ithingsboard.com/docs/user-guide/dashboards/)和部件可视化时序数据
- **过滤和分析** 基于[规则引擎](http://www.ithingsboard.com/docs/user-guide/rule-engine-2-0/re-getting-started/)过滤和分析数据
- **生成** 基于采集数据产生[警报](http://www.ithingsboard.com/docs/user-guide/alarms/)
- **转发** 基于[External](http://www.ithingsboard.com/docs/user-guide/rule-engine-2-0/external-nodes/)规则节点将数据发送到外部系统（例如：Kafka或RabbitMQ规则节点）



### 2.数据点

ThingsBoard 将遥测（时序）数据视为带时间戳的键值对，将一个带时间戳的键值对称为**数据点**，键值对的数据结构具有简单灵活的特点可以无缝与物联网设备集；key始终是一个字符串而值可以是string、boolean、double、integer或者JSON。

以下示例使用内部数据格式设备本身可以使用各种**协议和数据格式**上传数据，更多详细信息请参见[时间序列数据上传API](http://www.ithingsboard.com/docs/user-guide/telemetry/#time-series-data-upload-api) 。

以下数据包含5个数据点： 温度 (double), 湿度 (integer), hvac启用(boolean), hvac状态(string)和配置(JSON)：

```shell
{
 "temperature": 42.2, 
 "humidity": 70,
 "hvacEnabled": true,
 "hvacState": "IDLE",
 "configuration": {
    "someNumber": 42,
    "someArray": [1,2,3],
    "someNestedObject": {"key": "value"}
 }
}
```

你可能会注意到上面列出的JSON没有时间戳在这种情况下ThingsBoard使用当前服务器时间戳但是你可以在消息中包含时间戳信息。

请参见下面的例：

```shell
{
  "ts": 1527863043000,
  "values": {
    "temperature": 42.2,
    "humidity": 70
  }
}
```



### 3.时序数据API

你可以使用内置的传输协议：

- [MQTT](http://www.ithingsboard.com/docs/reference/mqtt-api/#telemetry-upload-api)
- [CoAP](http://www.ithingsboard.com/docs/reference/coap-api/#telemetry-upload-api)
- [HTTP](http://www.ithingsboard.com/docs/reference/http-api/#telemetry-upload-api)
- [LwM2M](http://www.ithingsboard.com/docs/reference/lwm2m-api/#telemetry-upload-api)

上面的数协议都支持JSON、Protobuf或自己的数据格式, 对于其他协议请查看[“如何连接设备”](http://www.ithingsboard.com/docs/getting-started-guides/connectivity/)指南。



### 4.MQTT协议

#### 4.1.MQTT基础

[MQTT](https://en.wikipedia.org/wiki/MQTT)是一种轻量级的发布-订阅消息传递协议，它可能最适合各种物联网设备。

你可以在[此处](http://mqtt.org/)找到有关MQTT的更多信息，ThingsBoard服务器支持QoS级别0（最多一次）和QoS级别1（至少一次）以及一组预定义主题的MQTT代理。



**客户端**

你可以在网上找到大量的MQTT客户端库，本文中的示例将基于Mosquitto和MQTT.js您可以使用我们的[Hello World](http://www.ithingsboard.com/docs/getting-started-guides/helloworld/)指南中的说明。



**MQTT连接**

我们将在本文中使用令牌凭据对进行设备访问，这些凭证稍后将称为**$ACCESS_TOKEN**应用程序需要发送用户名包含**$ACCESS_TOKEN**的MQTT CONNECT消息。

连接状态码说明：

- **0x00 连接成功** - 成功连接
- **0x04 连接失败** - 用户名或密码错误。
- **0x05 连接未授权** - -用户名包含无效的 **$ACCESS_TOKEN**。



**Key-value格式**

ThingsBoard支持以JSON格式的key-value字符串值可以是string、bool、float、long或者二进制格式的序列化字符串；有关更多详细信息请参见协议[自定义](http://www.ithingsboard.com/docs/reference/mqtt-api/#protocol-customization)。

```shell
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



#### 4.2.遥测上传API

发布遥测数据到ThingsBoard服务端必须PUBLISH消息发送到下面主题：

```shell
v1/devices/me/telemetry
```

支持的最简单的数据格式是：

```shell
{"key1":"value1", "key2":"value2"}
```

或者

```shell
[{"key1":"value1"}, {"key2":"value2"}]
```

**请注意** 在这种情况下服务端时间戳将分配给上传的数据!

如果您的设备能够获得客户端时间戳，则可以使用以下格式：

```shell
{"ts":1451649600512, "values":{"key1":"value1", "key2":"value2"}}
```

在上面的示例中我们假设“1451649600512”是具有毫秒精度的[Unix时间戳](https://en.wikipedia.org/wiki/Unix_time)。
例如：值’1451649600512’对应于’2016年1月1日 星期五 12：00：00.512 GMT’



#### 4.3.遥测数据格式

**1.telemetry-data-as-object.json**

```json
{
	"ts": 1451649600512,
	"values": {
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
}
```



**2.telemetry-data-with-ts.json**

```json
{
  "stringKey": "value1",
  "booleanKey": true,
  "doubleKey": 42.0,
  "longKey": 73,
  "jsonKey": {
    "someNumber": 42,
    "someArray": [1,2,3],
    "someNestedObject": {"key": "value"}
  }
}
```



**3.telemetry-data-as-array.json**

```json
[{"key1":"value1"}, {"key2":true}]
```



### 5.时序数据表

#### 5.1.时序数据表

- ts_kv：时序数据历史数据表
- ts_kv_dictionary：键值映射表
- ts_kv_latest：最新时序数据



![](/images/iot/device/device-telemetry/t-3.png)

**1.ts_kv_dictionary**

![](/images/iot/device/device-telemetry/t-4.png)



**2.ts_kv**

![](/images/iot/device/device-telemetry/t-5.png)



**3.ts_kv_latest**

![](/images/iot/device/device-telemetry/t-6.png)



#### 5.2.数据表关系

- ts_kv_dictionary：定义key和key_id的关系
- 通过key_id到ts_kv表查询历史数据
- 通过key_id到ts_kv_latest表查询最新数据





## 二、MQTT上传遥测数据



### 1.环境准备

1. 创建测试设备 test-3
2. 创建测试工程 tb-telemetry



**1.程序配置**

```yaml
mqtt:
  broker-url: tcp://192.168.202.188:1883
  client-id: emq-client-xxxxxx
  username: FO3JsWBgdukOr9pZjhab
  password:
```



**2.程序源码**

```java
package com.iiotos.telemetry;

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

        String data = getData(1);

        emqClient.publish("v1/devices/me/telemetry",data,
                QosEnum.QoS1,false);
    }

    private String getData(Integer type){

        if (type == 1) {
            // 携带时间戳
            String data = "{\n" +
                    "\t\"ts\": 1451649600512,\n" +
                    "\t\"values\": {\n" +
                    "\t\t\"stringKey\": \"value1\",\n" +
                    "\t\t\"booleanKey\": true,\n" +
                    "\t\t\"doubleKey\": 42.0,\n" +
                    "\t\t\"longKey\": 73,\n" +
                    "\t\t\"jsonKey\": {\n" +
                    "\t\t\t\"someNumber\": 42,\n" +
                    "\t\t\t\"someArray\": [1, 2, 3],\n" +
                    "\t\t\t\"someNestedObject\": {\n" +
                    "\t\t\t\t\"key\": \"value\"\n" +
                    "\t\t\t}\n" +
                    "\t\t}\n" +
                    "\t}\n" +
                    "}";

            return data;
            
        } else if (type == 2) {
            // 不携带时间戳
            String data = "{\n" +
                    "  \"stringKey\": \"value1\",\n" +
                    "  \"booleanKey\": true,\n" +
                    "  \"doubleKey\": 42.0,\n" +
                    "  \"longKey\": 73,\n" +
                    "  \"jsonKey\": {\n" +
                    "    \"someNumber\": 42,\n" +
                    "    \"someArray\": [1,2,3],\n" +
                    "    \"someNestedObject\": {\"key\": \"value\"}\n" +
                    "  }\n" +
                    "}";

            return data;

        }else {
            // 数组
            String data = "[{\"key1\":\"value1\"}, {\"key2\":true}]";
            return data;
        }

    }
}
```



### 2.测试

#### 2.1.携带时间戳数据

**1.telemetry-data-as-object.json**

```json
{
	"ts": 1451649600512,
	"values": {
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
}
```



![](/images/iot/device/device-telemetry/t-1.png)

![](/images/iot/device/device-telemetry/t-2.png)

**历史数据相同时间戳覆盖**

```sql
SELECT * from ts_kv where key > 56;
```

![](/images/iot/device/device-telemetry/t-7.png)



```sql
SELECT * from ts_kv_latest where key > 56;
```

![](/images/iot/device/device-telemetry/t-8.png)



#### 2.2.不携带时间戳数据

**2.telemetry-data-with-ts.json**

```json
{
  "stringKey": "value1",
  "booleanKey": true,
  "doubleKey": 42.0,
  "longKey": 73,
  "jsonKey": {
    "someNumber": 42,
    "someArray": [1,2,3],
    "someNestedObject": {"key": "value"}
  }
}
```



![](/images/iot/device/device-telemetry/t-9.png)



**历史数据**

```sql
SELECT * from ts_kv where key > 56;
```

![](/images/iot/device/device-telemetry/t-10.png)



```sql
SELECT * from ts_kv_latest where key > 56;
```

![](/images/iot/device/device-telemetry/t-8.png)



#### 2.3.数组数据

**3.telemetry-data-as-array.json**

```json
[{"key1":"value1"}, {"key2":true}]
```



![](/images/iot/device/device-telemetry/t-11.png)



