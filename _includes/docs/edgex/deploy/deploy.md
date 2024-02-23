* TOC
{:toc}



## 一、概述



### 1.官方文档

```shell
# Quick Start
https://docs.edgexfoundry.org/3.1/getting-started/quick-start/

# edgexfoundry/edgex-compose
https://github.com/edgexfoundry/edgex-compose

# Edgex Docker Compose Builder
https://github.com/edgexfoundry/edgex-compose/tree/main/compose-builder
```



### 2.Docker Compose 生成器

**生成 docker-compose.yml 选项**

```shell
gen [options]
Generates temporary single file compose file (`docker-compose.yml`) as specified by:

Options:
    no-secty:         Generates non-secure compose,
                      otherwise generates secure compose file
    arm64:            Generates compose file using ARM64 images
    dev:              Generates using local built images from edgex-go repo
                      'make docker' creates local docker images 
                      tagged with '0.0.0-dev'
    app-dev:          Generates using local built images from application 
                      service repos
                      'make docker' creates local docker images 
                      tagged with '0.0.0-dev`
    device-dev:       Generates using local built images from device service repos
                      'make docker' creates local docker images 
                      tagged with '0.0.0-dev'
    ui-dev:           Generates using local built images from edgex-ui-go repo
                      'make docker' creates local docker image tagged with '0.0.0-dev'
    delayed-start:    Generates compose file with delayed start services- spire 
                      related services and spiffe-token-provider service included
    ds-modbus:        Generates compose file with device-modbus included
    ds-bacnet-ip:     Generates compose file with device-bacnet-ip included
    ds-bacnet-mstp:   Generates compose file with device-bacnet-mstp included
    ds-onvif-camera:  Generates compose file with device-onvif-camera included
    ds-usb-camera:    Generates compose file with device-usb-camera included
    ds-mqtt:          Generates compose file with device-mqtt included
    ds-rest:          Generates compose file with device-rest included
    ds-snmp:          Generates compose file with device-snmp included
    ds-virtual:       Generates compose file with device-virtual included
    ds-coap:          Generates compose file with device-coap included
    ds-gpio:          Generates compose file with device-gpio included
    ds-uart:          Generates compose file with device-uart included
    ds-llrp:          Generates compose file with device-rfid-llrp included
    modbus-sim:       Generates compose file with ModBus simulator included
    asc-http:         Generates compose file with App Service HTTP Export included
    asc-mqtt:         Generates compose file with App Service MQTT Export included
    asc-metrics:      Generates compose file with App Service 
                      Metrics InfluxDb included
    asc-sample:       Generates compose file with App Service Sample included
    as-llrp:          Generates compose file with App RFID LLRP Inventory included
    as-record-replay: Generates compose file with App Record & Replay included
    asc-ex-mqtt:      Generates compose file with App Service 
                      External MQTT Trigger included
    mqtt-broker:      Generates compose file with a MQTT Broker service included
    mqtt-bus:         Generates compose file with services configured 
                      for MQTT Message Bus
                      The MQTT Broker service is also included.
    mqtt-verbose      Enables MQTT Broker verbose logging.
    nanomq:           ** Experimental ** 
                      Generates compose file with NonoMQ MQTT broker 
                      when mqtt-broker or mqtt-bus are specified
                      Not valid in secure mode when uses with mqtt-bus
    nats-bus:         Generates compose file with services 
                      configured for NAT Message Bus
                      The NATS Server service is also included.
```

![](/images/edgex/deploy/deploy/deploy-1.png)



**生成 docker-compose.yml 说明：**

- no secty：生成非安全的 docker-compose.yml 文件，否则生成安全的合成文件
- arm64:      生成 docker-compose.yml 文件用 ARM64 容器镜像
- ds-mqtt:    生成 docker-compose.yml 文件包含 MQTT 设备服务
- ds-modbus:    生成 docker-compose.yml 文件包含 Modbus 设备服务
- ds-rest:    生成 docker-compose.yml 文件包含 REST 设备服务
- ds-virtual:    生成 docker-compose.yml 文件包含虚拟设备服务
- mqtt-broker:    生成 docker-compose.yml 文件包含 mqtt-broker 设备服务
- asc-mqtt:    生成 docker-compose.yml 文件包含 MQTT 导出数据服务
- asc-http:    生成 docker-compose.yml 文件包含 HTTP 导出数据服务
- ......



### 3.创建  docker-compose 文件

```shell
# 1.克隆 edgex-compose
$ git clone https://github.com/edgexfoundry/edgex-compose.git
$ cd edgex-compose 
# 切换分支
$ git checkout v3.1


