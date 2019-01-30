---
title: 从源码角度深入理解OKHttp3
date: 2019-01-27 22:17:21
categories:  
- Android热门框架解析 #分类
tags:
- HTTP
- OKHttp3
- Android
- 源码分析
- Dispatcher
- Interceptor
---
<img src="https://github.com/maoqitian/MaoMdPhoto/raw/master/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3OKhttp/OKHttp-logo.png" width=100% />
>在日常开发中网络请求是很常见的功能。OkHttp作为Android开发中最常用的网络请求框架，在Android开发中我们经常结合retrofit一起使用，俗话说得好：“知其然知其所以然”，所以这篇文章我们通过源码来深入理解OKHttp3（基于3.12版本）
<!--more-->
### 常规使用
- 在了解源码前，我们先了解如何使用OKHttp这个框架（[框架地址](https://github.com/square/okhttp)）
```
  //框架引入项目
  implementation("com.squareup.okhttp3:okhttp:3.12.0")
  
  //引用官方Demo的例子
  @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //主线程不能进行耗时操作
         new Thread(){
            @Override
            public void run() {
                super.run();
                /**
                 * 同步请求
                 */
                GetExample getexample = new GetExample();
                String syncresponse = null;
                try {
                    syncresponse = getexample.run("https://raw.github.com/square/okhttp/master/README.md");
                    Log.i("maoqitian","异步请求返回参数"+syncresponse);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }.start();
        /**
         * 异步请求
         */
        PostExample postexample = new PostExample();
        String json = postexample.bowlingJson("Jesse", "Jake");
        try {
            postexample.post("http://www.roundsapp.com/post", json);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
     /**
      * 异步请求
      */
     class PostExample {
         final MediaType JSON = MediaType.get("application/json; charset=utf-8");
         //获取 OkHttpClient 对象
         OkHttpClient client = new OkHttpClient();

         void post(String url, String json) throws IOException {
             RequestBody body = RequestBody.create(JSON, json);
             Request request = new Request.Builder()
                     .url(url)
                     .post(body)
                     .build();
             client.newCall(request).enqueue(new Callback() {
                @Override
                public void onFailure(Call call, IOException e) {
                  Log.i("maoqitian","异步请求返回参数"+e.toString());
                }

                @Override
                public void onResponse(Call call, Response response) throws IOException {
                    String asynresponse= response.body().string();
                    Log.i("maoqitian","异步请求返回参数"+asynresponse);
                }
            });
         }

         String bowlingJson(String player1, String player2) {
             return "{'winCondition':'HIGH_SCORE',"
                     + "'name':'Bowling',"
                     + "'round':4,"
                     + "'lastSaved':1367702411696,"
                     + "'dateStarted':1367702378785,"
                     + "'players':["
                     + "{'name':'" + player1 + "','history':[10,8,6,7,8],'color':-13388315,'total':39},"
                     + "{'name':'" + player2 + "','history':[6,10,5,10,10],'color':-48060,'total':41}"
                     + "]}";
         }
     }
    /**
     * 同步请求
     */
    class GetExample {
        OkHttpClient client = new OkHttpClient();
        String run(String url) throws IOException {
            Request request = new Request.Builder()
                    .url(url)
                    .build();
            try (Response response = client.newCall(request).execute()) {
                return response.body().string();
            }
        }
    }
```
- 由例子我们可以看到，client.newCall(request).execute()执行的是异步请求，我们可以加入Callback来异步获取返回值，Response response = client.newCall(request).execute()执行的是同步请求，更多post请求方式例子可以查看官方[sample项目](https://github.com/square/okhttp/tree/master/samples/guide/src/main/java/okhttp3/recipes)
### 源码分析
#### OKHttp网络请求流程图  
- 首先看一个流程图，对于接下来的源码分析有个大体印象
![OKHttp网络请求流程图](https://github.com/maoqitian/MaoMdPhoto/raw/master/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3OKhttp/OKHttp%E7%BD%91%E7%BB%9C%E8%AF%B7%E6%B1%82%E6%B5%81%E7%A8%8B%E5%9B%BE.png)
#### RealCall 
- 通过上面的例子可以看到，不管是同步请求还是异步请求，首先调用的OkHttpClient的newCall(request)方法，先来看看这个方法
```
     /**
      * Prepares the {@code request} to be executed at some point in the future.
      */
      @Override public Call newCall(Request request) {
      return RealCall.newRealCall(this, request, false /* for web socket */);
     }
```
- 通过newCall方法的源码可以看到该方法返回值是Call，Call是一个接口，他的实现类是RealCall，所以我们执行的同步execute()方法或者异步enqueue()方法都是RealCall的方法。newCall方法接收了的网络请求参数，接下来我们看看execute()和enqueue()方法
```
    /**
     * 同步请求
     */
     @Override public Response execute() throws IOException {
      synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
      }
      captureCallStackTrace();
      timeout.enter();
      eventListener.callStart(this);
      try {
        client.dispatcher().executed(this);
        Response result = getResponseWithInterceptorChain();
        if (result == null) throw new IOException("Canceled");
        return result;
        } catch (IOException e) {
        e = timeoutExit(e);
        eventListener.callFailed(this, e);
        throw e;
        } finally {
        client.dispatcher().finished(this);
        }
       }
     /**
      * 异步请求
      */
     @Override public void enqueue(Callback responseCallback) {
      synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
      }
      captureCallStackTrace();
      eventListener.callStart(this);
      client.dispatcher().enqueue(new AsyncCall(responseCallback));
     }
```
- 这里先看异步的enqueue方法，很直观可以看到真正执行网络请求的是最后一句代码，而它是怎么做的呢，我们还得先弄明白dispatcher，**Dispatcher的本质是异步请求的调度器**，它内部持有一个线程池，结合线程池调配并发请求。官方文档描述也说了这一点。
    
![OKhttp Dispatcher文档描述](https://github.com/maoqitian/MaoMdPhoto/raw/master/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3OKhttp/OKhttp%20Dispatcher%E6%96%87%E6%A1%A3%E6%8F%8F%E8%BF%B0.png)
```
     /**最大并发请求数*/
     private int maxRequests = 64;
     /**每个主机最大请求数*/
     private int maxRequestsPerHost = 5;
     
     /** Ready async calls in the order they'll be run. 准备要执行的异步请求队列*/
     private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();

     /** Running asynchronous calls. Includes canceled calls that haven't finished yet.
     正在执行的异步请求队列*/
     private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();

     /** Running synchronous calls. Includes canceled calls that haven't finished yet. 
     正在执行的同步请求队列*/
     private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();
     
     /** Dispatcher 构造方法 */
     public Dispatcher(ExecutorService executorService) {
     this.executorService = executorService;
      }

      public Dispatcher() {
      }

     public synchronized ExecutorService  executorService() {
      if (executorService == null) {
      executorService = new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60, TimeUnit.SECONDS,
          new SynchronousQueue<Runnable>(), Util.threadFactory("OkHttp Dispatcher", false));
      }
      return executorService;
     }
```
#### Dispatcher
- 通过Dispatcher的构造方法我们知道我们可以使用自己的线程池，也可以使用Dispatcher默认的线程池，默认的线程池相当于CachedThreadPool线程池，这个线程池比较适合执行大量的耗时较少的任务（[线程池介绍](https://www.jianshu.com/p/c1ed876c17fe)）。
- 了解了Dispatcher之后，我们继续探究Dispatcher的enqueue方法
```
    void enqueue(AsyncCall call) {
      synchronized (this) {
      readyAsyncCalls.add(call);
     }
     promoteAndExecute();
     }
     /**
      * Promotes eligible calls from {@link #readyAsyncCalls} to {@link #runningAsyncCalls} and runs
      * them on the executor service. Must not be called with synchronization because executing calls
      * can call into user code.
      *
      * @return true if the dispatcher is currently running calls.
      */
      private boolean promoteAndExecute() {
      assert (!Thread.holdsLock(this));

      List<AsyncCall> executableCalls = new ArrayList<>();
      boolean isRunning;
      synchronized (this) {
      for (Iterator<AsyncCall> i = readyAsyncCalls.iterator(); i.hasNext(); ) {
        AsyncCall asyncCall = i.next();

        if (runningAsyncCalls.size() >= maxRequests) break; // Max capacity.
        if (runningCallsForHost(asyncCall) >= maxRequestsPerHost) continue; // Host max capacity.

        i.remove();
        executableCalls.add(asyncCall);
        runningAsyncCalls.add(asyncCall);
        }
        isRunning = runningCallsCount() > 0;
        }

        for (int i = 0, size = executableCalls.size(); i < size; i++) {
        AsyncCall asyncCall = executableCalls.get(i);
        asyncCall.executeOn(executorService());
        }
        return isRunning;
     }
```
- 这里分三步走，**首先**将传入的AsyncCall（这其实是一个Runnable对象）加入准备要执行的异步请求队列，**其次**调用promoteAndExecute()方法，变量准备要执行的异步请求队列，如果队列任务数超过最大并发请求数，则直接退出遍历，则不会进行下面的操作；如果超过每个主机最大请求数，则跳过这次循环，继续下一次遍历，否则将异步任务加入到正在执行的异步请求队列，**最后**遍历保存异步任务的队列，执行AsyncCall.executeOn(executorService())方法，并且传入了Dispatcher的默认线程池。
```
    /**
     * Attempt to enqueue this async call on {@code executorService}. This will attempt to clean up
     * if the executor has been shut down by reporting the call as failed.
     */
      void executeOn(ExecutorService executorService) {
      assert (!Thread.holdsLock(client.dispatcher()));
      boolean success = false;
      try {
        executorService.execute(this);
        success = true;
      } catch (RejectedExecutionException e) {
        InterruptedIOException ioException = new InterruptedIOException("executor rejected");
        ioException.initCause(e);
        eventListener.callFailed(RealCall.this, ioException);
        responseCallback.onFailure(RealCall.this, ioException);
      } finally {
        if (!success) {
          client.dispatcher().finished(this); // This call is no longer running!
        }
      }
    }
```
- 通过执行AsyncCall.executeOn()方法的源码，我们看到Dispatcher的线程池执行了execute(this)方法，**执行异步任务**，并且指向的是this，也就是当前的**AsyncCall**对象（RealCall的内部类），而AsyncCall实现了抽象类**NamedRunnable**
```
    /**
     * Runnable implementation which always sets its thread name.
     */
    public abstract class NamedRunnable implements Runnable {
    protected final String name;

    public NamedRunnable(String format, Object... args) {
    this.name = Util.format(format, args);
    }

     @Override public final void run() {
      String oldName = Thread.currentThread().getName();
      Thread.currentThread().setName(name);
     try {
      execute();
     } finally {
      Thread.currentThread().setName(oldName);
       }
     }
     protected abstract void execute();
     }
```
- 可以看到NamedRunnable中run()方法调用了抽象方法execute()，也就说明execute()的实现在AsyncCall对象中，而上面线程池执行的异步任务也是调用这个execute()方法，我们看看AsyncCall对象中execute()方法的实现
```
     @Override protected void execute() {
      boolean signalledCallback = false;
      timeout.enter();
      try {
        Response response = getResponseWithInterceptorChain();
        if (retryAndFollowUpInterceptor.isCanceled()) {
          signalledCallback = true;
          responseCallback.onFailure(RealCall.this, new IOException("Canceled"));
        } else {
          signalledCallback = true;
          responseCallback.onResponse(RealCall.this, response);
        }
       } catch (IOException e) {
        e = timeoutExit(e);
        if (signalledCallback) {
          // Do not signal the callback twice!
          Platform.get().log(INFO, "Callback failure for " + toLoggableString(), e);
        } else {
          eventListener.callFailed(RealCall.this, e);
          responseCallback.onFailure(RealCall.this, e);
        }
       } finally {
        client.dispatcher().finished(this);
       }
       }
       
        /**Dispatcher的finished方法*/
        /** Used by {@code AsyncCall#run} to signal completion. */
        void finished(AsyncCall call) {
        finished(runningAsyncCalls, call);
        }

        /** Used by {@code Call#execute} to signal completion. */
         void finished(RealCall call) {
         finished(runningSyncCalls, call);
        }

        private <T> void finished(Deque<T> calls, T call) {
          Runnable idleCallback;
         synchronized (this) {
         if (!calls.remove(call)) throw new AssertionError("Call wasn't in-flight!");
         idleCallback = this.idleCallback;
        }

        boolean isRunning = promoteAndExecute();

        if (!isRunning && idleCallback != null) {
         idleCallback.run();
        }
      }
```
- 我们可以先关注最后一行，不管前面请求如何，最后finally代码块中都执行了Dispatcher的finished方法，要是要将当前任务从runningAsyncCalls或runningSyncCalls 中移除， 同时把readyAsyncCalls的任务调度到runningAsyncCalls中并执行而finished方法中执行了promoteAndExecute()方法，经过前面对该方法分析，说明不管当前执行的任务如何，都会OkHttp都会去readyAsyncCalls（准备要执行的异步请求队列）取出下一个请求继续执行。接下来我们继续回到AsyncCall对象中的execute()方法，可以发现getResponseWithInterceptorChain()的方法返回了Response，说明在该方法中执行了我们的网络请求。而不管同步还是异步请求，都是通过getResponseWithInterceptorChain()完成网络请求。
```
       Response getResponseWithInterceptorChain() throws IOException {
       // Build a full stack of interceptors.
       List<Interceptor> interceptors = new ArrayList<>();
       interceptors.addAll(client.interceptors());
       interceptors.add(retryAndFollowUpInterceptor);
       interceptors.add(new BridgeInterceptor(client.cookieJar()));
       interceptors.add(new CacheInterceptor(client.internalCache()));
       interceptors.add(new ConnectInterceptor(client));
       if (!forWebSocket) {
        interceptors.addAll(client.networkInterceptors());
        }
       interceptors.add(new CallServerInterceptor(forWebSocket));

        Interceptor.Chain chain = new RealInterceptorChain(interceptors, null, null, null, 0,
        originalRequest, this, eventListener, client.connectTimeoutMillis(),
        client.readTimeoutMillis(), client.writeTimeoutMillis());

        return chain.proceed(originalRequest);
       }
```
#### 拦截器（Interceptor）
- 由getResponseWithInterceptorChain()方法我们看到添加了很多Interceptor（拦截器），首先要了解每个Interceptor的作用，也能大致了解OKHttp完成网络请求的过程。
- 1.首先加入我们自定义的interceptors
- 2.通过RetryAndFollowUpInterceptor处理网络请求重试
- 3.通过BridgeInterceptor处理请求对象转换，应用层到网络层
- 4.通过CacheInterceptor处理缓存
- 5.通过ConnectInterceptor处理网络请求链接
- 6.通过CallServerInterceptor处理读写，和服务器通信，进行真正的网络请求
#### 责任链模式
      
- 在阅读接下来源码之前，我们先要了解责任链模式。通俗化的讲**在责任链模式中有很多对象，这些对象可以理解为上面列出的拦截器，而每个对象之间都通过一条链子连接，网络请求在这条链子上传递，直到某一个对象处理了这个网络请求，也就是完成了网络请求**。使用这个模式的好处就是不管你用多少拦截器处理什么操作，最终都不会影响我们的发出请求的目的，就是完成网络请求，拦截过程你可以任意添加分配责任。
    
- 接着继续看Interceptor.Chain，他是Interceptor的内部接口，前面添加的每一个拦截器都实现了Interceptor接口，而RealInterceptorChain是Interceptor.Chain接口的实现类。先看RealInterceptorChain的proceed方法源码
```
      public Response proceed(Request request, StreamAllocation streamAllocation, HttpCodec httpCodec,
      RealConnection connection) throws IOException {
      if (index >= interceptors.size()) throw new AssertionError();

       calls++;
       ......
       // Call the next interceptor in the chain.
        RealInterceptorChain next = new RealInterceptorChain(interceptors, streamAllocation, httpCodec,
        connection, index + 1, request, call, eventListener, connectTimeout, readTimeout,
        writeTimeout);
        Interceptor interceptor = interceptors.get(index);
        Response response = interceptor.intercept(next);

        // Confirm that the next interceptor made its required call to chain.proceed().
        if (httpCodec != null && index + 1 < interceptors.size() && next.calls != 1) {
        throw new IllegalStateException("network interceptor " + interceptor
          + " must call proceed() exactly once");
        }
        .....
        return response;
       }
```
- 通过源码可以注意到interceptor.intercept(next)，RetryAndFollowUpInterceptor作为默认拦截器的第一个拦截器，也就是执行了它的intercept方法
#### RetryAndFollowUpInterceptor
- 前面说过RetryAndFollowUpInterceptor拦截器执行OKHttp网络重试，先看它的intercept方法
```
       /**RetryAndFollowUpInterceptor的intercept方法 **/
       @Override public Response intercept(Chain chain) throws IOException {
    Request request = chain.request();
    RealInterceptorChain realChain = (RealInterceptorChain) chain;
    Call call = realChain.call();
    EventListener eventListener = realChain.eventListener();

    StreamAllocation streamAllocation = new StreamAllocation(client.connectionPool(),
        createAddress(request.url()), call, eventListener, callStackTrace);
    this.streamAllocation = streamAllocation;

    int followUpCount = 0;
    Response priorResponse = null;
    while (true) {
      if (canceled) {
        streamAllocation.release();
        throw new IOException("Canceled");
      }
       //将请求通过链子chain传递到下一个拦截器
      Response response;
      boolean releaseConnection = true;
      try {
        response = realChain.proceed(request, streamAllocation, null, null);
        releaseConnection = false;
      } catch (RouteException e) {
        // 线路异常，连接失败，检查是否可以重新连接
        if (!recover(e.getLastConnectException(), streamAllocation, false, request)) {
          throw e.getFirstConnectException();
        }
        releaseConnection = false;
        continue;
      } catch (IOException e) {
        // An attempt to communicate with a server failed. The request may have been sent.
        // IO异常，连接失败，检查是否可以重新连接
        boolean requestSendStarted = !(e instanceof ConnectionShutdownException);
        if (!recover(e, streamAllocation, requestSendStarted, request)) throw e;
        releaseConnection = false;
        continue;
      } finally {
        // We're throwing an unchecked exception. Release any resources. 释放资源
        if (releaseConnection) {
          streamAllocation.streamFailed(null);
          streamAllocation.release();
        }
      }

      // Attach the prior response if it exists. Such responses never have a body.
      if (priorResponse != null) {
        response = response.newBuilder()
            .priorResponse(priorResponse.newBuilder()
                    .body(null)
                    .build())
            .build();
      }

      Request followUp;
      try {
        //效验状态码、身份验证头信息、跟踪重定向或处理客户端请求超时
        followUp = followUpRequest(response, streamAllocation.route());
      } catch (IOException e) {
        streamAllocation.release();
        throw e;
      }

      if (followUp == null) {
        streamAllocation.release();
         // 不需要重定向，正常返回结果
        return response;
      }

      closeQuietly(response.body());
      //超过重试次数
      if (++followUpCount > MAX_FOLLOW_UPS) {
        streamAllocation.release();
        throw new ProtocolException("Too many follow-up requests: " + followUpCount);
      }

      if (followUp.body() instanceof UnrepeatableRequestBody) {
        streamAllocation.release();
        throw new HttpRetryException("Cannot retry streamed HTTP body", response.code());
      }
      if (!sameConnection(response, followUp.url())) {
        streamAllocation.release();
        streamAllocation = new StreamAllocation(client.connectionPool(),
            createAddress(followUp.url()), call, eventListener, callStackTrace);
        this.streamAllocation = streamAllocation;
      } else if (streamAllocation.codec() != null) {
        throw new IllegalStateException("Closing the body of " + response
            + " didn't close its backing stream. Bad interceptor?");
      }
      request = followUp;
      priorResponse = response;
      }
     }
```
- 通过RetryAndFollowUpInterceptor拦截器intercept方法源码，能够理解到OKHttp的重试机制
- 1.首先创建StreamAllocation对象（稍后分析），在一个死循环中将通过链子chain传递到下一个拦截器，如果捕获异常，则判断异常是否恢复连接，不能连接则抛出异常，跳出循环并是否创建的连接池资源
- 2.第一步没有异常，还要返回值效验状态码、头部信息、是否需要重定向、连接超时等信息，捕获异常则抛出并退出循环
- 3.如果如果重定向，循环超出RetryAndFollowUpInterceptor拦截器的最大重试次数，也抛出异常，退出循环
![RetryAndFollowUpInterceptor拦截器重试机制流程图](https://github.com/maoqitian/MaoMdPhoto/raw/master/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3OKhttp/RetryAndFollowUpInterceptor%E6%8B%A6%E6%88%AA%E5%99%A8%E9%87%8D%E8%AF%95%E6%9C%BA%E5%88%B6%E6%B5%81%E7%A8%8B%E5%9B%BE.png)  
- 通过拦截器RetryAndFollowUpInterceptor调用(RealInterceptorChain) chain.proceed()方法，又再次回到了我们刚刚分析proceed()方法，而该方法继续调用下一个拦截器的intercept()方法，这个拦截器就是默认的第二个拦截器BridgeInterceptor
```
     /**
      * Bridges from application code to network code. First it builds a network request from a user
      * request. Then it proceeds to call the network. Finally it builds a user response from the network
      * response.
      * BridgeInterceptor的intercept方法
      */
     @Override public Response intercept(Chain chain) throws IOException {
      Request userRequest = chain.request();
      Request.Builder requestBuilder = userRequest.newBuilder();

      RequestBody body = userRequest.body();
      ......
      Response networkResponse = chain.proceed(requestBuilder.build());
      ......
     }  
```
- 该拦截器主要实现了网络请求中应用层到网络层之间的数据编码桥梁。根据用户请求建立网络连接，根据网络响应建立网络响应，也可以看到该方法 继续调用了chain.proceed()方法，同理，根据前面分析会调用第三个默认拦截器CacheInterceptor的intercept方法。
#### CacheInterceptor
- 前面我们说过这个拦截器是处理缓存的，接下来看看源码是如何实现的
```
     /**
      * 拦截器CacheInterceptor的intercept方法
      */
     @Override public Response intercept(Chain chain) throws IOException {
     Response cacheCandidate = cache != null
        ? cache.get(chain.request())
        : null;

     long now = System.currentTimeMillis();
     //获取策略，假设当前可以使用网络
     CacheStrategy strategy = new CacheStrategy.Factory(now, chain.request(), cacheCandidate).get();
     Request networkRequest = strategy.networkRequest;
     Response cacheResponse = strategy.cacheResponse;

     if (cache != null) {
      cache.trackResponse(strategy);
     }

     if (cacheCandidate != null && cacheResponse == null) {
      closeQuietly(cacheCandidate.body()); // The cache candidate wasn't applicable. Close it.
     }

      // If we're forbidden from using the network and the cache is insufficient, fail. 如果网络被禁止使用并且没有缓存，则请求失败
      if (networkRequest == null && cacheResponse == null) {
          return new Response.Builder()
          .request(chain.request())
          .protocol(Protocol.HTTP_1_1)
          .code(504)
          .message("Unsatisfiable Request (only-if-cached)")
          .body(Util.EMPTY_RESPONSE)
          .sentRequestAtMillis(-1L)
          .receivedResponseAtMillis(System.currentTimeMillis())
          .build();
     }

     // If we don't need the network, we're done.如果有缓存，则返回响应缓存，请求完成
     if (networkRequest == null) {
      return cacheResponse.newBuilder()
          .cacheResponse(stripBody(cacheResponse))
          .build();
     }

     Response networkResponse = null;
     try {
     //没有缓存，则进行网络请求，执行下一个拦截器
      networkResponse = chain.proceed(networkRequest);
      } finally {
      // If we're crashing on I/O or otherwise, don't leak the cache body.
      if (networkResponse == null && cacheCandidate != null) {
        closeQuietly(cacheCandidate.body());
      }
     }

      // If we have a cache response too, then we're doing a conditional get.
      if (cacheResponse != null) {
       //状态码 304 
       if (networkResponse.code() == HTTP_NOT_MODIFIED) {
        Response response = cacheResponse.newBuilder()
            .headers(combine(cacheResponse.headers(), networkResponse.headers()))
            .sentRequestAtMillis(networkResponse.sentRequestAtMillis())
            .receivedResponseAtMillis(networkResponse.receivedResponseAtMillis())
            .cacheResponse(stripBody(cacheResponse))
            .networkResponse(stripBody(networkResponse))
            .build();
        networkResponse.body().close();

        // Update the cache after combining headers but before stripping the
        // Content-Encoding header (as performed by initContentStream()).
        cache.trackConditionalCacheHit();
        cache.update(cacheResponse, response);
        return response;
       } else {
        closeQuietly(cacheResponse.body());
       }
      }

      Response response = networkResponse.newBuilder()
        .cacheResponse(stripBody(cacheResponse))
        .networkResponse(stripBody(networkResponse))
        .build();

      if (cache != null) {
      if (HttpHeaders.hasBody(response) && CacheStrategy.isCacheable(response, networkRequest)) {
        // Offer this request to the cache.
        //保存缓存
        CacheRequest cacheRequest = cache.put(response);
        return cacheWritingResponse(cacheRequest, response);
      }

       if (HttpMethod.invalidatesCache(networkRequest.method())) {
        try {
          cache.remove(networkRequest);
        } catch (IOException ignored) {
          // The cache cannot be written.
        }
       }
      }

      return response;
     }
```
- 先看看intercept方法的大致逻辑
- 1.首先通过CacheStrategy.Factory().get()获取缓存策略
- 2.如果网络被禁止使用并且没有缓存，则请求失败，返回504
- 3.如果有响应缓存，则返回响应缓存，请求完成
- 4.没有缓存，则进行网络请求，执行下一个拦截器
- 5.进行网络请求，如果响应状态码为304，说明客户端缓存了目标资源但不确定该缓存资源是否是最新版本，服务端数据没变化，继续使用缓存
- 6.最后保存缓存
- 缓存的场景也符合设计模式中的**策略模式**，需要CacheStrategy提供策略在不同场景下读缓存还是请求网络。
- 了解了缓存逻辑，继续深入了解OKHttp的缓存是如何做的。首先我们应该回到最初的缓存拦截器设置代码
```
      /**RealCall 设置缓存拦截器*/
      interceptors.add(new CacheInterceptor(client.internalCache()));
      
      /**OkHttpClient 设置缓存*/
      Cache cache;
      @Override public void setCache(OkHttpClient.Builder builder, InternalCache internalCache) {
        builder.setInternalCache(internalCache);
      }
      void setInternalCache(@Nullable InternalCache internalCache) {
      this.internalCache = internalCache;
      this.cache = null;
      }
      InternalCache internalCache() {
      return cache != null ? cache.internalCache : internalCache;
      }
      
      /**Cache类中 内部持有 InternalCache */
       final DiskLruCache cache;
       final InternalCache internalCache = new InternalCache() {
       @Override public Response get(Request request) throws IOException {
       return Cache.this.get(request);
       }

       @Override public CacheRequest put(Response response) throws IOException {
       return Cache.this.put(response);
       }

       @Override public void remove(Request request) throws IOException {
       Cache.this.remove(request);
       }

       @Override public void update(Response cached, Response network) {
       Cache.this.update(cached, network);
       }

       @Override public void trackConditionalCacheHit() {
       Cache.this.trackConditionalCacheHit();
       }

       @Override public void trackResponse(CacheStrategy cacheStrategy) {
       Cache.this.trackResponse(cacheStrategy);
       }
      };
```
- 上面我们分别截取了 RealCall类、OkHttpClient类和Cache类的源码，可以了解到拦截器使用的缓存类是DiskLruCache，设置缓存缓存只能通过OkHttpClient的builder来设置，缓存操作实现是在Cache类中，但是Cache没有实现InternalCache接口，而是持有InternalCache接口的内部类对象来实现缓存的操作方法，这样就使得缓存的操作实现只在Cache内部，外部用户是无法实现缓存操作的，方便框架内部使用，接口扩展也不影响外部。
![Cache和InternalCache类之间关系](https://github.com/maoqitian/MaoMdPhoto/raw/master/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3OKhttp/Cache%E5%92%8CInternalCache%E7%B1%BB%E4%B9%8B%E9%97%B4%E5%85%B3%E7%B3%BB.png)
#### ConnectInterceptor
- 根据前面的分析，缓存拦截器中也会调用chain.proceed方法，所以这时候执行到了第四个默认拦截器ConnectInterceptor，接着看它的intercept方法
```
    /**
      * 拦截器ConnectInterceptor的intercept方法
      */
    @Override public Response intercept(Chain chain) throws IOException {
    RealInterceptorChain realChain = (RealInterceptorChain) chain;
    Request request = realChain.request();
    StreamAllocation streamAllocation = realChain.streamAllocation();

    // We need the network to satisfy this request. Possibly for validating a conditional GET.
    boolean doExtensiveHealthChecks = !request.method().equals("GET");
    //打开连接
    HttpCodec httpCodec = streamAllocation.newStream(client, chain, doExtensiveHealthChecks);
    RealConnection connection = streamAllocation.connection();
    //交由下一个拦截器处理
    return realChain.proceed(request, streamAllocation, httpCodec, connection);
    }
```
- 我们看到intercept源码非常简单，通过StreamAllocation打开连接，然后就交由下一个拦截器处理请求。如何连接呢？我们需要搞懂StreamAllocation。
- StreamAllocation对象负责协调请求和连接池之间的联系。每一个OKHttpClient有它对应的一个连接池，经过前面的分析我们知道StreamAllocation对象的创建在RetryAndFollowUpInterceptor拦截器的intercept方法中创建，而StreamAllocation打开了连接，则连接池在哪创建呢，答案就在OKHttpClient的Builder类构造方法中
```
    public Builder() {
      .......
      connectionPool = new ConnectionPool();
      .......
    }
```
- 了解了StreamAllocation对象和ConnectionPool对象的创建，下面来分析StreamAllocation是如何打开连接的。首先是streamAllocation.newStream()方法
```
    public HttpCodec newStream(
      OkHttpClient client, Interceptor.Chain chain, boolean doExtensiveHealthChecks) {
      ........
      try {
      RealConnection resultConnection = findHealthyConnection(connectTimeout, readTimeout,
          writeTimeout, pingIntervalMillis, connectionRetryEnabled, doExtensiveHealthChecks);
      .......
        }
      } catch (IOException e) {
      throw new RouteException(e);
      }
     }
    
    /**
   * Finds a connection and returns it if it is healthy. If it is unhealthy the process is repeated
   * until a healthy connection is found.
   */
      private RealConnection findHealthyConnection(int connectTimeout, int readTimeout,
      int writeTimeout, int pingIntervalMillis, boolean connectionRetryEnabled,
      boolean doExtensiveHealthChecks) throws IOException {
      while (true) {
      RealConnection candidate = findConnection(connectTimeout, readTimeout, writeTimeout,
          pingIntervalMillis, connectionRetryEnabled);
      ........      
      return candidate;
      }
    } 
    /**
   * Returns a connection to host a new stream. This prefers the existing connection if it exists,
   * then the pool, finally building a new connection.
   */
  private RealConnection findConnection(int connectTimeout, int readTimeout, int writeTimeout,
      int pingIntervalMillis, boolean connectionRetryEnabled) throws IOException {
      ............
      //
      if (result == null) {
        // Attempt to get a connection from the pool.
        Internal.instance.get(connectionPool, address, this, null);
        if (connection != null) {
          //连接复用
          foundPooledConnection = true;
          result = connection;
        } else {
          selectedRoute = route;
        }
      }
     
     ..........

      if (!foundPooledConnection) {
        ........
        result = new RealConnection(connectionPool, selectedRoute);
        //记录每个连接的引用，每个调用必须与同一连接上的调用配对。
        acquire(result, false);
       }
      }
      .........
      synchronized (connectionPool) {
       .......
      // Pool the connection. 将连接放入连接池
      Internal.instance.put(connectionPool, result);
      ......
      }
     }
    .......
    return result;
    }
```
- 根据上面的源码，我们可以知道findHealthyConnection在循环找健康的连接，直到找到连接，说明findConnection方法是寻找连接的核心方法，该方法中存在可以复用的连接则复用，否则创建新的连接，并且记录连接引用，我们可以明白**StreamAllocation主要是为拦截器提供一个连接， 如果连接池中有复用的连接则复用连接， 如果没有则创建新的连接**。
![StreamAllocation创建和复用流程](https://github.com/maoqitian/MaoMdPhoto/raw/master/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3OKhttp/StreamAllocation%E5%88%9B%E5%BB%BA%E5%92%8C%E5%A4%8D%E7%94%A8%E6%B5%81%E7%A8%8B.png)
#### ConnectionPool连接池实现
- 明白StreamAllocation是如何创建和复用连接池，我们还要明白连接池（ConnectionPool）的是如何实现的。
- 理解ConnectionPool之前，我们需要明白[TCP连接的知识](https://www.jianshu.com/p/72de08b802e6)，Tcp建立连接三次握手和断开连接四次握手过程是需要消耗时间的，在http/1.0每一次请求只能打开一次连接，而在http/1.1是支持**持续连接（persistent connection）**，使得一次连接打开之后会保持一段时间，如果还是同一个请求并且使同一个服务器则在这段时间内继续请求连接是可以复用的。而ConnectionPool也实现了这个机制，在它内部持有一个线程池和一个缓存连接的双向列表，连接中最多只能存在5个空闲连接，空闲连接最多只能存活5分钟，空闲连接到期之后定时清理。
```
     public final class ConnectionPool {
     /**
      * Background threads are used to cleanup expired connections. There will be at most a single
      * thread running per connection pool. The thread pool executor permits the pool itself to be
      * garbage collected.
      */
      //线程池
      private static final Executor executor = new ThreadPoolExecutor(0 /* corePoolSize */,
      Integer.MAX_VALUE /* maximumPoolSize */, 60L /* keepAliveTime */, TimeUnit.SECONDS,
      new SynchronousQueue<Runnable>(), Util.threadFactory("OkHttp ConnectionPool", true));

      /** The maximum number of idle connections for each address. */
      private final int maxIdleConnections;
      private final long  keepAliveDurationNs;
      private final Runnable cleanupRunnable = new Runnable() {
      @Override public void run() {
      // 后台定期清理连接的线程
      while (true) {
        long waitNanos = cleanup(System.nanoTime());
        if (waitNanos == -1) return;
        if (waitNanos > 0) {
          long waitMillis = waitNanos / 1000000L;
          waitNanos -= (waitMillis * 1000000L);
          synchronized (ConnectionPool.this) {
            try {
              ConnectionPool.this.wait(waitMillis, (int) waitNanos);
            } catch (InterruptedException ignored) {
            }
            }
          }
        }
       }
      };
      //缓存连接的双向队列
      private final Deque<RealConnection> connections = new ArrayDeque<>();
      ............
       /**
       * Create a new connection pool with tuning parameters appropriate for a single-user  application.
       * The tuning parameters in this pool are subject to change in future OkHttp releases. Currently
       * this pool holds up to 5 idle connections which will be evicted after 5 minutes of inactivity.
       */
       public ConnectionPool() {
       this(5, 5, TimeUnit.MINUTES);
      }
      ............
     }
```
![ConnectionPool连接池缓存清理流程](https://github.com/maoqitian/MaoMdPhoto/raw/master/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3OKhttp/ConnectionPool%E8%BF%9E%E6%8E%A5%E6%B1%A0%E7%BC%93%E5%AD%98%E6%B8%85%E7%90%86%E6%B5%81%E7%A8%8B.png)
- 这里还要说的一点是streamAllocation.newStream()返回的HttpCodec对象就是我们编码HTTP请求并解码HTTP响应的接口，他的实现类Http2Codec和Http1Codec对应https和http的解析request与响应response对socket读写过程实现，并最终放到RealConnection对象newCodec类中创建。 
```
    /** 
    RealConnection类newCodec方法
    */
     public HttpCodec newCodec(OkHttpClient client, Interceptor.Chain chain,
      StreamAllocation streamAllocation) throws SocketException {
     if (http2Connection != null) {
      return new Http2Codec(client, chain, streamAllocation, http2Connection);
     } else {
      socket.setSoTimeout(chain.readTimeoutMillis());
      source.timeout().timeout(chain.readTimeoutMillis(), MILLISECONDS);
      sink.timeout().timeout(chain.writeTimeoutMillis(), MILLISECONDS);
      return new Http1Codec(client, streamAllocation, source, sink);
      }
    }
```
- streamAllocation得到连接对象，也就是RealConnection对象，它封装了套接字socket连接，也就是该类的connectSocket方法。并且使用OKio来对数据读写。OKio封装了Java的I/O操作，这里就不细说了。最后返回的ConnectInterceptor拦截器的intercept方法同样调用了Chain.proceed，将拿到的连接交由CallServerInterceptor做处理。
```
    /** Does all the work necessary to build a full HTTP or HTTPS connection on a raw socket. 
    RealConnection类connectSocket方法
    */
    private void connectSocket(int connectTimeout, int readTimeout, Call call,
      EventListener eventListener) throws IOException {
    Proxy proxy = route.proxy();
    Address address = route.address();

    rawSocket = proxy.type() == Proxy.Type.DIRECT || proxy.type() == Proxy.Type.HTTP
        ? address.socketFactory().createSocket()
        : new Socket(proxy);

    eventListener.connectStart(call, route.socketAddress(), proxy);
    rawSocket.setSoTimeout(readTimeout);
    try {
      //打开 socket 连接
      Platform.get().connectSocket(rawSocket, route.socketAddress(), connectTimeout);
    } catch (ConnectException e) {
      ConnectException ce = new ConnectException("Failed to connect to " + route.socketAddress());
      ce.initCause(e);
      throw ce;
    }

    // The following try/catch block is a pseudo hacky way to get around a crash on Android 7.0
    // More details:
    // https://github.com/square/okhttp/issues/3245
    // https://android-review.googlesource.com/#/c/271775/
    try {
      //使用OKio来对数据读写
      source = Okio.buffer(Okio.source(rawSocket));
      sink = Okio.buffer(Okio.sink(rawSocket));
     } catch (NullPointerException npe) {
      if (NPE_THROW_WITH_NULL.equals(npe.getMessage())) {
        throw new IOException(npe);
      }
     }
    }
```
- 最后返回的ConnectInterceptor拦截器的intercept方法同样调用了Chain.proceed，将拿到的连接交由CallServerInterceptor做处理。
#### CallServerInterceptor
- 在经过前面一系列拦截器之后，OKHttp最终把拿到网络请求连接给到CallServerInterceptor拦截器进行网络请求和服务器通信。
```
    /**CallServerInterceptor的intercept方法*/ 
    @Override public Response intercept(Chain chain) throws IOException {
    RealInterceptorChain realChain = (RealInterceptorChain) chain;
    HttpCodec httpCodec = realChain.httpStream();
    StreamAllocation streamAllocation = realChain.streamAllocation();
    RealConnection connection = (RealConnection) realChain.connection();
    Request request = realChain.request();
    long sentRequestMillis = System.currentTimeMillis();
    realChain.eventListener().requestHeadersStart(realChain.call());
    //按照HTTP协议，依次写入请求体
    httpCodec.writeRequestHeaders(request);
    .................
     if (responseBuilder == null) {
      realChain.eventListener().responseHeadersStart(realChain.call());
      //
      responseBuilder = httpCodec.readResponseHeaders(false);
     }

     Response response = responseBuilder
        .request(request)
        .handshake(streamAllocation.connection().handshake())
        .sentRequestAtMillis(sentRequestMillis)
        .receivedResponseAtMillis(System.currentTimeMillis())
        .build();
      ...............
      if (forWebSocket && code == 101) {
      // Connection is upgrading, but we need to ensure interceptors see a non-null response body.
      response = response.newBuilder()
          .body(Util.EMPTY_RESPONSE)
          .build();
      } else {
      //响应数据OKio写入
      response = response.newBuilder()
          .body(httpCodec.openResponseBody(response))
          .build();
       }
      }
      return response;
    }
    /**Http1Codec方法**/
    //OKio 读写对象
    final BufferedSource source;
    final BufferedSink sink;
    @Override public void writeRequestHeaders(Request request) throws IOException {
    //构造好请求头
    String requestLine = RequestLine.get(
        request, streamAllocation.connection().route().proxy().type());
     writeRequest(request.headers(), requestLine);
    }
     /** Returns bytes of a request header for sending on an HTTP transport.
     将请求信息写入sink
     */
    public void writeRequest(Headers headers, String requestLine) throws IOException {
    if (state != STATE_IDLE) throw new IllegalStateException("state: " + state);
    sink.writeUtf8(requestLine).writeUtf8("\r\n");
    for (int i = 0, size = headers.size(); i < size; i++) {
      sink.writeUtf8(headers.name(i))
          .writeUtf8(": ")
          .writeUtf8(headers.value(i))
          .writeUtf8("\r\n");
     }
    sink.writeUtf8("\r\n");
    state = STATE_OPEN_REQUEST_BODY;
    }
```
- 可以看到在CallServerInterceptor拦截器的方法中首先通过HttpCodec（上面贴的是Http1Codec的方法）writeRequestHeaders和writeRequest方法写入请求体，并将请求体写入OKio的写入对象sink中
```
    /**Http1Codec方法**/
    /**
     读取响应头信息
    */ 
    @Override public Response.Builder readResponseHeaders(boolean expectContinue) throws IOException {
    if (state != STATE_OPEN_REQUEST_BODY && state != STATE_READ_RESPONSE_HEADERS) {
      throw new IllegalStateException("state: " + state);
    }

     try {
      StatusLine statusLine = StatusLine.parse(readHeaderLine());

      Response.Builder responseBuilder = new Response.Builder()
          .protocol(statusLine.protocol)
          .code(statusLine.code)
          .message(statusLine.message)
          .headers(readHeaders());

      if (expectContinue && statusLine.code == HTTP_CONTINUE) {
        return null;
      } else if (statusLine.code == HTTP_CONTINUE) {
        state = STATE_READ_RESPONSE_HEADERS;
        return responseBuilder;
      }

      state = STATE_OPEN_RESPONSE_BODY;
      return responseBuilder;
    } catch (EOFException e) {
      // Provide more context if the server ends the stream before sending a response.
      IOException exception = new IOException("unexpected end of stream on " + streamAllocation);
      exception.initCause(e);
      throw exception;
      }
    }
     /**
     写入响应输入到ResponseBody
     */
     @Override public ResponseBody openResponseBody(Response response) throws IOException {
     streamAllocation.eventListener.responseBodyStart(streamAllocation.call);
     String contentType = response.header("Content-Type");

     if (!HttpHeaders.hasBody(response)) {
      Source source = newFixedLengthSource(0);
      return new RealResponseBody(contentType, 0, Okio.buffer(source));
     }

     if ("chunked".equalsIgnoreCase(response.header("Transfer-Encoding"))) {
      Source source = newChunkedSource(response.request().url());
      return new RealResponseBody(contentType, -1L, Okio.buffer(source));
     }

     long contentLength = HttpHeaders.contentLength(response);
     if (contentLength != -1) {
      Source source = newFixedLengthSource(contentLength);
      return new RealResponseBody(contentType, contentLength, Okio.buffer(source));
     }

      return new RealResponseBody(contentType, -1L, Okio.buffer(newUnknownLengthSource()));
    }
```
- 通过readResponseHeaders方法读取响应头信息，openResponseBody得到响应体信息。最终将网络请求的响应信息通过Callback()回调方法异步传递出去，同步请求则直接返回。到此OKHttp源码理解到此为止。
### 最后说点
- 通过OKHttp这个框架源码阅读，也是对自己的一个提升，不仅了解了框架原理，设计模式在适宜场景的运用，同时也是对自己耐心的一次考验，源码的阅读是枯燥的，但是只要静下心来，也能发现阅读源码的乐趣。由于本人水平有限，文章中如果有错误，请大家给我提出来，大家一起学习进步，如果觉得我的文章给予你帮助，也请给我一个喜欢和关注。

- 参考链接：

  - [深入解析OkHttp3](https://blog.csdn.net/u012124438/article/details/54236967)
  - [OkHttp3源码和设计模式-1](https://www.imooc.com/article/24025?block_id=tuijian_wz)
- 参考书籍：
  - 《计算机网络》第六版

    


    

    
    



      
    