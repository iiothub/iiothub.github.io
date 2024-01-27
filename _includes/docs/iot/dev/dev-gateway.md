* TOC
{:toc}



## 一、网关规划



### 1.网关规划

1. **Gateway API可以定制开发网关**
2. **能够实现：设备链接、遥测、属性、服务端RPC（单向、双向）**
3. **平台端设备可以认为网关是透明的，不用关注网关**





## 二、准备工作



### 1.RPC

#### 1.1.官网文档

```shell
# MQTT Gateway API Reference

https://thingsboard.io/docs/reference/gateway-mqtt-api/
```





#### 1.2.Server-side RPC

In order to subscribe to RPC commands from the server, send SUBSCRIBE message to the following topic:

```shell
v1/gateway/rpc
```

and expect messages with individual commands in the following format:

```json
{"device": "Device A", "data": {"id": $request_id, "method": "toggle_gpio", "params": {"pin":1}}}
```

Once command is processed by device, gateway can send commands back using following format:

```json
{"device": "Device A", "id": $request_id, "data": {"success": true}}
```

where **$request_id** is your integer request identifier, **Device A** is your device name and **method** is your RPC method name.



#### 1.3.RPC开发

1. 设备端（网关）订阅：v1/gateway/rpc

2. 设备端配置网关访问令牌

3. 应用端调用REST  API接口

4. REST  API数据格式

   ```json
   # 发送数据
   {
     "method": "getValue",
     "params": {
       "pin": 7,
       "value": 1
     },
     "persistent": false,
     "timeout": 5000
   }
   
   # 接收数据
   {
   	"device": "Device A",
   	"data": {
   		"id": 21,
   		"method": "setValue",
   		"params": {
   			"pin": 7,
   			"value": 1
   		}
   	}
   }
   ```
   
5. 双向RPC返回数据格式

   ```json
   # 发送数据
   {
     "method": "getValue",
     "params": {
       "pin": 7,
       "value": 1
     },
     "persistent": false,
     "timeout": 5000
   }
   
   # 接收数据
   {
   	"device": "Device A",
   	"data": {
   		"id": 21,
   		"method": "setValue",
   		"params": {
   			"pin": 7,
   			"value": 1
   		}
   	}
   }
   
   #返回值
   {
   	"device": "Device A",
   	"id": 21,
   	"data": {
   		"success": true
   	}
   }
   ```
   
   



## 三、服务端RPC



### 1.工程说明

```shell
server:
  port: 8999
spring:
  application:
    name: tb-gateway-api

mqtt:
  broker-url: tcp://82.157.166.86:1883
  client-id: emq-client-gateway-api
  username: mC81JBPikrtw82P2pn9Q    #网关访问令牌
  password:
```





### 2.设备端（MQTT）

```java
package com.iothub.gateway;
import com.iothub.mqtt.EmqClient;
import com.iothub.mqtt.MqttProperties;
import com.iothub.mqtt.QosEnum;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import javax.annotation.PostConstruct;

@Component
public class Rpc {

    @Autowired
    private EmqClient emqClient;

    @Autowired
    private MqttProperties properties;

    @PostConstruct
    public void init(){
        //连接服务端
        emqClient.connect(properties.getUsername(),properties.getPassword());
        //订阅一个主题
        emqClient.subscribe("v1/gateway/rpc", QosEnum.QoS1);
    }

    @Scheduled(fixedRate = 2000)
    public void publish(){
        String data = getData();
        //emqClient.publish("v1/gateway/attributes",data, QosEnum.QoS1,false);
    }

    private String getData(){

        String data = "";
        return data;
    }
}
```



```java
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

        // 双向RPC
/////////////////////////////////////////////////////////
//payload={"device":"Device A","data":{"id":6,"method":"getValue","params":{"pin":88,"value":99}}}
        String payload = new String(message.getPayload());
        ObjectMapper objectMapper = new ObjectMapper();
        JsonNode jsonNode = objectMapper.readTree(payload);
        JsonNode data = jsonNode.get("data");
        String id = data.get("id").toString();

        String retData = "{\"device\": \"Device A\", \"id\": $request_id, \"data\": {\"success\": true}}";
        String newData = retData.replace("$request_id", id);
        emqClient.publish("v1/gateway/rpc", newData, QosEnum.QoS1,false);
    }
```



