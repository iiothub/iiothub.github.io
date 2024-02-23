* TOC
{:toc}



## 一、概述



### 1.系统设计说明

- EdgeX Foundry 同时接入 MQTT、Modbus 两种设备
- 设备数据通过 MQTT 方式导出到第三方系统
- MQTT 设备使用官网的案例程序模拟（mock-device.js）
- Modbus 设备使用 Modbus Slave 工具模拟
- EMQX 作为导出数据的外部 mqtt-broker
- MQTTX 工具模拟第三方系统从 EMQX 订阅设备数据



### 2.docker-comepse

```shell
# 1.克隆 edgex-compose
$ git clone https://github.com/edgexfoundry/edgex-compose.git
$ cd edgex-compose 
$ git checkout v3.1


# 2.生成 docker-compose.yml 文件（注意这包括 mqtt-broker）
$ cd compose-builder
$ make gen ds-modbus ds-mqtt mqtt-broker asc-mqtt no-secty


# 3.检查生成的文件
$ ls | grep 'docker-compose.yml'
docker-compose.yml
```



### 3.MQTT 模拟设备

```shell
# 参考文档
https://docs.edgexfoundry.org/3.1/microservices/device/services/device-mqtt/Ch-ExamplesAddingMQTTDevice/
```



要实现模拟的自定义 MQTT 设备，请创建一个名为 的 javascript，`mock-device.js`其中包含以下内容：

```javascript
function getRandomFloat(min, max) {
    return Math.random() * (max - min) + min;
}

const deviceName = "my-custom-device";
let message = "test-message";
let json = {"name" : "My JSON"};

// DataSender sends async value to MQTT broker every 15 seconds
schedule('*/15 * * * * *', ()=>{
    var data = {};
    data.randnum = getRandomFloat(25,29).toFixed(1);
    data.ping = "pong"
    data.message = "Hello World"

    publish( 'incoming/data/my-custom-device/values', JSON.stringify(data));
});

// CommandHandler receives commands and sends response to MQTT broker
// 1. Receive the reading request, then return the response
// 2. Receive the set request, then change the device value
subscribe( "command/my-custom-device/#" , (topic, val) => {
    const words = topic.split('/');
    var cmd = words[2];
    var method = words[3];
    var uuid = words[4];
    var response = {};
    var data = val;

    if (method == "set") {
        switch(cmd) {
            case "message":
                message = data[cmd];
                break;
            case "json":
                json = data[cmd];
                break;
        }
    }else{
        switch(cmd) {
            case "ping":
                response.ping = "pong";
                break;
            case "message":
                response.message = message;
                break;
            case "randnum":
                response.randnum = 12.123;
                break;
            case "json":
                response.json = json;
                break;
        }
    }
    var sendTopic ="command/response/"+ uuid;
    publish( sendTopic, JSON.stringify(response));
});
```

要运行设备模拟器，请输入下面所示的命令并进行以下更改：

```shell
$ mv mock-device.js /path/to/mqtt-scripts
$ docker run --rm --name=mqtt-scripts \
    -v /path/to/mqtt-scripts:/scripts  --network host \
    dersimn/mqtt-scripts --dir /scripts
```

`/path/to/mqtt-scripts`将示例中的 mv 命令替换为正确的路径

然后 mqtt-scripts 显示日志如下：

```shell
2022-08-12 09:52:42.086 <info>  mqtt-scripts 1.2.2 starting
2022-08-12 09:52:42.227 <info>  mqtt connected mqtt://127.0.0.1
2022-08-12 09:52:42.733 <info>  /scripts/mock-device.js loading
```



```shell
$ mv mock-device.js /path/to/mqtt-scripts
$ docker run --rm --name=mqtt-scripts \
    -v /path/to/mqtt-scripts:/scripts  --network host \
    dersimn/mqtt-scripts --dir /scripts


# mv mock-device.js /edgex/mqtt-device/mqtt-scripts
  
# docker run -d --rm --name=mqtt-scripts \
    -v /edgex/mqtt-device/mqtt-scripts:/scripts  --network host \
    dersimn/mqtt-scripts --dir /scripts
```



