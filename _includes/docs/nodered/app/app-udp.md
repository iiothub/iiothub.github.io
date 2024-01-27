* TOC
{:toc}



## 一、概述



### 1.网络调试助手

<img src="/images/nodered/app/app-udp/udp-1.png" style="zoom:50%;" />



### 2.Node-RED

#### 2.1.udp  in

UDP输入节点。在`msg.payload`中生成Buffer，字符串或Base64编码的字符串。支持组播。

在`msg.ip`和`msg.port`中设置接收到的消息的IP地址和端口。

**注意：**在某些系统上，您可能需要root或管理员权限才能使用低于1024的端口或广播。

![](/images/nodered/app/app-udp/udp-2.png)



#### 2.2.udp  out

该节点将`msg.payload`发送到指定的UDP主机和端口。支持组播。

您也可以使用`msg.ip`和`msg.port`设置目标值，但是静态配置的值具有优先权。

如果选择广播，则将地址设置为本地广播IP地址。或者也可以尝试使用全局广播地址255.255.255.255。

**注意：**在某些系统上，您可能需要root或管理员权限才能使用低于1024的端口或广播。





## 二、udp in



### 1.udp

#### 1.1.配置流程

![](/images/nodered/app/app-udp/udp-3.png)

![](/images/nodered/app/app-udp/udp-4.png)



#### 1.2.调试助手

![](/images/nodered/app/app-udp/udp-5.png)



#### 1.3.测试

![](/images/nodered/app/app-udp/udp-6.png)

![](/images/nodered/app/app-udp/udp-7.png)





## 三、udp  out



### 1.udp

#### 1.1.配置流程

![](/images/nodered/app/app-udp/udp-8.png)

![](/images/nodered/app/app-udp/udp-9.png)



#### 1.2.配置助手

![](/images/nodered/app/app-udp/udp-10.png)



#### 1.3.测试

![](/images/nodered/app/app-udp/udp-11.png)

![](/images/nodered/app/app-udp/udp-12.png)

 

