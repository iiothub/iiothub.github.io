* TOC
{:toc}



## 一、概述



### 1.安装说明



![](/images/edgex/device/link-mqtt/mqtt-1.png)

```shell
# 官方文档

https://docs.edgexfoundry.org/3.1/microservices/device/services/device-mqtt/Ch-ExamplesAddingMQTTDevice/
```



**安装方式：**

- 使用 EdgeX Console 界面创建 MQTT 设备
- 使用 Spring Boot 实现 MQTT 设备模拟器



### 2.MQTT 设备模拟器

#### 2.1.模拟器设计

MQTT 设备模拟器使用 Spring Boot 开发。参考 mock-device.js。



mock-device.js

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



#### 2.2.Spring Boot 程序源码

##### 2.2.1.MQTT

```java
package com.iothub.mqtt;
import org.eclipse.paho.client.mqttv3.*;
import org.eclipse.paho.client.mqttv3.persist.MemoryPersistence;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

/**
 * Created by 传智播客*黑马程序员.
 */
@Component
public class EmqClient {
    
    private static final Logger log = LoggerFactory.getLogger(EmqClient.class);
    
    
    private IMqttClient mqttClient;
    
    @Autowired
    private MqttProperties mqttProperties;
    
    @Autowired
    private MqttCallback mqttCallback;
    
    
    @PostConstruct
    public void init(){
        MqttClientPersistence mempersitence = new MemoryPersistence();
        try {
            mqttClient = new MqttClient(mqttProperties.getBrokerUrl(),mqttProperties.getClientId(),mempersitence);
        } catch (MqttException e) {
            log.error("初始化客户端mqttClient对象失败,errormsg={},brokerUrl={},clientId={}",e.getMessage(),mqttProperties.getBrokerUrl(),mqttProperties.getClientId());
        }

    }

    /**
     * 连接broker
     * @param username
     * @param password
     */
    public void connect(String username,String password){
        MqttConnectOptions options = new MqttConnectOptions();
        options.setAutomaticReconnect(true);
        options.setUserName(username);
        options.setPassword(password.toCharArray());
        options.setCleanSession(true);
        
        mqttClient.setCallback(mqttCallback);

        try {
            mqttClient.connect(options);
        } catch (MqttException e) {
            log.error("mqtt客户端连接服务端失败,失败原因{}",e.getMessage());
        }
    }

    /**
     * 断开连接
     */
    @PreDestroy
    public void disConnect(){
        try {
            mqttClient.disconnect();
        } catch (MqttException e) {
            log.error("断开连接产生异常,异常信息{}",e.getMessage());
        }
    }

    /**
     * 重连
     */
    public void reConnect(){
        try {
            mqttClient.reconnect();
        } catch (MqttException e) {
            log.error("重连失败,失败原因{}",e.getMessage());
        }
    }

    /**
     * 发布消息
     * @param topic
     * @param msg
     * @param qos
     * @param retain
     */
    public void publish(String topic, String msg, QosEnum qos, boolean retain){

        MqttMessage mqttMessage = new MqttMessage();
        mqttMessage.setPayload(msg.getBytes());
        mqttMessage.setQos(qos.value());
        mqttMessage.setRetained(retain);
        try {
            mqttClient.publish(topic,mqttMessage);
        } catch (MqttException e) {
            log.error("发布消息失败,errormsg={},topic={},msg={},qos={},retain={}",e.getMessage(),topic,msg,qos.value(),retain);
        }

    }

    /**
     * 订阅
     * @param topicFilter
     * @param qos
     */
    public void subscribe(String topicFilter,QosEnum qos){
        try {
            mqttClient.subscribe(topicFilter,qos.value());
        } catch (MqttException e) {
            log.error("订阅主题失败,errormsg={},topicFilter={},qos={}",e.getMessage(),topicFilter,qos.value());
        }

    }

    /**
     * 取消订阅
     * @param topicFilter
     */
    public void unSubscribe(String topicFilter){
        try {
            mqttClient.unsubscribe(topicFilter);
        } catch (MqttException e) {
            log.error("取消订阅失败,errormsg={},topicfiler={}",e.getMessage(),topicFilter);
        }
    }
    
}
```



