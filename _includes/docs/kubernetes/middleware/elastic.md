* TOC
{:toc}


## 一、概述



### 1.Operator

 ```shell
 # awesome-operators
 https://github.com/operator-framework/awesome-operators
 
 
 # 官方
 Elasticsearch #1 (Official)	elastic/cloud-on-k8s
 https://github.com/elastic/cloud-on-k8s
 ```



| App Name                    | Github                                                       | Description                                                  |
| --------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Elasticsearch #1 (Official) | [elastic/cloud-on-k8s](https://github.com/elastic/cloud-on-k8s) | Elastic Cloud on Kubernetes (ECK) is the official Elastic Operator to deploy, provision, manage and orchestrate secured Elasticsearch clusters and Kibana on Kubernetes. |
| Elasticsearch #2            | [upmc-enterprises/elasticsearch-operator](https://github.com/upmc-enterprises/elasticsearch-operator) | Manages one or more elastic search clusters on Kubernetes.   |
| Elasticsearch #3            | [jetstack/navigator](https://github.com/jetstack/navigator)  | Create, scale and upgrade multi-AZ Elasticsearch clusters on Kubernetes |
| Elasticsearch #4            | [kudobuilder/operators/elastic](https://github.com/kudobuilder/operators/tree/master/repository/elastic) | An Operator for ElasticSearch built using [KUDO](https://kudo.dev/) |



### 2.Helm

**Bitnami**

```shell
https://bitnami.com/
https://github.com/bitnami
https://bitnami.com/stacks
```



```shell
[root@k8s-master elasticsearch]# helm search repo elasticsearch
NAME                         	CHART VERSION	APP VERSION	DESCRIPTION                                       
aliyun/elasticsearch-exporter	0.1.2        	1.0.2      	Elasticsearch stats exporter for Prometheus       
bitnami/elasticsearch        	17.3.3       	7.15.2     	A highly scalable open-source full-text search ...
elastic/elasticsearch        	7.15.0       	7.15.0     	Official Elastic helm chart for Elasticsearch     
gitlab/fluentd-elasticsearch 	6.2.8        	2.8.0      	A Fluentd Helm chart for Kubernetes with Elasti...
aliyun/elastalert            	0.1.1        	0.1.21     	ElastAlert is a simple framework for alerting o...
aliyun/kibana                	0.2.2        	6.0.0      	Kibana is an open source data visualization plu...
bitnami/dataplatform-bp2     	10.0.0       	1.0.1      	OCTO Data platform Kafka-Spark-Elasticsearch He...
bitnami/grafana              	7.2.5        	8.2.5      	Grafana is an open source, feature rich metrics...
bitnami/kibana               	9.1.3        	7.15.2     	Kibana is an open source, browser based analyti...
elastic/eck-operator         	1.8.0        	1.8.0      	A Helm chart for deploying the Elastic Cloud on...
elastic/eck-operator-crds    	1.8.0        	1.8.0      	A Helm chart for installing the ECK operator Cu...
```



**安装选择**

```shell
# elastic/elasticsearch
https://github.com/elastic/helm-charts
https://github.com/elastic/helm-charts/tree/main/elasticsearch
https://github.com/elastic/helm-charts/tree/main/kibana
https://github.com/elastic/helm-charts/tree/main/logstash
https://github.com/elastic/helm-charts/tree/main/filebeat


[root@k8s-master helm]# helm search repo elasticsearch
NAME                         	CHART VERSION	APP VERSION	DESCRIPTION                                     
elastic/elasticsearch        	7.15.0       	7.15.0     	Official Elastic helm chart for Elasticsearch 



helm repo add elastic https://helm.elastic.co
```





## 二、基础



### 1.Helm

#### 1.1.Elasticsearch

```shell
# helm-charts/elasticsearch/
https://github.com/elastic/helm-charts
https://github.com/elastic/helm-charts/tree/main/elasticsearch


# Helm
[root@k8s-master helm]# helm search repo elasticsearch
NAME                         	CHART VERSION	APP VERSION	DESCRIPTION                                     
elastic/elasticsearch        	7.15.0       	7.15.0     	Official Elastic helm chart for Elasticsearch  
```



#### 1.2.Kibana

```shell
# helm-charts/kibana/
https://github.com/elastic/helm-charts
https://github.com/elastic/helm-charts/tree/main/kibana


# Helm
[root@k8s-master helm]# helm search repo kibana
NAME                     	CHART VERSION	APP VERSION	DESCRIPTION                                       
elastic/kibana           	7.15.0       	7.15.0     	Official Elastic helm chart for Kibana            
```



#### 1.3.Logstash

