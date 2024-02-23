* TOC
{:toc}



## 一、概述



### 1.系统设计说明

EdgeX 连接物联网设备，EdgeX 作为边缘计算节点接入 ThingsBoard 平台，Node-RED 实现 EdgeX 与 ThingsBoard 之间的协议转换 ， 实现开源物联网平台的边缘计算方案。



**系统设计方案：**

- EdgeX Foundry 接入 Modbus 设备
- Modbus 设备使用 Modbus Slave 工具模拟
- EdgeX Foundry 的设备数据通过 MQTT 方式导出到外部系统
- EMQX 作为 EdgeX Foundry  导出数据的外部 mqtt-broker
- Node-RED 从 EMQX 订阅 EdgeX Foundry 的设备数据
- Node-RED 进行协议转换后，上报 ThingsBoard 平台



### 2.docker-comepse

```shell
# 1.克隆 edgex-compose
$ git clone https://github.com/edgexfoundry/edgex-compose.git
$ cd edgex-compose 
$ git checkout v3.1


# 2.生成 docker-compose.yml 文件（注意这包括 mqtt-broker）
$ cd compose-builder
$ make gen ds-modbus asc-mqtt no-secty


# 3.检查生成的文件
$ ls | grep 'docker-compose.yml'
docker-compose.yml
```



### 3.Modbus Slave

![](/images/edgex/app/link-thingsboard/device-1.png)

![](/images/edgex/app/link-thingsboard/device-2.png)



### 4.安装 EMQX

```shell
$ docker run -d --name emqx -p 18084:18083 -p 1884:1883 emqx/emqx:4.4.17
```



```shell
[root@edgex custom-config]# docker run -d --name emqx -p 18084:18083 -p 1884:1883 emqx/emqx:4.4.17
Unable to find image 'emqx/emqx:4.4.17' locally
4.4.17: Pulling from emqx/emqx
3d2430473443: Pull complete 
9affe486a1de: Pull complete 
59289c77776e: Pull complete 
7f4b43e76fd0: Pull complete 
a73c64d8e555: Pull complete 
4f4fb700ef54: Pull complete 
ff83ca607058: Pull complete 
7787af4360dd: Pull complete 
Digest: sha256:6b81ca26a7f7243e46186b35ac9f1580b497cf30fd09064ca7ee4875934b66bf
Status: Downloaded newer image for emqx/emqx:4.4.17
13a37d777b15408802ca87da2dd296a4359767dc4c224d72dab93bbf4b051e33
```



**访问EMQX**

```shell
# 访问地址

http://192.168.202.233:18084
账号：admin
初始密码：public
修改密码：1qaz2wsx
```



![](/images/edgex/app/link-thingsboard/device-3.png)



### 5.安装 Node-RED

```shell
# 安装部署

# docker run -d --network host --restart=always -v node_red_data:/data --name nodered nodered/node-red:latest
```

```shell
# 访问地址
http://192.168.202.233:1880
```



![](/images/edgex/app/link-thingsboard/device-4.png)



### 6.安装 ThingsBoard

ThingsBoard 的安装部署本文档就省略了，可以从下面的网址参考安装。

```shell
# 安装 ThingsBoard 参考

# ThingsBoard 单机部署
https://iothub.org.cn/docs/iot/deploy/deploy-single/

# ThingsBoard 集群部署
https://iothub.org.cn/docs/iot/deploy/deploy-cluster/
```



![](/images/edgex/app/link-thingsboard/device-5.png)





## 二、EdgeX Foundry 连接设备



### 1.docker-comepse

```shell
# 1.克隆 edgex-compose
$ git clone https://github.com/edgexfoundry/edgex-compose.git
$ cd edgex-compose 
$ git checkout v3.1


# 2.生成 docker-compose.yml 文件（注意这包括 mqtt-broker）
$ cd compose-builder
$ make gen ds-modbus asc-mqtt no-secty


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

[root@edgex compose-builder]# make gen ds-modbus asc-mqtt no-secty
echo MQTT_VERBOSE=
MQTT_VERBOSE=
docker compose  -p edgex -f docker-compose-base.yml -f add-device-modbus.yml -f add-asc-mqtt-export.yml convert > docker-compose.yml
rm -rf ./gen_ext_compose


[root@edgex compose-builder]# ls | grep 'docker-compose.yml'
docker-compose.yml
```



### 2.修改配置文件

