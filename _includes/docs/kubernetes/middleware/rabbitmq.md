* TOC
{:toc}



## 一、概述



### 1.Operator

 ```shell
 # awesome-operators
 https://github.com/operator-framework/awesome-operators
 
 
 # 安装
 # RabbitMQ #1 (Official)  rabbitmq/cluster-operator
 https://github.com/rabbitmq/cluster-operator
 ```



| App Name               | Github                                                       | Description                               |
| ---------------------- | ------------------------------------------------------------ | ----------------------------------------- |
| RabbitMQ #1 (Official) | [rabbitmq/cluster-operator](https://github.com/rabbitmq/cluster-operator) | RabbitMQ cluster operator for Kubernetes. |
| RabbitMQ #2            | [skylt/rabbitmq-operator](https://github.com/skylt/rabbitmq-operator) | RabbitMQ operator for Kubernetes.         |



### 2.Helm

**Bitnami**

```shell
https://bitnami.com/
https://github.com/bitnami
https://bitnami.com/stacks
```



```shell
[root@k8s-master redis-cluster-operator]# helm search repo rabbitmq
NAME                             	CHART VERSION	APP VERSION	DESCRIPTION                                 
bitnami/rabbitmq                 	8.23.0       	3.9.7      	Open source message broker software that implem...
bitnami/rabbitmq-cluster-operator	0.1.6        	1.9.0      	The RabbitMQ Cluster Kubernetes Operator automa...
......
```



## 二、基础



### 1.Operator

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



#### 1.1.Quickstart（单实例）

```shell
#RabbitMQ Cluster Kubernetes Operator Quickstart
https://www.rabbitmq.com/kubernetes/operator/quickstart-operator.html


Quickstart Steps
This guide goes through the following steps:
Install the RabbitMQ Cluster Operator
Deploy a RabbitMQ Cluster using the Operator
View RabbitMQ Logs
Access the RabbitMQ Management UI
Attach a Workload to the Cluster
Next Steps
```



**1.创建Operator**

```shell
#kubectl apply -f https://github.com/rabbitmq/cluster-operator/releases/latest/download/cluster-operator.yml


[root@k8s-master docs]# kubectl apply -f https://github.com/rabbitmq/cluster-operator/releases/latest/download/cluster-operator.yml
namespace/rabbitmq-system created
customresourcedefinition.apiextensions.k8s.io/rabbitmqclusters.rabbitmq.com created
serviceaccount/rabbitmq-cluster-operator created
role.rbac.authorization.k8s.io/rabbitmq-cluster-leader-election-role created
clusterrole.rbac.authorization.k8s.io/rabbitmq-cluster-operator-role created
rolebinding.rbac.authorization.k8s.io/rabbitmq-cluster-leader-election-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/rabbitmq-cluster-operator-rolebinding created
deployment.apps/rabbitmq-cluster-operator created


# 查看
# 会创建一个新的名称空间：rabbitmq-system
[root@k8s-master docs]# kubectl get all -n rabbitmq-system
NAME                                             READY   STATUS    RESTARTS   AGE
pod/rabbitmq-cluster-operator-7cbf865f89-5c4wk   1/1     Running   0          81s

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/rabbitmq-cluster-operator   1/1     1            1           81s

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/rabbitmq-cluster-operator-7cbf865f89   1         1         1       81s
```



**2.使用cluster-operator创建RabbitMQ**

简单的yaml文件下载地址：https://github.com/rabbitmq/cluster-operator/tree/main/docs/examples/hello-world
yaml文件名：rabbitmq.yaml
这是最简单的RabbitmqCluster定义。唯一显式指定的属性是集群的名称。其他一切都将根据集群运营商的默认值进行配置。

examples文件夹还有许多其他引用，比如用TLS、mTLS创建RabbitMQ集群，用生产默认值设置集群，添加社区插件等等。下载地址：https://github.com/rabbitmq/cluster-operator/tree/main/docs/examples/

```yaml
# rabbitmq.yaml

apiVersion: rabbitmq.com/v1beta1
kind: RabbitmqCluster
metadata:
  name: hello-world
```

