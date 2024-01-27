* TOC
{:toc}



## 一、概述



### 1.RPC功能

ThingsBoard允许你从**服务端应用程序向设备发送远程RPC调用**，你也可以将命令发送到设备并接收命令执行的结果。

同样你可以执行来自设备的请求，在后端应用进行某些计算或服务端逻辑处理然后将结果反馈到设备。

Thinsboard RPC功能可以分为两种类型：设备发起的RPC调用和服务端发起的RPC调用，为了使用更熟悉的名称我们将源自设备的RPC调用命名为**客户端**RPC调用，将源自服务器的RPC调用命名为**服务端**RPC调用。

- **服务端RPC：**从服务器发rpc到设备，向设备发送命令并接收命令执行的结果
- **客户端RPC：**从设备发rpc到服务器，执行来自设备的请求，在后端应用一些计算或其他服务器端逻辑并将响应发送回设备



### 2.服务端RPC

服务端RPC功能将请求从**平台发送到设备**，并可选择将响应返回给平台。

服务端RPC调用的典型用例是各种远程控制：重启、打开/关闭、修改gpio状态、修改配置参数等。

**服务端RPC调用可以分为单向和双向：**

- 单向RPC请求直接发送请求，并且不对设备响应做任何处理。

  ![](/images/iot/device/device-rpc/rpc-1.svg)

- 双向RPC请求会发送到设备，并且超时期间内接收到来自设备的响应。

  ![](/images/iot/device/device-rpc/rpc-2.svg)

ThingsBoard3.3以前版本仅支持**轻量级**的RPC调用只能控制在30秒以内这是平台的REST API调用的默认超时，因为没有将其存储到数据库中服务器挂掉之后部件会向其它服务器发送相同请求，轻量级的RPC消耗资源较少这是因为它处理不调用任何输入/输出只会存储审计日志和规则引擎消息。

