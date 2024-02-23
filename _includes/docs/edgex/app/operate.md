* TOC
{:toc}



![](/images/edgex/app/operate/base-5.png)



## 一、容器管理



### 1.容器操作

```shell
# 安装并启动EdgeX
sudo docker-compose up -d     # -d 后台运行容器
 
# 查看所有容器运行状况
sudo docker-compose ps
 
# 显示容器日志
docker-compose logs -f [compose-contatainer-name]
 
# 停止容器
sudo docker-compose stop
 
# 启动容器
sudo docker-compose start
 
# 停止和删除所有容器
sudo docker-compose down
```



![](/images/edgex/app/operate/base-1.png)

![](/images/edgex/app/operate/base-2.png)

![](/images/edgex/app/operate/base-3.png)



### 2.查看容器日志

```shell
# docker-compose logs
# docker-compose logs -f [compose-contatainer-name]
```

![](/images/edgex/app/operate/base-4.png)





## 二、EdgeX UI 操作



### 1.访问 UI

#### 1.1. consul

```shell
# 访问地址
http://192.168.202.233:8500
```

![](/images/edgex/app/operate/base-6.png)



#### 1.2. EdgeX Console

```shell
# 访问地址
http://192.168.202.233:4000/
```

![](/images/edgex/app/operate/base-7.png)



### 2.创建 MQTT 设备

#### 2.1.创建设备配置文件

![](/images/edgex/app/operate/base-8.png)

![](/images/edgex/app/operate/base-9.png)

![](/images/edgex/app/operate/base-10.png)



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



#### 2.2.添加设备

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



![](/images/edgex/app/operate/base-11.png)

![](/images/edgex/app/operate/base-12.png)

![](/images/edgex/app/operate/base-13.png)

![](/images/edgex/app/operate/base-14.png)

![](/images/edgex/app/operate/base-15.png)

![](/images/edgex/app/operate/base-16.png)

![](/images/edgex/app/operate/base-17.png)



### 3.设备配置文件

#### 3.1.配置文件管理

![](/images/edgex/app/operate/base-18.png)



#### 3.2.修改配置文件

![](/images/edgex/app/operate/base-19.png)

![](/images/edgex/app/operate/base-20.png)



### 4.设备

#### 4.1.设备管理

![](/images/edgex/app/operate/base-21.png)



#### 4.2.修改设备信息

![](/images/edgex/app/operate/base-22.png)



#### 4.3.命令

![](/images/edgex/app/operate/base-23.png)



#### 4.4.自动采集

![](/images/edgex/app/operate/base-24.png)



### 5.设备通信

#### 5.1.设备命令

![](/images/edgex/app/operate/base-25.png)



#### 5.2.事件

![](/images/edgex/app/operate/base-26.png)



#### 5.3.读值

![](/images/edgex/app/operate/base-27.png)





## 三、REST API



### 1.查询执行命令

```shell
$ curl http://localhost:59882/api/v3/device/all | json_pp

$ curl http://localhost:59882/api/v3/device/name/device-name
```



