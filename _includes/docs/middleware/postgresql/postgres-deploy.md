* TOC
{:toc}



## 一、PostgreSQL部署方式



### 1.Yum方式部署

```shell
# 下载安装包，本文以二进制包方式安装

# 下载地址：
https://www.postgresql.org/download/

# CentOS安装参考
https://www.postgresql.org/download/linux/redhat/
```



![](/images/middleware/postgresql/postgres-deploy/deploy-1.png)

![](/images/middleware/postgresql/postgres-deploy/deploy-2.png)



```shell
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
 
sudo yum install -y postgresql11-server
 
sudo /usr/pgsql-11/bin/postgresql-11-setup initdb
 
sudo systemctl enable postgresql-11
 
sudo systemctl start postgresql-11
```



### 2.RPM方式部署

```shell
# 下载地址: 
https://download.postgresql.org/pub/repos/yum/11/redhat/rhel-7-x86_64/
 
 
#或者直接执行wget下载
wget https://download.postgresql.org/pub/repos/yum/11/redhat/rhel-7.2-x86_64/postgresql11-11.4-1PGDG.rhel7.x86_64.rpm
 
wget https://download.postgresql.org/pub/repos/yum/11/redhat/rhel-7.2-x86_64/postgresql11-libs-11.4-1PGDG.rhel7.x86_64.rpm
 
wget https://download.postgresql.org/pub/repos/yum/11/redhat/rhel-7.2-x86_64/postgresql11-server-11.4-1PGDG.rhel7.x86_64.rpm
```



### 3.源码方式部署

```shell
# 获取官方源码包
准备装11.7版本，选择其他版本进入下面的第一个地址后退选择目的版本即可
 
# 安装官方教学
https://www.postgresql.org/docs/11/install-getsource.html
 
# 源码包
https://www.postgresql.org/ftp/source/
 
# 11.7:连接可能随着官方的更新而失效
https://ftp.postgresql.org/pub/source/v11.7/postgresql-11.7.tar.gz
 
https://www.postgresql.org/ftp/source/ 
https://www.postgresql.org/docs/11/install-getsource.html 
https://www.postgresql.org/download/ 
```



### 4.二进制方式部署

```shell
# 下载二进制包
pgsql有很多类型的包，对于不同linux发行版都有对应的编译好的包，安装很方便，另外如果对于通用的linux平台可以编译源码安装或者安装官方编译好的二进制包，源码包的安装仅仅比二进制安装多出一个编译步骤，其余的都一样，所以这里使用安装方式是安装编译好的二进制包


# pgsql官网地址
https://www.postgresql.org/
进入后点击download就来到下载页，这里点击Linux下面的Other Linux选项，然后点击下方的tar.gz archive下载二进制归档
```



![](/images/middleware/postgresql/postgres-deploy/deploy-3.png)



```shell
然后就来到最终的pgsql下载页了，地址为：
https://www.enterprisedb.com/download-postgresql-binaries
 

https://www.postgresql.org/
https://www.postgresql.org/download/
https://www.enterprisedb.com/download-postgresql-binaries 
 
注意：官网没有11以后的linux版本
```



### 5.Docker方式部署

```shell
https://hub.docker.com/_/postgres


$ docker run -d \
	--name some-postgres \
	-e POSTGRES_PASSWORD=mysecretpassword \
	-e PGDATA=/var/lib/postgresql/data/pgdata \
	-v /custom/mount:/var/lib/postgresql/data \
	postgres
```





## 二、PostgreSQL部署



### 1.Yum方式部署

#### 1.1.部署数据库

```shell
1.安装yum源
# yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
 
2.安装postgresql
# yum install -y postgresql11-server
 
3.数据库初始化
# /usr/pgsql-11/bin/postgresql-11-setup initdb
 
4.修改配置文件
# cd /var/lib/pgsql/11/data/
```

![](/images/middleware/postgresql/postgres-deploy/deploy-4.png)



```shell
# 修改配置文件
vim /var/lib/pgsql/11/data/postgresql.conf
 
# 修改为如下：
listen_addresses = '*'
port = 5432 
```

![](/images/middleware/postgresql/postgres-deploy/deploy-5.png)