```java
package com.iothub.mqtt;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.iothub.device.MqttData;
import com.iothub.utils.JsonUtils;
import org.eclipse.paho.client.mqttv3.IMqttDeliveryToken;
import org.eclipse.paho.client.mqttv3.MqttCallback;
import org.eclipse.paho.client.mqttv3.MqttMessage;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

/**
 * Created by 传智播客*黑马程序员.
 */
@Component
public class MessageCallback implements MqttCallback {
    
    private static final Logger log = LoggerFactory.getLogger(MessageCallback.class);


    @Autowired
    private EmqClient emqClient;


    @Autowired
    private MqttData mqttData;



    /**
     * 丢失了对服务端的连接后触发的回调
     * @param cause
     */
    @Override
    public void connectionLost(Throwable cause) {
        // 资源的清理  重连
        log.info("丢失了对服务端的连接");
    }

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

        String[] buff = topic.split("/");
        String cmd = buff[2];
        String method = buff[3];
        String uuid = buff[4];
        String response = "{}";
        String data = new String(message.getPayload());


        if (method.equals("set")) {
            log.info("修改 message ={}", data);
            switch (cmd) {
                case "message":
                    String msg= JsonUtils.jsonToNodeString( data, "message");
                    mqttData.setMessage(msg);
                    break;
                case "json":
                    String json= JsonUtils.jsonToNodeString( data, "json");
                    mqttData.setJson(json);
                    break;
            }
        } else {
            switch (cmd) {
                case "ping":
                    response = mqttData.getPing();
                    break;
                case "message":
                    response = mqttData.getMessage();
                    break;
                case "randnum":
                    response = mqttData.getRandnum();
                    break;
                case "json":
                    response = mqttData.getJson();
                    break;
            }


        }

        emqClient.publish("command/response/" + uuid, response, QosEnum.QoS1,false);
    }

    /**
     * 消息发布者消息发布完成产生的回调
     * @param token
     */
    @Override
    public void deliveryComplete(IMqttDeliveryToken token) {
        int messageId = token.getMessageId();
        String[] topics = token.getTopics();
        log.info("消息发布完成,messageid={},topics={}",messageId,topics);
    }
}
```



```java
package com.iothub.mqtt;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

/**
 * Created by 传智播客*黑马程序员.
 */
@Configuration
@ConfigurationProperties(prefix = "mqtt")
public class MqttProperties {
    
    private String brokerUrl;
    
    private String clientId;
    
    private String username;
    
    private String password;


    public String getBrokerUrl() {
        return brokerUrl;
    }

    public void setBrokerUrl(String brokerUrl) {
        this.brokerUrl = brokerUrl;
    }

    public String getClientId() {
        return clientId;
    }

    public void setClientId(String clientId) {
        this.clientId = clientId;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public String toString() {
        return "MqttProperties{" +
                "brokerUrl='" + brokerUrl + '\'' +
                ", clientId='" + clientId + '\'' +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}
```



```java
package com.iothub.mqtt;

/**
 * Created by 传智播客*黑马程序员.
 */
public enum QosEnum {
    QoS0(0),QoS1(1),QoS2(2);


    private final int value;

    QosEnum(int value) {
        this.value = value;
    }
    
    public int value(){
        return this.value;
    }
}
```



##### 2.2.2.JsonUtils

