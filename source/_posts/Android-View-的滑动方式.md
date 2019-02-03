---
title: Android View 的滑动方式
date: 2019-02-03 13:17:41
categories:  
- Android进阶 #分类
tags: 
- Android
- View体系
- View
---
>前言

自定义View作为Android进阶的基础，是我们开发者不得不学习的知识，而酷炫的自定义View效果，都离不开View的滑动，所以接下来我们来一起探究View的滑动方式，看看View是如何滑动的，为Android进阶的道路打下基础。
<!--more-->
## View 坐标系基本知识
- 了解View的滑动方式，首先我们得了解View在什么位置，我们可以把手机屏幕区域看成是像数学中坐标系一样的区域，只不过是手机屏幕坐标系的Y轴和数学中的坐标系的Y轴正方向相反
- 确定View的位置主要是根据View的left、top、right、bottom四个属性来决定，需要注意的是View的这四个属性是相对于它的父容器来说的，所以对应为left是View的左上角相对于父容器的横坐标，top为纵坐标，right为View右下角相对于父容器的横坐标，bottom为纵坐标。（具体可以看下方示意图A）
```
    //获取view位置的值
    left = View.getLeft();
    top = View.getTop();
    right = View.getRight();
    bottom = View.getBottom();
```

- 除了上面确定View位置的参数，还有x，y，translationX，translationY这四个参数，x和y代表View的左上角的坐标值，而translationX，translationY是左上角坐标相对于View的父容器的偏移量，默认为零，也就是view不移动，则x和y等于left和top，他们换算关系可看下面示意图A，在View的滑动过程中，left和top表示的是View原始位置的值，这是不会改变的，所以改变的是滑动偏移量加上原始值得到新的左上角坐标。
  
