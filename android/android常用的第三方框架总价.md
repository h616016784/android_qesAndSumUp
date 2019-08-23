
# 1、框架总览
![常用第三方框架总览]( https://github.com/h616016784/android_qesAndSumUp/raw/master/pic/thirdSum.jpg )
# 2、网络框架对比
  Apache 的HttpClient 和 Java 的HttpURLConnection，无论我们是自己封装的网络请求类还是第三方的网络请求框架，都离不开这两个类库。
  
  在Android 2.2版本及其之前，HttpURLConnection一直存在着一些令人厌烦的bug。比如说对一个可读的InputStream调用close（）方法时，就有可能会导致连接池失效。我们通常的解决办法就是直接禁用连接池的功能。所以在Android 2.2版本及其之前的版本使用HttpClient是较好的选择；而在Android 2.3版本及其之后，HttpURLConnection 则是最佳的选择，它的 API 简单，体积较小，因而非常适用于Android项目。HttpURLConnection的压缩和缓存机制可以有效地减少网络访问的流量，在提升速度和省电方面也起到了较大的作用。另外在Android 6.0版本中，HttpClient库被移除了，如果不引用HttpClient，HttpURLConnection则是以后我们唯一的选择。
 ## 2.1 Volley
  简单地进行HTTP通信、加载网络，适合去进行数据量不大，但通信频繁的网络操作，高分辨率的图像压缩上有很好的支持，封装了URL图片加载框架；可扩展性好，可支持HttpClient、HttpUrlConnection和Okhttp。
  参考地址： [郭神的volley系列](https://blog.csdn.net/guolin_blog/article/details/17482095)
 ## 2.2  OkHttp
  Android 2.3以上，可以理解为类似于HttpURLConnection的一个东西，非常高效，支持SPDY、连接池、GZIP和 HTTP 缓存；
  基于NIO和Okio，所以性能比较好，请求处理速度快（IO:阻塞式；NIO:非阻塞式；Okio是Square公司基于IO和NIO做的一个更简单、高效处理数据流的一个库）;
  4.4开始HttpURLConnection的底层实现采用的是okHttp。
  ## 2.3 Retrofit
  基于Okhttp,restful Api设计风格，支持NIO，速度上比volley更快。
  Square公司开源的， 基于注解，可以通过注解直接配置请求，你可以使用不同的 http 客户端，虽然默认是用 http ，可以使用不同 Json Converter 来序列化数据，同时提供对 RxJava 的支持

# 3、RxJava
  函数响应式编程可以极大地简化项目，特别是处理嵌套回调的异步事件、复杂的列表过滤和变换或者时间相关问题。
  RxJava是ReactiveX的一种Java实现。
## 3.1 为何要用RxJava
  说到异步操作，我们会想到Android的AsyncTask和Handler。但是随着请求的数量越来越多，代码逻辑将会变得越来越复杂而RxJava却仍旧能保持清晰的逻辑。RxJava的原理就是创建一个Observable对象来干活，然后使用各种操作符建立起来的链式操作，就如同流水线一样，把你想要处理的数据一步一步地加工成你想要的成品，然后发射给Subscriber处理。
## 3.2 RxJava与观察者模式
  RxJava的异步操作是通过扩展的观察者模式来实现的，不了解观察者模式的读者可以先看一下6.5.3节。RxJava有4个角色Observable、Observer、Subscriber和Suject，这4个角色后面会具体讲解。Observable和Observer 通过subscribe方法实现订阅关系，Observable就可以在需要的时候通知Observer。
# 4、图片框架
  UniversalImageLoader、Fresco，和主要想说的Glide
  从易用性上来讲，Glide和Picasso应该都是完胜其他框架的
## 4.1其中值得思考的是缓存机制
  内存缓存：采用了LruCacshe和弱引用Map的组合，更加高效，分担了LruCash的部分压力，并且key的运算也可借鉴并可以重写满足自己的需求
  硬盘缓存：内部参数DiskLruCashe类。
## 4.2图片的监听、预加载和转换功能
  监听可以与预加载结合起来，图片加载完成后做什么事情等；预加载方法是preload和downloadOnly
  转换原理：大致的流程是我们可以获取到原始的图片，然后对图片进行变换，再将变换完成后的图片返回给Glide
  转换功能：图片默认是FitCenter,而glide会根据传入的图片类型而进行转换导致图片变换、全屏等，这时可以通过设置override来修改dontTransform。
  转换包括：裁剪变换、颜色变换、模糊变换、圆角化、圆形化、黑白化、模糊化等
  
