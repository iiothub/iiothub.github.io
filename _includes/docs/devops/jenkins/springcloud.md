* TOC
{:toc}



## 一、概述



### 1.SpringCloud微服务源码

**1.微服务项目结构**

![](/images/devops/jenkins/springcloud/sc-1.png)



**2.持续集成微服务**

```xml
<!--  子模块  -->
<modules>
    <module>msa-eureka</module>
    <module>msa-gateway</module>
    <module>msa-deploy-producer</module>
    <module>msa-deploy-consumer</module>
</modules>
```



### 2.GitLab管理源码

**1.GitLab平台创建项目**

![](/images/devops/jenkins/springcloud/sc-2.png)

**2.微服务源码上传GitLab仓库**

![](/images/devops/jenkins/springcloud/sc-3.png)

```shell
# 仓库地址

git@39.96.178.134:test-group/k8s.git
```





## 二、基础



### 1.创建持续集成项目

![](/images/devops/jenkins/springcloud/sc-4.png)

![](/images/devops/jenkins/springcloud/sc-5.png)

![](/images/devops/jenkins/springcloud/sc-6.png)



### 2.从Gitlab拉取项目源码

**1.拉取Git代码**

![](/images/devops/jenkins/springcloud/sc-7.png)

```shell
# 拉取Git代码

checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: '53b412d5-ecd1-47a3-b571-4505d562634e', url: 'git@39.96.178.134:test-group/k8s.git']]])
```



**2.创建Jenkinsfile文件**  

在工程根目录创建Jenkinsfile文件，并上传GitLab仓库。

**Jenkinsfile**

```groovy
pipeline {
    agent any

    stages {
        stage('拉取源码') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: '53b412d5-ecd1-47a3-b571-4505d562634e', url: 'git@39.96.178.134:test-group/k8s.git']]])
            }
        }
    }
}
```

**重构代码**

```groovy

//gitlab的凭证
def git_auth = "68f2087f-a034-4d39-a9ff-1f776dd3dfa8"


pipeline {
    agent any

    stages {
        stage('拉取源码') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: "${git_auth}", url: 'git@39.96.178.134:test-group/k8s.git']]])
            }
        }
    }
}
```



**3.构建项目**

![](/images/devops/jenkins/springcloud/sc-8.png)



**查看/var/lib/jenkins/workspace/目录，发现已经从Gitlab成功拉取了代码到Jenkins中。**  

```shell
[root@aliyun-8g ~]# cd /var/lib/jenkins/workspace

[root@aliyun-8g workspace]# ll
total 8
drwxr-xr-x 26 root root 4096 Jan 19 17:01 msa-pipeline
drwxr-xr-x  2 root root 4096 Jan 19 17:01 msa-pipeline@tmp

[root@aliyun-8g workspace]# cd msa-pipeline
[root@aliyun-8g msa-pipeline]# ll
total 104
-rw-r--r-- 1 root root  719 Jan 19 17:01 Dockerfile
-rw-r--r-- 1 root root  338 Jan 19 17:01 Jenkinsfile
drwxr-xr-x 3 root root 4096 Jan 19 17:01 msa-admin
drwxr-xr-x 3 root root 4096 Jan 19 17:01 msa-admin-client
drwxr-xr-x 3 root root 4096 Jan 19 17:01 msa-consumer
drwxr-xr-x 3 root root 4096 Jan 19 17:01 msa-deploy-consumer
drwxr-xr-x 3 root root 4096 Jan 19 17:01 msa-deploy-job
drwxr-xr-x 3 root root 4096 Jan 19 17:01 msa-deploy-producer
drwxr-xr-x 3 root root 4096 Jan 19 17:01 msa-eureka
drwxr-xr-x 3 root root 4096 Jan 19 17:01 msa-eureka-client
drwxr-xr-x 3 root root 4096 Jan 19 17:01 msa-ext-elk
drwxr-xr-x 3 root root 4096 Jan 19 17:01 msa-ext-job
drwxr-xr-x 3 root root 4096 Jan 19 17:01 msa-ext-mysql
drwxr-xr-x 3 root root 4096 Jan 19 17:01 msa-ext-postgresql
drwxr-xr-x 3 root root 4096 Jan 19 17:01 msa-ext-prometheus
drwxr-xr-x 3 root root 4096 Jan 19 17:01 msa-ext-rabbitmq
drwxr-xr-x 3 root root 4096 Jan 19 17:01 msa-ext-redis
drwxr-xr-x 3 root root 4096 Jan 19 17:01 msa-gateway
drwxr-xr-x 3 root root 4096 Jan 19 17:01 msa-k8s-rabbitmq
drwxr-xr-x 3 root root 4096 Jan 19 17:01 msa-k8s-redis
drwxr-xr-x 3 root root 4096 Jan 19 17:01 msa-producer
drwxr-xr-x 3 root root 4096 Jan 19 17:01 msa-sentinel-consumer
drwxr-xr-x 3 root root 4096 Jan 19 17:01 msa-sentinel-producer
drwxr-xr-x 3 root root 4096 Jan 19 17:01 msa-zipkin-consumer
drwxr-xr-x 3 root root 4096 Jan 19 17:01 msa-zipkin-producer
-rw-r--r-- 1 root root 3406 Jan 19 17:01 pom.xml
```



### 3.SonarQube代码审查  

**1.设置参数**  

![](/images/devops/jenkins/springcloud/sc-9.png)

![](/images/devops/jenkins/springcloud/sc-10.png)

![](/images/devops/jenkins/springcloud/sc-11.png)



**2.创建sonar-project.properties**

每个项目的根目录下添加sonar-project.properties文件，上传GitLab仓库。

```xml
<!--  子模块  -->
<modules>
    <module>msa-eureka</module>
    <module>msa-gateway</module>
    <module>msa-deploy-producer</module>
    <module>msa-deploy-consumer</module>
</modules>
```



**sonar-project.properties**  

```properties
# must be unique in a given SonarQube instance
sonar.projectKey=msa-eureka
# this is the name and version displayed in the SonarQube UI. Was mandatory prior to SonarQube 6.1.
sonar.projectName=msa-eureka
sonar.projectVersion=1.0

# Path is relative to the sonar-project.properties file. Replace "\" by "/" on Windows.
# This property is optional if sonar.modules is set.
sonar.sources=.
sonar.exclusions=**/test/**,**/target/**
sonar.java.binaries=.

sonar.java.source=1.8
sonar.java.target=1.8
#sonar.java.libraries=**/target/classes/**

# Encoding of the source code. Default is default system encoding
sonar.sourceEncoding=UTF-8
```

**注意不同项目名称！**



