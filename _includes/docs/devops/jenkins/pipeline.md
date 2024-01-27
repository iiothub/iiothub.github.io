* TOC
{:toc}



## 一、概述



### 1.简介

**1.概念**

Pipeline，简单来说，就是一套运行在 Jenkins 上的工作流框架，将原来独立运行于单个或者多个节点
的任务连接起来，实现单个任务难以完成的复杂流程编排和可视化的工作。



**2.使用Pipeline有以下好处**

- 代码：Pipeline以代码的形式实现，通常被检入源代码控制，使团队能够编辑，审查和迭代其传送流
  程
- 持久：无论是计划内的还是计划外的服务器重启，Pipeline都是可恢复的
- 可停止：Pipeline可接收交互式输入，以确定是否继续执行Pipeline
-  多功能：Pipeline支持现实世界中复杂的持续交付要求。它支持fork/join、循环执行，并行执行任务的功能
-  可扩展：Pipeline插件支持其DSL的自定义扩展 ，以及与其他插件集成的多个选项



**3.如何创建 Jenkins Pipeline呢？**

- Pipeline 脚本是由 **Groovy** 语言实现的，但是我们没必要单独去学习 Groovy
- Pipeline 支持两种语法：**Declarative**(声明式)和 **Scripted Pipeline**(脚本式)语法
- Pipeline 也有两种创建方法：可以直接在 Jenkins 的 Web UI 界面中输入脚本；也可以通过创建一个 Jenkinsfile 脚本文件放入项目源码库中（一般我们都推荐在 Jenkins 中直接从源代码控制(SCM)中直接载入 Jenkinsfile Pipeline 这种方法）。  



### 2.什么是Jenkins的流水线?

Jenkins 流水线 (或简单的带有大写"P"的"Pipeline") 是一套插件，它支持实现和集成 *continuous delivery pipelines* 到Jenkins。

_continuous delivery (CD) pipeline_是你的进程的自动表达，用于从版本控制向用户和客户获取软件。 你的软件的每次的变更 (在源代码控制中提交)在它被释放的路上都经历了一个复杂的过程 on its way to being released. 这个过程包括以一种可靠并可重复的方式构建软件, 以及通过多个测试和部署阶段来开发构建好的软件 (c成为 "build") 。

