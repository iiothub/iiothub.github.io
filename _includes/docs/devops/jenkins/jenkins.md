* TOC
{:toc}



## 一、概述



![](/images/devops/jenkins/jenkins/jenkins-103.png)



Jenkins 是一款流行的开源持续集成（Continuous Integration）工具，广泛用于项目开发，具有自动
化构建、测试和部署等功能。官网： http://jenkins-ci.org/。

**Jenkins的特征：**

- 开源的Java语言开发持续集成工具，支持持续集成，持续部署
- 易于安装部署配置：可通过yum安装,或下载war包以及通过docker容器等快速实现安装部署，可方便web界面配置管理
- 消息通知及测试报告：集成RSS/E-mail通过RSS发布构建结果或当构建完成时通过e-mail通知，生成JUnit/TestNG测试报告  
- 分布式构建：支持Jenkins能够让多台计算机一起构建/测试
- 文件识别：Jenkins能够跟踪哪次构建生成哪些jar，哪次构建使用哪个版本的jar等
- 丰富的插件支持：支持扩展插件，你可以开发适合自己团队使用的工具，如git，svn，maven，docker等  



### 1.Jenkins介绍

Jenkins的前身是Hudson，采用JAVA编写的持续集成开源工具。Hudson由Sun公司在2004年启动，第一个版本于2005年在java.net发布。2007年开始Hudson逐渐取代CruiseControl和其他的开源构建工具的江湖地位。在2008年的JavaOne大会上在开发者解决方案中获得杜克选择大奖（Duke's Choice Award）。

在2010年11月期间，因为Oracle对Sun的收购带来了Hudson的所有权问题。主要的项目贡献者和Oracle之间，尽管达成了很多协议，但有个关键问题就是商标名称“Hudson”。甲骨文在2010年12月声明拥有该名称并申请商标的权利。  因此，2011年1月11日，有人要求投票将项目名称从“Hudson”改为“Jenkins”。2011年1月29日，该建议得到社区投票的批准，创建了Jenkins项目。

2011年2月1日，甲骨文表示，他们打算继续开发Hudson，并认为Jenkins只是一个分支，而不是重命名。因此，Jenkins和Hudson继续作为两个独立的项目，每个都认为对方是自己的分支。到2013年12月，GitHub上的Jenkins拥有567个项目成员和约1,100个公共仓库，与此相对的Hudson有32个项目成员和17个公共仓库。到现在两者的差异更多，应该说Jenkins已经全面超越了Hudson。此外，大家可能是出于讨厌Oracle的情绪，作为Java开发者天然地应该支持和使用Jenkins。

后面Hudson被Oracle捐给了Eclipse基金会，所以右边这老头有个Eclipse的光环加持。

为什么Jenkins更受大家欢迎。

**由开发者主导、面向开发者** 

![](/images/devops/jenkins/jenkins/jenkins-118.png)

首先，曾经是Hudson开发人员中的99%都转向了Jenkins的开发，其中包括最初的创建者川口清子（Kohsuke  Kawaguchi）。他独自写了大部分代码，并且他的经验是Hudson各种高级特性的关键来源。Jenkins的开发社区更活跃。所以对很多人而言，从血统上看Hudson是后娘养的，Jenkins才是亲生的！

 **治理和社区** 

Jenkins开发社区的管理是很开放的。 有一个独立的董事会，其中包括来自Yahoo!,  CloudBees，Cloudera和Apture等多家公司的长期以来的Hudson开发人员。每次会议后，他们定期举行治理会议并发表意见，征询公众意见。他们还将所有代码都捐赠给公共利益软件组织（SPI），以确保社区持续开放。

 **稳定性** 

分手后，针对Jenkins的贡献不断持续增加，Jenkins制定了新的长期支持发布线。社区定大约每三个月发布一次稳定版本的补丁。

 **插件的平台** 

Jenkins支持超过1000个插件。凭借多样而强大的插件Jenkins成了整个开发生命周期中的一个中心点。

到了2017年，两者的发展差异更大了。Jenkins应该说是CI工具中公认的老大，而Hudson不仅不能与Jenkins比，跟其他CI工具比也没什么优势，完全沉沦了。

slant网站对一系列CI工具做了一次对比，其中Jenkins和Hudson的情况如下。

 **1.基本面问题** 

| 工具               | Jenkins | Hudson |
| ------------------ | ------- | ------ |
| 最好的CI工具       | 1       | 22     |
| 最好的JAVA CI工具  | 1       | 7      |
| 最好的自托管CI工具 | 3       | 4      |

 

**2.其他支持** 

| 工具                     | Jenkins | Hudson |
| ------------------------ | ------- | ------ |
| 对Window支持最好         | 1       | 无排名 |
| 最好的开源CI工具         | 1       | 无排名 |
| 对BitBucket的支持        | 2       | 无排名 |
| 对移动开发者支持最好的CI | 4       | 无排名 |

 

**3.更多特征** 

