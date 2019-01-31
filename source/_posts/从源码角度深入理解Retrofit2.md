---
title: 从源码角度深入理解Retrofit2
date: 2019-01-31 19:52:37
categories:
- Android热门框架解析 #分类
tags:
- HTTP
- Retrofit2.0
- OKHttp3
- 源码分析
- 代理模式
- 配器模式
- 装饰模式
- 构造者模式
---
![retrofit_logo](https://github.com/maoqitian/MaoMdPhoto/raw/master/%E4%BB%8E%E6%BA%90%E7%A0%81%E8%A7%92%E5%BA%A6%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Retrofit2/retrofit_logo.png)
> Retrofit2作为目前最火的网络请求框架之一，它是一个由Square
组织开发的可以在Android和java中使用的安全型HTTP客户端（官方文档描述“Type-safe HTTP client for Android and Java by Square”）。本文将从Retrofit2简单使用入手，在对其源码进行分析来深入理解[Retrofit2（基于2.5.0版本）](https://github.com/square/retrofit)。
<!--more-->
## Retrofit2简单使用
- 1.1 下面，根据官方例子，简单使用一个get请求来演示Retrofit2简单使用。首先gradle中添加retrofit依赖，创建一个描述每一个请求的接口
```
     /**
      * gradle中添加依赖 
      */
     implementation 'com.squareup.retrofit2:retrofit:2.5.0'
    
    public interface GitHub {
    
    public static final String API_URL = "https://api.github.com";
    //使用Get 请求
    @GET("/repos/{owner}/{repo}/contributors")
    Call<List<SimpleService.Contributor>> contributors(
            @Path("owner") String owner,
            @Path("repo") String repo);
    }
```
- 1.2 创建网络请求数据bean对象
    
    ```
    public class SimpleService {
    public static class Contributor {
        public final String login;
        public final int contributions;

        public Contributor(String login, int contributions) {
            this.login = login;
            this.contributions = contributions;
        }
      }
    }
    ```
- 1.3 创建retrofit对象，传入网络请求的域名地址，传入刚刚创建的请求对象接口，而我们的网络请求默认返回JSON数据，而retrofit请求默认返回response.body()（异步请求），所以我们需要添加一个GsonConverterFactory转换器将JSON转化为我们的bean对象，需要在gradle中添加如下库的依赖
```
     /**
      * gradle中添加依赖 
      */
     implementation 'com.squareup.retrofit2:converter-gson:2.5.0'
     
      // Create a very simple REST adapter which points the GitHub API.
        Retrofit retrofit=new Retrofit.Builder()
                .baseUrl(GitHub.API_URL)
                .addConverterFactory(GsonConverterFactory.create())
                .build();
        // Create an instance of our GitHub API interface.
        GitHub gitHub = retrofit.create(GitHub.class);
        // Create a call instance for looking up Retrofit contributors.
        final retrofit2.Call<List<SimpleService.Contributor>> call = gitHub.contributors("square", "retrofit");
```
- 1.4 根据上一步拿到的call对象执行同步网络请求获取数据，耗时操作开启子线程执行  
```
       new Thread(){
            @Override
            public void run() {
                super.run();
                /**
                 * 同步请求
                try {
                    List<SimpleService.Contributor> contributors = call.execute().body();
                    for (SimpleService.Contributor contributor : contributors) {
                     Log.e("maoqitian","Retrofit同步请求返回参数"+contributor.login + " (" + contributor.contributions + ")");
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }.start();

```
- 1.5 根据上一步拿到的call对象执行异步网络请求获取数据 
      
```
      call.enqueue(new retrofit2.Callback<List<SimpleService.Contributor>>() {
            @Override
            public void onResponse(retrofit2.Call<List<SimpleService.Contributor>> call, retrofit2.Response<List<SimpleService.Contributor>> response) {
                List<SimpleService.Contributor> body = response.body();
                for (SimpleService.Contributor contributor : body) {
                    Log.e("maoqitian","Retrofit异步请求返回参数"+contributor.login + " (" + contributor.contributions + ")");
                }
            }

            @Override
            public void onFailure(retrofit2.Call<List<SimpleService.Contributor>> call, Throwable t) {
                Log.e("maoqitian","Retrofit异步请求失败"+t.getMessage());
            }
        });
```
## 源码分析
### retrofit2网络请求基本流程图
- 根据上面的简单使用retrofit的例子，我们可以概括一下大致流程图
![retrofit2网络请求基本示意流程图](https://github.com/maoqitian/MaoMdPhoto/raw/master/%E4%BB%8E%E6%BA%90%E7%A0%81%E8%A7%92%E5%BA%A6%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Retrofit2/retrofit2%E7%BD%91%E7%BB%9C%E8%AF%B7%E6%B1%82%E5%9F%BA%E6%9C%AC%E7%A4%BA%E6%84%8F%E6%B5%81%E7%A8%8B%E5%9B%BE.png)

### retrofit对象创建
- 我们先了解retrofit对象包含了哪些成员变量以及他们的含义和作用
```
    //ServiceMethod 的缓存，从接口中解析出来，放在这个 map 里面。
    private final Map<Method, ServiceMethod<?>> serviceMethodCache = new ConcurrentHashMap<>();
    //okhttp3.Call.Factory接口对象，接口声明了newCall方法
    final okhttp3.Call.Factory callFactory;
    //OKHttp3 的HttpUrl 对象，描述了一个 http 地址 
    final HttpUrl baseUrl;
    //保存ConverterFactory转换器的list，通过Retrofit.Builder的 addConverterFactory方法来添加
    final List<Converter.Factory> converterFactories;
    //保存CallAdapterFactory适配器的list，通过Retrofit.Builder的 addCallAdapterFactory方法来添加
    final List<CallAdapter.Factory> callAdapterFactories;
    //回调函数的执行器，也就是回调函数执行的线程，Android 中默认为 MainThreadExecutor
    final @Nullable Executor callbackExecutor;
    //创建动态代理对象之前，是否提前解析接口 Method，创建 ServiceMethod 并添加到 Cache 中。
    final boolean validateEagerly;
```
- 接着就是创建Retrofit对象new Retrofit.Builder()，一看到Builder我们就可以想到**构造者模式**，通过外部对各个参数的配置来尽可能的达到各种业务请求场景的要求。先看看Builder()中的操作
```
    /** Retrofit的 Builder方法*/
    public Builder() {
      this(Platform.get());
    }
    /**Platform类中的get方法*/
    static Platform get() {
    return PLATFORM;
    }
    
    private static final Platform PLATFORM = findPlatform();
    /**Platform类中的findPlatform方法*/
    private static Platform findPlatform() {
    try {
      Class.forName("android.os.Build");
      if (Build.VERSION.SDK_INT != 0) {
        return new Android();
      }
    } catch (ClassNotFoundException ignored) {
    }
    try {
      Class.forName("java.util.Optional");
      return new Java8();
    } catch (ClassNotFoundException ignored) {
    }
    return new Platform();
    }
    /**Platform类中的Android方法*/
    static class Android extends Platform {
    @IgnoreJRERequirement // Guarded by API check.
    @Override boolean isDefaultMethod(Method method) {
      if (Build.VERSION.SDK_INT < 24) {
        return false;
      }
      return method.isDefault();
    }

    @Override public Executor defaultCallbackExecutor() {
      return new MainThreadExecutor();
    }

    @Override List<? extends CallAdapter.Factory> defaultCallAdapterFactories(
        @Nullable Executor callbackExecutor) {
      if (callbackExecutor == null) throw new AssertionError();
      ExecutorCallAdapterFactory executorFactory = new ExecutorCallAdapterFactory(callbackExecutor);
      return Build.VERSION.SDK_INT >= 24
        ? asList(CompletableFutureCallAdapterFactory.INSTANCE, executorFactory)
        : singletonList(executorFactory);
    }

    @Override int defaultCallAdapterFactoriesSize() {
      return Build.VERSION.SDK_INT >= 24 ? 2 : 1;
    }

    @Override List<? extends Converter.Factory> defaultConverterFactories() {
      return Build.VERSION.SDK_INT >= 24
          ? singletonList(OptionalConverterFactory.INSTANCE)
          : Collections.<Converter.Factory>emptyList();
    }

    @Override int defaultConverterFactoriesSize() {
      return Build.VERSION.SDK_INT >= 24 ? 1 : 0;
    }

    static class MainThreadExecutor implements Executor {
      private final Handler handler = new Handler(Looper.getMainLooper());

      @Override public void execute(Runnable r) {
        handler.post(r);
      }
     }
    }
```
- 由上述源码可以看到，Retrofit的builder方法中调用了Platform.get()方法，最终调用的是findPlatform()，该方法使用反射判断当前的环境来得到不同的Platform对象，接着回到Retrofit的build方法
```
     /** Retrofit的 build方法*/
     public Retrofit build() {
      if (baseUrl == null) {
        throw new IllegalStateException("Base URL required.");
      }

      okhttp3.Call.Factory callFactory = this.callFactory;
      if (callFactory == null) {
        callFactory = new OkHttpClient();
      }

      Executor callbackExecutor = this.callbackExecutor;
      if (callbackExecutor == null) {
        callbackExecutor = platform.defaultCallbackExecutor();
      }

      // Make a defensive copy of the adapters and add the default Call adapter.
      List<CallAdapter.Factory> callAdapterFactories = new ArrayList<>(this.callAdapterFactories);
      callAdapterFactories.addAll(platform.defaultCallAdapterFactories(callbackExecutor));

      // Make a defensive copy of the converters.
      List<Converter.Factory> converterFactories = new ArrayList<>(
          1 + this.converterFactories.size() + platform.defaultConverterFactoriesSize());

      // Add the built-in converter factory first. This prevents overriding its behavior but also
      // ensures correct behavior when using converters that consume all types.
      converterFactories.add(new BuiltInConverters());
      converterFactories.addAll(this.converterFactories);
      converterFactories.addAll(platform.defaultConverterFactories());

      return new Retrofit(callFactory, baseUrl, unmodifiableList(converterFactories),
          unmodifiableList(callAdapterFactories), callbackExecutor, validateEagerly);
      }
     }
```
- 可以看到build方法中callFactory就是OkHttpClient对象，也就是Retrofit的网络请求也是通过OKHttp来完成的；在Android环境中，build中platform调用的方法都是前面提到的Platform的继承类Android中实现的方法，callbackExecutor在主线程执行，默认加载的CallAdapter.Factory为ExecutorCallAdapterFactory，如果Build.VERSION.SDK_INT >= 24(Android 7.0)，则Converter.Factory默认为OptionalConverterFactory，否则为空。最终新建retrofit对象并将设置和默认的参数传入。
### retrofit.create(class)
- 创建了retrofit对象之后，接着调用了retrofit的create方法，先看看该方法的实现
```
    /**Retrofit类的create方法*/
    public <T> T create(final Class<T> service) {
    Utils.validateServiceInterface(service);
    if (validateEagerly) {
      eagerlyValidateMethods(service);
    }
     return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
          private final Platform platform = Platform.get();
          private final Object[] emptyArgs = new Object[0];

          @Override public Object invoke(Object proxy, Method method, @Nullable Object[] args)
              throws Throwable {
            // If the method is a method from Object then defer to normal invocation.
            if (method.getDeclaringClass() == Object.class) {
              return method.invoke(this, args);
            }
            if (platform.isDefaultMethod(method)) {
              return platform.invokeDefaultMethod(method, service, proxy, args);
            }
            return loadServiceMethod(method).invoke(args != null ? args : emptyArgs);
          }
        });
     }
```
- 看到retrofit的create方法，代码很少，看到关键代码Proxy，我们就可以知道这里使用了Java的**动态代理**，为了方便下面代码的解读，我们先来简单了解什么是动态代理。
### 代理模式
- **代理是一种常用的设计模式，其目的就是为其他对象提供一个代理以控制对某个对象的访问。代理类负责为委托类预处理消息，过滤消息并转发消息，以及进行消息被委托类执行后的后续处理。而代理分为动态代理和静态代理，静态代理中每一个需要被代理的类都要创建一个代理类，这显然很麻烦，所以Java给我们提供了java.lang.reflect.InvocationHandler接口和 java.lang.reflect.Proxy 的类来支持动态代理** 
```
    //Object proxy:被代理的对象  
    //Method method:要调用的方法  
    //Object[] args:方法调用时所需要参数  
    public interface InvocationHandler {  
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;  
    } 
    
    //CLassLoader loader:类的加载器  
    //Class<?> interfaces:得到全部的接口  
    //InvocationHandler h:得到InvocationHandler接口的子类的实例  
    public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) throws IllegalArgumentException  
```
- [更详细动态代理知识可以自行了解](http://a.codekk.com/detail/Android/Caij/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8B%20Java%20%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86) 
- 现在我们回过头看retrofit的create方法，Proxy.newProxyInstance方法加载了我们的网络请求描述接口类和其中定义的方法，并实现了InvocationHandler接口，也就是说create方法实现了动态代理，则调用create方法，也就会调用InvocationHandler接口实现的invoke方法。
- 在InvocationHandler接口实现的invoke方法中，如果是 Object对象本身方法（比如equals、toString、hashCode等）以及 Platform 默认的方法（java8默认方法，1.8的新特性），则在正常使用的时候调用返回，如果不满足，最后调用 loadServiceMethod(method)方法，也就是该方法来解析对应接口中我们定义的描述网络请求的方法。
### 解析描述网络请求接口
- 上一小节我们说到loadServiceMethod(method)方法解析对应接口
```
     /**Retrofit类的loadServiceMethod方法 */
     ServiceMethod<?> loadServiceMethod(Method method) {
      ServiceMethod<?> result = serviceMethodCache.get(method);
      if (result != null) return result;

      synchronized (serviceMethodCache) {
      result = serviceMethodCache.get(method);
      if (result == null) {
        result = ServiceMethod.parseAnnotations(this, method);
        serviceMethodCache.put(method, result);
       }
      }
     return result;
     }
```
- 通过该方法，解析接口之后返回的对象是ServiceMethod，如果之前解析过，则直接从serviceMethodCache取出直接返回，否则调用ServiceMethod.parseAnnotations方法进行解析
```
     /**ServiceMethod<T> 抽象类的parseAnnotations方法**/
     static <T> ServiceMethod<T> parseAnnotations(Retrofit retrofit, Method method) {
     RequestFactory requestFactory = RequestFactory.parseAnnotations(retrofit, method);

      Type returnType = method.getGenericReturnType();
      if (Utils.hasUnresolvableType(returnType)) {
      throw methodError(method,
          "Method return type must not include a type variable or wildcard: %s", returnType);
      }
      if (returnType == void.class) {
      throw methodError(method, "Service methods cannot return void.");
     }

      return HttpServiceMethod.parseAnnotations(retrofit, method, requestFactory);
     }
```
- 如上源码，RequestFactory负责解析接口并且生成Request，继续看RequestFactory的 parseAnnotations方法 
```
     /**RequestFactory 类的parseAnnotations方法**/
     static RequestFactory parseAnnotations(Retrofit retrofit, Method method) {
     return new Builder(retrofit, method).build();
     }
     /**RequestFactory 类的build方法**/
     RequestFactory build() {
      for (Annotation annotation : methodAnnotations) {
      //解析网络请求方法注解
        parseMethodAnnotation(annotation);
      }
      //省略部分代码
      ......
      int parameterCount = parameterAnnotationsArray.length;
      parameterHandlers = new ParameterHandler<?>[parameterCount];
      for (int p = 0; p < parameterCount; p++) {
        //方法参数注解的解析
        parameterHandlers[p] = parseParameter(p, parameterTypes[p], parameterAnnotationsArray[p]);
      }
      //省略部分代码
      .......
      return new RequestFactory(this);
    }
    /**RequestFactory 类的parseMethodAnnotation方法**/
    private void parseMethodAnnotation(Annotation annotation) {
      if (annotation instanceof DELETE) {
        parseHttpMethodAndPath("DELETE", ((DELETE) annotation).value(), false);
      } else if (annotation instanceof GET) {
        parseHttpMethodAndPath("GET", ((GET) annotation).value(), false);
        //剩余其他请求方法注解解析
        ......
        isFormEncoded = true;
      }
    }
    /**RequestFactory 类的parseHttpMethodAndPath方法**/
    private void parseHttpMethodAndPath(String httpMethod, String value, boolean hasBody) {
      if (this.httpMethod != null) {
        throw methodError(method, "Only one HTTP method is allowed. Found: %s and %s.",
            this.httpMethod, httpMethod);
      }
      this.httpMethod = httpMethod;
      this.hasBody = hasBody;

      if (value.isEmpty()) {
        return;
      }
      // Get the relative URL path and existing query string, if present.
      int question = value.indexOf('?');
      if (question != -1 && question < value.length() - 1) {
        // Ensure the query string does not have any named parameters.
        String queryParams = value.substring(question + 1);
        Matcher queryParamMatcher = PARAM_URL_REGEX.matcher(queryParams);
        if (queryParamMatcher.find()) {
          throw methodError(method, "URL query string \"%s\" must not have replace block. "
              + "For dynamic query parameters use @Query.", queryParams);
        }
      }
      this.relativeUrl = value;
      this.relativeUrlParamNames = parsePathParameters(value);
    }
    /**RequestFactory 类的parsePathParameters方法**/
    static Set<String> parsePathParameters(String path) {
      Matcher m = PARAM_URL_REGEX.matcher(path);
      Set<String> patterns = new LinkedHashSet<>();
      while (m.find()) {
        patterns.add(m.group(1));
      }
      return patterns;
    }
```
- 在解析网络请求注解的方法parseMethodAnnotation中，通过instanceof判断描述网络请求接口的注解中是否包含对应网络请求类型，如果包含则调用parseHttpMethodAndPath设置httpMethod网络请求方法（get），联系到上面的简单例子，相对url(对应/repos/{owner}/{repo}/contributors)，hasBody在get请求中没有，在post请求中为post的参数体，relativeUrlParamNames（对应owner、repo）为相对路径中需要外部传递参数占位符名称，通过正则表达式获取。
- 上面我们的分析中只是解析了网络请求注解，还需要解析接口方法参数的注解，看到RequestFactory的parseAnnotations方法中的parseParameter方法
```
     /**RequestFactory 类的parseParameter方法**/
     private ParameterHandler<?> parseParameter(
        int p, Type parameterType, @Nullable Annotation[] annotations) {
      ParameterHandler<?> result = null;
      if (annotations != null) {
        for (Annotation annotation : annotations) {
          ParameterHandler<?> annotationAction =
              parseParameterAnnotation(p, parameterType, annotations, annotation);
              //省略部分代码
              ....
          }    
       }
      return result;
     }
      /**RequestFactory 类的parseParameterAnnotation方法**/
     @Nullable
     private ParameterHandler<?> parseParameterAnnotation(
        int p, Type type, Annotation[] annotations, Annotation annotation) {
        //省略部分代码
      .......     
      else if (annotation instanceof Path) {
        validateResolvableType(p, type);
        //各种检查判断
        .....
        gotPath = true;

        Path path = (Path) annotation;
        String name = path.value();
        validatePathName(p, name);

        Converter<?, String> converter = retrofit.stringConverter(type, annotations);
        return new ParameterHandler.Path<>(name, converter, path.encoded());
      }
      //省略代码
      ......
      }
      /**ParameterHandler 类的Path实现**/
      static final class Path<T> extends ParameterHandler<T> {
      //省略代码
      ......
      @Override void apply(RequestBuilder builder, @Nullable T value) throws IOException {
      if (value == null) {
        throw new IllegalArgumentException(
            "Path parameter \"" + name + "\" value must not be null.");
      }
      builder.addPathParam(name, valueConverter.convert(value), encoded);
      }
     }
     /**RequestBuilder 类的addPathParam方法**/
     void addPathParam(String name, String value, boolean encoded) {
      if (relativeUrl == null) {
      // The relative URL is cleared when the first query parameter is set.
      throw new AssertionError();
      }
     String replacement = canonicalizeForPath(value, encoded);
     String newRelativeUrl = relativeUrl.replace("{" + name + "}", replacement);
     if (PATH_TRAVERSAL.matcher(newRelativeUrl).matches()) {
      throw new IllegalArgumentException(
          "@Path parameters shouldn't perform path traversal ('.' or '..'): " + value);
      }
      relativeUrl = newRelativeUrl;
     }
```
- 由上源码（源码较多，截取Path注解处理原理），RequestFactory 类的parseParameter方法通过遍历参数注解，调用parseParameterAnnotation方法获取了注解中定义的参数值（path.value()），最后返回了new ParameterHandler.Path<>对象，在ParameterHandler类的Path实现中调用RequestBuilder类（创建请求）的addPathParam方法最终将相对路径relativeUrl的占位符通过描述网络请求接口方法传递的参数替换掉得到正确的相对路径relativeUrl。而得到正确相对路径的RequestBuilder对象创建则在RequestFactory类的create方法中。
### 调用接口中的方法返回Call对象
- 现在我们再次回到ServiceMethod的parseAnnotations方法，经过上面2.4小节的分析我们已经得到了解析到正确相对路径的RequestFactory对象，最终该方法调用了HttpServiceMethod.parseAnnotations()
```
     /**
      * Inspects the annotations on an interface method to construct a reusable service method that
      * speaks HTTP. This requires potentially-expensive reflection so it is best to build each service
      * method only once and reuse it.
      * HttpServiceMethod的parseAnnotations方法
      */
      static <ResponseT, ReturnT> HttpServiceMethod<ResponseT, ReturnT> parseAnnotations(
      Retrofit retrofit, Method method, RequestFactory requestFactory) {
      //最终通过retrofit对象拿到默认CallAdapter.Factory对象（ExecutorCallAdapterFactory）
      CallAdapter<ResponseT, ReturnT> callAdapter = createCallAdapter(retrofit, method);
      Type responseType = callAdapter.responseType();
      if (responseType == Response.class || responseType == okhttp3.Response.class) {
      throw methodError(method, "'"
          + Utils.getRawType(responseType).getName()
          + "' is not a valid response body type. Did you mean ResponseBody?");
       }
       if (requestFactory.httpMethod.equals("HEAD") && !Void.class.equals(responseType)) {
       throw methodError(method, "HEAD method must use Void as response type.");
       }
       //最终通过retrofit拿到设置的Converter.Factory对象（我们设置了GsonConverterFactory）
       Converter<ResponseBody, ResponseT> responseConverter =
        createResponseConverter(retrofit, method, responseType);

       okhttp3.Call.Factory callFactory = retrofit.callFactory;
       return new HttpServiceMethod<>(requestFactory, callFactory, callAdapter, responseConverter);
     }
     
```
- 通过该方法源码，首先我们可以知道最刚开始我们分析retrofit的create方法中loadServiceMethod(method)方法实际上就是**HttpServiceMethod对象**，接着继续看parseAnnotations方法中的createCallAdapter方法
```
    /**HttpServiceMethod 类的createCallAdapter方法**/
    private static <ResponseT, ReturnT> CallAdapter<ResponseT, ReturnT> createCallAdapter(
      Retrofit retrofit, Method method) {
    Type returnType = method.getGenericReturnType();
    Annotation[] annotations = method.getAnnotations();
     try {
      //noinspection unchecked
      return (CallAdapter<ResponseT, ReturnT>) retrofit.callAdapter(returnType, annotations);
      } catch (RuntimeException e) { // Wide exception range because factories are user code.
      throw methodError(method, e, "Unable to create call adapter for %s", returnType);
      }
    }
     /**Retrofit 类的callAdapter方法**/
     public CallAdapter<?, ?> callAdapter(Type returnType, Annotation[] annotations) {
     return nextCallAdapter(null, returnType, annotations);
     }
     /**Retrofit 类的nextCallAdapter方法**/
     public CallAdapter<?, ?> nextCallAdapter(@Nullable CallAdapter.Factory skipPast, Type returnType,
      Annotation[] annotations) {
    checkNotNull(returnType, "returnType == null");
    checkNotNull(annotations, "annotations == null");

    int start = callAdapterFactories.indexOf(skipPast) + 1;
    for (int i = start, count = callAdapterFactories.size(); i < count; i++) {
      CallAdapter<?, ?> adapter = callAdapterFactories.get(i).get(returnType, annotations, this);
      if (adapter != null) {
        return adapter;
      }
     }
     //省略部分代码
      ......
    }
```
- 由上源码HttpServiceMethod通过自身createCallAdapter调用Retrofit 类的callAdapter方法，而Retrofit 类的callAdapter方法又调用 Retrofit 类的nextCallAdapter方法遍历callAdapterFactories来得到CallAdapter.Factory对象；前面我们分析retrofit对象创建时已经说过Platform对象中提供了默认的CallAdapter.Factory对象为ExecutorCallAdapterFactory，该对象也就是HttpServiceMethod的createCallAdapter方法得到的CallAdapter.Factory对象。同理，HttpServiceMethod的createResponseConverter最终通过retrofit的nextResponseBodyConverter方法得到了Converter（GsonRequestBodyConverter）对象（我们设置了GsonConverterFactory.responseBodyConverter的方法创建了该对象），这里就不贴代码了。
- 接着上面的分析，我们在代理的invoke方法中返回了loadServiceMethod(method).invoke(args != null ? args : emptyArgs)这一句代码，当retrofit.create生成的的接口对象调用其中的接口的方法，则会触发动态代理执行invoke方法，最终返回loadServiceMethod(method).invoke，也就是执行了HttpServiceMethod.ivnoke方法
```
      /**HttpServiceMethod 类的invoke方法**/
     @Override ReturnT invoke(Object[] args) {
     return callAdapter.adapt(
        new OkHttpCall<>(requestFactory, args, callFactory, responseConverter));
     }
```
- 可以看到HttpServiceMethod类的invoke方法中调用了callAdapter.adapt，而这个callAdapter经过我们前面的分析已经知道是默认添加的ExecutorCallAdapterFactory对象，继续看ExecutorCallAdapterFactory对象
```
     /**ExecutorCallAdapterFactory 类**/
     final class ExecutorCallAdapterFactory extends CallAdapter.Factory {
     final Executor callbackExecutor;

     ExecutorCallAdapterFactory(Executor callbackExecutor) {
     this.callbackExecutor = callbackExecutor;
     }

      @Override public @Nullable CallAdapter<?, ?> get(
      Type returnType, Annotation[] annotations, Retrofit retrofit) {
      if (getRawType(returnType) != Call.class) {
      return null;
      }
      final Type responseType = Utils.getCallResponseType(returnType);
      return new CallAdapter<Object, Call<?>>() {
      @Override public Type responseType() {
        return responseType;
      }

      @Override public Call<Object> adapt(Call<Object> call) {
        return new ExecutorCallbackCall<>(callbackExecutor, call);
       }
      };
     }
     /**ExecutorCallbackCall 类**/
     static final class ExecutorCallbackCall<T> implements Call<T> {
     final Executor callbackExecutor;
     final Call<T> delegate;

     ExecutorCallbackCall(Executor callbackExecutor, Call<T> delegate) {
      this.callbackExecutor = callbackExecutor;
      this.delegate = delegate;
      }
     @Override public void enqueue(final Callback<T> callback) {
      checkNotNull(callback, "callback == null");
      delegate.enqueue(new Callback<T>() {
        @Override public void onResponse(Call<T> call, final Response<T> response) {
          callbackExecutor.execute(new Runnable() {
            @Override public void run() {
              if (delegate.isCanceled()) {
                // Emulate OkHttp's behavior of throwing/delivering an IOException on cancellation.
                callback.onFailure(ExecutorCallbackCall.this, new IOException("Canceled"));
              } else {
                callback.onResponse(ExecutorCallbackCall.this, response);
              }
            }
          });
        }
        @Override public void onFailure(Call<T> call, final Throwable t) {
          callbackExecutor.execute(new Runnable() {
            @Override public void run() {
              callback.onFailure(ExecutorCallbackCall.this, t);
            }
          });
        }
      });
      }
    //省略代码
    ......
    @Override public Response<T> execute() throws IOException {
      return delegate.execute();
    }
     //省略代码
    ......
    }
```
- 通过ExecutorCallAdapterFactory的源码应该会有一种让你恍然大悟的感觉
    
- 1.通过ExecutorCallAdapterFactory的adapt方法，我们我们已经得到了Call对象，它就是ExecutorCallbackCall
- 2.ExecutorCallbackCall的enqueue方法执行在主线程，callbackExecutor就是Platfrom默认添加的MainThreadExecutor（Android环境中），所以callback.onResponse中随意更新UI
- 3.delegate对象就是OkHttpCall对象，所以我们Call执行的enqueue和execute方法都是OkHttpCall对象对象的enqueue和execute方法。
### Retrofit中的适配器模式和装饰模式
- **适配器模式做为两个不同的接口之间的桥梁**，使得Retrofit2.Call接口可以使用OKHttp.Call的实现来执行网络请求，而适配器就是CallAdapter.Factory
- 装饰模式在不改变原有类的基础上进行扩展，也就是ExecutorCallbackCall，对OKHttp.Call进行装饰，本身它执行enqueue方法是在子线程中，而ExecutorCallbackCall对其装饰让其返回值执行在主线程中。
- 下面通过一张图来帮助理解
![retrofit适配器模式和装饰模式](https://github.com/maoqitian/MaoMdPhoto/raw/master/%E4%BB%8E%E6%BA%90%E7%A0%81%E8%A7%92%E5%BA%A6%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Retrofit2/retrofit%E9%80%82%E9%85%8D%E5%99%A8%E6%A8%A1%E5%BC%8F%E5%92%8C%E8%A3%85%E9%A5%B0%E6%A8%A1%E5%BC%8F.png)    
### Call对象执行网络请求
- 通过上一小节的分析，我们来看看OkHttpCall对象是如何执行网络请求的
```
     /*OkHttpCall类的enqueue方法*/
     @Override public void enqueue(final Callback<T> callback) {
     checkNotNull(callback, "callback == null");
     okhttp3.Call call;
     Throwable failure;
     synchronized (this) {
      if (executed) throw new IllegalStateException("Already executed.");
      executed = true;

      call = rawCall;
      failure = creationFailure;
      if (call == null && failure == null) {
        try {
          call = rawCall = createRawCall();
        } catch (Throwable t) {
          throwIfFatal(t);
          failure = creationFailure = t;
        }
      }
    }

     if (failure != null) {
      callback.onFailure(this, failure);
      return;
     }

     if (canceled) {
      call.cancel();
     }

     call.enqueue(new okhttp3.Callback() {
      @Override public void onResponse(okhttp3.Call call, okhttp3.Response rawResponse) {
        Response<T> response;
        try {
          response = parseResponse(rawResponse);
        } catch (Throwable e) {
          throwIfFatal(e);
          callFailure(e);
          return;
        }

        try {
          callback.onResponse(OkHttpCall.this, response);
        } catch (Throwable t) {
          t.printStackTrace();
        }
       }

      @Override public void onFailure(okhttp3.Call call, IOException e) {
        callFailure(e);
      }

      private void callFailure(Throwable e) {
        try {
          callback.onFailure(OkHttpCall.this, e);
        } catch (Throwable t) {
          t.printStackTrace();
        }
      }
      });
     }
    /*OkHttpCall类的execute方法*/
    @Override public Response<T> execute() throws IOException {
     okhttp3.Call call;
     synchronized (this) {
      if (executed) throw new IllegalStateException("Already executed.");
      executed = true;
      //省略部分代码
      .......
      call = rawCall;
      if (call == null) {
        try {
          call = rawCall = createRawCall();
        } catch (IOException | RuntimeException | Error e) {
         //省略部分代码
        .......
      }
      return parseResponse(call.execute());
     }
```
- 通过以上源码，我们可以看到执行完全请求的是okhttp3.Call对象，也就是说retrofit的网络请求时由OKHttp3来完成的，继续看得到okhttp3.Call对象的createRawCall()方法
```
     /*OkHttpCall类的createRawCall方法*/
     private okhttp3.Call createRawCall() throws IOException {
     okhttp3.Call call = callFactory.newCall(requestFactory.create(args));
     if (call == null) {
      throw new NullPointerException("Call.Factory returned null.");
     }
     return call;
     }
    /*retrofit类的build方法*/
     public Retrofit build() {
      okhttp3.Call.Factory callFactory = this.callFactory;
      //省略部分代码
        .......
      if (callFactory == null) {
        callFactory = new OkHttpClient();
      }
      //省略部分代码
        .......
     }
     
```
- 通过OkHttpCall类的createRawCall方法和retrofit类的build方法，callFactory.newCall，也就是OkHttpClient.nawCall，而requestFactory.create()返回的就是OKHttp的 request 对象，经过前面的分析，RequestFactory解析好的请求数据传递给了OkHttpClient。
- 关于OkHttpClient是如何执行网络请求的可以看我之前的一篇文章
    
- [从源码角度深入理解OKHttp3](https://www.jianshu.com/p/cdbb11f2abe8) 
    
### Converter对象转换网络请求成功返回数据
- 通过上一小节，我们已经通过OkHttp进行了实际的网络请求，请求成功的数据根据之前新建Retrofit对象的时候我们指定了GsonConverterFactory，并在HttpServiceMethod通过createResponseConverter方法得到了GsonResponseBodyConverter对象（可以查看2.5小节分析）。
- 请求成功返回的数据会经过OkHttpCall类的parseResponse方法进行处理
```
     /*OkHttpCall类的parseResponse方法*/
     Response<T> parseResponse(okhttp3.Response rawResponse) throws IOException {
     ResponseBody rawBody = rawResponse.body();

     // Remove the body's source (the only stateful object) so we can pass the response along.
     rawResponse = rawResponse.newBuilder()
        .body(new NoContentResponseBody(rawBody.contentType(), rawBody.contentLength()))
        .build();
    //省略部分代码
        .......
     ExceptionCatchingResponseBody catchingBody = new ExceptionCatchingResponseBody(rawBody);
     try {
      T body = responseConverter.convert(catchingBody);
      return Response.success(body, rawResponse);
     } catch (RuntimeException e) {
       //省略部分代码
        .......
     }
    }
    /* GsonResponseBodyConverter类*/
    final class GsonResponseBodyConverter<T> implements Converter<ResponseBody, T> {
    private final Gson gson;
    private final TypeAdapter<T> adapter;

    GsonResponseBodyConverter(Gson gson, TypeAdapter<T> adapter) {
    this.gson = gson;
    this.adapter = adapter;
     }
     @Override public T convert(ResponseBody value) throws IOException {
     JsonReader jsonReader = gson.newJsonReader(value.charStream());
     try {
      T result = adapter.read(jsonReader);
      if (jsonReader.peek() != JsonToken.END_DOCUMENT) {
        throw new JsonIOException("JSON document was not fully consumed.");
      }
      return result;
       } finally {
      value.close();
      }
     }
    }
```
- 通过以上源码可以看到通过Retrofit对象指定GsonConverterFactory后得到的GsonResponseBodyConverter的对象帮我们把Json数据通过Gson处理成我们指定的bean对象，很方便。到此，retrofit的源码分析就结束了。    
- 最后通过一张类之间调用流程图来帮助更好理解retrofit源码
    
![Retrofit网络请求类流程图(Android)](https://github.com/maoqitian/MaoMdPhoto/raw/master/%E4%BB%8E%E6%BA%90%E7%A0%81%E8%A7%92%E5%BA%A6%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Retrofit2/Retrofit%E7%BD%91%E7%BB%9C%E8%AF%B7%E6%B1%82%E7%B1%BB%E6%B5%81%E7%A8%8B%E5%9B%BE.png)
## 最后说点
> Retrofit整个框架源码量其实不算太多，其中巧妙的运用设计模式来完成整个框架设计。静下心来阅读源码对自己的提升还是很有帮助的。由于本人水平有限，文章中如果有错误，请大家给我提出来，大家一起学习进步，如果觉得我的文章给予你帮助，也请给我一个喜欢和关注。

- 参考链接
  
  - [Retrofit分析-经典设计模式案例](https://www.jianshu.com/p/fb8d21978e38)
  - [官方文档](https://square.github.io/retrofit/)
    

  