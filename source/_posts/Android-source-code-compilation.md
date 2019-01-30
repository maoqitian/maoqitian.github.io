---
title: Android 源码编译
date: 2019-01-10 20:23:03
toc: true #是否显示文章目录
categories:  
- Android进阶 #分类
tags: 
- Android
- 源码编译
---
> android源码编译的四个流程:1.源码下载;2.构建编译环境;3.编译源码;4运行.

## Ubuntu 18.04（虚拟机）环境下编译Android 源码
### 源码下载
   
  - 首先确保自己已经安装了Git.
    
    ```
    sudo apt-get install git 
    git config –global user.email “test@test.com” 
    git config –global user.name “test”

    ```
  - 使用清华大学镜像
    -  [Android 镜像使用帮助](https://mirror.tuna.tsinghua.edu.cn/help/AOSP/)
    -  首先要下载repo 工具
    
    ```
    mkdir ~/bin
    PATH=~/bin:$PATH
    curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
    chmod a+x ~/bin/repo
    //拒绝连接可以使用tuna的git-repo镜像
    详情查看网址https://mirrors.tuna.tsinghua.edu.cn/help/git-repo/
    ```
    - 使用每月更新的初始化包
    - 下载地址 [每月更新的初始化包](https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar)
    
    - 由于所有代码都是从隐藏的 .repo 目录中 checkout 出来的，所以只保留了 .repo 目录，下载后解压 再 repo sync 一遍即可得到完整的目录
    
    ```
    使用方法如下:

    wget -c https://mirrors.tuna.tsinghua.edu.cn/aosp-mo nthly/aosp-latest.tar # 下载初始化包
    tar xf aosp-latest.tar
    cd AOSP   # 解压得到的 AOSP 工程目录
    # 这时 ls 的话什么也看不到，因为只有一个隐藏的 .repo 目录
    repo sync # 正常同步一遍即可得到完整目录
    # 或 repo sync -l 仅checkout代码
    ```
    
    
### 构建编译环境
#### 硬件要求:
- 64位的操作系统只能编译2.3.x以上的版本,如果你想要编译2.3.x以下的,那么需要32位的操作系统. 
    磁盘空间越多越好,至少在100GB以上.意思就是,你可以去买个大点的硬盘了啊 
    如果你想要在是在虚拟机运行linux,那么至少需要16GB的RAM/swap. 
  
#### 软件要求 
  
- 安装 openJDK 8 
    ```
    sudo apt-get update
    sudo apt-get install openjdk-8-jdk
 
    ```
    
- 依赖设置:
     ```
     sudo apt-get install libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-dev g++-multilib 
     sudo apt-get install -y git flex bison gperf build-essential libncurses5-dev:i386 
     sudo apt-get install tofrodos python-markdown libxml2-utils xsltproc zlib1g-dev:i386 
     sudo apt-get install dpkg-dev libsdl1.2-dev libesd0-dev
     sudo apt-get install git-core gnupg flex bison gperf build-essential  
     sudo apt-get install zip curl zlib1g-dev gcc-multilib g++-multilib 
     sudo apt-get install libc6-dev-i386 
     sudo apt-get install lib32ncurses5-dev x11proto-core-dev libx11-dev 
     sudo apt-get install libgl1-mesa-dev libxml2-utils xsltproc unzip m4
     sudo apt-get install lib32z-dev ccache
     ```
- 依赖设置中有可能会出现“无法定位软件包 libesd0-dev” 这个问题
     ```
       解决方案：
       在etc/apt   的sources.list 添加镜像源   deb http://archive.ubuntu.com/ubuntu/ trusty main universe restricted multiverse
       
       然后 sudo apt-get update  接着继续使用该命令安装就可以了
     ```
- 操作系统要求 

    Android版本 | 编译要求的Ubuntu最低版本
    ---|---
    Android 6.0至AOSP master | Ubuntu 14.04
    Android 2.3.x至Android 5.x | Ubuntu 12.04
    Android 1.5至Android 2.2.x | Ubuntu 10.04

- JDK版本要求
  
Android版本 | 编译要求的JDK版本
---|---
AOSP的Android主线 | OpenJDK 8
Android 5.x至android 6.0 | Oracle JDK 7
Android 2.3.x至Android 4.4.x | 	Oracle JDK 6
Android 1.5至Android 2.2.x | 	Oracle JDK 5

- 官方编译环境搭建文档地址
   
  [搭建编译环境](https://source.android.com/source/initializing#installing-required-packages-ubuntu-1404)


### 初始化编译环境
   ```
   source build/envsetup.sh 
   或者
   . build/envsetup.sh

   ```
- 选择目标

   ```
   . lunch aosp_arm64-eng

   ```
- 该命令表示针对模拟器进行完整编译，并且所有调试功能均处于启用状态。
   如果您没有提供任何参数就运行命令，lunch 将提示您从菜单中选择一个目标。
   所有编译目标都采用 BUILD-BUILDTYPE 形式，其中 BUILD 是表示特定功能组合的代号。
- BUILDTYPE 是以下类型之一：
     
       编译类型 | 使用情况
       ---|---
       user | 权限受限；适用于生产环境（没有root权和dedug等）
       userdebug |在user版本的基础上开放了root权限和debug权限.
       eng | 开发工程师的版本,拥有最大的权限,此外还附带了许多debug工具


### 编译源码
- 您可以使用make编译任何代码。GNUMake可以借助 -jN参数处理并行任务，通常使用的任务数N介于编译时所用计算机上硬件线程数的1-2倍之间。例如，在一台双核 E5520 计算机（2 个 CPU，每个 CPU 4 个内核，每个内核2个线程）上，要实现最快的编译速度，可以使用介于make -j16 到 make -j32 之间的命令。
  ```
  make -j8
  ```
- 编译中
    ![Android源码等待编译过程](https://github.com/maoqitian/MaoMdPhoto/raw/master/Android%20%E6%BA%90%E7%A0%81%E7%BC%96%E8%AF%91/Android%E6%BA%90%E7%A0%81%E7%AD%89%E5%BE%85%E7%BC%96%E8%AF%91%E8%BF%87%E7%A8%8B.png)

### 编译完成
  ![编译完成](https://github.com/maoqitian/MaoMdPhoto/raw/master/Android%20%E6%BA%90%E7%A0%81%E7%BC%96%E8%AF%91/%E7%BC%96%E8%AF%91%E5%AE%8C%E6%88%90.png)
- 运行模拟器
     ```
     //依次输入以下命令（如果是在编译成功源码之后直接想运行模拟器，则直接输入emulator命令就行，因为前面编译源码已经输入过以上两条命令）
     
     . build/envsetup.sh
     
     lunch(选择刚才你编译源码设置的目标版本)
     
     emulator
     
     ```
     ![运行模拟器](https://github.com/maoqitian/MaoMdPhoto/raw/master/Android%20%E6%BA%90%E7%A0%81%E7%BC%96%E8%AF%91/%E8%BF%90%E8%A1%8C%E6%A8%A1%E6%8B%9F%E5%99%A8.png)

  
### 将源码导入Android Studio 查看
- 编译idegen
    
    ```
    source build/envsetup.sh // 将执行文件设置为临时变量
    mmm development/tools/idegen/  //生成idegen.jar文件（#### build completed successfully (49 seconds) #### 标识生成idegen.jar文件）

    ```
- 执行脚本 idegen.sh
    
    ```
    . development/tools/idegen/idegen.sh

    ```
    看到下图，表示编译idegen完成，执行成功后在asop的根目录下生成android.ipr和android.iml两个个文件：
     
     - android.ipr 一般保存了工程相关的设置，比如modules和modules libraries的路径，编译器配置，入口点等。
     - android.iml 用来描述modules。它包括modules路径、 依赖关系，顺序设置等。一个项目可以包含多个 *.iml 文件。

    ![编译idegen](https://github.com/maoqitian/MaoMdPhoto/raw/master/Android%20%E6%BA%90%E7%A0%81%E7%BC%96%E8%AF%91/%E7%BC%96%E8%AF%91idegen.png)
    
     -  打开Android studio，点击File>Open，选择刚刚生成的android.ipr导入就可以了，依据个人机子性能问题，导入时间有快有慢。
  


### 编译中遇到的坑
- Error: library-pathout/host/linux-x86/lib64/libsepolwrap.so does not exist
  
  ![Android源码编译失败1](https://github.com/maoqitian/MaoMdPhoto/raw/master/Android%20%E6%BA%90%E7%A0%81%E7%BC%96%E8%AF%91/Android%E6%BA%90%E7%A0%81%E7%BC%96%E8%AF%91%E5%A4%B1%E8%B4%A51.png) 
- 解决 ： 
>1.重新同步代码 并加入sudo apt-get install dpkg-dev libsdl1.2-dev libesd0-dev  
2.确认是否配置好了JDK的环境变量 

- openJDK 配置环境变量方法
    ```
    1.用gedit文本编辑器在/etc/profile中添加环境变量：
    命令 ： sudo gedit /etc/profile
    
    2.在打开的/etc/profile文件末尾添加下面几行：
    export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
    export JRE_HOME=${JAVA_HOME}/jre 
    export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib 
    export PATH=${JAVA_HOME}/bin:$PATH

    ```
### 最后说点 
- 到此，Android源码编译完成，源码编译是一个需要耐心的过程，希望看到文章的你也可以编译成功。文章中如果有错误，请大家给我提出来，大家一起学习进步，如果觉得我的文章给予你帮助，也请给我一个喜欢或者关注。    

- 参考资料
  
  [搭建编译环境](https://source.android.com/source/initializing#optimizing-a-build-environment)

  [动手实现Android源码（AOSP）的下载、编译、运行、导入、调试](https://blog.csdn.net/mcryeasy/article/details/60466837)
  
  [Android 镜像使用帮助](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)
  