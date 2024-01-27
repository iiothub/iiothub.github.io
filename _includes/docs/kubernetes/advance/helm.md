* TOC
{:toc}



## 一、概述



### 1.简介

**为什么需要Helm**

K8S上的应用对象，都是由特定的资源描述组成，包括deployment、service等。都保存各自文件中或者集中写到一个配置文件。然后kubectl apply –f 部署。

如果应用只由一个或几个这样的服务组成，上面部署方式足够了。

而对于一个复杂的应用，会有很多类似上面的资源描述文件，例如微服务架构应用，组成应用的服务可能多达十个，几十个。如果有更新或回滚应用的需求，可能要修改和维护所涉及的大量资源文件，而这种组织和管理应用的方式就显得力不从心了。

且由于缺少对发布过的应用版本管理和控制，使Kubernetes上的应用维护和更新等面临诸多的挑战，主要面临以下问题：

1. **如何将这些服务作为一个整体管理**
2. **这些资源文件如何高效复用**
3. **不支持应用级别的版本管理**



**什么是 Helm** 

Helm 为团队提供了在 Kubernetes 内部创建、安装和管理应用程序时需要协作的工具，有点类似于 Ubuntu 中的 APT 或 CentOS 中的 YUM。

有了 Helm，开发者可以：

- 查找要安装和使用的预打包软件（Chart）
- 轻松创建和托管自己的软件包
- 将软件包安装到任何 K8s 集群中
- 查询集群以查看已安装和正在运行的程序包
- 更新、删除、回滚或查看已安装软件包的历史记录



Helm 组件和相关术语

**helm**

- Helm 是一个命令行下的客户端工具。主要用于 Kubernetes 应用程序 Chart 的创建、打包、发布以及创建和管理本地和远程的 Chart 仓库。

**Chart**

- Helm 的软件包，采用 TAR 格式。类似于 APT 的 DEB 包或者 YUM 的 RPM 包，其包含了一组定义 Kubernetes 资源相关的 YAML 文件。

**Repoistory**

- Helm 的软件仓库，Repository 本质上是一个 Web 服务器，该服务器保存了一系列的 Chart 软件包以供用户下载，并且提供了一个该 Repository 的 Chart 包的清单文件以供查询。Helm 可以同时管理多个不同的 Repository。

**Release**

- 使用 helm install 命令在 Kubernetes 集群中部署的 Chart 称为 Release。可以理解为 Helm 使用 Chart 包部署的一个应用实例。



### 2.Helm架构

Helm是Kubernetes 应用的包管理工具，主要用来管理 Charts，类似Linux系统的yum。

Helm Chart 是用来封装 Kubernetes 原生应用程序的一系列 YAML 文件。可以在你部署应用的时候自定义应用程序的一些 Metadata，以便于应用程序的分发。

对于应用发布者而言，可以通过 Helm 打包应用、管理应用依赖关系、管理应用版本并发布应用到软件仓库。

对于使用者而言，使用 Helm 后不用需要编写复杂的应用部署文件，可以以简单的方式在 Kubernetes 上查找、安装、升级、回滚、卸载应用程序。
![](/images/kubernetes/advance/helm-1.png)



Helm V3 与 V2 最大的区别在于去掉了tiller

![](/images/kubernetes/advance/helm-2.png)

2019年11月13日，Helm团队发布 `Helm v3`的第一个稳定版本。

该版本主要变化如下：

- 架构变化：最明显的变化是 Tiller 的删除
- Release 名称可以在不同命名空间重用
- 支持将 Chart 推送至 Docker 镜像仓库中
- 使用JSONSchema验证chart values



### 3.部署Helm

**部署helm客户端**

Helm客户端下载地址：https://github.com/helm/helm/releases

解压移动到/usr/bin/目录即可。

```shell
wget https://get.helm.sh/helm-v3.7.0-linux-amd64.tar.gz

tar zxvf helm-v3.7.0-linux-amd64.tar.gz

mv linux-amd64/helm /usr/local/bin/

helm help
```



安装Helm
https://helm.sh/zh/docs/intro/install/



**Helm版本**

Helm版本支持策略
https://helm.sh/zh/docs/topics/version_skew/

```shell
Helm 版本	支持的 Kubernetes 版本
3.7.x	1.22.x - 1.19.x
3.6.x	1.21.x - 1.18.x
3.5.x	1.20.x - 1.17.x
3.4.x	1.19.x - 1.16.x
3.3.x	1.18.x - 1.15.x
3.2.x	1.18.x - 1.15.x
3.1.x	1.17.x - 1.14.x
3.0.x	1.16.x - 1.13.x
2.16.x	1.16.x - 1.15.x
2.15.x	1.15.x - 1.14.x
2.14.x	1.14.x - 1.13.x
2.13.x	1.13.x - 1.12.x
2.12.x	1.12.x - 1.11.x
2.11.x	1.11.x - 1.10.x
2.10.x	1.10.x - 1.9.x
2.9.x	1.10.x - 1.9.x
2.8.x	1.9.x - 1.8.x
2.7.x	1.8.x - 1.7.x
2.6.x	1.7.x - 1.6.x
2.5.x	1.6.x - 1.5.x
2.4.x	1.6.x - 1.5.x
2.3.x	1.5.x - 1.4.x
2.2.x	1.5.x - 1.4.x
2.1.x	1.5.x - 1.4.x
2.0.x	1.4.x - 1.3.x
```



### 4.Helm常用命令

**helm常用命令**

| 命令       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| create     | 创建一个chart并指定名字                                      |
| dependency | 管理chart依赖                                                |
| get        | 下载一个release。可用子命令：all、hooks、manifest、notes、values |
| history    | 获取release历史                                              |
| install    | 安装一个chart                                                |
| list       | 列出release                                                  |
| package    | 将chart目录打包到chart存档文件中                             |
| pull       | 从远程仓库中下载chart并解压到本地 # helm pull stable/mysql --untar |
| repo       | 添加，列出，移除，更新和索引chart仓库。可用子命令：add、index、list、remove、update |
| rollback   | 从之前版本回滚                                               |
| search     | 根据关键字搜索chart。可用子命令：hub、repo                   |
| show       | 查看chart详细信息。可用子命令：all、chart、readme、values    |
| status     | 显示已命名版本的状态                                         |
| template   | 本地呈现模板                                                 |
| uninstall  | 卸载一个release                                              |
| upgrade    | 更新一个release                                              |
| version    | 查看helm客户端版本                                           |



### 5.仓库

**配置国内chart仓库**

- 微软仓库（http://mirror.azure.cn/kubernetes/charts/）这个仓库推荐，基本上官网有的chart这里都有。
- 阿里云仓库（https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts ）
- 官方仓库（https://hub.kubeapps.com/charts/incubator）官方chart仓库，国内有点不好使。



添加存储库

```shell
helm repo add stable http://mirror.azure.cn/kubernetes/charts
helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts 
helm repo update
```

查看配置的存储库

```shell
helm repo list
helm search repo stable
```

删除存储库：

```shell
helm repo remove aliyun
```

添加常用仓库

```shell
$ helm repo add stable https://kubernetes-charts.storage.googleapis.com/
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
$ helm repo update # Make sure we get the latest list of charts
$ helm repo add ali-stable    https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts  #阿里云


helm repo add  elastic    https://helm.elastic.co       
helm repo add  gitlab     https://charts.gitlab.io       
helm repo add  harbor     https://helm.goharbor.io       
helm repo add  bitnami    https://charts.bitnami.com/bitnami       
helm repo add  incubator  https://kubernetes-charts-incubator.storage.googleapis.com       
helm repo add  stable     https://kubernetes-charts.storage.googleapis.com       
#添加国内仓库       
helm repo add stable http://mirror.azure.cn/kubernetes/charts       
helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts       
helm repo update       
helm repo list


# 安装
[root@k8s-master ~]# helm repo list
NAME         	URL                                                   
aliyun       	https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
bitnami      	https://charts.bitnami.com/bitnami                    
ingress-nginx	https://kubernetes.github.io/ingress-nginx            
gitlab       	https://charts.gitlab.io                              
elastic      	https://helm.elastic.co                               
harbor       	http://172.51.216.85:8888/chartrepo/charts            
chartmuseum  	http://172.51.216.85:9999  
```



### 6.编排资源创建顺序（Hook）

**Helm Hook**

helm官方文档：https://helm.sh/docs/topics/charts_hooks/



**Hooks**
Helm提供了Hook的机制，允许Chart开发人员在Release的生命周期中的某些节点来进行干预，比如我们可以利用Hooks来做下面的这些事情：

