* TOC
{:toc}



## 一、创建设备



### 1.创建设备配置

在 ThingsBoard 服务端创建设备配置 test-edge

![](/images/iot/tb-edge/edge-device/device-1.png)

![](/images/iot/tb-edge/edge-device/device-2.png)

![](/images/iot/tb-edge/edge-device/device-3.png)



在 Edge 端查看设备配置

![](/images/iot/tb-edge/edge-device/device-4.png)



### 2.创建设备

在 Edge 端创建设备 edge-device

![](/images/iot/tb-edge/edge-device/device-5.png)

![](/images/iot/tb-edge/edge-device/device-6.png)

![](/images/iot/tb-edge/edge-device/device-7.png)



在服务端查看设备

![](/images/iot/tb-edge/edge-device/device-8.png)



```shell
# 访问令牌
lMrdczEw1rJHhBejzumZ
```





## 二、上传遥测



### 1.MQTTX 工具

![](/images/iot/tb-edge/edge-device/device-9.png)



### 2.上传遥测

```shell
# 发布主题
v1/devices/me/telemetry


# 发布数据
{
  "stringKey": "value1",
  "booleanKey": true,
  "doubleKey": 42.0,
  "longKey": 73,
  "jsonKey": {
    "someNumber": 42,
    "someArray": [1,2,3],
    "someNestedObject": {"key": "value"}
  }
}
```



- **发送遥测数据**

![](/images/iot/tb-edge/edge-device/device-10.png)



- **Edge 端遥测数据**

![](/images/iot/tb-edge/edge-device/device-11.png)



- **ThingsBoard 服务端遥测数据**

![](/images/iot/tb-edge/edge-device/device-12.png)





## 三、属性



### 1.属性类型

**属性主要分为三种:**

- 服务端属性：服务端定义，服务端使用，设备端不能使用
- 共享属性：服务端定义，设备端可以使用，不能修改
- 客户端属性：设备端定义属性，服务端可以使用，不能修改



**1.服务端属性**

![](/images/iot/tb-edge/edge-device/a-1.svg)



**2.共享属性**

![](/images/iot/tb-edge/edge-device/a-2.svg)



**3.客户端属性**

![](/images/iot/tb-edge/edge-device/a-3.svg)



### 2.上传客户端属性

```java
package com.iothub.attribute;
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

        String data = getData();

        emqClient.publish("v1/devices/me/attributes",data,
                QosEnum.QoS1,false);
    }

    private String getData(){

        String data = "{\n" +
                "\t\"attribute1\": \"value1\",\n" +
                "\t\"attribute2\": true,\n" +
                "\t\"attribute3\": 42.0,\n" +
                "\t\"attribute4\": 73,\n" +
                "\t\"attribute5\": {\n" +
                "\t\t\"someNumber\": 42,\n" +
                "\t\t\"someArray\": [1, 2, 3],\n" +
                "\t\t\"someNestedObject\": {\n" +
                "\t\t\t\"key\": \"value\"\n" +
                "\t\t}\n" +
                "\t}\n" +
                "}";

        return data;
    }
}
```



- Edge 端显示

![](/images/iot/tb-edge/edge-device/device-13.png)



- 服务端显示

![](/images/iot/tb-edge/edge-device/device-14.png)



### 3.下载共享属性

```java
package com.iothub.attribute;
import com.iothub.mqtt.EmqClient;
import com.iothub.mqtt.MqttProperties;
import com.iothub.mqtt.QosEnum;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import javax.annotation.PostConstruct;

//@Component
public class Download {

    @Autowired
    private EmqClient emqClient;

    @Autowired
    private MqttProperties properties;

    @PostConstruct
    public void init(){
        //连接服务端
        emqClient.connect(properties.getUsername(),properties.getPassword());
        //订阅一个主题
        emqClient.subscribe("v1/devices/me/attributes/response/+", QosEnum.QoS1);
    }

    @Scheduled(fixedRate = 2000)
    public void publish(){

        String data = getData();

        emqClient.publish("v1/devices/me/attributes/request/1",data, QosEnum.QoS1,false);
    }

    private String getData(){

        String data = "{\n" +
                "\t\"clientKeys\": \"attribute1,attribute2\",\n" +
                "\t\"sharedKeys\": \"shared1,shared2\"\n" +
                "}";

        return data;
    }
}
```



**共享属性需要在 Edge 端创建，会同步到服务端**

![](/images/iot/tb-edge/edge-device/device-16.png)

![](/images/iot/tb-edge/edge-device/device-17.png)



### 4.订阅共享数据

```java
package com.iothub.attribute;
import com.iothub.mqtt.EmqClient;
import com.iothub.mqtt.MqttProperties;
import com.iothub.mqtt.QosEnum;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import javax.annotation.PostConstruct;

//@Component
public class Subscribe {
    @Autowired
    private EmqClient emqClient;

    @Autowired
    private MqttProperties properties;

    @PostConstruct
    public void init(){
        //连接服务端
        emqClient.connect(properties.getUsername(),properties.getPassword());
        //订阅一个主题
        emqClient.subscribe("v1/devices/me/attributes", QosEnum.QoS1);
    }

    @Scheduled(fixedRate = 2000)
    public void publish(){

        String data = getData();

        //emqClient.publish("v1/devices/me/attributes/request/1",data, QosEnum.QoS1,false);
    }

    private String getData(){

        String data = "{\n" +
                "\t\"clientKeys\": \"attribute1,attribute2\",\n" +
                "\t\"sharedKeys\": \"shared1,shared2\"\n" +
                "}";

        return data;
    }
}
```





## 四、设备告警



### 1.配置告警规则

在 ThingsBoard 服务端创建告警规则

![](/images/iot/tb-edge/edge-device/device-18.png)

![](/images/iot/tb-edge/edge-device/device-19.png)

![](/images/iot/tb-edge/edge-device/device-20.png)

![](/images/iot/tb-edge/edge-device/device-21.png)

![](/images/iot/tb-edge/edge-device/device-22.png)



### 2.清除报警规则

![](/images/iot/tb-edge/edge-device/device-23.png)

![](/images/iot/tb-edge/edge-device/device-24.png)

![](/images/iot/tb-edge/edge-device/device-25.png)

- Edge 端查看告警规则

![](/images/iot/tb-edge/edge-device/device-26.png)



### 3.测试

```shell
v1/devices/me/telemetry


{
	"temperature": 62.2,
	"humidity": 79
}
```



#### 3.1.设备告警

```shell
{
	"temperature": 62.2,
	"humidity": 79
}
```

![](/images/iot/tb-edge/edge-device/device-27.png)

- Edge 端告警

![](/images/iot/tb-edge/edge-device/device-28.png)

- 服务端告警

![](/images/iot/tb-edge/edge-device/device-29.png)



#### 3.1.清除告警

```shell
{
	"temperature": 42.2,
	"humidity": 79
}
```



![](/images/iot/tb-edge/edge-device/device-30.png)

![](/images/iot/tb-edge/edge-device/device-31.png)





## 五、规则链



### 1.规则管理

在服务端创建、修改规则链

![](/images/iot/tb-edge/edge-device/device-32.png)

![](/images/iot/tb-edge/edge-device/device-33.png)



### 2.Edge 查看规则链

![](/images/iot/tb-edge/edge-device/device-34.png)

![](/images/iot/tb-edge/edge-device/device-35.png)