ThingsBoard3.3版本开始提供了[**持久化**](http://www.ithingsboard.com/docs/user-guide/rpc/#persistent-rpc)的RPC调用支持并且具有可配置的生命周期存储，如果设备长时间无法访问时持久化则非常有用通常在网络连接不佳或[省电模式](http://www.ithingsboard.com/docs/user-guide/psm) (PSM)情况下。



**RPC结构**

**RPC请求主体由多个字段组成：**

- **method** - 必须，RPC调用方法。例如：”getCurrentTime”或”getWeatherForecast”
- **params** - 必须，RPC调用参数；JSON字符串如果不需要则可以是”{}”
- **timeout** - 可选，RPC调用超时；默认值为10000（10秒）最小值为5000（5秒）
- **expirationTime** - 可选，RPC到期时间(以毫秒为单位，UTC 时区). 如果存在则覆盖**timeout**
- **persistent** - 可选，RPC持久化；参加[persistent]与[lightweight]的RPC默认值为”false”
- **retries** - 可选，RPC重试持久化次数；网络或设备出现故障时重新发送持久化的RPC次数
- **additionalInfo** - 可选，RPC附加信息；定义将添加[persistent RPC events]的元数据

**请求示例：**

```json
{
   "method": "setGPIO",
   "params": {
     "pin": 4,
     "value": 1
   },
  "timeout": 30000
}
```

**响应示例：**

```json
{
   "pin": 4,
   "value": 1,
   "changed": true
}
```



### 3.客户端RPC

客户端RPC功能可以将请求从**设备发送平台**并将响应返回组设备。

客户端RPC调用的典型用例：

- 灌溉系统通过平台获取天预报气服务进行灌溉
- 设备没有系统时钟通过平台请求当前
- 门禁读卡器向第三方安全系统发送请求决定开门方式

设备向平台发送一条消息由[规则引擎](http://www.ithingsboard.com/docs/user-guide/rule-engine-2-0/re-getting-started/)处理和使用设备属性、遥测或存储在平台中的数据进行计算或者调用外部系统处理消息之后将结果返回给设备。

![](/images/iot/device/device-rpc/rpc-3.svg)

客户端RPC请求由两个必须的字段组成：

- **method** - 表示json字符串格式的方法名
- **params** - 表示json字符串格式的对象参数列表

请求示例：

```json
{
   "method": "getCurrentTime",
   "params": {}
}
```

响应实例：内容可以是数据，字符串，JOSN

```shell
1631881236974
```



### 4.MQTT RPC API

#### 4.1.服务端RPC

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



#### 4.2.客户端RPC

将RPC命令发送到服务端必须PUBLISH消息发送到下面主题：

```shell
v1/devices/me/rpc/request/$request_id
```

**$request_id**表示请求的整型标识符服务端必须发布到下面主题：

```shell
v1/devices/me/rpc/response/$request_id
```



### 5.数据库

rpc

![](/images/iot/device/device-rpc/rpc-10.png)





## 二、MQTT操作RPC



### 1.环境准备

1. 创建测试设备 test-3
2. 创建测试工程 tb-rpc



**1.程序配置**

```yaml
mqtt:
  broker-url: tcp://192.168.202.188:1883
  client-id: emq-client-xxxxxx
  username: FO3JsWBgdukOr9pZjhab
  password:
```



### 2.服务端PRC

#### 2.1.RPC消息主题

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



#### 2.2.程序源码

**ServerRpc**

```java
@Component
public class ServerRpc {

    @Autowired
    private EmqClient emqClient;

    @Autowired
    private MqttProperties properties;

    @PostConstruct
    public void init(){
        //连接服务端
        emqClient.connect(properties.getUsername(),properties.getPassword());
        //订阅一个主题
        emqClient.subscribe("v1/devices/me/rpc/request/+", QosEnum.QoS1);
    }
    
}
```



**MessageCallback**

```java
@Component
public class MessageCallback implements MqttCallback {
    
    /**
     * 应用收到消息后触发的回调
     * @param topic
     * @param message
     * @throws Exception
     */
    @Override
    public void messageArrived(String topic, MqttMessage message) throws Exception {
        log.info("订阅者订阅到了消息,topic={},messageid={},qos={},payload={}",
                topic,
                message.getId(),
                message.getQos(),
                new String(message.getPayload()));

        // 订阅者订阅到了消息,topic=v1/devices/me/rpc/request/7,messageid=6,qos=1,payload={"method":"setValue","params":false}
        // 订阅后客户端会收到一条命令作为对相应主题的PUBLISH命令：
        // v1/devices/me/rpc/request/$request_id
        String[] buff = topic.split("/");
        String request_id = buff[buff.length-1];

        // 客户端PUBLISH下面主题进行响应：
        // v1/devices/me/rpc/response/$request_id
        emqClient.publish("v1/devices/me/rpc/response/" + request_id, "{}", QosEnum.QoS1,false);

    }
```



#### 2.3.测试

![](/images/iot/device/device-rpc/rpc-4.png)

![](/images/iot/device/device-rpc/rpc-5.png)



```shell
2023-08-20 16:35:34.249  INFO 21332 --- [emq-client-2222] com.iiotos.mqtt.MessageCallback          : 订阅者订阅到了消息,topic=v1/devices/me/rpc/request/15,messageid=1,qos=1,payload={"method":"setValue","params":false}

2023-08-20 16:35:34.252  INFO 21332 --- [emq-client-2222] com.iiotos.mqtt.MessageCallback          : 消息发布完成,messageid=2,topics=[v1/devices/me/rpc/response/15]

2023-08-20 16:35:41.108  INFO 21332 --- [emq-client-2222] com.iiotos.mqtt.MessageCallback          : 订阅者订阅到了消息,topic=v1/devices/me/rpc/request/16,messageid=2,qos=1,payload={"method":"setValue","params":true}

2023-08-20 16:35:41.109  INFO 21332 --- [emq-client-2222] com.iiotos.mqtt.MessageCallback          : 消息发布完成,messageid=3,topics=[v1/devices/me/rpc/response/16]
```



### 3.客户端RPC

#### 3.1.RPC消息主题

将RPC命令发送到服务端必须PUBLISH消息发送到下面主题：

```shell
v1/devices/me/rpc/request/$request_id
```

**$request_id**表示请求的整型标识符服务端必须发布到下面主题：

```shell
v1/devices/me/rpc/response/$request_id
```

请求参数

```json
{"method": "getServerValue", "params": ""}
```



#### 3.2.程序源码

ClientRpc

```java
package com.iiotos.rpc;
import com.iiotos.mqtt.EmqClient;
import com.iiotos.mqtt.MqttProperties;
import com.iiotos.mqtt.QosEnum;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import javax.annotation.PostConstruct;

@Component
public class ClientRpc {

    @Autowired
    private EmqClient emqClient;

    @Autowired
    private MqttProperties properties;

    @PostConstruct
    public void init(){
        //连接服务端
        emqClient.connect(properties.getUsername(),properties.getPassword());
        //订阅一个主题
        emqClient.subscribe("v1/devices/me/rpc/response/+", QosEnum.QoS1);

    }

    @Scheduled(fixedRate = 3000)
    public void publish(){

        String data = getData();

        emqClient.publish("v1/devices/me/rpc/request/1",data,QosEnum.QoS1,false);
    }

    private String getData(){

        String data = "{\n" +
                "\t\"method\": \"getServerValue\",\n" +
                "\t\"params\": \"\"\n" +
                "}";

        return data;
    }
}
```



#### 3.3.规则链

![](/images/iot/device/device-rpc/rpc-6.png)

![](/images/iot/device/device-rpc/rpc-7.png)

![](/images/iot/device/device-rpc/rpc-8.png)



#### 3.4.测试

![](/images/iot/device/device-rpc/rpc-9.png)

```shell
2023-08-20 17:46:46.216  INFO 20548 --- [emq-client-2222] com.iiotos.mqtt.MessageCallback          : 消息发布完成,messageid=7,topics=[v1/devices/me/rpc/request/1]

2023-08-20 17:46:46.269  INFO 20548 --- [emq-client-2222] com.iiotos.mqtt.MessageCallback          : 订阅者订阅到了消息,topic=v1/devices/me/rpc/response/1,messageid=6,qos=1,payload={"method":"getServerValue","params":"","result":"server receive rpc requuest!!!"}
```



