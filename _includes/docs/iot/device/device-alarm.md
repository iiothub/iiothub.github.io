* TOC
{:toc}



## 一、概述



### 1.概念

**以下警报的主要概念：**

**发起者**

警报发起者是警报的实体例如：如果ThingsBoard收到来自它的温度读数并因读数超过阈值而引发“HighTemperature” 警报则设备A是警报的发起者。



**类型**

警报类型有助于确定警报的根本原因例如：”HighTemperature”和”LowHumidity”是两个不同的警报。



**级别**

警报支持级别如下：Critical, Major, Minor, Warning或Indeterminate（按优先级降序排序）。



**生命周期**

ThingsBoard创建警报时可能处于活动或已清除状态并保留**开始**和**结束**时间，警报默认将开始时间和结束时间设置成相同如果警报触发条件重复将更新结束时间，当警报清除条件匹配时自动清除警报，报警清除条件是可选项用户可以手动清除警报。

警报的状态除了有活动和清除外还会跟踪是否已经人为确认过警报通过仪表板或实体详细信息选项卡进行警报确认。

有4个”**状态**“字段：

- 活动未确认(ACTIVE_UNACK) - 警报未清除且尚未确认
- 活动已确认(ACTIVE_ACK) - 警报未清除但已确认
- 清除未确认(CLEARED_UNACK) - 警报已清除但尚未确认
- 清除已确认(CLEARED_ACK) - 警报已清除并确认



**标识**

ThingsBoard根据发起者、类型和开始时间的组合做为警报的判断依据，因此在相同时间点只能有一个相同的发起者、类型和开始时间的活动警报。

假设已配置警报规则以便在温度大于20时创建”HighTemperature”警报；此外还配置了警报规则以便在温度小于或等于20时清除”HighTemperature”警报。

假设事件序列如下：

- 12:00 - 温度等于18
- 12:30 - 温度等于22
- 13:00 - 温度等于25
- 13:30 - 温度等于18

因此应该创建一个”HighTemperature”警报开始时间=12：30结束时间=13：00。



### 2.告警规则

ThingsBoard3.2及以上版本引入警报规则进行简化配置过程而无需通过规则引警进行配置只需要使用”Device Profile”即可，因为在以前的版本中需要一定的编程技巧才能完成。

警报规则包含以下属性：

- **Alarm Type** - 警报类型在规则内唯一标识
- **Create Conditions** - 定义created/updated警报的条件须由以下属性组成：
  - Severity - 用于create/update警报，ThingsBoard按照严重级别的降序验证条件，例如：级别是Critical并且条件为true时只会产生Critical警报并不会产生”Major”、”Minor”或”Warning”条件的警报，每个警报的Severity必须唯一。（例如：同一个警报规则中创建的两个条件不能有相同的Severity）
  - Key Filters - 使用attributes或telemetry的值逻辑表达式,例如：”(temperature < 0 OR temperature > 20) AND softwareVersion = ‘2.5.5’“
  - Condition Type - 简单、持续时间或重复, 例如:如果在连续3次或5分钟内匹配第一个事件，触发简单条件并发出警报
  - Schedule - 定义规则处于活动状态的时间间隔，“始终启用”、“定时启用”或“自定义启用”
  - Details - 通过${attributeName}语法的警报的详细信息模板
- **Clear condition** - 定义清除警报的条件
- **Advanced settings** - 定义警报传播到相关资产、客户、租户或其他实体



### 3.简单报警条件

温度高于10度时创建**Critical**警报。

- 步骤1. 打开设置配置
- 步骤2. 单击警报规则
- 步骤3. 单击警报条件
- 步骤4. 单击过滤条件
- 步骤5. 选择数据键
- 步骤6. 设置条件
- 步骤7. 保存条件
- 步骤8. 应用更改



![](/images/iot/device/device-alarm/alarm-21.png)

![](/images/iot/device/device-alarm/alarm-22.png)

![](/images/iot/device/device-alarm/alarm-23.png)

![](/images/iot/device/device-alarm/alarm-24.png)

![](/images/iot/device/device-alarm/alarm-25.png)

![](/images/iot/device/device-alarm/alarm-26.png)

![](/images/iot/device/device-alarm/alarm-27.png)

![](/images/iot/device/device-alarm/alarm-28.png)



### 4.持续时间的报警条件

假设修改示例1仅当温度超过特定阈值1分钟时才发出警报。

