* TOC
{:toc}



## 一、概述



### 1.GitHub 简介  

​       GitHub是一个面向[开源](https://baike.baidu.com/item/开源/20720669)及私有[软件](https://baike.baidu.com/item/软件/12053)项目的托管平台，因为只支持Git作为唯一的版本库格式进行托管，故名GitHub。

​        GitHub于2008年4月10日正式上线，除了[Git](https://baike.baidu.com/item/Git/12647237)代码仓库托管及基本的Web管理界面以外，还提供了订阅、讨论组、文本渲染、在线文件编辑器、协作图谱（报表）、代码片段分享（Gist）等功能。目前，其注册用户已经超过350万，托管版本数量也是非常之多，其中不乏知名开源项目[Ruby](https://baike.baidu.com/item/Ruby/11419) on Rails、[jQuery](https://baike.baidu.com/item/jQuery/5385065)、[python](https://baike.baidu.com/item/python/407313)等。

​        2018年6月4日，[微软](https://baike.baidu.com/item/微软/124767)宣布，通过75亿美元的股票交易收购代码托管平台GitHub。



**团队内协作**

![](/images/devops/git/github/github-13.png)

**跨团队协作**

![](/images/devops/git/github/github-14.png)



### 2.GitHub 官网地址  

```shell
# 官网地址

https://github.com/
```





## 二、基础



### 1.GitHub远程仓库

**1.github.com 注册账户**



**2.github 上创建仓库**



**3.本地服务器生成 ssh 公钥**

```shell
[root@qfedu.com ~]# ssh-keygen -t rsa -C 'meteor@163.com'  # 邮箱要与github上注册的相同
[root@qfedu.com ~]# cat .ssh/id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDVThfq4brrlsPGtAknVB0TLPx+7Dd3qlxTbSIrUOsGC5Y8JuNqVTlIntZB4oNj8cSQrWvec9CKm0a8o7WwaJIiqpxurz+YpQHP2KbapftKIxsX4hPf/z+p0El1U6arQa35/xmNsq+cJLH/bDdRG+EMDhuCBmjVZOlLj/hEdeIT6s56AnnCkaWoF+sq58KCF7Tk54jRbs/YiyE4SN7FuA70r+07sA/uj0+lmuk4E190KtQUELhjX/E9stivlqiRhxnKvVUqXDywsjfM8Rtvbi4Fg9R8Wt9fpd4QwnWksYUoR5qZJFYXO4hSZrUnSMruPK14xXjDJcFDcP2eHIzKgLD1 meteor@163.com
```



**4.github 添加 ssh 公钥** 

复制以上的公钥，在 github 中添加ssh key



**5.测试连接**

```shell
[root@qfedu.com ~]# yum install git
........
[root@qfedu.com ~]# ssh -T git@qfedu.comhub.com
The authenticity of host 'github.com (13.250.177.223)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
RSA key fingerprint is MD5:16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'github.com,13.250.177.223' (RSA) to the list of known hosts.
Hi meteor! You've successfully authenticated, but GitHub does not provide shell access.
[root@qfedu.com ~]#
```



**6.连接远程仓库（创建一个测试存储库）**

![](/images/devops/git/github/github-1.png)

```shell
# 在 github 网站新建一个仓库，命名为linux
~~~
[root@qfedu.com ~]# cd /opt
[root@qfedu.com ~]# mkdir linux
[root@qfedu.com ~]# mkdir linux
[root@qfedu.com ~]# cd linux
~~~
# git 初始化，然后做第一个基本的git操作(需要在github上创建存储库)
[root@qfedu.com ~]# git init
[root@qfedu.com ~]# touch README
[root@qfedu.com ~]# git add README
[root@qfedu.com ~]# git commit -m 'first commit'
[root@qfedu.com ~]# git remote add origin git@qfedu.comhub.com:userhub/linux.git
~~~
# 若出现origin已经存在的错误，删除origin
[root@qfedu.com linux]# git remote rm origin
# 现在继续执行push到远端
~~~
[root@qfedu.com linux]# git remote add origin git@qfedu.comhub.com:userhub/linux.git
[root@qfedu.com linux]# git push -u origin master
Counting objects: 3, done.
Writing objects: 100% (3/3), 205 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@qfedu.comhub.com:fakehydra/linux-.git
 * [new branch]      master -> master
分支 master 设置为跟踪来自 origin 的远程分支 master。
# 注意
# 设置存储库链接
[root@qfedu.com ~]# git remote set-url origin git@qfedu.comhub.com:userhub/linux.git
# 如果 push 失败，合并分支到 master 再 push
[root@qfedu.com ~]# git pull --rebase origin master
```



### 2.基本概念

**1.仓库（Repository）**

仓库用来存放项目代码，每个项目对应一个仓库，多个开源项目则有多个仓库

 

**2.收藏（Star）**

收藏项目，方便下次查看

 

**3.复制克隆项目（Fork）**

![](/images/devops/git/github/github-2.png)

**注意：该fork的项目时独立存在的**

 

**3.发起请求（Pull Request）**

![](/images/devops/git/github/github-3.png)



**4.关注（Watch）**

关注项目，当项目更新可以接收到通知



**5.事务卡片（Issue）**

发现代码BUG，但是目前没有成型代码，需要讨论时用；

 

**6.Github主页**

账号创建成功或点击网址导航栏github图标都可进入github主页：该页左侧主要显示用户动态以及关注用户或关注仓库的动态；右侧显示所有的git库

 

**7.仓库主页**

仓库主页主要显示项目的信息，如：项目代码，版本，收藏/关注/fork情况等

 

**8.个人主页**

个人信息：头像，个人简介，关注我的人，我关注的人，我关注的git库，我的开源项目，我贡献的开源项目等信息



### 3.Github Issues

作用：发现代码BUG，但是目前没有成型代码，需要讨论时用；或者使用开源项目出现问题时使用

 

情景：张三发现李四开源git库，则发提交了一个issue；李四隔天登录在github主页看到通知并和张三交流，最后关闭issue

![](/images/devops/git/github/github-4.png)

![](/images/devops/git/github/github-5.png)

![](/images/devops/git/github/github-6.png)

 

![](/images/devops/git/github/github-7.png)

![](/images/devops/git/github/github-8.png)

 

### 4.开源项目贡献流程

**1.新建Issue**

提交使用问题或者建议或者想法

 

**2.Pull Request**

**步骤：**

1. fork项目
2. 修改自己仓库的项目代码
3. 新建 pull request
4. 等待作者操作审核



### 5.Github Pages 搭建网站

#### 5.1.个人站点

**1.访问：**

https://用户名.github.io 

 

**2.搭建步骤**

1） 创建个人站点  -> 新建仓库（注：仓库名必须是【用户名.github.io】）

2） 在仓库下新建index.html的文件即可

![](/images/devops/git/github/github-9.png)

![](/images/devops/git/github/github-10.png)

![](/images/devops/git/github/github-11.png)

![](/images/devops/git/github/github-12.png)



**注意：**

- github pages 仅支持静态网页
- 仓库里面是.html文件
- 个人主页也可以设置主题

 

#### 5.2.Project Pages 项目站点

**1.访问**

https://用户名.github.io/仓库名 



**2.原理**

gh-pages 用于构建和发布



**3.搭建步骤**

1. 进入项目主页，点击settings
2. 在settings页面，点击【Launch automatic page generator 】来自动生成主题页面
3. 新建站点基础信息设置
4. 选择主题
5. 生成网页

 



## 三、实践



### 1.设置用户签名

**用户签名最好与GitHub账号邮箱一致**

```shell
git config --global user.name xxxxxx
git config --global user.email xxxxxx@126.com

cat ~/.gitconfig
```

```shell
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-test/github/git-demo (main)
$ git config --global user.name xxxxxx

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-test/github/git-demo (main)
$ git config --global user.email xxxxxx@126.com

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-test/github/git-demo (main)
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
[credential "http://182.92.210.65:18080"]
        provider = generic
[core]
        autocrlf = true
```



### 2.SSH 免密登录

#### 2.1.Windows免密登录

**1.删除现有Key**

访问目录：C:\Users\Administrator\ .ssh，删除公钥：id_rsa.pub ，私钥：id_rsa

![](/images/devops/git/github/github-15.png)



**2.生成.ssh 秘钥**  

```shell
# 在目录：C:\Users\Administrator
# 运行命令生成.ssh 秘钥目录[注意：这里-C 这个参数是大写的 C]  
ssh-keygen -t rsa -C xxxxxx@126.com
```

```shell
Administrator@DESKTOP-AV12NNP MINGW64 ~
$ pwd
/c/Users/Administrator

# 三次回车
Administrator@DESKTOP-AV12NNP MINGW64 ~
$ ssh-keygen -t rsa -C xxxxxx@126.com
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/Administrator/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/Administrator/.ssh/id_rsa
Your public key has been saved in /c/Users/Administrator/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:g/Fz6kyzO89UHnmFEXQh7M50Cvk52mkT/hipb1fFp3Y xxxxxx@126.com
The key's randomart image is:
+---[RSA 3072]----+
|            .o+oo|
|             ..+ |
|      .     o ...|
|       +   o + o+|
|      . S . X =.o|
|         = o @+ E|
|        + . =++..|
|       +.= ..Bo. |
|        =+o.+o+. |
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
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC7SPDzi8MzAmFiNLBHI/4FU3kZUUG4YpKisf4GUwBfCxa0K2En/vr/cmLQ8jDWfV99M/qHq5LGlMenbxgqW4EVdv2PxF6LK4eoLRG4ShgZ51ykRlFI+1J4hN0wG0HUHuZGdKt6+N+nLF/tcQbHHoeabkFxyOo5A0rwRnseZNM01EtccHf1DmDUmZULpIPD8RdC6YBnlysOtrQmC+Ux9esIUvWNNSh5D0Lb+h0FpvyfdlRB7NPDk41/hq7xAkqXqKeP/yF+xuZ6WiR4/ReBhqJRkO/zv3peX1J/O/ML//9jOwOJoGckVOIZ5yPrnwMQ8Z01RHxoYPjxqFG5/z+iyIUUQD7hDCKKg44+Jf99SKmsmzLhzyKDOFqEhI1bh8/vy+O6XbPqlKSqjLdSj8z9mP22/47WOYjEW29+W8GGxZ1v9CFK/rHeUu1ekzKde2Bo85qirVpTzV2EQoAb7zXbDxCEiKYfCYghifnFTHoqiqVd1pOaGBZaEZfxSIKxb59YnSs= xxxxxx@126.com
```

```shell
# 公钥

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC7SPDzi8MzAmFiNLBHI/4FU3kZUUG4YpKisf4GUwBfCxa0K2En/vr/cmLQ8jDWfV99M/qHq5LGlMenbxgqW4EVdv2PxF6LK4eoLRG4ShgZ51ykRlFI+1J4hN0wG0HUHuZGdKt6+N+nLF/tcQbHHoeabkFxyOo5A0rwRnseZNM01EtccHf1DmDUmZULpIPD8RdC6YBnlysOtrQmC+Ux9esIUvWNNSh5D0Lb+h0FpvyfdlRB7NPDk41/hq7xAkqXqKeP/yF+xuZ6WiR4/ReBhqJRkO/zv3peX1J/O/ML//9jOwOJoGckVOIZ5yPrnwMQ8Z01RHxoYPjxqFG5/z+iyIUUQD7hDCKKg44+Jf99SKmsmzLhzyKDOFqEhI1bh8/vy+O6XbPqlKSqjLdSj8z9mP22/47WOYjEW29+W8GGxZ1v9CFK/rHeUu1ekzKde2Bo85qirVpTzV2EQoAb7zXbDxCEiKYfCYghifnFTHoqiqVd1pOaGBZaEZfxSIKxb59YnSs= xxxxxx@126.com
```



#### 2.2.GitHub配置SSH

**1.查看你生成的公钥：**

vim id_rsa.pub

就可以查看到你的公钥



**2.登录 GitHub，点击用户头像→Settings→SSH and GPG keys**

![](/images/devops/git/github/github-16.png)

![](/images/devops/git/github/github-17.png)

![](/images/devops/git/github/github-18.png)

![](/images/devops/git/github/github-19.png)

![](/images/devops/git/github/github-20.png)



#### 2.3.测试SSH

```shell
# 克隆
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-test/github
$ git clone git@github.com:xxxxxx/git-demo.git
Cloning into 'git-demo'...
The authenticity of host 'github.com (20.205.243.166)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
remote: Enumerating objects: 10, done.
remote: Counting objects: 100% (10/10), done.
remote: Compressing objects: 100% (6/6), done.
remote: Total 10 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (10/10), 5.93 KiB | 758.00 KiB/s, done.

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-test/github
$ ll
total 0
drwxr-xr-x 1 Administrator 197121 0 Feb  6 11:23 git-demo/


# 查看
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-test/github
$ cd git-demo/

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-test/github/git-demo (main)
$ ll
total 1
-rw-r--r-- 1 Administrator 197121 18 Feb  6 11:23 myfile1.txt

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-test/github/git-demo (main)
$ git remote -v
origin  git@github.com:xxxxxx/git-demo.git (fetch)
origin  git@github.com:xxxxxx/git-demo.git (push)


Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-test/github/git-demo (main)
$ touch hello.txt

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-test/github/git-demo (main)
$ git add .

Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-test/github/git-demo (main)
$ git commit -m "remote add file"
[main f461b2b] remote add file
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 hello.txt


# 推送数据
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-test/github/git-demo (main)
$ git push origin main
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 8 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 270 bytes | 270.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:xxxxxx/git-demo.git
   0928bf4..f461b2b  main -> main
```



### 3.创建远程仓库

![](/images/devops/git/github/github-23.png)

![](/images/devops/git/github/github-21.png)

![](/images/devops/git/github/github-22.png)



### 4.远程仓库操作

#### 4.1.推送本地分支到远程仓库

```shell
git push git@github.com:xxxxxx/git-demo.git master
```

```shell
# 本地仓库
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-test/github/git-demo (master)
$ ll
total 1
-rw-r--r-- 1 Administrator 197121  0 Feb  6 12:04 file2.txt
-rw-r--r-- 1 Administrator 197121  0 Feb  6 12:04 hello.txt
-rw-r--r-- 1 Administrator 197121 18 Feb  6 12:04 myfile1.txt


# 推送
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-test/github/git-demo (master)
$ git push git@github.com:xxxxxx/git-demo.git master
Enumerating objects: 15, done.
Counting objects: 100% (15/15), done.
Delta compression using up to 8 threads
Compressing objects: 100% (10/10), done.
Writing objects: 100% (15/15), 6.39 KiB | 3.20 MiB/s, done.
Total 15 (delta 0), reused 15 (delta 0), pack-reused 0
To github.com:xxxxxx/git-demo.git
 * [new branch]      master -> master
```

![](/images/devops/git/github/github-24.png)



#### 4.2.克隆远程仓库到本地

```shell
git clone git@github.com:xxxxxx/git-demo.git
```

```shell
Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-test/github
$ git clone git@github.com:xxxxxx/git-demo.git
Cloning into 'git-demo'...
remote: Enumerating objects: 15, done.
remote: Counting objects: 100% (15/15), done.
remote: Compressing objects: 100% (10/10), done.
Receiving objects: 100% (15/15), 6.39 KiB | 2.13 MiB/s, done.
remote: Total 15 (delta 0), reused 15 (delta 0), pack-reused 0


Administrator@DESKTOP-AV12NNP MINGW64 /d/dev-tct/Devops/dev/code/git-test/github
$ ll
total 0
drwxr-xr-x 1 Administrator 197121 0 Feb  6 12:44 git-demo/
```



#### 4.3.邀请加入团队

**1.选择邀请合作者**

![](/images/devops/git/github/github-25.png)

![](/images/devops/git/github/github-26.png)

![](/images/devops/git/github/github-27.png)



**2.复 制 地 址 并 通 过 微 信 钉 钉 等 方 式 发 送 给 该 用 户 ， 复 制 内 容 如 下 ：**  

https://github.com/xxxxxx/git-demo/invitations

![](/images/devops/git/github/github-28.png)

**3.在 xxxxxx 这个账号中的地址栏复制收到邀请的链接，点击接受邀请。**  

![](/images/devops/git/github/github-29.png)

**4.成功之后可以在xxxxxx 这个账号上看到 git-demo的远程仓库。**  

**注意项目需要收藏。**



#### 4.4. 删除项目

![](/images/devops/git/github/github-51.png)

![](/images/devops/git/github/github-52.png)

![](/images/devops/git/github/github-53.png)

![](/images/devops/git/github/github-54.png)



### 5.跨团队操作

**1.将远程仓库的地址复制发给邀请跨团队协作的人，比如东方不败**

![](/images/devops/git/github/github-30.png)

 

**2.在东方不败的GitHub 账号里的地址栏复制收到的链接，然后点击 Fork 将项目叉到自 己的本地仓库**

![](/images/devops/git/github/github-31.png)

**叉入中…**

 ![](/images/devops/git/github/github-32.png)

**叉成功后可以看到当前仓库信息**

![](/images/devops/git/github/github-33.png)

 

**3.东方不败就可以在线编辑叉取过来的文件**

![](/images/devops/git/github/github-34.png)

 

**4.编辑完毕后，填写描述信息并点击左下角绿色按钮提交**

![](/images/devops/git/github/github-35.png)

 

**5.接下来点击上方的 Pull 请求，并创建一个新的请求**

![](/images/devops/git/github/github-36.png)

![](/images/devops/git/github/github-37.png)



**6.回到岳岳GitHub 账号可以看到有一个 Pull request 请求**

![](/images/devops/git/github/github-38.png)

![](/images/devops/git/github/github-39.png)

![](/images/devops/git/github/github-40.png)



**7.如果代码没有问题，可以点击 Merge pull reque 合并代码**

![](/images/devops/git/github/github-41.png)



### 6.GitHub分支管理

#### 6.1.创建分支

![](/images/devops/git/github/github-42.png)

![](/images/devops/git/github/github-43.png)



#### 6.2.删除分支

![](/images/devops/git/github/github-44.png)

![](/images/devops/git/github/github-45.png)



### 7.标签管理

#### 7.1.创建标签

![](/images/devops/git/github/github-46.png)

![](/images/devops/git/github/github-47.png)

![](/images/devops/git/github/github-48.png)

![](/images/devops/git/github/github-49.png)

![](/images/devops/git/github/github-50.png)



