---
title: Centos下 Nexus 3.x 搭建Maven 私服
date: 2019-07-03 23:39:36
categories:  
- Nexus Maven 私服
tags:
- Nexus
- CentOS
- Maven
---
>Maven的原理就是将jar从远程中央仓库下载到PC磁盘的本地仓库,当本地仓库没有发现需要的jar就会去Maven默认的远程中央仓库Maven Central（由Apache维护）中寻找,每次需要新的jar后都要从远程中央仓库上下载。那么问题来了？这个远程的中央仓库一定有很多人使用那下载速度一定很慢，这个暂且不用考虑。 重要的是万一哪天公司外网连不上了咋办？而Nexus私服恰好可以解决这个问题。搭建私服的好处是Nexus有效解决了Maven对Apache的远程中央仓库的依赖，当项目需要新的jar时会先在nexus私服下载好以后才会下载到本地。如果发现私服已经存在这个jar包，则会直接从私服下载到本地Maven库，如果没有再去网络上下载。同时，我们也可打包自己的代码变成jar包上传到私服中供公司其他同事下载使用。
<!--more-->
# 准备工作
- 安装Nexus 之前先确定是否已经安装JDK,这里安装的是jdk8版本（如何安装JDK步骤叙述，）

![安装的JDK 版本](https://github.com/maoqitian/MaoMdPhoto/raw/master/Nexus%20maven%E7%A7%81%E6%9C%8D%E9%85%8D%E7%BD%AE/JDK%E5%AE%89%E8%A3%85%E7%89%88%E6%9C%AC.png)

# 安装Nexus 

- [nexus下载地址](https://www.sonatype.com/nexus-repository-oss)

##  下载完成后解压安装
- 解压
```
    tar -zvxf nexus-3.13.0-01-unix.tar.gz -C /opt/

```
- 环境变量配置
```
    vim /opt/nexus-3.13.0-01/bin/nexus
    
    //配置JDK 路径
    INSTALL4J_JAVA_HOME_OVERRIDE=/opt/jdk1.8.0_181
```
- 启动Nexus 
     
```
/opt/nexus-3.13.0-01/bin/nexus start
```
![启动Nexus](https://github.com/maoqitian/MaoMdPhoto/raw/master/Nexus%20maven%E7%A7%81%E6%9C%8D%E9%85%8D%E7%BD%AE/%E5%90%AF%E5%8A%A8Nexus.png)
    
- 浏览器打开Nexus界面，默认端口号是8081（注意需要判断服务器是否开通了该端口号的监听，这里我将默认端口号改成9190）
   
```
    //加入9190端口的监听
    vim /etc/sysconfig/iptables

    查看是否监听端口(如果配置了自己定义的端口，需要先访问该端口一次才能看到监听)

    netstat -ntlp

    //重启防火墙配置（不重启端口还是无法生效）
    service iptables restart
    
     //修改端口号
     vim /opt/nexus-3.13.0-01/etc/nexus-default.properties
     
     //重启Nexus
     /opt/nexus-3.13.0-01/bin/nexus restart
     
     Nexus其他命令
     
     //停止
     nexus stop

     //查看状态
     nexus status
     
     默认登录用户名密码
     admin 
     admin123
     
     卸载
     rm -rf nexus-3.13删除掉安装目录即可
     
     //可以看到Nexus在浏览器中可以打开界面，部署成功，如下图
```
![9190端口加入监听](https://github.com/maoqitian/MaoMdPhoto/raw/master/Nexus%20maven%E7%A7%81%E6%9C%8D%E9%85%8D%E7%BD%AE/9190%E7%AB%AF%E5%8F%A3%E5%8A%A0%E5%85%A5%E7%9B%91%E5%90%AC.png)
    
![编辑Nexus配置文件修改端口号为9190](https://github.com/maoqitian/MaoMdPhoto/raw/master/Nexus%20maven%E7%A7%81%E6%9C%8D%E9%85%8D%E7%BD%AE/%E7%BC%96%E8%BE%91Nexus%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E4%BF%AE%E6%94%B9%E7%AB%AF%E5%8F%A3%E5%8F%B7%E4%B8%BA9190.png)

![Nexus启动成功](https://github.com/maoqitian/MaoMdPhoto/raw/master/Nexus%20maven%E7%A7%81%E6%9C%8D%E9%85%8D%E7%BD%AE/Nexus%E5%90%AF%E5%8A%A8%E6%88%90%E5%8A%9F.png)
 
# 配置jenkins,maven更新到Nexus私服
  
- 修改Jenkins服务器上的Maven的settings.xml文件(路径是Maven安装路径 /opt/apache-maven-3.5.4/conf/)，加入maven访问nexus认证，访问Nexus的帐号密码为上面登录nexus的默认登录用户名密码
   
![maven访问nexus认证](https://github.com/maoqitian/MaoMdPhoto/raw/master/Nexus%20maven%E7%A7%81%E6%9C%8D%E9%85%8D%E7%BD%AE/maven%E8%AE%BF%E9%97%AEnexus%E8%AE%A4%E8%AF%81.png)
- maven 项目pom文件配置私服仓库
```
    <repositories>
        <repository>
            <id>nexus</id> <!--id要和上一步配置的id一致-->
            <name>local nexus</name>
            <url>http://xxxxx:9190/repository/maven-public/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
        </repository>
    </repositories>
    <pluginRepositories>
    <pluginRepository>
        <id>nexus</id>
        <name>local nexus</name>
        <url>http://xxxxx:9190/repository/maven-public/</url>
        <releases>
            <enabled>true</enabled>
        </releases>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </pluginRepository>
    </pluginRepositories>
```
## Nexus 默认的三种类型仓库，创建仓库的时候可以选择这三种
```
    1.group(仓库组类型)：又叫组仓库，用于方便开发人员自己设定的仓库；

    2.hosted(宿主类型)：内部项目的发布仓库（内部开发人员，发布上去存放的仓库）

    3.proxy(代理类型)：从远程中央仓库中寻找数据的仓库（可以点击对应的仓库的 Configuration 页签下 Remote Storage Location 属性的值即被代理的远程仓库的路径）
```
![Nexus 三种仓库类型](https://github.com/maoqitian/MaoMdPhoto/raw/master/Nexus%20maven%E7%A7%81%E6%9C%8D%E9%85%8D%E7%BD%AE/Nexus%20%E4%B8%89%E7%A7%8D%E4%BB%93%E5%BA%93%E7%B1%BB%E5%9E%8B.png)
### proxy(代理类型)
    
- 这里就是代理的意思，代理远程中央 Maven 仓库，当 项目构建访问中央库的时候，先通过代理去远程中央仓库下载依赖包到Nexus 仓库，然后再从Nexus仓库下载到本地。私服我们部署在内网服务器，只要其中一个人从远程中央库下来了，以后相同的依赖包就都是从Nexus私服上进行下载，这样大大加快下载速度，不怕远程中央仓库出现问题。
     
![代理仓库配置](https://github.com/maoqitian/MaoMdPhoto/raw/master/Nexus%20maven%E7%A7%81%E6%9C%8D%E9%85%8D%E7%BD%AE/%E4%BB%A3%E7%90%86%E4%BB%93%E5%BA%93%E9%85%8D%E7%BD%AE.png) 

### hosted(宿主类型)
- 创建布和代理方式创建差不多
- Hosted 是宿主机的意思，就是怎么把第三方的 Jar 放到私服上。 
      Hosted 有三种方式，Releases、SNAPSHOT、Mixed

### group(仓库组类型)
- 将其他仓库类型合并一起（maven public就是group类型），如图所示将其他仓库合在一下提供对外使用
    
![maven public仓库合并其他仓库变成组](https://github.com/maoqitian/MaoMdPhoto/raw/master/Nexus%20maven%E7%A7%81%E6%9C%8D%E9%85%8D%E7%BD%AE/maven%20public%E4%BB%93%E5%BA%93%E5%90%88%E5%B9%B6%E5%85%B6%E4%BB%96%E4%BB%93%E5%BA%93%E5%8F%98%E6%88%90%E7%BB%84.png) 
  
### 仓库属性说明
- maven-central：maven中央库，默认从https://repo1.maven.org/maven2/拉取jar
- maven-releases：私库发行版jar
- maven-snapshots：私库快照（调试版本）jar
- maven-public：仓库分组，把上面三个仓库组合在一起对外提供服务，在本地maven基础配置settings.xml中使用。

# Nexus jar 包上传与删除

## 上传jar包
- 如果我们使用的远程maven中心库有jar包无法下载，或者是我们自己编译好的jar包提供给公司其他人，则可以将本地jar包上传到私服仓库

![nexus 上传jar包](https://github.com/maoqitian/MaoMdPhoto/raw/master/Nexus%20maven%E7%A7%81%E6%9C%8D%E9%85%8D%E7%BD%AE/nexus%20%E4%B8%8A%E4%BC%A0jar%E5%8C%85.png)

## 私服jar删除

- 私服jar出现问题，我们也可以删除私服的jar包重新下载或自行上传

![私服jar删除](https://github.com/maoqitian/MaoMdPhoto/raw/master/Nexus%20maven%E7%A7%81%E6%9C%8D%E9%85%8D%E7%BD%AE/%E7%A7%81%E6%9C%8Djar%E5%88%A0%E9%99%A4.png)

> 到此，Nexus搭建Maven私服服务已经完成.