* TOC
{:toc}



## 一、云原生边缘计算



### 1.云、边、端协同

**边缘计算系统中云、边、端协同包括两层：**

- 云、边协同：云作为控制平面，边作为计算平台
- 云、边、端协同：在云、边协同的基础上，管理终端设备的服务作为边上的负载。云可以通过控制边来影响端，从而实现云、边、端协同



### 2.云原生边缘计算实现

云、边、端协同是通过 Kubernetes 的控制节点、KubeEdge 和 EdgeX Foundry 共同实现的，Kubernetes的控制节点下发指令到 KubeEdge 的边缘集群，操作 Edgex Foundry 的服务，从而影响终端设备。

目前，我们还不能通过 Kubernetes 的控制节点与终端设备直接交互。



### 3.边缘计算逻辑架构

**边缘计算系统侧重云、边、端各部分之间的交互和协同**

- 云、边协同：通过云部分 Kubernetes 的控制节点和边部分 KubeEdge 所运行的节点共同实现
- 边、端协同：通过边部分 KubeEdge 和端部分 EdgeX Foundry 共同实现
- 云、边、端协同：通过云解决方案 Kubernetes 的控制节点、边缘解决方案 KubeEdge 和端解决方案 EdgeX Foundry 共同实现





## 二、边缘计算架构



### 1.Kubernetes

![](/images/kubeedge/default/edge-computing/edge-1.png)



**Kubernetes** **主要由以下几个核心组件组成：**

- **Etcd：**保存了整个集群的状态

- **Apiserver：**提供了资源操作的唯一入口，并提供认证、授权、访问控制、API 注册和发现等机制

- **controller manager：**负责维护集群的状态，比如故障检测、自动扩展、滚动更新等

- **scheduler：**负责资源的调度，按照预定的调度策略将 Pod 调度到相应的机器上

- **kubelet：**负责维护容器的生命周期，同时也负责 Volume（CSI）和网络（CNI）的管理

- **Container runtime：**负责镜像管理以及 Pod 和容器的真正运行（CRI）

- **kube-proxy：**负责为 Service 提供 cluster 内部的服务发现和负载均衡

 

**除了核心组件，还有一些推荐的插件，其中有的已经成为CNCF中的托管项目：**

- **CoreDNS：**负责为整个集群提供 DNS 服务

- **Ingress Controller：**为服务提供外网入口

- **Prometheus：**提供资源监控

- **Dashboard：**提供 GUI





### 2.KubeEdge

![](/images/kubeedge/default/edge-computing/edge-2.png)



**KubeEdge 由以下组件组成：**

- **Edged:** 在边缘节点上运行并管理容器化应用程序的代理
- **EdgeHub:** Web 套接字客户端，负责与 Cloud Service 进行交互以进行边缘计算（例如 KubeEdge 体系结构中的 Edge Controller）。这包括将云侧资源更新同步到边缘，并将边缘侧主机和设备状态变更报告给云
- **CloudHub:** Web 套接字服务器，负责在云端缓存信息、监视变更，并向 EdgeHub 端发送消息
- **EdgeController:** kubernetes 的扩展控制器，用于管理边缘节点和 pod 的元数据，以便可以将数据定位到对应的边缘节点
- **EventBus:** 一个与 MQTT 服务器（mosquitto）进行交互的 MQTT 客户端，为其他组件提供发布和订阅功能
- **DeviceTwin:** 负责存储设备状态并将设备状态同步到云端。它还为应用程序提供查询接口
- **MetaManager:** Edged 端和 Edgehub 端之间的消息处理器。它还负责将元数据存储到轻量级数据库（SQLite）或从轻量级数据库（SQLite）检索元数据





### 3.EdgeX Foundry

![](/images/kubeedge/default/edge-computing/edge-3.png)



- **设备服务层：**负责与支持特定协议的设备交互，采集设备的数据，并下发指令控制设备
- **核心服务层：**负责接收设备服务层上报的设备数据，通过向设备服务层下发指令控制设备，管理注册到 EdgeXFoundry 中的设备及其元数据，在 EdgeX Foundry中 的微服务之间提供相关配置信息
- **支持服务层：**负责提供 EdgeX Foundry 中的微服务都需要的规则引擎、调度、报警与通知和日志等通用功能

- **导出服务层：**负责将 EdgeX Foundry 采集的相关设备数据导出并进行存储
- **安全组件：** 负责保护 Edgex Foundry 中采集到的数据以及 EdgeX Foundry 所管理的设备、传感器和其他物联网设备等
- **管理组件：**负责启动、停止和重启 EdgeX Foundry 中的微服务，监控微服务的操作和性能，获得 EdgeX Foundry 中微服务的配置