| Jenkins                                                      | Hudson                                    |
| ------------------------------------------------------------ | ----------------------------------------- |
| 免费且开源                                                   | 与Jenkins共享了很多代码，安装还是挺简单的 |
| 关键的环境变量可以安全存储                                   |                                           |
| 支持多个SCM，包括SVN, Mercurial, Git。集成了GitHub和Bitbucket |                                           |
| 高度可配置                                                   |                                           |
| 资源和教程很多                                               |                                           |
| 安装运行简单                                                 |                                           |
| 分布式的构建也能高效运行                                     |                                           |
| 可跨平台部署                                                 |                                           |
| 很多高质量的插件                                             |                                           |
| 得奖无数                                                     |                                           |
| 庞大的社区                                                   |                                           |



#### 1.1.Jenkins 功能

- 持续的软件版本发布/测试项目
- 监控外部调用执行的工作



#### 1.2.Jenkins 概念

Jenkins是一个功能强大的应用程序，允许**持续集成和持续交付项目**，无论用的是什么平台。这是一个免费的开源项目，可以处理任何类型的构建或持续集成。集成Jenkins可以用于一些测试和部署技术。Jenkins是一种软件允许持续集成。



#### 1.3.Jenkins 目的

- 持续、自动地构建/测试软件项目


- 监控软件开放流程，快速问题定位及处理，提提高开发效率



#### 1.4.Jenkins 特性

- 开源的java语言开发持续集成工具，支持CI，CD


- 易于安装部署配置：可通过yum安装,或下载war包以及通过docker容器等快速实现安装部署，可方便web界面配置管理


- 消息通知及测试报告：集成RSS/E-mail通过RSS发布构建结果或当构建完成时通过e-mail通知，生成JUnit/TestNG测试报告


- 分布式构建：支持Jenkins能够让多台计算机一起构建/测试


- 文件识别:Jenkins能够跟踪哪次构建生成哪些jar，哪次构建使用哪个版本的jar等


- 丰富的插件支持:支持扩展插件，你可以开发适合自己团队使用的工具，如git，svn，maven，docker等



#### 1.5.产品发布流程

产品设计成型 -> 开发人员开发代码 -> 测试人员测试功能 -> 运维人员发布上线


- 持续集成（Continuous integration，简称CI）


- 持续交付（Continuous delivery）


- 持续部署（continuous deployment） 



### 2.Jenkins CI/CD 流程

![](/images/devops/jenkins/jenkins/jenkins-104.png)

说明：这张图稍微更形象一点，上线之前先把代码git到版本仓库，然后通过Jenkins将Java项目通过maven去构建，这是在非容器之前，典型的自动化的一个版本上线流程。那它有哪些问题呢？

如：它的测试环境，预生产环境，测试环境。会存在一定的兼容性问题 （环境之间会有一定的差异） 



![](/images/devops/jenkins/jenkins/jenkins-105.png)

 

说明：它这里有一个docker harbor 的镜像仓库，通常会把你的环境打包为一个镜像，通过镜像的方式来部署。





## 二、基础



### 1.Jenkins用户权限管理

#### 1.1.安装插件

**利用Role-based Authorization Strategy 插件来管理Jenkins用户权限**  

**安装Role-based Authorization Strategy插件**  

![](/images/devops/jenkins/jenkins/jenkins-1.png)

![](/images/devops/jenkins/jenkins/jenkins-2.png)



#### 1.2.开启权限全局安全配置  

![](/images/devops/jenkins/jenkins/jenkins-3.png)

![](/images/devops/jenkins/jenkins/jenkins-4.png)



#### 1.3.创建角色

![](/images/devops/jenkins/jenkins/jenkins-5.png)

![](/images/devops/jenkins/jenkins/jenkins-6.png)

![](/images/devops/jenkins/jenkins/jenkins-7.png)

- **Global roles（全局角色）：**管理员等高级用户可以创建基于全局的角色 
- **Project roles（项目角色）：**针对某个或者某些项目的角色 Slave roles（奴隶角色）：节点相关的权限  



**我们添加以下三个角色：**

- baseRole：该角色为全局角色。这个角色需要绑定Overall下面的Read权限，是为了给所有用户绑定最基本的Jenkins访问权限。注意：如果不给后续用户绑定这个角色，会报错误：用户名 is missing the Overall/Read permission。
- role1：该角色为项目角色。使用正则表达式绑定"itcast.**"，意思是只能操作itcast开头的项目*
- role2：该角色也为项目角色。绑定"itheima.*"，意思是只能操作itheima开头的项目



![](/images/devops/jenkins/jenkins/jenkins-8.png)



#### 1.4.创建用户  

在系统管理页面进入 Manage Users  

![](/images/devops/jenkins/jenkins/jenkins-9.png)

![](/images/devops/jenkins/jenkins/jenkins-10.png)

![](/images/devops/jenkins/jenkins/jenkins-11.png)

**创建用户 jack、eric**

![](/images/devops/jenkins/jenkins/jenkins-12.png)



#### 1.5.分配角色

系统管理页面进入Manage and Assign Roles，点击Assign Roles

**绑定规则如下：**
eric用户分别绑定baseRole和role1角色
jack用户分别绑定baseRole和role2角色  

![](/images/devops/jenkins/jenkins/jenkins-13.png)

![](/images/devops/jenkins/jenkins/jenkins-14.png)



#### 1.6.创建项目

以hollysys管理员账户创建两个项目，分别为itcast01和itheima01  

