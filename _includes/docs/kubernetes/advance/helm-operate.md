* TOC
{:toc}


## 一、Helm



### 1.安装Helm

安装过程

```shell
wget https://get.helm.sh/helm-v3.7.0-linux-amd64.tar.gz

tar zxvf helm-v3.7.0-linux-amd64.tar.gz

cp linux-amd64/helm /usr/local/bin/

helm help
```

```shell
# 下载安装文件
[root@k8s-master install]# wget https://get.helm.sh/helm-v3.7.0-linux-amd64.tar.gz


# 解压
[root@k8s-master install]# tar zxvf helm-v3.7.0-linux-amd64.tar.gz
linux-amd64/
linux-amd64/helm
linux-amd64/LICENSE
linux-amd64/README.md

[root@k8s-master install]# tree
.
├── helm-v3.7.0-linux-amd64.tar.gz
└── linux-amd64
    ├── helm
    ├── LICENSE
    └── README.md


[root@k8s-master install]# cd linux-amd64/
[root@k8s-master linux-amd64]# ls
helm  LICENSE  README.md


# 复制
[root@k8s-master linux-arm64]# cp helm /usr/local/bin/

# 命令补全
# 添加环境变量
[root@k8s-master linux-amd64]# echo "source <(helm completion bash)" >> ~/.bashrc
[root@k8s-master linux-amd64]# source ~/.bashrc


# 查看版本
[root@k8s-master linux-amd64]# helm version
version.BuildInfo{Version:"v3.7.0", GitCommit:"eeac83883cb4014fe60267ec6373570374ce770b", GitTreeState:"clean", GoVersion:"go1.16.8"}
```



```shell
[root@k8s-master linux-amd64]# helm

[root@k8s-master linux-amd64]# helm help
The Kubernetes package manager

Common actions for Helm:

- helm search:    search for charts
- helm pull:      download a chart to your local directory to view
- helm install:   upload the chart to Kubernetes
- helm list:      list releases of charts

Environment variables:

| Name                               | Description                                                                       |
|------------------------------------|-----------------------------------------------------------------------------------|
| $HELM_CACHE_HOME                   | set an alternative location for storing cached files.                             |
| $HELM_CONFIG_HOME                  | set an alternative location for storing Helm configuration.                       |
| $HELM_DATA_HOME                    | set an alternative location for storing Helm data.                                |
| $HELM_DEBUG                        | indicate whether or not Helm is running in Debug mode                             |
| $HELM_DRIVER                       | set the backend storage driver. Values are: configmap, secret, memory, sql.       |
| $HELM_DRIVER_SQL_CONNECTION_STRING | set the connection string the SQL storage driver should use.                      |
| $HELM_MAX_HISTORY                  | set the maximum number of helm release history.                                   |
| $HELM_NAMESPACE                    | set the namespace used for the helm operations.                                   |
| $HELM_NO_PLUGINS                   | disable plugins. Set HELM_NO_PLUGINS=1 to disable plugins.                        |
| $HELM_PLUGINS                      | set the path to the plugins directory                                             |
| $HELM_REGISTRY_CONFIG              | set the path to the registry config file.                                         |
| $HELM_REPOSITORY_CACHE             | set the path to the repository cache directory                                    |
| $HELM_REPOSITORY_CONFIG            | set the path to the repositories file.                                            |
| $KUBECONFIG                        | set an alternative Kubernetes configuration file (default "~/.kube/config")       |
| $HELM_KUBEAPISERVER                | set the Kubernetes API Server Endpoint for authentication                         |
| $HELM_KUBECAFILE                   | set the Kubernetes certificate authority file.                                    |
| $HELM_KUBEASGROUPS                 | set the Groups to use for impersonation using a comma-separated list.             |
| $HELM_KUBEASUSER                   | set the Username to impersonate for the operation.                                |
| $HELM_KUBECONTEXT                  | set the name of the kubeconfig context.                                           |
| $HELM_KUBETOKEN                    | set the Bearer KubeToken used for authentication.                                 |

Helm stores cache, configuration, and data based on the following configuration order:

- If a HELM_*_HOME environment variable is set, it will be used
- Otherwise, on systems supporting the XDG base directory specification, the XDG variables will be used
- When no other location is set a default location will be used based on the operating system

By default, the default directories depend on the Operating System. The defaults are listed below:

| Operating System | Cache Path                | Configuration Path             | Data Path               |
|------------------|---------------------------|--------------------------------|-------------------------|
| Linux            | $HOME/.cache/helm         | $HOME/.config/helm             | $HOME/.local/share/helm |
| macOS            | $HOME/Library/Caches/helm | $HOME/Library/Preferences/helm | $HOME/Library/helm      |
| Windows          | %TEMP%\helm               | %APPDATA%\helm                 | %APPDATA%\helm          |

Usage:
  helm [command]

Available Commands:
  completion  generate autocompletion scripts for the specified shell
  create      create a new chart with the given name
  dependency  manage a chart's dependencies
  env         helm client environment information
  get         download extended information of a named release
  help        Help about any command
  history     fetch release history
  install     install a chart
  lint        examine a chart for possible issues
  list        list releases
  package     package a chart directory into a chart archive
  plugin      install, list, or uninstall Helm plugins
  pull        download a chart from a repository and (optionally) unpack it in local directory
  repo        add, list, remove, update, and index chart repositories
  rollback    roll back a release to a previous revision
  search      search for a keyword in charts
  show        show information of a chart
  status      display the status of the named release
  template    locally render templates
  test        run tests for a release
  uninstall   uninstall a release
  upgrade     upgrade a release
  verify      verify that a chart at the given path has been signed and is valid
  version     print the client version information

Flags:
      --debug                       enable verbose output
  -h, --help                        help for helm
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

Use "helm [command] --help" for more information about a command.
```



### 2.部署Nginx

#### 2.1.创建

```yaml
# 创建mynginx
[root@k8s-master test]# helm create mynginx -n dev
Creating mynginx

[root@k8s-master test]# ll
drwxr-xr-x 4 root root    93 Oct 11 11:25 mynginx

[root@k8s-master test]# tree mynginx
mynginx
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

3 directories, 10 files


--------------------------------------------
# 删除多余文件
# templates只保留deployment.yaml、service.yaml、NOTES.txt

[root@k8s-master test]# tree mynginx
mynginx
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── NOTES.txt
│   └── service.yaml
└── values.yaml
```

```yaml
#模板文件

# Chart.yaml
[root@k8s-master mynginx]# vim Chart.yaml

apiVersion: v2
name: mynginx
description: A Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: "1.15.0"


# values.yaml
[root@k8s-master mynginx]# vim values.yaml

replicas: 3
image: nginx
tag: 1.15
serviceport: 80
targetport: 80
label: nginx


# NOTES.txt
[root@k8s-master templates]# vim NOTES.txt

hello


# deployment.yaml
[root@k8s-master templates]# vim deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: {{ .Values.label }}
  name: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicas }}
  selector:
    matchLabels:
      app: {{ .Values.label }}
  template:
    metadata:
      labels:
        app: {{ .Values.label }}
    spec:
      containers:
      - image: {{ .Values.image }}:{{ .Values.tag }}
        name: web


# service.yaml
[root@k8s-master templates]# vim service.yaml

apiVersion: v1
kind: Service
metadata:
  labels:
    app: {{ .Values.label }}
  name: {{ .Release.Name }}
spec:
  ports:
  - port: {{ .Values.serviceport }}
    protocol: TCP
    targetPort: {{ .Values.targetport }}
  selector:
    app: {{ .Values.label }}
  type: NodePort
```



#### 2.2.安装

```shell
# 查看实际的模板被渲染过后的资源文件
# helm get manifest web
# helm install web nginx/

# 生成的最终的资源清单文件打印出来，而不会真正的去部署一个release实例
# helm install --name mychart --dry-run --debug -f global.yaml ./mychart/
# helm install --name mychart --dry-run --debug --set course="k8s" ./mychart/


# 调试
# Helm也提供了--dry-run --debug调试参数，帮助你验证模板正确性。在执行helm install时候带上这两个参数就可以把对应的values值和渲染的资源清单打印出来，而不会真正的去部署一个release。
[root@k8s-master test]# helm install web mynginx/ --dry-run --debug -n dev
install.go:178: [debug] Original chart version: ""
install.go:199: [debug] CHART PATH: /k8s/helm/test/mynginx

NAME: web
LAST DEPLOYED: Mon Oct 11 13:13:55 2021
NAMESPACE: default
STATUS: pending-install
REVISION: 1
TEST SUITE: None
USER-SUPPLIED VALUES:
{}

COMPUTED VALUES:
image: nginx
label: nginx
replicas: 3
serviceport: 80
tag: 1.15
targetport: 80

HOOKS:
MANIFEST:
---
# Source: mynginx/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: web
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: NodePort
---
# Source: mynginx/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.15
        name: web

NOTES:
hello


[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION


----------------------------------------------------------------------
# 安装
[root@k8s-master test]# helm install web mynginx/ -n dev
NAME: web
LAST DEPLOYED: Mon Oct 11 12:51:34 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
hello


[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                               	STATUS  	CHART        	APP VERSION
web 	dev      	1       	2021-10-11 13:16:37.74400198 +0800 CST	deployed	mynginx-0.1.0	1.15.0   


[root@k8s-master test]# helm status web -n dev
NAME: web
LAST DEPLOYED: Mon Oct 11 13:16:37 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
hello


# 检索已经发布的 release 的资源文件
[root@k8s-master test]# helm get manifest web -n dev
---
# Source: mynginx/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: web
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: NodePort
---
# Source: mynginx/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.15
        name: web


[root@k8s-master test]# helm history web -n dev
REVISION	UPDATED                 	STATUS  	CHART        	APP VERSION	DESCRIPTION     
1       	Mon Oct 11 13:16:37 2021	deployed	mynginx-0.1.0	1.15.0     	Install complete


[root@k8s-master test]# kubectl get all -n dev
NAME                       READY   STATUS    RESTARTS   AGE
pod/web-84b7cd6c7c-dhndc   1/1     Running   0          62s
pod/web-84b7cd6c7c-mnk4z   1/1     Running   0          62s
pod/web-84b7cd6c7c-pjbmg   1/1     Running   0          62s

NAME          TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/web   NodePort   10.103.41.189   <none>        80:31251/TCP   62s

NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/web   3/3     3            3           62s

NAME                             DESIRED   CURRENT   READY   AGE
replicaset.apps/web-84b7cd6c7c   3         3         3       62s


# 访问
http://172.51.216.81:31251/
curl 10.103.41.189


# 打包
# 修改Chart.yaml中的helm chart配置信息，然后使用下列命令将chart打包成一个压缩文件。
# 打包出mynginx-0.1.0.tgz文件。
[root@k8s-master test]# helm package mynginx
Successfully packaged chart and saved it to: /k8s/helm/test/mynginx-0.1.0.tgz
[root@k8s-master test]# ll
drwxr-xr-x 4 root root    93 Oct 11 13:12 mynginx
-rw-r--r-- 1 root root   869 Oct 11 13:43 mynginx-0.1.0.tgz
```