```shell
[root@edgex compose-builder]# curl http://localhost:59882/api/v3/device/all


{"apiVersion":"v3","statusCode":200,"totalCount":2,"deviceCoreCommands":[{"deviceName":"MQTT-test-device","profileName":"Test-Device-MQTT-Profile","coreCommands":[{"name":"ping","get":true,"path":"/api/v3/device/name/MQTT-test-device/ping","url":"http://edgex-core-command:59882","parameters":[{"resourceName":"ping","valueType":"String"}]},{"name":"message","get":true,"set":true,"path":"/api/v3/device/name/MQTT-test-device/message","url":"http://edgex-core-command:59882","parameters":[{"resourceName":"message","valueType":"String"}]},{"name":"json","get":true,"set":true,"path":"/api/v3/device/name/MQTT-test-device/json","url":"http://edgex-core-command:59882","parameters":[{"resourceName":"json","valueType":"Object"}]},{"name":"allValues","get":true,"set":true,"path":"/api/v3/device/name/MQTT-test-device/allValues","url":"http://edgex-core-command:59882","parameters":[{"resourceName":"randfloat32","valueType":"Float32"},{"resourceName":"randfloat64","valueType":"Float64"},{"resourceName":"message","valueType":"String"}]},{"name":"randfloat32","get":true,"set":true,"path":"/api/v3/device/name/MQTT-test-device/randfloat32","url":"http://edgex-core-command:59882","parameters":[{"resourceName":"randfloat32","valueType":"Float32"}]},{"name":"randfloat64","get":true,"set":true,"path":"/api/v3/device/name/MQTT-test-device/randfloat64","url":"http://edgex-core-command:59882","parameters":[{"resourceName":"randfloat64","valueType":"Float64"}]}]},{"deviceName":"my-custom-device","profileName":"my-custom-device-profile","coreCommands":[{"name":"values","get":true,"path":"/api/v3/device/name/my-custom-device/values","url":"http://edgex-core-command:59882","parameters":[{"resourceName":"randnum","valueType":"Float32"},{"resourceName":"ping","valueType":"String"},{"resourceName":"message","valueType":"String"}]},{"name":"message","get":true,"set":true,"path":"/api/v3/device/name/my-custom-device/message","url":"http://edgex-core-command:59882","parameters":[{"resourceName":"message","valueType":"String"}]},{"name":"json","get":true,"set":true,"path":"/api/v3/device/name/my-custom-device/json","url":"http://edgex-core-command:59882","parameters":[{"resourceName":"json","valueType":"Object"}]}]}]}
```



```shell
[root@edgex compose-builder]# curl http://localhost:59882/api/v3/device/name/my-custom-device


{
	"apiVersion": "v3",
	"statusCode": 200,
	"deviceCoreCommand": {
		"deviceName": "my-custom-device",
		"profileName": "my-custom-device-profile",
		"coreCommands": [{
			"name": "json",
			"get": true,
			"set": true,
			"path": "/api/v3/device/name/my-custom-device/json",
			"url": "http://edgex-core-command:59882",
			"parameters": [{
				"resourceName": "json",
				"valueType": "Object"
			}]
		}, {
			"name": "values",
			"get": true,
			"path": "/api/v3/device/name/my-custom-device/values",
			"url": "http://edgex-core-command:59882",
			"parameters": [{
				"resourceName": "randnum",
				"valueType": "Float32"
			}, {
				"resourceName": "ping",
				"valueType": "String"
			}, {
				"resourceName": "message",
				"valueType": "String"
			}]
		}, {
			"name": "message",
			"get": true,
			"set": true,
			"path": "/api/v3/device/name/my-custom-device/message",
			"url": "http://edgex-core-command:59882",
			"parameters": [{
				"resourceName": "message",
				"valueType": "String"
			}]
		}]
	}
}
```



![](/images/edgex/app/operate/base-28.png)



### 2.SET 命令

```shell
$ curl http://localhost:59882/api/v3/device/name/my-custom-device/message \
    -H "Content-Type:application/json" -X PUT  \
    -d '{"message":"Hello!"}'
```



```shell
# curl http://localhost:59882/api/v3/device/name/my-custom-device/message \
    -H "Content-Type:application/json" -X PUT  \
    -d '{"message":"Hello!"}'


[root@edgex compose-builder]# curl http://localhost:59882/api/v3/device/name/my-custom-device/message \
>     -H "Content-Type:application/json" -X PUT  \
>     -d '{"message":"Hello!"}'
{"apiVersion":"v3","statusCode":200}
```



![](/images/edgex/app/operate/base-29.png)



### 3.GET 命令

```shell
$ curl http://localhost:59882/api/v3/device/name/my-custom-device/message
```



