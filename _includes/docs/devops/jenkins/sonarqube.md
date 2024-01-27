* TOC
{:toc}



## 一、概述



### 1.SonarQube简介  

SonarQube是一个用于管理代码质量的开放平台，可以快速的定位代码中潜在的或者明显的错误。目前支持java,C#,C/C++,Python,PL/SQL,Cobol,JavaScrip,Groovy等二十几种编程语言的代码质量管理与检测。

```shell
# 官网：

https://www.sonarqube.org/
```



### 2.SonarQube代码审查

![](/images/devops/jenkins/sonarqube/sonarqube-1.png)



## 二、基础



### 1.安装SonarQube Scanner插件

![](/images/devops/jenkins/sonarqube/sonarqube-2.png)



### 2.添加SonarQube凭证

![](/images/devops/jenkins/sonarqube/sonarqube-3.png)

![](/images/devops/jenkins/sonarqube/sonarqube-4.png)

![](/images/devops/jenkins/sonarqube/sonarqube-5.png)

![](/images/devops/jenkins/sonarqube/sonarqube-6.png)

![](/images/devops/jenkins/sonarqube/sonarqube-7.png)

![](/images/devops/jenkins/sonarqube/sonarqube-8.png)



### 3.Jenkins进行SonarQube配置  

Manage Jenkins->Configure System->SonarQube servers  

![](/images/devops/jenkins/sonarqube/sonarqube-9.png)

Manage Jenkins->Global Tool Configuration  

![](/images/devops/jenkins/sonarqube/sonarqube-10.png)

![](/images/devops/jenkins/sonarqube/sonarqube-11.png)



### 4.SonarQube关闭审查结果上传到SCM功能

![](/images/devops/jenkins/sonarqube/sonarqube-12.png)





## 三、实践



### 1.非流水线项目代码审查

#### 1.1.创建自由风格项目

![](/images/devops/jenkins/sonarqube/sonarqube-13.png)

![](/images/devops/jenkins/sonarqube/sonarqube-14.png)

![](/images/devops/jenkins/sonarqube/sonarqube-15.png)

![](/images/devops/jenkins/sonarqube/sonarqube-16.png)

![](/images/devops/jenkins/sonarqube/sonarqube-17.png)



```shell
# must be unique in a given SonarQube instance
sonar.projectKey=sonarqube-freestyle

# this is the name and version displayed in the SonarQube UI. Was mandatoryprior to SonarQube 6.1.
sonar.projectName=sonarqube-freestyle
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



#### 1.2.构建项目

```shell
# 问题

ERROR: Error during SonarScanner execution
org.sonar.java.AnalysisException: Please provide compiled classes of your project with sonar.java.binaries property


# 官方文档解释sonarqube的sonar-java插件从4.1.2开始，强制要求sonar.java.binaries参数

https://www.cnblogs.com/lina-2159/p/13471460.html
https://www.jianshu.com/p/777227e316a2
```



#### 1.3.测试

![](/images/devops/jenkins/sonarqube/sonarqube-18.png)

![](/images/devops/jenkins/sonarqube/sonarqube-19.png)



### 2.流水线项目代码审查

#### 2.1.创建流水线项目

![](/images/devops/jenkins/sonarqube/sonarqube-20.png)

![](/images/devops/jenkins/sonarqube/sonarqube-21.png)

![](/images/devops/jenkins/sonarqube/sonarqube-22.png)



#### 2.2.修改项目源码

**1.项目根目录下，创建sonar-project.properties文件**  

```shell
# must be unique in a given SonarQube instance
sonar.projectKey=sonarqube-pipeline

# this is the name and version displayed in the SonarQube UI. Was mandatoryprior to SonarQube 6.1.
sonar.projectName=sonarqube-pipeline
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



**2.修改Jenkinsfile，加入SonarQube代码审查阶段**  

**Jenkinsfile**

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



#### 2.3.测试

![](/images/devops/jenkins/sonarqube/sonarqube-23.png)

![](/images/devops/jenkins/sonarqube/sonarqube-24.png)



