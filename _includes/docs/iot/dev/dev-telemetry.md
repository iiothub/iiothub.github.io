* TOC
{:toc}



## 一、遥测数据规划



### 1.遥测规划

1. **遥测数据上传ThingsBoard平台**
2. **ThingsBoard通过规则引擎整合元数据**
3. **ThingsBoard通过规则引擎按设备类型对数据分类**
4. **ThingsBoard通过规则引擎把消息路由到消息队列（RabbitMQ或Kafka、EMQX），根据设备类型建topic**
5. **消息队列（RabbitMQ）根据topic消费数据，保存到数据库**
6. **根据微服务划分创建数据库，根据时序数据规则创建表**





## 二、准备工作



### 1.上传遥测

```java
package com.iothub.telemetry;
import com.iothub.mqtt.EmqClient;
import com.iothub.mqtt.MqttProperties;
import com.iothub.mqtt.QosEnum;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import javax.annotation.PostConstruct;

@Component
public class Upload {

    @Autowired
    private EmqClient emqClient;

    @Autowired
    private MqttProperties properties;

    @PostConstruct
    public void init(){
        //连接服务端
        emqClient.connect(properties.getUsername(),properties.getPassword());
        //订阅一个主题
        //emqClient.subscribe("testtopic/#", QosEnum.QoS1);
    }

    @Scheduled(fixedRate = 2000)
    public void publish(){

        String data = getData(2);

        emqClient.publish("v1/devices/me/telemetry",data,
                QosEnum.QoS1,false);
    }

    private String getData(Integer type){

        if (type == 1) {
            // 携带时间戳
            String data = "{\n" +
                    "\t\"ts\": 1451649600512,\n" +
                    "\t\"values\": {\n" +
                    "\t\t\"stringKey\": \"value1\",\n" +
                    "\t\t\"booleanKey\": true,\n" +
                    "\t\t\"doubleKey\": 42.0,\n" +
                    "\t\t\"longKey\": 73,\n" +
                    "\t\t\"jsonKey\": {\n" +
                    "\t\t\t\"someNumber\": 42,\n" +
                    "\t\t\t\"someArray\": [1, 2, 3],\n" +
                    "\t\t\t\"someNestedObject\": {\n" +
                    "\t\t\t\t\"key\": \"value\"\n" +
                    "\t\t\t}\n" +
                    "\t\t}\n" +
                    "\t}\n" +
                    "}";

            return data;

        } else if (type == 2) {
            // 不携带时间戳
            String data = "{\n" +
                    "  \"stringKey\": \"value1\",\n" +
                    "  \"booleanKey\": true,\n" +
                    "  \"doubleKey\": 42.0,\n" +
                    "  \"longKey\": 73,\n" +
                    "  \"jsonKey\": {\n" +
                    "    \"someNumber\": 42,\n" +
                    "    \"someArray\": [1,2,3],\n" +
                    "    \"someNestedObject\": {\"key\": \"value\"}\n" +
                    "  }\n" +
                    "}";

            return data;
        }else {
            // 数组
            String data = "[{\"key1\":\"value1\"}, {\"key2\":true}]";
            return data;
        }
    }
}
```



```yaml
server:
  port: 8999
spring:
  application:
    name: tb-telemetry

mqtt:
  broker-url: tcp://82.157.166.86:1883
  client-id: emq-client-telemetry
  username: LuUoLXi3DMqOPqDjm20C
  password:
```



遥测数据

```json
{
    "stringKey": "value1",
    "booleanKey": true,
    "doubleKey": 42.0,
    "longKey": 73,
    "jsonKey": {
        "someNumber": 42,
        "someArray": [1, 2, 3],
        "someNestedObject": {
            "key": "value"
        }
    }
}
```



元数据

```json
{
    "deviceName": "Test-telemetry",
    "deviceType": "default",
    "ts": "1696730983079"
}
```



### 2.RabbitMQ

#### 2.1.Docker部署RabbitMQ

```shell
docker run -d  --network host --restart=always --name rabbitmq rabbitmq:management

# 访问地址
http://82.157.166.86:15672/ 
# 账户/密码：
guest/guest 
```



#### 2.2. 创建直连交换机

**1.创建直连交换机**

![](/images/iot/dev/dev-telemetry/telemetry-1.png)



**2.创建队列**

![](/images/iot/dev/dev-telemetry/telemetry-2.png)



**3.绑定队列**

![](/images/iot/dev/dev-telemetry/telemetry-3.png)



#### 2.3.创建主题交换机

**1.创建topic交换机**

![](/images/iot/dev/dev-telemetry/telemetry-4.png)



**2.创建队列**

![](/images/iot/dev/dev-telemetry/telemetry-5.png)



**3.绑定交换机**

![](/images/iot/dev/dev-telemetry/telemetry-6.png)



### 3.创建规则链

#### 3.1.创建遥测规则链

![](/images/iot/dev/dev-telemetry/telemetry-7.png)

![](/images/iot/dev/dev-telemetry/telemetry-8.png)

![](/images/iot/dev/dev-telemetry/telemetry-9.png)



#### 3.2.整合元数据

```shell
# copy keys

{
    "deviceName": "Test-telemetry",
    "deviceType": "default",
    "ts": "1696730983079"
}
```

![](/images/iot/dev/dev-telemetry/telemetry-10.png)

![](/images/iot/dev/dev-telemetry/telemetry-11.png)



#### 3.3.按设备类型消息分类

