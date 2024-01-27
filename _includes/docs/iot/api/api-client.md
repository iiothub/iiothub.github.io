* TOC
{:toc}



## 一、概述



### 1.REST API

ThingsBoard REST API交互式文档可通过Swagger UI获得。

```shell
# 访问地址
http://YOUR_HOST:PORT/swagger-ui.html
https://demo.thingsboard.io/swagger-ui/

http://192.168.202.188:8080/swagger-ui.html
tenant@thingsboard.org
tenant
```



![](/images/iot/api/api-client/api-1.png)

![](/images/iot/api/api-client/api-2.png)

![](/images/iot/api/api-client/api-3.png)



### 2.JWT Tokens

ThingsBoard使用JWT令牌在API客户端（浏览器、脚本等）和平台之间安全地表示声明。当您登录到该平台时，您的用户名和密码将交换到这对令牌中。

- 主令牌是应该用于执行API调用的短时间令牌
- 刷新令牌用于在主令牌过期后获取新的主令牌
- 主令牌和刷新令牌的到期时间可在系统设置中通过JWT_TOKEN_expiration_time和JWT_refresh_TOKEN_EEXPIRATION_time参数进行配置
- 默认的过期时间值分别为2.5小时和1周



请参阅下面的示例命令以获取用户的令牌“tenant@thingsboard.org”，密码“租户”和服务器“THINGSBOARD_URL”：

```shell
curl -X POST --header 'Content-Type: application/json' --header 'Accept: application/json' -d '{"username":"tenant@thingsboard.org", "password":"tenant"}' 'http://THINGSBOARD_URL/api/auth/login'

{"token":"$YOUR_JWT_TOKEN", "refreshToken":"$YOUR_JWT_REFRESH_TOKEN"}
```



