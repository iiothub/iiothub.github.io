* TOC
{:toc}



## 一、概述



API Server作为Kubernetes网关，是访问和管理资源对象的唯一入口，其各种集群组件访问资源都需要经过网关才能进行正常访问和管理。每一次的访问请求都需要进行合法性的检验，其中包括身份验证、操作权限验证以及操作规范验证等，需要通过一系列验证通过之后才能访问或者存储数据到etcd当中。如下图：

![](/images/kubernetes/advance/RBAC-1.png)

![](/images/kubernetes/advance/RBAC-2.png)



### 1.1.访问控制概述

Kubernetes作为一个分布式集群的管理工具，保证集群的安全性是其一个重要的任务。所谓的安全性其实就是保证对Kubernetes的各种 **客户端** 进行 **认证和鉴权** 操作。



**客户端**

```shell
在Kubernetes集群中，客户端通常有两类：
User Account：一般是独立于kubernetes之外的其他服务管理的用户账号。
Service Account：kubernetes管理的账号，用于为Pod中的服务进程在访问Kubernetes时提供身份标识。
```

![](/images/kubernetes/advance/RBAC-3.png)



**认证、授权与准入控制**

```shell
ApiServer是访问及管理资源对象的唯一入口。任何一个请求访问ApiServer，都要经过下面三个流程：

Authentication（认证）：身份鉴别，只有正确的账号才能够通过认证

Authorization（授权）： 判断用户是否有权限对访问的资源执行特定的动作

Admission Control（准入控制）：用于补充授权机制以实现更加精细的访问控制功能。
```

![](/images/kubernetes/advance/RBAC-4.png)




### 1.2.认证管理

Kubernetes集群安全的最关键点在于如何识别并认证客户端身份，它提供了3种客户端身份认证方式：

**HTTP Base认证：通过用户名+密码的方式认证**

```shell
这种认证方式是把“用户名:密码”用BASE64算法进行编码后的字符串放在HTTP请求中的Header
Authorization域里发送给服务端。服务端收到后进行解码，获取用户名及密码，然后进行用户身份认证的过程。 
```



**HTTP Token认证：通过一个Token来识别合法用户**

```shell
这种认证方式是用一个很长的难以被模仿的字符串--Token来表明客户身份的一种方式。
每个Token对应一个用户名，当客户端发起API调用请求时，需要在HTTP Header里放入Token，
API Server接到Token后会跟服务器中保存的token进行比对，然后进行用户身份认证的过程。
```



**HTTPS证书认证：基于CA根证书签名的双向数字证书认证方式**

```shell
这种认证方式是安全性最高的一种方式，但是同时也是操作起来最麻烦的一种方式。
```

![](/images/kubernetes/advance/RBAC-5.png)



**HTTPS认证大体分为3个过程：**

1.证书申请和下发

 HTTPS通信双方的服务器向CA机构申请证书，CA机构下发根证书、服务端证书及私钥给申请者

2.客户端和服务端的双向认证

- 客户端向服务器端发起请求，服务端下发自己的证书给客户端，
  客户端接收到证书后，通过私钥解密证书，在证书中获得服务端的公钥，
  客户端利用服务器端的公钥认证证书中的信息，如果一致，则认可这个服务器

- 客户端发送自己的证书给服务器端，服务端接收到证书后，通过私钥解密证书，
  在证书中获得客户端的公钥，并用该公钥认证证书信息，确认客户端是否合法

3.服务器端和客户端进行通信

服务器端和客户端协商好加密方案后，客户端会产生一个随机的秘钥并加密，然后发送到服务器端。

服务器端接收这个秘钥后，双方接下来通信的所有内容都通过该随机秘钥加密



**注意: Kubernetes允许同时配置多种认证方式，只要其中任意一个方式认证通过即可**



### 1.3.授权管理

授权发生在认证成功之后，通过认证就可以知道请求用户是谁，然后Kubernetes会根据事先定义的授权策略来决定用户是否有权限访问，这个过程就称为授权。

每个发送到ApiServer的请求都带上了用户和资源的信息：比如发送请求的用户、请求的路径、请求的动作等，授权就是根据这些信息和授权策略进行比较，如果符合策略，则认为授权通过，否则会返回错误。



**API Server目前支持以下几种授权策略：**

- AlwaysDeny：表示拒绝所有请求，一般用于测试

- AlwaysAllow：允许接收所有请求，相当于集群不需要授权流程（Kubernetes默认的策略）

- ABAC：基于属性的访问控制，表示使用用户配置的授权规则对用户请求进行匹配和控制

- Webhook：通过调用外部REST服务对用户进行授权

- Node：是一种专用模式，用于对kubelet发出的请求进行访问控制

- RBAC：基于角色的访问控制（kubeadm安装方式下的默认选项）
  

**RBAC(Role-Based Access Control) 基于角色的访问控制，主要是在描述一件事情：给哪些对象授予了哪些权限**

**其中涉及到了下面几个概念：**

- 对象：User、Groups、ServiceAccount

- 角色：代表着一组定义在资源上的可操作动作(权限)的集合

- 绑定：将定义好的角色跟用户绑定在一起


![](/images/kubernetes/advance/RBAC-6.png)



**RBAC引入了4个顶级资源对象：**

- **Role、ClusterRole**：角色，用于指定一组权限
- **RoleBinding、ClusterRoleBinding**：角色绑定，用于将角色（权限）赋予给对象



### 1.4.角色

**Role(角色)、ClusterRole(集群角色)**

**一个角色就是一组权限的集合，这里的权限都是许可形式的（白名单）。**



**Role和ClusterRole的区别：**

- Role只能对命名空间内的资源进行授权，需要指定nameapce
- ClusterRole可以对集群范围内资源、跨namespaces的范围资源、非资源类型进行授权



