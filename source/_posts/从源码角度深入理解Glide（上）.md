---
title: 从源码角度深入理解Glide（上）
date: 2019-02-19 21:26:23
categories:
- Android热门框架解析 #分类
tags:
- Glide
- 图片加载
- Android
- 源码分析
---
![image](https://raw.githubusercontent.com/bumptech/glide/master/static/glide_logo.png)
> 谈到Glide，从英文字面意思有滑行、滑动的意思；而Android从开发的角度我们知道它是一款图片加载框架，这里引用官方文档的一句话“Glide是一个快速高效的Android图片加载库，注重于平滑的滚动”，从官方文档介绍我们了解到用Glide框架来加载图片是快速并且高效的，接下来就来通过简单使用Glide和源码理解两个方面看看Glide是否是快速和高效（文中代码基于Glide 4.8版本）。
<!--more-->
## Glide简单使用
- 1.使用前需要添加依赖
    ```
    implementation 'com.github.bumptech.glide:glide:4.8.0'
    //使用Generated API需要引入 
    annotationProcessor 'com.github.bumptech.glide:compiler:4.8.0'
    ```
- 2.简单加载网络图片到ImageView，可以看到简单一句代码就能将网络图片加载到ImageView，也可以使用Generated API方式
    
    ```
    //直接使用
    Glide.with(Context).load(IMAGE_URL).into(mImageView)
    
    //使用Generated API， 作用范围Application 模块内使用
    //创建MyAppGlideModule类加上@GlideModule注解，make project 就能使用 GlideApp
    @GlideModule
    public final class MyAppGlideModule extends AppGlideModule {}
    
    //Generated API加载图片
    GlideApp.with(Context).load(IMAGE_URL).into(mImageView);
    ```
- 3.当加载网络图片的时候，网络请求是耗时操作，所以图片不可能马上就加载出来，网络请求这段时间ImageView是空白的，所以我们可以使用一个占位符显示图片来优化用户体验，占位符有三种
    - 加载占位符（placeholder）
    - 错误占位符（error）
    - 后备回调符（Fallback）
   ```
   //添加占位图
        RequestOptions requestOptions = new RequestOptions()
            .placeholder(R.drawable.ic_cloud_download_black_24dp)
            .error(R.drawable.ic_error_black_24dp)
            .diskCacheStrategy(DiskCacheStrategy.NONE);//不使用缓存
        Glide.with(Context).load(IMAGE_URL).apply(requestOptions).into(mImageView);
   
   //Generated API 方式(和Glide3 一样)
   GlideApp.with(Context).load(IMAGE_URL)
            .placeholder(R.drawable.ic_cloud_download_black_24dp)
            .error(R.drawable.ic_error_black_24dp)
            .diskCacheStrategy(DiskCacheStrategy.NONE)
            .into(mImageView);
            
   // 后备回调符(Fallback) Generated API 方式才有，在应用设置用户头像场景中，如果用户不设置，也就是为null的情况，可以使用后备回调符显示默认头像
   private static final String NULL_URL=null;
   GlideApp.with(Context).load(NULL_URL)
            .fallback(R.drawable.ic_account_circle_black_24dp)
            .into(mImageView);
   ```
    <img src="https://github.com/maoqitian/MaoMdPhoto/raw/master/%E4%BB%8E%E6%BA%90%E7%A0%81%E8%A7%92%E5%BA%A6%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Glide/%E6%98%BE%E7%A4%BA%E5%8D%A0%E4%BD%8D%E5%9B%BE.gif"  height="400" width="230">
- 4.指定加载图片的大小（override）
   
   ```
   RequestOptions requestOptions = new RequestOptions().override(200,100);
   Glide.with(Context).load(IMAGE_URL).apply(requestOptions).into(mImageView);
   
   //Generated API 方式
   GlideApp.with(Context).load(IMAGE_URL)
                .override(200,100)
                .into(mImageView);
   ```
- 5.缩略图 (Thumbnail)
     - 这个其实和占位符（placeholder）有些相似，但是占位符只能加载本地资源，而缩略图可以加载网络资源，thumbnail方法与我们的主动加载并行运行，如果主动加载已经完成，则缩略图不会显示
     
    ```
    //缩略图Options
    RequestOptions requestOptions = new RequestOptions()
            .override(200,100)
            .diskCacheStrategy(DiskCacheStrategy.NONE);
     Glide.with(Context)
                .load(IMAGE_URL)
                .thumbnail( Glide.with(this)
                .load(IMAGE_URL)
                .apply(requestOptions))
                .into(mImageView);
    //Generated API 方式
     GlideApp.with(Context).
                load(IMAGE_URL).
                thumbnail( GlideApp.with(this)
                .load(IMAGE_URL).override(200,100)
            .diskCacheStrategy(DiskCacheStrategy.NONE)).into(mImageView);
    ```
- 6.图像变化
  - Glide中内置了三种图片的变化操作，分别是CenterCrop（图片原图的中心区域进行裁剪显示），FitCenter（图片原始长宽铺满）和CircleCrop（圆形裁剪）
   
    ```
    //显示圆形裁剪到ImageView
    RequestOptions requestOptions = new RequestOptions()
                .circleCrop()
                .diskCacheStrategy(DiskCacheStrategy.NONE);

    Glide.with(Context)
                .load(IMAGE_URL)
                .apply(requestOptions)
                .into(mImageView);
                
    //RequestOptions都内置了使用者三种变化的静态方法
    Glide.with(Context)
                .load(IMAGE_URL)
                .apply(RequestOptions.circleCropTransform())
                .into(mImageView);
                
    //Generated API 方式
    GlideApp.with(Context).load(IMAGE_URL)
                .circleCrop()
                .diskCacheStrategy(DiskCacheStrategy.NONE)
                .into(mImageView);

    ```
  - 如果想要更酷炫的变化，可以使用第三方框架[glide-transformations](https://github.com/wasabeef/glide-transformations)来帮助我们实现，并且变化是可以组合的
    
    ```
    //第三方框架glide-transformations引入
    implementation 'jp.wasabeef:glide-transformations:4.0.0'
    
    //使用glide-transformations框架 变换图片颜色和加入模糊效果
     RequestOptions requestOptions=new RequestOptions()
               .placeholder(R.drawable.ic_cloud_download_black_24dp)
               .transforms(new ColorFilterTransformation(Color.argb(80, 255, 0, 0)),new BlurTransformation(30))
               .diskCacheStrategy(DiskCacheStrategy.NONE);
      
      Glide.with(Context).load(IMAGE_URL).
               apply(requestOptions).
               into(mImageView);

      //Generated API 方式
      GlideApp.with(Context).load(IMAGE_URL)
               .transforms(new ColorFilterTransformation(Color.argb(80, 255, 0, 0)),new BlurTransformation(30))
               .placeholder(R.drawable.ic_cloud_download_black_24dp)
               .diskCacheStrategy(DiskCacheStrategy.NONE)
               .into(mImageView);
              
    ```
    ![glide_transformation](https://github.com/maoqitian/MaoMdPhoto/raw/master/%E4%BB%8E%E6%BA%90%E7%A0%81%E8%A7%92%E5%BA%A6%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Glide/glide_transformation.gif)
  - 更多效果可以查看[官方例子](https://github.com/wasabeef/glide-transformations/blob/master/example/src/main/java/jp/wasabeef/example/glide/MainAdapter.kt) 
   
- 7.加载目标（Target）
  - Target是介于请求和请求者之间的中介者的角色，into方法的返回值就是target对象，之前我们一直使用的 into(ImageView) ，它其实是一个辅助方法，它接受一个 ImageView 参数并为其请求的资源类型包装了一个合适的 ImageViewTarget
  
    ```
    //加载
    Target<Drawable> target = 
     Glide.with(Context)
    .load(url)
    .into(new Target<Drawable>() {
      ...
    });
    
    //清除加载
    Glide.with(Context).clear(target);
    ```
   - 当我们使用Notification显示应用通知，如果想要自定义通知的界面，我们需要用到RemoteView，如果要给RemoteView设置ImageView，根据提供的setImageViewBitmap方法，如果通知界面需要加载网络图片，则需要将网络图片转换成bitmap，一般我们可以根据获取图片链接的流来转换成bitmap，或者使用本文的主题使用Glide框架，这些都是耗时操作，感觉操作起来很麻烦，而Glide框架很贴心的给我提供了NotificationTarget(继承SimpleTarget)，相对于我们加载目标变成Notification
   
   ```
   /**
    * 新建 NotificationTarget 对象参数说明，与Glide3不同，Glide4的asBitmap()方法必须在load方法前面
    * @param context 上下文对象          
    * @param viewId 需要加载ImageView的view的 id        
    * @param remoteViews RemoteView对象   
    * @param notification   Notification对象
    * @param notificationId Notification Id
    */
   String iamgeUrl = "http://p1.music.126.net/fX0HfPMAHJ2L_UeJWsL7ig==/18853325881511874.jpg?param=130y130";     
    
   NotificationTarget notificationTarget = new NotificationTarget(mContext,R.id.notification_Image_play,mRemoteViews,mNotification,notifyId);
   Glide.with(mContext.getApplicationContext())
                .asBitmap()
                .load(iamgeUrl)
                .into( notificationTarget );
                
   //Generated API 方式            
   GlideApp.with(mContext.getApplicationContext())
                .asBitmap()
                .load(iamgeUrl)
                .into( notificationTarget );
   ```
    ![glide_notification](https://github.com/maoqitian/MaoMdPhoto/raw/master/%E4%BB%8E%E6%BA%90%E7%A0%81%E8%A7%92%E5%BA%A6%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Glide/glide_notification.gif)
- 8.回调监听
  - 使用Glide加载图片，虽然在加载中或者加失败都有占位符方法处理，但是我们还是希望可以知道图片到底是加载成功还是失败，Glide也给我们提供了监听方法来知道图片到底是加载成功还是失败，结合listener和into方法来使用回调
  
  ```
  Glide.with(this).load(IMAGE_URL).
                listener(new RequestListener<Drawable>() {
                    @Override
                    public boolean onLoadFailed(@Nullable GlideException e, Object model, Target<Drawable> target, boolean isFirstResource) {
                   Toast.makeText(getApplicationContext(),"图片加载失败",Toast.LENGTH_SHORT).show();
                        return false;
                    }

                    @Override
                    public boolean onResourceReady(Drawable resource, Object model, Target<Drawable> target, DataSource dataSource, boolean isFirstResource) {
                   Toast.makeText(getApplicationContext(),"图片加载成功",Toast.LENGTH_SHORT).show();
                        return false;
                    }
                }).into(mImageView);*/
        //Generated API 方式
        GlideApp.with(this).load(IMAGE_URL)
                .listener(new RequestListener<Drawable>() {
                    @Override
                    public boolean onLoadFailed(@Nullable GlideException e, Object model, Target<Drawable> target, boolean isFirstResource) {
                        Toast.makeText(getApplicationContext(),"图片加载失败",Toast.LENGTH_SHORT).show();
                        return false;
                    }

                    @Override
                    public boolean onResourceReady(Drawable resource, Object model, Target<Drawable> target, DataSource dataSource, boolean isFirstResource) {
                        Toast.makeText(getApplicationContext(),"图片加载成功",Toast.LENGTH_SHORT).show();
                        return false;
                    }
                }).into(mImageView);
  ```
  - 可以看到监听实现的方法都有布尔类型的返回值，返回true，则代表处理了该回调事件，false则不进行处理，如果onResourceReady方法返回true，则into方法就不会执行，也就是图片不会加载到ImageView，同理onLoadFailed方法返回true，则error方法不会执行。
  
    
> Glide还有其他的一些使用方法，这里就不继续展开了，有兴趣的可以自行继续研究。

## Glide源码解析

### Glide加载图片到ImageView基本流程图
![Glide加载基本流程图](https://github.com/maoqitian/MaoMdPhoto/raw/master/%E4%BB%8E%E6%BA%90%E7%A0%81%E8%A7%92%E5%BA%A6%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Glide/Glide%E5%9F%BA%E6%9C%AC%E8%AF%B7%E6%B1%82%E6%B5%81%E7%A8%8B%E5%9B%BE.jpg)
### Glide加载图片到ImageView源码分析
- 在上一节简单的列出了一些Glide的使用方法，能用不代表你已经懂了，接下来就通过理解源码的方式来对Glide是如何工作的做深一层次理解，首先从最简单使用开始

```
Glide.with(Context).load(IMAGE_URL).into(mImageView);
```
#### with方法
- 来吧，开始是Glide的with()方法，直接上源码

```
/** Glide类的with()方法*/
@NonNull
  public static RequestManager with(@NonNull Context context) {
    return getRetriever(context).get(context);
  }

  @NonNull
  public static RequestManager with(@NonNull Activity activity) {
    return getRetriever(activity).get(activity);
  }

  @NonNull
  public static RequestManager with(@NonNull FragmentActivity activity) {
    return getRetriever(activity).get(activity);
  }

  @NonNull
  public static RequestManager with(@NonNull Fragment fragment) {
    return getRetriever(fragment.getActivity()).get(fragment);
  }

  @SuppressWarnings("deprecation")
  @Deprecated
  @NonNull
  public static RequestManager with(@NonNull android.app.Fragment fragment) {
    return getRetriever(fragment.getActivity()).get(fragment);
  }

  @NonNull
  public static RequestManager with(@NonNull View view) {
    return getRetriever(view.getContext()).get(view);
  }
```
- 通过源码，可以看到with有不同参数类型的重载方法，每个方法首先都是调用 getRetriever()方法

```
 /** Glide类的getRetriever()方法*/
 private static RequestManagerRetriever getRetriever(@Nullable Context context) {
    // Context could be null for other reasons (ie the user passes in null), but in practice it will
    // only occur due to errors with the Fragment lifecycle.
    Preconditions.checkNotNull(
        context,
        "You cannot start a load on a not yet attached View or a Fragment where getActivity() "
            + "returns null (which usually occurs when getActivity() is called before the Fragment "
            + "is attached or after the Fragment is destroyed).");
    return Glide.get(context).getRequestManagerRetriever();
  }
```
- Glide的get方法中通过new GlideBuilder()获取了Glide对象，并通过Glide的getRequestManagerRetriever()的方法最终得到RequestManagerRetriever对象，接下来我们看看RequestManagerRetriever对象的get方法
```
/** RequestManagerRetriever类的get()方法*/
 @NonNull
  public RequestManager get(@NonNull Context context) {
    if (context == null) {
      throw new IllegalArgumentException("You cannot start a load on a null Context");
    } else if (Util.isOnMainThread() && !(context instanceof Application)) {
      if (context instanceof FragmentActivity) {
        return get((FragmentActivity) context);
      } else if (context instanceof Activity) {
        return get((Activity) context);
      } else if (context instanceof ContextWrapper) {
        return get(((ContextWrapper) context).getBaseContext());
      }
    }
    return getApplicationManager(context);
  }

  @NonNull
  public RequestManager get(@NonNull FragmentActivity activity) {
    if (Util.isOnBackgroundThread()) {
      return get(activity.getApplicationContext());
    } else {
      assertNotDestroyed(activity);
      FragmentManager fm = activity.getSupportFragmentManager();
      return supportFragmentGet(
          activity, fm, /*parentHint=*/ null, isActivityVisible(activity));
    }
  }

  @NonNull
  public RequestManager get(@NonNull Fragment fragment) {
    Preconditions.checkNotNull(fragment.getActivity(),
          "You cannot start a load on a fragment before it is attached or after it is destroyed");
    if (Util.isOnBackgroundThread()) {
      return get(fragment.getActivity().getApplicationContext());
    } else {
      FragmentManager fm = fragment.getChildFragmentManager();
      return supportFragmentGet(fragment.getActivity(), fm, fragment, fragment.isVisible());
    }
  }

  @SuppressWarnings("deprecation")
  @NonNull
  public RequestManager get(@NonNull Activity activity) {
    if (Util.isOnBackgroundThread()) {
      return get(activity.getApplicationContext());
    } else {
      assertNotDestroyed(activity);
      android.app.FragmentManager fm = activity.getFragmentManager();
      return fragmentGet(
          activity, fm, /*parentHint=*/ null, isActivityVisible(activity));
    }
  }

  @SuppressWarnings("deprecation")
  @NonNull
  public RequestManager get(@NonNull View view) {
    if (Util.isOnBackgroundThread()) {
      return get(view.getContext().getApplicationContext());
    }
    Preconditions.checkNotNull(view);
    Preconditions.checkNotNull(view.getContext(),
        "Unable to obtain a request manager for a view without a Context");
    Activity activity = findActivity(view.getContext());
    // The view might be somewhere else, like a service.
    if (activity == null) {
      return get(view.getContext().getApplicationContext());
    }

    // Support Fragments.
    // Although the user might have non-support Fragments attached to FragmentActivity, searching
    // for non-support Fragments is so expensive pre O and that should be rare enough that we
    // prefer to just fall back to the Activity directly.
    if (activity instanceof FragmentActivity) {
      Fragment fragment = findSupportFragment(view, (FragmentActivity) activity);
      return fragment != null ? get(fragment) : get(activity);
    }

    // Standard Fragments.
    android.app.Fragment fragment = findFragment(view, activity);
    if (fragment == null) {
      return get(activity);
    }
    return get(fragment);
  }
```
- 同样，RequestManagerRetriever对象的get方法也有不同类型参数的重载，分别针对Application、Activity、Fragmenet、view做了不同的处理，先看Context参数的get方法，**在该方法中它把Context的参数分成了两个类型，一个Application类型的Context，另一个是非Application类型的Context**。如果是Application类型的Context，则创建的Glide的生命周期则跟随ApplicationContext的生命周期，也就是下面的getApplicationManager所做的事情。

```
/** RequestManagerRetriever类的getApplicationManager()方法*/
  @NonNull
  private RequestManager getApplicationManager(@NonNull Context context) {
    // Either an application context or we're on a background thread.
    if (applicationManager == null) {
      synchronized (this) {
        if (applicationManager == null) {
          // Normally pause/resume is taken care of by the fragment we add to the fragment or
          // activity. However, in this case since the manager attached to the application will not
          // receive lifecycle events, we must force the manager to start resumed using
          // ApplicationLifecycle.

          // TODO(b/27524013): Factor out this Glide.get() call.
          Glide glide = Glide.get(context.getApplicationContext());
          applicationManager =
              factory.build(
                  glide,
                  new ApplicationLifecycle(),
                  new EmptyRequestManagerTreeNode(),
                  context.getApplicationContext());
        }
      }
    }
    return applicationManager;
  }
```
- 接着，如果是非Application类型的，Activity、Fragmenet属于非Application；如果是Activity类型的Context，当前不再主线程，则继续跟随Application生命周期，否则给当前Activity添加一个隐藏的Fragment，然后Glide生命周期跟随这个隐藏的Fragment，分析到这里，我们再看Fragmenet类型的Context，或者是View类型，也是添加了一个隐藏的Fragment。这是为什么呢？首先Fragment的生命周期是和Activity同步的，Activity销毁Fragment也会销毁，其次，这也方便Glide知道自己什么时候需要停止加载，如果我们打开一个Activity并关闭它，如果Glide生命周期跟随Application，则Activity虽然已经销毁，但是应用还没退出，则Glide还在继续加载图片，这显然是不合理的，而Glide很巧妙的用一个隐藏Fragment来解决生命周期的监听。

```
/** RequestManagerRetriever类的fragmentGet()方法*/
  @SuppressWarnings({"deprecation", "DeprecatedIsStillUsed"})
  @Deprecated
  @NonNull
  private RequestManager fragmentGet(@NonNull Context context,
      @NonNull android.app.FragmentManager fm,
      @Nullable android.app.Fragment parentHint,
      boolean isParentVisible) {
    RequestManagerFragment current = getRequestManagerFragment(fm, parentHint, isParentVisible);
    RequestManager requestManager = current.getRequestManager();
    if (requestManager == null) {
      // TODO(b/27524013): Factor out this Glide.get() call.
      Glide glide = Glide.get(context);
      requestManager =
          factory.build(
              glide, current.getGlideLifecycle(), current.getRequestManagerTreeNode(), context);
      current.setRequestManager(requestManager);
    }
    return requestManager;
  }
  
  /** RequestManagerRetriever类的getRequestManagerFragment()方法*/
  @SuppressWarnings("deprecation")
  @NonNull
  private RequestManagerFragment getRequestManagerFragment(
      @NonNull final android.app.FragmentManager fm,
      @Nullable android.app.Fragment parentHint,
      boolean isParentVisible) {
    RequestManagerFragment current = (RequestManagerFragment) fm.findFragmentByTag(FRAGMENT_TAG);
    if (current == null) {
      current = pendingRequestManagerFragments.get(fm);
      if (current == null) {
        current = new RequestManagerFragment();
        current.setParentFragmentHint(parentHint);
        if (isParentVisible) {
          current.getGlideLifecycle().onStart();
        }
        pendingRequestManagerFragments.put(fm, current);
        fm.beginTransaction().add(current, FRAGMENT_TAG).commitAllowingStateLoss();
        handler.obtainMessage(ID_REMOVE_FRAGMENT_MANAGER, fm).sendToTarget();
      }
    }
    return current;
  }
```
- 经过对into方法的分析，最终获取的是跟随对应Context对象生命周期的**RequestManager对象**。

#### load方法 
- 经过上一小节的分析，Glide.with方法最终获取的是RequestManager对象，所以继续看RequestManager对象里面load方法，

```
/** RequestManager 类的as()方法*/
 @NonNull
  @CheckResult
  public <ResourceType> RequestBuilder<ResourceType> as(
      @NonNull Class<ResourceType> resourceClass) {
    return new RequestBuilder<>(glide, this, resourceClass, context);
  }
/** RequestManager 类的as()方法*/
 @NonNull
  @CheckResult
  public RequestBuilder<Drawable> asDrawable() {
    return as(Drawable.class);
  }
/** RequestManager 类的部分load()方法*/
 @NonNull
  @CheckResult
  @Override
  public RequestBuilder<Drawable> load(@Nullable Bitmap bitmap) {
    return asDrawable().load(bitmap);
  }
  @NonNull
  @CheckResult
  @Override
  public RequestBuilder<Drawable> load(@Nullable Drawable drawable) {
    return asDrawable().load(drawable);
  }

  @NonNull
  @CheckResult
  @Override
  public RequestBuilder<Drawable> load(@Nullable String string) {
    return asDrawable().load(string);
  }
  //省略其他参数类型 load() 方法
 .......
```
- 通过以上load方法，可以发现虽然RequestManager对象的load方法有多个类型参数的重载，但是不管load方法传递什么类型参数，该方法都是调用RequestBuilder对象的load方法

```
/** RequestBuilder 类的load()方法*/
 @NonNull
  @CheckResult
  @SuppressWarnings("unchecked")
  @Override
  public RequestBuilder<TranscodeType> load(@Nullable Object model) {
    return loadGeneric(model);
  }
/** RequestBuilder对象 类的loadGeneric()方法*/
  @NonNull
  private RequestBuilder<TranscodeType> loadGeneric(@Nullable Object model) {
    this.model = model;
    isModelSet = true;
    return this;
  }
```
- 通过以上RequestBuilder对象的load()方法，我们可以明白不管RequestManager对象的load方法方法传递什么类型的加载资源参数，RequestBuilder对象都把它看成时Object对象，并在loadGeneric方法中赋值给RequestBuilder对象的model对象。
- 通过查看RequestBuilder对象，我们还注意到apply(RequestOptions)这个方法，前面我们的例子中使用缓存，加载图像大小，设置加载占位符和错误占位符都需要新建RequestOptions对象，并设置我们的配置，现在我们分析的加载并没有apply一个RequestOptions对象，则Glide会使用requestOptions.clone()去加载默认配置，这里就先不进行展开了，先继续关注接下来的into方法。

```
/** RequestBuilder 类的apply方法*/
 @NonNull
  @CheckResult
  public RequestBuilder<TranscodeType> apply(@NonNull RequestOptions requestOptions) {
    Preconditions.checkNotNull(requestOptions);
    this.requestOptions = getMutableOptions().apply(requestOptions);
    return this;
  }
 
  @SuppressWarnings("ReferenceEquality")
  @NonNull
  protected RequestOptions getMutableOptions() {
    return defaultRequestOptions == this.requestOptions
        ? this.requestOptions.clone() : this.requestOptions;
  }
```
- 经过以上对with()方法和load方法的分析，经过这两步之后得到了**RequestBuilder**对象，也就说明真正的图片加载操作是在into方法来完成，也就是RequestBuilder对象的into方法。
#### into方法
- 通过上一小节的分析，经过load方法之后获取的对象是RequestBuilder，并且我们将load方法的参数赋值给了RequestBuilder对象的model参数，接下来就到了Glide最核心的方法，也就是RequestBuilder对象的into方法
##### 获取DrawableImageViewTarget
```
/** RequestBuilder 类的into方法*/
@NonNull
  public ViewTarget<ImageView, TranscodeType> into(@NonNull ImageView view) {
    Util.assertMainThread();
    Preconditions.checkNotNull(view);
    RequestOptions requestOptions = this.requestOptions;
    if (!requestOptions.isTransformationSet()
        && requestOptions.isTransformationAllowed()
        && view.getScaleType() != null) {
      // Clone in this method so that if we use this RequestBuilder to load into a View and then
      // into a different target, we don't retain the transformation applied based on the previous
      // View's scale type.
      switch (view.getScaleType()) {
        case CENTER_CROP:
          requestOptions = requestOptions.clone().optionalCenterCrop();
          break;
        case CENTER_INSIDE:
          requestOptions = requestOptions.clone().optionalCenterInside();
          break;
        case FIT_CENTER:
        case FIT_START:
        case FIT_END:
          requestOptions = requestOptions.clone().optionalFitCenter();
          break;
        case FIT_XY:
          requestOptions = requestOptions.clone().optionalCenterInside();
          break;
        case CENTER:
        case MATRIX:
        default:
          // Do nothing.
      }
    }

    return into(
        glideContext.buildImageViewTarget(view, transcodeClass),
        /*targetListener=*/ null,
        requestOptions);
  }
```
- RequestBuilder 对象的into方法中首先获取传递进来的ImageView的ScaleType，让Glide加载出来的ImageView保持一样的ScaleType变化，然后我们看到最后一句话，该方法返回了RequestBuilder 对象的另一个into方法，先看glideContext.buildImageViewTarget()做了什么操作
```
/** GlideContext 类的 buildImageViewTarget方法*/
  @NonNull
  public <X> ViewTarget<ImageView, X> buildImageViewTarget(
      @NonNull ImageView imageView, @NonNull Class<X> transcodeClass) {
    return imageViewTargetFactory.buildTarget(imageView, transcodeClass);
  }
  public class ImageViewTargetFactory {
  @NonNull
  @SuppressWarnings("unchecked")
  public <Z> ViewTarget<ImageView, Z> buildTarget(@NonNull ImageView view,
      @NonNull Class<Z> clazz) {
    if (Bitmap.class.equals(clazz)) {
      return (ViewTarget<ImageView, Z>) new BitmapImageViewTarget(view);
    } else if (Drawable.class.isAssignableFrom(clazz)) {
      return (ViewTarget<ImageView, Z>) new DrawableImageViewTarget(view);
    } else {
      //省略代码
      .....
    }
  }
}
```
- 通过以上源码，之前我们看RequestBuilder源码中as方法传入的是Drawable.class，所以以上的buildImageViewTarget方法最终返回的是**DrawableImageViewTarget对象**，接着我们继续看第一步into方法返回into方法中做了什么操作
##### 构建Request
```
/** RequestBuilder 类的into方法返回的into方法*/
 private <Y extends Target<TranscodeType>> Y into(
      @NonNull Y target,
      @Nullable RequestListener<TranscodeType> targetListener,
      @NonNull RequestOptions options) {
    Util.assertMainThread();
    Preconditions.checkNotNull(target);
    if (!isModelSet) {
      throw new IllegalArgumentException("You must call #load() before calling #into()");
    }
    options = options.autoClone();
    Request request = buildRequest(target, targetListener, options);
    //省略部分代码
    ......
    requestManager.clear(target);
    target.setRequest(request);
    requestManager.track(target, request);
    return target;
  }
```
- 通过以上源码，我们应该先明白**Request类是一个接口，他抽象了Glide加载图片请求**(Request类源码这里就不贴了)，它是一个非常重要的类，这里我们先看看buildRequest(target, targetListener, options)方法是如何创建Request对象的
```
 private Request buildRequest(
      Target<TranscodeType> target,
      @Nullable RequestListener<TranscodeType> targetListener,
      RequestOptions requestOptions) {
    return buildRequestRecursive(
        target,
        targetListener,
        /*parentCoordinator=*/ null,
        transitionOptions,
        requestOptions.getPriority(),
        requestOptions.getOverrideWidth(),
        requestOptions.getOverrideHeight(),
        requestOptions);
  }
private Request buildRequestRecursive(
      Target<TranscodeType> target,
      @Nullable RequestListener<TranscodeType> targetListener,
      @Nullable RequestCoordinator parentCoordinator,
      TransitionOptions<?, ? super TranscodeType> transitionOptions,
      Priority priority,
      int overrideWidth,
      int overrideHeight,
      RequestOptions requestOptions) {
   //省略部分代码 error Request build
    ..... 
    Request mainRequest =
        buildThumbnailRequestRecursive(
            target,
            targetListener,
            parentCoordinator,
            transitionOptions,
            priority,
            overrideWidth,
            overrideHeight,
            requestOptions);

    if (errorRequestCoordinator == null) {
      return mainRequest;
    }
    //省略部分代码 error Request build
    ..... 
  }
  
  private Request buildThumbnailRequestRecursive(
      Target<TranscodeType> target,
      RequestListener<TranscodeType> targetListener,
      @Nullable RequestCoordinator parentCoordinator,
      TransitionOptions<?, ? super TranscodeType> transitionOptions,
      Priority priority,
      int overrideWidth,
      int overrideHeight,
      RequestOptions requestOptions) {
    if (thumbnailBuilder != null) {
      //省略部分代码 缩略图操作
    .....
    } else {
      // Base case: no thumbnail.
      return obtainRequest(
          target,
          targetListener,
          requestOptions,
          parentCoordinator,
          transitionOptions,
          priority,
          overrideWidth,
          overrideHeight);
    }
  }

  private Request obtainRequest(
      Target<TranscodeType> target,
      RequestListener<TranscodeType> targetListener,
      RequestOptions requestOptions,
      RequestCoordinator requestCoordinator,
      TransitionOptions<?, ? super TranscodeType> transitionOptions,
      Priority priority,
      int overrideWidth,
      int overrideHeight) {
    return SingleRequest.obtain(
        context,
        glideContext,
        model,
        transcodeClass,
        requestOptions,
        overrideWidth,
        overrideHeight,
        priority,
        target,
        targetListener,
        requestListeners,
        requestCoordinator,
        glideContext.getEngine(),
        transitionOptions.getTransitionFactory());
  }
```
- 通过上面的源码，我们可以看到buildRequest方法调用了buildRequestRecursive方法，在buildRequestRecursive方法中大部分代码都在处理缩略图（thumbnail），我们主流程中没有设置缩略图，这里就不进行展开分析，接着buildRequestRecursive方法又调用了obtainRequest方法，obtainRequest方法传递了非常多参数，比如有我们熟悉的RequestOptions，设置图片尺寸的 overrideWidth， overrideHeight，还有第一步into方法中的target对象，也就是**DrawableImageViewTarget对象**，model也就是我们load传入的图片地址，也就说明不管load方法还是apply方法传入的参数最终都给到了这里传入SingleRequest.obtain方法，我们继续看看SingleRequest类
```
public final class SingleRequest<R> implements Request,
    SizeReadyCallback,
    ResourceCallback,
    FactoryPools.Poolable {
//省略部分代码
......
/**SingleRequest类的 obtain方法*/
 public static <R> SingleRequest<R> obtain(
      Context context,
      GlideContext glideContext,
      Object model,
      Class<R> transcodeClass,
      RequestOptions requestOptions,
      int overrideWidth,
      int overrideHeight,
      Priority priority,
      Target<R> target,
      RequestListener<R> targetListener,
      @Nullable List<RequestListener<R>> requestListeners,
      RequestCoordinator requestCoordinator,
      Engine engine,
      TransitionFactory<? super R> animationFactory) {
    @SuppressWarnings("unchecked") SingleRequest<R> request =
        (SingleRequest<R>) POOL.acquire();
    if (request == null) {
      request = new SingleRequest<>();
    }
    request.init(
        context,
        glideContext,
        model,
        transcodeClass,
        requestOptions,
        overrideWidth,
        overrideHeight,
        priority,
        target,
        targetListener,
        requestListeners,
        requestCoordinator,
        engine,
        animationFactory);
    return request;
  }
  //省略部分代码
  ......
}  
```
- 通过SingleRequest对象的obtain方法，我们可以看到request = new SingleRequest<>()；也就是最终我们构建的Request是**SingleRequest**对象，并在init方法中将上一步obtainRequest方法传递进来的各种参数进行赋值。
##### Request执行
- 构建完成Request对象，接下来继续看刚刚的into方法下面的操作
```
requestManager.clear(target);
target.setRequest(request);
requestManager.track(target, request);
return target
```
- 首先RequestManager对象清除target，此时不懂你是否还记得RequestManager，该对象是第一步with方法之后得到的，接着是将我们上一步得到的SingleRequest对象设置给target，接着又执行了RequestManager.track方法，继续跟进该方法看看

```
/** RequestManager 类的track方法*/
void track(@NonNull Target<?> target, @NonNull Request request) {
    targetTracker.track(target);
    requestTracker.runRequest(request);
  }
/** RequestTracker 类的runRequest方法*/  
private final List<Request> pendingRequests = new ArrayList<>();
 public void runRequest(@NonNull Request request) {
    requests.add(request);
    if (!isPaused) {
      request.begin();
    } else {
      request.clear();
      if (Log.isLoggable(TAG, Log.VERBOSE)) {
        Log.v(TAG, "Paused, delaying request");
      }
      pendingRequests.add(request);
    }
  }  
```
- 通过上面的源码，RequestManager对象的track方法中执行了RequestTracker 类的runRequest方法，该方法中简单判断当前Glide是否在暂停状态，不是暂停状态则执行Request的begin方法，否则将这个Request加入到请求队列List<Request>pendingRequests中.
##### 后备回调符、加载占位符和错误占位符加载
- 接下来我们看看Request的begin方法到底干了啥，要找到begin方法实现，根据前面分析，我们则应该去看SingleRequest对象的begin方法

```
/** SingleRequest 类的begin方法*/  
@Override
  public void begin() {
    assertNotCallingCallbacks();
    stateVerifier.throwIfRecycled();
    startTime = LogTime.getLogTime();
    if (model == null) {
      if (Util.isValidDimensions(overrideWidth, overrideHeight)) {
        width = overrideWidth;
        height = overrideHeight;
      }
      int logLevel = getFallbackDrawable() == null ? Log.WARN : Log.DEBUG;
      onLoadFailed(new GlideException("Received null model"), logLevel);
      return;
    }

    if (status == Status.RUNNING) {
      throw new IllegalArgumentException("Cannot restart a running request");
    }
    if (status == Status.COMPLETE) {
      onResourceReady(resource, DataSource.MEMORY_CACHE);
      return;
    }
    status = Status.WAITING_FOR_SIZE;
    if (Util.isValidDimensions(overrideWidth, overrideHeight)) {
      onSizeReady(overrideWidth, overrideHeight);
    } else {
      target.getSize(this);
    }
    if ((status == Status.RUNNING || status == Status.WAITING_FOR_SIZE)
        && canNotifyStatusChanged()) {
      target.onLoadStarted(getPlaceholderDrawable());
    }
    if (IS_VERBOSE_LOGGABLE) {
      logV("finished run method in " + LogTime.getElapsedMillis(startTime));
    }
  }
  
  private void onLoadFailed(GlideException e, int maxLogLevel) {
      //省略部分代码
      .......
      if (!anyListenerHandledUpdatingTarget) {
        setErrorPlaceholder();
      }
     //省略部分代码
     ....... 
  }
```
- 通过以上begin方法源码，如果model为空，也就是我们load传入的图片地址为空，则会调用onLoadFailed方法，而onLoadFailed方法又调用了setErrorPlaceholder方法，接着看看该方法中做了什么操作
```
/** SingleRequest 类的setErrorPlaceholder方法*/ 
private void setErrorPlaceholder() {
    if (!canNotifyStatusChanged()) {
      return;
    }
    Drawable error = null;
    if (model == null) {
      error = getFallbackDrawable();
    }
    // Either the model isn't null, or there was no fallback drawable set.
    if (error == null) {
      error = getErrorDrawable();
    }
    // The model isn't null, no fallback drawable was set or no error drawable was set.
    if (error == null) {
      error = getPlaceholderDrawable();
    }
    target.onLoadFailed(error);
  }
  
  private Drawable getErrorDrawable() {
    if (errorDrawable == null) {
      errorDrawable = requestOptions.getErrorPlaceholder();
      if (errorDrawable == null && requestOptions.getErrorId() > 0) {
        errorDrawable = loadDrawable(requestOptions.getErrorId());
      }
    }
    return errorDrawable;
  }
```
- 通过以上源码，如果我们传入图片地址为空，则首先查看是否有后备回调符设置，然后是错误占位符，最后是加载占位符，最终调用target.onLoadFailed方法，也就是ImageViewTarget的onLoadFailed方法

```
public abstract class ImageViewTarget<Z> extends ViewTarget<ImageView, Z>implements Transition.ViewAdapter {
  //省略部分代码
  .......
  public void setDrawable(Drawable drawable) {
    view.setImageDrawable(drawable);
  }
  
  @Override
  public void onLoadStarted(@Nullable Drawable placeholder) {
    super.onLoadStarted(placeholder);
    setResourceInternal(null);
    setDrawable(placeholder);
  }

  @Override
  public void onLoadFailed(@Nullable Drawable errorDrawable) {
    super.onLoadFailed(errorDrawable);
    setResourceInternal(null);
    setDrawable(errorDrawable);
  }
  //省略部分代码
  .......
}
```
- 通过以上源码，我想你应该已经明白了后备回调符、错误占位符加载，这里还有一个疑问，加载占位符呢？我们回到之前SingleRequest对象的begin方法，相信你会马上看到在加载状态为RUNNING的时候调用了target.onLoadStarted，也实现了加载中的占位符，到这里我们已经分析完了后备回调符、加载占位符和错误占位符加载底层实现逻辑。
##### 加载图片网络请求
- 前面分析完各种占位符实现，我们再次回到SingleRequest对象的begin方法，我们可以注意到onSizeReady()和target.getSize()这两句就是加载图片的入口，如果我们在使用glide的时候设置图片加载的大小尺寸，则会调用target.getSize()
```
/** ViewTarget 类的etSize方法*/ 
void getSize(@NonNull SizeReadyCallback cb) {
      int currentWidth = getTargetWidth();
      int currentHeight = getTargetHeight();
      if (isViewStateAndSizeValid(currentWidth, currentHeight)) {
        cb.onSizeReady(currentWidth, currentHeight);
        return;
      }
      //省略部分代码
      ......
    }
```
- 通过以上源码，target.getSize()会根据ImageView的宽高来得出图片的加载宽高，最终target.getSize()还是会调用onSizeReady()方法，所以我们就直接来看看SingleRequest对象onSizeReady()方法中做了什么操作。
```
/** SingleRequest 类的onSizeReady方法*/
@Override
  public void onSizeReady(int width, int height) {
    stateVerifier.throwIfRecycled();
    if (IS_VERBOSE_LOGGABLE) {
      logV("Got onSizeReady in " + LogTime.getElapsedMillis(startTime));
    }
    if (status != Status.WAITING_FOR_SIZE) {
      return;
    }
    status = Status.RUNNING;

    float sizeMultiplier = requestOptions.getSizeMultiplier();
    this.width = maybeApplySizeMultiplier(width, sizeMultiplier);
    this.height = maybeApplySizeMultiplier(height, sizeMultiplier);

    if (IS_VERBOSE_LOGGABLE) {
      logV("finished setup for calling load in " + LogTime.getElapsedMillis(startTime));
    }
    loadStatus = engine.load(
        glideContext,
        model,
        requestOptions.getSignature(),
        this.width,
        this.height,
        requestOptions.getResourceClass(),
        transcodeClass,
        priority,
        requestOptions.getDiskCacheStrategy(),
        requestOptions.getTransformations(),
        requestOptions.isTransformationRequired(),
        requestOptions.isScaleOnlyOrNoTransform(),
        requestOptions.getOptions(),
        requestOptions.isMemoryCacheable(),
        requestOptions.getUseUnlimitedSourceGeneratorsPool(),
        requestOptions.getUseAnimationPool(),
        requestOptions.getOnlyRetrieveFromCache(),
        this);

    // This is a hack that's only useful for testing right now where loads complete synchronously
    // even though under any executor running on any thread but the main thread, the load would
    // have completed asynchronously.
    if (status != Status.RUNNING) {
      loadStatus = null;
    }
    if (IS_VERBOSE_LOGGABLE) {
      logV("finished onSizeReady in " + LogTime.getElapsedMillis(startTime));
    }
  }
```
- 在onSizeReady方法中，主要调用了engine.load()方法并返回加载状态，engine.load方法继续接收我们之前传入的各种参数，其中也有我们model对象，也就是之前load方法传入的图片地址。首先我们需要了解engine是什么，顾名思义，engine的英文意思是发动机，而在Glide框架中他就是负责启动图片加载的发动机，主要负责启动加载，我们在前面with方法获取glide对象中得到了engine对象（这里就不贴源码了），我们接着看engine.load()方法进行什么操作
```
/** Engine 类的load方法*/
public <R> LoadStatus load(GlideContext glideContext, Object model,Key signature,int width,int height,Class<?> resourceClass,Class<R> transcodeClass, Priority priority,DiskCacheStrategy diskCacheStrategy, Map<Class<?>, Transformation<?>> transformations,boolean isTransformationRequired,boolean isScaleOnlyOrNoTransform,Options options,boolean isMemoryCacheable,boolean useUnlimitedSourceExecutorPool,boolean useAnimationPool,boolean onlyRetrieveFromCache,ResourceCallback cb) {
    Util.assertMainThread();
    long startTime = VERBOSE_IS_LOGGABLE ? LogTime.getLogTime() : 0;

    EngineKey key = keyFactory.buildKey(model, signature, width, height, transformations,
        resourceClass, transcodeClass, options);

    EngineResource<?> active = loadFromActiveResources(key, isMemoryCacheable);
    if (active != null) {
      cb.onResourceReady(active, DataSource.MEMORY_CACHE);
      if (VERBOSE_IS_LOGGABLE) {
        logWithTimeAndKey("Loaded resource from active resources", startTime, key);
      }
      return null;
    }
    EngineResource<?> cached = loadFromCache(key, isMemoryCacheable);
    if (cached != null) {
      cb.onResourceReady(cached, DataSource.MEMORY_CACHE);
      if (VERBOSE_IS_LOGGABLE) {
        logWithTimeAndKey("Loaded resource from cache", startTime, key);
      }
      return null;
    }
    EngineJob<?> current = jobs.get(key, onlyRetrieveFromCache);
    if (current != null) {
      current.addCallback(cb);
      if (VERBOSE_IS_LOGGABLE) {
        logWithTimeAndKey("Added to existing load", startTime, key);
      }
      return new LoadStatus(cb, current);
    }

    EngineJob<R> engineJob =
        engineJobFactory.build(
            key,
            isMemoryCacheable,
            useUnlimitedSourceExecutorPool,
            useAnimationPool,
            onlyRetrieveFromCache);

    DecodeJob<R> decodeJob =
        decodeJobFactory.build(
            glideContext,
            model,
            key,
            signature,
            width,
            height,
            resourceClass,
            transcodeClass,
            priority,
            diskCacheStrategy,
            transformations,
            isTransformationRequired,
            isScaleOnlyOrNoTransform,
            onlyRetrieveFromCache,
            options,
            engineJob);

    jobs.put(key, engineJob);

    engineJob.addCallback(cb);
    engineJob.start(decodeJob);

    if (VERBOSE_IS_LOGGABLE) {
      logWithTimeAndKey("Started new load", startTime, key);
    }
    return new LoadStatus(cb, engineJob);
  }
/**  DecodeJob 类的继承关系*/  
class DecodeJob<R> implements DataFetcherGenerator.FetcherReadyCallback,
    Runnable,
    Comparable<DecodeJob<?>>,
    Poolable   
```
- 通过以上源码，Engine对象的load方前面一段代码都是在处理缓存问题，这里先不进行展开，继续走我们加载图片的主线，往下看我们看到构建了一个EngineJob对象，还构建了一个DecodeJob对象，构建DecodeJob对象又继续接收我们之前传入的各种参数，由DecodeJob对象的继承关系我们可以知道它是Runnable对象，接着我们看到engineJob的start()方法，它直接传入了DecodeJob对象
```
/** EngineJob 类的start方法*/
public void start(DecodeJob<R> decodeJob) {
    this.decodeJob = decodeJob;
    GlideExecutor executor = decodeJob.willDecodeFromCache()
        ? diskCacheExecutor
        : getActiveSourceExecutor();
    executor.execute(decodeJob);
  }
/** GlideExecutor 类的newSourceExecutor方法*/  
public static GlideExecutor newSourceExecutor(
      int threadCount, String name, UncaughtThrowableStrategy uncaughtThrowableStrategy) {
    return new GlideExecutor(
        new ThreadPoolExecutor(
            threadCount /* corePoolSize */,
            threadCount /* maximumPoolSize */,
            0 /* keepAliveTime */,
            TimeUnit.MILLISECONDS,
            new PriorityBlockingQueue<Runnable>(),
            new DefaultThreadFactory(name, uncaughtThrowableStrategy, false)));
  }  
```
- 通过以上源码，EngineJob对象的start方法首先还是判断缓存，最终获取的GlideExecutor就是一个线程池执行器（Executor），GlideExecutor中有各种方法获得缓存线程池，还有资源线程池（SourceExecutor），以上源码贴出资源[线程池](https://www.maoqitian.com/2019/01/20/Java中的线程池/)。实际上EngineJob对象的start方法就是用来在线程池中启动DecodeJob这个Runnable对象，也就是说EngineJob的主要作用是开启线程来加载图片，接着我们来看看DecodeJob对象的run方法。
```
/** DecodeJob 类的run方法*/
public void run() {
    DataFetcher<?> localFetcher = currentFetcher;
    try {
      if (isCancelled) {
        notifyFailed();
        return;
      }
      runWrapped();
    } catch (Throwable t) {
      //省略部分代码
      .......
    } finally {
      if (localFetcher != null) {
        localFetcher.cleanup();
      }
      GlideTrace.endSection();
    }
  }
/** DecodeJob 类的runWrapped方法*/  
private void runWrapped() {
    switch (runReason) {
      case INITIALIZE:
        stage = getNextStage(Stage.INITIALIZE);
        currentGenerator = getNextGenerator();
        runGenerators();
        break;
      case SWITCH_TO_SOURCE_SERVICE:
        runGenerators();
        break;
      case DECODE_DATA:
        decodeFromRetrievedData();
        break;
      default:
        throw new IllegalStateException("Unrecognized run reason: " + runReason);
    }
  }
/** DecodeJob 类的getNextStage方法*/   
private Stage getNextStage(Stage current) {
    switch (current) {
      case INITIALIZE:
        return diskCacheStrategy.decodeCachedResource()
            ? Stage.RESOURCE_CACHE : getNextStage(Stage.RESOURCE_CACHE);
      case RESOURCE_CACHE:
        return diskCacheStrategy.decodeCachedData()
            ? Stage.DATA_CACHE : getNextStage(Stage.DATA_CACHE);
      case DATA_CACHE:
        // Skip loading from source if the user opted to only retrieve the resource from cache.
        return onlyRetrieveFromCache ? Stage.FINISHED : Stage.SOURCE;
      case SOURCE:
      case FINISHED:
        return Stage.FINISHED;
      default:
        throw new IllegalArgumentException("Unrecognized stage: " + current);
    }
  }
 /** DecodeJob 类的getNextGenerator方法*/   
private DataFetcherGenerator getNextGenerator() {
    switch (stage) {
      case RESOURCE_CACHE:
        return new ResourceCacheGenerator(decodeHelper, this);
      case DATA_CACHE:
        return new DataCacheGenerator(decodeHelper, this);
      case SOURCE:
        return new SourceGenerator(decodeHelper, this);
      case FINISHED:
        return null;
      default:
        throw new IllegalStateException("Unrecognized stage: " + stage);
    }
 /** DecodeJob 类的runGenerators方法*/       
 private void runGenerators() {
    currentThread = Thread.currentThread();
    startFetchTime = LogTime.getLogTime();
    boolean isStarted = false;
    while (!isCancelled && currentGenerator != null
        && !(isStarted = currentGenerator.startNext())) {
      stage = getNextStage(stage);
      currentGenerator = getNextGenerator();

      if (stage == Stage.SOURCE) {
        reschedule();
        return;
      }
    }
   //省略部分代码
      ....... 
  }    
```
- 上面我们再次贴出了一堆代码，我们来好好梳理一下逻辑，DecodeJob对象的run方法中逻辑很简单，就是调用了自身的runWrapped方法，runWrapped方法中首先判断Stage枚举，前面在创建DecodeJob对象时候设置初始状态为Stage.INITIALIZE，然后接着调用getNextStage方法，这里我们还是继续跳过缓存，所以getNextStage方法最终返回的是Stage.SOURCE状态，接着在getNextGenerator()方法中我们获取就是SourceGenerator对象，也就是run方法中的第一句话DataFetcher<?> localFetcher = currentFetcher中localFetcher就是我们刚刚获得的**SourceGenerator对象**，接着继续执行runGenerators()方法，在该方法的while循环判断条件执行了currentGenerator.startNext()方法，也就是**SourceGenerator对象**的startNext()方法
```
/** SourceGenerator 类的startNext()方法*/ 
@Override
  public boolean startNext() {
    //省略部分代码，跳过缓存部分判断
    ........
    loadData = null;
    boolean started = false;
    while (!started && hasNextModelLoader()) {
      loadData = helper.getLoadData().get(loadDataListIndex++);
      if (loadData != null
          && (helper.getDiskCacheStrategy().isDataCacheable(loadData.fetcher.getDataSource())
          || helper.hasLoadPath(loadData.fetcher.getDataClass()))) {
        started = true;
        loadData.fetcher.loadData(helper.getPriority(), this);
      }
    }
    return started;
  }
/** DecodeHelper 类的getLoadData() 方法*/
List<LoadData<?>> getLoadData() {
    if (!isLoadDataSet) {
      isLoadDataSet = true;
      loadData.clear();
      List<ModelLoader<Object, ?>> modelLoaders = glideContext.getRegistry().getModelLoaders(model);
      //noinspection ForLoopReplaceableByForEach to improve perf
      for (int i = 0, size = modelLoaders.size(); i < size; i++) {
        ModelLoader<Object, ?> modelLoader = modelLoaders.get(i);
        LoadData<?> current =
            modelLoader.buildLoadData(model, width, height, options);
        if (current != null) {
          loadData.add(current);
        }
      }
    }
    return loadData;
  }
/** HttpGlideUrlLoader 类的buildLoadData 方法*/ 
 @Override
  public LoadData<InputStream> buildLoadData(@NonNull GlideUrl model, int width, int height,
      @NonNull Options options) {
    // GlideUrls memoize parsed URLs so caching them saves a few object instantiations and time
    // spent parsing urls.
    GlideUrl url = model;
    if (modelCache != null) {
      url = modelCache.get(model, 0, 0);
      if (url == null) {
        modelCache.put(model, 0, 0, model);
        url = model;
      }
    }
    int timeout = options.get(TIMEOUT);
    return new LoadData<>(url, new HttpUrlFetcher(url, timeout));
  } 
```
- 通过上面源码，我们接着看到loadData=helper.getLoadData().get(loadDataListIndex++)这一句代码，helper就是DecodeHelper对象，在我们前面创建DecodeJob对象的时候已经把它创建，之前我们在load步骤中传入的model是图片url地址，所以经过DecodeHelper 类的getLoadData() 方法（更细的代码这里就不进行展开了），最终获取的ModelLoader<Object, ?> modelLoader对象则为**HttpGlideUrlLoader对象**，也就是laodData对象，所以modelLoader.buildLoadData创建则在HttpGlideUrlLoader对象的buildLoadData中实现，上方贴出的该方法源码中把我们model赋值给GlideUrl对象，也就是将其作为URL地址来进行处理，则经过modelLoader.buildLoadData获取的loadData.fetcher则对应**HttpUrlFetcher对象**，所以loadData.fetcher.loadData调用的就是HttpUrlFetcher对象loadData方法，
```
/**HttpUrlFetcher类的loadData方法 **/
@Override
  public void loadData(@NonNull Priority priority,
      @NonNull DataCallback<? super InputStream> callback) {
    long startTime = LogTime.getLogTime();
    try {
      InputStream result = loadDataWithRedirects(glideUrl.toURL(), 0, null, glideUrl.getHeaders());
      callback.onDataReady(result);
    } catch (IOException e) {
      if (Log.isLoggable(TAG, Log.DEBUG)) {
        Log.d(TAG, "Failed to load data for url", e);
      }
      callback.onLoadFailed(e);
    } finally {
      if (Log.isLoggable(TAG, Log.VERBOSE)) {
        Log.v(TAG, "Finished http url fetcher fetch in " + LogTime.getElapsedMillis(startTime));
      }
    }
  }
/**HttpUrlFetcher类的loadDataWithRedirects方法 **/  
private InputStream loadDataWithRedirects(URL url, int redirects, URL lastUrl,
      Map<String, String> headers) throws IOException {
    if (redirects >= MAXIMUM_REDIRECTS) {
      throw new HttpException("Too many (> " + MAXIMUM_REDIRECTS + ") redirects!");
    } else {
      // Comparing the URLs using .equals performs additional network I/O and is generally broken.
      // See http://michaelscharf.blogspot.com/2006/11/javaneturlequals-and-hashcode-make.html.
      try {
        if (lastUrl != null && url.toURI().equals(lastUrl.toURI())) {
          throw new HttpException("In re-direct loop");

        }
      } catch (URISyntaxException e) {
        // Do nothing, this is best effort.
      }
    }

    urlConnection = connectionFactory.build(url);
    for (Map.Entry<String, String> headerEntry : headers.entrySet()) {
      urlConnection.addRequestProperty(headerEntry.getKey(), headerEntry.getValue());
    }
    urlConnection.setConnectTimeout(timeout);
    urlConnection.setReadTimeout(timeout);
    urlConnection.setUseCaches(false);
    urlConnection.setDoInput(true);

    // Stop the urlConnection instance of HttpUrlConnection from following redirects so that
    // redirects will be handled by recursive calls to this method, loadDataWithRedirects.
    urlConnection.setInstanceFollowRedirects(false);

    // Connect explicitly to avoid errors in decoders if connection fails.
    urlConnection.connect();
    // Set the stream so that it's closed in cleanup to avoid resource leaks. See #2352.
    stream = urlConnection.getInputStream();
    if (isCancelled) {
      return null;
    }
    final int statusCode = urlConnection.getResponseCode();
    if (isHttpOk(statusCode)) {
      return getStreamForSuccessfulRequest(urlConnection);
    } else if (isHttpRedirect(statusCode)) {
      String redirectUrlString = urlConnection.getHeaderField("Location");
      if (TextUtils.isEmpty(redirectUrlString)) {
        throw new HttpException("Received empty or null redirect url");
      }
      URL redirectUrl = new URL(url, redirectUrlString);
      // Closing the stream specifically is required to avoid leaking ResponseBodys in addition
      // to disconnecting the url connection below. See #2352.
      cleanup();
      return loadDataWithRedirects(redirectUrl, redirects + 1, url, headers);
    } else if (statusCode == INVALID_STATUS_CODE) {
      throw new HttpException(statusCode);
    } else {
      throw new HttpException(urlConnection.getResponseMessage(), statusCode);
    }
  }  
```
- 通过以上源码，HttpUrlFetcher对象的loadData方法首先调用自身loadDataWithRedirects方法，接着我们看到该方法源码，这里使用了HttpURLConnection来执行了网络请求，看到这里内心还是有点开心的，前面看了这么多源码，终于看到Glide的网络请求了，开心之后还没完呢，还得接着往下看，执行完网络请求成功，loadDataWithRedirects方法中网络请求成功调用getStreamForSuccessfulRequest返回了一个**InputStream流**(记住这个InputStream，很关键)，然后执行了一个callback回调，而这个回调对象就是我们之前在SourceGenerator对象中调用loadData方法传入SourceGenerator对象本身，所以callback.onDataReady()调用的就是SourceGenerator对象的onDataReady方法
```
/**SourceGenerator类的onDataReady方法 **/  
  @Override
  public void onDataReady(Object data) {
    DiskCacheStrategy diskCacheStrategy = helper.getDiskCacheStrategy();
    if (data != null && diskCacheStrategy.isDataCacheable(loadData.fetcher.getDataSource())) {
      dataToCache = data;
      // We might be being called back on someone else's thread. Before doing anything, we should
      // reschedule to get back onto Glide's thread.
      cb.reschedule();
    } else {
      cb.onDataFetcherReady(loadData.sourceKey, data, loadData.fetcher,
          loadData.fetcher.getDataSource(), originalKey);
    }
  }
```
- 通过以上源码，不走缓存的情况下则调用cb.onDataFetcherReady，这个cb也就是前面我们new SourceGenerator对象传入的 DecodeJob对象，也就是调用DecodeJob对象onDataFetcherReady方法
```
/**DecodeJob类的onDataFetcherReady方法 **/ 
 @Override
  public void onDataFetcherReady(Key sourceKey, Object data, DataFetcher<?> fetcher,
      DataSource dataSource, Key attemptedKey) {
    this.currentSourceKey = sourceKey;
    this.currentData = data;
    this.currentFetcher = fetcher;
    this.currentDataSource = dataSource;
    this.currentAttemptingKey = attemptedKey;
    if (Thread.currentThread() != currentThread) {
      runReason = RunReason.DECODE_DATA;
      callback.reschedule(this);
    } else {
      GlideTrace.beginSection("DecodeJob.decodeFromRetrievedData");
      try {
        decodeFromRetrievedData();
      } finally {
        GlideTrace.endSection();
      }
    }
  }
```
- 通过以上源码，onDataFetcherReady方法中将之前网络请求得到的流赋值给当前的DecodeJob对象的currentData，其他数据都赋值给对应字段，最终调用的是**decodeFromRetrievedData方法**
##### 加载图片（解码，转码）
```
/**DecodeJob类的decodeFromRetrievedData方法 **/ 
private void decodeFromRetrievedData() {
    if (Log.isLoggable(TAG, Log.VERBOSE)) {
      logWithTimeAndKey("Retrieved data", startFetchTime,
          "data: " + currentData
              + ", cache key: " + currentSourceKey
              + ", fetcher: " + currentFetcher);
    }
    Resource<R> resource = null;
    try {
      resource = decodeFromData(currentFetcher, currentData, currentDataSource);
    } catch (GlideException e) {
      e.setLoggingDetails(currentAttemptingKey, currentDataSource);
      throwables.add(e);
    }
    if (resource != null) {
      notifyEncodeAndRelease(resource, currentDataSource);
    } else {
      runGenerators();
    }
  }
/**DecodeJob类的decodeFromData方法 **/   
private <Data> Resource<R> decodeFromData(DataFetcher<?> fetcher, Data data,
      DataSource dataSource) throws GlideException {
    try {
      if (data == null) {
        return null;
      }
      long startTime = LogTime.getLogTime();
      Resource<R> result = decodeFromFetcher(data, dataSource);
      if (Log.isLoggable(TAG, Log.VERBOSE)) {
        logWithTimeAndKey("Decoded result " + result, startTime);
      }
      return result;
    } finally {
      fetcher.cleanup();
    }
  }
/**DecodeJob类的decodeFromFetcher方法 **/  
 private <Data> Resource<R> decodeFromFetcher(Data data, DataSource dataSource)
      throws GlideException {
    LoadPath<Data, ?, R> path = decodeHelper.getLoadPath((Class<Data>) data.getClass());
    return runLoadPath(data, dataSource, path);
  }
```
- 通过以上源码，decodeFromRetrievedData方法调用了decodeFromFetcher方法，在该方法中首先通过decodeHelper.getLoadPath获取LoadPath对象，LoadPath对象其实是根据我们传入的处理数据来返回特定的数据解码转码处理器，我们跟进decodeHelper.getLoadPath看看
```
/** DecodeHelper类的getLoadPath方法*/
<Data> LoadPath<Data, ?, Transcode> getLoadPath(Class<Data> dataClass) {
    return glideContext.getRegistry().getLoadPath(dataClass, resourceClass, transcodeClass);
  }
/** Registry类的getLoadPath方法*/
 @Nullable
  public <Data, TResource, Transcode> LoadPath<Data, TResource, Transcode> getLoadPath(
      @NonNull Class<Data> dataClass, @NonNull Class<TResource> resourceClass,
      @NonNull Class<Transcode> transcodeClass) {
    //省略部分代码
    .......
     List<DecodePath<Data, TResource, Transcode>> decodePaths =
          getDecodePaths(dataClass, resourceClass, transcodeClass);
      if (decodePaths.isEmpty()) {
        result = null;
      } else {
        result =
            new LoadPath<>(
                dataClass, resourceClass, transcodeClass, decodePaths, throwableListPool);
      }
      loadPathCache.put(dataClass, resourceClass, transcodeClass, result);
    }
    return result;
  } 
/** Registry类的getDecodePaths方法*/ 
@NonNull
  private <Data, TResource, Transcode> List<DecodePath<Data, TResource, Transcode>> getDecodePaths(
      @NonNull Class<Data> dataClass, @NonNull Class<TResource> resourceClass,
      @NonNull Class<Transcode> transcodeClass) {
      //省略部分代码，去除干扰

        List<ResourceDecoder<Data, TResource>> decoders =
            decoderRegistry.getDecoders(dataClass, registeredResourceClass);
        ResourceTranscoder<TResource, Transcode> transcoder =
            transcoderRegistry.get(registeredResourceClass, registeredTranscodeClass);
        @SuppressWarnings("PMD.AvoidInstantiatingObjectsInLoops")
        DecodePath<Data, TResource, Transcode> path =
            new DecodePath<>(dataClass, registeredResourceClass, registeredTranscodeClass,
                decoders, transcoder, throwableListPool);
        decodePaths.add(path);
    }
    return decodePaths;
  }
 /** Registry类的getDecoders方法*/  
 public synchronized <T, R> List<ResourceDecoder<T, R>> getDecoders(@NonNull Class<T> dataClass,
      @NonNull Class<R> resourceClass) {
    List<ResourceDecoder<T, R>> result = new ArrayList<>();
    for (String bucket : bucketPriorityList) {
      List<Entry<?, ?>> entries = decoders.get(bucket);
      if (entries == null) {
        continue;
      }
      for (Entry<?, ?> entry : entries) {
        if (entry.handles(dataClass, resourceClass)) {
          result.add((ResourceDecoder<T, R>) entry.decoder);
        }
      }
    }
    // TODO: cache result list.

    return result;
  } 
```
- 通过以上源码，我们接着前面跟进DecodeHelper.getLoadPath方法，它调用了Registry对象的getLoadPath方法，Registry对象的getLoadPath方法又调用了自身的getDecodePaths方法，现在我前面提到过得我们网络请求获取的是**InputStream流**，所以上面源码getDecodePaths方法中Data泛型就是InputStream，在根据getDecoders方法遍历得到解码器ResourceDecoder能处理InputStream流的有StreamBitmapDecoder和StreamGifDecoder，StreamGifDecoder处理的是Gif，我们这里处理图片就之能是**StreamBitmapDecoder**，它将InputStream流解码成bitmap，然后能将bitmap转换成Drawable的转码器ResourceTranscoder对象则是**BitmapDrawableTranscoder**，最后getDecodePaths将我们刚刚分析得到的解码器和转码器传递给了新建的**DecodePath对象**，DecodePath对象就是用来帮助我们进行解码和转码的。
- 接着我们继续上一步的decodeFromFetcher方法，该方法返回的runLoadPath最终调用了上面获得的DecodePath对象的decode方法

```
/**DecodePath类的decode方法**/
public Resource<Transcode> decode(DataRewinder<DataType> rewinder, int width, int height,
      @NonNull Options options, DecodeCallback<ResourceType> callback) throws GlideException {
    Resource<ResourceType> decoded = decodeResource(rewinder, width, height, options);
    Resource<ResourceType> transformed = callback.onResourceDecoded(decoded);
    return transcoder.transcode(transformed, options);
  }
/**DecodePath类的decodeResource方法**/
@NonNull
  private Resource<ResourceType> decodeResource(DataRewinder<DataType> rewinder, int width,
      int height, @NonNull Options options) throws GlideException {
    List<Throwable> exceptions = Preconditions.checkNotNull(listPool.acquire());
    try {
      return decodeResourceWithList(rewinder, width, height, options, exceptions);
    } finally {
      listPool.release(exceptions);
    }
  }
/**DecodePath类的decodeResourceWithList方法**/
  @NonNull
  private Resource<ResourceType> decodeResourceWithList(DataRewinder<DataType> rewinder, int width,
      int height, @NonNull Options options, List<Throwable> exceptions) throws GlideException {
    Resource<ResourceType> result = null;
    //省略部分代码
    ........
      ResourceDecoder<DataType, ResourceType> decoder = decoders.get(i);
      try {
        DataType data = rewinder.rewindAndGet();
        if (decoder.handles(data, options)) {
          data = rewinder.rewindAndGet();
          result = decoder.decode(data, width, height, options);
        }
    //省略部分代码
    ........
    return result;
  }  
```
- 通过以上源码，DecodePath对象的decode方法调用了decodeResource方法，decodeResource又调用了decodeResourceWithList方法，经过前面分析，decodeResourceWithList方法中获得的decoder就是前面提到的解码器StreamBitmapDecoder对象，所以我们接着看StreamBitmapDecoder的decode方法
```
/**StreamBitmapDecoder类的decode方法**/
@Override
  public Resource<Bitmap> decode(@NonNull InputStream source, int width, int height,
      @NonNull Options options)
      throws IOException {

    // Use to fix the mark limit to avoid allocating buffers that fit entire images.
    final RecyclableBufferedInputStream bufferedStream;
    final boolean ownsBufferedStream;
    if (source instanceof RecyclableBufferedInputStream) {
      bufferedStream = (RecyclableBufferedInputStream) source;
      ownsBufferedStream = false;
    } else {
      bufferedStream = new RecyclableBufferedInputStream(source, byteArrayPool);
      ownsBufferedStream = true;
    }

    ExceptionCatchingInputStream exceptionStream =
        ExceptionCatchingInputStream.obtain(bufferedStream);
    MarkEnforcingInputStream invalidatingStream = new MarkEnforcingInputStream(exceptionStream);
    UntrustedCallbacks callbacks = new UntrustedCallbacks(bufferedStream, exceptionStream);
    try {
      return downsampler.decode(invalidatingStream, width, height, options, callbacks);
    } finally {
      exceptionStream.release();
      if (ownsBufferedStream) {
        bufferedStream.release();
      }
    }
  }
```
- 通过以上源码，StreamBitmapDecoder的decode方法中只是对InputStream进行包装（**装饰模式**），可以让Glide进行更多操作，最终调用了downsampler.decode，这个downsampler对象则是Downsampler对象（英文注释：Downsamples, decodes, and rotates images according to their exif orientation.），英文注释大致意思是对图像exif格式进行采样、解码和旋转。而我们这里调用了它的decode方法，也就是对我们前面包装的流进行解码
```
/**Downsampler类的decode方法**/
public Resource<Bitmap> decode(InputStream is, int requestedWidth, int requestedHeight,
      Options options, DecodeCallbacks callbacks) throws IOException {
    //省略部分代码
    try {
      Bitmap result = decodeFromWrappedStreams(is, bitmapFactoryOptions,
          downsampleStrategy, decodeFormat, isHardwareConfigAllowed, requestedWidth,
          requestedHeight, fixBitmapToRequestedDimensions, callbacks);
      return BitmapResource.obtain(result, bitmapPool);
    } finally {
      releaseOptions(bitmapFactoryOptions);
      byteArrayPool.put(bytesForOptions);
    }
  }
/**Downsampler类的decodeFromWrappedStreams方法**/  
private Bitmap decodeFromWrappedStreams(InputStream is,
      BitmapFactory.Options options, DownsampleStrategy downsampleStrategy,
      DecodeFormat decodeFormat, boolean isHardwareConfigAllowed, int requestedWidth,
      int requestedHeight, boolean fixBitmapToRequestedDimensions,
      DecodeCallbacks callbacks) throws IOException {
    //省略部分代码
    .........
    Bitmap downsampled = decodeStream(is, options, callbacks, bitmapPool);
    callbacks.onDecodeComplete(bitmapPool, downsampled);
    //省略部分代码
    .........
    Bitmap rotated = null;
    if (downsampled != null) {
      //缩放效正处理
      downsampled.setDensity(displayMetrics.densityDpi);
      rotated = TransformationUtils.rotateImageExif(bitmapPool, downsampled, orientation);
      if (!downsampled.equals(rotated)) {
        bitmapPool.put(downsampled);
      }
    }
    return rotated;
  }
/**Downsampler类的decodeStream方法**/  
private static Bitmap decodeStream(InputStream is, BitmapFactory.Options options,
      DecodeCallbacks callbacks, BitmapPool bitmapPool) throws IOException {
    if (options.inJustDecodeBounds) {
      is.mark(MARK_POSITION);
    } else {
      callbacks.onObtainBounds();
    }
    int sourceWidth = options.outWidth;
    int sourceHeight = options.outHeight;
    String outMimeType = options.outMimeType;
    final Bitmap result;
    TransformationUtils.getBitmapDrawableLock().lock();
    try {
      result = BitmapFactory.decodeStream(is, null, options);
    } catch (IllegalArgumentException e) {
      //省略部分代码
    .........
    return result;
  }  
```
- 通过以上源码，Downsampler对象的decode方法首先调用了decodeFromWrappedStreams方法，在decodeFromWrappedStreams方法中又调用了decodeStream方法，在该方法中调用用了BitmapFactory.decodeStream，到这里我们终于看到了Glide将InputStream流解析成了bitmap，而最终Downsampler对象的decode方法返回的Resource对象就是BitmapResource对象
- 经过前面的分析，Glide已经将InputStream**解码完成**，这时我们还得再次回到DecodePath对象的decode方法，解码完成还需转码，这里再次贴一下ecodePath对象的decode方法，前面已经分析了转码器为BitmapDrawableTranscoder对象，所以我们继续看BitmapDrawableTranscoder对象transcode做了什么操作
```
/**DecodePath类的decode方法**/  
public Resource<Transcode> decode(DataRewinder<DataType> rewinder, int width, int height,
      @NonNull Options options, DecodeCallback<ResourceType> callback) throws GlideException {
    Resource<ResourceType> decoded = decodeResource(rewinder, width, height, options);
    Resource<ResourceType> transformed = callback.onResourceDecoded(decoded);
    return transcoder.transcode(transformed, options);
  }
/**BitmapDrawableTranscoder类的transcode方法**/  
public Resource<BitmapDrawable> transcode(@NonNull Resource<Bitmap> toTranscode,
      @NonNull Options options) {
    return LazyBitmapDrawableResource.obtain(resources, toTranscode);
  } 
/**LazyBitmapDrawableResource类的obtain方法**/  
@Nullable
  public static Resource<BitmapDrawable> obtain(
      @NonNull Resources resources, @Nullable Resource<Bitmap> bitmapResource) {
    if (bitmapResource == null) {
      return null;
    }
    return new LazyBitmapDrawableResource(resources, bitmapResource);
  }
```
- 通过以上源码，BitmapDrawableTranscoder对象transcode方法最终返回了**LazyBitmapDrawableResource**对象，也就是将我们解码拿到的BitmapResource对象转换成了**LazyBitmapDrawableResource**对象
> 到此，Glide整个图片解码转码已近完成，接着我们再回到DecodeJob对象的decodeFromRetrievedData方法

##### 图片显示
```
/**DecodeJob类的decodeFromRetrievedData方法**/
private void decodeFromRetrievedData() {
    Resource<R> resource = null;
    try {
      resource = decodeFromData(currentFetcher, currentData, currentDataSource);
    } catch (GlideException e) {
      e.setLoggingDetails(currentAttemptingKey, currentDataSource);
      throwables.add(e);
    }
    if (resource != null) {
      notifyEncodeAndRelease(resource, currentDataSource);
    } else {
      runGenerators();
    }
  }
/**DecodeJob类的notifyEncodeAndRelease方法**/  
private void notifyEncodeAndRelease(Resource<R> resource, DataSource dataSource) {
    //省略部分代码
    .....
    Resource<R> result = resource;
    //省略部分代码
    .....
    notifyComplete(result, dataSource);
    stage = Stage.ENCODE;
    //省略部分代码
    .....
  }
/**DecodeJob类的notifyComplete方法**/    
 private void notifyComplete(Resource<R> resource, DataSource dataSource) {
    setNotifiedOrThrow();
    callback.onResourceReady(resource, dataSource);
  }  
```
- 通过以上源码，DecodeJob对象的decodeFromRetrievedData方法调用notifyEncodeAndRelease方法，将我们上一步获取的LazyBitmapDrawableResource传入notifyComplete方法中，在notifyComplete调用了callback.onResourceReady，而这个callback对象就是EngineJob对象（它实现了DecodeJob.Callback接口），也许到这里你已经忘了EngineJob对象是什么，前面我们开启线程执行加载的start方法就在EngineJob对象中，所以我们去看看EngineJob对象的onResourceReady方法
```
private static final Handler MAIN_THREAD_HANDLER =
      new Handler(Looper.getMainLooper(), new MainThreadCallback());
/**EngineJob类的onResourceReady方法**/
@Override
  public void onResourceReady(Resource<R> resource, DataSource dataSource) {
    this.resource = resource;
    this.dataSource = dataSource;
    MAIN_THREAD_HANDLER.obtainMessage(MSG_COMPLETE, this).sendToTarget();
  }
/**EngineJob类的handleMessage方法**/
 @Override
    public boolean handleMessage(Message message) {
      EngineJob<?> job = (EngineJob<?>) message.obj;
      switch (message.what) {
        case MSG_COMPLETE:
          job.handleResultOnMainThread();
          break;
        case MSG_EXCEPTION:
          job.handleExceptionOnMainThread();
          break;
        case MSG_CANCELLED:
          job.handleCancelledOnMainThread();
          break;
        default:
          throw new IllegalStateException("Unrecognized message: " + message.what);
      }
      return true;
    } 
/**EngineJob类的handleResultOnMainThread方法**/    
 @Synthetic
  void handleResultOnMainThread() {
    //省略部分代码
    ......
    engineResource = engineResourceFactory.build(resource, isCacheable);
    hasResource = true;
   //省略部分代码
    ......
    for (int i = 0, size = cbs.size(); i < size; i++) {
      ResourceCallback cb = cbs.get(i);
      if (!isInIgnoredCallbacks(cb)) {
        engineResource.acquire();
        cb.onResourceReady(engineResource, dataSource);
      }
    }
    //省略部分代码
    ......
  }
/**EngineJob类的addCallback方法**/ 
private final List<ResourceCallback> cbs = new ArrayList<>(2);
void addCallback(ResourceCallback cb) {
    Util.assertMainThread();
    stateVerifier.throwIfRecycled();
    if (hasResource) {
      cb.onResourceReady(engineResource, dataSource);
    } else if (hasLoadFailed) {
      cb.onLoadFailed(exception);
    } else {
      cbs.add(cb);
    }
  }  
```
- 通过以上源码，前面我们在Engine类的Load方法中已经将SingleRequest这个对象通过EngineJob对象的addCallback方法加入到了cbs这个List当中，EngineJob对象的onResourceReady方法中将我们加载好的图片对象通过Hanlder将数据又传递到了主线程（主线程更新UI），也就是handleResultOnMainThread方法中根据我们刚刚的分析通过cb.onResourceReady将数据回调通知，cb对象就是SingleRequest对象，我们接着看SingleRequest对象的onResourceReady回调方法
```
/**SingleRequest类的onResourceReady回调方法**/
@SuppressWarnings("unchecked")
@Override
public void onResourceReady(Resource<?> resource, DataSource dataSource) {
    //省略部分代码
    .......
    Object received = resource.get();
    //省略部分代码
    .......
    onResourceReady((Resource<R>) resource, (R) received, dataSource);
  }
/**SingleRequest类的onResourceReady方法**/  
private void onResourceReady(Resource<R> resource, R result, DataSource dataSource) {
    //省略部分代码
    .......
    status = Status.COMPLETE;
    this.resource = resource;
    //省略部分代码
    .......
      if (!anyListenerHandledUpdatingTarget) {
        Transition<? super R> animation =
            animationFactory.build(dataSource, isFirstResource);
        target.onResourceReady(result, animation);
      }
    } 
    //省略部分代码
    .......
  }
```
通过以上源码，这时候我们已经可以看到长征胜利的曙光了，SingleRequest对象的onResourceReady回调方法中调用了resource.get()，而这个resource就是前面我们经过解码、转码获取的**LazyBitmapDrawableResource对象**，然后又调用了SingleRequest对象的onResourceReady私有方法，在该方法中又调用了target.onResourceReady方法，在我们最开始进入into方法的时候我们已经分析过创建的target对象就是**DrawableImageViewTarget对象**，它继承了抽象类**ImageViewTarget**，所以我们看看抽象类ImageViewTarget的onResourceReady方法
```
/** LazyBitmapDrawableResource类的get()方法**/
@NonNull
  @Override
  public BitmapDrawable get() {
    return new BitmapDrawable(resources, bitmapResource.get());
  }

/** ImageViewTarget类的onResourceReady方法**/
 @Override
  public void onResourceReady(@NonNull Z resource, @Nullable Transition<? super Z> transition) {
    if (transition == null || !transition.transition(resource, this)) {
      setResourceInternal(resource);
    } else {
      maybeUpdateAnimatable(resource);
    }
  }
/** ImageViewTarget类的setResourceInternal方法**/  
 private void setResourceInternal(@Nullable Z resource) {
    // Order matters here. Set the resource first to make sure that the Drawable has a valid and
    // non-null Callback before starting it.
    setResource(resource);
    maybeUpdateAnimatable(resource);
  }
  protected abstract void setResource(@Nullable Z resource);
```
- 通过以上源码，LazyBitmapDrawableResource对象的get()方法获取了BitmapDrawable（实际就是Drawable对象），ImageViewTarget对象的onResourceReady方法通过前面分析被调用，然后该方法再调用了ImageViewTarget对象setResourceInternal方法，setResourceInternal方法最终setResource方法，setResource在ImageViewTarget对象是抽象方法，它在DrawableImageViewTarget对象中实现，最后我们看看DrawableImageViewTarget对象setResource方法
```
 @Override
  protected void setResource(@Nullable Drawable resource) {
    view.setImageDrawable(resource);
  }
```
> 到此，我们的万里长征终于结束了一半，Glide的简单加载图片流程已经分析完了。

## 最后说点
- 最后我还是想要把那句简单的代码给贴出来
```
Glide.with(Context).load(IMAGE_URL).into(mImageView);
```
- 就是这样一句简单的代码，它背后所走的逻辑却让人头皮发麻，此时我只想说一句话“read the fuck source code”。前面我们只是分析了Glide简单的加载图片流程，它的缓存使用，各种变换，回调和各种功能原理还没分析到，这只能等到下篇文章了。文章中如果有错误，请大家给我提出来，大家一起学习进步，如果觉得我的文章给予你帮助，也请给我一个喜欢和关注，同时也欢迎访问我的[**个人博客**](https://www.maoqitian.com)。
- 参考链接

  - [Gldie文档](http://bumptech.github.io/glide/)