![](/images/devops/jenkins/jenkins/jenkins-15.png)

![](/images/devops/jenkins/jenkins/jenkins-16.png)

**结果为：**
eric用户登录，只能看到itcast01项目
jack用户登录，只能看到itheima01项目  

![](/images/devops/jenkins/jenkins/jenkins-17.png)

![](/images/devops/jenkins/jenkins/jenkins-18.png)



### 2.Jenkins凭证管理  

**凭据可以用来存储需要密文保护的数据库密码、Gitlab密码信息、Docker私有仓库密码等，以便Jenkins可以和这些第三方的应用进行交互。**  



#### 2.1.安装插件

安装Credentials Binding插件
要在Jenkins使用凭证管理功能，需要安装Credentials Binding插件  

![](/images/devops/jenkins/jenkins/jenkins-19.png)

安装插件后，在这里管理所有凭证  

![](/images/devops/jenkins/jenkins/jenkins-20.png)

![](/images/devops/jenkins/jenkins/jenkins-21.png)

![](/images/devops/jenkins/jenkins/jenkins-22.png)



**可以添加的凭证有5种：**  

- **Username with password：用户名和密码**
- **SSH Username with private key： 使用SSH用户和密钥**
- Secret file：需要保密的文本文件，使用时Jenkins会将文件复制到一个临时目录中，再将文件路径
- 设置到一个变量中，等构建结束后，所复制的Secret file就会被删除。
- Secret text：需要保存的一个加密的文本串，如钉钉机器人或Github的api token
- Certificate：通过上传证书文件的方式  



**常用的凭证类型有：Username with password（用户密码）和SSH Username with private key（SSH密钥）**  



#### 2.2.用户密码类型凭据

**1.创建凭证**

**项目 => 配置 => Git配置**

![](/images/devops/jenkins/jenkins/jenkins-26.png)

![](/images/devops/jenkins/jenkins/jenkins-27.png)

![](/images/devops/jenkins/jenkins/jenkins-28.png)

![](/images/devops/jenkins/jenkins/jenkins-29.png)

![](/images/devops/jenkins/jenkins/jenkins-30.png)

**选择"Username with password"，输入Gitlab的用户名和密码，点击"确定"。**  

![](/images/devops/jenkins/jenkins/jenkins-31.png)



**2.测试凭证**

构建项目

![](/images/devops/jenkins/jenkins/jenkins-32.png)

![](/images/devops/jenkins/jenkins/jenkins-33.png)

**查看/var/lib/jenkins/workspace/目录，发现已经从Gitlab成功拉取了代码到Jenkins中。**  

```shell
root@aliyun-8g ~]# cd /var/lib/jenkins/workspace/
[root@aliyun-8g workspace]# ll
total 8
drwxr-xr-x 4 root root 4096 Jan 13 16:20 itcast01
drwxr-xr-x 2 root root 4096 Jan 13 16:20 itcast01@tmp
```



#### 2.3.SSH密码类型凭据

![](/images/devops/jenkins/jenkins/jenkins-34.png)



**1.使用root用户生成公钥和私钥**

```shell
ssh-keygen -t rsa
在/root/.ssh/目录保存了公钥和使用  

id_rsa：私钥文件
id_rsa.pub：公钥文件
```

