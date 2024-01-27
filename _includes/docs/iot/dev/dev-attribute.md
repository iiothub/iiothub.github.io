* TOC
{:toc}



## 一、属性规划



### 1.属性类型

**属性主要分为三种:**

- 服务端属性：服务端定义，服务端使用，设备端不能使用
- 共享属性：服务端定义，设备端可以使用，不能修改
- 客户端属性：设备端定义属性，服务端可以使用，不能修改



**1.服务端属性**

![img](/images/iot/dev/dev-attribute/a-1.svg)



**2.共享属性**

![img](/images/iot/dev/dev-attribute/a-2.svg)



**3.客户端属性**

![img](/images/iot/dev/dev-attribute/a-3.svg)



### 2.属性规划

1. **通过REST API对属性进行增、删、改、查，需评估**
2. **属性增、删、改，会发送消息；**
3. **属性变化的消息通过规则引擎发送到消息队列（RabbitMQ或Kafka、EMQX）**





## 二、准备工作



### 1.RabbitMQ

**1.创建队列**

![](/images/iot/dev/dev-attribute/attribute-1.png)

**2.创建交换机**

![](/images/iot/dev/dev-attribute/attribute-2.png)

![](/images/iot/dev/dev-attribute/attribute-3.png)



### 2.规则引擎

#### 2.1.创建规则链

![](/images/iot/dev/dev-attribute/attribute-4.png)

![](/images/iot/dev/dev-attribute/attribute-5.png)

![](/images/iot/dev/dev-attribute/attribute-18.png)



#### 2.2.整合设备信息

```json
{
    "sharedKey": "xxxx"
}

# 元数据
{
    "deviceData": "{\"configuration\":{\"type\":\"DEFAULT\"},\"transportConfiguration\":{\"type\":\"DEFAULT\"}}",
    "originatorId": "671878a0-6747-11ee-afb9-c790163a721a",
    "originatorName": "Test-attribute",
    "originatorType": "default",
    
    
    "scope": "SHARED_SCOPE",
    "userEmail": "tenant@thingsboard.org",
    "userId": "08f69680-64f4-11ee-b6d5-bdc7c43c6c8f",
    "userName": "tenant@thingsboard.org"
}
```



![](/images/iot/dev/dev-attribute/attribute-6.png)



#### 2.3.添加消息类型

```javascript
msg.msgType = msgType;
return {msg: msg, metadata: metadata, msgType: msgType};

{
    "sharedKey": "xxxx",
    "msgType": "ATTRIBUTES_UPDATED"
}
```



![](/images/iot/dev/dev-attribute/attribute-7.png)



#### 2.4.整合元数据

```json
{
    "sharedKey": "yyyyy",
    "msgType": "ATTRIBUTES_UPDATED",
    "originatorName": "Test-attribute",
    "scope": "SHARED_SCOPE",
    "userName": "tenant@thingsboard.org",
    "originatorType": "default"
}


{
    "deviceData": "{\"configuration\":{\"type\":\"DEFAULT\"},\"transportConfiguration\":{\"type\":\"DEFAULT\"}}",
    "originatorId": "671878a0-6747-11ee-afb9-c790163a721a",
    "originatorName": "Test-attribute",
    "originatorType": "default",
    "scope": "SHARED_SCOPE",
    "userEmail": "tenant@thingsboard.org",
    "userId": "08f69680-64f4-11ee-b6d5-bdc7c43c6c8f",
    "userName": "tenant@thingsboard.org"
}
```



![](/images/iot/dev/dev-attribute/attribute-9.png)

#### 2.5.RabbitMQ

![](/images/iot/dev/dev-attribute/attribute-8.png)





## 三、设备（MQTT）



### 1.上传客户端属性

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



### 2.下载服务端属性

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



### 3.订阅共享属性

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



## 四、属性更新测试



### 1.上传客户端属性