```shell
# helm-charts/logstash/
https://github.com/elastic/helm-charts
https://github.com/elastic/helm-charts/tree/main/logstash


# Helm
[root@k8s-master helm]# helm search repo logstash
NAME                    	CHART VERSION	APP VERSION	DESCRIPTION                                       
elastic/logstash        	7.15.0       	7.15.0     	Official Elastic helm chart for Logstash          
```



#### 1.4.Filebeat

```shell
# helm-charts/filebeat/
https://github.com/elastic/helm-charts
https://github.com/elastic/helm-charts/tree/main/filebeat


# Helm
[root@k8s-master helm]# helm search repo filebeat
NAME            	CHART VERSION	APP VERSION	DESCRIPTION                             
elastic/filebeat	7.15.0       	7.15.0     	Official Elastic helm chart for Filebeat      
```



#### 1.5.删除Elastic

```shell
# 查看
[root@k8s-master filebeat]# helm list
NAME         	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART               	APP VERSION
elasticsearch	default  	1       	2021-12-02 16:53:28.462669857 +0800 CST	deployed	elasticsearch-7.15.0	7.15.0     
filebeat     	default  	1       	2021-12-03 10:13:39.284547467 +0800 CST	deployed	filebeat-7.15.0     	7.15.0     
kibana       	default  	1       	2021-12-02 17:13:31.455789893 +0800 CST	deployed	kibana-7.15.0       	7.15.0     
logstash     	default  	1       	2021-12-03 09:29:16.020275923 +0800 CST	deployed	logstash-7.15.0     	7.15.0
```

```shell
# 删除

helm delete elasticsearch
helm delete kibana
helm delete logstash
helm delete filebeat
```





## 三、实践



### 1.Elasticsearch

```shell
# helm-charts/elasticsearch/
https://github.com/elastic/helm-charts
https://github.com/elastic/helm-charts/tree/main/elasticsearch


# Helm
[root@k8s-master helm]# helm search repo elasticsearch
NAME                         	CHART VERSION	APP VERSION	DESCRIPTION                                     
elastic/elasticsearch        	7.15.0       	7.15.0     	Official Elastic helm chart for Elasticsearch  
```



#### 1.1.下载安装包

```shell
[root@k8s-master elasticsearch]# helm repo list
NAME         	URL                                                   
aliyun       	https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
bitnami      	https://charts.bitnami.com/bitnami                    
ingress-nginx	https://kubernetes.github.io/ingress-nginx            
gitlab       	https://charts.gitlab.io                              
elastic      	https://helm.elastic.co                               
harbor       	http://172.51.216.85:8888/chartrepo/charts            
chartmuseum  	http://172.51.216.85:9999                             
presslabs    	https://presslabs.github.io/charts


[root@k8s-master helm]# helm search repo elasticsearch
NAME                         	CHART VERSION	APP VERSION	DESCRIPTION                                       
aliyun/elasticsearch-exporter	0.1.2        	1.0.2      	Elasticsearch stats exporter for Prometheus       
bitnami/elasticsearch        	17.3.3       	7.15.2     	A highly scalable open-source full-text search ...
elastic/elasticsearch        	7.15.0       	7.15.0     	Official Elastic helm chart for Elasticsearch     
gitlab/fluentd-elasticsearch 	6.2.8        	2.8.0      	A Fluentd Helm chart for Kubernetes with Elasti...
aliyun/elastalert            	0.1.1        	0.1.21     	ElastAlert is a simple framework for alerting o...
aliyun/kibana                	0.2.2        	6.0.0      	Kibana is an open source data visualization plu...
bitnami/dataplatform-bp2     	10.0.0       	1.0.1      	OCTO Data platform Kafka-Spark-Elasticsearch He...
bitnami/grafana              	7.2.5        	8.2.5      	Grafana is an open source, feature rich metrics...
bitnami/kibana               	9.1.3        	7.15.2     	Kibana is an open source, browser based analyti...
elastic/eck-operator         	1.8.0        	1.8.0      	A Helm chart for deploying the Elastic Cloud on...
elastic/eck-operator-crds    	1.8.0        	1.8.0      	A Helm chart for installing the ECK operator Cu...


[root@k8s-master helm]# helm fetch elastic/elasticsearch
[root@k8s-master helm]# ll
total 28
-rw-r--r-- 1 root root 27115 Dec  2 15:23 elasticsearch-7.15.0.tgz

[root@k8s-master helm]# tar -zxf elasticsearch-7.15.0.tgz 
[root@k8s-master helm]# ll
total 28
drwxr-xr-x 4 root root   128 Dec  2 15:25 elasticsearch
-rw-r--r-- 1 root root 27115 Dec  2 15:23 elasticsearch-7.15.0.tgz
```



#### 1.2.修改配置

