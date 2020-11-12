
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
  
  




