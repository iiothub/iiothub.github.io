* TOC
{:toc}


## 日志集合 - ELK



### 1.Docker部署EFK

#### 1.1.部署方式

在K8S内部部署Fluentd，Elasticsearch、Kibana部署在K8S集群外部。

Fluentd配置连接外部ES集群。

**版本：elasticsearch:7.4.2**



#### 1.2.下载EFK源码

**注意：EFK版本要与Kubernetes版本一致**

```shell
# fluentd-elasticsearch
https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/fluentd-elasticsearch


# 下载源码（与k8s版本对应）
https://codeload.github.com/kubernetes/kubernetes/tar.gz/refs/tags/v1.20.6
```

```shell
[root@k8s-master1 efk]# tar -zxf kubernetes-1.20.6.tar.gz


[root@k8s-master1 fluentd-elasticsearch]# pwd
/k8s/operation/efk/kubernetes-1.20.6/cluster/addons/fluentd-elasticsearch
[root@k8s-master1 fluentd-elasticsearch]# ll
total 48
drwxrwxr-x. 3 root root   169 Apr 15  2021 es-image
-rw-rw-r--. 1 root root   580 Apr 15  2021 es-service.yaml
-rw-rw-r--. 1 root root  3186 Apr 15  2021 es-statefulset.yaml
-rw-rw-r--. 1 root root 16125 Apr 15  2021 fluentd-es-configmap.yaml
-rw-rw-r--. 1 root root  2581 Apr 15  2021 fluentd-es-ds.yaml
drwxrwxr-x. 2 root root   112 Apr 15  2021 fluentd-es-image
-rw-rw-r--. 1 root root  1519 Apr 15  2021 kibana-deployment.yaml
-rw-rw-r--. 1 root root   354 Apr 15  2021 kibana-service.yaml
-rw-rw-r--. 1 root root   189 Apr 15  2021 OWNERS
drwxrwxr-x. 2 root root    33 Apr 15  2021 podsecuritypolicies
-rw-rw-r--. 1 root root  4550 Apr 15  2021 README.md



[root@k8s-master efk]# ll
total 48
# es
-rw-r--r-- 1 root root   580 Nov 19 16:41 es-service.yaml
-rw-r--r-- 1 root root  3186 Nov 19 16:41 es-statefulset.yaml

# fluentd
-rw-r--r-- 1 root root 16125 Nov 19 16:41 fluentd-es-configmap.yaml
-rw-r--r-- 1 root root  2581 Nov 19 16:41 fluentd-es-ds.yaml

# kibana
-rw-r--r-- 1 root root  1519 Nov 19 16:41 kibana-deployment.yaml
-rw-r--r-- 1 root root   354 Nov 19 16:41 kibana-service.yaml
```



#### 1.3.Docker部署Elasticsearch

```shell
# 下载镜像 查看镜像
docker pull elasticsearch:7.4.2


# 运行 elasticsearch
docker run -d --name elasticsearch --network host --restart=always \
-e "discovery.type=single-node" elasticsearch:7.4.2
 

# 检测 elasticsearch 是否启动成功
curl 127.0.0.1:9200

[root@localhost ~]# curl 127.0.0.1:9200
{
  "name" : "localhost.localdomain",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "evkv2ZA3QLGOVyKKKqFGvg",
  "version" : {
    "number" : "7.4.2",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "2f90bbf7b93631e52bafb59b3b049cb44ec25e96",
    "build_date" : "2019-10-28T20:40:44.881551Z",
    "build_snapshot" : false,
    "lucene_version" : "8.2.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

```shell
ES地址：
http://172.51.216.88:9200
```



#### 1.4.Docker部署Kibana

```shell
# 下载镜像 查看镜像
docker pull kibana:7.4.2
 

# 运行 Kibana
docker run -d --name kibana --network host --restart=always kibana:7.4.2



# 修改配置文件
docker exec -it kibana /bin/bash


