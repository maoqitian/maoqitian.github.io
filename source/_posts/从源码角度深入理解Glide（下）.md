---
title: 从源码角度深入理解Glide（下）
date: 2019-02-19 22:01:24
categories:
- Android热门框架解析 #分类
tags:
- Glide
- 图片加载
- Android
- 源码分析
---
![image](https://raw.githubusercontent.com/bumptech/glide/master/static/glide_logo.png)
> 上一篇文章[从源码角度深入理解Glide（上）](https://www.maoqitian.com/2019/02/19/%E4%BB%8E%E6%BA%90%E7%A0%81%E8%A7%92%E5%BA%A6%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3glide%EF%BC%88%E4%B8%8A%EF%BC%89/)中，我们已经把Glide加载图片的基本流程走了一遍，想必你已经对Glide的加载原理有了新的认识并且见识到了Glide源码的复杂逻辑，在我们感叹Glide源码复杂的同时我们也忽略了Glide加载图片过程的其它细节，特别是缓存方面，我们在上一篇文章中对于缓存的处理都是跳过的，这一篇文章我们就从Glide的缓存开始再次对Glide进行深入理解。
<!--more-->
## Glide缓存
- Glide加载默认情况下可以分为三级缓存，哪三级呢？他们分别是**内存、磁盘和网络。**
- 默认情况下，Glide 会在开始一个新的图片请求之前检查以下多级的缓存：
 
  - 1.活动资源 (Active Resources) - 现在是否有另一个 View 正在展示这张图片
  - 2.内存缓存 (Memory cache) - 该图片是否最近被加载过并仍存在于内存中
  - 3.资源类型（Resource） - 该图片是否之前曾被解码、转换并写入过磁盘缓存
  - 4.数据来源 (Data) - 构建这个图片的资源是否之前曾被写入过文件缓存
- 网络级别的加载我们已经在上一篇文章了解了，上面列出的前两种情况则是内存缓存，后两种情况则是磁盘缓存，如果以上四种情况都不存在，Glide则会通过返回到原始资源以取回数据（原始文件，Uri, Url（网络）等） 
### 缓存的key
- 提起缓存，我们首先要明白，Glide中缓存的图片肯定不止一个，当我们加载图片的同时，如果缓存中有我们正在加载的图片，我们怎么找到这个图片的缓存呢？所以为了找到对应的缓存，则每一个缓存都有它对应的标识，这个标识在Glide中用接口Key来描述
```
/**Key 接口*/
public interface Key {
  String STRING_CHARSET_NAME = "UTF-8";
  Charset CHARSET = Charset.forName(STRING_CHARSET_NAME);
  void updateDiskCacheKey(@NonNull MessageDigest messageDigest);
  @Override
  boolean equals(Object o);
  @Override
  int hashCode();
}
```
#### 缓存Key的生成
- 前面提到了缓存Key的接口，那这个缓存的Key实在哪里生成的，实现类又是什么呢？这我们就要看到加载发动机Engine类的load方法
```
private final EngineKeyFactory keyFactory;
/**Engine类的load方法*/
public <R> LoadStatus load(GlideContext glideContext,Object model, Key signature,int width,int height,Class<?> resourceClass,Class<R> transcodeClass,Priority priority,DiskCacheStrategy diskCacheStrategy,Map<Class<?>, Transformation<?>> transformations,boolean isTransformationRequired,boolean isScaleOnlyOrNoTransform, Options options,boolean isMemoryCacheable,boolean useUnlimitedSourceExecutorPool,boolean useAnimationPool,boolean onlyRetrieveFromCache,ResourceCallback cb) {
    Util.assertMainThread();
    long startTime = VERBOSE_IS_LOGGABLE ? LogTime.getLogTime() : 0;

    EngineKey key = keyFactory.buildKey(model, signature, width, height, transformations,
        resourceClass, transcodeClass, options);
    //省略部分代码
    ..........
  }
/**EngineKey类*/  
class EngineKey implements Key {
  private final Object model;
  private final int width;
  private final int height;
  private final Class<?> resourceClass;
  private final Class<?> transcodeClass;
  private final Key signature;
  private final Map<Class<?>, Transformation<?>> transformations;
  private final Options options;
  private int hashCode;

  EngineKey(
      Object model,
      Key signature,
      int width,
      int height,
      Map<Class<?>, Transformation<?>> transformations,
      Class<?> resourceClass,
      Class<?> transcodeClass,
      Options options) {
    this.model = Preconditions.checkNotNull(model);
    this.signature = Preconditions.checkNotNull(signature, "Signature must not be null");
    this.width = width;
    this.height = height;
    this.transformations = Preconditions.checkNotNull(transformations);
    this.resourceClass =
        Preconditions.checkNotNull(resourceClass, "Resource class must not be null");
    this.transcodeClass =
        Preconditions.checkNotNull(transcodeClass, "Transcode class must not be null");
    this.options = Preconditions.checkNotNull(options);
  }

  @Override
  public boolean equals(Object o) {
    if (o instanceof EngineKey) {
      EngineKey other = (EngineKey) o;
      return model.equals(other.model)
          && signature.equals(other.signature)
          && height == other.height
          && width == other.width
          && transformations.equals(other.transformations)
          && resourceClass.equals(other.resourceClass)
          && transcodeClass.equals(other.transcodeClass)
          && options.equals(other.options);
    }
    return false;
  }

  @Override
  public int hashCode() {
    if (hashCode == 0) {
      hashCode = model.hashCode();
      hashCode = 31 * hashCode + signature.hashCode();
      hashCode = 31 * hashCode + width;
      hashCode = 31 * hashCode + height;
      hashCode = 31 * hashCode + transformations.hashCode();
      hashCode = 31 * hashCode + resourceClass.hashCode();
      hashCode = 31 * hashCode + transcodeClass.hashCode();
      hashCode = 31 * hashCode + options.hashCode();
    }
    return hashCode;
  }
}  
```
- 由以上源码我们知道，通过EngineKeyFactory的buildKey方法Glide创建了缓存的Key实现类EngineKey对象，由生成EngineKey对象传入的参数我们可以明白，只要有一个参数不同，所生成的EngineKey对象都会是不同的。内存的速度是最快的，理所当然如果内存中有缓存的对应加载图片Glide会搜先从内存缓存中加载。
### LRU
- Glide[官方文档](https://bumptech.github.io/glide/doc/caching.html)说明
![资源管理官方描述](https://github.com/maoqitian/MaoMdPhoto/raw/39c2ad856ab065e47c96d50f8e28cd151c142750/%E4%BB%8E%E6%BA%90%E7%A0%81%E8%A7%92%E5%BA%A6%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Glide/%E8%B5%84%E6%BA%90%E7%AE%A1%E7%90%86%E6%8F%8F%E8%BF%B0.png)
#### LRU算法思想
- 从官方文档描述，我们可以知道Glide的缓存底层实现原理算法都是LRU（Least Recently Used），字面意思为最近最少使用。算法核心思想（个人理解）：**在一个有限的集合中，存入缓存，每一个缓存都有唯一标识，当要获取一个缓存，集合中没有则存入，有则直接从集合获取，存入缓存到集合时如果集合已经满了则找到集合中最近最少的缓存删除并存入需要存入的缓存。** 这样也就有效的避免了内存溢出（OOM）的问题。
- 接下来我们看一张图能够更好的理解LRU算法

![LRU图](https://github.com/maoqitian/MaoMdPhoto/raw/master/%E4%BB%8E%E6%BA%90%E7%A0%81%E8%A7%92%E5%BA%A6%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Glide/LRU%E5%9B%BE.jpg)
- 横线上方每个数字代表要存入的数据，横线下方代表三个内存页（也可以理解为缓存结合），缓存集合最多可以存入三个缓存数据，则从1开始依次按照数字代码的缓存读取并存入缓存集合，首先开始时三个页内存是空的，前三个缓存数据不同，依次存入缓存集合，当数字4在内存中并进行缓存时，根据LRU算法思想，则2和3相较于1使用时间间隔更少，所以淘汰1，缓存数据4替换1的位置，接下去同理。
- Glide内存缓存使用的是LruCache，磁盘缓存使用的DiskLruCache，他们核心思想都是LRU算法，而缓存集合使用的是LinkedHashMap，熟悉[集合框架](https://www.jianshu.com/p/972dfad0c95b)应该都明白LinkedHashMap集成HashMap，并且LinkedHashMap保证了key的唯一性，更符合LRU算法的实现。
- 更深入的了解LruCache和DiskLruCache可以查看郭霖大神的解析[Android DiskLruCache完全解析，硬盘缓存的最佳方案](https://blog.csdn.net/guolin_blog/article/details/28863651)
### 内存缓存
#### 内存缓存相关API
```
//跳过内存缓存
RequestOptions requestOptions =new RequestOptions().skipMemoryCache(true);
Glide.with(this).load(IMAGE_URL).apply(requestOptions).into(imageView);
//Generated API 方式
GlideApp.with(this).load(IMAGE_URL).skipMemoryCache(true).into(imageView); 
//清除内存缓存，必须在主线程中调用
Glide.get(context).clearMemory();
```
#### 内存缓存源码分析
- 内存缓存不需要你进行任何设置，它默认就是开启的，我们再次回到Engine类的load方法
```
/**Engine类的load方法*/
public <R> LoadStatus load(GlideContext glideContext,Object model, Key signature,int width,int height,Class<?> resourceClass,Class<R> transcodeClass,Priority priority,DiskCacheStrategy diskCacheStrategy,Map<Class<?>, Transformation<?>> transformations,boolean isTransformationRequired,boolean isScaleOnlyOrNoTransform, Options options,boolean isMemoryCacheable,boolean useUnlimitedSourceExecutorPool,boolean useAnimationPool,boolean onlyRetrieveFromCache,ResourceCallback cb) {
    //省略部分代码
    ..........
    EngineResource<?> active = loadFromActiveResources(key, isMemoryCacheable);
    if (active != null) {
      cb.onResourceReady(active, DataSource.MEMORY_CACHE);
      if (VERBOSE_IS_LOGGABLE) {
        logWithTimeAndKey("Loaded resource from active resources", startTime, key);
      }
      return null;
    }
    //省略部分代码
    ..........
  }
```
##### 活动资源 (Active Resources)
- 通过以上Engine类load的源码，首先调用loadFromActiveResources方法来从内存中获取缓存
```
private final ActiveResources activeResources;
/**Engine类的loadFromActiveResources方法*/
@Nullable
  private EngineResource<?> loadFromActiveResources(Key key, boolean isMemoryCacheable) {
    if (!isMemoryCacheable) {
      return null;
    }
    EngineResource<?> active = activeResources.get(key);
    if (active != null) {
      active.acquire();
    }
    return active;
  }
/**ActiveResources类*/
final class ActiveResources {
     @VisibleForTesting
  final Map<Key, ResourceWeakReference> activeEngineResources = new HashMap<>();
   //省略部分代码
    ........
   @VisibleForTesting
  static final class ResourceWeakReference extends WeakReference<EngineResource<?>> {
    //省略部分代码
    ........
  }
}
/**Engine类的onEngineJobComplete方法*/
@SuppressWarnings("unchecked")
  @Override
  public void onEngineJobComplete(EngineJob<?> engineJob, Key key, EngineResource<?> resource) {
    Util.assertMainThread();
    // A null resource indicates that the load failed, usually due to an exception.
    if (resource != null) {
      resource.setResourceListener(key, this);

      if (resource.isCacheable()) {
        activeResources.activate(key, resource);
      }
    }

    jobs.removeIfCurrent(key, engineJob);
  }

/**RequestOptions类的skipMemoryCache方法*/
public RequestOptions skipMemoryCache(boolean skip) {
    if (isAutoCloneEnabled) {
      return clone().skipMemoryCache(true);
    }
    this.isCacheable = !skip;
    fields |= IS_CACHEABLE;
    return selfOrThrowIfLocked();
  }
```
- 通过以上源码， 这里需要分几步来解读，首先如果是第一次加载，肯定没有内存缓存，所以如果第一次加载成功，则在加载成功之后调用了Engine对象的onEngineJobComplete方法，并在该方法中将加载成功的resource通过ActiveResources对象的activate方法保存在其内部维护的弱引用（WeakReference）HashMap中。下次再加载相同的资源，当你设置了skipMemoryCache（true），则表明你不想使用内存缓存，这时候Glide再次加载相同资源的时候则会跳过内存缓存的加载，否则可以从ActiveResources对象中获取，如果内存资源没被回收的话（关于弱引用的一下描述可以看看我以前写的一篇文章[Android 学习笔记之图片三级缓存](https://www.jianshu.com/p/bfc3e8cd64ae)）。如果该弱引用资源被回收了(GC)，则下一步就到内存中寻找是否有该资源的缓存。

##### 内存缓存 (Memory cache)
- 接着回到Engine类的load方法，如果弱引用缓存资源已经被回收，则调用loadFromCache方法在内存缓存中查找缓存资源
```
/**Engine类的load方法*/
public <R> LoadStatus load(GlideContext glideContext,Object model, Key signature,int width,int height,Class<?> resourceClass,Class<R> transcodeClass,Priority priority,DiskCacheStrategy diskCacheStrategy,Map<Class<?>, Transformation<?>> transformations,boolean isTransformationRequired,boolean isScaleOnlyOrNoTransform, Options options,boolean isMemoryCacheable,boolean useUnlimitedSourceExecutorPool,boolean useAnimationPool,boolean onlyRetrieveFromCache,ResourceCallback cb) {
    //省略部分代码
    ..........
    EngineResource<?> cached = loadFromCache(key, isMemoryCacheable);
    if (cached != null) {
      cb.onResourceReady(cached, DataSource.MEMORY_CACHE);
      if (VERBOSE_IS_LOGGABLE) {
        logWithTimeAndKey("Loaded resource from cache", startTime, key);
      }
      return null;
    }
    //省略部分代码
    ..........
  }
/**Engine类的loadFromCache方法*/
private EngineResource<?> loadFromCache(Key key, boolean isMemoryCacheable) {
    if (!isMemoryCacheable) {
      return null;
    }

    EngineResource<?> cached = getEngineResourceFromCache(key);
    if (cached != null) {
      cached.acquire();
      activeResources.activate(key, cached);
    }
    return cached;
  }
/**Engine类的getEngineResourceFromCache方法*/
private final MemoryCache cache;
private EngineResource<?> getEngineResourceFromCache(Key key) {
    Resource<?> cached = cache.remove(key);

    final EngineResource<?> result;
    if (cached == null) {
      result = null;
    } else if (cached instanceof EngineResource) {
      // Save an object allocation if we've cached an EngineResource (the typical case).
      result = (EngineResource<?>) cached;
    } else {
      result = new EngineResource<>(cached, true /*isMemoryCacheable*/, true /*isRecyclable*/);
    }
    return result;
  }
/**GlideBuilder类的build方法*/
private  MemoryCache cache;
@NonNull
  Glide build(@NonNull Context context) {
    //省略部分代码
    ..........
    if (memoryCache == null) {
      memoryCache = new LruResourceCache(memorySizeCalculator.getMemoryCacheSize());
    }

    if (diskCacheFactory == null) {
      diskCacheFactory = new InternalCacheDiskCacheFactory(context);
    }

    if (engine == null) {
      engine =
          new Engine(
              memoryCache,
              diskCacheFactory,
              diskCacheExecutor,
              sourceExecutor,
              GlideExecutor.newUnlimitedSourceExecutor(),
              GlideExecutor.newAnimationExecutor(),
              isActiveResourceRetentionAllowed);
    }
    //省略部分代码
    ..........
}    
/**LruResourceCache类的实现继承关系*/    
public class LruResourceCache extends LruCache<Key, Resource<?>> implements MemoryCache{......}    
```
- 通过以上源码，在loadFromCache同样也判断了Glide是否设置了skipMemoryCache（true）方法，没有设置则调用getEngineResourceFromCache方法，在该方法中我们可以看到cache对象就是MemoryCache对象，而该对象实际是一个接口，他的实现类是LruResourceCache，该对象我们前面在GlideBuilder的build方法中进行了新建（在第一步with方法中调用了Glide.get方法，在get方法中初始化Glide调用了在GlideBuilder的build方法），这里也就说明Glide的内存缓存还是使用LruCache来实现，这里如果获取到了内存缓存，则获取内容缓存的同时移除该缓存，并在loadFromCache方法中将该资源标记为正在使用同时加入在弱引用中。这样在ListView或者Recyclerview中加载图片则下次加载首先从弱引用Map中获取缓存资源，并且标志当前资源正在使用，可以防止该资源被LRU算法回收掉。
##### 内存缓存写入
- 前面我们只是分析了如何获取内存缓存，而内存缓存又是在哪里写入的呢？根据前面分析，首先获取在弱引用Map中的缓存资源，而前面我们在分析活动资源(Active Resources)时候已经说过是在onEngineJobComplete放中往弱引用Map存放缓存资源，而
onEngineJobComplete方法是在哪里调用呢，这我们就要回想起[上一篇文章](https://www.maoqitian.com/2019/02/19/%E4%BB%8E%E6%BA%90%E7%A0%81%E8%A7%92%E5%BA%A6%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3glide%EF%BC%88%E4%B8%8A%EF%BC%89/)中我们再网络加载图片成功后腰切换在主线程回调来显示图片，也就是EngineJob对象的handleResultOnMainThread方法
```
/**EngineJob类的handleResultOnMainThread方法*/
@Synthetic
  void handleResultOnMainThread() {
    //省略部分代码
    ..........
    engineResource = engineResourceFactory.build(resource, isCacheable);
    hasResource = true;
    //省略部分代码
    ..........
    engineResource.acquire();
    listener.onEngineJobComplete(this, key, engineResource);
    engineResource.release();
    //省略部分代码
    ..........
  }
/**EngineJob类的EngineResourceFactory内部类*/  
@VisibleForTesting
  static class EngineResourceFactory {
    public <R> EngineResource<R> build(Resource<R> resource, boolean isMemoryCacheable) {
      return new EngineResource<>(resource, isMemoryCacheable, /*isRecyclable=*/ true);
    }
  }
/**Engine类的onEngineJobComplete方法*/
@SuppressWarnings("unchecked")
  @Override
  public void onEngineJobComplete(EngineJob<?> engineJob, Key key, EngineResource<?> resource) {
    //省略部分代码
    ..........
    if (resource != null) {
      resource.setResourceListener(key, this);
      if (resource.isCacheable()) {
        activeResources.activate(key, resource);
      }
    }
    //省略部分代码
    ..........
  }  
```
- 通过以上源码，EngineJob类的handleResultOnMainThread方法首先构建了获取好的包含图片的资源，标记当前资源正在使用，通过listener.onEngineJobComplete回调，而listener就是Engine对象，也就到了Engine类的onEngineJobComplete方法，并在该方法中存入了图片资源到弱引用Map中。
- 上面我是分析了弱引用资源的缓存存入，接着我们看看内存缓存是在哪里存入的，在次看回handleResultOnMainThread方法，我们看到onEngineJobComplete回调前后分别调用了EngineResource对象的acquire方法和release方法
```
/**EngineResource类的acquire方法*/
void acquire() {
    if (isRecycled) {
      throw new IllegalStateException("Cannot acquire a recycled resource");
    }
    if (!Looper.getMainLooper().equals(Looper.myLooper())) {
      throw new IllegalThreadStateException("Must call acquire on the main thread");
    }
    ++acquired;
  }
/**EngineResource类的release方法*/
  void release() {
    if (acquired <= 0) {
      throw new IllegalStateException("Cannot release a recycled or not yet acquired resource");
    }
    if (!Looper.getMainLooper().equals(Looper.myLooper())) {
      throw new IllegalThreadStateException("Must call release on the main thread");
    }
    if (--acquired == 0) {
      listener.onResourceReleased(key, this);
    }
  }
/**Engine类的onResourceReleased方法*/
@Override
  public void onResourceReleased(Key cacheKey, EngineResource<?> resource) {
    Util.assertMainThread();
    activeResources.deactivate(cacheKey);
    if (resource.isCacheable()) {
      cache.put(cacheKey, resource);
    } else {
      resourceRecycler.recycle(resource);
    }
  }  
```
- 通过以上源码，其实我们应该能恍然大悟，Glide的内存缓存存入其实就是通过一个acquired变量来进行控制，如果当前弱引用资源不再使用，也就是acquired等于零的时候，则调用回调listener.onResourceReleased（listener就是Engine对象），在onResourceReleased方法中移除了弱引用资源资源，并且没有设置skipMemoryCache（true），则通过cache.put存入内存缓存。
- 总的来说Glide的内存缓存主要是结合了弱引用和内存来实现的。
##### Glide内存缓存机制示意图
![Glide内存缓存机制示意图](https://github.com/maoqitian/MaoMdPhoto/raw/master/%E4%BB%8E%E6%BA%90%E7%A0%81%E8%A7%92%E5%BA%A6%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Glide/Glide%E5%86%85%E5%AD%98%E7%BC%93%E5%AD%98%E6%9C%BA%E5%88%B6%E7%A4%BA%E6%84%8F%E5%9B%BE.jpg)
### 磁盘缓存
- 说去磁盘缓存，上一篇文章我们在简单使用Glide的例子中就已经使用了Glide的磁盘缓存
```
RequestOptions requestOptions = new RequestOptions()
         .diskCacheStrategy(DiskCacheStrategy.NONE);//不使用缓存
     Glide.with(Context).load(IMAGE_URL).apply(requestOptions).into(mImageView);
```
- 既然知道如何使用Glide的磁盘缓存，首先我们要了解Glide4中给我提供了哪几种磁盘缓存策略
#### 磁盘缓存策略
- 1.DiskCacheStrategy.NONE： 表示不使用磁盘缓存
- 2.DiskCacheStrategy.DATA： 表示磁盘缓存只缓存原始加载的图片
- DiskCacheStrategy.RESOURCE： 表示磁盘缓存只缓存经过解码转换后的图片
- DiskCacheStrategy.ALL： 表示磁盘缓存既缓存原始图片，也缓存经过解码转换后的图片
- DiskCacheStrategy.AUTOMATIC： 表示让Glide根据图片资源智能地选择使用哪一种磁盘缓存策略，该选项也是我们在不进行手动设置的时候Glide的默认设置
#### 磁盘缓存源码分析
- 不知你是否还记得上一篇文章中在加载图片的时候我们是在开启子线程任务在线程池中进行的，我们来回顾一下
```
/**DecodeJob类的start方法*/
public void start(DecodeJob<R> decodeJob) {
    this.decodeJob = decodeJob;
    GlideExecutor executor = decodeJob.willDecodeFromCache()
        ? diskCacheExecutor
        : getActiveSourceExecutor();
    executor.execute(decodeJob);
  }
/**DecodeJob类的willDecodeFromCache方法*/ 
 boolean willDecodeFromCache() {
    Stage firstStage = getNextStage(Stage.INITIALIZE);
    return firstStage == Stage.RESOURCE_CACHE || firstStage == Stage.DATA_CACHE;
  } 
/**DecodeJob类的getNextStage方法*/   
private Stage getNextStage(Stage current) {
    switch (current) {
      case INITIALIZE:
        return diskCacheStrategy.decodeCachedResource()
            ? Stage.RESOURCE_CACHE : getNextStage(Stage.RESOURCE_CACHE);
      case RESOURCE_CACHE:
        return diskCacheStrategy.decodeCachedData()
            ? Stage.DATA_CACHE : getNextStage(Stage.DATA_CACHE);
      case DATA_CACHE:
        return onlyRetrieveFromCache ? Stage.FINISHED : Stage.SOURCE;
     //省略部分代码
    ......
    }
  }
/**DiskCacheStrategy类的ALL对象*/     
public static final DiskCacheStrategy ALL = new DiskCacheStrategy() {
    @Override
    public boolean isDataCacheable(DataSource dataSource) {
      return dataSource == DataSource.REMOTE;
    }

    @Override
    public boolean isResourceCacheable(boolean isFromAlternateCacheKey, DataSource dataSource,
        EncodeStrategy encodeStrategy) {
      return dataSource != DataSource.RESOURCE_DISK_CACHE && dataSource != DataSource.MEMORY_CACHE;
    }

    @Override
    public boolean decodeCachedResource() {
      return true;
    }

    @Override
    public boolean decodeCachedData() {
      return true;
    }
  };  
/**GlideBuilder类的build方法*/  
@NonNull
  Glide build(@NonNull Context context) {
    //省略部分代码
    ......
    if (diskCacheExecutor == null) {
      diskCacheExecutor = GlideExecutor.newDiskCacheExecutor();
    }
    //省略部分代码
    ......
}
/**GlideExecutor类的newDiskCacheExecutor方法*/ 
private static final int DEFAULT_DISK_CACHE_EXECUTOR_THREADS = 1;
public static GlideExecutor newDiskCacheExecutor() {
    return newDiskCacheExecutor(
        DEFAULT_DISK_CACHE_EXECUTOR_THREADS,
        DEFAULT_DISK_CACHE_EXECUTOR_NAME,
        UncaughtThrowableStrategy.DEFAULT);
  }
/**GlideExecutor类的newDiskCacheExecutor方法*/ 
 public static GlideExecutor newDiskCacheExecutor(
      int threadCount, String name, UncaughtThrowableStrategy uncaughtThrowableStrategy) {
    return new GlideExecutor(
        new ThreadPoolExecutor(
            threadCount /* corePoolSize */,
            threadCount /* maximumPoolSize */,
            0 /* keepAliveTime */,
            TimeUnit.MILLISECONDS,
            new PriorityBlockingQueue<Runnable>(),
            new DefaultThreadFactory(name, uncaughtThrowableStrategy, true)));
  }  
```
- 通过以上源码，可以分两个步骤来进行解读：
   - 第一步在DecodeJob对象的start方法开启子线程来加载图片，这里使用了线程池，通过willDecodeFromCache方法和getNextStage放结合，主要通过Stage枚举来判断当前使用的缓存策略，而缓存策略的设置则通过DiskCacheStrategy对象的**decodeCachedResource和decodeCachedData方法来进行设置**，而这两个方法在DiskCacheStrategy抽象类都是抽象方法，而他们的实现就是我们前面提到的Glide磁盘缓存的五种策略，上面代码中列出其中一种ALL代码，decodeCachedResource和decodeCachedData方法都返回ture，也就说明磁盘缓存既缓存原始图片，也缓存经过解码转换后的图片；如果decodeCachedResource返回false和decodeCachedData方法返回true，也就代表DATA策略，磁盘缓存只缓存原始加载的图片，其他同理
   - 第二步通过前面对设置策略的判断，如果有缓存策略，则拿到的线程池就是磁盘缓存加载的线程池（线程池的理解可以看看我以前写的一篇[文章](https://www.maoqitian.com/2019/01/20/Java中的线程池/)），该线程的初始化还是在GlideExecutor对象的build方法中，通过以上源码，该线程池只有唯一一个核心线程，这就保证所有执行的的任务都在这一个线程中执行，并且是顺序执行，也就不用在考虑线程同步的问题了。
- 根据前面官网的说明，不管是内存缓存还是磁盘缓存，都是使用LRU，接着看看Glide磁盘缓存在哪获取LRU对象，还是得看到GlideBuilder对象的build方法

```
/**GlideBuilder类的build方法*/  
private DiskCache.Factory diskCacheFactory;
@NonNull
  Glide build(@NonNull Context context) {
    //省略部分代码
    ..........
    if (diskCacheFactory == null) {
      diskCacheFactory = new InternalCacheDiskCacheFactory(context);
    }
    //省略部分代码
    ..........
}
/**InternalCacheDiskCacheFactory类的继承关系*/
public final class InternalCacheDiskCacheFactory extends DiskLruCacheFactory {
   //省略实现代码
    ..........  
}
/**DiskLruCacheFactory类的部分代码*/
public class DiskLruCacheFactory implements DiskCache.Factory {
   //省略部分代码
    ..........
    @Override
  public DiskCache build() {
    File cacheDir = cacheDirectoryGetter.getCacheDirectory();
    //省略部分代码
    ..........
    return DiskLruCacheWrapper.create(cacheDir, diskCacheSize);
  }
}
/**DiskLruCacheWrapper类的部分代码*/
public class DiskLruCacheWrapper implements DiskCache {
//省略部分代码
    ..........
   private synchronized DiskLruCache getDiskCache() throws IOException {
    if (diskLruCache == null) {
      diskLruCache = DiskLruCache.open(directory, APP_VERSION, VALUE_COUNT, maxSize);
    }
    return diskLruCache;
  } 
  //省略部分代码
    ..........
}
/**DiskLruCache类的部分代码*/
public final class DiskLruCache implements Closeable {
    //省略部分代码
    ..........
    public static DiskLruCache open(File directory, int appVersion, int valueCount, long maxSize)
      throws IOException {
          DiskLruCache cache = new DiskLruCache(directory, appVersion, valueCount, maxSize);
          //省略部分代码
          ..........
          return cache;
      }
      //省略部分代码
      ..........
}

```
- 通过以上源码，我们在GlideBuilder对象的build方法中已经新建了InternalCacheDiskCacheFactory对象，也就是DiskLruCacheFactory对象，同样该对象已经被我们传入Engine对象的构造方法中，最终包装成LazyDiskCacheProvider对象（该对象代码就不贴出了），所以只要调用DiskLruCacheFactory对象的build方法就能够最终获取到**DiskLruCache对象**，该对象是Glide自己实现的，但是其原理和谷歌官方推荐的DiskLruCache也差不了太多，核心还是使用LRU算法来实现磁盘缓存。
##### 资源类型（Resource）
- 根据前面分分析，假定没有内存缓存，而是由磁盘缓存，则结合前面分析我们得到了磁盘缓存处理的线程池，也获得枚举Stage是RESOURCE_CACHE或DATA_CACHE，则在DecodeJob对象getNextGenerator方法，我们就能得到对应的Generator
```
/**DecodeJob的getNextGenerator方法*/
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
  }
```
- 通过getNextGenerator方法的源码，如果之前设置磁盘缓存策略为DiskCacheStrategy.RESOURCE，则应该对应的就是枚举Stage.RESOURCE_CACHE，也就是说接下来使用的资源Generator是ResourceCacheGenerator，结合[上一篇文章](https://www.maoqitian.com/2019/02/19/%E4%BB%8E%E6%BA%90%E7%A0%81%E8%A7%92%E5%BA%A6%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3glide%EF%BC%88%E4%B8%8A%EF%BC%89/)，我们分析网络加载流程是这里获取的是SourceGenerator，我们接着来看ResourceCacheGenerator的startNext()方法
```
/** ResourceCacheGenerator类的startNext方法*/
  @SuppressWarnings("PMD.CollapsibleIfStatements")
  @Override
  public boolean startNext() {
     //省略部分代码
      ..........
      currentKey =
          new ResourceCacheKey(// NOPMD AvoidInstantiatingObjectsInLoops
              helper.getArrayPool(),
              sourceId,
              helper.getSignature(),
              helper.getWidth(),
              helper.getHeight(),
              transformation,
              resourceClass,
              helper.getOptions());
      cacheFile = helper.getDiskCache().get(currentKey);
      if (cacheFile != null) {
        sourceKey = sourceId;
        modelLoaders = helper.getModelLoaders(cacheFile);
        modelLoaderIndex = 0;
      }
    }

    loadData = null;
    boolean started = false;
    while (!started && hasNextModelLoader()) {
      ModelLoader<File, ?> modelLoader = modelLoaders.get(modelLoaderIndex++);
      loadData = modelLoader.buildLoadData(cacheFile,
          helper.getWidth(), helper.getHeight(), helper.getOptions());
      if (loadData != null && helper.hasLoadPath(loadData.fetcher.getDataClass())) {
        started = true;
        loadData.fetcher.loadData(helper.getPriority(), this);
      }
    }
    return started;
  }
/**DecodeHelper类的getDiskCache方法*/
 DiskCache getDiskCache() {
    return diskCacheProvider.getDiskCache();
  }
/** LazyDiskCacheProvider类的getDiskCache方法 */
@Override
    public DiskCache getDiskCache() {
      if (diskCache == null) {
        synchronized (this) {
          if (diskCache == null) {
            diskCache = factory.build();
          }
          if (diskCache == null) {
            diskCache = new DiskCacheAdapter();
          }
        }
      }
      return diskCache;
    }  
```
- 通过以上源码其实已经很清晰，首先还是获取缓存的唯一key，然后helper.getDiskCache().get(currentKey)这一句话就是获取缓存，helper对象就是DecodeHelper，它的getDiskCache方法获取的对象也就是前面提到的包含DiskLruCacheFactory对象的LazyDiskCacheProvider对象，而LazyDiskCacheProvider对象的getDiskCache方法调用了factory.build()，factory对象DiskLruCacheFactory，也就是获取了我们前面所说的**DiskLruCache对象**。
- 接着继续看数据返回走的流程还是通过回调通cb.onDataFetcherReady将获取的缓存资源传递到DecodeJob，由DecodeJob继续执行剩余图片显示步骤，大致流程和网络加载差不多，这里就不进行讨论了
```
/** ResourceCacheGenerator类的startNext方法*/
 @Override
  public void onDataReady(Object data) {
    cb.onDataFetcherReady(sourceKey, data, loadData.fetcher, DataSource.RESOURCE_DISK_CACHE,
        currentKey);
  }
```
##### 数据来源 (Data)
- 同理资源类型（Resource），则设置磁盘缓存策略为DiskCacheStrategy.DATA，则应该对应的就是枚举Stage.DATA_CACHE，使用的资源Generator是DataCacheGenerator，所以直接看看DataCacheGenerator的startNext()方法，该方法源码如下，同样是根据key通过**DiskLruCache对象**来获取磁盘缓存（DATA），数据返回走的流程还是通过回调通cb.onDataFetcherReady将获取的缓存资源传递到DecodeJob，由DecodeJob继续执行剩余图片显示
```
/** DataCacheGenerator类的startNext方法*/
 @Override
  public boolean startNext() {
      //省略部分代码
      ..........
      Key originalKey = new DataCacheKey(sourceId, helper.getSignature());
      cacheFile = helper.getDiskCache().get(originalKey);
      if (cacheFile != null) {
        this.sourceKey = sourceId;
        modelLoaders = helper.getModelLoaders(cacheFile);
        modelLoaderIndex = 0;
      }
    }

    loadData = null;
    boolean started = false;
    while (!started && hasNextModelLoader()) {
      ModelLoader<File, ?> modelLoader = modelLoaders.get(modelLoaderIndex++);
      loadData =
          modelLoader.buildLoadData(cacheFile, helper.getWidth(), helper.getHeight(),
              helper.getOptions());
      if (loadData != null && helper.hasLoadPath(loadData.fetcher.getDataClass())) {
        started = true;
        loadData.fetcher.loadData(helper.getPriority(), this);
      }
    }
    return started;
  }
/** DataCacheGenerator类的onDataReady方法*/  
@Override
  public void onDataReady(Object data) {
    cb.onDataFetcherReady(sourceKey, data, loadData.fetcher, DataSource.DATA_DISK_CACHE, sourceKey);
  }  
```
##### 磁盘缓存数据存入
- 前面我们只是了解了磁盘缓存的获取，磁盘缓存又是在哪里存入的，接着往下看。
- 根据上一篇文章的分析，加载图片会走到DecodeJob对象的decodeFromRetrievedData方法
```
/** DecodeJob类的decodeFromRetrievedData方法*/  
private void decodeFromRetrievedData() {
    //省略部分代码
      ..........
    notifyEncodeAndRelease(resource, currentDataSource);
    //省略部分代码
      ..........
} 
/** DecodeJob类的notifyEncodeAndRelease方法*/  
private final DeferredEncodeManager<?> deferredEncodeManager = new DeferredEncodeManager<>();
private void notifyEncodeAndRelease(Resource<R> resource, DataSource dataSource) {
//省略部分代码
      ..........
    stage = Stage.ENCODE;
    try {
      if (deferredEncodeManager.hasResourceToEncode()) {
        deferredEncodeManager.encode(diskCacheProvider, options);
      }
    } 
    //省略部分代码
    ..........
  }
/** DeferredEncodeManager类的encode方法**/
void encode(DiskCacheProvider diskCacheProvider, Options options) {
      GlideTrace.beginSection("DecodeJob.encode");
      try {
        diskCacheProvider.getDiskCache().put(key,
            new DataCacheWriter<>(encoder, toEncode, options));
      } finally {
        toEncode.unlock();
        GlideTrace.endSection();
      }
    } 
```
- 通过以上源码可以看到DecodeJob对象的decodeFromRetrievedData方法通过调用notifyEncodeAndRelease方法，在该方法中调用了内部类DeferredEncodeManager的encode方法存入了磁盘缓存，这里存入的是转换后的磁盘缓存（Resource）。
- 原始数据也就是SourceGenerator第一次网络下载成功之后获取的图片数据，之后再做磁盘缓存，所以再次回到看到SourceGenerator的onDataReady方法
```
/**SourceGenerator类的onDataReady方法**/ 
private Object dataToCache;
@Override
  public void onDataReady(Object data) {
    DiskCacheStrategy diskCacheStrategy = helper.getDiskCacheStrategy();
    if (data != null && diskCacheStrategy.isDataCacheable(loadData.fetcher.getDataSource())) {
      dataToCache = data;
      cb.reschedule();
    } else {
      cb.onDataFetcherReady(loadData.sourceKey, data, loadData.fetcher,
          loadData.fetcher.getDataSource(), originalKey);
    }
  }  
/**SourceGenerator类的startNext方法**/
@Override
  public boolean startNext() {
    if (dataToCache != null) {
      Object data = dataToCache;
      dataToCache = null;
      cacheData(data);
    }

    if (sourceCacheGenerator != null && sourceCacheGenerator.startNext()) {
      return true;
    }
   //省略部分代码
  ..........
  }
/**SourceGenerator类的cacheData方法**/  
private void cacheData(Object dataToCache) {
    long startTime = LogTime.getLogTime();
    try {
      Encoder<Object> encoder = helper.getSourceEncoder(dataToCache);
      DataCacheWriter<Object> writer =
          new DataCacheWriter<>(encoder, dataToCache, helper.getOptions());
      originalKey = new DataCacheKey(loadData.sourceKey, helper.getSignature());
      helper.getDiskCache().put(originalKey, writer);
    } 
    //省略部分代码
    ..........
    sourceCacheGenerator =
        new DataCacheGenerator(Collections.singletonList(loadData.sourceKey), helper, this);
  }
/**DecodeJob类的reschedule方法**/ 
@Override
  public void reschedule() {
    runReason = RunReason.SWITCH_TO_SOURCE_SERVICE;
    callback.reschedule(this);
  }
/**Engine类的reschedule方法**/   
 @Override
  public void reschedule(DecodeJob<?> job) {
    getActiveSourceExecutor().execute(job);
  }  
```
- 通过以上源码，其实逻辑已经很清晰，会让你有“柳暗花明又一村”的感觉，onDataReady网络请求成功并且设置了缓存策略，则将图片资源赋值给Object类型的dataToCache，执行回调cb.reschedule，cb就是DecodeJob对象，所以接着执行了DecodeJob对象的reschedule方法，该方法再次执行回调也就是执行了Engine对象的reschedule方法，该方法再次执行DecodeJob，也就会再次触发SourceGenerator类的startNext方法，该方法首先判断了Object类型的dataToCache是否有值，前面分析该对象已经赋值，所以就进入到SourceGenerator对象的cacheData方法存入了我们的原始下载图片的缓存。
#### 仅从缓存加载图片
- 前面我基本把Glide的缓存模块梳理了一遍，但是还差个东西，那就是如果我只想Glide加载缓存呢？这种需求还是有的，比如说我们在有些应用看到的省流量模式，不就是正好对应这个需求，没关系Gldie也已经为我们考虑到了，那就是onlyRetrieveFromCache(true)，只要设置了这个，图片在内存缓存或在磁盘缓存中就会被加载出来，而没有缓存，则这一次加载失败。
- 我们看看如何使用
```
RequestOptions requestOptions = new RequestOptions().onlyRetrieveFromCache(true);
Glide.with(this).load(IMAGE_URL).apply(requestOptions).into(mImageView);
//Generated API 方式        
GlideApp.with(this)
  .load(url)
  .diskCacheStrategy(DiskCacheStrategy.ALL)
  .into(mImageView);        
```
- 使用起来还是很方便的，只要设置onlyRetrieveFromCache(true)方法就行，而它的原理也其实也很简单，我们再次回到DecodeJob对象的getNextStage方法，如果前面获取了缓存，则相应得到对应的Generator加载图片，如果获取不到缓存，则枚举Stage.FINISHED，DecodeJob对象的getNextGenerator方法则会返回null。(如下代码所示)
```
/**DecodeJob类的getNextStage方法**/ 
private Stage getNextStage(Stage current) {
    switch (current) {
    //省略部分代码
    ..........
      case DATA_CACHE:
        return onlyRetrieveFromCache ? Stage.FINISHED : Stage.SOURCE;
      case SOURCE:
      case FINISHED:
        return Stage.FINISHED;
    }
  }
/**DecodeJob类的getNextGenerator方法**/   
private DataFetcherGenerator getNextGenerator() {
    switch (stage) {
    //省略部分代码
    ..........
      case FINISHED:
        return null;
      default:
        throw new IllegalStateException("Unrecognized stage: " + stage);
    }
  }  
```
##### Glide磁盘缓存机制示意图
![Glide磁盘缓存机制示意图](https://github.com/maoqitian/MaoMdPhoto/raw/master/%E4%BB%8E%E6%BA%90%E7%A0%81%E8%A7%92%E5%BA%A6%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Glide/Glide%E7%A3%81%E7%9B%98%E7%BC%93%E5%AD%98%E6%9C%BA%E5%88%B6%E7%A4%BA%E6%84%8F%E5%9B%BE.jpg)
### Glide缓存小结
- 通过前面对Glide缓存的分析，让我再次认识到Glide的强大，使用时只是简单的几个方法设置或者不设置，Glide都能够在背后依靠其复杂的逻辑为我们快速的加载出图片并显示，缓存还有一些细节比如可以自定义key等，这里就不进行展开了，有兴趣的可以自行研究。

## Glide 回调与监听
### 图片加载成功回调原理 
- 由上一篇文章分析，我们来回顾一下图片加载成功之后的逻辑。数据加载成功之后切换主线程最终调用SingleRequest类的onResourceReady方法，在该方法中加载成功的数据通过target.onResourceReady方法将数据加载出来，target就是DrawableImageViewTarget对象，他继续了实现了Target接口的基类ImageViewTarget，所以调用它实现的onResourceReady方法或者父类实现的onResourceReady方法就实现了加载成功数据的回调，并由DrawableImageViewTarget对象显示加载成功的图片，这就是数据加载成功回调原理。 
```
/**SingleRequest类的onResourceReady方法**/    
@Nullable 
private List<RequestListener<R>> requestListeners;
private void onResourceReady(Resource<R> resource, R result, DataSource dataSource) {
    //省略部分代码
    ..........
    isCallingCallbacks = true;
    try {
      boolean anyListenerHandledUpdatingTarget = false;
      //省略部分代码
     ..........

      if (!anyListenerHandledUpdatingTarget) {
        Transition<? super R> animation =
            animationFactory.build(dataSource, isFirstResource);
        target.onResourceReady(result, animation);
      }
    } 
    //省略部分代码
    ..........
  }
/**Target 接口**/      
public interface Target<R> extends LifecycleListener {}
/**ImageViewTarget类的onResourceReady方法**/ 
@Override
  public void onResourceReady(@NonNull Z resource, @Nullable Transition<? super Z> transition) {
    if (transition == null || !transition.transition(resource, this)) {
      setResourceInternal(resource);
    } else {
      maybeUpdateAnimatable(resource);
    }
  }
/**ImageViewTarget类的setResourceInternal方法**/ 
private void setResourceInternal(@Nullable Z resource) {
    setResource(resource);
    maybeUpdateAnimatable(resource);
  }
DrawableImageViewTarget
/**DrawableImageViewTarget类的setResource方法**/ 
 @Override
  protected void setResource(@Nullable Drawable resource) {
    view.setImageDrawable(resource);
  }  
```
### Glide监听（listener）
- 再次看看Glide监听（listener）的例子
```
Glide.with(this).load(IMAGE_URL).listener(new RequestListener<Drawable>() {
           @Override
           public boolean onLoadFailed(@Nullable GlideException e, Object model, Target<Drawable> target, boolean isFirstResource) {
               return false;
           }

           @Override
           public boolean onResourceReady(Drawable resource, Object model, Target<Drawable> target, DataSource dataSource, boolean isFirstResource) {
               return false;
           }
       }).into(mImageView);
```
- Glide监听的实现同样还是基于我们上面分析的SingleRequest对象的onResourceReady方法，使用的时候调用RequestBuilder对象的listener方法，传入的RequestListener对象加入到requestListeners，这样在SingleRequest对象的onResourceReady方法中遍历requestListeners，来回调listener.onResourceReady方法，布尔类型的anyListenerHandledUpdatingTarget则接收回调listener.onResourceReady方法的返回值，如果返回true，则不会执会往下执行，则接着的into方法就不会被触发，说明我们自己在监听中处理，返回false则不拦截。
```
/**RequestBuilder类的listener方法**/ 
@Nullable 
private List<RequestListener<TranscodeType>> requestListeners;
  public RequestBuilder<TranscodeType> listener(
      @Nullable RequestListener<TranscodeType> requestListener) {
    this.requestListeners = null;
    return addListener(requestListener);
  }
/**RequestBuilder类的addListener方法**/  
  public RequestBuilder<TranscodeType> addListener(
      @Nullable RequestListener<TranscodeType> requestListener) {
    if (requestListener != null) {
      if (this.requestListeners == null) {
        this.requestListeners = new ArrayList<>();
      }
      this.requestListeners.add(requestListener);
    }
    return this;
  }

/**SingleRequest类的onResourceReady方法**/    
@Nullable 
private List<RequestListener<R>> requestListeners;
private void onResourceReady(Resource<R> resource, R result, DataSource dataSource) {
    //省略部分代码
    ..........
    isCallingCallbacks = true;
    try {
      boolean anyListenerHandledUpdatingTarget = false;
      if (requestListeners != null) {
        for (RequestListener<R> listener : requestListeners) {
          anyListenerHandledUpdatingTarget |=
              listener.onResourceReady(result, model, target, dataSource, isFirstResource);
        }
      }
      anyListenerHandledUpdatingTarget |=
          targetListener != null
              && targetListener.onResourceReady(result, model, target, dataSource, isFirstResource);

      if (!anyListenerHandledUpdatingTarget) {
        Transition<? super R> animation =
            animationFactory.build(dataSource, isFirstResource);
        target.onResourceReady(result, animation);
      }
    } 
    //省略部分代码
    ..........
  }
```
## Target（目标）
- Target在Glide中相当于中间人的作用，在图片的展示起到承上启下的功效，首先看看Target接口的继承关系图

![Target继承关系图](https://github.com/maoqitian/MaoMdPhoto/raw/master/%E4%BB%8E%E6%BA%90%E7%A0%81%E8%A7%92%E5%BA%A6%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Glide/Target%E7%BB%A7%E6%89%BF%E5%85%B3%E7%B3%BB%E5%9B%BE.png)
- 通过该图，我们可以把Target分为三类，一种是简单的Target，一种是加载到特定View的Target(ViewTarget)，还有一种是FutureTarget，可以知道异步执行的结果，得到缓存文件
- 上一篇文章分析into方法时我们是分析into(ImageView)这个方法开始的，它内部还是会得到特定的Target对象，也就是我们一直说的DrawableImageViewTarget，而他是属于ViewTarget的子类
### 简单的Target（SimpleTarget）
- SimpleTarget其实是在给我们更灵活的加载到各种各样对象准备的，只要指定我们加载获取的是什么对象asBitmap()，就能使用SimpleTarge或者集成它的我们自定义的对象，在其中通过获取的Bitmap显示在对应的控件上，比如上一篇文章例子提到的NotifivationTarget，就是加载到指定的Notifivation中，灵活加载。
```
//注意需要指定Glide的加载类型asBitmap，不指定Target不知道本身是是类型的Target
Glide.with(this).asBitmap().load(IMAGE_URL).into(new SimpleTarget<Bitmap>() {
            @Override
            public void onResourceReady(@NonNull Bitmap resource, @Nullable Transition<? super Bitmap> transition) {
            //加载完成已经在主线程
                mImageView.setImageBitmap(resource);
            }
        });
```

### 特定View的Target(ViewTarget)
- 由DrawableImageViewTarget和BitmapImageViewTarget我们就可以知道这是为了不同类型的图片资源准备的Target，但是还有一种需求，就是如果我们传入是要加载图片资源的View，但是该View不被Glide支持，目前into方法支持传入ImageView，没关系，ViewTarget可以帮上忙，比如我们需要加载到RelativeLayout
```
/**
 * @author maoqitian
 * @Description: 自定义RelativeLayout
 * @date 2019/2/18 0018 19:51
 */

public class MyView extends RelativeLayout {
    private ViewTarget<MyView, Drawable> viewTarget;
    public MyView(Context context) {
        super(context);
    }

    public MyView(Context context, AttributeSet attrs) {
        super(context, attrs);
        viewTarget =new ViewTarget<MyView, Drawable>(this) {
            @Override
            public void onResourceReady(@NonNull Drawable resource, @Nullable Transition<? super Drawable> transition) {
                setBackground(resource);
            }
        };
    }

    public ViewTarget<MyView, Drawable> getViewTarget() {
        return viewTarget;
    }
}

//使用Glide加载
MyView rl_view = findViewById(R.id.rl_view);
Glide.with(this).load(IMAGE_URL).into(rl_view.getViewTarget());
```
### FutureTarget 
- FutureTarget的一大用处就是可以得到缓存文件

```
new Thread(new Runnable() {
            @Override
            public void run() {
                FutureTarget<File> target = null;
                RequestManager requestManager = Glide.with(MainActivity.this);
                try {
                    target = requestManager
                            .downloadOnly()
                            .load(IMAGE_URL)
                            .submit();
                    final File downloadedFile = target.get();
                    Log.i(TAG,"缓存文件路径"+downloadedFile.getAbsolutePath());
                } catch (ExecutionException | InterruptedException e) {

                } finally {
                    if (target != null) {
                        target.cancel(true); // mayInterruptIfRunning
                    }
                }
            }
        }).start();
```

### preload（预加载）
- 如何使用
```
Glide.with(this).load(IMAGE_URL).preload();
```
- 预加载其实也是属于Target的范围，只是他加载的对象为空而已，也就是没有加载目标
```
/**RequestBuilder类的preload方法**/
 @NonNull
  public Target<TranscodeType> preload() {
    return preload(Target.SIZE_ORIGINAL, Target.SIZE_ORIGINAL);
  }
/**RequestBuilder类的preload方法**/ 
@NonNull
  public Target<TranscodeType> preload(int width, int height) {
    final PreloadTarget<TranscodeType> target = PreloadTarget.obtain(requestManager, width, height);
    return into(target);
  }
/**RequestBuilder类的onResourceReady方法**/   
public final class PreloadTarget<Z> extends SimpleTarget<Z> {

private static final Handler HANDLER = new Handler(Looper.getMainLooper(), new Callback() {
    @Override
    public boolean handleMessage(Message message) {
      if (message.what == MESSAGE_CLEAR) {
        ((PreloadTarget<?>) message.obj).clear();
        return true;
      }
      return false;
    }
  });

//省略部分代码
    ..........
public static <Z> PreloadTarget<Z> obtain(RequestManager requestManager, int width, int height) {
    return new PreloadTarget<>(requestManager, width, height);
  } 

@Override
  public void onResourceReady(@NonNull Z resource, @Nullable Transition<? super Z> transition) {
    HANDLER.obtainMessage(MESSAGE_CLEAR, this).sendToTarget();
  }
  //省略部分代码
    ..........
}  
```
- 通过以上源码，逻辑已经非常清晰，Glide的preload方法里使用的继承SimpleTarget的PreloadTarget对象来作为Target，在它的onResourceReady方法中并没有任何的加载操作，只是调用了Handler来释放资源，到这里也许你会有疑惑，不是说预加载么，怎么不加载。哈哈，其实到onResourceReady方法被调用经过前面的分析Glide已经走完缓存的所有逻辑，那就很容易理解了，预加载只是把图片加载到缓存当中，没有进行其他操作，自然是预加载，并且加载完成之后释放了资源。

## Generated API
- Generated API说白了就是Glide使用注解处理器生成一个API（GlideApp），该API可以代替Glide帮助我们完成图片加载。
- Generated API 目前仅可以在 Application 模块内使用，使用Generated API一方面在Application 模块中可将常用的选项组打包成一个选项在 Generated API 中使用，另一方面可以为Generated API 扩展自定义选项（扩展我们自定义的功能方法）。
- 在上一篇文章中例子中我们可以看到使用Generated API之后使用Glide的方式基本上和Glide3的用法一样流式API使用，先来回顾一下如何使用Generated API

```
//在app下的gradle添加Glide注解处理器的依赖
dependencies {
  annotationProcessor 'com.github.bumptech.glide:compiler:4.8.0'
}
//新建一个类集成AppGlideModule并添加上@GlideModule注解，重新rebuild项目就可以使用GlideApp了
@GlideModule
public final class MyAppGlideModule extends AppGlideModule {}

```
- 经过上的代码的操作，通过Glide注解处理器已经给我们生成了GlideApp类

```
/**GlideApp类部分代码**/ 
public final class GlideApp {
    //省略部分代码
    ..........
    @NonNull
    public static GlideRequests with(@NonNull Context context) {
    return (GlideRequests) Glide.with(context);
  }
  //省略部分代码
    ..........
}
/**GlideApp类部分代码**/ 
public class GlideRequest<TranscodeType> extends RequestBuilder<TranscodeType> implements Cloneable {
//省略部分代码
    ..........
  @NonNull
  @CheckResult
  public GlideRequest<TranscodeType> placeholder(@Nullable Drawable drawable) {
    if (getMutableOptions() instanceof GlideOptions) {
      this.requestOptions = ((GlideOptions) getMutableOptions()).placeholder(drawable);
    } else {
      this.requestOptions = new GlideOptions().apply(this.requestOptions).placeholder(drawable);
    }
    return this;
  }
  //省略部分代码
    ..........
}
/**RequestBuilder类的getMutableOptions方法**/ 
protected RequestOptions getMutableOptions() {
    return defaultRequestOptions == this.requestOptions
        ? this.requestOptions.clone() : this.requestOptions;
  }
```
- 通过以上源码，可以发现，GlideApp对象的with方法返回的是GlideRequests对象，GlideRequests对象继承的是RequestBuilder，这时应该又是豁然开朗的感觉，GlideApp能够适应流式API，其实就是对RequestBuilder包装了一层，GlideRequests对象通过其父类RequestBuilder对象的getMutableOptions方法获取到requestOptions，然后在相应的方法中操作requestOptions以达到可以使用流式API的功能。
### GlideExtension
- GlideExtension字面意思就是Glide扩展，它是一个作用于类上的注解，任何扩展 Glide API 的类都必须使用这个注解来标记，否则其中被注解的方法就会被忽略。
被 @GlideExtension 注解的类应以工具类的思维编写。这种类应该有一个私有的、空的构造方法，应为 final 类型，并且仅包含静态方法。
#### @GlideOption
- GlideOption注解是用来扩展RequestOptions，扩展功能方法第一个参数必须是RequestOptions。下面我们通过设置一个扩展默认设置占位符和错误符方法的例子来说明GlideOption注解。
```
/**
 * @author maoqitian
 * @Description: GlideApp 功能扩展类
 * @date 2019/2/19 0019 12:51
 */
@GlideExtension
public class MyGlideExtension {

    private MyGlideExtension() {
    }

    //可以为方法任意添加参数，但要保证第一个参数为 RequestOptions
    /**
     * 设置通用的加载占位图和错误加载图
     * @param options
     */
    @GlideOption
    public static void normalPlaceholder(RequestOptions options) {
        options.placeholder(R.drawable.ic_cloud_download_black_24dp).error(R.drawable.ic_error_black_24dp);
    }
}
/**GlideOptions类中生成对应的方法**/
/**
   * @see MyGlideExtension#normalPlaceholder(RequestOptions)
   */
  @CheckResult
  @NonNull
  public GlideOptions normalPlaceholder() {
    if (isAutoCloneEnabled()) {
      return clone().normalPlaceholder();
    }
    MyGlideExtension.normalPlaceholder(this);
    return this;
  }
/**GlideRequest类中生成对应的方法**/  
/**
   * @see GlideOptions#normalPlaceholder()
   */
  @CheckResult
  @NonNull
  public GlideRequest<TranscodeType> normalPlaceholder() {
    if (getMutableOptions() instanceof GlideOptions) {
      this.requestOptions = ((GlideOptions) getMutableOptions()).normalPlaceholder();
    } else {
      this.requestOptions = new GlideOptions().apply(this.requestOptions).normalPlaceholder();
    }
    return this;
  }
```
- 如上代码所示，我们可以通过@GlideExtension注解设置自己功能扩展类，使用@GlideOption注解标注对赢扩展功能静态方法，重构项目后Glide注解处理器则会自动在GlideOptions对象和GlideRequest对象中生成相应的方法能够被我们调用
```
//调用我们刚刚设置的扩展功能方法
GlideApp.with(this).load(IMAGE_URL)
                .normalPlaceholder()
                .into(mImageView);
```
#### GlideType
- GlideType注解是用于扩展RequestManager的，同理扩展的方法第一个参数必须是RequestManager，并设置类型为加载资源类型，该注解主要作用就是扩展Glide支持加载资源的类型，以下举出官方文档支持gif的一个例子，还是在我们刚刚扩展功能类中。
```
@GlideExtension
public class MyGlideExtension {

    private static final RequestOptions DECODE_TYPE_GIF = decodeTypeOf(GifDrawable.class).lock();

    @GlideType(GifDrawable.class)
    public static void asMyGif(RequestBuilder<GifDrawable> requestBuilder) {
        requestBuilder
                .transition(new DrawableTransitionOptions())
                .apply(DECODE_TYPE_GIF);
    }
}

/**GlideRequests类中生成的asMyGif方法**/

/**
   * @see MyGlideExtension#asMyGif(RequestBuilder)
   */
  @NonNull
  @CheckResult
  public GlideRequest<GifDrawable> asMyGif() {
    GlideRequest<GifDrawable> requestBuilder = this.as(GifDrawable.class);
    MyGlideExtension.asMyGif(requestBuilder);
    return requestBuilder;
  }
```
- 同理在我们加载Gif资源的时候可以直接使用
```
 GlideApp.with(this).asMyGif().load(IMAGE_URL)
                .into(mImageView);
```
## 源码阅读方法思路
- 看了这么多源码，其实我想说说框架源码阅读的方法思路：
  - 1.首先自己能把框架大体的流程走一遍，然后根据自己刚刚的思路把文章写出来，在写文章的同时也能发现自己刚刚的思路是否有问题，慢慢纠正
  - 2.文章写完，把整体流程图画出来，画图的过程一个是复习思路，还可以让自己对源码逻辑更加清晰
  - 3.阅读框架源码时看到英文注释可以先理解其含义，在你看源码没头绪的时候往往思路就在注释中，如果对源码中一个类很迷惑，可以直接看该类的头部注释往往注明了该类的作用。
-  
## 最后说点
- 到此，真的很想大叫一声宣泄一下，Glide源码就像一座山，一座高峰，你必须沉住气，慢慢的解读，要不然稍不留神就会掉入代码的海洋，迷失方向。回头看看，你不得不感叹正式由于Glide源码中成千上万行的代码，才造就了这样一个强大的框架。最后，也非常感谢您阅读我的文章，文章中如果有错误，请大家给我提出来，大家一起学习进步，如果觉得我的文章给予你帮助，也请给我一个喜欢和关注，同时也欢迎访问我的[**个人博客**](https://www.maoqitian.com)。

- 参考链接

  - [Gldie文档](http://bumptech.github.io/glide/)
