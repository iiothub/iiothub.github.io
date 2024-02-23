* TOC
{:toc}



## 1.EdgeX Foundry

EdgeX Foundry 是 LF Edge 旗下的一款开源、不受供应商限制的边缘物联网中间件平台。

该平台会从边缘处的传感器(即“物”)收集数据，并作为双向传输引擎向企业、云和本地应用发送数据，以及从这些应用接收数据。EdgeX 可在边缘实现自主运营和智能化(AI)。

![](/images/edgex/default/edgex-foundry/platform-1.png)

![](/images/edgex/default/edgex-foundry/platform-2.png)



开发人员、技术提供商和最终用户能够通过技术、资源共享和 **EdgeX** 生态系统的规模经济（无论是其自己的实践，还是通过向他人提供商业化的“**EdgeX** 就绪型”解决方案），以更低的成本和风险加速实现业务价值。

**EdgeX** 在许多方面都独具特色，比如服务范围、广泛的行业支持、可信度、投入，以及由 Linux 基金会旗下 LF Edge 组织所提供的不受供应商限制的 Apache 2.0 开源许可模式。**EdgeX** 本身也是在所有垂直市场物联网用例和企业中推动数字转型与 AI 技术发展的核心要素。



**项目服务范围**

![](/images/edgex/default/edgex-foundry/platform-4.jpg)

**EdgeX** Foundry 专注于充分运用云原生原则（例如，松耦合的微服务、平台独立性），以及实现满足特定物联网边缘需求的架构（包括不同的连接协议、广泛分布的计算节点的安全性和系统管理，以及缩减高度受限的设备规模），借此发挥边缘计算的优势。

该项目的“甜蜜点”在于，用例中的本地决策可以实时或以近乎实时的速度进行制定，与此同时，自动化和操作由多个数据源提供支持。在这里，**EdgeX** 可以解决边缘节点和数据规范化（比如，在分布式物联网边缘架构中需满足“南正对北、东正对西”的条件）方面的关键互操作性挑战。





## 2.平台架构

![](/images/edgex/default/edgex-foundry/platform-3.jpg)

 EdgeX 采用微服务风格架构，这些微服务组织为四个服务层和两个底层系统服务。

- **设备服务层：**负责与支持特定协议的设备交互，采集设备的数据，并下发指令控制设备
- **核心服务层：**负责接收设备服务层上报的设备数据，通过向设备服务层下发指令控制设备，管理注册到 EdgeXFoundry 中的设备及其元数据，在 EdgeX Foundry 中 的微服务之间提供相关配置信息
- **支持服务层：**负责提供 EdgeX Foundry 中的微服务都需要的规则引擎、调度、报警与通知和日志等通用功能
- **导出服务层：**负责将 EdgeX Foundry 采集的相关设备数据导出并进行存储
- **安全组件：** 负责保护 Edgex Foundry 中采集到的数据以及 EdgeX Foundry 所管理的设备、传感器和其他物联网设备等
- **管理组件：**负责启动、停止和重启 EdgeX Foundry 中的微服务，监控微服务的操作和性能，获得 EdgeX Foundry 中微服务的配置



## 3.平台服务

### 3.1.设备服务

![](/images/edgex/default/edgex-foundry/platform-5.jpg)

设备服务是与传感器/设备或物联网对象(“物”)交互的边缘连接器，其中包括机器、机器人、无人机、HVAC 设备、相机等。通过利用可用的连接器，可以控制设备并/或传输数据至 EdgeX 或从其传输数据。您还可以使用设备服务 SDK 来创建您自己的 EdgeX 设备服务。

![](/images/edgex/default/edgex-foundry/platform-6.jpg)



### 3.2.核心服务

![](/images/edgex/default/edgex-foundry/platform-7.jpg)



核心服务通过这些服务可大体了解给定部署中连接了哪些设备，正在传输哪些数据以及 EdgeX的配置方式。

![](/images/edgex/default/edgex-foundry/platform-8.jpg)



### 3.3.支持服务

![](/images/edgex/default/edgex-foundry/platform-9.jpg)

支持服务包括诸如边缘分析(也称为“本地分析”)等微服务，以及典型的软件应用功能，例如记录、计划和数据清理等。

![](/images/edgex/default/edgex-foundry/platform-10.jpg)



### 3.4.应用服务

![](/images/edgex/default/edgex-foundry/platform-11.jpg)

应用服务是指将感应到的数据从 Edgex 提取、处理/转换和发送到所选端点或应用的方式。这些服务可以是分析数据包、企业或本地应用，也可以是 Azure loTHub、AWSloT 或 Google loT Core 等云系统。

![](/images/edgex/default/edgex-foundry/platform-12.jpg)



### 3.5.安全服务

![](/images/edgex/default/edgex-foundry/platform-13.jpg)

安全服务可以保护设备、传感器，以及由 **EdgeX** Foundry 托管之其他物联网对象的数据和控制。

![](/images/edgex/default/edgex-foundry/platform-14.jpg)



### 3.6.管理服务

![](/images/edgex/default/edgex-foundry/platform-15.jpg)

系统管理设施为外部管理系统提供中央联络点，以便启动/停止/重启 **EdgeX** 服务、为服务获取配置、取得服务的状态/运行状况，或者取得关于 **EdgeX** 服务的指标（例如，内存使用量）以便能够对 **EdgeX** 服务进行监控。

![](/images/edgex/default/edgex-foundry/platform-16.jpg)