```shell
# 生成秘钥
[root@aliyun-8g ~]# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:CMexw8KnhW1DgikxutHkA9OR2v2e4EjYHMjXEt4flLQ root@aliyun-8g
The key's randomart image is:
+---[RSA 2048]----+
|=oo=. +..        |
|oB=o B =.        |
|+==.O &E         |
|o+o=.@ =         |
|.+..o.o S        |
|. + . ..         |
| . o o .         |
|  . . o          |
|                 |
+----[SHA256]-----+



[root@aliyun-8g ~]# cd .ssh/
[root@aliyun-8g .ssh]# ll
total 8
-rw------- 1 root root    0 Jan 13 10:20 authorized_keys
-rw------- 1 root root 1679 Jan 13 16:31 id_rsa
-rw-r--r-- 1 root root  396 Jan 13 16:31 id_rsa.pub


# 查看公钥
[root@aliyun-8g .ssh]# cat id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDCzDVndhIWuz0lnSPC9KH/8f1d3SJnByU9hyx/RJiM9vBDtBTcFl6ivMJMNe0Vez5cPva9KgEjNWBGT3M4rcLSypVFTdjTZqtj26dCgdgf3cdf6QdRjoce6fDL9Flw4muFCyS5XDY7DYpZK/nnTuv2v/VyaIhLeDAb5LVExwl6eKWGk07cMol6t2djd0pOUeey/4mH4STLPBb+oVFHVOe4R9JoMliEODN1zl4LU/zz+BcdFE6VvDYrZS7Xlhd3uouT2XY5PRELOq6yUTRbYUK4gLFU48kfFR2hX0GiZF+Hg+7d/mImvEJwFkHEEnnXBTQy7i5hQ5O21jHh3oVEMlYB root@aliyun-8g


# 查看私钥
[root@aliyun-8g .ssh]# cat id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEAwsw1Z3YSFrs9JZ0jwvSh//H9Xd0iZwclPYcsf0SYjPbwQ7QU
3BZeorzCTDXtFXs+XD72vSoBIzVgRk9zOK3C0sqVRU3Y02arY9unQoHYH93HX+kH
UY6HHunwy/RZcOJrhQskuVw2Ow2KWSv5507r9r/1cmiIS3gwG+S1RMcJenilhpNO
3DKJerdnY3dKTlHnsv+Jh+EkyzwW/qFRR1TnuEfSaDJYhDgzdc5eC1P88/gXHRRO
lbw2K2Uu15YXd7qLk9l2OT0RCzquslE0W2FCuICxVOPJHxUdoV9BomRfh4Pu3f5i
JrxCcBZBxBJ51wU0Mu4uYUOTttYx4d6FRDJWAQIDAQABAoIBAQCYU3evX/TlMaWv
NCIy4XmM2351V+b/CedlJb72Bn4EPVXEm510PUnjmBeX4NN0aNtq5xGq+p3JGoQe
dyJyv+4JR8FSYH2dUjvT6n/w0fhfct3lciP28q1WzzktQ/Zs/6F0eDJPgHwn0X7O
HEVfS6fZXGJjBLsPyPxV05KsJbiTu0bopZTkYmpzWvvNoZx/S8lViZ3gDj/VRi2f
j0kMgy6UuzGJWy2/sio+RY8kdlCvhz22pMszG8UgqihnERS0gg3Zm+HLSTto6U1x
gtjB3Yfmx50B4djxGBOZHe0wbor5b5DW/GeXQQRYFVDuhcJ58uBLWphr2SvowycH
1xvSmEABAoGBAOL195ih3n5+mlxhMT7pl8N15ULNnsN+A/emCPQK+2pP7rwKO3ET
N7RKHLds3lkTV3gshcNd/Oq2p3yzh1UyiSUZSz7eN4WMGex/a6cKlFE8BcLvQskm
WJpFsxE2SrhTqzq0zM5OvSm02FxxKlc2jB4uo63TFj9n7RWS8rwLjFlhAoGBANu4
v4fSTyxdWujCBtplFdo4sShjPYFVgFw4ZpuZKVHPFTMeCnNBhuGOZwOFk1i3/GN8
acTiWjpzJWSv9YYPDMhomowzPmsJS3ycKBQLtsFZjiycdpXleCdbvrNEjwGO1BjA
/DHWWt9hMqlH7HI70hJE5WDK/fmVaMB/2C+jAiChAoGASwDyNBS6TJ9WL9VGyv2z
U7rwauU85GoOsZbDOrMuZvHHeYkAH9wz+nbLiqqFyHYl3+cGxYuX+5ElRIan4LX0
sLftL/eL7axhHND3KJrMbRQi60rajVMI0OLbzIJeqw+rdJkvXbaTuOa04cfcMDos
kATlvpoVrhqQNSL86LwAQ8ECgYEAnjGw7Ig16ro4JtbzejBHgHtKycpR0RmPNlaB
QcwPXNBc8hXR7lOiWild78IvaTPmanZ77H4P+n9Gz+yEOIYDbRMrGoAWk5f4mnoP
vQcGCMWCwInSM3AohyXd8lINKFD+Ueg4a2VqvePMRub6zPBW+kJSZ9Me8qBo8Bfb
vch+UqECgYEApp69ozFpV942sWUiX2wJwOeXJQct+UbkI8JPYvGZHNa2psJZ5MGT
AF+pd5pqvCs77X0gDRYuMZSy2y/UaledQBM7FCqcF78jbEYm4JVY5/gocYvOWOxM
NuRQc/5mV7324bV9EUGweBJHIhHWyR+v/jtS4rDRHdslmRkyfOcL4jw=
-----END RSA PRIVATE KEY-----
```



**2.把生成的公钥放在Gitlab中**
以root账户登录->点击头像->Settings->SSH Keys
复制刚才id_rsa.pub文件的内容到这里，点击"Add Key"  

![](/images/devops/jenkins/jenkins/jenkins-35.png)



**3.在Jenkins中添加凭证，配置私钥**
在Jenkins添加一个新的凭证，类型为"SSH Username with private key"，把刚才生成私有文件内容复制过来  

![](/images/devops/jenkins/jenkins/jenkins-36.png)

![](/images/devops/jenkins/jenkins/jenkins-37.png)

![](/images/devops/jenkins/jenkins/jenkins-38.png)

![](/images/devops/jenkins/jenkins/jenkins-39.png)



**4.测试凭证**

构建项目成功。



#### 2.4.凭据管理

![](/images/devops/jenkins/jenkins/jenkins-99.png)

![](/images/devops/jenkins/jenkins/jenkins-100.png)

![](/images/devops/jenkins/jenkins/jenkins-101.png)

![](/images/devops/jenkins/jenkins/jenkins-102.png)



### 3.安装Git插件和Git工具  

#### 3.1.安装Git

CentOS7上安装Git工具：  

```shell
# yum install git -y
# git --version
```



#### 3.2.安装Git插件

![](/images/devops/jenkins/jenkins/jenkins-23.png)



### 4.Maven安装和配置  

