Android保活
===========
# android保活方案
 参考基本是照抄^_^ ^_^ ^_^ 以下文章
 >https://www.jianshu.com/p/b5371df6d7cb
 >http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2016/0418/4158.html
 >https://www.jianshu.com/p/1da4541b70ad
 >https://superxlcr.github.io/2017/04/24/Android%E8%BF%9B%E7%A8%8B%E4%BF%9D%E6%B4%BB%E6%80%BB%E7%BB%93/
 
## 1.android进程被杀死的原理
 Android系统把进程的划为了如下几种（重要性从高到低,备注：严格来说是划分了6种）====================
 
 ### 1.1前台进程(Foreground process)
 场景：
 某个进程持有一个正在与用户交互的Activity并且该Activity正处于resume的状态。
 某个进程持有一个Service，并且该Service与用户正在交互的Activity绑定。
 某个进程持有一个Service，并且该Service调用startForeground()方法使之位于前台运行。
 某个进程持有一个Service，并且该Service正在执行它的某个生命周期回调方法，比如onCreate()、 onStart()或onDestroy()。
 某个进程持有一个BroadcastReceiver，并且该BroadcastReceiver正在执行其onReceive()方法。

 用户正在使用的程序，一般系统是不会杀死前台进程的，除非用户强制停止应用或者系统内存不足等极端情况会杀死。
 ### 1.2、可见进程(Visible process)
 场景：

 拥有不在前台、但仍对用户可见的 Activity（已调用 onPause()）。
 拥有绑定到可见（或前台）Activity 的 Service

 用户正在使用，看得到，但是摸不着，没有覆盖到整个屏幕,只有屏幕的一部分可见进程不包含任何前台组件，一般系统也是不会杀死可见进程的，除非要在资源吃紧的情况下，要保持某个或多个前台进程存活
 ### 1.3、服务进程(Service process)

 场景：

 某个进程中运行着一个Service且该Service是通过startService()启动的，与用户看见的界面没有直接关联。
 在内存不足以维持所有前台进程和可见进程同时运行的情况下，服务进程会被杀死
 ### 1.4、后台进程(Background process)

 场景：

 在用户按了"back"或者"home"后,程序本身看不到了,但是其实还在运行的程序，比如Activity调用了onPause方法
 系统可能随时终止它们，回收内存
 ### 1.5、空进程(Empty process)

 场景：

 某个进程不包含任何活跃的组件时该进程就会被置为空进程，完全没用,杀了它只有好处没坏处,第一个干它!===================================

 上面是进程的分类，进程是怎么被杀的呢？系统出于体验和性能上的考虑，app在退到后台时系统并不会真正的kill掉这个进程，而是将其缓存起来。打开的应用越多，后台缓存的进程也越多。在系统内存不足的情况下，系统开始依据自身的一套进程回收机制来判断要kill掉哪些进程，以腾出内存来供给需要的app, 这套杀进程回收内存的机制就叫 Low Memory Killer。那这个不足怎么来规定呢，那就是内存阈值，我们可以使用cat /sys/module/lowmemorykiller/parameters/minfree来查看某个手机的内存阈值。内存阈值在不同的手机上不一样，一旦低于该值,Android便开始按顺序关闭进程. 因此Android开始结束优先级最低的空进程。android并不是一次性全部都清理掉。进程是有它的优先级的，这个优先级通过进程的adj值来反映，它是linux内核分配给每个系统进程的一个值，代表进程的优先级，进程回收机制就是根据这个优先级来决定是否进行回收，adj值定义在com.android.server.am.ProcessList类中，这个类路径是${android-sdk-path}\sources\android-23\com\android\server\am\ProcessList.java。oom_adj的值越小，进程的优先级越高，普通进程oom_adj值是大于等于0的，而系统进程oom_adj的值是小于0的，我们可以通过cat /proc/进程id/oom_adj可以看到当前进程的adj值。adj值是0就代表这个进程是属于前台进程，我们按下Back键，将应用至于后台，再次查看，adj值变成了8代表这个进程是属于不活跃的进程，你可以尝试其他情况下，oom_adj值是多少，但是每个手机的厂商可能不一样，其实系统在进程回收跟内存回收类似也是有一套严格的策略，可以自己去了解，大概是这个样子的，oom_adj越大，占用物理内存越多会被最先kill掉，OK，那么现在对于进程如何保活这个问题就转化成，如何降低oom_adj的值，以及如何使得我们应用占的内存最少。
 
 ## 2、保活方案
 通用的方法有（1像素Activity，前台服务，账号同步，Jobscheduler,相互唤醒，系统服务捆绑，如果你都了解了，请忽略）经过多方面的验证，Android系统中在没有白名单的情况下做一个任何情况下都不被杀死的应用是基本不可能的。
 ### 2.1 开启一个像素的Activity
 据说这个是手Q的进程保活方案，基本思想，系统一般是不会杀死前台进程的。所以要使得进程常驻，我们只需要在锁屏的时候在本进程开启一个Activity，为了欺骗用户，让这个Activity的大小是1像素，并且透明无切换动画，在开屏幕的时候，把这个Activity关闭掉，所以这个就需要监听系统锁屏广播。
 详细可看<https://www.jianshu.com/p/1da4541b70ad>
 ### 2.2 前台服务
 这种大部分人都了解，据说这个微信也用过的进程保活方案，移步微信Android客户端后台保活经验分享，这方案实际利用了Android前台service的漏洞。
