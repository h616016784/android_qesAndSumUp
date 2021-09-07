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
  - 资源图片由高清变成一般的（根据需求），PNG>JPG>WEP,这里涉及到UI
  - 单色的背景可用shape来画，也增加了适配性
## 2、无用资源剔除
  一般都不要全部剔除，使用android studio的工具非常方便
 注意！！！！
动态获取方式:
getResources().getIdentifier("name","defType",getPackageName());

  我们可以通过加入白名单的形式，把我们需要保留的文件。方法是搜索unused resources，然后操作
## 3、国际化资源配置优化
  多语言的设置会有很多别的国家的语言，如果不需要，可以去掉
  
  添加如下，可以筛选语言
  android{
    defaultConfig{
        // 只适配英语
        resConfigs 'en'
    }
}

## 4、动态库打包优化
  优化是整个瘦身的重头戏。
  
  首先我们需要知道 so文件是由ndk编译出来的动态库，是 c/c++ 写的，所以不是跨平台的。即每一个平台需要使用对应的so库。

ABI 是应用程序二进制接口简称（Application Binary Interface），定义了二进制文件（尤其是.so文件）如何运行在相应的系统平台上，从使用的指令集，内存对齐到可用的系统函数库。

在Android 系统上，每一个CPU架构对应一个ABI：armeabi，armeabi-v7a，arm64- v8a，x86，x86_64，mips，mips64。

  现在我们一般只需要配置armeabi-v7a即可。
  
  android{
    defaultConfig{
        ndk{
            abiFilters "armeabi-v7a"
        }
    }
}

## 5、代码压缩/代码混淆
将 minifyEnabled 设置为 true 即可 .

但是我们会发现，运行时会报错，因为 minifyEnabled 即压缩了代码，也混淆了代码，所以我们需要处理下混淆.


## 6、资源压缩/资源混淆

资源压缩只与代码压缩协同工作。

buildTypes{
  release{
    shrinkResources true
  }
}

# 三、图片优化之SVG
 ## 1、导入
 SVG(Scalable Vector Graphics)，可缩放矢量图。SVG不会像位图一样因为缩放而让图片质量下降。优点在于可以减小APK的尺寸。常用于简单小图标。缺点是加载的时候会比较耗时
svg是由xml定义的，标准svg根节点为<svg>。
Android中只支持 <vector>，我们可以通过 
vector 将svg的根节点 <svg> 转换为 <vector>。在Android Studio中打开工程，在res目录中点击右键。
## 2、批量转换
  如果有多个svg需要转换为android的vector，则可以通过第三方工具 svg2vector 进行批量转换。
  
  执行转换命令：

java -jar svg2vector-cli-1.0.0.jar -d . -o a -h 20 -w 20 

-d 指定svg文件所在目录
-o 输出android vector图像目录
-h 设置转换后svg的高
-w 设置转换后svg的宽

 ## 3、注意事项
  
不支持的功能举例：

滤镜效果：不支持投影，模糊和颜色矩阵等效果。
  
文本：建议使用其他工具将文本转换为形状。
  
5.0以下不能用
  ## 4、兼容适配
  Android 5.0（API 21）之前的版本不支持矢量图，使用 Vector Asset Studio 有两种方式适配。
  
方式一：生成 png 格式的图片

Vector Asset Studio 可在构建时 针对每种屏幕密度将矢量图转换为不同大小的位图，在 build.gradle 中配置如下： 

SVG 适用于 Gradle 插件1.5 及以上版本；

android{
    defaultConfig{
        // 5.0（API 21）版本以下,将svg图片生成指定维度的png图片
        generatedDensities = ['xhdpi','xxhdpi']
    }
}
  
方式二：支持库

在 build.gradle 中配置如下，适用于 Gradle 插件2.0及以上版本：
android{
    // Gradle Plugin 2.0+
    defaultConfig{
        // 利用支持库中的 VectorDrawableCompat 类,可实现 2.1 版本及更高版本中支持 VectorDrawable
        vectorDrawables.useSupportLibrary = true
    }
}
dependencies {
  // 支持库版本需要是 23.2 或更高版本
  compile 'com.android.support:appcompat-v7:23.2.0'
}

使用矢量图 必须使用 app:srcCompat 属性，而不是 android:src
  
   ## 5、颜色修改
虽然我们前面说了，可以直接在 xml 文件中修改矢量图的颜色，但是并不建议直接修改，我们一般让矢量图为黑色，然后用 Tint 着色器去修改矢量图的颜色。
  
创建两个选择器，然后正常使用即可



