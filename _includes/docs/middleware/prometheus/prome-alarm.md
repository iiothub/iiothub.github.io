* TOC
{:toc}



## 一、告警通知



### 1.邮件通知

- 配置信息

```shell
# 配置信息
 
 
### 配置文件 alertmanager.yml
/prometheus/alertmanager/alertmanager.yml
touch alertmanager.yml
 
global:  # 全局配置项
  resolve_timeout: 5m #超时,默认5min
  smtp_smarthost: 'smtp.163.com:465'
  smtp_from: 'hollysys_test@163.com'
  smtp_auth_username: 'hollysys_test@163.com'
  smtp_auth_password: 'XXXXXXXX'  # 授权码：XXXXXXXX
  smtp_require_tls: false
 
templates:  # 定义模板信息
  - 'template/*.tmpl'   # 路径
 
route:  # 路由
  group_by: ['alertname'] # 报警分组依据
  group_wait: 10s #组等待时间
  group_interval: 10s # 发送前等待时间
  repeat_interval: 1h #重复周期
  receiver: 'mail' # 默认警报接收者
 
receivers:  # 警报接收者
- name: 'mail' #警报名称
  email_configs:
  - to: '{{ template "email.to" . }}'  #接收警报的email
    html: '{{ template "email.to.html" . }}' # 模板
    send_resolved: true
 
inhibit_rules:  # 告警抑制
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```



- 自定义邮件模板

![](/images/middleware/prometheus/prome-alarm/alarm-9.png)



### 2.钉钉通知

#### 2.1.获取钉钉机器人webhook

- 配置信息

![](/images/middleware/prometheus/prome-alarm/alarm-1.png)

![](/images/middleware/prometheus/prome-alarm/alarm-2.png)

![](/images/middleware/prometheus/prome-alarm/alarm-3.png)

![](/images/middleware/prometheus/prome-alarm/alarm-4.png)

![](/images/middleware/prometheus/prome-alarm/alarm-5.png)

![](/images/middleware/prometheus/prome-alarm/alarm-6.png)



#### 2.2.prometheus-webhook-dingtalk

```shell
# prometheus-webhook-dingtalk
 
# timonwong /prometheus-webhook-dingtalk 
https://github.com/timonwong/prometheus-webhook-dingtalk
https://hub.docker.com/r/timonwong/prometheus-webhook-dingtalk
 
# Webhook
https://oapi.dingtalk.com/robot/send?access_token=d8e443b4fe8512dba6c764afad94bd361fbf71c6f612c8de3bcf88d8ae545ed53
```

```shell
# 拉取镜像
docker pull timonwong/prometheus-webhook-dingtalk
 
# 运行docker
docker run -d -p 8060:8060 --name webhook-dingding timonwong/prometheus-webhook-dingtalk:latest \
--ding.profile="webhook1=https://oapi.dingtalk.com/robot/send?access_token=d8e443b4fe8512dba6c764afad94bd361fbf71c6f612c8de3bcf88d8ae545ed53"
```



#### 2.3.配置信息

```shell
# 配置信息
 
 
### 配置文件 alertmanager.yml
/prometheus/alertmanager/alertmanager.yml
touch alertmanager.yml
 
global:  # 全局配置项
  resolve_timeout: 5m #超时,默认5min
 
route:  # 路由
  receiver: webhook
  group_by: ['alertname'] # 报警分组依据
  group_wait: 10s #组等待时间
  group_interval: 10s # 发送前等待时间
  repeat_interval: 1h #重复周期
  routes:
  - receiver: webhook
    group_wait: 10s
 
receivers:
- name: webhook
  webhook_configs:
  - url: http://172.17.88.22:8060/dingtalk/webhook1/send  
    send_resolved: true
```



#### 2.4.自定义模板

![](/images/middleware/prometheus/prome-alarm/alarm-10.png)

![](/images/middleware/prometheus/prome-alarm/alarm-11.png)

![](/images/middleware/prometheus/prome-alarm/alarm-12.png)

![](/images/middleware/prometheus/prome-alarm/alarm-13.png)

![](/images/middleware/prometheus/prome-alarm/alarm-14.png)



![](/images/middleware/prometheus/prome-alarm/alarm-7.png)

![](/images/middleware/prometheus/prome-alarm/alarm-8.png)



### 3.自定义

- 配置信息

```shell
# 配置信息

### 配置文件 alertmanager.yml
/prometheus/alertmanager/alertmanager.yml
touch alertmanager.yml
 
global:  # 全局配置项
  resolve_timeout: 5m #超时,默认5min
 
route:  # 路由
  group_by: ['alertname'] # 报警分组依据
  group_wait: 10s #组等待时间
  group_interval: 10s # 发送前等待时间
  repeat_interval: 1h #重复周期
  receiver: webhook
 
receivers:
- name: webhook
  webhook_configs:
  - url: http://172.17.88.22:8888/monitor  
```