- 在加载Chart的其它资源之前，先加载ConfigMap或Secret
- 在安装新Chart之前执行作业以备份数据库，然后在升级后执行第二个作业以恢复数据
- 在删除Release之前运行作业，以便在删除Release之前优雅地停止服务



Hooks和普通模板一样工作，但是它们具有特殊的注释，可以使Helm以不同的方式使用它们。

Hooks在资源清单中的metadata部分用annotations注解的方式进行声明：

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  annotations:
    "helm.sh/hook": post-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": hook-succeeded
......
```



**Hooks类型**
Helm中定义了如下一些可供我们使用的Hooks：

![](/images/kubernetes/advance/helm-4.png)

请注意，为了支持Helm 3中的crds目录，已删除了crd-install钩子。



**Hooks和Release生命周期**
Hooks允许开发人员在Release的生命周期中的一些关键节点执行一些钩子，我们正常安装一个Chart包的时候的生命周期如下所示：

- 用户运行helm install foo
- Helm客户端调用Helm library库的安装API
- 经过一些验证，Helm库渲染foo模板
- Helm库将产生的资源加载到Kubernetes中去
- Helm库将Release对象（和其它数据）返回给Helm客户端
- Helm客户端退出



如果开发人员在install的生命周期中定义了两个hook：pre-install和post-install，那么我们安装一个Chart包的生命周期就会多一些步骤了：

- 用户运行helm install foo
- Helm客户端调用Helm library库的安装API
- 安装crds目录下的CRDs
- 经过一些验证，Helm库渲染foo模板
- Helm库准备执行pre-install hooks（将hook资源加载到kubernetes中）
- Helm库根据权重对hooks进行排序（默认分配权重0），权重相同时按照名称升序排序
- Helm库然后加载最低权重的hook（从负到正）
- Helm库等待，直到hook就绪Ready（除了CRDs）
- Helm库将产生的资源加载到Kubernetes中去，注意：如果设置了–wait参数，Helm库将会等待直到所有资源为就绪状态，在所有
- 源就绪前post-install hook不会执行
- Helm库执行post-install hook（将hook资源加载到kubernetes中）
- Helm库等待，直到hook就绪Ready
- Helm库将Release对象（和其它数据）返回给Helm客户端
- Helm客户端退出



等待直到hook就绪Ready意味着什么？这是一个阻塞的操作，如果 hook 中声明的是一个 Job 资源，那么 Tiller 将等待 Job 成功完成，如果失败，则发布失败，在这个期间，Helm 客户端是处于暂停状态的。

对于所有其他类型，只要 kubernetes 将资源标记为加载（添加或更新），资源就被视为就绪状态，当一个 hook 声明了很多资源是，这些资源是被串行执行的。

另外需要注意的是 hook 创建的资源不会作为 release 的一部分进行跟踪和管理，一旦 Tiller Server 验证了 hook 已经达到了就绪状态，它就不会去管它了。

所以，如果我们在 hook 中创建了资源，那么不能依赖helm delete去删除资源，因为 hook 创建的资源已经不受控制了，要销毁这些资源，需要在pre-delete或者post-delete这两个 hook 函数中去执行相关操作，或者将helm.sh/hook-delete-policy这个 annotation 添加到 hook 模板文件中。

等待直到hook就绪Ready意味着什么？这取决于钩子中声明的资源。如果资源是Job或Pod类型，Helm将等待直到成功运行完成completion为止。如果钩子失败，Release就会失败。

对于所有其它资源类型，只要Kubernetes将资源标记为已加载loaded（添加added或更新updated），该资源即被视为就绪Ready。



**写一个Hook**
Hooks也是Kubernetes清单文件，只不过是在metadata中带有特殊的注解annotation。因为Hooks也是模板文件，所以可以使用所有常规模板功能，包括读取.Values，.Release和.Template。

例如，现在我们来创建一个Hook，在前面的示例templates/目录中添加一个post-install-job.yaml的文件，表示安装后执行的一个hook：

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ .Release.Name }}"
  labels:
    app.kubernetes.io/managed-by: {{ .Release.Service | quote }}
    app.kubernetes.io/instance: {{ .Release.Name | quote }}
    app.kubernetes.io/version: {{ .Chart.AppVersion }}
    helm.sh/chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
  annotations:
    # This is what defines this resource as a hook. Without this line, the
    # job is considered part of the release.
    "helm.sh/hook": post-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    metadata:
      name: "{{ .Release.Name }}"
      labels:
        app.kubernetes.io/managed-by: {{ .Release.Service | quote }}
        app.kubernetes.io/instance: {{ .Release.Name | quote }}
        helm.sh/chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
    spec:
      restartPolicy: Never
      containers:
      - name: post-install-job
        image: "alpine:3.3"
        command: ["/bin/sleep","{{ default "10" .Values.sleepyTime }}"]
```

使这个模板成为钩子的原因是以下的注解：

```yaml
  annotations:
    "helm.sh/hook": post-install
```

一种资源也可以实现多个挂钩，如下该资源可以在安装后和升级后执行。

```yaml
annotations:
  "helm.sh/hook": post-install,post-upgrade
```

同样，一种Hook也可以有多个资源支持。例如，可以将Secret和ConfigMap都声明为预安装钩子。

当subchart声明钩子时，也会对其进行评估。顶级Chart图表无法禁用子Chart所声明的钩子。

可以为钩子定义权重，这将有助于建立确定性的执行顺序。权重使用以下注释定义：

```yaml
annotations:
  "helm.sh/hook-weight": "5"
```



**钩子删除策略**

钩子删除策略定义了何时删除相应钩子资源的策略。挂钩删除策略使用以下注释定义：

```
annotations:
  "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded
```

![](/images/kubernetes/advance/helm-5.png)



**在 Helm3 中的执行顺序**

```java
// InstallOrder is the order in which manifests should be installed (by Kind).
//
// Those occurring earlier in the list get installed before those occurring later in the list.
var InstallOrder KindSortOrder = []string{
	"Namespace",
	"NetworkPolicy",
	"ResourceQuota",
	"LimitRange",
	"PodSecurityPolicy",
	"PodDisruptionBudget",
	"Secret",
	"ConfigMap",
	"StorageClass",
	"PersistentVolume",
	"PersistentVolumeClaim",
	"ServiceAccount",
	"CustomResourceDefinition",
	"ClusterRole",
	"ClusterRoleList",
	"ClusterRoleBinding",
	"ClusterRoleBindingList",
	"Role",
	"RoleList",
	"RoleBinding",
	"RoleBindingList",
	"Service",
	"DaemonSet",
	"Pod",
	"ReplicationController",
	"ReplicaSet",
	"Deployment",
	"HorizontalPodAutoscaler",
	"StatefulSet",
	"Job",
	"CronJob",
	"Ingress",
	"APIService",
}
```

Helm 不是按照文件的某个顺序来执行的，而是根据文件内容的 `Kind`，按照定义的顺序执行的，上面的执行顺序基本上和类型间的使用（依赖）顺序有一定关联。





## 二、基础

### 1.Helm常用命令

```shell
helm常用命令：
- helm search:    搜索charts
- helm fetch:     下载charts到本地目录
- helm install:   安装charts
- helm list:      列出charts的所有版本

用法:
  helm [command]

命令可用选项:
  completion  为指定的shell生成自动补全脚本（bash或zsh）
  create      创建一个新的charts
  delete      删除指定版本的release
  dependency  管理charts的依赖
  fetch       下载charts并解压到本地目录
  get         下载一个release
  history     release历史信息
  home        显示helm的家目录
  init        在客户端和服务端初始化helm
  inspect     查看charts的详细信息
  install     安装charts
  lint        检测包的存在问题
  list        列出release
  package     将chart目录进行打包
  plugin      add(增加), list（列出）, or remove（移除） Helm 插件
  repo        add(增加), list（列出）, remove（移除）, update（更新）, and index（索引） chart仓库
  reset       卸载tiller
  rollback    release版本回滚
  search      关键字搜索chart
  serve       启动一个本地的http server
  status      查看release状态信息
  template    本地模板
  test        release测试
  upgrade     release更新
  verify      验证chart的签名和有效期
  version     打印客户端和服务端的版本信息
```



