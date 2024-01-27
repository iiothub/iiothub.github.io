* TOC
{:toc}


### 1.安装方式

```shell
# 安装
# RabbitMQ #1 (Official)  rabbitmq/cluster-operator
https://github.com/rabbitmq/cluster-operator


#RabbitMQ Cluster Kubernetes Operator Quickstart
https://www.rabbitmq.com/kubernetes/operator/quickstart-operator.html

# Using RabbitMQ Cluster Kubernetes Operator
https://www.rabbitmq.com/kubernetes/operator/using-operator.html
```

```shell
# 参考

# Operator overview
https://www.rabbitmq.com/kubernetes/operator/operator-overview.html

# Deploying an operator
https://www.rabbitmq.com/kubernetes/operator/install-operator.html

# Deploying a RabbitMQ cluster
https://www.rabbitmq.com/kubernetes/operator/using-operator.html

# Monitoring the cluster
https://www.rabbitmq.com/kubernetes/operator/operator-monitoring.html

# Troubleshooting operator deployments
https://www.rabbitmq.com/kubernetes/operator/troubleshooting-operator.html
```



**下载rabbitmq/cluster-operator**

```shell
# git clone https://github.com/rabbitmq/cluster-operator.git
```



### 2.部署RabbitMQ集群

**1.下载rabbitmq/cluster-operator**

```shell
# git  git clone https://github.com/rabbitmq/cluster-operator.git


[root@k8s-master rabbitmq]# git clone https://github.com/rabbitmq/cluster-operator.git
Cloning into 'cluster-operator'...
remote: Enumerating objects: 11892, done.
remote: Counting objects: 100% (2279/2279), done.
remote: Compressing objects: 100% (976/976), done.
remote: Total 11892 (delta 1438), reused 1976 (delta 1240), pack-reused 9613
Receiving objects: 100% (11892/11892), 11.95 MiB | 1.66 MiB/s, done.
Resolving deltas: 100% (7740/7740), done.
```



**2.创建Operator**

```bash
# kubectl apply -f https://github.com/rabbitmq/cluster-operator/releases/latest/download/cluster-operator.yml


# 创建
[root@k8s-master rabbitmq]# kubectl apply -f https://github.com/rabbitmq/cluster-operator/releases/latest/download/cluster-operator.yml
namespace/rabbitmq-system created
customresourcedefinition.apiextensions.k8s.io/rabbitmqclusters.rabbitmq.com created
serviceaccount/rabbitmq-cluster-operator created
role.rbac.authorization.k8s.io/rabbitmq-cluster-leader-election-role created
clusterrole.rbac.authorization.k8s.io/rabbitmq-cluster-operator-role created
rolebinding.rbac.authorization.k8s.io/rabbitmq-cluster-leader-election-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/rabbitmq-cluster-operator-rolebinding created
deployment.apps/rabbitmq-cluster-operator created


[root@k8s-master1 production-ready]# kubectl get all -n rabbitmq-system
NAME                                             READY   STATUS    RESTARTS   AGE
pod/rabbitmq-cluster-operator-7cbf865f89-6rppq   1/1     Running   0          50s

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/rabbitmq-cluster-operator   1/1     1            1           50s

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/rabbitmq-cluster-operator-7cbf865f89   1         1         1       50s
```



**3.修改配置**

```yaml
# cluster-operator/docs/examples/production-ready
# rabbitmq.yaml
# cpu、memory
# storageClassName: rook-ceph-block


[root@k8s-master production-ready]# vim rabbitmq.yaml 

apiVersion: rabbitmq.com/v1beta1
kind: RabbitmqCluster
metadata:
  name: production-ready
spec:
  replicas: 3
  resources:
    requests:
      cpu: 1
      memory: 2Gi
    limits:
      cpu: 1
      memory: 2Gi
  rabbitmq:
    additionalConfig: |
      cluster_partition_handling = pause_minority
      vm_memory_high_watermark_paging_ratio = 0.99
      disk_free_limit.relative = 1.0
      collect_statistics_interval = 10000
  persistence:
    storageClassName: rook-ceph-block
    storage: "15Gi"
```



**4.使用cluster-operator创建RabbitMQ集群**