/usr/share/kibana/config/kibana.yml

server.name: kibana
server.host: "0"
elasticsearch.hosts: [ "http://172.51.216.88:9200" ]
xpack.monitoring.ui.container.elasticsearch.enabled: true



# 重启docker
docker restart kibana
```

```shell
#Kibana地址：

http://172.51.216.88:5601
```



#### 1.5.k8s安装Fluentd

```shell
[root@k8s-master1 fluentd-elasticsearch]# pwd
/k8s/operation/efk/kubernetes-1.20.6/cluster/addons/fluentd-elasticsearch
[root@k8s-master1 fluentd-elasticsearch]# ll
total 48
drwxrwxr-x. 3 root root   169 Apr 15  2021 es-image
-rw-rw-r--. 1 root root   580 Apr 15  2021 es-service.yaml
-rw-rw-r--. 1 root root  3186 Apr 15  2021 es-statefulset.yaml
-rw-rw-r--. 1 root root 16125 Apr 15  2021 fluentd-es-configmap.yaml
-rw-rw-r--. 1 root root  2581 Apr 15  2021 fluentd-es-ds.yaml
drwxrwxr-x. 2 root root   112 Apr 15  2021 fluentd-es-image
-rw-rw-r--. 1 root root  1519 Apr 15  2021 kibana-deployment.yaml
-rw-rw-r--. 1 root root   354 Apr 15  2021 kibana-service.yaml
-rw-rw-r--. 1 root root   189 Apr 15  2021 OWNERS
drwxrwxr-x. 2 root root    33 Apr 15  2021 podsecuritypolicies
-rw-rw-r--. 1 root root  4550 Apr 15  2021 README.md


# fluentd
-rw-r--r-- 1 root root 16125 Nov 19 16:41 fluentd-es-configmap.yaml
-rw-r--r-- 1 root root  2581 Nov 19 16:41 fluentd-es-ds.yaml
```



**1.修改配置**

```yaml
# fluentd-es-configmap.yaml


[root@k8s-master efk]# vim fluentd-es-configmap.yaml

  output.conf: |-
    <match **>
      @id elasticsearch
      @type elasticsearch
      @log_level info
      type_name _doc
      include_tag_key true
      
      # 外部ES端点配置
      host ext-elasticsearch.ext.svc.cluster.local
      
      port 9200
      logstash_format true
      <buffer>
        @type file
        path /var/log/fluentd-buffers/kubernetes.system.buffer
        flush_mode interval
        retry_type exponential_backoff
        flush_thread_count 2
        flush_interval 5s
        retry_forever
        retry_max_interval 30
        chunk_limit_size 2M
        total_limit_size 500M
        overflow_action block
      </buffer>
    </match>
```



**2.安装Fluentd**

```shell
# 安装

[root@k8s-master1 fluentd-elasticsearch]# kubectl apply -f fluentd-es-configmap.yaml
configmap/fluentd-es-config-v0.2.0 created


[root@k8s-master1 fluentd-elasticsearch]# kubectl apply -f fluentd-es-ds.yaml
serviceaccount/fluentd-es created
clusterrole.rbac.authorization.k8s.io/fluentd-es created
clusterrolebinding.rbac.authorization.k8s.io/fluentd-es created
daemonset.apps/fluentd-es-v3.1.0 created


# 查看
[root@k8s-master1 fluentd-elasticsearch]# kubectl get pod -n kube-system -o wide
NAME                                       READY   STATUS    RESTARTS   AGE    IP               NODE          NOMINATED NODE   READINESS GATES
fluentd-es-v3.1.0-d75cr                    1/1     Running   0          83s    10.244.169.131   k8s-node2     <none>           <none>
fluentd-es-v3.1.0-s55x6                    1/1     Running   0          83s    10.244.36.68     k8s-node1     <none>           <none>
fluentd-es-v3.1.0-vq44f                    1/1     Running   0          83s    10.244.107.194   k8s-node3     <none>           <none>
fluentd-es-v3.1.0-x75nm                    1/1     Running   0          83s    10.244.122.119   k8s-node4     <none>           <none>
......
```



#### 1.6.测试

```shell
# 访问地址：