```shell
helm常见应用操作
---------------------------------------
# 列出charts仓库中所有可用的应用
helm search
# 查询指定应用
helm search memcached
# 查询指定应用的具体信息
helm inspect stable/memcached
# 用helm安装软件包,--name:指定release名字
helm install --name memcached1 stable/memcached
# 查看安装的软件包
helm list
# 删除指定引用
helm delete memcached1


helm常用命令
----------------------------------------

# chart管理
create：根据给定的name创建一个新chart
fetch：从仓库下载chart，并(可选项)将其解压缩到本地目录中
inspect：chart详情
package：打包chart目录到一个chart归档
lint：语法检测
verify：验证位于给定路径的chart已被签名且有效

# release管理
get：下载一个release
delete：根据给定的release name，从Kubernetes中删除指定的release
install：安装一个chart
list：显示release列表
upgrade：升级release
rollback：回滚release到之前的一个版本
status：显示release状态信息
history：Fetch release历史信息


helm常见操作
------------------------------------------------
# 添加仓库
helm repo add REPO_INFO   # 如：helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator
##### 示例
helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator
helm repo add elastic https://helm.elastic.co

# 查看helm仓库列表
helm repo list
# 创建chart【可供参考，一般都是自己手动创建chart】
helm create CHART_PATH
# 根据指定chart部署一个release
helm install --name RELEASE_NAME CHART_PATH
# 根据指定chart模拟安装一个release，并打印处debug信息
helm install --dry-run --debug --name RELEASE_NAME CHART_PATH
# 列出已经部署的release
helm list
# 列出所有的release
helm list --all
# 查询指定release的状态
helm status Release_NAME
# 回滚到指定版本的release，这里指定的helm release版本
helm rollback Release_NAME REVISION_NUM
# 查看指定release的历史信息
helm history Release_NAME
# 对指定chart打包
helm package CHART_PATH    如：helm package my-test-app/
# 对指定chart进行语法检测
helm lint CHART_PATH
# 查看指定chart详情
helm inspect CHART_PATH
# 从Kubernetes中删除指定release相关的资源【helm list --all 中仍然可见release记录信息】
helm delete RELEASE_NAME
# 从Kubernetes中删除指定release相关的资源，并删除release记录
helm delete --purge RELEASE_NAME
```



Chart管理常用命令

```shell
# 创建chart目录结构, 仅保留deployment, service, 删除其他对象
helm create nginx


# 打成chart包
helm package nginx
Successfully packaged chart and saved it to: /root/nginx-0.1.0.tgz


# 基于chart包安装
helm install nginx-0.1.0.tgz --generate-name


# 查看部署好的 release
helm list


# 调整chart版本并重新打包, 如将 Chart.yaml 中 version 调整为0.1.1
$ helm package nginx
Successfully packaged chart and saved it to: /root/nginx-0.1.1.tgz


# 升级 release
helm upgrade nginx-0-1582344693 nginx-0.1.1.tgz


# 查看 releae 历史记录
helm history nginx-0-1582344693
REVISION    UPDATED                     STATUS        CHART          APP VERSION    DESCRIPTION     
1           Sat Feb 22 12:11:34 2020    superseded    nginx-0.1.0    1.16.0         Install complete
2           Sat Feb 22 12:38:09 2020    deployed      nginx-0.1.1    1.16.0         Upgrade complete


# 回滚到指定版本, 1 是上面的 revision
$ helm rollback nginx-0-1582344693 1
Rollback was a success! Happy Helming!


# 回滚后状态
REVISION    UPDATED                     STATUS        CHART          APP VERSION    DESCRIPTION     
1           Sat Feb 22 12:11:34 2020    superseded    nginx-0.1.0    1.16.0         Install complete
2           Sat Feb 22 12:38:09 2020    superseded    nginx-0.1.1    1.16.0         Upgrade complete
3           Sat Feb 22 12:40:31 2020    deployed      nginx-0.1.0    1.16.0         Rollback to 1


# 最后查看release的状态
$ helm status nginx-0-1582344693
NAME: nginx-0-1582344693
LAST DEPLOYED: Sat Feb 22 12:40:31 2020
NAMESPACE: default
STATUS: deployed
REVISION: 3
TEST SUITE: None


#删除release
$ helm uninstall nginx-0-1582344693
release "nginx-0-1582344693" uninstalled


----------------------------------------------------
# 检索chart
helm search repo stable/mysql
# 查看chart信息
helm show chart stable/mysql
# 查看chart values.yaml
helm show values stable/mysql
# 拉取mysql chart
helm pull stable/mysql
# 将chart目录打成chart压缩包
helm package dirName
# 模拟安装
helm install --dry-run --debug demo1 --generate-name


https://helm.sh/docs/helm/helm/
```



### 2.Chart仓库

- 微软仓库（http://mirror.azure.cn/kubernetes/charts/）这个仓库强烈推荐，基本上官网有的chart这里都有
- 阿里云仓库（https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts ）
- 官方仓库（https://hub.kubeapps.com/charts/incubator）官方chart仓库，国内有点不好使

```shell
# 添加存储库
helm repo add stable http://mirror.azure.cn/kubernetes/charts
helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts 
helm repo update

# 查看配置的存储库
helm repo list 
NAME    URL                                                   
stable  http://mirror.azure.cn/kubernetes/charts              
aliyun  https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts

# helm  search  repo  aliyun

# 删除存储库
helm repo remove aliyun
```



```shell
$ helm repo add stable https://kubernetes-charts.storage.googleapis.com/
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
$ helm repo update # Make sure we get the latest list of charts
$ helm repo add ali-stable    https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts  #阿里云


#  helm repo list
NAME            URL                                                       
ingress-nginx   https://kubernetes.github.io/ingress-nginx                
stable          https://kubernetes-charts.storage.googleapis.com/         
bitnami         https://charts.bitnami.com/bitnami                        
incubator       https://kubernetes-charts-incubator.storage.googleapis.com/
```



**Helm基本使用**

- 安装： chart  install
- 升级： chart  upgrade
- 回滚： chart  rollback



使用 chart 部署一个应用

- 查找 chart

```shell
helm search repo mysql


[root@k8s-master test]# helm search repo mysql
NAME                         	CHART VERSION	APP VERSION	DESCRIPTION                                       
aliyun/mysql                 	0.3.5        	           	Fast, reliable, scalable, and easy to use open-...
aliyun/percona               	0.3.0        	           	free, fully compatible, enhanced, open source d...
aliyun/percona-xtradb-cluster	0.0.2        	5.7.19     	free, fully compatible, enhanced, open source d...
aliyun/gcloud-sqlproxy       	0.2.3        	           	Google Cloud SQL Proxy                            
aliyun/mariadb               	2.1.6        	10.1.31    	Fast, reliable, scalable, and easy to use open-...
```

- 查看 charts

```shell
helm show values aliyun/mysql


[root@k8s-master test]# helm show values aliyun/mysql
## mysql image version
## ref: https://hub.docker.com/r/library/mysql/tags/
##
image: "mysql"
imageTag: "5.7.14"

## Specify password for root user
##
## Default: random 10 character string
# mysqlRootPassword: testing

## Create a database user
##
# mysqlUser:
# mysqlPassword:

## Allow unauthenticated access, uncomment to enable
##
# mysqlAllowEmptyPassword: true

## Create a database
##
# mysqlDatabase:

## Specify an imagePullPolicy (Required)
## It's recommended to change this to 'Always' if the image tag is 'latest'
## ref: http://kubernetes.io/docs/user-guide/images/#updating-images
##
imagePullPolicy: IfNotPresent

livenessProbe:
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  successThreshold: 1
  failureThreshold: 3

readinessProbe:
  initialDelaySeconds: 5
  periodSeconds: 10
  timeoutSeconds: 1
  successThreshold: 1
  failureThreshold: 3

## Persist data to a persistent volume
persistence:
  enabled: true
  ## database data Persistent Volume Storage Class
  ## If defined, storageClassName: <storageClass>
  ## If set to "-", storageClassName: "", which disables dynamic provisioning
  ## If undefined (the default) or set to null, no storageClassName spec is
  ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
  ##   GKE, AWS & OpenStack)
  ##
  # storageClass: "-"
  accessMode: ReadWriteOnce
  size: 8Gi

## Configure resource requests and limits
## ref: http://kubernetes.io/docs/user-guide/compute-resources/
##
resources:
  requests:
    memory: 256Mi
    cpu: 100m

# Custom mysql configuration files used to override default mysql settings
configurationFiles:
#  mysql.cnf: |-
#    [mysqld]
#    skip-name-resolve


## Configure the service
## ref: http://kubernetes.io/docs/user-guide/services/
service:
  ## Specify a service type
  ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services---service-types
  type: ClusterIP
  port: 3306
  # nodePort: 32000

```

- 安装包

