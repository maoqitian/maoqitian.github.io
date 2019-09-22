---
title: Docker容器学习（三）之Docker 镜像构建
date: 2019-09-22 23:43:34
categories:
- Docker #分类
tags:
- docker
- Maven
- Git
- CentOs
- Nexus
---
![dockerlogo](https://raw.githubusercontent.com/maoqitian/MaoMdPhoto/master/Docker/dockerlogo.jpg)
> Docker 镜像构建一般使用Dockerfile，首先我们需要了解Dockerfile语法（[Dockerfile官方文档](https://docs.docker.com/engine/reference/builder/)），然后我们编写好Dockerfile文件之后就可以开始构建我们的项目对应Docker镜像，如果构建呢？我们可以手动使用docker命令构建，也可以使用开源插件帮助构建，请往下看。
<!--more-->
## Dockerfile 构建镜像
### 自己手动构建
- 首先我们使用maven 命令将项目打包成 jar 包（jar包一般生成在项目的target目录中）

```
 mvn install
```
- 将jar包复制到服务器我们想要的位置，使用docker命令构建docker镜像
```
# 格式：docker build -t 标签名称 Dockerfile的相对位置
docker build -t configserver . 
```
-  Dockerfile 内容
```
# 基于哪个镜像
FROM java:8

# 将本地文件夹挂载到当前容器
VOLUME /data/cloudtest

# 拷贝文件到容器，也可以直接写成ADD xxxxx.jar /app.jar
ADD configserver-0.0.1-SNAPSHOT.jar  /app.jar

# 声明需要暴露的端口
EXPOSE 8666

# 配置容器启动后执行的命令
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]

```
- 启动构建好的镜像

```
docker run -p 8666:8666 configserver
```
### 使用插件来构建镜像
- 使用插件之前，如果本地环境中没有安装docker，则我们也可以使用远程服务器安装的docker来帮助我们构建，如何使用远程docker呢，首先我们要开启docker的远程访问
#### Centos7中docker开启远程访问
- 在/usr/lib/systemd/system/docker.service，配置远程访问。主要是在[Service]这个部分，加上下面两个参数
```
# vim /usr/lib/systemd/system/docker.service
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
```
- 以上配置完成之后我们需要重新读取docker配置文件，重新启动docker服务

```
systemctl daemon-reload
systemctl restart docker
```
- 看进程docker是否已经监听2375端口，命令为 ps aux | grep docker
```
root      8902  0.5  0.1 637948 37388 ?        Ssl  10:34   1:30 /usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
root      8909  0.0  0.0 432312 10904 ?        Ssl  10:34   0:09 docker-containerd -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --metrics-interval=0 --start-timeout 2m --state-dir /var/run/docker/libcontainerd/containerd --shim docker-containerd-shim --runtime docker-runc
```
- 此时docker已经开启远程访问地址为 服务器地址:2375(别忘了服务器防火墙需要开放2375端口)

#### docker-maven-plugin（spotify）
- 使用[docker-maven-plugin](https://github.com/spotify/docker-maven-plugin)来构建镜像
- 配置POM build 模块，使用docker 镜像私服（nexus）作为镜像仓库
```
<build>
        <plugins>
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>1.2.0</version>
                <configuration>
                    <imageName>172.31.116.12:9290/mao/microservice-discovery-eureka:0.0.1</imageName>
                    <dockerDirectory>${project.basedir}</dockerDirectory>
                    <!--远程docker 地址-->
                    <dockerHost>http://172.31.76.16:2375</dockerHost>
                    <!--<dockerCertPath></dockerCertPath>-->
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>

                    <!-- 与maven配置文件settings.xml中配置的server.id一致，用于推送镜像 -->
                    <serverId>docker-nexus</serverId>
                </configuration>
            </plugin>
        </plugins>
    </build>
```
- 创建并上传镜像执行

```
mvn clean package docker:build  -DpushImage
```
- 构建成功，使用docker 查看构建成功的镜像

![使用docker-maven-plugin来构建镜像成功](https://github.com/maoqitian/MaoMdPhoto/raw/master/Docker/%E4%BD%BF%E7%94%A8docker-maven-plugin%E6%9D%A5%E6%9E%84%E5%BB%BA%E9%95%9C%E5%83%8F%E6%88%90%E5%8A%9F.png)
![docker-maven-plugin构建成功的镜像](https://github.com/maoqitian/MaoMdPhoto/raw/master/Docker/docker-maven-plugin%E6%9E%84%E5%BB%BA%E6%88%90%E5%8A%9F%E7%9A%84%E9%95%9C%E5%83%8F.png)

- 可借助imageTags元素更为灵活地指定镜像名称和标签

```
<build>
        <plugins>
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>1.2.0</version>
                <configuration>
                    <imageName>172.31.116.12:9390/test/eureka</imageName>
                    <dockerDirectory>${project.basedir}</dockerDirectory>
                    <!--远程docker 地址-->
                    <dockerHost>http://172.31.76.16:2375</dockerHost>
                    <!--<dockerCertPath></dockerCertPath>-->
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                           <!--Dockerfile文件路径 此路径代表本项目根目录--> <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                    <!--重复构建相同标签名称的镜像，可将forceTags设为true，这样就会覆盖构建相同标签的镜像-->
                    <forceTags>true</forceTags>
                     <!--借助imageTags元素更为灵活地指定镜像名称和标签-->
                    <imageTags>
                        <imageTag>0.0.1</imageTag>
                        <imageTag>latest</imageTag>
                    </imageTags>
                    <!-- 与maven配置文件settings.xml中配置的server.id一致，用于推送镜像 -->
                    <serverId>docker-nexus</serverId>
                </configuration>
            </plugin>
        </plugins>
    </build>
```
- 使用如下命令进行构建（注意构建命令和没有加入imageTags有所不同）

```
 mvn clean package docker:build  -DpushImageTag
```

#### docker-maven-plugin 多项目共同构建

- 以上配置我们只是针对单个项目配置，如果项目中包含多个子项目，我们希望一起打包成镜像并上传到私服，则需要用到phase 将镜像构建绑定到maven 命令上
- 平常打包构建命令为mvn clean package docker:build，而对应maven的命令格式为mvn phase:goal，所以打包构建命令中package docker对应为phase，build则对应goal，这样根据官方文档提示将POM改造得出
```
<build>
        <plugins>
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>1.2.0</version>
                <!--将构建的触发绑定到对应的命令上-->
                <executions>
                    <execution>
                        <id>build-image</id>
                        <phase>package</phase>
                        <goals>
                            <goal>build</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>tag-image</id>
                        <phase>package</phase>
                        <goals>
                            <goal>tag</goal>
                        </goals>
                        <configuration>
                            <image>test1/eureka1:${project.version}</image>
                            <newName>172.31.116.12:9290/test1/eureka1:${project.version}</newName>
                        </configuration>
                    </execution>
                    <execution>
                        <id>push-image</id>
                        <phase>package</phase>
                        <goals>
                            <goal>push</goal>
                        </goals>
                        <configuration>
                            <imageName>172.31.116.12:9290/test1/eureka1:${project.version}</imageName>
                        </configuration>
                    </execution>
                </executions>
                <configuration>
                    <imageName>test1/eureka1:${project.version}</imageName> <!--镜像名-->
                    <!--Dockerfile文件路径 此路径代表本项目根目录-->
                    <dockerDirectory>${project.basedir}</dockerDirectory>
                    <!--本地不安装 docker 使用远程docker 地址，该地址为docker 开启远程访问地址 -->
                    <dockerHost>http://172.31.76.16:2375</dockerHost>
                    <!--<dockerCertPath></dockerCertPath>-->
                    <resources>
                        <resource> <!-- 指定资源文件 -->
                            <targetPath>/</targetPath>  <!-- 指定要复制的目录路径，这里是当前目录 -->
                            <directory>${project.build.directory}</directory> <!-- 指定要复制的根目录，这里是target目录 -->
                            <include>${project.build.finalName}.jar</include> <!-- 指定需要拷贝的文件，这里指最后生成的jar包 -->
                        </resource>
                    </resources>
                    <!--重复构建相同标签名称的镜像，可将forceTags设为true，这样就会覆盖构建相同标签的镜像-->
                    <forceTags>true</forceTags>  <!--覆盖相同标签镜像-->
                    <!-- 与maven配置文件settings.xml中配置的server.id一致，用于推送镜像 -->
                    <serverId>docker-nexus</serverId>
                </configuration>
            </plugin>
        </plugins>
    </build>
```
- 如此我们只要在项目根目录使用命令，则项目就会自动打包构建镜像并上传到nexus私服
```
## -DskipTests 表示 跳过Test
mvn clean package -DskipTests
```
![整个项目构建成功并上传私服](https://github.com/maoqitian/MaoMdPhoto/raw/master/Docker/%E6%95%B4%E4%B8%AA%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E6%88%90%E5%8A%9F%E5%B9%B6%E4%B8%8A%E4%BC%A0%E7%A7%81%E6%9C%8D.png)

## Docker 私有仓库结合 nexus 搭建与使用
- 每次构建好的镜像我们存放在哪呢，这里可以搭建自己的私服仓库来存放docker镜像。
### 安装nexus
- [Centos下 Nexus 搭建Maven 私服](https://juejin.im/post/5d1cd04b6fb9a07eb051de8e)
### 创建私有仓库
![nexus创建docker私服仓库](https://github.com/maoqitian/MaoMdPhoto/raw/master/Docker/nexus%E5%88%9B%E5%BB%BAdocker%E7%A7%81%E6%9C%8D%E4%BB%93%E5%BA%93.png)

### docker连接私服仓库
- 上一小节我们创建好了私服仓库，还需配置，在docker服务器新建文件

```
# 配置 vim /etc/docker/daemon.json 填写我们仓库地址和对应的仓库端口号

{
    "insecure-registries": ["xxx.xxx.xxx.xxx:9290"] 
    }
# 重启docker
systemctl daemon-reload
systemctl restart docker

```

- 登录仓库，输入nexus的用户名和密码，默认用户名为admin，密码为admin123

```
docker login xxx.xxx.xxx.xxx:9290
```