http://172.51.216.88:5601/
```

![](/images/kubernetes/pro/monitoring/efk-8.png)

![](/images/kubernetes/pro/monitoring/efk-9.png)





### 2.K8S部署EFK

#### 2.1.EFK

```shell
# 官方地址

# kubernetes
https://github.com/kubernetes/kubernetes/tree/master/cluster/addons

# fluentd-elasticsearch
https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/fluentd-elasticsearch

# 下载源码（与k8s版本对应）
https://codeload.github.com/kubernetes/kubernetes/tar.gz/refs/tags/v1.20.6


https://kubernetes.io/docs/concepts/cluster-administration/logging/
```



**下载源码**

**注意：EFK版本要与Kubernetes版本一致**

```shell
# fluentd-elasticsearch
https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/fluentd-elasticsearch


# 下载源码（与k8s版本对应）
https://codeload.github.com/kubernetes/kubernetes/tar.gz/refs/tags/v1.20.6
```

```shell
[root@k8s-master efk]# ll
total 48
drwxr-xr-x 3 root root   169 Nov 19 16:41 es-image
-rw-r--r-- 1 root root   580 Nov 19 16:41 es-service.yaml
-rw-r--r-- 1 root root  3186 Nov 19 16:41 es-statefulset.yaml
-rw-r--r-- 1 root root 16125 Nov 19 16:41 fluentd-es-configmap.yaml
-rw-r--r-- 1 root root  2581 Nov 19 16:41 fluentd-es-ds.yaml
drwxr-xr-x 2 root root   112 Nov 19 16:41 fluentd-es-image
-rw-r--r-- 1 root root  1519 Nov 19 16:41 kibana-deployment.yaml
-rw-r--r-- 1 root root   354 Nov 19 16:41 kibana-service.yaml
-rw-r--r-- 1 root root   189 Nov 19 16:41 OWNERS
drwxr-xr-x 2 root root    33 Nov 19 16:41 podsecuritypolicies
-rw-r--r-- 1 root root  4550 Nov 19 16:41 README.md



[root@k8s-master efk]# ll
total 48
# es
-rw-r--r-- 1 root root   580 Nov 19 16:41 es-service.yaml
-rw-r--r-- 1 root root  3186 Nov 19 16:41 es-statefulset.yaml

# fluentd
-rw-r--r-- 1 root root 16125 Nov 19 16:41 fluentd-es-configmap.yaml
-rw-r--r-- 1 root root  2581 Nov 19 16:41 fluentd-es-ds.yaml

# kibana
-rw-r--r-- 1 root root  1519 Nov 19 16:41 kibana-deployment.yaml
-rw-r--r-- 1 root root   354 Nov 19 16:41 kibana-service.yaml
```



#### 2.2.安装Elasticsearch

##### 2.2.1.修改配置



**默认安装，不修改配置，不配置存储。**

**修改：副本改成3个，配置ceph的存储方式。**

**ES部署方式是StatefulSet，选择有状态服务的存储方式，块存储（RBD）。**



```yaml
# es-statefulset.yaml

# 1.修改：replicas: 3
# 2.注释：      
#volumes:
#  - name: elasticsearch-logging
#    emptyDir: {}
# 3.增加配置
  volumeClaimTemplates:
  - metadata:
      name: elasticsearch-logging
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "rook-ceph-block"
      resources:
        requests:
          storage: 30Gi
          
          

