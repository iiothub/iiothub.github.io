* TOC
{:toc}



## 一、概述



### 1.Swagger

ThingsBoard REST API交互式文档可通过Swagger UI获得。

```shell
# 访问地址
http://YOUR_HOST:PORT/swagger-ui.html
https://demo.thingsboard.io/swagger-ui/

http://192.168.202.188:8080/swagger-ui.html
tenant@thingsboard.org
tenant
```



![](/images/iot/api/api-swagger/swagger-1.png)

![](/images/iot/api/api-swagger/swagger-2.png)

![](/images/iot/api/api-swagger/swagger-3.png)



### 2.API认证

![](/images/iot/api/api-swagger/swagger-4.png)

![](/images/iot/api/api-swagger/swagger-5.png)



![](/images/iot/api/api-swagger/swagger-6.png)

![](/images/iot/api/api-swagger/swagger-7.png)



### 3.Postman

#### 3.1.login-endpoint

![](/images/iot/api/api-swagger/swagger-8.png)

![](/images/iot/api/api-swagger/swagger-9.png)



```shell
http://192.168.202.188:8080/swagger-ui.html
tenant@thingsboard.org
tenant

# 登录信息
http://192.168.202.188:8080/api/auth/login
tenant@thingsboard.org
tenant

# Request body
{
  "username": "tenant@thingsboard.org",
  "password": "tenant"
}

# Response
{
    "token": "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ0ZW5hbnRAdGhpbmdzYm9hcmQub3JnIiwidXNlcklkIjoiNDRlYjkzYzAtMzc2OC0xMWVlLThmMzEtYjkyYjFhMDgxZDE1Iiwic2NvcGVzIjpbIlRFTkFOVF9BRE1JTiJdLCJzZXNzaW9uSWQiOiIyZDNmOTQ4OS03MmNmLTQ2ODMtOGQzMi0zOWNmM2YzMTJkZjEiLCJpc3MiOiJ0aGluZ3Nib2FyZC5pbyIsImlhdCI6MTY5MzI0ODQ3MCwiZXhwIjoxNjkzMjU3NDcwLCJlbmFibGVkIjp0cnVlLCJpc1B1YmxpYyI6ZmFsc2UsInRlbmFudElkIjoiNDRhOWE4YzAtMzc2OC0xMWVlLThmMzEtYjkyYjFhMDgxZDE1IiwiY3VzdG9tZXJJZCI6IjEzODE0MDAwLTFkZDItMTFiMi04MDgwLTgwODA4MDgwODA4MCJ9.nEe55XHEKHwBLc5a7434csWCMIN17TKv9YtCZzwMtJvGrTB1KfYT7KRZzJJVFjWefLjqAo4pjptmU9t4ym61Kg",
    "refreshToken": "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ0ZW5hbnRAdGhpbmdzYm9hcmQub3JnIiwidXNlcklkIjoiNDRlYjkzYzAtMzc2OC0xMWVlLThmMzEtYjkyYjFhMDgxZDE1Iiwic2NvcGVzIjpbIlJFRlJFU0hfVE9LRU4iXSwic2Vzc2lvbklkIjoiMmQzZjk0ODktNzJjZi00NjgzLThkMzItMzljZjNmMzEyZGYxIiwiaXNzIjoidGhpbmdzYm9hcmQuaW8iLCJpYXQiOjE2OTMyNDg0NzAsImV4cCI6MTY5Mzg1MzI3MCwiaXNQdWJsaWMiOmZhbHNlLCJqdGkiOiI2OGNlYzFkZC1mYmNhLTQ0OTQtOWM3Ni00NGE3NTIxY2VlNWQifQ.b8Q9mdXuyuyE3DSupAmTx8ALWddCx07LPP5Cavqthqi48yGBFCdeh55jjKm2vbZDcbQxDglfBfg4BL2MMRqzcw",
    "scope": null
}
```



![](/images/iot/api/api-swagger/swagger-10.png)



#### 3.2.通过ID获取设备

**1.获取设备**

![](/images/iot/api/api-swagger/swagger-11.png)

![](/images/iot/api/api-swagger/swagger-12.png)

![](/images/iot/api/api-swagger/swagger-13.png)