```shell
[root@k8s-master1 production-ready]# pwd
/k8s/middleware/rabbitmq/cluster-operator/docs/examples/production-ready
[root@k8s-master1 production-ready]# ll
total 16
-rw-r--r--. 1 root root  199 Dec 22 11:14 pod-disruption-budget.yaml
-rw-r--r--. 1 root root  500 Dec 22 11:16 rabbitmq.yaml
-rw-r--r--. 1 root root 3134 Dec 22 11:14 README.md
-rw-r--r--. 1 root root  409 Dec 22 11:14 ssd-gke.yaml


# 创建
[root@k8s-master1 production-ready]# kubectl apply -f rabbitmq.yaml
rabbitmqcluster.rabbitmq.com/production-ready created


[root@k8s-master1 production-ready]# kubectl apply -f pod-disruption-budget.yaml 
poddisruptionbudget.policy/production-ready-rabbitmq created


# 查看
[root@k8s-master1 ~]# kubectl get all
NAME                            READY   STATUS    RESTARTS   AGE
pod/production-ready-server-0   1/1     Running   0          10m
pod/production-ready-server-1   1/1     Running   2          88s
pod/production-ready-server-2   1/1     Running   0          10m

NAME                                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                         AGE
service/production-ready            ClusterIP   10.1.23.122    <none>        5672/TCP,15672/TCP,15692/TCP    10m
service/production-ready-nodes      ClusterIP   None           <none>        4369/TCP,25672/TCP              10m

NAME                                       READY   AGE
statefulset.apps/production-ready-server   3/3     10m

NAME                                            ALLREPLICASREADY   RECONCILESUCCESS   AGE
rabbitmqcluster.rabbitmq.com/production-ready   True               True               10m



[root@k8s-master production-ready]# kubectl get all -n rabbitmq-system
NAME                                             READY   STATUS    RESTARTS   AGE
pod/rabbitmq-cluster-operator-7cbf865f89-s95gq   1/1     Running   0          105m

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/rabbitmq-cluster-operator   1/1     1            1           105m

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/rabbitmq-cluster-operator-7cbf865f89   1         1         1       105m


# PVC
[root@k8s-master1 ~]# kubectl get pvc
NAME                                    STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
persistence-production-ready-server-0   Bound    pvc-837eb2ac-dd8d-4972-a0ec-51667f12b96f   15Gi       RWO            rook-ceph-block   11m
persistence-production-ready-server-1   Bound    pvc-d0732c03-f9f6-4826-b6bc-902d099e342d   15Gi       RWO            rook-ceph-block   11m
persistence-production-ready-server-2   Bound    pvc-8bc841df-20fe-4db1-a128-152539e7ab3d   15Gi       RWO            rook-ceph-block   11m
```



### 3.获取用户名和密码

**获取用户名和密码**

```shell
# user
kubectl -n NAMESPACE get secret INSTANCE-default-user -o jsonpath="{.data.username}" | base64 --decode
# pass
kubectl -n NAMESPACE get secret INSTANCE-default-user -o jsonpath="{.data.password}" | base64 --decode


# 获取获取用户名和密码  production-ready
username="$(kubectl get secret production-ready-default-user -o jsonpath='{.data.username}' | base64 --decode)"
echo "username: $username"
password="$(kubectl get secret production-ready-default-user -o jsonpath='{.data.password}' | base64 --decode)"
echo "password: $password"


root@k8s-master1 ~]# username="$(kubectl get secret production-ready-default-user -o jsonpath='{.data.username}' | base64 --decode)"
[root@k8s-master1 ~]# echo "username: $username"
username: default_user_VcSb2rx0RLFPBfw3kcv

[root@k8s-master1 ~]# password="$(kubectl get secret production-ready-default-user -o jsonpath='{.data.password}' | base64 --decode)"
[root@k8s-master1 ~]# echo "password: $password"
password: apexs6u5gj2wAu29O1YxGtXuQnINzEvs


# 登录信息：
username: default_user_skDybgNFObPZkgkKzJN
password: lojS77Jp5DvMURsjFp3XK73XnwLa1hWI


kubectl port-forward "service/production-ready" 15672
```

