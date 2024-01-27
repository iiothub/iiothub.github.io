* TOC
{:toc}



## 一、概述



### 1.版本控制工具

#### 1.1.集中式版本控制工具

**CVS、SVN(Subversion)、VSS……**

集中化的版本控制系统诸如 CVS、SVN 等，都有一个单一的集中管理的服务器，保存 所有文件的修订版本，而协同工作的人们都通过客户端连到这台服务器，取出最新的文件或 者提交更新。多年以来，这已成为版本控制系统的标准做法。

这种做法带来了许多好处，每个人都可以在一定程度上看到项目中的其他人正在做些什 么。而管理员也可以轻松掌控每个开发者的权限，并且管理一个集中化的版本控制系统，要 远比在各个客户端上维护本地数据库来得轻松容易。

事分两面，有好有坏。这么做显而易见的缺点是中央服务器的单点故障。如果服务器宕 机一小时，那么在这一小时内，谁都无法提交更新，也就无法协同工作。

![](/images/devops/git/git/git-1.gif)



#### 1.2.分布式版本控制工具

**Git、Mercurial、Bazaar、Darcs……**

像 Git 这种分布式版本控制工具，客户端提取的不是最新版本的文件快照，而是把代码 仓库完整地镜像下来（本地库）。这样任何一处协同工作用的文件发生故障，事后都可以用 其他客户端的本地仓库进行恢复。因为每个客户端的每一次文件提取操作，实际上都是一次 对整个文件仓库的完整备份。

分布式的版本控制系统出现之后,解决了集中式版本控制系统的缺陷:

1. 服务器断网的情况下也可以进行开发（因为版本控制是在本地进行的）

2. 每个客户端保存的也都是整个完整的项目（包含历史记录，更加安全）

![img](/images/devops/git/git/git-2.gif)



#### 1.3.Git 与 SVN 区别

GIT不仅仅是个版本控制系统，它也是个内容管理系统(CMS),工作管理系统等。

如果你是一个具有使用SVN背景的人，你需要做一定的思想转换，来适应GIT提供的一些概念和特征。

**Git 与 SVN 区别**

1. Git是分布式的，svn不是：这是GIT和其它非分布式的版本控制系统，例如SVN，CVS等，最核心的区别
2. GIT把内容按元数据方式存储，而SVN是按文件：所有的资源控制系统都是把文件的元信息隐藏在一个类似.svn,.cvs等的文件夹里
3. GIT分支和SVN的分支不同：分支在SVN中一点不特别，就是版本库中的另外的一个目录
4. GIT没有一个全局的版本号，而SVN有：目前为止这是跟SVN相比GIT缺少的最大的一个特征
5. GIT的内容完整性要优于SVN：GIT的内容存储使用的是SHA-1哈希算法。这能确保代码内容的完整性，确保在遇到磁盘故障和网络问题时降低对版本库的破坏

git   是分布式的版本控制器  没有客户端和服务器端的概念

svn 它是C/S结构的版本控制器  有客户端和服务器端  服务器如果宕机而且代码没有备份的情况下  完整代码就会丢失



### 2.Git

#### 2.1.Git简史

![](/images/devops/git/git/git-3.png)



#### 2.2.Git工作流程

**一般工作流程如下：**

- 克隆 Git 资源作为工作目录
- 在克隆的资源上添加或修改文件
- 如果其他人修改了，你可以更新资源
- 在提交前查看修改
- 提交修改
- 在修改完成后，如果发现错误，可以撤回提交并再次修改并提交

 　　Git 的工作流程示意图：

![img](/images/devops/git/git/git-4.png)



#### 2.3.Git基本概念

- **工作区：**就是你在电脑里能看到的目录
- **暂存区：**英文叫stage, 或index。一般存放在"git目录"下的index文件（.git/index）中，所以我们把暂存区有时也叫作索引（index）
- **版本库：**工作区有一个隐藏目录.git，这个不算工作区，而是Git的版本库

　　工作区、版本库中的暂存区和版本库之间的关系的示意图：

![img](/images/devops/git/git/git-5.png)

- 　　图中左侧为工作区，右侧为版本库。在版本库中标记为 "index" 的区域是暂存区（stage, index），标记为 "master" 的是 master 分支所代表的目录树


- 　　图中我们可以看出此时 "HEAD" 实际是指向 master 分支的一个"游标"。所以图示的命令中出现 HEAD 的地方可以用 master 来替换


- 　　图中的 objects 标识的区域为 Git 的对象库，实际位于 ".git/objects" 目录下，里面包含了创建的各种对象及内容


- 　　当对工作区修改（或新增）的文件执行 "git add" 命令时，暂存区的目录树被更新，同时工作区修改（或新增）的文件内容被写入到对象库中的一个新的对象中，而该对象的ID被记录在暂存区的文件索引中


- 　　当执行提交操作（git commit）时，暂存区的目录树写到版本库（对象库）中，master 分支会做相应的更新。即 master 指向的目录树就是提交时暂存区的目录树


- 　　当执行 "git reset HEAD" 命令时，暂存区的目录树会被重写，被 master 分支指向的目录树所替换，但是工作区不受影响


- 　　当执行 "git rm --cached <file>" 命令时，会直接从暂存区删除文件，工作区则不做出改变


- 　　当执行 "git checkout ." 或者 "git checkout -- <file>" 命令时，会用暂存区全部或指定的文件替换工作区的文件。这个操作很危险，会清除工作区中未添加到暂存区的改动


- 　　当执行 "git checkout HEAD ." 或者 "git checkout HEAD <file>" 命令时，会用 HEAD 指向的 master 分支中的全部或者部分文件替换暂存区和以及工作区中的文件。这个命令也是极具危险性的，因为不但会清除工作区中未提交的改动，也会清除暂存区中未提交的改动



#### 2.4.Git 和代码托管中心

代码托管中心是基于网络服务器的远程代码仓库，一般我们简单称为远程库。

➢  **局域网**

✓      GitLab

➢  **互联网**

✓      GitHub（外网）

✓  Gitee 码云（国内网站）



#### 2.5.Git、Github、Gitlab 的区别

Git是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。

是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。

Github是在线的基于Git的代码托管服务。 GitHub是2008年由Ruby on Rails编写而成。GitHub同时提供付费账户和免费账户。这两种账户都可以创建公开的代码仓库，只有付费账户可以创建私有的代码仓库。 Gitlab解决了这个问题, 可以在上面创建免费的私人repo。 

- git            是一套软件 可以做本地私有仓库

- github   本身是一个代码托管网站   公有和私有仓库(收费)   不能做本地私有仓库

- gitlab     本身也是一个代码托管的网站 功能上和github没有区别   公有和私有仓库（免费）  可以部署本地私有仓库





## 二、基础



### 1.常用Git命令

![img](/images/devops/git/git/git-6.png)

```shell
[root@qfedu.com ~]# git init                      # 初始化 
[root@qfedu.com ~]# git add main.cpp              # 将某一个文件添加到暂存区 
[root@qfedu.com ~]# git add .                     # 将文件夹下的所有的文件添加到暂存区 
[root@qfedu.com ~]# git commit -m ‘note‘          # 将暂存区中的文件保存成为某一个版本 
[root@qfedu.com ~]# git log                       # 查看所有的版本日志 
[root@qfedu.com ~]# git status                    # 查看现在暂存区的状况 
[root@qfedu.com ~]# git diff                      # 查看现在文件与上一个提交-commit版本的区别 
[root@qfedu.com ~]# git reset --hard HEAD^        # 回到上一个版本 
[root@qfedu.com ~]# git reset --hard XXXXX        # XXX为版本编号，回到某一个版本 
[root@qfedu.com ~]# git pull origin master        # 从主分支pull到本地 
[root@qfedu.com ~]# git push -u origin master     # 从本地push到主分支 
[root@qfedu.com ~]# git pull                      # pull默认主分支 
[root@qfedu.com ~]# git push                      # push默认主分支 ...
```



| **命令名称**                          | **作用**       |
| ------------------------------------- | -------------- |
| git config --global user.name 用户名  | 设置用户签名   |
| git config --global user.email   邮箱 | 设置用户签名   |
| git init                              | 初始化本地库   |
| git  status                           | 查看本地库状态 |
| git add 文件名                        | 添加到暂存区   |
| git commit -m "日志信息" 文件名       | 提交到本地库   |
| git reflog                            | 查看历史记录   |
| git reset --hard 版本号               | 版本穿梭       |