#### 2.3.打包并发布到本地仓库

```shell
[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                               	STATUS  	CHART        	APP VERSION
web 	dev      	1       	2021-10-11 13:16:37.74400198 +0800 CST	deployed	mynginx-0.1.0	1.15.0    

#从上面 helm list 输出的结果中我们可以看到有一个 Revision（更改历史）字段，该字段用于表示某一个 Release 被更新的次数，我们可以用该特性对已部署的 Release 进行回滚。
#修改 Chart.yaml 文件
#将版本号从 0.1.0 修改为 0.2.0, 然后使用 helm package 命令打包并发布到本地仓库。

# Chart.yaml
[root@k8s-master mynginx]# vim Chart.yaml
apiVersion: v2
name: mynginx
description: A Helm chart for Kubernetes
type: application
version: 0.2.0
appVersion: "1.15.0"


[root@k8s-master test]# helm package mynginx
Successfully packaged chart and saved it to: /k8s/helm/test/mynginx-0.2.0.tgz
[root@k8s-master test]# ll
drwxr-xr-x 4 root root    93 Oct 11 13:51 mynginx
-rw-r--r-- 1 root root   869 Oct 11 13:43 mynginx-0.1.0.tgz
-rw-r--r-- 1 root root   870 Oct 11 13:53 mynginx-0.2.0.tgz


# 查询本地仓库中的 Chart 信息
# 未测试通过
root@k8s-master test]# helm search mynginx -l
```



#### 2.4.升级

```shell
helm upgrade 命令将已部署的应用升级到新版本
发布新版本的chart时，或者当您要更改发布的配置时，可以使用该helm upgrade 命令。
# helm upgrade --set imageTag=1.17 web mynginx
# helm upgrade -f values.yaml web mynginx


values
Values对象是为Chart模板提供值，这个对象的值有4个来源：
chart 包中的 values.yaml 文件
父 chart 包的 values.yaml 文件
通过 helm install 或者 helm upgrade 的 -f或者 --values参数传入的自定义的 yaml 文件
通过 --set 参数传入的值
```

```shell

[root@k8s-master test]# helm upgrade web --set replicas=5 mynginx -n dev
Release "web" has been upgraded. Happy Helming!
NAME: web
LAST DEPLOYED: Mon Oct 11 14:12:25 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
hello

[root@k8s-master test]# kubectl get pod -n dev
NAME                   READY   STATUS              RESTARTS   AGE
web-84b7cd6c7c-dhndc   1/1     Running             0          56m
web-84b7cd6c7c-gzzqh   1/1     Running             0          30s
web-84b7cd6c7c-mnk4z   1/1     Running             0          56m
web-84b7cd6c7c-pjbmg   1/1     Running             0          56m
web-84b7cd6c7c-rlzpd   0/1     ContainerCreating   0          30s


[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                               	STATUS  	CHART        	APP VERSION
web 	dev      	2       	2021-10-11 14:12:25.81925385 +0800 CST	deployed	mynginx-0.2.0	1.15.0     

[root@k8s-master test]# helm history web -n dev
REVISION	UPDATED                 	STATUS    	CHART        	APP VERSION	DESCRIPTION     
1       	Mon Oct 11 13:16:37 2021	superseded	mynginx-0.1.0	1.15.0     	Install complete
2       	Mon Oct 11 14:12:25 2021	deployed  	mynginx-0.2.0	1.15.0     	Upgrade complete


[root@k8s-master test]# helm get manifest web -n dev
---
# Source: mynginx/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: web
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: NodePort
---
# Source: mynginx/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: web
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.15
        name: web
```



通过修改values.yaml升级

```shell
# 升级前
[root@k8s-master mynginx]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                               	STATUS  	CHART        	APP VERSION
web 	dev      	2       	2021-10-11 14:12:25.81925385 +0800 CST	deployed	mynginx-0.2.0	1.15.0     
[root@k8s-master mynginx]# helm status web -n dev
NAME: web
LAST DEPLOYED: Mon Oct 11 14:12:25 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
hello

[root@k8s-master mynginx]# helm history web -n dev
REVISION	UPDATED                 	STATUS    	CHART        	APP VERSION	DESCRIPTION     
1       	Mon Oct 11 13:16:37 2021	superseded	mynginx-0.1.0	1.15.0     	Install complete
2       	Mon Oct 11 14:12:25 2021	deployed  	mynginx-0.2.0	1.15.0     	Upgrade complete

[root@k8s-master test]# kubectl get pod -n dev
NAME                   READY   STATUS    RESTARTS   AGE
web-84b7cd6c7c-dhndc   1/1     Running   0          68m
web-84b7cd6c7c-gzzqh   1/1     Running   0          12m
web-84b7cd6c7c-mnk4z   1/1     Running   0          68m
web-84b7cd6c7c-pjbmg   1/1     Running   0          68m
web-84b7cd6c7c-rlzpd   1/1     Running   0          12m


# 修改文件
# 修改版本version: 0.3.0  tag: 1.16 

[root@k8s-master mynginx]# vim Chart.yaml
apiVersion: v2
name: mynginx
description: A Helm chart for Kubernetes
type: application
version: 0.3.0
appVersion: "1.16.0"


[root@k8s-master mynginx]# vim values.yaml
replicas: 3
image: nginx
tag: 1.16
serviceport: 80
targetport: 80
label: nginx


# 升级
[root@k8s-master test]# helm upgrade web mynginx/ -n dev
Release "web" has been upgraded. Happy Helming!
NAME: web
LAST DEPLOYED: Mon Oct 11 14:25:49 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 3
TEST SUITE: None
NOTES:
hello


[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART        	APP VERSION
web 	dev      	3       	2021-10-11 14:25:49.657012895 +0800 CST	deployed	mynginx-0.3.0	1.16.0     
[root@k8s-master test]# helm history web -n dev
REVISION	UPDATED                 	STATUS    	CHART        	APP VERSION	DESCRIPTION     
1       	Mon Oct 11 13:16:37 2021	superseded	mynginx-0.1.0	1.15.0     	Install complete
2       	Mon Oct 11 14:12:25 2021	superseded	mynginx-0.2.0	1.15.0     	Upgrade complete
3       	Mon Oct 11 14:25:49 2021	deployed  	mynginx-0.3.0	1.16.0     	Upgrade complete

[root@k8s-master test]# kubectl get pod -n dev
NAME                   READY   STATUS    RESTARTS   AGE
web-6fbb9b87dd-8wznf   1/1     Running   0          81s
web-6fbb9b87dd-k58gv   1/1     Running   0          30s
web-6fbb9b87dd-mgslf   1/1     Running   0          29s
web-6fbb9b87dd-spxvh   1/1     Running   0          81s
web-6fbb9b87dd-tgf84   1/1     Running   0          81s
[root@k8s-master test]# helm get manifest web -n dev
---
# Source: mynginx/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: web
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: NodePort
---
# Source: mynginx/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: web
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.16
        name: web
```



#### 2.5.回滚

```shell
# 如果在发布后没有达到预期的效果，则可以使用helm rollback回滚到之前的版本。
# 例如将应用回滚到第一个版本：
helm rollback web 1


# 回滚前
[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART        	APP VERSION
web 	dev      	3       	2021-10-11 14:25:49.657012895 +0800 CST	deployed	mynginx-0.3.0	1.16.0     
[root@k8s-master test]# helm status web -n dev
NAME: web
LAST DEPLOYED: Mon Oct 11 14:25:49 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 3
TEST SUITE: None
NOTES:
hello

[root@k8s-master test]# helm history web -n dev
REVISION	UPDATED                 	STATUS    	CHART        	APP VERSION	DESCRIPTION     
1       	Mon Oct 11 13:16:37 2021	superseded	mynginx-0.1.0	1.15.0     	Install complete
2       	Mon Oct 11 14:12:25 2021	superseded	mynginx-0.2.0	1.15.0     	Upgrade complete
3       	Mon Oct 11 14:25:49 2021	deployed  	mynginx-0.3.0	1.16.0     	Upgrade complete

[root@k8s-master test]# kubectl get pod -n dev
NAME                   READY   STATUS    RESTARTS   AGE
web-6fbb9b87dd-8wznf   1/1     Running   0          8m
web-6fbb9b87dd-k58gv   1/1     Running   0          7m9s
web-6fbb9b87dd-mgslf   1/1     Running   0          7m8s
web-6fbb9b87dd-spxvh   1/1     Running   0          8m
web-6fbb9b87dd-tgf84   1/1     Running   0          8m


# 回滚到版本1
[root@k8s-master test]# helm rollback web 1 -n dev
Rollback was a success! Happy Helming!


# 回滚后的状态
[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART        	APP VERSION
web 	dev      	4       	2021-10-11 14:35:43.553492797 +0800 CST	deployed	mynginx-0.1.0	1.15.0     
[root@k8s-master test]# helm status web -n dev
NAME: web
LAST DEPLOYED: Mon Oct 11 14:35:43 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 4
TEST SUITE: None
NOTES:
hello

[root@k8s-master test]# helm history web -n dev
REVISION	UPDATED                 	STATUS    	CHART        	APP VERSION	DESCRIPTION     
1       	Mon Oct 11 13:16:37 2021	superseded	mynginx-0.1.0	1.15.0     	Install complete
2       	Mon Oct 11 14:12:25 2021	superseded	mynginx-0.2.0	1.15.0     	Upgrade complete
3       	Mon Oct 11 14:25:49 2021	superseded	mynginx-0.3.0	1.16.0     	Upgrade complete
4       	Mon Oct 11 14:35:43 2021	deployed  	mynginx-0.1.0	1.15.0     	Rollback to 1 

[root@k8s-master test]# kubectl get pod -n dev
NAME                   READY   STATUS    RESTARTS   AGE
web-84b7cd6c7c-5hfzp   1/1     Running   0          95s
web-84b7cd6c7c-9snhp   1/1     Running   0          91s
web-84b7cd6c7c-z2fkg   1/1     Running   0          93s
```