**Role**

```yaml
# Role只能对命名空间内的资源进行授权，需要指定nameapce
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: dev
  name: authorization-role
rules:
  - apiGroups: [ "" ]   # 支持的API组列表,"" 空字符串，表示核心API群
    resources: [ "pods" ] # 支持的资源对象列表
    verbs: [ "get", "watch", "list" ] # 允许的对资源对象的操作方法列表
```

**ClusterRole**

```yaml
# ClusterRole可以对集群范围内资源、跨namespaces的范围资源、非资源类型进行授权
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: authorization-clusterrole
rules:
  - apiGroups: [ "" ]   # 支持的API组列表,"" 空字符串，表示核心API群
    resources: [ "pods" ] # 支持的资源对象列表
    verbs: [ "get", "watch", "list" ] # 允许的对资源对象的操作方法列表
```

**需要详细说明的是，rules中的参数：**

```shell
apiGroups: 支持的API组列表
"","apps", "autoscaling", "batch"
 
 
resources: 支持的资源对象列表
"services", "endpoints", "pods","secrets","configmaps","crontabs","deployments","jobs",
"nodes","rolebindings","clusterroles","daemonsets","replicasets","statefulsets",
"horizontalpodautoscalers","replicationcontrollers","cronjobs"


verbs: 对资源对象的操作方法列表
"get", "list", "watch", "create", "update", "patch", "delete", "exec"
```



### 1.5.角色绑定

**RoleBinding(角色绑定)、ClusterRoleBinding(集群角色绑定)**

**角色绑定用来把一个角色绑定到一个目标对象上，绑定目标可以是User、Group或者ServiceAccount。**



**RoleBinding和ClusterRoleBinding的区别：**

- RoleBinding可以将同一namespace中的subject(用户、用户组)绑定到某个Role(规则)下，则此subject即具有该Role定义的权限

- ClusterRoleBinding在整个集群级别和所有namespaces，将不同namespace中的subject(用户、用户组)与ClusterRole(集群范围内资源)绑定，授予权限

**RoleBinding**

```yaml
# RoleBinding可以将同一namespace中的subject(用户、用户组)绑定到某个Role下，则此subject即具有该Role定义的权限
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: authorization-role-binding
  namespace: dev
subjects:
  - kind: User
    name: nana
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: authorization-role
  apiGroup: rbac.authorization.k8s.io
```

**ClusterRoleBinding**

```yaml
# ClusterRoleBinding在整个集群级别和所有namespaces，将特定的subject与ClusterRole绑定，授予权限
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: authorization-clusterrole-binding
subjects:
  - kind: User
    name: nana
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: authorization-clusterrole
  apiGroup: rbac.authorization.k8s.io
```



**RoleBinding引用ClusterRole进行授权**

RoleBinding可以引用ClusterRole，对属于同一命名空间内ClusterRole定义的资源主体进行授权。

- 一种很常用的做法就是，集群管理员为集群范围预定义好一组角色（ClusterRole），然后在多个命名空间中重复使用这些ClusterRole

- 这样可以大幅提高授权管理工作效率，也使得各个命名空间下的基础性授权规则与使用体验保持一致
  

```yaml
# 虽然authorization-clusterrole是一个集群角色，但是因为使用了RoleBinding
# 所以nana只能读取dev命名空间中的资源
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: authorization-role-binding-ns
  namespace: dev
subjects:
  - kind: User
    name: nana
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: authorization-clusterrole
  apiGroup: rbac.authorization.k8s.io
```



**实战：创建一个只能管理dev空间下Pods资源的账号**

**1.创建账号**

```shell
# 1. 创建证书
[root@k8s-master-01 ~]# cd /etc/kubernetes/pki/
[root@k8s-master-01 pki]# (umask 077;openssl genrsa -out devman.key 2048)
Generating RSA private key, 2048 bit long modulus
.................................................+++
............+++
e is 65537 (0x10001)

# 2. 用apiserver的证书去签署
# 2-1. 签名申请，申请的用户是devman,组是devgroup
[root@k8s-master-01 pki]# openssl req -new -key devman.key -out devman.csr -subj "/CN=devman/O=devgroup"
# 2-2. 签署证书
[root@k8s-master-01 pki]# openssl x509 -req -in devman.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out devman.crt -days 3650
Signature ok
subject=/CN=devman/O=devgroup
Getting CA Private Key

# 3. 设置集群、用户、上下文信息
[root@k8s-master-01 pki]# kubectl config set-cluster kubernetes --embed-certs=true --certificate-authority=/etc/kubernetes/pki/ca.crt --server=https://192.168.15.11:6443
Cluster "kubernetes" set.

[root@k8s-master-01 pki]# kubectl config set-credentials devman --embed-certs=true --client-certificate=/etc/kubernetes/pki/devman.crt --client-key=/etc/kubernetes/pki/devman.key
User "devman" set.

[root@k8s-master-01 pki]#  kubectl config set-context devman@kubernetes --cluster=kubernetes --user=devman
Context "devman@kubernetes" created.

# 切换账户到devman
[root@k8s-master-01 pki]# kubectl config use-context devman@kubernetes
Switched to context "devman@kubernetes".

# 查看dev下pod，发现没有权限
[root@k8s-master-01 pki]# kubectl get pods -n dev
Error from server (Forbidden): pods is forbidden: User "devman" cannot list resource "pods" in API group "" in the namespace "dev"

# 切换到admin账户
[root@k8s-master-01 pki]# kubectl config use-context kubernetes-admin@kubernetes
Switched to context "kubernetes-admin@kubernetes".
```

**2.创建Role和RoleBinding，为devman用户授权**

**创建文件 dev-role.yaml**

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: dev
  name: dev-role
