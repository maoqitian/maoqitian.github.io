---
title: 从零开始 hexo 个人博客搭建完全指南
date: 2019-03-12 20:07:38
categories:  
- hexo博客搭建 #分类
tags: 
- hexo
- Next主题
- Github
- Git
---
![image](https://avatars2.githubusercontent.com/u/6375567?s=200&amp;v=4)
> 个人博客对于我们知识的积累过程中起到温故而知新的作用，并且也能达到展示自我的目的。接下来就大致介绍一下以[hexo](https://github.com/hexojs/hexo)为基础搭建个人博客的过程。
<!--more-->
### 准备工作
- 安装 [git](https://git-scm.com/downloads)[link](https://note.youdao.com/) node.js（直接安装就行） 
- 安装 [Hexo](https://hexo.io/zh-cn/)
  
  ```
  npm install -g hexo-cli
  ```
  ![hexo安装](https://github.com/maoqitian/MaoMdPhoto/raw/master/hexo%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA/hexo%E5%AE%89%E8%A3%85.png)
- 假设你已经有了自己的github账号
### 建站
- 选择一个目录初始化hexo
  ```
  hexo init
  ```
  ![初始化hexo](https://github.com/maoqitian/MaoMdPhoto/raw/master/hexo%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA/%E5%88%9D%E5%A7%8B%E5%8C%96hexo.png)
- 创建 hexo
  ```
  npm install
  ```
  ![创建hexo](https://github.com/maoqitian/MaoMdPhoto/raw/master/hexo%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA/%E5%88%9B%E5%BB%BAhexo.png)
  
- 开启hexo本地服务
  ```
  hexo s
  ```
  ![hexo本地启动](https://github.com/maoqitian/MaoMdPhoto/raw/master/hexo%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA/hexo%E6%9C%AC%E5%9C%B0%E5%90%AF%E5%8A%A8.png)
  
  ![访问启动本地hexo站](https://github.com/maoqitian/MaoMdPhoto/raw/master/hexo%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA/%E8%AE%BF%E9%97%AE%E5%90%AF%E5%8A%A8%E6%9C%AC%E5%9C%B0hexo%E7%AB%99.png)
  
### hexo关联github

- 首先需要有github账号（没有则先注册）
- 其次创建github仓库，仓库名称为<用户名>.github.io

- 安装hexo-deployer-git插件。在命令行（即Git Bash）运行以下命令即可
  
   ```
   npm install hexo-deployer-git --save

   ```
   ![安装hexo-deployer-git插件](https://github.com/maoqitian/MaoMdPhoto/raw/master/hexo%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA/%E5%AE%89%E8%A3%85hexo-deployer-git%E6%8F%92%E4%BB%B6.png)
- 修改_config.yml（在站点目录下）。文件末尾修改为（注意冒号之后必须添一个空格）
   ```
   # Deployment
   ## Docs: https://hexo.io/docs/deployment.html
   deploy:
    type: git
    repository: git@github.com:maoqitian/maoqitian.github.io.git
    branch: master
   ```
- 推送到GithubPages。在命令行（即Git Bash）依次输入以下命令， 返回INFO Deploy done: git即成功推送：
   ```
   $ hexo g
   $ hexo d
   ```
  ![推送到github成功](https://github.com/maoqitian/MaoMdPhoto/raw/master/hexo%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA/%E6%8E%A8%E9%80%81%E5%88%B0github%E6%88%90%E5%8A%9F.png)
- 访问我们刚刚搭建好的[githubPages](maoqitian.github.io)博客 
  
  ![访问githubPages博客](https://github.com/maoqitian/MaoMdPhoto/raw/master/hexo%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA/%E8%AE%BF%E9%97%AEgithubPages%E5%8D%9A%E5%AE%A2.png)

### 绑定域名
  - 域名解析（需要到购买域名的网站设置）
  ```
  类型选择为 CNAME；

  主机记录即域名前缀，填写为www；

  记录值填写为自定义域名；

  解析线路，TTL 默认即可

  ```
  ![设置域名解析添加记录](https://github.com/maoqitian/MaoMdPhoto/raw/master/hexo%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA/%E8%AE%BE%E7%BD%AE%E5%9F%9F%E5%90%8D%E8%A7%A3%E6%9E%90%E6%B7%BB%E5%8A%A0%E8%AE%B0%E5%BD%95.png)
- 绑定域名可能出现https连接不安全，可以获取免费证书，参照链接： [Hexo绑定自定义Https域名](https://juejin.im/entry/5a8bd9f25188257a5911cfc6)
### 美化hexo博客样式
#### 安装主题 NexT （官方文档已经写得很详细）
  
   - https://github.com/iissnan/hexo-theme-next
#### 开启动效背景
##### 方式1 主题自带的效果  
- 主题配置文件_config.yml将false改为 true
    
```
    # Canvas-nest
    canvas_nest: true

```
##### 方式2 设置不是主题自带的效果
- 主题中新添加内容 _layout.swig 文件

```
# 找到themes\next\layout\_layout.swig文件，添加内容：
# 在<body>里添加：
<div class="bg_content">
  <canvas id="canvas"></canvas>
</div>

# 仍是该文件，在末尾添加：
<script type="text/javascript" src="/js/src/dynamic_bg.js"></script>
```
- 添加动效背景js代码 dynamic_bg.js
- 在themes\next\source\js\src中新建文件dynamic_bg.js，js代码详情可见：[dynamic_bg.js](https://github.com/maoqitian/maoqitian.github.io/blob/master/js/src/dynamic_bg.js)
- 修改样式 custom.styl
```
# 在themes\next\source\css\_custom\custom.styl文件末尾添加内容：
.bg_content {
  position: fixed;
  top: 0;
  z-index: -1;
  width: 100%;
  height: 100%;
}
```

#### 设置圆形头头像
 
   > /themes/next/source/css/_common/components/sidebar/sidebar-author.styl 
  
    ```
    .site-author-image {
    display: block;
    margin: 0 auto;
    padding: $site-author-image-padding;
    max-width: $site-author-image-width;
    height: $site-author-image-height;
    border: $site-author-image-border-width solid $site-author-image-border-color;
    border-radius: 60%;
    transition: 2.5s all;  
    }

    .site-author-image:hover {
    transform: rotate(360deg);
    } 


    .site-author-name {
    margin: $site-author-name-margin;
    text-align: $site-author-name-align;
    color: $site-author-name-color;
    font-weight: $site-author-name-weight;
    }

    .site-description {
    margin-top: $site-description-margin-top;
    text-align: $site-description-align;
    font-size: $site-description-font-size;
    color: $site-description-color;
    }
    ```
#### 主题NexT修改网站标志
  
  > themes\next\source\images 路径下替换favicon-16x16-next.png

  > themes\next\source\images路径修改头像替换avatar.jpg，并且在主题配置文件中开启头像设置
  
  ```
  avatar: /images/avatar.jpg

  ```
#### 修改网页头背景图
  
  >themes\next\source\css\_common\components\header路径下
  
  
  ```
  .header {background-image: url(图片地址 或者图片路径images\xxx.jpg);}
  ```
#### 修改logo字体、menu背景（字体文件需要自行下载）
   
   - 在themes/*/source/css/_custom/custom.styl中添加如下代码：
  
  ```
  // Custom styles.
     @font-face {
     font-family: Bungasai;
     src: url('/fonts/Bungasai.ttf');
     }
    .site-title {
    font-size: 45px !important;
	font-family: 'Bungasai' !important;
    }
  ```
  - 其中字体文件在 themes/next/source/fonts 目录下，里面有个 .gitkeep 的隐藏文件，打开写入你要保留的字体文件
  - 去掉logo字体背景图
  > 在theme/next/source/css/_common/components/header文件夹下打开site-meta.styl文件，找到.brand{},去掉background: $brand-bg 字段

  - 修改menu样式（\blog\themes\next\source\css\_common\components\header\menu.styl文件）
  
  ```
  // Menu
  // --------------------------------------------------
  .menu {
  margin-top: 20px;
  padding-left: 0;
  text-align: center;
  background: rgba(255,255,255,0.55);
  margin-left: auto;
  margin-right: auto;
  width: 470px;
  border-radius: initial;
  }

  .menu .menu-item {
  display: inline-block;
  margin: 0 10px;
  list-style: none;

   @media screen and (max-width: 767px) {
    margin-top: 10px;
   }

   a {
    display: block;
    font-size: 18px;
    line-height: inherit;
    border-bottom: 1px solid $menu-link-border;
    transition-property: border-color;
    the-transition();

    &:hover { border-bottom-color: $menu-link-hover-border; }
   }

   .fa { margin-right: 5px; }
   }

  .use-motion .menu-item { opacity: 0; }

  ```
#### 添加RSS
  > 在博客文件夹下面 blog/ 使用git bash 下载插件
  
  ```
  npm install --save hexo-generator-feed
  ```
  > 打开主题配置文件搜索rss并修改为如下
  
  ```
  rss: /atom.xml
  ```
  > 重新启动发布博客hexo clean清除缓存后$ hexo g 生成静态文件，在文件夹(public)下看到 atom.xml 文件
#### 设置侧边栏社交小图标
  
  >打开主题配置文件搜索social,把#去掉就可以启用，如需新增在[图标库](https://fontawesome.com/icons)找自己喜欢的小图标，并将名字复制按social格式修改即可
   
  ```
  social:
  GitHub: https://github.com/maoqitian || github
  掘金: https://juejin.im/user/59e956626fb9a045204b57d4 || drupal 
  简书: https://www.jianshu.com/u/f58cd7ff1a08 || book（图标库图标名称）
  
  social_icons:
  enable: true   #是否启用图标
  icons_only: false  #是否启只显示图标不显示文字
  transition: false
  ```
#### 自定义网站底部
  > 打开主题配置文件搜索 footer 并按如下对应项修改
  
  ```
  footer:
  since: 2018     #指定建站时间，如果没有定义，则使用主题创建当年
  icon: heart     #修改底部心型图形
  powered: false  #关闭默认数据驱动支持

  #关闭主题名称和版本号
  theme:
    enable: false 
    version: false
  ```
  - 底部添加访客数和总访问量(编辑主题配置文件)
  
  ```
  busuanzi_count:
  # count values only if the other configs are false
  enable: true
  # custom uv span for the whole site
  site_uv: true
  #本站访客数
  site_uv_header: <i class="fa fa-user"></i> 本站访客数
  site_uv_footer: 人次
  # custom pv span for the whole site
  site_pv: true
  site_pv_header: <i class="fa fa-eye"></i> 本站总访问量
  site_pv_footer: 次
  # custom pv span for one page only
  page_pv: true
  page_pv_header: <i class="fa fa-file-o"></i> 本文阅读量
  page_pv_footer: 次
  ```
#### 网页加载样式
  >编辑主题配置文件
  
  ```
  # 开启网页加载
  pace: true
  # 可以设置的加载样式列表:
  #pace-theme-big-counter
  #pace-theme-bounce
  #pace-theme-barber-shop
  #pace-theme-center-atom
  #pace-theme-center-circle
  #pace-theme-center-radar
  #pace-theme-center-simple
  #pace-theme-corner-indicator
  #pace-theme-fill-left
  #pace-theme-flash
  #pace-theme-loading-bar
  #pace-theme-mac-osx
  #pace-theme-minimal
  # For example
  # pace_theme: pace-theme-center-simple
  # 设置网页加载样式
  pace_theme: pace-theme-center-atom
  ```
#### 在右上角添加fork me on github
  - 样式地址
    
     - [样式1](https://blog.github.com/2008-12-19-github-ribbons/)
     - [样式2](http://tholman.com/github-corners/)
  >在上面样式连接中 挑选自己喜欢的样式，并复制代码。然后把刚才复制的代码粘贴到 themes\next\layout\_layout.swig 文件中(放在 <div class="headband"></div> 的下面)，并把href改为你的github地址
  
  ```
  <a href="https://your-url" class="github-corner" aria-label="View source on GitHub"><svg width="80" height="80" viewBox="0 0 250 250" style="fill:#151513; color:#fff; position: absolute; top: 0; border: 0; right: 0;" aria-hidden="true"><path d="M0,0 L115,115 L130,115 L142,142 L250,250 L250,0 Z"></path><path d="M128.3,109.0 C113.8,99.7 119.0,89.6 119.0,89.6 C122.0,82.7 120.5,78.6 120.5,78.6 C119.2,72.0 123.4,76.3 123.4,76.3 C127.3,80.9 125.5,87.3 125.5,87.3 C122.9,97.6 130.6,101.9 134.4,103.2" fill="currentColor" style="transform-origin: 130px 106px;" class="octo-arm"></path><path d="M115.0,115.0 C114.9,115.1 118.7,116.5 119.8,115.4 L133.7,101.6 C136.9,99.2 139.9,98.4 142.2,98.6 C133.8,88.0 127.5,74.4 143.8,58.0 C148.5,53.4 154.0,51.2 159.7,51.0 C160.3,49.4 163.2,43.6 171.4,40.1 C171.4,40.1 176.1,42.5 178.8,56.2 C183.1,58.6 187.2,61.8 190.9,65.4 C194.5,69.0 197.7,73.2 200.1,77.6 C213.8,80.2 216.3,84.9 216.3,84.9 C212.7,93.1 206.9,96.0 205.4,96.6 C205.1,102.4 203.0,107.8 198.3,112.5 C181.9,128.9 168.3,122.5 157.7,114.1 C157.9,116.9 156.7,120.9 152.7,124.9 L141.0,136.5 C139.8,137.7 141.6,141.9 141.8,141.8 Z" fill="currentColor" class="octo-body"></path></svg></a><style>.github-corner:hover .octo-arm{animation:octocat-wave 560ms ease-in-out}@keyframes octocat-wave{0%,100%{transform:rotate(0)}20%,60%{transform:rotate(-25deg)}40%,80%{transform:rotate(10deg)}}@media (max-width:500px){.github-corner:hover .octo-arm{animation:none}.github-corner .octo-arm{animation:octocat-wave 560ms ease-in-out}}</style>
  ```
#### 自定义文章底部版权
  
  > 打开主题配置文件搜索 post_copyright 并按如下对应项修改：
  
  ```
  post_copyright:
  enable: true
  license: CC BY-NC-ND 4.0
  license_url: https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh
  ```
  > 经过上面配置底部版权部分只能出现文章作者，文章链接和版权声明，我们可以修改next\layout\_macro\post-copyright.swig 文件，添加文章标题和文章发布日期
  
  ```
  <script src="https://cdn.bootcss.com/jquery/2.0.0/jquery.min.js"></script>
  <script src="//cdn.bootcss.com/clipboard.js/1.5.10/clipboard.min.js"></script>
  <script src="https://unpkg.com/sweetalert/dist/sweetalert.min.js"></script>
   <ul class="post-copyright">
   <li class="post-copyright-title">
    <strong>{{ __('post.copyright.title') + __('symbol.colon') }}</strong>
    {{ post.title | default(config.title) }}
   </li>
   <li class="post-copyright-author">
    <strong>{{ __('post.copyright.author') + __('symbol.colon') }}</strong>
    {{ post.author | default(config.author) }}
   </li>
   <li class="post-copyright-created_at">
    <strong>{{ __('post.copyright.created_at') + __('symbol.colon') }}</strong>
    {{ date(post.date, config.date_format) }}
   </li>
   <li class="post-copyright-link">
    <strong>{{ __('post.copyright.link') + __('symbol.colon') }}</strong>
    <a href="{{ post.url | default(post.permalink) }}" title="{{ post.title }}">{{ post.url | default(post.permalink) }}</a>
	<span class="copy-path"  title="点击复制文章链接"><i class="fa fa-clipboard" data-clipboard-text="{{ page.permalink }}"  aria-label="复制成功！"></i></span>
   </li>
   <li class="post-copyright-license">
    <strong>{{ __('post.copyright.license_title') + __('symbol.colon') }} </strong>
    {{ __('post.copyright.license_content', theme.post_copyright.license_url, theme.post_copyright.license) }}
    </li>
    </ul>
    <script> 
    var clipboard = new Clipboard('.fa-clipboard');
    $(".fa-clipboard").click(function(){
      clipboard.on('success', function(){
        swal({
          title: "",
          text: '复制文章链接成功',
          icon: "success",
          showConfirmButton: true
          });
	    });
    });  
  </script>
  ```
  > 经过如上配置，文章标题和发布日期都显示出来了，但是只能显示英文，中文配置文件没有对应的中文，打开 themes\next\languages\zh-Hans.yml 搜索 copyright： 自定义修改类别名称如下
  
  ```
  copyright:
    title: 本文标题
    created_at: 发布时间
    author: 本文作者
    link: 本文链接
    license_title: 版权声明
    license_content: '本博客所有文章除特别声明外，均采用
      <a href="%s" rel="external nofollow" target="_blank">%s</a> 许可协议。转载请注明出处！'
  ```
#### 更好的管理文章
  >根据[官方说明](https://hexo.io/zh-cn/docs/asset-folders.html)，编辑博客配置文件
  
  ```
  方式一：（不管是方式一还是方式二，都是必须的）：
  _config.yml
  post_asset_folder: true
  方式二（下载插件，在bolg文件目录下执行命令）：
  npm install https://github.com/CodeFalling/hexo-asset-image --save
  ```
  > hexo新建文章后的目录结构
  
  ```
   _posts
  ├── demo文章
  |   ├── demo1.jpg
  |   ├── demo2.jpg
  |   └── demo3.jpg
  └── demo文章.md
  ```
  > 正确的引用图片方式是使用下列的标签插件
  
  ```
  方式一：
  {% asset_img example.jpg This is an example image %}
  方式二：
  <img src="/demo文章/demo1.jpg" alt="demo文章">
  ```
#### 新建页面
   
  - 比如新建标签页
  
  ```
  # 新建页面 tags 
  hexo new page tags
  
  # \blog\source\tags目录下 index.md，编辑设置type类型为tags
  title: 标签
  date: 2018-2-22 23:39:04
  type: "tags"
  
  //编辑主题配置文件，tags 加上刚刚新建页面的相对目录
  menu:
  home: / || home
  tags: /tags/ || tags(图标代码 Font Awesome)
  categories: /categories/ || th
  archives: /archives/ || archive
  about: /about/ || user
  ```
#### 显示当前浏览进度
  >修主题配置文件config.yml，把 false 改为 true
  
  ```
  # Scroll percent label in b2t button.
  scrollpercent: true

  # Enable sidebar on narrow view (only for Muse | Mist).
  onmobile: true
  ```
#### 添加字数统计
- 下载插件（bolg目录下）

```
 npm install hexo-wordcount --save
```
- 在主题配置文件中找到如下配置，并修改配置如下

```
post_meta:
  item_text: true
  created_at: true #发布时间
  updated_at: false #更新时间
  categories: true #分类

post_wordcount:
  item_text: true
  wordcount: true #单篇字数统计
  min2read: true #单篇阅读时长
  totalcount: false #站点字数统计
  separated_meta: true
```
- 显示样式加上“字”和“分钟”，则打开 themes\next\layout\_macro\post.swig 文件分别搜索代码，并把“字”和“分钟”按如下添加到代码后面

```
<span title="{{ __('post.wordcount') }}">
  {{ wordcount(post.content) }} 字
</span>

<span title="{{ __('post.min2read') }}">
  {{ min2read(post.content) }} 分钟
</span>
```
#### 添加浏览统计
##### 文章浏览统计
- 注册[LeanCloud](https://leancloud.cn/)账号
- 创建一个项目，在项目的存储模块新建新建Class用来专门保存我们博客的文章访问量等数据，我们前面对NexT主题的修改兼容，此处的新建Class名字必须为Counter，由于LeanCloud升级了默认的ACL权限，如果你想避免后续因为权限的问题导致次数统计显示不正常，建议在此处选择无限制

![leancloud创建class](https://github.com/maoqitian/MaoMdPhoto/raw/master/hexo%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA/leancloud%E5%88%9B%E5%BB%BAclass.png)
- 获取我们前面新建项目的app_id和app_key，主题配置文件中设置对应信息
```
leancloud_visitors:
  enable: true
  app_id: #<app_id>
  app_key: #<app_key>
```
- Web安全
因为AppID以及AppKey是暴露在外的，因此如果一些别用用心之人知道了之后用于其它目的是得不偿失的，为了确保只用于我们自己的博客，建议开启Web安全选项，这样就只能通过我们自己的域名才有权访问后台的数据了，可以进一步提升安全性。选择应用的设置的安全中心选项卡，加入我们域名保存。

![leancloud设置安全域名](https://github.com/maoqitian/MaoMdPhoto/raw/master/hexo%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA/leancloud%E8%AE%BE%E7%BD%AE%E5%AE%89%E5%85%A8%E5%9F%9F%E5%90%8D.png)
##### 站点浏览统计 不蒜子
- 显示站点统计
```
busuanzi_count:
  # count values only if the other configs are false
  enable: true
  # custom uv span for the whole site
  site_uv: true
  #本站访客数
  site_uv_header: <i class="fa fa-user"></i> 本站访客数
  site_uv_footer: 人次
  # custom pv span for the whole site
  site_pv: true
  site_pv_header: <i class="fa fa-eye"></i> 本站总访问量
  site_pv_footer: 次
  # custom pv span for one page only
```
- 如果无法显示字数统计，原因为不蒜子域名过期的问题
- 解决：进入 hexo 博客项目的 themes 目录下，在 next 主题目录中的 layout/_third-party/analytics/ 下找到 busuanzi-counter.swig 文件
把：

```
<script async src="//dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js"></script>
```
改为：

```
<script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"></script>
```
#### 添加搜索功能（基于Next主题）
- 安装插件searchdb
```
npm install hexo-generator-searchdb --save
```
- hexo博客配置文件中添加如下配置
```
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```
- Next主题配置_config.yml文件中更改如下配置（enable设置为true）

```
local_search:
  enable: true
  # if auto, trigger search by changing input
  # if manual, trigger search by pressing enter key or search button
  trigger: auto
  # show top n results per article, show all results by setting to -1
  top_n_per_article: 1
```

#### 能够显示大图
- 方法一：文章属性中加入图片链接
```
---
title: demo title 
date: 2019-01-14 15:55:49
toc: false #是否显示文章目录
categories:  
- 持续集成（CI） #分类
tags: 
- Jenkins
- git
- GitLab
- CentOS
- Maven
- Tomcat
photos:
- "https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/Jenkins.jpg"
---
```
- 方法二：由于markdown是支持原生html的，所以我们可以在正文引用img来为我们的文章设置摘要配图。

```
//在“<!-- more -->”之前的内容都会展示到摘要中(同时与你主题文件中配置的摘要字数有关).
---
title: demo title 
date: 2019-01-14 15:55:49
toc: false #是否显示文章目录
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
<img src="https://github.com/maoqitian/MaoMdPhoto/raw/master/Jenkins/Jenkins.jpg" width=50% />
<!--more-->
```
#### 添加评论
- 添加[来必力](https://www.livere.com/)评论系统
- 首先注册来必力账号（163邮箱可以注册）
- 安装创建（实际就是创建一个网站评论项目，填写你的）

![来必力评论安装](https://github.com/maoqitian/MaoMdPhoto/raw/39c2ad856ab065e47c96d50f8e28cd151c142750/hexo%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA/%E6%9D%A5%E5%BF%85%E5%8A%9B%E8%AF%84%E8%AE%BA%E5%AE%89%E8%A3%85.png)
- 获取livere_uid也就是（也就是代码管理的data-uid）

![来必力uid获取](https://github.com/maoqitian/MaoMdPhoto/raw/39c2ad856ab065e47c96d50f8e28cd151c142750/hexo%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA/%E6%9D%A5%E5%BF%85%E5%8A%9Buid%E8%8E%B7%E5%8F%96.png)
- 最后修改主题配置文件字段，填入上一步获取的livere_uid
```
livere_uid: you uid 
```
#### 修改文章代码显示样式
- 修改hexo配置文件
```
highlight:
  enable: true
  line_number: true
  auto_detect: true
  tab_replace:
```
#### 5.22 博客文章分页
- hexo博客配置文件中
```
## 每页显示的文章量 (0 = 关闭分页功能)
per_page: 10
pagination_dir: page
```
#### 博客文章阴影效果
- 修改主题目录下themes/next/source/css/_custom/custom.styl文件添加代码

```
//文章添加阴影效果 1
.post{
    margin-top:60px;
    .post-block {
      padding: 35px 30px;
      background: #fff;
      box-shadow: 0 2px 2px 0 rgba(0,0,0,0.12), 0 3px 1px -2px rgba(0,0,0,0.06), 0 1px 5px 0 rgba(0,0,0,0.12);
      border-radius: initial;
    }
  }
//文章添加阴影效果 2
.post{
    margin-top:30px;
    .post-block {
    margin-bottom: 12px;
    background: #fff;
    background-image: initial;
    background-position-x: initial;
    background-position-y: initial;
    background-size: initial;
    background-repeat-x: initial;
    background-repeat-y: initial;
    background-attachment: initial;
    background-origin: initial;
    background-clip: initial;
    background-color: rgb(255, 255, 255);
    padding: 40px;
    padding-top: 40px;
    padding-right: 40px;
    padding-bottom: 40px;
    padding-left: 40px;
    box-shadow: 0 2px 2px 0 rgba(0,0,0,0.12), 0 3px 1px -2px rgba(0,0,0,0.06), 0 1px 5px 0 rgba(0,0,0,0.12);
    border-radius: 10px 10px 10px 10px;
    border-top-left-radius: 10px;
    border-top-right-radius: 10px;
    border-bottom-right-radius: 10px;
    border-bottom-left-radius: 10px;
    }
  }
```
#### 添加站点友情链接
- 主题配置文件中

```
# Blog rolls
links_icon: pagelines #模块icon
links_title: 友情链接 #模块标题
links_layout: block   #模块layout 样式
#links_layout: inline
links:
  玩Android: http://www.wanandroid.com/  # 链接名称：链接地址
```
#### 添加文章分享功能
- 将主题配置_config.yml文件中关于baidushare部分的内容改为（其中type亦可以选择button）

```
baidushare:
  type: slide
  baidushare: true
```
- 集成百度分享你会发现配置文件有一句话表明不支持https

```
# Warning: Baidu Share does not support https.
```
##### 不支持https解决方案
- 下载新的静态资源 [static](https://github.com/hrwhisper/baiduShare)
- 下载压缩包到本地，解压后，将static文件夹保存至themes\next\source目录下
- 修改文件themes\next\layout\_partials\share\baidushare.swig
```
文件末尾 讲静态资源路径改为我刚刚下载好的静态资源
.src='http://bdimg.share.baidu.com/static/api/js/share.js?v=89860593.js?cdnversion='+~(-new Date()/36e5)];

改为 
.src='/static/api/js/share.js?v=89860593.js?cdnversion='+~(-new Date()/36e5)];
```
#### 页面点击桃心效果
- 创建clicklove.js文件，并写入如下内容代码
```
!function(e,t,a){function n(){c(".heart{width: 10px;height: 10px;position: fixed;background: #f00;transform: rotate(45deg);-webkit-transform: rotate(45deg);-moz-transform: rotate(45deg);}.heart:after,.heart:before{content: '';width: inherit;height: inherit;background: inherit;border-radius: 50%;-webkit-border-radius: 50%;-moz-border-radius: 50%;position: fixed;}.heart:after{top: -5px;}.heart:before{left: -5px;}"),o(),r()}function r(){for(var e=0;e<d.length;e++)d[e].alpha<=0?(t.body.removeChild(d[e].el),d.splice(e,1)):(d[e].y--,d[e].scale+=.004,d[e].alpha-=.013,d[e].el.style.cssText="left:"+d[e].x+"px;top:"+d[e].y+"px;opacity:"+d[e].alpha+";transform:scale("+d[e].scale+","+d[e].scale+") rotate(45deg);background:"+d[e].color+";z-index:99999");requestAnimationFrame(r)}function o(){var t="function"==typeof e.onclick&&e.onclick;e.onclick=function(e){t&&t(),i(e)}}function i(e){var a=t.createElement("div");a.className="heart",d.push({el:a,x:e.clientX-5,y:e.clientY-5,scale:1,alpha:1,color:s()}),t.body.appendChild(a)}function c(e){var a=t.createElement("style");a.type="text/css";try{a.appendChild(t.createTextNode(e))}catch(t){a.styleSheet.cssText=e}t.getElementsByTagName("head")[0].appendChild(a)}function s(){return"rgb("+~~(255*Math.random())+","+~~(255*Math.random())+","+~~(255*Math.random())+")"}var d=[];e.requestAnimationFrame=function(){return e.requestAnimationFrame||e.webkitRequestAnimationFrame||e.mozRequestAnimationFrame||e.oRequestAnimationFrame||e.msRequestAnimationFrame||function(e){setTimeout(e,1e3/60)}}(),n()}(window,document);
```
- 在\themes\next\layout\_layout.swig文件末尾添加：
```
<!-- 页面点击小红心 -->
<script type="text/javascript" src="/js/src/clicklove.js"></script>
```
#### 添加网页在线联系功能（DaoVoice）
- 首先注册[DaoVoice](https://dashboard.daovoice.io/)，注册可以使用GitHub账号注册(这里我就是使用Github 关联注册)
- 注册完成之后还需要创建我们网站对应的DaoVoice项目，邀请码为 2e5d695d

![DaoVoice创建项目](https://github.com/maoqitian/MaoMdPhoto/raw/master/hexo%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA/DaoVoice%E5%88%9B%E5%BB%BA%E9%A1%B9%E7%9B%AE.png)
- 创建成功之后DaoVoice会提示我们快速接入，如下图需要找到对应的app_id，然后需要修改/themes/next/layout/_partials/head.swig文件，并加入如下代码，注意"//widget.daovoice.io/widget/xxxx.js"中的xxxx就对应图中的app_id

```
{% if theme.daovoice %}
  <script>
  (function(i,s,o,g,r,a,m){i["DaoVoiceObject"]=r;i[r]=i[r]||function(){(i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;a.charset="utf-8";m.parentNode.insertBefore(a,m)})(window,document,"script",('https:' == document.location.protocol ? 'https:' : 'http:') + "//widget.daovoice.io/widget/xxxx.js","daovoice")
  daovoice('init', {
      app_id: "{{theme.daovoice_app_id}}"
    });
  daovoice('update');
  </script>
{% endif %}
```
![快速接入DaoVoice](https://github.com/maoqitian/MaoMdPhoto/raw/master/hexo%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA/%E5%BF%AB%E9%80%9F%E6%8E%A5%E5%85%A5DaoVoice.png)
- 最后在主题配置文件中加入如下代码（注意冒号后面的空格），并填入对应的app_id

```
# Online contact
daovoice: true
daovoice_app_id:   # 这里填你刚才获得的 app_id
```
- 其他的一些设置可以自行在daovoice控制台中进行设置，这里就不展开了
#### 每篇文章末尾统一添加“本文结束”标记
##### 添加“本文结束”标记
- 在\themes\next\layout\_macro下新建 passage-end-tag.swig 文件,并添加以下代码(新建文件格式必须是utf-8)：
```
<div>
    {% if not is_index %}
        <div style="text-align:center;color: #ccc;font-size:14px;">-------------本文结束<i class="fa fa-heart"></i>感谢阅读-------------</div>
    {% endif %}
</div>
```
- 打开\themes\next\layout\_macro\post.swig文件，在post-body 之后， post-footer （post-footer之前有两个DIV）之前添加如下代码：

```
<div>
  {% if not is_index %}
    {% include 'passage-end-tag.swig' %}
  {% endif %}
</div>
```
![文末添加文本结束标记](https://github.com/maoqitian/MaoMdPhoto/raw/master/hexo%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA/%E6%96%87%E6%9C%AB%E6%B7%BB%E5%8A%A0%E6%96%87%E6%9C%AC%E7%BB%93%E6%9D%9F%E6%A0%87%E8%AE%B0.png)
- 在主题配置文件_config.yml在末尾添加以下字段：

```
# 文章末尾添加“本文结束”标记
passage_end_tag:
  enabled: true
```
##### 修改文章底部的的标签样式
>打开模板文件/themes/next/layout/_macro/post.swig，找到rel="tag">#字段，
将# 换成<i class="fa fa-tag"></i>,其中tag是你选择标签图标的名字,也是可以自定义的,
如下:

```
<a href="{{ url_for(tag.path) }}" rel="tag"><i class="fa fa-tag"></i> {{ tag.name }}</a>
```


### 重新发布博客
- 经过我们各种修改美化后的博客，需要同步到github中，具体步骤为
  
  ```
  # 在博客目录下
  hexo clean

  hexo g 
  
  hexo d
  ```

### 新建页面，发布文章
- 新建页面命令，在命令中指定文章的布局（layout），默认为 post，可以通过修改 _config.yml 中的 default_layout 参数来指定默认布局
```
hexo new [layout] <title>
```
- layout的类型

布局 | 	路径| 布局含义
---|---|---
post | source/_posts | 文章
page | 	source | 页面
draft| source/_drafts | 草稿

- 默认三种layout模板的路径 \blog\scaffolds

- 如果需要删除文章，则到source/_posts目录下删除对应文章重新发布博客即可
### 博客同步管理
- 在github中默认博客仓库中新建hexo分支并且将博客仓库的默认分支变成hexo分支(setting中设置)

![博客仓库的默认分支变成hexo](https://github.com/maoqitian/MaoMdPhoto/raw/master/hexo%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA/%E5%8D%9A%E5%AE%A2%E4%BB%93%E5%BA%93%E7%9A%84%E9%BB%98%E8%AE%A4%E5%88%86%E6%94%AF%E5%8F%98%E6%88%90hexo.png)

- 克隆刚刚的分支到本地电脑（存在blog源文件的目录）

```
git clone git@github.com:maoqitian/maoqitian.github.io.git
```
- 将blog源文件的目录下所有文件复制到刚刚克隆hexo分支的文件目录username.github.io层级，并下载一下插件
```
npm install hexo-generator-index --save
npm install hexo-generator-archive --save
npm install hexo-generator-category --save
npm install hexo-generator-tag --save
npm install hexo-server --save
npm install hexo-deployer-git --save
npm install hexo-deployer-heroku --save
npm install hexo-deployer-rsync --save
npm install hexo-deployer-openshift --save
npm install hexo-renderer-marked@0.2 --save
npm install hexo-renderer-stylus@0.2 --save
npm install hexo-generator-feed@1 --save
npm install hexo-generator-sitemap@1 --save
npm install hexo-generator-search --save
npm install hexo-generator-searchdb --save
```
- 提交时考虑以下注意事项
 
  - 将themes目录以内中的主题的.git目录删除（如果有），因为一个git仓库中不能包含另一个git仓库，否则提交主题文件夹会失败
  - 后期需要更新主题时在另一个地方git clone下来该主题的最新版本，然后将内容拷到当前主题目录即可
- 然后提交该 hexo 分支 

#### 其他电脑需要操作blog
- 首先你需要在电脑上配置相关环境
- 安装Node.js、git和hexo
- 新电脑上克隆username.github.io仓库的hexo分支到本地，此时本地git仓库处于hexo分支
- 切换到username.github.io目录，执行npm install(由于仓库有一个.gitignore文件，里面默认是忽略掉 node_modules文件夹的，也就是说仓库的hexo分支并没有存储该目录，所以需要install下）
- 如果node_modules文件没有丢失, 可不执行该操作
- 到这里了就可以开始在自己的电脑上写博客了，新建文章，更新文章等
- 需要注意的是每次更新博客之后, 都要把相关修改上传到hexo分支，每次换电脑更新博客的时候, 在修改之前最好也要git pull拉取一下最新的更新


### 参考链接

  - [Hexo官方文档](https://hexo.io/zh-cn/docs/) 
  - [最全Hexo博客搭建+主题优化+插件配置+常用操作+错误分析](https://www.simon96.online/2018/10/12/hexo-tutorial/)
  - [HEXO搭配Next主题修改博客](http://blog.codesfile.com/2017/12/16/HEXO%E6%90%AD%E9%85%8DNext%E4%B8%BB%E9%A2%98%E4%BF%AE%E6%94%B9%E5%8D%9A%E5%AE%A2/)
  - [百度分享集成](https://blog.csdn.net/cl534854121/article/details/76121105?locationNum=6&fps=1)
  - [Hexo NexT主题添加点击爱心效果](https://asdfv1929.github.io/2018/01/26/click-love/)
  - [Hexo NexT主题内接入网页在线联系功能](https://asdfv1929.github.io/2018/01/21/daovoice/)