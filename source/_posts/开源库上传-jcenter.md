---
title: 开源库上传 jcenter
date: 2019-09-03 20:15:01
categories:  
- development tool #分类
tags:
- jcenter
---
![logo](https://github.com/maoqitian/MaoMdPhoto/raw/master/jcenter%20upload/logo.png)
> 平时撸代码避免不了在有些功能会使用到别人已经写好的轮子，别人的轮子开源库一般都已经上传了 jcenter仓库，我只需要比如 implementation 'com.mao:xxxxxxx:1.0.0'一句话就能引入别人的开源库，这是怎么弄的呢？一般可以使用bintray-release插件和gradle-bintray-plugin插件，gradle-bintray-plugin插件不够简便（想了解可以看这篇文章[https://www.cnblogs.com/mingfeng002/p/10255486.html](https://www.cnblogs.com/mingfeng002/p/10255486.html)），所以接下来我们就了解一下如何使用bintray-release插件将自己的开源库上传到jcenter。
<!--more-->
# 注册jcenter账号
## 注册
- 进入[注册地址](https://bintray.com/)选择右边sign up here 进行注册

![jcenterRegister](https://github.com/maoqitian/MaoMdPhoto/raw/master/jcenter%20upload/jcenterRegister.png)

## 创建Repository（仓库） 
- 注册成功之后创建Repository，作为存放开源库的仓库，选择为公共仓库（public），仓库名称和仓库类型为maven，仓库名称在后面上传时需要用到。

![createmavenrepository](https://github.com/maoqitian/MaoMdPhoto/raw/master/jcenter%20upload/createmavenrepository.png)

## 获取 API Key
- 登录bintray， Edit profile -> API Key 可以获取上传的秘钥key，后面上传项目的时候需要用到

![getApiKey](https://github.com/maoqitian/MaoMdPhoto/raw/master/jcenter%20upload/getApiKey.png) 

# 配置引入 bintray-release插件

- 项目根目录build.gradle配置，加入bintray-release插件

```
buildscript {
    repositories {
       //原有配置 ，保持不变
        
    }
    dependencies {
        // Android studio 原有IDE 配置，保持不变
        classpath 'com.android.tools.build:gradle:3.4.1'
        //加入bintray-release插件
        classpath 'com.novoda:bintray-release:0.9.1'
      
    }
}

allprojects {
     //原有配置 ，保持不变
}
.......
```
- 开源库目录build.gradle配置，每个配置描述都已经给出，完成这两个步骤，就可以准备上传开源库到jcenter了

```
apply plugin: 'com.android.library'
apply plugin: 'com.novoda.bintray-release'//添加 bintray-release 配置
android {
    compileSdkVersion 28
    defaultConfig {
       //原有配置 ，保持不变

    }

    buildTypes {
        release {
           //原有配置 ，保持不变
        }
    }

}

dependencies {
    //原有配置 ，保持不变 
}

//添加
publish {
    userOrg = 'maoqitian'//bintray.com用户名
    repoName = 'maolibrary'   // bintray上仓库的名字
    groupId = 'com.mao'//jcenter上的路径
    artifactId = 'flexibleflowlayout'//项目名称
    publishVersion = '1.0.0'//版本号
    desc = 'Make flow layouts simpler'// 描述
    website = 'https://github.com/maoqitian/FlowLayout'//一般填github 项目地址,一定是要有效的地址
}
```
# 上传开源库

## 使用上传命令上传开源库
- 上传命令解析

```
gradlew clean build bintrayUpload //根命令
-PbintrayUser=maoqitian //jcenter 账号用户名
-PbintrayKey=Xxxxxxxxx  //文章开头获取的API Key
-PdryRun=false //配置参数，true 执行所以细节但是不上传开源库，false上传开源库
```

- 在项目根目录下执行上传命令

```
# window 下执行
gradlew clean build bintrayUpload -PbintrayUser=maoqitian -PbintrayKey=Xxxxxxxxx -PdryRun=false

# linux 下执行
./gradlew clean build bintrayUpload -PbintrayUser=maoqitian -PbintrayKey=xxxxxxxxx -PdryRun=false
```

![Terminaluploadsuccess](https://github.com/maoqitian/MaoMdPhoto/raw/master/jcenter%20upload/Terminaluploadsuccess.png)

![uploadsuccess](https://github.com/maoqitian/MaoMdPhoto/raw/master/jcenter%20upload/uploadsuccess.png)

- 到这里，我们可以看到开源库已经上传成功，在jcenter也可以看到刚刚上传的开源库

## Add to Jcenter提交审核开源库

- 经过上面的步骤，我确实已经把开源库上传到Jcenter了，但是我们还不能引用，要想引用上传的开源库还得提交人工审核，人工审核通过会收到站内message，并且开源库中的Add to Jcenter 也会消失。

![addtojcenter](https://github.com/maoqitian/MaoMdPhoto/raw/master/jcenter%20upload/addtojcenter.png)

# 版本更新

- 开源库有bug，或者我们进行迭代，就会涉及到版本更新，那就只需要修改开源库目录build.gradle配置中的版本号，其他配置保持不变，再次执行上传开源库命令就可以达到版本更新的目的。

```
publish {
    userOrg = 'maoqitian'//bintray.com用户名
    repoName = 'maolibrary'   // bintray上仓库的名字
    groupId = 'com.mao'//jcenter上的路径
    artifactId = 'flexibleflowlayout'//项目名称
    publishVersion = '2.0.0'//版本号
    desc = 'Make flow layouts simpler'// 描述
    website = 'https://github.com/maoqitian/FlowLayout'//一般填github 项目地址,一定是要有效的地址
}
```

# 参考链接

- [Android 快速发布开源项目到jcenter](https://blog.csdn.net/lmj623565791/article/details/51148825)