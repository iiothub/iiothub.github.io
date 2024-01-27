* TOC
{:toc}



## 一、基础



### 1.定位Git程序

![](/images/devops/git/git-idea/idea-6.gif)

![](/images/devops/git/git-idea/idea-7.gif)



### 2.配置Git忽略文件

#### 2.1.Eclipse 忽略文件  

![](/images/devops/git/git-idea/idea-1.gif)



#### 2.2.gitignore插件

**1.在idea中安装gitignore插件**

点击File->Settings ，选择plugs，在右边搜索：.ignore，点击Install，安装完成后就可以愉快的使用了，不过在此之前得重启IDEA 

![](/images/devops/git/git-idea/idea-2.gif)



**2.现在项目中生成模板**

在项目上右键->New ->.ignore file ->.gitignore file(Git) 

![](/images/devops/git/git-idea/idea-3.gif)

![](/images/devops/git/git-idea/idea-4.gif)

在项目所在目录下生成文件：.gitignore

```shell
# .gitignore


# Created by .ignore support plugin (hsz.mobi)
### Java template
# Compiled class file
*.class

# Log file
*.log

# BlueJ files
*.ctxt

# Mobile Tools for Java (J2ME)
.mtj.tmp/

# Package Files #
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

# virtual machine crash logs, see http://www.java.com/en/download/help/error_hotspot.xml
hs_err_pid*
```



**或者选择自动追加到忽略文件中**

![](/images/devops/git/git-idea/idea-5.gif)



**3.添加的.gitignore可能不起作用**

4、添加的.gitignore可能不起作用

- 要先进入项目包所在的文件夹
- git rm -r -cached .
- git add .
- git commit -m "update .gitignore"

```shell
git rm -r --cached .
git add .
#注意: "update .gitignore" Linux是单引号，windows是双引号
git commit -m "update .gitignore"
git push -u origin master
```



#### 2.3.IDEA 忽略文件  

**1.IDEA特定文件**

![](/images/devops/git/git-idea/idea-8.gif)



**2.Maven 工程的 target 目录**

![](/images/devops/git/git-idea/idea-9.gif)



**问题 1:为什么要忽略他们？**
答： 与项目的实际功能无关，不参与服务器上部署运行。把它们忽略掉能够屏蔽 IDE 工具之
间的差异。

**问题 2：怎么忽略？**
**1） 创建忽略规则文件 xxxx.ignore（前缀名随便起，建议是 git.ignore）**
这个文件的存放位置原则上在哪里都可以，为了便于让~/.gitconfig 文件引用，建议也放在用
户家目录下  （C:\Users\Administrator）



**git.ignore 文件模版内容如下：**  

```shell
# Compiled class file
*.class

# Log file
*.log

# BlueJ files
*.ctxt

# Mobile Tools for Java (J2ME)
.mtj.tmp/

# Package Files #
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

# virtual machine crash logs, see
http://www.java.com/en/download/help/error_hotspot.xml
hs_err_pid*
.classpath
.project
.settings
target
.idea
*.iml
```



2） 在.gitconfig 文件中引用忽略配置文件（此文件在 Windows 的家目录中 C:\Users\Administrator）  

```shell
[user]
	name = hollysys
	email = hollysys@126.com
[core]
    excludesfile = C:/Users/Administrator/git.ignore

# 注意：这里要使用“正斜线（/）”，不要使用“反斜线（\）”
```



### 3.IDEA集成GitLab

#### 3.1.安装GitLab插件

**在File => Settings => Plugins 里面 搜索 gitlab**

![](/images/devops/git/git-idea/idea-34.gif)

**安装后重启**



#### 3.2.设置GitLab插件

![](/images/devops/git/git-idea/idea-35.gif)

Token：AVdN7mbs_4DzXFg6vSoH

![](/images/devops/git/git-idea/idea-36.gif)

![](/images/devops/git/git-idea/idea-37.gif)



#### 3.3.推送本地代码到 GitLab 远程库  

![](/images/devops/git/git-idea/idea-38.gif)

自定义远程连接  

![](/images/devops/git/git-idea/idea-39.gif)

![](/images/devops/git/git-idea/idea-40.gif)

选择 gitlab 远程连接，进行 push。  

![](/images/devops/git/git-idea/idea-41.gif)