```shell
# 系统创建service
# production-ready

[root@k8s-master1 production-ready]# kubectl get svc
NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                         AGE
production-ready            ClusterIP   10.1.23.122    <none>        5672/TCP,15672/TCP,15692/TCP    15m
production-ready-nodes      ClusterIP   None           <none>        4369/TCP,25672/TCP              15m


[root@k8s-master production-ready]# kubectl port-forward "service/production-ready" 15672
Forwarding from 127.0.0.1:15672 -> 15672
Forwarding from [::1]:15672 -> 15672



curl -u$username:$password localhost:15672/api/overview
{"management_version":"3.8.9","rates_mode":"basic", ...}


curl -udefault_user_VcSb2rx0RLFPBfw3kcv:apexs6u5gj2wAu29O1YxGtXuQnINzEvs 10.1.23.122:15672/api/overview


# 访问
[root@k8s-master production-ready]# curl -udefault_user_vmOg0aiF8VarwVhb0sE:rKr9N2v-Cxrec78zo1LUnigjyOla_YLv 10.111.187.65:15672/api/overview
{"management_version":"3.8.21","rates_mode":"basic","sample_retention_policies":{"global":[600,3600,28800,86400],"basic":[600,3600],"detailed":[600]},"exchange_types":[{"name":"direct","description":"AMQP direct exchange, as per the AMQP specification","enabled":true},{"name":"fanout","description":"AMQP fanout exchange, as per the AMQP specification","enabled":true},{"name":"headers","description":"AMQP headers exchange, as per the AMQP specification","enabled":true},{"name":"topic","description":"AMQP topic exchange, as per the AMQP specification","enabled":true}],"product_version":"3.8.21","product_name":"RabbitMQ","rabbitmq_version":"3.8.21","cluster_name":"production-ready","erlang_version":"24.0.5","erlang_full_version":"Erlang/OTP 24 [erts-12.0.3] [source] [64-bit] [smp:4:1] [ds:4:1:10] [async-threads:1] [jit]","disable_stats":false,"enable_queue_totals":false,"message_stats":{},"churn_rates":{"channel_closed":0,"channel_closed_details":{"rate":0.0},"channel_created":0,"channel_created_details":{"rate":0.0},"connection_closed":153,"connection_closed_details":{"rate":0.2},"connection_created":0,"connection_created_details":{"rate":0.0},"queue_created":0,"queue_created_details":{"rate":0.0},"queue_declared":0,"queue_declared_details":{"rate":0.0},"queue_deleted":0,"queue_deleted_details":{"rate":0.0}},"queue_totals":{},"object_totals":{"channels":0,"connections":0,"consumers":0,"exchanges":7,"queues":0},"statistics_db_event_queue":0,"node":"rabbit@production-ready-server-1.production-ready-nodes.default","listeners":[{"node":"rabbit@production-ready-server-0.production-ready-nodes.default","protocol":"amqp","ip_address":"::","port":5672,"socket_opts":{"backlog":128,"nodelay":true,"linger":[true,0],"exit_on_close":false}},{"node":"rabbit@production-ready-server-1.production-ready-nodes.default","protocol":"amqp","ip_address":"::","port":5672,"socket_opts":{"backlog":128,"nodelay":true,"linger":[true,0],"exit_on_close":false}},{"node":"rabbit@production-ready-server-2.production-ready-nodes.default","protocol":"amqp","ip_address":"::","port":5672,"socket_opts":{"backlog":128,"nodelay":true,"linger":[true,0],"exit_on_close":false}},{"node":"rabbit@production-ready-server-0.production-ready-nodes.default","protocol":"clustering","ip_address":"::","port":25672,"socket_opts":[]},{"node":"rabbit@production-ready-server-1.production-ready-nodes.default","protocol":"clustering","ip_address":"::","port":25672,"socket_opts":[]},{"node":"rabbit@production-ready-server-2.production-ready-nodes.default","protocol":"clustering","ip_address":"::","port":25672,"socket_opts":[]},{"node":"rabbit@production-ready-server-0.production-ready-nodes.default","protocol":"http","ip_address":"::","port":15672,"socket_opts":{"cowboy_opts":{"sendfile":false},"port":15672}},{"node":"rabbit@production-ready-server-1.production-ready-nodes.default","protocol":"http","ip_address":"::","port":15672,"socket_opts":{"cowboy_opts":{"sendfile":false},"port":15672}},{"node":"rabbit@production-ready-server-2.production-ready-nodes.default","protocol":"http","ip_address":"::","port":15672,"socket_opts":{"cowboy_opts":{"sendfile":false},"port":15672}},{"node":"rabbit@production-ready-server-0.production-ready-nodes.default","protocol":"http/prometheus","ip_address":"::","port":15692,"socket_opts":{"cowboy_opts":{"sendfile":false},"port":15692,"protocol":"http/prometheus"}},{"node":"rabbit@production-ready-server-1.production-ready-nodes.default","protocol":"http/prometheus","ip_address":"::","port":15692,"socket_opts":{"cowboy_opts":{"sendfile":false},"port":15692,"protocol":"http/prometheus"}},{"node":"rabbit@production-ready-server-2.production-ready-nodes.default","protocol":"http/prometheus","ip_address":"::","port":15692,"socket_opts":{"cowboy_opts":{"sendfile":false},"port":15692,"protocol":"http/prometheus"}}],"contexts":[{"ssl_opts":[],"node":"rabbit@production-ready-server-0.production-ready-nodes.default","description":"RabbitMQ Management","path":"/","cowboy_opts":"[{sendfile,false}]","port":"15672"},{"ssl_opts":[],"node":"rabbit@production-ready-server-1.production-ready-nodes.default","description":"RabbitMQ Management","path":"/","cowboy_opts":"[{sendfile,false}]","port":"15672"},{"ssl_opts":[],"node":"rabbit@production-ready-server-2.production-ready-nodes.default","description":"RabbitMQ Management","path":"/","cowboy_opts":"[{sendfile,false}]","port":"15672"},{"ssl_opts":[],"node":"rabbit@production-ready-server-0.production-ready-nodes.default","description":"RabbitMQ Prometheus","path":"/","cowboy_opts":"[{sendfile,false}]","port":"15692","protocol":"'http/prometheus'"},{"ssl_opts":[],"node":"rabbit@production-ready-server-1.production-ready-nodes.default","description":"RabbitMQ Prometheus","path":"/","cowboy_opts":"[{sendfile,false}]","port":"15692","protocol":"'http/prometheus'"},{"ssl_opts":[],"node":"rabbit@production-ready-server-2.production-ready-nodes.default","description":"RabbitMQ Prometheus","path":"/","cowboy_opts":"[{sendfile,false}]","port":"15692","protocol":"'http/prometheus'"}]}[root@k8s-master production-ready]# 
```