# ES完整配置
[root@k8s-master efk]# vim es-statefulset.yaml
......
---
# Elasticsearch deployment itself
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: elasticsearch-logging
  namespace: kube-system
  labels:
    k8s-app: elasticsearch-logging
    version: v7.4.3
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  serviceName: elasticsearch-logging
  replicas: 3
  selector:
    matchLabels:
      k8s-app: elasticsearch-logging
      version: v7.4.3
  template:
    metadata:
      labels:
        k8s-app: elasticsearch-logging
        version: v7.4.3
    spec:
      serviceAccountName: elasticsearch-logging
      containers:
        - image: quay.io/fluentd_elasticsearch/elasticsearch:v7.4.3
          name: elasticsearch-logging
          imagePullPolicy: Always
          resources:
            # need more cpu upon initialization, therefore burstable class
            limits:
              cpu: 1000m
              memory: 3Gi
            requests:
              cpu: 100m
              memory: 3Gi
          ports:
            - containerPort: 9200
              name: db
              protocol: TCP
            - containerPort: 9300
              name: transport
              protocol: TCP
          livenessProbe:
            tcpSocket:
              port: transport
            initialDelaySeconds: 5
            timeoutSeconds: 10
          readinessProbe:
            tcpSocket:
              port: transport
            initialDelaySeconds: 5
            timeoutSeconds: 10
          volumeMounts:
            - name: elasticsearch-logging
              mountPath: /data
          env:
            - name: "NAMESPACE"
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
            - name: "MINIMUM_MASTER_NODES"
              value: "1"
      #volumes:
      #  - name: elasticsearch-logging
      #    emptyDir: {}
      # Elasticsearch requires vm.max_map_count to be at least 262144.
      # If your OS already sets up this number to a higher value, feel free
      # to remove this init container.
      initContainers:
        - image: alpine:3.6
          command: ["/sbin/sysctl", "-w", "vm.max_map_count=262144"]
          name: elasticsearch-logging-init
          securityContext:
            privileged: true
  volumeClaimTemplates:
  - metadata:
      name: elasticsearch-logging
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "rook-ceph-block"
      resources:
        requests:
          storage: 30Gi
```



##### 2.2.2.安装

```shell
kubectl apply -f es-statefulset.yaml

kubectl apply -f es-service.yaml


# 创建
[root@k8s-master1 fluentd-elasticsearch]# kubectl apply -f es-statefulset.yaml 
serviceaccount/elasticsearch-logging created
clusterrole.rbac.authorization.k8s.io/elasticsearch-logging created
clusterrolebinding.rbac.authorization.k8s.io/elasticsearch-logging created
statefulset.apps/elasticsearch-logging created


[root@k8s-master1 fluentd-elasticsearch]# kubectl apply -f es-service.yaml
service/elasticsearch-logging created


# 查看
[root@k8s-master1 fluentd-elasticsearch]# kubectl get pvc -n kube-system
NAME                                            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
elasticsearch-logging-elasticsearch-logging-0   Bound    pvc-53a9d8e6-d857-4824-8627-a0e3ea9b2cca   30Gi       RWO            rook-ceph-block   44m
elasticsearch-logging-elasticsearch-logging-1   Bound    pvc-13113c7d-7dbd-4e0c-968c-6c86d0eeb5a3   30Gi       RWO            rook-ceph-block   11m
elasticsearch-logging-elasticsearch-logging-2   Bound    pvc-6301b623-08ad-442d-9682-7803eaac8a2c   30Gi       RWO            rook-ceph-block   3m30s


[root@k8s-master1 fluentd-elasticsearch]# kubectl get pv -n kube-system
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                                       STORAGECLASS      REASON   AGE
pvc-13113c7d-7dbd-4e0c-968c-6c86d0eeb5a3   30Gi       RWO            Delete           Bound    kube-system/elasticsearch-logging-elasticsearch-logging-1   rook-ceph-block            11m
pvc-53a9d8e6-d857-4824-8627-a0e3ea9b2cca   30Gi       RWO            Delete           Bound    kube-system/elasticsearch-logging-elasticsearch-logging-0   rook-ceph-block            44m
pvc-6301b623-08ad-442d-9682-7803eaac8a2c   30Gi       RWO            Delete           Bound    kube-system/elasticsearch-logging-elasticsearch-logging-2   rook-ceph-block            3m36s
```



#### 2.3.安装Fluentd

```shell
# 直接安装，不配置存储