```yaml
[root@k8s-master elasticsearch]# vim values.yaml


# 修改
# Rook: rook-ceph-block
volumeClaimTemplate:
  accessModes: ["ReadWriteOnce"]
  storageClassName: "rook-ceph-block"
  resources:
    requests:
      storage: 20Gi
```



#### 1.3.安装Elasticsearch

```shell
[root@k8s-master helm]# helm install elasticsearch elasticsearch
NAME: elasticsearch
LAST DEPLOYED: Thu Dec  2 16:53:28 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Watch all cluster members come up.
  $ kubectl get pods --namespace=default -l app=elasticsearch-master -w2. Test cluster health using Helm test.
  $ helm --namespace=default test elasticsearch


# 查看
[root@k8s-master helm]# kubectl get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/elasticsearch-master-0   1/1     Running   0          98s
pod/elasticsearch-master-1   1/1     Running   0          98s
pod/elasticsearch-master-2   1/1     Running   0          98s

NAME                                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE
service/elasticsearch-master            ClusterIP   10.101.45.52   <none>        9200/TCP,9300/TCP   98s
service/elasticsearch-master-headless   ClusterIP   None           <none>        9200/TCP,9300/TCP   98s
service/kubernetes                      ClusterIP   10.96.0.1      <none>        443/TCP             107d

NAME                                    READY   AGE
statefulset.apps/elasticsearch-master   3/3     98s
[root@k8s-master helm]# kubectl get pvc
NAME                                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
elasticsearch-master-elasticsearch-master-0   Bound    pvc-ba621542-896c-4f32-9df7-402fae961301   20Gi       RWO            rook-ceph-block   104s
elasticsearch-master-elasticsearch-master-1   Bound    pvc-be76aab5-e10a-4fbe-b62f-8fe26114359b   20Gi       RWO            rook-ceph-block   104s
elasticsearch-master-elasticsearch-master-2   Bound    pvc-c5d77eb8-622f-4dd7-8931-095197c86ead   20Gi       RWO            rook-ceph-block   104s


[root@k8s-master helm]# curl 10.101.45.52:9200
{
  "name" : "elasticsearch-master-0",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "_ez6gtEpRkGK2Vqh1RqZdg",
  "version" : {
    "number" : "7.15.0",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "79d65f6e357953a5b3cbcc5e2c7c21073d89aa29",
    "build_date" : "2021-09-16T03:05:29.143308416Z",
    "build_snapshot" : false,
    "lucene_version" : "8.9.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}


[root@k8s-master helm]# helm --namespace=default test elasticsearch
NAME: elasticsearch
LAST DEPLOYED: Thu Dec  2 16:53:28 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE:     elasticsearch-flgri-test
Last Started:   Thu Dec  2 16:56:33 2021
Last Completed: Thu Dec  2 16:56:34 2021
Phase:          Succeeded
NOTES:
1. Watch all cluster members come up.
  $ kubectl get pods --namespace=default -l app=elasticsearch-master -w2. Test cluster health using Helm test.
  $ helm --namespace=default test elasticsearch
```



#### 1.4.测试

```shell
[root@k8s-master helm]# kubectl get pod -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP               NODE        NOMINATED NODE   READINESS GATES
elasticsearch-master-0   1/1     Running   0          4m13s   10.244.36.113    k8s-node1   <none>           <none>
elasticsearch-master-1   1/1     Running   0          4m13s   10.244.169.177   k8s-node2   <none>           <none>
elasticsearch-master-2   1/1     Running   0          4m13s   10.244.107.198   k8s-node3   <none>           <none>


[root@k8s-master helm]# kubectl get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/elasticsearch-master-0   1/1     Running   0          4m31s
pod/elasticsearch-master-1   1/1     Running   0          4m31s
pod/elasticsearch-master-2   1/1     Running   0          4m31s

NAME                                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE
service/elasticsearch-master            ClusterIP   10.101.45.52   <none>        9200/TCP,9300/TCP   4m31s
service/elasticsearch-master-headless   ClusterIP   None           <none>        9200/TCP,9300/TCP   4m31s
service/kubernetes                      ClusterIP   10.96.0.1      <none>        443/TCP             107d

NAME                                    READY   AGE
statefulset.apps/elasticsearch-master   3/3     4m31s


[root@k8s-master helm]# curl 10.101.45.52:9200
{
  "name" : "elasticsearch-master-0",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "_ez6gtEpRkGK2Vqh1RqZdg",
  "version" : {
    "number" : "7.15.0",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "79d65f6e357953a5b3cbcc5e2c7c21073d89aa29",
    "build_date" : "2021-09-16T03:05:29.143308416Z",
    "build_snapshot" : false,
    "lucene_version" : "8.9.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```



### 2.Kibana

