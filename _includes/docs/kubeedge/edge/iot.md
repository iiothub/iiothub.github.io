* TOC
{:toc}


## 一、物联网云边协同



### 1.IoT云边协同设计

物联网平台（IoT）部署在云上，网关部署在边缘节点，设备通过边缘网关接入物联网平台。

**云边协同设计：**

- Kubernetes 在云端部署 IoT 平台
- KubeEdge 在边缘节点部署网关
- IoT 设备通过 MQTT 协议接入边缘网关



### 2.物联网平台设计

物联网平台选择  ThingsBoard，边缘网关选择 EMQX、Node-RED，MQTTX 工具模拟终端设备。

 **IoT平台设计：**

- 使用 Kubernetes 在云端部署 ThingsBoard 集群

- 使用 KubeEdge 在边缘节点部署 EMQX、Node-Red，实现边缘网关
- MQTTX 工具模拟终端设备，通过 MQTT 协议接入边缘网关



### 3.物联网平台实现

**物联网平台（IoT）具体实现：**

1. 部署 Kubernetes 集群
2. 部署 KubeEdge 
3. 部署 ThingsBoard 集群
4. 部署 EMQX、Node-RED 边缘网关
5. 配置 Node-RED、MQTTX
6. 测试 IoT 设备向 ThingsBoard 发送遥测数据





## 二、部署环境



### 1.节点配置

| 主机名     | IP地址          | 角色   |
| ---------- | --------------- | ------ |
| k8s-master | 192.168.202.201 | master |
| k8s-node1  | 192.168.202.202 | node   |
| k8s-node2  | 192.168.202.203 | node   |
| edge-1     | 192.168.202.211 | edge   |

![](/images/kubeedge/edge/iot/iot-23.png)



### 2.版本信息

| 信息        | 版本     | 备注                       |
| ----------- | -------- | -------------------------- |
| K8s         | v1.23.12 |                            |
| centos      | 7.8      | # cat  /etc/redhat-release |
| KubeEdge    | v1.13.4  |                            |
| ThingsBoard | v3.5.1   |                            |
| EMQX        | 4.4.17   |                            |
| Node-RED    | latest   |                            |
|             |          |                            |





## 三、IoT云边协同部署



### 1.部署Kubernetes集群

**部署 Kubernetes 集群参考**

```shell
# 部署 Kubernetes 集群

https://iothub.org.cn/docs/kubernetes/pro/deploy-kubernetes/
```



```shell
[root@k8s-master ~]# kubectl get nodes
NAME         STATUS   ROLES                  AGE    VERSION
k8s-master   Ready    control-plane,master   27d    v1.23.12
k8s-node1    Ready    <none>                 27d    v1.23.12
k8s-node2    Ready    <none>                 10d    v1.23.12
```

![](/images/kubeedge/edge/iot/iot-1.png)



### 2.部署KubeEdge

**部署 KubeEdge 参考**

```shell
# 部署 KubeEdge

https://iothub.org.cn/docs/kubeedge/deploy/deploy/
```



```shell
[root@k8s-master ~]# kubectl get nodes
NAME         STATUS   ROLES                  AGE    VERSION
edge-1       Ready    agent,edge             4d4h   v1.23.17-kubeedge-v1.13.4
k8s-master   Ready    control-plane,master   27d    v1.23.12
k8s-node1    Ready    <none>                 27d    v1.23.12
k8s-node2    Ready    <none>                 10d    v1.23.12


[root@k8s-master ~]# kubectl get all -n kubeedge
NAME                               READY   STATUS    RESTARTS        AGE
pod/cloud-iptables-manager-592m5   1/1     Running   3 (3d22h ago)   4d5h
pod/cloud-iptables-manager-pg4pl   1/1     Running   3 (3d22h ago)   4d5h
pod/cloudcore-5959c5986f-8hsc4     1/1     Running   3 (3d22h ago)   4d5h

NAME                TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                           AGE
service/cloudcore   NodePort   10.110.71.216   <none>        10000:30976/TCP,10001:31372/TCP,10002:31922/TCP,10003:30163/TCP,10004:31927/TCP   4d5h

NAME                                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/cloud-iptables-manager   2         2         2       2            2           <none>          4d5h

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cloudcore   1/1     1            1           4d5h

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/cloudcore-5959c5986f   1         1         1       4d5h
```

![](/images/kubeedge/edge/iot/iot-2.png)

![](/images/kubeedge/edge/iot/iot-3.png)



### 3.部署ThingsBoard集群

**部署 ThingsBoard 参考**

```shell
# ThingsBoard 单机部署
https://iothub.org.cn/docs/iot/deploy/deploy-single/

# ThingsBoard 集群部署
https://iothub.org.cn/docs/iot/deploy/deploy-cluster/
```



**备注：考虑测试环境资源有限，部署单机代替ThingsBoard集群**

![](/images/kubeedge/edge/iot/iot-4.png)



![](/images/kubeedge/edge/iot/iot-5.png)



### 4.部署Node-RED边缘网关

#### 4.1.边缘网关功能

1. 边缘网关使用 EMQX、Node-RED 实现 MQTT-Broker 功能
2. 边缘网关通过 Node-RED 与 ThingsBoard 平台通信
3. IoT 设备通过 MQTT 协议与边缘网关 EMQX 通信



#### 4.2.部署EMQX

**gateway-emqx.yaml** 

