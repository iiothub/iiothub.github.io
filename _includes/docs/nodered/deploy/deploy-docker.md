* TOC
{:toc}



## 一、概述



### 1.官网文档

```shell
https://nodered.org/
https://nodered.org/docs/getting-started/

# Running under Docker
https://nodered.org/docs/getting-started/docker
```



![](/images/nodered/deploy/deploy-docker/install-1.png)



### 2.Docker部署Node-RED

```shell
docker run -it -p 1880:1880 -v node_red_data:/data --name mynodered nodered/node-red


    docker run              - run this container, initially building locally if necessary
    -it                     - attach a terminal session so we can see what is going on
    -p 1880:1880            - connect local port 1880 to the exposed internal port 1880
    -v node_red_data:/data  - mount a docker named volume called `node_red_data` to the container /data directory so any changes made to flows are persisted
    --name mynodered        - give this machine a friendly local name
    nodered/node-red        - the image to base it on - currently Node-RED v1.2.0
```

![](/images/nodered/deploy/deploy-docker/install-2.png)



```shell
docker run -it -p 1880:1880 -v node_red_data:/data --name mynodered nodered/node-red:latest
```



```shell
# nodered/node-red


docker run -it -p 1880:1880 -v myNodeREDdata:/data --name mynodered nodered/node-red


docker run -it -p 1880:1880 -v myNodeREDdata:/data --name mynodered nodered/node-red:latest-minimal


docker run -it -p 1880:1880 -v myNodeREDdata:/data --name mynodered nodered/node-red:1.0.1-10-minimal-arm32v6



docker run --network host --restart=always -v node_red_data:/data --name nodered nodered/node-red:latest


docker run -p 1880:1880 --restart=always -v node_red_data:/data --name nodered nodered/node-red:latest


docker run -it -p 1880:1880 -v node_red_data:/data --name nodered nodered/node-red:latest


docker run -it -p 1880:1880 -v node_red_data:/data --name nodered nodered/node-red:3.1.0
```





## 二、部署Node-RED



### 1.安装Docker

- **安装Docker（所有节点）**

安装版本19.03.*

```shell
$ wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo

$ yum -y install docker-ce

$ systemctl enable docker && systemctl start docker

$ docker --version
```

- **添加阿里云加速镜像**

```shell
# 添加阿里云加速镜像

cat > /etc/docker/daemon.json << EOF
{
   "registry-mirrors": ["https://xxxxxx.aliyuncs.com"],
   "exec-opts": ["native.cgroupdriver=systemd"]
} 
EOF
```

- **重启docker**

```shell
#重启docker

systemctl restart docker
```



### 2.部署Node-RED

```shell
# 安装部署

# docker run --network host --restart=always -v node_red_data:/data --name nodered nodered/node-red:latest


# docker ps -a
CONTAINER ID   IMAGE                     COMMAND                  CREATED          STATUS                            PORTS     NAMES
8f7eb53acc80   nodered/node-red:latest   "npm --no-update-not…"   38 seconds ago   Up 2 seconds (health: starting)             nodered
```



```shell
# 访问地址
http://192.168.202.168:1880
```



![](/images/nodered/deploy/deploy-docker/install-4.png)