```shell
# helm-charts/kibana/
https://github.com/elastic/helm-charts
https://github.com/elastic/helm-charts/tree/main/kibana


# Helm
[root@k8s-master helm]# helm search repo kibana
NAME                     	CHART VERSION	APP VERSION	DESCRIPTION                                       
elastic/kibana           	7.15.0       	7.15.0     	Official Elastic helm chart for Kibana   
```



#### 2.1.下载安装包

```shell
[root@k8s-master helm]# helm search repo kibana
NAME                     	CHART VERSION	APP VERSION	DESCRIPTION                                       
aliyun/kibana            	0.2.2        	6.0.0      	Kibana is an open source data visualization plu...
bitnami/kibana           	9.1.3        	7.15.2     	Kibana is an open source, browser based analyti...
elastic/kibana           	7.15.0       	7.15.0     	Official Elastic helm chart for Kibana            
elastic/eck-operator     	1.8.0        	1.8.0      	A Helm chart for deploying the Elastic Cloud on...
bitnami/dataplatform-bp2 	10.0.0       	1.0.1      	OCTO Data platform Kafka-Spark-Elasticsearch He...
elastic/eck-operator-crds	1.8.0        	1.8.0      	A Helm chart for installing the ECK operator Cu...


[root@k8s-master kibana]# helm fetch elastic/kibana
[root@k8s-master kibana]# ll
total 12
-rw-r--r-- 1 root root 10142 Dec  2 17:02 kibana-7.15.0.tgz
[root@k8s-master kibana]# tar -zxf kibana-7.15.0.tgz 
[root@k8s-master kibana]# ll
total 12
drwxr-xr-x 4 root root   128 Dec  2 17:03 kibana
-rw-r--r-- 1 root root 10142 Dec  2 17:02 kibana-7.15.0.tgz
```



#### 2.2.修改配置

```yaml
[root@k8s-master elasticsearch]# vim values.yaml


# 不用修改
# 关键配置
elasticsearchHosts: "http://elasticsearch-master:9200"
replicas: 1
```



#### 2.3.安装Kibana

```shell
[root@k8s-master kibana]# helm install kibana kibana
NAME: kibana
LAST DEPLOYED: Thu Dec  2 17:13:31 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

[root@k8s-master kibana]# helm list
NAME         	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART               	APP VERSION
elasticsearch	default  	1       	2021-12-02 16:53:28.462669857 +0800 CST	deployed	elasticsearch-7.15.0	7.15.0     
kibana       	default  	1       	2021-12-02 17:13:31.455789893 +0800 CST	deployed	kibana-7.15.0       	7.15.0     


# 查看
[root@k8s-master kibana]# kubectl get all
NAME                                 READY   STATUS              RESTARTS   AGE
pod/kibana-kibana-84dd795594-v4vxk   0/1     ContainerCreating   0          60s

NAME                                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE
service/kibana-kibana                   ClusterIP   10.98.152.76   <none>        5601/TCP            60s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/kibana-kibana   0/1     1            0           60s

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/kibana-kibana-84dd795594   1         1         0       60s
```



#### 2.4.测试

**1.修改service类型**

```shell
[root@k8s-master kibana]# kubectl edit svc kibana-kibana
service/kibana-kibana edited


[root@k8s-master kibana]# kubectl get svc
NAME                            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE
kibana-kibana                   NodePort    10.98.152.76   <none>        5601:30914/TCP      7m33s
```

```shell
# 访问地址

http://172.51.216.81:30914/
```

![](/images/kubernetes/middleware/es-1.png)

![](/images/kubernetes/middleware/es-2.png)





### 3.Logstash

```shell
# helm-charts/logstash/
https://github.com/elastic/helm-charts
https://github.com/elastic/helm-charts/tree/main/logstash


# Helm
[root@k8s-master ~]# helm search repo logstash
NAME                    	CHART VERSION	APP VERSION	DESCRIPTION                                       
elastic/logstash        	7.15.0       	7.15.0     	Official Elastic helm chart for Logstash          
```



#### 3.1.下载安装包

```shell
[root@k8s-master ~]# helm search repo logstash
NAME                    	CHART VERSION	APP VERSION	DESCRIPTION                                       
bitnami/logstash        	3.6.18       	7.15.2     	Logstash is an open source, server-side data pr...
elastic/logstash        	7.15.0       	7.15.0     	Official Elastic helm chart for Logstash          
bitnami/dataplatform-bp2	10.0.0       	1.0.1      	OCTO Data platform Kafka-Spark-Elasticsearch He...


[root@k8s-master logstash]# helm fetch elastic/logstash
[root@k8s-master logstash]# ll
total 16
-rw-r--r-- 1 root root 13431 Dec  3 09:01 logstash-7.15.0.tgz
[root@k8s-master logstash]# tar -zxf logstash-7.15.0.tgz 
[root@k8s-master logstash]# ll
total 16
drwxr-xr-x 4 root root   128 Dec  3 09:01 logstash
-rw-r--r-- 1 root root 13431 Dec  3 09:01 logstash-7.15.0.tgz
```