```shell
$ git
usage: git [--version] [--help] [-C <path>] [-c <name>=<value>]
           [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
           [-p | --paginate | -P | --no-pager] [--no-replace-objects] [--bare]
           [--git-dir=<path>] [--work-tree=<path>] [--namespace=<name>]
           [--super-prefix=<path>] [--config-env=<name>=<envvar>]
           <command> [<args>]

These are common Git commands used in various situations:

start a working area (see also: git help tutorial)
   clone     Clone a repository into a new directory
   init      Create an empty Git repository or reinitialize an existing one

work on the current change (see also: git help everyday)
   add       Add file contents to the index
   mv        Move or rename a file, a directory, or a symlink
   restore   Restore working tree files
   rm        Remove files from the working tree and from the index

examine the history and state (see also: git help revisions)
   bisect    Use binary search to find the commit that introduced a bug
   diff      Show changes between commits, commit and working tree, etc
   grep      Print lines matching a pattern
   log       Show commit logs
   show      Show various types of objects
   status    Show the working tree status

grow, mark and tweak your common history
   branch    List, create, or delete branches
   commit    Record changes to the repository
   merge     Join two or more development histories together
   rebase    Reapply commits on top of another base tip
   reset     Reset current HEAD to the specified state
   switch    Switch branches
   tag       Create, list, delete or verify a tag object signed with GPG

collaborate (see also: git help workflows)
   fetch     Download objects and refs from another repository
   pull      Fetch from and integrate with another repository or a local branch
   push      Update remote refs along with associated objects

'git help -a' and 'git help -g' list available subcommands and some
concept guides. See 'git help <command>' or 'git help <concept>'
to read about a specific subcommand or concept.
See 'git help git' for an overview of the system.
```



### 2.版本穿梭

#### 2.1.版本回退

```shell
# 用 git log 命令查看：
# 每一个提交的版本都唯一对应一个 commit 版本号，
# 使用 git reset 命令退到上一个版本：

[root@qfedu.com ~]# git reset --hard HEAD^
```

```shell
[root@qfedu.com ~]# git reflog                    # 查看命令历史，以便确定要回到哪个版本
[root@qfedu.com ~]# git reset --hard commit_id    # 比如git reset --hard 3628164（不用全部输入，输入前几位即可）
```



#### 2.2.分支管理

**1.创建分支**    

```shell
[root@qfedu.com ~]# git checkout -b dev     #创建dev分支，然后切换到dev分支
[root@qfedu.com ~]# git checkout            #命令加上-b参数表示创建并切换，相当于以下两条命令：
[root@qfedu.com ~]# git branch dev git checkout dev
[root@qfedu.com ~]# git branch              #命令查看当前分支,
[root@qfedu.com ~]# git branch              #命令会列出所有分支，当前分支前面会标一个*号
[root@qfedu.com ~]# git branch * dev   master
[root@qfedu.com ~]# git add readme.txt git commit -m "branch test"  # 在dev分支上正常提交.
```



**2.分支切换**

```shell
[root@qfedu.com ~]# git checkout master     #切换回master分支
# 查看一个readme.txt文件，刚才添加的内容不见了，因为那个提交是在dev分支上，而master分支此刻的提交点并没有变  
```



**3.合并分支**

```shell
[root@qfedu.com ~]# git merge dev           #把dev分支的工作成果合并到master分支上
[root@qfedu.com ~]# git merge               #命令用于合并指定分支到当前分支。
# 合并后，再查看readme.txt的内容，就可以看到，和dev分支的最新提交是完全一样的。
```

```shell
# 注意到上面的Fast-forward信息，Git告诉我们，这次合并是“快进模式”，也就是直接把master指向dev的当前提交，所以合并速度非常快。
# 当然，也不是每次合并都能Fast-forward，我们后面会讲其他方式的合并。
```

```shell
[root@qfedu.com ~]# git branch -d dev       #删除dev分支了：
# 删除后，查看branch，就只剩下master分支了.
```



#### 2.3.解决冲突

```shell
[root@qfedu.com ~]# git checkout -b feature1        # 创建新的feature1分支
# 修改readme.txt最后一行，改为：
Creating a new branch is quick AND simple.

[root@qfedu.com ~]# git add readme.txt              # 在feature1分支上提交
[root@qfedu.com ~]# git commit -m "AND simple"
[root@qfedu.com ~]# git checkout master             #切换到master分支
Switched to branch 'master' Your branch is ahead of 'origin/master' by 1 commit.
Git还会自动提示我们当前master分支比远程的master分支要超前1个提交。

在master分支上把readme.txt文件的最后一行改为：
Creating a new branch is quick & simple.
[root@qfedu.com ~]# git add readme.txt 
[root@qfedu.com ~]# git commit -m "& simple"

现在，master分支和feature1分支各自都分别有新的提交
这种情况下，Git无法执行“快速合并”，只能试图把各自的修改合并起来，但这种合并就可能会有冲突，我们试试看：
git merge feature1 Auto-merging readme.txt CONFLICT (content): 
Merge conflict in readme.txt Automatic merge failed; 
fix conflicts and then commit the result.
```

```shell
readme.txt文件存在冲突，必须手动解决冲突后再提交。
[root@qfedu.com ~]# git status 可以显示冲突的文件;
直接查看readme.txt的内容：
Git is a distributed version control system.
Git is free software distributed under the GPL. 
Git has a mutable index called stage. 
Git tracks changes of files. 
<<<<<<< HEAD Creating a new branch is quick & simple. ======= Creating a new branch is quick AND simple. >>>>>>> feature1
Git用<<<<<<<，=======，>>>>>>>标记出不同分支的内容，我们修改后保存再提交：
[root@qfedu.com ~]# git add readme.txt  
[root@qfedu.com ~]# git commit -m "conflict fixed" 
[master 59bc1cb] conflict fixed
最后，删除feature1分支：
[root@qfedu.com ~]# git branch -d feature1 
Deleted branch feature1 (was 75a857c).
```



### 3.Git配置

Git 提供了一个叫做 git config 的工具，专门用来配置或读取相应的工作环境变量。

这些环境变量，决定了 Git 在各个环节的具体工作方式和行为。这些变量可以存放在以下三个不同的地方：

- `/etc/gitconfig` 文件：系统中对所有用户都普遍适用的配置。若使用 `git config` 时用 `--system` 选项，读写的就是这个文件
- `~/.gitconfig` 文件：用户目录下的配置文件只适用于该用户。若使用 `git config` 时用 `--global` 选项，读写的就是这个文件
- 当前项目的 Git 目录中的配置文件（也就是工作目录中的 `.git/config` 文件）：这里的配置仅仅针对当前项目有效。每一个级别的配置都会覆盖上层的相同配置，所以 `.git/config` 里的配置会覆盖 `/etc/gitconfig` 中的同名变量



#### 3.1.Git 用户信息

配置个人的用户名称和电子邮件地址：

```shell
[root@qfedu.com ~]# git config --global user.name "qfedu"
[root@qfedu.com ~]# git config --global user.email test@qq.com
```

如果用了 **--global** 选项，那么更改的配置文件就是位于你用户主目录下的那个，以后你所有的项目都会默认使用这里配置的用户信息。

如果要在某个特定的项目中使用其他名字或者电邮，只要去掉 --global 选项重新配置即可，新的设定保存在当前项目的 .git/config 文件里。



#### 3.2.文本编辑器

设置Git默认使用的文本编辑器, 一般可能会是 Vi 或者 Vim。如果你有其他偏好，比如 Emacs 的话，可以重新设置

```shell
[root@qfedu.com ~]# git config --global core.editor emacs
```



#### 3.3.差异分析工具

还有一个比较常用的是，在解决合并冲突时使用哪种差异分析工具。比如要改用 vimdiff 的话：

```shell
[root@qfedu.com ~]# git config --global merge.tool vimdiff
```

Git 可以理解 kdiff3，tkdiff，meld，xxdiff，emerge，vimdiff，gvimdiff，ecmerge，和 opendiff 等合并工具的输出信息。

当然，你也可以指定使用自己开发的工具



#### 3.4.查看配置信息

要检查已有的配置信息，可以使用 git config --list 命令：

```shell
[root@qfedu.com ~]# git config --list
http.postbuffer=2M
user.name=runoob
user.email=test@runoob.com
```

有时候会看到重复的变量名，那就说明它们来自不同的配置文件（比如 /etc/gitconfig 和 ~/.gitconfig），不过最终 Git 实际采用的是最后一个。

这些配置我们也可以在 **~/.gitconfig** 或 **/etc/gitconfig** 看到，如下所示：

```shell
[root@qfedu.com ~]# vim ~/.gitconfig 
```

显示内容如下所示：

```shell
[http]
    postBuffer = 2M
[user]
    name = git
    email = test@qfedu.com.com
```

也可以直接查阅某个环境变量的设定，只要把特定的名字跟在后面即可，像这样：

```shell
[root@qfedu.com ~]# git config user.name
git
```



### 4.Git分支操作

![](/images/devops/git/git/git-14.png)



#### 4.1.什么是分支