**3.修改Jenkinsfile构建脚本**  

```groovy

//gitlab的凭证
def git_auth = "68f2087f-a034-4d39-a9ff-1f776dd3dfa8"


pipeline {
    agent any

    stages {

        stage('拉取源码') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: "${git_auth}", url: 'git@39.96.178.134:test-group/k8s.git']]])
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
                     sh """
                         cd ${project_name}
                         ${scannerHome}/bin/sonar-scanner
                     """
                }
            }
        }


    }
}
```



**4.构建项目**

![](/images/devops/jenkins/springcloud/sc-12.png)

![](/images/devops/jenkins/springcloud/sc-13.png)



### 4.生成Docker镜像

**使用Dockerfile编译、生成镜像，利用dockerfile-maven-plugin插件构建Docker镜像。**



**1.在每个微服务项目的pom.xml加入dockerfile-maven-plugin插件**  

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            
            
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>dockerfile-maven-plugin</artifactId>
                <version>1.3.6</version>
                <configuration>
                    <repository>${project.artifactId}</repository>
                    <tag>${project.version}</tag>
                    <buildArgs>
                        <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
                    </buildArgs>
                </configuration>
            </plugin>
            
            
        </plugins>
    </build>
```



**2.在每个微服务项目根目录下建立Dockerfile文件**  

**Dockerfile**

```dockerfile
FROM openjdk:8-jdk-alpine

VOLUME /temp

ENV JVM_OPS="-Xms256m -Xmx256m -XX:PermSize=512M -XX:MaxPermSize=512m"

ENV JAVA_POS=""

ARG JAR_FILE
COPY ${JAR_FILE} app.jar

ENTRYPOINT java -Djava.security.egd=file:/dev/./urandom -jar ${JVM_OPS} ${ACTIVE} app.jar ${JAVA_OPS}

ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```



**3.修改Jenkinsfile构建脚本**  

```groovy

//gitlab的凭证
def git_auth = "68f2087f-a034-4d39-a9ff-1f776dd3dfa8"
//构建版本的名称
def tag = "latest"
//定义镜像名称
def imageName = "${project_name}:${tag}"


pipeline {
    agent any

    stages {

        stage('拉取源码') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: "${git_auth}", url: 'git@39.96.178.134:test-group/k8s.git']]])
            }
        }

        stage('代码审查') {
            steps {
                script {
                    //引入SonarQubeScanner工具
                    scannerHome = tool 'SonarQube Scanner'
                }
                //引入SonarQube的服务器环境
                withSonarQubeEnv('SonarQube7.6') {
                     sh """
                         cd ${project_name}
                         ${scannerHome}/bin/sonar-scanner
                     """
                }
            }
        }

        stage('编译，安装公共子工程') {
            steps {
                 echo '编译，安装公共子工程'
                //编译，安装公共工程
                // sh "mvn -f tensquare_common clean install"
            }
        }

        stage('编译，构建微服务镜像') {
            steps {

                //编译，构建本地镜像
                sh "mvn -f ${project_name} clean package dockerfile:build"
            }
        }

    }
}
```



**4.构建项目**

```shell
# 查看生成的Docker镜像


[root@aliyun-8g ~]# docker images
REPOSITORY                                                  TAG               IMAGE ID       CREATED              SIZE
msa-deploy-consumer                                         2.0.0             549544669bf8   About a minute ago   164MB
msa-deploy-producer                                         2.0.0             4023de8c9994   4 minutes ago        163MB
msa-gateway                                                 2.0.0             8acade409a9e   7 minutes ago        152MB
msa-eureka                                                  2.0.0             f62911643776   9 minutes ago        155MB
```



### 5.Docker镜像上传阿里云仓库

**1.阿里云创建镜像仓库**

![](/images/devops/jenkins/springcloud/sc-14.png)

```shell
# 将镜像推送到Registry
$ docker login --username=hollys**** registry.cn-beijing.aliyuncs.com
$ docker tag [ImageId] registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-xxxx:[镜像版本号]
$ docker push registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-xxxx:[镜像版本号]



docker login -u xxxxxx -p ******  registry.cn-beijing.aliyuncs.com
docker tag web-demo:latest registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-eureka:2.0.0
docker push registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-eureka:2.0.0
```



**2.修改Jenkinsfile构建脚本**  

```groovy

//gitlab的凭证
def git_auth = "68f2087f-a034-4d39-a9ff-1f776dd3dfa8"

//构建版本的名称
def tag = "2.0.0"

//定义镜像名称
def imageName = "${project_name}:${tag}"

//阿里云镜像仓库地址
def aliyun_url = "registry.cn-beijing.aliyuncs.com"

//阿里云名字空间
def aliyun_namespace = "devops-hollysys"


pipeline {
    agent any

    stages {

        stage('拉取源码') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: "${git_auth}", url: 'git@39.96.178.134:test-group/k8s.git']]])
            }
        }

        stage('代码审查') {
            steps {
                script {
                    //引入SonarQubeScanner工具
                    scannerHome = tool 'SonarQube Scanner'
                }
                //引入SonarQube的服务器环境
                withSonarQubeEnv('SonarQube7.6') {
                     sh """
                         cd ${project_name}
                         ${scannerHome}/bin/sonar-scanner
                     """
                }
            }
        }

        stage('编译，安装公共子工程') {
            steps {
                 echo '编译，安装公共子工程'
                //编译，安装公共工程
                // sh "mvn -f tensquare_common clean install"
            }
        }

        stage('编译，构建微服务镜像') {
            steps {

                //编译，构建本地镜像
                sh "mvn -f ${project_name} clean package dockerfile:build"
            }
        }

        stage('上传镜像') {
            steps {
                //给镜像打标签
                sh "docker tag ${imageName} ${aliyun_url}/${aliyun_namespace}/${imageName}"

                //登录阿里云镜像仓库
                sh "docker login -u xxxxxx -p ******  ${aliyun_url}"

                //上传镜像
                sh "docker push ${aliyun_url}/${aliyun_namespace}/${imageName}"

                //删除本地镜像
                sh """
                    docker rmi ${imageName}
                    docker rmi ${aliyun_url}/${aliyun_namespace}/${imageName}
                """
            }
        }

    }
}
```



**脚本方式**

```groovy

//gitlab的凭证
def git_auth = "68f2087f-a034-4d39-a9ff-1f776dd3dfa8"

//构建版本的名称
def tag = "2.0.0"

//定义镜像名称
def imageName = "${project_name}:${tag}"

//阿里云镜像仓库地址
def aliyun_url = "registry.cn-beijing.aliyuncs.com"

//阿里云名字空间
def aliyun_namespace = "devops-hollysys"


