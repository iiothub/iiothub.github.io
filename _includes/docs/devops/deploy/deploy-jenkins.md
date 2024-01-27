* TOC
{:toc}



## 一、概述



![](/images/devops/deploy/deploy-jenkins/jenkins-1.png)



**官网地址**

```shell
# 官网地址
https://www.jenkins.io/
https://www.jenkins.io/zh/

# 插件
http://updates.jenkins-ci.org/download/plugins/

# 下载
https://www.jenkins.io/zh/download/
https://pkg.jenkins.io/redhat-stable/
```



**安装步骤**

```shell
# 1.安装JDK
Jenkins需要依赖JDK，所以先安装JDK1.8
yum install java-1.8.0-openjdk* -y
安装目录为：/usr/lib/jvm


# 2.官网安装方式
# 导入jenkins源
# wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
# 导入jenkins官方证书
# rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
# 安装jenkins（安装的是最新的LTS版本）
# yum install -y jenkins


# 3.修改Jenkins配置
vi /etc/syscofig/jenkins
修改内容如下：
JENKINS_USER="root"
JENKINS_PORT="18888"


# 4.启动Jenkins
systemctl start jenkins


# 5.打开浏览器访问
http://182.92.210.65:18888   # 注意：本服务器把防火墙关闭了，如果开启防火墙，需要在防火墙添加端口


# 6.获取并输入admin账户密码
cat /var/lib/jenkins/secrets/initialAdminPassword


# 7.跳过插件安装
因为Jenkins插件需要连接默认官网下载，速度非常慢，而且经过会失败，所以我们暂时先跳过插件安装
```





## 二、安装部署



### 1.安装环境

```shell
# 操作系统：CentOS 7.6
# Jenkins版本：安装jenkins（安装的是最新的LTS版本）

# 内网IP：172.17.243.237
# 外网IP：182.92.210.65
# 端口：18888
```



### 2.环境准备

```shell
# 准备工作

# 1.安装JDK
Jenkins需要依赖JDK，所以先安装JDK1.8
yum install java-1.8.0-openjdk* -y
安装目录为：/usr/lib/jvm
```

```shell
# 安装JDK
[root@aliyun-19 ~]# yum install java-1.8.0-openjdk* -y


[root@aliyun-19 ~]# cd /usr/lib/jvm
[root@aliyun-19 jvm]# ll
total 4
lrwxrwxrwx 1 root root   26 Jan 12 16:35 java -> /etc/alternatives/java_sdk
lrwxrwxrwx 1 root root   32 Jan 12 16:35 java-1.8.0 -> /etc/alternatives/java_sdk_1.8.0
lrwxrwxrwx 1 root root   40 Jan 12 16:35 java-1.8.0-openjdk -> /etc/alternatives/java_sdk_1.8.0_openjdk
drwxr-xr-x 9 root root 4096 Jan 12 16:35 java-1.8.0-openjdk-1.8.0.312.b07-1.el7_9.x86_64
lrwxrwxrwx 1 root root   34 Jan 12 16:35 java-openjdk -> /etc/alternatives/java_sdk_openjdk
lrwxrwxrwx 1 root root   21 Jan 12 16:35 jre -> /etc/alternatives/jre
lrwxrwxrwx 1 root root   27 Jan 12 16:35 jre-1.8.0 -> /etc/alternatives/jre_1.8.0
lrwxrwxrwx 1 root root   35 Jan 12 16:35 jre-1.8.0-openjdk -> /etc/alternatives/jre_1.8.0_openjdk
lrwxrwxrwx 1 root root   51 Jan 12 16:35 jre-1.8.0-openjdk-1.8.0.312.b07-1.el7_9.x86_64 -> java-1.8.0-openjdk-1.8.0.312.b07-1.el7_9.x86_64/jre
lrwxrwxrwx 1 root root   29 Jan 12 16:35 jre-openjdk -> /etc/alternatives/jre_openjdk

[root@aliyun-19 jvm]# java -version
openjdk version "1.8.0_312"
OpenJDK Runtime Environment (build 1.8.0_312-b07)
OpenJDK 64-Bit Server VM (build 25.312-b07, mixed mode)
```



