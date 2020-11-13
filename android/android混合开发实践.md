
# android混合开发实践
## 一、移动端开发方式
移动应用开发的方式，目前主要有三种：

Native App： 本地应用程序（原生App）

Web App：网页应用程序（移动web）

Hybrid App：混合应用程序（混合App）

这几种方式各有优缺点，现在我们通常会选择混合开发方式。

## 二、混合开发方式的整理
1、H5(HTML5)+原生( Cordova、 Tonic、微信小程序)。

  H5占比为5%，用h5+webview。
  
  H5占比到25%，用H5+混合框架。
  
2、Javascript开发+原生渲染( React Native、Wex、快应用)。

3、自绘U+原生( QT Mobile、 Flutter)。

## 三、混合开发的选择

混合开发的方式的技术方法参考地址：[混合开发-最全常用的混合开发App方式总结](https://www.jianshu.com/p/09b00ebf5e15);

[混合开发 框架对比](https://www.jianshu.com/p/8e99b4aed464)

### 1、H5（webview或网络框架）+原生。
  承相对简单，但对于复杂的界面效果不友好，尤其在弱网下，所以各大厂也推出了相应的优化方案；
  
  常见的方案有：静态直出、离线预推、动态缓存、webview复用、冷热启动多进程、多线程并行、减冗余JS插件、预加载、反射、素材校验、进程冗余启动流程、redex等（以上是鹅厂的vasonic方法）。
  
  但是在实际的应用中各个公司也会有自己的系统实现方案：

- 鹅厂的微信vasonic
  github地址为[vasonic github地址](https://github.com/Tencent/VasSonic)；
  
  相关的资料也可参考[腾讯祭出大招VasSonic，让你的H5页面首屏秒开](https://blog.csdn.net/tencent__open/article/details/77324952);
- QQ的优化方案 
  可以参考资料[70%以上业务由H5开发，手机QQ Hybrid 的架构如何优化演进？](https://my.oschina.net/u/4586970/blog/4401835)
- 支付宝优化方案（蚂蚁金服）
  要使用支付宝方案要申请阿里云，并开通相关服务
  
  平台文档地址为[H5 容器简介](https://tech.antfin.com/docs/2/59192)
  
  原理和使用案例和参考地址[Nebula 来了，支付宝 App 跨平台动态化框架](https://www.mdeditor.tw/pl/27nt)和
  [阿里巴巴技术文章分享：极致的 Hybrid：航旅离线包再加速！](https://www.open-open.com/news/view/1cee25c)
  
- 今日头条优化方案
  参考地址[Android H5秒开方案调研—今日头条H5秒开方案详解](https://yuweiguocn.github.io/android-h5/)
  
- 百度优化方案
  参考地址[百度APP-Android H5首屏优化实践](https://mp.weixin.qq.com/s/AqQgDB-0dUp2ScLkqxbLZg)
  
- 网易优化方案
  参考地址[网易新闻客户端H5秒开优化](https://dy.163.com/article/F0RO3P1005376OPS.html)
  
- 58同城方案
  参考地址[58商家通Android端WebView加载优化方案](https://www.mdeditor.tw/pl/pAC5)
  
对于h5优化秒开的方案思路有很多，参考地址[转载自AlloyTeam](http://www.alloyteam.com/2019/10/h5-performance-optimize/)

以上的方案或多或少的都有一些侵入性，其实我们最想要的是干净的侵入少的带有离线包的方案，对于离线包的理解可以参考。

对于离线包的实现可参考一下地址

[web离线技术原理](https://www.jianshu.com/p/efb4f93b10de);

[前端遇上Go: 静态资源增量更新的新实践](https://zhuanlan.zhihu.com/p/39145666);

[hybrid in webview](https://myslide.cn/slides/17104)


  

  
  




