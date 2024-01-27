* TOC
{:toc}



## 一、概述



### 1.分布式关键技术

- **微服务：**采用微服务分布式架构
- **容器部署：**微服务容器化部署，支持容器的编排调度
- **系统监控：**系统监控、日志聚合、链路跟踪
- **可用性：**高可用、负载均衡、故障转移、水平扩展等
- **开发运维：**采用DevOps理念，搭建CI/CD流水线





## 二、环境规划



### 1.环境介绍

开发过程中四个环境分别是：pro、pre、test、dev环境，中文名字：生产环境、灰度环境、测试环境、开发环境。

- **pro环境：**生产环境，面向外部用户的环境，连接上互联网即可访问的正式环境。
- **pre环境：**灰度环境，外部用户可以访问，但是服务器配置相对低，其它和生产一样。
- **test环境：**测试环境，外部用户无法访问，专门给测试人员使用的，版本相对稳定。
- **dev环境：**开发环境，外部用户无法访问，开发人员使用，版本变动很大。

![](/images/kubernetes/pro/cluster-plan/sc-1.png)



### 2.微服务规划

**微服务部署原则：**

1. 微服务采用Helm部署；
2. pro、pre、test、dev环境，分别部署在名字空间：pro、pre、test、dev；
3. 通过修改 **values.yaml** 进行部署升级；
4. Harbor作为私有镜像仓库，存储微服务镜像；
4. Harbor作为Helm仓库。



### 3.生产环境

- **微服务：**k8s部署微服务，名字空间：pro；
- **中间件：**中间件部署在k8s外部，PostgreSQL、MySQL、Redis、RabbitMQ，名字空间：ext；
- **系统监控：**k8s部署Prometheus，Elastic Stack部署在k8s外部；
- **镜像仓库：**Harbor作为私有镜像仓库，创建springcloud项目；
- **Helm仓库：**Harbor作为Helm私有仓库。



### 4.测试环境

- **微服务：**k8s部署微服务，名字空间：test；
- **中间件：**k8s部署中间件，PostgreSQL、MySQL、Redis、RabbitMQ；
- **系统监控：**k8s部署Prometheus，Elastic Stack；
- **镜像仓库：**Harbor作为私有镜像仓库，创建springcloud项目；
- **Helm仓库：**Harbor作为Helm私有仓库。





## 三、关键技术



### 1.微服务

**微服务采用Spring Cloud技术栈。**

微服务是用一组小服务构建的一个应用，服务运行在不同的进程中，服务之间通过轻量的通讯机制进行交互，并且服务可以通过自动化部署方式独立部署。

**微服务优点**

- 服务解耦

- 独立的开发环境

- 独立的部署环境

- 更高的扩展性


**微服务重要功能**

- 服务注册与发现

- 服务网关

- 安全中心、配置中心

- 负载均衡、熔断限流

- 监控告警、日志审计



**1.架构图**



![](/images/kubernetes/pro/cluster-plan/dep-2.png)





**2.部署图**

![](/images/kubernetes/pro/cluster-plan/dep-3.png)





### 2.容器部署

**分布式集群容器选择Docker，k8s进行容器管理。**

**容器：**方便做持续集成并有助于整体发布的虚拟化技术。 容器之间互相隔离，每个容器有自己的文件系统 ，容器之间进程不会相互影响，能区分计算资源。

**一次构建，随处运行**

- 更快速的应用交付和部署

- 更便捷的升级和扩缩容

- 更简单的系统运维

- 更高效的计算资源利用


![](/images/kubernetes/pro/cluster-plan/plain-5.png)





### 3.系统监控

#### 3.1.监控方式

- **指标监控：**监控服务器、网络、应用服务、中间件、数据库、重要功能模块等资源利用情况，是否正常工作；
- **调用链监控：**生成TMS网络拓扑图，追踪调用关系，快速定位问题；性能瓶颈分析，优化系统；提高团队成员自律性；
- **日志聚合：**TMS是分布式系统，各种日志都是分散的，要创建一个集中式日志系统。



**告警分类**

**业务告警：**结合业务场景，实现可配置的业务告警规则，提供告警监控、管理UI；

**系统告警：**基础设置（服务器、网络、CPU、内存、硬盘）故障或资源利用超过阈值报警，应用服务、中间件、数据库宕机或异常等报警。

![](/images/kubernetes/pro/cluster-plan/plain-1.png)



![](/images/kubernetes/pro/cluster-plan/plain-2.png)



#### 3.2.系统监控 — Prometheus

![](/images/kubernetes/pro/cluster-plan/plain-6.png)



#### 3.3.日志聚合 — Elastic Stack

![](/images/kubernetes/pro/cluster-plan/plain-8.png)



#### 3.4.调用链监控 — Zipkin

![](/images/kubernetes/pro/cluster-plan/plain-7.png)





### 4.开发运维

**采用DevOps理念，搭建CI/CD流水线**

- **持续集成（CI）：**不断去构建，编译和测试，可以很早期发现问题，降低风险；
- **持续交付（CD）：**提供可使用的版本；
- **持续部署（CD）：**提供可部署镜像。

![](/images/kubernetes/pro/cluster-plan/plain-4.png)



![](/images/kubernetes/pro/cluster-plan/plain-9.png)