```yaml
# MQTT 导出数据

# 修改配置
WRITABLE_PIPELINE_FUNCTIONS_MQTTEXPORT_PARAMETERS_BROKERADDRESS导出地址及WRITABLE_PIPELINE_FUNCTIONS_MQTTEXPORT_PARAMETERS_TOPIC导出主题可以自定义以方便服务接受。
tcp://broker.mqttdashboard.com:1883
tcp://192.168.202.233:1884


[root@edgex compose-builder]# vim docker-compose.yml 

name: edgex
services:
  app-mqtt-export:
    container_name: edgex-app-mqtt-export
......

    environment:
      EDGEX_PROFILE: mqtt-export
      EDGEX_SECURITY_SECRET_STORE: "false"
      SERVICE_HOST: edgex-app-mqtt-export
      WRITABLE_LOGLEVEL: INFO
      WRITABLE_PIPELINE_FUNCTIONS_MQTTEXPORT_PARAMETERS_BROKERADDRESS: tcp://192.168.202.233:1884
      WRITABLE_PIPELINE_FUNCTIONS_MQTTEXPORT_PARAMETERS_TOPIC: edgex-events
```



### 3.启动 EdgeX Foundry

使用以下命令部署 EdgeX：

```shell
$ cd edgex-compose/compose-builder
$ docker compose pull
$ docker compose up -d


# 修改配置文件
替换IP地址 127.0.0.1 为 0.0.0.0


# 修改 MQTT 接口
WRITABLE_PIPELINE_FUNCTIONS_MQTTEXPORT_PARAMETERS_BROKERADDRESS: tcp://192.168.202.233:1884
WRITABLE_PIPELINE_FUNCTIONS_MQTTEXPORT_PARAMETERS_TOPIC: edgex-events
```



```shell
# docker compose pull

# docker compose up -d
```



![](/images/edgex/app/link-thingsboard/device-8.png)

![](/images/edgex/app/link-thingsboard/device-9.png)

![](/images/edgex/app/link-thingsboard/device-10.png)



### 3.访问 UI

#### 3.1. consul

```shell
# 访问地址
http://192.168.202.233:8500
```



![](/images/edgex/app/link-thingsboard/device-11.png)



#### 3.2. EdgeX Console

```shell
# 访问地址
http://192.168.202.233:4000/
```



![](/images/edgex/app/link-thingsboard/device-12.png)

![](/images/edgex/app/link-thingsboard/device-13.png)



### 4.创建 Modbus 设备

```shell
# 参考文档
https://docs.edgexfoundry.org/3.1/microservices/device/services/device-modbus/GettingStarted/
```



#### 4.1.创建设备配置文件

**设备配置文件**

```yaml
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
  -
    name: "telemetry1"
    isHidden: false
    description: "attribute for test"
    attributes:
      { primaryTable: "HOLDING_REGISTERS", startingAddress: 6, rawType: "Int16" }
    properties:
      valueType: "Int16"
      readWrite: "RW"
  -
    name: "telemetry2"
    isHidden: false
    description: "attribute for test"
    attributes:
      { primaryTable: "HOLDING_REGISTERS", startingAddress: 7, rawType: "Int16" }
    properties:
      valueType: "Int16"
      readWrite: "RW"
  -
    name: "telemetry3"
    isHidden: false
    description: "attribute for test"
    attributes:
      { primaryTable: "HOLDING_REGISTERS", startingAddress: 8, rawType: "Int16" }
    properties:
      valueType: "Int16"
      readWrite: "RW"

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



![](/images/edgex/app/link-thingsboard/device-14.png)

![](/images/edgex/app/link-thingsboard/device-15.png)

![](/images/edgex/app/link-thingsboard/device-16.png)



#### 4.2.添加设备

**设备配置**

```yaml
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
  -
    interval: "30s"
    onChange: false
    sourceName: "telemetry1"
  -
    interval: "30s"
    onChange: false
    sourceName: "telemetry2"
  -
    interval: "30s"
    onChange: false
    sourceName: "telemetry3"
