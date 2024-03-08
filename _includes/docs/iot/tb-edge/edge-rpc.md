* TOC
{:toc}



## 一、RPC 功能



### 1.服务端 RPC

**服务端 RPC 分：单向 RPC、双向 RPC。**



**服务端 RPC 调用可以分为单向和双向：**

- 单向 RPC 请求直接发送请求，并且不对设备响应做任何处理。

  ![](/images/iot/tb-edge/edge-rpc/rpc-1.svg)

- 双向 RPC 请求会发送到设备，并且超时期间内接收到来自设备的响应。

  ![](/images/iot/tb-edge/edge-rpc/rpc-2.svg)



### 2.客户端 RPC

**客户端 RPC 从设备端发送到平台**

![](/images/iot/tb-edge/edge-rpc/rpc-3.svg)



### 3.MQTT RPC API

#### 3.1.服务端RPC

客户端订阅服务端RPC命令必须SUBSCRIBE消息发送下面主题：

```
v1/devices/me/rpc/request/+
```

订阅后客户端会收到一条命令作为对相应主题的PUBLISH命令：

```
v1/devices/me/rpc/request/$request_id
```

**$request_id**表示请求的整型标识符。

客户端PUBLISH下面主题进行响应：

```
v1/devices/me/rpc/response/$request_id
```



#### 3.2.客户端RPC

将RPC命令发送到服务端必须PUBLISH消息发送到下面主题：

```
v1/devices/me/rpc/request/$request_id
```

**$request_id**表示请求的整型标识符服务端必须发布到下面主题：

```
v1/devices/me/rpc/response/$request_id
```





## 二、设备控制



### 1.环境准备

1. 创建测试设备 edge-device
2. 创建测试工程 tb-rpc



**1.程序配置**

```yaml
mqtt:
  broker-url: tcp://192.168.202.166:1883
  client-id: emq-client-rpc
  username: lMrdczEw1rJHhBejzumZ
  password:
```



### 2.创建设备

在 ThingsBoard 服务端创建设备配置 test-edge

在 Edge 端创建设备 edge-device

![](/images/iot/tb-edge/edge-rpc/rpc-8.png)

![](/images/iot/tb-edge/edge-rpc/rpc-9.png)

![](/images/iot/tb-edge/edge-rpc/rpc-10.png)



在服务端查看设备

![](/images/iot/tb-edge/edge-rpc/rpc-11.png)



```shell
# 访问令牌
lMrdczEw1rJHhBejzumZ
```





### 3.服务端PRC

#### 3.1.RPC消息主题

客户端订阅服务端RPC命令必须SUBSCRIBE消息发送下面主题：

```
v1/devices/me/rpc/request/+
```

订阅后客户端会收到一条命令作为对相应主题的PUBLISH命令：

```
v1/devices/me/rpc/request/$request_id
```

**$request_id**表示请求的整型标识符。

客户端PUBLISH下面主题进行响应：

```
v1/devices/me/rpc/response/$request_id
```



#### 3.2.程序源码

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



#### 3.3.创建仪表板

- 在服务端创建仪表板

![](/images/iot/tb-edge/edge-rpc/rpc-4.png)

![](/images/iot/tb-edge/edge-rpc/rpc-5.png)

![](/images/iot/tb-edge/edge-rpc/rpc-6.png)

![](/images/iot/tb-edge/edge-rpc/rpc-7.png)



#### 3.4.边缘分配仪表板

- 在服务端给 Edge 分配仪表板

![](/images/iot/tb-edge/edge-rpc/rpc-12.png)

![](/images/iot/tb-edge/edge-rpc/rpc-13.png)

![](/images/iot/tb-edge/edge-rpc/rpc-14.png)



- 在 Edge 端查看仪表板

![](/images/iot/tb-edge/edge-rpc/rpc-15.png)

![](/images/iot/tb-edge/edge-rpc/rpc-16.png)



#### 3.5.测试

- 在 Edge 端发送 RPC 命令

![](/images/iot/tb-edge/edge-rpc/rpc-17.png)

![](/images/iot/tb-edge/edge-rpc/rpc-18.png)



```shell
2023-08-20 16:35:34.249  INFO 21332 --- [emq-client-2222] com.iiotos.mqtt.MessageCallback          : 订阅者订阅到了消息,topic=v1/devices/me/rpc/request/15,messageid=1,qos=1,payload={"method":"setValue","params":false}

2023-08-20 16:35:34.252  INFO 21332 --- [emq-client-2222] com.iiotos.mqtt.MessageCallback          : 消息发布完成,messageid=2,topics=[v1/devices/me/rpc/response/15]
```



### 4.客户端RPC

#### 4.1.RPC消息主题

将RPC命令发送到服务端必须PUBLISH消息发送到下面主题：

```
v1/devices/me/rpc/request/$request_id
```

**$request_id**表示请求的整型标识符服务端必须发布到下面主题：

```
v1/devices/me/rpc/response/$request_id
```

请求参数

```json
{"method": "getServerValue", "params": ""}
```



#### 4.2.程序源码

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



#### 4.3.规则链

- 在服务端创建规则链

![](/images/iot/tb-edge/edge-rpc/rpc-19.png)

![](/images/iot/tb-edge/edge-rpc/rpc-20.png)

```shell
msg.result='server receive rpc requuest!!!'
return { msg: msg, metadata: metadata, msgType: msgType };
```



![](/images/iot/tb-edge/edge-rpc/rpc-21.png)



- 在 Edge 端 查看规则链

![](/images/iot/tb-edge/edge-rpc/rpc-22.png)



#### 4.4.测试

![](/images/iot/tb-edge/edge-rpc/rpc-23.png)



```shell
2023-08-20 17:46:46.216  INFO 20548 --- [emq-client-2222] com.iiotos.mqtt.MessageCallback          : 消息发布完成,messageid=7,topics=[v1/devices/me/rpc/request/1]


2023-08-20 17:46:46.269  INFO 20548 --- [emq-client-2222] com.iiotos.mqtt.MessageCallback          : 订阅者订阅到了消息,topic=v1/devices/me/rpc/response/1,messageid=6,qos=1,payload={"method":"getServerValue","params":"","result":"server receive rpc requuest!!!"}
```