- 自定义接口

```shell
# AlertController
@Slf4j
@Controller
@RequestMapping("/")
public class AlertController {

    @RequestMapping(value = "/monitor", produces = "application/json;charset=UTF-8")
    @ResponseBody
    public String monitor(@RequestBody String json) {

        log.info("alert notify  params: {}", json);


        Map<String, Object> result = new HashMap<>();
        result.put("msg", "报警失败");
        result.put("code", 0);
        return JSON.toJSONString(result);
    }
}
 
# 接口地址
url: http://172.17.88.22:8888/monitor  
```

```shell
# 报警数据
 
# 触发报警数据：
{
 "receiver": "webhook",
 "status": "firing",
 "alerts": [{
   "status": "firing",
   "labels": {
    "alertname": "node-up",
    "instance": "node-3",
    "job": "centos-3",
    "severity": "1",
    "team": "node"
   },
   "annotations": {
    "description": "node-3 检测到异常停止！请重点关注！！！",
    "summary": "node-3 已停止运行! alert"
   },
   "startsAt": "2020-08-20T07:09:35.987923059Z",
   "endsAt": "0001-01-01T00:00:00Z",
   "generatorURL": "http://test-1:9090/graph?g0.expr=up%7Bjob%3D%22centos-3%22%7D+%3D%3D+0\u0026g0.tab=1",
   "fingerprint": "d0412b7cebb1a039"
 }],
 "groupLabels": {
   "alertname": "node-up"
 },
 "commonLabels": {
   "alertname": "node-up",
   "instance": "node-3",
   "job": "centos-3",
   "severity": "1",
   "team": "node"
 },
 "commonAnnotations": {
   "description": "node-3 检测到异常停止！请重点关注！！！",
   "summary": "node-3 已停止运行! alert"
 },
 "externalURL": "http://test-1:9093",
 "version": "4",
 "groupKey": "{}:{alertname=\"node-up\"}",
 "truncatedAlerts": 0
}
 
# 恢复报警数据：
{
 "receiver": "webhook",
 "status": "resolved",
 "alerts": [{
   "status": "resolved",
   "labels": {
    "alertname": "node-up",
    "instance": "node-3",
    "job": "centos-3",
    "severity": "1",
    "team": "node"
   },
   "annotations": {
    "description": "node-3 检测到异常停止！请重点关注！！！",
    "summary": "node-3 已停止运行! alert"
   },
   "startsAt": "2020-08-20T07:09:35.987923059Z",
   "endsAt": "2020-08-20T07:14:05.987923059Z",
   "generatorURL": "http://test-1:9090/graph?g0.expr=up%7Bjob%3D%22centos-3%22%7D+%3D%3D+0\u0026g0.tab=1",
   "fingerprint": "d0412b7cebb1a039"
 }],
 "groupLabels": {
   "alertname": "node-up"
 },
 "commonLabels": {
   "alertname": "node-up",
   "instance": "node-3",
   "job": "centos-3",
   "severity": "1",
   "team": "node"
 },
 "commonAnnotations": {
   "description": "node-3 检测到异常停止！请重点关注！！！",
   "summary": "node-3 已停止运行! alert"
 },
 "externalURL": "http://test-1:9093",
 "version": "4",
 "groupKey": "{}:{alertname=\"node-up\"}",
 "truncatedAlerts": 0
}
```



## 二、告警规则



### 1.Prometheus

```yaml
# 文件
prometheus.yml 
 
groups:
- name: Prometheus   #报警规则组的名字
  rules:
  - alert: PrometheusTargetMissing
    expr: up == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Prometheus target missing (instance {{ $labels.instance }})"
      description: "A Prometheus target has disappeared. An exporter might be crashed.\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
  - alert: PrometheusAllTargetsMissing
    expr: count by (job) (up) == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Prometheus all targets missing (instance {{ $labels.instance }})"
      description: "A Prometheus job does not have living target anymore.\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
  - alert: PrometheusNotConnectedToAlertmanager
    expr: prometheus_notifications_alertmanagers_discovered < 1
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Prometheus not connected to alertmanager (instance {{ $labels.instance }})"
      description: "Prometheus cannot connect the alertmanager\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
  
  - alert: PrometheusAlertmanagerNotificationFailing
    expr: rate(alertmanager_notifications_failed_total[1m]) > 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Prometheus AlertManager notification failing (instance {{ $labels.instance }})"
      description: "Alertmanager is failing sending notifications\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```



### 2.Linux

