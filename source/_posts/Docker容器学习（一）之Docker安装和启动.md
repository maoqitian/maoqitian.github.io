---
title: Docker容器学习（一）之Docker安装和启动
date: 2019-09-15 23:06:53
categories:
- Docker #分类
tags:
- docker
- Maven
- Git
- CentOs
---
![dockerlogo](https://raw.githubusercontent.com/maoqitian/MaoMdPhoto/master/Docker/dockerlogo.jpg)
# 什么是Docker
> 首先我们了解什么是Docker
- [Docker介绍](https://github.com/eacdy/spring-cloud-book/blob/master/3%20%E4%BD%BF%E7%94%A8Docker%E6%9E%84%E5%BB%BA%E5%BE%AE%E6%9C%8D%E5%8A%A1/3.1%20Docker%E4%BB%8B%E7%BB%8D.md)
# Docker 准备工作
- 目前使用服务器为CentOS 7.6
- 使用Docker构建微服务首先我们需要Java环境（JDK）,Maven和Git
<!--more-->
## 安装JDK
- 到Oracle官网下载好 [jdk-8u211-linux-x64.rpm](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) 备用
- 首先查看系统自带java，并卸载
```
# 如果有结果出来，则说明自带了java
java -version  
# 查询出已经安装的java
rpm -qa|grep java    

yum -y remove [删除上面查出来的东西，多个用空格分隔]
```
- 安装JDK

```
cd /usr
mkdir /usr/java
rpm -ivh jdk-8u65-linux-x64.rpm
```
- 配置环境变量，编辑/etc/profile文件找到： export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL 这一行，并在其下面一行添加如下内容，最后使profile文件环境变量生效

```
#编辑/etc/profile文件
vim /etc/profile

# 设置java环境变量
export JAVA_HOME=/usr/java/jdk1.8.0_211-amd64 # 根据情况修改
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

#使profile文件环境变量生效
source /etc/profile
```
- 最后可以使用如下命令查看Java 版本
```
java -version
```
## Maven的安装
- 下载 [maven 3.6.1](http://maven.apache.org/download.cgi)
```
    # 执行以下命令
    tar -zxvf apache-maven-3.6.1-bin.tar.gz -C /data/opt
    
    # 在/etc/profile文件末尾增加以下配置
    # 设置Maven环境变量
    export MAVEN_HOME=/data/opt/apache-maven-3.6.1/
    export PATH=$MAVEN_HOME/bin:$PATH
     
    # 重载/etc/profile这个文件
     source /etc/profile 
```
- 测试

```
mvn -v
```
- Maven本地仓库配置（/data/opt/apache-maven-3.6.1/conf/settings.xml）

```
<localRepository>/data/maven/repo</localRepository>
```
- 配置maven私服地址 和 登录私服账号密码

```
 <!--私服账号配置-->
   <server>  
      <id>nexus</id>  
      <username>admin</username>  
      <password>admin123</password>  
    </server>
    <server>  
      <id>3rdParty</id>  
      <username>admin</username>  
      <password>admin123</password>  
    </server>	
 <!--私服地址配置-->
   <mirror>
	  <id>nexus</id>  
      <name>local nexus</name>
      <url>http://172.31.116.12:9190/repository/maven-public/</url>  
      <mirrorOf>central</mirrorOf> 
    </mirror>	
```
## 安装Git

```
    安装依赖
    yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel asciidoc
    
    yum install  gcc perl-ExtUtils-MakeMaker
    
    如果已经安装了git,但是版本太老，可以先卸载
    yum remove git 
    
    下载git包解压
    tar -vxf git-2.15.1.tar.gz -C /opt
    
    进入git 目录
    cd /opt/git-2.18.0/
    
    执行以下命令
    
    make prefix=/usr/local/git all
    
    make prefix=/usr/local/git install
    
    echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/profile  //配置环境变量
    
    source /etc/profile //跟新配置文件
    
    git --version 查看git版本
```
# Docker 安装
- [Docker 官方安装文档](https://docs.docker.com/install/linux/docker-ce/centos/)
## 准备工作
- 卸载老版本的Docker

```
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
- 在新的一台机器上安装Docker，首先我们需要设置Docker的存储仓库，然后我们就可以从存储仓库中安装和更新Docker

```
# 安裝所需的包。 yum-utils提供yum-config-manager實用程序，devicemapper存儲驅動程序需要device-mapper-persistent-data和lvm2。
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
# 使用以下命令设置稳定版本（stable）存储库。
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
- 需要安装epel源 才能yum安装container-selinux

```
# 安装wget 网络工具
yum -y install wget

wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo 

yum install epel-release   #阿里云上的epel源
```
## 开始安装
- 直接安装最新版本

```
yum install docker-ce docker-ce-cli containerd.io
```

- 查看可以安装版本

```
yum list docker-ce --showduplicates | sort -r
```
![Docker版本](https://user-gold-cdn.xitu.io/2019/8/30/16cde51224a123cb?w=742&h=434&f=png&s=46946    )
- 指定安装的版本

```
# 官方方法
yum install docker-ce-17.03.1 docker-ce-cli-17.03.1 containerd.io
```
```
# 直接安装
yum install docker-ce-17.09.1.ce-1.el7.centos
# 或者安装其他版本
yum install docker-ce-18.09.6 
```
## Docker 启动

```
systemctl start docker
```
- 放入测试镜像

```
docker pull library/hello-world
```
- 启动测试镜像

```
docker run hello-world
```
- 出现如下输出说明安装成功
```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

# 参考链接
-  [Docker官方文档](https://docs.docker.com/install/linux/docker-ce/centos/)