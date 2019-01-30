---
title: Jenkins+Gitlab+Maven+Tomcat 持续集成，自动部署项目
date: 2019-01-14 15:55:49
toc: true #是否显示文章目录
categories:  
- 持续集成（CI） #分类
tags: 
- Jenkins
- git
- GitLab
- CentOS
- Maven
- Tomcat
---
<img src="https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/Jenkins.jpg" width=80% />
>持续集成：Continuous Integration，简称CI，意思是，在一个项目中，任何人对代码库的任何改动，都会触发CI服务器自动对项目进行构建，自动运行测试，甚至自动部署到测试环境。这样做的好处就是，随时发现问题，随时修复。因为修复问题的成本随着时间的推移而增长，越早发现，修复成本越低。当你想要更新你的项目，只要动手提交代码到你的代码仓库，剩余的更新部署操作就只管交由CI服务器来完成就好，这次使用的CI工具是JenKins。
<!--more-->
- 搭建Jenkins持续集成服务器可以分为两大步骤，一是在服务器安装好所需的软件，二是配置我们的持续集成项目
<!--more-->
### 安装所需的各种软件
#### 安装启动 Jenkins
- 从Jenkins[官方网站](https://jenkins.io/)下载最新的rpm包
```
    执行命令 rpm -ivh xxx.rpm  安装Jenkins
    
    //启动JenKins
    /etc/init.d/jenkins start 
    
    浏览器输入 http://xxx服务器地址:8080/

    //默认端口号是8080
```
![JenKins启动1](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/Jenkins%E5%90%AF%E5%8A%A81.png)
![JenKins启动](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/JenKins%E5%90%AF%E5%8A%A8.png)
- JenKins默认端口号是8080，这与Tomcat默认端口号冲突，所有我们可以把Jenkins的端口号改成我们自己定义的端口号 9090
    - 修改端口号的文件为 /etc/sysconfig/jenkins，字段为JENKINS_PORT
```
      //执行命令进行修改，如果碰到无法启动Jnekins,无法启动（如遇此Starting Jenkins bash: /usr/bin/java: No such file or directory错误 ）修改 /etc/init.d/jenkins 加入 /opt/jdk1.8.0_181/bin/java原因
      //是Java的环境变量没有找到，一般使用centos服务默认安装openjdk，如果自己卸载openJdk并重新安装sun的JDK,则也需要在该文件中加入路径，如图所示
      
      vim /etc/sysconfig/jenkins 
      
      //添加Java地址
      
      vim /etc/init.d/jenkins
```
![修改Jenkins端口](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/%E4%BF%AE%E6%94%B9Jenkins%E7%AB%AF%E5%8F%A3.png)
![添加Java地址](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/%E6%B7%BB%E5%8A%A0Java%E5%9C%B0%E5%9D%80.png)
      
- 修改端口无法启动的情况，有可能是服务器防火墙没有添加端口的监听，导致无法访问
    
```
    vim /etc/sysconfig/iptables
  
    查看是否监听端口(如果配置了自己定义的端口，需要先访问该端口一次才能看到监听)
  
    netstat -ntlp
  
    //重启防火墙配置（不重启端口还是无法生效）
    service iptables restart
```
![添加新监听的端口](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/%E6%B7%BB%E5%8A%A0%E6%96%B0%E7%9B%91%E5%90%AC%E7%9A%84%E7%AB%AF%E5%8F%A3.png)

#### 关闭Jenkins
- 只需要在访问jenkins服务器的网址url地址后加上exit。例如我jenkins的地址http://localhost:8080/，那么我只需要在浏览器地址栏上输入http://xxx:9090/exit 网址就能关闭jenkins服务.
#### 重启Jenkies

```
     //xxx:8080 是搭建Jenkins服务器地址
     http://xxx:8080/restart
```
- 重新加载配置信息
```
     http://localhost:8080/reload
```
- Jenkins的卸载
```
        1. 卸载软件：rpm -e jenkins

        2. 删除遗留文件: find / -iname jenkins | xargs -n 1000 rm -rf
```
#### 安装jdk
```
    查看Java相关的包信息： 
     rpm -qa|grep java （或rpm -qa|grep jdk，rpm安装后，包名里没有Java）
    卸载 
    yum -y remove java [包名] 
    例如 
    yum –y remove java java-1.8.0-openjdk-1.8.0.131-3.b12.el7_3.x86_64 
    
    jdk 卸载方法 https://blog.csdn.net/xyj0808xyj/article/details/52444694
    
    //解压到指定目录
    tar -zxvf jdk-8u181-linux-x64.tar.gz -C /opt/
    
    //编辑配置文件
    vim /etc/profile
    
    export JAVA_HOME=/opt/jdk1.8.0_181
    export PATH=$JAVA_HOME/bin:$PATH
    export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$CLASSPATH 
      
    //更新配置文件
    source /etc/profile
```
#### 安装Tomcat
- 官网下载 [Tomcat8.5](https://tomcat.apache.org/download-80.cgi)
- 直接解压到服务器（注意这里指的服务器是项目部署的服务器，应该是和部署JenKins的服务器不同） 
- 强制关闭 tomcat 命令
```
      //强制关闭
      ps -ef|grep tomcat
    
      //杀掉无法关闭进程
      kill -9 XXXX
```
#### 安装 git (安装在部署JenKins服务器上)
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
![安装git版本](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/%E5%AE%89%E8%A3%85git%E7%89%88%E6%9C%AC.png)
#### 安装 maven(安装在部署JenKins服务器上)
- 后台项目为spring-boot搭建，需要安装maven
- 下载 [maven 3.5.4](http://maven.apache.org/download.cgi)
```
    执行以下命令
    tar -zxvf apache-maven-3.3.9-bin.tar.gz -C /opt
    
    在/etc/profile文件末尾增加以下配置
    M2_HOME=/opt/apache-maven-3.5.4 （注意这里是maven的安装路径）
     export PATH=${M2_HOME}/bin:${PATH}
     
    重载/etc/profile这个文件
      source /etc/profile 
```
![maven安装成功](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/maven%E5%AE%89%E8%A3%85%E6%88%90%E5%8A%9F.png)
  
#### GitLab服务器配置
- 由于公司已经搭建好Gitlab服务器，所以我也没有配置过Gitlab，不过Gitlab配置网上已经有很多资料，可以自行百度。
  
### Jenkins基础工具配置、新建项目配置
#### Jenkins配置
- 第一次进入Jnekins,首先根据提示找到安装服务器的密码
- 如果服务器可以联网，则选择他推荐的插件直接安装，如果服务器无法连接外网，只是在公司内网环境，则可以离线下载插件再上传到我们服务器的Jenkin中，[离线插件下载地址](https://plugins.jenkins.io/)，这种方式需要耐心，因为需要安装的插件可不止一两个，如果你的服务器不能上网，我这有一份下载好的插件，可以自行去下载（[下载地址](https://github.com/maoqitian/MaoMdPhoto/tree/master/Jenkins%E6%8F%92%E4%BB%B6)）
- 在Jenkins系统管理模块的系统配置中配置我们的Gitlab,需要登录到Gitlab中获取APIToken
![系统配置GitLab](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/%E7%B3%BB%E7%BB%9F%E9%85%8D%E7%BD%AEGitLab.png)
![获取API Token](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/%E8%8E%B7%E5%8F%96API%20Token.png)
![GitLab API Token配置 ](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/GitLab%20API%20Token%E9%85%8D%E7%BD%AE%20.png)
#### 生成ssh key（在部署Jenkins服务器上生成）
- 配置SSH KEY ,用于后续项目可以通过Jenkins部署到应用服务器
          
```
 //输入命令，一路回车
 ssh-keygen -t rsa 
          
 //现在你的私钥被放在了~/.ssh/id_rsa 这个文件里，而公钥被放在了 ~/.ssh/id_rsa.pub 这个文件里
          
 //可以将私钥配置到JneKins的系统设置中，配置框选项是需要Jenkins安装SSH插件的，如下图所示
          
 //公钥则配置到各个应用服务器的这个目录下/root/.ssh/authorized_keys，没有authorized_keys则创建这个文件，如下图所示
          
 //最后测试应用服务器是否都能成功连接，如下图所示
          
 ```
![ssh私钥配置](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/ssh%E7%A7%81%E9%92%A5%E9%85%8D%E7%BD%AE.png)
![公钥复制到应用服务器authorized_keys](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/%E5%85%AC%E9%92%A5%E5%A4%8D%E5%88%B6%E5%88%B0%E5%BA%94%E7%94%A8%E6%9C%8D%E5%8A%A1%E5%99%A8authorized_keys.png)
![测试配置公钥的应用服务器是否连接成功](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/%E6%B5%8B%E8%AF%95%E9%85%8D%E7%BD%AE%E5%85%AC%E9%92%A5%E7%9A%84%E5%BA%94%E7%94%A8%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%98%AF%E5%90%A6%E8%BF%9E%E6%8E%A5%E6%88%90%E5%8A%9F.png)

#### 工具配置
- 接下来还是系统管理模块中的全局工具配置 Jenkins的 JDK、git和maven。前面我们已经把这些工具都给安装了，现在配置到Jenkins中，如下图所示
        
![配置Jenkins jdk git](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/%E9%85%8D%E7%BD%AEJenkins%20jdk%20git.png)
![配置Jenkins maven](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/%E9%85%8D%E7%BD%AEJenkins%20maven.png)
    
#### 新建项目配置
       
- 新建一个maven项目（没有maven项目选项则需要下载对应插件）
         
![新建maven项目](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/%E6%96%B0%E5%BB%BAmaven%E9%A1%B9%E7%9B%AE.png)
         
- 首先配置源码管理，如图
       
![新建项目源码管理配置](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/%E6%96%B0%E5%BB%BA%E9%A1%B9%E7%9B%AE%E6%BA%90%E7%A0%81%E7%AE%A1%E7%90%86%E9%85%8D%E7%BD%AE.png)
- 配置项目构建触发器（Gitlab Hook Plugin， Outbound WebHook for build events，Build Authorization Token Root， Success
Build Token Trigger插件）
       
![配置项目构建触发器](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/%E9%85%8D%E7%BD%AE%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%A7%A6%E5%8F%91%E5%99%A8.png)
         
- 配置maven项目编译
        
![配置maven项目编译](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/%E9%85%8D%E7%BD%AEmaven%E9%A1%B9%E7%9B%AE%E7%BC%96%E8%AF%91.png)
         
- 项目构建成功后部署应用服务器的配置 
       
![项目构建成功后部署应用服务器的配置](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E6%88%90%E5%8A%9F%E5%90%8E%E9%83%A8%E7%BD%B2%E5%BA%94%E7%94%A8%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%9A%84%E9%85%8D%E7%BD%AE.png)
 - Tomcat重启脚本(应当放在与Tomcat目录同路径下)
```
         #! /bin/sh
         echo '####################开始自动部署####################'
         export JAVA_HOME=/usr/local/jdk1.8.0_181
         path=`pwd` #当前路径
         tomcatPath=/data/XXXX/tomcat_gxxmt_8080 #指定tomcat文件目录名称
         cd ../$tomcatPath/bin #进入tomcat的bin目录
         PID=$(ps -fu `whoami`|grep tomcat|grep -v grep|awk '{print $2}')
         if [ -z "$PID" ];then
          echo "no tomcat process"
         else
         ./shutdown.sh #停止tomcat服务
         fi
         cd ../webapps #进入tomcat的webapps目录
         rm -rf XXXX-api
         echo '####################删除完成####################'
         #rm -fr gxxmt-api.war #删除test文件目录
         #mv gxxmt-api.war gxxmt-api.war.$(date +%Y%m%d) #备份webapps下的test16 cp $path/test.war ./ #复制test.war到webapps路径下
         #cd /var/lib/jenkins/workspace/gxxmt-api/gxxmt-api/target/
         #cp gxxmt-api.war /data/gxxmt/tomcat_gxxmt_8080/webapps/
         cd ../bin
         ./startup.sh #启动tomcat服务
         echo '####################部署结束####################'
```
- 项目构建成功并发布到了对应服务器执行对应脚本，这里就可以看到JenKins的灵活性，可以配置多台发布的应用服务器的多个Tomcat,灵活自动部署应用服务器配置
        
- 项目构建编译部署成功
        
![项目构建编译部署成功](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E7%BC%96%E8%AF%91%E9%83%A8%E7%BD%B2%E6%88%90%E5%8A%9F.png)
          
### 集成部署遇到的问题
#### 问题一
- ERROR: Exception when publishing, exception message [Exec timed out or was interrupted after 120,000 ms]（执行脚本没有正常退出，导致部署超时）
 ![错误1](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/%E9%94%99%E8%AF%AF1.png)
- 解决：脚本执行加入忽略输入（nohup ....）
```
     nohup sh /data/gxxmt-api/restart.sh
     //当我们使用nohup命令的的时候，日志会被打印到nohup.out文件中去。
     //如果我们不做任何处理，会随着每次的重新启动，nohup.out会越来越大
     //。所以我在我执行的脚本中添加了
     cp /dev/null nohup.out
```
- 在jenkins项目配置SSH Publishers勾选了Exec in pty，表示执行完脚本立即退出
![错误1解决勾选Exec in pty](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/%E9%94%99%E8%AF%AF1%E8%A7%A3%E5%86%B3%E5%8B%BE%E9%80%89Exec%20in%20pty.png)
#### 问题二，Jenkins目录迁移     
- jenkins主目录迁移，jenkins默认主目录一般都是安装在系统盘，运行一段时间后项目部署的历史版本，日志文件，工作控件都会占用大量的系统空间，这样就会引发系统盘磁盘空间不足，首先我们可以修改jenkins主目录
```
     //更改主目录
     vim /etc/sysconfig/jenkins
     JENKINS_HOME="/data/jenkins"
     
     复制 /var/lib/jenkins/ 目录到 /data目录下
     
     修改目录用户权限
     
     chown -R jenkins:jenkins /data/jenkins
     
     重启 /etc/sysconfig/jenkins restart
```
![主目录修改成功](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/%E4%B8%BB%E7%9B%AE%E5%BD%95%E4%BF%AE%E6%94%B9%E6%88%90%E5%8A%9F.png)
     
- 其次我们还可以在项目配置中设置丢弃历史构建
     
![设置丢弃历史构建](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/%E8%AE%BE%E7%BD%AE%E4%B8%A2%E5%BC%83%E5%8E%86%E5%8F%B2%E6%9E%84%E5%BB%BA.png)

#### 问题三，代码提交触发构建
- Url is blocked: Requests to the local network are not allowed Gitlab设置Jenkins的webhook地址无法设置
     
![WebHook地址无法设置](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/WebHook%E5%9C%B0%E5%9D%80%E6%97%A0%E6%B3%95%E8%AE%BE%E7%BD%AE.png)

- 升级新版Gitlab，要允许WebHook，需要在在Gitlab的Admin账户中，在settings标签下面，找到OutBound Request，勾选上Allow requests to the local network from hooks and services ，保存更改即可解决问题（如下图所示）
     
![Gitlab 允许WebHook](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/Gitlab%20%E5%85%81%E8%AE%B8WebHook.png)
     
#### 问题四
- Gradle 编译Android 项目 /lib64/libc.so.6: version `GLIBC_2.14’ not found，系统是CentOS 6.9，最高支持glibc的版本为2.12，而研发程序要2.14版本，所以需要升级。    
    ![glibc_2.14库无法找到](https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/glibc_2.14%E5%BA%93%E6%97%A0%E6%B3%95%E6%89%BE%E5%88%B0.png)
```
    //查看系统版本
    cat /etc/redhat-release
    //查看glibc库版本
    strings /lib64/libc.so.6 |grep GLIBC_
    //下载glibc库 并安装
    #下载
    wget http://ftp.gnu.org/gnu/glibc/glibc-2.14.tar.gz 
    wget http://ftp.gnu.org/gnu/glibc/glibc-ports-2.14.tar.gz 
    #解压
    tar -xvf  glibc-2.14.tar.gz 
    tar -xvf  glibc-ports-2.14.tar.gz
    #创建相关目录
    mv glibc-ports-2.14 glibc-2.14/ports
    mkdir glibc-build-2.14
    cd glibc-build-2.14/ 

    #生成C编译的环境
    yum -y install gcc

    #编译C
    ../glibc-2.14/configure  --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin
    make

    # 安装刚才编译好的 libc.so
    makeinstall 

    # 查看glibc库版本
    strings /lib64/libc.so.6 |grep GLIBC_
```

### 最后说点
- 到此，我们的持续集成服务器已经搭建完成，这时候你只要动手提交一下代码到你前面构建触发器设置的分支（一般为主分支），剩余的项目构建，部署等一系列重复繁琐的工作就交由Jenkins帮我们自动完成就可以了，省时又方便。文章中如果有错误，请大家给我提出来，大家一起学习进步，如果觉得我的文章给予你帮助，也请给我一个喜欢或者关注。

- 参考链接:
    - [jenkins的安装与使用（基于 centos 7）](https://blog.csdn.net/xiyatu123/article/details/53039749)
    
    - [jenkins maven Spring Boot git Linux持续集成环境搭建教程](https://www.jianshu.com/p/d4f2953f3ce0)
    - [解决SDK升级到27.0.3遇到的GLIBC_2.14 not found](https://blog.csdn.net/ouyang_peng/article/details/79974407 )