```shell
# 按设备类型消息分类

if(msg.deviceType === 'test-telemetry') {
    return ['test-telemetry'];
} else {
    return ['default'];
}
```



![](/images/iot/dev/dev-telemetry/telemetry-12.png)

![](/images/iot/dev/dev-telemetry/telemetry-13.png)





## 三、上传遥测



### 1.创建设备

<img src="/images/iot/dev/dev-telemetry/telemetry-14.png" style="zoom:50%;" />

<img src="/images/iot/dev/dev-telemetry/telemetry-15.png" style="zoom:50%;" />



### 2.上传遥测数据

```json
{
    "stringKey": "value1",
    "booleanKey": true,
    "doubleKey": 42.0,
    "longKey": 73,
    "jsonKey": {
        "someNumber": 42,
        "someArray": [1, 2, 3],
        "someNestedObject": {
            "key": "value"
        }
    }
}


{
	"stringKey": "value1",
	"booleanKey": true,
	"doubleKey": 42.0,
	"longKey": 73
}
```



![](/images/iot/dev/dev-telemetry/telemetry-16.png)



```java
@Component
public class Upload {

    @Autowired
    private EmqClient emqClient;

    @Autowired
    private MqttProperties properties;

    @PostConstruct
    public void init(){
        //连接服务端
        emqClient.connect(properties.getUsername(),properties.getPassword());
        //订阅一个主题
        //emqClient.subscribe("testtopic/#", QosEnum.QoS1);
    }

    @Scheduled(fixedRate = 2000)
    public void publish(){

        String data = getData(2);

        emqClient.publish("v1/devices/me/telemetry",data,
                QosEnum.QoS1,false);
    }

    private String getData(Integer type){

        if (type == 1) {
            // 携带时间戳
            String data = "{\n" +
                    "\t\"ts\": 1451649600512,\n" +
                    "\t\"values\": {\n" +
                    "\t\t\"stringKey\": \"value1\",\n" +
                    "\t\t\"booleanKey\": true,\n" +
                    "\t\t\"doubleKey\": 42.0,\n" +
                    "\t\t\"longKey\": 73,\n" +
                    "\t\t\"jsonKey\": {\n" +
                    "\t\t\t\"someNumber\": 42,\n" +
                    "\t\t\t\"someArray\": [1, 2, 3],\n" +
                    "\t\t\t\"someNestedObject\": {\n" +
                    "\t\t\t\t\"key\": \"value\"\n" +
                    "\t\t\t}\n" +
                    "\t\t}\n" +
                    "\t}\n" +
                    "}";

            return data;
        } else if (type == 2) {
            // 不携带时间戳
            String data = "{\n" +
                    "  \"stringKey\": \"value1\",\n" +
                    "  \"booleanKey\": true,\n" +
                    "  \"doubleKey\": 42.0,\n" +
                    "  \"longKey\": 73,\n" +
                    "  \"jsonKey\": {\n" +
                    "    \"someNumber\": 42,\n" +
                    "    \"someArray\": [1,2,3],\n" +
                    "    \"someNestedObject\": {\"key\": \"value\"}\n" +
                    "  }\n" +
                    "}";

            return data;
        }else {
            // 数组
            String data = "[{\"key1\":\"value1\"}, {\"key2\":true}]";
            return data;
        }
    }
}
```



### 3.RabbitMQ

**1.创建队列**

**telemetry-default-queue、telemetry-queue**

![](/images/iot/dev/dev-telemetry/telemetry-17.png)



**2.创建交换机**

![](/images/iot/dev/dev-telemetry/telemetry-18.png)



### 4.配置规则引擎

#### 4.1.整体配置

![](/images/iot/dev/dev-telemetry/telemetry-19.png)

![](/images/iot/dev/dev-telemetry/telemetry-20.png)

![](/images/iot/dev/dev-telemetry/telemetry-23.png)



#### 4.2.整个元数据

```json
{
    "deviceName": "Test-telemetry",
    "deviceType": "default",
    "ts": "1696730983079"
}
```



![](/images/iot/dev/dev-telemetry/telemetry-21.png)

#### 4.3.按设备类型消息分类

```javascript
if(msg.deviceType === 'test-telemetry') {
    return ['test-telemetry'];
} else {
    return ['default'];
}
```



![](/images/iot/dev/dev-telemetry/telemetry-22.png)



### 5.测试

![](/images/iot/dev/dev-telemetry/telemetry-24.png)

![](/images/iot/dev/dev-telemetry/telemetry-25.png)

![](/images/iot/dev/dev-telemetry/telemetry-26.png)



```json
{
	"stringKey": "value1",
	"booleanKey": true,
	"doubleKey": 42.0,
	"longKey": 73,
	"jsonKey": {
		"someNumber": 42,
		"someArray": [1, 2, 3],
		"someNestedObject": {
			"key": "value"
		}
	},
	"deviceType": "default",
	"deviceName": "Test-telemetry",
	"ts": "1451649600512"
}
```

```json
{
	"stringKey": "value1",
	"booleanKey": true,
	"doubleKey": 42.0,
	"longKey": 73,
	"deviceType": "test-telemetry",
	"deviceName": "Test-telemetry1",
	"ts": "1696824491527"
}
```





## 四、REST API



### 1.遥测API

![](/images/iot/dev/dev-telemetry/telemetry-27.png)

![](/images/iot/dev/dev-telemetry/telemetry-28.png)