在Jenkins集成服务器上，我们需要安装Maven来编译和打包项目。  

#### 4.1.安装Maven
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



#### 4.2.配置环境变量

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



#### 4.3.全局配置JDK和Maven   

![](/images/devops/jenkins/jenkins/jenkins-40.png)

**Jenkins->Global Tool Configuration->JDK->新增JDK，配置如下：** 

**/usr/lib/jvm/java-1.8.0-openjdk**

![](/images/devops/jenkins/jenkins/jenkins-41.png)



**Jenkins->Global Tool Configuration->Maven->新增Maven，配置如下：**  

**/opt/maven**

![](/images/devops/jenkins/jenkins/jenkins-42.png)



#### 4.4.添加Jenkins全局变量  

![](/images/devops/jenkins/jenkins/jenkins-43.png)



**Manage Jenkins->Configure System->Global Properties ，添加三个全局变量**
**JAVA_HOME、M2_HOME、PATH+EXTRA**  

![](/images/devops/jenkins/jenkins/jenkins-44.png)

```shell
JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
M2_HOME=/opt/maven
PATH+EXTRA=$M2_HOME/bin
```

![](/images/devops/jenkins/jenkins/jenkins-45.png)

![](/images/devops/jenkins/jenkins/jenkins-46.png)



#### 4.5.修改Maven的settings.xml  

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

![](/images/devops/jenkins/jenkins/jenkins-47.png)

![](/images/devops/jenkins/jenkins/jenkins-48.png)



#### 4.6.测试Maven配置  

使用之前的gitlab密码测试项目，修改配置  

![](/images/devops/jenkins/jenkins/jenkins-49.png)

![](/images/devops/jenkins/jenkins/jenkins-50.png)

**输入：mvn clean package**  

![](/images/devops/jenkins/jenkins/jenkins-51.png)



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

![](/images/devops/jenkins/jenkins/jenkins-52.png)



### 5.Jenkins构建Maven项目  

#### 5.1.Jenkins构建的项目类型介绍  

**Jenkins中自动构建项目的类型有很多，常用的有以下三种：**

- 自由风格软件项目（FreeStyle Project）
- Maven项目（Maven Project）
- 流水线项目（Pipeline Project）

**每种类型的构建其实都可以完成一样的构建过程与结果，只是在操作方式、灵活度等方面有所区别，在实际开发中可以根据自己的需求和习惯来选择。**

**（PS：个人推荐使用流水线类型，因为灵活度非常高）**  



#### 5.2.自由风格项目构建

**1.创建项目**

![](/images/devops/jenkins/jenkins/jenkins-53.png)



**2.配置源码管理，从gitlab拉取代码**  

![](/images/devops/jenkins/jenkins/jenkins-54.png)



**3.编译打包**
构建->添加构建步骤->Executor Shell  

```shell
echo "开始编译和打包"
mvn clean package
echo "编译和打包结束"
```

![](/images/devops/jenkins/jenkins/jenkins-55.png)



**4.部署**

构建项目



#### 5.3.Maven项目构建

**1.安装Maven Integration插件**  

![](/images/devops/jenkins/jenkins/jenkins-56.png)



**2.创建项目**  

![](/images/devops/jenkins/jenkins/jenkins-57.png)



**3.配置源码管理，从gitlab拉取代码**  

![](/images/devops/jenkins/jenkins/jenkins-54.png)



**4.构建项目**
![](/images/devops/jenkins/jenkins/jenkins-58.png)



**5.部署**

构建项目



#### 5.4.Pipeline流水线项目构建 

**1.安装Pipeline插件**  

![](/images/devops/jenkins/jenkins/jenkins-59.png)



**2.创建项目**  

![](/images/devops/jenkins/jenkins/jenkins-60.png)



**3.构建项目**

![](/images/devops/jenkins/jenkins/jenkins-61.png)

流水线语法：

http://182.92.210.65:18888/job/web-demo-pipeline/pipeline-syntax/



**4.部署**

构建项目



### 6.常用构建触发器

**Jenkins内置4种构建触发器：**  

- 触发远程构建
- 其他工程构建后触发（Build after other projects are build）
- 定时构建（Build periodically）
- 轮询SCM（Poll SCM）  

![](/images/devops/jenkins/jenkins/jenkins-69.png)



#### 6.1.触发远程构建

![](/images/devops/jenkins/jenkins/jenkins-70.png)

```shell
Use the following URL to trigger build remotely: 
JENKINS_URL/job/itcast01/build?token=TOKEN_NAME 或者 /buildWithParameters?token=TOKEN_NAME


# itcast01触发构建url
http://182.92.210.65:18888/job/itcast01/build?token=666666
```



#### 6.2.其他工程构建后触发

![](/images/devops/jenkins/jenkins/jenkins-71.png)



#### 6.3.定时构建

**定时字符串从左往右分别为： 分 时 日 月 周**  

**一些定时表达式的例子：**

