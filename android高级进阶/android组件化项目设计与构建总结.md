
# 一、项目组件化的技术基础以及拆分方案
  目前项目模块化大体可以分为两种模式，分别是submodule和multi-project。根据字面意思，我们就可以很容易理解这两种模式，
 ## 1、submodule模式
项目中只有一个project工程，在project中构建多个module组件，每个module都有自己的git仓库，非常直观，这也是我们最常见的模块化架构。

优点
架构直观，可以让加入开发的新成员比较快速的理解项目的构建
团队协作灵活，在项目开发阶段（特别是起始不稳定的阶段），有更多的module依赖选择，例如直接依赖project，或者通过aar/jar依赖，或者是maven依赖，可以更加快速的进行开发调试

缺点
整个project的git分支会很复杂
团队协作的时候，大家都是在同一个app模块中做测试自己开发的模块，比较容易产生冲突
因为所有的module都在一个project中，每个人都可以修改他人负责的module，不是很安全
## 2、multi-project模式
这种模式是将每个功能模块都拆分成一个单独的功能project，每个功能project都至少包含自己测试使用的app模块和功能模块这两个module。此外，还有一个专门的project用来作为app壳工程，组合所有的功能模块。这样每个project都有自己的单独的git仓库和单独的测试使用的app模块。

优点
每个project有自己单独的git仓库，减少项目的git复杂度
每个project有自己的app模块用于测试，避免影响他人
开发成员只需要负责自己的project，不需要关注其他的功能模块，更加专注
开发成员只能关注自己负责的功能模块，无法修改到其他人负责的功能模块，更加安全
更加解耦

缺点
对于新加入的开发成员不是很友好，不能直观的了解项目的构建
需要在项目开发前达成一些规范协议，否则在协作时容易产生冲突，比如资源文件的命名，如果在两个module中出现命名一样的资源文件，则会报错
因为每个功能都是单独的project，所以开发调式时，只能使用aar/jar或者maven来依赖需要的module，不如submodule模式灵活，在项目初期不稳定时的开发成本要高于submodule模式
会增大项目的体积

项目初期我们通常会选择第一种，下面也是主要说的第一种方式
# 二、业务抽层
## 1、基于第一种方案的业务抽层
基本上都是按照这个思路

# 三、技术要点
## 1、基于第一种方案的技术要点
### a、Library module开发的难点
在把代码抽取到各个单独的Library Module中，会遇到各种问题。最常见的就是R文件问题，Android开发中，各个资源文件都是放在res目录中，在编译过程中，会生成R.java文件。R文件中包含有各个资源文件对应的id，这个id是静态常量，但是在Library Module中，这个id不是静态常量，那么在开发时候就要避开这样的问题。

举个常见的例子，同一个方法处理多个view的点击事件，有时候会使用switch(view.getId())这样的方式，然后用case R.id.btnLogin这样进行判断，这时候就会出现问题，因为id不是经常常量，那么这种方式就用不了。

同样开发时候，用的最多的一个第三方库就是ButterKnife，ButterKnife也是不可以用的，在使用ButterKnife的时候，需要用到注解配置一个id来找到对应view，或者绑定对应的各种事件处理，但是注解中的各个字段的赋值也是需要静态常量，那么就不能够使用ButterKnife了。

解决方案有下面几种：

1.重新一个Gradle插件，生成一个R2.java文件，这个文件中各个id都是静态常量，这样就可以正常使用了。

2.使用Android系统提供的最原始的方式，直接用findViewById以及setOnClickListener方式。

3.设置项目支持Databinding，然后使用Binding中的对象，但是会增加不少方法数，同时Databinding也会有编译问题和学习成本，但是这些也是小问题，个人觉的问题不大。

上面是主流的解决方法，个人推荐的使用优先级为 3 > 2 > 1。

当把个模块分开以后，每个人就可以单独分组对应的模块就行了，不过会有资源冲突问题，个人建议是对各个模块的资源名字添加前缀，比如user模块中的登录界面布局为activity_login.xml，那么可以写成这样us_activity_login.xml。这样就可以避免资源冲突问题。同时Gradle也提供的一个字段resourcePrefix，确保各个资源名字正确，具体用法可以参考官方文档。
### b、Application、全局Context、 Activity管理问题
在功能组件即Demo中的common_base封装BaseApplication，在BaseApplication对第三方库初始化、全局Context的获取等操作。在BaseActivity中对Activity进行添加和移除的管理.

