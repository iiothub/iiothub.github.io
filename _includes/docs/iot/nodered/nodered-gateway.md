* TOC
{:toc}



## 一、环境准备



### 1.测试环境

```shell
# ThingsBoard
http://192.168.202.188:8080/login

# Node-RED
http://192.168.202.188:1880

设备名称：Node-RED-gateway
访问令牌：y0b9w7P2dSIR2AKn7Dl4

Nodered流程: TB-gateway
```



### 2.创建网关

<img src="/images/iot/nodered/nodered-gateway/gateway-1.png" style="zoom:50%;" />



### 3.创建Node-RED流程

![](/images/iot/nodered/nodered-gateway/gateway-2.png)





## 二、设备链接



### 1.Connect

```shell
# 发布主题
v1/gateway/connect

{"device":"Device A"}
```



![](/images/iot/nodered/nodered-gateway/gateway-3.png)

![](/images/iot/nodered/nodered-gateway/gateway-4.png)

![](/images/iot/nodered/nodered-gateway/gateway-5.png)

![](/images/iot/nodered/nodered-gateway/gateway-6.png)

![](/images/iot/nodered/nodered-gateway/gateway-7.png)

![](/images/iot/nodered/nodered-gateway/gateway-8.png)



### 2.Disconnect

```shell
v1/gateway/disconnect

{"device":"Device A"}
```



![](/images/iot/nodered/nodered-gateway/gateway-8.png)

![](/images/iot/nodered/nodered-gateway/gateway-9.png)





## 三、属性



### 1.发布属性到服务端

**将客户端设备属性发布到ThingsBoard服务器节点。**

```shell
Topic: v1/gateway/attributes

Message: {"Device A":{"attribute1":"value1", "attribute2": 42}, "Device B":{"attribute1":"value1", "attribute2": 42}}
```



![](/images/iot/nodered/nodered-gateway/gateway-10.png)

![](/images/iot/nodered/nodered-gateway/gateway-11.png)

![](/images/iot/nodered/nodered-gateway/gateway-12.png)

![](/images/iot/nodered/nodered-gateway/gateway-13.png)



### 2.请求属性从服务端

**向ThingsBoard服务器节点请求客户端或共享设备属性。**

请向以下主题发送PUBLISH消息：

```shell
Topic: v1/gateway/attributes/request
Message: {"id": $request_id, "device": "Device A", "client": true, "key": "attribute1"}
```

其中$request_id是您的整数请求标识符，设备A是您的设备名称，client标识客户端或共享属性范围，key是属性键。

在发送PUBLISH消息和请求之前，客户端需要订阅

```shell
Topic: v1/gateway/attributes/response
```

并期望具有以下格式的结果的消息：

```shell
Message: {"id": $request_id, "device": "Device A", "value": "value1"}
```



![](/images/iot/nodered/nodered-gateway/gateway-14.png)

![](/images/iot/nodered/nodered-gateway/gateway-15.png)

![](/images/iot/nodered/nodered-gateway/gateway-16.png)

![](/images/iot/nodered/nodered-gateway/gateway-17.png)

![](/images/iot/nodered/nodered-gateway/gateway-18.png)



### 3.订阅属性更新从服务端

订阅共享设备属性更改，请向以下主题发送subscribe消息：

```shell
v1/gateway/attributes
```

并期望具有以下格式的结果的消息：

```shell
Message: {"device": "Device A", "data": {"attribute1": "value1", "attribute2": 42}}
```



![](/images/iot/nodered/nodered-gateway/gateway-19.png)

![](/images/iot/nodered/nodered-gateway/gateway-20.png)

![](/images/iot/nodered/nodered-gateway/gateway-21.png)

![](/images/iot/nodered/nodered-gateway/gateway-22.png)





## 四、遥测

将设备遥测发布到ThingsBoard服务器节点，请向以下主题发送publish消息：

```shell
Topic: v1/gateway/telemetry
```



**Message:**

```json
{
  "Device A": [
    {
      "ts": 1483228800000,
      "values": {
        "temperature": 42,
        "humidity": 80
      }
    },
    {
      "ts": 1483228801000,
      "values": {
        "temperature": 43,
        "humidity": 82
      }
    }
  ],
  "Device B": [
    {
      "ts": 1483228800000,
      "values": {
        "temperature": 42,
        "humidity": 80
      }
    }
  ]
}
```



![](/images/iot/nodered/nodered-gateway/gateway-23.png)

![](/images/iot/nodered/nodered-gateway/gateway-24.png)

![](/images/iot/nodered/nodered-gateway/gateway-25.png)

![](/images/iot/nodered/nodered-gateway/gateway-26.png)





## 五、服务端RPC

从服务器订阅RPC命令，请向以下主题发送subscribe消息：

```shell
v1/gateway/rpc
```

并期望具有以下格式的单个命令的消息：

```shell
{"device": "Device A", "data": {"id": $request_id, "method": "toggle_gpio", "params": {"pin":1}}}
```

一旦设备处理了命令，网关就可以使用以下格式发回命令：

```shell
{"device": "Device A", "id": $request_id, "data": {"success": true}}
```



![](/images/iot/nodered/nodered-gateway/gateway-27.png)

![](/images/iot/nodered/nodered-gateway/gateway-28.png)

![](/images/iot/nodered/nodered-gateway/gateway-29.png)





## 六、总结



![](/images/iot/nodered/nodered-gateway/gateway-30.png)



