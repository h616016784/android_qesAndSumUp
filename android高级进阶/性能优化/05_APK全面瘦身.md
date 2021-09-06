# 一、APK目录：
assets/: 包含了应用的资源，这些资源能够通过AssetManager对象获得。
lib/: 包含了针对处理器层面的被编译的代码。这个目录针对每个平台类型都有一个子目录，比如armeabi, armeabi-v7a, arm64-v8a, x86, x86_64和mips。
res/: 包含了没被编译到resources.arsc的资源。
META-INF/: 包含CERT.SF和CERT.RSA签名文件，也包含了MANIFEST.MF文件。（译注：校验这个APK是否被人改动过）

包含以下文件：
classes.dex: 包含了能被Dalvik/Art虚拟机理解的 dex 文件格式的类。
resources.arsc: 包含了被编译的资源。该文件包含了res/values目录的所有配置的 xml 内容。打包工具将 xml 内容编译成二进制形式并压缩。这些内容包含了语言字符串和styles，还包含了那些内容虽然不直接存储在resources.arsc文件中，但是给定了该内容的路径，比如布局文件和图片。所以又叫 资源映射表
AndroidManifest.xml: 包含了主要的Android配置文件。这个文件列出了应用名称、版本、访问权限、引用的库文件。该文件使用二进制 xml 格式存储。（译注：该文件还能看到应用的minSdkVersion, targetSdkVersion等信息）

# 二、瘦身纬度
## 1、图片优化
## 2、无用资源剔除
## 3、国际化资源配置优化
## 4、动态库打包优化
## 5、代码压缩/代码混淆
## 6、资源压缩/资源混淆

