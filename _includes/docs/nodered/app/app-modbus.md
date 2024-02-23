* TOC
{:toc}



## 一、概述



### 1.Modbus 通信协议

Modbus是一种串行[通信协议](https://so.csdn.net/so/search?q=通信协议&spm=1001.2101.3001.7020)，是Modicon公司（现在的施耐德电气 Schneider Electric）于1979年为使用可编程逻辑控制器（PLC）通信而发表。Modbus已经成为工业领域通信协议的业界标准（De facto），并且现在是工业电子设备之间常用的连接方式。
通俗的讲，Modbus的本质就是通过寄存器、线圈与其它设备交换数据。



**1.Modbus分类**

- Modbus TCP
- Modbus RTU
- Modbus ASCII



Modbus是一簇协议，包含RTU、TCP、ASCII，Modbus并没有规定物理层。标准的Modicon控制器使用RS232C实现串行的Modbus协议。ASCII与RTU协议规定了信息、数据的结构、命令和应答的方式，采用Master/Slave方式，即Master端发出数据请求信息，Slave端接收到正确信息后就可以发送数据到Master端以响应请求；Master端也可以直接发送信息修改Slave端的数据，实现双向读写。
Modbus协议会对数据数据进行校验，ASCII采用LRC校验，RTU采用16位CRC校验，TCP由于可靠传输无需校验。
这三者的具体实现也有会所不同，TCP与RTU方式差别较小，具体参照[详细](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.ni.com%2Fzh-cn%2Finnovations%2Fwhite-papers%2F14%2Fthe-modbus-protocol-in-depth.html)。
总的来说Modbus TCP/RTU/ASCII这三者是Modbus协议的具体实现。最显而易见的就是，TCP通过以太网传输，RTU通过RS232C或RS422/RS485传输。



**2.Modbus 功能码**

| 功能码    | 名词                    | 作用                                                         |
| :-------- | :---------------------- | :----------------------------------------------------------- |
| 01        | 读线圈状态              | 取得一组逻辑线圈的当前状态（ON/OFF）                         |
| 02        | 读取输入状态            | 取得一组开关输入的当前状态（ON/OFF）                         |
| 03        | 读取保持寄存器          | 在一个或多个保持寄存器中取得当前的二进制值                   |
| 04        | 读取输入寄存器          | 在一个或多个输入寄存器中取得当前的二进制值                   |
| 05        | 强置单线圈              | 强制一个逻辑线圈的通断状态                                   |
| 06        | 预置单寄存器            | 把具体二进制值装入一个保持寄存器                             |
| 07        | 读取异常状态            | 取得8个内部线圈的通断状态，这8个线圈的地址由控制器决定       |
| 08        | 回送诊断检验            | 把诊断检验报文送从机，以对通信处理进行评估                   |
| 09        | 编程 （只用于484）      | 使主机模拟编程器作用，修改PC从机逻辑                         |
| 10        | 控询（只用于484）       | 可使主机与一台正在执行长程序任务从机通信，探询该从机是够已完成其操作任务，仅在含有功能码9的报文发送后，本功能码才发送 |
| 11        | 读取事件计数            | 可使主机发出单询问，并随即判定操作是否成功，尤其是该命令或其他应答产生通信错误时 |
| 12        | 读取通信事件记录        | 可使主机检索每台从机的Modbus事务处理通信事件记录。如果某项事务处理完成，记录会给出相关错误。 |
| 13        | 编程（184/384/484/584） | 可使主机模拟编程器功能修改PC从机逻辑                         |
| 14        | 探询（184/384/484/584） | 可使主机与正在执行任务的从机通信，定期控询该从机是否已完成其程序操作，仅在含有功能码13的报文发送后，本功能才得发送 |
| 15        | 强置多线圈              | 强置一串连续逻辑线圈的通断                                   |
| 16        | 预置多寄存器            | 把具体的二进制值装入一串连续的保持寄存器                     |
| 17        | 报告从机标识            | 可使主机判断编址从机的类型及该从机运行指示灯的状态           |
| 18        | （884和Micro84）        | 可使主机模拟编程功能，修改PC的状态逻辑                       |
| 19        | 重置通信链路            | 发生非可修改错误后，使从机复位于已知状态，可重置顺序字节     |
| 20        | 读取通用参数（584L）    | 显示拓展存储器文件中的数据信息                               |
| 21        | 写入通用参数（584L）    | 把通用参数写入拓展存储文件，或修改之                         |
| 22 ~ 64   | 保留以备用户功能所用    | 留作用户功能的拓展编码                                       |
| 73 ~ 191  | 非法功能                |                                                              |
| 120 ~ 121 | 保留                    | 留作内部使用                                                 |
| 128 ~ 255 | 保留                    | 用于异常应答                                                 |

**备注：**
常用的为1、2、3、4、15、16. 这6个功能即可实现对下位机数字量、模拟量的读写操作。



### 2.Modbus Slave

Modbus Slave 工具模拟 Modbus server.

![](/images/nodered/app/app-modbus/modbus-1.png)

![](/images/nodered/app/app-modbus/modbus-2.png)



### 3.安装 Modbus 节点

```shell
# node-red-contrib-modbus
https://flows.nodered.org/node/node-red-contrib-modbus
```



![](/images/nodered/app/app-modbus/modbus-3.png)

![](/images/nodered/app/app-modbus/modbus-4.png)

![](/images/nodered/app/app-modbus/modbus-5.png)



## 二、Node-RED



### 1.配置说明

**在 Node-RED 上模拟 Modbus 通讯**

Modbus Slvave 在本地 192.168.3.4 上以 Modbus-TCP 的方式模拟了一个 Modbus Server。
其它设备可以直接通过 Modbus-TCP 直接连接该服务器。



![](/images/nodered/app/app-modbus/modbus-17.png)

![](/images/nodered/app/app-modbus/modbus-18.png)



### 2.配置 Modbus Server

![](/images/nodered/app/app-modbus/modbus-6.png)



### 3.配置 Modbus Slave

![](/images/nodered/app/app-modbus/modbus-2.png)



### 4.Modbus 读数据



![](/images/nodered/app/app-modbus/modbus-11.png)

![](/images/nodered/app/app-modbus/modbus-7.png)

![](/images/nodered/app/app-modbus/modbus-8.png)

![](/images/nodered/app/app-modbus/modbus-9.png)

![](/images/nodered/app/app-modbus/modbus-10.png)



### 5.Modbus 写数据

![](/images/nodered/app/app-modbus/modbus-12.png)

![](/images/nodered/app/app-modbus/modbus-13.png)

![](/images/nodered/app/app-modbus/modbus-14.png)

![](/images/nodered/app/app-modbus/modbus-15.png)

![](/images/nodered/app/app-modbus/modbus-16.png)