**SSH方式不需要输入密码**

**http方式：首次向连接 gitlab，需要登录帐号和密码。**  

![](/images/devops/git/git-idea/idea-42.gif)

代码 Push 成功。  

![](/images/devops/git/git-idea/idea-43.gif)



#### 3.4.克隆远程仓库代码到本地  

![](/images/devops/git/git-idea/idea-44.gif)

**或**

![](/images/devops/git/git-idea/idea-45.gif)

![](/images/devops/git/git-idea/idea-46.gif)



#### 3.5.拉取远程仓库代码

![](/images/devops/git/git-idea/idea-47.gif)



### 4.IDEA集成GitHub

#### 4.1.设置 GitHub 账号  

![](/images/devops/git/git-idea/idea-61.gif)

![](/images/devops/git/git-idea/idea-62.gif)

![](/images/devops/git/git-idea/idea-63.gif)

![](/images/devops/git/git-idea/idea-64.gif)

![](/images/devops/git/git-idea/idea-65.gif)

![](/images/devops/git/git-idea/idea-66.gif)

![](/images/devops/git/git-idea/idea-67.gif)

![](/images/devops/git/git-idea/idea-68.gif)

![](/images/devops/git/git-idea/idea-69.gif)



#### 4.2.分享工程到 GitHub

![](/images/devops/git/git-idea/idea-70.gif)

![](/images/devops/git/git-idea/idea-71.gif)



来到 GitHub 中发现已经帮我们创建好了 git-idea 的远程仓库。  

![](/images/devops/git/git-idea/idea-72.gif)



#### 4.3.推送本地库到远程库

**1.GitHub上创建项目git-idea**

![](/images/devops/git/git-idea/idea-73.gif)



**2.推送本地代码**

![](/images/devops/git/git-idea/idea-74.gif)

![](/images/devops/git/git-idea/idea-75.gif)

![](/images/devops/git/git-idea/idea-76.gif)

![](/images/devops/git/git-idea/idea-77.gif)

![](/images/devops/git/git-idea/idea-78.gif)

**注意：**

 push 是将本地库代码推送到远程库，如果本地库代码跟远程库代码版本不一致，push 的操作是会被拒绝的。也就是说， 要想 push 成功，一定要保证本地库的版本要比远程库的版本高！ 因此一个成熟的程序员在动手改本地代码之前，一定会先检查下远程库跟本地代码的区别！如果本地的代码版本已经落后，切记要先 pull 拉取一下远程库的代码，将本地代码更新到最新以后，然后再修改，提交，推送！  



#### 4.4.拉取远程库到本地库

![](/images/devops/git/git-idea/idea-79.gif)

![](/images/devops/git/git-idea/idea-80.gif)

![](/images/devops/git/git-idea/idea-81.gif)

**注意： pull 是拉取远端仓库代码到本地，如果远程库代码和本地库代码不一致，会自动合并，如果自动合并失败，还会涉及到手动解决冲突的问题。**  



#### 4.5.克隆远程库到本地

![](/images/devops/git/git-idea/idea-84.gif)

![](/images/devops/git/git-idea/idea-82.gif)



**或**

![](/images/devops/git/git-idea/idea-83.gif)





## 二、实践



### 1.创建Spring Boot项目

#### 1.1.IDEA中创建项目

项目名称：git-idea

```shell
@RestController
public class HelloController {

    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    public String hello() {

        String version = "V1";
        System.out.println("v1---commit!!!");


        return "-------" + version + "-------";
    }

}
```

```shell
# 访问地址：

http://localhost:8888/hello
```



#### 1.2.GitLab中创建项目

项目名称：git-idea

```shell
# 地址
git@172.17.243.237:hs-group/git-idea.git

http://172.17.243.237:18080/hs-group/git-idea.git
zhangsan/zhangshan123
```



### 2.初始化本地仓库

选择要创建 Git 本地仓库的工程。  

![](/images/devops/git/git-idea/idea-10.gif)

![](/images/devops/git/git-idea/idea-11.gif)

![](/images/devops/git/git-idea/idea-12.gif)



### 3.添加到暂存区

右键点击项目选择 Git -> Add 将项目添加到暂存区。  

![](/images/devops/git/git-idea/idea-13.gif)