```shell
curl -X POST --header 'Content-Type: application/json' --header 'Accept: application/json' -d '{"username":"tenant@thingsboard.org", "password":"tenant"}' 'http://192.168.202.188:8080/api/auth/login'


[root@192 ~]# curl -X POST --header 'Content-Type: application/json' --header 'Accept: application/json' -d '{"username":"tenant@thingsboard.org", "password":"tenant"}' 'http://192.168.202.188:8080/api/auth/login'
{"token":"eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ0ZW5hbnRAdGhpbmdzYm9hcmQub3JnIiwidXNlcklkIjoiNDRlYjkzYzAtMzc2OC0xMWVlLThmMzEtYjkyYjFhMDgxZDE1Iiwic2NvcGVzIjpbIlRFTkFOVF9BRE1JTiJdLCJzZXNzaW9uSWQiOiJkY2JmMTY3YS0zMDE0LTRhY2QtYmZhYS1jZWI4YjJkMWUzYzkiLCJpc3MiOiJ0aGluZ3Nib2FyZC5pbyIsImlhdCI6MTY5MzE1ODA1MCwiZXhwIjoxNjkzMTY3MDUwLCJlbmFibGVkIjp0cnVlLCJpc1B1YmxpYyI6ZmFsc2UsInRlbmFudElkIjoiNDRhOWE4YzAtMzc2OC0xMWVlLThmMzEtYjkyYjFhMDgxZDE1IiwiY3VzdG9tZXJJZCI6IjEzODE0MDAwLTFkZDItMTFiMi04MDgwLTgwODA4MDgwODA4MCJ9.HdYB1-vsN3FOaQCNiR5uh9vQRNAGg9eqfQ7Q24OuJdV8hqEaRgFLgkBFjxgs1446aVE_l5Mm98vaJTD3ISyuwQ","refreshToken":"eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ0ZW5hbnRAdGhpbmdzYm9hcmQub3JnIiwidXNlcklkIjoiNDRlYjkzYzAtMzc2OC0xMWVlLThmMzEtYjkyYjFhMDgxZDE1Iiwic2NvcGVzIjpbIlJFRlJFU0hfVE9LRU4iXSwic2Vzc2lvbklkIjoiZGNiZjE2N2EtMzAxNC00YWNkLWJmYWEtY2ViOGIyZDFlM2M5IiwiaXNzIjoidGhpbmdzYm9hcmQuaW8iLCJpYXQiOjE2OTMxNTgwNTAsImV4cCI6MTY5Mzc2Mjg1MCwiaXNQdWJsaWMiOmZhbHNlLCJqdGkiOiIzMmFjMDRkNy0zNGE3LTQ2MDktYjc0Ny0xY2FhNTcwZDViYzAifQ.w4tow_VoxEEt8M-CX5hLKa98tgim_Z5FgWGxseI8Y4Hnx-Mwz4599HvU--mq1IkEzz6w36TCz9Q1vuSZ9EpZDg","scope":null}


# 格式化后
{
	"token": "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ0ZW5hbnRAdGhpbmdzYm9hcmQub3JnIiwidXNlcklkIjoiNDRlYjkzYzAtMzc2OC0xMWVlLThmMzEtYjkyYjFhMDgxZDE1Iiwic2NvcGVzIjpbIlRFTkFOVF9BRE1JTiJdLCJzZXNzaW9uSWQiOiJkY2JmMTY3YS0zMDE0LTRhY2QtYmZhYS1jZWI4YjJkMWUzYzkiLCJpc3MiOiJ0aGluZ3Nib2FyZC5pbyIsImlhdCI6MTY5MzE1ODA1MCwiZXhwIjoxNjkzMTY3MDUwLCJlbmFibGVkIjp0cnVlLCJpc1B1YmxpYyI6ZmFsc2UsInRlbmFudElkIjoiNDRhOWE4YzAtMzc2OC0xMWVlLThmMzEtYjkyYjFhMDgxZDE1IiwiY3VzdG9tZXJJZCI6IjEzODE0MDAwLTFkZDItMTFiMi04MDgwLTgwODA4MDgwODA4MCJ9.HdYB1-vsN3FOaQCNiR5uh9vQRNAGg9eqfQ7Q24OuJdV8hqEaRgFLgkBFjxgs1446aVE_l5Mm98vaJTD3ISyuwQ",
	
	
	"refreshToken": "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ0ZW5hbnRAdGhpbmdzYm9hcmQub3JnIiwidXNlcklkIjoiNDRlYjkzYzAtMzc2OC0xMWVlLThmMzEtYjkyYjFhMDgxZDE1Iiwic2NvcGVzIjpbIlJFRlJFU0hfVE9LRU4iXSwic2Vzc2lvbklkIjoiZGNiZjE2N2EtMzAxNC00YWNkLWJmYWEtY2ViOGIyZDFlM2M5IiwiaXNzIjoidGhpbmdzYm9hcmQuaW8iLCJpYXQiOjE2OTMxNTgwNTAsImV4cCI6MTY5Mzc2Mjg1MCwiaXNQdWJsaWMiOmZhbHNlLCJqdGkiOiIzMmFjMDRkNy0zNGE3LTQ2MDktYjc0Ny0xY2FhNTcwZDViYzAifQ.w4tow_VoxEEt8M-CX5hLKa98tgim_Z5FgWGxseI8Y4Hnx-Mwz4599HvU--mq1IkEzz6w36TCz9Q1vuSZ9EpZDg",
	"scope": null
}
```



### 3.Java客户端

ThingsBoard REST API客户端可帮助您从Java应用程序与ThingsBoard REST API进行交互。使用Rest Client，您可以在ThingsBoard中以编程方式创建资产、设备、客户、用户和其他实体及其关系。

安装Rest Client的推荐方法是使用构建自动化工具，如Maven。REST客户端的版本取决于您正在使用的平台的版本。



注意：REST客户端是在Spring RestTemplate之上构建的，因此依赖于Spring Web（在撰写本文时为5.1.5.RELEASE）。

```xml
<dependencies>
    <dependency>
        <groupId>org.thingsboard</groupId>
        <artifactId>rest-client</artifactId>
        <version>3.5.1</version>
    </dependency>
</dependencies>
```



为了下载REST客户端依赖项，您应该将以下存储库添加到您的项目中。或者，您可以从源代码构建REST客户端。

```xml
<repositories>
    <repository>
        <id>thingsboard</id>
        <url>https://repo.thingsboard.io/artifactory/libs-release-public</url>
    </repository>
</repositories>
```