```shell
# kubectl apply -f https://raw.githubusercontent.com/rabbitmq/cluster-operator/main/docs/examples/hello-world/rabbitmq.yaml


[root@k8s-master docs]# kubectl apply -f https://raw.githubusercontent.com/rabbitmq/cluster-operator/main/docs/examples/hello-world/rabbitmq.yaml
rabbitmqcluster.rabbitmq.com/hello-world created


[root@k8s-master docs]# kubectl get all
NAME                       READY   STATUS    RESTARTS   AGE
pod/hello-world-server-0   0/1     Pending   0          74s

NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                        AGE
service/hello-world         ClusterIP   10.100.123.79   <none>        15672/TCP,15692/TCP,5672/TCP   74s
service/hello-world-nodes   ClusterIP   None            <none>        4369/TCP,25672/TCP             74s

NAME                                  READY   AGE
statefulset.apps/hello-world-server   0/1     74s

NAME                                       ALLREPLICASREADY   RECONCILESUCCESS   AGE
rabbitmqcluster.rabbitmq.com/hello-world   False              Unknown            74s


[root@k8s-master rabbitmq]# kubectl get pvc
NAME                               STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistence-hello-world-server-0   Pending                                                     16m


# 查找Pending愿意，没有存储
[root@k8s-master docs]# kubectl describe pod hello-world-server-0
Name:           hello-world-server-0
Namespace:      default
Priority:       0
Node:           <none>
Labels:         app.kubernetes.io/component=rabbitmq
                app.kubernetes.io/name=hello-world
                app.kubernetes.io/part-of=rabbitmq
                controller-revision-hash=hello-world-server-6669f4bc8f
                statefulset.kubernetes.io/pod-name=hello-world-server-0
Annotations:    prometheus.io/port: 15692
                prometheus.io/scrape: true
Status:         Pending
IP:             
IPs:            <none>
Controlled By:  StatefulSet/hello-world-server
Init Containers:
  setup-container:
    Image:      rabbitmq:3.8.21-management
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      cp /tmp/erlang-cookie-secret/.erlang.cookie /var/lib/rabbitmq/.erlang.cookie && chmod 600 /var/lib/rabbitmq/.erlang.cookie ; cp /tmp/rabbitmq-plugins/enabled_plugins /operator/enabled_plugins ; echo '[default]' > /var/lib/rabbitmq/.rabbitmqadmin.conf && sed -e 's/default_user/username/' -e 's/default_pass/password/' /tmp/default_user.conf >> /var/lib/rabbitmq/.rabbitmqadmin.conf && chmod 600 /var/lib/rabbitmq/.rabbitmqadmin.conf
    Limits:
      cpu:     100m
      memory:  500Mi
    Requests:
      cpu:        100m
      memory:     500Mi
    Environment:  <none>
    Mounts:
      /operator from rabbitmq-plugins (rw)
      /tmp/default_user.conf from rabbitmq-confd (rw,path="default_user.conf")
      /tmp/erlang-cookie-secret/ from erlang-cookie-secret (rw)
      /tmp/rabbitmq-plugins/ from plugins-conf (rw)
      /var/lib/rabbitmq/ from rabbitmq-erlang-cookie (rw)
      /var/lib/rabbitmq/mnesia/ from persistence (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from hello-world-server-token-xvxcp (ro)
Containers:
  rabbitmq:
    Image:       rabbitmq:3.8.21-management
    Ports:       4369/TCP, 5672/TCP, 15672/TCP, 15692/TCP
    Host Ports:  0/TCP, 0/TCP, 0/TCP, 0/TCP
    Limits:
      cpu:     2
      memory:  2Gi
    Requests:
      cpu:      1
      memory:   2Gi
    Readiness:  tcp-socket :amqp delay=10s timeout=5s period=10s #success=1 #failure=3
    Environment:
      MY_POD_NAME:                    hello-world-server-0 (v1:metadata.name)
      MY_POD_NAMESPACE:               default (v1:metadata.namespace)
      K8S_SERVICE_NAME:               hello-world-nodes
      RABBITMQ_ENABLED_PLUGINS_FILE:  /operator/enabled_plugins
      RABBITMQ_USE_LONGNAME:          true
      RABBITMQ_NODENAME:              rabbit@$(MY_POD_NAME).$(K8S_SERVICE_NAME).$(MY_POD_NAMESPACE)
      K8S_HOSTNAME_SUFFIX:            .$(K8S_SERVICE_NAME).$(MY_POD_NAMESPACE)
    Mounts:
      /etc/pod-info/ from pod-info (rw)
      /etc/rabbitmq/conf.d/10-operatorDefaults.conf from rabbitmq-confd (rw,path="operatorDefaults.conf")
      /etc/rabbitmq/conf.d/11-default_user.conf from rabbitmq-confd (rw,path="default_user.conf")
      /etc/rabbitmq/conf.d/90-userDefinedConfiguration.conf from rabbitmq-confd (rw,path="userDefinedConfiguration.conf")
      /operator from rabbitmq-plugins (rw)
      /var/lib/rabbitmq/ from rabbitmq-erlang-cookie (rw)
      /var/lib/rabbitmq/mnesia/ from persistence (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from hello-world-server-token-xvxcp (ro)
Conditions:
  Type           Status
  PodScheduled   False 
Volumes:
  persistence:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  persistence-hello-world-server-0
    ReadOnly:   false
  plugins-conf:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      hello-world-plugins-conf
    Optional:  false
  rabbitmq-confd:
    Type:                Projected (a volume that contains injected data from multiple sources)
    ConfigMapName:       hello-world-server-conf
    ConfigMapOptional:   <nil>
    SecretName:          hello-world-default-user
    SecretOptionalName:  <nil>
  rabbitmq-erlang-cookie:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  erlang-cookie-secret:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  hello-world-erlang-cookie
    Optional:    false
  rabbitmq-plugins:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  pod-info:
    Type:  DownwardAPI (a volume populated by information about the pod)
    Items:
      metadata.labels['skipPreStopChecks'] -> skipPreStopChecks
  hello-world-server-token-xvxcp:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  hello-world-server-token-xvxcp
    Optional:    false
QoS Class:       Burstable
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age    From               Message
  ----     ------            ----   ----               -------
  Warning  FailedScheduling  8m21s  default-scheduler  0/4 nodes are available: 4 pod has unbound immediate PersistentVolumeClaims.
  Warning  FailedScheduling  8m21s  default-scheduler  0/4 nodes are available: 4 pod has unbound immediate PersistentVolumeClaims.
```

```yaml
# 手动创建PVC
# persistence-hello-world-server-0
# persistence-hello-world-server-0.yaml


apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: persistence-hello-world-server-0
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: rook-ceph-block
  resources:
    requests:
      storage: 12Gi
      

[root@k8s-master rabbitmq]# kubectl apply -f persistence-hello-world-server-0.yaml 
persistentvolumeclaim/persistence-hello-world-server-0 created

[root@k8s-master rabbitmq]# kubectl get pvc
NAME                               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
persistence-hello-world-server-0   Bound    pvc-91f82092-4b29-42ab-87b4-2cf3f9099089   12Gi       RWO            rook-ceph-block   3s
```

```shell
# 服务正常


[root@k8s-master rabbitmq]# kubectl get all
NAME                       READY   STATUS    RESTARTS   AGE
pod/hello-world-server-0   1/1     Running   0          20m

NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                        AGE
service/hello-world         ClusterIP   10.100.123.79   <none>        15672/TCP,15692/TCP,5672/TCP   20m
service/hello-world-nodes   ClusterIP   None            <none>        4369/TCP,25672/TCP             20m

NAME                                  READY   AGE
statefulset.apps/hello-world-server   1/1     20m

NAME                                       ALLREPLICASREADY   RECONCILESUCCESS   AGE
rabbitmqcluster.rabbitmq.com/hello-world   True               True               20m


[root@k8s-master rabbitmq]# kubectl get pvc
NAME                               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
persistence-hello-world-server-0   Bound    pvc-91f82092-4b29-42ab-87b4-2cf3f9099089   12Gi       RWO            rook-ceph-block   2m27s
```



#### 1.2.获取用户名和密码

**获取用户名和密码**

```shell
# 获取获取用户名和密码
username="$(kubectl get secret hello-world-default-user -o jsonpath='{.data.username}' | base64 --decode)"
echo "username: $username"
password="$(kubectl get secret hello-world-default-user -o jsonpath='{.data.password}' | base64 --decode)"
echo "password: $password"


[root@k8s-master rabbitmq]# username="$(kubectl get secret hello-world-default-user -o jsonpath='{.data.username}' | base64 --decode)"
[root@k8s-master rabbitmq]# echo "username: $username"
username: default_user_HGTmPURsI5VhaVjmeEQ


[root@k8s-master rabbitmq]# password="$(kubectl get secret hello-world-default-user -o jsonpath='{.data.password}' | base64 --decode)"
[root@k8s-master rabbitmq]# echo "password: $password"
password: TawohNwWLpQ0D7xmpElx9bUsL_PuIhhX


# 登录信息：
username: default_user_HGTmPURsI5VhaVjmeEQ
password: TawohNwWLpQ0D7xmpElx9bUsL_PuIhhX


kubectl port-forward "service/hello-world" 15672
```