![View坐标系和点击事件示意图](https://github.com/maoqitian/MaoMdPhoto/raw/master/Android%20View%E7%9A%84%E6%BB%91%E5%8A%A8%E6%96%B9%E5%BC%8F/View%E5%9D%90%E6%A0%87%E7%B3%BB%E5%92%8C%E7%82%B9%E5%87%BB%E4%BA%8B%E4%BB%B6%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

- 当我们触摸屏幕，则可以通过点击事件来获取当前点击位置的值和相较于手机屏幕左上角的偏移量坐标（如上图B所示）。
  
## Android View 的滑动方式
### layout方法改变View位置滑动View
- 首先我们看看layout()方法源码
```
      @SuppressWarnings({"unchecked"})
      public void layout(int l, int t, int r, int b) {
            //省略部分代码
            .......
        if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
            onLayout(changed, l, t, r, b);
            //省略部分代码
            ........
            }
        }
```
- 了解过自定义View的各位应该都知道，onLayout()是View绘制过程中的一个方法，可以通过它确定View的位置，也就是说我们通过layout()方法可以改变View的位置，下面我们通过onLayout方法做一个可以随意滑动 view的例子
```
       @Override
       public boolean onTouchEvent(MotionEvent event) {
         //获取触屏时候的坐标
        Log.e("毛麒添","getLeft:"+getLeft()+"getTop:"+getTop()+"getRight:"+getRight()+"getBottom:"+getBottom());
         x = event.getRawX();
         y = event.getRawY();
        switch (event.getAction()){
            case MotionEvent.ACTION_DOWN:
                break;
            case MotionEvent.ACTION_MOVE:
                //手指移动偏移量
                int offsetX = (int) (x-lastX);
                int offsetY = (int) (y-lastY);
                layout(getLeft()+offsetX,getTop()+offsetY,getRight()+offsetX,getBottom()+offsetY);
                break;
            case MotionEvent.ACTION_UP:
                Log.e("毛麒添","getLeft:"+getLeft()+"getTop:"+getTop()+"getRight:"+getRight()+"getBottom:"+getBottom());
                break;
        }
        lastX=x;
        lastY=y;
        return super.onTouchEvent(event);
       }
```
- 通过打印left，top，right，bottom数值可以发现layout方法是真实改变了View的位置而不只是View的内容。

<img height=500 width=300  src="https://github.com/maoqitian/MaoMdPhoto/raw/master/Android%20View%E7%9A%84%E6%BB%91%E5%8A%A8%E6%96%B9%E5%BC%8F/layout1.gif">
      
### offsetLeftAndRight()与offsetTopAndBottom() 方法改变View的位置让其滑动
- 修改上面的方法，效果图和onLayout一样，同时offsetLeftAndRight()与offsetTopAndBottom()方法也是真实改变了View的位置而不只是View的内容。
```
     case MotionEvent.ACTION_MOVE:
                //手指移动偏移量
                int offsetX = (int) (x-lastX);
                int offsetY = (int) (y-lastY);
                offsetLeftAndRight(offsetX);
                offsetTopAndBottom(offsetY);
                break;
```
### 使用scrollTo()和scrollBy()滑动View
   
- scrollTo()和scrollBy()是View提供的滑动方法，scrollTo()移动到某个某个点，scrollBy()表示根据传入的偏移量进行移动。先看源码实现
    
```
    /**
     * Set the scrolled position of your view. This will cause a call to
     * {@link #onScrollChanged(int, int, int, int)} and the view will be
     * invalidated.
     * @param x the x position to scroll to
     * @param y the y position to scroll to
     */
     public void scrollTo(int x, int y) {
        if (mScrollX != x || mScrollY != y) {
            int oldX = mScrollX;
            int oldY = mScrollY;
            mScrollX = x;
            mScrollY = y;
            invalidateParentCaches();
            onScrollChanged(mScrollX, mScrollY, oldX, oldY);
            if (!awakenScrollBars()) {
                postInvalidateOnAnimation();
            }
        }
     }
    /**
     * Move the scrolled position of your view. This will cause a call to
     * {@link #onScrollChanged(int, int, int, int)} and the view will be
     * invalidated.
     * @param x the amount of pixels to scroll by horizontally
     * @param y the amount of pixels to scroll by vertically
     */
    public void scrollBy(int x, int y) {
        scrollTo(mScrollX + x, mScrollY + y);
    }

```
- 通过源码我们可以看到scrollBy()的实现其实是调用了scrollTo()方法。这里有个mScrollX和mScrollY的规则我们需要明白：**scrollTo()中mScrollX的值等于view左边缘和view内容左边缘在水平方向的距离，并且当view的左边缘在view的内容左边缘右边时，mScrollX为正，反之为负；同理mScrollY等于view上边缘和view内容上边缘在竖直方向的距离，并且当view的上边缘在view的内容上边缘下边时，mScrollY为正，反之为负**。当View没有使用scrollTo()和scrollBy()进行滑动的时候，mScrollX和mScrollY默认等于零，也就是view的左边缘与内容左边缘重合。
- 根据上面的规则，我们假设将view内容右下滑动，得到下图
    
![mScrollX和mScrollY值的判断](https://github.com/maoqitian/MaoMdPhoto/raw/master/Android%20View%E7%9A%84%E6%BB%91%E5%8A%A8%E6%96%B9%E5%BC%8F/mScrollX%E5%92%8CmScrollY%E5%80%BC%E7%9A%84%E5%88%A4%E6%96%AD.png)
- 结合上面的知识，我们将上面滑动的例子改写一下，如果使用scrollTo()则只是滑动到我们手指滑动偏移量的距离的点，达不到要求，而scrollBy()是在scrollTo()的基础上偏移滑动的位置，正好符合我们自由滑动的要求，并且根据上面的分析mScrollX和mScrollY为负值，则滑动偏移也应该为负值才能达到我们想要的自由滑动效果（这个大家需要自己好好想明白可能才会更加清楚理解）
    
```
       case MotionEvent.ACTION_MOVE:
                //手指移动偏移量
                int offsetX = (int) (x-lastX);
                int offsetY = (int) (y-lastY);
                //滑动方式1
                ((View)getParent()).scrollBy(-offsetX,-offsetY);
                break;
```
- 根据滑动打印的日志我们可以看出，scrollBy()和scrollTo()在滑动的过程中只是改变了View内容的位置，而没有改变初始的left，right，top，bottom的值
     
![view的位置没有发生改变](https://github.com/maoqitian/MaoMdPhoto/raw/master/Android%20View%E7%9A%84%E6%BB%91%E5%8A%A8%E6%96%B9%E5%BC%8F/view%E7%9A%84%E4%BD%8D%E7%BD%AE%E6%B2%A1%E6%9C%89%E5%8F%91%E7%94%9F%E6%94%B9%E5%8F%98.png)
### 使用动画让View滑动
   
#### xml补间动画的方式让View滑动  
- 定义一个xml文件，500ms移动到500,500的位置并保持位置
```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:fillAfter="true"
    android:duration="500">
    <translate android:fromXDelta="0"
               android:fromYDelta="0"
               android:toXDelta="500"
               android:toYDelta="500"/>
</set>
```
- 代码调用
```
        startAnimation(AnimationUtils.loadAnimation(mContext, R.anim.testscroll));
```
- 补间对View的滑动也只是改变了View的显示效果，不会对View的属性做真正的改变，也就是说补间动画也没有真正改变View的位置
      
#### 属性动画让View滑动
     
- 自从Android3.0开始加入了属性动画(了解属性动画可以查看[郭霖大佬博客](https://blog.csdn.net/guolin_blog/article/details/43536355))，属性动画不仅能作用于View产生动画效果，也能作用于其他属性来产生动画效果，可以说属性动画相较于补间动画是非常灵活的，并且属性动画是真正改变View的位置属性。
- 属性动画一般我们使用ObjectAnimator，让View2秒时间水平平移到300位置，并且移动完后我们点击View看还能响应点击事件（如下图所示）
        
```
        ObjectAnimator.ofFloat(testScroll,"translationX",0,300).setDuration(2000).start();

```
<img height=500 width=300  src="https://github.com/maoqitian/MaoMdPhoto/raw/master/Android%20View%E7%9A%84%E6%BB%91%E5%8A%A8%E6%96%B9%E5%BC%8F/objectAnimator.gif">

### 改变布局参数 LayoutParams 滑动View
- 平常我们开发设置View的位置可以在xml中设定，也可以在代码中设置。LayoutParams有一个View的所有布局参数信息，所有我们可以通过设置View的LayoutParams参数的leftMargin和topMargin达到上面自由滑动View的效果。
      
```
       //省略部分代码
       ......
       case MotionEvent.ACTION_MOVE:
                //手指移动偏移量
                int offsetX = (int) (x-lastX);
                int offsetY = (int) (y-lastY);
                //滑动方式5
                moveView(offsetX,offsetY);
                break;
    //省略部分代码
       ......
       
       private void moveView(int offsetX, int offsetY) {
        ViewGroup.MarginLayoutParams layoutParams = (ViewGroup.MarginLayoutParams) getLayoutParams();
        layoutParams.leftMargin = getLeft() + offsetX;
        layoutParams.topMargin = getTop() + offsetY;
        setLayoutParams(layoutParams);
       }
```
- 既然布局参数已经改变，则View的实际位置肯定也已经改变。
      
### Scroller弹性滑动
   
- 首先我们要明白什么是Scroller？
     
- Scroller是弹性滑动的帮助类，它本身并不能实现View的弹性滑动，它必须要配合scrollTo()或scrollBy()和实现View的computeScroll的方法才能实现View的弹性滑动
- Scroller实现弹性滑动的典型例子
```
        Scroller mScroller=new Scroller(context);
        
        public void smoothScrollTo(int desx,int desy){
        int scaleX = (int) getScaleX();
        int scaleY = (int) getScaleY();
        int deltaX = desx-scaleX;
        int deltaY = desy-scaleY;
        //3秒内弹性滑到desx desy 位置
        mScroller.startScroll(scaleX,scaleY,deltaX,deltaY,3000);
        //重新绘制界面 会调用computeScroll方法
        invalidate();
        }

        @Override
        public void computeScroll() {
        super.computeScroll();
        if(mScroller.computeScrollOffset()){//还没滑动到指定位置
            ((View)getParent()).scrollTo(mScroller.getCurrX(),mScroller.getCurrY());
            postInvalidate();
         }
        }
        
        //拿到自定View的实例对象调用smoothScrollTo实现右下方向3秒
        //内到指定位置的弹性滑动
        //为什么是-300 请看scrollTo()或scrollBy()滑动解析
        testScroll.smoothScrollTo(-300,-300);
```
#### Scroller弹性滑动源码分析
- 下面我们从源码角度来分析一下Scroller是如何实现弹性滑动的
     
- 先看startScroll()方法
         
```
        /**
         * Start scrolling by providing a starting point, the distance to travel,
         * and the duration of the scroll.
         * 
         * @param startX Starting horizontal scroll offset in pixels. Positive
         *        numbers will scroll the content to the left.
         * @param startY Starting vertical scroll offset in pixels. Positive numbers
         *        will scroll the content up.
         * @param dx Horizontal distance to travel. Positive numbers will scroll the
         *        content to the left.
         * @param dy Vertical distance to travel. Positive numbers will scroll the
         *        content up.
         * @param duration Duration of the scroll in milliseconds.
         */
         public void startScroll(int startX, int startY, int dx, int dy, int duration) {
         mMode = SCROLL_MODE;
         mFinished = false;
         mDuration = duration;
         mStartTime = AnimationUtils.currentAnimationTimeMillis();
         mStartX = startX;
         mStartY = startY;
         mFinalX = startX + dx;
         mFinalY = startY + dy;
         mDeltaX = dx;
         mDeltaY = dy;
         mDurationReciprocal = 1.0f / (float) mDuration;
         }
         
         //View 中 computeScroll()方法没有实现内容，需要子View 自行实现
           /**
            * Called by a parent to request that a child update its values for mScrollX
            * and mScrollY if necessary. This will typically be done if the child is
            * animating a scroll using a {@link android.widget.Scroller Scroller}
            * object.
            */
            public void computeScroll() {
            }
```
- 通过源码我们看到startScroll()方法只是传递了我们传入的参数，滑动的起点startX、startY，滑动的距离dx、dy，和弹性滑动的时间，没看到有滑动的操作，那Scroller是如何让View滑动呢？而答案就是我们再调用startScroll()方法之后又调用了invalidate()方法，该方法会引起view的重绘，而View的重绘会调用computeScroll()方法，通过上面的源码，我们知道computeScroll()方法在view中是空实现，所以我们自己实现该放法的时候则调用scrollTo方法获取scrollX和scrollY当前让view进行滑动，但是这只是滑动一段距离，好像还没有弹性滑动，别急，我们看看Scroller的computeScrollOffset()方法
       
```
        /**
         * Call this when you want to know the new location.  If it returns true,
         * the animation is not yet finished.
         */ 
         public boolean computeScrollOffset() {
          if (mFinished) {
            return false;
           }
          int timePassed = (int)(AnimationUtils.currentAnimationTimeMillis() - mStartTime);
    
          if (timePassed < mDuration) {
            switch (mMode) {
            case SCROLL_MODE:
                final float x = mInterpolator.getInterpolation(timePassed * mDurationReciprocal);
                mCurrX = mStartX + Math.round(x * mDeltaX);
                mCurrY = mStartY + Math.round(x * mDeltaY);
                break;
                .......
              }
          }
          else {
            .......
          }
         return true;
        }
```
- 通过computeScrollOffset()的源码，我们已经可以一目了然，根据时间流逝的百分比算出scrollX和scrollY改变的百分比并计算出他们的值，类似动画中的插值器的概念。每次重绘缓慢滑动一段距离，在一段时间内缓慢滑动就成了弹性滑动，就比scrollTo方法的一下滑动完舒服多了，我们还需要注意computeScrollOffset()的返回值，如果返回false表示滑动完了，true则表示没有滑动完。
- 这里我们梳理一下Scroller实现弹性滑动的工作原理：Scroller必须要配合scrollTo()或scrollBy()和实现View的computeScroll的方法才能实现View的弹性滑动，invalidate()引发第一次重绘，重绘距离滑动开始时间有一个时间间隔，在这个时间间隔中获取View滑动的位置，通过scrollTo()进行滑动，滑动完postInvalidate()再次进行重绘，没有滑动完则继续上面的操作，最终组成弹性滑动。
## 最后说点      
>到此，View的滑动方式就已经了解完了。如果文章中有写得不对的地方，请给我留言指出，大家一起学习进步。如果觉得我的文章给予你帮助，也请给我一个喜欢和关注。

- [测试Demo地址](https://github.com/maoqitian/MyPracticeView/blob/master/app/src/main/java/mao/com/mycustomview/view/scroll/TestScroll.java)

- 参考链接
 
  -  [Android 源码View](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/view/View.java)
  -  [Android 源码Scroller](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/widget/Scroller.java)

- 参考书籍
  
   - 《Android开发艺术探索》 
   - 《Android进阶之光》