```js
# 数据

{
    "attribute1": "value1",
    "attribute2": true,
    "attribute3": 42,
    "attribute4": 73,
    "attribute5": {
        "someNumber": 42,
        "someArray": [1, 2, 3],
        "someNestedObject": {
            "key": "value"
        }
    },
    "msgType": "POST_ATTRIBUTES_REQUEST",
    "originatorName": "Test-attribute",
    "deviceData": "{\"configuration\":{\"type\":\"DEFAULT\"},\"transportConfiguration\":{\"type\":\"DEFAULT\"}}",
    "originatorId": "671878a0-6747-11ee-afb9-c790163a721a",
    "originatorType": "default"
}


{
	"attribute1": "value1",
	"attribute2": true,
	"attribute3": 42,
	"attribute4": 73,
	"attribute5": {
		"someNumber": 42,
		"someArray": [1, 2, 3],
		"someNestedObject": {
			"key": "value"
		}
	},
	"msgType": "POST_ATTRIBUTES_REQUEST",
	"originatorName": "Test-attribute",
	"deviceData": "{\"configuration\":{\"type\":\"DEFAULT\"},\"transportConfiguration\":{\"type\":\"DEFAULT\"}}",
	"originatorId": "671878a0-6747-11ee-afb9-c790163a721a",
	"originatorType": "default"
}
```

```json
# 元数据

{
    "deviceData": "{\"configuration\":{\"type\":\"DEFAULT\"},\"transportConfiguration\":{\"type\":\"DEFAULT\"}}",
    "deviceName": "Test-attribute",
    "deviceType": "default",
    "notifyDevice": "false",
    "originatorId": "671878a0-6747-11ee-afb9-c790163a721a",
    "originatorName": "Test-attribute",
    "originatorType": "default"
}
```

![](/images/iot/dev/dev-attribute/attribute-10.png)



### 2.创建服务端属性

```json
# 数据

{
    "serverKey": "aaaaa",
    "msgType": "ATTRIBUTES_UPDATED",
    "originatorName": "Test-attribute",
    "scope": "SERVER_SCOPE",
    "deviceData": "{\"configuration\":{\"type\":\"DEFAULT\"},\"transportConfiguration\":{\"type\":\"DEFAULT\"}}",
    "originatorId": "671878a0-6747-11ee-afb9-c790163a721a",
    "userName": "tenant@thingsboard.org",
    "originatorType": "default"
}
```

```json
{
    "serverKey": "aaaaa",
    "msgType": "ATTRIBUTES_UPDATED",
    "originatorName": "Test-attribute",
    "scope": "SERVER_SCOPE",
    "deviceData": "{\"configuration\":{\"type\":\"DEFAULT\"},\"transportConfiguration\":{\"type\":\"DEFAULT\"}}",
    "originatorId": "671878a0-6747-11ee-afb9-c790163a721a",
    "userName": "tenant@thingsboard.org",
    "originatorType": "default"
}
```



![](/images/iot/dev/dev-attribute/attribute-20.png)



### 3.删除服务端属性

```json
# 数据

{
    "attributes": ["serverKey"],
    "msgType": "ATTRIBUTES_DELETED",
    "originatorName": "Test-attribute",
    "scope": "SERVER_SCOPE",
    "deviceData": "{\"configuration\":{\"type\":\"DEFAULT\"},\"transportConfiguration\":{\"type\":\"DEFAULT\"}}",
    "originatorId": "671878a0-6747-11ee-afb9-c790163a721a",
    "userName": "tenant@thingsboard.org",
    "originatorType": "default"
}
```

```json
# 元数据

{
    "deviceData": "{\"configuration\":{\"type\":\"DEFAULT\"},\"transportConfiguration\":{\"type\":\"DEFAULT\"}}",
    "originatorId": "671878a0-6747-11ee-afb9-c790163a721a",
    "originatorName": "Test-attribute",
    "originatorType": "default",
    "scope": "SERVER_SCOPE",
    "userEmail": "tenant@thingsboard.org",
    "userId": "08f69680-64f4-11ee-b6d5-bdc7c43c6c8f",
    "userName": "tenant@thingsboard.org"
}
```



### 4.创建共享属性

```json
# 数据

{
    "sharedKey": 33333,
    "msgType": "ATTRIBUTES_UPDATED",
    "originatorName": "Test-attribute",
    "scope": "SHARED_SCOPE",
    "deviceData": "{\"configuration\":{\"type\":\"DEFAULT\"},\"transportConfiguration\":{\"type\":\"DEFAULT\"}}",
    "originatorId": "671878a0-6747-11ee-afb9-c790163a721a",
    "userName": "tenant@thingsboard.org",
    "originatorType": "default"
}
```

```json
# 元数据

{
    "deviceData": "{\"configuration\":{\"type\":\"DEFAULT\"},\"transportConfiguration\":{\"type\":\"DEFAULT\"}}",
    "originatorId": "671878a0-6747-11ee-afb9-c790163a721a",
    "originatorName": "Test-attribute",
    "originatorType": "default",
    "scope": "SHARED_SCOPE",
    "userEmail": "tenant@thingsboard.org",
    "userId": "08f69680-64f4-11ee-b6d5-bdc7c43c6c8f",
    "userName": "tenant@thingsboard.org"
}
```





