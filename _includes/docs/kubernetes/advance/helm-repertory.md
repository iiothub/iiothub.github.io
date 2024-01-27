* TOC
{:toc}



## 一、概述



Helm仓库可以选择Harbor、ChartMuseum。



### 1.Harbor

Harbor，是一个英文单词，意思是港湾，港湾是干什么的呢，就是停放货物的，而货物呢，是装在集装箱中的，说到集装箱，就不得不提到Docker容器，因为docker容器的技术正是借鉴了集装箱的原理。所以，Harbor正是一个用于存储Docker镜像的企业级Registry服务。

Docker容器应用的开发和运行离不开可靠的镜像管理，虽然Docker官方也提供了公共的镜像仓库，但是从安全和效率等方面考虑，部署我们私有环境内的Registry也是非常必要的。Harbor是由VMware公司开源的企业级的Docker Registry管理项目，它包括权限管理(RBAC)、LDAP、日志审核、管理界面、自我注册、镜像复制和中文支持等功能。



```shell
# 官方下载地址：

https://goharbor.io/
https://github.com/goharbor/harbor
https://github.com/goharbor/harbor/releases
```



### 2.ChartMuseum

ChartMuseum对于经常用到Helm Charts存储库的开发人员来说，非常实用且方便。作为一个存储库，它被设计为能与各种流行的Kubernetes环境和服务进行协同工作。其中包括Microsoft Azure的Blob存储和Oracle的云架构对象存储。

Helm chart对仓库的要求并不高，需要你对外提供yaml文件和tar文件的web服务即可。但是实际的操作中我们还需要考虑更多的操作。

Chartmuseum除了给我们提供一个类似于web服务器的功能之外，还提供了其他有用的功能，便于日常我们私有仓库的管理。

根据chart文件自动生成index.yaml(无须使用helm repo index手动生成)
helm push的插件，可以在helm命令之上实现将chart文件推送到chartmuseum上
相应的tls配置，Basic认证，JWT认证（Bearer token认证）
提供了Restful的api（可以使用curl命令操作）和可以使用的cli命令行工具
提供了各种后端存储的支持（Amazon s3, Google Cloud Storage, 阿里、百度、腾讯，开源对象存储等）
提供了Prometheus的集成，对外提供自己的监控信息。
没有用户的概念，但是基于目录实现了一定程度上的多租户的需求
安装

官方提供了相应的helmchart，可以在kuberentes上直接安装。也提供了docker的镜像方式安装。本文介绍docker的方式进行安装部署。


```shell
docker run --rm -it \
  -p 8080:8080 \
  -e DEBUG=1 \
  -e STORAGE=local \
  -e STORAGE_LOCAL_ROOTDIR=/charts \
  -v $(pwd)/charts:/charts \
  ghcr.io/helm/chartmuseum:v0.13.1
```



```shell
# 官方地址：

https://chartmuseum.com/
https://github.com/helm/chartmuseum
https://github.com/chartmuseum/helm-push
https://github.com/chartmuseum/ui
```





## 二、Harbor



### 1.安装 push 插件

Helm服务器安装helm-push插件

```shell
# master(172.51.216.81，172.51.216.82，172.51.216.83)，安装helm服务器

# 安装
helm plugin install https://github.com/chartmuseum/helm-push 
# 查看已成功
helm plugin list


# 安装
[root@k8s-master ~]# helm plugin install https://github.com/chartmuseum/helm-push

# 查看已成功
[root@k8s-master helm]# helm plugin list
NAME   	VERSION	DESCRIPTION                      
cm-push	0.10.1 	Push chart package to ChartMuseum
```



**本地安装**