```java
package com.iothub.utils;

import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.JavaType;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.node.ArrayNode;
import org.apache.commons.lang3.StringUtils;

import java.nio.charset.StandardCharsets;
import java.text.SimpleDateFormat;
import java.util.List;
import java.util.Map;

/**
 * JSON转换工具类。
 *
 * @date 2017-06-29 10:07:51
 */
public final class JsonUtils {
	/**
	 * 日期字符串的格式
	 */
	public static final String DATE_FORMAT = "yyyy-MM-dd HH:mm:ss";

	private static ObjectMapper objectMapper = new ObjectMapper();

	static {
		// 设置日期字符串的格式
		objectMapper.setDateFormat(new SimpleDateFormat(DATE_FORMAT));

		// 反序列化时，允许字段名没有双引号
		objectMapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true);

		// 忽略未知属性
		objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);

		// 反序列化时，忽略未知的属性
		objectMapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
	}

	private JsonUtils() {
	}

	/**
	 * 获取ObjectMapper
	 *
	 * @return
	 */
	public static ObjectMapper getObjectMapper() {
		return objectMapper;
	}

	/**
	 * 对象转JSON字符串
	 *
	 * @param obj 对象
	 * @return JSON字符串
	 */
	public static String toJSONString(Object obj) {
		try {
			return objectMapper.writeValueAsString(obj);
		}
		catch (JsonProcessingException e) {
			throw new RuntimeException("JSON序列化时出现错误！", e);
		}
	}

	/**
	 * 对象转JSON字符串。
	 *
	 * @param obj        对象。
	 * @param dateFormat 日期格式。
	 * @return JSON字符串。
	 */
	public static String toJSONString(Object obj, String dateFormat) {
		if (StringUtils.isBlank(dateFormat)) {
			dateFormat = DATE_FORMAT;
		}

		objectMapper.setDateFormat(new SimpleDateFormat(dateFormat));
		try {
			return objectMapper.writeValueAsString(obj);
		}
		catch (JsonProcessingException e) {
			throw new RuntimeException("JSON序列化时出现错误！", e);
		}
	}

	/**
	 * 根据节点解析json中对象
	 * @param json json
	 * @param key 对象key
	 * @param clazz clazz
	 * @return json中对象
	 */
	public static <T> T jsonToNodeObject(String json,String key,Class<T> clazz){
		try {
			JsonNode jsonNode = objectMapper.readTree(json);
			return jsonNode.get(key).traverse(objectMapper).readValueAs(clazz);
		} catch (Exception e) {
			throw new RuntimeException("json解析节点错误");
		}
	}

	/**
	 * 根据节点解析json中嵌套对象
	 * @param json json
	 * @param key 对象key
	 * @param clazz clazz
	 * @param nodeNames 节点名
	 * @return json中对象
	 */
	public static <T> T jsonToNodeObject(String json,String key,Class<T> clazz,String... nodeNames){
		try {
			JsonNode jsonNode = objectMapper.readTree(json);
			for (String nodeName : nodeNames) {
				jsonNode = jsonNode.path(nodeName);
			}
			return jsonNode.get(key).traverse(objectMapper).readValueAs(clazz);
		} catch (Exception e) {
			throw new RuntimeException("json解析节点错误",e);
		}
	}


	public static String jsonToNodeString(String json,String key){
		try {
			JsonNode jsonNode = objectMapper.readTree(json);
			return jsonNode.get(key).asText();
		} catch (Exception e) {
			throw new RuntimeException("json解析节点错误",e);
		}
	}

	public static String jsonToNodeString(String json,String key,String nodeName){
		try {
			JsonNode jsonNode = objectMapper.readTree(json);
			jsonNode = jsonNode.path(nodeName);
			return jsonNode.get(key).toString();
		} catch (Exception e) {
			throw new RuntimeException("json解析节点错误",e);
		}
	}


	public static String jsonToNodeString(String json,String key,String... nodeName){
		try {
			JsonNode jsonNode = objectMapper.readTree(json);
			for (String name : nodeName) {
				jsonNode = jsonNode.path(name);
			}
			return jsonNode.get(key).asText();
		} catch (Exception e) {
			throw new RuntimeException("json解析节点错误",e);
		}
	}

	public static JsonNode jsonToNode(String json,String key,String... nodeName){
		try {
			JsonNode jsonNode = objectMapper.readTree(json);
			for (String name : nodeName) {
				jsonNode = jsonNode.path(name);
			}
			return jsonNode.get(key);
		} catch (Exception e) {
			throw new RuntimeException("json解析节点错误",e);
		}
	}

	public static ArrayNode jsonToArrayString(String json, String key, String... nodeName){
		try {
			JsonNode jsonNode = objectMapper.readTree(json);
			for (String name : nodeName) {
				jsonNode = jsonNode.path(name);
			}
			return (ArrayNode) jsonNode.get(key);
		} catch (Exception e) {
			throw new RuntimeException("json解析节点错误",e);
		}
	}



	/**
	 * 根据节点
	 * @param json
	 * @param key
	 * @param clazz
	 * @param <T>
	 * @return
	 */
	public static <T> T jsonToValue(String json,String key,Class<T> clazz){
		try {
			Map map = JsonUtils.jsonToMap(json);
			return (T)map.get(key);
		} catch (Exception e) {
			throw new RuntimeException("json解析节点错误",e);
		}
	}

	/**
	 * JSON字符串转对象。
	 *
	 * @param json  JSON字符串。
	 * @param clazz 对象类型。
	 * @param <T>   类型参数。
	 * @return 指定类型的对象。
	 */
	public static <T> T jsonToObject(String json, Class<T> clazz) {
		if (StringUtils.isBlank(json)) {
			throw new IllegalArgumentException("参数json不能为空！");
		}
		if (clazz == null) {
			throw new IllegalArgumentException("参数clazz不能为空！");
		}

		try {
			return objectMapper.readValue(json.getBytes(StandardCharsets.UTF_8), clazz);
		}
		catch (Exception e) {
			throw new RuntimeException("JSON字符串转对象时出现错误！", e);
		}
	}

	/**
	 * MAP转对象。
	 *
	 * @param map  MAP。
	 * @param clazz 对象类型。
	 * @param <T>   类型参数。
	 * @return 指定类型的对象。
	 */
	public static <T> T mapToObject(Map map, Class<T> clazz) {
		if (map == null) {
			throw new IllegalArgumentException("参数map不能为空！");
		}
		if (clazz == null) {
			throw new IllegalArgumentException("参数clazz不能为空！");
		}

		try {
			return objectMapper.convertValue(map, clazz);
		}
		catch (Exception e) {
			throw new RuntimeException("MAP字符串转对象时出现错误！", e);
		}
	}

	/**
	 * JSON字符串转对象数组。
	 *
	 * @param json  JSON字符串。
	 * @param clazz 数组元素的对象类型。
	 * @param <T>   类型参数。
	 * @return 指定类型的对象数组。
	 */
	public static <T> List<T> jsonToList(String json, Class<T> clazz) {
		if (StringUtils.isBlank(json)) {
			throw new IllegalArgumentException("参数json不能为空！");
		}
		if (clazz == null) {
			throw new IllegalArgumentException("参数clazz不能为空！");
		}

		try {
			JavaType javaType = objectMapper.getTypeFactory().constructParametricType(List.class, clazz);
			return objectMapper.readValue(json.getBytes(StandardCharsets.UTF_8), javaType);
		}
		catch (Exception e) {
			throw new RuntimeException("JSON字符串转数组时出现错误！", e);
		}
	}

	/**
	 * JSON字符串转MAP。
	 *
	 * @param json JSON字符串。
	 * @return MAP对象。
	 */
	public static Map jsonToMap(String json) {
		if (StringUtils.isBlank(json)) {
			throw new IllegalArgumentException("参数json不能为空！");
		}

		try {
			return objectMapper.readValue(json.getBytes(StandardCharsets.UTF_8), Map.class);
		}
		catch (Exception e) {
			throw new RuntimeException("JSON字符串转对象时出现错误！", e);
		}
	}


	/**
	 * 拷贝对象属性，并返回指定类型的新对象。
	 *
	 * @param src   源对象。
	 * @param clazz 目标对象类型。
	 * @param <T>   目标对象类型参数。
	 * @return 与原对象属性值相同的新对象。
	 */
	public static <T> T copyProperties(Object src, Class<T> clazz) {
		return jsonToObject(toJSONString(src), clazz);
	}

	/**
	 * 拷贝对象属性，并返回同类型的新对象。
	 *
	 * @param src 源对象。
	 * @param <T> 目标对象类型参数。
	 * @return 与原对象属性值相同的新对象。
	 */
	public static <T> T copyProperties(T src) {

		return (T) copyProperties(src, src.getClass());
	}
	/**
	 * @Description map转JSON
	 * @Date 16:32 2020/7/6
	 * @param map map对象
	 * @return JSON字符串
	 **/
	public static String mapToJson(Map map) {
		if (map == null) {
			throw new IllegalArgumentException("参数map不能为空！");
		}
		try {
			return objectMapper.writeValueAsString(map);
		} catch (JsonProcessingException e) {
			throw new RuntimeException("Map转JSON时出现错误！", e);
		}
	}
}
```