**2.通过Swagger UI获取**

![](/images/iot/api/api-swagger/swagger-14.png)

![](/images/iot/api/api-swagger/swagger-15.png)

```json
{
  "id": {
    "entityType": "DEVICE",
    "id": "452fa1a0-3768-11ee-8f31-b92b1a081d15"
  },
  "createdTime": 1691663147194,
  "additionalInfo": null,
  "tenantId": {
    "entityType": "TENANT",
    "id": "44a9a8c0-3768-11ee-8f31-b92b1a081d15"
  },
  "customerId": {
    "entityType": "CUSTOMER",
    "id": "44f83df0-3768-11ee-8f31-b92b1a081d15"
  },
  "name": "Test Device A2",
  "type": "default",
  "label": null,
  "deviceProfileId": {
    "entityType": "DEVICE_PROFILE",
    "id": "44b31ea0-3768-11ee-8f31-b92b1a081d15"
  },
  "deviceData": {
    "configuration": {
      "type": "DEFAULT"
    },
    "transportConfiguration": {
      "type": "DEFAULT"
    }
  },
  "firmwareId": null,
  "softwareId": null,
  "externalId": null
}
```



**3.Postman测试**

```shell
http://192.168.202.188:8080/swagger-ui.html
tenant@thingsboard.org
tenant

# 登录信息
http://192.168.202.188:8080/api/device/
tenant@thingsboard.org
tenant

# 设备ID
Test Device A2
452fa1a0-3768-11ee-8f31-b92b1a081d15

# 访问地址
http://192.168.202.188:8080/api/device/452fa1a0-3768-11ee-8f31-b92b1a081d15

# token
"eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ0ZW5hbnRAdGhpbmdzYm9hcmQub3JnIiwidXNlcklkIjoiNDRlYjkzYzAtMzc2OC0xMWVlLThmMzEtYjkyYjFhMDgxZDE1Iiwic2NvcGVzIjpbIlRFTkFOVF9BRE1JTiJdLCJzZXNzaW9uSWQiOiI1YjM5MjUwMC00ZjhlLTQ1MzQtOWE4My03N2U1OTFjYzc3NGUiLCJpc3MiOiJ0aGluZ3Nib2FyZC5pbyIsImlhdCI6MTY5MzI1Nzc1NywiZXhwIjoxNjkzMjY2NzU3LCJlbmFibGVkIjp0cnVlLCJpc1B1YmxpYyI6ZmFsc2UsInRlbmFudElkIjoiNDRhOWE4YzAtMzc2OC0xMWVlLThmMzEtYjkyYjFhMDgxZDE1IiwiY3VzdG9tZXJJZCI6IjEzODE0MDAwLTFkZDItMTFiMi04MDgwLTgwODA4MDgwODA4MCJ9.ZYwTwVjGpxPx-HlOUl5tJY_POz6_EzXLQMeIQvQN0VqCkvpaYE_eTF9XQZb6VX2acRzkntc_fsXfN9vceJJTgw",

# Response
{
    "id": {
        "entityType": "DEVICE",
        "id": "452fa1a0-3768-11ee-8f31-b92b1a081d15"
    },
    "createdTime": 1691663147194,
    "additionalInfo": null,
    "tenantId": {
        "entityType": "TENANT",
        "id": "44a9a8c0-3768-11ee-8f31-b92b1a081d15"
    },
    "customerId": {
        "entityType": "CUSTOMER",
        "id": "44f83df0-3768-11ee-8f31-b92b1a081d15"
    },
    "name": "Test Device A2",
    "type": "default",
    "label": null,
    "deviceProfileId": {
        "entityType": "DEVICE_PROFILE",
        "id": "44b31ea0-3768-11ee-8f31-b92b1a081d15"
    },
    "deviceData": {
        "configuration": {
            "type": "DEFAULT"
        },
        "transportConfiguration": {
            "type": "DEFAULT"
        }
    },
    "firmwareId": null,
    "softwareId": null,
    "externalId": null
}
```



![](/images/iot/api/api-swagger/swagger-16.png)





## 二、RestTemplate



### 1.login

