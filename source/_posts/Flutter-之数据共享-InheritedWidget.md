---
title: Flutter 之数据共享 InheritedWidget
date: 2019-12-01 13:19:04
categories:  
- Flutter探索 #分类
tags: 
- dart
- Flutter
top: true
---
![image](https://raw.githubusercontent.com/maoqitian/MaoMdPhoto/master/flutter/flutter-logo.png)
> Flutter 中Widget 多种多样，有UI的，当然也有功能型的组件InheritedWidget 组件就是Flutter 中的一个功能组件，它可以实现Flutter 组件之间的数据共享，他的数据传递方向在Widget树传递是从上到下的。
<!--more-->
# InheritedWidget 实现组件数据共享

- 既然要使用InheritedWidget，首先写一个Widget继承InheritedWidget

## 实现ShareDataWidget
```
/// Created with Android Studio.
/// User: maoqitian
/// Date: 2019/11/15 0015
/// email: maoqitian068@163.com
/// des:  InheritedWidget是Flutter中非常重要的一个功能型组件，它提供了一种数据在widget树中从上到下传递、共享的方式
import 'package:flutter/material.dart';


class ShareDataWidget extends InheritedWidget  {


  final int data; //需要在子树中共享的数据，保存点击次数

  ShareDataWidget( {@required this.data,Widget child})
      :super(child:child);


  // 子树中的widget通过该方法获取ShareDataWidget，从而获取共享数据
  static ShareDataWidget of(BuildContext context){
    return context.inheritFromWidgetOfExactType(ShareDataWidget);
  }


  //继承 InheritedWidget 实现的方法 返回值 决定当data发生变化时，是否通知子树中依赖data的Widget 更新数据
  @override
  bool updateShouldNotify(ShareDataWidget oldWidget) {
    //如果返回true，则子树中依赖(build函数中有调用)本widget的子widget的`state.didChangeDependencies`会被调用
    return oldWidget.data != data;
  }
}
```
- 由以上实现我们可以看到updateShouldNotify 返回值 决定当data发生变化时，是否通知子树中依赖data的Widget 更新数据，并且实现了of 方法方便子widget获取共享数据。

## 测试ShareDataWidget数据共享

- 前面我们已经实现了InheritedWidget，现在我们来看看如何使用随便写一个widget，让其显示ShareDataWidget的data 数据

```
 /// Created with Android Studio.
/// User: maoqitian
/// Date: 2019/11/15 0015
/// email: maoqitian068@163.com
/// des:  测试 ShareDataWidget
import 'package:flutter/material.dart';
import 'package:flutter_hellow_world/InheritedWidget/ShareDataWidget.dart';

class TestShareDataWidget extends StatefulWidget {
  @override
  _TestShareDataWidgetState createState() => _TestShareDataWidgetState();
}

class _TestShareDataWidgetState extends State<TestShareDataWidget> {


  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    //上层 widget中的InheritedWidget改变(updateShouldNotify返回true)时会被调用。
    //如果build中没有依赖InheritedWidget，则此回调不会被调用。
    print("didChangeDependencies");
  }

  @override
  Widget build(BuildContext context) {
    //显示 ShareDataWidget 数据变化，如果build中没有依赖InheritedWidget，则此回调不会被调用。
    return Text(ShareDataWidget.of(context).data.toString());

  }
}
```
- 接着新建widget 来使用ShareDataWidget，创建一个按钮，每点击一次，就将ShareDataWidget的值自增

```
/// Created with Android Studio.
/// User: maoqitian
/// Date: 2019/11/15 0015
/// email: maoqitian068@163.com
/// des:  创建一个按钮，每点击一次，就将ShareDataWidget的值自增
import 'package:flutter/material.dart';
import 'package:flutter_hellow_world/InheritedWidget/ShareDataWidget.dart';
import 'package:flutter_hellow_world/InheritedWidget/TestShareDataWidget.dart';

class InheritedWidgetTest extends StatefulWidget {
  @override
  _InheritedWidgetTestState createState() => _InheritedWidgetTestState();
}

class _InheritedWidgetTestState extends State<InheritedWidgetTest> {

  int count = 0;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: ShareDataWidget(
        data: count, //共享数据 data
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Padding(
              padding: const EdgeInsets.only(bottom: 20.0),
              child: TestShareDataWidget()//子widget中依赖ShareDataWidget
            ),
            RaisedButton(
              child: Text("计数增加"),
              onPressed: (){ 
                setState(() {
                  ++ count;
                });
              },
            )
          ],
        ),
      ),
    );
  }
}
```
- 代码很简单，创建一个按钮，每点击一次，就将ShareDataWidget的data值加一，而前面创建的TestShareDataWidget中依赖了ShareDataWidget的data值，如果数据共享则它的值就会跟随变化。

- 运行效果

<img src="https://raw.githubusercontent.com/maoqitian/MaoMdPhoto/master/flutter/InheritedWidget/demo.gif"  height="400" width="230">

## didChangeDependencies调用

- 运行上面的例子我们看到日志中会打印出如下日志，这就说明改变ShareDataWidget的data值时TestShareDataWidget的didChangeDependencies方法被调用了，该方法我们在写StatefulWidget时很少用到，我们可以在该方法中做一些耗时操作，比如数据持久化、网络请求等。

```
I/flutter ( 7082): didChangeDependencies
```
- 如果不想调用让didChangeDependencies被调用，也是有办法的，如下改变ShareDataWidget的of方法

```
 // 子树中的widget获取共享数据 方法
  static ShareDataWidget of(BuildContext context){
    //return context.inheritFromWidgetOfExactType(ShareDataWidget);
    //使用 ancestorInheritedElementForWidgetOfExactType 方法当数据变化则不会调用 子widget 的didChangeDependencies 方法 
    return context.ancestorInheritedElementForWidgetOfExactType(ShareDataWidget).widget;
  }
```
- 这里可以看到改变使用context.ancestorInheritedElementForWidgetOfExactType方法，而为什么使用这个方法didChangeDependencies就不会被调用呢？看源码就是最好的解释，我们直接翻到**framework.dart**中这两个方法的源码

```
/**
 * framework.dart  inheritFromWidgetOfExactType和ancestorInheritedElementForWidgetOfExactType方法源码
 */
 @override
  InheritedWidget inheritFromWidgetOfExactType(Type targetType, { Object aspect }) {
    assert(_debugCheckStateIsActiveForAncestorLookup());
    final InheritedElement ancestor = _inheritedWidgets == null ? null : _inheritedWidgets[targetType];
    if (ancestor != null) {
      assert(ancestor is InheritedElement);
      return inheritFromElement(ancestor, aspect: aspect);
    }
    _hadUnsatisfiedDependencies = true;
    return null;
  }


  @override
  InheritedElement ancestorInheritedElementForWidgetOfExactType(Type targetType) {
    assert(_debugCheckStateIsActiveForAncestorLookup());
    final InheritedElement ancestor = _inheritedWidgets == null ? null : _inheritedWidgets[targetType];
    return ancestor;
  }
```
- 显然，一对比我们就可以看到inheritFromWidgetOfExactType多调用了inheritFromElement方法，继续看该方法源码

```
/**
 * framework.dart  inheritFromElement方法源码
 */
 
@override
  InheritedWidget inheritFromElement(InheritedElement ancestor, { Object aspect }) {
    assert(ancestor != null);
    _dependencies ??= HashSet<InheritedElement>();
    _dependencies.add(ancestor);
    ancestor.updateDependencies(this, aspect);
    return ancestor.widget;
  }
```
- 到这里，一切都变得很清晰， inheritFromWidgetOfExactType方法中调用了inheritFromElement方法，而在该方法中InheritedWidget将其子widget添加了依赖关系，所以InheritedWidget发生改变，依赖它的子widget就会更新，也就会调用刚刚所说的didChangeDependencies方法，而ancestorInheritedElementForWidgetOfExactType方法没有和子widget注册依赖关系，当然也不会调用didChangeDependencies方法。

## 小结
- 以上通过一个使用InheritedWidget的简单例子，实现了InheritedWidget的使用，了解了didChangeDependencies调用，可以说对InheritedWidget这个组件有了一定了解，接下来通过对InheritedWidget封装，实现一个简易的Provider实现跨组件数据共享。

# 实现跨组件数据共享组件

- 作为一个原生Android 开发者，跨组件数据共享对于我们来说并不陌生，比如Android 开发中的Eventbus 就可以实现对事件订阅者的状态更新，Flutter中也有Eventbus的实现，但是这里直接使用Flutter 提供给我们的组件InheritedWidget来实现跨组件数据共享，Flutter中比较有名的[Provider](https://pub.dev/packages/provider)核心也是通过InheritedWidget来实现的，接着我们来实现一个自己的简易Provider。

## 实现通用InheritedWidget

- 要共享的数据多种多样，使用泛型来声明需要共享的数据

```
/// Created with Android Studio.
/// User: maoqitian
/// Date: 2019-11-17
/// email: maoqitian068@163.com
/// des:  实现InheritedWidget  保存需要共享的数据InheritedWidget

import 'package:flutter/material.dart';


class InheritedProvider<T> extends InheritedWidget{

  //共享数据  外部传入
  final T data;

  InheritedProvider({@required this.data, Widget child}):super(child:child);

  @override
  bool updateShouldNotify(InheritedProvider<T> oldWidget) {
    ///返回true，则每次更新都会调用依赖其的子孙节点的`didChangeDependencies`方法。
    return true;
  }

}
```
## InheritedWidget 封装
- 通过上面的实现，可以看到InheritedProvider中并没有方让调用者可以获取InheritedWidget组件，别着急，这里需要先明确两点；**首先，数据更新通知使用ChangeNotifier（FlultterSDK提供的一个Flutter风格的发布者-订阅者模式类）来进行通知，其次，接收到通知之后则由订阅者本身更新来重新构建InheritedProvider。**

```
/// Created with Android Studio.
/// User: maoqitian
/// Date: 2019-11-17
/// email: maoqitian068@163.com
/// des:  订阅者

import 'package:flutter/material.dart';
import 'package:flutter_theme_change/provider/InheritedProvider.dart';

// 该方法用于在Dart中获取模板类型
Type _typeOf<T>(){
  return T;
}
class ChangeNotifierProvider<T extends ChangeNotifier> extends StatefulWidget{


  final Widget child;
  final T data;

  ChangeNotifierProvider({Key key,this.child,this.data});


  //方便子树中的widget获取共享数据
  static T of<T> (BuildContext context,{bool listen = true}){ //listen 是否注册依赖关系 默认注册
    final type = _typeOf<InheritedProvider<T>>();
    final provider = listen ? context.inheritFromWidgetOfExactType(type) as InheritedProvider<T> :
    context.ancestorInheritedElementForWidgetOfExactType(type)?.widget as InheritedProvider<T>;
    return provider.data;

  }


  @override
  State<StatefulWidget> createState() {
    return _ChangeNotifierProviderState<T>();
  }
}

class _ChangeNotifierProviderState<T extends ChangeNotifier> extends State<ChangeNotifierProvider<T>>{

  @override
  Widget build(BuildContext context) {
  //构建 InheritedProvider
    return InheritedProvider<T>(
      data: widget.data,
      child: widget.child,
    );
  }
}
```
- 由上代码，创建了一个StatefulWidget，最终build构建的还是InheritedProvider，这时创建了返回对应data 数据的of方法，并且可以通过设置让子控件是否与InheritedWidget绑定（上一小节已经分析过），这样改变数据的控件就可以灵活的不与InheritedWidget绑定，也不用每次都更新改变数据的控件widget。
- 接着我们完善 _ChangeNotifierProviderState，当外部控件更新数据，并通过ChangeNotifier通知更新，ChangeNotifierProvider能够更新自身，让新数据生效，如何更新，那就是是使用setState方法，这也是创建StatefulWidget的目的。
```
class _ChangeNotifierProviderState<T extends ChangeNotifier> extends State<ChangeNotifierProvider<T>>{

  @override
  void initState() {
    // 给model添加监听器
    widget.data.addListener(update);
    super.initState();
  }


  @override
  void didUpdateWidget(ChangeNotifierProvider<T> oldWidget) {
    //当Provider更新时，如果新旧数据不"=="，则解绑旧数据监听，同时添加新数据监听
    if(widget.data != oldWidget.data){
       oldWidget.data.removeListener(update);
       widget.data.addListener(update);
    }
    super.didUpdateWidget(oldWidget);
  }

  // build方法 省略
  ........

  @override
  void dispose() {
    // 移除model监听器
    widget.data.removeListener(update);
    super.dispose();
  }

  void update() {
    //如果数据发生变化（model类调用了notifyListeners），重新构建InheritedProvider
    setState(() => {

    });
  }
```
## 数据消费者封装（Consumer）
- 数据有更新，有消息发出，还得有人消费，这样订阅者-消费者模式才完整，消费数说白了就是调用ChangeNotifierProvider的of方法来获取新数据，上一步我们已经触发订阅者的更新，间接就会重新构建它的子widget，子widget重新构建也就是对应消费消费数据，因为消费者依赖了订阅者本身，来看代码

```
/// Created with Android Studio.
/// User: maoqitian
/// Date: 2019/11/18 0018
/// email: maoqitian068@163.com
/// des:  事件 消费者 获得当前context和指定数据类型的Provider
import 'package:flutter/material.dart';
import 'package:flutter_theme_change/provider/ChangeNotifierProvider.dart';

class Consumer<T> extends StatelessWidget{

  final Widget child;
  //获得当前context
  final Widget Function(BuildContext context, T value) builder;

  Consumer({Key key,@required this.builder,this.child}):assert(builder !=null),super(key:key);


  @override
  Widget build(BuildContext context) {  //默认绑定 注册依赖关系
    return builder(context,ChangeNotifierProvider.of<T>(context)); //自动获取Model 获取更新的数据
  }

}
```
- 由上代码，Consumer的build调用ChangeNotifierProvider.of方法默认就注册了依赖关系，所以由Consumer实现的widget就会由InheritedWidget的功能更新数据。

## 小结
- 以上小结可以用一个流程图代替

![Provider 数据共享原理流程图](https://raw.githubusercontent.com/maoqitian/MaoMdPhoto/master/flutter/InheritedWidget/Provider%E6%95%B0%E6%8D%AE%E5%85%B1%E4%BA%AB%E5%8E%9F%E7%90%86%E6%B5%81%E7%A8%8B%E5%9B%BE.jpg)

# 数据共享组件实践切换主题

- 上一节中手写了一个非常简单基于InheritedWidget的Provider数据共享组件，接下来通过一个切换主题的例子来使用刚刚写好的ChangeNotifierProvider。

- 主题切换这里简单的改变主题颜色，所以共享数据就是颜色值，Demo 思路为使用Dialog，提供可选择的主题颜色，然后点击对应颜色则切换应用主题颜色，接下来一起实现。

## 创建主题model
- model 也可以看做是共享数据，继承ChangeNotifier，这样就能够调用notifyListeners方法触发ChangeNotifierProvider收到数据改变通知

```
/// Created with Android Studio.
/// User: maoqitian
/// Date: 2019/11/18 0018
/// email: maoqitian068@163.com
/// des:  主题 model
import 'package:flutter/material.dart';

class ThemeModel extends ChangeNotifier {
  int settingThemeColor ;
  ThemeModel(this.settingThemeColor);

  void changeTheme (int themeColor){
    this.settingThemeColor = themeColor;
    // 通知监听器（订阅者），重新构建InheritedProvider， 更新状态。
    notifyListeners();
  }
}
```
## MaterialApp作为ChangeNotifierProvider子widget
- 改变主题颜色，也就是MaterialApp的theme 属性，所以讲 MaterialApp作为ChangeNotifierProvider子widget，这样MaterialApp就能收到共享的主题颜色数据值

```
class _MyHomePageState extends State<MyHomePage> {

  int themeColor =0;
  
  @override
  void initState() {
    super.initState();
    themeColor = sp.getInt(SharedPreferencesKeys.themeColor);
    if(themeColor == null ){
      themeColor = 0xFF3391EA;//默认蓝色
    }
  }
  @override
  Widget build(BuildContext context) {
    return Center(
      child: ChangeNotifierProvider<ThemeModel>(
        data: ThemeModel(themeColor),
        child: Consumer<ThemeModel>(
          builder: (BuildContext context,themeModel){
            return MaterialApp(
              theme: ThemeData(
                primaryColor: Color(themeModel.settingThemeColor),
              ),
              home: Scaffold(
                  appBar: AppBar(
                    title: Text("Flutter Theme Change"),
                    actions: <Widget>[
                      Builder(builder: (context){
                        return IconButton(icon: new Icon(Icons.color_lens), onPressed: (){
                          _changeColor(context);
                        });
                      },)
                      // onPressed 点击事件
                    ],
                  ),
                  body: Center(
                    child: Text("主题变化测试"),
                  )
              ),
            );
          },
        ),
      ),
    );
  }

  void _changeColor(BuildContext context) {
      buildSimpleDialog(context);
  }
```
- 在AppBar 加入IconButton 让其点击能显示颜色选择Dialog，Dialog 显示的是一个颜色值数组widget，每个widget实现如下

```
class SingleThemeColor extends StatelessWidget {

  final int themeColor;
  final String colorName;

  const SingleThemeColor({Key key,this.themeColor, this.colorName}):
        super(key:key);

  @override
  Widget build(BuildContext context) {
    return InkWell(
      onTap: () async{
         print("点击了改变主题");
         //改变主题
         ChangeNotifierProvider.of<ThemeModel>(context,listen: false).changeTheme(this.themeColor);
         await SpUtil.getInstance()..putInt(SharedPreferencesKeys.themeColor, this.themeColor);
         Navigator.pop(context);
      },
      child: new Column( // 竖直布局
        children: <Widget>[
           Container(
             width: 50,
             height: 50,
             margin: const EdgeInsets.all(5.0),
             decoration: BoxDecoration( //圆形背景装饰
               borderRadius:BorderRadius.all(
                  Radius.circular(50)
               ),
               color: Color(this.themeColor)
             ),
           ),
           Text(
             colorName,
             style: TextStyle(
               color: Color(this.themeColor),
               fontSize: 14.0),
           ),
        ],
      ),
    );
  }
}
```
- 可以看到每个widget点击响应onTap 则调用ChangeNotifierProvider.of获取ThemeModel对象调用changeTheme方法来触发notifyListeners方法。还有一些细节，比如通过SharedPreferences保存颜色值等代码，具体可以查看文末demo 项目源码地址。
- Demo 运行效果

<img src="https://raw.githubusercontent.com/maoqitian/MaoMdPhoto/master/flutter/InheritedWidget/theme-change.gif"  height="400" width="230">

# 最后

- 看到这里，相信你应该对InheritedWidget有了比较好的理解，了解了原理，使用起轮子来也会更加得心应手吧。如果要使用跨组件数据共享，还是直接使用功能完整的[Provider](https://pub.dev/packages/provider)吧。又一篇文章完成了，相信多少都会对看到文章的你有帮助，文章中如果有错误，请大家给我提出来，大家一起学习进步，如果觉得我的文章给予你帮助，也请给我一个喜欢和关注，同时也欢迎访问我的[**个人博客**](https://www.maoqitian.com)。

### Demo 地址

- [https://github.com/maoqitian/flutter_demo](https://github.com/maoqitian/flutter_demo)

### 参考 

- [Flutter 实战电子书](https://book.flutterchina.club/)

# About me
### blog：
- [个人博客](https://www.maoqitian.com/)
- [掘金](https://juejin.im/user/59e956626fb9a045204b57d4)
- [简书](https://www.jianshu.com/u/f58cd7ff1a08)
- [Github](https://github.com/maoqitian)
### mail：
- [maoqitian@gmail.com]()
- [maoqitian068@163.com]()