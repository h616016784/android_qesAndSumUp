8秒定律：web网页如果打开界面或者操作超过8秒，那么就会造成用户的流失，放在app上，别说8秒，3秒用户可能都受不了。

# 一、App的启动流程
可以参考【Android质量与性能优化实践】的app启动流程
## 1、冷启动的流程
![app冷启动流程](https://github.com/h616016784/android_qesAndSumUp/blob/master/vippic/app%E5%86%B7%E5%90%AF%E5%8A%A8.jpg)
如上图，只有在走到第5步的时候，也就是app在加载完布局开始绘制的时候，才会替换空白启动窗口

为何APP启动后会先弄一个空白窗体？？
答：要让用户感觉立刻响应的，而不是没反应

## 2、热启动
热启动中，系统的所有工作就是将您的Activity带到前台。

如果应用的所有Activity都还驻留在内存中，则应用可以无需重复对象初始化、布局扩充和呈现。

## 3、温启动
温启动涵盖在冷启动期间发生的操作的一些子集。

同时，它的开销比热启动多。

总结：不管启动方式有几种，针对冷启动的优化做完，其他启动方式的优化也就完成了。

# 二、黑白屏问题

![app冷启动流程](https://github.com/h616016784/android_qesAndSumUp/blob/master/vippic/App%E9%BB%91%E7%99%BD%E5%B1%8F%E4%BC%98%E5%8C%96.jpg)

方案1:直接将预览页面去掉或者改为透明的；缺点是还是让用感觉到卡顿。

方案2:将背景主题设置为一张图片，设置windowbackground。

方案2的变种：自定义一个主题，在第一个activiy的theme设置成这个主题。第一个activity开始绘制UI的时候再改回原来的theme；缺点是代码不够优雅。

网易云音乐的做法：自定义一个主题，background设置一个布局，布局里面有一个layer-list达到适配效果；第一个界面底部渐入效果只要在第一界面的布局上的image设置ic_lancher。

为何要这么设计？，答：1、屏幕适配容易，设计简洁。

# 三、启动优化

## 1、启动时间的测量
方式1、系统日志的输出（app的启动时间相对比较准确）

在androidstudio的logcat里面，搜索ActivityManager的时候，过滤出来的日志中查看“Displayed 包名/activity名称 时间”，得到启动的时间

方式2、通过ADB的命令方式获取

在terminal中输入 adb shell am start -w 包名/.activity名称 后得到数据：thistime totaltime waittime；有时得到的数据是0，是因为已经启动了，要得到正确的数据，要先杀掉app进程，再输入这个命令，就可以得到数据了。需要注意的是：有的电脑会多显示一个launchmode的方式（冷启动、热启动、温启动）

其中thistime：代表最后一个activity的启动时间；TotalTime： 代表所有Activity的启动时间；WaitTime： 所有时间：ams启动activity总耗时；

方式3、手动获取

在Application的onattach方法里面开始计时，然后在第一个activity的onresume中记录结束时间，在onwindowfocuschange（首帧绘制时间）中也记录结束时间，在treeobserver中也记录结束时间；注意：此方法只能记录app内部总时间，是app启动时间的一大部分。

## 2、优化步骤

    APP启动的流程 MNApplication启动->onCreate()->SplashActivity(onCreate())->MainActivity(onCreate())->MainActivity(onResume())，我们只能在这里面来处理
    
    先来耗时统计
    
        // 方法耗时统计
        Debug.startMethodTracing("Launcher");
        代码。。。。。。。
        Debug.stopMethodTracing();
        此方法会把 代码部分 运行时间的相关信息保存到sdcard中的Launcher.trace文件中，然后把这个文件直接拖到androidstudio中可直接查看
        
    优化相关项：
       - 有些资源懒加载，异步加载；
       - 并不是所有资源都能这么做；资源要用，没初始化完怎么办？
       - 如果你初始化的资源跟UI线程有关，也不能放在子线程；
           1：把资源拆分；
           
    启动优化的两大类：
      - 业务流程优化（视觉欺骗）
            预览窗口优化、业务流程优化（延迟加载、动画效果等）
      - 代码优化 （减少加载时间）
            UI优化、内存优化、图片优化、数据获取/缓存优化、代码量化等
            
    不管什么优化都是（时间、空间）：慢（数据太多），把数据拆分，变小；
     
