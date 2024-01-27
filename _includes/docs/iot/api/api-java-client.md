* TOC
{:toc}



## 一、概述



### 1.工程配置

**pom.xml**

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
    <artifactId>tb-client</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>tb-client</name>
    <description>tb-client</description>

    <properties>
        <java.version>11</java.version>
    </properties>

    <repositories>
        <repository>
            <id>thingsboard</id>
            <url>https://repo.thingsboard.io/artifactory/libs-release-public</url>
        </repository>
    </repositories>

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





## 二、程序测试



### 1.客户端登录

```java
public class Login {

    public static void main(String[] args) {
        // ThingsBoard REST API URL
        String url = "http://192.168.202.188:8080";

        // Default Tenant Administrator credentials
        //String username = "tenant@thingsboard.org";
        //String password = "tenant";
        
        String username = "iiotos@thingsboard.org";
        String password = "iiotos";

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

![](/images/iot/api/api-java-client/client-1.png)



### 2.租户设备

```java
public class TenantDevice {

    public static void main(String[] args) {

        // ThingsBoard REST API URL
        String url = "http://192.168.202.188:8080";

        // Default Tenant Administrator credentials
        String username = "tenant@thingsboard.org";
        String password = "tenant";
        //String username = "iiotos@thingsboard.org";
        //String password = "iiotos";

        // Creating new rest client and auth with credentials
        RestClient client = new RestClient(url);
        client.login(username, password);

        PageData<Device> tenantDevices;
        PageLink pageLink = new PageLink(10);
        do {
            // Fetch all tenant devices using current page link and print each of them
            tenantDevices = client.getTenantDevices("", pageLink);
            tenantDevices.getData().forEach(System.out::println);
            pageLink = pageLink.nextPageLink();
        } while (tenantDevices.hasNext());

        // Perform logout of current user and close the client
        client.logout();
        client.close();
    }
}
```

![](/images/iot/api/api-java-client/client-2.png)



### 3.租户仪表板

```java
public class TenantDashboard {

    public static void main(String[] args) {

        // ThingsBoard REST API URL
        String url = "http://192.168.202.188:8080";

        // Default Tenant Administrator credentials
        String username = "tenant@thingsboard.org";
        String password = "tenant";
        //String username = "iiotos@thingsboard.org";
        //String password = "iiotos";

        // Creating new rest client and auth with credentials
        RestClient client = new RestClient(url);
        client.login(username, password);

        PageData<DashboardInfo> pageData;
        PageLink pageLink = new PageLink(10);
        do {
            // Fetch all tenant dashboards using current page link and print each of them
            pageData = client.getTenantDashboards(pageLink);
            pageData.getData().forEach(System.out::println);
            pageLink = pageLink.nextPageLink();
        } while (pageData.hasNext());

        // Perform logout of current user and close the client
        client.logout();
        client.close();
    }
}
```

![](/images/iot/api/api-java-client/client-3.png)



### 4.客户设备

```java
public class CustomerDevice {