```shell
# 系统创建service
# hello-world

[root@k8s-master rabbitmq]# kubectl get svc
NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                        AGE
hello-world         ClusterIP   10.100.123.79   <none>        15672/TCP,15692/TCP,5672/TCP   33m
hello-world-nodes   ClusterIP   None            <none>        4369/TCP,25672/TCP             33m


[root@k8s-master rabbitmq]# kubectl port-forward "service/hello-world" 15672
Forwarding from 127.0.0.1:15672 -> 15672
Forwarding from [::1]:15672 -> 15672


curl -u$username:$password localhost:15672/api/overview
{"management_version":"3.8.9","rates_mode":"basic", ...}


curl -udefault_user_HGTmPURsI5VhaVjmeEQ:TawohNwWLpQ0D7xmpElx9bUsL_PuIhhX 10.100.123.79:15672/api/overview

# 访问
[root@k8s-master rabbitmq]# curl -udefault_user_HGTmPURsI5VhaVjmeEQ:TawohNwWLpQ0D7xmpElx9bUsL_PuIhhX 10.100.123.79:15672/api/overview{"management_version":"3.8.21","rates_mode":"basic","sample_retention_policies":{"global":[600,3600,28800,86400],"basic":[600,3600],"detailed":[600]},"exchange_types":[{"name":"direct","description":"AMQP direct exchange, as per the AMQP specification","enabled":true},{"name":"fanout","description":"AMQP fanout exchange, as per the AMQP specification","enabled":true},{"name":"headers","description":"AMQP headers exchange, as per the AMQP specification","enabled":true},{"name":"topic","description":"AMQP topic exchange, as per the AMQP specification","enabled":true}],"product_version":"3.8.21","product_name":"RabbitMQ","rabbitmq_version":"3.8.21","cluster_name":"hello-world","erlang_version":"24.0.5","erlang_full_version":"Erlang/OTP 24 [erts-12.0.3] [source] [64-bit] [smp:4:2] [ds:4:2:10] [async-threads:1] [jit]","disable_stats":false,"enable_queue_totals":false,"message_stats":{},"churn_rates":{"channel_closed":0,"channel_closed_details":{"rate":0.0},"channel_created":0,"channel_created_details":{"rate":0.0},"connection_closed":312,"connection_closed_details":{"rate":0.2},"connection_created":0,"connection_created_details":{"rate":0.0},"queue_created":0,"queue_created_details":{"rate":0.0},"queue_declared":0,"queue_declared_details":{"rate":0.0},"queue_deleted":0,"queue_deleted_details":{"rate":0.0}},"queue_totals":{},"object_totals":{"channels":0,"connections":0,"consumers":0,"exchanges":7,"queues":0},"statistics_db_event_queue":0,"node":"rabbit@hello-world-server-0.hello-world-nodes.default","listeners":[{"node":"rabbit@hello-world-server-0.hello-world-nodes.default","protocol":"amqp","ip_address":"::","port":5672,"socket_opts":{"backlog":128,"nodelay":true,"linger":[true,0],"exit_on_close":false}},{"node":"rabbit@hello-world-server-0.hello-world-nodes.default","protocol":"clustering","ip_address":"::","port":25672,"socket_opts":[]},{"node":"rabbit@hello-world-server-0.hello-world-nodes.default","protocol":"http","ip_address":"::","port":15672,"socket_opts":{"cowboy_opts":{"sendfile":false},"port":15672}},{"node":"rabbit@hello-world-server-0.hello-world-nodes.default","protocol":"http/prometheus","ip_address":"::","port":15692,"socket_opts":{"cowboy_opts":{"sendfile":false},"port":15692,"protocol":"http/prometheus"}}],"contexts":[{"ssl_opts":[],"node":"rabbit@hello-world-server-0.hello-world-nodes.default","description":"RabbitMQ Management","path":"/","cowboy_opts":"[{sendfile,false}]","port":"15672"},{"ssl_opts":[],"node":"rabbit@hello-world-server-0.hello-world-nodes.default","description":"RabbitMQ Prometheus","path":"/","cowboy_opts":"[{sendfile,false}]","port":"15692","protocol":"'http/prometheus'"}]}
```



#### 1.3.访问RabbitMQ管理UI

rabbitmq-k8s-svc.yaml

```yaml
# rabbitmq-k8s-svc.yaml


apiVersion: v1
kind: Service
metadata:
  namespace: default
  name: rabbitmq-k8s-svc
  labels:
    statefulset.kubernetes.io/pod-name: hello-world-server-0
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
    statefulset.kubernetes.io/pod-name: hello-world-server-0
```

```shell
[root@k8s-master rabbitmq]# kubectl apply -f rabbitmq-svc.yaml 
service/rabbitmq-svc created


[root@k8s-master rabbitmq]# kubectl get svc
NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                        AGE
hello-world         ClusterIP   10.100.123.79   <none>        15672/TCP,15692/TCP,5672/TCP   141m
hello-world-nodes   ClusterIP   None            <none>        4369/TCP,25672/TCP             141m
rabbitmq-k8s-svc    NodePort    10.96.247.212   <none>        15672:30672/TCP                6s


curl -udefault_user_HGTmPURsI5VhaVjmeEQ:TawohNwWLpQ0D7xmpElx9bUsL_PuIhhX 10.96.247.212:15672/api/overview
```

```shell
# 地址

http://172.51.216.81:30672
username: default_user_HGTmPURsI5VhaVjmeEQ
password: TawohNwWLpQ0D7xmpElx9bUsL_PuIhhX
```



#### 1.4.删除Rabbitmq

```shell
# 删除Rabbitmq

# 根据创建选择
kubectl delete -f https://raw.githubusercontent.com/rabbitmq/cluster-operator/main/docs/examples/hello-world/rabbitmq.yaml

# 删除operator
kubectl delete -f https://github.com/rabbitmq/cluster-operator/releases/latest/download/cluster-operator.yml
```