```yaml
# 文件
node-exporter.yml 
 
groups:
- name: CentOS   #报警规则组的名字
  rules:
  - alert: HostOutOfMemory
    expr: node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100 < 10
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host out of memory (instance {{ $labels.instance }})"
      description: "Node memory is filling up (< 10% left)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: HostMemoryUnderMemoryPressure
    expr: rate(node_vmstat_pgmajfault[1m]) > 1000
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host memory under memory pressure (instance {{ $labels.instance }})"
      description: "The node is under heavy memory pressure. High rate of major page faults\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: HostUnusualNetworkThroughputIn
    expr: sum by (instance) (irate(node_network_receive_bytes_total[2m])) / 1024 / 1024 > 100
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host unusual network throughput in (instance {{ $labels.instance }})"
      description: "Host network interfaces are probably receiving too much data (> 100 MB/s)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: HostUnusualNetworkThroughputOut
    expr: sum by (instance) (irate(node_network_transmit_bytes_total[2m])) / 1024 / 1024 > 100
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host unusual network throughput out (instance {{ $labels.instance }})"
      description: "Host network interfaces are probably sending too much data (> 100 MB/s)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: HostUnusualDiskReadRate
    expr: sum by (instance) (irate(node_disk_read_bytes_total[2m])) / 1024 / 1024 > 50
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host unusual disk read rate (instance {{ $labels.instance }})"
      description: "Disk is probably reading too much data (> 50 MB/s)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: HostUnusualDiskWriteRate
    expr: sum by (instance) (irate(node_disk_written_bytes_total[2m])) / 1024 / 1024 > 50
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host unusual disk write rate (instance {{ $labels.instance }})"
      description: "Disk is probably writing too much data (> 50 MB/s)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: HostOutOfDiskSpace
    expr: (node_filesystem_avail_bytes{mountpoint="/rootfs"}  * 100) / node_filesystem_size_bytes{mountpoint="/rootfs"} < 10
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host out of disk space (instance {{ $labels.instance }})"
      description: "Disk is almost full (< 10% left)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: HostDiskWillFillIn4Hours
    expr: predict_linear(node_filesystem_free_bytes{fstype!~"tmpfs"}[1h], 4 * 3600) < 0
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host disk will fill in 4 hours (instance {{ $labels.instance }})"
      description: "Disk will fill in 4 hours at current write rate\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: HostOutOfInodes
    expr: node_filesystem_files_free{mountpoint ="/rootfs"} / node_filesystem_files{mountpoint ="/rootfs"} * 100 < 10
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host out of inodes (instance {{ $labels.instance }})"
      description: "Disk is almost running out of available inodes (< 10% left)\n  VALUE = {{ $value }}\n LABELS: {{ $labels }}"
 
   
  - alert: HostUnusualDiskReadLatency
    expr: rate(node_disk_read_time_seconds_total[1m]) / rate(node_disk_reads_completed_total[1m]) > 100
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host unusual disk read latency (instance {{ $labels.instance }})"
      description: "Disk latency is growing (read operations > 100ms)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: HostUnusualDiskWriteLatency
    expr: rate(node_disk_write_time_seconds_total[1m]) / rate(node_disk_writes_completed_total[1m]) > 100
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host unusual disk write latency (instance {{ $labels.instance }})"
      description: "Disk latency is growing (write operations > 100ms)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: HostHighCpuLoad
    expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host high CPU load (instance {{ $labels.instance }})"
      description: "CPU load is > 80%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  # 1000 context switches is an arbitrary number.
  # Alert threshold depends on nature of application.
  # Please read: https://github.com/samber/awesome-prometheus-alerts/issues/58
  - alert: HostContextSwitching
    expr: (rate(node_context_switches_total[5m])) / (count without(cpu, mode) (node_cpu_seconds_total{mode="idle"})) > 1000
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host context switching (instance {{ $labels.instance }})"
      description: "Context switching is growing on node (> 1000 / s)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: HostSwapIsFillingUp
    expr: (1 - (node_memory_SwapFree_bytes / node_memory_SwapTotal_bytes)) * 100 > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host swap is filling up (instance {{ $labels.instance }})"
      description: "Swap is filling up (>80%)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: HostSystemdServiceCrashed
    expr: node_systemd_unit_state{state="failed"} == 1
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host SystemD service crashed (instance {{ $labels.instance }})"
      description: "SystemD service crashed\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: HostPhysicalComponentTooHot
    expr: node_hwmon_temp_celsius > 75
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host physical component too hot (instance {{ $labels.instance }})"
      description: "Physical hardware component too hot\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: HostNodeOvertemperatureAlarm
    expr: node_hwmon_temp_alarm == 1
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Host node overtemperature alarm (instance {{ $labels.instance }})"
      description: "Physical node temperature alarm triggered\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: HostRaidArrayGotInactive
    expr: node_md_state{state="inactive"} > 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Host RAID array got inactive (instance {{ $labels.instance }})"
      description: "RAID array {{ $labels.device }} is in degraded state due to one or more disks failures. Number of spare drives is insufficient to fix issue automatically.\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: HostRaidDiskFailure
    expr: node_md_disks{state="fail"} > 0
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host RAID disk failure (instance {{ $labels.instance }})"
      description: "At least one device in RAID array on {{ $labels.instance }} failed. Array {{ $labels.md_device }} needs attention and possibly a disk swap\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: HostKernelVersionDeviations
    expr: count(sum(label_replace(node_uname_info, "kernel", "$1", "release", "([0-9]+.[0-9]+.[0-9]+).*")) by (kernel)) > 1
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host kernel version deviations (instance {{ $labels.instance }})"
      description: "Different kernel versions are running\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: HostOomKillDetected
    expr: increase(node_vmstat_oom_kill[5m]) > 0
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host OOM kill detected (instance {{ $labels.instance }})"
      description: "OOM kill detected\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: HostEdacCorrectableErrorsDetected
    expr: increase(node_edac_correctable_errors_total[5m]) > 0
    for: 5m
    labels:
      severity: info
    annotations:
      summary: "Host EDAC Correctable Errors detected (instance {{ $labels.instance }})"
      description: "{{ $labels.instance }} has had {{ printf "%.0f" $value }} correctable memory errors reported by EDAC in the last 5 minutes.\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: HostEdacUncorrectableErrorsDetected
    expr: node_edac_uncorrectable_errors_total > 0
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host EDAC Uncorrectable Errors detected (instance {{ $labels.instance }})"
      description: "{{ $labels.instance }} has had {{ printf "%.0f" $value }} uncorrectable memory errors reported by EDAC in the last 5 minutes.\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: HostNetworkReceiveErrors
    expr: increase(node_network_receive_errs_total[5m]) > 0
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host Network Receive Errors (instance {{ $labels.instance }})"
      description: "{{ $labels.instance }} interface {{ $labels.device }} has encountered {{ printf "%.0f" $value }} receive errors in the last five minutes.\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: HostNetworkTransmitErrors
    expr: increase(node_network_transmit_errs_total[5m]) > 0
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host Network Transmit Errors (instance {{ $labels.instance }})"
      description: "{{ $labels.instance }} interface {{ $labels.device }} has encountered {{ printf "%.0f" $value }} transmit errors in the last five minutes.\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```