node {

    stage('拉取源码') {
        checkout([$class: 'GitSCM', branches: [[name: '*/${branch}']], extensions: [], userRemoteConfigs: [[credentialsId: "${git_auth}", url: 'git@39.96.178.134:test-group/k8s.git']]])
    }

    stage('代码审查') {
        script {
            //引入SonarQubeScanner工具
            scannerHome = tool 'SonarQube Scanner'
        }
        //引入SonarQube的服务器环境
        withSonarQubeEnv('SonarQube7.6') {
            sh """
                cd ${project_name}
                ${scannerHome}/bin/sonar-scanner
            """
        }
    }

    stage('编译，安装公共子工程') {
        echo '编译，安装公共子工程'
        //编译，安装公共工程
        // sh "mvn -f tensquare_common clean install"
    }

    stage('编译，构建微服务镜像') {
        //编译，构建本地镜像
        sh "mvn -f ${project_name} clean package dockerfile:build"
    }

    stage('上传镜像') {
        //给镜像打标签
        sh "docker tag ${imageName} ${aliyun_url}/${aliyun_namespace}/${imageName}"

        //登录阿里云镜像仓库
        sh "docker login -u XXXXXX -p ******  ${aliyun_url}"

        //上传镜像
        sh "docker push ${aliyun_url}/${aliyun_namespace}/${imageName}"

        //删除本地镜像
        sh """
            docker rmi ${imageName}
            docker rmi ${aliyun_url}/${aliyun_namespace}/${imageName}
        """
    }

}
```



**3.构建项目**

成功上传阿里云镜像仓库。



### 6.部署微服务

**1.创建自由风格项目**

![](/images/devops/jenkins/springcloud/sc-15.png)

![](/images/devops/jenkins/springcloud/sc-36.png)

```shell
# 执行脚本

sh /root/deploy.sh
```



**2.应用服务器脚本**

```shell
# 应用服务
IP：82.157.166.86


# 脚本deploy.sh
[root@tencent-22 ~]# vim deploy.sh

#! /bin/sh

echo "登录阿里云 ---"
docker login -u XXXXXX -p ******  registry.cn-beijing.aliyuncs.com

echo "运行注册中心 ---"
docker pull registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-eureka:2.0.0
docker run -d --network host --restart=always --name msa-eureka registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-eureka:2.0.0

echo "运行网关 ---"
docker pull registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-gateway:2.0.0
docker run -d --network host --restart=always --name msa-gateway registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-gateway:2.0.0

echo "运行生产者 ---"
docker pull registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-deploy-producer:2.0.0
docker run -d --network host --restart=always --name msa-deploy-producer registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-deploy-producer:2.0.0

echo "运行消费者 ---"
docker pull registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-deploy-consumer:2.0.0
docker run -d --network host --restart=always --name msa-deploy-consumer registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-deploy-consumer:2.0.0

echo "完成 ---"
```



**3.构建项目**

```shell
# 镜像
[root@tencent-22 ~]# docker ps -a
CONTAINER ID   IMAGE                                                                        COMMAND                  CREATED              STATUS              PORTS     NAMES
b7bdd93d54e6   registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-deploy-consumer:2.0.0   "/bin/sh -c 'java -D…"   25 seconds ago       Up 24 seconds                 msa-deploy-consumer
c8c14ce23f36   registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-deploy-producer:2.0.0   "/bin/sh -c 'java -D…"   40 seconds ago       Up 40 seconds                 msa-deploy-producer
6a1e785dba7b   registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-gateway:2.0.0           "/bin/sh -c 'java -D…"   54 seconds ago       Up 54 seconds                 msa-gateway
3912483a0ada   registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-eureka:2.0.0            "/bin/sh -c 'java -D…"   About a minute ago   Up About a minute             msa-eureka


# 容器
[root@tencent-22 ~]# docker ps -a
CONTAINER ID   IMAGE                                                                        COMMAND                  CREATED              STATUS              PORTS     NAMES
b7bdd93d54e6   registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-deploy-consumer:2.0.0   "/bin/sh -c 'java -D…"   About a minute ago   Up About a minute             msa-deploy-consumer
c8c14ce23f36   registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-deploy-producer:2.0.0   "/bin/sh -c 'java -D…"   About a minute ago   Up About a minute             msa-deploy-producer
6a1e785dba7b   registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-gateway:2.0.0           "/bin/sh -c 'java -D…"   2 minutes ago        Up 2 minutes                  msa-gateway
3912483a0ada   registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-eureka:2.0.0            "/bin/sh -c 'java -D…"   2 minutes ago        Up 2 minutes                  msa-eureka


# 调用接口
[root@tencent-22 ~]# curl http://localhost:8911/hello
hello，this is first messge! ### -----V2Thu Jan 20 14:28:44 CST 2022

[root@tencent-22 ~]# curl http://localhost:8912/hello
hello，this is first messge! ### -----V2Thu Jan 20 14:29:14 CST 2022  openfeign + sentinel!!!


# 注册中心地址
http://82.157.166.86:10001/
```

![](/images/devops/jenkins/springcloud/sc-16.png)



4.部署脚本进行版本管理

```shell
# 脚本deploy.sh
[root@tencent-22 ~]# vim deploy.sh

#! /bin/sh

echo "克隆项目 ---"
rm -rf k8s
git clone git@39.96.178.134:test-group/k8s.git
cd k8s
chmod +x deploy.sh

echo "部署服务 ---"
./deploy.sh

echo "删除文件 ---"
cd ..
rm -rf k8s
```





## 三、实践



### 1.持续集成容器

#### 1.1.创建持续集成项目

![](/images/devops/jenkins/springcloud/sc-17.png)

![](/images/devops/jenkins/springcloud/sc-18.png)

![](/images/devops/jenkins/springcloud/sc-19.png)

![](/images/devops/jenkins/springcloud/sc-20.png)

![](/images/devops/jenkins/springcloud/sc-21.png)

![](/images/devops/jenkins/springcloud/sc-25.png)

![](/images/devops/jenkins/springcloud/sc-22.png)

![](/images/devops/jenkins/springcloud/sc-23.png)



#### 1.2.Jenkinsfile构建脚本  

**Jenkinsfile构建脚本**

```groovy
//gitlab的凭证
def git_auth = "68f2087f-a034-4d39-a9ff-1f776dd3dfa8"

//构建版本的名称
def build_tag = "latest"

//阿里云镜像仓库地址
def aliyun_url = "registry.cn-beijing.aliyuncs.com"

//阿里云名字空间
def aliyun_namespace = "devops-hollysys"