### 3.安装Jenkins

```shell
# 2.官网安装方式
# yum install -y ca-certificates
# 导入jenkins源
# wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
# 导入jenkins官方证书
# rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
# 安装jenkins（安装的是最新的LTS版本）
# yum install -y jenkins
```

```shell
[root@aliyun-8g ~]# yum install -y ca-certificates

# 导入jenkins源
[root@aliyun-8g ~]# wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo

# 导入jenkins官方证书
[root@aliyun-8g ~]# rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

# 安装jenkins（安装的是最新的LTS版本）
[root@aliyun-8g ~]# yum install -y jenkins


 # 查看yum安装文件
[root@aliyun-8g ~]# rpm -ql jenkins
/etc/init.d/jenkins
/etc/logrotate.d/jenkins
/etc/sysconfig/jenkins
/usr/lib/jenkins
/usr/lib/jenkins/jenkins.war
/usr/sbin/rcjenkins
/var/cache/jenkins
/var/lib/jenkins
/var/log/jenkins
```



### 4.配置Jenkins

```shell
# 3.修改Jenkins配置
vim /etc/sysconfig/jenkins

# 修改内容如下：
JENKINS_USER="root"
JENKINS_PORT="18888"
```

```shell
# 查询 yum 下载 Jenkins 安装的文件

[root@qfedu.com ~]# rpm -ql jenkins
/etc/init.d/jenkins         # 启动文件
/etc/logrotate.d/jenkins    # 日志分割配置文件
/etc/sysconfig/jenkins      # jenkins主配置文件
/usr/lib/jenkins            # 存放war包目录
/usr/lib/jenkins/jenkins.war   # war 包 
/usr/sbin/rcjenkins         # 命令
/var/cache/jenkins          # war包解压目录 jenkins网页代码目录
/var/lib/jenkins            # jenkins 工作目录
/var/log/jenkins            # 日志
```



### 5.启动Jenkins服务  

```shell
# 4.启动Jenkins
systemctl start jenkins
```



### 6.访问 Jenkins

**1.打开浏览器访问**

```shell
# 注意：本服务器把防火墙关闭了，如果开启防火墙，需要在防火墙添加端口

http://182.92.210.65:18888
```

![](/images/devops/deploy/deploy-jenkins/jenkins-2.png)



**2.获取并输入admin账户密码**

```shell
# 获取并输入admin账户密码
cat /var/lib/jenkins/secrets/initialAdminPassword

# admin密码
[root@aliyun-8g ~]# cat /var/lib/jenkins/secrets/initialAdminPassword
8d26ac1ad0b640f3aaa6b7d47bdb1437
```



**3.跳过插件安装**

**因为Jenkins插件需要连接默认官网下载，速度非常慢，而且经过会失败，所以我们暂时先跳过插件安装**

![](/images/devops/deploy/deploy-jenkins/jenkins-3.png)

![](/images/devops/deploy/deploy-jenkins/jenkins-4.png)



**4.创建新管理员**

![](/images/devops/deploy/deploy-jenkins/jenkins-5.png)

![](/images/devops/deploy/deploy-jenkins/jenkins-6.png)

![](/images/devops/deploy/deploy-jenkins/jenkins-7.png)

![](/images/devops/deploy/deploy-jenkins/jenkins-8.png)



```shell
# 登录信息

http://182.92.210.65:18888/
hollysys
******
```



### 7.修改Jenkins插件下载地址  

Jenkins本身不提供很多功能，我们可以通过使用插件来满足我们的使用。例如从Gitlab拉取代码，使用Maven构建项目等功能需要依靠插件完成。接下来演示如何下载插件。



**修改Jenkins插件下载地址**
Jenkins国外官方插件地址下载速度非常慢，所以可以修改为国内插件地址



**1.下载插件地址**

**Jenkins->Manage Jenkins->Manage Plugins，点击Available**  



![](/images/devops/deploy/deploy-jenkins/jenkins-9.png)

