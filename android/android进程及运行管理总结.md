进程及运行管理（综合）
==============
Android应用程序中，任务（Task）是个什么样的概念。我们知道，Activity是Android应用程序的基础组件之一，在应用程序运行时，每一个Activity代表一个用户操作。用户为了完成某个功能而执行的一系列操作就形成了一个Activity序列，这个序列在Android应用程序中就称之为任务，它是从用户体验的角度出发，把一组相关的Activity组织在一起而抽象出来的概念。

# 1、理解binder机制
参考老罗的博客 <https://blog.csdn.net/Luoshengyang/article/details/6618363>,可看老罗的系列文章学习详情。
 在Android系统中，每一个应用程序都是由一些Activity和Service组成的，这些Activity和Service有可能运行在同一个进程中，也有可能运行在不同的进程中。Android系统是基于Linux内核的，而Linux内核继承和兼容了丰富的Unix系统进程间通信（IPC）机制。有传统的管道（Pipe）、信号（Signal）和跟踪（Trace），这三项通信手段只能用于父进程与子进程之间，或者兄弟进程之间；后来又增加了命令管道（Named Pipe），使得进程间通信不再局限于父子进程或者兄弟进程之间；为了更好地支持商业应用中的事务处理，在AT&T的Unix系统V中，又增加了三种称为“System V IPC”的进程间通信机制，分别是报文队列（Message）、共享内存（Share Memory）和信号量（Semaphore）；后来BSD Unix对“System V IPC”机制进行了重要的扩充，提供了一种称为插口（Socket）的进程间通信机制

  但是，Android系统没有采用上述提到的各种进程间通信机制，而是采用Binder机制，难道是因为考虑到了移动设备硬件性能较差、内存较低的特点？不得而知。Binder其实也不是Android提出来的一套新的进程间通信机制，它是基于OpenBinder来实现的。OpenBinder最先是由Be Inc.开发的，接着Palm Inc.也跟着使用。现在OpenBinder的作者Dianne Hackborn就是在Google工作，负责Android平台的开发工作。

 前面一再提到，Binder是一种进程间通信机制，它是一种类似于COM和CORBA分布式组件架构，通俗一点，其实是提供远程过程调用（RPC）功能。从英文字面上意思看，Binder具有粘结剂的意思，那么它把什么东西粘结在一起呢？在Android系统的Binder机制中，由一系统组件组成，分别是Client、Server、Service Manager和Binder驱动程序，其中Client、Server和Service Manager运行在用户空间，Binder驱动程序运行内核空间。Binder就是一种把这四个组件粘合在一起的粘结剂了，其中，核心组件便是Binder驱动程序了，Service Manager提供了辅助管理的功能，Client和Server正是在Binder驱动和Service Manager提供的基础设施上，进行Client-Server之间的通信。Service Manager和Binder驱动已经在Android平台中实现好，开发者只要按照规范实现自己的Client和Server组件就可以了。说起来简单，做起难，对初学者来说，Android系统的Binder机制是最难理解的了，而Binder机制无论从系统开发还是应用开发的角度来看，都是Android系统中最重要的组成，因此，很有必要深入了解Binder的工作方式。要深入了解Binder的工作方式，最好的方式莫过于是阅读Binder相关的源代码了，Linux的鼻祖Linus Torvalds曾经曰过一句名言RTFSC：Read The Fucking Source Code。

ServiceManager存在的意义（具体参照<http://www.cnblogs.com/innost/archive/2011/01/09/1931456.html>）

为何需要一个这样的东西呢？  以MediaPlayerService为例子

原来，Android系统中Service信息都是先add到ServiceManager中，由ServiceManager来集中管理，这样就可以查询当前系统有哪些服务。而且,Android系统中某个服务例如MediaPlayerService的客户端想要和MediaPlayerService通讯的话，必须先向ServiceManager查询MediaPlayerService的信息，然后通过ServiceManager返回的东西再来和MediaPlayerService交互。

毕竟，要是MediaPlayerService身体不好，老是挂掉的话，客户的代码就麻烦了，就不知道后续新生的MediaPlayerService的信息了，所以只能这样：

l         MediaPlayerService向SM注册

l         MediaPlayerClient查询当前注册在SM中的MediaPlayerService的信息

l         根据这个信息，MediaPlayerClient和MediaPlayerService交互

另外，ServiceManager的handle标示是0，所以只要往handle是0的服务发送消息了，最终都会被传递到ServiceManager中去。

三 MediaService的运行

