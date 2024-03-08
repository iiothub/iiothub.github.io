* TOC
{:toc}



## 一、物联网平台设计



### 1.物联网平台设计

**IoT 平台总体设计:**

- 物联网平台（IoT）部署在云上，物联网设备通过边缘计算节点接入物联网平台
- 物联网平台采用云原生设计，应用容器化部署，在云端实现容器的调度编排
- 物理网平台实现云边协同，云作为控制平面，边作为计算平台
- 云端物联网平台选择开源物联网平台 ThingsBoard 社区版
- 边缘计算节点选择 ThingsBoard Edge 接入物联网设备

![](/images/iot/tb-edge/edge-iot/iot-2.png)



**IoT 云边协同设计:**

- Kubernetes 在云端部署 ThingsBoard 平台
- KubeEdge 在边缘节点部署 ThingsBoard Edge
- IoT 设备通过 MQTT 协议接入边缘计算节点
- 通过 Kubernetes 、KubeEdge 实现云边协同



### 2.物联网平台实现

**物联网平台（IoT）具体实现：**

1. 部署 Kubernetes 集群
2. 部署 KubeEdge 
3. 部署 ThingsBoard 集群
4. 部署 ThingsBoard Edge 边缘节点
5. IoT 设备通过 MQTT 链接 ThingsBoard Edge
6. MQTTX 工具模拟物联网设备

![](/images/iot/tb-edge/edge-iot/iot-3.png)





## 二、部署环境



### 1.节点配置

| 主机名     | IP地址          | 角色   |
| ---------- | --------------- | ------ |
| k8s-master | 192.168.202.201 | master |
| k8s-node1  | 192.168.202.202 | node   |
| k8s-node2  | 192.168.202.203 | node   |
| edge-1     | 192.168.202.211 | edge   |

![](/images/iot/tb-edge/edge-iot/iot-1.png)



### 2.版本信息

| 信息             | 版本      | 备注                       |
| ---------------- | --------- | -------------------------- |
| K8s              | v1.23.12  |                            |
| centos           | 7.8       | # cat  /etc/redhat-release |
| KubeEdge         | v1.13.4   |                            |
| ThingsBoard      | v3.5.1    |                            |
| ThingsBoard Edge | 3.5.1EDGE |                            |
|                  |           |                            |





## 三、物联网平台部署



### 1.部署 Kubernetes 集群

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

![](/images/iot/tb-edge/edge-iot/iot-4.png)



### 2.部署 KubeEdge

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

![](/images/iot/tb-edge/edge-iot/iot-5.png)

![](/images/iot/tb-edge/edge-iot/iot-6.png)



### 3.部署 ThingsBoard 集群

**部署 ThingsBoard 参考**

```shell
# ThingsBoard 单机部署
https://iothub.org.cn/docs/iot/deploy/deploy-single/

# ThingsBoard 集群部署
https://iothub.org.cn/docs/iot/deploy/deploy-cluster/
```



**备注：考虑测试环境资源有限，部署单机代替云端 ThingsBoard 集群**

![](/images/iot/tb-edge/edge-iot/iot-7.png)



![](/images/iot/tb-edge/edge-iot/iot-8.png)



### 4.部署 ThingsBoard Edge

#### 4.1.创建 Edge 实例

在 ThingsBoard 服务端上配置 Edge

![](/images/iot/tb-edge/edge-iot/iot-9.png)

![](/images/iot/tb-edge/edge-iot/iot-10.png)

![](/images/iot/tb-edge/edge-iot/iot-11.png)



```shell
# ThingsBoard 服务器地址
82.157.166.86

# 边缘键
1d2952e8-227e-b019-7eab-d29664b605c1

# 边缘密钥
iihb0i793etbqct62gpf
```



#### 4.2.部署 PostgreSQL

**postgresql.yaml** 

```shell
[root@k8s-master kubeedge]# vim postgresql.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgresql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgresql
  template:
    metadata:
      labels:
        app: postgresql
    spec:
      nodeName: edge-1    #调度到指定机器
      hostNetwork: true   # 使用主机网络
      containers:
      - name: postgresql
        image: postgres:12
        env:
          - name: LANG
            value: "C.UTF-8"
          - name: TZ
            value: "Asia/Shanghai"      
          - name: POSTGRES_DB
            value: "postgres"
          - name: POSTGRES_USER
            value: "postgres"
          - name: POSTGRES_PASSWORD
            value: "postgres"
```



```shell
[root@k8s-master thingsboard]# kubectl apply -f postgresql.yaml 
deployment.apps/postgresql created


[root@k8s-master thingsboard]# kubectl get all
NAME                              READY   STATUS    RESTARTS   AGE
pod/postgresql-867f894fd4-tthq8   1/1     Running   0          31s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   84d

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/postgresql   1/1     1            1           31s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/postgresql-867f894fd4   1         1         1       31s
```



![](/images/iot/tb-edge/edge-iot/iot-12.png)



**访问PostgreSQL**

```shell
# 访问地址

192.168.202.211
5432
postgres/postgres
```

![](/images/iot/tb-edge/edge-iot/iot-13.png)