## 五、REST API



### 1.属性REST API

![](/images/iot/dev/dev-attribute/attribute-21.png)



### 2.创建、更新设备属性

![](/images/iot/dev/dev-attribute/attribute-11.png)



### 3.删除设备属性

![](/images/iot/dev/dev-attribute/attribute-12.png)



### 4.获得实体属性Key

![](/images/iot/dev/dev-attribute/attribute-13.png)

![](/images/iot/dev/dev-attribute/attribute-14.png)



### 5.获得实体属性

![](/images/iot/dev/dev-attribute/attribute-15.png)

![](/images/iot/dev/dev-attribute/attribute-16.png)





## 六、REST API测试



### 1.创建、更新设备属性

```shell
# url
/api/plugins/telemetry/{deviceId}/{scope}

# 设备ID
671878a0-6747-11ee-afb9-c790163a721a

#scope 
CLIENT_SCOPE, SERVER_SCOPE, SHARED_SCOPE


{
 "stringKey":"value1", 
 "booleanKey":true, 
 "doubleKey":42.0, 
 "longKey":73, 
 "jsonKey": {
    "someNumber": 42,
    "someArray": [1,2,3],
    "someNestedObject": {"key": "value"}
 }
}

http://localhost:8999/attri/updatedevice
```



![](/images/iot/dev/dev-attribute/attribute-17.png)



```java
    public void deviceUpdeate(){
        String baseURL = "http://82.157.166.86:8080";
        String token = Login.getToken();

        // 地址：/api/plugins/telemetry/{deviceId}/{scope}
        // 设备ID：  671878a0-6747-11ee-afb9-c790163a721a
        String apiUrl = "/api/plugins/telemetry/{deviceId}/{scope}";
        String deviceId = "671878a0-6747-11ee-afb9-c790163a721a";
        HttpHeaders headers = new HttpHeaders();
        //headers.add("Authorization", "Bearer "+ mainToken);
        headers.set("X-Authorization", "Bearer " + token);
        //headers.set("Content-Type", "application/json");
        headers.setContentType(MediaType.APPLICATION_JSON);

        String body = "{\n" +
                " \"stringKey\":\"value1\", \n" +
                " \"booleanKey\":true, \n" +
                " \"doubleKey\":42.0, \n" +
                " \"longKey\":73, \n" +
                " \"jsonKey\": {\n" +
                "    \"someNumber\": 42,\n" +
                "    \"someArray\": [1,2,3],\n" +
                "    \"someNestedObject\": {\"key\": \"value\"}\n" +
                " }\n" +
                "}";

        HttpEntity entity = new HttpEntity<>(body, headers);
        RestTemplate restTemplate = new RestTemplate();
        ResponseEntity response = restTemplate.exchange(baseURL + apiUrl, HttpMethod.POST, entity, String.class, new Object[]{deviceId,"SHARED_SCOPE"});
        System.out.println(response);
        if (response.getStatusCodeValue() == 200) {
            String device = (String) response.getBody();
            System.out.println(response.getBody());
        }
    }
```



### 2.删除设备属性

```shell
# url
/api/plugins/telemetry/{deviceId}/{scope}{?keys}
/api/plugins/telemetry/{deviceId}/SHARED_SCOPE?keys=stringKey

# 设备ID
671878a0-6747-11ee-afb9-c790163a721a

#scope 
CLIENT_SCOPE, SERVER_SCOPE, SHARED_SCOPE


sharedKey

http://localhost:8999/attri/deletedevice
```



