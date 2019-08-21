
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

