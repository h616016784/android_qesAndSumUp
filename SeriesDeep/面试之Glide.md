# 一、Glide的运行原理
  RequestManager with = Glide.with(this);
    RequestBuilder<Drawable> load = with.load(url);
load.into(iv); 
  
  三条主线：
 ## 1、搞清楚这个请求发送到哪里去了
  RequestTracker  两个队列 一个requests  pendingRequests  请求过来之后，都是存入这两个队列；
从Glide具有生命周期的特性开始推算：

RequestManager在进行维护队列；
问题：不知道谁去维护？
谁在处理； 
  
 ## 2、为什么RequestManager能够管理生命周期？
  通过RequestMangerRetriever创建一个无UI的Fragment，并将这个Fragment的生命周期绑定到RequestManager上；

Glide生命周期怎么处理的？
为什么要有生命周期？
Glide队列请求如何处理的？
  
1：当我们load.into(iv);方法时，所有的请求都会被添加到一个叫RequestTracker的队列中，而且这个队列有两个
一个是运行时队列，一个是等待队列；
如果当前页面停止，onStop方法被调用，所有的运行中的请求都会停止，并且全部添加到等待队列中；
当开始运行时，又会把所有等待队列中的请求 放到运行中去！

队列是如何维护的？
RequestManager with = Glide.with(this);
通过这句代码，创建了一个RequestManager，并且在Glide.with方法中，为传入的this（Acitivity）创建一个无UI的Fragment，
并且将Fragment的生命周期绑定到RequestManager上，当Activity触发了onstop等方法时，则会隐式的调用Fragment的onstop方法，再通过fragment的onstop调用RequestManager的onstop方法，以此来管理两个请求队列中的请求；

## 3、第三条主线
  1:当RequestManager调用requestTracker.runRequest的时候，
  
2: 所有的请求都会构建成一个SingleRequest，并且开始调用begin方法运行；
   
3：begin方法最终会调用engine.load方法
   
4：根据request构建EngineKey
  
5：根据EngineKey去活动缓存中获取数据
  
6：如果获取不到，去内存缓存中获取数据
  
7：如果获取不到，通过硬盘缓存的线程池去获取本地硬盘的数据
  
8：如果获取不到本地的，通过网络的线程池去获取网络的数据
  
# 二、Glide跟其他框架相比优势在哪里？

   1：生命周期得管理
  
   2：支持gif  picasso支持gif
   
   3：三级缓存，内存缓存中还分为活动缓存和内存缓存；活动缓存指得是讲正在使用得图片用弱引用缓存，使用完成后到内存缓存；再到磁盘缓存；
  
   4：占用内存小，它默认得编码格式是rgb565；  picasso用得argb8888 ImageLoader不支持gif图片加载 而且也很老了
  
 # 三、如何设计自己的图片加载框架
1：Glide手写资源封装
2：活动缓存
3：内存缓存
 4：磁盘缓存
5：生命周期
  
  Glide源码流程分析；
Glide生命周期怎么做到的？














