性能优化
==================
# 1、总体目标
性能优化的目的提高用户体验，对于用户来说表现为：
  流畅
  稳定
  省电、省流量
  安装包小
  
  对于程序员来说则是尽量减少内存消耗、提高运行效率、减少代码冗余等...
  
# 2、原理
 ## 2.1、 流畅
  ### 卡顿的场景有很多，按场景可以分为4类：UI 绘制、应用启动、页面跳转、事件响应，如图：
  ![卡顿场景]( https://github.com/h616016784/android_qesAndSumUp/raw/master/pic/clipboard.png )
  
  这4种卡顿场景的根本原因可以分为两大类：

  界面绘制。主要原因是绘制的层级深、页面复杂、刷新不合理，由于这些原因导致卡顿的场景更多出现在 UI 和启动后的初始界面以及跳转到页面的绘制上。
  数据处理。导致这种卡顿场景的原因是数据处理量太大，一般分为三种情况，
    一是数据在处理 UI 线程，
    二是数据处理占用 CPU 高，导致主线程拿不到时间片，
    三是内存增加导致 GC 频繁，从而引起
    
 ### 卡顿根本原因
  根据Android 系统显示原理可以看到，影响绘制的根本原因有以下两个方面：
  绘制任务太重，绘制一帧内容耗时太长。
  主线程太忙，根据系统传递过来的 VSYNC 信号来时还没准备好数据导致丢帧
主线程主要做以下几个方面工作：
 ### UI 生命周期控制
  系统事件处理
  消息处理
  界面布局
  界面绘制
  界面刷新
  除此之外，应该尽量避免将其他处理放在主线程中，特别复杂的数据计算和网络请求等。
  
# 3、优化方案
 ## 3.1、布局优化（思路：选择性能耗少布局、减少层级嵌套、提高布局复用性、减少测量和绘制时间）
   优化点:
  - 避免OverDraw
  - 优化布局层级
  - 避免过多无用嵌套
  - 使用标签 重用layout
  - 尽量避免使用权重，其会导致绘制两遍
  - 使用延迟加载
  - 尽可能少用wrap_content。wrap_content会增加布局 measure时计算成本，在已知宽高为固定值时，不用wrap_content。
  - Hierarchy View  layout inspace进行层级分析
  
  ## 3.2、内存优化
    内存优化的点有很多,这里我主要分为两大块:
   ###  Bitmap优化
      主动释放Bitmap资源，手动调用recycle()方法，释放其Native内存
      主动释放ImageView的图片资源，避免大图小view显示
      Png图，NinePatch图多重考虑
      使用大图之前，尽量先对其进行压缩（尺寸或者清晰度）
   ### 防止内存泄露
    单例（主要原因还是因为一般情况下单例都是全局的，有时候会引用一些实际生命周期比较短的变量，导致其无法释放），ActivityManager 单例添加Activity实例对象引发的内存泄漏
    静态变量（同样也是因为生命周期比较长）
    Handler内存泄露[7]
    context引起的内存泄露
    匿名内部类（匿名内部类会引用外部类，导致无法释放，比如各种回调）
    资源使用完未关闭（BraodcastReceiver，ContentObserver，File，Cursor，Stream，Bitmap）
    防止内存抖动：
      主要是循环中大量创建、回收对象
    
   ## 3.3 代码优化
  优化点:
  - 对常量使用static修饰符
  - 使用静态方法
  - 减少不必要的成员变量
  - 尽量不要使用枚举,少用迭代器
  - 使用android优化过的集合如，用android的稀疏家族（SparseArray内部原理）
  - 对Cursor、Receiver、Sensor、File等对象,要注意它们的创建、回收与注册、反注册
  - 避免大量使用注解（现在有编译期就生成代码的可以）、反射
  - 使用RenderScript、OpenGL来进行复杂的绘图操作
  - 使用SurfaceView来替代View进行大量、频繁的绘图操作
  - 尽量使用视图缓存,而不是每次都执行inflate()方法解析视图
  
  ## 3.4 安装包大小优化
   ### 3.4.1、不必每个dpi都适配（x xx xxx就可以）
  建议取720p的资源，放到xhdpi目录
   ### 3.4.2、建议开启minifyEnabled混淆代码，开启shrinkResources去除无用资源，删除无用的语言资源。
   使用 lint 去除无用资源文件
   ### 3.4.3、使用tinypng有损压缩，或者缩小大图、压缩第三库里的大图
  Tinypng的官方网站：http://tinypng.com/
   ### 3.4.、4使用jpg格式
  对于非透明的大图，jpg将会比png的大小有显著的优势
   ### 3.4.5、使用webp格式，
  webp支持透明度，压缩比比jpg更高但显示效果却不输于jpg，从Android 4.0+开始原生支持，但是不支持包含透明度，直到Android 4.2.1+才支持显示含透明度的webp，使用时要特别注意。
  ### 3.4.6、使用shape背景
  一下的两个方案可选
  第15条：使用着色方案
  相信你的工程里也有很多selector文件，也有很多相似的图片只是颜色不同，通过着色方案我们能大大减轻这样的工作量，减少这样的文件。借助于android support库可实现一个全版本兼容的着色方案，参考代码：DrawableLess.java（https://github.com/openproject）
第12条：使用微信资源压缩打包工具（也许好用？？？）
其他：删除armable-v7包下的so、删除x86包下的so、避免重复库、使用更小的库、在线化素材库。
  ## 3.5 耗电优化
  计算优化，避开浮点运算等。
  避免 WaleLock 使用不当。
  使用 Job Scheduler
  ## 3.6 　网络方面的优化
  网络方面的优化，本项目做过的优化有： token失效优化（okhttp拦截器），默认的压缩与缓存机制，监听设备状态在wifi的情况下splash图片预加载、断点续传、上传错误日志，监听网络状态好的时候上传图片，弱网下的判断和断线处理机制、
  ### 3.6.1 token失效问题
  参考地址[Android处理token失效的处理方法](https://blog.csdn.net/xulike1990/article/details/60581466)
  ### 3.6.2 网络重连机制
  参考地址:[Volley请求的重发机制](https://blog.csdn.net/jez/article/details/85762630)
  
  而okhttp可以在创建okhttpclient的时候设置是否断线重连，默认是true。其重连机制参考地址[okHttp重试机制](https://blog.csdn.net/fengrui_sd/article/details/79004691)
  如果我们想自定义重连机制，则需要自定义拦截器, 参考地址[OkHttp自定义重试次数](https://blog.csdn.net/zhousenshan/article/details/86664768)
   ### 3.6.3 弱网条件下的优化
   参考地址：[android弱网下优化](https://blog.csdn.net/u010648159/article/details/76511786)
   ### 3.6.4 网络优化方案
   其中要先知道的的基础知识：
   
   1,Http的报文压缩原理  参考地址[Http之报文压缩](https://blog.csdn.net/lvxiangan/article/details/77568801)
   
   okhttp和volley的使用可参考[HttpUrlConnection之gzip相关](https://blog.csdn.net/devilnov/article/details/53540585)
   
   关于http压缩与网络框架的也可以参考[聊聊HTTP gzip压缩与常见的Android网络框架](https://www.cnblogs.com/ct2011/p/5835990.html)
   
   2,http的缓存原理，参考地址[Http缓存机制与原理](https://blog.csdn.net/jutal_ljt/article/details/80021545)
   okhttp使用缓存的策略实例有：有网时请求网络，无网络时使用缓存数据，参考地址[OkHttp缓存使用指南](https://segmentfault.com/a/1190000012922833)
   3，监听设备状态来做优化处理
   可以参考地址[JobScheduler](https://www.cnblogs.com/leipDao/p/8268918.html)
   
   4，弱网条件下优化

# 4、标准的制定
其实标准的制定应该是第一步，要不然优化永无止境。

(Android性能测试关注的指标整理)[https://www.jb51.net/article/172834.htm]

(移动端性能标准值及定义整理)[http://120.221.160.5:9002/lrhealth_dicomh5/#/?studyid=2751060106853687296&accessToken=D7567D58FC584E7BB23A229218EBF14B&tenantKey=2715972430958149632]