### 3.Docker

```yaml
# 文件
cadvisor.yml 
 
groups:
- name: Docker   #报警规则组的名字
  rules:
  - alert: ContainerKilled
    expr: time() - container_last_seen > 60
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Container killed (instance {{ $labels.instance }})"
      description: "A container has disappeared\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: ContainerCpuUsage
    expr: (sum(rate(container_cpu_usage_seconds_total[3m])) BY (instance, name) * 100) > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Container CPU usage (instance {{ $labels.instance }})"
      description: "Container CPU usage is above 80%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: ContainerMemoryUsage
    expr: (sum(container_memory_usage_bytes) BY (instance, name) / sum(container_spec_memory_limit_bytes) BY (instance, name) * 100) > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Container Memory usage (instance {{ $labels.instance }})"
      description: "Container Memory usage is above 80%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: ContainerVolumeUsage
    expr: (1 - (sum(container_fs_inodes_free) BY (instance) / sum(container_fs_inodes_total) BY (instance)) * 100) > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Container Volume usage (instance {{ $labels.instance }})"
      description: "Container Volume usage is above 80%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: ContainerVolumeIoUsage
    expr: (sum(container_fs_io_current) BY (instance, name) * 100) > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Container Volume IO usage (instance {{ $labels.instance }})"
      description: "Container Volume IO usage is above 80%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: ContainerHighThrottleRate
    expr: rate(container_cpu_cfs_throttled_seconds_total[3m]) > 1
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Container high throttle rate (instance {{ $labels.instance }})"
      description: "Container is being throttled\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```



### 4.Nginx

```yaml
# 文件
nginx-vts-exporter.yml 
 
groups:
- name: Nginx   #报警规则组的名字
  rules:
  - alert: NginxHighHttp4xxErrorRate
    expr: sum(rate(nginx_http_requests_total{status=~"^4.."}[1m])) / sum(rate(nginx_http_requests_total[1m])) * 100 > 5
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Nginx high HTTP 4xx error rate (instance {{ $labels.instance }})"
      description: "Too many HTTP requests with status 4xx (> 5%)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: NginxHighHttp5xxErrorRate
    expr: sum(rate(nginx_http_requests_total{status=~"^5.."}[1m])) / sum(rate(nginx_http_requests_total[1m])) * 100 > 5
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Nginx high HTTP 5xx error rate (instance {{ $labels.instance }})"
      description: "Too many HTTP requests with status 5xx (> 5%)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: NginxLatencyHigh
    expr: histogram_quantile(0.99, sum(rate(nginx_http_request_duration_seconds_bucket[30m])) by (host, node)) > 10
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Nginx latency high (instance {{ $labels.instance }})"
      description: "Nginx p99 latency is higher than 10 seconds\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```



### 5.Redis