node {

    //获取当前选择的项目名称数组
    def selectedProjectNames = "${project_name}".split(",")


    stage('拉取源码') {
        checkout([$class: 'GitSCM', branches: [[name: '*/${branch}']], extensions: [], userRemoteConfigs: [[credentialsId: "${git_auth}", url: 'git@39.96.178.134:test-group/k8s.git']]])
    }

    stage('代码审查') {
        for(int i=0;i<selectedProjectNames.length;i++){
            //当前遍历的项目名称
            def currentProjectName = selectedProjectNames[i];

            //定义当前Jenkins的SonarQubeScanner工具
            def scannerHome = tool 'SonarQube Scanner'
            //引用当前JenkinsSonarQube环境
            withSonarQubeEnv('SonarQube7.6') {
                sh """
                    cd ${currentProjectName}
                    ${scannerHome}/bin/sonar-scanner
                """
            }
        }
    }

    stage('编译，安装公共子工程') {
        sh "echo 编译，安装公共子工程"

        //编译，安装公共工程
        // sh "mvn -f tensquare_common clean install"
    }

    stage('编译，构建镜像，上传镜像') {
        for(int i=0;i<selectedProjectNames.length;i++){
            //当前遍历的项目名称
            def currentProjectName = selectedProjectNames[i];

            //编译，构建本地镜像
            sh "mvn -f ${currentProjectName} clean package dockerfile:build"

            //本地镜像名称
            def buildImageName = "${currentProjectName}:${build_tag}"
            //推送镜像名称
            def pushImageName = "${currentProjectName}:${docker_tag}"

            //对镜像打上标签
            sh "docker tag ${buildImageName} ${aliyun_url}/${aliyun_namespace}/${pushImageName}"

            //登录阿里云镜像仓库
            sh "docker login -u xxxxxx -p ******  ${aliyun_url}"

            //上传镜像
            sh "docker push ${aliyun_url}/${aliyun_namespace}/${pushImageName}"

            //删除本地镜像
            sh """
                docker rmi ${buildImageName}
                docker rmi ${aliyun_url}/${aliyun_namespace}/${pushImageName}
            """
        }
    }

}
```



**单项目配置参考**

**脚本化流水线**

```groovy

//gitlab的凭证
def git_auth = "68f2087f-a034-4d39-a9ff-1f776dd3dfa8"

//构建版本的名称
def tag = "2.0.0"

//定义镜像名称
def imageName = "${project_name}:${tag}"

//阿里云镜像仓库地址
def aliyun_url = "registry.cn-beijing.aliyuncs.com"

//阿里云名字空间
def aliyun_namespace = "devops-hollysys"


node {

    stage('拉取源码') {
        checkout([$class: 'GitSCM', branches: [[name: '*/${branch}']], extensions: [], userRemoteConfigs: [[credentialsId: "${git_auth}", url: 'git@39.96.178.134:test-group/k8s.git']]])
    }

    stage('代码审查') {
        script {
            //引入SonarQubeScanner工具
            scannerHome = tool 'SonarQube Scanner'
        }
        //引入SonarQube的服务器环境
        withSonarQubeEnv('SonarQube7.6') {
            sh """
                cd ${project_name}
                ${scannerHome}/bin/sonar-scanner
            """
        }
    }

    stage('编译，安装公共子工程') {
        echo '编译，安装公共子工程'
        //编译，安装公共工程
        // sh "mvn -f tensquare_common clean install"
    }

    stage('编译，构建微服务镜像') {
        //编译，构建本地镜像
        sh "mvn -f ${project_name} clean package dockerfile:build"
    }

    stage('上传镜像') {
        //给镜像打标签
        sh "docker tag ${imageName} ${aliyun_url}/${aliyun_namespace}/${imageName}"

        //登录阿里云镜像仓库
        sh "docker login -u xxxxxx -p ******  ${aliyun_url}"

        //上传镜像
        sh "docker push ${aliyun_url}/${aliyun_namespace}/${imageName}"

        //删除本地镜像
        sh """
            docker rmi ${imageName}
            docker rmi ${aliyun_url}/${aliyun_namespace}/${imageName}
        """
    }

}
```

**声明式流水线**

```groovy

//gitlab的凭证
def git_auth = "68f2087f-a034-4d39-a9ff-1f776dd3dfa8"

//构建版本的名称
def tag = "2.0.0"

//定义镜像名称
def imageName = "${project_name}:${tag}"

//阿里云镜像仓库地址
def aliyun_url = "registry.cn-beijing.aliyuncs.com"

//阿里云名字空间
def aliyun_namespace = "devops-hollysys"


pipeline {
    agent any

    stages {

        stage('拉取源码') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: "${git_auth}", url: 'git@39.96.178.134:test-group/k8s.git']]])
            }
        }

        stage('代码审查') {
            steps {
                script {
                    //引入SonarQubeScanner工具
                    scannerHome = tool 'SonarQube Scanner'
                }
                //引入SonarQube的服务器环境
                withSonarQubeEnv('SonarQube7.6') {
                     sh """
                         cd ${project_name}
                         ${scannerHome}/bin/sonar-scanner
                     """
                }
            }
        }

        stage('编译，安装公共子工程') {
            steps {
                 echo '编译，安装公共子工程'
                //编译，安装公共工程
                // sh "mvn -f tensquare_common clean install"
            }
        }

        stage('编译，构建微服务镜像') {
            steps {

                //编译，构建本地镜像
                sh "mvn -f ${project_name} clean package dockerfile:build"
            }
        }

        stage('上传镜像') {
            steps {
                //给镜像打标签
                sh "docker tag ${imageName} ${aliyun_url}/${aliyun_namespace}/${imageName}"

                //登录阿里云镜像仓库
                sh "docker login -u xxxxxx -p ******  ${aliyun_url}"

                //上传镜像
                sh "docker push ${aliyun_url}/${aliyun_namespace}/${imageName}"

                //删除本地镜像
                sh """
                    docker rmi ${imageName}
                    docker rmi ${aliyun_url}/${aliyun_namespace}/${imageName}
                """
            }
        }

    }
}
```



#### 1.3.构建项目

镜像成功上传阿里云镜像仓库。

![](/images/devops/jenkins/springcloud/sc-24.png)



### 2.持续部署（Docker）

#### 2.1.GitLab创建部署项目

**1.GitLab创建项目**

![](/images/devops/jenkins/springcloud/sc-26.png)

**2.编写部署脚本**

```shell
# 克隆
# git clone git@39.96.178.134:test-group/k8s-deploy.git


Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/jenkins/springcloud
$ git clone git@39.96.178.134:test-group/k8s-deploy.git
Cloning into 'k8s-deploy'...
warning: You appear to have cloned an empty repository.

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/jenkins/springcloud
$ ll
total 0
drwxr-xr-x 1 Administrator 197121 0 Jan 23 16:32 k8s-deploy/
```

```shell
# 创建脚本