参考地址[组建化项目实践](https://mp.weixin.qq.com/s/8_8gGpkpO2QFNkWgSRBwIg)
### c、组建之间的跳转与通信
- 常用的通信方式
当项目被拆分成多个模块后，模块之间的良好的通信是我们必须考虑的问题。ARouter本身也提供一套通信机制，但是一般很难满足我们所有的需求，所以我们会容易想到的常用的几种通信方式：EvenBus、协议通信、广播或者是将通信的部分下沉到公共组件库。对于这几种方式，在一些大厂的技术文章中都有提到一些他们的看法，下面我简单总结一下：
EventBus： 我们非常熟悉的事件总线型的通信框架，非常灵活，采用注解方式实现，但是难以追溯事件，微信、饿了么认为这是个极大的缺点，不是很推荐，但是美团觉得只要自身控制的好就行（自己设计了一套基于LiveData的简易事件总线通信框架）。
协议通信： 通信双发必须得都知晓协议，且协议需要放在一个公共部分保存。虽然解耦能力强，但是协议一旦变化，通讯双方的同步会变的复杂，不方便。
广播： 安卓的四大组件之一，常见的通信方式，但是相对EventBus来说，过重。
下沉到公共组件库： 这是在模块化中常见的做法，不断的将各种方法、数据模型等公共部分下成到公共组件库，这样一来，公共组件库会变的越来越庞大，越来越中心化，违背了项目模块化的初衷。最后，越来越难以维护，不得不在重新拆分公共组件库。
- 对外暴露接口
解决了通信手段的问题，我们就得考虑另一个问题，为其他模块提供的接口+数据结构我们应该放在哪里？下沉到公共模块吗？或者另外新建一个module用来维护这些接口+数据结构？或者是用arouter直接跳转到接口然后强转返回的对象来使用？

参考地址[关于Android模块化我有一些话不知当讲不当讲](https://www.cnblogs.com/chenergougou/p/6892764.html)

微信的api方案暂时不太理解，可参考[Android项目模块化/组件化开发（非原创）](https://www.cnblogs.com/WUXIAOCHANG/p/11015391.html)
### d、组建的生命周期

在组件化开发时，每个组件都应该有自己独立的生命周期，这个生命周期类似于组件自己的Application，在这个生命周期中，组件可以做一些类库的初始化等工作，否则如果每个组件都将这些工作集中到壳工程的Applicaiton中实现的话，会显得壳工程的Application太过中心化，并且一旦需要修改，会很麻烦，且容易产生冲突。

基于上述原因，我们可以自己搭建一个简易的组件生命周期管理器，主要分为两步：

构建组件的生命周期模型，构建的模型持有整个app的Application引用，同时提供三个基础方法：生命周期创建、生命周期停止以及生命周期的优先级设置。
具体参考地址[Android项目模块化/组件化开发（非原创）](https://www.cnblogs.com/WUXIAOCHANG/p/11015391.html)

组建生命周期的初始化可以参考[多个Module的Application初始化共存问题](https://blog.csdn.net/weixin_43115440/article/details/106057876),如果业务简单，可以参考这种方式来做

如果业务复杂，则最好是引入第三方的工具，可参考这两篇[多模块中application的初始化方式](https://juejin.cn/post/6844904017752145927),[组件化多Module的Application解决方案](https://www.jianshu.com/p/e5b37e62aff6)
### e、模块在调试与发布模式之间的切换
项目开发时，一般提供调试环境与正式发布环境。在不同的环境中，app的有些功能是不需要用到的，或者是有所不同的。另外，在模块化开发时，有些业务模块在调试时，可以作为单独的app运行调试，不必每次都编译所有的模块，极大的加快编译速度，节省时间成本
具体可参考地址[Android项目模块化/组件化开发（非原创）](https://www.cnblogs.com/WUXIAOCHANG/p/11015391.html)
# 四、网络请求架构设计

# 五、项目中组件化开发的管理小技巧
## 1、基于第一种方案的小技巧
 ### a、android studio统一管理版本号
 参考地址[Android Studio Gradle 版本统一管理](https://www.jianshu.com/p/63e08c6eb1c6)
 ### b、项目分包 multiDexEnabled true
 ### c、AndroidManifest的管理
 我们知道APP在打包的时候最后会把所有的AndroidManifest进行合并，所以每个业务组件的Activity只需要在各自模块的AndroidManifest中注册即可。如果业务组件需要独立运行，则需要单独配置一份AndroidManifest，在gradle的sourceSets根据不同的模式加载不同的AndroidManifest文件。
 
 参考地址[组建化项目实践](https://mp.weixin.qq.com/s/8_8gGpkpO2QFNkWgSRBwIg)
 需要注意的是如果在组件开发模式下，组件的Applicaion必须继承自BaseApplicaion
 
 ### d、android的多渠道打包
 具体参考地址[手把手教你AndroidStudio多渠道打包](https://blog.csdn.net/mynameishuangshuai/article/details/51783303)

## 1、android P网络请求，遇到security policy
在Android P系统的设备上，如果应用使用的是非加密的明文流量的http网络请求，则会导致该应用无法进行网络请求，https则不会受影响，同样地，如果应用嵌套了webview，webview也只能使用https请求。
解决方案参考[Android P(9.0) 中 not permitted by network security policy](https://github.com/wenhelinlu/spark/wiki/Android-P(9.0)-%E4%B8%ADOKHttp3%E7%BD%91%E7%BB%9C%E8%AF%B7%E6%B1%82%E5%87%BA%E7%8E%B0java.net.UnknownServiceException:-CLEARTEXT-communication-**-not-permitted-by-network-security-policy-%E5%BC%82%E5%B8%B8%E7%9A%84%E5%8E%9F%E5%9B%A0%E5%8F%8A%E8%A7%A3%E5%86%B3%E6%96%B9%E6%B3%95)
## 2、android 7.0以上增加了fileProvider
在andriod7.0以上权限限制更加高，像代码调用安装apk要进行适配，在application中添加xml路径，并在xml文件中添加xml文件，具体可参考度娘
## 3、android p的读写权限
要在application标签中 添加android:requestLegacyExternalStorage="true"
## 4、检测是否开启状态栏权限
在android8.0以后方法有区别，并且小米魅族等手机也不同，所以跳转到系统设置界面
具体参考[Android:检查通知权限并跳转到通知设置界面](https://www.jianshu.com/p/1e27efb1dcac)
## 5、检测是否开启通知栏
具体参考[Android:检查通知权限并跳转到通知设置界面](https://www.jianshu.com/p/1e27efb1dcac)
