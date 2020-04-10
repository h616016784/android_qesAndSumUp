# 一，本地化你的应用
许多地区的许多设备上都搭载了 Android。为了覆盖大多数用户，您的应用应该以适合其所在语言区域的方式处理文本、音频文件、数字、货币和图形。
使用 Android 资源框架将应用的本地化部分与基于 Java 的核心功能尽可能分开，是一种很好的做法：

- 您可以将应用界面的大部分内容或全部内容放在资源文件中，如本文档和提供资源中所述。
- 另一方面，界面的行为由基于 Java 的代码来驱动。例如，如果用户输入的数据需要根据语言区域使用不同的格式或排序方式，则可以使用 Java 编程语言以编程方式处理数据。本文档不讨论如何本地化基于 Java 的代码。
具体参考官网的[本地化](https://developer.android.com/guide/topics/resources/localization?hl=zh-cn)

## 1、基础原理及应用
### 1、 Android 中的资源切换
资源是指文本字符串、布局、声音、图形和您的 Android 应用需要的任何其他静态数据。应用可以包含多组资源，每组资源针对不同的设备配置进行定制。当用户运行应用时，Android 会自动选择并加载与设备最匹配的资源。
- 创建默认资源
  将应用的默认文本放在 res/values/strings.xml 中。
  
  res/drawable/（必需的目录，包含至少一个图形文件，用作 Google Play 上的应用图标）
  res/layout/（必需的目录，包含定义默认布局的 XML 文件）
  res/anim/（如果您有任何 res/anim-<qualifiers> 文件夹，则为必需）
  res/xml/（如果您有任何 res/xml-<qualifiers> 文件夹，则为必需）
  res/raw/（如果您有任何 res/raw-<qualifiers> 文件夹，则为必需）
 - 创建备用资源
  本地化应用的很大一部分工作是提供不同语言的备用文本。在某些情况下，您还需要提供备用的图形、声音、布局和其他特定于语言区域的资源。
  
  res/drawable/
  包含默认图形。
  res/drawable-small-land-stylus/
  包含经过优化的图形，专用于需要使用触控笔输入且配有横向 QVGA 低密度屏幕的设备。
  res/drawable-ja/
  包含针对日语环境优化的图形。
  
  如果多个资源文件与设备的配置匹配，则 Android 会遵循一组规则来确定使用哪个文件。在可通过资源目录名称指定的限定符中，语言区域几乎总是处于优先地位。
  - 本地化核对清单
  - 管理要进行本地化的字符串
  ### 2、关于本地化的建议
  - 将您的应用设计为无论在什么设备上都能正常运行，并在无法运行时能够安全地退出。
  - 请确保您的应用包含一套完整的默认资源！！
  - 确保包含 res/drawable/ 和 res/values/ 文件夹（文件夹名称中没有任何其他修饰符），这两个文件夹中包含了应用需要的所有图片和文本
  - 设计灵活的布局,如果您需要重新排列布局以适应某种语言（例如单词较长的德语），您可以针对该语言创建备用布局（例如 res/layout-de/main.xml）。不过，这样做会增加应用的维护难度。最好是创建一个更灵活的布局。
  
  ### 3、国际化的支持
  对 Unicode 和国际化支持的讨论分为两个部分：首先是 Android 6.0（API 级别 23）及更低版本，然后是 Android 7.0（API 级别 24）及更高版本。7.0以上支持同时设置多种语言
  ## 2、区域语言的解决方案
  在 Android 7.0 之前，Android 并非始终能成功匹配应用和系统语言区域。
  - 可以是用Translations Editor来辅助开发，具体参考[Translations Editor](https://developer.android.com/studio/write/translations-editor?hl=zh-cn)
  具体的操作步骤可参考地址：[支持不同的语言和文化](https://developer.android.com/training/basics/supporting-devices/languages?hl=zh-cn#java)
  - 也可以通过android studio的插件来辅助开发 [Android-国际化(多语言)切换详解及实例](https://segmentfault.com/a/1190000011583713)
  
  # 二、具体的解决思路与开发过程中遇到的困难
  ## 1、思路
    - 可以随着系统切换语言而切换语言，不支持的语言显示默认
    - 用户可以选择语言，且不会随着系统切换语言或者应用重启而还原
    
    具体可参考[Android国际化(多语言)实现，支持8.0](https://juejin.im/post/5ac8d62c518825557e78a514)
    
    和[Android国际化之多语言(配置及应用内设置)](https://juejin.im/post/5d3c13976fb9a07ee30e60fd)
  ## 2、遇到的困难
  ### 1、适配问题
    参考地址[api的不同](https://www.itcodemonkey.com/article/12098.html)