```shell
# 下载 helm-push_0.10.1_linux_amd64.tar.gz
https://github.com/chartmuseum/helm-push/releases/download/v0.10.1/helm-push_0.10.1_linux_amd64.tar.gz


# 解压
[root@k8s-master2 helm]# tar -zxf helm-push_0.10.1_linux_amd64.tar.gz 
[root@k8s-master2 helm]# ll
total 23636
drwxr-xr-x. 2 root root         26 Dec 23 10:51 bin
-rw-r--r--. 1 root root   10481411 Dec 23 10:51 helm-push_0.10.1_linux_amd64.tar.gz
-rw-r--r--. 1 1001 docker    11357 Oct 12 22:09 LICENSE
-rw-r--r--. 1 1001 docker      407 Oct 12 22:09 plugin.yaml


# 安装
[root@k8s-master2 helm]# helm plugin install .
sh: scripts/install_plugin.sh: No such file or directory
Error: plugin install hook for "cm-push" exited with error


# 测试
[root@k8s-master2 helm]# helm cm-push --help
Helm plugin to push chart package to ChartMuseum

Examples:

  $ helm cm-push mychart-0.1.0.tgz chartmuseum       # push .tgz from "helm package"
  $ helm cm-push . chartmuseum                       # package and push chart directory
  $ helm cm-push . --version="1.2.3" chartmuseum     # override version in Chart.yaml
  $ helm cm-push . https://my.chart.repo.com         # push directly to chart repo URL

Usage:
  helm cm-push [flags]

Flags:
      --access-token string             Send token in Authorization header [$HELM_REPO_ACCESS_TOKEN]
  -a, --app-version string              Override app version pre-push
      --auth-header string              Alternative header to use for token auth [$HELM_REPO_AUTH_HEADER]
      --ca-file string                  Verify certificates of HTTPS-enabled servers using this CA bundle [$HELM_REPO_CA_FILE]
      --cert-file string                Identify HTTPS client using this SSL certificate file [$HELM_REPO_CERT_FILE]
      --check-helm-version              outputs either "2" or "3" indicating the current Helm major version
      --context-path string             ChartMuseum context path [$HELM_REPO_CONTEXT_PATH]
      --debug                           Enable verbose output
  -d, --dependency-update               update dependencies from "requirements.yaml" to dir "charts/" before packaging
  -f, --force                           Force upload even if chart version exists
  -h, --help                            help for helm
      --home string                     Location of your Helm config. Overrides $HELM_HOME (default "/root/.helm")
      --host string                     Address of Tiller. Overrides $HELM_HOST
      --insecure                        Connect to server with an insecure way by skipping certificate verification [$HELM_REPO_INSECURE]
      --key-file string                 Identify HTTPS client using this SSL key file [$HELM_REPO_KEY_FILE]
      --keyring string                  location of a public keyring (default "/root/.gnupg/pubring.gpg")
      --kube-context string             Name of the kubeconfig context to use
      --kubeconfig string               Absolute path of the kubeconfig file to be used
  -p, --password string                 Override HTTP basic auth password [$HELM_REPO_PASSWORD]
      --tiller-connection-timeout int   The duration (in seconds) Helm will wait to establish a connection to Tiller (default 300)
      --tiller-namespace string         Namespace of Tiller (default "kube-system")
  -t, --timeout int                     The duration (in seconds) Helm will wait to get response from chartmuseum (default 30)
  -u, --username string                 Override HTTP basic auth username [$HELM_REPO_USERNAME]
  -v, --version string                  Override chart version pre-push
```



### 2.Push命令