```java
    public void deviceDelete(){

        String baseURL = "http://82.157.166.86:8080";
        String token = Login.getToken();

        // 地址：/api/plugins/telemetry/{deviceId}/{scope}{?keys}
        // 设备ID：  671878a0-6747-11ee-afb9-c790163a721a
        String apiUrl = "/api/plugins/telemetry/{deviceId}/SHARED_SCOPE?keys=stringKey";
        //String apiUrl = "/api/plugins/telemetry/{deviceId}/{scope}{?keys}";
        String deviceId = "671878a0-6747-11ee-afb9-c790163a721a";
        HttpHeaders headers = new HttpHeaders();
        //headers.add("Authorization", "Bearer "+ mainToken);
        headers.set("X-Authorization", "Bearer " + token);
        //headers.set("Content-Type", "application/json");
        headers.setContentType(MediaType.APPLICATION_JSON);

        //String keys = "sharedKey";
        Map<String, Object> paramMap = new HashMap<>();
        paramMap.put("deviceId", deviceId);
//        paramMap.put("scope", "SHARED_SCOPE");
//        paramMap.put("keys", "?keys=stringKey");


        HttpEntity entity = new HttpEntity<>(headers);
        RestTemplate restTemplate = new RestTemplate();
        //ResponseEntity response = restTemplate.exchange(baseURL+apiUrl, HttpMethod.DELETE, entity, String.class, new Object[]{deviceId,"SHARED_SCOPE","keys=stringKey"});
        ResponseEntity response = restTemplate.exchange(baseURL+apiUrl, HttpMethod.DELETE, entity, String.class, paramMap);
        System.out.println(response);
        if (response.getStatusCodeValue() == 200) {
            String device = (String) response.getBody();
            System.out.println(response.getBody());
        }
    }
```



### 3.删除实体属性

```shell
# url
/api/plugins/telemetry/{entityType}/{entityId}/{scope}{?keys}
/api/plugins/telemetry/{entityType}/{entityId}/CLIENT_SCOPE?keys=attribute4

# 设备ID
671878a0-6747-11ee-afb9-c790163a721a

# entityType 
DEVICE

#scope 
CLIENT_SCOPE, SERVER_SCOPE, SHARED_SCOPE


sharedKey


http://localhost:8999/attri/deleteentity
```



```java
    public void entityDelete() {

        String baseURL = "http://82.157.166.86:8080";
        String token = Login.getToken();

        // 地址：/api/plugins/telemetry/{entityType}/{entityId}/{scope}{?keys}
        // 设备ID：  671878a0-6747-11ee-afb9-c790163a721a
        String apiUrl = "/api/plugins/telemetry/{entityType}/{entityId}/CLIENT_SCOPE?keys=attribute4";
        String deviceId = "671878a0-6747-11ee-afb9-c790163a721a";
        HttpHeaders headers = new HttpHeaders();
        //headers.add("Authorization", "Bearer "+ mainToken);
        headers.set("X-Authorization", "Bearer " + token);
        //headers.set("Content-Type", "application/json");
        headers.setContentType(MediaType.APPLICATION_JSON);

        //String keys = "sharedKey";
        Map<String, Object> paramMap = new HashMap<>();
        paramMap.put("entityType", "DEVICE");
        paramMap.put("entityId", deviceId);


        HttpEntity entity = new HttpEntity<>(headers);
        RestTemplate restTemplate = new RestTemplate();
        //ResponseEntity response = restTemplate.exchange(baseURL+apiUrl, HttpMethod.DELETE, entity, String.class, new Object[]{deviceId,"SHARED_SCOPE","sharedKey"});
        ResponseEntity response = restTemplate.exchange(baseURL+apiUrl, HttpMethod.DELETE, entity, String.class, paramMap);
        System.out.println(response);
        if (response.getStatusCodeValue() == 200) {
            String device = (String) response.getBody();
            System.out.println(response.getBody());
        }
    }
```



### 4.获得属性keys

```shell
# url
/api/plugins/telemetry/{entityType}/{entityId}/keys/attributes

# 设备ID
671878a0-6747-11ee-afb9-c790163a721a

# entityType 
DEVICE

#scope 
CLIENT_SCOPE, SERVER_SCOPE, SHARED_SCOPE


http://localhost:8999/attri/getentitykeys


# 只能获取以下服务端属性
["lastConnectTime","lastDisconnectTime","lastActivityTime","active","inactivityAlarmTime"]
与swagger测试不符
```



