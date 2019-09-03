---
title: Android 基本架构之MVP分析与实践
date: 2019-08-30 20:27:22
categories:  
- Android架构 #分类
tags: 
- MVP
- Dagger2
top: true
---
> 开发一个App，和起房子应该有异曲同工之处，起房子需要画好设计图纸，而我们开发App则需要先设计好App整个架构模式。目前Android一般有MVC、MVP和MVVM，本文则先来说说MVP架构。在了解MVP架构之前，有人可能会说，MVP架构是不是有点落后了，但是我想说，如果你公司有老项目，他就是用MVP架构写的，这时候我们MVP知识是不是就派上用场了；任何架构都有它存在的理由，学习架构的思想才是关键。MVP分别代表Model、View、Presenter三个英文字母，和传统的MVC 相比，C替换成了P。Presenter英文单词有主持人意思，也就是说Presenter是View 和 Model 的主持人，按照惯例我们先来看两张图。
# MVC MVP 架构对比图
### mvc
![mvc](https://raw.githubusercontent.com/maoqitian/MaoMdPhoto/master/MVP/mvc.jpg)
### mvp
![mvp](https://raw.githubusercontent.com/maoqitian/MaoMdPhoto/master/MVP/MVP1.jpg)
<!--more-->
- 通过以上两张图对比，MVC在Android中就是我们刚开始学习Android时输出Android代码的真实写照，Activity不仅负责显示View，它还是Controller，我们可以在Activity开始网络请求，请求完成更新UI，也可以在Activity中通过UI组件获取用户输入数据，然后执行网络请求再更新UI，这样一来，一个功能复杂的页面一个Activity三四千行代码是很常见的事情，这也会导致后面维护代码人来读你的Activity代码可能会直接崩溃，同时代码的耦合度也很高。
- 而我们再看MVP架构，这就会很清晰，它把MVC中的VC进行解耦，也就是说**把Activity中的UI逻辑抽象成View 接口 ,把业务逻辑抽象成 presenter 接口, model 还是原来的model**，这样其实就呼应了文章我们所说presenter主持人的意思，model 更新UI需要通过presenter，view 更新model数据也需要通过presenter，相当于presenter主持大局。
- 说了这么多，其实MVC和MVP的区别可以用一句话代替，那就是View能否直接操作Model，接下来我们就看看MVP架构如何在Android中实践。

# Android 实现 mvp 架构

## UI逻辑抽象成View接口
```
/**
 * @author maoqitian
 * @Description View 的基类
 */
public interface BaseView {

    /**
     * 正常显示
     */
    void showNormal();

    /**
     * 显示错误
     */
    void showError();

    /**
     * 正在加载
     */
    void showLoading();
    /**
     * 显示错误信息
     * @param errorMsg 错误信息
     */
    void showErrorMsg(String errorMsg);
}
```
## 业务逻辑抽象成 Presenter 接口
- 抽象之前我们可以想一想，每个presenter都对应一个View 界面，所以我们需要一个方法来绑定对应的View，绑定的目的是为了方便我们在presenter中更新view，当界面销毁的时候也需要一个方法类解绑View。此外，界面肯定不止一个，并且肯定实现前面我们写的BaseView接口，我们用泛型代替，就有了如下BasePresenter 接口

```
/**
 * @author maoqitian
 * @Description Presenter 基类接口
 */
public interface AbstractBasePresenter<T extends BaseView> {

    /**
     * 绑定View
     * @param view
     */
    void attachView(T view);

    /**
     * 解绑View
     */
    void detachView();
}
```
# MVP工作流
- 前面我们已经抽象出了View、Presenter接口，接下来从结合文章开头MVP  architectural pattern图从用户打开App获取数据开始展现整体MVP工作流。

## View 与 Presenter 结合

### View 获取Presenter对象
- View 获取数据需要通过Presenter对象，view在Android 中一般代表Avtivity、或者Fragment。先创建Avtivity和Fragment抽象类类做基础封装。

```
/**
 * @author maoqitian
 * @Description activity基类
 */
public abstract class AbstractActivity extends AppCompatActivity{

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(getLayout());
        onViewCreated();
        initToolbar();
        initEventAndData();
    }
    /**
     * view 的创建 留给子类实现
     */
    protected abstract void onViewCreated();
    /**
     * 初始化 toolbar
     */
    protected abstract void initToolbar();
    /**
     * 初始化数据留给子类实现
     */
    protected abstract void initEventAndData();

    /**
     * 获取布局对象 留给子类实现
     */
    protected abstract int getLayout();

}
```
- 接着我们实现MVP Activity基类

```
/**
 * @author maoqitian
 * @Description MVP BaseActivity 基类
 */
public abstract class  BaseActivity <T extends AbstractBasePresenter> extends AbstractActivity implements BaseView{

    protected T mPresenter;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        mPresenter = createPresenter();
        super.onCreate(savedInstanceState);
        
    }

    @Override
    protected void onViewCreated() {
        if (mPresenter != null) {
            mPresenter.attachView(this);
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if(mPresenter != null){
            mPresenter.detachView();
            mPresenter = null;
        }
    }

    /**
     * 创建Presenter
     */
    protected abstract T createPresenter();

    @Override
    public void showNormal() {

    }

    @Override
    public void showError() {

    }

    @Override
    public void showLoading() {

    }

    @Override
    public void showErrorMsg(String errorMsg) {

    }
}    
```
- 到此，只要我们界面继承BaseActivity，并且实现createPresenter方法，我们就可以很直接在View中通过Presenter来获获取数据，如何获取呢？接着往下看。

### Presenter 获取View 对象
- 现在我们创建一个Presenter基类将其与View结合，为后续步骤做准备，注意我们RxBasePresenter基类构造方法中需要传入DataClient，该类其实就可以概括代表Module。

```
/**
 * @author maoqitian
 * @Description 基于Presenter封装 
 */
public class RxBasePresenter<T extends BaseView> implements AbstractBasePresenter<T>{

    protected T mView;
    
    private DataClient mDataClient;


    public RxBasePresenter(DataClient dataClient){
        this.mDataClient=dataClient;
    }

    @Override
    public void attachView(T view) {
       this.mView=view;
    }

    @Override
    public void detachView() {
        this.mView = null;
    }
}
```
### Presenter 与 View 之间连接 
- 当我们创建View 对应Presenter让其继承 RxBasePresenter，则 Presenter便可以执行Updates view，如何操作呢？我们可以通过接口来进行数据获取与显示的扩展。以下举个例子

```
public interface MainContract {

    interface MainView extends BaseView{
        void showMainData();
    }


    interface MainActivityPresenter extends AbstractBasePresenter<MainView>{
        void getMainData();
    }
}
```
- 至此，我们基本MVP架构其实就已经搭建完成，我们来看看使用

```
/**
 * @author maoqitian
 * @Description MainPresenter （Presenter）
 */
public class MainPresenter extends RxBasePresenter<MainContract.MainView> implements MainContract.MainActivityPresenter {

    //（Model）
    private DataClient mDataClient;

    public MainPresenter(DataClient dataClient) {
        super(dataClient);
        this.mDataClient = dataClient;
    }
    @Override
    public void attachView(MainContract.MainView view) {
        super.attachView(view);
        
    }
    //获取数据
    @Override
    public void getMainData() {
        //mDataClient 网络请求获取数据
        .......
        // 数据获取成功展示数据
        mView.showMainData();
    }
}
/**
 * @author maoqitian
 * @Description MainActivity （View）
 */
public class MainActivity extends BaseActivity<MainPresenter>implements MainContract.MainView{
    ..........
    @Override
    protected MainPresenter createPresenter() {
        return new MainPresenter(new DataClient());
    }
    
    @Override
    protected void initEventAndData() {
        mPresenter.getMainData();
    }
    
    @Override
    public void showMainData() {
    //显示数据
    }   
    
}
```
- 通过以上示例代码，再次对比文章开头的Android MVP  architectural pattern图，从用户打开App获取数据开始展现整体MVP工作流已经走完。

###  谷歌官方示例MVP demo
- 当然上面只是简单的讲解了在Android中搭建基本MVP架构，其实谷歌官方也给我提供了MVP示例代码，具体代码可以自行去了解。
- [谷歌官方MVP示例Demo](https://github.com/googlesamples/android-architecture/tree/todo-mvp/)

# 使用dagger2优化MVP 架构

- 前面我们大致搭建了一个基础的MVP架构，每个Presnter需要我们在View 中去创建，创建Presnter的时候还需要传入Model，这就说明他们之间的解耦还不够。这里我们换一种思路，在原有基础，不管是DataClient（Model）还是对应的Presnter都可以直接提供对应的对象，然后对应的类创建我们就将其注入，这样不就省去了对象创建，Model、Presnter、View 之间耦合度就进一步降低，如何实现？还是强大的谷歌爸爸给我们提供了方案，使用[**Dagger2**](https://github.com/google/dagger)(Dagger是一个完全静态的编译时依赖注入框架，适用于Java和Android)。

## 项目添加对应dagger依赖
- 使用dagger对应版本为2.22.1
```
dependencies {
  implementation 'com.google.dagger:dagger:2.22.1'
  annotationProcessor 'com.google.dagger:dagger-compiler:2.22.1'
  //
  implementation 'com.google.dagger:dagger-android:2.22.1'
  implementation 'com.google.dagger:dagger-android-support:2.22.1'
  annotationProcessor 'com.google.dagger:dagger-android-processor:2.22.1'
}
```
## 改造Presnter类
- 这里我们以上面例子中MainPresenter为例，在其构造方法添加@Inject注解，表明Dagger2 可以从这获取对应MainPresenter实例，注意构造方法中需要DataClient对象，这里也使用Dagger来提供对象（稍后再说）
```
public class MainPresenter extends RxBasePresenter<MainContract.MainView> implements MainContract.MainActivityPresenter {

    private DataClient mDataClient;
    //@Inject注解表示Dagger2 可以从这获取Presenter 实例
    @Inject 
    public MainPresenter(DataClient dataClient) {
        super(dataClient);
        this.mDataClient = dataClient;
    }
}
```
## 改造View 
- View 中结合 Dagger2 本应该继承 DaggerAppCompatActivity，但是我们基类为AbstractActivity，查看DaggerAppCompatActivity源码，直接手动实现DaggerAppCompatActivity中代码。
```
/**
 * @author maoqitian
 * @Description MVP BaseActivity 基类
 */
public abstract class  BaseActivity <T extends AbstractBasePresenter> extends AbstractSimpleActivity implements BaseView, HasFragmentInjector,HasSupportFragmentInjector {

    //Presenter 对象注入 (注意不能使用 private )
    @Inject
    protected T mPresenter;
    //手动实现DaggerAppCompatActivity功能
    @Inject DispatchingAndroidInjector<Fragment> supportFragmentInjector;
    @Inject DispatchingAndroidInjector<android.app.Fragment> frameworkFragmentInjector;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        //必须在super.onCreate之前调用AndroidInjection.inject
        AndroidInjection.inject(this);
        super.onCreate(savedInstanceState);
    }
     @Override
    public AndroidInjector<Fragment> supportFragmentInjector() {
        return supportFragmentInjector;
    }

    @Override
    public AndroidInjector<android.app.Fragment> fragmentInjector() {
        return frameworkFragmentInjector;
    }
}    
```
## 添加Dagger Module和Component

### 创建MainActivityModule
- 抽象类MainActivityModule加入@Module注解，并添加返回我们对应MainActivityPresenter接口的抽象方法，@Binds注解就可以帮我们把MainActivityPresenter接口绑定到MainPresenter上。
```
/**
 * @author maoqitian
 * @Description MainActivity 可以提供的注入对象Module
 * @Time 2019/3/27 0027 23:59
 */
@Module
public abstract class MainActivityModule {
    @ActivityScope
    @Binds
    abstract MainContract.MainActivityPresenter bindPresenter(MainPresenter presenter);
}
```
### 创建用于生成Activity注入器的Module类

```
/**
 * @author maoqitian
 * @Description 用于生成Activity注入器的Module，使用@ContributesAndroidInjector注解并指定modules为
 * @Time 2019/4/14 0014 14:09
 */
@Module
public abstract class ActivityBindingModule {
   
    @ActivityScope
    @ContributesAndroidInjector(modules = MainActivityModule.class)
    abstract MainActivity contributeMainActivity();
    
}
```
### 创建提供我们需要对象的Module类
- 与前文对应，这里我们提供了对应了DataClient对象，也就是MVP中的Model，在注入Presenter的时候将其一起注入。
```
@Module
public class MyAppModule {

    @Provides
    @Singleton
    public DataClient providerDataClient(){
        return new DataClient();
    }
}

```
### 使用@Component注解创建AppCompontent接口类
- Dagger会帮我们自动生成DaggerAppComponent类，继承自AndroidInjector并指定我们自己的Application类，指定AndroidSupportInjectionModule帮助把Android中四大组件以及Fragment进行绑定，@Singleton注解指定单例
```
@Singleton
@Component(modules = {
        MyAppModule.class,
        ActivityBindingModule.class,
        AndroidSupportInjectionModule.class
})
public interface AppComponent extends AndroidInjector<MyApplication> {
}
```
### 改造Application类继承自DaggerApplication
- 按照如下改造MyApplication之后我们从新编译编译一下代码，如果编译通过，dagger就会帮我们生成对应DaggerAppComponent.create()方法，将其返回在applicationInjector()方法中。
```
public class MyApplication extends DaggerApplication {
     @Override
    protected AndroidInjector<? extends DaggerApplication> applicationInjector() {
        return DaggerAppComponent.create();
    }
}
```
- 项目编译通过dagger会在build目录生成对应对象注入类，具体源码以后再出文章分析，这里就先告一段落了。到此，使用dagger2优化MVP 架构基本完成了，但是还有其他细节这里没有提及，比如每个Presenter之间该如何通信，可以使用EventBus，也可以Rxbus等等，具体细节可以看接下架构实践中我写的开源项目的代码。

<img src="https://raw.githubusercontent.com/maoqitian/MaoMdPhoto/master/MVP/buildgenerated.png"  height="400" width="350">


# 架构实践
## 项目介绍
- 通过前面对MVP架构分析介绍，接下来我给大家推荐我的一个开源项目，**这是一款有较好用户体验的开源玩Android客户端。提供丰富完整的功能，更好的体验，旨在更好的浏览https://www.wanandroid.com/网站内容，更好的在手机上进行学习。项目使用Retrofit2 + RxJava2 + Dagger2 +MVP+RxBus架构，尽量使用Material Design控件进行开发。**

## 项目架构图

![MVP-WanAndroid-Architecture](https://raw.githubusercontent.com/maoqitian/MaoMdPhoto/master/WanAndroid/MVP-WanAndroid-Architecture1.jpg)

## 项目地址
> https://github.com/maoqitian/MaoWanAndoidClient

# 总结
-  以前所说的知识一种MVP架构的写法，我们也可以根据自己理解写出不一样的MVP，其实MVP架构看似不错，但也还是会有缺点，那就是写一个页面会产生很多个类，虽然结构清晰，但是要写的代码变多了，凡事都会有利弊。如果你不想自己写这么多的类，github上也有大神写好了轮子([MVPArms](https://github.com/JessYanCoding/MVPArms))专门帮我们生成MVP架构的框架，但是用框架生成代码总归是别人的，只有自己撸一遍，把逻辑流程梳理清楚才会变成自己的东西，才会成长。文章中如果有错误，请大家给我提出来，大家一起学习进步，如果觉得我的文章给予你帮助，也请给我一个喜欢和关注，同时也欢迎访问我的[**个人博客**](https://www.maoqitian.com)。