```java
        String username = "tenant@thingsboard.org";
        String password = "tenant";
        String baseURL = "http://192.168.202.188:8080";
        RestTemplate restTemplate = new RestTemplate();

        // 登录
        long ts = System.currentTimeMillis();
        Map<String, String> loginRequest = new HashMap();
        loginRequest.put("username", username);
        loginRequest.put("password", password);
        ResponseEntity<JsonNode> tokenInfo1 = restTemplate.postForEntity(baseURL + "/api/auth/login", loginRequest, JsonNode.class, new Object[0]);
        JsonNode tokenInfo = (JsonNode)tokenInfo1.getBody();
        System.out.println(tokenInfo);

        // 解析数据
        String mainToken = tokenInfo.get("token").asText();
        String refreshToken = tokenInfo.get("refreshToken").asText();
        //long mainTokenExpTs = JWT.decode(mainToken).getExpiresAtAsInstant().toEpochMilli();
        //long refreshTokenExpTs = JWT.decode(refreshToken).getExpiresAtAsInstant().toEpochMilli();
        //long clientServerTimeDiff = JWT.decode(mainToken).getIssuedAtAsInstant().toEpochMilli() - ts;
```



```java
        String username = "tenant@thingsboard.org";
        String password = "tenant";
        String baseURL = "http://192.168.202.188:8080";
        RestTemplate restTemplate = new RestTemplate();

        // 登录
        Map<String, String> loginRequest = new HashMap();
        loginRequest.put("username", username);
        loginRequest.put("password", password);
        HttpHeaders headers1 = new HttpHeaders();
        //headers.set("Content-Type", "application/json");
        headers1.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity entity1 = new HttpEntity<>(loginRequest,headers1);
        //ResponseEntity response1 = restTemplate.exchange(baseURL+"/api/auth/login", HttpMethod.POST, entity1, String.class, new Object[]{});
        ResponseEntity response1 = restTemplate.exchange(baseURL+"/api/auth/login", HttpMethod.POST, entity1, String.class);
        System.out.println(response1);

        String mainToken = "";
        String refreshToken = "";

        if( response1.getStatusCodeValue() == 200 ){
            System.out.println(response1.getBody());

            String login = (String)response1.getBody();
            ObjectMapper objectMapper = new ObjectMapper();
            JsonNode jsonNode = objectMapper.readTree(login);
            // 解析数据
            mainToken = jsonNode.get("token").asText();
            refreshToken = jsonNode.get("refreshToken").asText();
            //long mainTokenExpTs = JWT.decode(mainToken).getExpiresAtAsInstant().toEpochMilli();
            //long refreshTokenExpTs = JWT.decode(refreshToken).getExpiresAtAsInstant().toEpochMilli();
            //long clientServerTimeDiff = JWT.decode(mainToken).getIssuedAtAsInstant().toEpochMilli() - ts;

        }
```



```java
package com.iiotos.rest;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.http.*;
import org.springframework.web.client.RestTemplate;
import java.util.HashMap;
import java.util.Map;


public class Login {

    public static void main(String[] args) throws JsonProcessingException {

        String username = "tenant@thingsboard.org";
        String password = "tenant";
        String baseURL = "http://192.168.202.188:8080";
        RestTemplate restTemplate = new RestTemplate();

        // 登录
        long ts = System.currentTimeMillis();
        Map<String, String> loginRequest = new HashMap();
        loginRequest.put("username", username);
        loginRequest.put("password", password);
        ResponseEntity<JsonNode> tokenInfo1 = restTemplate.postForEntity(baseURL + "/api/auth/login", loginRequest, JsonNode.class, new Object[0]);
        JsonNode tokenInfo = (JsonNode)tokenInfo1.getBody();
        System.out.println(tokenInfo);

        // 解析数据
        String mainToken = tokenInfo.get("token").asText();
        String refreshToken = tokenInfo.get("refreshToken").asText();
        //long mainTokenExpTs = JWT.decode(mainToken).getExpiresAtAsInstant().toEpochMilli();
        //long refreshTokenExpTs = JWT.decode(refreshToken).getExpiresAtAsInstant().toEpochMilli();
        //long clientServerTimeDiff = JWT.decode(mainToken).getIssuedAtAsInstant().toEpochMilli() - ts;


        // 获取设备  452fa1a0-3768-11ee-8f31-b92b1a081d15
        String deviceId = "452fa1a0-3768-11ee-8f31-b92b1a081d15";
        HttpHeaders headers = new HttpHeaders();
        //headers.add("Authorization", "Bearer "+ mainToken);
        headers.set("X-Authorization", "Bearer "+ mainToken);
        //headers.set("Content-Type", "application/json");
        headers.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity entity = new HttpEntity<>(headers);
        //ResponseEntity response = restTemplate.exchange(baseURL+"/api/device/452fa1a0-3768-11ee-8f31-b92b1a081d15", HttpMethod.GET, entity, String.class);
        //ResponseEntity response = restTemplate.exchange(baseURL+"/api/device/{deviceId }", HttpMethod.GET, entity, String.class, new Object[]{deviceId});
        ResponseEntity response = restTemplate.exchange(baseURL+"/api/device/{deviceId }", HttpMethod.GET, entity, String.class, new Object[]{deviceId});
        System.out.println(response);
        if( response.getStatusCodeValue() == 200 ){
            String device = (String)response.getBody();
            System.out.println(response.getBody());

            ObjectMapper objectMapper = new ObjectMapper();
            JsonNode jsonNode = objectMapper.readTree(device);
            String id = jsonNode.get("id").toString();
            System.out.println(id);
        }
    }
}
```