在版本控制过程中，同时推进多个任务，为每个任务，我们就可以创建每个任务的单独 分支。使用分支意味着程序员可以把自己的工作从开发主线上分离开来，开发自己分支的时 候，不会影响主线分支的运行。对于初学者而言，分支可以简单理解为副本，一个分支就是 一个单独的副本。（分支底层其实也是指针的引用）

![](/images/devops/git/git/git-15.png)



#### 4.2.分支的好处

同时并行推进多个功能开发，提高开发效率。 各个分支在开发过程中，如果某一个分支开发失败，不会对其他分支有任何影响。失败的分支删除重新开始即可。

 

#### 4.3.分支的操作

| **命令名称**        | **作用**                     |
| ------------------- | ---------------------------- |
| git branch 分支名   | 创建分支                     |
| git branch -v       | 查看分支                     |
| git checkout 分支名 | 切换分支                     |
| git merge 分支名    | 把指定的分支合并到当前分支上 |

 

#### 4.4.分支图解

![](/images/devops/git/git/git-17.png)

![](/images/devops/git/git/git-18.png)



- **master、 hot-fix 其实都是指向具体版本记录的指针**
- **当前所在的分支，其实是由 HEAD决定的。所以创建分支的本质就是多创建一个指针**
- **HEAD 如果指向 master，那么我们现在就在 master 分支上**
- **HEAD 如果执行 hotfix，那么我们现在就在 hotfix 分支上**
- **所以切换分支的本质就是移动 HEAD 指针**  



### 5.Git 团队协作机制 

#### 5.1.团队内协作

![](/images/devops/git/git/git-19.png)



#### 5.2.跨团队协作  

![](/images/devops/git/git/git-20.png)



### 6.Git标签

#### 6.1.标签管理

**1.什么是Git标签**

Git中的标签用于标记某一提交点，唯一绑定一个固定的commitId，相当于为这次提交记录指定一个别名，方便提取文件。
可以为重要的版本打上标签，标签可以是一个对象，也可以是一个简单的指针，但是指针不会移动。



**2.为什么要使用标签**

在开发的一些关键节点,使用标签来记录这些关键节点, 例如发布版本, 有重大修改, 升级的时候, 会使用标签记录这些节点, 来长久标记项目中的关键历史时刻;

发布一个版本时，我们通常先在版本库中打一个标签（tag)，当该版本有急需要修复的bug的时候在该标签上做分支修改。

当然我们也可以针对某一次的提交打上一个标签，有点类似于给某次提交做个标记，比如1.0版本发布时打个标签叫tag1.0，2.0版本发布时打个标签叫tag2.0，因为每次版本提交的结果都是一连串的哈希码，不容易记忆，打上tag1.0,tag2.0这些具有某种含义的标签后，可以方便我们进行版本管理。



**3.怎么新建标签**

打标签流程如下：一个项目的一个版本完成后需要打标签做记录。

**开发项目---》完成任务----》打标签---》本地标签---》推送标签到远程服务器---》查看标签----》在标签基础上做bug修复**

#### 6.2.标签操作

**1.创建标签**

```shell
git tag <tag_name> #为当前分支指向的commit记录创建标签

git tag <tag_name> <hash_val> #为指定的commitId创建标签

git tag -a <tag_name> -m "msg" <hash_val> #创建标签同时添加说明信息
```

**Git 支持两种标签：轻量标签（lightweight）与附注标签（annotated）。**
**lightweight ：**轻量级标签就像是个不会变化的分支，实际上它就是个指向特定提交对象的引用。
**annotated：**含附注标签，实际上是存储在仓库中的一个独立对象，它有自身的校验和信息，包含着标签的名字，电子邮件地址和日期，以及标签说明，标签本身也允许使用 GNU Privacy Guard (GPG) 来签署或验证。

一般我们都建议使用含附注型的标签，以便保留相关信息；当然，如果只是临时性加注标签，或者不需要旁注额外信息，用轻量级标签也没问题。




**2.查看标签**

```shell
git tag #查看所有标签名称

git show <tag_name> #查看标签的详细信息(包含commit的信息)

git tag -ln [tag_name] #显示标签名及其描述信息
```



**3.远程推送标签**

```shell
git push <remote_name> <tag_name> #将标签推送到远程服务器

git push <remote_name> --tags #将本地的全部tag推送到远程服务器
```



**4.删除标签**

```shell
git tag -d <tag_name> #删除本地的标签

git push <remote_name> :refs/tags/<tag_name> #删除远程标签
```



**5.标签内容提取**

```shell
git archive --format=zip --output=src/xxx.zip <tag_name> 

#提取为zip格式，src可以是相对路径，也可以是绝对路径示例：在d盘下生成包含0.8标签内容的压缩包
git archive --format=zip --output=d:/v0.8.zip v0.8
```



**6.检出标签**

```shell
# 如果我们不想直接提取出标签的代码，而是希望在指定标签下继续进行开发，此时可以切换到标签。
git checkout <tag_name>  #切换到指定标签
```



如果你想查看某个标签所指向的文件版本，可以使用 `git checkout` 命令， 虽然这会使你的仓库处于“分离头指针（detached HEAD）”的状态——这个状态有些不好的副作用：

```shell
$ git checkout 2.0.0
Note: checking out '2.0.0'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch>

HEAD is now at 99ada87... Merge pull request #89 from schacon/appendix-final

$ git checkout 2.0-beta-0.1
Previous HEAD position was 99ada87... Merge pull request #89 from schacon/appendix-final
HEAD is now at df3f601... add atlas.json and cover image
```

在“分离头指针”状态下，如果你做了某些更改然后提交它们，标签不会发生变化， 但你的新提交将不属于任何分支，并且将无法访问，除非通过确切的提交哈希才能访问。 因此，如果你需要进行更改，比如你要修复旧版本中的错误，那么通常需要创建一个新分支：

```console
$ git checkout -b version2 v2.0.0
Switched to a new branch 'version2'
```

如果在这之后又进行了一次提交，`version2` 分支就会因为这个改动向前移动， 此时它就会和 `v2.0.0` 标签稍微有些不同，这时就要当心了





## 三、实践



### 1.常用Git命令

| **命令名称**                          | **作用**           |
| ------------------------------------- | ------------------ |
| git config --global user.name 用户名  | 设置用户签名       |
| git config --global user.email   邮箱 | 设置用户签名       |
| **git init**                          | **初始化本地库**   |
| **git  status**                       | **查看本地库状态** |
| **git add 文件名**                    | **添加到暂存区**   |
| **git commit -m "日志信息" 文件名**   | **提交到本地库**   |
| **git reflog**                        | **查看历史记录**   |
| **git reset --hard 版本号**           | **版本穿梭**       |



#### 1.1.设置用户签名

**1.基本语法**

```shell
git config --global user.name 用户名
git config --global user.email 邮箱

# 邮箱可以是虚拟邮箱
```

```shell
# 说明：

# 签名的作用是区分不同操作者身份。用户的签名信息在每一个版本的提交信息中能够看 到，以此确认本次提交是谁做的。Git  首次安装必须设置一下用户签名，否则无法提交代码。
# ※注意：这里设置用户签名和将来登录 GitHub（或其他代码托管中心）的账号没有任 何关系。
```



**2.案例实操**

```shell
git config --global user.name  xxxxxx
git config --global user.email xxxxxx@126.com
```

![](/images/devops/git/git/git-7.png)



**3.结果查看**

查看文件：.gitconfig

当前用户：Administrator

路径：C:\Users\Administrator

![](/images/devops/git/git/git-8.png)



```shell
# 查看配置信息

# 方法一
$ cat ~/.gitconfig
[credential]
        helper = manager
[user]
        name = xxxxxx
        email = xxxxxx@126.com
[filter "lfs"]
        required = true
        clean = git-lfs clean -- %f
        smudge = git-lfs smudge -- %f
        process = git-lfs filter-process


# 方法二
$ git config --list
diff.astextplain.textconv=astextplain
filter.lfs.clean=git-lfs clean -- %f
filter.lfs.smudge=git-lfs smudge -- %f
filter.lfs.process=git-lfs filter-process
filter.lfs.required=true
http.sslbackend=openssl
http.sslcainfo=C:/Program Files/Git/mingw64/ssl/certs/ca-bundle.crt
core.autocrlf=true
core.fscache=true
core.symlinks=true
pull.rebase=false
credential.helper=manager-core
credential.https://dev.azure.com.usehttppath=true
init.defaultbranch=master
credential.helper=manager

user.name=xxxxxx
user.email=xxxxxx@126.com

filter.lfs.required=true
filter.lfs.clean=git-lfs clean -- %f
filter.lfs.smudge=git-lfs smudge -- %f
filter.lfs.process=git-lfs filter-process
```



#### 1.2.初始化本地库

