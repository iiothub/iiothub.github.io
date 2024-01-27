* TOC
{:toc}



## 一、RPC规划



### 1.服务端RPC

1. **服务端RPC分：单向RPC、双向RPC**
2. **应用端通过REST API发送RPC请求**
3. **双向RPC通response返回命令结果**



**服务端RPC调用可以分为单向和双向：**

- 单向RPC请求直接发送请求，并且不对设备响应做任何处理

  ![](/images/iot/dev/dev-rpc/rpc-1.svg)

- 双向RPC请求会发送到设备，并且超时期间内接收到来自设备的响应

  ![](/images/iot/dev/dev-rpc/rpc-2.svg)



### 2.客户端RPC

1. **客户端RPC从设备端发送到平台**
2. **ThingsBoard没有提供REST API**
3. **ThingsBoard通过规则引擎整合元数据**
4. **ThingsBoard通过规则引擎把消息路由到消息队列（RabbitMQ或Kafka、EMQX），根据设备类型建topic**
5. **消息队列（RabbitMQ）根据topic消费数据，处理命令**



![](/images/iot/dev/dev-rpc/rpc-3.svg)





## 二、准备工作



### 1.创建设备

<img src="/images/iot/dev/dev-rpc/rpc-7.png" style="zoom:50%;" />



### 2.创建规则引擎

#### 2.1.创建RPC规则链

![](/images/iot/dev/dev-rpc/rpc-8.png)

![](/images/iot/dev/dev-rpc/rpc-9.png)

![](/images/iot/dev/dev-rpc/rpc-18.png)



### 3.RPC数据

#### 3.1.双向PRC数据

```json
{
    "method": "getValue",
    "params": "{\"pin\":88,\"value\":99}",
    "additionalInfo": null
}
```



```json
{
    "deviceName": "Test-rpc",
    "deviceType": "default",
    "expirationTime": "1696921879114",
    "oneway": "false",
    "originServiceId": "VM-24-10-centos",
    "persistent": "false",
    "requestUUID": "2e023965-6e0b-4b28-8523-26380de2aa98"
}
```



#### 3.2.单向RPC数据

```json
{
    "method": "setValue",
    "params": "{\"pin\":7,\"value\":1}",
    "additionalInfo": null
}
```



```json
{
    "deviceName": "Test-rpc",
    "deviceType": "default",
    "expirationTime": "1696921942198",
    "oneway": "true",
    "originServiceId": "VM-24-10-centos",
    "persistent": "false",
    "requestUUID": "a238826d-0f8d-49dc-8240-d3382c14f9a2"
}
```



####  3.3.客户端RPC数据

```json
{
    "method": "getServerValue",
    "params": ""
}
```



```json
{
    "deviceName": "Test-rpc",
    "deviceType": "default",
    "requestId": "1",
    "serviceId": "VM-24-10-centos",
    "sessionId": "e8c36497-9601-4df5-b8d6-3755bf6c500c"
}
```





## 三、REST API



### 1.RPC API

![](/images/iot/dev/dev-rpc/rpc-4.png)



### 2.单向RPC

![](/images/iot/dev/dev-rpc/rpc-5.png)

```shell
# POST路径
/api/rpc/oneway/{deviceId}

# 设备ID  Test-rpc
eb9cc990-6711-11ee-afb9-c790163a721a

# 发送数据
{
  "method": "setValue",
  "params": {
    "pin": 7,
    "value": 1
  },
  "persistent": false,
  "timeout": 5000
}

http://localhost:8999/rpc/oneway
```



