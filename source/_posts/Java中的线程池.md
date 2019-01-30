---
title: Java中的线程池和作用
date: 2019-01-20 22:03:59
categories:  
- Java基础回顾 #分类
tags:
- Java
- Thread
- ThreadPool
---
>在Java开发中，多线程执行任务是很常见的，Java也提供了线程类Thread来让我们方便创建一个线程如下代码所示
<!--more-->
### Thread开启线程
```
    new Thread(){
    @Override
    public void run() {
        .....
        }
    }.start();

```
- 这样创建新的线程有几个缺点
  
   - 每次要开启新的线程都需要创建一个，性能差
   - 线程随意创建，缺乏统一的管理
   - 不能做到线程的中断

- 处理上面的这些问题，我们就需要使用**线程池**来管理线程。
 
### 线程池描述
> Java SE5d的java.util.concurrent包 提供了Executor(执行器)来管理线程对象。Executor是一个接口，而ExecutorService继承了Excutor接口，ExecutorService是一个具有生命周期的Executor，它知道如何构建恰当的上下文来执行Runnable对象。而ExecutorService对象是使用Executors的静态方法得到Java中的线程池

### Executor接口实现
```
public interface Executor {

    /**
     * Executes the given command at some time in the future.  The command
     * may execute in a new thread, in a pooled thread, or in the calling
     * thread, at the discretion of the {@code Executor} implementation.
     *
     * @param command the runnable task
     * @throws RejectedExecutionException if this task cannot be
     * accepted for execution
     * @throws NullPointerException if command is null
     */
    void execute(Runnable command);
}

```
### Java 的四种线程池
- Java中为我们提供了四种线程池，他们分别是FixedThreadPool、CachedThreadPool、ScheduledThreadPool、SingleThreadExector

#### FixedThreadPool
- 创建方式为使用Executors的newFixedThreadPool()方法来创建，这种线程池的线程数量是固定的，可以看到他的静态方法需要传入线程的数量，在空闲状态下不会被系统回收，除非它被关闭了。当它的所有线程都在执行任务的时候，新加入的线程就会出来等待状态，等到有线程空闲，新任务才会被执行,如果新任务加入时线程池中有空闲的线程，则意味着它可以快速响应处理任务。
```
        public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
        }
    
    
        /*
         * 获取使用方法
         */
       Runnable r=new Runnable(){
        @Override
        run(){
          .....  
          }
       }
    
       ExecutorService executor = Executors.newFixedThreadPool(2);
       executor.execute(r)

```
#### CachedThreadPool
- 创建方式为使用Executors的newCachedThreadPool()方法来创建，由其静态方法可以看出，他的线程数是Integer.MAX_VALUE，可以说他的线程数是无限大，也就是说只要有任务，线程就会立即执行，但是它的每一个线程在空闲状态下是有超时机制的，这个时间为60秒，只要线程空闲时间超过60秒该线程就会被回收，如果所有的线程都处于由空闲状态并且超过了60秒，则相当于线程池中没有任何，线程，也就是说这时的线程池是不占用任何资源的，所以这个线程池比较适合执行大量的耗时较少的任务
       
