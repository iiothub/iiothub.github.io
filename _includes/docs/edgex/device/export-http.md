* TOC
{:toc}



## 一、概述



### 1.安装说明

**安装方式：**

- 部署虚拟设备服务 ds-virtual 模拟设备
- EdgeX 导出数据到 HTTP 服务
- 使用 Spring Boot 开发 HTTP 服务，接收导出数据



### 2.HTTP 服务

Spring Boot 程序

```java
@RestController
public class EdgexController {

    @RequestMapping("/income")
    public void edgexCallback(@RequestBody HashMap hashMap) {

        System.out.println(hashMap);

    }
}
```



```shell
# 导出数据接口
http://192.168.3.4:8888/income
```

![](/images/edgex/device/export-http/export-1.png)





## 二、安装部署



### 1.docker-comepse

```shell
# 1.克隆 edgex-compose
$ git clone git@github.com:edgexfoundry/edgex-compose.git 
$ git clone https://github.com/edgexfoundry/edgex-compose.git
$ cd edgex-compose 
$ git checkout v3.1


# 2.生成 docker-compose.yml 文件（注意这包括 mqtt-broker）
$ cd compose-builder
$ make gen ds-virtual asc-http no-secty


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

[root@edgex compose-builder]# make gen ds-virtual asc-http no-secty
echo MQTT_VERBOSE=
MQTT_VERBOSE=
docker compose  -p edgex -f docker-compose-base.yml -f add-device-virtual.yml -f add-asc-http-export.yml convert > docker-compose.yml
rm -rf ./gen_ext_compose


[root@edgex compose-builder]# ls | grep 'docker-compose.yml'
docker-compose.yml
```



```shell
[root@edgex compose-builder]# vim docker-compose.yml 

name: edgex
services:
  app-http-export:
    container_name: edgex-app-http-export
    depends_on:
      consul:
        condition: service_started
        required: true
      core-data:
        condition: service_started
        required: true
    environment:
      EDGEX_PROFILE: http-export
      EDGEX_SECURITY_SECRET_STORE: "false"
      SERVICE_HOST: edgex-app-http-export
      WRITABLE_LOGLEVEL: INFO
      WRITABLE_PIPELINE_FUNCTIONS_HTTPEXPORT_PARAMETERS_URL: http://EXPORT_HOST_PLACE_HOLDER:7770
    hostname: edgex-app-http-export
    image: edgexfoundry/app-service-configurable:3.1.0
    networks:
      edgex-network: null
    ports:
      - mode: ingress
        host_ip: 127.0.0.1
        target: 59704
        published: "59704"
        protocol: tcp
    read_only: true
    restart: always
    security_opt:
      - no-new-privileges:true
    user: 2002:2001
    volumes:
      - type: bind
        source: /etc/localtime
        target: /etc/localtime
        read_only: true
        bind:
          create_host_path: true

......
```



### 2.修改配置

```shell
# 修改配置
注意WRITABLE_PIPELINE_FUNCTIONS_HTTPEXPORT_PARAMETERS_URL: http://192.168.10.131:8080/income这里配置了我们的导出端点，即http://192.168.10.131:8080/income,这里是我自定义的一个springboot应用


WRITABLE_PIPELINE_FUNCTIONS_HTTPEXPORT_PARAMETERS_URL: http://192.168.3.4:8888/income


[root@edgex compose-builder]# vim docker-compose.yml 

name: edgex
services:
  app-http-export:
    container_name: edgex-app-http-export
......

    environment:
      EDGEX_PROFILE: http-export
      EDGEX_SECURITY_SECRET_STORE: "false"
      SERVICE_HOST: edgex-app-http-export
      WRITABLE_LOGLEVEL: INFO
      WRITABLE_PIPELINE_FUNCTIONS_HTTPEXPORT_PARAMETERS_URL: http://192.168.3.4:8888/income
```



### 3.启动 EdgeX Foundry

使用以下命令部署 EdgeX：

```shell
$ cd edgex-compose/compose-builder
$ docker compose pull
$ docker compose up -d


# 修改配置文件
替换IP地址 127.0.0.1 为 0.0.0.0


# 修改 http接口
WRITABLE_PIPELINE_FUNCTIONS_HTTPEXPORT_PARAMETERS_URL: http://192.168.3.4:8888/income
```

```shell
# docker compose pull

# docker compose up -d
```



![](/images/edgex/device/export-http/export-2.png)

![](/images/edgex/device/export-http/export-3.png)



### 4.访问 UI

#### 4.1. consul

```shell
# 访问地址
http://192.168.202.233:8500
```

![](/images/edgex/device/export-http/export-4.png)



#### 4.2. EdgeX Console

```shell
# 访问地址
http://192.168.202.233:4000/
```



![](/images/edgex/device/export-http/export-5.png)

![](/images/edgex/device/export-http/export-6.png)

![](/images/edgex/device/export-http/export-7.png)



### 5.测试

![](/images/edgex/device/export-http/export-8.png)



```shell
{
	profileName = Random - Boolean - Device, apiVersion = v3, readings = [{
		id = 638 d5e03 - 5 f65 - 4007 - 95 f6 - 856986e cf3b0,
		origin = 1708432044141665586,
		deviceName = Random - Boolean - Device,
		resourceName = Bool,
		profileName = Random - Boolean - Device,
		valueType = Bool,
		value = true
	}], origin = 1708432044141665586, id = 5 c05ac22 - 59e6 - 41e e - 8 f9b - c3973b0e17d6, sourceName = Bool, deviceName = Random - Boolean - Device
} 


{
	profileName = Random - Float - Device, apiVersion = v3, readings = [{
		id = 33e07 d1d - dc31 - 4e5 f - b4ca - 4 d2505da81cf,
		origin = 1708432044120816042,
		deviceName = Random - Float - Device,
		resourceName = Float64,
		profileName = Random - Float - Device,
		valueType = Float64,
		value = -1.669694e+308
	}], origin = 1708432044120816042, id = bc40b33a - 9270 - 40 a2 - 90 cb - 510995 d92a02, sourceName = Float64, deviceName = Random - Float - Device
} 


{
	profileName = Random - Integer - Device, apiVersion = v3, readings = [{
		id = 450 d2e8d - 057 d - 4e85 - a09e - d9165d97473c,
		origin = 1708432044127607940,
		deviceName = Random - Integer - Device,
		resourceName = Int8,
		profileName = Random - Integer - Device,
		valueType = Int8,
		value = 93
	}], origin = 1708432044127607940, id = 292 b0bd7 - 8e a1 - 4 aa4 - a903 - 462 df4361169, sourceName = Int8, deviceName = Random - Integer - Device
} 


{
	profileName = Random - UnsignedInteger - Device, apiVersion = v3, readings = [{
		id = 403e305 c - 300 d - 468 a - a057 - edc97a0b3cdf,
		origin = 1708432044131510252,
		deviceName = Random - UnsignedInteger - Device,
		resourceName = Uint8,
		profileName = Random - UnsignedInteger - Device,
		valueType = Uint8,
		value = 223
	}], origin = 1708432044131510252, id = f8ed3c1b - cdb6 - 4189 - bf76 - 3 b3d209bffd0, sourceName = Uint8, deviceName = Random - UnsignedInteger - Device
} 

......
```