### 4.Modbus Slave

![](/images/edgex/app/link-device/device-1.png)

![](/images/edgex/app/link-device/device-2.png)



### 5.安装 EMQX

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

![](/images/edgex/app/link-device/device-4.png)



### 6.MQTTX 工具

![](/images/edgex/app/link-device/device-3.png)

![](/images/edgex/app/link-device/device-5.png)



## 二、连接设备服务



### 1.docker-comepse

```shell
# 1.克隆 edgex-compose
$ git clone https://github.com/edgexfoundry/edgex-compose.git
$ cd edgex-compose 
$ git checkout v3.1


# 2.生成 docker-compose.yml 文件（注意这包括 mqtt-broker）
$ cd compose-builder
$ make gen ds-modbus ds-mqtt mqtt-broker asc-mqtt no-secty


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

[root@edgex compose-builder]# make gen ds-modbus ds-mqtt mqtt-broker asc-mqtt no-secty
echo MQTT_VERBOSE=
MQTT_VERBOSE=
docker compose  -p edgex -f docker-compose-base.yml -f add-device-modbus.yml -f add-device-mqtt.yml -f add-asc-mqtt-export.yml -f add-mqtt-broker-mosquitto.yml convert > docker-compose.yml
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



![](/images/edgex/app/link-device/device-6.png)

![](/images/edgex/app/link-device/device-7.png)

![](/images/edgex/app/link-device/device-8.png)



### 3.访问 UI

#### 3.1. consul

```shell
# 访问地址
http://192.168.202.233:8500
```

![](/images/edgex/app/link-device/device-9.png)



#### 3.2. EdgeX Console

```shell
# 访问地址
http://192.168.202.233:4000/
```



![](/images/edgex/app/link-device/device-10.png)

![](/images/edgex/app/link-device/device-11.png)



### 4.创建 MQTT 设备

```shell
# 参考文档
https://docs.edgexfoundry.org/3.1/microservices/device/services/device-mqtt/Ch-ExamplesAddingMQTTDevice/
```



#### 4.1.创建设备配置文件

![](/images/edgex/app/link-device/device-12.png)

![](/images/edgex/app/link-device/device-13.png)

![](/images/edgex/app/link-device/device-14.png)



**设备配置文件**

```yaml
name: "my-custom-device-profile"
manufacturer: "iot"
model: "MQTT-DEVICE"
description: "Test device profile"
labels:
  - "mqtt"
  - "test"
deviceResources:
  -
    name: randnum
    isHidden: true
    description: "device random number"
    properties:
      valueType: "Float32"
      readWrite: "R"
  -
    name: ping
    isHidden: true
    description: "device awake"
    properties:
      valueType: "String"
      readWrite: "R"
  -
    name: message
    isHidden: false
    description: "device message"
    properties:
      valueType: "String"
      readWrite: "RW"
  -
    name: json
    isHidden: false
    description: "JSON message"
    properties:
      valueType: "Object"
      readWrite: "RW"
      mediaType: "application/json"

deviceCommands:
  -
    name: values
    readWrite: "R"
    isHidden: false
    resourceOperations:
        - { deviceResource: "randnum" }
        - { deviceResource: "ping" }
        - { deviceResource: "message" }
```



#### 4.2.添加设备

**设备配置**

使用此配置文件来定义设备和调度作业。device-mqtt 在启动时生成一个相对实例。

```yaml
# Pre-define Devices
deviceList:
- name: "my-custom-device"
  profileName: "my-custom-device-profile"
  description: "MQTT device is created for test purpose"
  labels: [ "MQTT", "test" ]
  protocols:
    mqtt:
      CommandTopic: "command/my-custom-device"
  autoEvents:
    - interval: "30s"
      onChange: false
      sourceName: "message"