#### 2.6.删除

```shell
卸载发行版，请使用以下helm uninstall命令：
# helm uninstall web -n dev
# helm delete web -n dev


[root@k8s-master test]# helm uninstall web -n dev
release "web" uninstalled

[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
```



**使用包安装**

```shell
# 使用 mynginx-0.2.0.tgz 安装

[root@k8s-master test]# ll
drwxr-xr-x 4 root root    93 Oct 11 14:28 mynginx
-rw-r--r-- 1 root root   869 Oct 11 13:43 mynginx-0.1.0.tgz
-rw-r--r-- 1 root root   870 Oct 11 13:53 mynginx-0.2.0.tgz


[root@k8s-master test]# helm install web mynginx-0.2.0.tgz -n dev
NAME: web
LAST DEPLOYED: Mon Oct 11 15:23:23 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
hello


[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART        	APP VERSION
web 	dev      	1       	2021-10-11 15:23:23.307351412 +0800 CST	deployed	mynginx-0.2.0	1.15.0     
[root@k8s-master test]# helm history web -n dev
REVISION	UPDATED                 	STATUS  	CHART        	APP VERSION	DESCRIPTION     
1       	Mon Oct 11 15:23:23 2021	deployed	mynginx-0.2.0	1.15.0     	Install complete

[root@k8s-master test]# helm get manifest web -n dev
---
# Source: mynginx/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: web
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: NodePort
---
# Source: mynginx/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.15
        name: web

[root@k8s-master test]# kubectl get all -n dev
NAME                       READY   STATUS    RESTARTS   AGE
pod/web-84b7cd6c7c-jfz8l   1/1     Running   0          4m4s
pod/web-84b7cd6c7c-tjtgx   1/1     Running   0          4m4s
pod/web-84b7cd6c7c-xr5hs   1/1     Running   0          4m4s

NAME          TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/web   NodePort   10.110.118.3   <none>        80:32692/TCP   4m4s

NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/web   3/3     3            3           4m4s

NAME                             DESIRED   CURRENT   READY   AGE
replicaset.apps/web-84b7cd6c7c   3         3         3       4m4s


# 删除
[root@k8s-master test]# helm delete web -n dev
release "web" uninstalled
[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
```





## 二、微服务



### 1.微服务配置

**1.注册中心**

 msa-eureka.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-eureka
  labels:
    app: msa-eureka
spec:
  type: NodePort
  ports:
    - port: 10001
      name: msa-eureka
      targetPort: 10001
      nodePort: 30001 #对外暴露30001端口
  selector:
    app: msa-eureka

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  namespace: dev
  name: msa-eureka
spec:
  serviceName: msa-eureka
  replicas: 3
  selector:
    matchLabels:
      app: msa-eureka
  template:
    metadata:
      labels:
        app: msa-eureka
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-eureka
          image: 172.51.216.85:8888/springcloud/msa-eureka:2.0.0
          ports:
          - containerPort: 10001
          env:
            - name: MY_POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: EUREKA_INSTANCE_HOSTNAME
              value: ${MY_POD_NAME}.msa-eureka.dev
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
  podManagementPolicy: "Parallel"
```



**2.网关**

msa-gateway.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-gateway
  labels:
    app: msa-gateway
spec:
  type: NodePort
  ports:
    - port: 8888
      targetPort: 8888
      nodePort: 30008 #对外暴露30008端口
  selector:
    app: msa-gateway

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-gateway
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-gateway
  template:
    metadata:
      labels:
        app: msa-gateway
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-gateway
          image: 172.51.216.85:8888/springcloud/msa-gateway:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8888
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
```



**3.生产者**

msa-deploy-producer.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-deploy-producer
  labels:
    app: msa-deploy-producer
spec:
  type: ClusterIP
  ports:
    - port: 8911
      targetPort: 8911
  selector:
    app: msa-deploy-producer

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-deploy-producer
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-deploy-producer
  template:
    metadata:
      labels:
        app: msa-deploy-producer
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-deploy-producer
          image: 172.51.216.85:8888/springcloud/msa-deploy-producer:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8911
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"
```



**4.消费者**

msa-deploy-consumer.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-deploy-consumer
  labels:
    app: msa-deploy-consumer
spec:
  type: ClusterIP
  ports:
    - port: 8912
      targetPort: 8912
  selector:
    app: msa-deploy-consumer

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-deploy-consumer
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-deploy-consumer
  template:
    metadata:
      labels:
        app: msa-deploy-consumer
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-deploy-consumer
          image: 172.51.216.85:8888/springcloud/msa-deploy-consumer:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8912
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"
```



### 2.部署微服务

**创建**

```yaml
# 创建 msa
[root@k8s-master test]# helm create cloud -n dev
Creating cloud


[root@k8s-master test]# ll
drwxr-xr-x 4 root root 93 Nov 11 13:50 cloud


[root@k8s-master test]# tree
.
└── cloud
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

4 directories, 10 files


--------------------------------------------
# templates删除所有文件
# templates创建文件：NOTES.txt、msa-eureka.yaml、msa-gateway.yaml、msa-deploy-producer.yaml、msa-deploy-consumer.yaml

[root@k8s-master test]# tree
.
└── cloud
    ├── charts
    ├── Chart.yaml
    ├── templates
    │   ├── msa-deploy-consumer.yaml
    │   ├── msa-deploy-producer.yaml
    │   ├── msa-eureka.yaml
    │   ├── msa-gateway.yaml
    │   └── NOTES.txt
    └── values.yaml

3 directories, 7 files
```

```yaml
#模板文件

# Chart.yaml
[root@k8s-master cloud]# vim Chart.yaml

apiVersion: v2
name: cloud
description: A Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: "1.0.0"


# values.yaml 暂时不用
[root@k8s-master cloud]# vim values.yaml


# NOTES.txt
[root@k8s-master cloud]# vim NOTES.txt

Spring Cloud!!!
```

**安装**

```shell
# 查看实际的模板被渲染过后的资源文件
# helm get manifest web
# helm install web nginx/

# 生成的最终的资源清单文件打印出来，而不会真正的去部署一个release实例
# helm install --name mychart --dry-run --debug -f global.yaml ./mychart/
# helm install --name mychart --dry-run --debug --set course="k8s" ./mychart/


# 调试
# Helm也提供了--dry-run --debug调试参数，帮助你验证模板正确性。在执行helm install时候带上这两个参数就可以把对应的values值和渲染的资源清单打印出来，而不会真正的去部署一个release。

[root@k8s-master test]# helm install msa cloud --dry-run --debug -n dev
install.go:173: [debug] Original chart version: ""
install.go:190: [debug] CHART PATH: /k8s/springcloud-helm/test/cloud

NAME: msa
LAST DEPLOYED: Thu Nov 11 14:22:13 2021
NAMESPACE: dev
STATUS: pending-install
REVISION: 1
TEST SUITE: None
USER-SUPPLIED VALUES:
{}

COMPUTED VALUES:
affinity: {}
autoscaling:
  enabled: false
  maxReplicas: 100
  minReplicas: 1
  targetCPUUtilizationPercentage: 80
fullnameOverride: ""
image:
  pullPolicy: IfNotPresent
  repository: nginx
  tag: ""
imagePullSecrets: []
ingress:
  annotations: {}
  className: ""
  enabled: false
  hosts:
  - host: chart-example.local
    paths:
    - path: /
      pathType: ImplementationSpecific
  tls: []
nameOverride: ""
nodeSelector: {}
podAnnotations: {}
podSecurityContext: {}
replicaCount: 1
resources: {}
securityContext: {}
service:
  port: 80
  type: ClusterIP
serviceAccount:
  annotations: {}
  create: true
  name: ""
tolerations: []

HOOKS:
MANIFEST:
---
# Source: cloud/templates/msa-deploy-consumer.yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-deploy-consumer
  labels:
    app: msa-deploy-consumer
spec:
  type: ClusterIP
  ports:
    - port: 8912
      targetPort: 8912
  selector:
    app: msa-deploy-consumer
---
# Source: cloud/templates/msa-deploy-producer.yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-deploy-producer
  labels:
    app: msa-deploy-producer
spec:
  type: ClusterIP
  ports:
    - port: 8911
      targetPort: 8911
  selector:
    app: msa-deploy-producer
---
# Source: cloud/templates/msa-eureka.yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-eureka
  labels:
    app: msa-eureka
spec:
  type: NodePort
  ports:
    - port: 10001
      name: msa-eureka
      targetPort: 10001
      nodePort: 30001 #对外暴露30001端口
  selector:
    app: msa-eureka
---
# Source: cloud/templates/msa-gateway.yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-gateway
  labels:
    app: msa-gateway
spec:
  type: NodePort
  ports:
    - port: 8888
      targetPort: 8888
      nodePort: 30008 #对外暴露30008端口
  selector:
    app: msa-gateway
---
# Source: cloud/templates/msa-deploy-consumer.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-deploy-consumer
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-deploy-consumer
  template:
    metadata:
      labels:
        app: msa-deploy-consumer
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-deploy-consumer
          image: 172.51.216.85:8888/springcloud/msa-deploy-consumer:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8912
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"
---
# Source: cloud/templates/msa-deploy-producer.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-deploy-producer
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-deploy-producer
  template:
    metadata:
      labels:
        app: msa-deploy-producer
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-deploy-producer
          image: 172.51.216.85:8888/springcloud/msa-deploy-producer:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8911
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"
---
# Source: cloud/templates/msa-gateway.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-gateway
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-gateway
  template:
    metadata:
      labels:
        app: msa-gateway
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-gateway
          image: 172.51.216.85:8888/springcloud/msa-gateway:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8888
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
---
# Source: cloud/templates/msa-eureka.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  namespace: dev
  name: msa-eureka
spec:
  serviceName: msa-eureka
  replicas: 3
  selector:
    matchLabels:
      app: msa-eureka
  template:
    metadata:
      labels:
        app: msa-eureka
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-eureka
          image: 172.51.216.85:8888/springcloud/msa-eureka:2.0.0
          ports:
          - containerPort: 10001
          env:
            - name: MY_POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: EUREKA_INSTANCE_HOSTNAME
              value: ${MY_POD_NAME}.msa-eureka.dev
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
  podManagementPolicy: "Parallel"

NOTES:
Spring Cloud!!!



[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION


----------------------------------------------------------------------
# 安装
[root@k8s-master test]# helm install msa cloud -n dev
NAME: msa
LAST DEPLOYED: Thu Nov 11 14:25:55 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Spring Cloud!!!



[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	dev      	1       	2021-11-11 14:25:55.000327298 +0800 CST	deployed	cloud-0.1.0	1.0.0   


[root@k8s-master test]# helm status msa -n dev
NAME: msa
LAST DEPLOYED: Thu Nov 11 14:25:55 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Spring Cloud!!!


# 检索已经发布的 release 的资源文件
[root@k8s-master test]# helm get manifest msa -n dev
---
# Source: cloud/templates/msa-deploy-consumer.yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-deploy-consumer
  labels:
    app: msa-deploy-consumer
spec:
  type: ClusterIP
  ports:
    - port: 8912
      targetPort: 8912
  selector:
    app: msa-deploy-consumer
---
# Source: cloud/templates/msa-deploy-producer.yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-deploy-producer
  labels:
    app: msa-deploy-producer
spec:
  type: ClusterIP
  ports:
    - port: 8911
      targetPort: 8911
  selector:
    app: msa-deploy-producer
---
# Source: cloud/templates/msa-eureka.yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-eureka
  labels:
    app: msa-eureka
spec:
  type: NodePort
  ports:
    - port: 10001
      name: msa-eureka
      targetPort: 10001
      nodePort: 30001 #对外暴露30001端口
  selector:
    app: msa-eureka
---
# Source: cloud/templates/msa-gateway.yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-gateway
  labels:
    app: msa-gateway
spec:
  type: NodePort
  ports:
    - port: 8888
      targetPort: 8888
      nodePort: 30008 #对外暴露30008端口
  selector:
    app: msa-gateway
---
# Source: cloud/templates/msa-deploy-consumer.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-deploy-consumer
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-deploy-consumer
  template:
    metadata:
      labels:
        app: msa-deploy-consumer
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-deploy-consumer
          image: 172.51.216.85:8888/springcloud/msa-deploy-consumer:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8912
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"
---
# Source: cloud/templates/msa-deploy-producer.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-deploy-producer
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-deploy-producer
  template:
    metadata:
      labels:
        app: msa-deploy-producer
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-deploy-producer
          image: 172.51.216.85:8888/springcloud/msa-deploy-producer:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8911
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"
---
# Source: cloud/templates/msa-gateway.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-gateway
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-gateway
  template:
    metadata:
      labels:
        app: msa-gateway
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-gateway
          image: 172.51.216.85:8888/springcloud/msa-gateway:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8888
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
---
# Source: cloud/templates/msa-eureka.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  namespace: dev
  name: msa-eureka
spec:
  serviceName: msa-eureka
  replicas: 3
  selector:
    matchLabels:
      app: msa-eureka
  template:
    metadata:
      labels:
        app: msa-eureka
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-eureka
          image: 172.51.216.85:8888/springcloud/msa-eureka:2.0.0
          ports:
          - containerPort: 10001
          env:
            - name: MY_POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: EUREKA_INSTANCE_HOSTNAME
              value: ${MY_POD_NAME}.msa-eureka.dev
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
  podManagementPolicy: "Parallel"


# 打包
# 修改Chart.yaml中的helm chart配置信息，然后使用下列命令将chart打包成一个压缩文件。
# 打包出cloud-0.1.0.tgz文件。
[root@k8s-master test]# helm package cloud
Successfully packaged chart and saved it to: /k8s/springcloud-helm/test/cloud-0.1.0.tgz
[root@k8s-master test]# ll
total 4
drwxr-xr-x 4 root root   93 Nov 11 14:19 cloud
-rw-r--r-- 1 root root 2361 Nov 11 14:30 cloud-0.1.0.tgz
```