**1.基本语法**

```shell
git init
```

```shell
# git init 和 git init --bare的区别

# init：适用于本地仓库初始化，有完整的Git命令集，可以提交工作空间的代码和文件。
# init --bare:：适用于远程仓库初始化，默认没有工作空间。



# Git init
# 通常，我们初始化本地仓库时，使用git init：建立一个标准的Git仓库。
# 这样的仓库初始化后，其项目目录为工作空间，其下的.git目录是版本控制器。

# Git init --bare
# 通常，我们初始化远程服务器仓库时，使用git init --bare：建立一个“裸”的Git仓库。
#这样的仓库初始化后，其项目目录下就是标准仓库.git目录里的内容，没有工作空间。
# 这个仓库只保存git历史提交的版本信息，而不允许用户在上面进行各种git操作（如：push、commit操作）。但是，你依旧可以使用git show命令查看提交内容
```



**2.案例实操**

```shell
$ pwd
/d/dev-tct/Devops/dev/code

$ mkdir git-demo

$ cd git-demo/

# 初始化
$ git init
Initialized empty Git repository in D:/dev-tct/Devops/dev/code/git-demo/.git/
```



**3.结果查看**

```shell
# 查看
$ ll -a
total 4
drwxr-xr-x 1 Administrator 197121 0 Jan  5 17:19 ./
drwxr-xr-x 1 Administrator 197121 0 Jan  5 17:18 ../
drwxr-xr-x 1 Administrator 197121 0 Jan  5 17:19 .git/

$ cd .git/

$ ll
total 7
-rw-r--r-- 1 Administrator 197121  23 Jan  5 17:19 HEAD
-rw-r--r-- 1 Administrator 197121 112 Jan  5 17:19 config
-rw-r--r-- 1 Administrator 197121  73 Jan  5 17:19 description
drwxr-xr-x 1 Administrator 197121   0 Jan  5 17:19 hooks/
drwxr-xr-x 1 Administrator 197121   0 Jan  5 17:19 info/
drwxr-xr-x 1 Administrator 197121   0 Jan  5 17:19 objects/
drwxr-xr-x 1 Administrator 197121   0 Jan  5 17:19 refs/
```



#### 1.3.查看本地库状态

**1.基本语法**

```shell
git status
```



##### 1.3.1. 首次查看（工作区没有任何文件）

```shell
$ git status
On branch master

No commits yet

nothing to commit (create/copy files and use "git add" to track)

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
```



##### 1.3.2.新增文件（hello.txt）

```shell
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ vim hello.txt
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!


$ ll
total 1
-rw-r--r-- 1 Administrator 197121 250 Jan  5 20:49 hello.txt
```



##### 1.3.3.再次查看（检测到未追踪的文件）  

```shell
$ git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        hello.txt

nothing added to commit but untracked files present (use "git add" to track)

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
```

![](/images/devops/git/git/git-9.png)



#### 1.4.添加暂存区

##### 1.4.1.将工作区的文件添加到暂存区

**1.基本语法**

```shell
git add 文件名
```



**2.案例实操**

```shell
$ git add hello.txt
warning: LF will be replaced by CRLF in hello.txt.
The file will have its original line endings in your working directory
```



##### 1.4.2.查看状态（检测到暂存区有新文件）  

```shell
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   hello.txt
```

![](/images/devops/git/git/git-10.png)



#### 1.5.提交本地库  

##### 1.5.1.将暂存区的文件提交到本地库  

**1.基本语法**

```shell
git commit -m "日志信息" 文件名
```



**2.案例实操**

```shell
$ git commit -m "my first commit" hello.txt
warning: LF will be replaced by CRLF in hello.txt.
The file will have its original line endings in your working directory
[master (root-commit) 4490c7d] my first commit
 1 file changed, 10 insertions(+)
 create mode 100644 hello.txt
```



##### 1.5.2.查看状态（没有文件需要提交 ）  

```shell
$ git status
On branch master
nothing to commit, working tree clean
```



#### 1.6.修改文件（hello.txt）

```shell
$ vim hello.txt
hello git! hello devops!  2222222222
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
```



##### 1.6.1.查看状态（检测到工作区有文件被修改）

```shell
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   hello.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

![](/images/devops/git/git/git-11.png)



##### 1.6.2 将修改的文件再次添加暂存区  

```shell
$ git add hello.txt
warning: LF will be replaced by CRLF in hello.txt.
The file will have its original line endings in your working directory
```



##### 1.6.3 查看状态（工作区的修改添加到了暂存区）  

```shell
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   hello.txt
```

![](/images/devops/git/git/git-12.png)



##### 1.6.4.将暂存区的文件提交到本地库  

```shell
$ git commit -m "my second commit" hello.txt
warning: LF will be replaced by CRLF in hello.txt.
The file will have its original line endings in your working directory
[master ba38ff1] my second commit
 1 file changed, 1 insertion(+), 1 deletion(-)
```



##### 1.6.5.查看状态（没有文件需要提交 ）  

```shell
$ git status
On branch master
nothing to commit, working tree clean
```



#### 1.7.历史版本  

##### 1.7.1 查看历史版本  

**1.基本语法**

```shell
git reflog 查看版本信息
git log 查看版本详细信息
```



**2.案例实操**

```shell
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git reflog
ba38ff1 (HEAD -> master) HEAD@{0}: commit: my second commit
4490c7d HEAD@{1}: commit (initial): my first commit



Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git log
commit ba38ff17db9a56ad757333d708dae46efd1bf429 (HEAD -> master)
Author: hollysys <hollysys@126.com>
Date:   Wed Jan 5 21:24:20 2022 +0800

    my second commit

commit 4490c7da8d4c2f060db5080e0974013e2fd8dced
Author: hollysys <hollysys@126.com>
Date:   Wed Jan 5 21:08:57 2022 +0800

    my first commit
    

# git log --oneline --decorate --graph
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$  git log --oneline --decorate --graph
* 0f5c144 (HEAD -> master, git-demo/master) root ssh test
* 4bc730a linux ssh test
* f937aaf ssh test!
* 5ac9e20 (tag: v2.1.0-a) pull test!
*   b2d55cc (tag: v2.0.0-a) merge hot-fix
|\
| * 2f19631 (git-demo/hot-fix, hot-fix) hot-fix commit
* | 624e9d2 (tag: v1.0.0-l) my six commit
|/
* 85c57c0 (tag: v1.1.0-l) my five commit
* cae68ce my forth commit
* 3873d56 my forth commit
* ba38ff1 my second commit
* 4490c7d my first commit
```



##### 1.7.2 版本穿梭  

**1.基本语法**

```shell
git reset --hard 版本号
```

```shell
# 用 git log 命令查看：
# 每一个提交的版本都唯一对应一个 commit 版本号，
# 使用 git reset 命令退到上一个版本：
[root@qfedu.com ~]# git reset --hard HEAD^


[root@qfedu.com ~]# git reflog                    # 查看命令历史，以便确定要回到哪个版本
[root@qfedu.com ~]# git reset --hard commit_id    # 比如git reset --hard 3628164（不用全部输入，输入前几位即可）
```



**2.案例实操**

```shell
# --首先查看当前的历史记录， 可以看到当前是在 ba38ff1 这个版本
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git reflog
ba38ff1 (HEAD -> master) HEAD@{0}: commit: my second commit
4490c7d HEAD@{1}: commit (initial): my first commit


# --切换到 4490c7d 版本，也就是我们第一次提交的版本
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git reset --hard 4490c7d
HEAD is now at 4490c7d my first commit


# --切换完毕之后再查看历史记录，当前成功切换到了 4490c7d 版本
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git reflog
4490c7d (HEAD -> master) HEAD@{0}: reset: moving to 4490c7d
ba38ff1 HEAD@{1}: commit: my second commit
4490c7d (HEAD -> master) HEAD@{2}: commit (initial): my first commit


# --然后查看文件 hello.txt，发现文件内容已经变化
$ cat hello.txt
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
```



**Git 切换版本， 底层其实是移动的 HEAD 指针，具体原理如下图所示。**  

![](/images/devops/git/git/git-13.png)

```shell
# HEAD文件记录head指针
# 文件：git-demo\.git\HEAD
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo/.git (GIT_DIR!)
$ vim HEAD
ref: refs/heads/master


# 记录版本
# 文件：git-demo\.git\refs\heads\master
ba38ff17db9a56ad757333d708dae46efd1bf429
```



#### 1.8.删除文件

**1.基本语法**

```shell
git rm 文件名
git commit 提交
```



**2.案例实操**

```shell
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ ll
total 1
-rw-r--r-- 1 Administrator 197121   0 Jan 12 10:25 2.txt
-rw-r--r-- 1 Administrator 197121 382 Jan  9 12:29 hello.txt


Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git rm 2.txt
rm '2.txt'

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git commit -m "delete file"
[master 7e7d7c6] delete file
 1 file changed, 0 insertions(+), 0 deletions(-)
 delete mode 100644 2.txt