### 2.Get

```java
        // 地址：api/device/{deviceId}
        // 设备ID  452fa1a0-3768-11ee-8f31-b92b1a081d15
        String deviceId = "452fa1a0-3768-11ee-8f31-b92b1a081d15";
        HttpHeaders headers = new HttpHeaders();
        //headers.add("Authorization", "Bearer "+ mainToken);
        headers.set("X-Authorization", "Bearer "+ mainToken);
        //headers.set("Content-Type", "application/json");
        headers.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity entity = new HttpEntity<>(headers);
        //ResponseEntity response = restTemplate.exchange(baseURL+"/api/device/452fa1a0-3768-11ee-8f31-b92b1a081d15", HttpMethod.GET, entity, String.class);
        //ResponseEntity response = restTemplate.exchange(baseURL+"/api/device/{deviceId }", HttpMethod.GET, entity, String.class, new Object[]{deviceId});
        ResponseEntity response = restTemplate.exchange(baseURL+"/api/device/{deviceId}", HttpMethod.GET, entity, String.class, new Object[]{deviceId});
        System.out.println(response);
        if( response.getStatusCodeValue() == 200 ){
            String device = (String)response.getBody();
            System.out.println(response.getBody());

            ObjectMapper objectMapper = new ObjectMapper();
            JsonNode jsonNode = objectMapper.readTree(device);
            String id = jsonNode.get("id").toString();
            System.out.println(id);
        }
```