### 3.单向RPC（REST  API）

```java
package com.iothub.rest;
import com.fasterxml.jackson.core.JsonProcessingException;
import org.springframework.http.*;
import org.springframework.web.client.RestTemplate;

public class OneRPC {

    public void sendRpc() throws JsonProcessingException {

        String baseURL = "http://82.157.166.86:8080";
        String token = Login.getToken();

        // 地址：/api/rpc/oneway/{deviceId}
        // 设备ID：  eb9cc990-6711-11ee-afb9-c790163a721a
        String apiUrl = "/api/rpc/oneway/{deviceId}";
        String deviceId = "9ae2b110-68ae-11ee-afb9-c790163a721a";
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



```json
{
  "method": "getValue",
  "params": {
    "pin": 7,
    "value": 1
  },
  "persistent": false,
  "timeout": 5000
}
```



### 4.双向RPC （REST   API）

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
        String deviceId = "9ae2b110-68ae-11ee-afb9-c790163a721a";
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



```json
{
  "method": "getValue",
  "params": {
    "pin": 88,
    "value": 99
  },
  "persistent": false,
  "timeout": 5000
}
```



### 5.测试

#### 5.1.单向RPC

```shell
http://localhost:8999/rpc/oneway

{
  "method": "getValue",
  "params": {
    "pin": 7,
    "value": 1
  },
  "persistent": false,
  "timeout": 5000
}
```

```shell
# 设备端输出

2023-10-13 09:58:34.184  INFO 5728 --- [ent-gateway-api] com.iothub.mqtt.MessageCallback          : 订阅者订阅到了消息,topic=v1/gateway/rpc,messageid=1,qos=1,payload={"device":"Device A","data":{"id":21,"method":"setValue","params":{"pin":7,"value":1}}}

{
	"device": "Device A",
	"data": {
		"id": 21,
		"method": "setValue",
		"params": {
			"pin": 7,
			"value": 1
		}
	}
}

# REST API端
<200,[Vary:"Origin", "Access-Control-Request-Method", "Access-Control-Request-Headers", X-Content-Type-Options:"nosniff", X-XSS-Protection:"1; mode=block", Cache-Control:"no-cache, no-store, max-age=0, must-revalidate", Pragma:"no-cache", Expires:"0", Content-Length:"0", Date:"Fri, 13 Oct 2023 01:58:33 GMT", Keep-Alive:"timeout=60", Connection:"keep-alive"]>
null
```



#### 5.2.双向RPC

```shell
http://localhost:8999/rpc/twoway

{
  "method": "getValue",
  "params": {
    "pin": 88,
    "value": 99
  },
  "persistent": false,
  "timeout": 5000
}
```

```shell
# 设备端输出

: 订阅者订阅到了消息,topic=v1/gateway/rpc,messageid=2,qos=1,payload={"device":"Device A","data":{"id":22,"method":"getValue","params":{"pin":88,"value":99}}}


{
	"device": "Device A",
	"data": {
		"id": 22,
		"method": "getValue",
		"params": {
			"pin": 88,
			"value": 99
		}
	}
}

#返回值
{
	"device": "Device A",
	"id": 22,
	"data": {
		"success": true
	}
}


# REST API端
<200,{"success":true},[Vary:"Origin", "Access-Control-Request-Method", "Access-Control-Request-Headers", X-Content-Type-Options:"nosniff", X-XSS-Protection:"1; mode=block", Cache-Control:"no-cache, no-store, max-age=0, must-revalidate", Pragma:"no-cache", Expires:"0", Content-Type:"application/json", Transfer-Encoding:"chunked", Date:"Fri, 13 Oct 2023 02:11:27 GMT", Keep-Alive:"timeout=60", Connection:"keep-alive"]>

{"success":true}
```