```



### 2.Git分支操作

| **命令名称**        | **作用**                     |
| ------------------- | ---------------------------- |
| git branch 分支名   | 创建分支                     |
| git branch -v       | 查看分支                     |
| git checkout 分支名 | 切换分支                     |
| git merge 分支名    | 把指定的分支合并到当前分支上 |



#### 2.1.查看分支  

**1.基本语法**

```shell
git branch -v
```



**2.案例实操**

```shell
#（*代表当前所在的分支）

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git branch -v
* master ba38ff1 my second commit
```



#### 2.2.创建分支  

**1.基本语法**

```shell
git branch 分支名
```



**2.案例实操**

```shell
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git branch hot-fix

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git branch -v
  hot-fix ba38ff1 my second commit
* master  ba38ff1 my second commit   # （*代表当前所在的分支）
```



#### 2.3.修改分支  

```shell
# --在 master 分支上做修改
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ vim hello.txt


# --添加暂存区
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git add hello.txt


# --提交本地库
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git commit -m "my forth commit" hello.txt
[master 3873d56] my forth commit
 1 file changed, 1 insertion(+), 1 deletion(-)


# --查看分支
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git branch -v
  hot-fix ba38ff1 my second commit   # （hot-fix 分支并未做任何改变）
* master  3873d56 my forth commit    #  当前 master 分支已更新为最新一次提交的版本）


# --查看 master 分支上的文件内容
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ cat hello.txt
hello git! hello devops!  2222222222
hello git! hello devops!  3333333333
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!  master test!
hello git! hello devops!
```



#### 2.4.切换分支  

**1.基本语法**

```shell
git checkout 分支名
```



**2.案例实操**

```shell
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git branch -v
  hot-fix ba38ff1 my second commit
* master  3873d56 my forth commit

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git checkout hot-fix
Switched to branch 'hot-fix'
M       hello.txt

# --发现当先分支已由 master 改为 hot-fix
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (hot-fix)
$ git branch -v
* hot-fix ba38ff1 my second commit
  master  3873d56 my forth commit


# --查看 hot-fix 分支上的文件内容发现与 master 分支上的内容不同
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (hot-fix)
$ cat hello.txt
hello git! hello devops!  2222222222
hello git! hello devops!  3333333333
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!


# --在 hot-fix 分支上做修改
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (hot-fix)
$ cat hello.txt
hello git! hello devops!  2222222222
hello git! hello devops!  3333333333
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!  hot-fix test!


# --添加暂存区
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (hot-fix)
$ git add hello.txt


# --提交本地库
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (hot-fix)
$ git commit -m "hot-fix commit" hello.txt
[hot-fix 2f19631] hot-fix commit
 1 file changed, 1 insertion(+), 1 deletion(-)
```



#### 2.5.合并分支  

**1.基本语法**

```shell
git merge 分支名
```



**2.案例实操**

```shell
# 在 master 分支上合并 hot-fix 分支


Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (hot-fix)
$ git branch -v
* hot-fix 2f19631 hot-fix commit
  master  624e9d2 my six commit

# 切换到master分支
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (hot-fix)
$ git checkout master
Switched to branch 'master'

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git branch -v
  hot-fix 2f19631 hot-fix commit
* master  624e9d2 my six commit


# 合并分支
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git merge hot-fix
Auto-merging hello.txt
CONFLICT (content): Merge conflict in hello.txt
Automatic merge failed; fix conflicts and then commit the result.

# 产生冲突：CONFLICT (content): Merge conflict in hello.txt
```



#### 2.6.产生冲突

```shell
# 冲突产生的表现： 后面状态为 MERGING
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master|MERGING)


# hello.txt
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master|MERGING)
$ cat hello.txt
hello git! hello devops!  2222222222
hello git! hello devops!  3333333333
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
<<<<<<< HEAD
hello git! hello devops!  master test!
hello git! hello devops!
=======
hello git! hello devops!
hello git! hello devops!  hot-fix test!
>>>>>>> hot-fix
```

冲突产生的原因：
合并分支时，两个分支在**同一个文件的同一个位置**有两套完全不同的修改。 Git 无法替
我们决定使用哪一个。必须**人为决定**新代码内容。
查看状态（检测到有文件有两处修改）  

```shell
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master|MERGING)
$ git status
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   hello.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

![](/images/devops/git/git/git-16.png)



#### 2.7.解决冲突

**1.编辑有冲突的文件，删除特殊符号，决定要使用的内容**  

```shell
# 特殊符号： <<<<<<< HEAD 当前分支的代码 ======= 合并过来的代码 >>>>>>> hot-fix


Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master|MERGING)
$ cat hello.txt
hello git! hello devops!  2222222222
hello git! hello devops!  3333333333
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!  master test!
hello git! hello devops!  hot-fix test!
```



**2.添加到暂存区**

```shell
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master|MERGING)
$ git add hello.txt
```



**3.执行提交**

```shell
# 注意： 此时使用 git commit 命令时不能带文件名

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master|MERGING)
$ git commit -m "merge hot-fix"
[master b2d55cc] merge hot-fix


# --发现后面 MERGING 消失， 变为正常
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git status
On branch master
nothing to commit, working tree clean
```





### 3.远程仓库操作

| **命令名称**                             | **作用**                                                     |
| ---------------------------------------- | ------------------------------------------------------------ |
| git remote -v                            | 查看当前所有远程地址别名                                     |
| git remote add 别名 远程地址             | 起别名                                                       |
| **git push   别名 分支**                 | **推送本地分支上的内容到远程仓库**                           |
| **git clone 远程地址**                   | **将远程仓库的内容克隆到本地**                               |
| **git pull   远程库地址别名 远程分支名** | **将远程仓库对于分支最新内容拉下来后与 当前本地分支直接合并** |



#### 3.1.创建远程仓库

**GitLab创建远程仓库**

```shell
# 远程仓库地址
http://182.92.210.65:18080/users/sign_in
root
******

zhangsan
zhangsan123


# 创建仓库：git-demo
http://182.92.210.65:18080/hs-group/git-demo.git
git@182.92.210.65:hs-group/git-demo.git
```



#### 3.2.创建远程仓库别名

**1.基本语法**

```shell
git remote -v 查看当前所有远程地址别名
git remote add 别名 远程地址
git remote remove 别名
```



**2.案例实操**

```shell
# 查看别名
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git remote -v


# 添加别名
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git remote add git-demo http://182.92.210.65:18080/hs-group/git-demo.git

# 查看
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git remote -v
git-demo        http://182.92.210.65:18080/hs-group/git-demo.git (fetch)
git-demo        http://182.92.210.65:18080/hs-group/git-demo.git (push)
```



#### 3.3.推送本地分支到远程仓库  

**1.基本语法**

```shell
git push 别名 分支
```



**2.案例实操**

```shell
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git push git-demo master
git: 'credential-manager' is not a git command. See 'git --help'.

The most similar command is
        credential-manager-core
Enumerating objects: 20, done.
Counting objects: 100% (20/20), done.
Delta compression using up to 8 threads
Compressing objects: 100% (14/14), done.
Writing objects: 100% (20/20), 1.55 KiB | 791.00 KiB/s, done.
Total 20 (delta 7), reused 0 (delta 0), pack-reused 0
To http://182.92.210.65:18080/hs-group/git-demo.git
 * [new branch]      master -> master


# 首次访问
# 弹出登陆登陆对话框，输入
# 用户名：zhangsan
# 密码：******
```



![](/images/devops/git/git/git-21.png)

**推送成功**

![](/images/devops/git/git/git-22.png)



#### 3.4.Windows凭据管理器

使用账号密码登陆代码仓库，会在Windows凭据管理器中生成备份，后面不需要再输入账号密码，如果要更换账号，需要删除当前凭据。

![](/images/devops/git/git/git-23.png)



#### 3.5.克隆远程仓库到本地  

**1.基本语法**

```shell
git clone 远程地址
```



**2.案例实操**