- 每30分钟构建一次：H代表形参 H/30 * * * * 10:02 10:32
- 每2个小时构建一次: H H/2 * * *
- 每天的8点，12点，22点，一天构建3次： (多个时间点中间用逗号隔开) 0 8,12,22 * * *
- 每天中午12点定时构建一次 H 12 * * *
- 每天下午18点定时构建一次 H 18 * * *
- 在每个小时的前半个小时内的每10分钟 H(0-29)/10 * * * *
- 每两小时一次，每个工作日上午9点到下午5点(也许是上午10:38，下午12:38，下午2:38，下午
  4:38) H H(9-16)/2 * * 1-5  

![](/images/devops/jenkins/jenkins/jenkins-72.png)



#### 6.4.轮询SCM

轮询SCM，是指定时扫描本地代码仓库的代码是否有变更，如果代码有变更就触发项目构建。  

注意：这个构建触发器，Jenkins会定时扫描本地整个项目的代码，增大系统的开销，不建议使用。  

![](/images/devops/jenkins/jenkins/jenkins-73.png)



### 7.Git hook自动触发构建

Jenkins的内置构建触发器中，轮询SCM可以实现Gitlab代码更新，项目自动构建，但是该方案的性能不佳。那有没有更好的方案呢？ 有的。就是利用Gitlab的webhook实现代码push到仓库，立即触发项目自动构建。  

![](/images/devops/jenkins/jenkins/jenkins-74.png)



#### 7.1.安装Gitlab Hook插件

需要安装两个插件：Gitlab Hook和GitLab  

![](/images/devops/jenkins/jenkins/jenkins-75.png)

![](/images/devops/jenkins/jenkins/jenkins-76.png)



#### 7.2.Jenkins设置自动构建

![](/images/devops/jenkins/jenkins/jenkins-77.png)

等会需要把生成的webhook URL配置到Gitlab中  

```shell
# webhook URL

http://182.92.210.65:18888/project/itcast01
```



#### 7.3.Gitlab配置webhook

**1.开启webhook功能**
使用root账户登录到后台，点击Admin Area -> Settings -> Network
勾选"Allow requests to the local network from web hooks and services"  

![](/images/devops/jenkins/jenkins/jenkins-78.png)



**2.在项目添加webhook**
点击项目->Settings->Integrations  

![](/images/devops/jenkins/jenkins/jenkins-79.png)



注意：以下设置必须完成，否则会报错！
Manage Jenkins->Configure System  

![](/images/devops/jenkins/jenkins/jenkins-80.png)

**3.测试**

![](/images/devops/jenkins/jenkins/jenkins-81.png)

![](/images/devops/jenkins/jenkins/jenkins-82.png)



### 8.Jenkins的参数化构建

有时在项目构建的过程中，我们需要根据用户的输入动态传入一些参数，从而影响整个构建结果，这时我们可以使用参数化构建。
Jenkins支持非常丰富的参数类型  

![](/images/devops/jenkins/jenkins/jenkins-83.png)

![](/images/devops/jenkins/jenkins/jenkins-84.png)



#### 8.1.字符串型参数

![](/images/devops/jenkins/jenkins/jenkins-85.png)

![](/images/devops/jenkins/jenkins/jenkins-86.png)

![](/images/devops/jenkins/jenkins/jenkins-87.png)



```shell
pipeline {
    agent any

    stages {
        stage('pull code') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/${branch}']], extensions: [], userRemoteConfigs: [[credentialsId: '53b412d5-ecd1-47a3-b571-4505d562634e', url: 'git@39.96.178.134:test-group/web-demo.git']]])
            }
        }
        stage('build project') {
            steps {
                sh "mvn clean package dockerfile:build"

                sh "docker login -u hollysyshs -p 1qaz2wsx  registry.cn-beijing.aliyuncs.com"

                sh "docker tag web-demo:1.1.0 registry.cn-beijing.aliyuncs.com/devops-hollysys/web-demo:1.0.1"

                sh "docker push registry.cn-beijing.aliyuncs.com/devops-hollysys/web-demo:1.0.1"
            }
        }
        stage('publish project') {
            steps {
                echo 'publish project'
            }
        }
    }
}
```



![](/images/devops/jenkins/jenkins/jenkins-88.png)



#### 8.2.选项参数

![](/images/devops/jenkins/jenkins/jenkins-106.png)

![](/images/devops/jenkins/jenkins/jenkins-107.png)

![](/images/devops/jenkins/jenkins/jenkins-108.png)



#### 8.3.多选参数

**1.安装Extended Choice Parameter插件**  

![](/images/devops/jenkins/jenkins/jenkins-109.png)



**2.配置**

![](/images/devops/jenkins/jenkins/jenkins-110.png)

![](/images/devops/jenkins/jenkins/jenkins-111.png)

![](/images/devops/jenkins/jenkins/jenkins-112.png)

![](/images/devops/jenkins/jenkins/jenkins-113.png)



#### 8.4.Git分支标签选择参数

**1.安装插件**

![](/images/devops/jenkins/jenkins/jenkins-114.png)

**2.创建项目**

![](/images/devops/jenkins/jenkins/jenkins-119.png)

![](/images/devops/jenkins/jenkins/jenkins-115.png)

![](/images/devops/jenkins/jenkins/jenkins-116.png)

**3.构建项目**

![](/images/devops/jenkins/jenkins/jenkins-117.png)





### 9.配置邮箱服务器发送  

#### 9.1.安装Email Extension插件