```shell
# curl http://localhost:59882/api/v3/device/name/my-custom-device/message


[root@edgex compose-builder]# curl http://localhost:59882/api/v3/device/name/my-custom-device/message


{
	"apiVersion": "v3",
	"statusCode": 200,
	"event": {
		"apiVersion": "v3",
		"id": "2bac78a7-b959-44d9-ac9c-c8e39b8f4b56",
		"deviceName": "my-custom-device",
		"profileName": "my-custom-device-profile",
		"sourceName": "message",
		"origin": 1708510731459679263,
		"readings": [{
			"id": "8d14ecae-a1a1-439b-8fe1-f85f50783548",
			"origin": 1708510731459675461,
			"deviceName": "my-custom-device",
			"resourceName": "message",
			"profileName": "my-custom-device-profile",
			"valueType": "String",
			"value": "Hello!"
		}]
	}
}
```



### 4.调度作业

`autoEvents`调度作业在设备定义文件的部分中定义：

```
    autoEvents:
       Interval: "30s"
       OnChange: false
       SourceName: "message"
```

服务启动后，查询core-data的读取API。结果显示服务每30秒自动执行一次命令，如下所示：

```shell
$ curl http://localhost:59880/api/v3/reading/resourceName/message | json_pp
```



```shell
# curl http://localhost:59880/api/v3/reading/resourceName/message


[root@edgex compose-builder]# curl http://localhost:59880/api/v3/reading/resourceName/message


{
	"apiVersion": "v3",
	"statusCode": 200,
	"totalCount": 840,
	"readings": [{
		"id": "2beff7bf-fe84-4301-9baa-ef283b99c0b3",
		"origin": 1708510957760593943,
		"deviceName": "my-custom-device",
		"resourceName": "message",
		"profileName": "my-custom-device-profile",
		"valueType": "String",
		"value": "Hello!"
	}, 
	
	{
		"id": "e45e0f64-3f28-41a8-91be-0f4c0865a2f7",
		"origin": 1708510950004786813,
		"deviceName": "my-custom-device",
		"resourceName": "message",
		"profileName": "my-custom-device-profile",
		"valueType": "String",
		"value": "Hello World"
	}, 
	
	......
	
	]
}
```



### 5.异步设备读值

订阅`device-mqtt`a `DataTopic`，等待[真实设备向 MQTT Broker 发送值](https://docs.edgexfoundry.org/3.1/microservices/device/services/device-mqtt/Ch-ExamplesAddingMQTTDevice/#creating-and-running-a-mqtt-device-simulator)，然后`device-mqtt`解析该值并转发到北向。

数据格式包含以下值：

- name = device name
- cmd = deviceResource name
- method = get or set
- cmd = device reading

以下结果显示模拟设备每 15 秒发送一次读数：

```shell
$ curl http://localhost:59880/api/v3/reading/resourceName/randnum | json_pp
```



```shell
# curl http://localhost:59880/api/v3/reading/resourceName/randnum


[root@edgex compose-builder]# curl http://localhost:59880/api/v3/reading/resourceName/randnum


{
	"apiVersion": "v3",
	"statusCode": 200,
	"totalCount": 824,
	"readings": [{
		"id": "c4acd21d-6ea5-4a29-a865-4d748c7740b6",
		"origin": 1708514190002112819,
		"deviceName": "my-custom-device",
		"resourceName": "randnum",
		"profileName": "my-custom-device-profile",
		"valueType": "Float32",
		"value": "2.510000e+01"
	}, {
		"id": "d23c8625-d5ea-4fb2-ac95-659d533573ae",
		"origin": 1708514115001993970,
		"deviceName": "my-custom-device",
		"resourceName": "randnum",
		"profileName": "my-custom-device-profile",
		"valueType": "Float32",
		"value": "2.810000e+01"
	}, {
		"id": "c71a58ac-9846-43fe-9533-bf378fbee649",
		"origin": 1708514070003642414,
		"deviceName": "my-custom-device",
		"resourceName": "randnum",
		"profileName": "my-custom-device-profile",
		"valueType": "Float32",
		"value": "2.720000e+01"
	},
	
	......
	
	]
}
```