```java
package com.iothub.rest;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.http.*;
import org.springframework.web.client.RestTemplate;

public class OneRPC {

    public void sendRpc() throws JsonProcessingException {
        String baseURL = "http://82.157.166.86:8080";
        String token = Login.getToken();
        // 地址：/api/rpc/oneway/{deviceId}
        // 设备ID：  eb9cc990-6711-11ee-afb9-c790163a721a
        String apiUrl = "/api/rpc/oneway/{deviceId}";
        String deviceId = "eb9cc990-6711-11ee-afb9-c790163a721a";
        HttpHeaders headers = new HttpHeaders();
        //headers.add("Authorization", "Bearer "+ mainToken);
        headers.set("X-Authorization", "Bearer " + token);
        //headers.set("Content-Type", "application/json");
        headers.setContentType(MediaType.APPLICATION_JSON);

        String body = "{\n" +
                "  \"method\": \"setValue\",\n" +
                "  \"params\": {\n" +
                "    \"pin\": 7,\n" +
                "    \"value\": 1\n" +
                "  },\n" +
                "  \"persistent\": false,\n" +
                "  \"timeout\": 5000\n" +
                "}";

        HttpEntity entity = new HttpEntity<>(body, headers);
        RestTemplate restTemplate = new RestTemplate();
        ResponseEntity response = restTemplate.exchange(baseURL + apiUrl, HttpMethod.POST, entity, String.class, new Object[]{deviceId});
        System.out.println(response);
        if (response.getStatusCodeValue() == 200) {
            String device = (String) response.getBody();
            System.out.println(response.getBody());
        }
    }
}
```



### 3.双向RPC

![](/images/iot/dev/dev-rpc/rpc-6.png)



```shell
# POST路径
/api/rpc/twoway/{deviceId}

# 设备ID  Test-rpc
eb9cc990-6711-11ee-afb9-c790163a721a

# 发送数据
{
  "method": "getValue",
  "params": {
    "pin": 88,
    "value": 99
  },
  "persistent": false,
  "timeout": 5000
}

http://localhost:8999/rpc/twoway
```

```shell
# 响应
v1/devices/me/rpc/request/$request_id
v1/devices/me/rpc/response/$request_id

{
   "pin": 4,
   "value": 1,
   "changed": true
}

订阅者订阅到了消息,topic=v1/devices/me/rpc/request/10,messageid=1,qos=1,payload={"method":"getValue","params":{"pin":88,"value":99}}
```



```java
package com.iothub.rest;
import com.fasterxml.jackson.core.JsonProcessingException;
import org.springframework.http.*;
import org.springframework.web.client.RestTemplate;

public class TwoRPC {

    public void sendRpc() throws JsonProcessingException {

        String baseURL = "http://82.157.166.86:8080";
        String token = Login.getToken();

        // 地址：/api/rpc/oneway/{deviceId}
        // 设备ID：  eb9cc990-6711-11ee-afb9-c790163a721a
        String apiUrl = "/api/rpc/twoway/{deviceId}";
        String deviceId = "eb9cc990-6711-11ee-afb9-c790163a721a";
        HttpHeaders headers = new HttpHeaders();
        //headers.add("Authorization", "Bearer "+ mainToken);
        headers.set("X-Authorization", "Bearer " + token);
        //headers.set("Content-Type", "application/json");
        headers.setContentType(MediaType.APPLICATION_JSON);

        String body = "{\n" +
                "  \"method\": \"getValue\",\n" +
                "  \"params\": {\n" +
                "    \"pin\": 88,\n" +
                "    \"value\": 99\n" +
                "  },\n" +
                "  \"persistent\": false,\n" +
                "  \"timeout\": 5000\n" +
                "}";

        HttpEntity entity = new HttpEntity<>(body, headers);
        RestTemplate restTemplate = new RestTemplate();
        ResponseEntity response = restTemplate.exchange(baseURL + apiUrl, HttpMethod.POST, entity, String.class, new Object[]{deviceId});
        System.out.println(response);
        if (response.getStatusCodeValue() == 200) {
            String device = (String) response.getBody();
            System.out.println(response.getBody());
        }
    }
}
```





## 四、服务端RPC



### 1.双向PRC

#### 1.1.REST API

