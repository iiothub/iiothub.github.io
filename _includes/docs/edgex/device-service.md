* TOC
{:toc}



## 1.设备服务



设备服务是与传感器/设备或物联网对象(“物”)交互的边缘连接器，其中包括机器、机器人、无人机、HVAC 设备、相机等。通过利用可用的连接器，可以控制设备并/或传输数据至 EdgeX 或从其传输数据。您还可以使用设备服务 SDK 来创建您自己的 EdgeX 设备服务。

![](/images/edgex/default/device-service/device-1.png)

设备配置文件描述了EdgeX 系统内的一种设备类型。设备服务管理的每个设备都与设备配置文件关联，设备配置文件根据其支持的操作定义该设备类型。

通常，设备配置文件由设备服务从文件加载，并在首次启动时推送到核心元数据进行存储。一旦存储在核心元数据中，设备配置文件就会在后续启动时从核心元数据加载。



## 2.设备配置文件

设备配置文件由以下字段组成

| Field Name      | Type                    | Required? | Description                                                  |
| :-------------- | :---------------------- | :-------- | :----------------------------------------------------------- |
| name            | String                  | Y         | Must be unique in the EdgeX deployment.                      |
| description     | String                  | N         | Description of the Device Profile                            |
| manufacturer    | String                  | N         | Manufacturer of the device described by the Device Profile   |
| model           | String                  | N         | Model of the device(s) described by the Device Profile       |
| labels          | Array of String         | N         | Free form labels for querying device profiles                |
| deviceResources | Array of DeviceResource | Y         | See [Device Resources](https://docs.edgexfoundry.org/3.1/microservices/device/details/DeviceProfiles/#device-resources) below |
| deviceCommands  | Array of DeviceCommand  | N         | See [Device Commands](https://docs.edgexfoundry.org/3.1/microservices/device/details/DeviceProfiles/#device-commands) below |



DeviceProfile定义了设备的值和操作方法，可以是Read或Write。

创建名为 的设备配置文件`my.custom.device.profile.yml`，其中包含以下内容：

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



## 3.设备资源

设备资源指定设备内的传感器值，可以单独或作为设备命令的一部分读取或写入该值。它具有用于识别的名称和用于提供信息的描述。

设备资源由以下字段组成：

The device resource consists of the following fields:

| Field Name  | Type                 | Required? | Notes                                                        |
| :---------- | :------------------- | :-------- | :----------------------------------------------------------- |
| name        | String               | Y         | Must be unique in the EdgeX deployment.                      |
| description | String               | N         | Description of the device resource                           |
| isHidden    | Bool                 | N         | Hide the device resource as command via the Command Service, default false |
| tags        | String-Interface Map | N         | User define collection of tags                               |
| attributes  | String-Interface Map | N         | See [Resource Attributes](https://docs.edgexfoundry.org/3.1/microservices/device/details/DeviceProfiles/#resource-attributes) below |
| properties  | ResourceProperties   | Y         | See [Resource Properties](https://docs.edgexfoundry.org/3.1/microservices/device/details/DeviceProfiles/#resource-properties) below |



## 4.资源属性（Attributes）

设备资源中的`attributes`参数是访问设备上的特定值所需的设备服务特定参数。每个设备服务实现都有自己的一组所需的命名值，例如，BACnet 设备服务可能需要对象标识符和属性标识符，而蓝牙设备服务可以使用 UUID 来标识值。

设备 ONVIF 摄像头的资源属性示例

```
    attributes:
      service: "Device"
      getFunction: "GetDNS"
      setFunction: "SetDNS"
```



## 5.资源属性（Properties）

设备资源`properties`的 描述了值以及要对其执行的可选简单处理。

资源属性由以下字段组成：

The resource properties consists of the following fields:

| Field Name   | Type           | Required? | Notes                                                        |
| :----------- | :------------- | :-------- | :----------------------------------------------------------- |
| valueType    | Enum           | Y         | The data type of the value. Supported types are: `Uint8`, `Uint16`, `Uint32`, `Uint64`, `Int8`, `Int16`, `Int32`, `Int64`, `Float32`, `Float64`, `Bool`, `String`, `Binary`, `Object`, `Uint8Array`, `Uint16Array`, `Uint32Array`, `Uint64Array`, `Int8Array`, `Int16Array`, `Int32Array`, `Int64Array`, `Float32Array`, `Float64Array`, `BoolArray` |
| readWrite    | Enum           | Y         | Indicates whether the value is readable or writable or both. `R` - Read only , `W` - Write only, `RW` - Read or Write |
| units        | String         | N         | Developer defined units of value such as secs, mins, etc     |
| minimum      | Float64        | N         | Minimum value the resource value can be set to. Error if SET command value out of minimum range |
| maximum      | Float64        | N         | Maximum value the resource value can be set to. Error if SET command value out of maximum range |
| defaultValue | String         | N         | Default value to use when no value is present for a set command. If present, should be compatible with the valueType field |
| mask         | Uint64         | N         | A binary mask which will be applied to an integer reading. Only valid when valueType is one of the unsigned integer types |
| shift        | Int64          | N         | A number of bits by which an integer reading will be shifted right. Only valid when valueType is one of the unsigned integer types |
| scale        | Float64        | N         | A factor by which to multiply a reading before it is returned. Only valid when valueType is one of the integer or float types |
| offset       | Float64        | N         | A value to be added to a reading before it is returned. Only valid when valueType is one of the integer or float types |
| base         | Float64        | N         | A value to be raised to the power of the raw reading before it is returned. Only valid when valueType is one of the integer or float types |
| assertion    | String         | N         | A string value to which a reading (after processing) is compared. If the reading is not the same as the assertion value, the device's operating state will be set to disabled. This can be useful for health checks. |
| mediaType    | String         | N         | A string indicating the content type of the `Binary` value. Required when valueType is `Binary`. |
| optional     | String-Any Map | N         | Optional mapping for developer use                           |



## 6.设备命令

设备命令定义同时访问多个资源。每个命名设备命令应包含多个`resource operations`.具有单个资源操作的设备命令不会比 SDK 为同一设备资源创建的隐式设备命令增加任何值。

当读数在逻辑上相关时，设备命令可能很有用，例如，对于 3 轴加速度计，一起读取所有轴会很有帮助。

每个设备命令由以下字段组成：

Each device command consists of the following fields:

| Field Name         | Type                       | Required? | Notes                                                        |
| :----------------- | :------------------------- | :-------- | :----------------------------------------------------------- |
| name               | String                     | Y         | Must be unique in this profile.                              |
| isHidden           | Bool                       | N         | Hide the Device Command for use via Command Service, default false |
| readWrite          | Enum                       | Y         | Indicates whether the command is readable or writable or both. `R` - Read only , `W` - Write only, `RW` - Read or Write. Resources' readWrite included in the command must be consistent with the value chosen here. |
| resourceOperations | Array of ResourceOperation | Y         | see [Resource Operation](https://docs.edgexfoundry.org/3.1/microservices/device/details/DeviceProfiles/#resource-operation) below |



## 7.资源操作

资源操作由以下字段组成：

A resource operation consists of the following fields:

| Field Name     | Type              | Required? | Notes                                                        |
| :------------- | :---------------- | :-------- | :----------------------------------------------------------- |
| deviceResource | String            | Y         | Must name a Device Resource in this profile                  |
| defaultValue   | String            | N         | If present, should be compatible with the Type field of the named Device Resource |
| mappings       | String-String Map | N         | Map the GET resourceOperation value to another string value  |



## 8.REST 命令端点

命令端点是在服务上为设备配置文件中指定的每个设备资源和每个设备命令隐式创建的。有关更多详细信息，请参阅[设备服务 API 参考](https://docs.edgexfoundry.org/3.1/microservices/device/ApiReference/)中的 GET 和 SET 设备命令 API 。





## 9.推送事件

SDK 应实现除了收到设备 GET 请求之外生成事件的方法。 AutoEvent 机制提供以固定时间间隔生成事件的功能。异步事件队列使设备服务能够根据特定于实现的逻辑在任意时间生成事件。



**自动事件**

`AutoEvents`每个设备可以具有与其关联的多个元数据作为其定义的一部分。 An`AutoEvent`具有以下字段：

- **resource**：deviceResource 或 deviceCommand 的名称，指示要读取的内容。
- **Frequency**：一个字符串，表示读取事件之间等待的时间，表示为整数后跟 ms、s、m 或 h 单位。
- **onchange**：布尔值：如果设置为 true，则仅当自上次事件以来包含的一个或多个读数发生更改时才生成新事件。

设备 SDK 应根据这些`AutoEvent`定义从实现中安排设备读取。它应该使用与通过 REST 请求读数时相同的逻辑。



**异步事件队列**

SDK 应提供一种机制，使实现可以随时提交设备读数而不会阻塞。这可以以适合于实现语言的方式来完成，例如，Go SDK提供可以在其上推送读数的通道，C SDK提供将读数提交到工作队列的功能。