```shell
# 查看
[root@k8s-master rabbitmq]# kubectl get all -n rabbitmq-system
NAME                                             READY   STATUS    RESTARTS   AGE
pod/rabbitmq-cluster-operator-7cbf865f89-5c4wk   1/1     Running   1          19h

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/rabbitmq-cluster-operator   1/1     1            1           19h

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/rabbitmq-cluster-operator-7cbf865f89   1         1         1       19h


[root@k8s-master rabbitmq]# kubectl get all
NAME                       READY   STATUS    RESTARTS   AGE
pod/hello-world-server-0   1/1     Running   1          19h

NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                          AGE
service/hello-world         ClusterIP   10.100.123.79    <none>        15672/TCP,15692/TCP,5672/TCP     19h
service/hello-world-nodes   ClusterIP   None             <none>        4369/TCP,25672/TCP               19h
service/rabbitmq-k8s-svc    NodePort    10.106.168.227   <none>        15672:30672/TCP,5672:30673/TCP   29m

NAME                                  READY   AGE
statefulset.apps/hello-world-server   1/1     19h

NAME                                       ALLREPLICASREADY   RECONCILESUCCESS   AGE
rabbitmqcluster.rabbitmq.com/hello-world   True               True               19h


# 删除
# 删除svc
[root@k8s-master rabbitmq]# kubectl delete -f rabbitmq-k8s-svc.yaml 
service "rabbitmq-k8s-svc" deleted

# 删除rabbitmq
[root@k8s-master rabbitmq]# kubectl delete -f https://raw.githubusercontent.com/rabbitmq/cluster-operator/main/docs/examples/hello-world/rabbitmq.yaml
rabbitmqcluster.rabbitmq.com "hello-world" deleted

[root@k8s-master rabbitmq]# kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   105d

# 删除operator
[root@k8s-master rabbitmq]# kubectl delete -f https://github.com/rabbitmq/cluster-operator/releases/latest/download/cluster-operator.yml
namespace "rabbitmq-system" deleted
customresourcedefinition.apiextensions.k8s.io "rabbitmqclusters.rabbitmq.com" deleted
serviceaccount "rabbitmq-cluster-operator" deleted
role.rbac.authorization.k8s.io "rabbitmq-cluster-leader-election-role" deleted
clusterrole.rbac.authorization.k8s.io "rabbitmq-cluster-operator-role" deleted
rolebinding.rbac.authorization.k8s.io "rabbitmq-cluster-leader-election-rolebinding" deleted
clusterrolebinding.rbac.authorization.k8s.io "rabbitmq-cluster-operator-rolebinding" deleted
deployment.apps "rabbitmq-cluster-operator" deleted

[root@k8s-master rabbitmq]# kubectl get all -n rabbitmq-system
No resources found in rabbitmq-system namespace.

# 删除PVC
[root@k8s-master rabbitmq]# kubectl delete -f persistence-hello-world-server-0.yaml 
persistentvolumeclaim "persistence-hello-world-server-0" deleted

[root@k8s-master rabbitmq]# kubectl get pvc
No resources found in default namespace.
```



#### 1.5.安装方式选择

单实例安装方式

yaml文件下载地址：https://github.com/rabbitmq/cluster-operator/tree/main/docs/examples/hello-world
yaml文件名：rabbitmq.yaml

这是最简单的RabbitmqCluster定义。唯一显式指定的属性是集群的名称。其他一切都将根据集群运营商的默认值进行配置。



examples文件夹还有许多其他引用，比如用TLS、mTLS创建RabbitMQ集群，用生产默认值设置集群，添加社区插件等等。

下载地址：https://github.com/rabbitmq/cluster-operator/tree/main/docs/examples/



**多种安装方式**

https://github.com/rabbitmq/cluster-operator/tree/main/docs/examples

```shell
[root@k8s-master examples]# pwd
/k8s/middleware/rabbitmq/cluster-operator/docs/examples
[root@k8s-master examples]# ll
total 4
drwxr-xr-x 2 root root   59 Nov 30 11:52 additionalPorts
drwxr-xr-x 2 root root   59 Nov 30 11:52 community-plugins
drwxr-xr-x 2 root root   60 Nov 30 11:52 custom-configuration
drwxr-xr-x 2 root root   59 Nov 30 11:52 default-security-context
drwxr-xr-x 2 root root  177 Nov 30 11:52 federation-over-tls
drwxr-xr-x 2 root root   60 Nov 30 11:52 hello-world
drwxr-xr-x 2 root root   99 Nov 30 11:52 import-definitions
drwxr-xr-x 2 root root   59 Nov 30 11:52 json-log
drwxr-xr-x 2 root root   60 Nov 30 11:52 mtls
drwxr-xr-x 2 root root  161 Nov 30 11:52 mtls-inter-node
drwxr-xr-x 2 root root   59 Nov 30 11:52 multiple-disks
drwxr-xr-x 2 root root  167 Nov 30 11:52 network-policies
drwxr-xr-x 2 root root   60 Nov 30 11:52 plugins
drwxr-xr-x 2 root root   60 Nov 30 11:52 pod-anti-affinity
drwxr-xr-x 2 root root  114 Nov 30 11:52 production-ready
-rw-r--r-- 1 root root 2372 Nov 30 11:52 README.md
drwxr-xr-x 2 root root   60 Nov 30 11:52 resource-limits
drwxr-xr-x 2 root root   59 Nov 30 11:52 set-login-password-username
drwxr-xr-x 2 root root   59 Nov 30 11:52 sidecar
drwxr-xr-x 2 root root   99 Nov 30 11:52 tls
drwxr-xr-x 2 root root   60 Nov 30 11:52 tolerations
drwxr-xr-x 2 root root   75 Nov 30 11:52 vault-default-user
drwxr-xr-x 2 root root   75 Nov 30 11:52 vault-tls
```

```shell
# 单实例部署
drwxr-xr-x 2 root root   60 Nov 30 11:52 hello-world


# 集群部署
drwxr-xr-x 2 root root  114 Nov 30 11:52 production-ready
```





## 三、实践



### 1.部署说明

**集群部署：**

https://github.com/rabbitmq/cluster-operator/tree/main/docs/examples

https://github.com/rabbitmq/cluster-operator/tree/main/docs/examples/production-ready



安装说明

https://github.com/rabbitmq/cluster-operator/blob/main/docs/examples/production-ready/README.md

```shell
[root@k8s-master examples]# cd production-ready/
[root@k8s-master production-ready]# ll
total 16
-rw-r--r-- 1 root root  199 Nov 30 11:52 pod-disruption-budget.yaml
-rw-r--r-- 1 root root 1184 Nov 30 11:52 rabbitmq.yaml
-rw-r--r-- 1 root root 3134 Nov 30 11:52 README.md
-rw-r--r-- 1 root root  409 Nov 30 11:52 ssd-gke.yaml


# 部署
kubectl apply -f rabbitmq.yaml
kubectl apply -f pod-disruption-budget.yaml
```

