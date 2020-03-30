---
title: Flutter-WanAndroid
date: 2020-03-30 23:03:57
categories:  
- Flutter探索 #分类
tags: 
- dart
- Flutter
top: true
---
![image](https://raw.githubusercontent.com/maoqitian/MaoMdPhoto/master/flutter/flutter-logo.png)
## 前言
- Flutter作为当下最火的移动跨平台技术，它是谷歌的推出的移动UI框架，可以快速在iOS和Android上构建高质量的原生用户界面。

[![Flutter][1]][2]  [![Dart][3]][4] [![Release][5]][6]  [![GitHub license][7]][8]

[1]:https://img.shields.io/badge/Flutter-1.12.13-5bc7f8.svg
[2]:https://flutter.dev

[3]:https://img.shields.io/badge/Dart-2.7.0%2B-00B4AB.svg
[4]:https://dart.dev

[5]:https://img.shields.io/github/release/maoqitian/flutter_wanandroid.svg
[6]:https://github.com/maoqitian/flutter_wanandroid/releases/latest

[7]:https://img.shields.io/badge/license-Apache%202-blue.svg
[8]:https://github.com/maoqitian/flutter_wanandroid/blob/master/LICENSE

## 项目简介
- 这是一款跨平台的开源Flutter版本玩Android App。首先感谢[**鸿洋**](https://github.com/hongyangAndroid)大佬提供的[玩Android开放API](https://www.wanandroid.com/blog/show/2)；其次，本应用提供丰富完整的功能，更好的体验，旨在随时随地都能更好的浏览[https://www.wanandroid.com/](https://www.wanandroid.com/)网站内容，更好的在手机上进行学习。整个应用涉及到了大部分常用的Flutter组件，Flutter界面搭建，页面跳转，网络请求，Json解析转换，数据持久化，组件间消息通信等Flutter学习尝试，可以说是一个比较好的Flutter学习项目，也希望能对看到此项目的您有或多或少的帮助。项目如果对您有帮助，不妨点个**Star**，您的支持是我前进的动力。


## 编译运行环境
```
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, v1.12.13+hotfix.8, on Mac OS X 10.15.2 19C57, locale en-CN)
[✓] Android toolchain - develop for Android devices (Android SDK version 29.0.2)
[✓] Xcode - develop for iOS and macOS (Xcode 11.1)
[✓] Android Studio (version 3.5)

Dart 2.7.0
```
<!--more-->


## 项目结构图
![image](https://github.com/maoqitian/MaoMdPhoto/raw/master/flutter/flutter_wanandroid/flutter-wanandroid%E7%BB%93%E6%9E%84%E5%9B%BE.jpg)

## 下载

- [**历史版本下载地址**](https://github.com/maoqitian/flutter_wanandroid/releases)

### **最新版本下载**
- 可以手机浏览器输入以下地址下载 [http://d.alphaqr.com/9n12](http://d.alphaqr.com/9n12)


类型 | 二维码
---|---
Apk 下载二维码 |<img src="https://github.com/maoqitian/MaoMdPhoto/raw/master/flutter/flutter_wanandroid/version/v1.0.5/flutter-wanandroid-dimensional-v1.0.5.png"  height="200" width="200">
ios 下载| 暂无下载，可以自行clone项目编译体验


## 项目截图展示

### gif （debug 模式略显卡顿，可下载release版本体验丝滑顺畅）

<img src="https://github.com/maoqitian/flutter_wanandroid/raw/master/preview/showapp1.gif"  height="300" width="170"><img src="https://github.com/maoqitian/flutter_wanandroid/raw/master/preview/showapp2.gif"  height="300" width="170"><img src="https://github.com/maoqitian/flutter_wanandroid/raw/master/preview/showapp3.gif"  height="300" width="170">

### ios 截图

<img src="https://github.com/maoqitian/flutter_wanandroid/raw/master/preview/flutter-home-page.png"  height="300" width="170"><img src="https://github.com/maoqitian/flutter_wanandroid/raw/master/preview/flutter-knowledge.png"  height="300" width="170"><img src="https://github.com/maoqitian/flutter_wanandroid/raw/master/preview/flutter-nav.png"  height="300" width="170"><img src="https://github.com/maoqitian/flutter_wanandroid/raw/master/preview/flutter-project.png"  height="300" width="170"><img src="https://github.com/maoqitian/flutter_wanandroid/raw/master/preview/flutter-search.png"  height="300" width="170">

<img src="https://github.com/maoqitian/flutter_wanandroid/raw/master/preview/flutter-user-center.png"  height="300" width="170"><img src="https://github.com/maoqitian/flutter_wanandroid/raw/master/preview/flutter-theme-change.png"  height="300" width="170"><img src="https://github.com/maoqitian/flutter_wanandroid/raw/master/preview/flutter-wechat.png"  height="300" width="170"><img src="https://github.com/maoqitian/flutter_wanandroid/raw/master/preview/flutter-login1.png"  height="300" width="170"><img src="https://github.com/maoqitian/flutter_wanandroid/raw/master/preview/flutter-coin-rank.png" height="300" width="170">

<img src="https://github.com/maoqitian/flutter_wanandroid/raw/master/preview/flutter-article-list.png"  height="300" width="170"><img src="https://github.com/maoqitian/flutter_wanandroid/raw/master/preview/flutter-common-web.png" height="300" width="170"><img src="https://github.com/maoqitian/flutter_wanandroid/raw/master/preview/flutter-todo.png" height="300" width="170"><img src="https://github.com/maoqitian/flutter_wanandroid/raw/master/preview/flutter-add-todo.png" height="300" width="170"><img src="https://github.com/maoqitian/flutter_wanandroid/raw/master/preview/flutter-todo-empty.png" height="300" width="170">

<img src="https://github.com/maoqitian/flutter_wanandroid/raw/master/preview/flutter-dark-home.png"  height="300" width="170"><img src="https://github.com/maoqitian/flutter_wanandroid/raw/master/preview/flutter-dark-knowledge.png"  height="300" width="170"><img src="https://github.com/maoqitian/flutter_wanandroid/raw/master/preview/flutter-dark-nav.png"  height="300" width="170"><img src="https://github.com/maoqitian/flutter_wanandroid/raw/master/preview/flutter-dark-project.png"  height="300" width="170"><img src="https://github.com/maoqitian/flutter_wanandroid/raw/master/preview/flutter-dark-wechat.png" height="300" width="170">


## API
[**玩Android开放API**](https://www.wanandroid.com/blog/show/2)

## 项目功能
### 首页
- 首页文章列表
- 首页banner
- 常用网站
- 搜索热词（包含在搜索界面）
- 置顶文章
- 最新项目tab (首页的第二个tab)

### 知识体系
- 体系数据
- 知识体系下的文章
- 按照作者昵称搜索文章

### 公众号
- 获取公众号列表
- 查看某个公众号历史数据

### 导航
- 导航数据

### 项目
- 项目分类
- 项目列表数据

### 登录与注册
- 登录、注册功能

### 收藏
- 收藏文章列表
- 收藏站内文章
- 收藏站外文章
- 取消收藏
- 收藏网站列表
- 收藏网址
- 编辑收藏网站
- 删除收藏网站

### 搜索
- 首页文章搜索
- 在某个公众号中搜索历史文章

### TODO工具
- TODO 列表
- 新增一个 TODO
- 更新一个 TODO
- 删除一个 Todo
- 仅更新完成状态TODO

### 积分
- 积分排行榜
- 获取个人积分
- 获取个人积分获取列表

### 广场
- 广场列表数据
- 分享人对应列表数据
- 自己的分享的文章列表（个人中心）
- 删除自己分享的文章（个人中心）
- 分享文章

### 问答

- 问答列表文章

### 设置
- 夜间模式
- 清除缓存
- 版本信息
- 退出登录

### 主题切换
- 切换App 主题

### 个人中心
点击头像进入个人中心，仿B站个人中心效果

## Thanks
- 感谢所有开源库的作者
### 参考项目
- [flutter-go](https://github.com/alibaba/flutter-go)

### 使用的第三方库
第三方库 | 功能
---|---
[fluro](https://github.com/theyakka/fluro) | 页面跳转路由框架
[shared_preferences](https://github.com/flutter/plugins) | 本地存储
[dio](https://github.com/flutterchina/dio) | 网络
[json_annotation](https://github.com/dart-lang/json_serializable) | json 序列化
[flutter_webview](https://github.com/flutter/plugins) | webview
[fluttertoast](https://github.com/PonnamKarthik/FlutterToast) | Toast
[provider](https://github.com/rrousselGit/provider) | 跨组件数据共享
[event_bus](https://github.com/marcojakob/dart-event-bus) | 事件总线
[flutter_spinkit](https://github.com/marcojakob/dart-event-bus) | 加载中指示器动画
[extended_nested_scroll_view](https://github.com/fluttercandies/extended_nested_scroll_view) | NestedScrollView 扩展
[flutter_easyrefresh](https://github.com/xuelongqy/flutter_easyrefresh) | 配合NestedScrollView扩展下拉刷新以及上拉加载
[flutter_staggered_grid_view](https://github.com/letsar/flutter_staggered_grid_view) | 瀑布流
[package_info](https://github.com/flutter/plugins) | 方便获取应用信息
[flutter_html](https://github.com/Sub6Resources/flutter_html) | 加载html 字符串
[expandable](https://github.com/aryzhov/flutter-expandable) | 扩展显示隐藏
[date_format](https://github.com/tejainece/date_format) | 日期转换
[share](https://github.com/flutter/plugins/tree/master/packages/share) | 分享
## 版本更新日志

### v1.0.5 (2020/03/15)

#### 完善功能

- 个人中心添加积分显示
- 修复文章item显示越界问题

### v1.0.4 (2020/03/12)

#### 完善功能

- 添加侧边栏个人中心入口
- 完善文章tag显示
- 修复上个版本存在的bug

### v1.0.3 (2020/03/10)

#### 完善功能

- 添加文章分享功能
- 优化主题切换功能
- 修复上个版本存在的bug

### v1.0.2 (2020/03/08)

#### 添加TODO模块
- TODO 列表
- 新增一个 TODO
- 更新一个 TODO
- 删除一个 Todo
- 仅更新完成状态TODO
- 修复上个版本存在的bug

### v1.0.1 (2020/02/18)

- 添加问答模块
- 修复上个版本存在的bug

### v1.0.0 (2020/02/15)
- Flutter 项目第一个版本， 完成WanAndroid基本功能

## Statement
项目中的 API 均来自于 [wanandroid.com](https://www.wanandroid.com/) 网站，纯属学习交流使用，不得用于商业用途。