# 一、问题总结
1、OSI模型

2、Http协议概述

3、为什么Okhttp使用socket而不是HttpConnection

4、Okhttp的核心类有哪些？

5、构建者/责任链模式在OkHttp中的使用

6、OkHttp流程

7、OkHttp是如何通过缓存相应数据来减少重复的网络请求

8、Okhttp对于网络请求都有哪些优化

9、Http以及Https原理简单介绍下，加密过程怎么做的

10、怎么设计一个自己的网络访问框架，为什么真么设计？

## 1、OSI模型
http1.0与Http1.1的最大区别：
http1.0的每一次请求都会有3次握手、4次分手。
http1.1在进行一次握手链接后，在一段时间内都不会断开，直到时间结束再断开。
## 2、OKHttp相关内容
### 1）、基本流程
![Okhttp基本流程]( https://github.com/h616016784/android_qesAndSumUp/raw/master/vippic/okhttp基本流程.png )

1：OkHttpClient okHttpClient = new OkHttpClient.Builder()
构建一个okhttpClient对象，传入你想传入的对象，不传就是默认的；

2：构建request对象
Request request = new Request.Builder()  

3：okHttpClient.newCall  实际上返回的realCall类 继续调用RealCall.newRealCall

4：调用enqueue方法，传入我们需要的回调接口，而且会判断，
synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
如果当前这个call对象已经被运行的话，则抛出异常；

5：继续调用dispatcher的enqueue方法，如果当前运行队列<64并且正在运行，访问同一个服务器地址的请求<5
就直接添加到运行队列，并且开始运行；
不然就添加到等待队列；

6：运行AsyncCall，调用它的execute方法

7：在execute方法中处理完response之后，会在finally中调用dispathcer的finished方法；

8：当当前已经处理完毕的call从运行队列中移除掉；并且调用promoteCalls方法

9：promoteCalls方法中进行判断，
如果运行队列数目大于等于64，如果等待队列里啥都没有，也直接return？
循环等待队列，
将等待队列中的数据进行移除，移除是根据运行队列中还能存放多少来决定；
移到了运行队列中，并且开始运行；

## 3、构建者模式/责任链模式
OkHttp中为什么使用构建者模式？

使用多个简单的对象一步一步构建成一个复杂的对象；
优点：当内部数据过于复杂的时候，可以非常方便的构建出我们想要的对象，并且不是所有的参数我们都需要进行传递；
缺点：代码会有冗余

拦截器有：

  a 重试/重定向：
  b 桥拦截器：封装header属性 host keep-live gzip    
     header 进行基本设置，
  c 缓存拦截器
  d 连接拦截器
  e CallServerInterceptor 