#### 3.2.修改配置

```yaml
[root@k8s-master logstash]# vim values.yaml


# 修改配置
# 先修改存储配置，采集配置先不处理
volumeClaimTemplate:
  accessModes: ["ReadWriteOnce"]
  storageClassName: "rook-ceph-block"
  resources:
    requests:
      storage: 1Gi
```



#### 3.3.安装Logstash

```shell
[root@k8s-master logstash]# helm install logstash logstash
NAME: logstash
LAST DEPLOYED: Fri Dec  3 09:29:16 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. Watch all cluster members come up.
  $ kubectl get pods --namespace=default -l app=logstash-logstash -w
  

[root@k8s-master logstash]# helm list
NAME         	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART               	APP VERSION
elasticsearch	default  	1       	2021-12-02 16:53:28.462669857 +0800 CST	deployed	elasticsearch-7.15.0	7.15.0     
kibana       	default  	1       	2021-12-02 17:13:31.455789893 +0800 CST	deployed	kibana-7.15.0       	7.15.0     
logstash     	default  	1       	2021-12-03 09:29:16.020275923 +0800 CST	deployed	logstash-7.15.0     	7.15.0


# 查看
[root@k8s-master logstash]# kubectl get all | grep logstash
pod/logstash-logstash-0              1/1     Running   0          12m

service/logstash-logstash-headless      ClusterIP   None           <none>        9600/TCP            12m

statefulset.apps/logstash-logstash      1/1     12m
```



### 4.Filebeat

```shell
# helm-charts/filebeat/
https://github.com/elastic/helm-charts
https://github.com/elastic/helm-charts/tree/main/filebeat

# Helm
[root@k8s-master helm]# helm search repo filebeat
NAME            	CHART VERSION	APP VERSION	DESCRIPTION                             
elastic/filebeat	7.15.0       	7.15.0     	Official Elastic helm chart for Filebeat
```



#### 4.1.下载安装包

```shell
[root@k8s-master logstash]# helm search repo filebeat
NAME            	CHART VERSION	APP VERSION	DESCRIPTION                             
elastic/filebeat	7.15.0       	7.15.0     	Official Elastic helm chart for Filebeat


[root@k8s-master filebeat]# helm fetch elastic/filebeat
[root@k8s-master filebeat]# ll
total 12
-rw-r--r-- 1 root root 12225 Dec  3 09:46 filebeat-7.15.0.tgz
[root@k8s-master filebeat]# tar -zxf filebeat-7.15.0.tgz 
[root@k8s-master filebeat]# ll
total 12
drwxr-xr-x 4 root root   128 Dec  3 09:46 filebeat
-rw-r--r-- 1 root root 12225 Dec  3 09:46 filebeat-7.15.0.tgz
```



#### 4.2.修改配置

```yaml
[root@k8s-master logstash]# vim values.yaml


# 不修改配置
# 默认配置
---
daemonset:
  # Annotations to apply to the daemonset
  annotations: {}
  # additionals labels
  labels: {}
  affinity: {}
  # Include the daemonset
  enabled: true
  # Extra environment variables for Filebeat container.
  envFrom: []
  # - configMapRef:
  #     name: config-secret
  extraEnvs: []
  #  - name: MY_ENVIRONMENT_VAR
  #    value: the_value_goes_here
  extraVolumes:
    []
    # - name: extras
    #   emptyDir: {}
  extraVolumeMounts:
    []
    # - name: extras
    #   mountPath: /usr/share/extras
    #   readOnly: true
  hostNetworking: false
  # Allows you to add any config files in /usr/share/filebeat
  # such as filebeat.yml for daemonset
  filebeatConfig:
    filebeat.yml: |
      filebeat.inputs:
      - type: container
        paths:
          - /var/log/containers/*.log
        processors:
        - add_kubernetes_metadata:
            host: ${NODE_NAME}
            matchers:
            - logs_path:
                logs_path: "/var/log/containers/"

      output.elasticsearch:
        host: '${NODE_NAME}'
        hosts: '${ELASTICSEARCH_HOSTS:elasticsearch-master:9200}'
  # Only used when updateStrategy is set to "RollingUpdate"


# 每个node节点创建daemonset，生成一个filebeat
# filebeat采集docker日志，写入ES
# hosts: '${ELASTICSEARCH_HOSTS:elasticsearch-master:9200}'
```



#### 4.3.安装Filebeat

