* TOC
{:toc}



## 一、核心元数据微服务



![](/images/edgex/default/core-metadata/metadata-1.png)

核心元数据微服务管理有关设备和传感器的知识。其他服务（设备、命令等）使用此信息与设备进行通信。

具体来说，元数据具有以下能力：

- 管理有关连接到 EdgeX Foundry 并由 EdgeX Foundry 操作的设备的信息
- 了解设备报告的数据的类型和组织
- 知道如何命令设备

尽管元数据具有知识，但它不执行以下活动：

- 它不负责从设备实际收集数据，这是由设备服务和核心数据执行的
- 它不负责向设备发出命令，该命令由核心命令和设备服务执行





## 二、数据模型



### 1.设备配置文件

设备配置文件定义了设备的一般特征、它们提供的数据以及如何命令它们。将设备配置文件视为设备类型或分类的模板。例如，BACnet 恒温器的设备配置文件提供 BACnet 恒温器发送的数据类型的一般特征，例如当前温度和湿度水平。它还定义了 EdgeX 可以发送到 BACnet 恒温器的命令或操作类型。示例可能包括设置冷却或加热点的操作。设备配置文件通常在 YAML 文件中指定并上传到 EdgeX。下面提供了更多详细信息。



**设备配置文件详细信息**

![](/images/edgex/default/core-metadata/metadata-2.png)



#### 1.1.一般属性

设备配置文件具有许多高级属性来提供配置文件上下文和标识。其名称字段是必需的，并且在 EdgeX 部署中必须是唯一的。其他字段是可选的 - 设备服务不使用它们，但可以填充它们以供参考：

- Description
- Manufacturer
- Model
- Labels



