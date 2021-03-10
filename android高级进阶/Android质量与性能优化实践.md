# 一、概要：

  服务类产品中一般用 SLA (Service-LevelAgreement)作为衡量产品服务等级的量化指标， 而移动 App 有着自己独特的运行环境和应用场景，移动 App 的核心质量需要关注稳定、用户 体验、性能等多方面，具体到产品，需要根据业务制定对应的量化指标，例如 App 的 Crash 率就是衡量 App 稳定性的 一个重要的数据指标。
  
Google Android 官方提供了一套应用核心质量的质量标准，让我们的 App在发布之前参 考这些标准进行测试，确保在众多的设备上正常稳定运行，满足 Android 导航和设计标准， 井为 Google Play 商店开展推广做好准备(当然，具体我们的 App 测试范围远不止这些， Android 官方提供的只是 App 应具备的基本质量特征，更多测试参考本章 的质量和稳定性手
段以及测试专场中的详细讨论) 。Google 官方的标准[ 15]具体分为以下 4 个部分 。
 - 视觉设计和用户互动，提供一致、直观的用户体验。
 - 功能，遵循这些标准确保你的应用使用合适的权限级别，提供预期功能行为
 - 性能和稳定性，遵循这些标准确保你的应用提供用户预期的性能、稳定性和响应速度 。
 - Google Play，遵循这些标准确保你的应用做好在 Google Play 发布的准备 。

注意：质量优化是一个大而系统的功能，需要产品、UI、研发、测试统一认知、协同合作，方可完成，而不是单一一方努力的结果。
# 二、稳定性与性能优化
这一部分主要研发的工作。目的提高用户体验，对于用户来说表现为： 流畅（不卡顿） 稳定 省电、省流量 安装包小；对于程序员来说则是尽量减少内存消耗、提高运行效率、减少代码冗余等...

## 1、制定指标
优化无止境，只有先制定指标才能确定做的度，才不会陷入优化的泥潭中。

### 1）崩溃：
具体的指标其实也是要按实际情况来具体定的。

- 崩溃率（优秀：0～2%%，标准：2～4%%，轻微隐患：4～12%%，严重隐患：12%%以上）常表现为出现crash
  
  UV崩溃率，UV崩溃率=日活跃用户中崩溃用户数/日活跃用户数
  PV崩溃率，PV崩溃率=（一段时间）所有活跃用户中崩溃用户数/所有活跃用户数  
- 崩溃Top

  Top崩溃问题
  Top崩溃机型
- 崩溃次数

  人均崩溃次数
  页面崩溃次数