```java
package com.iothub.rest;
import com.fasterxml.jackson.core.JsonProcessingException;
import org.springframework.http.*;
import org.springframework.web.client.RestTemplate;

public class TwoRPC {

    public void sendRpc() throws JsonProcessingException {

        String baseURL = "http://82.157.166.86:8080";
        String token = Login.getToken();

        // 地址：/api/rpc/oneway/{deviceId}
        // 设备ID：  eb9cc990-6711-11ee-afb9-c790163a721a
        String apiUrl = "/api/rpc/twoway/{deviceId}";
        String deviceId = "eb9cc990-6711-11ee-afb9-c790163a721a";
        HttpHeaders headers = new HttpHeaders();
        //headers.add("Authorization", "Bearer "+ mainToken);
        headers.set("X-Authorization", "Bearer " + token);
        //headers.set("Content-Type", "application/json");
        headers.setContentType(MediaType.APPLICATION_JSON);

        String body = "{\n" +
                "  \"method\": \"getValue\",\n" +
                "  \"params\": {\n" +
                "    \"pin\": 88,\n" +
                "    \"value\": 99\n" +
                "  },\n" +
                "  \"persistent\": false,\n" +
                "  \"timeout\": 5000\n" +
                "}";

        HttpEntity entity = new HttpEntity<>(body, headers);
        RestTemplate restTemplate = new RestTemplate();
        ResponseEntity response = restTemplate.exchange(baseURL + apiUrl, HttpMethod.POST, entity, String.class, new Object[]{deviceId});
        System.out.println(response);
        if (response.getStatusCodeValue() == 200) {
            String device = (String) response.getBody();
            System.out.println(response.getBody());
        }
    }
}
```



#### 1.2.设备（MQTT）

```java
package com.iothub.rpc;
import com.iothub.mqtt.EmqClient;
import com.iothub.mqtt.MqttProperties;
import com.iothub.mqtt.QosEnum;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import javax.annotation.PostConstruct;

@Component
public class ServerRpc {

    @Autowired
    private EmqClient emqClient;

    @Autowired
    private MqttProperties properties;

    @PostConstruct
    public void init(){
        //连接服务端
        emqClient.connect(properties.getUsername(),properties.getPassword());
        //订阅一个主题
        emqClient.subscribe("v1/devices/me/rpc/request/+", QosEnum.QoS1);
    }
}
```



```shell
# MessageCallback
 
     /**
     * 应用收到消息后触发的回调
     * @param topic
     * @param message
     * @throws Exception
     */
    @Override
    public void messageArrived(String topic, MqttMessage message) throws Exception {
        log.info("订阅者订阅到了消息,topic={},messageid={},qos={},payload={}",
                topic,
                message.getId(),
                message.getQos(),
                new String(message.getPayload()));

/////////////////////////////////////////////////////////
        // 双向RPC
        // v1/devices/me/rpc/request/$request_id
        String[] buff = topic.split("/");
        String request_id = buff[buff.length-1];
        // 客户端PUBLISH下面主题进行响应：
        // v1/devices/me/rpc/response/$request_id
        String retData = "{\n" +
                "   \"pin\": 4,\n" +
                "   \"value\": 1,\n" +
                "   \"changed\": true\n" +
                "}";

        emqClient.publish("v1/devices/me/rpc/response/" + request_id, retData, QosEnum.QoS1,false);
    }
```



#### 1.3.测试

```shell
# POST路径
/api/rpc/twoway/{deviceId}

# 设备ID  Test-rpc
eb9cc990-6711-11ee-afb9-c790163a721a

# 发送数据
{
  "method": "getValue",
  "params": {
    "pin": 88,
    "value": 99
  },
  "persistent": false,
  "timeout": 5000
}

# 返回结果
{
	"pin": 4,
	"value": 1,
	"changed": true
}

http://localhost:8999/rpc/twoway
```