```java
package com.iiotos.rest;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.http.*;
import org.springframework.web.client.RestTemplate;
import java.util.HashMap;
import java.util.Map;

public class GetRest {

    public static void main(String[] args) throws JsonProcessingException {

        String username = "tenant@thingsboard.org";
        String password = "tenant";
        String baseURL = "http://192.168.202.188:8080";
        RestTemplate restTemplate = new RestTemplate();

        // 登录
        Map<String, String> loginRequest = new HashMap();
        loginRequest.put("username", username);
        loginRequest.put("password", password);
        HttpHeaders headers1 = new HttpHeaders();
        //headers.set("Content-Type", "application/json");
        headers1.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity entity1 = new HttpEntity<>(loginRequest,headers1);
        //ResponseEntity response1 = restTemplate.exchange(baseURL+"/api/auth/login", HttpMethod.POST, entity1, String.class, new Object[]{});
        ResponseEntity response1 = restTemplate.exchange(baseURL+"/api/auth/login", HttpMethod.POST, entity1, String.class);
        System.out.println(response1);

        String mainToken = "";
        String refreshToken = "";

        if( response1.getStatusCodeValue() == 200 ){
            System.out.println(response1.getBody());

            String login = (String)response1.getBody();
            ObjectMapper objectMapper = new ObjectMapper();
            JsonNode jsonNode = objectMapper.readTree(login);
            // 解析数据
            mainToken = jsonNode.get("token").asText();
            refreshToken = jsonNode.get("refreshToken").asText();
            //long mainTokenExpTs = JWT.decode(mainToken).getExpiresAtAsInstant().toEpochMilli();
            //long refreshTokenExpTs = JWT.decode(refreshToken).getExpiresAtAsInstant().toEpochMilli();
            //long clientServerTimeDiff = JWT.decode(mainToken).getIssuedAtAsInstant().toEpochMilli() - ts;

        }

        // 地址：api/device/{deviceId}
        // 设备ID  452fa1a0-3768-11ee-8f31-b92b1a081d15
        String deviceId = "452fa1a0-3768-11ee-8f31-b92b1a081d15";
        HttpHeaders headers = new HttpHeaders();
        //headers.add("Authorization", "Bearer "+ mainToken);
        headers.set("X-Authorization", "Bearer "+ mainToken);
        //headers.set("Content-Type", "application/json");
        headers.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity entity = new HttpEntity<>(headers);
        //ResponseEntity response = restTemplate.exchange(baseURL+"/api/device/452fa1a0-3768-11ee-8f31-b92b1a081d15", HttpMethod.GET, entity, String.class);
        //ResponseEntity response = restTemplate.exchange(baseURL+"/api/device/{deviceId }", HttpMethod.GET, entity, String.class, new Object[]{deviceId});
        ResponseEntity response = restTemplate.exchange(baseURL+"/api/device/{deviceId}", HttpMethod.GET, entity, String.class, new Object[]{deviceId});
        System.out.println(response);
        if( response.getStatusCodeValue() == 200 ){
            String device = (String)response.getBody();
            System.out.println(response.getBody());

            ObjectMapper objectMapper = new ObjectMapper();
            JsonNode jsonNode = objectMapper.readTree(device);
            String id = jsonNode.get("id").toString();
            System.out.println(id);
        }
    }
}
```



### 3.Post

```java
        // 地址：api/customer/{customerId}/device/{deviceId}
        // 设备ID：  452fa1a0-3768-11ee-8f31-b92b1a081d15
        // 租户ID：  44f83df0-3768-11ee-8f31-b92b1a081d15
        //http://192.168.202.188:8080/api/customer/44f83df0-3768-11ee-8f31-b92b1a081d15/device/452fa1a0-3768-11ee-8f31-b92b1a081d15'
        String deviceId = "452fa1a0-3768-11ee-8f31-b92b1a081d15";
        String customerId = "44f83df0-3768-11ee-8f31-b92b1a081d15";
        HttpHeaders headers = new HttpHeaders();
        //headers.add("Authorization", "Bearer "+ mainToken);
        headers.set("X-Authorization", "Bearer "+ mainToken);
        //headers.set("Content-Type", "application/json");
        headers.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity entity = new HttpEntity<>(headers);
        //ResponseEntity response = restTemplate.exchange(baseURL+"/api/device/452fa1a0-3768-11ee-8f31-b92b1a081d15", HttpMethod.GET, entity, String.class);
        //ResponseEntity response = restTemplate.exchange(baseURL+"/api/device/{deviceId }", HttpMethod.GET, entity, String.class, new Object[]{deviceId});
        ResponseEntity response = restTemplate.exchange(baseURL+"/api/customer/{customerId}/device/{deviceId}", HttpMethod.POST, entity, String.class, new Object[]{customerId,deviceId});
        System.out.println(response);
        if( response.getStatusCodeValue() == 200 ){
            String device = (String)response.getBody();
            System.out.println(response.getBody());

            ObjectMapper objectMapper = new ObjectMapper();
            JsonNode jsonNode = objectMapper.readTree(device);
            String id = jsonNode.get("id").toString();
            System.out.println(id);
        }
```



