---
title: git 使用
date: 2019-05-07 23:41:54
categories:  
- development tool #分类
tags:
- Git
---
![Git logo](https://github.com/maoqitian/MaoMdPhoto/raw/master/Git/Git%20logo.png)
> 在日常开发的过程中，对于代码版本的控制已经是是一个习以为常的功能了，接下就记录一下使用Git来作为版本控制的一些常用操作命名，方便自己查看回顾。
<!--more-->
## git 基本操作
- git 提交流程
```
git init (初始化本地仓库)

git add .

git commit -m "提交描述"

git remote add origin git@github.com:maoqitian/MyPracticeView.git （首先确定ssh key 是否存在，否则会拉取失败）


git pull --rebase origin master （合并操作）


git push -u origin master (提交到远程仓库)

```
- master 分支链接成功之后，拉取分支代码

```
git fetch origin wiki 把远程分支拉到本地
git checkout -b wiki origin/wiki 在本地创建分支wiki并切换到该分支
git pull origin wiki 拉分支
```


- 查看用户名

```
git config user.name
git config user.email
```
- 修改配置用户名 

```
git config --global user.name "your name"
git config --global user.email "your email"
```

### 分支切换与合并

- [参考链接](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001375840038939c291467cc7c747b1810aab2fb8863508000://note.youdao.com/)

- 创建新分支

```
git checkout -b dev
```
- 切换主分支 （切换分支出现Your local changes to the following files would be overwritten by checkout [解决办法](https://blog.csdn.net/qq_32452623/article/details/75645578)，说白了就是有文件没跟踪到，重新提交一下就可以解决，也可强制切换(不推荐)）

```
git checkout master
```
- 合并分支

```
 git merge dev
```
- 拉取、提交到远程代码

```
#拉取
 git pull origin master
# 提交
git push origin master
```
## .gitignore 文件不起作用
- 清除对应文件夹的提交记录缓存 
```
git rm --cached --force -r gxxmt-admin/target/
```
- 清除所有文件夹的缓存记录 
```
git rm -r --cached .
```
- 清除完成之后重复 add 和 commit操作则能使用新的忽略规则  
 
- 删除GitLab 上的文件，首先克隆代码
```
git rm -r --cached target
git commit -m 'delete'
git push -u origin master

```
## fork 同步远程代码到本账户仓库
  
```
git remote -v(查看链接库情况)

git remote add upstream git@172.17.118.127:activity/gxxmt.git（添加远程连接）

git fetch upstream （拉取远程更新）

git merge upstream/master （合并远程更新到本地，此时只是本地仓库和远程主分支同步，要保持gitlab 仓库和远程同步，还需提交）

前两步合并操作 git pull upstream master

git push origin master


//分支获取
git checkout wiki（切换分支）

git pull upstream wiki （获取wiki分支更新 相当于git fetch 和merge 一起操作）

git push origin wiki(提交更新到远程库（自己fork的库）)
```

- 获取代码并切换分支

```
 git clone git@172.31.116.11:maoqitian/gxxmt.git
 
 git checkout -b wiki origin/wiki
```
- 提交代码到github不显示正确的提交记录（github不显示小绿点），查询git 设置的邮箱和github对应的邮箱是否一致
```
//查看设置的邮箱
git config  user.email “username@mail.com”
//设置邮箱
git config --global user.email “username@mail.com”
```
## git tag
- git tag 打标签(漫长版本迭代中比较重要)
  
- 正常提交一个版本流程 
```
  git checkout master (切换主分支)
  git merge dev(开发分支)
  git add .
  git commit -m "提交说明"
  git tag -a v1.8.1 -m '版本说明'
  git push origin master (远程同步分支)
  git push --tag
```
- 对以前提交记录加入 tag 
  
```
  git log --pretty=oneline (显示提交历史)
  
  git tag -a v1.2 9fceb02 -m "版本说明" (提交对象的校验和（或前几位字符）) 
  git push origin v1.2 (远程同步 tag )
  (或者 远程同步所有tags) git push origin --tags 
```
- 删除 tag 
  
```
  git tag (查看tag )
  git tag -d 标签名 (本地删除)
  git push origin :refs/tags/标签名 (远程标签删除)
```

 

