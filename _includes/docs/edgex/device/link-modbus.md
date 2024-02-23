* TOC
{:toc}



## 一、概述



### 1.安装说明

```shell
# 官方文档

https://docs.edgexfoundry.org/3.1/microservices/device/services/device-modbus/GettingStarted/
```



**安装方式：**

- 使用 EdgeX Console 界面创建 Modbus 设备
- 使用 Modbus Slave 模拟 Modbus 设备



### 2.Modbus Slave 工具

![](/images/edgex/device/link-modbus/modbus-1.png)

![](/images/edgex/device/link-modbus/modbus-2.png)



## 二、连接 Modbus 设备



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

**1.设备配置文件**

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



**2.设备配置**

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
    interval: "30s"
    onChange: false
    sourceName: "Temperature"
```





### 3.启动 EdgeX Foundry

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

# docker compose up -d
```



![](/images/edgex/device/link-modbus/modbus-3.png)

![](/images/edgex/device/link-modbus/modbus-4.png)

![](/images/edgex/device/link-modbus/modbus-5.png)



### 4.访问 UI

#### 4.1. consul

```shell
# 访问地址
http://192.168.202.233:8500
```

![](/images/edgex/device/link-modbus/modbus-6.png)



#### 4.2. EdgeX Console

```shell
# 访问地址
http://192.168.202.233:4000/
```



![](/images/edgex/device/link-modbus/modbus-7.png)

![](/images/edgex/device/link-modbus/modbus-8.png)

![](/images/edgex/device/link-modbus/modbus-9.png)



### 5.创建 Modbus 设备

#### 5.1.创建设备配置文件

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



![](/images/edgex/device/link-modbus/modbus-10.png)

![](/images/edgex/device/link-modbus/modbus-11.png)

![](/images/edgex/device/link-modbus/modbus-12.png)



#### 5.2.添加设备

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
    interval: "30s"
    onChange: false
    sourceName: "Temperature"
```



![](/images/edgex/device/link-modbus/modbus-13.png)

![](/images/edgex/device/link-modbus/modbus-14.png)

![](/images/edgex/device/link-modbus/modbus-15.png)

![](/images/edgex/device/link-modbus/modbus-16.png)



![](/images/edgex/device/link-modbus/modbus-17.png)

![](/images/edgex/device/link-modbus/modbus-18.png)

![](/images/edgex/device/link-modbus/modbus-19.png)



### 6.测试

#### 6.1.命令

![](/images/edgex/device/link-modbus/modbus-20.png)

![](/images/edgex/device/link-modbus/modbus-21.png)



#### 6.2.事件

![](/images/edgex/device/link-modbus/modbus-22.png)

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



#### 6.3.读值

![](/images/edgex/device/link-modbus/modbus-23.png)

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