![](/images/devops/jenkins/jenkins/jenkins-89.png)



**email.html**

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>${ENV, var="JOB_NAME"}-第${BUILD_NUMBER}次构建日志</title>
</head>

<body leftmargin="8" marginwidth="0" topmargin="8" marginheight="4"
      offset="0">
<table width="95%" cellpadding="0" cellspacing="0"
       style="font-size: 11pt; font-family: Tahoma, Arial, Helvetica, sans-serif">
    <tr>
        <td>(本邮件是程序自动下发的，请勿回复！)</td>
    </tr>
    <tr>
        <td><h2>
            <font color="#0000FF">构建结果 - ${BUILD_STATUS}</font>
        </h2></td>
    </tr>
    <tr>
        <td><br />
            <b><font color="#0B610B">构建信息</font></b>
            <hr size="2" width="100%" align="center" /></td>
    </tr>
    <tr>
        <td>
            <ul>
                <li>项目名称&nbsp;：&nbsp;${PROJECT_NAME}</li>
                <li>构建编号&nbsp;：&nbsp;第${BUILD_NUMBER}次构建</li>
                <li>触发原因：&nbsp;${CAUSE}</li>
                <li>构建日志：&nbsp;<a href="${BUILD_URL}console">${BUILD_URL}console</a></li>
                <li>构建&nbsp;&nbsp;Url&nbsp;：&nbsp;<a href="${BUILD_URL}">${BUILD_URL}</a></li>
                <li>工作目录&nbsp;：&nbsp;<a href="${PROJECT_URL}ws">${PROJECT_URL}ws</a></li>
                <li>项目&nbsp;&nbsp;Url&nbsp;：&nbsp;<a href="${PROJECT_URL}">${PROJECT_URL}</a></li>
            </ul>
        </td>
    </tr>
    <tr>
        <td><b><font color="#0B610B">Changes Since Last
            Successful Build:</font></b>
            <hr size="2" width="100%" align="center" /></td>
    </tr>
    <tr>
        <td>
            <ul>
                <li>历史变更记录 : <a href="${PROJECT_URL}changes">${PROJECT_URL}changes</a></li>
            </ul> ${CHANGES_SINCE_LAST_SUCCESS,reverse=true, format="Changes for Build #%n:<br />%c<br />",showPaths=true,changesFormat="<pre>[%a]<br />%m</pre>",pathFormat="&nbsp;&nbsp;&nbsp;&nbsp;%p"}
        </td>
    </tr>
    <tr>
        <td><b>Failed Test Results</b>
            <hr size="2" width="100%" align="center" /></td>
    </tr>
    <tr>
        <td><pre
                style="font-size: 11pt; font-family: Tahoma, Arial, Helvetica, sans-serif">$FAILED_TESTS</pre>
            <br /></td>
    </tr>
    <tr>
        <td><b><font color="#0B610B">构建日志 (最后 100行):</font></b>
            <hr size="2" width="100%" align="center" /></td>
    </tr>
    <tr>
        <td><textarea cols="80" rows="30" readonly="readonly"
                      style="font-family: Courier New">${BUILD_LOG, maxLines=100}</textarea>
        </td>
    </tr>
</table>
</body>
</html>
```





## 三、实践



### 1.创建WEB项目

#### 1.1.创建工程

```shell
# 名称：web-demo
# 端口：19999
# 接口地址： http://localhost:19999/hello
```

```java
# 源码

@Slf4j
@RestController
public class HelloController {


    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    public String hello() {

        // http://localhost:19999/hello
        String msg = "hello，this is first messge! ### -----v1.0.0.0" + new Date();
        log.info(msg);
        return msg;
    }

}
```



#### 1.2.上传GitLab仓库

**1.GitLab创建项目**

![](/images/devops/jenkins/jenkins/jenkins-24.png)

```shell
# 克隆地址

git@39.96.178.134:test-group/web-demo.git
http://39.96.178.134:18080/test-group/web-demo.git
```



**2.上传项目**

![](/images/devops/jenkins/jenkins/jenkins-25.png)



### 2.Web项目发布

#### 2.1.发布说明

```shell
# 服务器：82.157.166.86
# 以Jar包的形式部署服务
```



#### 2.2.安装SSH 插件

##### 2.2.1.Publish Over SSH 插件

Publish Over SSH有安全漏洞，暂时无法安装

https://updates.jenkins-ci.org/download/plugins/



##### 2.2.2.安装SSH插件

![](/images/devops/jenkins/jenkins/jenkins-62.png)



##### 2.2.3.配置远程部署服务器

![](/images/devops/jenkins/jenkins/jenkins-63.png)



**1.添加凭证**

![](/images/devops/jenkins/jenkins/jenkins-64.png)

![](/images/devops/jenkins/jenkins/jenkins-65.png)



**2.配置远程服务器**

![](/images/devops/jenkins/jenkins/jenkins-66.png)



##### 2.2.4.测试远程发布

**1.创建自由风格项目 test-ssh**

![](/images/devops/jenkins/jenkins/jenkins-67.png)

![](/images/devops/jenkins/jenkins/jenkins-68.png)



```shell
# 应用服务
IP：82.157.166.86