以下是随 BACnet 设备服务提供的示例 KMC 9001 BACnet 恒温器设备配置文件的一般信息部分示例（您可以在 Github 中找到该[配置文件](https://github.com/edgexfoundry/device-bacnet-c/blob/v3.1/sample-profiles/BAC-9001.json)）。设备配置文件的此部分仅需要名称。设备配置文件的名称在任何 EdgeX 部署中都必须是唯一的。制造商、型号和标签都是可选的信息位，可以更好地查询系统中的设备配置文件。

```yaml
name: "BAC-9001"
manufacturer: "KMC"
model: "BAC-9001"
labels: 
    - "B-AAC"
description: "KMC BAC-9001 BACnet thermostat"
```

标签提供了一种对各种配置文件进行标记、组织或分类的方法。它们在 EdgeX 内部没有任何实际用途。

[BAC-9001.json](https://github.com/edgexfoundry/device-bacnet-c/blob/v3.1/sample-profiles/BAC-9001.json)

```json
{
  "name": "BAC-9001",
  "manufacturer": "KMC",
  "model": "BAC-9001",
  "labels": [ "B-AAC" ],
  "description": "KMC BAC-9001 BACnet thermostat",
  "deviceResources":
  [
    {
      "name": "Temperature",
      "description": "Get the current temperature",
      "attributes": { "type": "analog-value", "instance": 1, "property": "present-value" },
      "properties": { "valueType": "Float32", "readWrite": "R", "units": "Degrees Fahrenheit" }
    },
    {
      "name": "ActiveCoolingSetpoint",
      "description": "The active cooling set point",
      "attributes": { "type": "analog-value", "instance": 3, "property": "present-value" },
      "properties": { "valueType": "Float32", "readWrite": "RW", "units": "Degrees Fahrenheit" }
    }
  ]
}
```



#### 1.2.设备资源

设备资源（在 YAML 文件的 deviceResources 部分中）指定设备内的传感器值，可以单独或作为设备命令的一部分（见下文）读取或写入该值。将设备资源视为可以从底层设备获取的特定值或可以设置给底层设备的值。在恒温器中，设备资源可以是温度或湿度（从设备感测到的值）或冷却点或加热点（可以设置/启动以允许恒温器确定相关加热/冷却系统何时打开或离开）。设备资源具有用于识别的名称和用于提供信息的描述。

设备资源的属性部分也得到了极大的简化。请参阅下面的详细信息。

回到 BACnet 示例，这里有两个设备资源。一个用于获取（读取）当前温度，另一个用于设置（写入或启动）主动冷却设定点。必须提供设备资源名称，并且它在任何 EdgeX 部署中也必须是唯一的。

```yaml
name: Temperature
description: "Get the current temperature"
isHidden: false

name: ActiveCoolingSetpoint
description: "The active cooling set point"
isHidden: false
```



虽然在此示例中明确表示，`isHidden`但未指定时默认为 false。`isHidden`是否将设备资源暴露给核心命令服务。

设备服务允许通过 REST 端点访问设备资源。可以通过以下 URL 模式访问设备配置文件的设备资源部分中指定的值：

- http://:/api/v3/device/name//



#### 1.3.Attributes

与设备资源关联的属性是设备服务访问特定值所需的特定参数。换句话说，属性是“面向内的”，设备服务使用属性来确定如何与设备对话以读取或写入（获取或设置）其某些值。属性是详细的协议和/或设备特定信息，通知设备服务如何与设备通信以获取（或设置）感兴趣的值。

回到 BACnet 设备配置文件示例，下面是示例设备的温度和 ActiveCoolingSetPoint 的完整设备资源部分（包括属性）。

```yaml
-
    name: Temperature
    description: "Get the current temperature"
    isHidden: false
    attributes: 
        { type: "analogValue", instance: "1", property: "presentValue", index: "none"  }
-
    name: ActiveCoolingSetpoint
    description: "The active cooling set point"
    isHidden: false
    attributes:
        { type: "analogValue", instance: "3", property: "presentValue", index: "no
```



#### 1.4. properties

设备资源的属性描述了在设备上获取或设置的值。这些属性可以选择通知设备服务对值执行一些简单的处理。再次以 BACnet 配置文件为例，以下是与恒温器的温度设备资源关联的属性。

```yaml
name: Temperature
description: "Get the current temperature"
attributes: 
    { type: "analogValue", instance: "1", property: "presentValue", index: "none"  }
properties: 
    valueType: "Float32"
    readWrite: "R"
    units: "Degrees Fahrenheit"
```

属性的“valueType”属性提供了有关收集或设置的值的更多详细信息。在本例中给出了要设置的温度值的详细信息。该值提供详细信息，例如收集或设置的数据的类型、该值是否可以读取、写入或两者兼而有之。

The following fields are available in the value property:

- valueType - Required. The data type of the value. Supported types are Bool, Int8 - Int64, Uint8 - Uint64, Float32, Float64, String, Binary, Object and arrays of the primitive types (ints, floats, bool). Arrays are specified as eg. Float32Array, BoolArray etc.
- readWrite - R, RW, or W indicating whether the value is readable or writable.
- units - gives more detail about the unit of measure associated with the value. In this case, the temperature unit of measure is in degrees Fahrenheit.
- min - minimum allowed value
- max - maximum allowed value
- defaultValue - a value used for PUT requests which do not specify one.
- base - a value to be raised to the power of the raw reading before it is returned.
- scale - a factor by which to multiply a reading before it is returned.
- offset - a value to be added to a reading before it is returned.
- mask - a binary mask which will be applied to an integer reading.
- shift - a number of bits by which an integer reading will be shifted right.

The processing defined by base, scale, offset, mask and shift is applied in that order. This is done within the SDK. A reverse transformation is applied by the SDK to incoming data on set operations (NB mask transforms on set are NYI)



value 属性中提供以下字段：

- valueType - 必需。值的数据类型。支持的类型包括 Bool、Int8 - Int64、Uint8 - Uint64、Float32、Float64、String、Binary、Object 和基本类型数组（int、float、bool）。数组被指定为例如。 Float32Array、BoolArray 等
- readWrite - R、RW 或 W 指示该值是否可读或可写。
- units - 提供有关与值相关的测量单位的更多详细信息。在这种情况下，温度测量单位为华氏度。
- min - 最小允许值
- max - 最大允许值
- defaultValue - 用于未指定的 PUT 请求的值。
- base - 在返回之前要计算原始读数次幂的值。
- scale - 在返回读数之前乘以读数的系数。
- offset - 在返回读数之前要添加到读数的值。
- mask - 将应用于整数读数的二进制掩码。
- shift - 整数读数右移的位数。

由基础、缩放、偏移、掩码和移位定义的处理按该顺序应用。这是在 SDK 内完成的。 SDK 将反向转换应用于集合操作中的传入数据（NB 集合上的掩码转换为 NYI）



#### 1.5.设备命令

设备命令（在 YAML 文件的 deviceCommands 部分中）定义对多个同时设备资源的读取和写入的访问权限。设备命令是可选的。每个命名设备命令应包含许多获取和/或设置资源操作，分别描述读取或写入。

当读数在逻辑上相关时，设备命令可能很有用，例如，对于 3 轴加速度计，一起读取所有轴（X、Y 和 Z）会很有帮助。

设备命令由以下属性组成：

- name - 命令的名称
- readWrite - R、RW 或 W 指示操作是否可读或可写。
- isHidden - 指示是否将设备命令公开给核心命令服务（可选，默认为 false）
- ResourceOperations - 命令中包含的设备资源操作的列表。

每个资源操作将指定：

- deviceResource - 设备资源的名称
- defaultValue - 可选，当操作未提供时返回的值
- 参数 - 可选，如果 PUT 请求未指定该值，则将使用该值。
- 映射 - 可选，允许重新映射字符串类型的读数。

还可以通过设备服务的 REST API 以与设备资源所述类似的方式访问设备命令。

- http://：/api/v3/设备/名称//

如果设备命令和设备资源同名，则设备命令可用。



#### 1.6.核心命令

未隐藏的设备资源或设备命令可通过 EdgeX 核心命令服务查看和使用。

其他服务（例如规则引擎）或 EdgeX 的外部客户端应通过核心命令服务向设备服务发出请求，并且当它们这样做时，它们正在调用设备服务的未隐藏设备命令或设备资源。直接访问设备服务的设备命令或设备资源是不受欢迎的。通过 EdgeX 命令服务提供的命令允许 EdgeX 采用者添加额外的安全性或控制，以控制在实际设备上触发和调用事件的人员/内容/时间。

![](/images/edgex/default/core-metadata/metadata-3.png)



### 2.设备

有关实际设备的数据是元数据微服务存储和管理的另一种类型的信息。 EdgeX Foundry 管理的每个设备都会注册元数据（通过其所属的设备服务）。每个设备必须有一个与其关联的唯一名称。

元数据根据数据库中的名称存储有关设备的信息（例如其地址）。每个设备还与设备配置文件相关联。这种关联使元数据能够将设备配置文件提供的知识应用到每个设备。例如，恒温器配置文件会说它以摄氏度报告温度值。将特定恒温器（例如大厅中的恒温器）与恒温器配置文件相关联允许元数据知道大厅恒温器报告以摄氏度为单位的温度值。

![](/images/edgex/default/core-metadata/metadata-4.png)



### 3.设备服务

元数据还存储和管理有关设备服务的信息。设备服务充当 EdgeX 与实际设备和传感器的接口。

设备服务是通过设备的协议与设备进行通信的其他微服务。例如，Modbus 设备服务促进所有类型的 Modbus 设备之间的通信。 Modbus 设备的示例包括电机控制器、接近传感器、恒温器和功率计。设备服务简化了 EdgeX 其余部分与设备的通信。

当设备服务启动时，它会使用元数据注册自身。当 EdgeX 配置新设备时，该设备会与其所属的设备服务关联。该关联也存储在元数据中。

![](/images/edgex/default/core-metadata/metadata-5.png)



**元数据设备、设备服务和设备配置文件模型**

![](/images/edgex/default/core-metadata/metadata-6.png) 

元数据的设备配置文件、设备和设备服务对象模型以及它们之间的关联



### 4.供给观察者

设备服务可以包含自动供应新设备的逻辑。这可以静态或动态地完成。在静态设备配置（也称为静态配置）中，设备服务连接到并建立一个新设备，并根据提供的设备服务配置在 EdgeX（特别是元数据）中管理该设备。例如，可以向设备服务提供其在启动时要加入的一个或多个设备的特定IP地址和附加设备详细信息。在静态配置中，假设设备将在那里并且在通过配置指定的地址或位置可用。设备和这些设备的连接信息在设备服务启动时就已知。

在动态发现（也称为自动配置）中，设备服务会获得一些有关查找位置的一般信息以及一个或多个设备的一般参数。例如，设备服务可能会被给予一定范围的 BLE 地址空间，并被告知在该范围内寻找某种性质的设备。但是，设备服务并不知道设备实际存在，并且设备在启动时可能不存在。它必须在其操作期间（通常按某种计划）不断扫描配置提供的位置和设备参数指南中的新设备。

并非所有设备服务都支持动态发现。如果它确实支持动态发现，则有关新设备的查找内容和位置（换句话说，扫描位置）的配置由配置观察程序指定。供应观察程序是提供给存储在元数据中的设备服务（通常在启动时）的特定配置信息。除了提供有关在扫描期间查找哪些设备的详细信息之外，配置观察器还可能包含“阻止”指示器，该指示器定义有关不自动配置的设备的参数。这允许缩小设备扫描的范围或允许避开特定设备。

![](/images/edgex/default/core-metadata/metadata-7.png) 

元数据的供应观察者对象模型



### 5.事件和读值

从传感器收集的数据被编组到 EdgeX 事件和读取对象中（作为 JSON 对象或编码为核心数据的CBOR 的二进制对象传递）。一个事件代表一个或多个传感器读值的集合。

读值的数量取决于所连接的设备/传感器。

一个事件必须至少有一个读值。事件与传感器或设备相关，即感知环境并产生的读值。读值是一个事件的组成部分。读值是一个简单的键/值对，其中键（ResourceName）是感测的度量，值是感测到的实际数据。

读值可以包括其他信息位，以便为该数据的用户提供更多上下文（例如，值的数据类型）。读值数据可以被数据可视化系统、分析工具等使用。

例子

来自“motor123”设备的事件有两个读值（或感测值）。第一个读值表明 motor123 设备报告的电机压力为 1300（测量单位可能类似于 PSI）。

![](/images/edgex/default/core-metadata/metadata-8.png)

读值的值类型属性（如上面的类型所示）让信息的使用者知道该值是一个整数，以 64 为基数。第二个读值表明 motor123 设备同时报告的电机温度为 120报告压力的时间（可能以华氏度为单位）。





## 三、设备配置



### 1.设备配置文件

设备配置文件描述了 EdgeX  系统内的一种设备类型。设备服务管理的每个设备都与设备配置文件关联，设备配置文件根据其支持的操作定义该设备类型。

有关设备配置文件字段及其所需值的完整列表，请参阅[设备配置文件参考](https://docs.edgexfoundry.org/3.1/microservices/core/metadata/details/DeviceProfile/#device-profile-reference)。

有关设备配置文件模型及其所有属性的详细信息，请参阅[元数据设备配置文件数据模型](https://docs.edgexfoundry.org/3.1/microservices/core/metadata/GettingStarted/#data-models)。



#### 1.1.Identification

配置文件包含各种标识字段。“名称”字段是必需的，并且在 EdgeX 部署中必须是唯一的。其他字段是可选的-设备服务不使用这些字段，但可以出于提供信息的目的填充这些字段：

- Description
- Manufacturer
- Model
- Labels



#### 1.2.DeviceResources

deviceResource 指定设备内的传感器值，可以单独或作为 deviceCommand 的一部分读取或写入该值。它具有用于识别的名称和用于提供信息的描述。

设备服务允许通过其`device` REST 端点访问 deviceResources。

deviceResource 中的`Attributes`是访问特定值所需的设备服务特定参数。每个设备服务实现都有自己的一组所需的命名值，例如，BACnet 设备服务可能需要对象标识符和属性标识符，而蓝牙设备服务可以使用 UUID 来标识值。

deviceResource`Properties`的 描述该值并可选择请求对其执行一些简单处理。以下字段可用：

- valueType - 必需。值的数据类型。支持的类型有`Bool`、 `Int8`- `Int64`、`Uint8`- `Uint64`、`Float32`、`Float64`、`String`、`Binary`和 `Object`基本类型（int、float、bool）的数组。数组被指定为例如。`Float32Array`，`BoolArray`ETC。
- readWrite - `R`、`RW`、 或`W`指示该值是否可读或可写。
- units - 指示值的单位，例如安培、摄氏度等。
- minimum- 允许 SET 命令的最小值，超出范围将导致错误。
- maximum - 允许 SET 命令的最大值，超出范围将导致错误。
- defaultValue - 用于未指定 SET 命令的值。
- assertion - 与读数（处理后）进行比较的字符串值。如果读数与断言值不同，设备的操作状态将被设置为禁用。这对于健康检查很有用。
- base - 在返回之前要计算原始读数次幂的值。
- scale - 在返回读数之前乘以读数的系数。
- offset - 在返回读数之前要添加到读数的值。
- mask - 将应用于整数读数的二进制掩码。
- offset - 整数读数右移的位数。
- mediaType - 指示值格式的字符串`Binary`。
- optional  - 给定设备资源的可选属性映射。

由基础、缩放、偏移、掩码和移位定义的处理按该顺序应用。这是在 SDK 内完成的。 SDK 将反向转换应用于集合操作中的传入数据（NB 集合上的掩码转换为 NYI）



The `Properties` of a deviceResource describe the value and optionally request some simple processing to be performed on it. The following fields are available:

- valueType - Required. The data type of the value. Supported types are `Bool`, `Int8` - `Int64`, `Uint8` - `Uint64`, `Float32`, `Float64`, `String`, `Binary`, `Object` and arrays of the primitive types (ints, floats, bool). Arrays are specified as eg. `Float32Array`, `BoolArray` etc.
- readWrite - `R`, `RW`, or `W` indicating whether the value is readable or writable.
- units - indicate the units of the value, eg Amperes, degrees C, etc.
- minimum - minimum value a SET command is allowed, out of range will result in error.
- maximum - maximum value a SET command is allowed, out of range will result in error.
- defaultValue - a value used for SET command which do not specify one.
- assertion - a string value to which a reading (after processing) is compared. If the reading is not the same as the assertion value, the device's operating state will be set to disable. This can be useful for health checks.
- base - a value to be raised to the power of the raw reading before it is returned.
- scale - a factor by which to multiply a reading before it is returned.
- offset - a value to be added to a reading before it is returned.
- mask - a binary mask which will be applied to an integer reading.
- shift - a number of bits by which an integer reading will be shifted right.
- mediaType - a string indicating the format of the `Binary` value.
- optional - a optional properties mapping for the given device resource.

The processing defined by base, scale, offset, mask and shift is applied in that order. This is done within the SDK. A reverse transformation is applied by the SDK to incoming data on set operations (NB mask transforms on set are NYI)



#### 1.3.DeviceCommands

DeviceCommands 定义对多个同时设备资源的读取和写入的访问。每个命名的 deviceCommand 应包含多个 `resourceOperations`.

当读数在逻辑上相关时，设备命令可能很有用，例如，对于 3 轴加速度计，一起读取所有轴会很有帮助。

资源操作由以下属性组成：

- deviceResource - 要访问的 deviceResource 的名称。
- defaultValue - 可选，如果 SET 命令未指定该值，则将使用该值。
- mappings - 可选，允许重新映射字符串类型的读数。

设备服务允许通过与`device`用于访问 deviceResources 相同的 REST 端点来访问 deviceCommands。



### 2.设备配置文件参考

#### 2.1.Device Profile

| Field Name      | Type                    | Required? | Notes                                                        |
| :-------------- | :---------------------- | :-------- | :----------------------------------------------------------- |
| name            | String                  | Y         | Must be unique in the EdgeX deployment. Only allow unreserved characters as defined in https://datatracker.ietf.org/doc/html/rfc3986#section-2.3. |
| description     | String                  | N         |                                                              |
| manufacturer    | String                  | N         |                                                              |
| model           | String                  | N         |                                                              |
| labels          | Array of String         | N         |                                                              |
| deviceResources | Array of DeviceResource | Y         |                                                              |
| deviceCommands  | Array of DeviceCommand  | N         |                                                              |



#### 2.2.DeviceResource

| Field Name  | Type                 | Required? | Notes                                                        |
| :---------- | :------------------- | :-------- | :----------------------------------------------------------- |
| name        | String               | Y         | Must be unique in the EdgeX deployment. Only allow unreserved characters as defined in https://datatracker.ietf.org/doc/html/rfc3986#section-2.3. |
| description | String               | N         |                                                              |
| isHidden    | Bool                 | N         | Expose the DeviceResource to Command Service or not, default false |
| tag         | String               | N         |                                                              |
| attributes  | String-Interface Map | N         | Each Device Service should define required and optional keys |
| properties  | ResourceProperties   | Y         |                                                              |



#### 2.3.ResourceProperties

| Field Name   | Type           | Required? | Notes                                                        |
| :----------- | :------------- | :-------- | :----------------------------------------------------------- |
| valueType    | Enum           | Y         | `Uint8`, `Uint16`, `Uint32`, `Uint64`, `Int8`, `Int16`, `Int32`, `Int64`, `Float32`, `Float64`, `Bool`, `String`, `Binary`, `Object`, `Uint8Array`, `Uint16Array`, `Uint32Array`, `Uint64Array`, `Int8Array`, `Int16Array`, `Int32Array`, `Int64Array`, `Float32Array`, `Float64Array`, `BoolArray` |
| readWrite    | Enum           | Y         | `R`, `W`, `RW`                                               |
| units        | String         | N         | Developer is open to define units of value                   |
| minimum      | Float64        | N         | Error if SET command value out of minimum range              |
| maximum      | Float64        | N         | Error if SET command value out of maximum range              |
| defaultValue | String         | N         | If present, should be compatible with the Type field         |
| mask         | Uint64         | N         | Only valid where Type is one of the unsigned integer types   |
| shift        | Int64          | N         | Only valid where Type is one of the unsigned integer types   |
| scale        | Float64        | N         | Only valid where Type is one of the integer or float types   |
| offset       | Float64        | N         | Only valid where Type is one of the integer or float types   |
| base         | Float64        | N         | Only valid where Type is one of the integer or float types   |
| assertion    | String         | N         | String value to which the reading is compared                |
| mediaType    | String         | N         | Only required when valueType is `Binary`                     |
| optional     | String-Any Map | N         | Optional mapping for the given resource                      |



#### 2.4.DeviceCommand

| Field Name         | Type                       | Required? | Notes                                                        |
| :----------------- | :------------------------- | :-------- | :----------------------------------------------------------- |
| name               | String                     | Y         | Must be unique in this profile. A DeviceCommand with a single DeviceResource is redundant unless renaming and/or restricting R/W access. For example DeviceResource is RW, but DeviceCommand is read-only. Only allow unreserved characters as defined in https://datatracker.ietf.org/doc/html/rfc3986#section-2.3. |
| isHidden           | Bool                       | N         | Expose the DeviceCommand to Command Service or not, default false |
| readWrite          | Enum                       | Y         | `R`, `W`, `RW`                                               |
| resourceOperations | Array of ResourceOperation | Y         |                                                              |



#### 2.5.ResourceOperation

| Field Name     | Type              | Required? | Notes                                                        |
| :------------- | :---------------- | :-------- | :----------------------------------------------------------- |
| deviceResource | String            | Y         | Must name a DeviceResource in this profile                   |
| defaultValue   | String            | N         | If present, should be compatible with the Type field of the named DeviceResource |
| mappings       | String-String Map | N         | Map the GET resourceOperation value to another string value  |