流水线提供了一组可扩展的工具，通过 [Pipeline domain-specific language (DSL) syntax](https://www.jenkins.io/zh/doc/book/pipeline/syntax). [[1](https://www.jenkins.io/zh/doc/book/pipeline/#_footnotedef_1)]对从简单到复杂的交付流水线 "作为代码" 进行建模。

对Jenkins 流水线的定义被写在一个文本文件中 (成为 [`Jenkinsfile`](https://www.jenkins.io/zh/doc/book/pipeline/jenkinsfile))，该文件可以被提交到项目的源代码的控制仓库。 [[2](https://www.jenkins.io/zh/doc/book/pipeline/#_footnotedef_2)] 这是"流水线即代码"的基础; 将CD 流水线作为应用程序的一部分，像其他代码一样进行版本化和审查。 创建 `Jenkinsfile`并提交它到源代码控制中提供了一些即时的好处:

- 自动地为所有分支创建流水线构建过程并拉取请求
- 在流水线上代码复查/迭代 (以及剩余的源代码)
- 对流水线进行审计跟踪
- 该流水线的真正的源代码 [[3](https://www.jenkins.io/zh/doc/book/pipeline/#_footnotedef_3)], 可以被项目的多个成员查看和编辑

While定义流水线的语法, 无论是在 web UI 还是在 `Jenkinsfile` 中都是相同的, 通常认为在`Jenkinsfile` 中定义并检查源代码控制是最佳实践。



### 3.声明式和脚本化的流水线语法

`Jenkinsfile` 能使用两种语法进行编写 - 声明式和脚本化。

声明式和脚本化的流水线从根本上是不同的。 声明式流水线的是 Jenkins 流水线更近的特性:

- 相比脚本化的流水线语法，它提供更丰富的语法特性
- 是为了使编写和读取流水线代码更容易而设计的

然而，写到`Jenkinsfile`中的许多单独的语法组件(或者 "步骤"), 通常都是声明式和脚本化相结合的流水线。 在下面的 [[pipeline-concepts\]](https://www.jenkins.io/zh/doc/book/pipeline/#pipeline-concepts) 和 [[pipeline-syntax-overview\]](https://www.jenkins.io/zh/doc/book/pipeline/#pipeline-syntax-overview) 了解更多这两种语法的不同。



### 4.Why Pipeline?

本质上，Jenkins 是一个自动化引擎，它支持许多自动模式。 流水线向Jenkins中添加了一组强大的工具, 支持用例 简单的持续集成到全面的CD流水线。通过对一系列的相关任务进行建模, 用户可以利用流水线的很多特性:

- **Code**: 流水线是在代码中实现的，通常会检查到源代码控制, 使团队有编辑, 审查和迭代他们的交付流水线的能力
- **Durable**: 流水线可以从Jenkins的主分支的计划内和计划外的重启中存活下来
- **Pausable**: 流水线可以有选择的停止或等待人工输入或批准，然后才能继续运行流水线
- **Versatile**: 流水线支持复杂的现实世界的 CD 需求, 包括fork/join, 循环, 并行执行工作的能力
- **Extensible**:流水线插件支持扩展到它的DSL [[1](https://www.jenkins.io/zh/doc/book/pipeline/#_footnotedef_1)]的惯例和与其他插件集成的多个选项

然而， Jenkins一直允许以将自由式工作链接到一起的初级形式来执行顺序任务, [[4](https://www.jenkins.io/zh/doc/book/pipeline/#_footnotedef_4)] 流水线使这个概念成为了Jenkins的头等公民。

构建一个的可扩展的核心Jenkins值, 流水线也可以通过 [Pipeline Shared Libraries](https://www.jenkins.io/zh/doc/book/pipeline/shared-libraries) 的用户和插件开发人员来扩展。 [[5](https://www.jenkins.io/zh/doc/book/pipeline/#_footnotedef_5)]

下面的流程图是一个 CD 场景的示例，在Jenkins中很容易对该场景进行建模:

![](/images/devops/jenkins/pipeline/pipeline-1.png)





## 二、基础



### 1.流水线概念

**流水线**

流水线是用户定义的一个CD流水线模型 。流水线的代码定义了整个的构建过程, 他通常包括构建, 测试和交付应用程序的阶段 。

另外 ， `pipeline` 块是 [声明式流水线语法的关键部分](https://www.jenkins.io/zh/doc/book/pipeline/#declarative-pipeline-fundamentals)。



**节点**

节点是一个机器 ，它是Jenkins环境的一部分 and is capable of执行流水线。

另外, `node`块是 [脚本化流水线语法的关键部分](https://www.jenkins.io/zh/doc/book/pipeline/#scripted-pipeline-fundamentals)。



**阶段**

`stage` 块定义了在整个流水线的执行任务的概念性地不同的的子集(比如 "Build", "Test" 和 "Deploy" 阶段), 它被许多插件用于可视化 或Jenkins流水线目前的 状态/进展. [[6](https://www.jenkins.io/zh/doc/book/pipeline/#_footnotedef_6)]



**步骤**

本质上 ，一个单一的任务, a step 告诉Jenkins 在特定的时间点要做_what_ (或过程中的 "step")。 举个例子,要执行shell命令 ，请使用 `sh` 步骤: `sh 'make'`。当一个插件扩展了流水线DSL, [[1](https://www.jenkins.io/zh/doc/book/pipeline/#_footnotedef_1)] 通常意味着插件已经实现了一个新的 *step*。



#### 1.1.声明式流水线基础

在声明式流水线语法中, `pipeline` 块定义了整个流水线中完成的所有的工作。

![](/images/devops/jenkins/pipeline/pipeline-2.png)

![](/images/devops/jenkins/pipeline/pipeline-3.png)



#### 1.2.脚本化流水线基础

在脚本化流水线语法中, 一个或多个 `node` 块在整个流水线中执行核心工作。 虽然这不是脚本化流水线语法的强制性要求, 但它限制了你的流水线的在`node`块内的工作做两件事:

1. 通过在Jenkins队列中添加一个项来调度块中包含的步骤。 节点上的执行器一空闲, 该步骤就会运行
2. 创建一个工作区(特定为特定流水间建立的目录)，其中工作可以在从源代码控制检出的文件上完成。
   **Caution:** 根据你的 Jenkins 配置,在一系列的空闲后，一些工作区可能不会自动清理 。参考 [JENKINS-2111](https://issues.jenkins-ci.org/browse/JENKINS-2111) 了解更多信息

![](/images/devops/jenkins/pipeline/pipeline-4.png)



#### 1.3.流水线示例

这有一个使用声明式流水线的语法编写的 `Jenkinsfile` 文件 - 可以通过点击下面 **Toggle Scripted Pipeline** 链接来访问它的等效的脚本化语法：

![](/images/devops/jenkins/pipeline/pipeline-5.png)

![](/images/devops/jenkins/pipeline/pipeline-6.png)



### 2.安装Pipeline插件

**Manage Jenkins->Manage Plugins->可选插件**  

![](/images/devops/jenkins/pipeline/pipeline-7.png)



**安装插件后，创建项目的时候多了“流水线”类型**  

![](/images/devops/jenkins/pipeline/pipeline-37.png)



### 3.流水线项目构建

#### 3.1.声明式流水线

![](/images/devops/jenkins/pipeline/pipeline-8.png)

流水线->选择HelloWorld模板  

![](/images/devops/jenkins/pipeline/pipeline-9.png)

生成内容如下：  

```groovy
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

- stages：代表整个流水线的所有执行阶段。通常stages只有1个，里面包含多个stage  
- stage：代表流水线中的某个阶段，可能出现n个。一般分为拉取代码，编译构建，部署等阶段
- steps：代表一个阶段内需要执行的逻辑。steps里面是shell脚本，git拉取代码，ssh远程发布等任意内
  容



**编写一个简单声明式Pipeline：**  

```groovy
pipeline {
    agent any
    
    stages {
        stage('拉取代码') {
            steps {
                echo '拉取代码'
            }
        } 
        stage('编译构建') {
            steps {
                echo '编译构建'
            }
            
        } 
        stage('项目部署') {
            steps {
                echo '项目部署'
            }
        }
    }
}
```

点击构建，可以看到整个构建过程  



#### 3.2.脚本式流水线

![](/images/devops/jenkins/pipeline/pipeline-10.png)

选择"Scripted Pipeline"  

![](/images/devops/jenkins/pipeline/pipeline-11.png)

生成内容如下：

![](/images/devops/jenkins/pipeline/pipeline-12.png)

- Node：节点，一个 Node 就是一个 Jenkins 节点，Master 或者 Agent，是执行 Step 的具体运行环境，后续讲到Jenkins的Master-Slave架构的时候用到
- Stage：阶段，一个 Pipeline 可以划分为若干个 Stage，每个 Stage 代表一组操作，比如：Build、Test、Deploy，Stage 是一个逻辑分组的概念
- Step：步骤，Step 是最基本的操作单元，可以是打印一句话，也可以是构建一个 Docker 镜像，由各类 Jenkins 插件提供，比如命令：sh ‘make’，就相当于我们平时 shell 终端中执行 make 命令一样  



**编写一个简单的脚本式Pipeline**  

```groovy
node {
    def mvnHome
    stage('拉取代码') {
        echo '拉取代码'
    } 
    stage('编译构建') {
        echo '编译构建'
    } 
    stage('项目部署') {
        echo '项目部署'
    }
}
```

构建结果和声明式一样！  



### 4.流水线语法

#### 4.1.流水线片段生成器

![](/images/devops/jenkins/pipeline/pipeline-14.png)

![](/images/devops/jenkins/pipeline/pipeline-15.png)

![](/images/devops/jenkins/pipeline/pipeline-16.png)

![](/images/devops/jenkins/pipeline/pipeline-17.png)



#### 4.2.拉取Git代码

![](/images/devops/jenkins/pipeline/pipeline-18.png)

![](/images/devops/jenkins/pipeline/pipeline-20.png)

```shell
# 拉取Git代码

checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: '53b412d5-ecd1-47a3-b571-4505d562634e', url: 'git@39.96.178.134:test-group/web-demo.git']]])
```

```groovy
pipeline {
    agent any

    stages {
        stage('pull code') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: '53b412d5-ecd1-47a3-b571-4505d562634e', url: 'git@39.96.178.134:test-group/web-demo.git']]])
            }
        }
    }
}
```

**查看/var/lib/jenkins/workspace/目录，发现已经从Gitlab成功拉取了代码到Jenkins中。**  

```shell
root@aliyun-8g ~]# cd /var/lib/jenkins/workspace/
```



#### 4.3.编译打包

![](/images/devops/jenkins/pipeline/pipeline-21.png)

```shell
# 编译打包

sh 'mvn clean package'
```

```groovy
pipeline {
    agent any

    stages {
        stage('pull code') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: '53b412d5-ecd1-47a3-b571-4505d562634e', url: 'git@39.96.178.134:test-group/web-demo.git']]])
            }
        }
        stage('build project') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
}
```



#### 4.3.部署

```shell
pipeline {
    agent any

    stages {
        stage('pull code') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: '53b412d5-ecd1-47a3-b571-4505d562634e', url: 'git@39.96.178.134:test-group/web-demo.git']]])
            }
        }
        stage('build project') {
            steps {
                sh 'mvn clean package'
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



#### 4.4.Jenkinsfile脚本文件

直接在Jenkins的UI界面编写Pipeline代码，这样不方便脚本维护，建议把Pipeline脚本放在项目中（一起进行版本控制）



**1.在项目创建Jenkinsfile文件**

在项目根目录建立Jenkinsfile文件，把内容复制到该文件中，把Jenkinsfile上传到Gitlab  

![](/images/devops/jenkins/pipeline/pipeline-22.png)



**2.在项目中引用该文件**  

![](/images/devops/jenkins/pipeline/pipeline-24.png)

![](/images/devops/jenkins/pipeline/pipeline-25.png)



#### 4.5.Git分支标签动态选择（非流水线）

**1.安装插件**

![](/images/devops/jenkins/pipeline/pipeline-26.png)

**2.创建项目**

![](/images/devops/jenkins/pipeline/pipeline-27.png)



![](/images/devops/jenkins/pipeline/pipeline-28.png)

![](/images/devops/jenkins/pipeline/pipeline-29.png)

**3.构建项目**

![](/images/devops/jenkins/pipeline/pipeline-30.png)

#### 4.6.Git标签动态选择（流水线）

![](/images/devops/jenkins/pipeline/pipeline-31.png)

![](/images/devops/jenkins/pipeline/pipeline-32.png)

![](/images/devops/jenkins/pipeline/pipeline-33.png)

![](/images/devops/jenkins/pipeline/pipeline-34.png)

![](/images/devops/jenkins/pipeline/pipeline-35.png)

![](/images/devops/jenkins/pipeline/pipeline-36.png)



```shell
pipeline {
    agent any

    stages {
        stage('pull code') {
            steps {
               checkout([$class: 'GitSCM', branches: [[name: 'refs/tags/${tag}']], extensions: [], userRemoteConfigs: [[credentialsId: '53b412d5-ecd1-47a3-b571-4505d562634e', url: 'git@39.96.178.134:test-group/web-demo.git']]])
            }
        }
    }
}
```



```shell
# Git拉取代码参考


checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: '53b412d5-ecd1-47a3-b571-4505d562634e', url: 'git@39.96.178.134:test-group/web-demo.git']]])


checkout([$class: 'GitSCM', branches: [[name: '${branch}']], extensions: [], userRemoteConfigs: [[credentialsId: '53b412d5-ecd1-47a3-b571-4505d562634e', url: 'git@39.96.178.134:test-group/web-demo.git']]])


checkout([$class: 'GitSCM', branches: [[name: 'refs/tags/git-2.3.0']], extensions: [], userRemoteConfigs: [[credentialsId: '53b412d5-ecd1-47a3-b571-4505d562634e', url: 'git@39.96.178.134:test-group/web-demo.git']]])
```





## 三、实践



### 1.流水线项目构建

#### 1.1.创建项目

![](/images/devops/jenkins/pipeline/pipeline-13.png)

![](/images/devops/jenkins/pipeline/pipeline-23.png)

```groovy
pipeline {
    agent any

    stages {
        stage('pull code') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: '53b412d5-ecd1-47a3-b571-4505d562634e', url: 'git@39.96.178.134:test-group/web-demo.git']]])
            }
        }
        stage('build project') {
            steps {
                sh 'mvn clean package'
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