##### 2.2.3.Device

```java
package com.iothub.device;

import com.iothub.mqtt.EmqClient;
import com.iothub.mqtt.MqttProperties;
import com.iothub.mqtt.QosEnum;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import javax.annotation.PostConstruct;

@Component
public class MqttDevice {

    @Autowired
    private EmqClient emqClient;

    @Autowired
    private MqttData mqttData;

    @Autowired
    private MqttProperties properties;

    @PostConstruct
    public void init(){
        //连接服务端
        emqClient.connect(properties.getUsername(),properties.getPassword());
        //订阅一个主题
        emqClient.subscribe("command/my-custom-device/#", QosEnum.QoS1);
    }


    @Scheduled(fixedRate = 50000)
    public void publish(){

        String data = getData(1);

        emqClient.publish("incoming/data/my-custom-device/values",data,
                QosEnum.QoS1,false);
    }


    private String getData(Integer type){

        if (type == 1) {
            // 携带时间戳
            String data = mqttData.getValues();
            return data;

        } else if (type == 2) {
            // 不携带时间戳
            String data = "";
            return data;
        }else {
            // 数组
            String data = "[{\"key1\":\"value1\"}, {\"key2\":true}]";
            return data;
        }
    }
}

```



```shell
package com.iothub.device;
import com.iothub.utils.JsonUtils;
import org.springframework.stereotype.Component;
import java.util.HashMap;
import java.util.Map;

@Component
public class MqttData {

    public MqttData(){
        ping = "pong ......";
        randnum = 1000;
        message = "Hello World !!!";
        //json = "{\"name\" : \"My JSON\"}";
        Map<Object,Object> map = new HashMap<>();
        map.put("name", "My JSON ......");
        json = JsonUtils.toJSONString(map);
    }

    public void setRandnum(float randnum) {
        this.randnum = randnum;
    }

    public void setPing(String ping) {
        this.ping = ping;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public void setJson(String json) {
        this.json = json;
    }

    public String getRandnum() {
        Map<Object,Object> map = new HashMap<>();
        map.put("randnum", randnum );
        return JsonUtils.toJSONString(map);
    }

    public String getPing() {
        Map<Object,Object> map = new HashMap<>();
        map.put("ping", ping);
        return JsonUtils.toJSONString(map);
    }

    public String getMessage() {
        Map<Object,Object> map = new HashMap<>();
        map.put("message", message);
        return JsonUtils.toJSONString(map);
    }

    public String getJson() {
        Map<Object,Object> map = new HashMap<>();
        map.put("json", json);
        return JsonUtils.toJSONString(map);
    }

    public String getValues() {
        Map<Object,Object> map = new HashMap<>();
        map.put("randnum", randnum );
        map.put("ping", ping);
        map.put("message", message);
        return JsonUtils.toJSONString(map);
    }

    private float randnum;

    private String ping;

    private String message;

    private String json;
}
```