```shell
helm install mydb aliyun/mysql


[root@k8s-master test]# helm install mydb aliyun/mysql
Error: INSTALLATION FAILED: unable to build kubernetes objects from release manifest: unable to recognize "": no matches for kind "Deployment" in version "extensions/v1beta1"

# 居然报错了，deploument不支持的版本
# 把charts下载下来，看看里面的内容
# helm pull aliyun/mysql
# tar -zxvf mysql-0.3.5.tgz
# more mysql/templates/deployment.yaml 
# apiVersion: extensions/v1beta1 ===>>> apps/v1

[root@k8s-master mysql]# tree
.
├── Chart.yaml
├── README.md
├── templates
│   ├── configmap.yaml
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── NOTES.txt
│   ├── pvc.yaml
│   ├── secrets.yaml
│   └── svc.yaml
└── values.yaml



[root@k8s-master test]# helm pull aliyun/mysql
[root@k8s-master test]# ll
total 8
-rw-r--r-- 1 root root 5536 Oct  8 17:19 mysql-0.3.5.tgz

[root@k8s-master test]# tar -zxvf mysql-0.3.5.tgz

[root@k8s-master test]# ll
total 8
drwxr-xr-x 3 root root   96 Oct  8 17:20 mysql
-rw-r--r-- 1 root root 5536 Oct  8 17:19 mysql-0.3.5.tgz


[root@k8s-master test]# vim mysql/templates/deployment.yaml
# apiVersion: extensions/v1beta1 ===>>> apps/v1


[root@k8s-master test]# helm install db stable/mysql
Error: INSTALLATION FAILED: failed to download "stable/mysql"

```

- 查看 release 状态

```
helm list 


[root@k8s-master test]# helm list
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
```

- 卸载

```
helm uninstall mydb
```



**bitnami/mysql**

```shell
# 查询
[root@k8s-master test]# helm search repo mysql
NAME                         	CHART VERSION	APP VERSION	DESCRIPTION                                       
aliyun/mysql                 	0.3.5        	           	Fast, reliable, scalable, and easy to use open-...
bitnami/mysql                	8.8.8        	8.0.26     	Chart to create a Highly available MySQL cluster  
aliyun/percona               	0.3.0        	           	free, fully compatible, enhanced, open source d...
aliyun/percona-xtradb-cluster	0.0.2        	5.7.19     	free, fully compatible, enhanced, open source d...
bitnami/phpmyadmin           	8.2.16       	5.1.1      	phpMyAdmin is an mysql administration frontend    
aliyun/gcloud-sqlproxy       	0.2.3        	           	Google Cloud SQL Proxy                            
aliyun/mariadb               	2.1.6        	10.1.31    	Fast, reliable, scalable, and easy to use open-...
bitnami/mariadb              	9.6.2        	10.5.12    	Fast, reliable, scalable, and easy to use open-...
bitnami/mariadb-cluster      	1.0.2        	10.2.14    	DEPRECATED Chart to create a Highly available M...
bitnami/mariadb-galera       	6.0.1        	10.6.4     	MariaDB Galera is a multi-master database clust...


# 安装
[root@k8s-master test]# helm install mydb bitnami/mysql
NAME: mydb
LAST DEPLOYED: Sat Oct  9 09:49:19 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

Tip:

  Watch the deployment status using the command: kubectl get pods -w --namespace default

Services:

  echo Primary: mydb-mysql.default.svc.cluster.local:3306

Execute the following to get the administrator credentials:

  echo Username: root
  MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default mydb-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode)

To connect to your database:

  1. Run a pod that you can use as a client:

      kubectl run mydb-mysql-client --rm --tty -i --restart='Never' --image  docker.io/bitnami/mysql:8.0.26-debian-10-r60 --namespace default --command -- bash

  2. To connect to primary service (read/write):

      mysql -h mydb-mysql.default.svc.cluster.local -uroot -p"$MYSQL_ROOT_PASSWORD"



To upgrade this helm chart:

  1. Obtain the password as described on the 'Administrator credentials' section and set the 'root.password' parameter as shown below:

      ROOT_PASSWORD=$(kubectl get secret --namespace default mydb-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode)
      helm upgrade --namespace default mydb bitnami/mysql --set auth.rootPassword=$ROOT_PASSWORD
      

[root@k8s-master test]# helm list
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
mydb	default  	1       	2021-10-09 09:49:19.701934698 +0800 CST	deployed	mysql-8.8.8	8.0.26    


[root@k8s-master test]# kubectl get pod,sts
NAME               READY   STATUS    RESTARTS   AGE
pod/mydb-mysql-0   0/1     Pending   0          7m26s

NAME                          READY   AGE
statefulset.apps/mydb-mysql   0/1     7m26s
```



**安装weave**

```shell

[root@k8s-master test]# helm repo list
NAME  	URL                                                   
aliyun	https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts


[root@k8s-master test]# helm search repo weave
NAME              	CHART VERSION	APP VERSION	DESCRIPTION                                       
aliyun/weave-cloud	0.1.2        	           	Weave Cloud is a add-on to Kubernetes which pro...
aliyun/weave-scope	0.9.2        	1.6.5      	A Helm chart for the Weave Scope cluster visual...


helm install ui aliyun/weave-scope
```



### 3.构建一个Helm Chart

```shell
# helm create mychart
Creating mychart
# tree mychart/
mychart/
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   └── service.yaml
└── values.yaml


Chart.yaml：用于描述这个 Chart的基本信息，包括名字、描述信息以及版本等。
values.yaml ：用于存储 templates 目录中模板文件中用到变量的值。
Templates： 目录里面存放所有yaml模板文件。
charts：目录里存放这个chart依赖的所有子chart。
NOTES.txt ：用于介绍Chart帮助信息， helm install 部署后展示给用户。例如：如何使用这个 Chart、列出缺省的设置等。
_helpers.tpl：放置模板助手的地方，可以在整个 chart 中重复使用
```

```shell
[root@k8s-master test]# helm create mychart
Creating mychart
[root@k8s-master test]# ll
total 8
drwxr-xr-x 4 root root   93 Oct  8 21:50 mychart

[root@k8s-master test]# cd mychart/

[root@k8s-master mychart]# ll
total 8
drwxr-xr-x 2 root root    6 Oct  8 21:50 charts
-rw-r--r-- 1 root root 1143 Oct  8 21:50 Chart.yaml
drwxr-xr-x 3 root root  162 Oct  8 21:50 templates
-rw-r--r-- 1 root root 1874 Oct  8 21:50 values.yaml


[root@k8s-master mychart]# tree
.
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
```



**chart安装方式**

```shell
- Chart仓库（helm install harbor/harbor）

- 本地的Chart压缩包（helm install harbor-1.1.1.tgz）

- Chart目录（helm install path/to/harbor）

- 完整的URL（helm install https://test.com/charts/harbor- 1.1.1.tgz）。
```



创建Chart后，接下来就是将其部署：

```shell
[root@k8s-master test]# helm install web mychart/ -n dev
NAME: web
LAST DEPLOYED: Fri Oct  8 22:04:31 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace dev -l "app.kubernetes.io/name=mychart,app.kubernetes.io/instance=web" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace dev $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace dev port-forward $POD_NAME 8080:$CONTAINER_PORT


# 查看资源
[root@k8s-master test]# kubectl get pod,svc,deployment,hpa,ingress -n dev
NAME                               READY   STATUS    RESTARTS   AGE
pod/web-mychart-5b58c47cd6-mplxw   1/1     Running   0          2m21s

NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/web-mychart   ClusterIP   10.108.18.255   <none>        80/TCP    2m21s

NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/web-mychart   1/1     1            1           2m21s


# 访问nginx
[root@k8s-master test]# curl 10.108.18.255
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>


# 查看pod信息
[root@k8s-master test]# kubectl describe pod web-mychart-5b58c47cd6-mplxw -n dev
Name:         web-mychart-5b58c47cd6-mplxw
Namespace:    dev
Priority:     0
Node:         k8s-node2/172.51.216.83
Start Time:   Fri, 08 Oct 2021 22:04:19 +0800
Labels:       app.kubernetes.io/instance=web
              app.kubernetes.io/name=mychart
              pod-template-hash=5b58c47cd6
Annotations:  cni.projectcalico.org/containerID: a2c0626c48b02b0bbf1329f8787bc649a5aaa56854d39b5bc37087ddadfb5102
              cni.projectcalico.org/podIP: 10.244.169.166/32
              cni.projectcalico.org/podIPs: 10.244.169.166/32
Status:       Running
IP:           10.244.169.166
IPs:
  IP:           10.244.169.166
Controlled By:  ReplicaSet/web-mychart-5b58c47cd6
Containers:
  mychart:
    Container ID:   docker://9c06c218aca4088d274a9d30d39527e4b2133db35bcac83392601a3f2df01a18
    Image:          nginx:1.16.0
    Image ID:       docker-pullable://nginx@sha256:3e373fd5b8d41baeddc24be311c5c6929425c04cabf893b874ac09b72a798010
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 08 Oct 2021 22:05:06 +0800
    Ready:          True
    Restart Count:  0
    Liveness:       http-get http://:http/ delay=0s timeout=1s period=10s #success=1 #failure=3
    Readiness:      http-get http://:http/ delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from web-mychart-token-qp5r4 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  web-mychart-token-qp5r4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  web-mychart-token-qp5r4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Pulling    4m47s  kubelet            Pulling image "nginx:1.16.0"
  Normal  Scheduled  4m36s  default-scheduler  Successfully assigned dev/web-mychart-5b58c47cd6-mplxw to k8s-node2
  Normal  Pulled     4m2s   kubelet            Successfully pulled image "nginx:1.16.0" in 45.352543738s
  Normal  Created    4m1s   kubelet            Created container mychart
  Normal  Started    4m1s   kubelet            Started container mychart
```