```java
// ThingsBoard REST API URL
String url = "http://localhost:8080";

// Default Tenant Administrator credentials
String username = "tenant@thingsboard.org";
String password = "tenant";

// Creating new rest client and auth with credentials
RestClient client = new RestClient(url);
client.login(username, password);

// Get information of current logged in user and print it
client.getUser().ifPresent(System.out::println);

// Perform logout of current user and close the client
client.logout();
client.close();
```





## 二、Java客户端



### 1.客户端登录

#### 1.1.pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.10.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <groupId>com.iiotos</groupId>
    <artifactId>tb-test</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>tb-test</name>
    <description>tb-test</description>

    <properties>
        <java.version>11</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.thingsboard</groupId>
            <artifactId>rest-client</artifactId>
            <version>3.5.1</version>
        </dependency>


        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <repositories>
        <repository>
            <id>thingsboard</id>
            <url>https://repo.thingsboard.io/artifactory/libs-release-public</url>
        </repository>
    </repositories>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```



```xml
    <properties>
        <java.version>11</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.thingsboard</groupId>
            <artifactId>rest-client</artifactId>
            <version>3.5.1</version>
        </dependency>
    </dependencies>

    <repositories>
        <repository>
            <id>thingsboard</id>
            <url>https://repo.thingsboard.io/artifactory/libs-release-public</url>
        </repository>
    </repositories>
```



#### 1.2.程序源码

```java
package com.iiotos.api;
import org.thingsboard.rest.client.RestClient;

public class Login {