```shell
# 查看
[root@k8s-master test]# kubectl get all -n dev
NAME                                       READY   STATUS    RESTARTS   AGE
pod/msa-deploy-consumer-6b75cf55d-qg5rh    1/1     Running   0          6m2s
pod/msa-deploy-consumer-6b75cf55d-zmd5x    1/1     Running   0          6m2s
pod/msa-deploy-consumer-6b75cf55d-zqfpp    1/1     Running   0          6m2s
pod/msa-deploy-producer-7965c98bbf-8gqh2   1/1     Running   0          6m2s
pod/msa-deploy-producer-7965c98bbf-hfhb6   1/1     Running   0          6m2s
pod/msa-deploy-producer-7965c98bbf-ngmp4   1/1     Running   0          6m2s
pod/msa-eureka-0                           1/1     Running   0          6m2s
pod/msa-eureka-1                           1/1     Running   0          6m2s
pod/msa-eureka-2                           1/1     Running   0          6m2s
pod/msa-gateway-597494c7f4-9rkxf           1/1     Running   0          6m2s
pod/msa-gateway-597494c7f4-wbjhl           1/1     Running   0          6m2s
pod/msa-gateway-597494c7f4-wl2bj           1/1     Running   0          6m2s

NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
service/msa-deploy-consumer   ClusterIP   10.103.23.244    <none>        8912/TCP          6m2s
service/msa-deploy-producer   ClusterIP   10.109.200.26    <none>        8911/TCP          6m2s
service/msa-eureka            NodePort    10.109.68.183    <none>        10001:30001/TCP   6m2s
service/msa-gateway           NodePort    10.102.204.100   <none>        8888:30008/TCP    6m2s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/msa-deploy-consumer   3/3     3            3           6m2s
deployment.apps/msa-deploy-producer   3/3     3            3           6m2s
deployment.apps/msa-gateway           3/3     3            3           6m2s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/msa-deploy-consumer-6b75cf55d    3         3         3       6m2s
replicaset.apps/msa-deploy-producer-7965c98bbf   3         3         3       6m2s
replicaset.apps/msa-gateway-597494c7f4           3         3         3       6m2s

NAME                          READY   AGE
statefulset.apps/msa-eureka   3/3     6m2s


# 访问
# Eureka
http://172.51.216.81:30001/

# 网关访问地址
http://172.51.216.81:30008/dconsumer/hello

http://172.51.216.81:30008/dconsumer/say

http://172.51.216.81:30008/dproducer/hello

http://172.51.216.81:30008/dproducer/say
```



### 3.打包并发布到本地仓库

```shell
[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	dev      	1       	2021-11-11 14:25:55.000327298 +0800 CST	deployed	cloud-0.1.0	1.0.0     


#从上面 helm list 输出的结果中我们可以看到有一个 Revision（更改历史）字段，该字段用于表示某一个 Release 被更新的次数，我们可以用该特性对已部署的 Release 进行回滚。
#修改 Chart.yaml 文件
#将版本号从 0.1.0 修改为 0.2.0, 然后使用 helm package 命令打包并发布到本地仓库。

# Chart.yaml
[root@k8s-master cloud]# vim Chart.yaml

apiVersion: v2
name: cloud
description: A Helm chart for Kubernetes
type: application
version: 0.2.0
appVersion: "1.0.0"


[root@k8s-master test]# helm package cloud
Successfully packaged chart and saved it to: /k8s/springcloud-helm/test/cloud-0.2.0.tgz
[root@k8s-master test]# ll
total 8
drwxr-xr-x 4 root root   93 Nov 11 14:44 cloud
-rw-r--r-- 1 root root 2361 Nov 11 14:30 cloud-0.1.0.tgz
-rw-r--r-- 1 root root 2361 Nov 11 14:45 cloud-0.2.0.tgz
```



### 4.升级

```shell
helm upgrade 命令将已部署的应用升级到新版本
发布新版本的chart时，或者当您要更改发布的配置时，可以使用该helm upgrade 命令。
# helm upgrade --set imageTag=1.17 web mynginx
# helm upgrade -f values.yaml web mynginx


values
Values对象是为Chart模板提供值，这个对象的值有4个来源：
chart 包中的 values.yaml 文件
父 chart 包的 values.yaml 文件
通过 helm install 或者 helm upgrade 的 -f或者 --values参数传入的自定义的 yaml 文件
通过 --set 参数传入的值
```

