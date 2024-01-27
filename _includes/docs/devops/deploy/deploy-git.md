* TOC
{:toc}



## 一、概述



**官网地址**

```shell
# 官网地址

https://git-scm.com/
https://git-scm.com/downloads
```



**下载安装文件**

```shell
https://git-scm.com/downloads

# windows
https://git-scm.com/download/win

# linux
https://git-scm.com/download/linux
```



![](/images/devops/deploy/deploy-git/git-1.png)

<img src="/images/devops/deploy/deploy-git/git-2.png" style="zoom:67%;" />





## 二、Windows安装部署



### 1.下载安装文件

```shell
# 下载版本

Git-2.31.1-64-bit.exe
```



### 2.安装Git

**1.查看 GNU 协议，可以直接点击下一步**



![img](/images/devops/deploy/deploy-git/git-3.png)

 

**2.选择 Git 安装位置，要求是非中文并且没有空格的目录，然后下一步**

 

![img](/images/devops/deploy/deploy-git/git-4.png)



**3.Git 选项配置，推荐默认设置，然后下一步**

 

![img](/images/devops/deploy/deploy-git/git-5.png)



**4.Git 安装目录名，不用修改，直接点击下一步**

 

![img](/images/devops/deploy/deploy-git/git-6.png)



 **5.Git编译器，不用修改，直接点击下一步**



![img](/images/devops/deploy/deploy-git/git-7.png)

 

**6.默认分支名设置，选择让 Git 决定，分支名默认为 master，下一步**

 

![img](/images/devops/deploy/deploy-git/git-8.png)



 **7.Git环境变量，不用修改，直接点击下一步**

 

![img](/images/devops/deploy/deploy-git/git-9.png)

 

**8.选择后台客户端连接协议，选默认值 OpenSSL，然后下一步**

 

![img](/images/devops/deploy/deploy-git/git-10.png)



**9.配置 Git 文件的行末换行符，Windows 使用 CRLF，Linux 使用 LF，选择第一个自动 转换，然后继续下一步**

 

![img](/images/devops/deploy/deploy-git/git-11.png)



**10.选择 Git 终端类型，选择默认的 Git Bash 终端，然后继续下一步**

 

![img](/images/devops/deploy/deploy-git/git-12.png)

 

**11.选择 Git pull 合并的模式，选择默认，然后下一步**

 

![img](/images/devops/deploy/deploy-git/git-13.png)



**12.选择 Git 的凭据管理器，选择默认的跨平台的凭据管理器，然后下一步**

 

![img](/images/devops/deploy/deploy-git/git-14.png)

 

**13.其他配置，选择默认设置，然后下一步**

 

![img](/images/devops/deploy/deploy-git/git-15.png)



**14.实验室功能，技术还不成熟，有已知的 bug，不要勾选，然后点击右下角的 Install**

**按钮，开始安装 Git**

 

![img](/images/devops/deploy/deploy-git/git-16.png)



**15.点击 Finsh 按钮，Git 安装成功**



![img](/images/devops/deploy/deploy-git/git-17.png)

 

### 3.测试

右键任意位置，在右键菜单里选择 Git Bash Here 即可打开 Git Bash 命令行终端。

 

![img](/images/devops/deploy/deploy-git/git-18.png)



在 Git Bash 终端里输入 git --version 查看 git 版本，如图所示，说明 Git 安装成功。



![img](/images/devops/deploy/deploy-git/git-19.png)

 

 ```shell
 $ git
 usage: git [--version] [--help] [-C <path>] [-c <name>=<value>]
            [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
            [-p | --paginate | -P | --no-pager] [--no-replace-objects] [--bare]
            [--git-dir=<path>] [--work-tree=<path>] [--namespace=<name>]
            <command> [<args>]
 
 These are common Git commands used in various situations:
 
 start a working area (see also: git help tutorial)
    clone             Clone a repository into a new directory
    init              Create an empty Git repository or reinitialize an existing one
 
 work on the current change (see also: git help everyday)
    add               Add file contents to the index
    mv                Move or rename a file, a directory, or a symlink
    restore           Restore working tree files
    rm                Remove files from the working tree and from the index
    sparse-checkout   Initialize and modify the sparse-checkout
 
 examine the history and state (see also: git help revisions)
    bisect            Use binary search to find the commit that introduced a bug
    diff              Show changes between commits, commit and working tree, etc
    grep              Print lines matching a pattern
    log               Show commit logs
    show              Show various types of objects
    status            Show the working tree status
 
 grow, mark and tweak your common history
    branch            List, create, or delete branches
    commit            Record changes to the repository
    merge             Join two or more development histories together
    rebase            Reapply commits on top of another base tip
    reset             Reset current HEAD to the specified state
    switch            Switch branches
    tag               Create, list, delete or verify a tag object signed with GPG
 
 collaborate (see also: git help workflows)
    fetch             Download objects and refs from another repository
    pull              Fetch from and integrate with another repository or a local branch
    push              Update remote refs along with associated objects
 
 'git help -a' and 'git help -g' list available subcommands and some
 concept guides. See 'git help <command>' or 'git help <concept>'
 to read about a specific subcommand or concept.
 See 'git help git' for an overview of the system.
 ```