```
        public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
         }
     
         /*
         * 获取使用方法
         */
         
         Runnable r=new Runnable(){
         @Override
           run(){
          .....  
          }
         }
         
         ExecutorService executor = Executors.newCachedThreadPool();
         executor.execute(r)
```
#### ScheduledThreadPool
- 创建方式为使用Executors的newCachedThreadPool()方法来创建，这种线程池的核心线程数是固定的，而非核心线程数据是没有限制的，并且当非核心线程空闲的的时候该线程就会被立即回收，所以我们可以使用他来操作定时任务和重复的任务（和Task TimeTask 有些像）
```
        public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
        }
        
        public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE,
              DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
              new DelayedWorkQueue());
        }
        
        /*
         * 获取使用方法
         */
        Runnable r=new Runnable(){
          @Override
           run(){
             .....  
            }
         }
        
         ScheduledExecutorService executor = Executors.newScheduledThreadPool(2);
         // 1000ms 后执行任务
         executor.schedule(r,1000,TimeUnit.MICROSECONDS)
        
         // 延迟1000ms 每隔1000ms 重复执行 任务
         executor.scheduleAtFixedRate(r,1000,1000,TimeUnit.MICROSECONDS)
```
#### SingleThreadExector
- 创建方式为使用Executors的newCachedThreadPool()方法来创建，这种线程只有唯一一个核心线程，并且保证所有执行的的任务都在这一个线程中执行，并且是顺序执行，也就不用在考虑线程同步的问题了。
```
        public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
         }
         
          /*
           * 获取使用方法
           */
         ExecutorService executor = Executors.newSingleThreadExecutor();
         executor.execute(r)
         
         
         Runnable r=new Runnable(){
          @Override
           run(){
          .....  
          }
         }
        
         ExecutorService executor = Executors.newSingleThreadExecutor();
         executor.execute(r)
```
### 线程池核心构造方法
- 通过上面对Java四种线程池的介绍,我们可以发现最终都是新建ThreadPoolExecutor对象，也就是说ThreadPoolExecutor才是线程池的核心实现。
- ThreadPoolExecutor 比较常用的一个构造方法
```
       public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             threadFactory, defaultHandler);
       }
    
      //参数含义
      corePoolSize：默认情况下核心线程数会一直存在，不管是否处于闲置状态，
      但是如果线程池设置了核心线程数，也就是ThreadPoolExecutor的allowCoreThreadTimeOut这个boolean
      为true，如果核心线程数为零，则allowCoreThreadTimeOut的值为true，超时时间为keepAliveTime的值,
      也就是CachedThreadPool线程池的所有线程都能够回收的原因，他的核心线程数为零,也就是没有核心线程
      
      maximumPoolSize：最大线程数
      
      keepAliveTime：非核心线程超时时长，如果allowCoreThreadTimeOut的值为true，
      则该超时时长也会作用于空闲状态的核心线程
      
      unit：超时时长的时间单位 TimeUnit.MILLISECONDS/SECONDS/MINUTES(毫秒/秒/分钟)
      
      workQueue：线程任务队列，存放线程任务
      
      threadFactory：线程池线程生产工厂，为线程池创建新线程
     
      //我们看到构造放中还有一个defaultHandler参数，他其实是RejectedExecutionHandler对象
      defaultHandler：当线程队列满了，或者线程任务无法执行则用该参数抛出通知RejectedExecutionException，这里构造方法暂时没用到
```
- 在Android中我们可以写自己的线程管理类接，下面就来实现一个自己的线程池管理类来管理我们的线程
```
      /**
       * Created by 毛麒添 on 2018/8/1 0010.
       * 线程管理类，线程池为单例
       */

      public class ThreadManager {

      private static ThreadPool mThreadPool;

      public static ThreadPool getmThreadPool(){
        if (mThreadPool==null){
            synchronized (ThreadManager.class){
                if(mThreadPool==null){
                    //线程安全
                    mThreadPool=new ThreadPool(5,10,1L);
                }
            }
          }
         return mThreadPool;
      }


      //线程池
      public static class ThreadPool{

        private int corePoolSize;//核心线程数 5
        private int maximumPoolSize;//最大线程数 10
        private long keepAliveTime;//线程休眠时间 1秒

        private ThreadPoolExecutor executor;

        private ThreadPool(  int corePoolSize, int maximumPoolSize,long keepAliveTime){
              this.corePoolSize=corePoolSize;
              this.maximumPoolSize=maximumPoolSize;
              this.keepAliveTime=keepAliveTime;
        }


        public void execute(Runnable runnable){
            /**
             * int corePoolSize, 核心线程数
             * int maximumPoolSize, 最大线程数
             * long keepAliveTime, 线程休眠时间
             * TimeUnit unit, 时间单位
             * BlockingQueue<Runnable> workQueue, 线程队列
             * ThreadFactory threadFactory, 生成线程的工厂
             */
            if(executor==null){
                executor = new ThreadPoolExecutor(corePoolSize,maximumPoolSize,keepAliveTime,
                        TimeUnit.SECONDS,new LinkedBlockingQueue<Runnable>(),
                        Executors.defaultThreadFactory(),new ThreadPoolExecutor.AbortPolicy());
            }
            //核心线程也有超时机制
            executor.allowCoreThreadTimeOut(true);
            executor.execute(runnable);
        }
        //取消任务，从任务队列中将其移除
        public void cancelTask(Runnable runnable){
            if(runnable!=null){
                executor.getQueue().remove(runnable);
             }

           }
        }

      }
      
      
      //使用
      ThreadManager.getmThreadPool().execute(new Runnable() {
                @Override
                public void run() {
                    //执行任务
                }
            });
```
### 最后说点
- 以上这就是我所了解的线程池知识，如果有错，请给我留言指出，大家一起学习进步。

- 参考资料:
          
  - 《Android开发艺术探索》
  - 《Java编程思想》(第四版)
   