```shell
[root@k8s-master test]# kubectl get all -n dev
NAME                                       READY   STATUS    RESTARTS   AGE
pod/msa-deploy-consumer-6b75cf55d-qg5rh    1/1     Running   0          26m
pod/msa-deploy-consumer-6b75cf55d-zmd5x    1/1     Running   0          26m
pod/msa-deploy-consumer-6b75cf55d-zqfpp    1/1     Running   0          26m
pod/msa-deploy-producer-7965c98bbf-8gqh2   1/1     Running   0          26m
pod/msa-deploy-producer-7965c98bbf-hfhb6   1/1     Running   0          26m
pod/msa-deploy-producer-7965c98bbf-ngmp4   1/1     Running   0          26m
pod/msa-eureka-0                           1/1     Running   0          26m
pod/msa-eureka-1                           1/1     Running   0          26m
pod/msa-eureka-2                           1/1     Running   0          26m
pod/msa-gateway-597494c7f4-9rkxf           1/1     Running   0          26m
pod/msa-gateway-597494c7f4-wbjhl           1/1     Running   0          26m
pod/msa-gateway-597494c7f4-wl2bj           1/1     Running   0          26m

NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
service/msa-deploy-consumer   ClusterIP   10.103.23.244    <none>        8912/TCP          26m
service/msa-deploy-producer   ClusterIP   10.109.200.26    <none>        8911/TCP          26m
service/msa-eureka            NodePort    10.109.68.183    <none>        10001:30001/TCP   26m
service/msa-gateway           NodePort    10.102.204.100   <none>        8888:30008/TCP    26m

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/msa-deploy-consumer   3/3     3            3           26m
deployment.apps/msa-deploy-producer   3/3     3            3           26m
deployment.apps/msa-gateway           3/3     3            3           26m

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/msa-deploy-consumer-6b75cf55d    3         3         3       26m
replicaset.apps/msa-deploy-producer-7965c98bbf   3         3         3       26m
replicaset.apps/msa-gateway-597494c7f4           3         3         3       26m

NAME                          READY   AGE
statefulset.apps/msa-eureka   3/3     26m


# 此方法改变不了副本数
[root@k8s-master test]# helm upgrade msa --set replicas=5 cloud -n dev
Release "msa" has been upgraded. Happy Helming!
NAME: msa
LAST DEPLOYED: Thu Nov 11 14:54:21 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
Spring Cloud!!!


# 没变化
[root@k8s-master test]# kubectl get all -n dev
NAME                                       READY   STATUS    RESTARTS   AGE
pod/msa-deploy-consumer-6b75cf55d-qg5rh    1/1     Running   0          30m
pod/msa-deploy-consumer-6b75cf55d-zmd5x    1/1     Running   0          30m
pod/msa-deploy-consumer-6b75cf55d-zqfpp    1/1     Running   0          30m
pod/msa-deploy-producer-7965c98bbf-8gqh2   1/1     Running   0          30m
pod/msa-deploy-producer-7965c98bbf-hfhb6   1/1     Running   0          30m
pod/msa-deploy-producer-7965c98bbf-ngmp4   1/1     Running   0          30m
pod/msa-eureka-0                           1/1     Running   0          30m
pod/msa-eureka-1                           1/1     Running   0          30m
pod/msa-eureka-2                           1/1     Running   0          30m
pod/msa-gateway-597494c7f4-9rkxf           1/1     Running   0          30m
pod/msa-gateway-597494c7f4-wbjhl           1/1     Running   0          30m
pod/msa-gateway-597494c7f4-wl2bj           1/1     Running   0          30m

NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
service/msa-deploy-consumer   ClusterIP   10.103.23.244    <none>        8912/TCP          30m
service/msa-deploy-producer   ClusterIP   10.109.200.26    <none>        8911/TCP          30m
service/msa-eureka            NodePort    10.109.68.183    <none>        10001:30001/TCP   30m
service/msa-gateway           NodePort    10.102.204.100   <none>        8888:30008/TCP    30m

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/msa-deploy-consumer   3/3     3            3           30m
deployment.apps/msa-deploy-producer   3/3     3            3           30m
deployment.apps/msa-gateway           3/3     3            3           30m

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/msa-deploy-consumer-6b75cf55d    3         3         3       30m
replicaset.apps/msa-deploy-producer-7965c98bbf   3         3         3       30m
replicaset.apps/msa-gateway-597494c7f4           3         3         3       30m

NAME                          READY   AGE
statefulset.apps/msa-eureka   3/3     30m



[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	dev      	2       	2021-11-11 14:54:21.874398457 +0800 CST	deployed	cloud-0.2.0	1.0.0      


[root@k8s-master test]# helm history msa -n dev
REVISION	UPDATED                 	STATUS    	CHART      	APP VERSION	DESCRIPTION     
1       	Thu Nov 11 14:25:55 2021	superseded	cloud-0.1.0	1.0.0      	Install complete
2       	Thu Nov 11 14:54:21 2021	deployed  	cloud-0.2.0	1.0.0      	Upgrade complete



# 没变化
[root@k8s-master test]# helm get manifest msa -n dev
---
# Source: cloud/templates/msa-deploy-consumer.yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-deploy-consumer
  labels:
    app: msa-deploy-consumer
spec:
  type: ClusterIP
  ports:
    - port: 8912
      targetPort: 8912
  selector:
    app: msa-deploy-consumer
---
# Source: cloud/templates/msa-deploy-producer.yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-deploy-producer
  labels:
    app: msa-deploy-producer
spec:
  type: ClusterIP
  ports:
    - port: 8911
      targetPort: 8911
  selector:
    app: msa-deploy-producer
---
# Source: cloud/templates/msa-eureka.yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-eureka
  labels:
    app: msa-eureka
spec:
  type: NodePort
  ports:
    - port: 10001
      name: msa-eureka
      targetPort: 10001
      nodePort: 30001 #对外暴露30001端口
  selector:
    app: msa-eureka
---
# Source: cloud/templates/msa-gateway.yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-gateway
  labels:
    app: msa-gateway
spec:
  type: NodePort
  ports:
    - port: 8888
      targetPort: 8888
      nodePort: 30008 #对外暴露30008端口
  selector:
    app: msa-gateway
---
# Source: cloud/templates/msa-deploy-consumer.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-deploy-consumer
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-deploy-consumer
  template:
    metadata:
      labels:
        app: msa-deploy-consumer
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-deploy-consumer
          image: 172.51.216.85:8888/springcloud/msa-deploy-consumer:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8912
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"
---
# Source: cloud/templates/msa-deploy-producer.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-deploy-producer
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-deploy-producer
  template:
    metadata:
      labels:
        app: msa-deploy-producer
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-deploy-producer
          image: 172.51.216.85:8888/springcloud/msa-deploy-producer:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8911
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"
---
# Source: cloud/templates/msa-gateway.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-gateway
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-gateway
  template:
    metadata:
      labels:
        app: msa-gateway
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-gateway
          image: 172.51.216.85:8888/springcloud/msa-gateway:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8888
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
---
# Source: cloud/templates/msa-eureka.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  namespace: dev
  name: msa-eureka
spec:
  serviceName: msa-eureka
  replicas: 3
  selector:
    matchLabels:
      app: msa-eureka
  template:
    metadata:
      labels:
        app: msa-eureka
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-eureka
          image: 172.51.216.85:8888/springcloud/msa-eureka:2.0.0
          ports:
          - containerPort: 10001
          env:
            - name: MY_POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: EUREKA_INSTANCE_HOSTNAME
              value: ${MY_POD_NAME}.msa-eureka.dev
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
  podManagementPolicy: "Parallel"
  

# 又做了一次副本没变，版本变了
[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	dev      	3       	2021-11-11 15:03:34.282943143 +0800 CST	deployed	cloud-0.2.0	1.0.0      
[root@k8s-master test]# 
[root@k8s-master test]# 
[root@k8s-master test]# helm history msa -n dev
REVISION	UPDATED                 	STATUS    	CHART      	APP VERSION	DESCRIPTION     
1       	Thu Nov 11 14:25:55 2021	superseded	cloud-0.1.0	1.0.0      	Install complete
2       	Thu Nov 11 14:54:21 2021	superseded	cloud-0.2.0	1.0.0      	Upgrade complete
3       	Thu Nov 11 15:03:34 2021	deployed  	cloud-0.2.0	1.0.0      	Upgrade complete
```



通过修改模板yaml升级

```shell
# 升级前
[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	dev      	3       	2021-11-11 15:03:34.282943143 +0800 CST	deployed	cloud-0.2.0	1.0.0     


[root@k8s-master test]# helm status msa -n dev
NAME: msa
LAST DEPLOYED: Thu Nov 11 15:03:34 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 3
TEST SUITE: None
NOTES:
Spring Cloud!!!



[root@k8s-master test]# helm history msa -n dev
REVISION	UPDATED                 	STATUS    	CHART      	APP VERSION	DESCRIPTION     
1       	Thu Nov 11 14:25:55 2021	superseded	cloud-0.1.0	1.0.0      	Install complete
2       	Thu Nov 11 14:54:21 2021	superseded	cloud-0.2.0	1.0.0      	Upgrade complete
3       	Thu Nov 11 15:03:34 2021	deployed  	cloud-0.2.0	1.0.0      	Upgrade complete



[root@k8s-master test]# kubectl get all -n dev
NAME                                       READY   STATUS    RESTARTS   AGE
pod/msa-deploy-consumer-6b75cf55d-qg5rh    1/1     Running   0          46m
pod/msa-deploy-consumer-6b75cf55d-zmd5x    1/1     Running   0          46m
pod/msa-deploy-consumer-6b75cf55d-zqfpp    1/1     Running   0          46m
pod/msa-deploy-producer-7965c98bbf-8gqh2   1/1     Running   0          46m
pod/msa-deploy-producer-7965c98bbf-hfhb6   1/1     Running   0          46m
pod/msa-deploy-producer-7965c98bbf-ngmp4   1/1     Running   0          46m
pod/msa-eureka-0                           1/1     Running   0          46m
pod/msa-eureka-1                           1/1     Running   0          46m
pod/msa-eureka-2                           1/1     Running   0          46m
pod/msa-gateway-597494c7f4-9rkxf           1/1     Running   0          46m
pod/msa-gateway-597494c7f4-wbjhl           1/1     Running   0          46m
pod/msa-gateway-597494c7f4-wl2bj           1/1     Running   0          46m

NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
service/msa-deploy-consumer   ClusterIP   10.103.23.244    <none>        8912/TCP          46m
service/msa-deploy-producer   ClusterIP   10.109.200.26    <none>        8911/TCP          46m
service/msa-eureka            NodePort    10.109.68.183    <none>        10001:30001/TCP   46m
service/msa-gateway           NodePort    10.102.204.100   <none>        8888:30008/TCP    46m

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/msa-deploy-consumer   3/3     3            3           46m
deployment.apps/msa-deploy-producer   3/3     3            3           46m
deployment.apps/msa-gateway           3/3     3            3           46m

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/msa-deploy-consumer-6b75cf55d    3         3         3       46m
replicaset.apps/msa-deploy-producer-7965c98bbf   3         3         3       46m
replicaset.apps/msa-gateway-597494c7f4           3         3         3       46m

NAME                          READY   AGE
statefulset.apps/msa-eureka   3/3     46m



# 修改文件
# 修改版本version: 0.3.0  tag: 1.16 
[root@k8s-master cloud]# vim Chart.yaml 

apiVersion: v2
name: cloud
description: A Helm chart for Kubernetes
type: application
version: 0.3.0
appVersion: "1.0.1"


# 生产者副本数从3改成6
[root@k8s-master cloud]# vim templates/msa-deploy-producer.yaml
spec:
  replicas: 6


# 升级
[root@k8s-master test]# helm upgrade msa cloud -n dev
Release "msa" has been upgraded. Happy Helming!
NAME: msa
LAST DEPLOYED: Thu Nov 11 15:19:27 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 4
TEST SUITE: None
NOTES:
Spring Cloud!!!



[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	dev      	4       	2021-11-11 15:19:27.147696331 +0800 CST	deployed	cloud-0.3.0	1.0.1  



[root@k8s-master test]# helm history msa -n dev
REVISION	UPDATED                 	STATUS    	CHART      	APP VERSION	DESCRIPTION     
1       	Thu Nov 11 14:25:55 2021	superseded	cloud-0.1.0	1.0.0      	Install complete
2       	Thu Nov 11 14:54:21 2021	superseded	cloud-0.2.0	1.0.0      	Upgrade complete
3       	Thu Nov 11 15:03:34 2021	superseded	cloud-0.2.0	1.0.0      	Upgrade complete
4       	Thu Nov 11 15:19:27 2021	deployed  	cloud-0.3.0	1.0.1      	Upgrade complete



[root@k8s-master test]# kubectl get all -n dev
NAME                                       READY   STATUS    RESTARTS   AGE
pod/msa-deploy-consumer-6b75cf55d-qg5rh    1/1     Running   0          55m
pod/msa-deploy-consumer-6b75cf55d-zmd5x    1/1     Running   0          55m
pod/msa-deploy-consumer-6b75cf55d-zqfpp    1/1     Running   0          55m
pod/msa-deploy-producer-7965c98bbf-4cdvz   1/1     Running   0          2m13s
pod/msa-deploy-producer-7965c98bbf-8gqh2   1/1     Running   0          55m
pod/msa-deploy-producer-7965c98bbf-hfhb6   1/1     Running   0          55m
pod/msa-deploy-producer-7965c98bbf-k5ppd   1/1     Running   0          2m13s
pod/msa-deploy-producer-7965c98bbf-ngmp4   1/1     Running   0          55m
pod/msa-deploy-producer-7965c98bbf-s7mfs   1/1     Running   0          2m13s
pod/msa-eureka-0                           1/1     Running   0          55m
pod/msa-eureka-1                           1/1     Running   0          55m
pod/msa-eureka-2                           1/1     Running   0          55m
pod/msa-gateway-597494c7f4-9rkxf           1/1     Running   0          55m
pod/msa-gateway-597494c7f4-wbjhl           1/1     Running   0          55m
pod/msa-gateway-597494c7f4-wl2bj           1/1     Running   0          55m

NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
service/msa-deploy-consumer   ClusterIP   10.103.23.244    <none>        8912/TCP          55m
service/msa-deploy-producer   ClusterIP   10.109.200.26    <none>        8911/TCP          55m
service/msa-eureka            NodePort    10.109.68.183    <none>        10001:30001/TCP   55m
service/msa-gateway           NodePort    10.102.204.100   <none>        8888:30008/TCP    55m

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/msa-deploy-consumer   3/3     3            3           55m
deployment.apps/msa-deploy-producer   6/6     6            6           55m
deployment.apps/msa-gateway           3/3     3            3           55m

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/msa-deploy-consumer-6b75cf55d    3         3         3       55m
replicaset.apps/msa-deploy-producer-7965c98bbf   6         6         6       55m
replicaset.apps/msa-gateway-597494c7f4           3         3         3       55m

NAME                          READY   AGE
statefulset.apps/msa-eureka   3/3     55m


[root@k8s-master test]# helm get manifest msa -n dev
---
# Source: cloud/templates/msa-deploy-consumer.yaml
.....

---
# Source: cloud/templates/msa-deploy-producer.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-deploy-producer
spec:
  replicas: 6
  selector:
    matchLabels:
      app: msa-deploy-producer
  template:
    metadata:
      labels:
        app: msa-deploy-producer
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-deploy-producer
          image: 172.51.216.85:8888/springcloud/msa-deploy-producer:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8911
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"
---
......
```



