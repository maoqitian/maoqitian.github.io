---
title: Docker容器学习（四）之Docker Compose
date: 2019-09-30 15:26:49
categories:
- Docker #分类
tags:
- docker
- Maven
- Git
- CentOs
- docker-compose
- wait-for-it
---
![dockerlogo](https://raw.githubusercontent.com/maoqitian/MaoMdPhoto/master/Docker/dockerlogo.jpg)
>之前的文章中，我们使用docker run 命令来启动一个容器，而作为真正的线上业务环境，我们服务肯定不止一个，也就说明容器肯定不止一个，而如果还是手动的一个个来启动容器这未免会让人头皮发麻，幸好有**Docker Compose**，用于定义和运行多容器Docker应用程序的工具，有了它我们可以一次启动多个容器，这也非常适合与持续集成工具（Jenkins）来配合。
<!--more-->
### 安装 Docker Compose
- 根据[官方](https://docs.docker.com/compose/install/)安装最新版本

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
- 安装脚本添加执行权限

```
chmod +x /usr/local/bin/docker-compose
```

- **注意：** 根据前面的步骤理论上是安装完成了，但是我们执行官方命令只是下载到了/usr/local/bin/这个目录，添加脚本执行权限之后docker-compose并不能生效，依据官方文档提示复制该文件到/usr/bin/目录下，并未该文件添加脚本执行权限

```
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

chmod +x /usr/bin/docker-compose
```
- 此时我们执行docker-compose版本命令就能看到版本号打印

```
# docker-compose --version
docker-compose version 1.24.0, build 0aa59064
```
- 命令补全工具

```
sudo curl -L https://raw.githubusercontent.com/docker/compose/1.24.0/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
```
- 版本
![Docker Compose版本对应](https://github.com/maoqitian/MaoMdPhoto/raw/master/Docker/Docker%20Compose%E7%89%88%E6%9C%AC%E5%AF%B9%E5%BA%94.png)

###  docker-compose.yml 命令
- https://docs.docker.com/compose/compose-file/
###  docker-compose 命令
- 和前面学习Docker容器命令一样，docker-compose也有不少命令

命令 | 含义| 示例
---|---|---
build | 构建或重新构建服务| build [options] [--build-arg key=val...] [SERVICE...]（使用 docker-compose help build 查看详细使用）
help | 查看docker-compose命令帮助文档| docker-compose help COMMAND（标识要看的命令）
up  | 构建、创建、重新创建、启动，连接服务的相关容器。所有连接的服务都会启动，除非它们已经运行(前提该目录下已经存在docker-compose.yml文件)| docker-compose up(直接启动，该命令退出，所有容器停止) docker-compose up -d (后台运行所有容器)
kill | 发送SIGKILL 信号停止指定服务的容器| docker-compose kill api-feign(注意该名称为docker-compose.yml中定义的服务名称) 
start | 启动指定服务已存在的容器| docker-compose start api-feign
stop | 停止指定服务已存在的容器| docker-compose stop api-feign
logs | 查看服务的日志输出 |docker-compose logs --tail="all" api-feign（查看api-feign 全部日志输出）
ps   |列出所有容器| docker-compose ps （和docker ps -a 一样可以查看容器，显示信息不一样）
rm   |删除指定服务的容器| docker-compose rm api-feign

- 更多docker-compose命令查看[docker-compose官方文档](https://docs.docker.com/compose/reference/overview/)

### 编写docker-compose.yml 启动多服务

- 服务器任意目录编写文件 docker-compose.yml
```
version: '3.4'
services:
  configerver:  # 指定一个服务名称
    image: mao/configserver:0.0.1-SNAPSHOT  # 镜像名称
    ports:
      - 8666:8666  # 指定端口映射
  eureka:
    image: test1/eureka1:0.0.1-SNAPSHOT
    ports:
      - 8805:8805
  server-admin:
    image: mao/server-admin:0.0.1-SNAPSHOT
    ports:
      - 8806:8806
  api-feign:
    image: mao/api-feign:0.0.1-SNAPSHOT
    ports:
      - 8840:8840
  ribbon-consumer:
    image: mao/ribbon-consumer:0.0.1-SNAPSHOT
    ports:
      - 8830:8830
  ribbon-provider:
    image: mao/ribbon-provider:0.0.1-SNAPSHOT
    ports:
      - 8820:8820
```
- 在docker-compose.yml目录下执行命令启动多个服务(安装好Docker Compose前提下)

```
# docker compose 构建镜像并使用镜像启动容器（-d 表示后台启动）
docker-compose up -d 
```
![docker-compose.yml启动对应的服务](https://github.com/maoqitian/MaoMdPhoto/raw/master/Docker/docker-compose.yml%E5%90%AF%E5%8A%A8%E5%AF%B9%E5%BA%94%E7%9A%84%E6%9C%8D%E5%8A%A1.png)

- 由上图我们发现各个服务都已经启动在各自的容器当中，但是访问服务的时候只有configserver（配置中心）能够访问，其他服务都不能访问，我们仔细想想就能知道，除了配置中心，其他服务的配置文件都要通过配置中心来获取，但是docker-compose启动是同时的，所以配置中心服务还没提供其他服务就已经启动了，这显然会让其他服务报错，所以在生产环境中我们必须要控制服务的启动顺序，也就是最先启动配置中心，然后启动注册中心，最后再启动其他服务。

### Docker Compose控制服务启动顺序
- [Docker Compose控制服务启动顺序官方文档说明](https://docs.docker.com/compose/startup-order/)，本文使用[wait-for-it](https://github.com/vishnubob/wait-for-it)方案，除了wait-for-it，还有[dockerize](https://github.com/jwilder/dockerize)和 [wait-for](https://github.com/Eficode/wait-for) 方案。
- wait-for-it是一个bash脚本，执行该脚本将等待主机和TCP端口的可用性。它可用于同步相互依赖的服务的启动，例如链接的docker容器
### 如何使用
- 接下来我将介绍如何使用该脚本来控制我们服务启动顺序
- 首先我们可以将脚本打包到我们的镜像中，修改Dockerfile文件
```
#Dockerfile中 
....
COPY wait-for-it.sh /wait-for-it.sh #在本项目模块根目录下复制wait-for-it.sh 到镜像/目录下
RUN chmod +x /wait-for-it.sh # 修改脚本权限
....
```
- 再次使用docker-maven-plugin打包镜像
- 重新编写docker-compose.yml文件，添加entrypoint执行我们前面打包如镜像的wait-for-it.sh脚本监控配置中心是否已经提供服务，注意ENTRYPOINT指令是不会被覆盖的，最终会执行监控./wait-for-it.sh configerver:8666配置中心是否提供服务来通过配置中心获取配置来启动其他服务

```
version: '3.7'
services:
  configerver:  # 指定一个服务名词
    image: mao/configserver:0.0.1-SNAPSHOT  # 镜像名称
    ports:
      - 8666:8666  # 指定端口映射
    depends_on:
      - eureka
  eureka:
    image: mao/eureka:0.0.1-SNAPSHOT
    ports:
      - 8805:8805
    entrypoint: "./wait-for-it.sh configerver:8666 -- java -jar /app.jar"
  server-admin:
    image: mao/server-admin:0.0.1-SNAPSHOT
    ports:
      - 8806:8806
    depends_on:
      - eureka
      - configerver
    entrypoint: "./wait-for-it.sh configerver:8666 -- java -jar /app.jar"
  api-feign:
    image: mao/api-feign:0.0.1-SNAPSHOT
    ports:
      - 8840:8840
    depends_on:
      - eureka
      - configerver
    entrypoint: "./wait-for-it.sh configerver:8666 -- java -jar /app.jar"
  ribbon-consumer:
    image: mao/ribbon-consumer:0.0.1-SNAPSHOT
    ports:
      - 8830:8830
    depends_on:
      - eureka
      - configerver
    entrypoint: "./wait-for-it.sh configerver:8666 -- java -jar /app.jar"
  ribbon-provider:
    image: mao/ribbon-provider:0.0.1-SNAPSHOT
    ports:
      - 8820:8820
    depends_on:
      - eureka
      - configerver
    entrypoint: "./wait-for-it.sh configerver:8666 -- java -jar /app.jar"
  gateway:
    image: mao/gateway:0.0.1-SNAPSHOT
    ports:
      - 8081:8081
    depends_on:
      - eureka
      - configerver
    entrypoint: "./wait-for-it.sh configerver:8666 -- java -jar /app.jar"
```
- 然后我们在服务器中执行docker-compose命令后台一键启动服务，不出意外服务就能够全部启动

```
docker-compose up -d
```
![后台有顺序启动服务成功](https://github.com/maoqitian/MaoMdPhoto/raw/master/Docker/%E5%90%8E%E5%8F%B0%E6%9C%89%E9%A1%BA%E5%BA%8F%E5%90%AF%E5%8A%A8%E6%9C%8D%E5%8A%A1%E6%88%90%E5%8A%9F.png)
### 最后
- 本篇文章我们学习了如何使用Docker Compose来启动多个容器，而多个容器改如何管理呢？谷歌已经给了我们答案，那就是使用k8s，而k8s是什么呢，请看我的这一篇文章[Kubeadm 部署 Kubernetes 1.14.2 集群](https://juejin.im/post/5d4145696fb9a06b2a201c47)

## 参考链接
- [Docker is installed but Docker Compose is not ? why?
](https://stackoverflow.com/questions/36685980/docker-is-installed-but-docker-compose-is-not-why/47061271)
- [wait for it Usage with Docker](https://github.com/vishnubob/wait-for-it/issues/57)