也可以打包推送的charts仓库共享别人使用。

```shell
[root@k8s-master test]# helm package mychart/
Successfully packaged chart and saved it to: /k8s/helm/test/mychart-0.1.0.tgz
[root@k8s-master test]# ll
total 12
drwxr-xr-x 4 root root   93 Oct  8 21:58 mychart
-rw-r--r-- 1 root root 3751 Oct  8 22:11 mychart-0.1.0.tgz
```



```shell
# 查看安装的软件包
[root@k8s-master mychart]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART        	APP VERSION
web 	dev      	1       	2021-10-08 22:04:31.194590415 +0800 CST	deployed	mychart-0.1.0	1.16.0


# 从Kubernetes中删除指定release相关的资源【helm list --all 中仍然可见release记录信息】
helm delete RELEASE_NAME
# 从Kubernetes中删除指定release相关的资源，并删除release记录
helm delete --purge RELEASE_NAME


[root@k8s-master mychart]# helm delete web -n dev
release "web" uninstalled
[root@k8s-master mychart]# helm list --all -n dev
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION


[root@k8s-master mychart]# kubectl get pod,svc,deployment -n dev
No resources found in dev namespace.
```



**主要chart文件：**

```shell
[root@k8s-master mychart]# tree
.
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
```

Chart.yaml

```shell
[root@k8s-master mychart]# vim Chart.yaml 

apiVersion: v2
name: mychart
description: A Helm chart for Kubernetes

# A chart can be either an 'application' or a 'library' chart.
#
# Application charts are a collection of templates that can be packaged into versioned archives
# to be deployed.
#
# Library charts provide useful utilities or functions for the chart developer. They're included as
# a dependency of application charts to inject those utilities and functions into the rendering
# pipeline. Library charts do not define any templates and therefore cannot be deployed.
type: application

# This is the chart version. This version number should be incremented each time you make changes
# to the chart and its templates, including the app version.
# Versions are expected to follow Semantic Versioning (https://semver.org/)
version: 0.1.0

# This is the version number of the application being deployed. This version number should be
# incremented each time you make changes to the application. Versions are not expected to
# follow Semantic Versioning. They should reflect the version the application is using.
# It is recommended to use it with quotes.
appVersion: "1.16.0"
```

values.yaml

```shell
[root@k8s-master mychart]# vim values.yaml 

# Default values for mychart.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: nginx
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: ""

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

serviceAccount:
  # Specifies whether a service account should be created
  create: true
  # Annotations to add to the service account
  annotations: {}
  # The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  name: ""

podAnnotations: {}

podSecurityContext: {}
  # fsGroup: 2000

securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  className: ""
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80
  # targetMemoryUtilizationPercentage: 80

nodeSelector: {}

tolerations: []

affinity: {}
```

templates/deployment.yaml

```shell
[root@k8s-master mychart]# vim templates/deployment.yaml 

    metadata:
      {{- with .Values.podAnnotations }}
      annotations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      labels:
        {{- include "mychart.selectorLabels" . | nindent 8 }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      serviceAccountName: {{ include "mychart.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

templates/service.yaml

```shell
[root@k8s-master mychart]# vim templates/service.yaml 

apiVersion: v1
kind: Service
metadata:
  name: {{ include "mychart.fullname" . }}
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    {{- include "mychart.selectorLabels" . | nindent 4 }}
```

templates/NOTES.txt

```shell
[root@k8s-master mychart]# vim templates/NOTES.txt 