```shell
# 修改配置文件
vim /var/lib/pgsql/11/data/pg_hba.conf
 
# 修改为如下：
host    all             all             0.0.0.0/0               md5
 
# 或
host    all             all             all                          md5
```

![](/images/middleware/postgresql/postgres-deploy/deploy-6.png)

```shell
# 说明：
TYPE：pg的连接方式，local：本地unix套接字，host：tcp/ip连接
DATABASE：指定数据库
USER：指定数据库用户
ADDRESS：ip地址，可以定义某台主机或某个网段，32代表检查整个ip地址，相当于固定的ip，24代表只检查前三位，最后一位是0~255之间的任何一个
METHOD：认证方式，常用的有ident，md5，password，trust，reject。
    md5是常用的密码认证方式。
    password是以明文密码传送给数据库，建议不要在生产环境中使用。
               trust是只要知道数据库用户名就能登录，建议不要在生产环境中使用。
               reject是拒绝认证。
 
 
5.postgresql 安装目录授权 
# chown postgres:root -R /usr/pgsql-11/ 
 
6.启动服务 
# systemctl start postgresql-11 
 
# 服务自启动
# systemctl enable postgresql-11

#查看端口
# netstat -lntp
# netstat -nat
```

![](/images/middleware/postgresql/postgres-deploy/deploy-7.png)



#### 1.2.连接数据库

```shell
1.切换用户，设置数据库密码 
 
# su - postgres
$ psql -U postgres
# ALTER USER postgres with encrypted password 'postgres';
```

![](/images/middleware/postgresql/postgres-deploy/deploy-8.png)

```shell
退出: \q
列出所有库 \l
列出所有用户 \du
列出库下所有表\d
```

![](/images/middleware/postgresql/postgres-deploy/deploy-9.png)



### 2.RPM方式部署

#### 2.1.部署数据库

```shell
1.下载RPM包
下载地址: https://download.postgresql.org/pub/repos/yum/11/redhat/rhel-7-x86_64/
 
#或者直接执行wget下载 
wget https://download.postgresql.org/pub/repos/yum/11/redhat/rhel-7.2-x86_64/postgresql11-11.4-1PGDG.rhel7.x86_64.rpm
 
wget https://download.postgresql.org/pub/repos/yum/11/redhat/rhel-7.2-x86_64/postgresql11-libs-11.4-1PGDG.rhel7.x86_64.rpm
 
wget https://download.postgresql.org/pub/repos/yum/11/redhat/rhel-7.2-x86_64/postgresql11-server-11.4-1PGDG.rhel7.x86_64.rpm
 
 
2.安装
先安装相应依赖
# yum install -y libicu systemd-sysv 
 
安装rpm包，需要按照这个顺序安装，卸载就反序卸载
# rpm -ivh postgresql11-libs-11.4-1PGDG.rhel7.x86_64.rpm
# rpm -ivh postgresql11-11.4-1PGDG.rhel7.x86_64.rpm
# rpm -ivh postgresql11-server-11.4-1PGDG.rhel7.x86_64.rpm
 
卸载
rpm -e postgresql11-server-11.4-1PGDG.rhel7.x86_64
rpm -e postgresql11-11.4-1PGDG.rhel7.x86_64
rpm -e postgresql11-libs-11.4-1PGDG.rhel7.x86_64
rm -rf /usr/pgsql-11/
rm -rf /var/lib/pgsql/ 
 
3.数据库初始化
# /usr/pgsql-11/bin/postgresql-11-setup initdb 
 
4.修改配置文件
# cd /var/lib/pgsql/11/data/
```

![](E:\dev-middleware\postgresql\md\deploy\deploy-4.png)



```shell
# 修改配置文件
vim /var/lib/pgsql/11/data/postgresql.conf
 
# 修改为如下：
listen_addresses = '*'
port = 5432 
```

![](/images/middleware/postgresql/postgres-deploy/deploy-5.png)



```shell
# 修改配置文件
vim /var/lib/pgsql/11/data/pg_hba.conf
 
# 修改为如下：
host    all             all             0.0.0.0/0               md5
 
# 或
host    all             all             all                          md5
```

![](/images/middleware/postgresql/postgres-deploy/deploy-6.png)