上一节的知识，我们知道了：

l         defaultServiceManager得到了BpServiceManager，然后MediaPlayerService 实例化后，调用BpServiceManager的addService函数

l         这个过程中，是service_manager收到addService的请求，然后把对应信息放到自己保存的一个服务list中

到这儿，我们可看到，service_manager有一个binder_looper函数，专门等着从binder中接收请求。虽然service_manager没有从BnServiceManager中派生，但是它肯定完成了BnServiceManager的功能。

同样，我们创建了MediaPlayerService即BnMediaPlayerService，那它也应该：

l         打开binder设备

l         也搞一个looper循环，然后坐等请求

service，service，这个和网络编程中的监听socket的工作很像嘛！

# 2、activity组件
 在Android系统中，Activity和Service是应用程序的核心组件，它们以松藕合的方式组合在一起构成了一个完整的应用程序，这得益于应用程序框架层提供了一套完整的机制来协助应用程序启动这些Activity和Service，以及提供Binder机制帮助它们相互间进行通信。
 
 Android应用程序架构中非常核心的一点：MainActivity不需要知道SubActivity的存在，即它不直接拥有SubActivity的接口，但是它可以通过一个字符串来告诉应用程序框架层，它要启动的Activity的名称是什么，其它的事情就交给应用程序框架层来做，当然，应用程序框架层会根据这个字符串来找到其对应的Activity，然后把它启动起来。这样，就使得Android应用程序中的Activity藕合性很松散，从而使得Android应用程序的模块性程度很高，并且有利于以后程序的维护和更新，对于大型的客户端软件来说，这一点是非常重要的。
 
 无论是通过点击应用程序图标来启动Activity，还是通过Activity内部调用startActivity接口来启动新的Activity，都要借助于应用程序框架层的ActivityManagerService服务进程。Service也是由ActivityManagerService进程来启动的。在Android应用程序框架层中，ActivityManagerService是一个非常重要的接口，它不但负责启动Activity和Service，还负责管理Activity和Service。
 
 activity的启动过程可以参考<https://blog.csdn.net/Luoshengyang/article/details/6685853>
 ## 2.1、四种启动模式
 基础知识可参考<https://blog.csdn.net/carson_ho/article/details/54669547> 、<https://www.jianshu.com/p/b60d8097e519>
 
   尤其是<https://www.jianshu.com/p/b3a95747ee91>
   ### 2.1.1 简单基本launchmode
本文假定大家对于Activity的Task栈已经有初步了解，首先，看一下Activity常见的四种启动模式及大众理解，这也是面试时最长问的：

standard：标准启动模式（默认启动模式），每次都会启动一个新的activity实例。

singleTop：单独使用使用这种模式时，如果Activity实例位于当前任务栈顶，就重用栈顶实例，而不新建，并回调该实例onNewIntent()方法，否则走新建流程。

singleTask：这种模式启动的Activity只会存在相应的Activity的taskAffinit任务栈中，同一时刻系统中只会存在一个实例，已存在的实例被再次启动时，会重新唤起该实例，并清理当前Task任务栈该实例之上的所有Activity，同时回调onNewIntent()方法。

singleInstance：这种模式启动的Activity独自占用一个Task任务栈，同一时刻系统中只会存在一个实例，已存在的实例被再次启动时，只会唤起原实例，并回调onNewIntent()方法。

需要说明的是：上面的场景仅仅适用于Activity启动Activity，并且采用的都是默认Intent，没有额外添加任何Flag，否则表现就可能跟上面的完全不一致，尤其要注意的是FLAG_ACTIVITY_NEW_TASK的使用。

在非activity启动activity的时候要加FLAG_ACTIVITY_NEW_TASK（Activity对象包含任务栈信息，可以直接在任务栈中启动新的Activity，其他Context对象则不行，不加FLAG_ACTIVITY_NEW_TASK，会直接导致crash。）

关于taskAffinit的详情可参考<https://juejin.im/post/5aef0d215188253dc612991b> 的3.2条目。

## 2.2、app进程
### 2.1、app是否正在运行
主要是通过ActivityManager和PackageManager 提供的api来进行判断。

参考<https://www.cnblogs.com/zyw-205520/p/4746056.html>

### 2.2、app的前后台状态

网上有6种方法，可参考 <https://blog.csdn.net/u011386173/article/details/79095757> ，比较推荐用ActivityLifecycleCallbacks这种方法来实现。
 
### 2.3、第三方app或者推送 打开app


 
