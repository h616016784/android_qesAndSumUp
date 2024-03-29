
# 一、Kotlin学习
官网语言学习地址 [Kotlin中文官网](https://www.kotlincn.net/docs/tutorials/getting-started.html)

也可以从Android官网进入 [Kotlin](https://developer.android.com/kotlin/add-kotlin#kts)

# 二、Kotlin+Retrofit+ Flow + coroutines + mvvm框架的构建

1、对于Kotlin的Flow与retrofit如何结合参考[Google 推荐在 MVVM 架构中使用 Kotlin Flow](https://juejin.cn/post/6854573211930066951#heading-4),此博客将flow与retrofit结合的基本使用功能罗列出来。flow并没有Rxjava结合好，不过我相信以后会越来越好，其中要注意的是retrofit 2.4.0之前的版本要使用jake的kotlin-coroutines-adapter，在gradle依赖中添加，然后给retrofit添加addCallAdapterFactory(CoroutineCallAdapterFactory())才能很好解析成我们想要的类，要不我们就的手动去解析Response；不过retrofit 2.6.0之后就集成了这个adapter，直接用就可以了。

2、对于Kotlin的Flow与retrofit如何结合也可以简单的参考[Android Architecture: MVVM with Coroutines + Retrofit + Hilt + Kotlin Flow + Room](https://narendrasinhdodiya.medium.com/android-architecture-mvvm-with-coroutines-retrofit-hilt-kotlin-flow-room-48e67ca3b2c8)

3、对于kotlin的coroutine的Context 和 scope的理解
参考[Coroutine Context and Scope](https://elizarov.medium.com/coroutine-context-and-scope-c8b255d59055)

也可以查看考[kotlin中CoroutineScope CoroutineContext的理解](https://blog.csdn.net/cpcpcp123/article/details/113348214)

有一个简单的理解[Kotlin协程教程（2）：协程作用域与各种builder们](https://segmentfault.com/a/1190000020231598)

这一篇比较综合[Kotlin协程作用域（4）](https://blog.csdn.net/baidu_34012226/article/details/100938585)