```yaml
# 文件
redis-exporter.yml 
 
groups:
- name: Redis   #报警规则组的名字
  rules:
  - alert: RedisDown
    expr: redis_up == 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Redis down (instance {{ $labels.instance }})"
      description: "Redis instance is down\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: RedisMissingMaster
    expr: count(redis_instance_info{role="master"}) == 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Redis missing master (instance {{ $labels.instance }})"
      description: "Redis cluster has no node marked as master.\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: RedisTooManyMasters
    expr: count(redis_instance_info{role="master"}) > 1
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Redis too many masters (instance {{ $labels.instance }})"
      description: "Redis cluster has too many nodes marked as master.\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: RedisDisconnectedSlaves
    expr: count without (instance, job) (redis_connected_slaves) - sum without (instance, job) (redis_connected_slaves) - 1 > 1
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Redis disconnected slaves (instance {{ $labels.instance }})"
      description: "Redis not replicating for all slaves. Consider reviewing the redis replication status.\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: RedisReplicationBroken
    expr: delta(redis_connected_slaves[1m]) < 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Redis replication broken (instance {{ $labels.instance }})"
      description: "Redis instance lost a slave\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: RedisClusterFlapping
    expr: changes(redis_connected_slaves[5m]) > 2
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Redis cluster flapping (instance {{ $labels.instance }})"
      description: "Changes have been detected in Redis replica connection. This can occur when replica nodes lose connection to the master and reconnect (a.k.a flapping).\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: RedisMissingBackup
    expr: time() - redis_rdb_last_save_timestamp_seconds > 60 * 60 * 24
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Redis missing backup (instance {{ $labels.instance }})"
      description: "Redis has not been backuped for 24 hours\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: RedisOutOfMemory
    expr: redis_memory_used_bytes / redis_total_system_memory_bytes * 100 > 90
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Redis out of memory (instance {{ $labels.instance }})"
      description: "Redis is running out of memory (> 90%)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: RedisTooManyConnections
    expr: redis_connected_clients > 100
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Redis too many connections (instance {{ $labels.instance }})"
      description: "Redis instance has too many connections\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: RedisNotEnoughConnections
    expr: redis_connected_clients < 5
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Redis not enough connections (instance {{ $labels.instance }})"
      description: "Redis instance should have more connections (> 5)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: RedisRejectedConnections
    expr: increase(redis_rejected_connections_total[1m]) > 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Redis rejected connections (instance {{ $labels.instance }})"
      description: "Some connections to Redis has been rejected\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```



### 6.PostgreSQL