```java
package com.iiotos.rest;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.http.*;
import org.springframework.web.client.RestTemplate;
import java.util.HashMap;
import java.util.Map;

public class PostRest {

    public static void main(String[] args) throws JsonProcessingException {

        String username = "tenant@thingsboard.org";
        String password = "tenant";
        String baseURL = "http://192.168.202.188:8080";
        RestTemplate restTemplate = new RestTemplate();

        // 登录
        Map<String, String> loginRequest = new HashMap();
        loginRequest.put("username", username);
        loginRequest.put("password", password);
        HttpHeaders headers1 = new HttpHeaders();
        //headers.set("Content-Type", "application/json");
        headers1.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity entity1 = new HttpEntity<>(loginRequest,headers1);
        //ResponseEntity response1 = restTemplate.exchange(baseURL+"/api/auth/login", HttpMethod.POST, entity1, String.class, new Object[]{});
        ResponseEntity response1 = restTemplate.exchange(baseURL+"/api/auth/login", HttpMethod.POST, entity1, String.class);
        System.out.println(response1);

        String mainToken = "";
        String refreshToken = "";

        if( response1.getStatusCodeValue() == 200 ){
            System.out.println(response1.getBody());

            String login = (String)response1.getBody();
            ObjectMapper objectMapper = new ObjectMapper();
            JsonNode jsonNode = objectMapper.readTree(login);
            // 解析数据
            mainToken = jsonNode.get("token").asText();
            refreshToken = jsonNode.get("refreshToken").asText();
            //long mainTokenExpTs = JWT.decode(mainToken).getExpiresAtAsInstant().toEpochMilli();
            //long refreshTokenExpTs = JWT.decode(refreshToken).getExpiresAtAsInstant().toEpochMilli();
            //long clientServerTimeDiff = JWT.decode(mainToken).getIssuedAtAsInstant().toEpochMilli() - ts;

        }

        // 地址：api/customer/{customerId}/device/{deviceId}
        // 设备ID：  452fa1a0-3768-11ee-8f31-b92b1a081d15
        // 租户ID：  44f83df0-3768-11ee-8f31-b92b1a081d15
        //http://192.168.202.188:8080/api/customer/44f83df0-3768-11ee-8f31-b92b1a081d15/device/452fa1a0-3768-11ee-8f31-b92b1a081d15'
        String deviceId = "452fa1a0-3768-11ee-8f31-b92b1a081d15";
        String customerId = "44f83df0-3768-11ee-8f31-b92b1a081d15";
        HttpHeaders headers = new HttpHeaders();
        //headers.add("Authorization", "Bearer "+ mainToken);
        headers.set("X-Authorization", "Bearer "+ mainToken);
        //headers.set("Content-Type", "application/json");
        headers.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity entity = new HttpEntity<>(headers);
        //ResponseEntity response = restTemplate.exchange(baseURL+"/api/device/452fa1a0-3768-11ee-8f31-b92b1a081d15", HttpMethod.GET, entity, String.class);
        //ResponseEntity response = restTemplate.exchange(baseURL+"/api/device/{deviceId }", HttpMethod.GET, entity, String.class, new Object[]{deviceId});
        ResponseEntity response = restTemplate.exchange(baseURL+"/api/customer/{customerId}/device/{deviceId}", HttpMethod.POST, entity, String.class, new Object[]{customerId,deviceId});
        System.out.println(response);
        if( response.getStatusCodeValue() == 200 ){
            String device = (String)response.getBody();
            System.out.println(response.getBody());

            ObjectMapper objectMapper = new ObjectMapper();
            JsonNode jsonNode = objectMapper.readTree(device);
            String id = jsonNode.get("id").toString();
            System.out.println(id);
        }
    }
}
```



### 4.Delete

```java
        // 地址：api/customer/device/{deviceId}
        // 设备ID：  a9b9bf70-46bd-11ee-98b8-53a2451d428d
        //http://192.168.202.188:8080/api/customer/device/a9b9bf70-46bd-11ee-98b8-53a2451d428d'
        String deviceId = "a9b9bf70-46bd-11ee-98b8-53a2451d428d";
        HttpHeaders headers = new HttpHeaders();
        //headers.add("Authorization", "Bearer "+ mainToken);
        headers.set("X-Authorization", "Bearer "+ mainToken);
        //headers.set("Content-Type", "application/json");
        headers.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity entity = new HttpEntity<>(headers);
        //ResponseEntity response = restTemplate.exchange(baseURL+"/api/device/452fa1a0-3768-11ee-8f31-b92b1a081d15", HttpMethod.GET, entity, String.class);
        //ResponseEntity response = restTemplate.exchange(baseURL+"/api/device/{deviceId }", HttpMethod.GET, entity, String.class, new Object[]{deviceId});
        ResponseEntity response = restTemplate.exchange(baseURL+"/api/customer/device/{deviceId}", HttpMethod.DELETE, entity, String.class, new Object[]{deviceId});
        System.out.println(response);
        if( response.getStatusCodeValue() == 200 ){
            String device = (String)response.getBody();
            System.out.println(response.getBody());

            ObjectMapper objectMapper = new ObjectMapper();
            JsonNode jsonNode = objectMapper.readTree(device);
            String id = jsonNode.get("id").toString();
            System.out.println(id);
        }
```