```shell
2023-10-10 15:28:30.926  INFO 23200 --- [ emq-client-rpc] com.iothub.mqtt.MessageCallback          : 订阅者订阅到了消息,topic=v1/devices/me/rpc/request/32,messageid=5,qos=1,payload={"method":"getValue","params":{"pin":88,"value":99}}
2023-10-10 15:28:30.933  INFO 23200 --- [ emq-client-rpc] com.iothub.mqtt.MessageCallback          : 消息发布完成,messageid=6,topics=[v1/devices/me/rpc/response/32]
<200,{"pin":4,"value":1,"changed":true},[Vary:"Origin", "Access-Control-Request-Method", "Access-Control-Request-Headers", X-Content-Type-Options:"nosniff", X-XSS-Protection:"1; mode=block", Cache-Control:"no-cache, no-store, max-age=0, must-revalidate", Pragma:"no-cache", Expires:"0", Content-Type:"application/json", Transfer-Encoding:"chunked", Date:"Tue, 10 Oct 2023 07:28:30 GMT", Keep-Alive:"timeout=60", Connection:"keep-alive"]>
{"pin":4,"value":1,"changed":true}
```



### 2.单向RPC

#### 2.1.RESTAPI

```java
package com.iothub.rest;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.http.*;
import org.springframework.web.client.RestTemplate;

public class OneRPC {

    public void sendRpc() throws JsonProcessingException {

        String baseURL = "http://82.157.166.86:8080";
        String token = Login.getToken();

        // 地址：/api/rpc/oneway/{deviceId}
        // 设备ID：  eb9cc990-6711-11ee-afb9-c790163a721a
        String apiUrl = "/api/rpc/oneway/{deviceId}";
        String deviceId = "eb9cc990-6711-11ee-afb9-c790163a721a";
        HttpHeaders headers = new HttpHeaders();
        //headers.add("Authorization", "Bearer "+ mainToken);
        headers.set("X-Authorization", "Bearer " + token);
        //headers.set("Content-Type", "application/json");
        headers.setContentType(MediaType.APPLICATION_JSON);

        String body = "{\n" +
                "  \"method\": \"setValue\",\n" +
                "  \"params\": {\n" +
                "    \"pin\": 7,\n" +
                "    \"value\": 1\n" +
                "  },\n" +
                "  \"persistent\": false,\n" +
                "  \"timeout\": 5000\n" +
                "}";

        HttpEntity entity = new HttpEntity<>(body, headers);
        RestTemplate restTemplate = new RestTemplate();
        ResponseEntity response = restTemplate.exchange(baseURL + apiUrl, HttpMethod.POST, entity, String.class, new Object[]{deviceId});
        System.out.println(response);
        if (response.getStatusCodeValue() == 200) {
            String device = (String) response.getBody();
            System.out.println(response.getBody());
        }
    }
}
```



```java
package com.iothub.rest;
import com.fasterxml.jackson.databind.JsonNode;
import org.springframework.http.ResponseEntity;
import org.springframework.web.client.RestTemplate;
import java.util.HashMap;
import java.util.Map;

public class Login {

    public static String getToken(){

        String username = "tenant@thingsboard.org";
        String password = "tenant";
        String baseURL = "http://82.157.166.86:8080";
        RestTemplate restTemplate = new RestTemplate();

        // 登录
        long ts = System.currentTimeMillis();
        Map<String, String> loginRequest = new HashMap();
        loginRequest.put("username", username);
        loginRequest.put("password", password);
        ResponseEntity<JsonNode> tokenInfo1 = restTemplate.postForEntity(baseURL + "/api/auth/login", loginRequest, JsonNode.class, new Object[0]);
        JsonNode tokenInfo = (JsonNode)tokenInfo1.getBody();
        //System.out.println(tokenInfo);

        // 解析数据
        String mainToken = tokenInfo.get("token").asText();
        String refreshToken = tokenInfo.get("refreshToken").asText();
        //System.out.println("token: " + tokenInfo);
        //System.out.println("refreshToken: " + refreshToken);

        return mainToken;
    }
}
```



#### 2.2.设备（MQTT）

同  1.2.设备（MQTT）



