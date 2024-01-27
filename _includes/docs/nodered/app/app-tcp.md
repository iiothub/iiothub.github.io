* TOC
{:toc}



## 一、概述



### 1.网络调试助手

<img src="/images/nodered/app/app-tcp/tcp-1.png" style="zoom:50%;" />



### 2.Node-RED

#### 2.1.tcp  in

提供TCP输入选择。可以连接到远程TCP端口，或接受传入连接。

**注意：**在某些系统上，您可能需要root或管理员权限来访问低于1024的端口。

![](/images/nodered/app/app-tcp/tcp-2.png)



#### 2.2.tcp  out

提供TCP输出的选择。可以连接到远程TCP端口，接受传入的连接，或回复从TCP In节点收到的消息。

仅发送`msg.payload`。

如果`msg.payload`是包含二进制数据的Base64编码的字符串，则Base64解码选项将导致它在发送之前先转换回二进制。

如果不存在`msg._session`，则有效负载将发送到**所有**连接的客户端。

**注意：**在某些系统上，您可能需要root或管理员权限来访问低于1024的端口。

![](/images/nodered/app/app-tcp/tcp-3.png)





## 二、tcp  in



**注意：重启容器，否则端口不生效**



### 1.服务端

#### 1.1.配置流程

![](/images/nodered/app/app-tcp/tcp-4.png)

![](/images/nodered/app/app-tcp/tcp-5.png)

![](/images/nodered/app/app-tcp/tcp-6.png)



#### 1.2.调试助手

![](/images/nodered/app/app-tcp/tcp-7.png)



#### 1.3.测试

![](/images/nodered/app/app-tcp/tcp-8.png)

![](/images/nodered/app/app-tcp/tcp-16.png)



### 2.客户端

#### 2.1.配置流程

![](/images/nodered/app/app-tcp/tcp-9.png)

![](/images/nodered/app/app-tcp/tcp-12.png)

![](/images/nodered/app/app-tcp/tcp-13.png)



#### 2.2.调试助手

<img src="/images/nodered/app/app-tcp/tcp-11.png" style="zoom:50%;" />



#### 2.3.测试

![](/images/nodered/app/app-tcp/tcp-14.png)

![](/images/nodered/app/app-tcp/tcp-15.png)





## 三、tcp  out



### 1.服务端

#### 1.1.配置流程

![](/images/nodered/app/app-tcp/tcp-17.png)

![](/images/nodered/app/app-tcp/tcp-18.png)



#### 1.2.调试助手

![](/images/nodered/app/app-tcp/tcp-19.png)



#### 1.3.测试

![](/images/nodered/app/app-tcp/tcp-20.png)

![](/images/nodered/app/app-tcp/tcp-21.png)



### 2.客户端

#### 2.1.配置流程

![](/images/nodered/app/app-tcp/tcp-22.png)

![](/images/nodered/app/app-tcp/tcp-23.png)



#### 2.2.调试助手

![](/images/nodered/app/app-tcp/tcp-24.png)



#### 2.3.测试

![](/images/nodered/app/app-tcp/tcp-25.png)

![](/images/nodered/app/app-tcp/tcp-26.png)



### 3.TCP响应

![](/images/nodered/app/app-tcp/tcp-27.png)