```yaml
# 文件
postgres-exporter.yml 
 
groups:
- name: PostgreSQL   #报警规则组的名字
  rules:
  - alert: PostgresqlDown
    expr: pg_up == 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Postgresql down (instance {{ $labels.instance }})"
      description: "Postgresql instance is down\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: PostgresqlRestarted
    expr: time() - pg_postmaster_start_time_seconds < 60
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Postgresql restarted (instance {{ $labels.instance }})"
      description: "Postgresql restarted\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: PostgresqlExporterError
    expr: pg_exporter_last_scrape_error > 0
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Postgresql exporter error (instance {{ $labels.instance }})"
      description: "Postgresql exporter is showing errors. A query may be buggy in query.yaml\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: PostgresqlReplicationLag
    expr: (pg_replication_lag) > 10 and ON(instance) (pg_replication_is_replica == 1)
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Postgresql replication lag (instance {{ $labels.instance }})"
      description: "PostgreSQL replication lag is going up (> 10s)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: PostgresqlTableNotVaccumed
    expr: time() - pg_stat_user_tables_last_autovacuum > 60 * 60 * 24
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Postgresql table not vaccumed (instance {{ $labels.instance }})"
      description: "Table has not been vaccum for 24 hours\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: PostgresqlTableNotAnalyzed
    expr: time() - pg_stat_user_tables_last_autoanalyze > 60 * 60 * 24
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Postgresql table not analyzed (instance {{ $labels.instance }})"
      description: "Table has not been analyzed for 24 hours\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: PostgresqlTooManyConnections
    expr: sum by (datname) (pg_stat_activity_count{datname!~"template.*|postgres"}) > pg_settings_max_connections * 0.9
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Postgresql too many connections (instance {{ $labels.instance }})"
      description: "PostgreSQL instance has too many connections\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: PostgresqlNotEnoughConnections
    expr: sum by (datname) (pg_stat_activity_count{datname!~"template.*|postgres"}) < 5
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Postgresql not enough connections (instance {{ $labels.instance }})"
      description: "PostgreSQL instance should have more connections (> 5)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: PostgresqlDeadLocks
    expr: rate(pg_stat_database_deadlocks{datname!~"template.*|postgres"}[1m]) > 0
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Postgresql dead locks (instance {{ $labels.instance }})"
      description: "PostgreSQL has dead-locks\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: PostgresqlSlowQueries
    expr: pg_slow_queries > 0
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Postgresql slow queries (instance {{ $labels.instance }})"
      description: "PostgreSQL executes slow queries\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: PostgresqlHighRollbackRate
    expr: rate(pg_stat_database_xact_rollback{datname!~"template.*"}[3m]) / rate(pg_stat_database_xact_commit{datname!~"template.*"}[3m]) > 0.02
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Postgresql high rollback rate (instance {{ $labels.instance }})"
      description: "Ratio of transactions being aborted compared to committed is > 2 %\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: PostgresqlCommitRateLow
    expr: rate(pg_stat_database_xact_commit[1m]) < 10
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Postgresql commit rate low (instance {{ $labels.instance }})"
      description: "Postgres seems to be processing very few transactions\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: PostgresqlLowXidConsumption
    expr: rate(pg_txid_current[1m]) < 5
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Postgresql low XID consumption (instance {{ $labels.instance }})"
      description: "Postgresql seems to be consuming transaction IDs very slowly\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: PostgresqllowXlogConsumption
    expr: rate(pg_xlog_position_bytes[1m]) < 100
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Postgresqllow XLOG consumption (instance {{ $labels.instance }})"
      description: "Postgres seems to be consuming XLOG very slowly\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: PostgresqlWaleReplicationStopped
    expr: rate(pg_xlog_position_bytes[1m]) == 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Postgresql WALE replication stopped (instance {{ $labels.instance }})"
      description: "WAL-E replication seems to be stopped\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: PostgresqlHighRateStatementTimeout
    expr: rate(postgresql_errors_total{type="statement_timeout"}[5m]) > 3
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Postgresql high rate statement timeout (instance {{ $labels.instance }})"
      description: "Postgres transactions showing high rate of statement timeouts\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: PostgresqlHighRateDeadlock
    expr: rate(postgresql_errors_total{type="deadlock_detected"}[1m]) * 60 > 1
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Postgresql high rate deadlock (instance {{ $labels.instance }})"
      description: "Postgres detected deadlocks\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: PostgresqlReplicationLabBytes
    expr: (pg_xlog_position_bytes and pg_replication_is_replica == 0) - GROUP_RIGHT(instance) (pg_xlog_position_bytes and pg_replication_is_replica == 1) > 1e+09
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Postgresql replication lab bytes (instance {{ $labels.instance }})"
      description: "Postgres Replication lag (in bytes) is high\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: PostgresqlUnusedReplicationSlot
    expr: pg_replication_slots_active == 0
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Postgresql unused replication slot (instance {{ $labels.instance }})"
      description: "Unused Replication Slots\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: PostgresqlTooManyDeadTuples
    expr: ((pg_stat_user_tables_n_dead_tup > 10000) / (pg_stat_user_tables_n_live_tup + pg_stat_user_tables_n_dead_tup)) >= 0.1 unless ON(instance) (pg_replication_is_replica == 1)
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Postgresql too many dead tuples (instance {{ $labels.instance }})"
      description: "PostgreSQL dead tuples is too large\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: PostgresqlSplitBrain
    expr: count(pg_replication_is_replica == 0) != 1
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Postgresql split brain (instance {{ $labels.instance }})"
      description: "Split Brain, too many primary Postgresql databases in read-write mode\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: PostgresqlPromotedNode
    expr: pg_replication_is_replica and changes(pg_replication_is_replica[1m]) > 0
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Postgresql promoted node (instance {{ $labels.instance }})"
      description: "Postgresql standby server has been promoted as primary node\n  VALUE = {{ $value }}\n LABELS: {{ $labels }}"
 
   
  - alert: PostgresqlConfigurationChanged
    expr: {__name__=~"pg_settings_.*"} != ON(__name__) {__name__=~"pg_settings_([^t]|t[^r]|tr[^a]|tra[^n]|tran[^s]|trans[^a]|transa[^c]|transac[^t]|transact[^i]|transacti[^o]|transactio[^n]|transaction[^_]|transaction_[^r]|transaction_r[^e]|transaction_re[^a]|transaction_rea[^d]|transaction_read[^_]|transaction_read_[^o]|transaction_read_o[^n]|transaction_read_on[^l]|transaction_read_onl[^y]).*"} OFFSET 5m
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Postgresql configuration changed (instance {{ $labels.instance }})"
      description: "Postgres Database configuration change has occurred\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: PostgresqlSslCompressionActive
    expr: sum(pg_stat_ssl_compression) > 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Postgresql SSL compression active (instance {{ $labels.instance }})"
      description: "Database connections with SSL compression enabled. This may add significant jitter in replication delay. Replicas should turn off SSL compression via `sslcompression=0` in `recovery.conf`.\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: PostgresqlTooManyLocksAcquired
    expr: ((sum (pg_locks_count)) / (pg_settings_max_locks_per_transaction * pg_settings_max_connections)) > 0.20
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Postgresql too many locks acquired (instance {{ $labels.instance }})"
      description: "Too many locks acquired on the database. If this alert happens frequently, we may need to increase the postgres setting max_locks_per_transaction.\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```



