* TOC
{:toc}



## 一、概述



### 1.Eclipse Paho

#### 1.1.Eclipse Paho Java Client

```shell
https://eclipse.dev/paho/index.php?page=downloads.php
https://eclipse.dev/paho/index.php?page=clients/java/index.php

https://github.com/eclipse/paho.mqtt.java
```



![](/images/iot/device/device/test-9.png)

![](/images/iot/device/device/test-10.png)



#### 1.2.Getting Started

**Dependency definition for MQTTv3 client** 

```xml
<project ...>
<repositories>
    <repository>
        <id>Eclipse Paho Repo</id>
        <url>%REPOURL%</url>
    </repository>
</repositories>
...
<dependencies>
    <dependency>
        <groupId>org.eclipse.paho</groupId>
        <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
        <version>%VERSION%</version>
    </dependency>
</dependencies>
</project>
```



**Dependency definition for MQTTv5 client** 

```xml
<project ...>
<repositories>
    <repository>
        <id>Eclipse Paho Repo</id>
        <url>%REPOURL%</url>
    </repository>
</repositories>
...
<dependencies>
    <dependency>
        <groupId>org.eclipse.paho</groupId>
        <artifactId>org.eclipse.paho.mqttv5.client</artifactId>
        <version>%VERSION%</version>
    </dependency>
</dependencies>
</project>
```



The included code below is a very basic sample that connects to a server and publishes a message using the MQTTv3 synchronous API. More extensive samples demonstrating the use of the MQTTv3 and MQTTv5 Asynchronous API can be found in the `org.eclipse.paho.sample` directory of the source.

```java
import org.eclipse.paho.client.mqttv3.MqttClient;
import org.eclipse.paho.client.mqttv3.MqttConnectOptions;
import org.eclipse.paho.client.mqttv3.MqttException;
import org.eclipse.paho.client.mqttv3.MqttMessage;
import org.eclipse.paho.client.mqttv3.persist.MemoryPersistence;

public class MqttPublishSample {

    public static void main(String[] args) {

        String topic        = "MQTT Examples";
        String content      = "Message from MqttPublishSample";
        int qos             = 2;
        String broker       = "tcp://iot.eclipse.org:1883";
        String clientId     = "JavaSample";
        MemoryPersistence persistence = new MemoryPersistence();

        try {
            MqttClient sampleClient = new MqttClient(broker, clientId, persistence);
            MqttConnectOptions connOpts = new MqttConnectOptions();
            connOpts.setCleanSession(true);
            System.out.println("Connecting to broker: "+broker);
            sampleClient.connect(connOpts);
            System.out.println("Connected");
            System.out.println("Publishing message: "+content);
            MqttMessage message = new MqttMessage(content.getBytes());
            message.setQos(qos);
            sampleClient.publish(topic, message);
            System.out.println("Message published");
            sampleClient.disconnect();
            System.out.println("Disconnected");
            System.exit(0);
        } catch(MqttException me) {
            System.out.println("reason "+me.getReasonCode());
            System.out.println("msg "+me.getMessage());
            System.out.println("loc "+me.getLocalizedMessage());
            System.out.println("cause "+me.getCause());
            System.out.println("excep "+me);
            me.printStackTrace();
        }
    }
}
```



#### 1.3.黑马封装

```shell
Java自学教程Java物联网开发“尚方宝剑”之EMQ_黑马程序员
https://www.bilibili.com/video/BV1o5411E7V1?p=34&vd_source=1a41e015388b00909f69083f9831969b

```





## 二、MQTT客户端



### 1.客户端程序

**1.pom.xml**

```xml
        <dependency>
            <groupId>org.eclipse.paho</groupId>
            <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
            <version>1.2.5</version>
        </dependency>
```



**2.application.yml**

```yaml
server:
  port: 8999
spring:
  application:
    name: mqtt-client

mqtt:
  # MQTT地址
  broker-url: tcp://192.168.202.188:1883
  # 客户端ID，随机定义
  client-id: emq-client-1111
  # 设备访问令牌
  username: 12Wt6lPaEplx5qKWg9qZ
  # 密码
  password:
```