**这样做是为了把Jenkins官方的插件列表下载到本地，接着修改地址文件，替换为国内插件地址**  

```shell
# Jenkins安装地址
[root@aliyun-19 jenkins]# pwd
/var/lib/jenkins
[root@aliyun-19 jenkins]# ll
total 68
-rw-r--r-- 1 root root 1644 Jan 12 17:17 config.xml
-rw-r--r-- 1 root root  156 Jan 12 16:54 hudson.model.UpdateCenter.xml
-rw------- 1 root root 1712 Jan 12 16:54 identity.key.enc
-rw-r--r-- 1 root root    7 Jan 12 17:17 jenkins.install.InstallUtil.lastExecVersion
-rw-r--r-- 1 root root    7 Jan 12 17:17 jenkins.install.UpgradeWizard.state
-rw-r--r-- 1 root root  184 Jan 12 17:16 jenkins.model.JenkinsLocationConfiguration.xml
-rw-r--r-- 1 root root  171 Jan 12 16:54 jenkins.telemetry.Correlator.xml
drwxr-xr-x 2 root root 4096 Jan 12 16:54 jobs
drwxr-xr-x 3 root root 4096 Jan 12 16:54 logs
-rw-r--r-- 1 root root  907 Jan 12 16:54 nodeMonitors.xml
drwxr-xr-x 2 root root 4096 Jan 12 16:54 nodes
drwxr-xr-x 2 root root 4096 Jan 12 16:54 plugins
-rw-r--r-- 1 root root   64 Jan 12 16:54 secret.key
-rw-r--r-- 1 root root    0 Jan 12 16:54 secret.key.not-so-secret
drwx------ 4 root root 4096 Jan 12 17:20 secrets
drwxr-xr-x 2 root root 4096 Jan 12 16:54 updates
drwxr-xr-x 2 root root 4096 Jan 12 16:54 userContent
drwxr-xr-x 3 root root 4096 Jan 12 17:14 users


# 插件下载地址 default.json
[root@aliyun-19 jenkins]# cd updates/
[root@aliyun-19 updates]# pwd
/var/lib/jenkins/updates
[root@aliyun-19 updates]# ll
total 2484
-rw-r--r-- 1 root root 2533571 Jan 12 16:54 default.json
-rw-r--r-- 1 root root    5902 Jan 12 16:54 hudson.tasks.Maven.MavenInstaller
```



**2.替换为国内插件地址**

```shell
# 替换位置 /var/lib/jenkins/updates

[root@aliyun-19 updates]# cd /var/lib/jenkins/updates
[root@aliyun-19 updates]# ll
total 2484
-rw-r--r-- 1 root root 2533571 Jan 12 16:54 default.json
-rw-r--r-- 1 root root    5902 Jan 12 16:54 hudson.tasks.Maven.MavenInstaller
[root@aliyun-19 updates]# pwd
/var/lib/jenkins/updates
```

```shell
# 替换代码
sed -i 's/http:\/\/updates.jenkinsci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json && sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' default.json


[root@aliyun-19 updates]# sed -i 's/http:\/\/updates.jenkinsci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json && sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' default.json
```



**3.更新站点地址**

**最后，Manage Plugins点击Advanced，把Update Site改为国内插件下载地址**  

```shell
https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
```

![](/images/devops/deploy/deploy-jenkins/jenkins-10.png)

![](/images/devops/deploy/deploy-jenkins/jenkins-11.png)

**4.重启Jenkins**

Sumbit后，在浏览器输入： http://182.92.210.65:18888/restart ，重启Jenkins。  



### 8.系统中文汉化

**下载中文汉化插件**

**Jenkins->Manage Jenkins->Manage Plugins，点击Available，搜索"Chinese"**  

![](/images/devops/deploy/deploy-jenkins/jenkins-12.png)

![](/images/devops/deploy/deploy-jenkins/jenkins-13.png)

![](/images/devops/deploy/deploy-jenkins/jenkins-14.png)



### 9.安装Git插件和Git工具

**1.安装Git**

**CentOS7上安装Git工具：**  

```shell
# yum install git -y
# git --version
```