### 7.MySQL

```yaml
# 文件
mysqld-exporter.yml 
 
groups:
- name: MySQL   #报警规则组的名字
  rules:
  - alert: MysqlDown
    expr: mysql_up == 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "MySQL down (instance {{ $labels.instance }})"
      description: "MySQL instance is down on {{ $labels.instance }}\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
 
  - alert: MysqlTooManyConnections
    expr: avg by (instance) (max_over_time(mysql_global_status_threads_connected[5m])) / avg by (instance) (mysql_global_variables_max_connections) * 100 > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "MySQL too many connections (instance {{ $labels.instance }})"
      description: "More than 80% of MySQL connections are in use on {{ $labels.instance }}\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
 
  - alert: MysqlHighThreadsRunning
    expr: avg by (instance) (max_over_time(mysql_global_status_threads_running[5m])) / avg by (instance) (mysql_global_variables_max_connections) * 100 > 60
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "MySQL high threads running (instance {{ $labels.instance }})"
      description: "More than 60% of MySQL connections are in running state on {{ $labels.instance }}\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
 
  - alert: MysqlSlaveIoThreadNotRunning
    expr: mysql_slave_status_master_server_id > 0 and ON (instance) mysql_slave_status_slave_io_running == 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "MySQL Slave IO thread not running (instance {{ $labels.instance }})"
      description: "MySQL Slave IO thread not running on {{ $labels.instance }}\n  VALUE = {{ $value }}\n LABELS: {{ $labels }}"
 
 
  - alert: MysqlSlaveSqlThreadNotRunning
    expr: mysql_slave_status_master_server_id > 0 and ON (instance) mysql_slave_status_slave_sql_running == 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "MySQL Slave SQL thread not running (instance {{ $labels.instance }})"
      description: "MySQL Slave SQL thread not running on {{ $labels.instance }}\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
 
  - alert: MysqlSlaveReplicationLag
    expr: mysql_slave_status_master_server_id > 0 and ON (instance) (mysql_slave_status_seconds_behind_master - mysql_slave_status_sql_delay) > 300
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "MySQL Slave replication lag (instance {{ $labels.instance }})"
      description: "MysqL replication lag on {{ $labels.instance }}\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
 
  - alert: MysqlSlowQueries
    expr: mysql_global_status_slow_queries > 0
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "MySQL slow queries (instance {{ $labels.instance }})"
      description: "MySQL server is having some slow queries.\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
 
  - alert: MysqlRestarted
    expr: mysql_global_status_uptime < 60
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "MySQL restarted (instance {{ $labels.instance }})"
      description: "MySQL has just been restarted, less than one minute ago on {{ $labels.instance }}.\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```



### 8.RabbitMQ

```yaml
# 文件
rabbitmq-exporter.yml 
 
groups:
- name: Rabbitmq   #报警规则组的名字
  rules:
  - alert: RabbitmqDown
    expr: rabbitmq_up == 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Rabbitmq down (instance {{ $labels.instance }})"
      description: "RabbitMQ node down\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
 
  - alert: RabbitmqClusterDown
    expr: sum(rabbitmq_running) < 3
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Rabbitmq cluster down (instance {{ $labels.instance }})"
      description: "Less than 3 nodes running in RabbitMQ cluster\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
 
  - alert: RabbitmqClusterPartition
    expr: rabbitmq_partitions > 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Rabbitmq cluster partition (instance {{ $labels.instance }})"
      description: "Cluster partition\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
 
  - alert: RabbitmqOutOfMemory
    expr: rabbitmq_node_mem_used / rabbitmq_node_mem_limit * 100 > 90
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Rabbitmq out of memory (instance {{ $labels.instance }})"
      description: "Memory available for RabbmitMQ is low (< 10%)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
 
  - alert: RabbitmqTooManyConnections
    expr: rabbitmq_connectionsTotal > 1000
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Rabbitmq too many connections (instance {{ $labels.instance }})"
      description: "RabbitMQ instance has too many connections (> 1000)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
 
  - alert: RabbitmqDeadLetterQueueFillingUp
    expr: rabbitmq_queue_messages{queue="my-dead-letter-queue"} > 10
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Rabbitmq dead letter queue filling up (instance {{ $labels.instance }})"
      description: "Dead letter queue is filling up (> 10 msgs)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
 
  - alert: RabbitmqTooManyMessagesInQueue
    expr: rabbitmq_queue_messages_ready{queue="my-queue"} > 1000
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Rabbitmq too many messages in queue (instance {{ $labels.instance }})"
      description: "Queue is filling up (> 1000 msgs)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
 
  - alert: RabbitmqSlowQueueConsuming
    expr: time() - rabbitmq_queue_head_message_timestamp{queue="my-queue"} > 60
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Rabbitmq slow queue consuming (instance {{ $labels.instance }})"
      description: "Queue messages are consumed slowly (> 60s)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
 
  - alert: RabbitmqNoConsumer
    expr: rabbitmq_queue_consumers == 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Rabbitmq no consumer (instance {{ $labels.instance }})"
      description: "Queue has no consumer\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
 
  - alert: RabbitmqTooManyConsumers
    expr: rabbitmq_queue_consumers > 1
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Rabbitmq too many consumers (instance {{ $labels.instance }})"
      description: "Queue should have only 1 consumer\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
 
 
  - alert: RabbitmqUnactiveExchange
    expr: rate(rabbitmq_exchange_messages_published_in_total{exchange="my-exchange"}[1m]) < 5
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Rabbitmq unactive exchange (instance {{ $labels.instance }})"
      description: "Exchange receive less than 5 msgs per second\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```