kubectl apply -f fluentd-es-configmap.yaml
kubectl apply -f fluentd-es-ds.yaml 
```

```shell
# 查看


[root@k8s-master1 fluentd-elasticsearch]#  kubectl get all -n kube-system
NAME                                           READY   STATUS    RESTARTS   AGE
pod/elasticsearch-logging-0                    1/1     Running   0          11m
pod/elasticsearch-logging-1                    1/1     Running   0          7m52s
pod/elasticsearch-logging-2                    1/1     Running   0          6m34s
pod/fluentd-es-v3.1.0-fhk5r                    1/1     Running   0          21s
pod/fluentd-es-v3.1.0-kfkhb                    1/1     Running   0          21s
pod/fluentd-es-v3.1.0-lt6xd                    1/1     Running   0          21s
pod/fluentd-es-v3.1.0-rztcj                    1/1     Running   0          21s


NAME                            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                        AGE
service/elasticsearch-logging   ClusterIP   None          <none>        9200/TCP,9300/TCP              47m


NAME                               DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/fluentd-es-v3.1.0   4         4         4       4            4           <none>                   21s


NAME                                     READY   AGE
statefulset.apps/elasticsearch-logging   3/3     47m



# 每个Node一个DaemonSet
[root@k8s-master1 fluentd-elasticsearch]# kubectl get pod -n kube-system -o wide
NAME                                       READY   STATUS    RESTARTS   AGE     IP               NODE          NOMINATED NODE   READINESS GATES
elasticsearch-logging-0                    1/1     Running   0          14m     10.244.122.101   k8s-node4     <none>           <none>
elasticsearch-logging-1                    1/1     Running   0          10m     10.244.122.103   k8s-node4     <none>           <none>
elasticsearch-logging-2                    1/1     Running   0          9m34s   10.244.107.225   k8s-node3     <none>           <none>
fluentd-es-v3.1.0-fhk5r                    1/1     Running   0          3m21s   10.244.36.111    k8s-node1     <none>           <none>
fluentd-es-v3.1.0-kfkhb                    1/1     Running   0          3m21s   10.244.122.107   k8s-node4     <none>           <none>
fluentd-es-v3.1.0-lt6xd                    1/1     Running   0          3m21s   10.244.169.163   k8s-node2     <none>           <none>
fluentd-es-v3.1.0-rztcj                    1/1     Running   0          3m21s   10.244.107.226   k8s-node3     <none>           <none>
```



#### 2.4.安装Kibana

```shell
# 修改配置安装

kubectl apply -f kibana-deployment.yaml 

kubectl apply -f kibana-service.yaml
```

```yaml
# kibana-deployment.yaml 
# 修改 replicas: 3 
# NodePort映射端口需要注释 
#- name: SERVER_BASEPATH
#  value: /api/v1/namespaces/kube-system/services/kibana-logging/proxy


