* TOC
{:toc}



## 一、概述



### 1.分布式系统关键技术

- **微服务：**采用微服务分布式架构
- **容器部署：**微服务容器化部署，支持容器的编排调度
- **系统监控：**系统监控、日志聚合、链路跟踪
- **可用性：**高可用、负载均衡、故障转移、水平扩展等
- **开发运维：**采用DevOps理念，搭建CI/CD流水线
- **容灾：**建立不同地理位置的控制中心，容灾切换



![](/images/devops/cicd/cicd-plan/plan-10.png)

![](/images/devops/cicd/cicd-plan/plan-11.png)



### 2.分布式系统CI/CD流水线

**分布式系统CI/CD流水线主要实现微服务的持续集成（CI）、持续交付（CD）、持续部署（CD）。**

- 微服务采用Spring Cloud技术栈
- 微服务采用容器化部署（Docker）
- kubernetes进行容器管理（可选）



#### 2.1.容器化CI/CD流水线

![](/images/devops/cicd/cicd-plan/plan-4.png)



#### 2.2.容器部署CI/CD流水线

![](/images/devops/cicd/cicd-plan/plan-6.png)



#### 2.3.Kubernetes部署CI/CD流水线

![](/images/devops/cicd/cicd-plan/plan-7.png)





## 二、环境规划



### 1.环境介绍

开发过程中四个环境分别是：pro、pre、test、dev环境，中文名字：生产环境、灰度环境、测试环境、开发环境。

- **pro环境：**生产环境，面向外部用户的环境，连接上互联网即可访问的正式环境
- **pre环境：**灰度环境，外部用户可以访问，但是服务器配置相对低，其它和生产一样
- **test环境：**测试环境，外部用户无法访问，专门给测试人员使用的，版本相对稳定
- **dev环境：**开发环境，外部用户无法访问，开发人员使用，版本变动很大

![](/images/devops/cicd/cicd-plan/plan-1.png)



### 2.微服务规划

**微服务部署原则：**

1. 微服务采用Helm部署，通过修改 **values.yaml** 进行部署升级
2. pro、pre、test、dev环境，分别部署在名字空间：pro、pre、test、dev
3. Harbor作为私有镜像仓库，存储微服务镜像
5. Harbor作为Helm仓库



**1.架构图**

![](/images/devops/cicd/cicd-plan/plan-2.png)



**2.部署图**

![](/images/devops/cicd/cicd-plan/plan-3.png)



### 3.生产环境

- **微服务：**k8s部署微服务，名字空间：pro
- **中间件：**中间件部署在k8s外部，PostgreSQL、MySQL、Redis、RabbitMQ，名字空间：ext
- **系统监控：**k8s部署Prometheus，Elastic Stack部署在k8s外部
- **镜像仓库：**Harbor作为私有镜像仓库，创建springcloud项目
- **Helm仓库：**Harbor作为Helm私有仓库



### 4.测试环境

- **微服务：**k8s部署微服务，名字空间：test
- **中间件：**k8s部署中间件，PostgreSQL、MySQL、Redis、RabbitMQ
- **系统监控：**k8s部署Prometheus，Elastic Stack
- **镜像仓库：**Harbor作为私有镜像仓库，创建springcloud项目
- **Helm仓库：**Harbor作为Helm私有仓库





## 三、CI/CD流水线设计



### 1.CI/CD流水线规划

**分布式系统CI/CD流水线总体规划：**

- **分开搭建持续集成（CI）、持续交付（CD）、持续部署（CD）流水线**
- **持续集成创建微服务镜像，容器（Docker）镜像上传Harbor私有镜像仓库**
- **CD流水线容器化部署，Kubernetes根据实际需求选择**
- **持续交付流水线（CD）面向开发环境（dev）、测试环境（test）**
- **持续部署流水线（CD）面向生产环境（pro）、灰度环境（pre）**
- **CD流水线部署脚本使用GitLab进行版本维护**



### 2.持续集成（CI）

![](/images/devops/cicd/cicd-plan/plan-4.png)



#### 2.1.持续集成规划

**持续集成方案： Jenkins + GitLab + Maven + Docker + SonarQube + Harbor**

- **Jenkins：**开源持续集成工具，用于项目开发，具有自动化构建、测试和部署等功能

- **GitLab：**源代码仓库，使用Git作为代码管理工具
- **SonarQube：**用于管理代码质量的开放平台，可以快速的定位代码中潜在的或者明显的错误

- **Harbor：**开源的企业级DockerRegistry项目，搭建企业级的Dockerregistry服务



#### 2.2.Git分支管理

**下面是Git Flow的流程图**



![](/images/devops/cicd/cicd-plan/plan-9.png)



**根据生命周期区分**

- **主分支：**master，develop
- **临时分支：**feature/*，release/*，hotfix/*

**根据用途区分**

- **发布/预发布分支：**master，release/*
- **开发分支：**develop
- **功能分支：**feature/*
- **热修复分支：**hotfix/*



#### 2.3.容器镜像管理

**容器镜像构建：**

- **发布镜像：** 是稳定版本，运行于生产环境、灰度环境，从master分支构建，使用master分支tag作为标签，镜像标签命名规则 xxx.xxx.xxx，例如 eureka:1.0.0，eureka:1.5.0
- **测试镜像：** 是临时镜像，使用完成删除，运行于开发环境、测试环境，是从master以外的分支构建的， 镜像标签是发布镜像对应的版本标签加后缀 -xxx，例如 eureka:1.0.0-release1.1，eureka:1.5.0-dev1.5，eureka:1.5.0-hotfix1.0



**容器运行环境：**

- **发布镜像：** 生产环境（pro）、灰度环境（pre）
- **测试镜像：** 开发环境（dev）、测试环境（test）



### 3.持续部署（CD）

#### 3.1.持续部署规划

**持续部署规划：**

- **部署环境：** 生产环境（pro）、灰度环境（pre）
- **部署脚本：**部署shell脚本文件使用GitLab进行版本控制，尽量保持与微服务版本关联
- **容器环境：** 可以部署到Kubernetes或Docker
- **K8S环境：**使用Helm部署微服务



#### 3.2.部署方案

- **方案一： Jenkins + GitLab +  Harbor + Docker**

![](/images/devops/cicd/cicd-plan/plan-5.png)



- **方案二： Jenkins + GitLab +  Harbor + Kubernetes**

![](/images/devops/cicd/cicd-plan/plan-8.png)



### 4.持续交付（CD）

#### 4.1.持续交付规划

**持续交付规划：**

- **部署环境：** 开发环境（dev）、测试环境（test）
- **部署脚本：**部署shell脚本文件使用GitLab进行版本控制，尽量保持与微服务版本关联
- **容器环境：** 可以部署到Kubernetes或Docker
- **K8S环境：**使用Helm部署微服务



#### 4.2.部署方案

**与持续部署的部署方案相同。**