$ cd k8s-deploy/
$ mkdir docker
$ cd docker/

$ vim deploy.sh
```

```shell
# 脚本deploy.sh


#! /bin/sh

echo "登录阿里云 ---"
docker login -u xxxxxx -p ******  registry.cn-beijing.aliyuncs.com

echo "运行注册中心 ---"
docker pull registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-eureka:2.0.0
docker run -d --network host --restart=always --name msa-eureka registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-eureka:2.0.0

echo "运行网关 ---"
docker pull registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-gateway:2.0.0
docker run -d --network host --restart=always --name msa-gateway registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-gateway:2.0.0

echo "运行生产者 ---"
docker pull registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-deploy-producer:2.0.0
docker run -d --network host --restart=always --name msa-deploy-producer registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-deploy-producer:2.0.0

echo "运行消费者 ---"
docker pull registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-deploy-consumer:2.0.0
docker run -d --network host --restart=always --name msa-deploy-consumer registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-deploy-consumer:2.0.0

echo "完成 ---"
```



**3.上传GitLab仓库**

```sh
$ git add .
$ git commit -m "创建部署脚本"

# 推送远程仓库
$ git push origin master
```

![](/images/devops/jenkins/springcloud/sc-27.png)



#### 2.2.Jenkins创建持续部署项目

![](/images/devops/jenkins/springcloud/sc-28.png)

![](/images/devops/jenkins/springcloud/sc-29.png)

![](/images/devops/jenkins/springcloud/sc-30.png)



**执行脚本**

```shell
echo "克隆项目 ---"
rm -rf k8s-deploy
git clone git@39.96.178.134:test-group/k8s-deploy.git

echo "部署服务 ---"
cd k8s-deploy/docker
chmod +x deploy.sh
./deploy.sh

echo "删除文件 ---"
cd ..
cd ..
rm -rf k8s-deploy
```



#### 2.3.构建项目

```shell
# 镜像
[root@tencent-22 ~]# docker images
REPOSITORY                                            TAG             IMAGE ID       CREATED         SIZE
registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-deploy-consumer   2.0.0           f53258c0a16e   23 hours ago    164MB
registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-deploy-producer   2.0.0           b92630f3bbd2   23 hours ago    163MB
registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-gateway           2.0.0           ce4c7c546da7   23 hours ago    152MB
registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-eureka            2.0.0           841d2d7f323b   23 hours ago    155MB


# 容器
[root@tencent-22 ~]# docker ps -a
CONTAINER ID   IMAGE                                                                        COMMAND                  CREATED              STATUS              PORTS     NAMES
b7bdd93d54e6   registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-deploy-consumer:2.0.0   "/bin/sh -c 'java -D…"   About a minute ago   Up About a minute             msa-deploy-consumer
c8c14ce23f36   registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-deploy-producer:2.0.0   "/bin/sh -c 'java -D…"   About a minute ago   Up About a minute             msa-deploy-producer
6a1e785dba7b   registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-gateway:2.0.0           "/bin/sh -c 'java -D…"   2 minutes ago        Up 2 minutes                  msa-gateway
3912483a0ada   registry.cn-beijing.aliyuncs.com/devops-hollysys/msa-eureka:2.0.0            "/bin/sh -c 'java -D…"   2 minutes ago        Up 2 minutes                  msa-eureka


# 调用接口
[root@tencent-22 ~]# curl http://localhost:8911/hello
hello，this is first messge! ### -----V2Thu Jan 20 14:28:44 CST 2022

[root@tencent-22 ~]# curl http://localhost:8912/hello
hello，this is first messge! ### -----V2Thu Jan 20 14:29:14 CST 2022  openfeign + sentinel!!!


# 注册中心地址
http://82.157.166.86:10001/
```

![](/images/devops/jenkins/springcloud/sc-16.png)



### 3.持续部署（K8S）

#### 3.1.Helm构建微服务

##### 3.1.1.创建Helm Chart

```shell
# 创建 msa
[root@k8s-master devops]# helm create cloud -n devops
Creating cloud


[root@k8s-master devops]# ll
drwxr-xr-x 4 root root 93 Jan 23 17:38 cloud



[root@k8s-master devops]# tree
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
# templates创建配置文件

[root@k8s-master devops]# tree
.
└── cloud
    ├── charts
    ├── Chart.yaml
    ├── templates
    │   ├── msa-deploy-consumer.yaml
    │   ├── msa-deploy-producer.yaml
    │   ├── msa-eureka.yaml
    │   └── msa-gateway.yaml
    └── values.yaml

3 directories, 6 files
```

```shell
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



##### 3.1.2.注册中心（eureka-server）

 msa-eureka.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.eureka.name }}
  labels:
    app: {{ .Values.eureka.labels }}
spec:
  type: {{ .Values.eureka.service.type }}
  ports:
    - port: {{ .Values.eureka.service.port }}
      name: {{ .Values.eureka.name }}
      targetPort: {{ .Values.eureka.service.targetPort }}
      nodePort: {{ .Values.eureka.service.nodePort }} #对外暴露30011端口
  selector:
    app: {{ .Values.eureka.labels }}

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.eureka.name }}
spec:
  serviceName: {{ .Values.eureka.name }}
  replicas: {{ .Values.eureka.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.eureka.labels }}
  template:
    metadata:
      labels:
        app: {{ .Values.eureka.labels }}
    spec:
      imagePullSecrets:
        - name: {{ .Values.global.imagePullSecrets }}   #对应创建私有镜像密钥Secret
      containers:
        - name: {{ .Values.eureka.image.name }}
          imagePullPolicy: {{ .Values.eureka.image.pullPolicy }}
          image: {{ .Values.global.repository }}{{ .Values.eureka.image.name }}:{{ .Values.eureka.image.tag }}
          ports:
          - containerPort: {{ .Values.eureka.image.containerPort }}
          env:
            - name: MY_POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: EUREKA_SERVER
              value: {{ .Values.global.env.eureka }}
            - name: EUREKA_INSTANCE_HOSTNAME
              value: ${MY_POD_NAME}.msa-eureka.dev
            - name: ACTIVE
              value: {{ .Values.eureka.env.profiles }}
  podManagementPolicy: "Parallel"
