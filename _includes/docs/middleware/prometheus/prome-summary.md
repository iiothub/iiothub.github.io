* TOC
{:toc}



### 1.Prometheus简介

Prometheus 是一套开源的系统监控报警框架。它启发于 Google 的 borgmon 监控系统，由工作在 SoundCloud 的 google 前员工在 2012 年创建，作为社区开源项目进行开发，并于 2015 年正式发布。2016 年，Prometheus 正式加入 Cloud Native Computing Foundation，成为受欢迎度仅次于 Kubernetes 的项目。

![](/images/middleware/prometheus/prome-summary/sum-1.png)



**Prometheus特点**

- 强大的多维度数据模型：
  - 时间序列数据通过 metric 名和键值对来区分
  - 所有的 metrics 都可以设置任意的多维标签
  - 数据模型更随意，不需要刻意设置为以点分隔的字符串
  - 可以对数据模型进行聚合，切割和切片操作
  - 支持双精度浮点类型，标签可以设为全 unicode
- 灵活而强大的查询语句（PromQL）：在同一个查询语句，可以对多个 metrics 进行乘法、加法、连接、取分数位等操作
- 易于管理： Prometheus server 是一个单独的二进制文件，可直接在本地工作，不依赖于分布式存储
- 高效：平均每个采样点仅占 3.5 bytes，且一个 Prometheus server 可以处理数百万的 metrics
- 使用 pull 模式采集时间序列数据，这样不仅有利于本机测试而且可以避免有问题的服务器推送坏的 metrics
- 可以采用 push gateway 的方式把时间序列数据推送至 Prometheus server 端
- 可以通过服务发现或者静态配置去获取监控的 targets
- 有多种可视化图形界面
- 易于伸缩



### 2.Prometheus架构

![](/images/middleware/prometheus/prome-summary/sum-2.png)

**Prometheus组件：**

- Prometheus Server: 用于收集和存储时间序列数据
- Client Library: 客户端库，为需要监控的服务生成相应的 metrics 并暴露给 Prometheus server。当 Prometheus server 来 pull 时，直接返回实时状态的 metrics
- Push Gateway: 主要用于短期的 jobs。由于这类 jobs 存在时间较短，可能在 Prometheus 来 pull 之前就消失了。为此，这次 jobs 可以直接向 Prometheus server 端推送它们的 metrics。这种方式主要用于服务层面的 metrics，对于机器层面的 metrices，需要使用 node exporter
- Exporters: 用于暴露已有的第三方服务的 metrics 给 Prometheus
- Alertmanager: 从 Prometheus server 端接收到 alerts 后，会进行去除重复数据，分组，并路由到对收的接受方式，发出报警。常见的接收方式有：电子邮件，pagerduty，OpsGenie, webhook 等
- 一些其他的工具



### 3.数据模型

- 数据模型

![](/images/middleware/prometheus/prome-summary/sum-3.png)

```shell
Prometheus将所有数据存储为时间序列；具有相同度量名称以及标签属于同一个指标。
每个时间序列都由度量标准名称和一组键值对（也成为标签）唯一标识。
时间序列格式：
<metric name>{<label name>=<label value>, ...}

示例：
api_http_requests_total{method="POST", handler="/messages"}
度量名称{标签名=值}值
HELP 说明指标是干什么的
TYPE 指标类型，这个数据的指标类型
 
注：度量名通常是一英文命名清晰。标签名英文、值推荐英文。
```

![](/images/middleware/prometheus/prome-summary/sum-4.png)



- Prometheus四种数据类型

![](/images/middleware/prometheus/prome-summary/sum-5.png)



### 4.Prometheus Server

Prometheus Server是Prometheus组件中的核心部分，负责实现对监控数据的获取，存储以及查询。 Prometheus Server可以通过静态配置管理监控目标，也可以配合使用Service Discovery的方式动态管理监控目标，并从这些监控目标中获取数据。其次Prometheus Server需要对采集到的监控数据进行存储，Prometheus Server本身就是一个时序数据库，将采集到的监控数据按照时间序列的方式存储在本地磁盘当中。最后Prometheus Server对外提供了自定义的PromQL语言，实现对数据的查询以及分析。

Prometheus Server内置的Express Browser UI，通过这个UI可以直接通过PromQL实现数据的查询以及可视化。
Prometheus Server的联邦集群能力可以使其从其他的Prometheus Server实例中获取数据，因此在大规模监控的情况下，可以通过联邦集群以及功能分区的方式对Prometheus Server进行扩展。

![](/images/middleware/prometheus/prome-summary/sum-6.png)



### 5.AlertManager

告警能力在Prometheus的架构中被划分成两个独立的部分。如下所示，通过在Prometheus中定义AlertRule（告警规则），Prometheus会周期性的对告警规则进行计算，如果满足告警触发条件就会向Alertmanager发送告警信息。