```shell
#  克隆
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/gitlab
$ git clone http://182.92.210.65:18080/hs-group/git-demo.git
Cloning into 'git-demo'...
git: 'credential-manager' is not a git command. See 'git --help'.

The most similar command is
        credential-manager-core
remote: Enumerating objects: 20, done.
remote: Counting objects: 100% (20/20), done.
remote: Compressing objects: 100% (14/14), done.
remote: Total 20 (delta 7), reused 0 (delta 0)
Unpacking objects: 100% (20/20), 1.53 KiB | 35.00 KiB/s, done.

# 查看
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/gitlab
$ ll
total 0
drwxr-xr-x 1 Administrator 197121 0 Jan  7 17:58 git-demo/

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/gitlab
$ cd git-demo/

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/gitlab/git-demo (master)
$ ll
total 1
-rw-r--r-- 1 Administrator 197121 315 Jan  7 17:58 hello.txt

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/gitlab/git-demo (master)
$ cat hello.txt
hello git! hello devops!  2222222222
hello git! hello devops!  3333333333
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!  master test!
hello git! hello devops!  hot-fix test!


Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/gitlab/git-demo (master)
$ git remote -v
origin  http://182.92.210.65:18080/hs-group/git-demo.git (fetch)
origin  http://182.92.210.65:18080/hs-group/git-demo.git (push)
```



#### 3.6.拉取远程库内容

**1.基本语法**

```shell
git pull 远程库地址别名 远程分支名
```



**2.案例实操**

```shell
# 拉取
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git pull git-demo master
git: 'credential-manager' is not a git command. See 'git --help'.

The most similar command is
        credential-manager-core
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 1), reused 0 (delta 0)
Unpacking objects: 100% (3/3), 256 bytes | 21.00 KiB/s, done.
From http://182.92.210.65:18080/hs-group/git-demo
 * branch            master     -> FETCH_HEAD
   b2d55cc..5ac9e20  master     -> git-demo/master
Updating b2d55cc..5ac9e20
Fast-forward
 hello.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)


# 查看
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ cat hello.txt
hello git! hello devops!  2222222222
hello git! hello devops!  3333333333
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!  master test!
hello git! hello devops!  hot-fix test!
pull test!!
```



### 4.SSH 免密登录  

#### 4.1.Windows免密登录

**1.删除现有Key**

访问目录：C:\Users\Administrator\ .ssh，删除公钥：id_rsa.pub ，私钥：id_rsa

![](/images/devops/git/git/git-25.png)



**2.生成.ssh 秘钥**  

```shell
# 在目录：C:\Users\Administrator
# 运行命令生成.ssh 秘钥目录[注意：这里-C 这个参数是大写的 C]  
ssh-keygen -t rsa -C zhangsan@126.com
```

```shell
Administrator@DESKTOP-AV12NNP MINGW64 ~
$ pwd
/c/Users/Administrator

# 三次回车
Administrator@DESKTOP-AV12NNP MINGW64 ~
$ ssh-keygen -t rsa -C zhangsan@126.com
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/Administrator/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/Administrator/.ssh/id_rsa
Your public key has been saved in /c/Users/Administrator/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:5QMY36Tm3sdI5EnPK9SXCeileELzBvuXGoE1DnOmtWI zhangsan@126.com
The key's randomart image is:
+---[RSA 3072]----+
|      .   .      |
|       + + .     |
|      . @ % o    |
|       + ^ X . o |
|        E ^ + +  |
|       o X = +   |
|        . * *    |
|           *     |
|          .      |
+----[SHA256]-----+
```



**3.查看公钥**

```shell
Administrator@DESKTOP-AV12NNP MINGW64 ~
$ pwd
/c/Users/Administrator

Administrator@DESKTOP-AV12NNP MINGW64 ~
$ cd .ssh

Administrator@DESKTOP-AV12NNP MINGW64 ~/.ssh
$ ll
total 5
-rw-r--r-- 1 Administrator 197121 2602 Jan  7 18:51 id_rsa
-rw-r--r-- 1 Administrator 197121  570 Jan  7 18:51 id_rsa.pub


# 查看公钥
Administrator@DESKTOP-AV12NNP MINGW64 ~/.ssh
$ cat id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDWJgwiZvuLP9OwHboesmnB235AaORdFRymeTVTl+RckZW3QNFl6AAhojJk04Sp9Jsg+l5pzHhlr/40FSZjtxZMraKaEdeMl3yr1Ph7V+wHxTl2tf24XpbOYALFySreo02+ZnyHAmjj+UOi950r/3Gh/QmATx5uId63ehtQ/aRhMHwZCtE3j1X2vPk0UFh06xxxPqeaQOLk1w5UQJ7Co9xdsgW9yzLTd6VGCgVApGB0wdBBLAgA7dYFs3vpU/H68GuYyolwCEsWqWv3CDeA99H80WUFeCHsXwBWcC1OYO1IQonjSmxf8ybDKNGpP+Tlpb8AvL8EWTPoluLFjTMQ0i1Jpux2n+p2nwvyqXGeAKvhjJPZx2jhnesp7rc02nfzdDFQ57eNqqhvnBdm1xa9SRcUVToA4fkxlfU20qT+HZ/ylw4Xzepwl8TSMDYvwv9Rcc1xTR/a01hEFCk82QAgmjJSzsXsh2s1emZOlY/lw3Tmz8VfzGFw9ZqJqVKm25zdeec= zhangsan@126.com
```

```shell
# 公钥

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDWJgwiZvuLP9OwHboesmnB235AaORdFRymeTVTl+RckZW3QNFl6AAhojJk04Sp9Jsg+l5pzHhlr/40FSZjtxZMraKaEdeMl3yr1Ph7V+wHxTl2tf24XpbOYALFySreo02+ZnyHAmjj+UOi950r/3Gh/QmATx5uId63ehtQ/aRhMHwZCtE3j1X2vPk0UFh06xxxPqeaQOLk1w5UQJ7Co9xdsgW9yzLTd6VGCgVApGB0wdBBLAgA7dYFs3vpU/H68GuYyolwCEsWqWv3CDeA99H80WUFeCHsXwBWcC1OYO1IQonjSmxf8ybDKNGpP+Tlpb8AvL8EWTPoluLFjTMQ0i1Jpux2n+p2nwvyqXGeAKvhjJPZx2jhnesp7rc02nfzdDFQ57eNqqhvnBdm1xa9SRcUVToA4fkxlfU20qT+HZ/ylw4Xzepwl8TSMDYvwv9Rcc1xTR/a01hEFCk82QAgmjJSzsXsh2s1emZOlY/lw3Tmz8VfzGFw9ZqJqVKm25zdeec= zhangsan@126.com
```



#### 4.2.GitLab配置SSH

**1.查看你生成的公钥：**

vim id_rsa.pub

就可以查看到你的公钥

**2.登陆GitLab账号，点击用户图像，然后 Settings -> 左栏点击 SSH keys**

![](/images/devops/git/git/git-24.png)

**3.复制公钥内容，粘贴进“Key”文本区域内，取名字**

**4.点击Add Key**

![](/images/devops/git/git/git-26.png)

![](/images/devops/git/git/git-27.png)



#### 4.3.测试SSH

```shell
# 克隆
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/gitlab
$ git clone git@182.92.210.65:hs-group/git-demo.git
Cloning into 'git-demo'...
The authenticity of host '182.92.210.65 (182.92.210.65)' can't be established.
ED25519 key fingerprint is SHA256:nW0TUTa8Vm+Vk52njgK9af9OjVPWwtrBXgMr0/Rbq5M.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '182.92.210.65' (ED25519) to the list of known hosts.
remote: Enumerating objects: 23, done.
remote: Counting objects: 100% (23/23), done.
remote: Compressing objects: 100% (16/16), done.
remote: Total 23 (delta 8), reused 0 (delta 0)
Receiving objects: 100% (23/23), done.
Resolving deltas: 100% (8/8), done.


# 查看
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/gitlab
$ ll
total 0
drwxr-xr-x 1 Administrator 197121 0 Jan  7 19:12 git-demo/

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/gitlab
$ cd git-demo/

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/gitlab/git-demo (master)
$ ll
total 1
-rw-r--r-- 1 Administrator 197121 326 Jan  7 19:12 hello.txt


Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/gitlab/git-demo (master)
$ git push origin master
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 275 bytes | 275.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0), pack-reused 0
To 182.92.210.65:hs-group/git-demo.git
   5ac9e20..f937aaf  master -> master
```



#### 4.4.Linux免密登录

**1.Linux安装Git**

```shell
# yum install git -y

# git version
```



**2.Linux添加用户**

```shell
# useradd lisi
# su - lisi
```

配置Git

```shell
[lisi@VM-24-10-centos ~]$ git config --global user.email "lisi@126.com"
[lisi@VM-24-10-centos ~]$ git config --global user.name "lisi"

cat ~/.gitconfig
```



**3.生成秘钥**