```shell
# 说明：
TYPE：pg的连接方式，local：本地unix套接字，host：tcp/ip连接
DATABASE：指定数据库
USER：指定数据库用户
ADDRESS：ip地址，可以定义某台主机或某个网段，32代表检查整个ip地址，相当于固定的ip，24代表只检查前三位，最后一位是0~255之间的任何一个
METHOD：认证方式，常用的有ident，md5，password，trust，reject。
    md5是常用的密码认证方式。
    password是以明文密码传送给数据库，建议不要在生产环境中使用。
               trust是只要知道数据库用户名就能登录，建议不要在生产环境中使用。
               reject是拒绝认证。
 
 
5.postgresql 安装目录授权  
# chown postgres:root -R /usr/pgsql-11/ 
 
6.启动服务 
# systemctl start postgresql-11 
 
服务自启动
# systemctl enable postgresql-11

查看端口
# netstat -lntp
# netstat -nat
```

![](/images/middleware/postgresql/postgres-deploy/deploy-7.png)



#### 2.2.连接数据库

```shell
1.切换用户，设置数据库密码 
 
# su - postgres
$ psql -U postgres
# ALTER USER postgres with encrypted password 'postgres';
```

![](/images/middleware/postgresql/postgres-deploy/deploy-8.png)

```shell
退出: \q
列出所有库 \l
列出所有用户 \du
列出库下所有表\d
```

![](/images/middleware/postgresql/postgres-deploy/deploy-9.png)



### 3.源码方式部署

#### 3.1.准备工作

```shell
1.1.安装gcc编辑器
# gcc --version 
 
1.2.安装make
# make --version
 
1.3.安装readline
# yum install readline
# yum -y install -y readline-devel
 
1.4.安装zlib-devel
# yum install zlib
# yum install zlib-devel
 
1.5.下载postgresql源码
postgresql的官方网站下载：
https://www.postgresql.org/
```



#### 3.2.编译安装

```shell
2.1.解压下载的压缩包
# cd /root
# tar -xzvf postgresql-11.0.tar.gz
 
创建postgresql的安装目录
# mkdir -p /home/app/postgresql/data 
 
2.2.生成makefile
# cd postgresql-11.0
# ./configure --prefix=/home/app/postgresql
 
2.3.编译安装
# make && make install
  
2.4.安装工具集
# cd /home/software/postgresql-11.0/contrib
# make && make install
```



#### 3.3.配置数据库

```shell
3.1.创建postgres用户：
# groupadd postgres
# useradd -g postgres postgres
 
3.2.修改data目录的用户为postgres
chown -R postgres:postgres /home/app/postgresql/data
 
3.3.修改环境变量
# su postgres
$ vim /home/postgres/.bash_profile
 
添加环境变量：
    export PGHOME=/home/app/postgresql
    export PGDATA=/home/app/postgresql/data
    export PATH=$PGHOME/bin:$PATH
    export MANPATH=$PGHOME/share/man:$MANPATH
    export LANG=en_US.utf8
    export DATE=`date +"%Y-%m-%d %H:%M:%S"`
    export LD_LIBRARY_PATH=$PGHOME/lib:$LD_LIBRARY_PATH
    alias rm='rm  -i'
    alias ll='ls -lh'
然后使环境变量立即生效，否则initdb命令会找不到：
 
$ source /home/postgres/.bash_profile
 
3.4.初始化数据库
$ initdb -D /home/app/postgresql/data/
Success. You can now start the database server using:
 
    pg_ctl -D /home/app/postgresql/data/ -l logfile start
 
3.5.修改配置文件
# cd /home/app/postgresql/data/
```

![](/images/middleware/postgresql/postgres-deploy/deploy-4.png)



```shell
修改配置文件
vim /home/app/postgresql/data/postgresql.conf
 
# 修改为如下：
listen_addresses = '*'
port = 5432 
```

![](/images/middleware/postgresql/postgres-deploy/deploy-5.png)



```shell
# 修改配置文件
vim /home/app/postgresql/data/pg_hba.conf
 
# 修改为如下：
host    all             all             0.0.0.0/0               md5
 
# 或
host    all             all             all                          md5
```

![](/images/middleware/postgresql/postgres-deploy/deploy-6.png)