###  5.回滚

```shell
# 如果在发布后没有达到预期的效果，则可以使用helm rollback回滚到之前的版本。
# 例如将应用回滚到第一个版本：
helm rollback web 1


# 回滚前
[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                               	STATUS  	CHART      	APP VERSION
msa 	dev      	5       	2021-11-11 15:28:30.73309279 +0800 CST	deployed	cloud-0.3.0	1.0.1   



[root@k8s-master test]# helm status msa -n dev
NAME: msa
LAST DEPLOYED: Thu Nov 11 15:28:30 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 5
TEST SUITE: None
NOTES:
Spring Cloud!!!


[root@k8s-master test]# helm history msa -n dev
REVISION	UPDATED                 	STATUS    	CHART      	APP VERSION	DESCRIPTION     
1       	Thu Nov 11 14:25:55 2021	superseded	cloud-0.1.0	1.0.0      	Install complete
2       	Thu Nov 11 14:54:21 2021	superseded	cloud-0.2.0	1.0.0      	Upgrade complete
3       	Thu Nov 11 15:03:34 2021	superseded	cloud-0.2.0	1.0.0      	Upgrade complete
4       	Thu Nov 11 15:19:27 2021	superseded	cloud-0.3.0	1.0.1      	Upgrade complete
5       	Thu Nov 11 15:28:30 2021	deployed  	cloud-0.3.0	1.0.1      	Upgrade complete


[root@k8s-master test]# kubectl get pod -n dev
NAME                                   READY   STATUS    RESTARTS   AGE
msa-deploy-consumer-6b75cf55d-qg5rh    1/1     Running   0          65m
msa-deploy-consumer-6b75cf55d-zmd5x    1/1     Running   0          65m
msa-deploy-consumer-6b75cf55d-zqfpp    1/1     Running   0          65m
msa-deploy-producer-7965c98bbf-4cdvz   1/1     Running   0          11m
msa-deploy-producer-7965c98bbf-8gqh2   1/1     Running   0          65m
msa-deploy-producer-7965c98bbf-hfhb6   1/1     Running   0          65m
msa-deploy-producer-7965c98bbf-k5ppd   1/1     Running   0          11m
msa-deploy-producer-7965c98bbf-ngmp4   1/1     Running   0          65m
msa-deploy-producer-7965c98bbf-s7mfs   1/1     Running   0          11m
msa-eureka-0                           1/1     Running   0          65m
msa-eureka-1                           1/1     Running   0          65m
msa-eureka-2                           1/1     Running   0          65m
msa-gateway-597494c7f4-9rkxf           1/1     Running   0          65m
msa-gateway-597494c7f4-wbjhl           1/1     Running   0          65m
msa-gateway-597494c7f4-wl2bj           1/1     Running   0          65m


# 回滚到版本1
[root@k8s-master test]# helm rollback msa 1 -n dev
Rollback was a success! Happy Helming!


# 回滚后的状态
[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	dev      	6       	2021-11-11 15:32:24.965626915 +0800 CST	deployed	cloud-0.1.0	1.0.0  


[root@k8s-master test]# helm status msa -n dev
NAME: msa
LAST DEPLOYED: Thu Nov 11 15:32:24 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 6
TEST SUITE: None
NOTES:
Spring Cloud!!!


[root@k8s-master test]# helm history msa -n dev
REVISION	UPDATED                 	STATUS    	CHART      	APP VERSION	DESCRIPTION     
1       	Thu Nov 11 14:25:55 2021	superseded	cloud-0.1.0	1.0.0      	Install complete
2       	Thu Nov 11 14:54:21 2021	superseded	cloud-0.2.0	1.0.0      	Upgrade complete
3       	Thu Nov 11 15:03:34 2021	superseded	cloud-0.2.0	1.0.0      	Upgrade complete
4       	Thu Nov 11 15:19:27 2021	superseded	cloud-0.3.0	1.0.1      	Upgrade complete
5       	Thu Nov 11 15:28:30 2021	superseded	cloud-0.3.0	1.0.1      	Upgrade complete
6       	Thu Nov 11 15:32:24 2021	deployed  	cloud-0.1.0	1.0.0      	Rollback to 1  


[root@k8s-master test]# kubectl get pod -n dev
NAME                                   READY   STATUS    RESTARTS   AGE
msa-deploy-consumer-6b75cf55d-qg5rh    1/1     Running   0          68m
msa-deploy-consumer-6b75cf55d-zmd5x    1/1     Running   0          68m
msa-deploy-consumer-6b75cf55d-zqfpp    1/1     Running   0          68m
msa-deploy-producer-7965c98bbf-8gqh2   1/1     Running   0          68m
msa-deploy-producer-7965c98bbf-hfhb6   1/1     Running   0          68m
msa-deploy-producer-7965c98bbf-ngmp4   1/1     Running   0          68m
msa-eureka-0                           1/1     Running   0          68m
msa-eureka-1                           1/1     Running   0          68m
msa-eureka-2                           1/1     Running   0          68m
msa-gateway-597494c7f4-9rkxf           1/1     Running   0          68m
msa-gateway-597494c7f4-wbjhl           1/1     Running   0          68m
msa-gateway-597494c7f4-wl2bj           1/1     Running   0          68m
```



### 6.删除

```shell
卸载发行版，请使用以下helm uninstall命令：
# helm uninstall web -n dev
# helm delete web -n dev


[root@k8s-master test]# helm uninstall msa -n dev
release "msa" uninstalled



[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION



[root@k8s-master test]# helm history msa -n dev
Error: release: not found
```



### 7.使用包安装