rules:
  - apiGroups: [ "" ]
    resources: [ "pods" ]
    verbs: [ "get","watch","list" ]

---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: authorization-role-binding
  namespace: dev
subjects:
  - kind: User
    name: devman
    apiGroup: rbac.authorization.k8s.io
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: dev-role
```

```shell
[root@k8s-master-01 pki]# kubectl apply -f dev-role.yaml
role.rbac.authorization.k8s.io/dev-role created
rolebinding.rbac.authorization.k8s.io/authorization-role-binding created
```



**3.切换账户，再次验证**

```shell
# 切换账户到devman
[root@k8s-master-01 pki]# kubectl config use-context devman@kubernetes
Switched to context "devman@kubernetes".

# 再次查看
[root@k8s-master-01 pki]# kubectl get pods -n dev
No resources found in dev namespace.

# 没有添加service类型,因此devman用户访问不到
[root@k8s-master-01 pki]# kubectl get svc -n dev
Error from server (Forbidden): services is forbidden: User "devman" cannot list resource "services" in API group "" in the namespace "dev"

# 为了不影响后面的学习,切回admin账户
[root@k8s-master-01 pki]# kubectl config use-context kubernetes-admin@kubernetes
Switched to context "kubernetes-admin@kubernetes".
```



### 1.6.准入控制

**通过了前面的认证和授权之后，还需要经过准入控制处理通过之后，apiserver才会处理这个请求。**

**准入控制是一个可配置的控制器列表，可以通过在Api-Server上通过命令行设置选择执行哪些准入控制器：**

```shell
--admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,
DefaultStorageClass,ResourceQuota,DefaultTolerationSeconds
```

**只有当所有的准入控制器都检查通过之后，apiserver才执行该请求，否则返回拒绝。**

**当前可配置的Admission Control准入控制如下：**

- AlwaysAdmit：允许所有请求

- AlwaysDeny：禁止所有请求，一般用于测试

- AlwaysPullimages：在启动容器之前总去下载镜像

- DenyExecOnPrivileged：它会拦截所有想在Privileged Container上执行命令的请求

- ImagePolicyWebhook：这个插件将允许后端的一个Webhook程序来完成admission controller的功能。

- Service Account：实现ServiceAccount实现了自动化

- SecurityContextDeny：这个插件将使用SecurityContext的Pod中的定义全部失效

- ResourceQuota：用于资源配额管理目的，观察所有请求，确保在namespace上的配额不会超标

- LimitRanger：用于资源限制管理，作用于namespace上，确保对Pod进行资源限制

- InitialResources：为未设置资源请求与限制的Pod，根据其镜像的历史资源的使用情况进行设置

- NamespaceLifecycle：如果尝试在一个不存在的namespace中创建资源对象，则该创建请求将被拒绝。当删除一个namespace时，系统将会删除该namespace中所有对象。

- DefaultStorageClass：为了实现共享存储的动态供应，为未指定StorageClass或PV的PVC尝试匹配默认的StorageClass，尽可能减少用户在申请PVC时所需了解的后端存储细节

- DefaultTolerationSeconds：这个插件为那些没有设置forgiveness tolerations并具有 notready:NoExecute 和 unreachable:NoExecute 两种taints的Pod设置默认的“容忍”时间，为5min

- PodSecurityPolicy：这个插件用于在创建或修改Pod时决定是否根据Pod的security context和可用的PodSecurityPolicy对Pod的安全策略进行控制
  



### 1.7.Service Account

Service account是为了方便Pod里面的进程调用Kubernetes API或其他外部服务而设计的。它与User account不同

- User account是为人设计的，而service account则是为Pod中的进程调用Kubernetes API而设计
- User account是跨namespace的，而service account则是仅局限它所在的namespace
- 每个namespace都会自动创建一个default service account
- Token controller检测service account的创建，并为它们创建[secret](https://www.kubernetes.org.cn/secret)
- 开启ServiceAccount Admission Controller后
  - 每个Pod在创建后都会自动设置spec.serviceAccount为default（除非指定了其他ServiceAccout）
  - 验证Pod引用的service account已经存在，否则拒绝创建
  - 如果Pod没有指定ImagePullSecrets，则把service account的ImagePullSecrets加到Pod中
  - 每个container启动后都会挂载该service account的token和ca.crt到/var/run/secrets/kubernetes.io/serviceaccount/



当创建 pod 的时候，如果没有指定一个 service account，系统会自动在与该pod 相同的 namespace 下为其指派一个default service account。而pod和apiserver之间进行通信的账号，称为serviceAccountName。如下：



**验证：**

```shell
[root@k8s-master01 ~]# kubectl create namespace qiangungun  #创建一个名称空间
namespace "qiangungun" created
[root@k8s-master01 ~]# kubectl get sa -n qiangungun  #名称空间创建完成后会自动创建一个sa
NAME      SECRETS   AGE
default   1         11s
[root@k8s-master01 ~]# kubectl get secret -n qiangungun  #同时也会自动创建一个secret
NAME                  TYPE                                  DATA      AGE
default-token-5jtz2   kubernetes.io/service-account-token   3         19s
```

**在创建的名称空间中新建一个pod**

```yaml
[root@k8s-master01 pod-example]# cat pod_demo.yaml 
kind: Pod
apiVersion: v1
metadata:
  name: task-pv-pod
  namespace: qiangungun
spec:
  containers:
  - name: nginx
    image: ikubernetes/myapp:v1
    ports:
     - containerPort: 80
       name: www