    public static void main(String[] args) {
        // ThingsBoard REST API URL
        String url = "http://192.168.202.188:8080";
        // Default Tenant Administrator credentials
        String username = "tenant@thingsboard.org";
        String password = "tenant";

        // Creating new rest client and auth with credentials
        RestClient client = new RestClient(url);
        client.login(username, password);

        // Get information of current logged in user and print it
        client.getUser().ifPresent(System.out::println);

        // Perform logout of current user and close the client
        client.logout();
        client.close();
    }
}
```



```shell
Connected to the target VM, address: '127.0.0.1:4339', transport: 'socket'
09:29:11.448 [main] WARN org.springframework.http.converter.json.Jackson2ObjectMapperBuilder - For Jackson Kotlin classes support please add "com.fasterxml.jackson.module:jackson-module-kotlin" to the classpath
09:29:19.187 [main] DEBUG org.springframework.web.client.RestTemplate - HTTP POST http://192.168.202.188:8080/api/auth/login
09:29:19.199 [main] DEBUG org.springframework.web.client.RestTemplate - Accept=[application/json, application/cbor, application/*+json]
09:29:19.216 [main] DEBUG org.springframework.web.client.RestTemplate - Writing [{password=tenant, username=tenant@thingsboard.org}] with org.springframework.http.converter.json.MappingJackson2HttpMessageConverter
09:29:19.336 [main] DEBUG org.springframework.web.client.RestTemplate - Response 200 OK
09:29:19.344 [main] DEBUG org.springframework.web.client.RestTemplate - Reading to [com.fasterxml.jackson.databind.JsonNode]
09:29:28.232 [main] DEBUG org.springframework.web.client.RestTemplate - HTTP GET http://192.168.202.188:8080/api/auth/user
09:29:28.402 [main] DEBUG org.springframework.web.client.RestTemplate - Accept=[application/json, application/cbor, application/*+json]
09:29:28.419 [main] DEBUG org.springframework.web.client.RestTemplate - Response 200 OK
09:29:28.420 [main] DEBUG org.springframework.web.client.RestTemplate - Reading to [org.thingsboard.server.common.data.User]
User [tenantId=44a9a8c0-3768-11ee-8f31-b92b1a081d15, customerId=13814000-1dd2-11b2-8080-808080808080, email=tenant@thingsboard.org, authority=TENANT_ADMIN, firstName=null, lastName=null, additionalInfo={"failedLoginAttempts":0,"lastLoginTs":1693214959005}, createdTime=1691663146748, id=44eb93c0-3768-11ee-8f31-b92b1a081d15]
09:29:35.959 [main] DEBUG org.springframework.web.client.RestTemplate - HTTP POST http://192.168.202.188:8080/api/auth/logout
09:29:35.978 [main] DEBUG org.springframework.web.client.RestTemplate - Response 200 OK
Disconnected from the target VM, address: '127.0.0.1:4339', transport: 'socket'

Process finished with exit code 0
```

![](/images/iot/api/api-client/api-4.png)



### 2.获取客户设备

#### 2.1.pom.xml

```xml
    <properties>
        <java.version>11</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.thingsboard</groupId>
            <artifactId>rest-client</artifactId>
            <version>3.5.1</version>
        </dependency>

    </dependencies>

    <repositories>
        <repository>
            <id>thingsboard</id>
            <url>https://repo.thingsboard.io/artifactory/libs-release-public</url>
        </repository>
    </repositories>
```



#### 2.2.程序源码

```java
package com.iiotos.api;
import org.thingsboard.rest.client.RestClient;
import org.thingsboard.server.common.data.Device;
import org.thingsboard.server.common.data.User;
import org.thingsboard.server.common.data.page.PageData;
import org.thingsboard.server.common.data.page.PageLink;

public class GetDevice {

    public static void main(String[] args) {

        // ThingsBoard REST API URL
        String url = "http://192.168.202.188:8080";
        // Perform login with default Customer User credentials
        //String username = "tenant@thingsboard.org";
        //String password = "tenant";
        String username = "customer@thingsboard.org";
        String password = "customer";
        RestClient client = new RestClient(url);
        client.login(username, password);

        PageData<Device> pageData;
        PageLink pageLink = new PageLink(10);
        do {
            // Get current user
            User user = client.getUser().orElseThrow(() -> new IllegalStateException("No logged in user has been found"));
            // Fetch customer devices using current page link
            pageData = client.getCustomerDevices(user.getCustomerId(), "", pageLink);
            pageData.getData().forEach(System.out::println);
            pageLink = pageLink.nextPageLink();
        } while (pageData.hasNext());

        // Perform logout of current user and close the client
        client.logout();
        client.close();
    }
}
```



```shell
"C:\Program Files\OpenLogic\jdk-11.0.20.8-hotspot\bin\java.exe" -agentlib:jdwp=transport=dt_socket,address=127.0.0.1:6435,suspend=y,server=n -javaagent:C:\Users\Administrator\AppData\Local\JetBrains\IntelliJIdea2022.2\captureAgent\debugger-agent.jar -Dfile.encoding=UTF-8 -classpath "D:\ThingsBoard\dev\code\tb-api\tb-test\target\classes;F:\dev\maven\MavenRepository\org\thingsboard\rest-client\3.5.1\rest-client-3.5.1.jar;F:\dev\maven\MavenRepository\org\thingsboard\common\data\3.5.1\data-3.5.1.jar;F:\dev\maven\MavenRepository\javax\validation\validation-api\2.0.1.Final\validation-api-2.0.1.Final.jar;F:\dev\maven\MavenRepository\org\owasp\antisamy\antisamy\1.7.2\antisamy-1.7.2.jar;F:\dev\maven\MavenRepository\net\sourceforge\htmlunit\neko-htmlunit\2.66.0\neko-htmlunit-2.66.0.jar;F:\dev\maven\MavenRepository\org\apache\httpcomponents\client5\httpclient5\5.2\httpclient5-5.2.jar;F:\dev\maven\MavenRepository\org\apache\httpcomponents\core5\httpcore5-h2\5.2\httpcore5-h2-5.2.jar;F:\dev\maven\MavenRepository\org\apache\httpcomponents\core5\httpcore5\5.2\httpcore5-5.2.jar;F:\dev\maven\MavenRepository\org\apache\xmlgraphics\batik-css\1.16\batik-css-1.16.jar;F:\dev\maven\MavenRepository\org\apache\xmlgraphics\batik-shared-resources\1.16\batik-shared-resources-1.16.jar;F:\dev\maven\MavenRepository\org\apache\xmlgraphics\batik-util\1.16\batik-util-1.16.jar;F:\dev\maven\MavenRepository\org\apache\xmlgraphics\batik-constants\1.16\batik-constants-1.16.jar;F:\dev\maven\MavenRepository\org\apache\xmlgraphics\batik-i18n\1.16\batik-i18n-1.16.jar;F:\dev\maven\MavenRepository\org\apache\xmlgraphics\xmlgraphics-commons\2.7\xmlgraphics-commons-2.7.jar;F:\dev\maven\MavenRepository\commons-io\commons-io\2.11.0\commons-io-2.11.0.jar;F:\dev\maven\MavenRepository\xerces\xercesImpl\2.12.2\xercesImpl-2.12.2.jar;F:\dev\maven\MavenRepository\xml-apis\xml-apis\1.4.01\xml-apis-1.4.01.jar;F:\dev\maven\MavenRepository\xml-apis\xml-apis-ext\1.3.04\xml-apis-ext-1.3.04.jar;F:\dev\maven\MavenRepository\org\slf4j\slf4j-api\1.7.30\slf4j-api-1.7.30.jar;F:\dev\maven\MavenRepository\org\slf4j\log4j-over-slf4j\1.7.30\log4j-over-slf4j-1.7.30.jar;F:\dev\maven\MavenRepository\ch\qos\logback\logback-core\1.2.3\logback-core-1.2.3.jar;F:\dev\maven\MavenRepository\ch\qos\logback\logback-classic\1.2.3\logback-classic-1.2.3.jar;F:\dev\maven\MavenRepository\com\fasterxml\jackson\core\jackson-databind\2.11.4\jackson-databind-2.11.4.jar;F:\dev\maven\MavenRepository\com\fasterxml\jackson\core\jackson-annotations\2.11.4\jackson-annotations-2.11.4.jar;F:\dev\maven\MavenRepository\com\fasterxml\jackson\core\jackson-core\2.11.4\jackson-core-2.11.4.jar;F:\dev\maven\MavenRepository\org\springframework\data\spring-data-commons\2.3.9.RELEASE\spring-data-commons-2.3.9.RELEASE.jar;F:\dev\maven\MavenRepository\com\squareup\wire\wire-schema\3.4.0\wire-schema-3.4.0.jar;F:\dev\maven\MavenRepository\com\google\guava\guava\20.0\guava-20.0.jar;F:\dev\maven\MavenRepository\org\jetbrains\kotlin\kotlin-stdlib\1.3.72\kotlin-stdlib-1.3.72.jar;F:\dev\maven\MavenRepository\org\jetbrains\annotations\13.0\annotations-13.0.jar;F:\dev\maven\MavenRepository\com\squareup\wire\wire-runtime\3.4.0\wire-runtime-3.4.0.jar;F:\dev\maven\MavenRepository\org\jetbrains\kotlin\kotlin-stdlib-jdk8\1.3.72\kotlin-stdlib-jdk8-1.3.72.jar;F:\dev\maven\MavenRepository\org\jetbrains\kotlin\kotlin-stdlib-jdk7\1.3.72\kotlin-stdlib-jdk7-1.3.72.jar;F:\dev\maven\MavenRepository\org\jetbrains\kotlin\kotlin-stdlib-common\1.3.72\kotlin-stdlib-common-1.3.72.jar;F:\dev\maven\MavenRepository\com\squareup\okio\okio\2.8.0\okio-2.8.0.jar;F:\dev\maven\MavenRepository\org\thingsboard\protobuf-dynamic\1.0.4TB\protobuf-dynamic-1.0.4TB.jar;F:\dev\maven\MavenRepository\com\google\protobuf\protobuf-java\3.21.9\protobuf-java-3.21.9.jar;F:\dev\maven\MavenRepository\org\apache\commons\commons-lang3\3.10\commons-lang3-3.10.jar;F:\dev\maven\MavenRepository\commons-codec\commons-codec\1.14\commons-codec-1.14.jar;F:\dev\maven\MavenRepository\io\swagger\swagger-annotations\1.6.3\swagger-annotations-1.6.3.jar;F:\dev\maven\MavenRepository\de\ruedigermoeller\fst\2.57\fst-2.57.jar;F:\dev\maven\MavenRepository\org\javassist\javassist\3.21.0-GA\javassist-3.21.0-GA.jar;F:\dev\maven\MavenRepository\com\google\protobuf\protobuf-java-util\3.21.9\protobuf-java-util-3.21.9.jar;F:\dev\maven\MavenRepository\com\google\errorprone\error_prone_annotations\2.5.1\error_prone_annotations-2.5.1.jar;F:\dev\maven\MavenRepository\com\google\j2objc\j2objc-annotations\1.3\j2objc-annotations-1.3.jar;F:\dev\maven\MavenRepository\com\google\code\findbugs\jsr305\3.0.2\jsr305-3.0.2.jar;F:\dev\maven\MavenRepository\com\google\code\gson\gson\2.8.6\gson-2.8.6.jar;F:\dev\maven\MavenRepository\org\eclipse\leshan\leshan-core\2.0.0-M5\leshan-core-2.0.0-M5.jar;F:\dev\maven\MavenRepository\com\eclipsesource\minimal-json\minimal-json\0.9.5\minimal-json-0.9.5.jar;F:\dev\maven\MavenRepository\com\fasterxml\jackson\dataformat\jackson-dataformat-cbor\2.11.4\jackson-dataformat-cbor-2.11.4.jar;F:\dev\maven\MavenRepository\com\upokecenter\cbor\4.4.4\cbor-4.4.4.jar;F:\dev\maven\MavenRepository\com\github\peteroupc\numbers\1.8.0\numbers-1.8.0.jar;F:\dev\maven\MavenRepository\org\springframework\spring-web\5.2.14.RELEASE\spring-web-5.2.14.RELEASE.jar;F:\dev\maven\MavenRepository\org\springframework\spring-beans\5.2.14.RELEASE\spring-beans-5.2.14.RELEASE.jar;F:\dev\maven\MavenRepository\org\thingsboard\common\util\3.5.1\util-3.5.1.jar;F:\dev\maven\MavenRepository\javax\annotation\javax.annotation-api\1.3.2\javax.annotation-api-1.3.2.jar;F:\dev\maven\MavenRepository\com\github\oshi\oshi-core\6.4.2\oshi-core-6.4.2.jar;F:\dev\maven\MavenRepository\net\java\dev\jna\jna\5.13.0\jna-5.13.0.jar;F:\dev\maven\MavenRepository\net\java\dev\jna\jna-platform\5.13.0\jna-platform-5.13.0.jar;F:\dev\maven\MavenRepository\com\auth0\java-jwt\4.2.1\java-jwt-4.2.1.jar;F:\dev\maven\MavenRepository\org\springframework\boot\spring-boot-starter-web\2.3.10.RELEASE\spring-boot-starter-web-2.3.10.RELEASE.jar;F:\dev\maven\MavenRepository\org\springframework\boot\spring-boot-starter\2.3.10.RELEASE\spring-boot-starter-2.3.10.RELEASE.jar;F:\dev\maven\MavenRepository\org\springframework\boot\spring-boot\2.3.10.RELEASE\spring-boot-2.3.10.RELEASE.jar;F:\dev\maven\MavenRepository\org\springframework\boot\spring-boot-autoconfigure\2.3.10.RELEASE\spring-boot-autoconfigure-2.3.10.RELEASE.jar;F:\dev\maven\MavenRepository\org\springframework\boot\spring-boot-starter-logging\2.3.10.RELEASE\spring-boot-starter-logging-2.3.10.RELEASE.jar;F:\dev\maven\MavenRepository\org\apache\logging\log4j\log4j-to-slf4j\2.13.3\log4j-to-slf4j-2.13.3.jar;F:\dev\maven\MavenRepository\org\apache\logging\log4j\log4j-api\2.13.3\log4j-api-2.13.3.jar;F:\dev\maven\MavenRepository\org\slf4j\jul-to-slf4j\1.7.30\jul-to-slf4j-1.7.30.jar;F:\dev\maven\MavenRepository\jakarta\annotation\jakarta.annotation-api\1.3.5\jakarta.annotation-api-1.3.5.jar;F:\dev\maven\MavenRepository\org\yaml\snakeyaml\1.26\snakeyaml-1.26.jar;F:\dev\maven\MavenRepository\org\springframework\boot\spring-boot-starter-json\2.3.10.RELEASE\spring-boot-starter-json-2.3.10.RELEASE.jar;F:\dev\maven\MavenRepository\com\fasterxml\jackson\datatype\jackson-datatype-jdk8\2.11.4\jackson-datatype-jdk8-2.11.4.jar;F:\dev\maven\MavenRepository\com\fasterxml\jackson\datatype\jackson-datatype-jsr310\2.11.4\jackson-datatype-jsr310-2.11.4.jar;F:\dev\maven\MavenRepository\com\fasterxml\jackson\module\jackson-module-parameter-names\2.11.4\jackson-module-parameter-names-2.11.4.jar;F:\dev\maven\MavenRepository\org\springframework\boot\spring-boot-starter-tomcat\2.3.10.RELEASE\spring-boot-starter-tomcat-2.3.10.RELEASE.jar;F:\dev\maven\MavenRepository\org\apache\tomcat\embed\tomcat-embed-core\9.0.45\tomcat-embed-core-9.0.45.jar;F:\dev\maven\MavenRepository\org\glassfish\jakarta.el\3.0.3\jakarta.el-3.0.3.jar;F:\dev\maven\MavenRepository\org\apache\tomcat\embed\tomcat-embed-websocket\9.0.45\tomcat-embed-websocket-9.0.45.jar;F:\dev\maven\MavenRepository\org\springframework\spring-webmvc\5.2.14.RELEASE\spring-webmvc-5.2.14.RELEASE.jar;F:\dev\maven\MavenRepository\org\springframework\spring-aop\5.2.14.RELEASE\spring-aop-5.2.14.RELEASE.jar;F:\dev\maven\MavenRepository\org\springframework\spring-context\5.2.14.RELEASE\spring-context-5.2.14.RELEASE.jar;F:\dev\maven\MavenRepository\org\springframework\spring-expression\5.2.14.RELEASE\spring-expression-5.2.14.RELEASE.jar;F:\dev\maven\MavenRepository\org\projectlombok\lombok\1.18.20\lombok-1.18.20.jar;F:\dev\maven\MavenRepository\org\objenesis\objenesis\2.6\objenesis-2.6.jar;F:\dev\maven\MavenRepository\org\springframework\spring-core\5.2.14.RELEASE\spring-core-5.2.14.RELEASE.jar;F:\dev\maven\MavenRepository\org\springframework\spring-jcl\5.2.14.RELEASE\spring-jcl-5.2.14.RELEASE.jar;C:\Program Files\JetBrains\IntelliJ IDEA 2022.2\lib\idea_rt.jar" com.iiotos.api.GetDevice
Connected to the target VM, address: '127.0.0.1:6435', transport: 'socket'
09:59:41.990 [main] WARN org.springframework.http.converter.json.Jackson2ObjectMapperBuilder - For Jackson Kotlin classes support please add "com.fasterxml.jackson.module:jackson-module-kotlin" to the classpath
09:59:42.799 [main] DEBUG org.springframework.web.client.RestTemplate - HTTP POST http://192.168.202.188:8080/api/auth/login
09:59:42.811 [main] DEBUG org.springframework.web.client.RestTemplate - Accept=[application/json, application/cbor, application/*+json]
09:59:42.828 [main] DEBUG org.springframework.web.client.RestTemplate - Writing [{password=customer, username=customer@thingsboard.org}] with org.springframework.http.converter.json.MappingJackson2HttpMessageConverter
09:59:42.948 [main] DEBUG org.springframework.web.client.RestTemplate - Response 200 OK
09:59:42.956 [main] DEBUG org.springframework.web.client.RestTemplate - Reading to [com.fasterxml.jackson.databind.JsonNode]
09:59:44.593 [main] DEBUG org.springframework.web.client.RestTemplate - HTTP GET http://192.168.202.188:8080/api/auth/user
09:59:44.751 [main] DEBUG org.springframework.web.client.RestTemplate - Accept=[application/json, application/cbor, application/*+json]
09:59:44.770 [main] DEBUG org.springframework.web.client.RestTemplate - Response 200 OK
09:59:44.770 [main] DEBUG org.springframework.web.client.RestTemplate - Reading to [org.thingsboard.server.common.data.User]
09:59:45.451 [main] DEBUG org.springframework.web.client.RestTemplate - HTTP GET http://192.168.202.188:8080/api/customer/44f83df0-3768-11ee-8f31-b92b1a081d15/devices?type=&pageSize=10&page=0
09:59:45.527 [main] DEBUG org.springframework.web.client.RestTemplate - Accept=[application/json, application/cbor, application/*+json]
09:59:45.540 [main] DEBUG org.springframework.web.client.RestTemplate - Response 200 OK
09:59:45.541 [main] DEBUG org.springframework.web.client.RestTemplate - Reading to [org.thingsboard.server.common.data.page.PageData<org.thingsboard.server.common.data.Device>]
Device [tenantId=44a9a8c0-3768-11ee-8f31-b92b1a081d15, customerId=44f83df0-3768-11ee-8f31-b92b1a081d15, name=Test Device A1, type=default, label=null, deviceProfileId=44b31ea0-3768-11ee-8f31-b92b1a081d15, deviceData=null, firmwareId=DeviceData(configuration=DefaultDeviceConfiguration(), transportConfiguration=DefaultDeviceTransportConfiguration()), additionalInfo=null, createdTime=1691663147138, id=45271620-3768-11ee-8f31-b92b1a081d15]
Device [tenantId=44a9a8c0-3768-11ee-8f31-b92b1a081d15, customerId=44f83df0-3768-11ee-8f31-b92b1a081d15, name=Test Device A2, type=default, label=null, deviceProfileId=44b31ea0-3768-11ee-8f31-b92b1a081d15, deviceData=null, firmwareId=DeviceData(configuration=DefaultDeviceConfiguration(), transportConfiguration=DefaultDeviceTransportConfiguration()), additionalInfo=null, createdTime=1691663147194, id=452fa1a0-3768-11ee-8f31-b92b1a081d15]
Device [tenantId=44a9a8c0-3768-11ee-8f31-b92b1a081d15, customerId=44f83df0-3768-11ee-8f31-b92b1a081d15, name=Test Device A3, type=default, label=null, deviceProfileId=44b31ea0-3768-11ee-8f31-b92b1a081d15, deviceData=null, firmwareId=DeviceData(configuration=DefaultDeviceConfiguration(), transportConfiguration=DefaultDeviceTransportConfiguration()), additionalInfo=null, createdTime=1691663147204, id=45312840-3768-11ee-8f31-b92b1a081d15]
10:00:01.094 [main] DEBUG org.springframework.web.client.RestTemplate - HTTP POST http://192.168.202.188:8080/api/auth/logout
10:00:01.117 [main] DEBUG org.springframework.web.client.RestTemplate - Response 200 OK
Disconnected from the target VM, address: '127.0.0.1:6435', transport: 'socket'

Process finished with exit code 0
```



```shell
09:59:45.541 [main] DEBUG org.springframework.web.client.RestTemplate - Reading to [org.thingsboard.server.common.data.page.PageData<org.thingsboard.server.common.data.Device>]
Device [tenantId=44a9a8c0-3768-11ee-8f31-b92b1a081d15, customerId=44f83df0-3768-11ee-8f31-b92b1a081d15, name=Test Device A1, type=default, label=null, deviceProfileId=44b31ea0-3768-11ee-8f31-b92b1a081d15, deviceData=null, firmwareId=DeviceData(configuration=DefaultDeviceConfiguration(), transportConfiguration=DefaultDeviceTransportConfiguration()), additionalInfo=null, createdTime=1691663147138, id=45271620-3768-11ee-8f31-b92b1a081d15]

Device [tenantId=44a9a8c0-3768-11ee-8f31-b92b1a081d15, customerId=44f83df0-3768-11ee-8f31-b92b1a081d15, name=Test Device A2, type=default, label=null, deviceProfileId=44b31ea0-3768-11ee-8f31-b92b1a081d15, deviceData=null, firmwareId=DeviceData(configuration=DefaultDeviceConfiguration(), transportConfiguration=DefaultDeviceTransportConfiguration()), additionalInfo=null, createdTime=1691663147194, id=452fa1a0-3768-11ee-8f31-b92b1a081d15]

Device [tenantId=44a9a8c0-3768-11ee-8f31-b92b1a081d15, customerId=44f83df0-3768-11ee-8f31-b92b1a081d15, name=Test Device A3, type=default, label=null, deviceProfileId=44b31ea0-3768-11ee-8f31-b92b1a081d15, deviceData=null, firmwareId=DeviceData(configuration=DefaultDeviceConfiguration(), transportConfiguration=DefaultDeviceTransportConfiguration()), additionalInfo=null, createdTime=1691663147204, id=45312840-3768-11ee-8f31-b92b1a081d15]
```