#### 2.3.程序配置

- pom.xml

```xml
        <dependency>
            <groupId>org.eclipse.paho</groupId>
            <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
            <version>1.2.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.4</version>
        </dependency>
```



- application.yaml


```yaml
server:
  port: 8888
spring:
  application:
    name: device-mqtt-simulator

mqtt:
  broker-url: tcp://192.168.202.233:1883
  client-id: device-mqtt-simulator
  username:
  password:
```





## 二、连接 MQTT 设备



### 1.docker-comepse

```shell
# 1.克隆 edgex-compose
$ git clone git@github.com:edgexfoundry/edgex-compose.git 
$ git clone https://github.com/edgexfoundry/edgex-compose.git
$ cd edgex-compose 
$ git checkout v3.1


# 2.生成 docker-compose.yml 文件（注意这包括 mqtt-broker）
$ cd compose-builder
$ make gen ds-mqtt mqtt-broker no-secty


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

[root@edgex compose-builder]# make gen ds-mqtt mqtt-broker no-secty
echo MQTT_VERBOSE=
MQTT_VERBOSE=
docker compose  -p edgex -f docker-compose-base.yml -f add-device-mqtt.yml -f add-mqtt-broker-mosquitto.yml convert > docker-compose.yml
rm -rf ./gen_ext_compose


[root@edgex compose-builder]# ls | grep 'docker-compose.yml'
docker-compose.yml
```



### 2.设备配置文件

**1.设备配置文件**

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



**2.设备配置**

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

`CommandTopic` 用于发布GET或SET命令请求



### 3.启动 EdgeX Foundry

使用以下命令部署 EdgeX：

```shell
$ cd edgex-compose/compose-builder
$ docker compose pull
$ docker compose -f docker-compose.yml up -d
$ docker compose up -d


# 修改配置文件
替换IP地址 127.0.0.1 为 0.0.0.0
```



```shell
# docker compose pull

# docker compose up -d
```



![](/images/edgex/device/link-mqtt/mqtt-2.png)

![](/images/edgex/device/link-mqtt/mqtt-3.png)

![](/images/edgex/device/link-mqtt/mqtt-4.png)



### 4.访问 UI

#### 4.1. consul

```shell
# 访问地址
http://192.168.202.233:8500
```

![](/images/edgex/device/link-mqtt/mqtt-5.png)



#### 4.2. EdgeX Console

```shell
# 访问地址
http://192.168.202.233:4000/
```



![](/images/edgex/device/link-mqtt/mqtt-6.png)

![](/images/edgex/device/link-mqtt/mqtt-7.png)

![](/images/edgex/device/link-mqtt/mqtt-8.png)

![](/images/edgex/device/link-mqtt/mqtt-9.png)



### 5.创建 MQTT 设备