1. Get the application URL by running these commands:
{{- if .Values.ingress.enabled }}
{{- range $host := .Values.ingress.hosts }}
  {{- range .paths }}
  http{{ if $.Values.ingress.tls }}s{{ end }}://{{ $host.host }}{{ .path }}
  {{- end }}
{{- end }}
{{- else if contains "NodePort" .Values.service.type }}
  export NODE_PORT=$(kubectl get --namespace {{ .Release.Namespace }} -o jsonpath="{.spec.ports[0].nodePort}" services {{ include "mychart.fullname" . }})
  export NODE_IP=$(kubectl get nodes --namespace {{ .Release.Namespace }} -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
{{- else if contains "LoadBalancer" .Values.service.type }}
     NOTE: It may take a few minutes for the LoadBalancer IP to be available.
           You can watch the status of by running 'kubectl get --namespace {{ .Release.Namespace }} svc -w {{ include "mychart.fullname" . }}'
  export SERVICE_IP=$(kubectl get svc --namespace {{ .Release.Namespace }} {{ include "mychart.fullname" . }} --template "{{"{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}"}}")
  echo http://$SERVICE_IP:{{ .Values.service.port }}
{{- else if contains "ClusterIP" .Values.service.type }}
  export POD_NAME=$(kubectl get pods --namespace {{ .Release.Namespace }} -l "app.kubernetes.io/name={{ include "mychart.name" . }},app.kubernetes.io/instance={{ .Release.Name }}" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace {{ .Release.Namespace }} $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace {{ .Release.Namespace }} port-forward $POD_NAME 8080:$CONTAINER_PORT
{{- end }}
```



```shell
# helm pull bitnami/nginx
# helm fetch bitnami/nginx


[root@k8s-master test]# helm fetch bitnami/nginx
[root@k8s-master test]# ll
-rw-r--r-- 1 root root 37197 Oct  9 11:26 nginx-9.5.7.tgz

[root@k8s-master test]# tar -zxvf nginx-9.5.7.tgz
[root@k8s-master test]# ll
drwxr-xr-x 5 root root   164 Oct  9 11:28 nginx
-rw-r--r-- 1 root root 37197 Oct  9 11:26 nginx-9.5.7.tgz


[root@k8s-master test]# tree nginx
nginx
├── Chart.lock
├── charts
│   └── common
│       ├── Chart.yaml
│       ├── README.md
│       ├── templates
│       │   ├── _affinities.tpl
│       │   ├── _capabilities.tpl
│       │   ├── _errors.tpl
│       │   ├── _images.tpl
│       │   ├── _ingress.tpl
│       │   ├── _labels.tpl
│       │   ├── _names.tpl
│       │   ├── _secrets.tpl
│       │   ├── _storage.tpl
│       │   ├── _tplvalues.tpl
│       │   ├── _utils.tpl
│       │   ├── validations
│       │   │   ├── _cassandra.tpl
│       │   │   ├── _mariadb.tpl
│       │   │   ├── _mongodb.tpl
│       │   │   ├── _postgresql.tpl
│       │   │   ├── _redis.tpl
│       │   │   └── _validations.tpl
│       │   └── _warnings.tpl
│       └── values.yaml
├── Chart.yaml
├── ci
│   ├── ct-values.yaml
│   └── values-with-ingress-metrics-and-serverblock.yaml
├── README.md
├── templates
│   ├── deployment.yaml
│   ├── extra-list.yaml
│   ├── health-ingress.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── ldap-daemon-secrets.yaml
│   ├── NOTES.txt
│   ├── pdb.yaml
│   ├── server-block-configmap.yaml
│   ├── serviceaccount.yaml
│   ├── servicemonitor.yaml
│   ├── svc.yaml
│   └── tls-secrets.yaml
├── values.schema.json
└── values.yaml
```



### 4.helm安装nginx

通过`Helm`在`Repo`中查询可安装的`Nginx`包

```shell
[root@k8s-master mysql]# helm search repo nginx
NAME                            	CHART VERSION	APP VERSION	DESCRIPTION                                       
aliyun/nginx-ingress            	0.9.5        	0.10.2     	An nginx Ingress controller that uses ConfigMap...
aliyun/nginx-lego               	0.3.1        	           	Chart for nginx-ingress-controller and kube-lego  
bitnami/nginx                   	9.5.7        	1.21.3     	Chart for the nginx server                        
bitnami/nginx-ingress-controller	7.6.21       	0.48.1     	Chart for the nginx Ingress controller            
bitnami/kong                    	4.1.4        	2.6.0      	Kong is a scalable, open source API layer (aka ...
aliyun/gcloud-endpoints         	0.1.0        	           	Develop, deploy, protect and monitor your APIs ...
```

创建`Namespace`并且部署应用

```shell
#  创建命名空间test
[root@k8s-master test]# kubectl create namespace dev


# 查看创建的命名空间
[root@k8s-master test]# kubectl get ns


# 选择一个chart在k8s上部署我们的应用
[root@k8s-master test]# helm install nginx bitnami/nginx -n dev
NAME: nginx
LAST DEPLOYED: Sat Oct  9 10:26:43 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

NGINX can be accessed through the following DNS name from within your cluster:

    nginx.dev.svc.cluster.local (port 80)

To access NGINX from outside the cluster, follow the steps below:

1. Get the NGINX URL by running these commands:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace dev -w nginx'

    export SERVICE_PORT=$(kubectl get --namespace dev -o jsonpath="{.spec.ports[0].port}" services nginx)
    export SERVICE_IP=$(kubectl get svc --namespace dev nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    echo "http://${SERVICE_IP}:${SERVICE_PORT}"


# 查看应用状态
[root@k8s-master test]# helm status nginx -n dev
NAME: nginx
LAST DEPLOYED: Sat Oct  9 10:26:43 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

NGINX can be accessed through the following DNS name from within your cluster:

    nginx.dev.svc.cluster.local (port 80)

To access NGINX from outside the cluster, follow the steps below:

1. Get the NGINX URL by running these commands:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace dev -w nginx'

    export SERVICE_PORT=$(kubectl get --namespace dev -o jsonpath="{.spec.ports[0].port}" services nginx)
    export SERVICE_IP=$(kubectl get svc --namespace dev nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    echo "http://${SERVICE_IP}:${SERVICE_PORT}"
    
[root@k8s-master test]# helm list -n dev
NAME 	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
nginx	dev      	1       	2021-10-09 10:26:43.301121208 +0800 CST	deployed	nginx-9.5.7	1.21.3    


# 查看pod的状态
[root@k8s-master test]# kubectl get pod -n dev
NAME                     READY   STATUS              RESTARTS   AGE
nginx-7d5d5cd58c-hcshj   0/1     ContainerCreating   0          103s

[root@k8s-master test]# kubectl get deploy -n dev
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   0/1     1            0           2m21s
```



查看部署的结果

```shell
[root@k8s-master test]# kubectl get svc -n dev
NAME    TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx   LoadBalancer   10.99.29.166   <pending>     80:32233/TCP   20m


# 访问结果
[root@k8s-master test]# curl 10.99.29.166
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```



### 5.helm的核心概念

**Chart**

`Helm`采用`Chart`的格式来标准化描述一个应用（K8S 资源文件集合），`Chart`有自身标准的目录结构，可以将目录打包成版本化的压缩包进行部署。就像我们下载一个软件包之后，就可以在电脑上直接安装一样，同理`Chart`包可以通过`Helm`部署到任意的`K8S`集群中。

**Config**

`Config`指应用配置参数，在`Chart`中由`values.yaml`和命令行参数组成。`Chart`采用`Go Template`的特性 + `values.yaml`对部署的模板文件进行参数渲染，也可以通过`Helm Client`的命令`–set key=value`的方式进行参数赋值。

**Repository**

类似于`Docker Hub`，`Helm`官方、阿里云等社区都提供了`Helm Repository`，我们可以通过`helm repo add`导入仓库地址，便可以检索仓库并选择别人已经制作好的`Chart`包，开箱即用。

**Release**

`Release`代表`Chart`在集群中的运行实例，同一个集群的同一个`Namespace`下`Release`名称是唯一的。`Helm`围绕`Release`对应用提供了强大的生命周期管理能力，包括`Release`的查询、安装、更新、删除、回滚等。



**基本使用**

chart的目录

```shell
chart-demo/
├── Chart.yaml # chart原数据信息
├── charts # 应用依赖集合
├── templates # k8s资源模板集合
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml # 资源配置文件


---------------------------------------------------------------
myapp                                   - chart 包目录名
├── charts                              - 依赖的子包目录，里面可以包含多个依赖的chart包
├── Chart.yaml                          - chart定义，可以定义chart的名字，版本号信息。
├── templates                           - k8s配置模版目录， 我们编写的k8s配置都在这个目录， 除了NOTES.txt和下划线开头命名的文件，其他文件可以随意命名。
│   ├── deployment.yaml
│   ├── _helpers.tpl                    - 下划线开头的文件，helm视为公共库定义文件，主要用于定义通用的子模版、函数等，helm不会将这些公共库文件的渲染结果提交给k8s处理。
│   ├── ingress.yaml
│   ├── NOTES.txt                       - chart包的帮助信息文件，执行helm install命令安装成功后会输出这个文件的内容。
│   └── service.yaml
└── values.yaml                         - chart包的参数配置文件，模版可以引用这里参数。
```



**模板管理**

创建 Chart 骨架

```shell
helm create ./chart-demo
```

Chart 打包

```shell
helm package ./chart-demo
```

获取 Chart 包元数据信息

```shell
helm inspect chart ./chart-demo
```

本地渲染模板文件

```shell
helm template ${chart-demo-release-name} --namespace ${namespace} ./chart-demo
```

查询 Chart 依赖信息

```shell
helm dependency list ./chart-demo
```

检查依赖和模板配置是否正确

```shell
$ helm lint chart-demo
==> Linting charts/chart-demo/
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```



**模板部署**

查询 Release 列表

```shell
helm list --namespace xxxx
```

Chart 安装

```shell
helm install ${chart-demo-release-name} ./chart-demo --namespace ${namespace}
```

Chart 版本升级

```shell
helm upgrade ${chart-demo-release-name} ./chart-demo-new-version --namespace ${namespace}
```

Chart 版本回滚

```shell
helm rollback ${chart-demo-release-name} ${revision} --namespace ${namespace}
```

查看 Release 历史版本

```shell
helm history ${chart-demo-release-name} --namespace ${namespace}
```



**卸载应用**

卸载应用，并保留安装记录

```shell
helm uninstall ${chart-demo-release-name} -n ${namespace} --keep-history
```

查看全部应用（包含安装和卸载的应用）

```shell
helm list -n ${namespace} --all
```

卸载应用，不保留安装记录

```shell
helm delete ${chart-demo-release-name} -n ${namespace}
```

**${}中的替换成自己的名字**



**自定义参数安装应用**

`Helm`中支持使用自定义`yaml`文件和`--set`命令参数对要安装的应用进行参数配置，使用如下：

方式一：使用自定义`values.yaml`文件安装应用

我们知道chart的目录结构中有一个`values.yaml`，里面就是用来放参数的配置文件，修改对应的`values.yaml`就可以了

```shell
# 展示对应配置参数信息
$ helm show values bitnami/nginx
image:
  registry: docker.io
  repository: bitnami/nginx
  tag: 1.19.10-debian-10-r14
...
```

方式二：使用`--set`配置参数进行安装

`--set`参数是在使用`helm`命令时候添加的参数，可以在执行`helm`安装与更新应用时使用，多个参数间用,隔开，使用如下：

注意：如果配置文件和`--set`同时使用，则`--set`设置的参数会覆盖配置文件中的参数配置。

```shell
# 使用set创建一个release
helm install --set 'registry.registry=docker.io,registry.repository=bitnami/nginx' nginx bitnami/nginx -n blog

# 更新一个release
helm upgrade --set 'servers[0].port=8080' nginx bitnami/nginx -n blog
```



**应用发布顺序依赖**

虽然`Chart`可以通过`requirements.yaml`来管理依赖关系，并按照顺序下发模板资源，但是并无法控制子`Chart`之间的发布顺序。例如服务 B 部署必须依赖服务 A 的资源全部`Ready`。可以通过自定义子`Chart`之间的依赖顺序，在产品层控制每个子`Chart`的发布过程。



**发布应用**

上面我们使用helm初始化了一个`charts`结构，我们使用上面初始化的好的结构，把我之前打包的一个镜像，发布到k8s环境中

镜像`liz2019/main-test:1.1.72`

开始部署，为了方便查看结果，我们使用`NodePort`的类型部署，修改`values.yaml`

```
service:
#  type: ClusterIP
#  port: 80
  type: NodePort
  port: 80
```



**部署**

```
$ helm upgrade --install --force --wait --namespace test  --set image.repository=liz2019/main-test --set image.tag=1.1.72 chart-demo  ./chart-demo

Release "chart-demo" does not exist. Installing it now.
NAME: chart-demo
LAST DEPLOYED: Tue Jun 15 16:27:22 2021
NAMESPACE: test
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace test -o jsonpath="{.spec.ports[0].nodePort}" services chart-demo)
  export NODE_IP=$(kubectl get nodes --namespace test -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
```



### 6.Chart模板

https://helm.sh/docs/chart_template_guide/getting_started/



**Chart 文件结构**

使用 Helm 创建一个 nginx chart demo，文件结构如下：

```yaml
$helm create nginx
$tree nginx
nginx
├── Chart.yaml # chart版本和配置信息
├── charts     # 依赖信息
├── templates  # K8s 资源模板信息, 结合 values.yaml 可生成K8s对象的manifest文件
│   ├── NOTES.txt    # helm 提示信息
│   ├── _helpers.tpl # 下划线开头,作为子模板,可被其他模板文件引用,Helm不会交给K8s处理
│   ├── deployment.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml # chart默认配置

3 directories, 9 files
```

- `templates/` 目录下的文件都会作为K8s的 manifests 处理
- NOTES.txt 不会被处理
- **“_”** 下划线开头的都不会被处理



**Chart 模板**

Helm 使用 [Go Template](https://godoc.org/text/template) 渲染chart的数据。

https://pkg.go.dev/text/template



**内置对象**

https://helm.sh/docs/chart_template_guide/builtin_objects/



Chart 预定义对象可直接在各模板中使用。

- Release：代表Release对象，属性包含：Release.Name、Release.Namespace、Release.Revision等
- Values：表示 `values.yaml` 文件数据
- Chart：表示`Chart.yaml` 数据
- Files：用于访问chart中非标准文件
- Capabilities：用于获取k8s集群的一些信息
  - Capabilities.KubeVersion.Major：K8s的主版本
- Template：表示当前被执行的模板
  - Name：表示模板名，如：`mychart/templates/mytemplate.yaml`
  - BasePath：表示路径，如：`mychart/templates`



**流程控制**

https://helm.sh/docs/chart_template_guide/control_structures/



**if/else**

语法：

```yaml
{{ if PIPELINE }}
  # Do something
{{ else if OTHER PIPELINE }}
  # Do something else
{{ else }}
  # Default case
{{ end }}
```

demo:

```yaml
{{- if .Values.serviceAccount.create -}}
apiVersion: v1
kind: ServiceAccount
{{- end -}}
```



**with 指定范围**

with 用于修改作用域，正常 **“.”** 代表全局作用域，with 可以修改 **“.”** 的含义。

语法：

```yaml
{{ with 被引用的对象 }}
. 用来引用with指定的对象
{{ end }}
```

`values.yaml` ：

```yaml
ingress:
  enabled: false
  annotations: 
    kubernetes.io/ingress.class: nginx
    kubernetes.io/tls-acme: "true"
```

使用 with：

```yaml
kind: Ingress
metadata:
  # 指定引用对象为 ingress.annotations
  {{- with .Values.ingress.annotations }}
  annotations:
    # "." 点代表该对象
    {{- toYaml . | nindent 4 }}
  {{- end }}
```



**range 遍历**

用于遍历数组

语法：

```yaml
{{- range 数组 }}
"." 用于引用数组元素
{{- end }}
```

遍历map示例：

`values.yaml` 数据：

```yaml
env:
  open:
    JAVA_OPTS: ""
    SPRING_CLOUD_CONFIG_ENABLED: true
```

使用示例：

```yaml
    spec:
      containers:
        env:
{{- range $k, $v := .Values.env.open }}
        - name: {{ $k | quote }}
          value: {{ $v | quote }}
{{- end }}
```

遍历数组示例：

```yaml
fruits:
  - apple
  - orange
  
  
{{- range .Values.fruits}}
# 用 . 表示数组元素
{{ . | quote }} 
{{- end}}
```



**函数与管道**

模板中可使用的函数与管道，管道与Unix中类似，Helm 有超过60个函数。

官网例子：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  # 语法1: functionName arg1 arg2
  food: {{ quote .Values.favorite.food }}
  # 语法2: 使用管道
  drink: {{ .Values.favorite.drink | quote }}
```

通过 **quote** 函数为参数添加 “双引号”，渲染后的数据如下：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  food: "Pizza"
  drink: "Coffe"
```



**include 函数**

在chart中以 “下划线” 开头的文件，称为”子模版”。

例如在 `_helper.tpl` 中定义子模块，格式：`{{- define "模版名字" -}} 模版内容 {{- end -}}`

```yaml
{{- define "nginx.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" -}}
{{- end -}}
```

引用模板，格式：{{ include "模版名字" 作用域}}

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "nginx.fullname" . }}
```



**default 默认值**

通过 default 函数设置默认值

```yaml
drink: {{ .Values.favorite.drink | default "tea" | quote }}
```



**indent 缩进**

indent 表示缩进。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "nginx.fullname" . }}
  labels:
    # 缩进 4个空格
    {{- include "nginx.labels" . | indent 4 }}
```



**toYaml 转yaml**

将数据转为yaml格式

```yaml
spec:
  strategy:
{{ toYaml .Values.strategy | indent 4 }}
```

values.yaml数据：

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
```

渲染效果：

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
```



**变量定义与引用**

```yaml
# 定义变量 fullName
{{- $fullName := include "nginx.fullname" . -}}
kind: Ingress
metadata:
  # 引用变量
  name: {{ $fullName }}
```



**子模版**

子模版也叫 Named Templates、SubTemplate.



**使用 define 定义子模版**

语法：

```yaml
{{ define "MY.NAME" }}
  # body of template here
{{ end }}
```

例子：

```yaml
{{- define "mychart.labels" }}
  labels:
    generator: helm
    date: {{ now | htmlDate }}
{{- end }}
```



**使用template 引用子模板**

```yaml
{{- define "mychart.labels" }}
  labels:
    generator: helm
    date: {{ now | htmlDate }}
{{- end }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- template "mychart.labels" }}
data:
  myvalue: "Hello World"
  {{- range $key, $val := .Values.favorite }}
  {{ $key }}: {{ $val | quote }}
  {{- end }}
```



**调试模板**

Helm提供几种调试命令：

```yaml
# 验证chart是否符合最佳实践
helm lint

# 适合渲染本地的chart
helm install --dry-run --debug
helm template --debug chartDir

# 适合用于查看服务器上安装好的chart
helm get manifest
```



### 7.templates中的语法

官方文档：https://helm.sh/docs/chart_template_guide/function_list/



**_helpers.tpl**

在chart中以 “下划线” 开头的文件，称为”子模版”。
例如在 _helper.tpl 中定义子模块，格式：{{- define "模版名字" -}} 模版内容 {{- end -}}

```yaml
{{- define "nginx.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" -}}
{{- end -}}

# 若 .Values.nameOverride 为空，则默认值为 .Chart.Name
```

引用模板，格式：{{ include "模版名字" 作用域}}

```ymal
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "nginx.fullname" . }}
```



**内置对象**

Build-in Objects: https://helm.sh/docs/chart_template_guide/builtin_objects/

Chart 预定义对象可直接在各模板中使用。

```yaml
Release：      代表Release对象，属性包含：Release.Name、Release.Namespace、Release.Revision等
Values：       表示 values.yaml 文件数据
Chart：        表示 Chart.yaml 数据
Files：        用于访问 chart 中非标准文件