```shell
[root@k8s-master2 helm]# helm cm-push --help
Helm plugin to push chart package to ChartMuseum

Examples:

  $ helm cm-push mychart-0.1.0.tgz chartmuseum       # push .tgz from "helm package"
  $ helm cm-push . chartmuseum                       # package and push chart directory
  $ helm cm-push . --version="1.2.3" chartmuseum     # override version in Chart.yaml
  $ helm cm-push . https://my.chart.repo.com         # push directly to chart repo URL

Usage:
  helm cm-push [flags]

Flags:
      --access-token string             Send token in Authorization header [$HELM_REPO_ACCESS_TOKEN]
  -a, --app-version string              Override app version pre-push
      --auth-header string              Alternative header to use for token auth [$HELM_REPO_AUTH_HEADER]
      --ca-file string                  Verify certificates of HTTPS-enabled servers using this CA bundle [$HELM_REPO_CA_FILE]
      --cert-file string                Identify HTTPS client using this SSL certificate file [$HELM_REPO_CERT_FILE]
      --check-helm-version              outputs either "2" or "3" indicating the current Helm major version
      --context-path string             ChartMuseum context path [$HELM_REPO_CONTEXT_PATH]
      --debug                           Enable verbose output
  -d, --dependency-update               update dependencies from "requirements.yaml" to dir "charts/" before packaging
  -f, --force                           Force upload even if chart version exists
  -h, --help                            help for helm
      --home string                     Location of your Helm config. Overrides $HELM_HOME (default "/root/.helm")
      --host string                     Address of Tiller. Overrides $HELM_HOST
      --insecure                        Connect to server with an insecure way by skipping certificate verification [$HELM_REPO_INSECURE]
      --key-file string                 Identify HTTPS client using this SSL key file [$HELM_REPO_KEY_FILE]
      --keyring string                  location of a public keyring (default "/root/.gnupg/pubring.gpg")
      --kube-context string             Name of the kubeconfig context to use
      --kubeconfig string               Absolute path of the kubeconfig file to be used
  -p, --password string                 Override HTTP basic auth password [$HELM_REPO_PASSWORD]
      --tiller-connection-timeout int   The duration (in seconds) Helm will wait to establish a connection to Tiller (default 300)
      --tiller-namespace string         Namespace of Tiller (default "kube-system")
  -t, --timeout int                     The duration (in seconds) Helm will wait to get response from chartmuseum (default 30)
  -u, --username string                 Override HTTP basic auth username [$HELM_REPO_USERNAME]
  -v, --version string                  Override chart version pre-push
```



### 3.添加Helm仓库

```shell
# 创建项目springcloud


# 添加 helm repo
helm repo add --username admin --password admin harborcloud http://172.51.216.88:8888/chartrepo/springcloud
helm repo list


# 查看
[root@k8s-master1 chartmuseum]# helm repo list
NAME         	URL                                                   
stable       	http://mirror.azure.cn/kubernetes/charts              
aliyun       	https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
bitnami      	https://charts.bitnami.com/bitnami                    
ingress-nginx	https://kubernetes.github.io/ingress-nginx            
elastic      	https://helm.elastic.co                               
presslabs    	https://presslabs.github.io/charts                    
timescale    	https://charts.timescale.com


# 添加 helm repo
[root@k8s-master chartmuseum]# helm repo add --username admin --password admin harborcloud http://172.51.216.88:8888/chartrepo/springcloud
"harborcloud" has been added to your repositories

[root@k8s-master1 test]# helm repo list
NAME         	URL                                                   
stable       	http://mirror.azure.cn/kubernetes/charts              
aliyun       	https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
bitnami      	https://charts.bitnami.com/bitnami                    
ingress-nginx	https://kubernetes.github.io/ingress-nginx            
elastic      	https://helm.elastic.co                               
presslabs    	https://presslabs.github.io/charts                    
timescale    	https://charts.timescale.com                          
chartmuseum  	http://172.51.216.88:9999                             
harborcloud  	http://172.51.216.88:8888/chartrepo/springcloud


# 更新
[root@k8s-master chartmuseum]# helm repo update
```



### 4.上传Helm仓库

```shell
$ helm cm-push mychart/ chartmuseum
Pushing mychart-0.3.2.tgz to chartmuseum...
Done.


$ helm cm-push mychart/ --version="1.2.3" chartmuseum
Pushing mychart-1.2.3.tgz to chartmuseum...
Done.


$ helm cm-push mychart-0.3.2.tgz chartmuseum
Pushing mychart-0.3.2.tgz to chartmuseum...
Done.
```



```shell
# s上传

[root@k8s-master1 test]# helm cm-push cloud/ harborcloud
Pushing cloud-1.0.0.tgz to harborcloud...
Done.


[root@k8s-master1 test]# helm cm-push cloud-1.0.0.tgz harborcloud
Pushing cloud-1.0.0.tgz to harborcloud...
Done.
```



![](/images/kubernetes/advance/harbor-11.png)

![](/images/kubernetes/advance/harbor-12.png)

![](/images/kubernetes/advance/harbor-13.png)