### 9.JVM

```yaml
# 文件
jvm.yml 
 
groups:
- name: JVM   #报警规则组的名字
  rules:
  - alert: JvmMemoryFillingUp
    expr: jvm_memory_bytes_used / jvm_memory_bytes_max{area="heap"} > 0.8
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "JVM memory filling up (instance {{ $labels.instance }})"
      description: "JVM memory is filling up (> 80%)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}" 
```



### 10.Elasticsearch

```yaml
# 文件
elasticsearch-exporter.yml 
 
groups:
- name: Elasticsearch   #报警规则组的名字
  rules:
  - alert: ElasticsearchHeapUsageTooHigh
    expr: (elasticsearch_jvm_memory_used_bytes{area="heap"} / elasticsearch_jvm_memory_max_bytes{area="heap"}) * 100 > 90
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Elasticsearch Heap Usage Too High (instance {{ $labels.instance }})"
      description: "The heap usage is over 90% for 5m\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: ElasticsearchHeapUsageWarning
    expr: (elasticsearch_jvm_memory_used_bytes{area="heap"} / elasticsearch_jvm_memory_max_bytes{area="heap"}) * 100 > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Elasticsearch Heap Usage warning (instance {{ $labels.instance }})"
      description: "The heap usage is over 80% for 5m\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: ElasticsearchDiskSpaceLow
    expr: elasticsearch_filesystem_data_available_bytes / elasticsearch_filesystem_data_size_bytes * 100 < 20
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Elasticsearch disk space low (instance {{ $labels.instance }})"
      description: "The disk usage is over 80%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: ElasticsearchDiskOutOfSpace
    expr: elasticsearch_filesystem_data_available_bytes / elasticsearch_filesystem_data_size_bytes * 100 < 10
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Elasticsearch disk out of space (instance {{ $labels.instance }})"
      description: "The disk usage is over 90%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: ElasticsearchClusterRed
    expr: elasticsearch_cluster_health_status{color="red"} == 1
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Elasticsearch Cluster Red (instance {{ $labels.instance }})"
      description: "Elastic Cluster Red status\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: ElasticsearchClusterYellow
    expr: elasticsearch_cluster_health_status{color="yellow"} == 1
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Elasticsearch Cluster Yellow (instance {{ $labels.instance }})"
      description: "Elastic Cluster Yellow status\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: ElasticsearchHealthyNodes
    expr: elasticsearch_cluster_health_number_of_nodes < number_of_nodes
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Elasticsearch Healthy Nodes (instance {{ $labels.instance }})"
      description: "Number Healthy Nodes less then number_of_nodes\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: ElasticsearchHealthyDataNodes
    expr: elasticsearch_cluster_health_number_of_data_nodes < number_of_data_nodes
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Elasticsearch Healthy Data Nodes (instance {{ $labels.instance }})"
      description: "Number Healthy Data Nodes less then number_of_data_nodes\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
  - alert: ElasticsearchRelocationShards
    expr: elasticsearch_cluster_health_relocating_shards > 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Elasticsearch relocation shards (instance {{ $labels.instance }})"
      description: "Number of relocation shards for 20 min\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: ElasticsearchInitializingShards
    expr: elasticsearch_cluster_health_initializing_shards > 0
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Elasticsearch initializing shards (instance {{ $labels.instance }})"
      description: "Number of initializing shards for 10 min\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: ElasticsearchUnassignedShards
    expr: elasticsearch_cluster_health_unassigned_shards > 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Elasticsearch unassigned shards (instance {{ $labels.instance }})"
      description: "Number of unassigned shards for 2 min\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: ElasticsearchPendingTasks
    expr: elasticsearch_cluster_health_number_of_pending_tasks > 0
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Elasticsearch pending tasks (instance {{ $labels.instance }})"
      description: "Number of pending tasks for 10 min. Cluster works slowly.\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
 
   
  - alert: ElasticsearchNoNewDocuments
    expr: rate(elasticsearch_indices_docs{es_data_node="true"}[10m]) < 1
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Elasticsearch no new documents (instance {{ $labels.instance }})"
      description: "No new documents for 10 min!\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```



