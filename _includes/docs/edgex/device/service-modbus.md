* TOC
{:toc}



## 一、Modbus 设备服务



### 1.概述

Modbus 设备服务的目的是将 Modbus 设备连接到 EdgeX。



### 2.协议属性

设备 Modbus 为 Modbus-TCP 和 Modbus-RTU 定义了以下协议属性。

- Modbus TCP 设备
  - 这些属性位于每个设备定义部分的`modbus-tcp`键下`protocols`。



| 属性        | 描述                                                   |
| :---------- | :----------------------------------------------------- |
| Address     | Modbus TCP 的 IP 地址或主机名                          |
| Port        | 用于通过 Modbus TCP 进行通信的端口                     |
| UnitID      | Modbus 站或从站标识符                                  |
| Timeout     | 连接或读取Device Modbus Service到device-modbus时的超时 |
| IdleTimeout | 关闭连接的空闲超时（秒）                               |

![](/images/edgex/device/service-modbus/modbus-1.png)



- Modbus RTU 设备
  - 这些属性位于每个设备定义部分的`modbus-rtu`键下`protocols`。



| 属性        | 描述                                                   |
| :---------- | :----------------------------------------------------- |
| Address     | Modbus TCP 的 IP 地址或主机名                          |
| UnitID      | Modbus 站或从站标识符                                  |
| BaudRate    | 串行设备的波特率，必须与使用相同地址的设备匹配         |
| DataBits    | 数据位数                                               |
| StopBits    | 停止位数                                               |
| Parity      | 奇偶校验值：N 表示无校验/E 表示偶校验/O 表示奇校验     |
| Timeout     | 连接或读取Device Modbus Service到device-modbus时的超时 |
| IdleTimeout | 关闭连接的空闲超时（秒）                               |

![](/images/edgex/device/service-modbus/modbus-2.png)



### 3.数据类型转换

在设备资源使用具有浮点刻度的整数数据类型的用例中，转换后可能会丢失精度。

例如，Modbus 设备将温度和湿度存储在 Int16 数据类型中，浮点数为 0.01。如果温度为26.53，则读取值为2653。但转换后，值为26。

为了避免这种情况，设备资源数据类型必须与值描述符数据类型不同。这是通过使用设备配置文件中的可选 `rawType`属性来定义从 Modbus 设备读取的二进制数据以及`valueType`指示用户想要接收的数据类型来实现的。

如果`rawType`属性存在，设备服务根据定义解析二进制数据，然后根据设备资源的定义`rawType`转换值。`valueType``properties`

以下设备配置文件摘录将 定义`rawType`为 Int16 并将 定义`valueType`为 Float32：

![](/images/edgex/device/service-modbus/modbus-3.png)



#### 3.1.读命令

读取命令执行如下：

1. 设备服务执行Read命令读取二进制数据
2. 二进制读取数据被解析为Int16数据类型
3. 整数值被转换为 Float32 值

![](/images/edgex/device/service-modbus/modbus-4.png)



#### 3.2.写命令

写入命令执行如下：

1. 设备服务将请求的 Float32 值转换为整数值
2. 整数值转换为二进制数据
3. 设备服务执行Write命令

![](/images/edgex/device/service-modbus/modbus-5.png)



#### 3.3.何时转换数据

在 16 位整数和浮点值之间缩放读数时，通常需要转换数据。

以下限制适用：

- `rawType`仅支持 Int16、Uint16 和 Int32 数据类型
- 对应的`valueType`必须是Float32或者Float64

如果为属性定义了不受支持的数据类型`rawType`，设备服务将引发类似于以下内容的异常：

```
Read command failed. Cmd:temperature err:the raw type Int64 is not supported
```



#### 3.4.支持的转换

支持的转换如下：

| From `rawType` | To `valueType` |
| :------------- | :------------- |
| Int16          | Float32        |
| Int16          | Float64        |
| Int32          | Float64        |
| Uint16         | Float32        |
| Uint16         | Float64        |





## 二、连接 Modbus 设备



```shell
# 官方文档

https://docs.edgexfoundry.org/3.1/microservices/device/services/device-modbus/GettingStarted/
```



### 1.docker-comepse

```shell
# 1.克隆 edgex-compose
$ git clone git@github.com:edgexfoundry/edgex-compose.git 
$ git clone https://github.com/edgexfoundry/edgex-compose.git
$ cd edgex-compose 
$ git checkout v3.1


# 2.生成 docker-compose.yml 文件（注意这包括 mqtt-broker）
$ cd compose-builder
$ make gen ds-modbus no-secty


# 3.检查生成的文件
$ ls | grep 'docker-compose.yml'
docker-compose.yml
```



