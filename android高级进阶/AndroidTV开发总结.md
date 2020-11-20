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