```



![](/images/edgex/app/link-thingsboard/device-17.png)

![](/images/edgex/app/link-thingsboard/device-18.png)

![](/images/edgex/app/link-thingsboard/device-19.png)

![](/images/edgex/app/link-thingsboard/device-20.png)

![](/images/edgex/app/link-thingsboard/device-21.png)

![](/images/edgex/app/link-thingsboard/device-22.png)

![](/images/edgex/app/link-thingsboard/device-23.png)



### 5.运行 EMQX

```shell
$ docker run -d --name emqx -p 18084:18083 -p 1884:1883 emqx/emqx:4.4.17
```



### 6.运行 Modbus 模拟器

![](/images/edgex/app/link-thingsboard/device-2.png)





## 三、Node-RED 协议转换 



### 1.ThingsBoard 创建设备

![](/images/edgex/app/link-thingsboard/device-6.png)

![](/images/edgex/app/link-thingsboard/device-7.png)



```shell
设备名称：edgex-foundry-device
访问令牌：mAS0pvYc4VbyIHry6CYp
```



### 2.运行 Node-RED

```shell
docker run -d --network host --restart=always -v node_red_data:/data --name nodered nodered/node-red:latest
```



### 3.配置 Node-RED

#### 3.1.配置说明

**配置步骤：**

- 配置 MQTT 方式向 ThingsBoard 上传遥测数据
- 配置 MQTT 方式从 EMQX 订阅 EdgeX Foundry 设备数据
- 设备数据进行协议转换，上传到 ThingsBoard



![](/images/edgex/app/link-thingsboard/device-24.png)



#### 3.2.ThingsBoard 遥测上传

```shell
设备名称：edgex-foundry-device
访问令牌：mAS0pvYc4VbyIHry6CYp
```



![](/images/edgex/app/link-thingsboard/device-25.png)

![](/images/edgex/app/link-thingsboard/device-26.png)

![](/images/edgex/app/link-thingsboard/device-27.png)



#### 3.3.订阅 EdgeX Foundry 设备数据

![](/images/edgex/app/link-thingsboard/device-28.png)

![](/images/edgex/app/link-thingsboard/device-29.png)



#### 3.4.协议转换

**EdgeX Foundry 导出的 Modbus 设备数据**

```json
{
	"apiVersion": "v3",
	"id": "828a8ac7-ecaa-4d18-983b-13f196089626",
	"deviceName": "Modbus-TCP-Temperature-Sensor",
	"profileName": "Ethernet-Temperature-Sensor",
	"sourceName": "telemetry1",
	"origin": 1708609098226528187,
	"readings": [{
		"id": "bb53d4fa-df2c-4b15-8f57-e33b76ff86e5",
		"origin": 1708609098226456534,
		"deviceName": "Modbus-TCP-Temperature-Sensor",
		"resourceName": "telemetry1",
		"profileName": "Ethernet-Temperature-Sensor",
		"valueType": "Int16",
		"value": "60"
	}]
}


{
	"apiVersion": "v3",
	"id": "b06be718-9166-46d5-aa94-f5b7825c973b",
	"deviceName": "Modbus-TCP-Temperature-Sensor",
	"profileName": "Ethernet-Temperature-Sensor",
	"sourceName": "telemetry2",
	"origin": 1708609100268535655,
	"readings": [{
		"id": "38088a65-b336-41cc-b13a-26d6a0caee66",
		"origin": 1708609100268474454,
		"deviceName": "Modbus-TCP-Temperature-Sensor",
		"resourceName": "telemetry2",
		"profileName": "Ethernet-Temperature-Sensor",
		"valueType": "Int16",
		"value": "700"
	}]
}


{
	"apiVersion": "v3",
	"id": "95435a12-e779-48dc-9b57-4d01ba3d2d50",
	"deviceName": "Modbus-TCP-Temperature-Sensor",
	"profileName": "Ethernet-Temperature-Sensor",
	"sourceName": "telemetry3",
	"origin": 1708609096226486863,
	"readings": [{
		"id": "53504b9c-0912-4e22-add0-36b3d243a35a",
		"origin": 1708609096226388959,
		"deviceName": "Modbus-TCP-Temperature-Sensor",
		"resourceName": "telemetry3",
		"profileName": "Ethernet-Temperature-Sensor",
		"valueType": "Int16",
		"value": "8000"
	}]
}
```



**协议转换 JavaScript** 

```javascript
var payload = msg.payload;
var resourceName = payload.readings[0].resourceName;
var value = payload.readings[0].value;

if (resourceName == "telemetry1") {
    
    msg.payload = {telemetry1: parseInt(value)};
    
} else if (resourceName == "telemetry2") {
    
    msg.payload = {telemetry2: parseInt(value)};
    
} else {
    
    msg.payload = {telemetry3: parseInt(value)};
    
}

return msg;
```



![](/images/edgex/app/link-thingsboard/device-30.png)

![](/images/edgex/app/link-thingsboard/device-31.png)





## 四、系统测试



### 1.Modbus 设备

![](/images/edgex/app/link-thingsboard/device-32.png)

![](/images/edgex/app/link-thingsboard/device-33.png)



### 2.Node-RED

![](/images/edgex/app/link-thingsboard/device-34.png)

![](/images/edgex/app/link-thingsboard/device-35.png)



### 3.ThingsBoard

![](/images/edgex/app/link-thingsboard/device-36.png)

![](/images/edgex/app/link-thingsboard/device-37.png)

![](/images/edgex/app/link-thingsboard/device-38.png)

![](/images/edgex/app/link-thingsboard/device-33.png)