```shell
# 使用 cloud-0.2.0.tgz 安装

[root@k8s-master test]# ll
total 8
drwxr-xr-x 4 root root   93 Nov 11 15:16 cloud
-rw-r--r-- 1 root root 2361 Nov 11 14:30 cloud-0.1.0.tgz
-rw-r--r-- 1 root root 2361 Nov 11 14:45 cloud-0.2.0.tgz


[root@k8s-master test]# helm install msa cloud-0.2.0.tgz -n dev
NAME: msa
LAST DEPLOYED: Thu Nov 11 15:38:30 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Spring Cloud!!!


[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	dev      	1       	2021-11-11 15:38:30.699684217 +0800 CST	deployed	cloud-0.2.0	1.0.0   


[root@k8s-master test]#  helm get manifest msa -n dev
---
# Source: cloud/templates/msa-deploy-consumer.yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-deploy-consumer
  labels:
    app: msa-deploy-consumer
spec:
  type: ClusterIP
  ports:
    - port: 8912
      targetPort: 8912
  selector:
    app: msa-deploy-consumer
---
# Source: cloud/templates/msa-deploy-producer.yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-deploy-producer
  labels:
    app: msa-deploy-producer
spec:
  type: ClusterIP
  ports:
    - port: 8911
      targetPort: 8911
  selector:
    app: msa-deploy-producer
---
# Source: cloud/templates/msa-eureka.yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-eureka
  labels:
    app: msa-eureka
spec:
  type: NodePort
  ports:
    - port: 10001
      name: msa-eureka
      targetPort: 10001
      nodePort: 30001 #对外暴露30001端口
  selector:
    app: msa-eureka
---
# Source: cloud/templates/msa-gateway.yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-gateway
  labels:
    app: msa-gateway
spec:
  type: NodePort
  ports:
    - port: 8888
      targetPort: 8888
      nodePort: 30008 #对外暴露30008端口
  selector:
    app: msa-gateway
---
# Source: cloud/templates/msa-deploy-consumer.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-deploy-consumer
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-deploy-consumer
  template:
    metadata:
      labels:
        app: msa-deploy-consumer
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-deploy-consumer
          image: 172.51.216.85:8888/springcloud/msa-deploy-consumer:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8912
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"
---
# Source: cloud/templates/msa-deploy-producer.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-deploy-producer
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-deploy-producer
  template:
    metadata:
      labels:
        app: msa-deploy-producer
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-deploy-producer
          image: 172.51.216.85:8888/springcloud/msa-deploy-producer:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8911
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"
---
# Source: cloud/templates/msa-gateway.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-gateway
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-gateway
  template:
    metadata:
      labels:
        app: msa-gateway
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-gateway
          image: 172.51.216.85:8888/springcloud/msa-gateway:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8888
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
---
# Source: cloud/templates/msa-eureka.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  namespace: dev
  name: msa-eureka
spec:
  serviceName: msa-eureka
  replicas: 3
  selector:
    matchLabels:
      app: msa-eureka
  template:
    metadata:
      labels:
        app: msa-eureka
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-eureka
          image: 172.51.216.85:8888/springcloud/msa-eureka:2.0.0
          ports:
          - containerPort: 10001
          env:
            - name: MY_POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: EUREKA_INSTANCE_HOSTNAME
              value: ${MY_POD_NAME}.msa-eureka.dev
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
  podManagementPolicy: "Parallel"


[root@k8s-master test]# kubectl get all -n dev
NAME                                       READY   STATUS    RESTARTS   AGE
pod/msa-deploy-consumer-6b75cf55d-bssl8    1/1     Running   0          101s
pod/msa-deploy-consumer-6b75cf55d-bxxp2    1/1     Running   0          101s
pod/msa-deploy-consumer-6b75cf55d-w658j    1/1     Running   0          101s
pod/msa-deploy-producer-7965c98bbf-7r4tb   1/1     Running   0          101s
pod/msa-deploy-producer-7965c98bbf-8wnbc   1/1     Running   0          101s
pod/msa-deploy-producer-7965c98bbf-rxw99   1/1     Running   0          101s
pod/msa-eureka-0                           1/1     Running   0          100s
pod/msa-eureka-1                           1/1     Running   0          100s
pod/msa-eureka-2                           1/1     Running   0          100s
pod/msa-gateway-597494c7f4-48zm2           1/1     Running   0          101s
pod/msa-gateway-597494c7f4-94r9m           1/1     Running   0          101s
pod/msa-gateway-597494c7f4-mn8mc           1/1     Running   0          101s

NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
service/msa-deploy-consumer   ClusterIP   10.110.49.38     <none>        8912/TCP          101s
service/msa-deploy-producer   ClusterIP   10.98.244.64     <none>        8911/TCP          101s
service/msa-eureka            NodePort    10.100.79.138    <none>        10001:30001/TCP   101s
service/msa-gateway           NodePort    10.109.116.147   <none>        8888:30008/TCP    101s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/msa-deploy-consumer   3/3     3            3           101s
deployment.apps/msa-deploy-producer   3/3     3            3           101s
deployment.apps/msa-gateway           3/3     3            3           101s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/msa-deploy-consumer-6b75cf55d    3         3         3       101s
replicaset.apps/msa-deploy-producer-7965c98bbf   3         3         3       101s
replicaset.apps/msa-gateway-597494c7f4           3         3         3       101s

NAME                          READY   AGE
statefulset.apps/msa-eureka   3/3     101s


# 删除
[root@k8s-master test]# helm delete msa -n dev
release "msa" uninstalled

[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
```



### 8.配置values

#### 8.1.修改配置信息

values.yaml 

```yaml
global:
  namespace: "dev"

producer:
  name: "msa-deploy-producer"
  serviceType: "ClusterIP"
  port: 8911
  targetPort: 8911
  replicas: 4
```



**生产者**

msa-deploy-producer.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.producer.name }}
  labels:
    app: {{ .Values.producer.name }}
spec:
  type: {{ .Values.producer.serviceType }}
  ports:
    - port: {{ .Values.producer.port }}
      targetPort: {{ .Values.producer.targetPort }}
  selector:
    app: {{ .Values.producer.name }}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.producer.name }}
spec:
  replicas: {{ .Values.producer.replicas }}
  selector:
    matchLabels:
      app: {{ .Values.producer.name }}
  template:
    metadata:
      labels:
        app: {{ .Values.producer.name }}
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: {{ .Values.producer.name }}
          image: 172.51.216.85:8888/springcloud/{{ .Values.producer.name }}:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: {{ .Values.producer.targetPort }}
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"
```

原配置

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-deploy-producer
  labels:
    app: msa-deploy-producer
spec:
  type: ClusterIP
  ports:
    - port: 8911
      targetPort: 8911
  selector:
    app: msa-deploy-producer

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-deploy-producer
spec:
  replicas: 3
  selector:
    matchLabels:
      app: msa-deploy-producer
  template:
    metadata:
      labels:
        app: msa-deploy-producer
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-deploy-producer
          image: 172.51.216.85:8888/springcloud/msa-deploy-producer:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8911
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"
```



#### 8.2.部署服务

```shell
[root@k8s-master test]# helm install msa cloud --dry-run --debug -n dev
install.go:173: [debug] Original chart version: ""
install.go:190: [debug] CHART PATH: /k8s/springcloud-helm/test/cloud

NAME: msa
LAST DEPLOYED: Thu Nov 11 16:31:39 2021
NAMESPACE: dev
STATUS: pending-install
REVISION: 1
TEST SUITE: None
USER-SUPPLIED VALUES:
{}

COMPUTED VALUES:
global:
  namespace: dev
producer:
  name: msa-deploy-producer
  port: 8911
  replicas: 4
  serviceType: ClusterIP
  targetPort: 8911


---

# Source: cloud/templates/msa-deploy-producer.yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-deploy-producer
  labels:
    app: msa-deploy-producer
spec:
  type: ClusterIP
  ports:
    - port: 8911
      targetPort: 8911
  selector:
    app: msa-deploy-producer

---
# Source: cloud/templates/msa-deploy-producer.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-deploy-producer
spec:
  replicas: 4
  selector:
    matchLabels:
      app: msa-deploy-producer
  template:
    metadata:
      labels:
        app: msa-deploy-producer
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-deploy-producer
          image: 172.51.216.85:8888/springcloud/msa-deploy-producer:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8911
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"
---


----------------------------------------------------------------------
# 安装
[root@k8s-master test]# helm install msa cloud -n dev
NAME: msa
LAST DEPLOYED: Thu Nov 11 16:35:10 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Spring Cloud!!!




[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                               	STATUS  	CHART      	APP VERSION
msa 	dev      	1       	2021-11-11 16:35:10.13704314 +0800 CST	deployed	cloud-0.3.0	1.0.1



# 检索已经发布的 release 的资源文件
[root@k8s-master test]# helm get manifest msa -n dev
---

# Source: cloud/templates/msa-deploy-producer.yaml
apiVersion: v1
kind: Service
metadata:
  namespace: dev
  name: msa-deploy-producer
  labels:
    app: msa-deploy-producer
spec:
  type: ClusterIP
  ports:
    - port: 8911
      targetPort: 8911
  selector:
    app: msa-deploy-producer

---
# Source: cloud/templates/msa-deploy-producer.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dev
  name: msa-deploy-producer
spec:
  replicas: 4
  selector:
    matchLabels:
      app: msa-deploy-producer
  template:
    metadata:
      labels:
        app: msa-deploy-producer
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: msa-deploy-producer
          image: 172.51.216.85:8888/springcloud/msa-deploy-producer:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: 8911
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"

---


[root@k8s-master test]# kubectl get all -n dev
NAME                                       READY   STATUS    RESTARTS   AGE
pod/msa-deploy-consumer-6b75cf55d-kxsvz    1/1     Running   0          4m23s
pod/msa-deploy-consumer-6b75cf55d-qg84f    1/1     Running   0          4m23s
pod/msa-deploy-consumer-6b75cf55d-tqktz    1/1     Running   0          4m23s
pod/msa-deploy-producer-7965c98bbf-6d976   1/1     Running   0          4m23s
pod/msa-deploy-producer-7965c98bbf-6npwt   1/1     Running   0          4m23s
pod/msa-deploy-producer-7965c98bbf-fhmgw   1/1     Running   0          4m23s
pod/msa-deploy-producer-7965c98bbf-nwx8k   1/1     Running   0          4m23s
pod/msa-eureka-0                           1/1     Running   0          4m23s
pod/msa-eureka-1                           1/1     Running   0          4m23s
pod/msa-eureka-2                           1/1     Running   0          4m23s
pod/msa-gateway-597494c7f4-2lcvs           1/1     Running   0          4m23s
pod/msa-gateway-597494c7f4-5vh8r           1/1     Running   0          4m23s
pod/msa-gateway-597494c7f4-7jt6x           1/1     Running   0          4m23s

NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
service/msa-deploy-consumer   ClusterIP   10.103.121.58    <none>        8912/TCP          4m23s
service/msa-deploy-producer   ClusterIP   10.108.38.255    <none>        8911/TCP          4m23s
service/msa-eureka            NodePort    10.111.194.177   <none>        10001:30001/TCP   4m23s
service/msa-gateway           NodePort    10.103.34.236    <none>        8888:30008/TCP    4m23s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/msa-deploy-consumer   3/3     3            3           4m23s
deployment.apps/msa-deploy-producer   4/4     4            4           4m23s
deployment.apps/msa-gateway           3/3     3            3           4m23s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/msa-deploy-consumer-6b75cf55d    3         3         3       4m23s
replicaset.apps/msa-deploy-producer-7965c98bbf   4         4         4       4m23s
replicaset.apps/msa-gateway-597494c7f4           3         3         3       4m23s

NAME                          READY   AGE
statefulset.apps/msa-eureka   3/3     4m23s
```