```shell
# 添加仓库
helm repo add --ca-file <ca file> --cert-file <cert file> --key-file <key file>     --username <username> --password <password> <repo name> http://172.51.216.88:8888/chartrepo/springcloud


# 安装chart
helm install --ca-file <ca file> --cert-file <cert file> --key-file <key file>     --username=<username> --password=<password> --version 1.0.0 <repo name>/cloud
```



### 5.日常操作

```shell
# 常用命令

[root@k8s-master2 harbor]# helm repo --help

This command consists of multiple subcommands to interact with chart repositories.

It can be used to add, remove, list, and index chart repositories.

Usage:
  helm repo [command]

Available Commands:
  add         add a chart repository
  index       generate an index file given a directory containing packaged charts
  list        list chart repositories
  remove      remove one or more chart repositories
  update      update information of available charts locally from chart repositories

Flags:
  -h, --help   help for repo

Global Flags:
      --debug                       enable verbose output
      --kube-apiserver string       the address and the port for the Kubernetes API server
      --kube-as-group stringArray   group to impersonate for the operation, this flag can be repeated to specify multiple groups.
      --kube-as-user string         username to impersonate for the operation
      --kube-ca-file string         the certificate authority file for the Kubernetes API server connection
      --kube-context string         name of the kubeconfig context to use
      --kube-token string           bearer token used for authentication
      --kubeconfig string           path to the kubeconfig file
  -n, --namespace string            namespace scope for this request
      --registry-config string      path to the registry config file (default "/root/.config/helm/registry.json")
      --repository-cache string     path to the file containing cached repository indexes (default "/root/.cache/helm/repository")
      --repository-config string    path to the file containing repository names and URLs (default "/root/.config/helm/repositories.yaml")

Use "helm repo [command] --help" for more information about a command.
```



#### 5.1.添加仓库

```shell
# 添加仓库


helm repo add --ca-file <ca file> --cert-file <cert file> --key-file <key file>     --username <username> --password <password> <repo name> http://172.51.216.88:8888/chartrepo/springcloud
```

```shell
[root@k8s-master2 harbor]# helm repo add --username admin --password admin harborcloud http://172.51.216.88:8888/chartrepo/springcloud
"harborcloud" has been added to your repositories


[root@k8s-master2 harbor]# helm repo list
NAME         	URL                                                   
stable       	http://mirror.azure.cn/kubernetes/charts              
aliyun       	https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
bitnami      	https://charts.bitnami.com/bitnami                    
ingress-nginx	https://kubernetes.github.io/ingress-nginx            
elastic      	https://helm.elastic.co                               
harborcloud  	http://172.51.216.88:8888/chartrepo/springcloud


[root@k8s-master2 harbor]# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "harborcloud" chart repository
...Successfully got an update from the "ingress-nginx" chart repository
...Successfully got an update from the "aliyun" chart repository
...Successfully got an update from the "elastic" chart repository
...Successfully got an update from the "bitnami" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈
```



#### 5.2.上传

```shell
# 参考 
 
  $ helm cm-push mychart-0.1.0.tgz chartmuseum       # push .tgz from "helm package"
  $ helm cm-push . chartmuseum                       # package and push chart directory
  $ helm cm-push . --version="1.2.3" chartmuseum     # override version in Chart.yaml
  $ helm cm-push . https://my.chart.repo.com         # push directly to chart repo URL
```

```shell
# 查看基本信息

[root@k8s-master2 harbor]# vim cloud/Chart.yaml 

apiVersion: v2
# 名称
name: cloud

description: A Helm chart for Kubernetes
type: application

# 版本
version: 1.0.0

appVersion: "1.0.1"



[root@k8s-master2 harbor]# tree
.
└── cloud
    ├── charts
    ├── Chart.yaml
    ├── templates
    │   ├── msa-deploy-consumer.yaml
    │   ├── msa-deploy-job.yaml
    │   ├── msa-deploy-producer.yaml
    │   ├── msa-eureka.yaml
    │   ├── msa-gateway.yaml
    │   ├── msa-test-http.yaml
    │   ├── sentinel-server.yaml
    │   ├── xxl-job.yaml
    │   └── zipkin-server.yaml
    └── values.yaml

3 directories, 11 files
```