![](/images/devops/git/git-idea/idea-14.gif)



### 4.提交到本地仓库

![](/images/devops/git/git-idea/idea-15.gif)

![](/images/devops/git/git-idea/idea-16.gif)

![](/images/devops/git/git-idea/idea-17.gif)



### 5.切换版本

#### 5.1.修改两次版本

![](/images/devops/git/git-idea/idea-18.gif)



#### 5.2.切换版本

在 IDEA 的左下角，点击 Version Control，然后点击 Log 查看版本  

![](/images/devops/git/git-idea/idea-19.gif)



右键选择要切换的版本，然后在菜单里点击 Checkout Revision。  

![](/images/devops/git/git-idea/idea-20.gif)

![](/images/devops/git/git-idea/idea-21.gif)



### 6.创建分支

选择 Git， 在 Repository 里面，点击 Branches 按钮。  

![](/images/devops/git/git-idea/idea-22.gif)

在弹出的 Git Branches 框里， 点击 New Branch 按钮。  

![](/images/devops/git/git-idea/idea-23.gif)

填写分支名称，创建 hot-fix 分支。  

![](/images/devops/git/git-idea/idea-24.gif)

然后再 IDEA 的右下角看到 hot-fix，说明分支创建成功，并且当前已经切换成 hot-fix 分支  

![](/images/devops/git/git-idea/idea-25.gif)



### 7.分支切换

在 IDEA 窗口的右下角，切换到 master 分支  

![](/images/devops/git/git-idea/idea-26.gif)

然后在 IDEA 窗口的右下角看到了 master，说明 master 分支切换成功。  

![](/images/devops/git/git-idea/idea-27.gif)



### 8.合并分支

在 IDEA 窗口的右下角，将 hot-fix 分支合并到当前 master 分支  

![](/images/devops/git/git-idea/idea-28.gif)

如果代码没有冲突， 分支直接合并成功，分支合并成功以后，代码自动提交，无需手动提交本地库。  



### 9.解决冲突

如图所示，如果 master 分支和 hot-fix 分支都修改了代码，在合并分支的时候就会发生冲突。  

![](/images/devops/git/git-idea/idea-29.gif)

![](/images/devops/git/git-idea/idea-30.gif)

参考说明

![](/images/devops/git/git-idea/idea-31.gif)

手动合并完代码以后，点击右下角的 Apply 按钮。  

![](/images/devops/git/git-idea/idea-32.gif)

代码冲突解决，自动提交本地库。  

![](/images/devops/git/git-idea/idea-33.gif)



### 10.Git标签

#### 10.1.创建标签

![](/images/devops/git/git-idea/idea-48.gif)

![](/images/devops/git/git-idea/idea-49.gif)

![](/images/devops/git/git-idea/idea-50.gif)



#### 10.2.远程推送标签

由于不是在当前最新版本打入的标签，push时需要选择push tags （all）,不然不能push

![](/images/devops/git/git-idea/idea-51.gif)

GitLab查看

![](/images/devops/git/git-idea/idea-52.gif)



#### 10.3.检出标签

**注意：此时的软件版本不让修改**

![](/images/devops/git/git-idea/idea-53.gif)

![](/images/devops/git/git-idea/idea-54.gif)



#### 10.4.在指定标签继续开发

![](/images/devops/git/git-idea/idea-55.gif)

![](/images/devops/git/git-idea/idea-56.gif)

![](/images/devops/git/git-idea/idea-57.gif)



#### 10.5.删除标签

**在idea中由于没有找到删除标签的功能，所以只能采用命令行的方式进行。**

1. 进入工程目录-->右键，Git Bash Here进入命令窗口
2. 删除本地标签命令   git tag -d v1.1.0
3. 删除远程标签命令  git push origin :refs/tags/v1.1.0

```shell
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-idea (master)
$ git tag
v1.0.0
v1.1.0


# 删除本地标签
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-idea (master)
$ git tag -d v1.1.0
Deleted tag 'v1.1.0' (was b413108)

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-idea (master)
$ git tag
v1.0.0


# 删除远程标签
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-idea (master)
$ git push origin :refs/tags/v1.1.0
To 182.92.210.65:hs-group/git-idea.git
 - [deleted]         v1.1.0
```