```java
    public void getEntityKeys(){

        String baseURL = "http://82.157.166.86:8080";
        String token = Login.getToken();

        // 地址：/api/plugins/telemetry/{entityType}/{entityId}/keys/attributes
        // 设备ID  452fa1a0-3768-11ee-8f31-b92b1a081d15
        String deviceId = "bbac67d0-6519-11ee-afb9-c790163a721a";
        String apiUrl = "/api/plugins/telemetry/{entityType}/{entityId}/keys/attributes";
        HttpHeaders headers = new HttpHeaders();
        //headers.add("Authorization", "Bearer "+ mainToken);
        headers.set("X-Authorization", "Bearer "+ token);
        //headers.set("Content-Type", "application/json");
        headers.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity entity = new HttpEntity<>(headers);

        Map<String, Object> paramMap = new HashMap<>();
        paramMap.put("entityType", "DEVICE");
        paramMap.put("entityId", deviceId);

        RestTemplate restTemplate = new RestTemplate();
        //ResponseEntity response = restTemplate.exchange(baseURL+apiUrl, HttpMethod.GET, entity, String.class, new Object[]{"DEVICE",deviceId});
        ResponseEntity response = restTemplate.exchange(baseURL+apiUrl, HttpMethod.GET, entity, String.class, paramMap);
        System.out.println(response);
        if( response.getStatusCodeValue() == 200 ){
            String device = (String)response.getBody();
            System.out.println(response.getBody());

        }
    }
```



### 5.获得属性keys（scope）

```shell
# url
/api/plugins/telemetry/{entityType}/{entityId}/keys/attributes/{scope}

# 设备ID
671878a0-6747-11ee-afb9-c790163a721a

# entityType 
DEVICE

#scope 
CLIENT_SCOPE, SERVER_SCOPE, SHARED_SCOPE


http://localhost:8999/attri/getentitykeysscope


# 只能获取以下服务端属性
["lastConnectTime","lastDisconnectTime","lastActivityTime","active","inactivityAlarmTime"]
#共享属性、客户端属性获取不了
与swagger测试不符
```



```java
    public void getEntityKeysScope(){
        String baseURL = "http://82.157.166.86:8080";
        String token = Login.getToken();

        // 地址：/api/plugins/telemetry/{entityType}/{entityId}/keys/attributes
        // 设备ID  452fa1a0-3768-11ee-8f31-b92b1a081d15
        String deviceId = "bbac67d0-6519-11ee-afb9-c790163a721a";
        String apiUrl = "/api/plugins/telemetry/{entityType}/{entityId}/keys/attributes/{scope}";
        HttpHeaders headers = new HttpHeaders();
        //headers.add("Authorization", "Bearer "+ mainToken);
        headers.set("X-Authorization", "Bearer "+ token);
        //headers.set("Content-Type", "application/json");
        headers.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity entity = new HttpEntity<>(headers);

        Map<String, Object> paramMap = new HashMap<>();
        paramMap.put("entityType", "DEVICE");
        paramMap.put("entityId", deviceId);
        paramMap.put("scope", "SHARED_SCOPE");

        RestTemplate restTemplate = new RestTemplate();
        //ResponseEntity response = restTemplate.exchange(baseURL+apiUrl, HttpMethod.GET, entity, String.class, new Object[]{"DEVICE",deviceId});
        ResponseEntity response = restTemplate.exchange(baseURL+apiUrl, HttpMethod.GET, entity, String.class, paramMap);
        System.out.println(response);
        if( response.getStatusCodeValue() == 200 ){
            String device = (String)response.getBody();
            System.out.println(response.getBody());

        }

    }
```



### 6.获得属性

```shell
# url
/api/plugins/telemetry/{entityType}/{entityId}/values/attributes{?keys}
/api/plugins/telemetry/{entityType}/{entityId}/values/attributes?keys=active,testKey

# 设备ID
671878a0-6747-11ee-afb9-c790163a721a

# entityType 
DEVICE

#scope 
CLIENT_SCOPE, SERVER_SCOPE, SHARED_SCOPE


http://localhost:8999/attri/getentityattri


# 获取以下属性
[{"lastUpdateTs":1696903103362,"key":"active","value":false}]



# 例子
/api/plugins/telemetry/{entityType}/{entityId}/values/attributes
[{
	"lastUpdateTs": 1696902420986,
	"key": "lastConnectTime",
	"value": 1696902420986
}, {
	"lastUpdateTs": 1696902453145,
	"key": "lastDisconnectTime",
	"value": 1696902453145
}, {
	"lastUpdateTs": 1696902455853,
	"key": "lastActivityTime",
	"value": 1696902453130
}, {
	"lastUpdateTs": 1696903103362,
	"key": "active",
	"value": false
}, {
	"lastUpdateTs": 1696903103362,
	"key": "inactivityAlarmTime",
	"value": 1696903103362
}]
```



