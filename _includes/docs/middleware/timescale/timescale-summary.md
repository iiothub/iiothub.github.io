* TOC
{:toc}



### 1.TimescaleDB介绍

TimescaleDB是基于PostgreSQL数据库打造的一款时序数据库，插件化的形式，随着PostgreSQL的版本升级而升级。

**TimescaleDB具备以下特点**

1. 基于时序优化
2. 自动分片（按时间、空间自动分片(chunk)）
3. 全SQL接口
4. 支持垂直于横向扩展
5. 支持时间维度、空间维度自动分区。空间维度指属性字段（例如传感器ID，用户ID等）
6. 支持多个SERVER，多个CHUNK的并行查询。分区在TimescaleDB中被称为chunk
7. 自动调整CHUNK的大小
8. 内部写优化（批量提交、内存索引、事务支持、数据倒灌）
内存索引，因为chunk size比较适中，所以索引基本上都不会被交换出去，写性能比较好
数据倒灌，因为有些传感器的数据可能写入延迟，导致需要写以前的chunk，timescaleDB允许这样的事情发生(可配置)
9. 复杂查询优化（根据查询条件自动选择chunk，最近值获取优化(最小化的扫描,类似递归收敛)，limit子句pushdown到不同的server,chunks，并行的聚合操作）
《时序数据合并场景加速分析和实现 - 复合索引，窗口分组查询加速，变态递归加速》
10. 利用已有的PostgreSQL特性（支持GIS，JOIN等），方便的管理（流复制、PITR）
11. 支持自动的按时间保留策略（自动删除过旧数据）



```shell
# 官网地址
https://www.timescale.com/
 
# 文档
https://docs.timescale.com/latest/main
 
# 安装
https://docs.timescale.com/latest/getting-started/installation/rhel-centos/installation-yum 
 
# github
https://github.com/timescale/timescaledb 
 
# docker
https://hub.docker.com/r/timescale/timescaledb
```



### 2.Hypertable 和 chunk

TimescaleDB作为PostgreSQL的扩展实现，这意味着Timescale数据库在整个PostgreSQL实例中运行。 该扩展模型允许数据库利用PostgreSQL的许多属性，如可靠性，安全性以及与各种第三方工具的连接性。 同时，TimescaleDB通过在PostgreSQL的查询规划器，数据模型和执行引擎中添加钩子，充分利用扩展可用的高度自定义。
从用户的角度来看，TimescaleDB公开了一些看起来像单数表的称为hypertable的表，它们实际上是一个抽象或许多单独表的虚拟视图，这些表包含称为块的数据。

通过将hypertable的数据划分为一个或多个维度来创建块：所有可编程元素按时间间隔进行分区，并且可以通过诸如设备ID，位置，用户ID等的关键字进行分区。我们有时将此称为分区 横跨“时间和空间”。 



- **Hypertable**

与数据交互的主要点是一个可以抽象化的跨越所有空间和时间间隔的单个连续表，从而可以通过标准SQL查询它。
实际上，所有与TimescaleDB的用户交互都是使用可调整的。 创建表格和索引，修改表格，插入数据，选择数据等都可以（也应该）在hypertable上执行。

在TimescaleDB中创建一个超表需要两个简单的SQL命令：创建表（使用标准SQL语法），然后选择CLEATEYHYTABLE（）。



- **chunk**

在内部，TimescaleDB自动将每个可分区块分割成块，每个块对应于特定的时间间隔和分区键空间的一个区域（使用散列）。 这些分区是不相交的（非重叠的），这有助于查询计划人员最小化它必须接触以解决查询的组块集合。
每个块都使用标准数据库表来实现。 （在PostgreSQL内部，这个块实际上是一个“父”可变的“子表”。）
块是正确的大小，确保表的索引的所有B树可以在插入期间驻留在内存中。 这可以避免在修改这些树中的任意位置时发生颠簸。

```shell
SELECT show_chunks('conditions');
SELECT show_chunks('conditions', older_than => INTERVAL '3 months');
SELECT show_chunks('conditions', older_than => DATE '2017-01-01');
```



### 3.Hypertable

```shell
create_hypertable
 
SELECT * FROM create_hypertable(...) 
 
# 创建超表
SELECT create_hypertable('conditions', 'time');
 
# 将表条件转换为超表，将chunk_time_interval设置为24小时。 
SELECT create_hypertable('conditions', 'time', chunk_time_interval => 86400000000);
SELECT create_hypertable('conditions', 'time', chunk_time_interval => INTERVAL '1 day');
 
chunk_time_interval 
Interval in event time that each chunk covers. Must be > 0. As of TimescaleDB v0.11.0, default is 7 days. For previous versions, default is 1 month.  
 
 
# 使用时间分区和位置分区（4个分区）将表条件转换为超表： 
SELECT create_hypertable('conditions', 'time', 'location', 4); 
```



![](/images/middleware/timescale/deploy/deploy-7.png)



- **create_hypertable()**
- ![](/images/middleware/timescale/deploy/deploy-8.png)



- **add_dimension()**

![](/images/middleware/timescale/deploy/deploy-9.png)



### 4.Hypertable操作