#### 4.3.创建数据库

在边缘节点创建数据库 tb-edge

![](/images/iot/tb-edge/edge-iot/iot-14.png)



#### 4.4.部署 ThingsBoard Edge

**tb-edge.yaml** 

```shell
[root@k8s-master kubeedge]# vim tb-edge.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: tb-edge
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tb-edge
  template:
    metadata:
      labels:
        app: tb-edge
    spec:
      nodeName: edge-1    #调度到指定机器
      containers:
      - name: tb-edge
        image: thingsboard/tb-edge:3.5.1EDGE
        ports:
        - containerPort: 1883
          hostPort: 11883
        - containerPort: 8080
          hostPort: 18080
        env:
          - name: SPRING_DATASOURCE_URL
            value: "jdbc:postgresql://192.168.202.211:5432/tb-edge"          
          - name: SPRING_DATASOURCE_USERNAME
            value: "postgres"
          - name: SPRING_DATASOURCE_PASSWORD
            value: "postgres"
          - name: CLOUD_ROUTING_KEY
            value: "1d2952e8-227e-b019-7eab-d29664b605c1"     
          - name: CLOUD_ROUTING_SECRET
            value: "iihb0i793etbqct62gpf"     
          - name: CLOUD_RPC_HOST
            value: "82.157.166.86"
          - name: CLOUD_RPC_PORT
            value: "7070"
          - name: CLOUD_RPC_SSL_ENABLED
            value: "false"
```



```shell
[root@k8s-master thingsboard]# kubectl apply -f tb-edge.yaml 
deployment.apps/tb-edge created


[root@k8s-master thingsboard]# kubectl get all
NAME                              READY   STATUS    RESTARTS   AGE
pod/postgresql-867f894fd4-tthq8   1/1     Running   0          14m
pod/tb-edge-67f4b7c8-xbpjg        1/1     Running   0          2m34s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   84d

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/postgresql   1/1     1            1           14m
deployment.apps/tb-edge      1/1     1            1           2m34s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/postgresql-867f894fd4   1         1         1       14m
replicaset.apps/tb-edge-67f4b7c8        1         1         1       2m34s
```



![](/images/iot/tb-edge/edge-iot/iot-15.png)



**访问 ThingsBoard Edge**

```shell
# 访问地址

http://192.168.202.211:18080/login
tenant@thingsboard.org
tenant
```



![](/images/iot/tb-edge/edge-iot/iot-16.png)

![](/images/iot/tb-edge/edge-iot/iot-17.png)

![](/images/iot/tb-edge/edge-iot/iot-18.png)



```shell
#  参考
docker run -d --network host --name tb-edge --restart=always \
-e "SPRING_DATASOURCE_URL=jdbc:postgresql://192.168.202.166:5432/tb-edge" \
-e "SPRING_DATASOURCE_USERNAME=postgres" \
-e "SPRING_DATASOURCE_PASSWORD=postgres" \
-e "CLOUD_ROUTING_KEY=672b5ad6-cf07-c8af-e7cf-ac8a85114902" \
-e "CLOUD_ROUTING_SECRET=tuhk87tqb4l1463revxf" \
-e "CLOUD_RPC_HOST=82.157.166.86" \
-e "CLOUD_RPC_PORT=7070" \
-e "CLOUD_RPC_SSL_ENABLED=false" \
-v ~/.mytb-edge-data:/data \
-v ~/.mytb-edge-logs:/var/log/tb-edge \
thingsboard/tb-edge:3.5.1EDGE
```





## 四、物联网设备接入



### 1.创建设备

在 Edge 端创建设备 iot-device

![](/images/iot/tb-edge/edge-iot/iot-19.png)

![](/images/iot/tb-edge/edge-iot/iot-20.png)



在服务端查看设备

![](/images/iot/tb-edge/edge-iot/iot-21.png)



```shell
# 访问令牌
1ThJ4grl3mXxxw7egHNo
```



### 2.MQTTX 工具

```shell
192.168.202.211
11883
1ThJ4grl3mXxxw7egHNo
```



![](/images/iot/tb-edge/edge-iot/iot-22.png)



### 3.上传遥测

```shell
v1/devices/me/telemetry


{
 "temperature": 42.2, 
 "humidity": 70,
 "hvacEnabled": true,
 "hvacState": "IDLE",
 "configuration": {
    "someNumber": 42,
    "someArray": [1,2,3],
    "someNestedObject": {"key": "value"}
 }
}
```



![](/images/iot/tb-edge/edge-iot/iot-23.png)

![](/images/iot/tb-edge/edge-iot/iot-24.png)

![](/images/iot/tb-edge/edge-iot/iot-25.png)



### 4.上传属性

```shell
v1/devices/me/attributes


{
	"attribute1": "value1",
	"attribute2": true,
	"attribute3": 42.0,
	"attribute4": 73,
	"attribute5": {
		"someNumber": 42,
		"someArray": [1, 2, 3],
		"someNestedObject": {
			"key": "value"
		}
	}
}
```



![](/images/iot/tb-edge/edge-iot/iot-26.png)

![](/images/iot/tb-edge/edge-iot/iot-27.png)