```shell
# 上传
[root@k8s-master2 harbor]# helm cm-push cloud/ harborcloud
Pushing cloud-1.0.0.tgz to harborcloud...
Done.


# 更新
[root@k8s-master2 harbor]# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "harborcloud" chart repository
...Successfully got an update from the "elastic" chart repository
...Successfully got an update from the "aliyun" chart repository
...Successfully got an update from the "ingress-nginx" chart repository
...Successfully got an update from the "presslabs" chart repository
...Successfully got an update from the "stable" chart repository
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈


# 查询
[root@k8s-master2 harbor]# helm search repo harborcloud/cloud
NAME             	CHART VERSION	APP VERSION	DESCRIPTION                
harborcloud/cloud	1.0.0        	1.0.1      	A Helm chart for Kubernetes


# 获取
[root@k8s-master2 harbor]# helm fetch harborcloud/cloud
[root@k8s-master2 harbor]# ll
total 8
drwxr-xr-x 4 root root   93 Dec 27 10:56 cloud
-rw-r--r-- 1 root root 2952 Dec 27 11:07 cloud-1.0.0.tgz
```



#### 5.3.安装

```shell
# 安装chart

helm install --ca-file <ca file> --cert-file <cert file> --key-file <key file>     --username=<username> --password=<password> --version 1.0.0 <repo name>/cloud
```

```shell
# 远程安装
[root@k8s-master2 harbor]# helm install msa harborcloud/cloud --version 1.0.0 -n test
NAME: msa
LAST DEPLOYED: Thu Dec 23 15:11:10 2021
NAMESPACE: test
STATUS: deployed
REVISION: 1
TEST SUITE: None


# 查看
[root@k8s-master2 harbor]# helm list -n test
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	test     	1       	2021-12-23 15:11:10.110605035 +0800 CST	deployed	cloud-1.0.0	1.0.1  


[root@k8s-master2 harbor]# kubectl get all -n test
NAME                                       READY   STATUS    RESTARTS   AGE
pod/msa-deploy-consumer-84477b66c4-2nrjb   1/1     Running   0          2m22s
pod/msa-deploy-consumer-84477b66c4-4xz7v   1/1     Running   0          2m22s
pod/msa-deploy-consumer-84477b66c4-gzj2n   1/1     Running   0          2m22s
pod/msa-deploy-job-85cd75b9fc-dftk6        1/1     Running   0          2m22s
pod/msa-deploy-job-85cd75b9fc-qflxs        1/1     Running   0          2m22s
pod/msa-deploy-job-85cd75b9fc-qkpmn        1/1     Running   0          2m22s
pod/msa-deploy-producer-75b4f85887-8m92v   1/1     Running   0          2m22s
pod/msa-deploy-producer-75b4f85887-pzgb6   1/1     Running   0          2m22s
pod/msa-deploy-producer-75b4f85887-zq7mf   1/1     Running   0          2m22s
pod/msa-eureka-0                           1/1     Running   0          2m22s
pod/msa-eureka-1                           1/1     Running   0          2m22s
pod/msa-eureka-2                           1/1     Running   0          2m22s
pod/msa-gateway-799b85ffd8-ffxjd           1/1     Running   0          2m22s
pod/msa-gateway-799b85ffd8-xlgnk           1/1     Running   0          2m22s
pod/msa-gateway-799b85ffd8-zrr5f           1/1     Running   0          2m22s
pod/sentinel-server-8597dd6fcc-2kkd5       1/1     Running   0          2m22s
pod/xxl-job-77bc8f696d-9zhv4               1/1     Running   0          2m22s
pod/xxl-job-77bc8f696d-n44rp               1/1     Running   0          2m22s
pod/xxl-job-77bc8f696d-thnml               1/1     Running   0          2m22s
pod/zipkin-server-75d485f8d8-4czhc         1/1     Running   0          2m22s
pod/zipkin-server-75d485f8d8-6rvxb         1/1     Running   0          2m22s
pod/zipkin-server-75d485f8d8-xvt2j         1/1     Running   0          2m22s

NAME                          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
service/msa-deploy-consumer   ClusterIP   10.1.156.132   <none>        8912/TCP    2m22s
service/msa-deploy-job        ClusterIP   10.1.13.226    <none>        8913/TCP    2m22s
service/msa-deploy-producer   ClusterIP   10.1.33.181    <none>        8911/TCP    2m22s
service/msa-eureka            ClusterIP   10.1.0.2       <none>        10001/TCP   2m22s
service/msa-gateway           ClusterIP   10.1.78.250    <none>        8888/TCP    2m22s
service/sentinel-server       ClusterIP   10.1.101.31    <none>        8858/TCP    2m22s
service/xxl-job               ClusterIP   10.1.251.6     <none>        8080/TCP    2m22s
service/zipkin-server         ClusterIP   10.1.166.132   <none>        9411/TCP    2m22s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/msa-deploy-consumer   3/3     3            3           2m22s
deployment.apps/msa-deploy-job        3/3     3            3           2m22s
deployment.apps/msa-deploy-producer   3/3     3            3           2m22s
deployment.apps/msa-gateway           3/3     3            3           2m22s
deployment.apps/sentinel-server       1/1     1            1           2m22s
deployment.apps/xxl-job               3/3     3            3           2m22s
deployment.apps/zipkin-server         3/3     3            3           2m22s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/msa-deploy-consumer-84477b66c4   3         3         3       2m22s
replicaset.apps/msa-deploy-job-85cd75b9fc        3         3         3       2m22s
replicaset.apps/msa-deploy-producer-75b4f85887   3         3         3       2m22s
replicaset.apps/msa-gateway-799b85ffd8           3         3         3       2m22s
replicaset.apps/sentinel-server-8597dd6fcc       1         1         1       2m22s
replicaset.apps/xxl-job-77bc8f696d               3         3         3       2m22s
replicaset.apps/zipkin-server-75d485f8d8         3         3         3       2m22s

NAME                          READY   AGE
statefulset.apps/msa-eureka   3/3     2m22s
```