```

**查看pod信息**

```shell
[root@k8s-master01 pod-example]# kubectl apply -f  pod_demo.yaml 
pod "task-pv-pod" created
[root@k8s-master01 pod-example]# kubectl get pod -n qiangungun 
NAME          READY     STATUS    RESTARTS   AGE
task-pv-pod   1/1       Running   0          13s
[root@k8s-master01 pod-example]# kubectl get  pod task-pv-pod -o yaml   -n qiangungun 
......
volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-5jtz2
......
volumes:  #挂载sa的secret
  - name: default-token-5jtz2
    secret:
      defaultMode: 420
      secretName: default-token-5jtz2 
......
```

**名称空间新建的pod如果不指定sa，会自动挂载当前名称空间中默认的sa(default)**



#### 1.7.1.创建serviceaccount

**serviceaccount以下简称sa**

```shell
[root@k8s-master01 ~]#  kubectl create  serviceaccount admin   #创建一个sa 名称为admin
serviceaccount "admin" created

[root@k8s-master01 ~]# kubectl get sa 
NAME      SECRETS   AGE
admin     1         6s
default   1         28d

[root@k8s-master01 ~]# kubectl describe sa admin   #查看名称为admin的sa的信息，系统会自动创建一个token信息
Name:                admin
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   admin-token-rxtrc
Tokens:              admin-token-rxtrc
Events:              <none>
[root@k8s-master01 ~]# kubectl get secret  #会自动创建一个secret(admin-token-rxtrc),用于当前sa连接至当前API server时使用的认证信息
NAME                    TYPE                                  DATA      AGE
admin-token-rxtrc       kubernetes.io/service-account-token   3         1m
default-token-tcwjz     kubernetes.io/service-account-token   3         28d
myapp-ingress-secret    kubernetes.io/tls                     2         6h
mysql-passwd            Opaque                                1         17d
tomcat-ingress-secret   kubernetes.io/tls                     2         7h
```



**创建一个pod应用刚刚创建的sa**

```shell
[root@k8s-master01 service_account]# cat deploy-demon.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: sa-demo
  labels:
    app: myapp
    release: canary
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v2
    ports:
    - name: httpd
      containerPort: 80
  serviceAccountName: admin  #此处指令为指定sa的名称
  
[root@k8s-master01 service_account]# kubectl apply -f deploy-demon.yaml 
pod "sa-demo" created

[root@k8s-master01 service_account]# kubectl describe pod sa-demo 
......
Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from admin-token-rxtrc (ro) #pod会自动挂载自己sa的证书
......
  Volumes:    admin-token-rxtrc:      Type:        Secret (a volume populated by a Secret)      SecretName:  admin-token-rxtrc
......
```

集群交互的时候少不了的是身份认证，使用 kubeconfig（即证书） 和 token 两种认证方式是最简单也最通用的认证方式，下面我使用kubeconfing来进行认证

使用kubeconfig文件来组织关于集群，用户，名称空间和身份验证机制的信息。使用 kubectl命令行工具对kubeconfig文件来查找选择群集并与群集的API服务器进行通信所需的信息。

默认情况下 kubectl使用的配置文件名称是在$HOME/.kube目录下 config文件，可以通过设置环境变量KUBECONFIG或者--kubeconfig指定其他的配置文件



**查看系统的kubeconfig**

```shell
[root@k8s-master01 ~]# kubectl config view 
apiVersion: v1
clusters:   #集群列表 
- cluster:
    certificate-authority-data: REDACTED  #认证集群的方式
    server: https://172.16.150.212:6443    #访问服务的APIserver的路径
  name: kubernetes #集群的名称
contexts: #上下文列表
- context:
    cluster: kubernetes  #访问kubernetes这个集群
    user: kubernetes-admin  #使用 kubernetes-admin账号
  name: kubernetes-admin@kubernetes #给定一个名称
current-context: kubernetes-admin@kubernetes #当前上下文，表示使用哪个账号访问哪个集群
kind: Config
preferences: {}
users:  #用户列表
- name: kubernetes-admin #用户名称
  user:
    client-certificate-data: REDACTED #客户端证书，用于与apiserver进行认证
    client-key-data: REDACTED #客户端私钥