[root@k8s-master efk]# vim kibana-deployment.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: kibana-logging
  namespace: kube-system
  labels:
    k8s-app: kibana-logging
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  replicas: 3
  selector:
    matchLabels:
      k8s-app: kibana-logging
  template:
    metadata:
      labels:
        k8s-app: kibana-logging
    spec:
      securityContext:
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: kibana-logging
          image: docker.elastic.co/kibana/kibana-oss:7.4.2
          resources:
            # need more cpu upon initialization, therefore burstable class
            limits:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kibana-logging
  namespace: kube-system
  labels:
    k8s-app: kibana-logging
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  replicas: 3
  selector:
    matchLabels:
      k8s-app: kibana-logging
  template:
    metadata:
      labels:
        k8s-app: kibana-logging
    spec:
      securityContext:
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: kibana-logging
          image: docker.elastic.co/kibana/kibana-oss:7.4.2
          resources:
            # need more cpu upon initialization, therefore burstable class
            limits:
              cpu: 1000m
            requests:
              cpu: 100m
          env:
            - name: ELASTICSEARCH_HOSTS
              value: http://elasticsearch-logging:9200
            - name: SERVER_NAME
              value: kibana-logging
            #- name: SERVER_BASEPATH
            #  value: /api/v1/namespaces/kube-system/services/kibana-logging/proxy
            - name: SERVER_REWRITEBASEPATH
              value: "false"
          ports:
            - containerPort: 5601
              name: ui
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /api/status
              port: ui
            initialDelaySeconds: 5
            timeoutSeconds: 10
          readinessProbe:
            httpGet:
              path: /api/status
              port: ui
            initialDelaySeconds: 5
            timeoutSeconds: 10
```

```yaml
# kibana-service.yaml
# 修改 type: NodePort
# 修改nodePort: 30601 


[root@k8s-master efk]# vim kibana-service.yaml

apiVersion: v1
kind: Service
metadata:
  name: kibana-logging
  namespace: kube-system
  labels:
    k8s-app: kibana-logging
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/name: "Kibana"
spec:
  type: NodePort
  ports:
  - port: 5601
    protocol: TCP
    targetPort: ui
    nodePort: 30601
  selector:
    k8s-app: kibana-logging
```



#### 2.5.测试

```shell
# 查看


[root@k8s-master1 fluentd-elasticsearch]# kubectl get all -n kube-system
NAME                                           READY   STATUS             RESTARTS   AGE
pod/elasticsearch-logging-0                    1/1     Running            0          21m
pod/elasticsearch-logging-1                    1/1     Running            0          18m
pod/elasticsearch-logging-2                    1/1     Running            0          17m

pod/fluentd-es-v3.1.0-fhk5r                    1/1     Running            0          10m
pod/fluentd-es-v3.1.0-kfkhb                    1/1     Running            0          10m
pod/fluentd-es-v3.1.0-lt6xd                    1/1     Running            0          10m
pod/fluentd-es-v3.1.0-rztcj                    1/1     Running            0          10m

pod/kibana-logging-7b5d6867cd-2dj97            0/1     ImagePullBackOff   0          4m6s
pod/kibana-logging-7b5d6867cd-ghsxf            0/1     ImagePullBackOff   0          4m6s
pod/kibana-logging-7b5d6867cd-hdlht            0/1     ImagePullBackOff   0          4m6s


NAME                            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                        AGE
service/elasticsearch-logging   ClusterIP   None          <none>        9200/TCP,9300/TCP              57m
service/kibana-logging          NodePort    10.1.75.241   <none>        5601:30601/TCP                 3m59s


NAME                               DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/fluentd-es-v3.1.0   4         4         4       4            4           <none>                   10m


NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/kibana-logging            0/3     3            0           4m7s


NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/kibana-logging-7b5d6867cd            3         3         0       4m7s


NAME                                     READY   AGE
statefulset.apps/elasticsearch-logging   3/3     58m



[root@k8s-master1 fluentd-elasticsearch]# kubectl get svc -n kube-system
NAME                    TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                        AGE
elasticsearch-logging   ClusterIP   None          <none>        9200/TCP,9300/TCP              64m
kibana-logging          NodePort    10.1.75.241   <none>        5601:30601/TCP                 11m
```

```shell
# Kibana地址


http://172.51.216.81:30601/



# k8s内部访问地址
# K8s 中的容器使用访问
svcname.namespace.svc.cluster.local:port
elasticsearch-logging.kube-system.svc.cluster.local:9200
elasticsearch-logging.kube-system.svc.cluster.local:9300
```

![](/images/kubernetes/pro/monitoring/efk-5.png)

![](/images/kubernetes/pro/monitoring/efk-6.png)