![]/images/middleware/prometheus/prome-summary/sum-7.png)

Alertmanager作为一个独立的组件，负责接收并处理来自Prometheus Server(也可以是其它的客户端程序)的告警信息。Alertmanager可以对这些告警信息进行进一步的处理，比如当接收到大量重复告警时能够消除重复的告警信息，同时对告警信息进行分组并且路由到正确的通知方，Prometheus内置了对邮件，Slack等多种通知方式的支持，同时还支持与Webhook的集成，以支持更多定制化的场景。例如，目前Alertmanager还不支持钉钉，那用户完全可以通过Webhook与钉钉机器人进行集成，从而通过钉钉接收告警信息。同时AlertManager还提供了静默和告警抑制机制来对告警通知行为进行优化。



**Alertmanager特性**

- 分组

分组机制可以将详细的告警信息合并成一个通知。在某些情况下，比如由于系统宕机导致大量的告警被同时触发，在这种情况下分组机制可以将这些被触发的告警合并为一个告警通知，避免一次性接受大量的告警通知，而无法对问题进行快速定位。

- 抑制

抑制是指当某一告警发出后，可以停止重复发送由此告警引发的其它告警的机制。

- 静默

静默提供了一个简单的机制可以快速根据标签对告警进行静默处理。如果接收到的告警符合静默的配置，Alertmanager则不会发送告警通知。



### 6.Exporters

广义上讲所有可以向Prometheus提供监控样本数据的程序都可以被称为一个Exporter。而Exporter的一个实例称为target，如下所示，Prometheus通过轮询的方式定期从这些target中获取样本数据:

![](/images/middleware/prometheus/prome-summary/sum-8.png)



**Exporter的来源**

 **从Exporter的来源上来讲，主要分为两类：**

- 社区提供的

Prometheus社区提供了丰富的Exporter实现，涵盖了从基础设施，中间件以及网络等各个方面的监控功能。这些Exporter可以实现大部分通用的监控需求。

![](/images/middleware/prometheus/prome-summary/sum-9.png)

- 用户自定义的

除了直接使用社区提供的Exporter程序以外，用户还可以基于Prometheus提供的Client Library创建自己的Exporter程序，目前Promthues社区官方提供了对以下编程语言的支持：Go、Java/Scala、Python、Ruby。同时还有第三方实现的如：Bash、C++、Common Lisp、Erlang,、Haskeel、Lua、Node.js、PHP、Rust等。



### 7.Pushgateway

Pushgateway 是 Prometheus 生态中一个重要工具，使用它的原因主要是：

- Prometheus 采用 pull 模式，可能由于不在一个子网或者防火墙原因，导致 Prometheus 无法直接拉取各个 target 数据
- 在监控业务数据的时候，需要将不同数据汇总, 由 Prometheus 统一收集

由于以上原因，不得不使用 pushgateway，但在使用之前，有必要了解一下它的一些弊端：

- 将多个节点数据汇总到 pushgateway, 如果 pushgateway 挂了，受影响比多个 target 大
- Prometheus 拉取状态 up 只针对 pushgateway, 无法做到对每个节点有效
- Pushgateway 可以持久化推送给它的所有监控数据

因此，即使你的监控已经下线，prometheus 还会拉取到旧的监控数据，需要手动清理 pushgateway 不要的数据。



### 8.Grafana

Grafana是一个通用的可视化工具。‘通用’意味着Grafana不仅仅适用于展示Prometheus下的监控数据，也同样适用于一些其他的数据可视化需求。在开始使用Grafana之前，我们首先需要明确一些Grafana下的基本概念，以帮助用户能够快速理解Grafana。



- **数据源**

对于Grafana而言，Prometheus这类为其提供数据的对象均称为数据源（Data Source）。目前，Grafana官方提供了对：Graphite, InfluxDB, OpenTSDB, Prometheus, Elasticsearch, CloudWatch的支持。对于Grafana管理员而言，只需要将这些对象以数据源的形式添加到Grafana中，Grafana便可以轻松的实现对这些数据的可视化工作。



- **仪表盘**

通过数据源定义好可视化的数据来源之后，对于用户而言最重要的事情就是实现数据的可视化。在Grafana中，我们通过Dashboard来组织和管理我们的数据可视化图表：

![](/images/middleware/prometheus/prome-summary/sum-10.png)

![](/images/middleware/prometheus/prome-summary/sum-11.png)



### 9.微服务监控

- 监控方式

![](/images/middleware/prometheus/prome-summary/sum-22.png)



- 监控分类

![](/images/middleware/prometheus/prome-summary/sum-23.png)



- 监控场景

![](/images/middleware/prometheus/prome-summary/sum-24.png)



- 监控意义

![](/images/middleware/prometheus/prome-summary/sum-25.png)



