android混合开发
==========
混合开发英文名叫：Hybrid App。
优缺点：
混合开发最大的优点是：节约成本和时间，缩短App开发周期。
最大的缺点我个人认为有两个：一是性能不是很好，二是兼容性比较差。
但随着Android 5.0+的普及以及iOS 9.0+的普及，性能缺陷和兼容性问题都在下降，也就是说如果哪一天Android最低支持版本从5.0开始，iOS最低支持版本从9.0开始了，那么混合开发App的缺点就明显会下降了，而这一天将在2017年末至2018年初到来。

混合开发由简单到繁。
详情可参考<https://www.jianshu.com/p/d2d4f652029d> 和 <https://bxbxbai.github.io/2015/08/16/talk-about-bybird-app/>
# 1、基础的webview开发
 ## 2.1、webview简介
 WebView是一个基于webkit引擎、展现web页面的控件。
 Android的Webview在低版本和高版本采用了不同的webkit版本内核，4.4后直接使用了Chrome。
 ## 2.2 、作用
  - 在 Android 客户端上加载h5页面
  - 在本地 与 h5页面实现交互 & 调用
  - 其他：对 url 请求、页面加载、渲染、对话框 进行额外处理。
## 2.3、具体使用
 Webview的使用主要包括：Webview类 及其 工具类（WebSettings类、WebViewClient类、WebChromeClient类）
 详情参照<https://www.jianshu.com/p/3c94ae673e2a>
 
 ### 2.3.1、webview的常用的方法
  - 加载url
  - WebView的状态（暂时没用到）
  - 关于前进 / 后退网页
  - 清除缓存数据
 ### 2.3.2、WebSettings类的常用的方法
  - 设置WebView缓存
   当加载 html 页面时，WebView会在/data/data/包名目录下生成 database 与 cache 两个文件夹。
   请求的 URL记录保存在 WebViewCache.db，而 URL的内容是保存在 WebViewCache 文件夹下
   是否启用缓存
   结合使用（离线加载）
  注意： 每个 Application 只调用一次 WebSettings.setAppCachePath()，WebSettings.setAppCacheMaxSize()
 ### 2.3.3、WebViewClient类（处理各种通知 & 请求事件）的常见方法
  - shouldOverrideUrlLoading();打开网页时不调用系统浏览器， 而是在本WebView中显示；在网页上的所有加载都经过这个方法。
  - onPageStarted();开始载入页面调用的，我们可以设定一个loading的页面，告诉用户程序在等待网络响应
  - onPageFinished();在页面加载结束时调用。我们可以关闭loading 条，切换程序动作。
  - onLoadResource();在加载页面资源时会调用，每一个资源（比如图片）的加载都会调用一次。
  - onReceivedError();加载页面的服务器出现错误时（如404）调用。
  - onReceivedSslError();webView默认是不处理https请求的，页面显示空白。
 ### 2.3.4、WebChromeClient类的常见用法（辅助 WebView 处理 Javascript 的对话框,网站图标,网站标题等等）：
  - onProgressChanged(); 获得网页的加载进度并显示
  - onReceivedTitle(); 获取Web页中的标题
  - onJsAlert(); 支持javascript的警告框
  - onJsAlert（）;支持javascript的警告框
  - onJsConfirm（）;支持javascript的确认框
  - onJsPrompt（）;支持javascript输入框
 ### 2.3.5 遇到的问题
 -webview加载url跳转到系统浏览器。 重定向问题参考<https://juejin.im/entry/5977598d51882548c0045bde>
 
 # 2、webview与js的交互
 具体可参照 <https://www.jianshu.com/p/345f4d8a5cfa>
 ## 2.1、对于Android调用JS代码的方法有2种：

 ### 2.1.1、通过WebView的loadUrl（）
  // webview只是载体，内容的渲染需要使用webviewChromClient类去实现
  特别注意：JS代码调用一定要在 onPageFinished（） 回调之后才能调用，否则不会调用
 ### 2.1.2、通过WebView的evaluateJavascript（）
  特点：
  - 该方法比第一种方法效率更高、使用更简洁。
  - 因为该方法的执行不会使页面刷新，而第一种方法（loadUrl ）的执行则会。
  - Android 4.4 后才可使用
  
  建议两种方法混合使用
  ## 2.2 JS通过WebView调用 Android 代码
  对于JS调用Android代码的方法有3种：

 ### 2.2.1、通过WebView的addJavascriptInterface（）进行对象映射
 - 优点：使用简单仅将Android对象和JS对象映射即可
 - 缺点：存在严重的漏洞问题 具体参考<https://www.jianshu.com/p/3a345d27cd42>
 
 ### 2.2.2、通过 WebViewClient 的shouldOverrideUrlLoading ()方法回调拦截 url
 具体原理：
 - Android通过 WebViewClient 的回调方法shouldOverrideUrlLoading ()拦截 url
 - 解析该 url 的协议
 - 如果检测到是预先约定好的协议，就调用相应方法
 
 特点
 - 优点：不存在方式1的漏洞；
 - 缺点：JS获取Android方法的返回值复杂。
 ### 2.2.3、通过 WebChromeClient 的onJsAlert()、onJsConfirm()、onJsPrompt（）方法回调拦截JS对话框alert()、confirm()、prompt（）消息
  原理：Android通过 WebChromeClient 的onJsAlert()、onJsConfirm()、onJsPrompt（）方法回调分别拦截JS对话框
（即上述三个方法），得到他们的消息内容，然后解析即可。

 用拦截 JS的输入框（即prompt（）方法）说明 ：
  常用的拦截是：拦截 JS的输入框（即prompt（）方法）
  因为只有prompt（）可以返回任意类型的值，操作最全面方便、更加灵活；而alert（）对话框没有返回值；confirm（）对话框只能返回两种状态（确定 / 取消）两个值

# 3、webview的使用漏洞
 WebView 使用过程中存在许多漏洞，容易造成用户数据泄露等等危险，而很多人往往会忽视这个问题
 WebView中，主要漏洞有3类：任意代码执行漏洞、密码明文存储漏洞、域控制不严格漏洞
 
 具体可参考<https://www.jianshu.com/p/3a345d27cd42>
  
# 4、webview缓存机制
通过 H5缓存机制 + 资源预加载 + 资源拦截的方式 构建了一套WebView缓存机制，从而解决Android WebView的性能问题，最终提高用户使用体验

具体参考 <https://www.jianshu.com/p/5e7075f4875f>

# 5、混合开发框架简介
 参考 <https://kangzubin.com/2018-mobile-end-cross-platform-dev/>
 参考地址：[浅谈几种跨平台方案](https://zhuanlan.zhihu.com/p/148820818)
 参考地址：[Hybrid-Native+H5，主流框架深度分析](https://www.jianshu.com/p/cd44823db35f)
  
  
