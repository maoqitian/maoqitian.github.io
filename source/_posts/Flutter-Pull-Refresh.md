---
title: Flutter Pull Refresh
date: 2020-01-11 22:52:04
categories:  
- Flutter探索 #分类
tags: 
- dart
- Flutter
---
![image](https://raw.githubusercontent.com/maoqitian/MaoMdPhoto/master/flutter/flutter-logo.png)

> 基础页面实现
###  TabBar + TabBarView 实现页面切换联动（类似Android tablayout + ViewPage）效果
<!--more-->
- 直接上代码

```
List <String>_titles=['湖人','勇士','雄鹿','快船','凯尔特人','马刺','76人','猛龙'];
TabController  _tabController;
///省略部分代码
class MyHomePage extends StatefulWidget {
  MyHomePage({Key key, this.title}) : super(key: key);

  final String title;

  ///省略部分代码

class _MyHomePageState extends State<MyHomePage> with SingleTickerProviderStateMixin{

  @override
  void initState() {
    super.initState();
    //初始化控制器 
    _tabController = new TabController(length: _titles.length,vsync: this);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        leading: Icon(Icons.menu),
        title: buildTabBar(),
        //bottom: buildTabBar(),
      ),
      body: TabBarViewLayout()
    );
  }

  Widget buildTabBar() {
    return  TabBar(
          //构造Tab集合
          tabs: _titles.map((String title){
            return Tab(
              text: title,
            );
          }).toList(),
          ///省略部分代码
          controller: _tabController,
        );
  }
}

// TabBarView Widget
class TabBarViewLayout extends StatelessWidget {

  @override
  Widget build(BuildContext context) {
    print("TabBarViewLayout build.......");
    return TabBarView(
      controller: _tabController,
      children: _titles.map((String title){
        return TabPageView(title);
      }).toList(),
    );
  }
}
```
- 如果代码，可以看到在AppBar这个widget的title属性中加入TabBar，也就是AppBat的title模块显示TabBar，也可在AppBar的bottom属性加入；还需要注意TabBar和TabBarView正是通过同一个controller来实现菜单切换和滑动状态同步的，最终运行结果如下，分被设置tabbar在title 和bottom属性

<img src="https://github.com/maoqitian/MaoMdPhoto/raw/master/flutter/refreshPage/page_tab_title.jpg"  height="400" width="230">

<img src="https://github.com/maoqitian/MaoMdPhoto/raw/master/flutter/refreshPage/page_tab_bottom.jpg"  height="400" width="230">

### 下拉刷新，上拉加载更多实现（RefreshIndicator）

- 下拉刷新 Flutter SDK中已经提供了一个RefreshIndicator控件，所以结合RefreshIndicator控件，让其包裹ListView控件，结合滑动监听ScrollController，并且设置头部，尾部加载更多等界面，就可以完成一个通用的下拉刷新，上拉加载更多的通用控件。首先来看看RefreshIndicator构造方法

```
const RefreshIndicator({
  Key key,
  @required this.child, //包装一个可滚动widget
  this.displacement = 40.0,
  @required this.onRefresh, //触发刷新调用方法
  this.color, //指示器颜色
  this.backgroundColor,
  this.notificationPredicate = defaultScrollNotificationPredicate,
  this.semanticsLabel,
  this.semanticsValue,
})
```
- RefreshIndicator包装一个可滚动widget，这里使用ListView

```
@override
  Widget build(BuildContext context) {
    return RefreshIndicator(
      child: ListView.builder(
          ///保持ListView任何情况都能滚动，解决在RefreshIndicator的兼容问题。
          physics: const AlwaysScrollableScrollPhysics(),
          itemBuilder: (context,index){
              return _getItem(index);
          },
          ///根据状态返回绘制 item 数量
          itemCount: _getListCount(),
          ///滑动监听
          controller: _scrollController,
      ),
      onRefresh: _handleRefresh,
      color: Theme.of(context).primaryColor, //指示器颜色
    );
  }
```
- ListView有两个重要方法设置，一个是itemBuilder构建列表item的每一个页面，另一个构建item页面数量itemCount。首先看itemCount方法

```
///根据配置状态返回实际列表数量
  _getListCount() {
    ///是否需要头部
    if (widget.isHaveHeader) {
      return (items.length > 0) ? items.length + 2 : items.length + 1;
    } else {
      if (items.length == 0) {
        return 1;
      }
      return (items.length > 0) ? items.length + 1 : items.length;
    }
  }
```
- 该方法中，做了几种内容类型判断，如果需要头部，用Item 0 的 Widget 作为ListView的头部，列表数量大于0时，因为头部和底部加载更多选项，需要对列表数据总数+2，如果不需要头部，在数据获取为零时，固定返回数量1用于空页面呈现或者错误页面；如果有数据，加上外部加载更多选项，需要对列表数据总数+1。接着看_getItem()方法，返回对应渲染页面。

```
///根据配置状态返回实际列表渲染Item
  _getItem(int index) {
    if (!widget.isHaveHeader && index == items.length && items.length != 0) {
      return _buildProgressIndicator();
    } else if (widget.isHaveHeader && index == _getListCount()-1 && items.length != 0) {
      return _buildProgressIndicator();
    } else if (widget.isHaveHeader && index == 0 && items.length != 0) {
      return widget.headerView();
    } else if (!widget.isHaveHeader && items.length == 0) {
      ///如果不需要头部，并且数据为0，渲染空页面
      if(isLoading){
        return _buildIsLoading();
      }else{
        return _buildEmpty();
      }
    } else if(widget.isHaveHeader && items.length == 0){
      if(isLoading){
        return _buildIsLoading();
      }else{
        return _buildEmpty();
      }
    } else {
      return widget.renderItem(index, items[widget.isHaveHeader ? index-1 : index]);
    }
  }
```
- 该方法中，如果没有设置头部，并且数据不为0，当index等于数据长度时，渲染加载更多页面（因为index是从0开始）；如果设置了头部页面，并且数据不为0，当index等于实际渲染长度 - 1时，渲染加载更多页面（在该方法判断是否已经加载到底）；接着如果设置了头部widget，并且数据不为0，当index = 0 ，渲染头部widget；如果没设置头部，并且数据为0，如果当前正在刷新，渲染Loading页面，否则渲染空页面或者Error页面；同理，如果设置头部，并且数据为0，并且当前正在刷新，渲染Loading页面，否则渲染空页面或者Error页面；如果不是上面情况，则渲染正常渲染Item，如果这里有需要，可以直接返回相对位置的index，如果有头部 index 减一 ，保持不会忽略 index = 0 的数据。

- 接着封装一个统一网络请求方法，外部请求安装固定格式的 Map 将数据返回给下拉刷新上拉加载更多widget，达到通用的目的。

```
 //网络请求获取数据 isRefresh 是否为下拉刷新
  Future<List> makeHttpRequest(bool isRefresh) async {
    if (widget.requestApi is Function) {
      Map listObj = new Map<String, dynamic>();
      if(isRefresh){
        //下拉刷新
        listObj = await widget.requestApi({'pageIndex': 0});
      }else{
        //上拉加载更多
        listObj = await widget.requestApi({'pageIndex': _pageIndex});
      }
      _pageIndex = listObj['pageIndex'];
      _pageTotal = listObj['total'];
      return listObj['list'];
    } else {
      return Future.delayed(Duration(seconds: 2), () {
        return [];
      });
    }
  }
```
- 基础东西写好了，loading 加载动画这里直接就使用现成的轮子好了，推荐一个loading库，[flutter_spinkit](https://github.com/jogboms/flutter_spinkit)
- 贴上loading加载代码（更多实现细节请看文末demo地址代码）

```
Widget _buildIsLoading() {
    return Container(
      width: MediaQuery.of(context).size.width,
      height: MediaQuery.of(context).size.height*0.85,
      child: new Center(
        child: Column(
            crossAxisAlignment: CrossAxisAlignment.center,
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
                 Row(
                   mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                   children: <Widget>[
                     SpinKitCircle(size: 55.0, color: Theme.of(context).primaryColor),
                   ],
                 ),
                Padding(
                 child: Text("正在加载..",
                    style: TextStyle(color: Colors.black54, fontSize: 15.0)),
                padding: EdgeInsets.all(15.0),)
                ],)
    ));
  }
```
- 最后，通过构造方法设置设置需要加载的item值和是否支持下拉刷新和上来加载更多等，灵活配置控件


```
 // 模块item
  final renderItem;
  //数据获取方法
  final requestApi;
  //头部
  final headerView;
  //是否添加头部 默认不添加
  final bool isHaveHeader;
  //是否支持下拉刷新 默认可以下拉刷新
  final bool isCanRefresh;
  //是否支持下拉加载更多 默认可以加载更多
  final bool isCanLoadMore;
  const RefreshPage({@required this.requestApi,
                     @required this.renderItem,
                     this.headerView,
                     this.isHaveHeader = false,
                     this.isCanRefresh = true,
                     this.isCanLoadMore = true })
                     : assert(requestApi is Function),
                       assert(renderItem is Function),
                      super();
```


### 最终demo 效果
<img src="https://github.com/maoqitian/MaoMdPhoto/raw/master/flutter/refreshPage/loadingdata.gif"  height="400" width="230">

<img src="https://github.com/maoqitian/MaoMdPhoto/raw/master/flutter/refreshPage/loadingerror.gif"  height="400" width="230">

<img src="https://github.com/maoqitian/MaoMdPhoto/raw/master/flutter/refreshPage/noloadmore.gif"  height="400" width="230">

### Demo 地址

- [https://github.com/maoqitian/flutter_demo](https://github.com/maoqitian/flutter_demo) 



### About me
#### blog：
- [个人博客](https://www.maoqitian.com/)
- [掘金](https://juejin.im/user/59e956626fb9a045204b57d4)
- [简书](https://www.jianshu.com/u/f58cd7ff1a08)
- [Github](https://github.com/maoqitian)
#### mail：
- [maoqitian@gmail.com]()
- [maoqitian068@163.com]()