#### 5.1.创建设备配置文件

![](/images/edgex/device/link-mqtt/mqtt-10.png)

![](/images/edgex/device/link-mqtt/mqtt-11.png)

![](/images/edgex/device/link-mqtt/mqtt-12.png)



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



#### 5.2.添加设备

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



![](/images/edgex/device/link-mqtt/mqtt-13.png)

![](/images/edgex/device/link-mqtt/mqtt-14.png)

![](/images/edgex/device/link-mqtt/mqtt-15.png)

![](/images/edgex/device/link-mqtt/mqtt-16.png)

![](/images/edgex/device/link-mqtt/mqtt-17.png)

![](/images/edgex/device/link-mqtt/mqtt-18.png)

![](/images/edgex/device/link-mqtt/mqtt-19.png)



### 6.运行模拟器

![](/images/edgex/device/link-mqtt/mqtt-20.png)



### 7.测试

#### 7.1.命令

![](/images/edgex/device/link-mqtt/mqtt-21.png)



#### 7.2.事件

![](/images/edgex/device/link-mqtt/mqtt-22.png)

```json
{
	"apiVersion": "v3",
	"id": "af46944b-e7e0-4d8b-bb6d-18d42721399b",
	"deviceName": "my-custom-device",
	"profileName": "my-custom-device-profile",
	"sourceName": "message",
	"origin": 1708341080603620600,
	"readings": [{
		"id": "edb1630b-7d15-49b8-97e3-1662520f7799",
		"origin": 1708341080603617300,
		"deviceName": "my-custom-device",
		"resourceName": "message",
		"profileName": "my-custom-device-profile",
		"valueType": "String",
		"value": "1111111"
	}]
}


{
	"apiVersion": "v3",
	"id": "99809eb7-83b9-49e3-9a5b-d65adc38a522",
	"deviceName": "my-custom-device",
	"profileName": "my-custom-device-profile",
	"sourceName": "values",
	"origin": 1708341090003785700,
	"readings": [{
		"id": "668df4eb-1404-4267-b0ee-464f1296e50b",
		"origin": 1708341090003772400,
		"deviceName": "my-custom-device",
		"resourceName": "randnum",
		"profileName": "my-custom-device-profile",
		"valueType": "Float32",
		"value": "2.650000e+01"
	}, {
		"id": "65067425-d134-4fc3-922f-2337d228cf0f",
		"origin": 1708341090003773700,
		"deviceName": "my-custom-device",
		"resourceName": "ping",
		"profileName": "my-custom-device-profile",
		"valueType": "String",
		"value": "pong"
	}, {
		"id": "fffe8f12-536c-43d9-9989-9d450d3b0b7b",
		"origin": 1708341090003774200,
		"deviceName": "my-custom-device",
		"resourceName": "message",
		"profileName": "my-custom-device-profile",
		"valueType": "String",
		"value": "Hello World"
	}]
}
```



#### 7.3.读值

![](/images/edgex/device/link-mqtt/mqtt-23.png)

```json
{
	"id": "aa29ebc7-2d31-4a4d-b58a-a8d85ba7903e",
	"origin": 1708341330008859000,
	"deviceName": "my-custom-device",
	"resourceName": "message",
	"profileName": "my-custom-device-profile",
	"valueType": "String",
	"value": "Hello World"
}

{
	"id": "448591a3-d4d7-4188-9943-88a289f9f54a",
	"origin": 1708341345006348800,
	"deviceName": "my-custom-device",
	"resourceName": "randnum",
	"profileName": "my-custom-device-profile",
	"valueType": "Float32",
	"value": "2.530000e+01"
}

{
	"id": "4af98f08-888a-495d-a07e-4c87dc6d2b82",
	"origin": 1708341345006350000,
	"deviceName": "my-custom-device",
	"resourceName": "ping",
	"profileName": "my-custom-device-profile",
	"valueType": "String",
	"value": "pong"
}

{
	"id": "26600864-c850-4d45-bf42-835dcf966249",
	"origin": 1708341345006350600,
	"deviceName": "my-custom-device",
	"resourceName": "message",
	"profileName": "my-custom-device-profile",
	"valueType": "String",
	"value": "Hello World"
}

{
	"id": "d70a679e-3532-4f7d-a3d1-0e53c8ad5f08",
	"origin": 1708341315007203800,
	"deviceName": "my-custom-device",
	"resourceName": "message",
	"profileName": "my-custom-device-profile",
	"valueType": "String",
	"value": "Hello World"
}
```