```shell
[root@k8s-master filebeat]# helm install filebeat filebeat
NAME: filebeat
LAST DEPLOYED: Fri Dec  3 10:13:39 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. Watch all containers come up.
  $ kubectl get pods --namespace=default -l app=filebeat-filebeat -w


[root@k8s-master filebeat]# helm list
NAME         	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART               	APP VERSION
elasticsearch	default  	1       	2021-12-02 16:53:28.462669857 +0800 CST	deployed	elasticsearch-7.15.0	7.15.0     
filebeat     	default  	1       	2021-12-03 10:13:39.284547467 +0800 CST	deployed	filebeat-7.15.0     	7.15.0     
kibana       	default  	1       	2021-12-02 17:13:31.455789893 +0800 CST	deployed	kibana-7.15.0       	7.15.0     
logstash     	default  	1       	2021-12-03 09:29:16.020275923 +0800 CST	deployed	logstash-7.15.0     	7.15.0


# 查看
[root@k8s-master logstash]# kubectl get all -owide | grep filebeat
pod/filebeat-filebeat-bfz5p          1/1     Running   0          70m    10.244.36.77     k8s-node1   <none>           <none>
pod/filebeat-filebeat-hpcrf          1/1     Running   0          70m    10.244.169.150   k8s-node2   <none>           <none>
pod/filebeat-filebeat-mv2m2          1/1     Running   0          70m    10.244.107.211   k8s-node3   <none>           <none>

daemonset.apps/filebeat-filebeat   3         3         3       3            3           <none>          70m   filebeat     docker.elastic.co/beats/filebeat:7.15.0   app=filebeat-filebeat,release=filebeat
```



#### 4.4.测试

```shell
# 每个node节点的Docker日志写入ES
# 通过Kibana查看
# 创建索引、查看日志
```

```shell
# 访问地址

# Kibana
http://172.51.216.81:30914/
```

![](/images/kubernetes/middleware/es-3.png)

![](/images/kubernetes/middleware/es-4.png)

![](/images/kubernetes/middleware/es-5.png)

![](/images/kubernetes/middleware/es-6.png)



### 5.服务器Docker日志采集

默认安装的Filebeat，自动采集node节点docker日

```yaml
---
daemonset:
......
  filebeatConfig:
    filebeat.yml: |
      filebeat.inputs:
      - type: container
        paths:
          - /var/log/containers/*.log
        processors:
        - add_kubernetes_metadata:
            host: ${NODE_NAME}
            matchers:
            - logs_path:
                logs_path: "/var/log/containers/"

      output.elasticsearch:
        host: '${NODE_NAME}'
        hosts: '${ELASTICSEARCH_HOSTS:elasticsearch-master:9200}'
  # Only used when updateStrategy is set to "RollingUpdate"


# 每个node节点创建daemonset，生成一个filebeat
# filebeat采集docker日志，写入ES
# hosts: '${ELASTICSEARCH_HOSTS:elasticsearch-master:9200}'
```

![](/images/kubernetes/middleware/es-7.png)



![](/images/kubernetes/middleware/es-8.png)



### 6.Logstash采集MQ日志

#### 6.1.微服务日志发送到RabbitMQ

微服务msa-ext-elk，通过开发配置把日志发送到RabbitMQ。

```yaml
# application-dev.yml


server:
  port: 8114

spring:
  application:
    name: msa-ext-elk
    appkey: 3333

logging:
  rabbitmq:
    addresses: 172.51.216.98:5672
    username: guest
    password: guest
    exchangeName: msa-ext-elk-exchange
    routingkeypattern: log
```

访问地址：http://localhost:8114/hello



#### 6.2.修改Logstash

**修改Logstash配置，从新安装。**

```yaml
# vim values.yaml


# 修改配置

# 1
logstashConfig:
  logstash.yml: |
    http.host: "0.0.0.0"
    xpack.monitoring.elasticsearch.hosts: [ "http://elasticsearch-master.default.svc.cluster.local:9200" ]

# 2
logstashPipeline:
  logstash.conf: |
    input {
      rabbitmq {
        type =>"msa"
        durable => true
        exchange => "msa-ext-elk-exchange"
        exchange_type => "direct"
        key => "log"
        host => "rabbitmq-svc.default.svc.cluster.local"
        port => 5672
        user => "guest"
        password => "guest"
        queue => "msa-ext-elk-queue"
        auto_delete => false
      }
    }
    output {
      elasticsearch {
        hosts => ["elasticsearch-master.default.svc.cluster.local:9200"]
        index => "%{appname}-%{+YYYY.MM.dd}"
      }
      stdout {
        codec => rubydebug
      }
    }


# 关键配置信息：
# K8S 中的容器使用访问
# rabbitmq地址
rabbitmq-svc.default.svc.cluster.local:5672
# ES地址
elasticsearch-master.default.svc.cluster.local:9200
```