```shell
[root@k8s-master kubeedge]# vim gateway-emqx.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: gateway-emqx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: emqx
  template:
    metadata:
      labels:
        app: emqx
    spec:
      nodeName: edge-1    #调度到指定机器
      containers:
      - name: emqx
        image: emqx/emqx:4.4.17
        ports:
        - containerPort: 1883
          hostPort: 1884
        - containerPort: 18083
          hostPort: 18084
        - containerPort: 8081
          hostPort: 8082
        - containerPort: 8083
          hostPort: 8085
        - containerPort: 8883
          hostPort: 8884
        - containerPort: 8084
          hostPort: 8086
```



```shell
[root@k8s-master iot]# kubectl apply -f gateway-emqx.yaml 
deployment.apps/gateway-emqx created


[root@k8s-master iot]# kubectl get all
NAME                                READY   STATUS    RESTARTS   AGE
pod/gateway-emqx-6fcb56cb4f-n8z5s   1/1     Running   0          12s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   27d

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/gateway-emqx   1/1     1            1           12s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/gateway-emqx-6fcb56cb4f   1         1         1       12s
```

![](/images/kubeedge/edge/iot/iot-14.png)



**访问EMQX**

```shell
# 访问地址

http://192.168.202.211:18084
账号：admin
初始密码：public
修改密码：1qaz2wsx
```

![](/images/kubeedge/edge/iot/iot-15.png)



```shell
# 参考

$ docker run -d --name emqx -p 18083:18083 -p 1883:1883 emqx/emqx:latest

$ docker run -d --name emqx -p 1883:1883 -p 8081:8081 -p 8083:8083 -p 8883:8883 -p 8084:8084 -p 18083:18083 emqx/emqx:v4.0.0
```



#### 4.2.部署Node-RED

**gateway-node-red.yaml** 

```shell
[root@k8s-master kubeedge]# vim gateway-node-red.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: gateway-node-red
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nodered
  template:
    metadata:
      labels:
        app: nodered
    spec:
      nodeName: edge-1    #调度到指定机器
      hostNetwork: true   # 使用主机网络
      containers:
      - name: nodered
        image: nodered/node-red:latest
```



**部署 Node-RED 到边缘节点**

```shell
[root@k8s-master iot]# kubectl apply -f gateway-node-red.yaml 
deployment.apps/gateway-node-red created


[root@k8s-master iot]# kubectl get all
NAME                           READY   STATUS    RESTARTS   AGE
pod/gateway-5847c4f88c-bqx5j   1/1     Running   0          30s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   27d

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/gateway   1/1     1            1           31s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/gateway-5847c4f88c   1         1         1       30s
```

![](/images/kubeedge/edge/iot/iot-6.png)



**访问 Node-RED**

```shell
# 访问地址

http://192.168.202.211:1880
```

![](/images/kubeedge/edge/iot/iot-7.png)



```shell
# 参考

# docker run --network host --restart=always -v node_red_data:/data --name nodered nodered/node-red:latest
```



### 5.配置边缘网关

#### 5.1.ThingsBaord创建网关

![](/images/kubeedge/edge/iot/iot-8.png)

![](/images/kubeedge/edge/iot/iot-9.png)



#### 5.2.Node-RED上传遥测

将设备遥测发布到ThingsBoard服务器节点，请向以下主题发送publish消息：

```shell
Topic: v1/gateway/telemetry
```



**Message:**

```json
{
  "Device A": [
    {
      "ts": 1483228800000,
      "values": {
        "temperature": 42,
        "humidity": 80
      }
    },
    {
      "ts": 1483228801000,
      "values": {
        "temperature": 43,
        "humidity": 82
      }
    }
  ],
  "Device B": [
    {
      "ts": 1483228800000,
      "values": {
        "temperature": 42,
        "humidity": 80
      }
    }
  ]
}
```



![](/images/kubeedge/edge/iot/iot-10.png)

![](/images/kubeedge/edge/iot/iot-11.png)



![](/images/kubeedge/edge/iot/iot-12.png)

![](/images/kubeedge/edge/iot/iot-13.png)



#### 5.3.Node-RED连接EMQX

Node-RED 连接 EMQX，接收 IoT 设备遥测数据

```shell
Topic: v1/gateway/telemetry
```



![](/images/kubeedge/edge/iot/iot-16.png)

![](/images/kubeedge/edge/iot/iot-17.png)

![](/images/kubeedge/edge/iot/iot-18.png)



#### 5.4.MQTTX连接EMQX

![](/images/kubeedge/edge/iot/iot-25.png)

![](/images/kubeedge/edge/iot/iot-19.png)



### 6.云边协同测试

#### 6.1.测试方法

1. 使用 MQTTX 工具模拟物联网设备，向 EMQX 发送遥测数据
2. Node-RED 通过 EMQX  订阅遥测数据
3. Node-RED 把遥测数据上传到 ThingsBoard



#### 6.2.上传遥测数据

- **通过 MQTTX 工具 上传遥测数据**

```shell
v1/gateway/telemetry


{
  "Device A": [
    {
      "ts": 1483228800000,
      "values": {
        "temperature": 42,
        "humidity": 80
      }
    },
    {
      "ts": 1483228801000,
      "values": {
        "temperature": 43,
        "humidity": 82
      }
    }
  ],
  "Device B": [
    {
      "ts": 1483228800000,
      "values": {
        "temperature": 42,
        "humidity": 80
      }
    }
  ]
}
```



![](/images/kubeedge/edge/iot/iot-24.png)



- **ThingsBoard**

![](/images/kubeedge/edge/iot/iot-20.png)

![](/images/kubeedge/edge/iot/iot-21.png)



- **Node-RED**

![](/images/kubeedge/edge/iot/iot-22.png)