因此需要编辑报警条件并将条件类型从“简单”修改为“持续时间”还应该指定持续时间值和单位。

- 步骤1. 修改条件类型
- 步骤2. 应用更改

![](/images/iot/device/device-alarm/alarm-29.png)

![](/images/iot/device/device-alarm/alarm-30.png)



假设要将1分钟的持续时间替换为指定的设备、客户或租户的设置的动态值。

建议使用[服务端属性](http://www.ithingsboard.com/docs/user-guide/attributes/#server-side-attributes)这个功能。

请为设备创建一个服务器端属性*“highTemperatureDurationThreshold”* 其整数值是 *“1”*。

- 步骤3. 设置动态值
- 步骤4. 选择实体并指定获取警报阈值的属性
- 步骤5. 应用更改



![](/images/iot/device/device-alarm/alarm-31.png)

![](/images/iot/device/device-alarm/alarm-32.png)

![](/images/iot/device/device-alarm/alarm-33.png)



### 5.重复的报警条件

假设我们想修改示例1仅当传感器连续3次报告温度超过阈值时才发出警报。

因此需要编辑报警条件并将条件类型从“简单”修改为“重复”还应该将“3”指定为“事件数量”。

- 步骤1. 修改条件类型
- 步骤2. 应用更改



![](/images/iot/device/device-alarm/alarm-34.png)

![](/images/iot/device/device-alarm/alarm-35.png)



假设要将指定的设备、客户或租户的设置的动态值替换警报条件超出的设定次数。

建议使用[服务端属性](http://www.ithingsboard.com/docs/user-guide/attributes/#server-side-attributes)这个功能。

请为设备创建一个服务器端属性*“highTemperatureRepeatingThreshold”* 其整数值是 *“3”*。

- 步骤4. 设置动态值
- 步骤5. 选择实体并指定获取警报阈值的属性
- 步骤6. 应用更改



![](/images/iot/device/device-alarm/alarm-36.png)

![](/images/iot/device/device-alarm/alarm-37.png)

![](/images/iot/device/device-alarm/alarm-38.png)



### 6.清除警报规则

假设希望温度恢复正常时自动清除警报。

- 步骤1. 单击添加清除条件
- 步骤2. 单击过滤条件
- 步骤3. 选择数据键
- 步骤4. 保存条件
- 步骤5. 应用更改



![](/images/iot/device/device-alarm/alarm-39.png)

![](/images/iot/device/device-alarm/alarm-40.png)

![](/images/iot/device/device-alarm/alarm-41.png)

![](/images/iot/device/device-alarm/alarm-42.png)

![](/images/iot/device/device-alarm/alarm-43.png)



### 7.自定义警报规则时间

假设希望警报规则只在工作时进行预警。

- 步骤1. 编辑警报规则时间
- 步骤2. 选择时间
- 步骤3. 应用更改



![](/images/iot/device/device-alarm/alarm-44.png)

![](/images/iot/device/device-alarm/alarm-45.png)

![](/images/iot/device/device-alarm/alarm-46.png)



### 8.高级

假设我们的用户能够从仪表板UI设置阈值并启用或禁用每个设备的某些警报，因为我们可以在警报规则中使用动态值进行匹配通过布尔值temperatureAlarmFlag和数字temperatureAlarmThreshold两个属性进行控制，然后匹配条件则是”*temperatureAlarmFlag* = True AND *temperature* is greater than *temperatureAlarmThreshold*“同步满足是产生警报。

- 步骤1. 修改过滤动态值
- 步骤2. 选择实体并指定获取警报阈值的属性
- 步骤3. 添加*temperatureAlarmFlag*数据键"
- 步骤4. 应用更改
- 步骤5. 添加属性



![](/images/iot/device/device-alarm/alarm-47.png)

![](/images/iot/device/device-alarm/alarm-48.png)

![](/images/iot/device/device-alarm/alarm-49.png)

![](/images/iot/device/device-alarm/alarm-50.png)

![](/images/iot/device/device-alarm/alarm-51.png)



### 9.租户或客户属性的动态阈值

示例6演示了如何根据设备的“temperatureAlarmFlag”属性值启用或禁用规则，如果想为属于租户或客户的所有设备启用或禁用某些规则怎么办？为避免为每个设备配置属性可以配置警报规则以将常量值与租户或客户属性的值进行比较因此使用“常量”键类型并将其与动态值进行比较。

请参阅下面的配置示例：

- 选择动态值与租户或客户属性进行比较



![](/images/iot/device/device-alarm/alarm-52.png)



### 10.Device profile

设备配置规则节点根据设备配置中定义的警报规则创建和清除警报。 默认情况下这是处理链中的第一个规则节点并对所有输入消息的属性和遥测值做处理。

![image](/images/iot/device/device-alarm/alarm-53.png)


**规则节点中有两个重要的设置：**

**Persist state of alarm rules** - 强制规则节点存储处理状态默认为禁用如果有持续时间或重复条件可以启用此设置；假设有一个条件“温度大于50度并持续1小时”并且在下午1点上报第一个温度大于50度的事件； 下午2点应该会收到警报（假设温度条件不会改变）； 如果将在下午1点之后和下午2点之前重新启动服务器则规则节点需要从数据库中查找状态。

如果启用此选项和‘Fetch state of alarm rules’选项规则节点将能够产生警报否则规则节点将不会产生警报，由于性能原因默认禁用此设置如果需要启用并且输入消息至少符合一个警报条件。

**Fetch state of alarm rules** - 强制规则节点恢复初始化时的处理状态默认为禁用如果有持续时间或重复条件可以启用此设置；必须与’Persist state of alarm rules’选项同时使用，但是在较少情况下可能设置’Persist state of alarm rules’为禁用。

假设有许多频繁或持续发送数据的设备可以避免在初始化时从数据库加载状态当收到指定设备的第一条消息到达时规则节点将从数据库中获取状态。

![](/images/iot/device/device-alarm/alarm-54.png)



### 11.通知

假设已经配置了警报规则还希望在ThingsBoard创建或更新警报时收到通知； 设备配置规则节点有三种可以使用输入关系类型：’Alarm Created’,’Alarm Severity Updated’和’Alarm Cleared’。 请参阅下面的示例规则链继续在规则节点中配置自己的设置请确保系统管理员已配置SMS/电子邮件提供商。

使用现有指南： [发送警报电子邮件](http://www.ithingsboard.com/docs/user-guide/rule-engine-2-0/tutorials/send-email/)（’to email’和’send email’”节点的节能） 或[Telegram通知](http://www.ithingsboard.com/docs/user-guide/rule-engine-2-0/tutorials/integration-with-telegram-bot/)； 还有一个额外的’Alarm Updated’关系类型在大多数情况下应该忽略它以避免重复通知。

![](/images/iot/device/device-alarm/alarm-55.png)





## 二、设备报警



### 1.创建设备

#### 1.1.创建设备配置

![](/images/iot/device/device-alarm/alarm-1.png)

![](/images/iot/device/device-alarm/alarm-2.png)

![](/images/iot/device/device-alarm/alarm-3.png)



#### 1.2.创建设备

![](/images/iot/device/device-alarm/alarm-4.png)



### 2.配置告警规则

#### 2.1.创建告警规则

![](/images/iot/device/device-alarm/alarm-5.png)

![](/images/iot/device/device-alarm/alarm-6.png)

![](/images/iot/device/device-alarm/alarm-7.png)

![](/images/iot/device/device-alarm/alarm-8.png)

![](/images/iot/device/device-alarm/alarm-9.png)



#### 2.2.清除告警规则

![](/images/iot/device/device-alarm/alarm-10.png)

![](/images/iot/device/device-alarm/alarm-11.png)

![](/images/iot/device/device-alarm/alarm-12.png)

![](/images/iot/device/device-alarm/alarm-13.png)

![](/images/iot/device/device-alarm/alarm-14.png)



### 3.测试告警

#### 3.1.设备告警

```shell
{
	"temperature": 62.2,
	"humidity": 79
}
```

![](/images/iot/device/device-alarm/alarm-15.png)

![](/images/iot/device/device-alarm/alarm-16.png)

![](/images/iot/device/device-alarm/alarm-17.png)



#### 3.2.清除告警

```shell
{
	"temperature": 40.2,
	"humidity": 79
}
```

![](/images/iot/device/device-alarm/alarm-18.png)

![](/images/iot/device/device-alarm/alarm-19.png)

![](/images/iot/device/device-alarm/alarm-20.png)



### 4.数据库

**alarm**

![](/images/iot/device/device-alarm/alarm-56.png)



**alarm_comment**

![](/images/iot/device/device-alarm/alarm-57.png)



