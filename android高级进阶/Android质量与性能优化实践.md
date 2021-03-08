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
 
### 2）网络错误率
常见的网络错误包括网络异常、Http错误、网络超时等

也是基于平均值UV PV的统计
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
   页面切换
   前后台切换
 - 响应
   按键
   系统事件
   滑动
   

### 4）网络请求响应时间
（优秀：0～400ms，标准：400ms～2000ms，轻微隐患：2000ms～5000ms，严重隐患：5000ms以上),应用发出一个HTTP请求到主机，主机端返回响应所用的时间，可分为强网和弱网，强网不做介绍，弱网下，如电梯里、地铁上网络信号差时，app页面一直转圈加载

### 5）卡顿、黑白屏
### 6）内存泄漏
### 7）流量占用情况
  每秒钟平均流量，建议值<5.12kb，每10分钟平均流量，建议值<3MB,存在app偷跑流量等行为，当用户看app占用流量时，如你
### 8）APP的Size优化
### 9）耗电量

的app占据第一位，流量跑的离谱，则存在果断卸载的可能
### 10）FPS
FPS大于18帧比率，建议值大于90%
### 11）cpu使用率
建议值>90%，cpu频率设置过高时会导致过热,导致耗电更严重，cpu频率设置过低导致手机滞后，应用处理缓慢同样导致耗电，则优就好，避免被卸掉
### 12）存储