```shell
1. 创建时序表(hypertable)
# Create a schema for a new hypertable  
CREATE TABLE sensor_data (  
"time" timestamp with time zone NOT NULL,  
device_id TEXT NOT NULL,  
location TEXT NULL,  
temperature NUMERIC NULL,  
humidity NUMERIC NULL,  
pm25 NUMERIC  
);  
  
# Create a hypertable from this data  
SELECT create_hypertable  
('sensor_data', 'time', 'device_id', 16);  
 
2. 迁移数据到hyper table
# Migrate data from existing Postgres table into  
# a TimescaleDB hypertable  
INSERT INTO sensor_data (SELECT * FROM old_data);  
 
3. 查询hyper table
# Query hypertable like any SQL table  
SELECT device_id, AVG(temperature) from sensor_data  
WHERE temperature IS NOT NULL AND humidity > 0.5  
AND time > now() - interval '7 day'  
GROUP BY device_id;  
 
4. 查询最近异常的数据
# Metrics about resource-constrained devices  
SELECT time, cpu, freemem, battery FROM devops  
WHERE device_id='foo'  
AND cpu > 0.7 AND freemem < 0.2  
ORDER BY time DESC  
LIMIT 100;  
 
5. 计算最近7天，每小时的异常次数
# Calculate total errors by latest firmware versions  
# per hour over the last 7 days  
SELECT date_trunc('hour', time) as hour, firmware,  
COUNT(error_msg) as errno FROM data  
WHERE firmware > 50  
AND time > now() - interval '7 day'  
GROUP BY hour, firmware  
ORDER BY hour DESC, errno DESC;  
 
6. 计算巴士的每小时平均速度
# Find average bus speed in last hour  
# for each NYC borough  
SELECT loc.region, AVG(bus.speed) FROM bus  
INNER JOIN loc ON (bus.bus_id = loc.bus_id)  
WHERE loc.city = 'nyc'  
AND bus.time > now() - interval '1 hour'  
GROUP BY loc.region;  
 
7. 展示最近12小时，每小时的平均值
=#  SELECT date_trunc('hour', time) AS hour, AVG(weight)  
    FROM logs  
    WHERE device_type = 'pressure-sensor' AND customer_id = 440  
      AND time > now() - interval '12 hours'  
    GROUP BY hour;  
  
 hour               | AVG(weight)  
--------------------+--------------  
 2017-01-04 12:00   | 170.0  
 2017-01-04 13:00   | 174.2  
 2017-01-04 14:00   | 174.0  
 2017-01-04 15:00   | 178.6  
 2017-01-04 16:00   | 173.0  
 2017-01-04 17:00   | 169.9  
 2017-01-04 18:00   | 168.1  
 2017-01-04 19:00   | 170.2  
 2017-01-04 20:00   | 167.4  
 2017-01-04 21:00   | 168.6  
 
8. 监控每分钟过载的设备数量
=#  SELECT date_trunc('minute', time) AS minute, COUNT(device_id)  
    FROM logs  
    WHERE cpu_level > 0.9 AND free_mem < 1024  
      AND time > now() - interval '24 hours'  
    GROUP BY minute  
    ORDER BY COUNT(device_id) DESC LIMIT 25;  
  
 minute             | heavy_load_devices  
--------------------+---------------------  
 2017-01-04 14:59   | 1653  
 2017-01-04 15:01   | 1650  
 2017-01-04 15:00   | 1605  
 2017-01-04 15:02   | 1594  
 2017-01-04 15:03   | 1594  
 2017-01-04 15:04   | 1561  
 2017-01-04 15:06   | 1499  
 2017-01-04 15:05   | 1460  
 2017-01-04 15:08   | 1459  
 
9. 最近7天，按固件版本，输出每个固件版本的报错次数
=#  SELECT firmware_version, SUM(error_count) FROM logs  
    WHERE time > now() - interval '7 days'  
    GROUP BY firmware_version  
    ORDER BY SUM(error_count) DESC LIMIT 10;  
  
 firmware_version  | SUM(error_count)  
-------------------+-------------------  
 1.0.10            | 191  
 1.1.0             | 180  
 1.1.1             | 179  
 1.0.8             | 164  
 1.1.3             | 161  
 1.1.2             | 152  
 1.2.1             | 144  
 1.2.0             | 137  
 1.0.7             | 130  
 1.0.5             | 112  
 1.2.2             | 110  
 
10. 某个范围，每小时，温度高于90度的设备数量。
=#  SELECT date_trunc('hour', time) AS hour, COUNT(logs.device_id)  
    FROM logs  
    JOIN devices ON logs.device_id = devices.id  
    WHERE logs.temperature > 90 AND devices.location = 'SITE-1'  
    GROUP BY hour;  
  
 hour               | COUNT(logs.device_id)  
--------------------+------------------------  
 2017-01-04 12:00   | 994  
 2017-01-04 13:00   | 905  
 2017-01-04 14:00   | 875  
 2017-01-04 15:00   | 910  
 2017-01-04 16:00   | 905  
 2017-01-04 17:00   | 840  
 2017-01-04 18:00   | 801  
 2017-01-04 19:00   | 813  
 2017-01-04 20:00   | 798  
```