```shell
[root@edgex mqtt-device]# git clone https://github.com/edgexfoundry/edgex-compose.git
Cloning into 'edgex-compose'...
remote: Enumerating objects: 4779, done.
remote: Counting objects: 100% (2916/2916), done.
remote: Compressing objects: 100% (173/173), done.
remote: Total 4779 (delta 2831), reused 2804 (delta 2741), pack-reused 1863
Receiving objects: 100% (4779/4779), 1.22 MiB | 450.00 KiB/s, done.
Resolving deltas: 100% (4042/4042), done.


[root@edgex mqtt-device]# ll
total 4
drwxr-xr-x. 6 root root 4096 Feb  1 04:10 edgex-compose


[root@edgex mqtt-device]# cd edgex-compose/
[root@edgex edgex-compose]# git checkout v3.1
Note: checking out 'v3.1'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b new_branch_name

HEAD is now at 488a3fe... Merge pull request #424 from lenny-intel/device-mqtt-secure-mode-napa


[root@edgex edgex-compose]# cd compose-builder/

[root@edgex compose-builder]# make gen ds-modbus no-secty
echo MQTT_VERBOSE=
MQTT_VERBOSE=
docker compose  -p edgex -f docker-compose-base.yml -f add-device-modbus.yml convert > docker-compose.yml
rm -rf ./gen_ext_compose


[root@edgex compose-builder]# ls | grep 'docker-compose.yml'
docker-compose.yml
```



### 2.设备配置文件

在本部分中，我们创建文件夹，其中包含部署自定义设备配置以与现有设备服务配合使用所需的文件：

```shell
[root@edgex custom-config]# pwd
/edgex/modbus-device-svc/custom-config

[root@edgex custom-config]# tree
.
├── device.config.yaml
└── temperature.profile.yml
```



**1.设备配置**

创建设备配置文件，命名为 `device.config.yaml`，如下所示：

```yaml
[root@edgex custom-config]# vim device.config.yaml

deviceList:
  name: "Modbus-TCP-Temperature-Sensor"
  profileName: "Ethernet-Temperature-Sensor"Ethernet-Temperature-Sensor
  description: "This device is a product for monitoring the temperature via the ethernet"
  labels: 
    - "temperature"
    - "modbus"
    - "TCP"
  protocols:
    modbus-tcp:
      Address: "192.168.3.4"
      Port: "502"
      UnitID: "1"
      Timeout: "5"
      IdleTimeout: "5"
  autoEvents:
    interval: "30s"
    onChange: false
    sourceName: "Temperature"
```



**2.设备配置文件**

DeviceProfile定义了设备的值和操作方法，可以是 Read 或 Write。

创建名为 的设备配置文件 `temperature.profile.yml`，其中包含以下内容：

```yaml
[root@edgex custom-config]# vim temperature.profile.yml 

name: "Ethernet-Temperature-Sensor"
manufacturer: "Audon Electronics"
model: "Temperature"
labels:
  - "Web"
  - "Modbus TCP"
  - "SNMP"
description: "The NANO_TEMP is a Ethernet Thermometer measuring from -55°C to 125°C with a web interface and Modbus TCP communications."

deviceResources:
  -
    name: "ThermostatL"
    isHidden: true
    description: "Lower alarm threshold of the temperature"
    attributes:
      { primaryTable: "HOLDING_REGISTERS", startingAddress: 0, rawType: "Int16" }
    properties:
      valueType: "Float32"
      readWrite: "RW"
      scale: 0.1
  -
    name: "ThermostatH"
    isHidden: true
    description: "Upper alarm threshold of the temperature"
    attributes:
      { primaryTable: "HOLDING_REGISTERS", startingAddress: 1, rawType: "Int16" }
    properties:
      valueType: "Float32"
      readWrite: "RW"
      scale: 0.1
  -
    name: "AlarmMode"
    isHidden: true
    description: "1 - OFF (disabled), 2 - Lower, 3 - Higher, 4 - Lower or Higher"
    attributes:
      { primaryTable: "HOLDING_REGISTERS", startingAddress: 2 }
    properties:
      valueType: "Int16"
      readWrite: "RW"
  -
    name: "Temperature"
    isHidden: false
    description: "Temperature x 10 (np. 10,5 st.C to 105)"
    attributes:
      { primaryTable: "HOLDING_REGISTERS", startingAddress: 4, rawType: "Int16" }
    properties:
      valueType: "Float32"
      readWrite: "R"
      scale: 0.1

deviceCommands:
  -
    name: "AlarmThreshold"
    readWrite: "RW"
    isHidden: false
    resourceOperations:
      - { deviceResource: "ThermostatL" }
      - { deviceResource: "ThermostatH" }
  -
    name: "AlarmMode"
    readWrite: "RW"
    isHidden: false
    resourceOperations:
      - { deviceResource: "AlarmMode", mappings: { "1":"OFF","2":"Lower","3":"Higher","4":"Lower or Higher"} }
```