### 3）交互性能差 （不涉及网络，只是本地渲染）
（优秀：0～300ms,标准：300ms～400ms，轻微隐患：400ms～1000ms，严重隐患：1000ms以上）电话短信干扰、低电量提醒、push提醒、usb数据线插拔提醒、充电提醒

 - UI 绘制与刷新
   也可具体操作也可以参考[Android App性能评测分析－流畅度篇](https://www.jianshu.com/p/642f47989c7c)
   主要是计算出帧率、丢帧率、流畅度，纬度为高、平局、低
  ![流畅度度量]( https://github.com/h616016784/android_qesAndSumUp/raw/master/pic/1614916218804.jpg )
   A）、基于gfxinfo方案
   
      FPS（系统合成帧率）

         获取方法：
         GPU呈现模式分析；
         通过gfxinfo获取，adb shell dumpsys gfxinfo yourpackagename；
         数据形式最为直观（FPS 是最早的显示性能指标，而且在多个平台中都有着类似的定义），且对系统平台的要求最低（API level 1），游戏、视频等连续绘制的应用可以考虑选用，但不适用于绝大多数非连续绘制的应用；
         计算公式：1000ms / 60 frames ≈ 16.67 ms/frames

         用途：监控

   B）、基于FrameStats 方案( Android 6.0, API 23+)
   
     通过以下命令，可以获取每一帧每个关键点的绘制过程中的耗时信息（纳秒时间戳），仅针对应用生成的最后120帧数据。
     命令：adb shell dumpsys gfxinfo pkg name framestats
   C)、基于SurfaceFlinger方案
   
         通过系统层级SurfaceFlinger获取，adb shell dumpsys SurfaceFlinger packagename；
   D）、基于Choreographer的方案（Android 4.1, API 16+）
   E）、android的block Canary组件

 - 启动 ：安装启动、冷启动、热启动(优秀：1s以内；标准：1s-2s；差：2s之外)

    建议质量标准为 1、快开慢开比；2、90%用户的启动时间

    一些额外的学习可参考地址[Android App性能评测分析-启动时间篇](https://www.jianshu.com/p/fe81e4b4c5ba),[如何统计Android App启动时间](https://www.jianshu.com/p/59a2ca7df681)
 
    非常不错，实战性非常好的一篇文章[深入探索Android启动速度优化](https://jsonchao.github.io/2019/11/10/%E6%B7%B1%E5%85%A5%E6%8E%A2%E7%B4%A2Android%E5%90%AF%E5%8A%A8%E9%80%9F%E5%BA%A6%E4%BC%98%E5%8C%96/)
 
   App启动时间的度量方式
   
   A）、adb shell 方式。命令为 adb shell am start -W [pkg_name]/[activity]
         缺点:
      应用的启动过程往往不只一个Activity，有可能是先进入一个启动页，然后再从启动页打开真正的首页。某些情况下还有可能中间经过更多的Activity，这个时候需要将多个Activity的时间加起来。
      将多个Activity启动时间加起来并不完全等于用户感知的启动时间。例如在启动页可能是先等待某些初始化完成或者某些动画播放完毕后再进入首页。使用命令行统计的方式只是计算了Activity的启动以及初始化       时间，并不能体现这种等待任务的时间。
      没有在AndroidManifest.xml对应的Activity声明中指定<intent-filter>或者属性没有android:exported="true"的Activity不能使用这种命令行的形式计算启动时间
   
   B）、logcat 方式。android4.4之后，Android在系统Log中添加Display的Log信息，可以通过过滤ActivityManager及Display关键字，抓去Log中的启动时间信息。命令为adb logcat｜grep “ActivityManager”。
   
   C）、TraceView 工具。我们在 UI 和 CPU 性能优化中介绍了 TraceView，其可以完整地显示每个函数/方法的时间消耗，有两种使用方式。

        直接通过 DDMS 的 starttraceview 启动，弹窗选择 trace 模式开始记录 。
        代码集成方式，在需要调试的地方加入 Debug.startMethodTracing(”xx”)，在结束的地方加入 Debug.stopMethodTracing()，运行后将生成 XX.trace文件，然后
        通过 DDMS 打开该 trace 文件即可分析，注意需要” android.permission.WRITE_EXTERNAL STORAGE”权限。
   D)、线上统计
   
       要考虑冷启动、热启动、温启动三种情况下的代码的统计时间 
 - 跳转
   页面切换   前后台切换
      
      相关知识可以学习：[Android Activity启动耗时统计方案](https://juejin.cn/post/6844903935627493383#heading-1)
      
      页面切换包括 ActiivtyA Pause流程、ActivityB Launch流程、ActivityB Render流程 三个过程（优秀：0-300，标准：300-500，良：500-1000）

 - 响应
   按键
   系统事件
   滑动
### 5）卡顿、黑白屏   

### 4）网络请求响应时间

相关原理参考[OkHttp请求耗时统计和实践](https://juejin.cn/post/6875626117265358855)

（优秀：0～400ms，标准：400ms～2000ms，轻微隐患：2000ms～5000ms，严重隐患：5000ms以上),应用发出一个HTTP请求到主机，主机端返回响应所用的时间，可分为强网和弱网，强网不做介绍，弱网下，如电梯里、地铁上网络信号差时，app页面一直转圈加载。


  开发如何测量：1）简单的在网络请求的开始和结束计算时间并打印
              2）如果用okhttp，那么可以用EventListener回调原理来测量各个时间阶段的时间
              
  开发如何分析：1）抓包工具 TcpDump、Fiddler、Wireshark、Charles等
              2）借助于Android Monitor（Network）的DDMS进行统计分析
   
### 2）网络错误率
常见的网络错误包括网络异常、Http错误、网络超时等

也是基于平均值UV PV的统计

  开发主要关心两个维度：1）纯网络错误、2）业务错误
  
  开发如何测量：代码介入，在发生异常的地方打印日志或者搜集数据
  
  开发如何分析：借助于Android Monitor（Network）的DDMS进行统计分析
### 7）流量占用情况
的app占据第一位，流量跑的离谱，则存在果断卸载的可能

网络流量度量标准，分为总接收量、总发送量、Mobile的总接收量、Mobile的总发送量、Uid进程的总接收流量、Uid进程的总发送流量。流量的度量慢慢的不那么重要，但是对性能有一定的参考意义。如果流量过大说明发送的数据可压缩，响应时间也会变长。
 
  每秒钟平均流量，建议值<5.12kb，每10分钟平均流量，建议值<3MB,存在app偷跑流量等行为
  
  开发如何测量：流量可以通过TrafficStats （总流量）来获取，也可以用Traffic tool 中setThreadStatsTag(int)在代码中具 体标记，标记的流量数据在 Android 中存放的目录为/proc/net/xt_qtaguid/stats，通过adb shell手动获取。
  

### 6）内存性能
  每个Android为APP的内存分配有限制，初始的内存是16M，但是各厂家都会定制化自己的内存限制，通常的限制是512M，可以通过代码获取的APP的内存限制
  每个APP的内存标准分为 APP空运行内存量（0M-200M）；APP一段时间内高峰操作内存值（300M）；

  开发端测量方法：使用android studio的profiler工具。


### 8）APP的Size优化
  具体也可参考 [深入探索 Android 包体积优化（匠心制作-上）](https://juejin.cn/post/6844904103131234311)

  App包的size应该根据业务的发展而定，项目初期size要小一些，随着业务的发展逐步扩大。
  Google Play 要求用户下载的压缩后 APK 大小不超过 100 MB。使用 Android App Bundle 上传您的应用，这种方式允许的压缩后下载大小上限为 150 MB。
  
  项目初期的APK大小为（0-30M，30M-50M）项目功能复杂（10多个功能模块）（50M-90M）项目极其复杂的时候（90M以上）
  
  开发如何分析：1）使用Android studio 的APK Analyzer
              2）使用 ApkTool 反编译工具分析 APK
              3）使用 nimbledroid 进行 APK 性能分析
              4）使用 android-classshark 进行 APK 分析
### 9）耗电量
  原理参考[深入探索 Android 电量优化](https://juejin.cn/post/6844904195523346439#heading-35)

  平均标准：
  无网络待机：<1OmA；WIFI待机：<20mA ；3G网络待机：<20mA；亮屏无操作：<300mA ；看视频：<500mA ；灭屏下载：<300mA
  手机中的耗电大户/主要耗电场 景
  - CPU 相关 。
        复杂运算逻辑、无限循环等会直接导致 CPU 负载过高，耗电剧增 。
  - 手机屏幕。
        毋庸置疑，于机中最耗电的模块肯定是屏幕了 。亮屏时间越长，电量消耗越快 。

  - 网络相关 。
        一般情况下，网络相关(网络请求、数据传输、网络切换等)是仅次于屏幕的耗电大户 。例如网络请求，涉及通过内置的射频模块与基站通信，而该射频模块又涉及一系列驱动和底层的支持，非常耗电，再如大量数据的传输等。2009 年 Google I/O 大会 Jeffrey Sharkey 的演讲“ Coding for Life- Battery Life， That is[8]中就总结了 Android应用耗电主要在大数据传输、不停的网络间切换以及解析大量文本数据 3 个方面，而这 3 方面其实都是直接或间接地跟网络相关的 。
  - WakeLock(Android)。 
        WakeLock是Android系统中用于优化电量使用的一种手段，通过在用户 一段时间没有操作的情况下让屏幕和CPU 进入休眠状态来减少电量消耗 。一些应用中出于特定业务场景调用 PowerManager.WakeLock 来使 CPU 保持 持续运转，而释放需要时间，甚至你根本就忘记释放了，灭屏后 CPU 却还一直运 转着 ，从而大大增加了耗电量。
  - GPS。 
        GPS 定位涉及 GPS 位置传感器，也是一位不折不扣的耗电大户 。平时不使用 GPS 的时候，记得把它给关了 。
  - CameraoCamera
        涉及前后摄像头硬件，如果一直使用(录屏等)，耗电也会非常可观。
  
  开发如何测量：1）dumpsys batterystats，Android 5.0 提供的工具，它可以获取各个 App 的 WakeLock、CPU 时间占用等信息，同时增加了一个 Estimated power use（mAh）功能，预估耗电量。
              2）battery-historian 

### 10）FPS
FPS大于18帧比率，建议值大于90%，具体参考1）步骤
### 11）cpu使用率
建议值>90%，cpu频率设置过高时会导致过热,导致耗电更严重，cpu频率设置过低导致手机滞后，应用处理缓慢同样导致耗电，则优就好，避免被卸掉
### 12）存储

