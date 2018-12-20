Android常用的技术功能总结
===========
#android保活方案
 >https://www.jianshu.com/p/b5371df6d7cb
 >http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2016/0418/4158.html
 >https://www.jianshu.com/p/1da4541b70ad
 
##1.android进程被杀死的原理
 Android系统把进程的划为了如下几种（重要性从高到低,备注：严格来说是划分了6种）
 
 ###1.1前台进程(Foreground process)
 场景：
 某个进程持有一个正在与用户交互的Activity并且该Activity正处于resume的状态。
 某个进程持有一个Service，并且该Service与用户正在交互的Activity绑定。
 某个进程持有一个Service，并且该Service调用startForeground()方法使之位于前台运行。
 某个进程持有一个Service，并且该Service正在执行它的某个生命周期回调方法，比如onCreate()、 onStart()或onDestroy()。
 某个进程持有一个BroadcastReceiver，并且该BroadcastReceiver正在执行其onReceive()方法。

 用户正在使用的程序，一般系统是不会杀死前台进程的，除非用户强制停止应用或者系统内存不足等极端情况会杀死。