```yaml
# 完整配置


[root@k8s-master logstash]# vim values.yaml

---
replicas: 1

# Allows you to add any config files in /usr/share/logstash/config/
# such as logstash.yml and log4j2.properties
#
# Note that when overriding logstash.yml, `http.host: 0.0.0.0` should always be included
# to make default probes work.
#logstashConfig: {}
logstashConfig:
  logstash.yml: |
    http.host: "0.0.0.0"
    xpack.monitoring.elasticsearch.hosts: [ "http://elasticsearch-master.default.svc.cluster.local:9200" ]

#  logstash.yml: |
#    key:
#      nestedkey: value
#  log4j2.properties: |
#    key = value

# Allows you to add any pipeline files in /usr/share/logstash/pipeline/
### ***warn*** there is a hardcoded logstash.conf in the image, override it first
logstashPipeline:
  logstash.conf: |
    input {
      rabbitmq {
        type =>"msa"
#  logstash.yml: |
#    key:
#      nestedkey: value
#  log4j2.properties: |
#    key = value

# Allows you to add any pipeline files in /usr/share/logstash/pipeline/
### ***warn*** there is a hardcoded logstash.conf in the image, override it first
logstashPipeline:
  logstash.conf: |
    input {
      rabbitmq {
        type =>"msa"
        durable => true
        exchange => "msa-ext-elk-exchange"
        exchange_type => "direct"
        key => "log"
        host => "rabbitmq-svc.default.svc.cluster.local"
        port => 5672
        user => "guest"
        password => "guest"
        queue => "msa-ext-elk-queue"
        auto_delete => false
      }
    }
    output {
      elasticsearch {
        hosts => ["elasticsearch-master.default.svc.cluster.local:9200"]
        index => "%{appname}-%{+YYYY.MM.dd}"
      }
      stdout {
        codec => rubydebug
      }
    }

#  logstash.conf: |
#    input {
#      exec {
#        command => "uptime"
#        interval => 30
#      }
#    }
#    output { stdout { } }

# Allows you to add any pattern files in your custom pattern dir
logstashPatternDir: "/usr/share/logstash/patterns/"
logstashPattern: {}
#    pattern.conf: |
#      DPKG_VERSION [-+~<>\.0-9a-zA-Z]+

# Extra environment variables to append to this nodeGroup
# This will be appended to the current 'env:' key. You can use any of the kubernetes env
# syntax here
extraEnvs: []
#  - name: MY_ENVIRONMENT_VAR
#    value: the_value_goes_here

# Allows you to load environment variables from kubernetes secret or config map
envFrom: []
# - secretRef:
#     name: env-secret
# - configMapRef:
#     name: config-map

# Add sensitive data to k8s secrets
secrets: []
#  - name: "env"
#    value:
#      ELASTICSEARCH_PASSWORD: "LS1CRUdJTiBgUFJJVkFURSB"
#      api_key: ui2CsdUadTiBasRJRkl9tvNnw
#  - name: "tls"
#    value:
#      ca.crt: |
#        LS0tLS1CRUdJT0K
#        LS0tLS1CRUdJT0K
#        LS0tLS1CRUdJT0K
#        LS0tLS1CRUdJT0K
#      cert.crt: "LS0tLS1CRUdJTiBlRJRklDQVRFLS0tLS0K"
#      cert.key.filepath: "secrets.crt" # The path to file should be relative to the `values.yaml` file.

# A list of secrets and their paths to mount inside the pod
secretMounts: []

hostAliases: []
#- ip: "127.0.0.1"
#  hostnames:
#  - "foo.local"
#  - "bar.local"

image: "docker.elastic.co/logstash/logstash"
imageTag: "7.15.0"
imagePullPolicy: "IfNotPresent"
imagePullSecrets: []

podAnnotations: {}

# additionals labels
labels: {}

logstashJavaOpts: "-Xmx1g -Xms1g"

resources:
  requests:
    cpu: "100m"
    memory: "1536Mi"
  limits:
    cpu: "1000m"
    memory: "1536Mi"

volumeClaimTemplate:
  accessModes: ["ReadWriteOnce"]
  storageClassName: "rook-ceph-block"
  resources:
    requests:
      storage: 1Gi

rbac:
  create: false
  serviceAccountAnnotations: {}
  serviceAccountName: ""
  annotations:
    {}
    #annotation1: "value1"
    #annotation2: "value2"
    #annotation3: "value3"

podSecurityPolicy:
  create: false
  name: ""
  spec:
    privileged: false
    fsGroup:
      rule: RunAsAny
    runAsUser:
      rule: RunAsAny
    seLinux:
      rule: RunAsAny
    supplementalGroups:
      rule: RunAsAny
    volumes:
      - secret
      - configMap
      - persistentVolumeClaim

persistence:
  enabled: false
  annotations: {}

extraVolumes:
  ""
  # - name: extras
  #   emptyDir: {}

extraVolumeMounts:
  ""
  # - name: extras
  #   mountPath: /usr/share/extras
  #   readOnly: true

extraContainers:
  ""
  # - name: do-something
  #   image: busybox
  #   command: ['do', 'something']

extraInitContainers:
  ""
  # - name: do-something
  #   image: busybox
  #   command: ['do', 'something']

# This is the PriorityClass settings as defined in
# https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/#priorityclass
priorityClassName: ""

# By default this will make sure two pods don't end up on the same node
# Changing this to a region would allow you to spread pods across regions
antiAffinityTopologyKey: "kubernetes.io/hostname"

# Hard means that by default pods will only be scheduled if there are enough nodes for them
# and that they will never end up on the same node. Setting this to soft will do this "best effort"
antiAffinity: "hard"

# This is the node affinity settings as defined in
# https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#node-affinity
nodeAffinity: {}

# This is inter-pod affinity settings as defined in
# https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#inter-pod-affinity-and-anti-affinity
podAffinity: {}

# The default is to deploy all pods serially. By setting this to parallel all pods are started at
# the same time when bootstrapping the cluster
podManagementPolicy: "Parallel"

httpPort: 9600

# Custom ports to add to logstash
extraPorts:
  []
  # - name: beats
  #   containerPort: 5001

updateStrategy: RollingUpdate

# This is the max unavailable setting for the pod disruption budget
# The default value of 1 will make sure that kubernetes won't allow more than 1
# of your pods to be unavailable during maintenance
maxUnavailable: 1

podSecurityContext:
  fsGroup: 1000
  runAsUser: 1000

securityContext:
  capabilities:
    drop:
      - ALL
  # readOnlyRootFilesystem: true
  runAsNonRoot: true
  runAsUser: 1000

# How long to wait for logstash to stop gracefully
terminationGracePeriod: 120

# Probes
# Default probes are using `httpGet` which requires that `http.host: 0.0.0.0` is part of
# `logstash.yml`. If needed probes can be disabled or overrided using the following syntaxes:
#
# disable livenessProbe
# livenessProbe: null
#
# replace httpGet default readinessProbe by some exec probe
# readinessProbe:
#   httpGet: null
#   exec:
#     command:
#       - curl
#      - localhost:9600

livenessProbe:
  httpGet:
    path: /
    port: http
  initialDelaySeconds: 300
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3
  successThreshold: 1

readinessProbe:
  httpGet:
    path: /
    port: http
  initialDelaySeconds: 60
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3
  successThreshold: 3

## Use an alternate scheduler.
## ref: https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/
##
schedulerName: ""

nodeSelector: {}
tolerations: []

nameOverride: ""
fullnameOverride: ""

lifecycle:
  {}
  # preStop:
  #   exec:
  #     command: ["/bin/sh", "-c", "echo Hello from the postStart handler > /usr/share/message"]
  # postStart:
  #   exec:
  #     command: ["/bin/sh", "-c", "echo Hello from the postStart handler > /usr/share/message"]

service: {}
#  annotations: {}
#  type: ClusterIP
#  loadBalancerIP: ""
#  ports:
#    - name: beats
#      port: 5044
#      protocol: TCP
#      targetPort: 5044
#    - name: http
#      port: 8080
#      protocol: TCP
#      targetPort: 8080

ingress:
  enabled: false
#  annotations: {}
#  hosts:
#    - host: logstash.local
#      paths:
#        - path: /logs
#          servicePort: 8080
#  tls: []
```



#### 6.3.测试

![](/images/kubernetes/middleware/es-9.png)



![](/images/kubernetes/middleware/es-10.png)



### 7.删除集群

```shell
# 查看
[root@k8s-master filebeat]# helm list
NAME         	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART               	APP VERSION
elasticsearch	default  	1       	2021-12-02 16:53:28.462669857 +0800 CST	deployed	elasticsearch-7.15.0	7.15.0     
filebeat     	default  	1       	2021-12-03 10:13:39.284547467 +0800 CST	deployed	filebeat-7.15.0     	7.15.0     
kibana       	default  	1       	2021-12-02 17:13:31.455789893 +0800 CST	deployed	kibana-7.15.0       	7.15.0     
logstash     	default  	1       	2021-12-03 09:29:16.020275923 +0800 CST	deployed	logstash-7.15.0     	7.15.0
```

```shell
# 删除

helm delete elasticsearch
helm delete kibana
helm delete logstash
helm delete filebeat
```