**3.修改docker-compose.yml**

```
 device-modbus:
    ...
    environment:
      ...
      DEVICE_DEVICESDIR: /custom-config
      DEVICE_PROFILESDIR: /custom-config
    volumes:
    ...
    - /path/to/custom-config:/custom-config
```



```shell
 device-modbus:
    ...
    environment:
      ...
      DEVICE_DEVICESDIR: /custom-config
      DEVICE_PROFILESDIR: /custom-config
    volumes:
    ...
    - /edgex/modbus-device-svc/custom-config:/custom-config
```



### 3.Modbus 配置

Modbus 寄存器表

您可以在用户手册中找到可用的寄存器。

Modbus TCP – Holding Registers

| Address | Name            | R/W  | Description                                                  |
| :------ | :-------------- | :--- | :----------------------------------------------------------- |
| 4000    | ThermostatL     | R/W  | Lower alarm threshold                                        |
| 4001    | ThermostatH     | R/W  | Upper alarm threshold                                        |
| 4002    | Alarm mode      | R/W  | 1 - OFF (disabled), 2 - Lower, 3 - Higher, 4 - Lower or Higher |
| 4004    | Temperature x10 | R    | Temperature x 10 (np. 10,5 st.C to 105)                      |



device-modbus提供两种类型的协议，Modbus TCP和Modbus RTU，其定义如下

| protocol   | Name            | Protocol | Address     | Port | UnitID | BaudRate | DataBits | StopBits | Parity | Timeout | IdleTimeout |
| :--------- | :-------------- | :------- | :---------- | :--- | :----- | :------- | :------- | :------- | :----- | :------ | :---------- |
| Modbus TCP | Gateway address | TCP      | 10.211.55.6 | 502  | 1      |          |          |          |        | 5       | 5           |
| Modbus RTU | Gateway address | RTU      | /tmp/slave  | 502  | 2      | 19200    | 8        | 1        | N      | 5       | 5           |



### 4.启动 EdgeX Foundry

使用以下命令部署 EdgeX：

```shell
$ cd edgex-compose/compose-builder
$ docker compose pull
$ docker compose up -d


# 修改配置文件
替换IP地址 127.0.0.1 为 0.0.0.0
```

```shell
# docker compose pull

# docker compose -f docker-compose.yml -f docker-compose.override.yml up -d
```



![](/images/edgex/device/service-modbus/modbus-6.png)

![](/images/edgex/device/service-modbus/modbus-7.png)

![](/images/edgex/device/service-modbus/modbus-8.png)



### 5.Modbus 测试工具

![](/images/edgex/device/service-modbus/modbus-9.png)

![](/images/edgex/device/service-modbus/modbus-10.png)



### 6.访问 UI

#### 6.1. consul

```shell
# 访问地址
http://192.168.202.233:8500
```

![](/images/edgex/device/service-modbus/modbus-11.png)



#### 6.2. EdgeX Console

```shell
# 访问地址
http://192.168.202.233:4000/
```



![](/images/edgex/device/service-modbus/modbus-12.png)

![](/images/edgex/device/service-modbus/modbus-13.png)

![](/images/edgex/device/service-modbus/modbus-14.png)



### 7.测试

#### 7.1.命令

![](/images/edgex/device/service-modbus/modbus-15.png)

![](/images/edgex/device/service-modbus/modbus-16.png)



#### 7.2.事件

![](/images/edgex/device/service-modbus/modbus-17.png)

```json
{
	"apiVersion": "v3",
	"id": "84e3cb89-5f8c-4777-aa8f-e8574d9b0221",
	"deviceName": "Modbus-TCP-Temperature-Sensor",
	"profileName": "Ethernet-Temperature-Sensor",
	"sourceName": "Temperature",
	"origin": 1708368064375222000,
	"readings": [{
		"id": "9626ea3f-476e-49c3-86d1-12f6245aa39f",
		"origin": 1708368064375173600,
		"deviceName": "Modbus-TCP-Temperature-Sensor",
		"resourceName": "Temperature",
		"profileName": "Ethernet-Temperature-Sensor",
		"valueType": "Float32",
		"value": "0.000000e+00"
	}]
}
```



#### 7.3.读值

![](/images/edgex/device/service-modbus/modbus-18.png)

```json
{
	"id": "d56f2ba4-beb7-44c4-a6e9-3dad9cd87d86",
	"origin": 1708368214506509000,
	"deviceName": "Modbus-TCP-Temperature-Sensor",
	"resourceName": "Temperature",
	"profileName": "Ethernet-Temperature-Sensor",
	"valueType": "Float32",
	"value": "0.000000e+00"
}
```