```



##### 3.1.3.网关（msa-gateway）

 msa-gateway.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.gateway.name }}
  labels:
    app: {{ .Values.gateway.labels }}
spec:
  type: {{ .Values.gateway.service.type }}
  ports:
    - port: {{ .Values.gateway.service.port }}
      targetPort: {{ .Values.gateway.service.targetPort }}
  selector:
    app: {{ .Values.gateway.labels }}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.gateway.name }}
spec:
  replicas: {{ .Values.gateway.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.gateway.labels }}
  template:
    metadata:
      labels:
        app: {{ .Values.gateway.labels }}
    spec:
      imagePullSecrets:
        - name: {{ .Values.global.imagePullSecrets }}   #对应创建私有镜像密钥Secret    
      containers:
        - name: {{ .Values.gateway.image.name }}
          image: {{ .Values.global.repository }}{{ .Values.gateway.image.name }}:{{ .Values.gateway.image.tag }}
          imagePullPolicy: {{ .Values.gateway.image.pullPolicy }} #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: {{ .Values.gateway.image.containerPort }}
          env:
            - name: EUREKA_SERVER
              value:  {{ .Values.global.env.eureka }}
            - name: ACTIVE
              value: {{ .Values.gateway.env.profiles }}
```



##### 3.1.4.生产者（msa-deploy-producer）

msa-deploy-producer.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.producer.name }}
  labels:
    app: {{ .Values.producer.labels }}
spec:
  type: {{ .Values.producer.service.type }}
  ports:
    - port: {{ .Values.producer.service.port }}
      targetPort: {{ .Values.producer.service.targetPort }}
  selector:
    app: {{ .Values.producer.labels }}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.producer.name }}
spec:
  replicas: {{ .Values.producer.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.producer.labels }}
  template:
    metadata:
      labels:
        app: {{ .Values.producer.labels }}
    spec:
      imagePullSecrets:
        - name: {{ .Values.global.imagePullSecrets }}   #对应创建私有镜像密钥Secret
      containers:
        - name: {{ .Values.producer.image.name }}
          image: {{ .Values.global.repository }}{{ .Values.producer.image.name }}:{{ .Values.producer.image.tag }}
          imagePullPolicy: {{ .Values.producer.image.pullPolicy }} #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: {{ .Values.producer.image.containerPort }}
          env:
            - name: EUREKA_SERVER
              value:  {{ .Values.global.env.eureka }}
            - name: ACTIVE
              value: {{ .Values.producer.env.profiles }}
            - name: DASHBOARD
              value: {{ .Values.producer.env.dashboard }}
```



##### 3.1.5.消费者（msa-deploy-consumer）

msa-deploy-consumer.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.consumer.name }}
  labels:
    app: {{ .Values.consumer.labels }}
spec:
  type: {{ .Values.consumer.service.type }}
  ports:
    - port: {{ .Values.consumer.service.port }}
      targetPort: {{ .Values.consumer.service.targetPort }}
  selector:
    app: {{ .Values.consumer.labels }}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: {{ .Values.global.namespace }}
  name: {{ .Values.consumer.name }}
spec:
  replicas: {{ .Values.consumer.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.consumer.labels }}
  template:
    metadata:
      labels:
        app: {{ .Values.consumer.labels }}
    spec:
      imagePullSecrets:
        - name: {{ .Values.global.imagePullSecrets }}   #对应创建私有镜像密钥Secret
      containers:
        - name: {{ .Values.consumer.image.name }}
          image: {{ .Values.global.repository }}{{ .Values.consumer.image.name }}:{{ .Values.consumer.image.tag }}
          imagePullPolicy: {{ .Values.consumer.image.pullPolicy }} #如果省略imagePullPolicy,策略为IfNotPresent
          ports:
          - containerPort: {{ .Values.consumer.image.containerPort }}
          env:
            - name: EUREKA_SERVER
              value:  {{ .Values.global.env.eureka }}
            - name: ACTIVE
              value: {{ .Values.consumer.env.profiles }}
            - name: DASHBOARD
              value: {{ .Values.consumer.env.dashboard }}
```



##### 3.1.6.values

values.yaml 

```yaml
global:
  namespace: "devops"
  imagePullSecrets: "aliyunregistrysecret"
  repository: "registry.cn-beijing.aliyuncs.com/devops-hollysys/"
  env:
    eureka: "http://msa-eureka-0.msa-eureka.devops:10001/eureka/,http://msa-eureka-1.msa-eureka.devops:10001/eureka/,http://msa-eureka-2.msa-eureka.devops:10001/eureka/"


eureka:
  name: "msa-eureka"
  labels: "msa-eureka"
  service:
    type: "NodePort"
    port: 10001
    targetPort: 10001
    nodePort: 30011
  replicaCount: 3
  image:
    name: "msa-eureka"
    pullPolicy: "IfNotPresent"
    tag: "2.0.0"
    containerPort: 10001
  env:
    profiles: "-Dspring.profiles.active=k8s"


gateway:
  name: "msa-gateway"
  labels: "msa-gateway"
  service:
    type: "ClusterIP"
    port: 8888
    targetPort: 8888
  replicaCount: 3
  image:
    name: "msa-gateway"
    pullPolicy: "IfNotPresent"
    tag: "2.0.0"
    containerPort: 8888
  env:
    profiles: "-Dspring.profiles.active=k8s"


producer:
  name: "msa-deploy-producer"
  labels: "msa-deploy-producer"
  service:
    type: "ClusterIP"
    port: 8911
    targetPort: 8911
  replicaCount: 3
  image:
    name: "msa-deploy-producer"
    pullPolicy: "IfNotPresent"
    tag: "2.0.0"
    containerPort: 8911
  env:
    profiles: "-Dspring.profiles.active=k8s"
    dashboard: "sentinel-server.devops.svc.cluster.local:8858"


consumer:
  name: "msa-deploy-consumer"
  labels: "msa-deploy-consumer"
  service:
    type: "ClusterIP"
    port: 8912
    targetPort: 8912
  replicaCount: 3
  image:
    name: "msa-deploy-consumer"
    pullPolicy: "IfNotPresent"
    tag: "2.0.0"
    containerPort: 8912
  env:
    profiles: "-Dspring.profiles.active=k8s"
    dashboard: "sentinel-server.devops.svc.cluster.local:8858"
```



#### 3.2.创建阿里云secret秘钥

**1.Node使用docker login阿里云的私有仓库，保证调度过来pod时可以及时拉取到镜像**

```shell
# docker login --username=xxx@hotmail.com registry.cn-hangzhou.aliyuncs.com

# 登录阿里云
docker login -u xxxxxx -p ******  registry.cn-beijing.aliyuncs.com
```



**2.需要在master上生成secret秘钥**

