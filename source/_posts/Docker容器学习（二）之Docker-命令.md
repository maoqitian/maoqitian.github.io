---
title: Docker容器学习（二）之Docker 命令
date: 2019-09-22 23:23:11
categories:
- Docker #分类
tags:
- docker
- Maven
- Git
- CentOs
---
![dockerlogo](https://raw.githubusercontent.com/maoqitian/MaoMdPhoto/master/Docker/dockerlogo.jpg)
# Docker 命令
> 上一篇文章我们了解如何在CentOs安装Docker,接下来我们学习Docker 命令
## Docker镜像常用命令
命令 | 解释
---|---
docker images 或者 docker image ls | 列表本地所有镜像
docker search 关键词 | 在Docker Hub中搜索镜像
docker pull 镜像名称 | 下载Docker镜像
docker rmi 镜像id | 删除Docker镜像。加参数-f表示强制删除。
docker run 镜像名称称 | 下载Docker镜像
docker build -t 标签名称 目录 | 构建Docker镜像，-t 表示指定一个标签
docker tag | 为镜像打标签
<!--more-->
## Docker 容器常用命令
命令 | 解释
---|---
docker ps | 列表所有运行中的Docker容器（包括已停止的容器）。该命令参数比较多，-a：列表所有容器；-f：过滤；-q 只列表容器的id。
docker version | 查看docker 版本信息
docker --version| 查看docker 版本
docker info | 查看Docker系统信息，例如：CPU、内存、容器个数等等
docker kill 容器id | 杀死id对应容器
docker start / stop / restart 容器id | 启动、停止、重启指定容器

- 更多命令，请输入--help参数查询；如果想看docker命令可输入docker --help；如果想查询docker run命令的用法，可输入docker run --help。
### 搜索、下载、删除镜像

```
# 搜索镜像 nginx
NAME                                                   DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
nginx                                                  Official build of Nginx.                        11281               [OK]                
jwilder/nginx-proxy                                    Automated Nginx reverse proxy for docker c...   1586                                    [OK]
richarvey/nginx-php-fpm                                Container running Nginx + PHP-FPM capable ...   708                                     [OK]
jrcs/letsencrypt-nginx-proxy-companion                 LetsEncrypt container to use with nginx as...   504                                     [OK]
webdevops/php-nginx                                    Nginx with PHP-FPM                              125                                     [OK]
zabbix/zabbix-web-nginx-mysql                          Zabbix frontend based on Nginx web-server ...   96                                      [OK]
bitnami/nginx                                          Bitnami nginx Docker Image                      65                                      [OK]
linuxserver/nginx                                      An Nginx container, brought to you by Linu...   59                                      
1and1internet/ubuntu-16-nginx-php-phpmyadmin-mysql-5   ubuntu-16-nginx-php-phpmyadmin-mysql-5          50                                      [OK]
```
- NAME：镜像仓库名称。
- DESCRIPTION：镜像仓库描述。
- STARS：镜像仓库收藏数，表示该镜像仓库的受欢迎程度，类似于GitHub的Stars。
- OFFICAL：表示是否为官方仓库，该列标记为[OK]的镜像均由各软件的官方项目组创建和维护。由结果可知，java这个镜像仓库是官方仓库，而其他的仓库都不是镜像仓库。
- AUTOMATED：表示是否是自动构建的镜像仓库。

```
#下载镜像
docker pull nginx
    
```

### 列出镜像
- docker images 或者 docker image ls 
```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              27a188018e18        7 days ago          109MB
hello-world         latest              fce289e99eb9        3 months ago        1.84kB

```
- REPOSITORY：镜像所属仓库名称。

- TAG：镜像标签。默认是latest，表示最新。

- IMAGE ID：镜像ID，表示镜像唯一标识。

- CREATED：镜像创建时间。

- SIZE：镜像大小。
### 删除镜像

```
# 删除指定名称镜像
docker rmi hello-world
#删除所有镜像
docker rmi -f $(docker images)
```
## docker run 命令启动容器
- 该命令即可新建并启动一个容器

选项 | 含义
---|---
-d | 表示后台运行2
-P | 随机端口映射（指定端口映射）

```
# 指定端口映射，有以下四种格式。
     ip:hostPort:containerPort
     ip::containerPort
     hostPort:containerPort
     containerPort

```
### 例子 启动一个nginx容器

```
 # 前面我们已经pull了一个nginx的镜像 
 # -d                           # 后台运行
 # -p 宿主机端口:容器端口         # 开放容器端口到宿主机端口 （服务器防火墙必须开放91端口）

 docker run -d -p 91:80 nginx

 浏览器访问 http://服务器地址:91/ 能够成功访问说明我们
 
 # 需要注意的是，使用docker run命令创建容器时，会先检查本地是否存在指定镜像。如果本地不存在该名称的镜像，Docker就会自动从Docker Hub下载镜像并启动一个Docker容器。
```
## Docker基本命令实践
### 列出容器
- docker ps -a
```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
ded3613de77d        nginx               "nginx -g 'daemon ..."   4 hours ago         Up About an hour    0.0.0.0:91->80/tcp   nervous_curie

```
- CONTAINER_ID：表示容器ID。

- IMAGE：表示镜像名称。

- COMMAND：表示启动容器时运行的命令。

- CREATED：表示容器的创建时间。

- STATUS：表示容器运行的状态。Up表示运行中，Exited表示已停止。

- PORTS：表示容器对外的端口号。

- NAMES：表示容器名称。该名称默认由Docker自动生成，也可使用docker run命令的–name选项自行指定。
     
### 停止、强制停止、重启、删除容器

```
# 停止容器
docker stop ded3613de77d

# 强制停止容器
docker kill ded3613de77d

# 启动或者重启已经停止的容器
docker start ded3613de77d
docker restart ded3613de77d

# 删除容器
docker rm ded3613de77d
# 删除所有容器
docker rm -f $(docker ps -a -q)
```
### 进入、退出容器

- 使用nsenter工具进入容器（nsenter工具包含在util-linux 2.23或更高版本中）
```
# 找到容器第一个进程的PID，可通过以下命令获取
docker inspect --format "{{.State.Pid}}" $CONTAINER_ID
# 获得PID，使用nsenter命令进入容器
nsenter --target "$PID" --mount --uts --ipc --net --pid
```
- 实战例子

```
[root@gxst_docker_76_16 ~]# docker restart ded3613de77d 
ded3613de77d
[root@gxst_docker_76_16 ~]# docker inspect --format "{{.State.Pid}}" ded3613de77d
26878
[root@gxst_docker_76_16 ~]# nsenter --target 26878 --mount --uts --ipc --net --pid
mesg: ttyname failed: No such file or directory
root@ded3613de77d:/# 
```
- 退出容器，在容器中使用exit命令即可

```
root@ded3613de77d:/# exit
logout
[root@gxst_docker_76_16 ~]# 
```
- 可以结合以上两条命令写一个进入容器脚本

```
# 新建脚本文件 
vim docker-enter.sh
```
- 脚本代码
```
#!/bin/bash
#Use nsenter to access docker
CNAME=$1

CPID=$(docker inspect --format "{{.State.Pid}}" $CNAME)

if [ "$#" -gt 1 ]; then

    nsenter --target $CPID --mount --uts --ipc --net --pid -- "$2"

else

    nsenter --target $CPID --mount --uts --ipc --net --pid -- /bin/bash

fi

```
- 使用脚本进入容器

```
sh docker-enter.sh ded3613de77d（容器名称或者ID）
```