#### 5.4.升级

```shell
# 修改

[root@k8s-master2 cloud]# vim Chart.yaml 

apiVersion: v2
name: cloud
description: A Helm chart for Kubernetes
type: application

# 修改版本号
version: 2.0.0
appVersion: "2.0.1"
```

```shell
# 上传

[root@k8s-master2 harbor]# helm cm-push cloud/ harborcloud
Pushing cloud-2.0.0.tgz to harborcloud...
Done.
```

![](/images/kubernetes/advance/harbor-14.png)

```shell
# 升级

[root@k8s-master2 harbor]# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "harborcloud" chart repository
...Successfully got an update from the "aliyun" chart repository
...Successfully got an update from the "elastic" chart repository
...Successfully got an update from the "ingress-nginx" chart repository
...Successfully got an update from the "bitnami" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈


[root@k8s-master2 harbor]# helm upgrade msa harborcloud/cloud --version 2.0.0 -n test
Release "msa" has been upgraded. Happy Helming!
NAME: msa
LAST DEPLOYED: Thu Dec 23 15:24:31 2021
NAMESPACE: test
STATUS: deployed
REVISION: 2
TEST SUITE: None


[root@k8s-master2 harbor]# helm list -n test
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	test     	2       	2021-12-23 15:24:31.198256379 +0800 CST	deployed	cloud-2.0.0	2.0.1
```



#### 5.5.删除

```shell
# Harbor中的文件从页面删除

[root@k8s-master2 harbor]# helm repo list
NAME         	URL                                                   
stable       	http://mirror.azure.cn/kubernetes/charts              
aliyun       	https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
bitnami      	https://charts.bitnami.com/bitnami                    
ingress-nginx	https://kubernetes.github.io/ingress-nginx            
elastic      	https://helm.elastic.co                               
harborcloud  	http://172.51.216.88:8888/chartrepo/springcloud


[root@k8s-master2 harbor]# helm repo remove harborcloud
"harborcloud" has been removed from your repositories


[root@k8s-master2 harbor]# helm repo list
NAME         	URL                                                   
stable       	http://mirror.azure.cn/kubernetes/charts              
aliyun       	https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
bitnami      	https://charts.bitnami.com/bitnami                    
ingress-nginx	https://kubernetes.github.io/ingress-nginx            
elastic      	https://helm.elastic.co
```



