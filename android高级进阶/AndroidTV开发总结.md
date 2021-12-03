# Android TV设计以及开发总结
为什么说要设计呢，因为UI的设计真的是随心所欲，开发也要懂一点设计！！
## 一、TV交互设计
  与使用手机或平板电脑相比，用户在看电视时有不同的期望。典型的电视用户坐在离屏幕约 10 英尺的位置，所以小细节没那么引起注意，小文本难以阅读。由于用户离电视较远，因此必须使用一种遥控设备来导航和进行选择，而不是轻触屏幕上的元素。这些差异大大影响了提供出色的电视用户体验的要求。
### 1、视觉设计和用户交互准则质量标准
  这个标准是google提出的规范用户设计的，但很多设计并没有遵循，也很少遵循

参考地址[电视应用质量](https://developer.android.google.cn/docs/quality-guidelines/tv-app-quality)里面的视觉设计和用户交互准则。

### 2、Android设计指南
  可以参考google的设计原则，但是你懂得。。。。
  
  参考地址[ Android TV 设计指南](https://designguidelines.withgoogle.com/android-tv/tv-apps/app-structure.html)
##### A、设计原则：
  - 创造性的想象力（Creative vision）
  
    Big and beautiful content is the center of the TV interface. Content appears cinematic, with elegant transitions, and minimal text. The TV screen favors low-density, curated, actionable content, and categories.（大而漂亮的内容是电视界面的中心。内容看起来像电影，有优雅的过渡和最小的文本。电视屏幕更倾向于低密度、精心策划、可操作的内容和分类）
    
    提炼：大而美、过渡优雅、低密度、精心策划、可操作的内容和分类。
  - 休闲消费（Casual consumption）
  
    People often use TV in a relaxed mindset. Casual consumption is the primary use case of Android TV.

While searching for content, every part of the experience should be simple. New content should be easy to discover. There should be a reduced number of choices to make on content and settings screens. The default action should be one click away.（人们经常在放松的状态下看电视。休闲消费是Android电视的主要用例。在搜索内容时，体验的每一部分都应该是简单的。新内容应该容易发现。应该减少在内容和设置屏幕上的选择。默认操作应该是单击一键即可完成。）

  - Cinematic experience（电影的经验）
  
    Tell your story with pictures and sound. The large screen affords the opportunity for large, high-resolution graphics and visual elements that build a rich and dynamic experience.

We celebrate cinematic motions that are amplified on the large screen. During transitions and feedback operations, use movement to bring actions to life.（用图片和声音讲述你的故事。大屏幕为高分辨率的大图形和视觉元素提供了机会，从而构建了丰富而动态的体验。我们庆祝在大屏幕上被放大的电影动作。在过渡和反馈操作中，使用移动使动作栩栩如生。）

 - Lightweight interaction（轻量级的交互）
 
   Android TV is simple and magical. It’s all about finding and enjoying content and apps with the least amount of friction. Minimize the number of navigation steps required to perform actions. Build apps with the fewest screens possible between app entry and content immersion. Avoid making users enter text whenever possible, and use voice interfaces when you require text input.（安卓电视简单而神奇。这一切都是关于寻找和享受与最少的摩擦的内容和应用程序。尽量减少执行操作所需的导航步骤。在应用程序入口和内容沉浸之间尽可能减少屏幕。尽可能避免让用户输入文本，并在需要文本输入时使用语音界面。）

 - Shared context（）
 
 TV is embedded in social experiences, like watching movies, looking at photos, or playing games. This makes a TV an inherently shared device. Because anyone in a trusted household might have access to it, app content should be appropriate for mixed audiences.（电视根植于社会体验中，比如看电影、看照片或玩游戏。这使得电视成为一种固有的共享设备。因为一个受信任的家庭中的任何人都可以访问它，应用程序内容应该适合混合受众。）

##### B、TV的设计：（实际是设计原则里面的）
 - TV appropriate apps（电视适当的应用程序）
 
 最适合电视的应用程序提供沉浸式的娱乐体验。提供学习、游戏、交流和内容消费的应用程序都是很好的例子，但这并不是一个全面的列表。
 
 电视应用程序应该:                               电视应用程序不应该:

    涉及简单的设置                                  是文字

    利用有限的屏幕                                  强调管理任务

    分享社会背景

    把一些关键的事情做得很好
  
  - 简单的导航

  电视应该提供最短的内容路径。

  在观众和内容之间放置尽可能少的屏幕。当屏幕是必要的，他们是一致的和简单的操作输入设备。
  
  - 信息密度
  
  由于看电视的距离，用户在电视上处理的信息可能不如在电脑或移动设备上处理的那么多。限制电视屏幕上的文字和阅读量。
  
##### C、系统的概述：
  - Navigation
  
  操作Android电视应该很容易。为安卓电视设计应用程序需要重新考虑用户输入方式，因为用户使用遥控器而不是触摸屏来浏览应用程序。
  
  Controllers（控制器）：Android电视控制器可以多种多样，许多控制器包括方向键(D-pad)、选择键、播放/暂停键、返回键和麦克风键。
  
  Focus（焦点）：总是有一个焦点。文本、按钮、卡片和其他一些元素可以被聚焦。清楚地指示哪个控件处于焦点或被选中。
  
  Structure（）：路径---用户应该能够以清晰的方向导航你的UI。如果没有到达控件的直接路径，可以考虑重新定位它；轴---设计你的布局，让它利用水平和垂直轴。给每个方向一个特定的功能，使其快速导航大层次结构。
##### D、TV apps的设计：
  Android电视应用程序的结构一般有以下四种:
 - Browse View（浏览视图）
 
  浏览视图通常是Android TV应用程序的入口点。

  通过使用相关类别简化决策，使用户可以轻松浏览内容矩阵
  
  See [Browse View](https://designguidelines.withgoogle.com/android-tv/tv-apps/browse-view.html#browse-view-browse-lane) for more details.

 - Detail View（详细信息视图）
 
 Detail视图为所选内容提供深入的相关信息。

 将可操作的内容和最相关的信息放在屏幕最直接可见的部分，而不需要用户滚动屏幕。
 
 See [Detail View](https://designguidelines.withgoogle.com/android-tv/tv-apps/detail-view.html) for more details.
 
 - Consumption View(消费的观点)
 
 消费视图是供用户参与或观看内容的。

 提供控制和上下文信息来增强体验
 
 See [Consumption View](https://designguidelines.withgoogle.com/android-tv/tv-apps/consumption-view.html#) for more details.
 
 - In-app Search(应用内搜索)
 
 应用内搜索覆盖提供了一种快速的方法来搜索应用内的内容。

 要进行搜索，请选择屏幕上的搜索按钮或按下控制器上的mic按钮。
 
 See [In-app Search](https://designguidelines.withgoogle.com/android-tv/tv-apps/in-app-search.html) for more details.
 
##### E、Style的设计：

  - Color
  
  Testing colors：跨设备测试颜色。电视上的颜色看起来可能与电脑或移动设备上的颜色非常不同，有些颜色的组合可能不能同时在两者上工作。为了确保你的应用程序的颜色适用于电视，在多种设备和环境中测试你的设计。也测试图像质量使用电视的显示设置(如黑白，信箱等)。推荐使用比移动设备颜色深两到三个层次的颜色。或者，使用调色板中700-900范围的颜色。
  
  TV White：纯白色(#FFFFFF)在明亮的电视屏幕上很刺眼。浅灰色(#EEEEEE)推荐作为深色背景的默认文本颜色。
  
  - Typography（排版）
  
  Style：以下字体样式和大小通常在电视用户界面中使用.

      Content title:Light 34sp   Content title:Light 24sp   Browse lane title:Condensed 20sp    Subhead Regular:16sp   Button Label Condensed: 16sp

      Body Regular:14sp    List item/card primary text:Condensed Regular 14sp    Secondary text:Condensed Regular: 12sp
  
  Dynamic font size（动态字体大小）： 内容标题的文本长度可以从短到长。考虑使用动态类型根据可用空间和估计的字母大小来改变文本大小。
  
  Condensed fonts（浓缩的字体）：对于英语和类似英语的脚本，使用压缩字体来有效地容纳有限空间的文本。
  
  - Branding（品牌）
  
  在安卓电视上有多种方式来表达你的品牌。应用程序可以在整个UI中使用一致的配色方案，例如推荐卡的重音着色、浏览通道和传输控件。应用程序中的背景图像也是一种身临其境的方式来表达你的品牌。

  ## 二、TV应用的构建
  注意：TV 应用在 TV 设备上本地运行。如需进一步了解如何将视频和音频从 Android 应用流式传输到 TV 设备，请参阅 Google Cast 开发者文档。
  
  ### 1、TV 应用使用入门
    TV 应用使用的结构与手机和平板电脑应用相同。这种相似性意味着，您可以将现有应用改造成在 TV 设备上也能运行，也可以运用您已有的 Android 应用构建知识打造新应用。
    
    主要参考[TV 应用使用入门](https://developer.android.com/training/tv/start/start)
  
  #### A、确定媒体格式支持
  
    如需了解 Android TV 支持的编解码器、协议和格式，请参阅以下文档。

   - [支持的媒体格式](https://developer.android.com/guide/topics/media/media-formats)
   - DRM
   - android.drm
   - ExoPlayer
   - android.media.MediaPlayer
   
 #### B、设置 TV 项目
 注意：我们建议您让一个应用同时支持移动设备和 TV 设备。如果您需要为移动设备和 TV 设备分别构建单独的应用，那么可以使用多 APK 支持在 Google Play 上的同一商品详情下发布多个应用
 下面是创建在 TV 设备上运行的应用时应使用的主要组件：

    TV Activity（必需）- 在您的应用清单中，声明想要在 TV 设备上运行的 Activity。
    TV 库（可选）- 有几个可用于 TV 设备的 androidx 库，它们提供了用于构建界面的微件。
  - 前提条件：
    将您的 SDK 工具更新为 24.0.0 或更高版本。

    将您的 SDK 平台更新为 Android 5.0 (API 21) 或更高版本。

    创建或更新您的应用项目
  
  - 声明 TV Activity
  
  - 声明 Leanback 支持
  
  - 将触摸屏声明为非必备条件
  
  - 提供主屏幕横幅
  
  - 更改启动器颜色
  
  #### C、添加 TV 库
  Jetpack 包含用于 TV 应用的 androidx 软件包库。这些库为 TV 设备提供了 API 和界面微件。
  
  #### D、运行 TV 应用
  
  运行您的应用是开发过程的一个重要环节。您可以在配置为支持 USB 调试的 TV 设备上运行您的应用，也可以使用虚拟 TV 设备运行您的应用
  
  - 在真实设备上运行
  
  - 在虚拟设备上运行
  
  ### 2、AndroidX TV libraries
  
  主要是Leanback,主要使用参考[AndroidX TV libraries](https://developer.android.com/training/tv/start/libraries)
  
  ### 3、处理 TV 硬件
  
  TV 硬件与其他 Android 设备截然不同。TV 不提供其他 Android 设备上提供的一些硬件功能，如触摸屏、相机和 GPS 接收器。此外，TV 还完全依赖于辅助硬件设备。用户必须使用遥控器或游戏手柄才能与 TV 应用进行交互。构建 TV 应用时，您必须仔细考虑在 TV 硬件上运行应用的硬件限制和要求。
  
  主要参考[处理 TV 硬件](https://developer.android.com/training/tv/start/hardware)
  
  ### 4、管理 TV 控制器
  
  TV 设备需要通过基本遥控器或游戏控制器形式的辅助硬件设备与应用进行交互。这意味着您的应用必须支持方向键输入。此外，这还意味着您的应用可能需要处理离线控制器以及来自多种控制器的输入
  
  主要参考[管理 TV 控制器](https://developer.android.com/training/tv/start/controllers)
 
 ### 5、构建 TV 布局
 
 TV 屏幕的通常观看距离约为 10 英尺，尽管其尺寸比大多数其他 Android 设备的显示屏大得多，但这种屏幕不能提供与小型设备相同程度的精确细节和色彩表现。鉴于这些因素，您在设计应用布局时必须谨记它是用于 TV 设备这一点，才能创造出实用并且令人愉悦的用户体验。
 
  主要参考[构建 TV 布局](https://developer.android.com/training/tv/start/layouts)
  
 ### 6、屏幕键盘
 
 可用系统键盘或者自己写键盘

  主要参考[屏幕键盘](https://developer.android.com/training/tv/start/onscreen-keyboard)
  
 ### 7、创建 TV 导航
TV 设备为应用提供了一组有限的导航控件。能否为您的 TV 应用创建有效的导航架构取决于对这些有限控件的了解，以及用户在操作您的应用时的感知极限。在构建 Android TV 应用时，请特别注意用户实际如何使用遥控器按钮（而非触摸屏）在您的应用内导航。

  主要参考[创建 TV 导航](https://developer.android.com/training/tv/start/navigation)
  
 ### 8、谷歌TV上的最佳实践
 
  主要参考[谷歌TV上的最佳实践](https://developer.android.com/training/tv/start/google-tv)
 #### A、Baseline requirements（基线需求）
 
 ## 三、构建 TV 播放应用
 
  ## 四、TV 应用之实践
  ### 1、leanback使用与理解
  
  基础使用可参考[AndroidTv Home界面实现原理（一）——Leanback 库的使用](https://cloud.tencent.com/developer/article/1128174?from=article.detail.1564402)
              [AndroidTv Home界面实现原理（二）——Leanback 库的主页卡位缩放动画源码解析](https://cloud.tencent.com/developer/article/1128153?from=article.detail.1128174)
              
              [聊一聊 Leanback 中的 HorizontalGridView](http://events.jianshu.io/p/52f67a414a10)
  
  
  