```shell
kubectl create secret docker-registry alidockerregistryssecret --docker-server=registry.cn-hangzhou.aliyuncs.com --docker-username=xxx --docker-password=xxx --docker-email=xxxx@xxx.com

说明：
alidockerregistryssecret :指定秘钥的键名称，可自行定义
--docker-server ：指定docker仓库的地址
--docker-username :指定docker仓库账号
--docker-password ：指定docker仓库密码
--docker-email： 指定docker邮件地址（选填）

alidockerregistryssecret 只能在默认namespace下使用，其他要使用则在创建时指定namespace（-n xxx）
```

```shell
kubectl create secret docker-registry aliyunregistrysecret -n devops \
--docker-server=registry.cn-hangzhou.aliyuncs.com \
--docker-username=xxxxxx \
--docker-password=****** \
--docker-email=xxxxxxxxx


# 创建secret
[root@k8s-master ~]# kubectl create secret docker-registry aliyunregistrysecret -n devops \
> --docker-server=registry.cn-hangzhou.aliyuncs.com \
> --docker-username=xxxxxx \
> --docker-password=****** \
> --docker-email=xxxxxx
secret/aliyunregistrysecret created


[root@k8s-master ~]#  kubectl get secret -n devops
NAME                   TYPE                                  DATA   AGE
aliyunregistrysecret   kubernetes.io/dockerconfigjson        1      7s
default-token-zfdkm    kubernetes.io/service-account-token   3      4h59m
```





#### 3.3.k8s测试

**1.创建名字空间devops**

```shell
[root@k8s-master devops]# kubectl create ns devops
```



**2.安装部署**

```shell
# 调试
# Helm也提供了--dry-run --debug调试参数，帮助你验证模板正确性。在执行helm install时候带上这两个参数就可以把对应的values值和渲染的资源清单打印出来，而不会真正的去部署一个release。

[root@k8s-master devops]# helm install msa cloud --dry-run --debug -n devops


# 安装
[root@k8s-master devops]# helm install msa cloud -n devops
NAME: msa
LAST DEPLOYED: Sun Jan 23 18:17:33 2022
NAMESPACE: devops
STATUS: deployed
REVISION: 1
TEST SUITE: None


[root@k8s-master devops]# helm list -n devops
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	devops   	1       	2022-01-23 18:17:33.702678573 +0800 CST	deployed	cloud-0.1.0	1.0.0 

[root@k8s-master devops]# kubectl get all -n devops
NAME                                       READY   STATUS    RESTARTS   AGE
pod/msa-deploy-consumer-bb8b466ff-62gn9    1/1     Running   0          22m
pod/msa-deploy-consumer-bb8b466ff-npp8x    1/1     Running   0          22m
pod/msa-deploy-consumer-bb8b466ff-s87m6    1/1     Running   0          22m
pod/msa-deploy-producer-5b568c48c6-2ww8z   1/1     Running   0          22m
pod/msa-deploy-producer-5b568c48c6-6dbjr   1/1     Running   0          22m
pod/msa-deploy-producer-5b568c48c6-qf985   1/1     Running   0          22m
pod/msa-eureka-0                           1/1     Running   0          22m
pod/msa-eureka-1                           1/1     Running   0          22m
pod/msa-eureka-2                           1/1     Running   0          22m
pod/msa-gateway-5f7df6485f-8hp2z           1/1     Running   0          22m
pod/msa-gateway-5f7df6485f-8lc9q           1/1     Running   0          22m
pod/msa-gateway-5f7df6485f-hnlkc           1/1     Running   0          22m

NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
service/msa-deploy-consumer   ClusterIP   10.97.9.158      <none>        8912/TCP          22m
service/msa-deploy-producer   ClusterIP   10.108.147.240   <none>        8911/TCP          22m
service/msa-eureka            NodePort    10.110.70.23     <none>        10001:30011/TCP   22m
service/msa-gateway           ClusterIP   10.96.88.171     <none>        8888/TCP          22m

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/msa-deploy-consumer   3/3     3            3           22m
deployment.apps/msa-deploy-producer   3/3     3            3           22m
deployment.apps/msa-gateway           3/3     3            3           22m

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/msa-deploy-consumer-bb8b466ff    3         3         3       22m
replicaset.apps/msa-deploy-producer-5b568c48c6   3         3         3       22m
replicaset.apps/msa-gateway-5f7df6485f           3         3         3       22m

NAME                          READY   AGE
statefulset.apps/msa-eureka   3/3     22m


[root@k8s-master devops]# helm uninstall msa -n devops
release "msa" uninstalled
[root@k8s-master devops]# helm list -n devops
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
```



#### 3.4.k8s配置免密登陆GitLab

**1.设置用户签名**

```shell
git config --global user.name xxxxxx
git config --global user.email xxxxxx@126.com


[root@k8s-master ~]# git config --list
user.name=xxxxxx
user.email=xxxxxx@126.com


[root@k8s-master ~]#  vim ~/.gitconfig 
```



**2.生成秘钥**

```shell
$ ssh-keygen
$ ssh-keygen -C xxxxxx@126.com


[root@k8s-master ~]# ssh-keygen -C xxxxxx@126.com
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:RbHXtTx5SadDYGlgCmqLIVwZFrhoKA9WjBT9/WYmPPQ xxxxxx@126.com
The key's randomart image is:
+---[RSA 2048]----+
|.o*=+ .   =ooo..o|
|.oo= . . + ooooo=|
|+.+ + . . o.. +*.|
|== + o o . .   .o|
|+o. . o S        |
|  .    + E       |
|        *        |
|                 |
|                 |
+----[SHA256]-----+


[root@k8s-master ~]# cat .ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCkkF3p1M4kLggpRFG5pdgDBxQFZmkFAepkmvUiWiOJOzYcQpW7hyxLDle/rRky9W+Eg0hO6tZJLxMaw7Zby+60C08dQgHyxAknAAmqHRWyhncpy4AxIIuYErc4CxXCvHGwypSOG8hCgYNuswTZewJrRnXvz8BqExWApWIzQ/6MbzVK7xQeNsYoCXt0nNQccZOgIkWbUfUOv7Z02ZR8hpmjaSr2Y88ir7Pu0fpCT1SADFSC+LWE6zOSfSKcT1sjhxPMnfa61NFvn/W5o7opJNZWACLdNLX4lEujcbetbKQl1f91SF2oiH3HiMTTcqMG6zqU8lBNFNvpnZl9Om1Ss7kP xxxxxx@126.com
```



**3.GitLab配置SSH**

**查看你生成的公钥：**

vim id_rsa.pub

就可以查看到你的公钥



