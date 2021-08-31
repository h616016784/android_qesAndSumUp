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

# 1、OSI模型
http1.0与Http1.1的最大区别：
http1.0的每一次请求都会有3次握手、4次分手。
http1.1在进行一次握手链接后，在一段时间内都不会断开，直到时间结束再断开。
# 2、OKHttp相关内容
## 1）、基本流程
 ![image](https://github.com/h616016784/android_qesAndSumUp/tree/master/vippic) 











