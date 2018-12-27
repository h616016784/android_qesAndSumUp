适配方案
============
# 1、屏幕适配
  ## 1.1、屏幕适配原理
  android手机会根据不同的手机像素密度（dpi）来加载对应文件的资源（这些资源包括drawable、valuse、layout等）。
  不明白的可上网查询、至于dp dip dpi sp等的区别请网上自行搜索。
  ## 1.2、适配方案
  ### 1.2.1 布局适配
    1、 使用密度无关像素指定尺寸 dp
    2、使用相对布局或线性布局，不要使用绝对布局（但相对布局性能有不好，所以要权衡）
    3、 使用wrap_content、match_parent、权重（与性能权衡）
    4、使用minWidth、minHeight、lines等属性
  为了保证界面的对齐、数据显示完整等等的原因。
    5、在不同文件夹中的适配
      布局别名，res/layout的名称不同（最小宽度限定符）
      dimens使用，res/values的名称不同
      Strings使用，res/values的名称不同
    以上并不是最好的方案，各有优缺点，今日头条的方案网上都说好 <https://mp.weixin.qq.com/s/d9QCoBP6kV9VSWvVldVVwA>
    
   网上许多代码都是集成字头条的，也要鸿神的方案 <https://blog.csdn.net/lmj623565791/article/details/45460089>
    
  ### 1.2.2 图片适配
    1、 LOGO 图标
      建议按官方标准准备好各个图标；
    2、普通图片和图标
     一般我们只需xhdpi，特殊的也可以用其他的密度
    3、自动拉伸位图：Nine-Patch的图片类型
    4、动画、自定义view、shape（主要是shape）
    5、ImageView的ScaleType适配
    
# 2、权限适配
  Android 安全架构的中心设计点是：在默认情况下任何应用都没有权限执行对其他应用、操作系统或用户有不利影响的任何操作。这包括读取或写入用户的私有数据（例如联系人或电子邮件）、读取或写入其他应用程序的文件、执行网络访问、使设备保持唤醒状态等。

  由于每个 Android 应用都是在进程沙盒中运行，因此应用必须显式共享资源和数据。它们的方法是声明需要哪些权限来获取基本沙盒未提供的额外功能。应用以静态方式声明它们需要的权限，然后 Android 系统提示用户同意。
  应用沙盒不依赖用于开发应用的技术。特别是，Dalvik VM 不是安全边界，任何应用都可运行原生代码。各类应用 — Java、原生和混合 — 以同样的方式放在沙盒中，彼此采用相同程度的安全防护。
  
  从Android6.0（API23）开始，对系统权限做了很大的改变。
  权限等级：
权限主要分为normal、dangerous、signature和signatureOrSystem四个等级，常规情况下我们只需要了解前两种，即正常权限和危险权限
  a、正常权限

正常权限涵盖应用需要访问其沙盒外部数据或资源，但对用户隐私或其他应用操作风险很小的区域。应用声明其需要正常权限，系统会自动授予该权限。例如设置时区，只要应用声明过权限，系统就直接授予应用此权限。

  b、危险权限

危险权限涵盖应用需要涉及用户隐私信息的数据或资源，或者可能对用户存储的数据或其他应用的操作产生影响的区域。例如读取用户联系人，在6.0以上系统中，需要在运行时明确向用户申请权限。

  ## 2.1、关于android6.0、7.0/8.0的权限以下博客（很全了） 
  <https://www.cnblogs.com/JLZT1223/p/8108783.html>
  
  这个网址更牛b<https://www.jianshu.com/p/a8fd3d1fa0a3>
  

  ## 2.2、解决方案
   一种解决方案是用google提供的EasyPermissions框架,参考如下地址
   <https://segmentfault.com/a/1190000012247350>
   如果特定机型权限不能找到要try一下进行相应处理
   
  ### 2.2.1、关于权限遇到的问题
  跳转安装界面崩溃、调用系统照相和裁剪、选择图片等都会遇到问题，参照以下方案
   <https://blog.csdn.net/sinat_14849739/article/details/65758033>或者
        <https://www.cnblogs.com/vijozsoft/p/9304096.html>、<http://yifeng.studio/2017/05/03/android-7-0-compat-fileprovider/>  
       <https://juejin.im/post/5974ca356fb9a06bba4746bc>
# 3、状态栏适配  
关于沉浸式大概可以分成三个阶段：

Android4.4（API 19） - Android 5.0（API 21）： 这个阶段可以实现沉浸式，但是表现得还不是很好，实现方式为: 通过FLAG_TRANSLUCENT_STATUS设置状态栏为透明并且为全屏模式，然后通过添加一个与StatusBar 一样大小的View，将View 的 background 设置为我们想要的颜色，从而来实现沉浸式。

Android 5.0（API 21）以上版本： 在Android 5.0的时候，加入了一个重要的属性和方法 android:statusBarColor （对应方法为 setStatusBarColor），通过这个方法我们就可以轻松实现沉浸式。也就是说，从Android5.0开始，系统才真正的支持沉浸式。

Android 6.0（API 23）以上版本：其实Android6.0以上的实现方式和Android 5.0 +是一样，为什么要将它归为一个单独重要的阶段呢？是因为从Android 6.0（API 23）开始，我们可以改状态栏的绘制模式，可以显示白色或浅黑色的内容和图标（除了魅族手机，魅族自家有做源码更改，6.0以下就能实现）

## 3.2、解决方案
 可参照 <https://juejin.im/post/5989ded56fb9a03c3b6c8bde> 或者
 <https://www.jianshu.com/p/47f34bee13a9>
    
    
  
  
   
  
