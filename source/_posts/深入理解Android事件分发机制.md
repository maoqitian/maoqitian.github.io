---
title: 深入理解Android事件分发机制
date: 2019-01-30 20:46:12
categories:  
- Android进阶 #分类
tags: 
- Android
- 事件分发机制
- View
- ViewGroup
---
> 在理解事件分发机制之前，我们先要明白，事件分发机制是为View服务的，而View是Android中所有控件的基类，View可以是单个的，而多个View组成可以叫做ViewGroup。不管什么View控件，他们基类都是View，在Android多个View的叠加有点像Web中的DOM树形结构，所以当我们点击一个区域有多个View的情况下，**到底这时候该哪个View来响应我们的点击事件呢？事件分发机制就是为了解决这个问题而产生的。**
![ViewGroup官方文档集成关系](https://github.com/maoqitian/MaoMdPhoto/raw/master/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91%E6%9C%BA%E5%88%B6/ViewGroup%E5%AE%98%E6%96%B9%E6%96%87%E6%A1%A3%E7%BB%A7%E6%89%BF%E5%85%B3%E7%B3%BB.png)
<!--more-->
## 事件
- 理解事件分发机制，首先我们要了解事件是什么，这里事件主要指我们操作手机的触摸事件。在Android中所有的输入事件都放在了MotionEvent中。
- MotionEvent是个很庞大的东西，有单点触控、多点触控、鼠标事件等等，这里简单列出基本的单点事件，不做更多深入讨论。

事件 | 简介 
---|---  
ACTION_DOWN | 手指**初次接触到屏幕**时触发  
ACTION_MOVE |手指在**屏幕上滑动**时触发，会会多次触发
ACTION_UP | 手指**离开屏幕**时触发
ACTION_CANCEL | 事件**被上层拦截**时触发
- 正常情况下触摸一次屏幕触发事件序列为ACTION_DOWN-->ACTION_UP
- 有滑动动作的单点序列为ACTION_DOWN-->ACTION_MOVE ..... ACTION_MOVE-->ACTION_UP
 
## 点击事件分发流程
### 事件分发机制场景例子
- 首先我们来看一个比较有意思的例子来带入，我们定义一个公司的几个角色
    
- 老板（Activity）
```
      /**
       * Created by maoqitian on 2018/5/10 0010.
       * 事件分发机制测试 老板
       */

       public class DispatchTouchEventTestActivity extends AppCompatActivity {
       private static final String TAG = Action.TAG1;
       @Override
       protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_dispatch_touch_event_test);
        }
        //Actiivty 只有 dispatchTouchEvent 和 onTouchEvent 方法
        @Override
        public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN){
            Log.i(TAG,Action.dispatchTouchEvent+"经理,现在项目做到什么程度了?");
        }
        return super.dispatchTouchEvent(ev);
        }

        @Override
        public boolean onTouchEvent(MotionEvent event) {
        if (event.getAction() == MotionEvent.ACTION_DOWN){
            Log.i(TAG, Action.onTouchEvent);
         }
        return super.onTouchEvent(event);
       }
```
- 经理（RootView）
```
      /**
       * 经理
       */
      public class RootView extends RelativeLayout {

      private static final String TAG = Action.TAG2;

       public RootView(Context context) {
        super(context);
       }

       public RootView(Context context, AttributeSet attrs) {
        super(context, attrs);
       }

       public RootView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        }

        @Override
        public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            Log.i(TAG, Action.dispatchTouchEvent + "技术部,你们的app快做完了么?");
        }
        return super.dispatchTouchEvent(ev);
        }

        @Override
        public boolean onInterceptTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
             Action.onInterceptTouchEvent+"老板问项目进度" );
           }
          return super.onInterceptTouchEvent(ev);
         }
         @Override
         public boolean onTouchEvent(MotionEvent event) {
         if (event.getAction() == MotionEvent.ACTION_DOWN) {
            Log.i(TAG, Action.onTouchEvent +".....");
          }
          return super.onTouchEvent(event);
        }
       }
```
- 组长（ViewGroup）
```
      /**
       * 组长
       */
       public class ViewGroupA extends RelativeLayout {
       private static final String TAG = Action.TAG3;

        public ViewGroupA(Context context) {
        super(context);
        }

        public ViewGroupA(Context context, AttributeSet attrs) {
        super(context, attrs);
        }

        public ViewGroupA(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        }

        @Override
        public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            Log.i(TAG, Action.dispatchTouchEvent + "项目进度?");
        }
        return super.dispatchTouchEvent(ev);
        }

        @Override
        public boolean onInterceptTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            Log.i(TAG, Action.onInterceptTouchEvent + "我问问程序员");
        }
         return super.onInterceptTouchEvent(ev);
        }

        @Override
        public boolean onTouchEvent(MotionEvent event) {
        if (event.getAction() == MotionEvent.ACTION_DOWN) {
            Log.i(TAG, Action.onTouchEvent);
        }
        return super.onTouchEvent(event);
        }
       }
```
- 程序员（View1）
```
      /**
       * 码农
       */
       public class View1 extends View {
       private static final String TAG = Action.TAG4;

       public View1(Context context) {
         super(context);
       }

       public View1(Context context, AttributeSet attrs) {
        super(context, attrs);
       }

       public View1(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
       }
       //View最为事件传递的最末端，要么消费掉事件，要么不处理进行回传，根本没必要进行事件拦截
       @Override
       public boolean dispatchTouchEvent(MotionEvent event) {
        if (event.getAction() == MotionEvent.ACTION_DOWN) {
            Log.i(TAG, Action.dispatchTouchEvent+"app完成进度么？");
         }
        return super.dispatchTouchEvent(event);
        }
        @Override
        public boolean onTouchEvent(MotionEvent event) {
        if (event.getAction() == MotionEvent.ACTION_DOWN) {
            Log.i(TAG, Action.onTouchEvent+"做好了.");
        }
        return true;
        }
       }
```
- 扫地阿姨（View2）
```
      /**
       * 扫地阿姨
       */
       public class View2 extends View {
       private static final String TAG = Action.TAG5;
       public View2(Context context) {
        super(context);
       }

       public View2(Context context, AttributeSet attrs) {
        super(context, attrs);
       }

       public View2(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
       }

       @Override
       public boolean dispatchTouchEvent(MotionEvent event) {
        if(event.getAction() == MotionEvent.ACTION_DOWN){
            Log.i(TAG, Action.dispatchTouchEvent+"我只是个扫地阿姨，我不懂你说什么");
        }
        return super.dispatchTouchEvent(event);
       }

       @Override
       public boolean onTouchEvent(MotionEvent event) {
        if (event.getAction() == MotionEvent.ACTION_DOWN) {
            Log.i(TAG, Action.onTouchEvent+"经理你问错人了，去问老板吧");
        }
        return super.onTouchEvent(event);
        }
       }
```

![Demo界面截图](https://github.com/maoqitian/MaoMdPhoto/raw/master/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91%E6%9C%BA%E5%88%B6/Demo%E6%88%AA%E5%9B%BE.png)
    
- **场景一**：老板询问App项目进度，事件经过每个领导传递到达程序员处，程序员完成了项目（点击事件被View1消费了）
    
![场景一运行结果](https://github.com/maoqitian/MaoMdPhoto/raw/master/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91%E6%9C%BA%E5%88%B6/%E5%9C%BA%E6%99%AF%E4%B8%80%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C.png)

- **场景二** ：老板异想天开，想造宇宙飞船，事件经过每个领导传递到达程序员处，程序员表示做不了，反馈给老板（事件没有被消费）
```
     @Override
    public boolean onTouchEvent(MotionEvent event) {
        if (event.getAction() == MotionEvent.ACTION_DOWN) {
            Log.i(TAG, Action.onTouchEvent+"这个真心做不了啊，把我做了吧");
        }
        return super.onTouchEvent(event);
    }
```
![场景二运行截图](https://github.com/maoqitian/MaoMdPhoto/raw/master/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91%E6%9C%BA%E5%88%B6/%E5%9C%BA%E6%99%AF%E4%BA%8C%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C.png)
- **场景三**：老板询问技术部本月表现，只需要组长汇报就行，不需要通知程序员（ViewGroup 拦截并消费了事件）
```
     @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            Log.i(TAG, Action.onInterceptTouchEvent + "我看看组员绩效情况");
        }
         //return super.onInterceptTouchEvent(ev);
        return true;//拦截事件 onTouchEvent 中进行处理
    }
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        if (event.getAction() == MotionEvent.ACTION_DOWN) {
             Action.onTouchEvent+"技术部组员最近表现都很好,项目按时完成，没有迟到早退");
        }
        return true;//消费事件 
    }
```
![场景三运行截图](https://github.com/maoqitian/MaoMdPhoto/raw/master/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91%E6%9C%BA%E5%88%B6/%E5%9C%BA%E6%99%AF%E4%B8%89%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C.png)
### 事件分发机制的三个重要方法
- 从这三个场景我们可看出事件分发机制主要有三个方法来处理
#### public boolean dispatchTouchEvent(MotionEvent ev)
  - 该方法的作用是事件的分发，返回结果表示是否消耗事件，消耗则会调用当前View的onTouchEvent，否则传递事件，调用子View的dispatchTouchEvent方法，只要时间传递到该View，dispatchTouchEvent方法必定是会被首先调用的。
#### public boolean onInterceptTouchEvent(MotionEvent ev)
- 该方法表示是否对分发的事件进行拦截，如果进行了拦截，则该方法在这一次的时间传递序列中奖不会被再调用，该方法在dispatchTouchEvent被调用，我们需要注意一点，View是没有该方法的，View是单个的，我们可以理解它为事件传递的终点，终点要么消费事件，要么不消费事件把事件进行回传，而ViewGroup则包含不止一个View，所以他可以把时间传递给子View，也可以拦截事件自己处理不传递给子View。

#### public boolean onTouchEvent(MotionEvent event)
      
- 该方法表示处理拦截的事件，如果不进行处理（事件消耗），也就是不反回true，则当前View不会再次接收到该事件
       
#### 三个方法之间的关系
```
       public boolean dispatchTouchEvent(MotionEvent ev) {
        boolean isDispatch;
        if(onInterceptTouchEvent(ev)){
           isDispatch=onTouchEvent(ev);
        }else {
           isDispatch=childView.dispatchTouchEvent(ev);
        }
        return isDispatch;
       }
```
- 结合这段伪代码和前面的例子的场景三，我们可以发现ViewGroup的事件分发规则是这样的，时间传递到ViewGroup首先调用它的dispatchTouchEvent方法，接下来是调用onInterceptTouchEvent方法，如果该方法但会true，则说明当前ViewGroup要拦截该事件，拦截之后则调用当前ViewGroup的onTouchEvent方法，如果不进行拦截则调用子View的dispatchTouchEvent方法，结合场景二，如果到最后事件都没有被消费掉，则最后返回Activity，Activity不处理则事件消失。
- 结合场景一、场景二，View接收到事件，如果进行处理，则直接在onTouchEvent进行处理返回true就表示事件被消费了，不进行处理则调用父类onTouchEvent方法或者返回false表示不消费该事件，然后事件再原路返回向上传递。
### Activity 传递事件   
- 前面我们只是描述了ViewGroup和View之间的时间传递，我们看到例子中的场景事件都是从老板（Activity）开始的，而Activity本身并不是继承View，所以我们需要了解Activity是如何把事件传递到View的，从源码的角度来看是比较清晰的，下面一起来看看。
- Activity 本身并不是View，那他去哪里加载View呢？setContentView()这个方法相信大家都不陌生，他加载我们的布局，布局中包括控件，也就是加载我们的View，
```
   /**
     * Set the activity content from a layout resource.  The resource will be
     * inflated, adding all top-level views to the activity.
     *
     * @param layoutResID Resource ID to be inflated.
     *
     * @see #setContentView(android.view.View)
     * @see #setContentView(android.view.View, android.view.ViewGroup.LayoutParams)
     */
    public void setContentView(@LayoutRes int layoutResID) {
        getWindow().setContentView(layoutResID);
        initWindowDecorActionBar();
    }
```
- 我们可以看到调用的是 getWindow().setContentView(layoutResID)这个方法，继续找getWindow()
```
   /**
     * Retrieve the current {@link android.view.Window} for the activity.
     * This can be used to directly access parts of the Window API that
     * are not available through Activity/Screen.
     *
     * @return Window The current window, or null if the activity is not
     *         visual.
     */
    public Window getWindow() {
        return mWindow;
    }
```
- getWindow()方法返回的是mWindow，继续找mWindow对象，发现在Activity中定义的是Window对象
```
private Window mWindow;
```
- 查看Window源码，注释说得非常清楚，Window的唯一实现类是PhoneWindow
```
     /**
      * <p>The only existing implementation of this abstract class is
      * android.view.PhoneWindow, which you should instantiate when needing a
      * Window.
      */
      public abstract class Window {
      //省略部分代码
      ...
      }
```
- 在Activity源码的attch()方法中我们也看到 mWindow 的实例对象确实是PhoneWindow
```
     final void attach(Context context, ActivityThread aThread,
            Instrumentation instr, IBinder token, int ident,
            Application application, Intent intent, ActivityInfo info,
            CharSequence title, Activity parent, String id,
            NonConfigurationInstances lastNonConfigurationInstances,
            Configuration config, String referrer, IVoiceInteractor voiceInteractor,
            Window window, ActivityConfigCallback activityConfigCallback) {
        .....
        mWindow = new PhoneWindow(this, window, activityConfigCallback);
        ......}
```
- 所以我们继续看PhoneWindow，这时必须要记住，我们还在找setContentView()方法，PhoneWindow的setContentView()方法
```
    @Override
    public void setContentView(int layoutResID) {
        // Note: FEATURE_CONTENT_TRANSITIONS may be set in the process of installing the window
        // decor, when theme attributes and the like are crystalized. Do not check the feature
        // before this happens.
        if (mContentParent == null) {
            installDecor();
        } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            mContentParent.removeAllViews();
        }
        if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                    getContext());
            transitionTo(newScene);
        } else {
            mLayoutInflater.inflate(layoutResID, mContentParent);
        }
        mContentParent.requestApplyInsets();
        final Callback cb = getCallback();
        if (cb != null && !isDestroyed()) {
            cb.onContentChanged();
        }
        mContentParentExplicitlySet = true;
    }
```
- 该方法中我们重点看installDecor()方法
```
    private void installDecor() {
        mForceDecorInstall = false;
        if (mDecor == null) {
            mDecor = generateDecor(-1);
            mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
            mDecor.setIsRootNamespace(true);
            if (!mInvalidatePanelMenuPosted && mInvalidatePanelMenuFeatures != 0) {
                mDecor.postOnAnimation(mInvalidatePanelMenuRunnable);
            }
        } else {
            mDecor.setWindow(this);
        }
        if (mContentParent == null) {
            mContentParent = generateLayout(mDecor);
            .......
            }
            .......
        }    
```
- 好像没发现什么，继续看generateDecor(int featureId)方法
```
    protected DecorView generateDecor(int featureId) {
        // System process doesn't have application context and in that case we need to directly use
        // the context we have. Otherwise we want the application context, so we don't cling to the
        // activity.
        Context context;
        ......
        return new DecorView(context, featureId, this, getAttributes());
    }
```
- 到此我们发现，他返回的是DecorView，DecorView是PhoneWindow的内部类，我们再看generateLayout(mDecor)方法
```
    protected ViewGroup generateLayout(DecorView decor){
    ....
    // Inflate the window decor.
        int layoutResource;
        int features = getLocalFeatures();
        // System.out.println("Features: 0x" + Integer.toHexString(features));
        if ((features & (1 << FEATURE_SWIPE_TO_DISMISS)) != 0) {
            layoutResource = R.layout.screen_swipe_dismiss;
            setCloseOnSwipeEnabled(true);
        } else if ((features & ((1 << FEATURE_LEFT_ICON) | (1 << FEATURE_RIGHT_ICON))) != 0) {
            if (mIsFloating) {
                TypedValue res = new TypedValue();
                getContext().getTheme().resolveAttribute(
                        R.attr.dialogTitleIconsDecorLayout, res, true);
                layoutResource = res.resourceId;
            } else {
                layoutResource = R.layout.screen_title_icons;
            }
            // XXX Remove this once action bar supports these features.
            removeFeature(FEATURE_ACTION_BAR);
            // System.out.println("Title Icons!");
        } else if ((features & ((1 << FEATURE_PROGRESS) | (1 << FEATURE_INDETERMINATE_PROGRESS))) != 0
                && (features & (1 << FEATURE_ACTION_BAR)) == 0) {
            // Special case for a window with only a progress bar (and title).
            // XXX Need to have a no-title version of embedded windows.
            layoutResource = R.layout.screen_progress;
            // System.out.println("Progress!");
        } else if ((features & (1 << FEATURE_CUSTOM_TITLE)) != 0) {
            // Special case for a window with a custom title.
            // If the window is floating, we need a dialog layout
            if (mIsFloating) {
                TypedValue res = new TypedValue();
                getContext().getTheme().resolveAttribute(
                        R.attr.dialogCustomTitleDecorLayout, res, true);
                layoutResource = res.resourceId;
            } else {
                layoutResource = R.layout.screen_custom_title;
            }
            // XXX Remove this once action bar supports these features.
            removeFeature(FEATURE_ACTION_BAR);
        } else if ((features & (1 << FEATURE_NO_TITLE)) == 0) {
            // If no other features and not embedded, only need a title.
            // If the window is floating, we need a dialog layout
            if (mIsFloating) {
                TypedValue res = new TypedValue();
                getContext().getTheme().resolveAttribute(
                        R.attr.dialogTitleDecorLayout, res, true);
                layoutResource = res.resourceId;
            } else if ((features & (1 << FEATURE_ACTION_BAR)) != 0) {
                layoutResource = a.getResourceId(
                        R.styleable.Window_windowActionBarFullscreenDecorLayout,
                        R.layout.screen_action_bar);
            } else {
                layoutResource = R.layout.screen_title;
            }
            // System.out.println("Title!");
        } else if ((features & (1 << FEATURE_ACTION_MODE_OVERLAY)) != 0) {
            layoutResource = R.layout.screen_simple_overlay_action_mode;
        } else {
            // Embedded, so no decoration is needed.
            layoutResource = R.layout.screen_simple;
            // System.out.println("Simple!");
        }
        .......
    }
```
- 该方法比较长，只截取一部分，方法根据不同的情况加载不同的布局给layoutResource，看其中一个layout.screen_title布局
```
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:fitsSystemWindows="true">
    <!-- Popout bar for action modes -->
    <ViewStub android:id="@+id/action_mode_bar_stub"
              android:inflatedId="@+id/action_mode_bar"
              android:layout="@layout/action_mode_bar"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:theme="?attr/actionBarTheme" />
    <FrameLayout
        android:layout_width="match_parent" 
        android:layout_height="?android:attr/windowTitleSize"
        style="?android:attr/windowTitleBackgroundStyle">
        <TextView android:id="@android:id/title" 
            style="?android:attr/windowTitleStyle"
            android:background="@null"
            android:fadingEdge="horizontal"
            android:gravity="center_vertical"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />
    </FrameLayout>
    <FrameLayout android:id="@android:id/content"
        android:layout_width="match_parent" 
        android:layout_height="0dip"
        android:layout_weight="1"
        android:foregroundGravity="fill_horizontal|top"
        android:foreground="?android:attr/windowContentOverlay" />
    </LinearLayout>
```
- 这时我们只是了解了Activity的setContentView方法，我们看看Activity的dispatchTouchEvent方法
```
    /**
     * Called to process touch screen events.  You can override this to
     * intercept all touch screen events before they are dispatched to the
     * window.  Be sure to call this implementation for touch screen events
     * that should be handled normally.
     *
     * @param ev The touch screen event.
     *
     * @return boolean Return true if this event was consumed.
     */
    public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            onUserInteraction();
        }
        if (getWindow().superDispatchTouchEvent(ev)) {
            return true;
        }
        return onTouchEvent(ev);
    }
```
- 显然调用getWindow().superDispatchTouchEvent(ev)，根据前面的分析也就是PhoneWindow的dispatchTouchEvent方法
```
    // This is the top-level view of the window, containing the window decor.
    private DecorView mDecor;
    
    @Override
    public boolean superDispatchTouchEvent(MotionEvent event) {
        return mDecor.superDispatchTouchEvent(event);
    }
    
    // This is the view in which the window contents are placed. It is either
    // mDecor itself, or a child of mDecor where the contents go.
    ViewGroup mContentParent;
```
- 可以看到PhoneWindow的superDispatchTouchEvent调用的是DecorView的superDispatchTouchEvent方法，前面我们知道DecorView其实是ViewGroup（上述generateLayout(mDecor)返回值），到此我们可以串联起来，**Activity的setContentView其实是Window对象的实现是其唯一实现类PhoneWindown的内部类DecorView来作为Activity的根View，也就是说从Activity开始传递的是从PhoneWindow开始**，也就是源码中的installDecor得到的DecorView充当了Activity传递事件的View，DecorView可以理解为当前页面的底层容器，底层容器DecorView在根据自己是ViewGroup把事件再向他的子View传递，也就是我们平时写的界面最上层View，也就是setContentView加载的布局根布局View，下图结合实例很清晰的可以表示出Activity的构成。
    
![Activity构成对比图](https://github.com/maoqitian/MaoMdPhoto/raw/master/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91%E6%9C%BA%E5%88%B6/Activity%E6%9E%84%E6%88%90%E5%AF%B9%E6%AF%94%E5%9B%BE.png)

- 到此我们可以写出一个事件传递的流程为
```
    Activity －> PhoneWindow －> DecorView －> ViewGroup －> ... －> View
```
- 总结一下每个传递者具有的方法，我们注意到Activity没有onInterceptTouchEvent方法，其实很容易理解，Activity作为事件的初始传递者如果拦截了事件，也就是我们点击界面无响应，这也就使得我们用户的点击没什么意义，肯定是我们点击界面中的某个view响应才符合操作。(PhoneWindow在Android都是隐藏的，不做记录)
    
类型 | 相关方法 | Activity| ViewGroup| View|
---|--- |--- |--- |---
事件分发 | dispatchTouchEvent    | 有 | 有 | 有
事件拦截 | onInterceptTouchEvent | 无 | 有 | 无
事件消费 | onTouchEvent | 有 | 有| 有 

#### 点击事件分发原则
      
- onInterceptTouchEvent拦截事件，该View的onTouchEvent方法才会被调用，只有onTouchEvent返回true才表示该事件被消费，否则回传到上层View的onTouchEvent方法。
- 如果事件一直不被消费，则最终回传给Activity，Activity不消费则事件消失。
- 事件是否被消费是根据返回值，true表示消费，false表示不消费。
### 从源码角度继续分析ViewGroup和View事件传递流程
>经过前面的研究，我们回顾一下,一个点击事件用MotionEvent表示，事件最先传递到Activity，调用Activity的dispatchTouchEvent方法，事件处理工作交给PhoneWindow，PhoneWindow在把事件传递给DecorView，最后DecorView作为我们界面底层容器装载我们setContentView的布局，我们写布局一般都是啥layout作为根布局，也就是ViewGroup，DecorView把事件传递到ViewGroup的dispatchTouchEvent方法，我们就从ViewGroup的dispatchTouchEvent源码开始分析
#### ViewGroup事件传递流程
- ViewGroup方法比较长，我们一段一段来
```
    /**
     * When set, this ViewGroup should not intercept touch events.
     * {@hide}
     */
    @UnsupportedAppUsage
    protected static final int FLAG_DISALLOW_INTERCEPT = 0x80000;
    
    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
    ......
    // Handle an initial down.
            if (actionMasked == MotionEvent.ACTION_DOWN) {
                // Throw away all previous state when starting a new touch gesture.
                // The framework may have dropped the up or cancel event for the previous gesture
                // due to an app switch, ANR, or some other state change.
                cancelAndClearTouchTargets(ev);
                resetTouchState();
            }
    // Check for interception.
            final boolean intercepted;
            if (actionMasked == MotionEvent.ACTION_DOWN
                    || mFirstTouchTarget != null) {
                final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
                if (!disallowIntercept) {
                    intercepted = onInterceptTouchEvent(ev);
                    ev.setAction(action); // restore action in case it was changed
                } else {
                    intercepted = false;
                }
            } else {
                // There are no touch targets and this action is not an initial down
                // so this view group continues to intercept touches.
                intercepted = true;
            }
        .....}
        
    /**
     * Resets all touch state in preparation for a new cycle.
     */
    private void resetTouchState() {
        clearTouchTargets();
        .....
        mGroupFlags &= ~FLAG_DISALLOW_INTERCEPT;
        ......
    }        
        
    /**
     * Clears all touch targets.
     */
     private void clearTouchTargets() {
        ......
            mFirstTouchTarget = null;
        }
    }
```
- 一上来首先判断了事件是否为ACTION_DOWN，如果是ACTION_DOWN事件，则调用resetTouchState()方法，resetTouchState()钟调用了clearTouchTargets()使mFirstTouchTarget=null，而前面我们了解事件的时候也说过一个事件是ACTION_DOWN开始到ACTION_UP结束，也就是说ACTION_DOWN出现表示一个新的事件的开始；接下来再次判断为ACTION_DOWN和mFirstTouchTarget！=null，我们看到条件成立之后才能调用onInterceptTouchEvent方法，也就是说mFirstTouchTarget！=null成立说明此时不拦截事件，而mFirstTouchTarget==null成立则说明事件已经被拦截，并且不会再有ACTION_DOWN，因为此时这个一个事件还没结束，此时不管ACTION_MOVE还是ACTION_UP动作，都交由现在拦截了事件的ViewGroup来处理，**并且不会再次调用onInterceptTouchEvent方法**（说明该方法并不是每次都会调用的）。
     
- 我们还看到一个标记位FLAG_DISALLOW_INTERCEPT，它一般是由子View的requestDisallowInterceptTouchEvent方法设置的，表示ViewGroup无法拦截除了ACTION_DOWN以外的其他动作，我们看到源码第一个判断就会明白，只要是ACTION_DOWN动作，这个标记位都会被重置，并且ViewGroup会调用自己onInterceptTouchEvent方法表达是否需要拦截这新一轮的点击事件。
     
- 接着看dispatchTouchEvent方法剩下的其他代码段
```
    public boolean dispatchTouchEvent(MotionEvent ev) {
    .......
    if (newTouchTarget == null && childrenCount != 0) {
                        final float x = ev.getX(actionIndex);
                        final float y = ev.getY(actionIndex);
                        // Find a child that can receive the event.
                        // Scan children from front to back.
                        final ArrayList<View> preorderedList = buildTouchDispatchChildList();
                        final boolean customOrder = preorderedList == null
                                && isChildrenDrawingOrderEnabled();
                        final View[] children = mChildren;
                        for (int i = childrenCount - 1; i >= 0; i--) {
                            final int childIndex = getAndVerifyPreorderedIndex(
                                    childrenCount, i, customOrder);
                            final View child = getAndVerifyPreorderedView(
                                    preorderedList, children, childIndex);
                            // If there is a view that has accessibility focus we want it
                            // to get the event first and if not handled we will perform a
                            // normal dispatch. We may do a double iteration but this is
                            // safer given the timeframe.
                            if (childWithAccessibilityFocus != null) {
                                if (childWithAccessibilityFocus != child) {
                                    continue;
                                }
                                childWithAccessibilityFocus = null;
                                i = childrenCount - 1;
                            }
                            if (!canViewReceivePointerEvents(child)
                                    || !isTransformedTouchPointInView(x, y, child, null)) {
                                ev.setTargetAccessibilityFocus(false);
                                continue;
                            }
                            newTouchTarget = getTouchTarget(child);
                            if (newTouchTarget != null) {
                                // Child is already receiving touch within its bounds.
                                // Give it the new pointer in addition to the ones it is handling.
                                newTouchTarget.pointerIdBits |= idBitsToAssign;
                                break;
                            }
                            resetCancelNextUpFlag(child);
                            if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                                // Child wants to receive touch within its bounds.
                                mLastTouchDownTime = ev.getDownTime();
                                if (preorderedList != null) {
                                    // childIndex points into presorted list, find original index
                                    for (int j = 0; j < childrenCount; j++) {
                                        if (children[childIndex] == mChildren[j]) {
                                            mLastTouchDownIndex = j;
                                            break;
                                        }
                                    }
                                } else {
                                    mLastTouchDownIndex = childIndex;
                                }
                                mLastTouchDownX = ev.getX();
                                mLastTouchDownY = ev.getY();
                                newTouchTarget = addTouchTarget(child, idBitsToAssign);
                                alreadyDispatchedToNewTouchTarget = true;
                                break;
                            }
        ......                    
       }
       
       
         /**
          * Transforms a motion event into the coordinate space of a particular child view,
          * filters out irrelevant pointer ids, and overrides its action if necessary.
          * If child is null, assumes the MotionEvent will be sent to this ViewGroup instead.
          */
    private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
            View child, int desiredPointerIdBits) {
       .......
            if (child == null) {
                handled = super.dispatchTouchEvent(event);
            } else {
                handled = child.dispatchTouchEvent(event);
        .......
        }
```
- 这里显示的逻辑还是非常清晰的，如果ViewGroup不拦截点击事件，则首先遍历子View的最外层，获取点击事件的X坐标和Y坐标判断是否和当前子View的坐标相匹配，而dispatchTransformedTouchEvent方法实际上就是调用子View的dispatchTouchEvent方法，这样就完成了ViewGroup到子View的事件分发。
- ViewGroup默认不拦截任何事件，他的onInterceptTouchEvent方法默认返回false
```
     public boolean onInterceptTouchEvent(MotionEvent ev) {
        if (ev.isFromSource(InputDevice.SOURCE_MOUSE)
        //鼠标点击处理
                && ev.getAction() == MotionEvent.ACTION_DOWN
                && ev.isButtonPressed(MotionEvent.BUTTON_PRIMARY)
                && isOnScrollbarThumb(ev.getX(), ev.getY())) {
            return true;
        }
        return false;
      }
```
- 如下源码，如果ViewGroup将事件传递到子View，则会调用addTouchTarget(child, idBitsToAssign)方法，并退出遍历子View的循环
```
    public boolean dispatchTouchEvent(MotionEvent ev) {
    .......
    newTouchTarget = addTouchTarget(child, idBitsToAssign);
    break;
    .....
     // Dispatch to touch targets.
            if (mFirstTouchTarget == null) {
                // No touch targets so treat this as an ordinary view.
                handled = dispatchTransformedTouchEvent(ev, canceled, null,
                        TouchTarget.ALL_POINTER_IDS);
            } 
       .....     
    }
    /**
     * Adds a touch target for specified child to the beginning of the list.
     * Assumes the target child is not already present.
     */
    private TouchTarget addTouchTarget(@NonNull View child, int pointerIdBits) {
        final TouchTarget target = TouchTarget.obtain(child, pointerIdBits);
        target.next = mFirstTouchTarget;
        mFirstTouchTarget = target;
        return target;
    }                            
```
- 如上源码，调用addTouchTarget方法，会给mFirstTouchTarget赋值，也就是说mFirstTouchTarget！=null，前面我们已经讨论过，mFirstTouchTarget==null则拦截所有的事件给该ViewGroup处理，可见mFirstTouchTarget是否赋值对于ViewGroup的事件拦截起了关键的作用。
- 接着往下看，如果子View遍历结束后事件还是没有进行处理，这样的情况有两种可能，一个就是上面提到的例子场景二，ViewGroup的子View没有消费事件，也就是子View的onTouchEvent返回了false，另一个情况则是则是ViewGroup子View，也就不存在事件传递子View的情况。我们看如下代码，是在上面分析的代码之后出现，第三个参数子View为null，也就是调用super.dispatchTouchEvent(event)方法，ViewGroup是继承View，也就是说不管是否拦截，ViewGropu最终还是将点击事件交由到View来处理了，只是child.dispatchTouchEvent还是super.dispatchTouchEvent的问题。 ViewGroup的源码事件分发就到这里，接下来我们分析一下View的事件分发流程。
```
    public boolean dispatchTouchEvent(MotionEvent ev) {
    .....
     // Dispatch to touch targets.
            if (mFirstTouchTarget == null) {
                // No touch targets so treat this as an ordinary view.
                handled = dispatchTransformedTouchEvent(ev, canceled, null,
                        TouchTarget.ALL_POINTER_IDS);
            } 
       .....     
    }
```
#### View的事件分发流程
- 首先我们看View的dispatchTouchEvent方法
```
     /**
     * Pass the touch screen motion event down to the target view, or this
     * view if it is the target.
     *
     * @param event The motion event to be dispatched.
     * @return True if the event was handled by the view, false otherwise.
     */
    public boolean dispatchTouchEvent(MotionEvent event) {
    boolean result = false;
    ....
    if (onFilterTouchEventForSecurity(event)) {
            if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
                result = true;
            }
            //noinspection SimplifiableIfStatement
            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnTouchListener != null
                    && (mViewFlags & ENABLED_MASK) == ENABLED
                    && li.mOnTouchListener.onTouch(this, event)) {
                result = true;
            }
            if (!result && onTouchEvent(event)) {
                result = true;
            }
        }
    .....  
    return result;
    }
```
- 我们看到上面的源码中，View对于点击的事件的处理首先是判断是注册OnTouchListener，并且如果OnTouchListener的onTouch放回true，则整个dispatchTouchEvent返回true，已经拦截了事件，则不会执行下面的onTouchEvent方法的调用，也就是说事件拦截了，但是不调用onTouchEvent方法，这里其实很好理解，如果开发者注册了OnTouchListener并在onTouch放回true，说明开发者是想自己来处理触摸事件，而onTouchEvent是属于Android的事件传递机制方法，是系统帮我们处理的，所以当我们自己处理了点击事件，就不需要系统来再次处理了。所以OnTouchListener的调用有先级高于onTouchEvent。
- 如果Ciew没有注册OnTouchListener方法，接下来事件传递到onTouchEvent方法，我们接着看onTouchEvent源码
```
     /**
     * Implement this method to handle touch screen motion events.
     * <p>
     * If this method is used to detect click actions, it is recommended that
     * the actions be performed by implementing and calling
     * {@link #performClick()}. This will ensure consistent system behavior,
     * including:
     * <ul>
     * <li>obeying click sound preferences
     * <li>dispatching OnClickListener calls
     * <li>handling {@link AccessibilityNodeInfo#ACTION_CLICK ACTION_CLICK} when
     * accessibility features are enabled
     * </ul>
     *
     * @param event The motion event.
     * @return True if the event was handled, false otherwise.
     */
    public boolean onTouchEvent(MotionEvent event) {
        ....
        final boolean clickable = ((viewFlags & CLICKABLE) == CLICKABLE
                || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
                || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE;
        if ((viewFlags & ENABLED_MASK) == DISABLED) {
            if (action == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
                setPressed(false);
            }
            mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
            // A disabled view that is clickable still consumes the touch
            // events, it just doesn't respond to them.
            return clickable;
        }
      ....
      }
```
- 通过上面源码和注释，我们可以知道，View即使是处于不可用状态，他还是会消费(consumes)点击事件，只是他不会响应点击事件，也就是返回各种点击的状态（点击，长按）。
- 接着看看剩下源码对点击事件的处理
```
    public boolean onTouchEvent(MotionEvent event) {
    if (clickable || (viewFlags & TOOLTIP) == TOOLTIP) {
            switch (action) {
                case MotionEvent.ACTION_UP:
                   ......
                    if (!clickable) {
                        removeTapCallback();
                        removeLongPressCallback();
                        mInContextButtonPress = false;
                        mHasPerformedLongPress = false;
                        mIgnoreNextUpEvent = false;
                        break;
                    }
                    boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
                    if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {
                    .......
                        if (!mHasPerformedLongPress && !mIgnoreNextUpEvent) {
                            // This is a tap, so remove the longpress check
                            removeLongPressCallback();
                            // Only perform take click actions if we were in the pressed state
                            if (!focusTaken) {
                                // Use a Runnable and post this rather than calling
                                // performClick directly. This lets other visual state
                                // of the view update before click actions start.
                                if (mPerformClick == null) {
                                    mPerformClick = new PerformClick();
                                }
                                if (!post(mPerformClick)) {
                                    performClickInternal();
                                }
                            }
                        }
                    .....
                    }
                break;
            }
            ....
            return true;
        }
        return false;
    }
```
- 触摸事件结束，也就是ACTION_UP，所以这里我们看对于ACTION_UP的处理就可以了。我们看到对于ACTION_UP，，如果没有!clickable，也就是没有View的CLICKABLE、LONG_CLICKABLE和CONTEXT_CLICKABLE都不存在，则清除所有的状态回调等，如果其中一个存在，则直接消费这个时间，我们看到方法后面有个retrun true存在，也证实事件被消费了，也就是onTouchEvent方法返回了true。而如果ACTION_UP没有消费事件，最终onTouchEvent方法是返回false。
- 到这里，我们还看到ACTION_UP事件会触发performClickInternal();方法，我们看看他做了什么
```
     private boolean performClickInternal() {
        // Must notify autofill manager before performing the click actions to avoid scenarios where
        // the app has a click listener that changes the state of views the autofill service might
        // be interested on.
        notifyAutofillManagerOnClick();
        return performClick();
    }

    public boolean performClick() {
        // We still need to call this method to handle the cases where performClick() was called
        // externally, instead of through performClickInternal()
        notifyAutofillManagerOnClick();
        final boolean result;
        final ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnClickListener != null) {
            playSoundEffect(SoundEffectConstants.CLICK);
            li.mOnClickListener.onClick(this);
            result = true;
        } else {
            result = false;
        }
        sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);
        notifyEnterOrExitForAutoFillIfNeeded(true);
        return result;
    }
```
- 可以从源码看到他最终调用的是performClick()方法，如果View设置了OnClickListener，则会调用onClick方法。我们知道View默认的LONG_CLICKABLE是false，而CLICKABLE需要根据具体View才能知道，比如Button是可点击的，则CLICKABLE为true，而ImageView默认是不可点击的，所以CLICKABLE为false，但是开发中我们也发现，不管View是否可以点击，只要我们设置setOnClickListener()或者setOnLongClickListener()方法，则该View就是可以被点击或者长按的，也就是LONG_CLICKABLE或者CLICKABLE为true。我们从源码可以看出。到此，从源码角度分析事件分发机制的流程我们已经走完。
```
     /**
     * Register a callback to be invoked when this view is clicked. If this view is not
     * clickable, it becomes clickable.
     *
     * @param l The callback that will run
     *
     * @see #setClickable(boolean)
     */
    public void setOnClickListener(@Nullable OnClickListener l) {
        if (!isClickable()) {
            setClickable(true);
        }
        getListenerInfo().mOnClickListener = l;
    }
    
    public void setOnLongClickListener(@Nullable OnLongClickListener l) {
        if (!isLongClickable()) {
            setLongClickable(true);
        }
        getListenerInfo().mOnLongClickListener = l;
    }
```
- 经过前面的分析，我们还可以排出与View相关的事件调度优先顺序为onTouchListener>onTouchEvent > onLongClickListener > onClickListener 
>最后，总结事件分发机制的核心知识点
- 正常情况下触摸一次屏幕触发事件序列为ACTION_DOWN-->ACTION_UP
- 当一个View决定拦截，那么这一个事件序列只能由这个View来处理，onInterceptTouchEvent方法并不是每次产生动作都会被调用到，当我们需要提前出来想要拦截的动作需要在事件必须传递到该ViewGroup的前提下在dispatchTouchEvent方法中进程操作。
- 一个View开始处理事件，但是它不消耗ACTION_DOWN，也就是onTouchEvent返回false，则这个事件会交由他的父元素的onTouchEvent方法来进行处理，而这个事件序列的其他剩余ACACTION_MOVE，ACTION_UP也不会再给该View来处理。
- View没有onInterceptTouchEvent方法，View一旦接收到事件就调用onTouchEvent方法
- ViewGroup默认不拦截任何事件（onInterceptTouchEvent方法默认返回false）。
- View的onTouchEvent方法默认是处理点击事件的，除非他是不可点击的（clickable和longClickable同时为false）
- 事件分发机制的核心原理就是责任链模式，事件层层传递，直到被消费。
## 最后说点
- 终于，把事件分发机制给回顾了一遍，其实五月份的时候我就复习过一次事件分发机制，但是当时没有记录，所以这次在回头看以前记得有些知识点感觉还是模糊，所以记录下来才能在以后忘记的时候去回顾再总结。如果文章中有写得不对的地方，请给我留言指出，大家一起学习进步。如果觉得我的文章给予你帮助，也请给我一个喜欢和关注。
- 参考链接
 
  - [Android 源码 Activity](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/app/Activity.java) 
  - [Android 源码 Window](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/view/Window.java)
  - [Android 源码 PhoneWindow](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/com/android/internal/policy/PhoneWindow.java)
  - [Android 源码 screen_title.xml](https://android.googlesource.com/platform/frameworks/base/+/master/core/res/res/layout/screen_title.xml)
  - [Android 源码 ViewGroup](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/view/ViewGroup.java)
  - [Android 源码 View](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/view/View.java)
  - [Android事件传递机制分析](http://wuxiaolong.me/2015/12/19/MotionEvent/)
- 参考书籍
  
   - 《Android开发艺术探索》
   - 《Android进阶之光》