```

```shell
[root@k8s-master01 ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP             29d
my-nginx     NodePort    10.104.13.148    <none>        80:32008/TCP        18h
myapp        ClusterIP   10.102.229.150   <none>        80/TCP              19h
tomcat       ClusterIP   10.106.222.72    <none>        8080/TCP,8009/TCP   19h

[root@k8s-master01 ~]# kubectl describe svc kubernetes 
Name:              kubernetes
Namespace:         default
Labels:            component=apiserver
                   provider=kubernetes
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP:                10.96.0.1
Port:              https  443/TCP
TargetPort:        6443/TCP
Endpoints:         172.16.150.212:6443  #可以看到此处svc后端的Endpoint是当前节点的IP地址，通过svc的IP地址进行映射，以确保cluster中的pod可以通过该sa与集群内api进行通讯，仅仅是身份认证
Session Affinity:  ClientIP
Events:            <none>
```



**查看kubeconfig命令行配置帮助**

```shell
[root@k8s-master01 ~]# kubectl config --help
Modify kubeconfig files using subcommands like "kubectl config set current-context my-context" 

The loading order follows these rules: 

  1. If the --kubeconfig flag is set, then only that file is loaded.  The flag may only be set once
and no merging takes place.  
  2. If $KUBECONFIG environment variable is set, then it is used a list of paths (normal path
delimitting rules for your system).  These paths are merged.  When a value is modified, it is
modified in the file that defines the stanza.  When a value is created, it is created in the first
file that exists.  If no files in the chain exist, then it creates the last file in the list.  
  3. Otherwise, ${HOME}/.kube/config is used and no merging takes place.

Available Commands:
  current-context 显示 current_context
  delete-cluster  删除 kubeconfig 文件中指定的集群
  delete-context  删除 kubeconfig 文件中指定的 context
  get-clusters    显示 kubeconfig 文件中定义的集群
  get-contexts    描述一个或多个 contexts
  rename-context  Renames a context from the kubeconfig file.
  set             设置 kubeconfig 文件中的一个单个值
  set-cluster     设置 kubeconfig 文件中的一个集群条目
  set-context     设置 kubeconfig 文件中的一个 context 条目
  set-credentials 设置 kubeconfig 文件中的一个用户条目
  unset           取消设置 kubeconfig 文件中的一个单个值
  use-context     设置 kubeconfig 文件中的当前上下文
  view            显示合并的 kubeconfig 配置或一个指定的 kubeconfig 文件

Usage:
  kubectl config SUBCOMMAND [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
```



#### 1.7.2.创建一个cluster用户及context

**使用当前系统的ca证书认证一个私有证书**

```shell
[root@k8s-master01 ~]# cd /etc/kubernetes/pki/
[root@k8s-master01 pki]# (umask 077;openssl genrsa -out qiangungun.key 2048)
Generating RSA private key, 2048 bit long modulus
.........................+++
..........................................................+++
e is 65537 (0x10001)

[root@k8s-master01 pki]# openssl req -new -key qiangungun.key -out qiangungun.csr -subj "/CN=qiangungun"  #qiangungun是后面我们创建的用户名称，需要保持一致
[root@k8s-master01 pki]# openssl x509 -req -in qiangungun.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out qiangungun.crt -days 3650
Signature ok
subject=/CN=qiangungun
Getting CA Private Key
```

**查看证书内容**

```shell
[root@k8s-master01 pki]# openssl x509 -in qiangungun.crt -text -noout
Certificate:
    Data:
        Version: 1 (0x0)
        Serial Number:
            b6:06:cb:30:86:e3:fe:84
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=kubernetes  #由谁签署的
        Validity  #证书的有效时间
            Not Before: Nov 27 15:09:41 2018 GMT
            Not After : Nov 24 15:09:41 2028 GMT
        Subject: CN=qiangungun  #证书使用的用户
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048
                 ......
```

**创建一个当前集群用户**

```yaml
[root@k8s-master01 pki]#  kubectl config set-credentials qiangungun --client-certificate=./qiangungun.crt --client-key=./qiangungun.key --embed-certs=true  #--embed-certs表示是否隐藏证书路径及名称,默认不隐藏
User "qiangungun" set.

[root@k8s-master01 pki]# kubectl config view 
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: REDACTED
    server: https://172.16.150.212:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
- name: qiangungun  #我们新建的用户
  user: 
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

**为qiangungun用户创建一个context**

```shell
[root@k8s-master01 pki]# kubectl config set-context  qiangungun@kubernetes --cluster=kubernetes --user=qiangungun 
Context "qiangungun@kubernetes" created.
[root@k8s-master01 pki]# kubectl config view 
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: REDACTED
    server: https://172.16.150.212:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
- context:  #新创建的context
    cluster: kubernetes
    user: qiangungun
  name: qiangungun@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED- name: qiangungun
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

**切换serviceaccount**

```shell
[root@k8s-master01 pki]# kubectl config use-context qiangungun@kubernetes 
Switched to context "qiangungun@kubernetes".
[root@k8s-master01 pki]# kubectl get pod
Error from server (Forbidden): pods is forbidden: User "qiangungun" cannot list pods in the namespace "default"
```

**自定义一个cluster**

```shell
[root@k8s-master01 pki]# kubectl config set-cluster  mycluster --kubeconfig=/tmp/test.conf --server="https://172.16.150.212:6443" --certificate-authority=/etc/kubernetes/pki/ca.crt --embed-certs=true
Cluster "mycluster" set.
[root@k8s-master01 pki]# kubectl config view --kubeconfig=/tmp/test.conf 
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: REDACTED
    server: https://172.16.150.212:6443
  name: mycluster
contexts: []
current-context: ""
kind: Config
preferences: {}
users: []
```





## 二、基础



### 1.创建用户账号

**使用openssl方法创建普通用户**



1.创建证书：使用当前系统的ca证书认证一个私有证书

```shell
[root@k8s-master ~]# cd /etc/kubernetes/pki

[root@k8s-master pki]# ll
total 56
-rw-r--r-- 1 root root 1269 Aug 17 12:38 apiserver.crt
-rw-r--r-- 1 root root 1135 Aug 17 12:38 apiserver-etcd-client.crt
-rw------- 1 root root 1675 Aug 17 12:38 apiserver-etcd-client.key
-rw------- 1 root root 1675 Aug 17 12:38 apiserver.key
-rw-r--r-- 1 root root 1143 Aug 17 12:38 apiserver-kubelet-client.crt
-rw------- 1 root root 1679 Aug 17 12:38 apiserver-kubelet-client.key
-rw-r--r-- 1 root root 1066 Aug 17 12:38 ca.crt
-rw------- 1 root root 1679 Aug 17 12:38 ca.key
drwxr-xr-x 2 root root  162 Aug 17 12:38 etcd
-rw-r--r-- 1 root root 1078 Aug 17 12:38 front-proxy-ca.crt
-rw------- 1 root root 1679 Aug 17 12:38 front-proxy-ca.key
-rw-r--r-- 1 root root 1103 Aug 17 12:38 front-proxy-client.crt
-rw------- 1 root root 1675 Aug 17 12:38 front-proxy-client.key
-rw------- 1 root root 1679 Aug 17 12:38 sa.key
-rw------- 1 root root  451 Aug 17 12:38 sa.pub


# 创建user私钥
[root@k8s-master pki]# (umask 077;openssl genrsa -out testuser.key 2048)
Generating RSA private key, 2048 bit long modulus
............+++
..............+++
e is 65537 (0x10001)

[root@k8s-master pki]# ll
total 60
...
-rw------- 1 root root 1675 Oct 19 11:35 testuser.key


# 创建证书签署请求
# O=组织信息，CN=用户名
[root@k8s-master pki]# openssl req -new -key testuser.key -out testuser.csr -subj "/O=k8s/CN=testuser"

[root@k8s-master pki]# ll
total 64
...
-rw-r--r-- 1 root root  907 Oct 19 11:39 testuser.csr
-rw------- 1 root root 1675 Oct 19 11:35 testuser.key


# 签署证书
[root@k8s-master pki]# openssl  x509 -req -in testuser.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out testuser.crt -days 365
Signature ok
subject=/O=k8s/CN=testuser
Getting CA Private Key

[root@k8s-master pki]# ll
total 72
...
-rw-r--r-- 1 root root  997 Oct 19 11:42 testuser.crt
-rw-r--r-- 1 root root  907 Oct 19 11:39 testuser.csr
-rw------- 1 root root 1675 Oct 19 11:35 testuser.key
```



2.创建配置文件

创建配置文件主要有以下几个步骤：

```shell
kubectl config set-cluster --kubeconfig=/PATH/TO/SOMEFILE      #集群配置
kubectl config set-credentials NAME --kubeconfig=/PATH/TO/SOMEFILE #用户配置
kubectl config set-context    #context配置
kubectl config use-context    #切换context


# --embed-certs=true的作用是不在配置文件中显示证书信息。
# --kubeconfig=/root/cbmljs.conf用于创建新的配置文件，如果不加此选项,则内容会添加到家目录下.kube/config文件中，可以使用use-context来切换不同的用户管理k8s集群。
# context简单的理解就是用什么用户来管理哪个集群，即用户和集群的结合。
```



创建集群配置

```shell

kubectl config set-cluster k8s --server=https://172.51.216.81:6443 \
--certificate-authority=ca.crt \
--embed-certs=true  \
--kubeconfig=/root/testuser.conf

kubectl config view --kubeconfig=/root/testuser.conf


[root@k8s-master pki]# kubectl config set-cluster k8s --server=https://172.51.216.81:6443 \
> --certificate-authority=ca.crt \
> --embed-certs=true  \
> --kubeconfig=/root/testuser.conf
Cluster "k8s" set.

[root@k8s-master pki]# kubectl config set-cluster k8s --server=https://172.51.216.81:6443 \
> --certificate-authority=ca.crt \
> --embed-certs=true  \
> --kubeconfig=/root/testuser.conf
Cluster "k8s" set.
[root@k8s-master pki]# kubectl config view --kubeconfig=/root/testuser.conf
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://172.51.216.81:6443
  name: k8s
contexts: null
current-context: ""
kind: Config
preferences: {}
users: null
```



创建用户配置

```shell
kubectl config set-credentials testuser \
--client-certificate=testuser.crt \
--client-key=testuser.key \
--embed-certs=true \
--kubeconfig=/root/testuser.conf


[root@k8s-master pki]# kubectl config set-credentials testuser \
> --client-certificate=testuser.crt \
> --client-key=testuser.key \
> --embed-certs=true \
> --kubeconfig=/root/testuser.conf
User "testuser" set.

[root@k8s-master pki]# kubectl config view --kubeconfig=/root/testuser.conf
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://172.51.216.81:6443
  name: k8s
contexts: null
current-context: ""
kind: Config
preferences: {}
users:
- name: testuser
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```



创建context配置

```shell
kubectl config set-context testuser@k8s \
--cluster=k8s \
--user=testuser \
--kubeconfig=/root/testuser.conf


[root@k8s-master pki]# kubectl config set-context testuser@k8s \
> --cluster=k8s \
> --user=testuser \
> --kubeconfig=/root/testuser.conf
Context "testuser@k8s" created.

[root@k8s-master pki]# kubectl config view --kubeconfig=/root/testuser.conf
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://172.51.216.81:6443
  name: k8s
contexts:
- context:
    cluster: k8s
    user: testuser
  name: testuser@k8s
current-context: ""
kind: Config
preferences: {}
users:
- name: testuser
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```



切换context

```shell
kubectl config use-context testuser@k8s --kubeconfig=/root/testuser.conf
kubectl config view --kubeconfig=/root/testuser.conf


[root@k8s-master pki]# kubectl config use-context testuser@k8s --kubeconfig=/root/testuser.conf
Switched to context "testuser@k8s".

[root@k8s-master pki]# kubectl config view --kubeconfig=/root/testuser.conf
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://172.51.216.81:6443
  name: k8s
contexts:
- context:
    cluster: k8s
    user: testuser
  name: testuser@k8s
current-context: testuser@k8s
kind: Config
preferences: {}
users:
- name: testuser
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

创建系统用户

```shell
[root@k8s-master pki]# useradd testuser
[root@k8s-master pki]# mkdir -p /home/testuser/.kube
[root@k8s-master pki]# cp /root/testuser.conf /home/testuser/.kube/config
[root@k8s-master pki]# chown testuser.testuser -R /home/testuser/
[root@k8s-master pki]# su - testuser
Last failed login: Wed Oct 13 09:28:09 CST 2021 from localhost on ssh:notty
There were 12 failed login attempts since the last successful login.
[testuser@k8s-master ~]$ 


# k8s验证文件
[testuser@k8s-master ~]$ kubectl get pod
Error from server (Forbidden): pods is forbidden: User "testuser" cannot list resource "pods" in API group "" in the namespace "default"


# 默认新用户是没有任何权限的。
```



3.创建Role

此role只有pod的get、list、watch权限

```yaml
[testuser@k8s-master rbac]$ cat pods-reader.yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pods-reader
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
  - list
  - watch
```



4.创建Rolebinding

用户testuser和role pods-reader的绑定

```yaml
[testuser@k8s-master rbac]$ cat testuser-pods-reader.yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: testuser-pods-reader
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pods-reader
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: testuser
```



5.验证结果

如果没有指定命名空间的话，默认就是default命名空间。

```shell
[root@k8s-master rbac]# ll
total 8
-rw-r--r-- 1 root root 167 Oct 19 15:23 pods-reader.yaml
-rw-r--r-- 1 root root 256 Oct 19 15:12 testuser-pods-reader.yaml

[root@k8s-master rbac]# kubectl apply -f pods-reader.yaml 
role.rbac.authorization.k8s.io/pods-reader created

[root@k8s-master rbac]# kubectl apply -f testuser-pods-reader.yaml 
rolebinding.rbac.authorization.k8s.io/testuser-pods-reader created


# testuser
[testuser@k8s-master rbac]$ kubectl get pod
NAME                                      READY   STATUS    RESTARTS   AGE
drc-example-distributedrediscluster-0-0   1/1     Running   0          4d5h
drc-example-distributedrediscluster-0-1   1/1     Running   0          4d5h
drc-example-distributedrediscluster-1-0   1/1     Running   0          4d5h
drc-example-distributedrediscluster-1-1   1/1     Running   0          4d5h
drc-example-distributedrediscluster-2-0   1/1     Running   0          4d5h
drc-example-distributedrediscluster-2-1   1/1     Running   0          4d5h
hello-world-server-0                      0/1     Pending   0          3d23h
redis-cluster-operator-6669898858-9b7kh   1/1     Running   0          4d5h


[testuser@k8s-master rbac]$ kubectl get pod -n kube-system
Error from server (Forbidden): pods is forbidden: User "testuser" cannot list resource "pods" in API group "" in the namespace "kube-system"


# root
[root@k8s-master rbac]# kubectl get pod
NAME                                      READY   STATUS    RESTARTS   AGE
drc-example-distributedrediscluster-0-0   1/1     Running   0          4d5h
drc-example-distributedrediscluster-0-1   1/1     Running   0          4d5h
drc-example-distributedrediscluster-1-0   1/1     Running   0          4d5h
drc-example-distributedrediscluster-1-1   1/1     Running   0          4d5h
drc-example-distributedrediscluster-2-0   1/1     Running   0          4d5h
drc-example-distributedrediscluster-2-1   1/1     Running   0          4d5h
hello-world-server-0                      0/1     Pending   0          3d23h
redis-cluster-operator-6669898858-9b7kh   1/1     Running   0          4d5h
```

所以我们是可以查看查看default命名空间的pod，但是其他空间的pod是无法查看的。



**6.创建ClusterRole**

```yaml
[root@k8s-master rbac]# cat cluster-reader.yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-reader
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
  - list
  - watch
  

[root@k8s-master rbac]# kubectl apply -f cluster-reader.yaml 
clusterrole.rbac.authorization.k8s.io/cluster-reader created
```



7.创建ClusterRoleBinding

```yaml
[root@k8s-master rbac]# cat testuser-read-all-pod.yaml 
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: billy-read-all-pods
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-reader
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: testuser


[root@k8s-master rbac]# kubectl apply -f testuser-read-all-pod.yaml 
Warning: rbac.authorization.k8s.io/v1beta1 ClusterRoleBinding is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 ClusterRoleBinding
clusterrolebinding.rbac.authorization.k8s.io/billy-read-all-pods created
```



8.验证结果

创建了ClusterRole和ClusterRoleBinding后就可以看到所有命名空间的pod了。

```shell
[testuser@k8s-master rbac]$ kubectl get pod
NAME                                      READY   STATUS    RESTARTS   AGE
drc-example-distributedrediscluster-0-0   1/1     Running   0          4d5h
drc-example-distributedrediscluster-0-1   1/1     Running   0          4d5h
drc-example-distributedrediscluster-1-0   1/1     Running   0          4d5h
drc-example-distributedrediscluster-1-1   1/1     Running   0          4d5h
drc-example-distributedrediscluster-2-0   1/1     Running   0          4d5h
drc-example-distributedrediscluster-2-1   1/1     Running   0          4d5h
hello-world-server-0                      0/1     Pending   0          3d23h
redis-cluster-operator-6669898858-9b7kh   1/1     Running   0          4d5h


[testuser@k8s-master rbac]$ kubectl get pod -n kube-system
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-5f6cfd688c-2tlm5   1/1     Running   7          63d
calico-node-2kgrn                          1/1     Running   7          63d
calico-node-4hmgw                          1/1     Running   8          63d
calico-node-svzpr                          1/1     Running   8          63d
calico-node-sxlrj                          1/1     Running   7          63d
coredns-7f89b7bc75-m84rp                   1/1     Running   7          63d
coredns-7f89b7bc75-xcn97                   1/1     Running   8          63d
etcd-k8s-master                            1/1     Running   7          63d
kube-apiserver-k8s-master                  1/1     Running   9          63d
kube-controller-manager-k8s-master         1/1     Running   7          63d
kube-proxy-8gwpb                           1/1     Running   7          63d
kube-proxy-dbntt                           1/1     Running   9          63d
kube-proxy-h6v7c                           1/1     Running   9          63d
kube-proxy-xprl7                           1/1     Running   7          63d
kube-scheduler-k8s-master                  1/1     Running   7          63d
```



### 2.serviceaccount

**Servic Account（服务账号）：**是指由`Kubernetes API`管理的账号，用于为`Pod`之中的服务进程在访问`Kubernetes API`时提供身份标识。`Service Account`通常绑定于特定的名称空间，由`API Server`创建，或者通过`API`调用手动创建。

**User Account（用户账号）：**独立于`Kubernetes`之外的其他服务管理用户账号，例如由管理员分发秘钥、`Keystone`一类的用户存储（账号库）、甚至是保函有用户名和密码列表的文件等。

- `User Account`是为人设计的，而`Service Account`则是为`Pod`中的进程调用`Kubernetes API`而设计
- `User Account`是跨`namespace`的，而`Service Account`则是仅局限它所在的`namespace`
- 每个`namespace`都会自动创建一个`default service account`

在创建`Pod`资源时，如果没有指定一个`service account`，系统会自动在与该`Pod`相同的`namespace`下为其指派一个`default service account`。而`pod`和`apiserver`之间进行通信的账号，称为`serviceAccountName`。如下：

```shell
[root@k8s-master ~]# kubectl get pods  
NAME                  READY   STATUS    RESTARTS   AGE
nginx-statefulset-0   1/1     Running   0          43h
nginx-statefulset-1   1/1     Running   0          43h
nginx-statefulset-2   1/1     Running   0          43h
nginx-statefulset-3   1/1     Running   0          43h

[root@k8s-master ~]# kubectl get pods/nginx-statefulset-0 -o yaml |grep "serviceAccountName"
  serviceAccountName: default
  
[root@k8s-master ~]# kubectl describe pods/nginx-statefulset-0
Name:           nginx-statefulset-0
Namespace:      default
......
Volumes:
  default-token-blm9l:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-blm9l
    Optional:    false


# 通过上面可以看出每个Pod无论定义与否都会有一个存储卷，这个存储卷为default-token-* token令牌，这就是Pod和serviceaccount认证信息。通过secret进行定义，由于认证信息属于敏感信息，所以需要保存在secret资源当中，并以存储卷的方式挂载到Pod当中。从而让Pod内运行的应用通过对应的secret中的信息来连接apiserver，并完成认证。每个namespace中都有一个默认的叫做default的service account资源。进行查看名称空间内的secret，也可以看到对应的default-token。让当前名称空间中所有的pod在连接apiserver时可以使用的预制认证信息，从而保证pod之间的通信。
```



**1.Service Account创建**

```shell
# 查看serviceaccount资源
[root@k8s-master ~]# kubectl get sa
NAME      SECRETS   AGE
default   1         7d19h


# 创建一个名为admin的serviceaccount资源
[root@k8s-master rbac]# kubectl create serviceaccount admin
serviceaccount/admin created


# 查看serviceaccount资源
[root@k8s-master rbac]# kubectl get sa
NAME                     SECRETS   AGE
admin                    1         50s
default                  1         63d


# 查看serviceaccount资源admin的详细信息，可以看出已经自动生成了一个Tokens：admin-token-tvnbc
[root@k8s-master rbac]# kubectl describe sa admin
Name:                admin
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   admin-token-tvnbc
Tokens:              admin-token-tvnbc
Events:              <none>


#查看secret，可以查看也生成了一个admin-token-tvnbc的secret资源
[root@k8s-master rbac]# kubectl get secret
NAME                                 TYPE                                  DATA   AGE
admin-token-tvnbc                    kubernetes.io/service-account-token   3      2m58s
default-token-b442w                  kubernetes.io/service-account-token   3      63d
...
```



**2.Pod中引用service account**

每个`Pod`对象均可附加其所属名称空间中的一个`Service Account`资源，且只能附加一个。不过，一个`Service Account`资源可由所属名称空间中的多个`Pod`对象共享使用。创建`Pod`时，通过“`spec.serviceAccountName`”进行定义。示例如下：

```shell
#编辑资源清单文件
[root@k8s-master rbac]# vim pod-sa-demo.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-sa-demo
  namespace: default
  labels:
    app: myapp
    tier: frontend
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    ports:
    - name: http
      containerPort: 80
  serviceAccountName: admin    #指定serviceAccount资源名称
  
  
[root@k8s-master rbac]# kubectl apply -f pod-sa-demo.yaml
pod/pod-sa-demo created
  
  
[root@k8s-master rbac]# kubectl get pods -l app=myapp
NAME          READY   STATUS    RESTARTS   AGE
pod-sa-demo   1/1     Running   0          21s


[root@k8s-master rbac]# kubectl describe pods/pod-sa-demo
Name:         pod-sa-demo
Namespace:    default
Priority:     0
Node:         k8s-node3/172.51.216.84
Start Time:   Tue, 19 Oct 2021 16:36:46 +0800
Labels:       app=myapp
              tier=frontend
Annotations:  cni.projectcalico.org/containerID: 8fa3db8c8d49a25258dc32cd2158b07a805aac29c3cf5f7c03dc647940908bdb
              cni.projectcalico.org/podIP: 10.244.107.196/32
              cni.projectcalico.org/podIPs: 10.244.107.196/32
Status:       Running
IP:           10.244.107.196
IPs:
  IP:  10.244.107.196
Containers:
  myapp:
    Container ID:   docker://4a697b75270aa0c11c50fb06839d0614f759355a8e38e16b4cfa3f8d6ef47e2e
    Image:          ikubernetes/myapp:v1
    Image ID:       docker-pullable://ikubernetes/myapp@sha256:9c3dc30b5219788b2b8a4b065f548b922a34479577befb54b03330999d30d513
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 19 Oct 2021 16:36:47 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from admin-token-tvnbc (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  admin-token-tvnbc:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  admin-token-tvnbc  #这里可以看出挂载token就是上面创建的sa所生成的那个。
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  63s   default-scheduler  Successfully assigned default/pod-sa-demo to k8s-node3
  Normal  Pulled     69s   kubelet            Container image "ikubernetes/myapp:v1" already present on machine
  Normal  Created    69s   kubelet            Created container myapp
  Normal  Started    69s   kubelet            Started container myapp
```