#### 8.3.升级

通过修改values.yaml升级

```shell
# 升级前
[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                               	STATUS  	CHART      	APP VERSION
msa 	dev      	1       	2021-11-11 16:35:10.13704314 +0800 CST	deployed	cloud-0.3.0	1.0.1  


[root@k8s-master test]# helm status msa -n dev
NAME: msa
LAST DEPLOYED: Thu Nov 11 16:35:10 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Spring Cloud!!!


[root@k8s-master test]# helm history msa -n dev
REVISION	UPDATED                 	STATUS  	CHART      	APP VERSION	DESCRIPTION     
1       	Thu Nov 11 16:35:10 2021	deployed	cloud-0.3.0	1.0.1      	Install complete


[root@k8s-master test]# kubectl get pod -n dev
NAME                                   READY   STATUS    RESTARTS   AGE
msa-deploy-consumer-6b75cf55d-kxsvz    1/1     Running   0          8m59s
msa-deploy-consumer-6b75cf55d-qg84f    1/1     Running   0          8m59s
msa-deploy-consumer-6b75cf55d-tqktz    1/1     Running   0          8m59s
msa-deploy-producer-7965c98bbf-6d976   1/1     Running   0          8m59s
msa-deploy-producer-7965c98bbf-6npwt   1/1     Running   0          8m59s
msa-deploy-producer-7965c98bbf-fhmgw   1/1     Running   0          8m59s
msa-deploy-producer-7965c98bbf-nwx8k   1/1     Running   0          8m59s
msa-eureka-0                           1/1     Running   0          8m59s
msa-eureka-1                           1/1     Running   0          8m59s
msa-eureka-2                           1/1     Running   0          8m59s
msa-gateway-597494c7f4-2lcvs           1/1     Running   0          8m59s
msa-gateway-597494c7f4-5vh8r           1/1     Running   0          8m59s
msa-gateway-597494c7f4-7jt6x           1/1     Running   0          8m59s


# 修改文件
# 修改版本version: 0.3.0  tag: 1.16 
[root@k8s-master cloud]# vim Chart.yaml 

apiVersion: v2
name: cloud
description: A Helm chart for Kubernetes
type: application
version: 0.4.0
appVersion: "1.0.4"

# 修改成8副本
[root@k8s-master cloud]# vim values.yaml
global:
  namespace: "dev"

producer:
  name: "msa-deploy-producer"
  serviceType: "ClusterIP"
  port: 8911
  targetPort: 8911
  replicas: 8


# 升级
[root@k8s-master test]# helm upgrade msa cloud -n dev
Release "msa" has been upgraded. Happy Helming!
NAME: msa
LAST DEPLOYED: Thu Nov 11 16:47:03 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
Spring Cloud!!!


[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	dev      	2       	2021-11-11 16:47:03.064552385 +0800 CST	deployed	cloud-0.4.0	1.0.4  



[root@k8s-master test]# helm history msa -n dev
REVISION	UPDATED                 	STATUS    	CHART      	APP VERSION	DESCRIPTION     
1       	Thu Nov 11 16:35:10 2021	superseded	cloud-0.3.0	1.0.1      	Install complete
2       	Thu Nov 11 16:47:03 2021	deployed  	cloud-0.4.0	1.0.4      	Upgrade complete


[root@k8s-master test]# kubectl get pod -n dev
NAME                                   READY   STATUS    RESTARTS   AGE
msa-deploy-consumer-6b75cf55d-kxsvz    1/1     Running   0          13m
msa-deploy-consumer-6b75cf55d-qg84f    1/1     Running   0          13m
msa-deploy-consumer-6b75cf55d-tqktz    1/1     Running   0          13m
msa-deploy-producer-7965c98bbf-4kgnh   1/1     Running   0          80s
msa-deploy-producer-7965c98bbf-6d976   1/1     Running   0          13m
msa-deploy-producer-7965c98bbf-6npwt   1/1     Running   0          13m
msa-deploy-producer-7965c98bbf-clnwr   1/1     Running   0          80s
msa-deploy-producer-7965c98bbf-fhmgw   1/1     Running   0          13m
msa-deploy-producer-7965c98bbf-jvwrl   1/1     Running   0          80s
msa-deploy-producer-7965c98bbf-nwx8k   1/1     Running   0          13m
msa-deploy-producer-7965c98bbf-p2hk4   1/1     Running   0          80s
msa-eureka-0                           1/1     Running   0          13m
msa-eureka-1                           1/1     Running   0          13m
msa-eureka-2                           1/1     Running   0          13m
msa-gateway-597494c7f4-2lcvs           1/1     Running   0          13m
msa-gateway-597494c7f4-5vh8r           1/1     Running   0          13m
msa-gateway-597494c7f4-7jt6x           1/1     Running   0          13m


[root@k8s-master test]# helm get manifest msa -n dev
```



### 9.Chart Hooks

#### 9.1.Hook配置

Hooks在资源清单中的metadata部分用annotations注解的方式进行声明：

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  annotations:
    "helm.sh/hook": post-install,post-upgrade
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": hook-succeeded
......
```



#### 9.2.生产者加Hook注解

msa-deploy-producer.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.producer.name }}
  labels:
    app: {{ .Values.producer.name }}
  annotations:
    "helm.sh/hook": post-install,post-upgrade
    "helm.sh/hook-weight": "-5"
spec:
  type: {{ .Values.producer.serviceType }}
  ports:
    - port: {{ .Values.producer.port }}
      targetPort: {{ .Values.producer.targetPort }}
  selector:
    app: {{ .Values.producer.name }}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.producer.name }}
  annotations:
    "helm.sh/hook": post-install,post-upgrade
    "helm.sh/hook-weight": "-5"
spec:
  replicas: {{ .Values.producer.replicas }}
  selector:
    matchLabels:
      app: {{ .Values.producer.name }}
  template:
    metadata:
      labels:
        app: {{ .Values.producer.name }}
    spec:
      imagePullSecrets:
        - name: harborsecret   #对应创建私有镜像密钥Secret
      containers:
        - name: {{ .Values.producer.name }}
          image: 172.51.216.85:8888/springcloud/{{ .Values.producer.name }}:2.0.0
          imagePullPolicy: Always #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: {{ .Values.producer.targetPort }}
          env:
            - name: EUREKA_SERVER
              value:  "http://msa-eureka-0.msa-eureka.dev:10001/eureka/,http://msa-eureka-1.msa-eureka.dev:10001/eureka/,http://msa-eureka-2.msa-eureka.dev:10001/eureka/"
            - name: ACTIVE
              value: "-Dspring.profiles.active=k8s"
            - name: DASHBOARD
              value: "sentinel-server.dev.svc.cluster.local:8858"
```



```shell
[root@k8s-master test]# helm install msa cloud --dry-run --debug -n dev


# 安装
[root@k8s-master test]# helm install msa cloud -n dev
NAME: msa
LAST DEPLOYED: Fri Nov 12 11:36:58 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Spring Cloud!!!


[root@k8s-master test]# helm list -n dev
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	dev      	1       	2021-11-12 11:36:58.339808934 +0800 CST	deployed	cloud-0.5.0	1.0.5 


# 检索已经发布的 release 的资源文件
[root@k8s-master test]# helm get manifest msa -n dev


[root@k8s-master test]# kubectl get all -n dev
NAME                                       READY   STATUS             RESTARTS   AGE
pod/msa-deploy-consumer-6b75cf55d-8mvcj    1/1     Running            0          4m19s
pod/msa-deploy-consumer-6b75cf55d-nbs2x    1/1     Running            0          4m19s
pod/msa-deploy-consumer-6b75cf55d-q9b98    1/1     Running            0          4m19s
pod/msa-deploy-producer-786f6dff44-4tn6h   0/1     ImagePullBackOff   0          4m18s
pod/msa-deploy-producer-786f6dff44-7wcbw   0/1     ImagePullBackOff   0          4m18s
pod/msa-deploy-producer-786f6dff44-b56r8   0/1     ImagePullBackOff   0          4m18s
pod/msa-eureka-0                           1/1     Running            0          4m19s
pod/msa-eureka-1                           1/1     Running            0          4m19s
pod/msa-eureka-2                           1/1     Running            0          4m19s
pod/msa-gateway-597494c7f4-lkqnx           1/1     Running            0          4m19s
pod/msa-gateway-597494c7f4-ltfpp           1/1     Running            0          4m19s
pod/msa-gateway-597494c7f4-mnn7g           1/1     Running            0          4m19s

NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
service/msa-deploy-consumer   ClusterIP   10.105.146.252   <none>        8912/TCP          4m19s
service/msa-deploy-producer   ClusterIP   10.101.112.240   <none>        8911/TCP          4m18s
service/msa-eureka            NodePort    10.103.75.142    <none>        10001:30001/TCP   4m19s
service/msa-gateway           NodePort    10.103.81.61     <none>        8888:30008/TCP    4m19s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/msa-deploy-consumer   3/3     3            3           4m19s
deployment.apps/msa-deploy-producer   0/3     3            0           4m18s
deployment.apps/msa-gateway           3/3     3            3           4m19s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/msa-deploy-consumer-6b75cf55d    3         3         3       4m19s
replicaset.apps/msa-deploy-producer-786f6dff44   3         3         0       4m18s
replicaset.apps/msa-gateway-597494c7f4           3         3         3       4m19s

NAME                          READY   AGE
statefulset.apps/msa-eureka   3/3     4m19s


# 卸载
[root@k8s-master test]# helm uninstall msa -n dev
release "msa" uninstalled

# Hook资源需要自己删除
[root@k8s-master test]# kubectl get all -n dev
NAME                                       READY   STATUS             RESTARTS   AGE
pod/msa-deploy-producer-786f6dff44-4tn6h   0/1     ImagePullBackOff   0          5m24s
pod/msa-deploy-producer-786f6dff44-7wcbw   0/1     ImagePullBackOff   0          5m24s
pod/msa-deploy-producer-786f6dff44-b56r8   0/1     ImagePullBackOff   0          5m24s

NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/msa-deploy-producer   ClusterIP   10.101.112.240   <none>        8911/TCP   5m24s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/msa-deploy-producer   0/3     3            0           5m24s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/msa-deploy-producer-786f6dff44   3         3         0       5m24s
```