```java
package com.iiotos.rest;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.http.*;
import org.springframework.web.client.RestTemplate;
import java.util.HashMap;
import java.util.Map;

public class DeleteRest {

    public static void main(String[] args) throws JsonProcessingException {

        String username = "tenant@thingsboard.org";
        String password = "tenant";
        String baseURL = "http://192.168.202.188:8080";
        RestTemplate restTemplate = new RestTemplate();

        // 登录
        Map<String, String> loginRequest = new HashMap();
        loginRequest.put("username", username);
        loginRequest.put("password", password);
        HttpHeaders headers1 = new HttpHeaders();
        //headers.set("Content-Type", "application/json");
        headers1.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity entity1 = new HttpEntity<>(loginRequest,headers1);
        //ResponseEntity response1 = restTemplate.exchange(baseURL+"/api/auth/login", HttpMethod.POST, entity1, String.class, new Object[]{});
        ResponseEntity response1 = restTemplate.exchange(baseURL+"/api/auth/login", HttpMethod.POST, entity1, String.class);
        System.out.println(response1);

        String mainToken = "";
        String refreshToken = "";

        if( response1.getStatusCodeValue() == 200 ){
            System.out.println(response1.getBody());

            String login = (String)response1.getBody();
            ObjectMapper objectMapper = new ObjectMapper();
            JsonNode jsonNode = objectMapper.readTree(login);
            // 解析数据
            mainToken = jsonNode.get("token").asText();
            refreshToken = jsonNode.get("refreshToken").asText();
            //long mainTokenExpTs = JWT.decode(mainToken).getExpiresAtAsInstant().toEpochMilli();
            //long refreshTokenExpTs = JWT.decode(refreshToken).getExpiresAtAsInstant().toEpochMilli();
            //long clientServerTimeDiff = JWT.decode(mainToken).getIssuedAtAsInstant().toEpochMilli() - ts;

        }

        // 地址：api/customer/device/{deviceId}
        // 设备ID：  a9b9bf70-46bd-11ee-98b8-53a2451d428d
        //http://192.168.202.188:8080/api/customer/device/a9b9bf70-46bd-11ee-98b8-53a2451d428d'
        String deviceId = "a9b9bf70-46bd-11ee-98b8-53a2451d428d";
        HttpHeaders headers = new HttpHeaders();
        //headers.add("Authorization", "Bearer "+ mainToken);
        headers.set("X-Authorization", "Bearer "+ mainToken);
        //headers.set("Content-Type", "application/json");
        headers.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity entity = new HttpEntity<>(headers);
        //ResponseEntity response = restTemplate.exchange(baseURL+"/api/device/452fa1a0-3768-11ee-8f31-b92b1a081d15", HttpMethod.GET, entity, String.class);
        //ResponseEntity response = restTemplate.exchange(baseURL+"/api/device/{deviceId }", HttpMethod.GET, entity, String.class, new Object[]{deviceId});
        ResponseEntity response = restTemplate.exchange(baseURL+"/api/customer/device/{deviceId}", HttpMethod.DELETE, entity, String.class, new Object[]{deviceId});
        System.out.println(response);
        if( response.getStatusCodeValue() == 200 ){
            String device = (String)response.getBody();
            System.out.println(response.getBody());

            ObjectMapper objectMapper = new ObjectMapper();
            JsonNode jsonNode = objectMapper.readTree(device);
            String id = jsonNode.get("id").toString();
            System.out.println(id);
        }
    }
}
```