**3.工具类程序源码**

EmqClient

```java
package com.iiotos.mqtt;
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



MessageCallback

```java
package com.iiotos.mqtt;

import org.eclipse.paho.client.mqttv3.IMqttDeliveryToken;
import org.eclipse.paho.client.mqttv3.MqttCallback;
import org.eclipse.paho.client.mqttv3.MqttMessage;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

/**
 * Created by 传智播客*黑马程序员.
 */
@Component
public class MessageCallback implements MqttCallback {
    
    private static final Logger log = LoggerFactory.getLogger(MessageCallback.class);

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



MqttProperties

```java
package com.iiotos.mqtt;

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



QosEnum

```java
package com.iiotos.mqtt;

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



**4.测试代码**

```java
@EnableScheduling
@SpringBootApplication
public class MqttClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(MqttClientApplication.class, args);
    }

    @Autowired
    private EmqClient emqClient;

    @Autowired
    private MqttProperties properties;

    @PostConstruct
    public void init(){
        //连接服务端
        emqClient.connect(properties.getUsername(),properties.getPassword());
        //订阅一个主题
        emqClient.subscribe("testtopic/#", QosEnum.QoS2);
        //开启一个新的线程 每隔5秒去向 testtopic/123
        new Thread(()->{
            while (true){
                String content = "{\n" +
                        "\t\"mqtt-1\": 1003,\n" +
                        "\t\"mqtt-2\": \"java data3\"\n" +
                        "}";

                emqClient.publish("v1/devices/me/telemetry",content,
                        QosEnum.QoS1,false);
                try {
                    TimeUnit.SECONDS.sleep(5);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

            }
        }).start();
    }
}
```



### 2.客户端测试

**参考工程：mqtt-client**

**1.案例**

```java
@Component
public class CaseTest {

    @Autowired
    private EmqClient emqClient;

    @Autowired
    private MqttProperties properties;

    @PostConstruct
    public void init(){
        //连接服务端
        emqClient.connect(properties.getUsername(),properties.getPassword());
        //订阅一个主题
        emqClient.subscribe("testtopic/#", QosEnum.QoS2);
    }

    @Scheduled(fixedRate = 2000)
    public void publish(){
//        String content = "{\n" +
//                "\t\"mqtt-100\": 1000,\n" +
//                "\t\"mqtt-200\": \"java data\"\n" +
//                "}";

        String content = "{\n" +
                "\t\"mqtt-10\": 10011,\n" +
                "\t\"mqtt-20\": \"java data11\"\n" +
                "}";

        emqClient.publish("v1/devices/me/telemetry",content,
                QosEnum.QoS1,false);
    }
}
```



2.案例

```java
import com.iiotos.mqtt.EmqClient;
import com.iiotos.mqtt.MqttProperties;
import com.iiotos.mqtt.QosEnum;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import javax.annotation.PostConstruct;
import java.util.concurrent.TimeUnit;

@Component
public class BasicTest {

    @Autowired
    private EmqClient emqClient;

    @Autowired
    private MqttProperties properties;

    @PostConstruct
    public void init(){
        //连接服务端
        emqClient.connect(properties.getUsername(),properties.getPassword());
        //订阅一个主题
        emqClient.subscribe("testtopic/#", QosEnum.QoS2);
        //开启一个新的线程 每隔5秒去向 testtopic/123
        new Thread(()->{
            while (true){
                String content = "{\n" +
                        "\t\"mqtt-10\": 10011,\n" +
                        "\t\"mqtt-20\": \"java data11\"\n" +
                        "}";

                emqClient.publish("v1/devices/me/telemetry",content,
                        QosEnum.QoS1,false);
                try {
                    TimeUnit.SECONDS.sleep(5);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

            }
        }).start();
    }
}
```



