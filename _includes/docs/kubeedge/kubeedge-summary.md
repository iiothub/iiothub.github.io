* TOC
{:toc}



### 1.KubeEdge

**KubeEdge** 是一个开源的系统，可将本机容器化应用编排和管理扩展到边缘端设备。它基于 Kubernetes 构建，为网络和应用程序提供核心基础架构支持，并在云端和边缘端部署应用，同步元数据。

**KubeEdge** 还支持 **MQTT** 协议，允许开发人员编写客户逻辑，并在边缘端启用设备通信的资源约束。KubeEdge 包含云端和边缘端两部分。

**KubeEdge** 可以很容易地将已有的复杂机器学习、图像识别、事件处理和其他高级应用程序部署到边缘端并进行使用。 随着业务逻辑在边缘端上运行，可以在本地保护和处理大量数据。 通过在边缘端处理数据，响应速度会显著提高，并且可以更好地保护数据隐私。

**KubeEdge** 是一个由 [Cloud Native Computing Foundation](https://cncf.io/) (CNCF) 托管的孵化级项目，CNCF 对 KubeEdge 的 [孵化公告](https://www.cncf.io/blog/2020/09/16/toc-approves-kubeedge-as-incubating-project/)



### 2.KubeEdge 特点

KubeEdge 的优势主要包括：

- **边缘计算**

  借助在 Edge 上运行的业务逻辑，可以让本地生成的数据，进行大量数据处理操作并对其进行保护。这样可以减少边缘和云之间的网络带宽需求和消耗，提高响应速度，降低成本并保护客户的数据隐私。

- **简化开发**

  开发人员可以编写基于 HTTP 或 MQTT 的常规应用程序，对其进行容器化，然后在 Edge 或 Cloud 中的任何一个更合适的位置运行应用程序。

- **Kubernetes 原生支持**

  借助 KubeEdge，用户可以像在传统的 Kubernetes 集群一样，在 Edge 节点上编排应用程序，管理设备并监视应用程序和设备状态。

- **丰富的应用**

  可以轻松地将现有的复杂机器学习，图像识别，事件处理等其他高级应用程序部署到 Edge。

  

  

### 3.KubeEdge 组成

**KubeEdge 由以下组件组成：**

- **Edged:** 在边缘节点上运行并管理容器化应用程序的代理
- **EdgeHub:** Web 套接字客户端，负责与 Cloud Service 进行交互以进行边缘计算（例如 KubeEdge 体系结构中的 Edge Controller）。这包括将云侧资源更新同步到边缘，并将边缘侧主机和设备状态变更报告给云
- **CloudHub:** Web 套接字服务器，负责在云端缓存信息、监视变更，并向 EdgeHub 端发送消息
- **EdgeController:** kubernetes 的扩展控制器，用于管理边缘节点和 pod 的元数据，以便可以将数据定位到对应的边缘节点
- **EventBus:** 一个与 MQTT 服务器（mosquitto）进行交互的 MQTT 客户端，为其他组件提供发布和订阅功能
- **DeviceTwin:** 负责存储设备状态并将设备状态同步到云端。它还为应用程序提供查询接口
- **MetaManager:** Edged 端和 Edgehub 端之间的消息处理器。它还负责将元数据存储到轻量级数据库（SQLite）或从轻量级数据库（SQLite）检索元数据



### 4.KubeEdge 架构

![](/images/kubeedge/default/edge-computing/kubeedge-1.png)