# 脚本deploy.sh
[root@tencent-22 ~]# vim deploy.sh

#! /bin/sh

echo "Begin pull image ---"

docker pull nginx

echo "End pull image ---"
```



**2.构建项目**

```shell
# 构建成功，服务器成功拉取镜像

[root@tencent-22 ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        latest    605c77e624dd   2 weeks ago   141MB
```



#### 2.3.应用服务器安装JDK

```shell
# yum install java-1.8.0-openjdk* -y
# java -version
```



#### 2.4.发布应用

暂时不做.



### 3.WEB项目持续集成（CI）

#### 3.1.创建流水线项目

![](/images/devops/jenkins/jenkins/jenkins-90.png)

![](/images/devops/jenkins/jenkins/jenkins-91.png)

![](/images/devops/jenkins/jenkins/jenkins-92.png)



#### 3.2.项目流水线配置

**1.Jenkinsfile**

```groovy
pipeline {
    agent any

    stages {
        stage('pull code') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: '53b412d5-ecd1-47a3-b571-4505d562634e', url: 'git@39.96.178.134:test-group/web-demo.git']]])
            }
        }
      stage('code checking') {
         steps {

            script {
                 //引入SonarQubeScanner工具
                scannerHome = tool 'SonarQube Scanner'
            }
            //引入SonarQube的服务器环境
            withSonarQubeEnv('SonarQube7.6') {
                sh "${scannerHome}/bin/sonar-scanner"
            }
         }
      }
        stage('build project') {
            steps {
                sh "mvn clean package dockerfile:build"

                sh "docker login -u hollysyshs -p 1qaz2wsx  registry.cn-beijing.aliyuncs.com"

                sh "docker tag web-demo:1.1.0 registry.cn-beijing.aliyuncs.com/devops-hollysys/web-demo:1.0.1"

                sh "docker push registry.cn-beijing.aliyuncs.com/devops-hollysys/web-demo:1.0.1"
            }
        }
        stage('publish project') {
            steps {
                echo 'publish project'
            }
        }
    }
}
```



**2.sonar-project.properties**  

```shell
# must be unique in a given SonarQube instance
sonar.projectKey=web-demo-ci

# this is the name and version displayed in the SonarQube UI. Was mandatoryprior to SonarQube 6.1.
sonar.projectName=web-demo-ci
sonar.projectVersion=1.0

# Path is relative to the sonar-project.properties file. Replace "\" by "/" on Windows.
# This property is optional if sonar.modules is set.
sonar.sources=.
sonar.exclusions=**/test/**,**/target/**
sonar.java.binaries=.

sonar.java.source=1.8
sonar.java.target=1.8

# Encoding of the source code. Default is default system encoding
sonar.sourceEncoding=UTF-8
```



#### 3.3.构建项目

![](/images/devops/jenkins/jenkins/jenkins-93.png)

![](/images/devops/jenkins/jenkins/jenkins-94.png)

![](/images/devops/jenkins/jenkins/jenkins-95.png)



### 4.WEB项目持续部署（CD）

#### 4.1.创建自由风格项目

![](/images/devops/jenkins/jenkins/jenkins-96.png)

![](/images/devops/jenkins/jenkins/jenkins-97.png)

![](/images/devops/jenkins/jenkins/jenkins-98.png)



```shell
echo "Login aliyun ---"
docker login -u hollysyshs -p 1qaz2wsx  registry.cn-beijing.aliyuncs.com

echo "Begin pull image ---"
docker pull registry.cn-beijing.aliyuncs.com/devops-hollysys/web-demo:1.0.0

echo "docker run ---"
docker run -d --network host --restart=always --name web-demo registry.cn-beijing.aliyuncs.com/devops-hollysys/web-demo:1.0.0

echo "end ---"
```

```shell
#! /bin/sh

echo "Login aliyun ---"
docker login -u hollysyshs -p 1qaz2wsx  registry.cn-beijing.aliyuncs.com

echo "Begin pull image ---"
docker pull registry.cn-beijing.aliyuncs.com/devops-hollysys/web-demo:1.0.0

echo "docker run ---"
docker run -d --network host --restart=always --name web-demo registry.cn-beijing.aliyuncs.com/devops-hollysys/web-demo:1.0.0

echo "end ---"
```



#### 4.2.构建项目

```shell
# 应用服务器分别查看


# 查看镜像
[root@tencent-22 ~]# docker images
REPOSITORY                                                                  TAG       IMAGE ID       CREATED             SIZE
registry.cn-beijing.aliyuncs.com/devops-hollysys/web-demo                   1.0.0     8720822a77a7   About an hour ago   123MB


# 查看容器
[root@tencent-22 ~]# docker ps -a
CONTAINER ID   IMAGE                                                             COMMAND                  CREATED              STATUS              PORTS     NAMES
7b05e2198a52   registry.cn-beijing.aliyuncs.com/devops-hollysys/web-demo:1.0.0   "/bin/sh -c 'java -D…"   About a minute ago   Up About a minute             web-demo


# 调用接口
[root@tencent-22 ~]# curl http://localhost:19999/hello
hello，this is first messge! ### -----v1.1.0.0Mon Jan 17 22:13:25 CST 2022
```



