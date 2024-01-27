* TOC
{:toc}


### 1.ChartMuseum

#### 1.1.安装ChartMuseum

直接使用最简单的 docker run 方式，使用local 本地存储方式，通过 -v 映射到宿主机 /k8s/chartmuseum/charts
更多支持安装方式见官网

```shell
# 官方参考

https://chartmuseum.com/
https://github.com/helm/chartmuseum
https://github.com/chartmuseum/helm-push
https://github.com/chartmuseum/ui
```



```shell
# 搭建服务器：172.51.216.88


# 创建目录
[root@localhost k8s]# mkdir /k8s/chartmuseum/charts -p



# 运行容器
docker run -d --name chartmuseum-charts --restart=always \
-p 9999:8080 \
-e DEBUG=1 \
-e STORAGE=local \
-e STORAGE_LOCAL_ROOTDIR=/charts \
-v /k8s/chartmuseum/charts:/charts \
ghcr.io/helm/chartmuseum:v0.13.1
```

**注意：安装Harbor时，自动安装了一个chartmuseum，要注意区分。**



#### 1.2.测试

```shell
# 使用 curl 测试下接口，没有报错就行，当前仓库内容还是空的
[root@dev charts]# curl localhost:9999/api/charts
{}


[root@dev charts]# curl localhost:9999/api/charts |jq
-bash: jq: command not found
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100     2  100     2    0     0    358      0 --:--:-- --:--:-- --:--:--   400
(23) Failed writing body


# 访问地址
http://172.51.216.85:9999
```



#### 1.3.安装 push 插件

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



#### 1.4.添加Helm仓库

```shell
# 添加 helm repo
helm repo add chartmuseum http://172.51.216.88:9999
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
[root@k8s-master chartmuseum]# helm repo add chartmuseum http://172.51.216.88:9999
"chartmuseum" has been added to your repositories

[root@k8s-master1 chartmuseum]# helm repo list
NAME         	URL                                                   
stable       	http://mirror.azure.cn/kubernetes/charts              
aliyun       	https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
bitnami      	https://charts.bitnami.com/bitnami                    
ingress-nginx	https://kubernetes.github.io/ingress-nginx            
elastic      	https://helm.elastic.co                               
presslabs    	https://presslabs.github.io/charts                    
timescale    	https://charts.timescale.com                          
chartmuseum  	http://172.51.216.88:9999    


# 更新
[root@k8s-master chartmuseum]# helm repo update
```



#### 1.5.上传Helm仓库

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
# 推送失败

[root@k8s-master1 test]# helm cm-push cloud/ chartmuseum
Pushing cloud-1.0.0.tgz to chartmuseum...
Error: 500: open /charts/cloud-1.0.0.tgz: permission denied
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

Error: plugin "cm-push" exited with error




[root@k8s-master1 test]# helm cm-push cloud-1.0.0.tgz chartmuseum
Pushing cloud-1.0.0.tgz to chartmuseum...
Error: 500: open /charts/cloud-1.0.0.tgz: permission denied
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

Error: plugin "cm-push" exited with error
```