```java
    public void getEntityAttri(){
        String baseURL = "http://82.157.166.86:8080";
        String token = Login.getToken();

        // 地址：/api/plugins/telemetry/{entityType}/{entityId}/values/attributes{?keys}
        // 设备ID  452fa1a0-3768-11ee-8f31-b92b1a081d15
        String deviceId = "bbac67d0-6519-11ee-afb9-c790163a721a";
        //String apiUrl = "/api/plugins/telemetry/{entityType}/{entityId}/values/attributes?keys=active,testKey";
        String apiUrl = "/api/plugins/telemetry/{entityType}/{entityId}/values/attributes";
        HttpHeaders headers = new HttpHeaders();
        //headers.add("Authorization", "Bearer "+ mainToken);
        headers.set("X-Authorization", "Bearer "+ token);
        //headers.set("Content-Type", "application/json");
        headers.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity entity = new HttpEntity<>(headers);

        Map<String, Object> paramMap = new HashMap<>();
        paramMap.put("entityType", "DEVICE");
        paramMap.put("entityId", deviceId);


        RestTemplate restTemplate = new RestTemplate();
        //ResponseEntity response = restTemplate.exchange(baseURL+apiUrl, HttpMethod.GET, entity, String.class, new Object[]{"DEVICE",deviceId});
        ResponseEntity response = restTemplate.exchange(baseURL+apiUrl, HttpMethod.GET, entity, String.class, paramMap);
        System.out.println(response);
        if( response.getStatusCodeValue() == 200 ){
            String device = (String)response.getBody();
            System.out.println(response.getBody());

        }

    }
```



### 7.获得属性（scope）

```shell
# url
/api/plugins/telemetry/{entityType}/{entityId}/values/attributes/{scope}{?keys}
/api/plugins/telemetry/{entityType}/{entityId}/values/attributes/SERVER_SCOPE?keys=active,testKey

# 设备ID
671878a0-6747-11ee-afb9-c790163a721a

# entityType 
DEVICE

#scope 
CLIENT_SCOPE, SERVER_SCOPE, SHARED_SCOPE


http://localhost:8999/attri/getentityattriscope


# 获取以下属性
[{"lastUpdateTs":1696903103362,"key":"active","value":false}]


# 例子
String apiUrl = "/api/plugins/telemetry/{entityType}/{entityId}/values/attributes/SERVER_SCOPE";
[{
	"lastUpdateTs": 1696902420986,
	"key": "lastConnectTime",
	"value": 1696902420986
}, {
	"lastUpdateTs": 1696902453145,
	"key": "lastDisconnectTime",
	"value": 1696902453145
}, {
	"lastUpdateTs": 1696902455853,
	"key": "lastActivityTime",
	"value": 1696902453130
}, {
	"lastUpdateTs": 1696903103362,
	"key": "active",
	"value": false
}, {
	"lastUpdateTs": 1696903103362,
	"key": "inactivityAlarmTime",
	"value": 1696903103362
}]
```



```java
    public void getEntityAttriScope(){
        String baseURL = "http://82.157.166.86:8080";
        String token = Login.getToken();

        // 地址：/api/plugins/telemetry/{entityType}/{entityId}/values/attributes/{scope}{?keys}
        // 设备ID  452fa1a0-3768-11ee-8f31-b92b1a081d15
        String deviceId = "bbac67d0-6519-11ee-afb9-c790163a721a";
        //String apiUrl = "/api/plugins/telemetry/{entityType}/{entityId}/values/attributes/SERVER_SCOPE?keys=active,testKey";
        String apiUrl = "/api/plugins/telemetry/{entityType}/{entityId}/values/attributes/SERVER_SCOPE";
        HttpHeaders headers = new HttpHeaders();
        //headers.add("Authorization", "Bearer "+ mainToken);
        headers.set("X-Authorization", "Bearer "+ token);
        //headers.set("Content-Type", "application/json");
        headers.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity entity = new HttpEntity<>(headers);

        Map<String, Object> paramMap = new HashMap<>();
        paramMap.put("entityType", "DEVICE");
        paramMap.put("entityId", deviceId);


        RestTemplate restTemplate = new RestTemplate();
        //ResponseEntity response = restTemplate.exchange(baseURL+apiUrl, HttpMethod.GET, entity, String.class, new Object[]{"DEVICE",deviceId});
        ResponseEntity response = restTemplate.exchange(baseURL+apiUrl, HttpMethod.GET, entity, String.class, paramMap);
        System.out.println(response);
        if( response.getStatusCodeValue() == 200 ){
            String device = (String)response.getBody();
            System.out.println(response.getBody());

        }

    }
```




