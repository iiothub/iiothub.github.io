* TOC
{:toc}



## 一、概述



### 1.测试工具

```shell
http://www.jsons.cn/websocket/

http://www.websocket-test.com/
```



![](/images/nodered/app/app-websocket/ws-13.png)



### 2.Node-RED

#### 2.1.websocket in

WebSocket输入节点。

默认情况下，从WebSocket接收的数据将位于`msg.payload`中。可以将套接字配置为期望格式正确的JSON字符串，在这种情况下，它将解析JSON并将结果对象作为整个消息发送。

![](/images/nodered/app/app-websocket/ws-1.png)



#### 2.2.websocket out

WebSocket输出节点。

默认情况下，`msg.payload`将通过WebSocket发送。可以将套接字配置为将整个`msg`对象编码为JSON字符串，然后通过WebSocket发送。

如果到达此节点的消息是从WebSocket In节点开始的，则该消息将发送回触发流程的客户端。否则，消息将广播给所有连接的客户端。

如果要广播从“WebSocket输入”节点开始的消息，则可以应该删除流中的`msg._session`属性。





## 二、websocket  in



### 1.服务端

#### 1.1.配置流程

![](/images/nodered/app/app-websocket/ws-2.png)

![](/images/nodered/app/app-websocket/ws-3.png)

![](/images/nodered/app/app-websocket/ws-4.png)

![](/images/nodered/app/app-websocket/ws-5.png)

![](/images/nodered/app/app-websocket/ws-6.png)



#### 1.2.测试工具

```shell
http://www.jsons.cn/websocket/

ws://192.168.202.168:1880/test
```

![](/images/nodered/app/app-websocket/ws-7.png)



#### 1.3.测试

![](/images/nodered/app/app-websocket/ws-8.png)

![](/images/nodered/app/app-websocket/ws-9.png)



方式二：

![](/images/nodered/app/app-websocket/ws-10.png)

![](/images/nodered/app/app-websocket/ws-11.png)

![](/images/nodered/app/app-websocket/ws-12.png)



### 2.客户端

#### 2.1.配置流程

![](/images/nodered/app/app-websocket/ws-14.png)

![](/images/nodered/app/app-websocket/ws-15.png)



#### 2.2.测试工具

![](/images/nodered/app/app-websocket/ws-16.png)



#### 2.3.测试

![](/images/nodered/app/app-websocket/ws-17.png)

![](/images/nodered/app/app-websocket/ws-18.png)





## 三、websocket  out



### 1.服务端

#### 1.1.配置流程

![](/images/nodered/app/app-websocket/ws-19.png)

![](/images/nodered/app/app-websocket/ws-20.png)



#### 1.2.测试工具

![](/images/nodered/app/app-websocket/ws-21.png)



#### 1.3.测试

![](/images/nodered/app/app-websocket/ws-22.png)

![](/images/nodered/app/app-websocket/ws-23.png)



### 2.客户端

#### 2.1.配置流程

![](/images/nodered/app/app-websocket/ws-28.png)

![](/images/nodered/app/app-websocket/ws-24.png)



#### 2.2.测试工具

![](/images/nodered/app/app-websocket/ws-25.png)



#### 2.3.测试

![](/images/nodered/app/app-websocket/ws-26.png)

![](/images/nodered/app/app-websocket/ws-27.png)