# 2.生成 docker-compose.yml 文件
$ cd compose-builder
$ make gen ds-virtual no-secty


# 3.检查生成的文件
$ ls | grep 'docker-compose.yml'
docker-compose.yml
```





## 二、安装准备



### 1. 克隆服务器

```shell
# 克隆机器

# 修改IP地址
cd /etc/sysconfig/network-scripts
vim ifcfg-ens33
192.168.202.233

# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

# 设置主机名
hostnamectl set-hostname edgex
```



### 2.安装 Docker

安装版本19.03.*

```shell
$ wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo

$ yum -y install docker-ce

$ systemctl enable docker && systemctl start docker

$ docker --version
```

- **添加阿里云加速镜像**

```shell
# 添加阿里云加速镜像

cat > /etc/docker/daemon.json << EOF
{
   "registry-mirrors": ["https://gcctk8ld.mirror.aliyuncs.com"],
   "exec-opts": ["native.cgroupdriver=cgroupfs"]
} 
EOF
```

- **重启docker**

```shell
#重启docker
systemctl restart docker
```



### 3.安装 docker-compose

**安装 docker-compose**

```shell
#下载源码
curl -L https://github.com/docker/compose/releases/download/1.25.0/docker-compose-Linux-x86_64 -o /usr/local/bin/docker-compose


#给docker-compose添加执行权限
sudo chmod +x /usr/local/bin/docker-compose

#查看docker-compose是否安装成功
docker-compose -version
```



**docker-compose 基本操作**

```shell
# 安装并启动EdgeX
sudo docker-compose up -d     # -d 后台运行容器
 
# 查看所有容器运行状况
sudo docker-compose ps
 
# 显示容器日志
docker-compose logs -f [compose-contatainer-name]
 
# 停止容器
sudo docker-compose stop
 
# 启动容器
sudo docker-compose start
 
# 停止和删除所有容器
sudo docker-compose down


# 常用命令：
启动：docker-compose up -d 注意这里需要在yml配置文件路径执行，其他路径执行需要-f指定配置文件地址。
查看日志：docker-compose logs -f ${compose-contatainer-name}
停止：docker-compose stop
停止并删除容器：docker-compose down
其他命令帮助：docker-compose --help
```





## 三、非安全模式部署



### 1.docker-comepse

```shell
# 1.克隆 edgex-compose
$ git clone git@github.com:edgexfoundry/edgex-compose.git 
$ git clone https://github.com/edgexfoundry/edgex-compose.git
$ cd edgex-compose 
$ git checkout v3.1


# 2.生成 docker-compose.yml 文件（注意这包括 mqtt-broker）
$ cd compose-builder
$ make gen ds-virtual no-secty


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

[root@edgex compose-builder]# make gen ds-virtual no-secty
echo MQTT_VERBOSE=
MQTT_VERBOSE=
docker compose  -p edgex -f docker-compose-base.yml -f add-device-virtual.yml convert > docker-compose.yml
rm -rf ./gen_ext_compose


[root@edgex compose-builder]# ls | grep 'docker-compose.yml'
docker-compose.yml
```



### 2.启动 EdgeX Foundry

使用以下命令部署 EdgeX：

```shell
$ cd edgex-compose/compose-builder
$ docker compose pull
$ docker compose up -d


# 修改配置文件
替换IP地址 127.0.0.1 为 0.0.0.0
```

```shell
# docker compose pull

# docker compose up -d
```



![](/images/edgex/deploy/deploy/deploy-2.png)

![](/images/edgex/deploy/deploy/deploy-3.png)

![](/images/edgex/deploy/deploy/deploy-4.png)



### 3.访问 UI

#### 3.1. consul

```shell
# 访问地址
http://192.168.202.233:8500
```

![](/images/edgex/deploy/deploy/deploy-5.png)



#### 3.2. EdgeX Console

```shell
# 访问地址
http://192.168.202.233:4000/
```



![](/images/edgex/deploy/deploy/deploy-6.png)

![](/images/edgex/deploy/deploy/deploy-7.png)

![](/images/edgex/deploy/deploy/deploy-8.png)