配置文件

```yaml
# rabbitmq.yaml


[root@k8s-master production-ready]# vim rabbitmq.yaml 

apiVersion: rabbitmq.com/v1beta1
kind: RabbitmqCluster
metadata:
  name: production-ready
spec:
  replicas: 3
  resources:
    requests:
      cpu: 4
      memory: 10Gi
    limits:
      cpu: 4
      memory: 10Gi
  rabbitmq:
    additionalConfig: |
      cluster_partition_handling = pause_minority
      vm_memory_high_watermark_paging_ratio = 0.99
      disk_free_limit.relative = 1.0
      collect_statistics_interval = 10000
  persistence:
    storageClassName: ssd
    storage: "500Gi"
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
            - key: app.kubernetes.io/name
              operator: In
              values:
              - production-ready
        topologyKey: kubernetes.io/hostname
  override:
    statefulSet:
      spec:
        template:
          spec:
            containers: []
            topologySpreadConstraints:
            - maxSkew: 1
              topologyKey: "topology.kubernetes.io/zone"
              whenUnsatisfiable: DoNotSchedule
              labelSelector:
                matchLabels:
                  app.kubernetes.io/name: production-ready
```

```yaml
# pod-disruption-budget.yaml 


[root@k8s-master production-ready]# vim pod-disruption-budget.yaml 

apiVersion: policy/v1beta1
kind: PodDisruptionBudget
metadata:
  name: production-ready-rabbitmq
spec:
  maxUnavailable: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: production-ready
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

```shell
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


[root@k8s-master rabbitmq]# kubectl get all -n rabbitmq-system
NAME                                             READY   STATUS    RESTARTS   AGE
pod/rabbitmq-cluster-operator-7cbf865f89-s95gq   1/1     Running   0          33s

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/rabbitmq-cluster-operator   1/1     1            1           33s

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/rabbitmq-cluster-operator-7cbf865f89   1         1         1       33s
```



**3.修改配置**

```yaml
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
    storage: "12Gi"
```



**4.使用cluster-operator创建RabbitMQ集群**

```shell
[root@k8s-master production-ready]# pwd
/k8s/middleware/rabbitmq/cluster-operator/docs/examples/production-ready
[root@k8s-master production-ready]# ll
total 16
-rw-r--r-- 1 root root  199 Nov 30 11:52 pod-disruption-budget.yaml
-rw-r--r-- 1 root root 1193 Dec  1 10:28 rabbitmq.yaml
-rw-r--r-- 1 root root 3134 Nov 30 11:52 README.md
-rw-r--r-- 1 root root  409 Nov 30 11:52 ssd-gke.yaml


# 创建
[root@k8s-master production-ready]# kubectl apply -f rabbitmq.yaml
rabbitmqcluster.rabbitmq.com/production-ready created
[root@k8s-master production-ready]# 
[root@k8s-master production-ready]# kubectl apply -f pod-disruption-budget.yaml 
poddisruptionbudget.policy/production-ready-rabbitmq created


# 查看
[root@k8s-master production-ready]# kubectl get all -owide
NAME                            READY   STATUS    RESTARTS   AGE   IP               NODE        NOMINATED NODE   READINESS GATES
pod/production-ready-server-0   1/1     Running   0          50s   10.244.36.117    k8s-node1   <none>           <none>
pod/production-ready-server-1   1/1     Running   0          50s   10.244.169.164   k8s-node2   <none>           <none>
pod/production-ready-server-2   1/1     Running   0          50s   10.244.107.251   k8s-node3   <none>           <none>

NAME                             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                        AGE    SELECTOR
service/kubernetes               ClusterIP   10.96.0.1       <none>        443/TCP                        105d   <none>
service/production-ready         ClusterIP   10.111.187.65   <none>        5672/TCP,15672/TCP,15692/TCP   50s    app.kubernetes.io/name=production-ready
service/production-ready-nodes   ClusterIP   None            <none>        4369/TCP,25672/TCP             50s    app.kubernetes.io/name=production-ready

NAME                                       READY   AGE   CONTAINERS   IMAGES
statefulset.apps/production-ready-server   3/3     50s   rabbitmq     rabbitmq:3.8.21-management

NAME                                            ALLREPLICASREADY   RECONCILESUCCESS   AGE
rabbitmqcluster.rabbitmq.com/production-ready   True               True               50s


[root@k8s-master production-ready]# kubectl get all -n rabbitmq-system
NAME                                             READY   STATUS    RESTARTS   AGE
pod/rabbitmq-cluster-operator-7cbf865f89-s95gq   1/1     Running   0          105m

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/rabbitmq-cluster-operator   1/1     1            1           105m

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/rabbitmq-cluster-operator-7cbf865f89   1         1         1       105m


# PVC
[root@k8s-master production-ready]# kubectl get pvc
NAME                                    STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
persistence-production-ready-server-0   Bound    pvc-71d4c7b8-554c-4f9e-856e-325e80ed88f2   12Gi       RWO            rook-ceph-block   6m10s
persistence-production-ready-server-1   Bound    pvc-6be2c9b3-a0b0-43ae-af1f-74ae92bce5ce   12Gi       RWO            rook-ceph-block   6m10s
persistence-production-ready-server-2   Bound    pvc-2243a65e-f38e-4164-a2f1-ec0a65be486d   12Gi       RWO            rook-ceph-block   6m10s
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


[root@k8s-master production-ready]# username="$(kubectl get secret production-ready-default-user -o jsonpath='{.data.username}' | base64 --decode)"
[root@k8s-master production-ready]# echo "username: $username"
username: default_user_vmOg0aiF8VarwVhb0sE


[root@k8s-master production-ready]# password="$(kubectl get secret production-ready-default-user -o jsonpath='{.data.password}' | base64 --decode)"
[root@k8s-master production-ready]# echo "password: $password"
password: rKr9N2v-Cxrec78zo1LUnigjyOla_YLv


# 登录信息：
username: default_user_vmOg0aiF8VarwVhb0sE
password: rKr9N2v-Cxrec78zo1LUnigjyOla_YLv


kubectl port-forward "service/production-ready" 15672
```

```shell
# 系统创建service
# production-ready

[root@k8s-master production-ready]# kubectl get svc
NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                        AGE
production-ready         ClusterIP   10.111.187.65   <none>        5672/TCP,15672/TCP,15692/TCP   6m48s
production-ready-nodes   ClusterIP   None            <none>        4369/TCP,25672/TCP             6m48s


[root@k8s-master production-ready]# kubectl port-forward "service/production-ready" 15672
Forwarding from 127.0.0.1:15672 -> 15672
Forwarding from [::1]:15672 -> 15672