```shell
$ ssh-keygen
$ ssh-keygen -C lisi@126.com


[lisi@VM-24-10-centos ~]$ ssh-keygen -C lisi@126.com
Generating public/private rsa key pair.
Enter file in which to save the key (/home/lisi/.ssh/id_rsa): 
Created directory '/home/lisi/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/lisi/.ssh/id_rsa.
Your public key has been saved in /home/lisi/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:zFhOiY5Ohx4D6oGkjI1XMAIBTIQfQabswsY+Rm3PQRE lisi@126.com
The key's randomart image is:
+---[RSA 2048]----+
|%=*. E.          |
|o=.o  .. .       |
|.+..... +        |
|X+oo.+ B         |
|BOoo*.+ S        |
|*.o+o+.          |
| =  oo           |
|. .              |
|                 |
+----[SHA256]-----+


[lisi@VM-24-10-centos ~]$ cat .ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDQPp5Kad3m2sMCUJ3A+nmOUorGKPEBRgfgNYui1ofMfCcARcgHOHjyuZ1OTXWrcCpL00hKHcx0PbkNM6juMa4QUm+yR9W+yQg/v7CA8zEAZp2gIYBnIZAylr52z4/KfyoZ1wReMdZpvtGUdZUxkaaU6alTLuX7nvYA6e9XjHFg148mb/jHWO/+tGJOLAZTKxOWWDRA34iVx2Db9ChAS8vtL94R+/v8bditUKv1tZuXSdzIG3NCYqWs4GMwscA3OaqyRxtpY/ewHWmjO+5JDg+7ujqeyKM81SoR6gUgRMxop8zn0mXob8d+QZl57FKJ9533xLkXmLs9YSAsHzcGIJaR lisi@126.com
```



**4.GitLab配置SSH**

参考 **4.2.GitLab配置SSH**



**5.克隆项目**

```shell
# 克隆
[lisi@VM-24-10-centos ~]$ git clone git@182.92.210.65:hs-group/git-demo.git
Cloning into 'git-demo'...
The authenticity of host '182.92.210.65 (182.92.210.65)' can't be established.
ECDSA key fingerprint is SHA256:BIIdOhKgAbqGX4cbl9AQHC3s/WshohtuI1mhoH3Kres.
ECDSA key fingerprint is MD5:92:69:1e:70:08:c5:7a:25:cb:ee:e1:5c:ab:3b:67:9d.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '182.92.210.65' (ECDSA) to the list of known hosts.
remote: Enumerating objects: 26, done.
remote: Counting objects: 100% (26/26), done.
remote: Compressing objects: 100% (18/18), done.
remote: Total 26 (delta 9), reused 0 (delta 0)
Receiving objects: 100% (26/26), done.
Resolving deltas: 100% (9/9), done.

# 查看
[lisi@VM-24-10-centos ~]$ ll
total 4
drwxrwxr-x 3 lisi lisi 4096 Jan  8 20:05 git-demo
[lisi@VM-24-10-centos ~]$ cd git-demo/
[lisi@VM-24-10-centos git-demo]$ ll
total 4
-rw-rw-r-- 1 lisi lisi 327 Jan  8 20:05 hello.txt
```



**6.推送本地分支到远程仓库** 

```shell
[lisi@VM-24-10-centos git-demo]$ vim hello.txt 
[lisi@VM-24-10-centos git-demo]$ cat hello.txt 
hello git! hello devops!  2222222222
hello git! hello devops!  3333333333
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!  master test!
hello git! hello devops!  hot-fix test!
pull test!!
ssh test!!!
linux ssh test!!!


[lisi@VM-24-10-centos git-demo]$ git remote -v
origin	git@182.92.210.65:hs-group/git-demo.git (fetch)
origin	git@182.92.210.65:hs-group/git-demo.git (push)

# 推送
[lisi@VM-24-10-centos git-demo]$ git push git@182.92.210.65:hs-group/git-demo.git master
Counting objects: 5, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 280 bytes | 0 bytes/s, done.
Total 3 (delta 1), reused 0 (delta 0)
To git@182.92.210.65:hs-group/git-demo.git
   f937aaf..4bc730a  master -> master
```

从GitLab页面查看推送成功。



### 5.Git标签

#### 5.1.创建标签

**Git 支持两种标签：轻量标签（lightweight）与附注标签（annotated）。**

**1.轻量标签**

```shell
git tag <tag_name> #为当前分支指向的commit记录创建标签

git tag <tag_name> <hash_val> #为指定的commitId创建标签
```

```shell
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git reflog
0f5c144 (HEAD -> master, git-demo/master) HEAD@{0}: pull git-demo master: Fast-forward
5ac9e20 HEAD@{1}: pull git-demo master: Fast-forward
b2d55cc HEAD@{2}: commit (merge): merge hot-fix
624e9d2 HEAD@{3}: checkout: moving from hot-fix to master
2f19631 (git-demo/hot-fix, hot-fix) HEAD@{4}: commit: hot-fix commit
85c57c0 HEAD@{5}: checkout: moving from master to hot-fix
624e9d2 HEAD@{6}: commit: my six commit
85c57c0 HEAD@{7}: commit: my five commit
cae68ce HEAD@{8}: commit: my forth commit
3873d56 HEAD@{9}: checkout: moving from hot-fix to master
3873d56 HEAD@{10}: checkout: moving from master to hot-fix
3873d56 HEAD@{11}: checkout: moving from hot-fix to master
ba38ff1 HEAD@{12}: checkout: moving from master to hot-fix
3873d56 HEAD@{13}: commit: my forth commit
ba38ff1 HEAD@{14}: reset: moving to ba38ff1
4490c7d HEAD@{15}: reset: moving to 4490c7d
ba38ff1 HEAD@{16}: commit: my second commit
4490c7d HEAD@{17}: commit (initial): my first commit


# 打标签
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git tag v1.0.0-l 624e9d2

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git tag v1.1.0-l 85c57c0

# 查看
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git tag
v1.0.0-l
v1.1.0-l


Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git reflog
0f5c144 (HEAD -> master, git-demo/master) HEAD@{0}: pull git-demo master: Fast-forward
5ac9e20 HEAD@{1}: pull git-demo master: Fast-forward
b2d55cc HEAD@{2}: commit (merge): merge hot-fix
624e9d2 (tag: v1.0.0-l) HEAD@{3}: checkout: moving from hot-fix to master
2f19631 (git-demo/hot-fix, hot-fix) HEAD@{4}: commit: hot-fix commit

85c57c0 (tag: v1.1.1-l, tag: v1.1.0-l) HEAD@{5}: checkout: moving from master to hot-fix
624e9d2 (tag: v1.0.0-l) HEAD@{6}: commit: my six commit
85c57c0 (tag: v1.1.1-l, tag: v1.1.0-l) HEAD@{7}: commit: my five commit

cae68ce HEAD@{8}: commit: my forth commit
3873d56 HEAD@{9}: checkout: moving from hot-fix to master
3873d56 HEAD@{10}: checkout: moving from master to hot-fix
3873d56 HEAD@{11}: checkout: moving from hot-fix to master
ba38ff1 HEAD@{12}: checkout: moving from master to hot-fix
3873d56 HEAD@{13}: commit: my forth commit
ba38ff1 HEAD@{14}: reset: moving to ba38ff1
4490c7d HEAD@{15}: reset: moving to 4490c7d
ba38ff1 HEAD@{16}: commit: my second commit
4490c7d HEAD@{17}: commit (initial): my first commit
```



**2.附注标签**

**实际开发建议使用附注标签**

```shell
it tag -a <tag_name> -m "msg" <hash_val> #创建标签同时添加说明信息
```