**2.安装Git插件**

![](/images/devops/deploy/deploy-jenkins/jenkins-15.png)



### 10.Maven安装和配置

在Jenkins集成服务器上，我们需要安装Maven来编译和打包项目。  



#### 10.1.安装Maven

- 先上传Maven软件到Jenkins服务器
- tar -xzf apache-maven-3.6.2-bin.tar.gz 解压
- mkdir -p /opt/maven 创建目录
- mv apache-maven-3.6.2/* /opt/maven 移动文件  



```shell
[root@aliyun-8g ~]# ll
total 8932
-rw-r--r-- 1 root root 9142315 Jan 13 17:24 apache-maven-3.6.2-bin.tar.gz

[root@aliyun-8g ~]# tar -zxf apache-maven-3.6.2-bin.tar.gz 
[root@aliyun-8g ~]# ll
total 8936
drwxr-xr-x 6 root root    4096 Jan 13 17:25 apache-maven-3.6.2
-rw-r--r-- 1 root root 9142315 Jan 13 17:24 apache-maven-3.6.2-bin.tar.gz


[root@aliyun-8g ~]# mkdir -p /opt/maven
[root@aliyun-8g ~]# mv apache-maven-3.6.2/* /opt/maven
```



#### 10.2.配置环境变量

```shell
[root@aliyun-8g maven]# vi /etc/profile

export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
export MAVEN_HOME=/opt/maven
export PATH=$PATH:$JAVA_HOME/bin:$MAVEN_HOME/bin


# 配置生效
[root@aliyun-8g ~]# source /etc/profile


# 查找Maven版本
[root@aliyun-8g ~]# mvn -v
Apache Maven 3.6.2 (40f52333136460af0dc0d7232c0dc0bcf0d9e117; 2019-08-27T23:06:16+08:00)
Maven home: /opt/maven
Java version: 1.8.0_312, vendor: Red Hat, Inc., runtime: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-1.el7_9.x86_64/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-957.21.3.el7.x86_64", arch: "amd64", family: "unix"
```



#### 10.3.全局配置JDK和Maven   

![](/images/devops/deploy/deploy-jenkins/jenkins-16.png)

**Jenkins->Global Tool Configuration->JDK->新增JDK，配置如下：** 

**/usr/lib/jvm/java-1.8.0-openjdk**

![](/images/devops/deploy/deploy-jenkins/jenkins-17.png)



**Jenkins->Global Tool Configuration->Maven->新增Maven，配置如下：**  

**/opt/maven**

![](/images/devops/deploy/deploy-jenkins/jenkins-18.png)



#### 10.4.添加Jenkins全局变量  

![](/images/devops/deploy/deploy-jenkins/jenkins-19.png)



**Manage Jenkins->Configure System->Global Properties ，添加三个全局变量**
**JAVA_HOME、M2_HOME、PATH+EXTRA**  

![](/images/devops/deploy/deploy-jenkins/jenkins-20.png)

```shell
JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
M2_HOME=/opt/maven
PATH+EXTRA=$M2_HOME/bin
```

![](/images/devops/deploy/deploy-jenkins/jenkins-21.png)

![](/images/devops/deploy/deploy-jenkins/jenkins-22.png)



#### 10.5.修改Maven的settings.xml  

```shell
mkdir /root/repo 创建本地仓库目录
vim /opt/maven/conf/settings.xml

本地仓库改为：/root/repo/
添加阿里云私服地址：

	 <mirror>
      <id>alimaven</id>
      <mirrorOf>central</mirrorOf>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
     </mirror>
```

![](/images/devops/deploy/deploy-jenkins/jenkins-23.png)

![](/images/devops/deploy/deploy-jenkins/jenkins-24.png)



#### 10.6.测试Maven配置  

使用之前的gitlab密码测试项目，修改配置  

![](/images/devops/deploy/deploy-jenkins/jenkins-25.png)

![](/images/devops/deploy/deploy-jenkins/jenkins-26.png)

**输入：mvn clean package**  

![](/images/devops/deploy/deploy-jenkins/jenkins-27.png)