    public static void main(String[] args) {

        // ThingsBoard REST API URL
        String url = "http://192.168.202.188:8080";
        // Perform login with default Customer User credentials
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

![](/images/iot/api/api-java-client/client-4.png)



### 5.实体计数

使用实体数据查询API对实体进行计数

下面的示例代码显示了如何使用实体数据查询API来计算设备总数、活动设备总数。

```java
public class CountEntities {

    public static void main(String[] args) {

        // ThingsBoard REST API URL
        String url = "http://192.168.202.188:8080";

        // Perform login with default Customer User credentials
        String username = "tenant@thingsboard.org";
        String password = "tenant";
        //String username = "iiotos@thingsboard.org";
        //String password = "iiotos";

        RestClient client = new RestClient(url);
        client.login(username, password);

        // Create entity filter to get all devices
        EntityTypeFilter typeFilter = new EntityTypeFilter();
        typeFilter.setEntityType(EntityType.DEVICE);

        // Create entity count query with provided filter
        EntityCountQuery totalDevicesQuery = new EntityCountQuery(typeFilter);

        // Execute entity count query and get total devices count
        Long totalDevicesCount = client.countEntitiesByQuery(totalDevicesQuery);
        System.out.println("Total devices by the first query: " + totalDevicesCount);

        // Set key filter to existing query to get only active devices
        KeyFilter keyFilter = new KeyFilter();
        keyFilter.setKey(new EntityKey(EntityKeyType.ATTRIBUTE, "active"));
        keyFilter.setValueType(EntityKeyValueType.BOOLEAN);

        BooleanFilterPredicate filterPredicate = new BooleanFilterPredicate();
        filterPredicate.setOperation(BooleanFilterPredicate.BooleanOperation.EQUAL);
        filterPredicate.setValue(new FilterPredicateValue<>(true));

        keyFilter.setPredicate(filterPredicate);

        // Create entity count query with provided filter
        EntityCountQuery totalActiveDevicesQuery =
                new EntityCountQuery(typeFilter, List.of(keyFilter));

        // Execute active devices query and print total devices count
        Long totalActiveDevicesCount = client.countEntitiesByQuery(totalActiveDevicesQuery);
        System.out.println("Total devices by the second query: " + totalActiveDevicesCount);

        // Perform logout of current user and close the client
        client.logout();
        client.close();
    }
}
```

![](/images/iot/api/api-java-client/client-5.png)



### 6.查询实体

使用实体数据查询API查询实体

以下示例代码显示如何使用实体数据查询API获取所有活动设备。

```java
public class QueryEntities {

    public static void main(String[] args) {

        // ThingsBoard REST API URL
        String url = "http://192.168.202.188:8080";

        // Perform login with default Customer User credentials
        String username = "tenant@thingsboard.org";
        String password = "tenant";
        RestClient client = new RestClient(url);
        client.login(username, password);

        // Create entity filter to get only devices
        EntityTypeFilter typeFilter = new EntityTypeFilter();
        typeFilter.setEntityType(EntityType.DEVICE);

        // Create key filter to query only active devices
        KeyFilter keyFilter = new KeyFilter();
        keyFilter.setKey(new EntityKey(EntityKeyType.ATTRIBUTE, "active"));
        keyFilter.setValueType(EntityKeyValueType.BOOLEAN);

        BooleanFilterPredicate filterPredicate = new BooleanFilterPredicate();
        filterPredicate.setOperation(BooleanFilterPredicate.BooleanOperation.EQUAL);
        filterPredicate.setValue(new FilterPredicateValue<>(true));

        keyFilter.setPredicate(filterPredicate);

        // Prepare list of queried device fields
        List<EntityKey> fields = List.of(
                new EntityKey(EntityKeyType.ENTITY_FIELD, "name"),
                new EntityKey(EntityKeyType.ENTITY_FIELD, "type"),
                new EntityKey(EntityKeyType.ENTITY_FIELD, "createdTime")
        );

        // Prepare list of queried device attributes
        List<EntityKey> attributes = List.of(
                new EntityKey(EntityKeyType.ATTRIBUTE, "active")
        );

        // Prepare page link
        EntityDataSortOrder sortOrder = new EntityDataSortOrder();
        sortOrder.setKey(new EntityKey(EntityKeyType.ENTITY_FIELD, "createdTime"));
        sortOrder.setDirection(EntityDataSortOrder.Direction.DESC);
        EntityDataPageLink entityDataPageLink = new EntityDataPageLink(10, 0, "", sortOrder);

        // Create entity query with provided entity filter, key filter, queried fields and page link
        EntityDataQuery dataQuery =
                new EntityDataQuery(typeFilter, entityDataPageLink, fields, attributes, List.of(keyFilter));

        PageData<EntityData> entityPageData;
        do {
            // Fetch active devices using entities query and print them
            entityPageData = client.findEntityDataByQuery(dataQuery);
            entityPageData.getData().forEach(System.out::println);
            dataQuery = dataQuery.next();
        } while (entityPageData.hasNext());

        // Perform logout of current user and close client
        client.logout();
        client.close();
    }
}
```

![](/images/iot/api/api-java-client/client-6.png)



### 7.管理设备

管理设备示例

以下示例代码演示了设备管理API的基本概念（添加/获取/删除设备，获取/保存设备属性）。

```java
public class ManageDevice {

    public static void main(String[] args) {

        // ThingsBoard REST API URL
        String url = "http://192.168.202.188:8080";

        // Perform login with default Customer User credentials
        //String username = "tenantg@thingsboard.org";
        //String password = "tenant";
        String username = "iiotos@thingsboard.org";
        String password = "iiotos";

        RestClient client = new RestClient(url);
        client.login(username, password);

        // Construct device object
        String newDeviceName = "Test Device";
        Device newDevice = new Device();
        newDevice.setName(newDeviceName);

        // Create Json Object Node and set it as additional info
        ObjectMapper mapper = new ObjectMapper();
        ObjectNode additionalInfoNode = mapper.createObjectNode().put("description", "My brand new device");
        newDevice.setAdditionalInfo(additionalInfoNode);

        // Save device
        Device savedDevice = client.saveDevice(newDevice);
        System.out.println("Saved device: " + savedDevice);

        // Find device by device id or throw an exception otherwise
        Optional<DeviceInfo> optionalDevice = client.getDeviceInfoById(savedDevice.getId());
        DeviceInfo foundDevice = optionalDevice
                .orElseThrow(() -> new IllegalArgumentException("Device with id " + newDevice.getId().getId() + " hasn't been found"));

        // Save device shared attributes
        ObjectNode requestNode = mapper.createObjectNode().put("temperature", 22.4).put("humidity", 57.4);
        boolean isSuccessful = client.saveEntityAttributesV2(foundDevice.getId(), "SHARED_SCOPE", requestNode);
        System.out.println("Attributes have been successfully saved: " + isSuccessful);

        // Get device shared attributes
        var attributes = client.getAttributesByScope(foundDevice.getId(), "SHARED_SCOPE", List.of("temperature", "humidity"));
        System.out.println("Found attributes: ");
        attributes.forEach(System.out::println);

        // Delete the device
        client.deleteDevice(savedDevice.getId());

        // Perform logout of current user and close client
        client.logout();
        client.close();
    }
}
```

![](/images/iot/api/api-java-client/client-7.png)



### 8.其他实例

```shell
public class RestClientExample {
    public static void main(String[] args) {
        // ThingsBoard REST API URL
        String url = "http://localhost:8080";

        // Default Tenant Administrator credentials
        String username = "tenant@thingsboard.org";
        String password = "tenant";

        // Creating new rest client and auth with credentials
        RestClient client = new RestClient(url);
        client.login(username, password);

        // Creating an Asset
        Asset asset = new Asset();
        asset.setName("Building 1");
        asset.setType("building");
        asset = client.saveAsset(asset);

        // creating a Device
        Device device = new Device();
        device.setName("Thermometer 1");
        device.setType("thermometer");
        device = client.saveDevice(device);

        // creating relations from device to asset
        EntityRelation relation = new EntityRelation();
        relation.setFrom(asset.getId());
        relation.setTo(device.getId());
        relation.setType("Contains");
        client.saveRelation(relation);
    }
}
```