```shell
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git reflog
0f5c144 (HEAD -> master, git-demo/master) HEAD@{0}: pull git-demo master: Fast-forward
5ac9e20 HEAD@{1}: pull git-demo master: Fast-forward
b2d55cc HEAD@{2}: commit (merge): merge hot-fix
624e9d2 (tag: v1.0.0-l) HEAD@{3}: checkout: moving from hot-fix to master
2f19631 (git-demo/hot-fix, hot-fix) HEAD@{4}: commit: hot-fix commit
85c57c0 (tag: v1.1.1-l, tag: v1.1.0-l) HEAD@{5}: checkout: moving from master to hot-fix
624e9d2 (tag: v1.0.0-l) HEAD@{6}: commit: my six commit
85c57c0 (tag: v1.1.1-l, tag: v1.1.0-l) HEAD@{7}: commit: my five commit
cae68ce HEAD@{8}: commit: my forth commit
3873d56 HEAD@{9}: checkout: moving from hot-fix to master
3873d56 HEAD@{10}: checkout: moving from master to hot-fix
3873d56 HEAD@{11}: checkout: moving from hot-fix to master
ba38ff1 HEAD@{12}: checkout: moving from master to hot-fix
3873d56 HEAD@{13}: commit: my forth commit
ba38ff1 HEAD@{14}: reset: moving to ba38ff1
4490c7d HEAD@{15}: reset: moving to 4490c7d
ba38ff1 HEAD@{16}: commit: my second commit
4490c7d HEAD@{17}: commit (initial): my first commit


# 打标签
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git tag -a v2.0.0-a -m "fuzhu--1" b2d55cc

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git tag -a v2.1.0-a -m "fuzhu--1" 5ac9e20

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git tag
v1.0.0-l
v1.1.0-l
v1.1.1-l
v2.0.0-a
v2.1.0-a

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git reflog
0f5c144 (HEAD -> master, git-demo/master) HEAD@{0}: pull git-demo master: Fast-forward


5ac9e20 (tag: v2.1.0-a) HEAD@{1}: pull git-demo master: Fast-forward
b2d55cc (tag: v2.0.0-a) HEAD@{2}: commit (merge): merge hot-fix


624e9d2 (tag: v1.0.0-l) HEAD@{3}: checkout: moving from hot-fix to master
2f19631 (git-demo/hot-fix, hot-fix) HEAD@{4}: commit: hot-fix commit
85c57c0 (tag: v1.1.1-l, tag: v1.1.0-l) HEAD@{5}: checkout: moving from master to hot-fix
624e9d2 (tag: v1.0.0-l) HEAD@{6}: commit: my six commit
85c57c0 (tag: v1.1.1-l, tag: v1.1.0-l) HEAD@{7}: commit: my five commit
cae68ce HEAD@{8}: commit: my forth commit
3873d56 HEAD@{9}: checkout: moving from hot-fix to master
3873d56 HEAD@{10}: checkout: moving from master to hot-fix
3873d56 HEAD@{11}: checkout: moving from hot-fix to master
ba38ff1 HEAD@{12}: checkout: moving from master to hot-fix
3873d56 HEAD@{13}: commit: my forth commit
ba38ff1 HEAD@{14}: reset: moving to ba38ff1
4490c7d HEAD@{15}: reset: moving to 4490c7d
ba38ff1 HEAD@{16}: commit: my second commit
4490c7d HEAD@{17}: commit (initial): my first commit
```



#### 5.2.查看标签

```shell
git tag #查看所有标签名称

git show <tag_name> #查看标签的详细信息(包含commit的信息)

git tag -ln [tag_name] #显示标签名及其描述信息
```

```shell
# 查看所有标签名称
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git tag
v1.0.0-l
v1.1.0-l
v1.1.1-l
v2.0.0-a
v2.1.0-a


# 查看附注标签
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git show v2.0.0-a
tag v2.0.0-a
Tagger: hollysys <hollysys@126.com>
Date:   Tue Jan 11 15:01:55 2022 +0800

fuzhu--1

commit b2d55cce40a242ac7b28b2dcc2902baabbfbf068 (tag: v2.0.0-a)
Merge: 624e9d2 2f19631
Author: hollysys <hollysys@126.com>
Date:   Thu Jan 6 12:17:40 2022 +0800

    merge hot-fix

diff --cc hello.txt
index 813f7f0,ac29ef6..1feec1f
--- a/hello.txt
+++ b/hello.txt
@@@ -6,5 -6,5 +6,6 @@@ hello git! hello devops
  hello git! hello devops!
  hello git! hello devops!
  hello git! hello devops!
 -hello git! hello devops!
 +hello git! hello devops!  master test!
- hello git! hello devops!
+ hello git! hello devops!  hot-fix test!
++


# 查看轻量标签
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git show v1.0.0-l
commit 624e9d274dd804b73335bfec332b6df57915431d (tag: v1.0.0-l)
Author: hollysys <hollysys@126.com>
Date:   Thu Jan 6 11:52:20 2022 +0800

    my six commit

diff --git a/hello.txt b/hello.txt
index b0f97d4..813f7f0 100644
--- a/hello.txt
+++ b/hello.txt
@@ -6,5 +6,5 @@ hello git! hello devops!
 hello git! hello devops!
 hello git! hello devops!
 hello git! hello devops!
-hello git! hello devops!
+hello git! hello devops!  master test!
 hello git! hello devops!
 

# 显示标签名及其描述信息
 Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git tag -ln v2.0.0-a
v2.0.0-a        fuzhu--1

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git tag -ln v1.0.0-l
v1.0.0-l        my six commit
```



#### 5.3.远程推送标签

```shell
git push <remote_name> <tag_name> #将标签推送到远程服务器

git push <remote_name> --tags #将本地的全部tag推送到远程服务器（未推送过的标签）
```

```shell
# 将标签推送到远程服务器
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git push git-demo v2.0.0-a
git: 'credential-manager' is not a git command. See 'git --help'.

The most similar command is
        credential-manager-core
Enumerating objects: 1, done.
Counting objects: 100% (1/1), done.
Writing objects: 100% (1/1), 156 bytes | 156.00 KiB/s, done.
Total 1 (delta 0), reused 0 (delta 0), pack-reused 0
To http://182.92.210.65:18080/hs-group/git-demo.git
 * [new tag]         v2.0.0-a -> v2.0.0-a
 
 
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git push git-demo v1.1.0-l
git: 'credential-manager' is not a git command. See 'git --help'.

The most similar command is
        credential-manager-core
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
To http://182.92.210.65:18080/hs-group/git-demo.git
 * [new tag]         v1.1.0-l -> v1.1.0-l
 

# 将本地的全部tag推送到远程服务器
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git push git-demo --tags
git: 'credential-manager' is not a git command. See 'git --help'.

The most similar command is
        credential-manager-core
Enumerating objects: 1, done.
Counting objects: 100% (1/1), done.
Writing objects: 100% (1/1), 158 bytes | 158.00 KiB/s, done.
Total 1 (delta 0), reused 0 (delta 0), pack-reused 0
To http://182.92.210.65:18080/hs-group/git-demo.git
 * [new tag]         v1.0.0-l -> v1.0.0-l
 * [new tag]         v1.1.1-l -> v1.1.1-l
 * [new tag]         v2.1.0-a -> v2.1.0-a
```



#### 5.4.删除标签

```shell
git tag -d <tag_name> #删除本地的标签

git push <remote_name> :refs/tags/<tag_name> #删除远程标签
```

```shell
#删除本地的标签
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git tag -d v1.1.1-l
Deleted tag 'v1.1.1-l' (was 85c57c0)


Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git tag
v1.0.0-l
v1.1.0-l
v2.0.0-a
v2.1.0-a
```

```shell
# 删除远程标签
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-demo (master)
$ git push git-demo :refs/tags/v1.1.1-l
git: 'credential-manager' is not a git command. See 'git --help'.

The most similar command is
        credential-manager-core
To http://182.92.210.65:18080/hs-group/git-demo.git
 - [deleted]         v1.1.1-l
```



#### 5.5.检出标签

```shell
git checkout <tag_name>  #切换到指定标签
```

```shell
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/gitlab/test/git-demo (master)
$ cat hello.txt
hello git! hello devops!  2222222222
hello git! hello devops!  3333333333
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!  master test!
hello git! hello devops!  hot-fix test!
pull test!!
ssh test!!!
linux ssh test!!!
linux root ssh test!!!


# 检出标签
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/gitlab/test/git-demo (master)
$ git checkout v1.0.0-l
Note: switching to 'v1.0.0-l'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at 624e9d2 my six commit

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/gitlab/test/git-demo ((v1.0.0-l))
$ cat hello.txt
hello git! hello devops!  2222222222
hello git! hello devops!  3333333333
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!  master test!
hello git! hello devops
```



#### 5.6.在指定标签继续开发

- 先 git clone 整个仓库，然后 git checkout tag_name 就可以取得 tag 对应的代码了。
- 但是这时候 git 可能会提示你当前处于一个“detached HEAD” 状态，因为 tag 相当于是一个快照，是不能更改它的代码的，如果要在 tag 代码的基础上做修改，你需要一个分支：
  git checkout -b branch_name tag_name
  这样会从 tag 创建一个分支，然后就和普通的 git 操作一样了。

```shell
# 在指定标签下创建新分支
git checkout -b branch_name tag_name
```

```shell
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/gitlab/test/git-demo (master)
$ git branch
* master

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/gitlab/test/git-demo (master)
$ git remote -v
origin  git@182.92.210.65:hs-group/git-demo.git (fetch)
origin  git@182.92.210.65:hs-group/git-demo.git (push)

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/gitlab/test/git-demo (master)
$ git tag
v1.0.0-l
v1.1.0-l
v2.0.0-a
v2.1.0-a


# 创建新分支
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/gitlab/test/git-demo (master)
$ git checkout -b branch_tag v1.0.0-l
Switched to a new branch 'branch_tag'


Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/gitlab/test/git-demo (branch_tag)
$ cat hello.txt
hello git! hello devops!  2222222222
hello git! hello devops!  3333333333
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!
hello git! hello devops!  master test!
hello git! hello devops!

# 查看分支
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/gitlab/test/git-demo (branch_tag)
$ git branch -v
* branch_tag 624e9d2 my six commit
  master     0f5c144 root ssh test
  
# 按普通分支继续研发
```