Capabilities： 用于获取 k8s 集群的一些信息
   - Capabilities.KubeVersion.Major：K8s的主版本

Template：     表示当前被执行的模板
   - Name：表示模板名，如：mychart/templates/mytemplate.yaml
   - BasePath：表示路径，如：mychart/templates
```



**变量**

默认情况点( . ), 代表全局作用域，用于引用全局对象。
helm 全局作用域中有两个重要的全局对象：Values 和 Release

```yaml
# Values
# 这里引用了全局作用域下的Values对象中的key属性。 
{{ .Values.key }}
Values代表的就是values.yaml定义的参数，通过.Values可以引用任意参数。
例子：
{{ .Values.replicaCount }}

# 引用嵌套对象例子，跟引用json嵌套对象类似
{{ .Values.image.repository }}

# Release 
其代表一次应用发布，下面是Release对象包含的属性字段：
Release.Name       - release的名字，一般通过Chart.yaml定义，或者通过helm命令在安装应用的时候指定。
Release.Time       - release安装时间
Release.Namespace  - k8s名字空间
Release.Revision   - release版本号，是一个递增值，每次更新都会加一
Release.IsUpgrade  - true代表，当前release是一次更新.
Release.IsInstall  - true代表，当前release是一次安装
Release.Service:   - The service that is rendering the present template. On Helm, this is always Helm.
```

自定义模版变量。

```yaml
# 变量名以$开始命名， 赋值运算符是 := (冒号+等号)
{{- $relname := .Release.Name -}}