#### 2.3.测试

```shell
# POST路径
/api/rpc/oneway/{deviceId}

# 设备ID  Test-rpc
eb9cc990-6711-11ee-afb9-c790163a721a

# 发送数据
{
  "method": "setValue",
  "params": {
    "pin": 7,
    "value": 1
  },
  "persistent": false,
  "timeout": 5000
}

http://localhost:8999/rpc/oneway
```



```shell
<200,[Vary:"Origin", "Access-Control-Request-Method", "Access-Control-Request-Headers", X-Content-Type-Options:"nosniff", X-XSS-Protection:"1; mode=block", Cache-Control:"no-cache, no-store, max-age=0, must-revalidate", Pragma:"no-cache", Expires:"0", Content-Length:"0", Date:"Tue, 10 Oct 2023 07:33:01 GMT", Keep-Alive:"timeout=60", Connection:"keep-alive"]>
null
2023-10-10 15:33:01.593  INFO 23200 --- [ emq-client-rpc] com.iothub.mqtt.MessageCallback          : 订阅者订阅到了消息,topic=v1/devices/me/rpc/request/33,messageid=6,qos=1,payload={"method":"setValue","params":{"pin":7,"value":1}}
2023-10-10 15:33:01.600  INFO 23200 --- [ emq-client-rpc] com.iothub.mqtt.MessageCallback          : 消息发布完成,messageid=7,topics=[v1/devices/me/rpc/response/33]
```





## 五、客户端RPC



### 1.规则引擎

#### 1.1.整合元数据

![](/images/iot/dev/dev-rpc/rpc-10.png)

```json
{
    "deviceName": "Test-rpc",
    "deviceType": "default",
    "requestId": "1",
    "serviceId": "VM-24-10-centos",
    "sessionId": "e8c36497-9601-4df5-b8d6-3755bf6c500c"
}
```



#### 1.2.按设备类型流转命令

![](/images/iot/dev/dev-rpc/rpc-11.png)

```javascript
# 按设备类型流转命令

if(msg.deviceType === 'test-telemetry') {
    return ['test-telemetry'];
} else {
    return ['default'];
}
```



#### 1.3.RabbitMQ

![](/images/iot/dev/dev-rpc/rpc-12.png)

![](/images/iot/dev/dev-rpc/rpc-13.png)



#### 1.4.完整规则链

![](/images/iot/dev/dev-rpc/rpc-17.png)



### 2.RabbitMQ

**队列：telemetry-queue、telemetry-default-queue**

**交换机：telemetry-devicetype-exchange**

![](/images/iot/dev/dev-rpc/rpc-14.png)



### 3.设备（MQTT）

```java
package com.iothub.rpc;
import com.iothub.mqtt.EmqClient;
import com.iothub.mqtt.MqttProperties;
import com.iothub.mqtt.QosEnum;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import javax.annotation.PostConstruct;

@Component
public class ClientRpc {

    @Autowired
    private EmqClient emqClient;

    @Autowired
    private MqttProperties properties;

    @PostConstruct
    public void init(){
        //连接服务端
        emqClient.connect(properties.getUsername(),properties.getPassword());
        //订阅一个主题
        emqClient.subscribe("v1/devices/me/rpc/response/+", QosEnum.QoS1);
    }

    @Scheduled(fixedRate = 3000)
    public void publish(){
        String data = getData();
        emqClient.publish("v1/devices/me/rpc/request/1",data,QosEnum.QoS1,false);
    }

    private String getData(){

        String data = "{\n" +
                "\t\"method\": \"getServerValue\",\n" +
                "\t\"params\": \"\"\n" +
                "}";

        return data;
    }
}
```



### 4.测试

```json
{
	"method": "getServerValue",
	"params": "",
	"deviceType": "default",
	"deviceName": "Test-rpc"
}
```



![](/images/iot/dev/dev-rpc/rpc-16.png)

![](/images/iot/dev/dev-rpc/rpc-15.png)