### 4.访问RabbitMQ管理UI

rabbitmq-cluster-svc.yaml

```yaml
# rabbitmq-cluster-svc.yaml


apiVersion: v1
kind: Service
metadata:
  namespace: default
  name: rabbitmq-cluster-svc
  labels:
    app.kubernetes.io/name: production-ready
spec:
  type: NodePort
  ports:
    - port: 15672
      name: rabbitmq-http
      targetPort: 15672
      nodePort: 30672
    - port: 5672
      name: rabbitmq-tcp
      targetPort: 5672
      nodePort: 30673
  selector:
    app.kubernetes.io/name: production-ready
```

```shell
[root@k8s-master1 rabbitmq]# kubectl apply -f rabbitmq-cluster-svc.yaml 
service/rabbitmq-cluster-svc created


[root@k8s-master1 rabbitmq]# kubectl get svc
NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                          AGE
production-ready            ClusterIP   10.1.23.122    <none>        5672/TCP,15672/TCP,15692/TCP     20m
production-ready-nodes      ClusterIP   None           <none>        4369/TCP,25672/TCP               20m
rabbitmq-cluster-svc        NodePort    10.1.182.218   <none>        15672:30672/TCP,5672:30673/TCP   20s


curl -udefault_user_vmOg0aiF8VarwVhb0sE:rKr9N2v-Cxrec78zo1LUnigjyOla_YLv 10.106.36.141:15672/api/overview
```

```shell
# 地址

http://172.51.216.81:30672
username: default_user_skDybgNFObPZkgkKzJN
password: lojS77Jp5DvMURsjFp3XK73XnwLa1hWI


# k8s内部访问地址
# K8s 中的容器使用访问
svcname.namespace.svc.cluster.local:port
production-ready.default.svc.cluster.local:5672
```



