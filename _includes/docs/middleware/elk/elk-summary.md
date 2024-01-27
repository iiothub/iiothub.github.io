* TOC
{:toc}



### 1.ELK介绍

ELK是三个开源软件的缩写，分别表示：Elasticsearch , Logstash和Kibana。

ELK提供了一整套解决方案，并且都是开源软件，之间互相配合使用，完美衔接，高效的满足了很多场合的应用，是目前主流的一种日志分析平台。

一个完整的集中式日志系统，需要包含以下几个主要特点：

- 收集－能够采集多种来源的日志数据
- 传输－能够稳定的把日志数据传输到中央系统
- 存储－如何存储日志数据
- 分析－可以支持 UI 分析
- 警告－能够提供错误报告，监控机制



**简单架构**

![](/images/middleware/elk/elk-summary/sum-1.png)

这是最简单的一种ELK部署架构方式， 由Logstash分布于各个节点上搜集相关日志、数据，并经过分析、过滤后发送给远端服务器上的Elasticsearch进行存储。 优点是搭建简单， 易于上手， 缺点是Logstash耗资源较大， 依赖性强， 没有消息队列缓存， 存在数据丢失隐患。



**消息队列架构**

![](/images/middleware/elk/elk-summary/sum-2.png)

该队列架构引入了KAFKA消息队列， 解决了各采集节点上Logstash资源耗费过大， 数据丢失的问题， 各终端节点上的Logstash Agent 先将数据/日志传递给Kafka， 消息队列再将数据传递给Logstash， Logstash过滤、分析后将数据传递给Elasticsearch存储， 由Kibana将日志和数据呈现给用户。



### 2.Elastic Stack

 Elastic Stack 是 ELK Stack 的更新换代产品

实际上ELK是三款软件的简称，分别是Elasticsearch、Logstash、Kibana组成，在发展的过程中，又有新成员Beats的加入，所以就形成了Elastic Stack。
所以说，ELK是旧的称呼，Elastic Stack是新的名字。

![](/images/middleware/elk/elk-summary/sum-3.png)



**全系的Elastic Stack技术栈包括：**

![](/images/middleware/elk/elk-summary/sum-4.png)



- **Elasticsearch**

Elasticsearch 基于java，是个开源分布式搜索引擎，它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等。



- **Logstash**

Logstash 基于java，是一个开源的用于收集,分析和存储日志的工具。



- **Kibana**

Kibana 基于nodejs，也是一个开源和免费的工具，Kibana可以为 Logstash 和ElasticSearch 提供的日志分析友好的 Web 界面，可以汇总、分析和搜索重要数据日志。



- **Beats**

Beats是elastic公司开源的一款采集系统监控数据的代理agent，是在被监控服务器上以客户端形式运行的数据收集器的统称，可以直接把数据发送给Elasticsearch或者通过Logstash发送给Elasticsearch，然后进行后续的数据分析活动。



Beats由如下组成:

- Packetbeat：是一个网络数据包分析器，用于监控、收集网络流量信息，Packetbeat嗅探服务器之间的流量，解析应用层协议，并关联到消息的处理，其支 持ICMP (v4 and v6)、DNS、HTTP、Mysql、PostgreSQL、Redis、MongoDB、Memcache等协议

- Filebeat：用于监控、收集服务器日志文件，其已取代 logstash forwarder


- Metricbeat：可定期获取外部系统的监控指标信息，其可以监控、收集 Apache、HAProxy、MongoDB、MySQL、Nginx、PostgreSQL、Redis、System、Zookeeper等服务


- Winlogbeat：用于监控、收集Windows系统的日志信息