引用自定义变量:
#不需要 . 引用
{{ $relname }}
```



**include**

```yaml
include 是一个函数，所以他的输出结果是可以传给其他函数的

# 例子1：
env:
  {{- include "xiaomage" . }}

# 结果：
          env:
- name: name
  value: xiaomage
- name: age
  value: secret
- name: favourite
  value: "Cloud Native DevSecOps"
- name: wechat
  value: majinghe11

# 例子2：
env:
  {{- include "xiaomage" . | indent 8}}

# 结果：
          env:
            - name: name
              value: xiaomage
            - name: age
              value: secret
            - name: favourite
              value: "Cloud Native DevSecOps"
            - name: wechat
              value: majinghe11
```



**with**

with 关键字可以控制变量的作用域,主要就是用来修改 . 作用域的，默认 . 代表全局作用域，with 语句可以修改 . 的含义

```yaml
# 例子：
# .Values.favorite 是一个 object 类型

{{- with .Values.favorite }}
drink: {{ .drink | default "tea" | quote }}   # 相当于.Values.favorite.drink
food:  {{ .food  | upper | quote }}
{{- end }}
```



**toYaml 转 yaml**

将数据转为yaml格式

```yaml
spec:
  strategy:
{{ toYaml .Values.strategy | indent 4 }}

------------------------------------------------------------------
values.yaml数据：
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0

------------------------------------------------------------------
渲染效果：
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
```



**Values 对象**

values 对象的值有四个来源

1. chart 包中的 values.yaml 文件
2. 父 chart 包的 values.yaml 文件
3. 使用 helm install 或者 helm upgrade 的 -f 或者 --values 参数传入的自定义的 yaml 文件
4. 通过 --set 参数传入的值

```yaml
cat global.yaml 
course: k8s

cat mychart/templates/configmap.yaml 
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  course:  {{ .Values.course }}

helm install --name mychart --dry-run --debug -f global.yaml ./mychart/
helm install --name mychart --dry-run --debug --set course="k8s" ./mychart/

# 运行部分结果:
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mychart-configmap
data:
  myvalue: "Hello World"
  course:  k8s
# 编辑 mychart/values.yaml，在最后加入
course:
  k8s: klvchen
  python: lily

cat mychart/templates/configmap.yaml 
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  k8s:  {{ quote .Values.course.k8s }}      # quote 叫双引号
  python:  {{ .Values.course.python }}

helm install --name mychart --dry-run --debug ./mychart/

# 运行结果：
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mychart-configmap
data:
  myvalue: "Hello World"
  k8s:  "klvchen"
  python:  lily
```



**管道**

```yaml
k8s:  {{ quote .Values.course.k8s }} # 加双引号
k8s:  {{ .Values.course.k8s | upper | quote }} # 大写字符串加双引号
k8s:  {{ .Values.course.k8s | repeat 3 | quote }} # 加双引号和重复3次字符串
```



**if/else 条件**

```yaml
if/else 块是用于在模板中有条件地包含文本块的方法，条件块的基本结构

{{ if PIPELINE }}
   # Do something
{{ else if OTHER PIPELINE }}
   # Do something else
{{ else }}
   # Default case
{{ end }}

# 判断条件，如果值为以下几种情况，管道的结果为 false：
1. 一个布尔类型的假
2. 一个数字零
3. 一个空的字符串
4. 一个 nil(空或null)
5. 一个空的集合(map, slice, tuple, dict, array)
除了上面的这些情况外，其他所有的条件都为真。

# 例子
cat mychart/templates/configmap.yaml 
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: {{ .Values.hello | default "Hello World" | quote }}
  k8s:  {{ .Values.course.k8s | upper | quote | repeat 3 }}
  python:  {{ .Values.course.python | repeat 3 | quote }}
  {{ if eq .Values.course.python "django" }}web: true{{ end }}

helm install --name mychart --dry-run --debug ./mychart/

运行部分结果：
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mychart-configmap
data:
  myvalue: "Hello World"
  k8s:  "KLVCHEN""KLVCHEN""KLVCHEN"
  python:  "djangodjangodjango"
  web: true

# 空格控制
{{- if eq .Values.course.python "django" }}
web: true
{{- end }}
```



**With 关键字**

```yaml
with 关键字可以控制变量的作用域

{{ .Release.xxx }} 其中的.就是表示对当前范围的引用，.Values就是告诉模板在当前范围中查找Values对象的值。

with 语句可以允许将当前范围 . 设置为特定的对象，比如我们前面一直使用的 .Values.course,我们可以
使用 with 来将范围指向 .Values.course:(templates/configmap.yaml)

with主要就是用来修改 . 作用域的，默认 . 代表全局作用域，with语句可以修改.的含义.

语法:
{{ with 引用的对象 }}
这里可以使用 . (点)， 直接引用with指定的对象
{{ end }}

例子:
#.Values.favorite是一个object类型
{{- with .Values.favorite }}
drink: {{ .drink | default "tea" | quote }}   #相当于.Values.favorite.drink
food: {{ .food | upper | quote }}
{{- end }}
```



**range 关键字**

```yaml
range主要用于循环遍历数组类型。

语法1:
# 遍历map类型，用于遍历键值对象
# 变量key代表对象的属性名，val代表属性值
{{- range key,val := 键值对象 }}
{{ $key }}: {{ $val | quote }}
{{- end}}

语法2：
{{- range 数组 }}
{{ . | title | quote }} # . (点)，引用数组元素值。
{{- end }}

例子:
# values.yaml定义
# map类型
favorite:
  drink: coffee
  food: pizza
 
# 数组类型
pizzaToppings:
  - mushrooms
  - cheese
  - peppers
  - onions
 
# map类型遍历例子:
{{- range $key, $val := .Values.favorite }}
{{ $key }}: {{ $val | quote }}
{{- end}}
 
# 数组类型遍历例子:
{{- range .Values.pizzaToppings}}
{{ . | quote }}
{{- end}}
```



### 8.CI/CD

采用 Helm 可以把零散的 Kubernetes 应用配置文件作为一个 Chart 管理，Chart 源码可以和源代码一起放到 Git 库中管理。通过把 Chart 参数化，可以在测试环境和生产环境采用不同的 Chart 参数配置。

下图是采用了 Helm 的一个 CI/CD 流程
![](/images/kubernetes/advance/helm-3.png)

- Helm 如何管理多环境下 (Test、Staging、Production) 的业务配置


Chart 是支持参数替换的，可以把业务配置相关的参数设置为模板变量。使用 helm install 命令部署的时候指定一个参数值文件，这样就可以把业务参数从 Chart 中剥离了。例如： helm install --values=values-production.yaml wordpress。

- Helm 如何解决服务依赖


在 Chart 里可以通过 requirements.yaml 声明对其它 Chart 的依赖关系。如下面声明表明 Chart 依赖 Apache 和 MySQL 这两个第三方 Chart。

```yaml
dependencies:
- name: mariadb
version: 2.1.1
repository: https://kubernetes-charts.storage.googleapis.com/
condition: mariadb.enabled
tags:
- wordpress-database
- name: apache
version: 1.4.0
repository: https://kubernetes-charts.storage.googleapis.com/
```

- 
  如何让 Helm 连接到指定 Kubernetes 集群

Helm 默认使用和 kubectl 命令相同的配置访问 Kubernetes 集群，其配置默认在 ~/.kube/config 中。

- 如何在部署时指定命名空间


helm install 默认情况下是部署在 default 这个命名空间的。如果想部署到指定的命令空间，可以加上 --namespace 参数，比如：

```shell
$ helm install local/mychart --name mike-test --namespace mynamespace
```

- 如何查看已部署应用的详细信息

```shell
$ helm get wordpress-test
```

默认情况下会显示最新的版本的相关信息，如果想要查看指定发布版本的信息可加上 --revision 参数。

```shell
$ helm get  --revision 1  wordpress-test
```