```shell
3.6.启动数据库
$ su postgres
$ cd /home/postgres
$ pg_ctl -D /home/app/postgresql/data/ -l logfile start
 
    cd /home/postgres  //logfile需要在postgres的用户目录下创建
    pg_ctl -D /home/app/postgresql/data/ -l logfile start
 
 
查看端口
# netstat -lntp
# netstat -nat
```

![](/images/middleware/postgresql/postgres-deploy/deploy-7.png)



```shell
3.7.添加postgresql至服务(未验证)
    cd /home/software/postgresql-11.4/contrib/start-scripts
    chmod a+x linux.
    cp linux /etc/init.d/postgresql
此时就可以使用 /etc/init.d/postgresql stop 来停止postgresql
也可以使用：service postgresql start 来启动postgresql
修改postgresql脚本中prefix和PGDATA的内容
chkconfig --add postgresql //设置服务开机启动
```



#### 3.4.连接数据库

参考    **1.2.连接数据库**



### 4.二进制方式部署

#### 4.1.准备工作

```shell
1.下载软件
# wget https://get.enterprisedb.com/postgresql/postgresql-10.12-1-linux-x64-binaries.tar.gz
 
2.创建postgres用户
# groupadd postgres
# useradd -g postgres postgres
 
3.创建postgresql的安装目录
# mkdir -p /home/app
 
进入下载目录解压
# tar -xzvf postgresql-10.12-1-linux-x64-binaries.tar.gz -C /home/app
 
创建data目录：
# mkdir -p /home/app/pgsql/data
# mkdir -p /home/app/pgsql/log
 
授权：
# cd /home/app/pgsql
# chown -R postgres.postgres pgsql
```



#### 4.2.配置数据库

```shell
1.初始化数据库
切换用户 postgres，并执行初始化操作
# su postgres
 
# cd /home/app/pgsql/bin
 
$ ./initdb -E utf8 -D /home/app/pgsql/data
Success. You can now start the database server using:
 
    ./pg_ctl -D /home/app/pgsql/data -l logfile start
 
 
2.配置环境变量
~/.bash_profile 添加如下内容
注：这个文件目录是在当前用户（postgres）的根目录下，即/home/{User}/
 
[postgres@iZ2ze72jggg737pb9vp1g6Z data]$ vim ~/.bash_profile
 
PATH=/home/app/pgsql/bin:$PATH
export PATH
 
配置文件生效
$ source ~/.bash_profile
 
修改后的配置文件
# .bash_profile
 
# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi
 
# User specific environment and startup programs
 
PATH=$PATH:$HOME/.local/bin:$HOME/bin
 
PATH=/home/app/pgsql/bin:$PATH
 
export PATH
 
 
3.修改配置文件
# cd /home/app/postgresql/data/
```

![](/images/middleware/postgresql/postgres-deploy/deploy-4.png)



```shell
# 修改配置文件
vim /home/app/postgresql/data/postgresql.conf
 
# 修改为如下：
listen_addresses = '*'
port = 5432 
```

![](/images/middleware/postgresql/postgres-deploy/deploy-5.png)



```shell
修改配置文件
vim /home/app/postgresql/data/pg_hba.conf
 
# 修改为如下：
host    all             all             0.0.0.0/0               md5
 
或
host    all             all             all                          md5
```

![](/images/middleware/postgresql/postgres-deploy/deploy-6.png)

```shell
4.启动数据库
$ cd /home/app/pgsql/bin
$ ./pg_ctl -D /home/app/pgsql/data -l logfile start
 
查看端口
# netstat -lntp
# netstat -nat
```



#### 4.3.连接数据库

参考    **1.2.连接数据库**



### 5.Docker方式部署

#### 5.1.部署数据库

```shell
1.创建目录
# mkdir -p /data/psql
 
2.运行容器
docker run -d --network host --name pg12 --restart=always \
-e LANG="C.UTF-8" \
-e 'TZ=Asia/Shanghai' \
-e "POSTGRES_DB=postgres" \
-e "POSTGRES_USER=postgres" \
-e "POSTGRES_PASSWORD=postgres" \
-v /data/psql:/var/lib/postgresql/data \
postgres:12
 
3.进入容器
# docker exec -it pg12 /bin/sh
 
切换用户
# su - postgres
$ psql 
# \l
```