curl -u$username:$password localhost:15672/api/overview
{"management_version":"3.8.9","rates_mode":"basic", ...}


curl -udefault_user_vmOg0aiF8VarwVhb0sE:rKr9N2v-Cxrec78zo1LUnigjyOla_YLv 10.111.187.65:15672/api/overview


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
[root@k8s-master rabbitmq]# kubectl apply -f rabbitmq-svc.yaml 
service/rabbitmq-svc created


[root@k8s-master rabbitmq]# kubectl get svc
NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                          AGE
production-ready         ClusterIP   10.111.187.65   <none>        5672/TCP,15672/TCP,15692/TCP     10m
production-ready-nodes   ClusterIP   None            <none>        4369/TCP,25672/TCP               10m
rabbitmq-cluster-svc     NodePort    10.106.36.141   <none>        15672:30672/TCP,5672:30673/TCP   5s


curl -udefault_user_vmOg0aiF8VarwVhb0sE:rKr9N2v-Cxrec78zo1LUnigjyOla_YLv 10.106.36.141:15672/api/overview
```

```shell
# 地址

http://172.51.216.81:30672
username: default_user_vmOg0aiF8VarwVhb0sE
password: rKr9N2v-Cxrec78zo1LUnigjyOla_YLv
```

![](/images/kubernetes/middleware/rabbitmq-1.png)



### 5.扩容

**修改配置**

```yaml
# rabbitmq.yaml
# cpu、memory
# storageClassName: rook-ceph-block


[root@k8s-master production-ready]# vim rabbitmq.yaml 

apiVersion: rabbitmq.com/v1beta1
kind: RabbitmqCluster
metadata:
  name: production-ready
spec:
  replicas: 4
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
    storage: "12Gi"
```

```shell
[root@k8s-master production-ready]# kubectl apply -f rabbitmq.yaml 
rabbitmqcluster.rabbitmq.com/production-ready configured


# 查看
[root@k8s-master production-ready]# kubectl get all -owide
NAME                            READY   STATUS    RESTARTS   AGE   IP               NODE        NOMINATED NODE   READINESS GATES
pod/production-ready-server-0   1/1     Running   0          24m   10.244.36.117    k8s-node1   <none>           <none>
pod/production-ready-server-1   1/1     Running   0          24m   10.244.169.164   k8s-node2   <none>           <none>
pod/production-ready-server-2   1/1     Running   0          24m   10.244.107.251   k8s-node3   <none>           <none>
pod/production-ready-server-3   1/1     Running   0          45s   10.244.169.159   k8s-node2   <none>           <none>

NAME                             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                          AGE    SELECTOR
service/kubernetes               ClusterIP   10.96.0.1       <none>        443/TCP                          105d   <none>
service/production-ready         ClusterIP   10.111.187.65   <none>        5672/TCP,15672/TCP,15692/TCP     24m    app.kubernetes.io/name=production-ready
service/production-ready-nodes   ClusterIP   None            <none>        4369/TCP,25672/TCP               24m    app.kubernetes.io/name=production-ready
service/rabbitmq-cluster-svc     NodePort    10.106.36.141   <none>        15672:30672/TCP,5672:30673/TCP   14m    app.kubernetes.io/name=production-ready

NAME                                       READY   AGE   CONTAINERS   IMAGES
statefulset.apps/production-ready-server   4/4     24m   rabbitmq     rabbitmq:3.8.21-management

NAME                                            ALLREPLICASREADY   RECONCILESUCCESS   AGE
rabbitmqcluster.rabbitmq.com/production-ready   True               True               24m


[root@k8s-master production-ready]# kubectl get pvc
NAME                                    STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
persistence-production-ready-server-0   Bound    pvc-71d4c7b8-554c-4f9e-856e-325e80ed88f2   12Gi       RWO            rook-ceph-block   27m
persistence-production-ready-server-1   Bound    pvc-6be2c9b3-a0b0-43ae-af1f-74ae92bce5ce   12Gi       RWO            rook-ceph-block   27m
persistence-production-ready-server-2   Bound    pvc-2243a65e-f38e-4164-a2f1-ec0a65be486d   12Gi       RWO            rook-ceph-block   27m
persistence-production-ready-server-3   Bound    pvc-26fca32f-d0d9-44af-8378-1a2aeefb1d94   12Gi       RWO            rook-ceph-block   3m21s
```

![](/images/kubernetes/middleware/rabbitmq-2.png)



### 6.缩容

**修改配置**

```yaml
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
    storage: "12Gi"
```

```shell
[root@k8s-master production-ready]# kubectl apply -f rabbitmq.yaml 
rabbitmqcluster.rabbitmq.com/production-ready configured


# 查看
[root@k8s-master production-ready]# kubectl get all -owide
NAME                            READY   STATUS    RESTARTS   AGE   IP               NODE        NOMINATED NODE   READINESS GATES
pod/production-ready-server-0   1/1     Running   0          24m   10.244.36.117    k8s-node1   <none>           <none>
pod/production-ready-server-1   1/1     Running   0          24m   10.244.169.164   k8s-node2   <none>           <none>
pod/production-ready-server-2   1/1     Running   0          24m   10.244.107.251   k8s-node3   <none>           <none>
pod/production-ready-server-3   1/1     Running   0          45s   10.244.169.159   k8s-node2   <none>           <none>

NAME                             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                          AGE    SELECTOR
service/kubernetes               ClusterIP   10.96.0.1       <none>        443/TCP                          105d   <none>
service/production-ready         ClusterIP   10.111.187.65   <none>        5672/TCP,15672/TCP,15692/TCP     24m    app.kubernetes.io/name=production-ready
service/production-ready-nodes   ClusterIP   None            <none>        4369/TCP,25672/TCP               24m    app.kubernetes.io/name=production-ready
service/rabbitmq-cluster-svc     NodePort    10.106.36.141   <none>        15672:30672/TCP,5672:30673/TCP   14m    app.kubernetes.io/name=production-ready

NAME                                       READY   AGE   CONTAINERS   IMAGES
statefulset.apps/production-ready-server   4/4     24m   rabbitmq     rabbitmq:3.8.21-management

NAME                                            ALLREPLICASREADY   RECONCILESUCCESS   AGE
rabbitmqcluster.rabbitmq.com/production-ready   True               True               24m