```



![](/images/edgex/app/link-device/device-15.png)

![](/images/edgex/app/link-device/device-16.png)

![](/images/edgex/app/link-device/device-17.png)

![](/images/edgex/app/link-device/device-18.png)

![](/images/edgex/app/link-device/device-19.png)

![](/images/edgex/app/link-device/device-20.png)



### 5.创建 Modbus 设备

```shell
# 参考文档
https://docs.edgexfoundry.org/3.1/microservices/device/services/device-modbus/GettingStarted/
```



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



![](/images/edgex/app/link-device/device-21.png)

![](/images/edgex/app/link-device/device-22.png)

![](/images/edgex/app/link-device/device-23.png)



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



![](/images/edgex/app/link-device/device-24.png)

![](/images/edgex/app/link-device/device-25.png)

![](/images/edgex/app/link-device/device-26.png)

![](/images/edgex/app/link-device/device-27.png)

![](/images/edgex/app/link-device/device-28.png)

![](/images/edgex/app/link-device/device-29.png)

![](/images/edgex/app/link-device/device-30.png)



### 6.运行 EMQX

```shell
$ docker run -d --name emqx -p 18084:18083 -p 1884:1883 emqx/emqx:4.4.17
```



### 7.运行 MQTT 模拟器

```shell
# docker run -d --rm --name=mqtt-scripts \
    -v /edgex/mqtt-device/mqtt-scripts:/scripts  --network host \
    dersimn/mqtt-scripts --dir /scripts
```



### 8.运行 Modbus 模拟器

![](/images/edgex/app/link-device/device-2.png)



## 三、系统测试



### 1.MQTT 设备

- **命令**

![](/images/edgex/app/link-device/device-31.png)



- **事件**

![](/images/edgex/app/link-device/device-32.png)



- **读值**

![](/images/edgex/app/link-device/device-33.png)



### 2.Modbus 设备

- **命令**

![](/images/edgex/app/link-device/device-34.png)

![](/images/edgex/app/link-device/device-35.png)



- **事件**

![](/images/edgex/app/link-device/device-36.png)



- **读值**

![](/images/edgex/app/link-device/device-37.png)



### 3.导出设备数据

![](/images/edgex/app/link-device/device-38.png)



**1.MQTT 设备数据**

```json
{
	"apiVersion": "v3",
	"id": "c7d356a2-eb11-48e5-8aef-dcbc4f23e24c",
	"deviceName": "my-custom-device",
	"profileName": "my-custom-device-profile",
	"sourceName": "values",
	"origin": 1708522920004623808,
	"readings": [{
		"id": "6fcfbe71-0a4f-4f7f-9162-91d0e62bc728",
		"origin": 1708522920004611629,
		"deviceName": "my-custom-device",
		"resourceName": "randnum",
		"profileName": "my-custom-device-profile",
		"valueType": "Float32",
		"value": "2.780000e+01"
	}, {
		"id": "4015c2e6-b71f-4943-ab0b-6ee5bb4d15ec",
		"origin": 1708522920004612390,
		"deviceName": "my-custom-device",
		"resourceName": "ping",
		"profileName": "my-custom-device-profile",
		"valueType": "String",
		"value": "pong"
	}, {
		"id": "3ec44cdf-c5d2-4c2c-aad0-67d767bb6f63",
		"origin": 1708522920004612803,
		"deviceName": "my-custom-device",
		"resourceName": "message",
		"profileName": "my-custom-device-profile",
		"valueType": "String",
		"value": "Hello World"
	}]
}
```



**2.Modbus 设备数据**

```json
{
	"apiVersion": "v3",
	"id": "ff9b40a6-fdf0-4b47-ad01-ffd46f15c77f",
	"deviceName": "Modbus-TCP-Temperature-Sensor",
	"profileName": "Ethernet-Temperature-Sensor",
	"sourceName": "Temperature",
	"origin": 1708522958633208207,
	"readings": [{
		"id": "1970297d-f13e-4573-ad9a-8688a94aae4e",
		"origin": 1708522958633161656,
		"deviceName": "Modbus-TCP-Temperature-Sensor",
		"resourceName": "Temperature",
		"profileName": "Ethernet-Temperature-Sensor",
		"valueType": "Float32",
		"value": "0.000000e+00"
	}]
}
```



