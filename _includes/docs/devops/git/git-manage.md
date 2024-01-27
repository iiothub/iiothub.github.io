* TOC
{:toc}



## 一、概述



### 1.Git分支分类

**根据生命周期区分**

- 主分支：master，develop
- 临时分支：feature/*，release/*，hotfix/*

**根据用途区分**

- 发布/预发布分支：master，release/*
- 开发分支：develop
- 功能分支：feature/*
- 热修复分支：hotfix/*



### 2.分支的用途

- master：作为发布分支，随时可以将分支上的代码部署到生产环境上。如果在生产环境上发现问题，则以 master 为基准创建 hotfix/* 分支来修复问题
- develop：作为开发分支，所有最新的功能都将在该分支下进行开发，develop 也将是所有分支中功能最全，代码最新的一个分支
- feature/*：命名规则`feature/功能名称`，作为新功能的开发分支，该分支从 develop 创建，开发完毕之后需要重新合并到 develop
- release/*：命名规则`release/v+发布的版本号`，作为预发布分支，release/* 只能从 develop 创建，且在 git flow 中同一个时间点，只能存在一个预发布分支。只有当上一个版本发布成功之后删除该分支，之后才能进行下一个版本的发布。如果在预发布过程中发现了问题，只能在 release/* 分支上进行修改
- hotfix/*：命名规则`hotfix/v+bug修复的版本号`，作为热修复分支，只能从 master 分支分离出来。主要是用来修复在生产环境上发现的 bug，修复完成并测试通过后需要将该分支合并回 develop 及 master 上，并删除该分支



### 3.Git 分支管理流程

 Git 分支管理的整个流程。先来看一下分支在完整的功能开发中是如何演变的：

![](/images/devops/git/git-manage/branch-1.png)





## 二、基础



### 1.Git Flow

就像代码需要代码规范一样，代码管理同样需要一个清晰的流程和规范

Vincent Driessen 同学为了解决这个问题提出了 [A Successful Git Branching Model](http://nvie.com/posts/a-successful-git-branching-model/)

**下面是Git Flow的流程图**



![](/images/devops/git/git-manage/branch-2.png)



#### 1.1.Git Flow常用的分支

- **Production 分支**

也就是我们经常使用的Master分支，这个分支最近发布到生产环境的代码，最近发布的Release， 这个分支只能从其他分支合并，不能在这个分支直接修改

- **Develop 分支**

这个分支是我们是我们的主开发分支，包含所有要发布到下一个Release的代码，这个主要合并与其他分支，比如Feature分支

- **Feature 分支**

这个分支主要是用来开发一个新的功能，一旦开发完成，我们合并回Develop分支进入下一个Release

- **Release分支**

当你需要一个发布一个新Release的时候，我们基于Develop分支创建一个Release分支，完成Release后，我们合并到Master和Develop分支

- **Hotfix分支**

当我们在Production发现新的Bug时候，我们需要创建一个Hotfix, 完成Hotfix后，我们合并回Master和Develop分支，所以Hotfix的改动会进入下一个Release



#### 1.2.Git Flow如何工作



- **初始分支**

所有在Master分支上的Commit应该Tag

![](/images/devops/git/git-manage/branch-3.png)



- **Feature 分支**

分支名 feature/*

Feature分支做完后，必须合并回Develop分支, 合并完分支后一般会删点这个Feature分支，但是我们也可以保留

![img](/images/devops/git/git-manage/branch-4.png)



- **Release分支**

分支名 release/*

Release分支基于Develop分支创建，打完Release分之后，我们可以在这个Release分支上测试，修改Bug等。同时，其它开发人员可以基于开发新的Feature (记住：一旦打了Release分支之后不要从Develop分支上合并新的改动到Release分支)

发布Release分支时，合并Release到Master和Develop， 同时在Master分支上打个Tag记住Release版本号，然后可以删除Release分支了。

![](/images/devops/git/git-manage/branch-5.png)



- **维护分支 Hotfix**

分支名 hotfix/*

hotfix分支基于Master分支创建，开发完后需要合并回Master和Develop分支，同时在Master上打一个tag

![img](/images/devops/git/git-manage/branch-6.png)



#### 1.3.Git Flow代码示例

**a. 创建develop分支**

```shell
git branch develop
git push -u origin develop    
```



**b. 开始新Feature开发**

```shell
git checkout -b some-feature develop
# Optionally, push branch to origin:
git push -u origin some-feature    

# 做一些改动    
git status
git add some-file
git commit    
```



**c. 完成Feature**

```shell
git pull origin develop
git checkout develop
git merge --no-ff some-feature
git push origin develop

git branch -d some-feature

# If you pushed branch to origin:
git push origin --delete some-feature    
```



**d. 开始Relase**

```shell
git checkout -b release-0.1.0 develop

# Optional: Bump version number, commit
# Prepare release, commit
```



**e. 完成Release**

```shell
git checkout master
git merge --no-ff release-0.1.0
git push

git checkout develop
git merge --no-ff release-0.1.0
git push

git branch -d release-0.1.0

# If you pushed branch to origin:
git push origin --delete release-0.1.0   


git tag -a v0.1.0 master
git push --tags
```



**f. 开始Hotfix**

```shell
git checkout -b hotfix-0.1.1 master    
```



**g. 完成Hotfix**

```shell
git checkout master
git merge --no-ff hotfix-0.1.1
git push


git checkout develop
git merge --no-ff hotfix-0.1.1
git push

git branch -d hotfix-0.1.1

git tag -a v0.1.1 master
git push --tags
```



### 2.Git 分支管理流程

![](/images/devops/git/git-manage/branch-1.png)



从上图我们可以看出，我们同时开始了两个功能的开发/研究任务，下面我将以此为基础来讲解分支管理的流程。

1. 先拉取最新的 develop 分支，然后以最新的 develop 为基准创建两个新的功能分支 feature/f1 和 feature/f2

   ```shell
   git pull origin develop
   git checkout -b feature/f1 develop
   ```

2. 开发人员在各自的功能分支上进行开发工作，等当前功能分支开发完之后，将当前分支交由测试人员进行测试，测试过程中的问题修复及功能修改均在当前功能分支上进行

3. 当 feature/f1 上的开发及测试任务均完成之后，将 feature/f1 合并回 develop ，并删除 feature/f1 

   ```shell
   git checkout develop
   git merge --no-ff feature/f1
   git branch -d feature/f1
   ```

4. 从 develop 分支创建新的预发布分支 release/0.2，并部署到预发布环境上测试。在预发布过程中发现问题则直接在 release/0.2 上进行修复

   ```shell
   git checkout -b release/0.2 develop
   ```

5. 在生产环境中发现一个 bug，从 master 上分离出一个热修复分支 hotfix/bug1，并在上面进行修复、测试并在预发布环境中验证，当都验证通过之后将分支重新合并回 develop 及 master，并在 master 上打一个热修复 tag v0.1.1，最后删除 hotfix/bug1

   ```shell
   git checkout -b hotfix/bug1 master
   ....................
   ....修复bug..ing....
   ....................
   git checkout develop
   git merge --no-ff hotfix/bug1
   git checkout master
   git merge --no-ff hotfix/bug1
   git tag v0.1.1
   git branch -d hotfix/bug1
   ```

6. 在 feature/f2 分支上的功能 f2 已经开发并测试完成，然后将 feature/f2 合并回 develop，并删除 feature/f2，此时已经存在新功能 f1 的预发布分支 release/0.2，所以需要等待其发布完成之后才能创建预发布分支 release/0.3

   ```shell
   git checkout develop
   git merge --no-ff feature/f2
   git branch -d feature/f2
   ```

7. 预发布分支 release/0.2 在预发布环境中完全测试通过，随时可以部署到生产环境。但在部署到生产环境之前，需要将分支合并回 develop 及 master，并在 release/0.2 上打一个正式发布版本的 tag v0.2，最后删除 release/0.2

   ```shell
   git checkout develop
   git merge --no-ff release/0.2
   git checkout master
   git merge --no-ff release/0.2
   git tag v0.2
   git branch -d release/0.2
   ```

8. 当前已经不存在 release/* 预发布分支，所以可以开始功能 f2 的预发布上线。创建预发布分支 release/0.3，并部署预发布环境测试

   ```shell
   git checkout -b release/0.3 develop
   ```

9. 分支 release/0.3 测试通过，将 release/0.3 合并回 develop 及 master，然后在 master 上打一个正式发布版本的 tag v0.3，最后删除 release/0.3



**Git Flow**

上述过程中未使用到 git flow，均是以约定的规范流程进行，大家可以尝试使用 git flow 来管理分支。

```shell
#初始化 git flow
# 设置 feature、release、hotfix、tag 的前缀名
git flow init
 
#开始一个新功能 f1 的开发，以 develop 为基准创建 feature/f1
git flow feature start f1
 
#....................
#....f1 功能开发中....
#....................
 
#新功能 f1 开发完成
# 合并回 develop
# 删除 feature/f1 分支
git flow feature finish f1
 
#开始新功能 f1 的预发布验证，版本定为 0.2
git flow release start 0.2
 
#新功能 f1 预发布验证通过
# 合并到 master 分支
# 在 release 上打 tag v0.2
# 将 tag v0.2 合并到 develop 分支
# 删除 release/0.2 分支
git flow release finish 0.2
 
#开始 bug1 的修复，以 master 为基准创建 hotfix/bug1
git flow hotfix start 0.2.1
 
# bug1 修复完成
# 合并到 master 分支
# 在 hotfix 上打 tag v0.2.1
# 将 tag v0.2.1 合并到 develop 分支
# 删除 hotfix/0.2.1 分支
git flow hotfix finish 0.2.1
```



### 3.Git flow工具

实际上，当你理解了上面的流程后，你完全不用使用工具，但是实际上我们大部分人很多命令就是记不住呀，流程就是记不住呀，肿么办呢？

总有聪明的人创造好的工具给大家用, 那就是Git flow script.



**安装**

- OS X

brew install git-flow

- Linux

apt-get install git-flow

- Windows

wget -q -O - --no-check-certificate https://github.com/nvie/gitflow/raw/develop/contrib/gitflow-installer.sh | bash



**使用**

- **初始化:** git flow init
- **开始新Feature:** git flow feature start MYFEATURE
- **Publish一个Feature(也就是push到远程):** git flow feature publish MYFEATURE
- **获取Publish的Feature:** git flow feature pull origin MYFEATURE
- **完成一个Feature:** git flow feature finish MYFEATURE
- **开始一个Release:** git flow release start RELEASE [BASE]
- **Publish一个Release:** git flow release publish RELEASE
- **发布Release:** git flow release finish RELEASE
  别忘了git push --tags
- **开始一个Hotfix:** git flow hotfix start VERSION [BASENAME]
- **发布一个Hotfix:** git flow hotfix finish VERSION

![](/images/devops/git/git-manage/branch-7.png)



### 4.Git Flow GUI

上面讲了这么多，我知道还有人记不住，那么又有人做出了GUI 工具，你只需要点击下一步就行，工具帮你干这些事！！！



**SourceTree**

当你用Git-flow初始化后，基本上你只需要点击git flow菜单选择start feature, release或者hotfix, 做完后再次选择git flow菜单，点击Done Action. 我勒个去，我实在想不到还有比这更简单的了。

目前SourceTree支持Mac, Windows, Linux.

这么好的工具请问多少钱呢？ **免费!!!!**

![](/images/devops/git/git-manage/branch-8.png)

![](/images/devops/git/git-manage/branch-9.png)