[root@k8s-master production-ready]# kubectl get pvc
NAME                                    STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
persistence-production-ready-server-0   Bound    pvc-71d4c7b8-554c-4f9e-856e-325e80ed88f2   12Gi       RWO            rook-ceph-block   27m
persistence-production-ready-server-1   Bound    pvc-6be2c9b3-a0b0-43ae-af1f-74ae92bce5ce   12Gi       RWO            rook-ceph-block   27m
persistence-production-ready-server-2   Bound    pvc-2243a65e-f38e-4164-a2f1-ec0a65be486d   12Gi       RWO            rook-ceph-block   27m
persistence-production-ready-server-3   Bound    pvc-26fca32f-d0d9-44af-8378-1a2aeefb1d94   12Gi       RWO            rook-ceph-block   3m21s
```

![](/images/kubernetes/middleware/rabbitmq-2.png)

**缩容不起作用。**



### 7.Spring Boot测试

#### 7.1.服务配置

msa-k8s-rabbitmq.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-k8s-rabbitmq
  labels:
    app: msa-k8s-rabbitmq
spec:
  type: ClusterIP
  ports:
    - port: 8143
      targetPort: 8143
  selector:
    app: msa-k8s-rabbitmq

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-k8s-rabbitmq
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-k8s-rabbitmq
  template:
    metadata:
      labels:
        app: msa-k8s-rabbitmq
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-k8s-redis
          image: 172.51.216.85:8888/springcloud/msa-k8s-rabbitmq:1.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8143
          env:
            - name: ACTIVE
              value: "-Dspring.profiles.active=dev"
```



application-k8s.yml

```yaml
# Redis集群配置


server:
  port: 8143

spring:
  application:
    name: msa-k8s-rabbitmq
  #配置rabbitMq 服务器
  rabbitmq:
    host: production-ready.default.svc.cluster.local
    port: 5672
    username: default_user_IdbGSVeW1plrpskejhI
    password: mYaPNH_Hlb-g1rNYMiPWGdKex-3l8po5
    #虚拟host 可以不设置,使用server默认host
    #virtual-host: JCcccHost
    

# 说明
# 集群内部访问：
production-ready
#完整配置
host: production-ready.default.svc.cluster.local
# 或
host: rabbitmq-cluster-svc.default.svc.cluster.local


[root@k8s-master rabbitmq]# kubectl get svc
NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                          AGE
production-ready         ClusterIP   10.102.238.59   <none>        5672/TCP,15672/TCP,15692/TCP     89m
production-ready-nodes   ClusterIP   None            <none>        4369/TCP,25672/TCP               89m
rabbitmq-cluster-svc     NodePort    10.106.36.141   <none>        15672:30672/TCP,5672:30673/TCP   145m


# 集群外部访问，需要每个实例配置一个service,暴露端口
# 参考
erver:
  port: 8143

spring:
  application:
    name: msa-k8s-rabbitmq
  rabbitmq:
    # 集群地址，用逗号分隔
    addresses: 192.168.11.71:5672,192.168.11.72:5672,192.168.11.71:5673
    connection-timeout: 15000
    password: guest
    # 使用启用消息确认模式
    publisher-confirms: true
    username: guest
    virtual-host: /
 
 
 # 配置每个实例的service  
 addresses: 192.168.11.71:5672,192.168.11.72:5672,192.168.11.71:5673 
```



#### 7.2.创建服务

```shell
docker build -t 172.51.216.85:8888/springcloud/msa-k8s-rabbitmq:1.1.0 .

docker push 172.51.216.85:8888/springcloud/msa-k8s-rabbitmq:1.1.0


[root@k8s-master operator]# kubectl apply -f msa-k8s-rabbitmq.yaml 
service/msa-k8s-redis created
deployment.apps/msa-k8s-redis created


root@k8s-master rabbitmq]# kubectl get all -n dev | grep msa-k8s-rabbitmq
pod/msa-k8s-rabbitmq-84976bbd47-8mh6p      1/1     Running            0          8m25s
pod/msa-k8s-rabbitmq-84976bbd47-9swjn      1/1     Running            0          8m25s
pod/msa-k8s-rabbitmq-84976bbd47-nsqbt      1/1     Running            0          8m25s

service/msa-k8s-rabbitmq         ClusterIP   10.107.32.2      <none>        8143/TCP          8m25s

deployment.apps/msa-k8s-rabbitmq      3/3     3            3           8m25s

replicaset.apps/msa-k8s-rabbitmq-84976bbd47      3         3         3       8m25s
```

```shell
# 测试

curl 10.107.32.2:8143/send


[root@k8s-master rabbitmq]# curl 10.107.32.2:8143/send
发送消息成功！
[root@k8s-master rabbitmq]# curl 10.107.32.2:8143/send
发送消息成功！


[root@k8s-master rabbitmq]# kubectl logs -f msa-k8s-rabbitmq-84976bbd47-8mh6p
Error from server (NotFound): pods "msa-k8s-rabbitmq-84976bbd47-8mh6p" not found
[root@k8s-master rabbitmq]# kubectl logs -f msa-k8s-rabbitmq-84976bbd47-8mh6p -n dev
OpenJDK 64-Bit Server VM warning: ignoring option PermSize=512M; support was removed in 8.0
OpenJDK 64-Bit Server VM warning: ignoring option MaxPermSize=512m; support was removed in 8.0

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::       (v2.3.10.RELEASE)

2021-12-01 14:15:22.184  INFO 1 --- [           main] com.k8s.msa.MsaK8sRabbitmqApplication    : Starting MsaK8sRabbitmqApplication v1.0.0 on msa-k8s-rabbitmq-84976bbd47-8mh6p with PID 1 (/app.jar started by root in /)
2021-12-01 14:15:22.189  INFO 1 --- [           main] com.k8s.msa.MsaK8sRabbitmqApplication    : The following profiles are active: dev
2021-12-01 14:15:23.880  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8143 (http)
2021-12-01 14:15:23.896  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2021-12-01 14:15:23.896  INFO 1 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.45]
2021-12-01 14:15:23.999  INFO 1 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2021-12-01 14:15:23.999  INFO 1 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1721 ms
2021-12-01 14:15:24.898  INFO 1 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2021-12-01 14:15:25.349  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8143 (http) with context path ''
2021-12-01 14:15:25.351  INFO 1 --- [           main] o.s.a.r.c.CachingConnectionFactory       : Attempting to connect to: [production-ready.default.svc.cluster.local:5672]
2021-12-01 14:15:25.447  INFO 1 --- [           main] o.s.a.r.c.CachingConnectionFactory       : Created new connection: rabbitConnectionFactory#185a6e9:0/SimpleConnection@6cd28fa7 [delegate=amqp://default_user_IdbGSVeW1plrpskejhI@10.102.238.59:5672/, localPort= 33022]
2021-12-01 14:15:25.530  INFO 1 --- [           main] com.k8s.msa.MsaK8sRabbitmqApplication    : Started MsaK8sRabbitmqApplication in 4.14 seconds (JVM running for 4.824)
2021-12-01 14:16:30.850  INFO 1 --- [nio-8143-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2021-12-01 14:16:30.850  INFO 1 --- [nio-8143-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2021-12-01 14:16:30.856  INFO 1 --- [nio-8143-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 6 ms
发送消息成功！：send，this is first messge！ $$$ -----V2Wed Dec 01 14:16:30 CST 2021
发送消息成功！：send，this is first messge！ $$$ -----V2Wed Dec 01 14:16:32 CST 2021
消费者消费消息了：send，this is first messge！ $$$ -----V2Wed Dec 01 14:16:32 CST 2021
发送消息成功！：send，this is first messge！ $$$ -----V2Wed Dec 01 14:16:35 CST 2021
消费者消费消息了：send，this is first messge！ $$$ -----V2Wed Dec 01 14:16:35 CST 2021
发送消息成功！：send，this is first messge！ $$$ -----V2Wed Dec 01 14:16:38 CST 2021
消费者消费消息了：send，this is first messge！ $$$ -----V2Wed Dec 01 14:16:40 CST 2021
发送消息成功！：send，this is first messge！ $$$ -----V2Wed Dec 01 14:16:41 CST 2021
消费者消费消息了：send，this is first messge！ $$$ -----V2Wed Dec 01 14:16:44 CST 2021
```



