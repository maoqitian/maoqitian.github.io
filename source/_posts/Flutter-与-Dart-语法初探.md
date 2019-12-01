---
title: Flutter 与 Dart 语法初探
date: 2019-12-01 12:24:15
categories:  
- Flutter探索 #分类
tags: 
- dart
- Flutter
---
![image](https://raw.githubusercontent.com/maoqitian/MaoMdPhoto/master/flutter/flutter-logo.png)

# 什么是flutter

- google 推出的跨平台UI框架

# 环境搭建(MAC环境)
- Flutter 依赖下面这些命令行工具
```
bash, mkdir, rm, git, curl, unzip, which
```
<!--more-->
## 获取Flutter SDK
- flutter官网下载其最新可用的[安装包](https://flutter.dev/docs/development/tools/sdk/releases?tab=macos#macos)

- 安装包下载完成则可以进行解压

```
unzip /指定解压目录/flutter_macos_v1.9.1+hotfix.4-stable.zip
```
- 解压完成之后我们会在解压好的目录下会多出一个flutter目录，获取并记住该目录路径，下一步我们会用到

```
# 使用pwd 命令查看目录路径
/Users/XXXXX/development/flutter
```


## 设置环境变量

- 设置环境变量目的是以便我们可以运行flutter命令在任何终端会话中

- 确定Flutter SDK的目录，上一步我们解压获取了flutter的路径/Users/XXXXX/development/flutter

- 打开(或创建) $HOME/.bash_profile. 文件路径和文件名可能在您的机器上不同（注意$HOME 指的是 路径是 /Users/用户名XX/ ）

```
vim $HOME/.bash_profile
```
- 文件加入环境变量

```
export PUB_HOSTED_URL=https://pub.flutter-io.cn //国内用户需要设置
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn //国内用户需要设置
//
export PATH=/Users/XXXXX/development/flutter/bin:$PATH
```
- 最后我们执行创建好的.bash_profile文件

```
source $HOME/.bash_profile
注意: 如果你使用的是zsh，终端启动时 ~/.bash_profile 将不会被加载，解决办法就是修改 ~/.zshrc ，在其中添加：source ~/.bash_profile
```
- [flutter dev 网站](https://pub.dev/)
## 执行 flutter docter 检查本机软件环境，没安装的插件根据提示安装就行了

```
flutter doctor
```
<img src="https://raw.githubusercontent.com/maoqitian/MaoMdPhoto/master/flutter/%E8%BF%90%E8%A1%8Cflutter-doctor%E5%91%BD%E4%BB%A4.png"  height="200" width="550">


## 手动升级 Flutter
- flutter 版本升级迭代很快，前面我们下载SDK默认是stable分支，也就是稳定版，如何手动升级呢？只需要下面一条flutter命令

```
flutter upgrade
```

# Dart 语法

![logo-dart](https://raw.githubusercontent.com/maoqitian/MaoMdPhoto/master/flutter/dart/logo-dart.png)

- 我们知道Flutter框架是使用Dart 语言来编写的，Dart 是一个面向对象编程语言， 每个对象都是一个类的实例，所有的类都继承于 Object.如果熟悉Java，语言是很容易上手的。首先来熟悉一下Dart语法

## Dart 变量
- var 声明变量，和 kt 、js语法很像，需要注意的是如下示例 name 只要复制字符串，则他就是String类型，number就是int 类型，不能再更改它的类型，而没有初始化的变量自动获取一个默认值为 null。

```
var name = 'maoqitian';
var number = 1;
```

## Dart 常量

- final 和 const声明都是表示常量，一个 final 变量只能赋值一次，可以省略变量类型，如下声明一个存放WordPair值的List 数组

```
final List _suggestions = new List<WordPair>();

final _suggestions = <WordPair>[];

```
- const 关键字不仅仅只用来定义常量， 也可以用来创建不变的值

```
//如下定义一个字体大小的值一直都是 18 ，不会改变
final _biggerFont = const TextStyle(fontSize: 18.0)
```
### final 和 const区别
-  const 的值在编译期确定，final 的值要到运行时才确定

## Dart 函数方法
- Dart 是一个真正的面向对象语言，方法也是对象他的类型是 Function。 这意味着，方法可以赋值给变量，也可以当做其他方法的参数。

```
//定义一个返回 bool(布尔)类型的方法 
bool isNoble(int atomicNumber) {
  return _nobleGases[atomicNumber] != null;
}
//转换如下可以忽略类型定义
isNoble(atomicNumber) {
  return _nobleGases[atomicNumber] != null;
}

//只有一个表达式的方法，你可以选择 使用缩写语法来定义
// => expr 语法是 { return expr; } 形式的缩写
bool isNoble(int atomicNumber) => _nobleGases[atomicNumber] != null;

```
### Dart 方法参数
- 方法可以定义两种类型的参数：必需的和可选的。 必需的参数在参数列表前面， 后面是可选参数，必选参数没啥好说的，我们来了解可选参数。可选参数可以是自己命名参数或者基于可选位置的参数，但是这两种参数不能同时当做可选参数来一起用。

#### 可选命名参数

- 调用可选命名参数方法的时候可以使用 paramName: value （key:value形式，只不是过key 是参数名称）来指定参数值

```
//调用有可选命名参数方法 playGames
playGames(bold: true, hidden: false);

//playGames 方法
playGames({bool bold, bool hidden}) {
  // ...
}
```
#### 可选位置参数

- 方法参数列表中用[]修饰的参数就是可选位置参数
```
// 定义可选位置参数方法
String playGames (String from, String msg, [String sports]) {
  var result = '$from suggest $msg';
  if (sports != null) {
    result = '$result playing $sports together';
  }
  return result;
}
 
// 不用可选参数 
playGames('Bob', 'Howdy'); // 返回值 Bob suggest Howdy

//使用可选参数
playGames('I', 'Xiao Ming', 'basketball'); //返回值 I suggest Xiao Ming playing basketball together.
```
### Dart 方法参数默认值

- 在定义方法的时候，可以使用 = 来定义可选参数的默认值。 默认值只能是编译时常量。 如果没有提供默认值，则默认值为 null

```
// 定义可选位置参数方法
String playGames (String from , String msg, [String sports = 'football']) {
  var result = '$from suggest $msg';
  if (sports != null) {
    result = '$result playing $sports together';
  }
  return result;
}

playGames('I', 'Xiao Ming'); //返回值 I suggest Xiao Ming playing football together.

```
### 入口函数（The main() function）

- 每个应用都需要有个顶级的 main() 入口方法才能执行

```
// Android studio 创建Demo 项目  main.dart 文件开头 
void main() => runApp(MyApp());

//可以转换为
void main(){
    runApp(MyApp());
}
```

### 异步操作
- async 方法和 await 异步操作，直接看看一个网络请求例子就能够了解

```
static Future<ArticleListData> getArticleData(int pageNum) async{
    String path = '/article/list/$pageNum/json';
    Response response = await HttpUtils.get(Api.BASE_URL+path);
    ArticleBaseData articleBaseData = ArticleBaseData.fromJson(response.data);
    return articleBaseData.data;
  }
```


> 先了解这么多，更多Dart 相关内容可以查看[Dart语言官网](http://dart.goodev.org/)


# Flutter 
## 官方文档

- [flutter.dev](https://flutter.dev/)
- [Flutter中文网](https://flutter.dev/)

## Flutter Hello world
- 在开始Flutter Hello world程序之前，作为一名Android 开发者，首先我们要认识到Flutter中没有原生开发的XML，所有界面和逻辑代码都在.dart文件中，Flutter给我提供了一套视觉、结构、平台、和交互式的Widgets，所以在Flutter中一构架的一切界面都是Widgets。接下来我们先看一个简单的Hello World Flutter应用。

- Android Studio 新建Flutter demo

```
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget{
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return new MaterialApp(
      theme: ThemeData(
        primaryColor: Colors.blueAccent,
      ),
      home: new Scaffold(
        appBar: new AppBar(
            title: new Center(
              child: new Text("Welcome to Flutter"),
            )
        ),
        body: DemoStatelessWidget("Flutter Hello World ! 无状态的Widget"), 
      ),
    );
  }

}

//无状态 Widget
class DemoStatelessWidget extends StatelessWidget{

  final String text;

  //构造方法传入 text 值
  DemoStatelessWidget(this.text);
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return Container(
      constraints: BoxConstraints.expand(
        height: Theme.of(context).textTheme.display1.fontSize * 1.1 + 200.0,
      ),
      padding: const EdgeInsets.all(8.0),
      color: Colors.blue[600],
      alignment: Alignment.center,
      child: Text(text,
          style: Theme.of(context)
              .textTheme
              .display1
              .copyWith(color: Colors.white)),
      transform: Matrix4.rotationZ(0.1),
    );
  }
}
```
- 运行结果

<img src="https://github.com/maoqitian/MaoMdPhoto/raw/master/flutter/Flutter%20Hello%20world.jpg"  height="300" width="170">

- 如上代码所示，首先main函数运行MyApp，MyApp 是一个DemoStatelessWidget，字面可以理解为一个没有状态的Widget，他的build 方法创建了MaterialApp这个Widget才使得应用可以跑在手机上，接着创建了Scaffold 这个Widget，可以让我们创建AppBar和页面内容，body 的页面内容又包含了一个无状态的DemoStatelessWidget，通过构造方法可以传入我们想要现实的页面内容，该widget包含白色背景和一个现实文字的Text widget。

## Flutter ListView

- 上个小例子中我们提到无状态 StatelessWidget，想必也能猜到，肯定会有一个有状态的widget，这个widget就是StatefulWidget，该widget为何说是有状态的呢，主要是在其管理的State中我们可以调用setState来动态改变页面显示。接着我们看一个显示数据列表的例子，并加入一个可以点击收藏的按钮。
```
import 'package:flutter/material.dart';
import 'package:english_words/english_words.dart';

void main() => runApp(MyApp());

// StatelessWidget 无状态的widget
class MyApp extends StatelessWidget { //Stateless widgets是不可变的, 这意味着它们的属性不能改变 - 所有的值都是最终的.
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      //title: 'Welcome to Flutter',
      theme: ThemeData(
        primaryColor: Colors.blueAccent,
      ),
     home: new RandomWords(),
    );
  }
}

//StatefulWidget 有状态的widget
class RandomWords extends StatefulWidget{
  @override
  createState() => new RandomWordsState();
}

// 返回 显示单词对的ListView Widget
class RandomWordsState extends State<RandomWords> {
  //保存建议的单词对列表(变量以下划线（_）开头，在Dart语言中使用下划线前缀标识符，会强制其变成私有的)  final _suggestions = <WordPair>[];
  final List _suggestions = new List<WordPair>();
  //设置字体大小的变量
  final _biggerFont = const TextStyle(fontSize: 18.0);
  // 保存喜欢单词组的集合 set 集合不允许值重复
  final Set _saved = new Set<WordPair>();

  /// State 生命周期方法
  @override
  void initState() {
    // state 初始化
    super.initState();
  }

  @override
  void didChangeDependencies() {
    // 在 initState 之后调用，此时可以获取其他 State
    super.didChangeDependencies();
  }

  @override
  void dispose() {
    // state 销毁
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    //return new Text(new WordPair.random().asPascalCase);
    //返回单词对的ListView。
    return new Scaffold(
      appBar: new AppBar(
        title:new Center( //居中显示
          child: new Text('Flutter ListView'),
        ),
      ),
      body: _buildSuggestions(),
    );
  }
  
  //构建显示建议单词对的ListView。
  Widget _buildSuggestions(){
    return new ListView.builder(
        padding: const EdgeInsets.all(16.0),
        // 对于每个建议的单词对都会调用一次itemBuilder，然后将单词对添加到ListTile行中
        // 在偶数行，该函数会为单词对添加一个ListTile row.
        // 在奇数行，该函数会添加一个分割线widget，来分隔相邻的词对。

        //itemBuilder 值是一个匿名回调函数， 接受两个参数- BuildContext和行迭代器i。迭代器从0开始，
        // 每调用一次该函数，i就会自增1，对于每个建议的单词对都会执行一次。该模型允许建议的单词对列表在用户滚动时无限增长。
          itemBuilder: (context,i){
          // 在每一列之前，添加一个1像素高的分隔线widget
          if(i.isOdd) return new Divider();
          // 语法 "i ~/ 2" 表示i除以2，但返回值是整形（向下取整），比如i为：1, 2, 3, 4, 5
          // 时，结果为0, 1, 1, 2, 2， 这可以计算出ListView中减去分隔线后的实际单词对数量
          final index = i~/2;
          
          if(index >= _suggestions.length){
            //  如果是建议单词列表中最后一个单词对 接着再生成10个单词对，然后添加到建议列表
            _suggestions.addAll(generateWordPairs().take(10));
          }

          return _buildRow(_suggestions[index]);
        }
    );
  }
  //创建 ListTile中显示每个新词对
  Widget _buildRow(WordPair suggestion) {
    //获取是否保存了该单词状态
    final isSaved = _saved.contains(suggestion);

    return new ListTile(
      // 设置 标题
      title: new Text(
        suggestion.asPascalCase,
        style: _biggerFont,
      ),
      //图标
      trailing: new Icon(
        //星型图标状态
        isSaved ? Icons.favorite : Icons.favorite_border,
        color: isSaved ? Colors.deepOrange : null ,
      ),
      onTap: (){  // 当用户点击 ListTile 击时， ListTile 会调用它的onTap回调
        setState(() { //调用setState() 会为State对象触发build()方法，从而导致对UI的更新
          if(isSaved){
             _saved.remove(suggestion);
          }else{
            _saved.add(suggestion);
          }
        });
      },
    );
  }
}
```
- 运行效果

<img src="https://github.com/maoqitian/MaoMdPhoto/raw/master/flutter/Flutter%20ListView.jpg"  height="300" width="170">

- 如上代码，将原来的无状态Widget改成了StatefulWidget，并在build中构建ListView，到此你可能有疑惑，不是说有状态的Widget，怎么还是创建Widget，有状态如何体现呢? 别急，我们看到_buildRow方法，方法中构建了ListTile 这个Widget，它响应点击事件回到为onTap方法，也就是当我们点击ListTile，我们在onTap方法中就可以调用setState方法来动态改变页面显示，也就是改变桃心收藏按钮变化（注意setState方法需要在State类中才能调用）。

- State 是有周期的，其中包括三个函数：
1. initState()：初始化方法
2. didChangeDependencies()：在 initState 之后调用，此时可以获取其他 State
3. dispose()：state 销毁

- 如果你使用Android Studio 我们可以使用快捷键快速创建StatelessWidget和StatefulWidget，创建StatelessWidget快捷键为**stl**，创建StatefulWidget快捷键为**stf**。

### 小节总结

- Flutter 页面都是由一个个 widget 搭建而来的
- widget类型有两种，一种是无状态页面内容固定的StatelessWidget，一种是页面内容可以动态改变的 StatefulWidget

## Widget 速览
- 在开始开发实战之前，我们需要对基本常用的Widget有个大概的认识。

### layout Widget

- 在刚开始写原生界面的时候，我们最先了解的也应该是布局，Flutter 也提供了不少[布局widget](https://flutterchina.club/widgets/layout/)，接下来列举一些常用的布局widget


布局名称 | 特点描述
---|---
Container | 拥有单个子元素的布局widget，可以灵活设置
Padding | 拥有单个子元素，给其子widget添加指定的填充
Center | 将其子widget居中显示
Align | 将其子widget对齐，并可以根据子widget的大小自动调整大小。
Row | 可以拥有多个子元素，在水平方向上排列子widget的列表，和原生控件 LinerLayout orientation="horizontal" 类似 
Column | 可以拥有多个子元素，在竖直方向上排列子widget的列表，和原生控件 LinerLayout orientation="vertical" 类似 
Stack | 可以拥有多个子元素，允许其子widget简单的堆叠在一起
Flow | 实现流式布局算法的widget
ListView | 可滚动的列表控件

- 了解更多的widget请参考 [Flutter widget 目录](https://flutterchina.club/widgets/).

### 界面 Widget

- 有了布局，还需要在布局中填放各种控件，才最终组成我们的页面，比如我们开发Material Design 风格的App，Flutter 就给我们提供了Material Components Widgets，接下来选取一些常用的控件来了解。了解更多的widget请参考 [Material Components Widgets 目录](https://flutterchina.club/widgets/material/)。

Widget名称 | 特点描述
---|---
MaterialApp | 封装了应用程序实现Material Design所需要的一些widget，由前面demo可以发现它一般为应用顶层入口widget
Scaffold | Material Design布局结构的基本实现。此类提供了用于显示drawer、snackbar和底部sheet的API。
Appbar | 一般和Scaffold结合使用，可以设置页面标题和各种按钮等(Toolbar)
BottomNavigationBar| 底部导航条，可以很容易地在tap之间切换和浏览顶级视图
Drawer | 和Scaffold结合使用，从Scaffold边缘水平滑动以显示应用程序中导航链接的Material Design面板
RaisedButton | Material Design中的button，响应点击事件(button)
IconButton | 一个Material图标按钮，可以设置icon，点击时会有水波动画
TextField | 文本输入框 （EditText）
image | 显示图片的widget(ImageView)
Text | 单一格式的文本 (TextView)


## 导航栏返回按钮监听

- WillPopScope ，Flutter中可以通过WillPopScope来实现返回按钮拦截
- 以下示例提供两种效果，双击返回Toast提示（Toast库 fluttertoast: ^3.1.3），或者弹出提示dialog是否退出。
```
import 'package:flutter/material.dart';

class AppPage extends StatefulWidget {
  @override
  _AppPageState createState() => _AppPageState();
}

class _AppPageState extends State<AppPage> {
    @override
  Widget build(BuildContext context) {
    return WillPopScope( ///通过WillPopScope 嵌套，可以用于监听处理 Android 返回键的逻辑。 WillPopScope 并不是监听返回按键，只是当前页面将要被pop时触发的回调
        child: Container(),
        onWillPop: () async{
           return _doubleExitApp();
        }
    );
  }
  
  //双击返回 退出应用
  bool _doubleExitApp(){
    if (_lastPressedAt == null ||
        DateTime.now().difference(_lastPressedAt) > Duration(seconds: 1)) {
      ToolUtils.ShowToast(msg: "再点一次退出应用");
      //两次点击间隔超过1秒则重新计时
      _lastPressedAt = DateTime.now();
      return false;
    }
    //应用关闭直接取消 Toast
    Fluttertoast.cancel();
    return true;
}


  ///如果返回 return new Future.value(false); popped 就不会被处理
  ///如果返回 return new Future.value(true); popped 就会触发
  ///这里可以通过 showDialog 弹出确定框，在返回时通过 Navigator.of(context).pop(true);决定是否退出
  /// 单击提示退出
  Future<bool> _dialogExitApp(BuildContext context) {
    return showDialog(
        context: context,
        builder: (context) => new AlertDialog(
          content: new Text("是否退出"),
          actions: <Widget>[
            new FlatButton(onPressed: () => Navigator.of(context).pop(false), child:  new Text("取消")),
            new FlatButton(
                onPressed: () {
                  Navigator.of(context).pop(true);
                },
                child: new Text("确定"))
          ],
        ));
  }
  
}

```


# 最后

- 万事开头难，对于Flutter到此可以说是迈出了第一步，对于未尝试的东西，开始可能会有畏惧心理，但只要敢于尝试，敢于迈出第一步，相信难不难只在于自己的用心程度而言。本篇文章就先到这里，文章中如果有错误，请大家给我提出来，大家一起学习进步，如果觉得我的文章给予你帮助，也请给我一个喜欢和关注，同时也欢迎访问我的[**个人博客**](https://www.maoqitian.com)。

## 参考
- [Flutter 官方网站](https://flutter.dev/)
- [Flutter中文网](https://flutterchina.club/)
- [Dart 官方网站](https://dart.dev/)
- [Dart 语言中文网](http://dart.goodev.org/)

# About me
### blog：
- [个人博客](https://www.maoqitian.com/)
- [掘金](https://juejin.im/user/59e956626fb9a045204b57d4)
- [简书](https://www.jianshu.com/u/f58cd7ff1a08)
- [Github](https://github.com/maoqitian)
### mail：
- [maoqitian@gmail.com]()
- [maoqitian068@163.com]() 