**登陆GitLab账号，点击用户图像，然后 Settings -> 左栏点击 SSH keys**

![](/images/devops/jenkins/springcloud/sc-31.png)

**复制公钥内容，粘贴进“Key”文本区域内，取名字**

**点击Add Key**

![](/images/devops/jenkins/springcloud/sc-31.png)

![](/images/devops/jenkins/springcloud/sc-32.png)



**测试**

```shell
[root@k8s-master ~]# git clone git@39.96.178.134:test-group/k8s-deploy.git
Cloning into 'k8s-deploy'...
The authenticity of host '39.96.178.134 (39.96.178.134)' can't be established.
ECDSA key fingerprint is SHA256:vTsIUoWxtS+jKunBhSVWi8YIw5zeQwALBxbgjhTMOs4.
ECDSA key fingerprint is MD5:d3:8d:e7:d1:81:3b:22:9e:9f:d3:ff:26:61:4d:9b:a1.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '39.96.178.134' (ECDSA) to the list of known hosts.
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 4 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (4/4), done.


[root@k8s-master ~]# ll
drwxr-xr-x   4 root root        32 Jan 23 19:04 k8s-deploy
```



#### 3.5.Helm源码上传GitLab仓库

```shell
# 克隆
[root@k8s-master devops]# git clone git@39.96.178.134:test-group/k8s-deploy.git


[root@k8s-master devops]# cd k8s-deploy/
[root@k8s-master k8s-deploy]# mkdir k8s
[root@k8s-master k8s-deploy]# ll
total 0
drwxr-xr-x 2 root root 23 Jan 23 19:09 docker
drwxr-xr-x 2 root root  6 Jan 23 19:15 k8s

# 复制文件
[root@k8s-master devops]# cp -r cloud k8s-deploy/k8s/cloud
```

```shell
# 提交本地仓库
[root@k8s-master k8s-deploy]# git add .
[root@k8s-master k8s-deploy]# git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#	new file:   k8s/cloud/.helmignore
#	new file:   k8s/cloud/Chart.yaml
#	new file:   k8s/cloud/templates/msa-deploy-consumer.yaml
#	new file:   k8s/cloud/templates/msa-deploy-producer.yaml
#	new file:   k8s/cloud/templates/msa-eureka.yaml
#	new file:   k8s/cloud/templates/msa-gateway.yaml
#	new file:   k8s/cloud/values.yaml
#

[root@k8s-master k8s-deploy]# git commit -m "增加helm文件"
[master 8f4919f] 增加helm文件
 7 files changed, 287 insertions(+)
 create mode 100644 k8s/cloud/.helmignore
 create mode 100644 k8s/cloud/Chart.yaml
 create mode 100644 k8s/cloud/templates/msa-deploy-consumer.yaml
 create mode 100644 k8s/cloud/templates/msa-deploy-producer.yaml
 create mode 100644 k8s/cloud/templates/msa-eureka.yaml
 create mode 100644 k8s/cloud/templates/msa-gateway.yaml
 create mode 100644 k8s/cloud/values.yaml


# 上传仓库
[root@k8s-master k8s-deploy]# git remote -v
origin	git@39.96.178.134:test-group/k8s-deploy.git (fetch)
origin	git@39.96.178.134:test-group/k8s-deploy.git (push)

[root@k8s-master k8s-deploy]# git push origin master
Counting objects: 13, done.
Delta compression using up to 6 threads.
Compressing objects: 100% (11/11), done.
Writing objects: 100% (12/12), 2.51 KiB | 0 bytes/s, done.
Total 12 (delta 3), reused 0 (delta 0)
To git@39.96.178.134:test-group/k8s-deploy.git
   42d20b1..8f4919f  master -> master
```



#### 3.6.Jenkins创建持续集成部署项目

![](/images/devops/jenkins/springcloud/sc-33.png)

![](/images/devops/jenkins/springcloud/sc-34.png)

![](/images/devops/jenkins/springcloud/sc-35.png)



**执行脚本**

```shell
echo "克隆项目 ---"
rm -rf k8s-deploy
git clone git@39.96.178.134:test-group/k8s-deploy.git

echo "部署服务 ---"
helm install msa k8s-deploy/k8s/cloud -n devops

echo "删除文件 ---"
rm -rf k8s-deploy
```



#### 3.7.构建项目

```shell
[root@k8s-master devops]# helm list -n devops
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
msa 	devops   	1       	2022-01-23 18:17:33.702678573 +0800 CST	deployed	cloud-0.1.0	1.0.0 

[root@k8s-master devops]# kubectl get all -n devops
NAME                                       READY   STATUS    RESTARTS   AGE
pod/msa-deploy-consumer-bb8b466ff-62gn9    1/1     Running   0          22m
pod/msa-deploy-consumer-bb8b466ff-npp8x    1/1     Running   0          22m
pod/msa-deploy-consumer-bb8b466ff-s87m6    1/1     Running   0          22m
pod/msa-deploy-producer-5b568c48c6-2ww8z   1/1     Running   0          22m
pod/msa-deploy-producer-5b568c48c6-6dbjr   1/1     Running   0          22m
pod/msa-deploy-producer-5b568c48c6-qf985   1/1     Running   0          22m
pod/msa-eureka-0                           1/1     Running   0          22m
pod/msa-eureka-1                           1/1     Running   0          22m
pod/msa-eureka-2                           1/1     Running   0          22m
pod/msa-gateway-5f7df6485f-8hp2z           1/1     Running   0          22m
pod/msa-gateway-5f7df6485f-8lc9q           1/1     Running   0          22m
pod/msa-gateway-5f7df6485f-hnlkc           1/1     Running   0          22m

NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
service/msa-deploy-consumer   ClusterIP   10.97.9.158      <none>        8912/TCP          22m
service/msa-deploy-producer   ClusterIP   10.108.147.240   <none>        8911/TCP          22m
service/msa-eureka            NodePort    10.110.70.23     <none>        10001:30011/TCP   22m
service/msa-gateway           ClusterIP   10.96.88.171     <none>        8888/TCP          22m

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/msa-deploy-consumer   3/3     3            3           22m
deployment.apps/msa-deploy-producer   3/3     3            3           22m
deployment.apps/msa-gateway           3/3     3            3           22m

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/msa-deploy-consumer-bb8b466ff    3         3         3       22m
replicaset.apps/msa-deploy-producer-5b568c48c6   3         3         3       22m
replicaset.apps/msa-gateway-5f7df6485f           3         3         3       22m

NAME                          READY   AGE
statefulset.apps/msa-eureka   3/3     22m
```

**注意：从阿里云镜像仓库拉取失败，配置有问题。以后解决。**