**再次构建，如果可以把项目打成jar包，代表maven环境配置成功啦！**  

**查看/var/lib/jenkins/workspace/目录，发现已经从Gitlab成功拉取了代码到Jenkins中。**  

```shell
root@aliyun-8g ~]# cd /var/lib/jenkins/workspace/
[root@aliyun-8g workspace]# ll
total 16
drwxr-xr-x 5 root root 4096 Jan 14 09:27 itcast01
drwxr-xr-x 2 root root 4096 Jan 14 09:26 itcast01@tmp

[root@aliyun-8g workspace]# cd itcast01
[root@aliyun-8g itcast01]# ll
total 12
-rw-r--r-- 1 root root 1580 Jan 13 16:51 pom.xml
drwxr-xr-x 4 root root 4096 Jan 13 16:51 src
drwxr-xr-x 9 root root 4096 Jan 14 09:28 target
[root@aliyun-8g itcast01]# cd target/
[root@aliyun-8g target]# ll
total 18100
drwxr-xr-x 3 root root     4096 Jan 14 09:27 classes
drwxr-xr-x 3 root root     4096 Jan 14 09:27 generated-sources
drwxr-xr-x 3 root root     4096 Jan 14 09:27 generated-test-sources
drwxr-xr-x 2 root root     4096 Jan 14 09:27 maven-archiver
drwxr-xr-x 3 root root     4096 Jan 14 09:27 maven-status
drwxr-xr-x 2 root root     4096 Jan 14 09:27 surefire-reports
drwxr-xr-x 3 root root     4096 Jan 14 09:27 test-classes

-rw-r--r-- 1 root root 18497955 Jan 14 09:28 web-demo-1.0.0.jar   # 生成了jar包

-rw-r--r-- 1 root root     3499 Jan 14 09:27 web-demo-1.0.0.jar.original
```

![](/images/devops/deploy/deploy-jenkins/jenkins-28.png)





## 三、常用插件



### 1.系统中文汉化插件

**下载中文汉化插件**

**Jenkins->Manage Jenkins->Manage Plugins，点击Available，搜索"Chinese"**  

![](/images/devops/deploy/deploy-jenkins/jenkins-29.png)



### 2.用户权限管理插件

**利用Role-based Authorization Strategy 插件来管理Jenkins用户权限**  

**安装Role-based Authorization Strategy插件**  

![](/images/devops/deploy/deploy-jenkins/jenkins-30.png)



### 3.Git插件

![](/images/devops/deploy/deploy-jenkins/jenkins-31.png)



### 4.Jenkins凭证管理插件

**凭据可以用来存储需要密文保护的数据库密码、Gitlab密码信息、Docker私有仓库密码等，以便Jenkins可以和这些第三方的应用进行交互。**  

安装Credentials Binding插件
要在Jenkins使用凭证管理功能，需要安装Credentials Binding插件  

![](/images/devops/deploy/deploy-jenkins/jenkins-32.png)

**注意：新版本好像自动安装了**



### 5.安装Gitlab Hook插件

需要安装两个插件：Gitlab Hook和GitLab  

![](/images/devops/deploy/deploy-jenkins/jenkins-33.png)

![](/images/devops/deploy/deploy-jenkins/jenkins-34.png)



### 6.Email Extension插件

![](/images/devops/deploy/deploy-jenkins/jenkins-35.png)



### 7.Git分支标签选择参数插件

![](/images/devops/deploy/deploy-jenkins/jenkins-36.png)



### 8.多选参数插件

![](/images/devops/deploy/deploy-jenkins/jenkins-37.png)



### 9.SSH 插件

**1.Publish Over SSH 插件**

Publish Over SSH有安全漏洞，暂时无法安装

https://updates.jenkins-ci.org/download/plugins/



**2.安装SSH插件**

![](/images/devops/deploy/deploy-jenkins/jenkins-38.png)



### 10.SonarQube Scanner插件

![](/images/devops/deploy/deploy-jenkins/jenkins-39.png)



### 11.Pipeline插件

![](/images/devops/deploy/deploy-jenkins/jenkins-40.png)