#### 7.3.总结

**1.开发测试：**

每个RabbitMQ实例单独配置service，以NodePort方式暴露端口



**2.集群内服访问**

```shell
# Spring Boot

# 集群内部访问：
production-ready
#完整配置
host: production-ready.default.svc.cluster.local
# 或
host: rabbitmq-cluster-svc.default.svc.cluster.local
```



### 8.删除集群

```shell
# 删除Rabbitmq

[root@k8s-master production-ready]# pwd
/k8s/middleware/rabbitmq/cluster-operator/docs/examples/production-ready
[root@k8s-master production-ready]# ll
total 16
-rw-r--r-- 1 root root  199 Nov 30 11:52 pod-disruption-budget.yaml
-rw-r--r-- 1 root root  500 Dec  1 12:14 rabbitmq.yaml
-rw-r--r-- 1 root root 3134 Nov 30 11:52 README.md
-rw-r--r-- 1 root root  409 Nov 30 11:52 ssd-gke.yaml


# 删除
kubectl delete -f pod-disruption-budget.yaml

kubectl delete -f rabbitmq.yaml

# 删除operator
kubectl delete -f https://github.com/rabbitmq/cluster-operator/releases/latest/download/cluster-operator.yml
```

```shell
# 查看
[root@k8s-master production-ready]# kubectl get all -n rabbitmq-system
NAME                                             READY   STATUS    RESTARTS   AGE
pod/rabbitmq-cluster-operator-7cbf865f89-s95gq   1/1     Running   0          4h31m

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/rabbitmq-cluster-operator   1/1     1            1           4h31m

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/rabbitmq-cluster-operator-7cbf865f89   1         1         1       4h31m


[root@k8s-master production-ready]# kubectl get all
NAME                            READY   STATUS    RESTARTS   AGE
pod/production-ready-server-0   1/1     Running   0          102m
pod/production-ready-server-1   1/1     Running   0          102m
pod/production-ready-server-2   1/1     Running   0          102m

NAME                             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                          AGE
service/kubernetes               ClusterIP   10.96.0.1       <none>        443/TCP                          106d
service/production-ready         ClusterIP   10.102.238.59   <none>        5672/TCP,15672/TCP,15692/TCP     102m
service/production-ready-nodes   ClusterIP   None            <none>        4369/TCP,25672/TCP               102m
service/rabbitmq-cluster-svc     NodePort    10.106.36.141   <none>        15672:30672/TCP,5672:30673/TCP   158m

NAME                                       READY   AGE
statefulset.apps/production-ready-server   3/3     102m

NAME                                            ALLREPLICASREADY   RECONCILESUCCESS   AGE
rabbitmqcluster.rabbitmq.com/production-ready   True               True               102m


[root@k8s-master production-ready]# kubectl get pvc
NAME                                    STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
persistence-production-ready-server-0   Bound    pvc-0c3c4767-669c-497e-a6a7-c6ee4b6ff283   12Gi       RWO            rook-ceph-block   102m
persistence-production-ready-server-1   Bound    pvc-8ecc6b70-26ff-483d-a456-2325dd4376b3   12Gi       RWO            rook-ceph-block   102m
persistence-production-ready-server-2   Bound    pvc-6a9f3349-0396-46c1-a7a9-7fd8ad9c218b   12Gi       RWO            rook-ceph-block   102m


# 删除
# 删除svc
[root@k8s-master rabbitmq]# kubectl delete -f rabbitmq-cluster-svc.yaml 
service "rabbitmq-cluster-svc" deleted

# 删除rabbitmq
[root@k8s-master production-ready]# kubectl delete -f pod-disruption-budget.yaml 
poddisruptionbudget.policy "production-ready-rabbitmq" deleted
[root@k8s-master production-ready]# kubectl delete -f rabbitmq.yaml 
rabbitmqcluster.rabbitmq.com "production-ready" deleted

[root@k8s-master rabbitmq]# kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   105d


# 删除operator
[root@k8s-master rabbitmq]# kubectl delete -f https://github.com/rabbitmq/cluster-operator/releases/latest/download/cluster-operator.yml
namespace "rabbitmq-system" deleted
customresourcedefinition.apiextensions.k8s.io "rabbitmqclusters.rabbitmq.com" deleted
serviceaccount "rabbitmq-cluster-operator" deleted
role.rbac.authorization.k8s.io "rabbitmq-cluster-leader-election-role" deleted
clusterrole.rbac.authorization.k8s.io "rabbitmq-cluster-operator-role" deleted
rolebinding.rbac.authorization.k8s.io "rabbitmq-cluster-leader-election-rolebinding" deleted
clusterrolebinding.rbac.authorization.k8s.io "rabbitmq-cluster-operator-rolebinding" deleted
deployment.apps "rabbitmq-cluster-operator" deleted

[root@k8s-master rabbitmq]# kubectl get all -n rabbitmq-system
No resources found in rabbitmq-system namespace.

[root@k8s-master rabbitmq]# kubectl get pvc
No resources found in default namespace.
```