原理如下
对于 API level < 18 ：调用startForeground(ID， new Notification())，发送空的Notification ，图标则不会显示。
对于 API level >= 18：在需要提优先级的service A启动一个InnerService，两个服务同时startForeground，且绑定同样的 ID。Stop 掉InnerService ，这样通知栏图标即被移除。
其实Google察觉到了此漏洞的存在，并逐步进行封堵。这就是为什么这种保活方式分 API >= 18 和 API < 18 两种情况，当某一天 API >= 18 的方案也失效的时候，我们就又要另谋出路了
详细可看<https://www.jianshu.com/p/1da4541b70ad>
也可以用adb shell命令行来查看正在运行的service <http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2016/0418/4158.html>


 ### 2.3、相互唤醒(以下都是杀死后唤起的操作)
  相互唤醒的意思就是，假如你手机里装了支付宝、淘宝、天猫、UC等阿里系的app，那么你打开任意一个阿里系的app后，有可能就顺便把其他阿里系的app给唤醒了。这个完全有可能的。此外，开机，网络切换、拍照、拍视频时候，利用系统产生的广播也能唤醒app，不过Android N已经将这三种广播取消了。
  如果应用想保活，要是QQ，微信愿意救你也行，有多少手机上没有QQ，微信呢？或者像友盟，信鸽这种推送SDK，也存在唤醒app的功能。
拉活方法。（LBE安全大师）

 ### 2.4、JobSheduler
 JobService和JobScheduler是Android5.0（API 21）引入的新API，我们可以通过该机制来拉活我们的Service所在进程。
 JobSheduler是作为进程死后复活的一种手段，native进程方式最大缺点是费电， Native 进程费电的原因是感知主进程是否存活有两种实现方式，在 Native 进程中通过死循环或定时器，轮训判断主进程是否存活，当主进程不存活时进行拉活。其次5.0以上系统不支持。 但是JobSheduler可以替代在Android5.0以上native进程方式，这种方式即使用户强制关闭，也能被拉起来。
 ### 2.4、粘性服务&与系统服务捆绑

这个是系统自带的，onStartCommand方法必须具有一个整形的返回值，这个整形的返回值用来告诉系统在服务启动完毕后，如果被Kill，系统将如何操作，这种方案虽然可以，但是在某些情况or某些定制ROM上可能失效，一种保保守方案。
``` java
public int onStartCommand(Intent intent, int flags, int startId) {
    return START_REDELIVER_INTENT;
} 
``` 
 START_STICKY
 如果系统在onStartCommand返回后被销毁，系统将会重新创建服务并依次调用onCreate和onStartCommand（注意：根据测试Android2.3.3以下版本只会调用  onCreate根本不会调用onStartCommand，Android4.0可以办到），这种相当于服务又重新启动恢复到之前的状态了）。
 START_NOT_STICKY
 如果系统在onStartCommand返回后被销毁，如果返回该值，则在执行完onStartCommand方法后如果Service被杀掉系统将不会重启该服务。
 START_REDELIVER_INTENT
 START_STICKY的兼容版本，不同的是其不保证服务被杀后一定能重启
 参考<https://superxlcr.github.io/2017/04/24/Android%E8%BF%9B%E7%A8%8B%E4%BF%9D%E6%B4%BB%E6%80%BB%E7%BB%93/>
<https://blog.csdn.net/aigestudio/article/details/51348408> 

也可以用远程service来相互唤起 参考 <http://www.52im.net/thread-1135-1-1.html>


听说账号同步唤醒APP这种机制很不错，用户强制停止都杀不起创建一个账号并设置同步器，创建周期同步，系统会自动调用同步器，这样就能激活我们的APP，局限是国产机会修改最短同步周期（魅蓝NOTE2长达30分钟），并且需要联网才能使用。在国内各大ROM"欣欣向荣"的大背景下，关于进程保活，不加入白名单，我也很想知道有没有一个应用永活的方案，这种方案性能好，不费电，或许做不到，或许有牛人可以，但是，通过上面几种措施，在绝大部分的机型下，绝大部分用户手机中，我们的进程寿命确实得到了提高


