
gradle使用总结
========
查看了网络上的很多文章，大多都是从基本语法说起，感觉看完了也是云里雾里的，到自己真的上手写的时候还是搞不定。这里从实际应用出发，总结gradle的具体应用。
参考<http://www.jcodecraeer.com/a/anzhuokaifa/Android_Studio/2015/0810/3281.html> 
  <https://juejin.im/entry/57fde5da8ac2470058d1b986>
  
# 1、基础知识
这里知识简单的罗列出需要的基础知识，有时间的可以仔细查看学习，最起码粗略的查看一番，对基本的语法有了解。
## 1.1、Groovy语言
Gradle基于Groovy语言，虽然接触Gradle比较久，甚至写过一点Groovy语句，但对语言本身并不了解。为什么用Groovy呢？Groovy运行在JVM上，在Java语言的基础上，借鉴了脚本语言的诸多特性，相比Java代码量更少，Groovy兼容Java，可以使用Groovy和Java混合编程，可以直接使用各种Java类库。
gradle的语法学习推荐如下地址:
<https://www.ibm.com/developerworks/cn/education/java/j-groovy/j-groovy.html> 
<http://www.groovy-lang.org/differences.html>  
了解了基本语法后才会读懂语句比如如下语句：
  - 比如为何在gradle脚本中使用InputStream不用import包，而使用ZipFile需要import包？因为groovy默认import了下面的包和类，无需再import.
   ```groovy
   java.io.*
  java.lang.*
  java.math.BigDecimal
  java.math.BigInteger
  java.net.*
  java.util.*
  groovy.lang.*
  groovy.util.*
   ```
  - 经常看到${var1}的用法是怎么回事？ 这是Groovy中的GString，可以在双引号中直接使用，用于字符串叠加非常方便。
  ```groovy
  def dx = tasks.findByName("dex${variant.name.capitalize()}")
  ```
  - 如下代码
  ```groovy
  //apply是一个方法，plugin是参数，值为'com.android.application'
apply plugin: 'com.android.application'
 
/**
*buildscript,repositories和dependencies本身是方法名。
*后面跟的大括号部分，都是一个闭包，作为方法的参数。
*闭包可以简单的理解为一个代码块或方法指针。
*/
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.2.3'
    }
}
 
//groovy遍历的一种写法 each后面是闭包
android.applicationVariants.each { variant ->
}
  ```
以下是gradle的几个重要的概念，更多的东西还是要自己去看Gradle User Guide
- 生命周期
Gradle构建系统有自己的生命周期，初始化、配置和运行三个阶段。详细参考<https://docs.gradle.org/current/userguide/build_lifecycle.html>   

  初始化阶段，会去读取根工程中setting.gradle中的include信息，决定有哪几个工程加入构建，创建project实例，比如下面有三个工程： include ':app', ':lib1', ':lib2'
  配置阶段，会去执行所有工程的build.gradle脚本，配置project对象，一个对象由多个任务组成，此阶段也会去创建、配置task及相关信息。
  运行阶段，根据gradle命令传递过来的task名称，执行相关依赖任务

- 任务创建
 ```
 task hello {
    doLast {
        println "hello"
    }
}
 ```
 这种写法是在gradle的运行阶段打印出来的，
 ```
 task hello {
    println "hello"
}
 ```
 这种写法是在gradle的配置阶段打印出来的，所以如何写要按照自己的需求来写。
 另外task中有一个action list，task运行时会顺序执行action list中的action，doLast或者doFirst后面跟的闭包就是一个action，doLast是把action插入到list的最后面，而doFirst是把action插入到list的最前面。
 
 - 任务依赖
 当我们在Android工程中执行./gradlew build的时候，会有很多任务运行，因为build任务依赖了很多任务，要先执行依赖任务才能运行当前任务。任务依赖主要使用dependsOn方法，如下所示：
 ```
 task A << {println 'Hello from A'}
task B << {println 'Hello from B'}
task C << {println 'Hello from C'}
B.dependsOn A
C.dependsOn B
 ```
 了解更多，可以看一下侦跃翻译的<https://blog.csdn.net/lzyzsd/article/details/46935405>
 
 - 增量构建
 你在执行gradle命令的时候，是不是经常看到有些任务后面跟着[UP-TO-DATE]，这是怎么回事？

在Gradle中，每一个task都有inputs和outputs，如果在执行一个Task时，如果它的输入和输出与前一次执行时没有发生变化，那么Gradle便会认为该Task是最新的，因此Gradle将不予执行，这就是增量构建的概念。

一个task的inputs和outputs可以是一个或多个文件，可以是文件夹，还可以是project的某个property，甚至可以是某个闭包所定义的条件。自定义task默认每次执行，但通过指定inputs和outputs，可以达到增量构建的效果。
 - 依赖传递
 Gradle默认支持传递性依赖，比如当前工程依赖包A，包A依赖包B，那么当前工程会自动依赖包B。同时，Gradle支持排除和关闭依赖性传递。
 之前引入远程AAR，一般会这样写：
 ```
 compile 'com.somepackage:LIBRARY_NAME:1.0.0@aar'
 ```
 上面的写法会关闭依赖性传递，所以有时候可能就会出问题，为什么呢？本来以为@aar是指定下载的格式，但其实不然，远程仓库文件下载格式应该是由pom文件中packaging属性决定的，@符号的真正作用是Artifact only notation，也就是只下载文件本身，不下载依赖，相当于变相的关闭了依赖传递，可以看一下sf的这个问题，通过添加transitive=true可以解决。但其实如果远程仓库有pom文件存在，compile后面根本不需要加"@aar"，也就不会遇到这个问题了。
 
 ## 1.2、android studio中的gradle文件
 ![gradle文件]( https://github.com/h616016784/android_qesAndSumUp/raw/master/pic/gradlefile.png )
 
  标准的 As 项目中, 包含三大部分:
  ### 1.2.1、 Top-level Gradle:用于配置所有的 Module 属性
  - buildscript
  简单来说, buildscript 就是配置构建脚本所需要用到的 class 的 path. 其内部会把 configuration closure 传递给 ScriptHandler (你不用关心这是什么) 然后实现设置 repositories 和 dependencies.
一般而言, 这个 script block 都写在开头, 声明这个脚本本身所需要的依赖.
  - allprojects
  这个 configuration closure 里的内容会被包括当前的 project 以及其所有 sub-project 执行. 你在这里配置的 repositories 会在上述地方都生效.

这里的写法意味着, 所有的 project 都会在 jcenter() 中寻找依赖. 除此之外, 还可以指定 flatDir, maven, ivy 等. 可以参考 RepositoryHandler. 
<https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.dsl.RepositoryHandler.html> 

  - task clean
  这是定义了一个新的 task, 叫做 clean. 其类型是 Delete. 实际上, Android Plugin 内置了 clean 方法, 该方法位于 module 中. Module 中内置的 clean 方法只会清理 Module 中的文件并删除 Module 中的 build 目录, 但是工程根目录下的 build 文件是没人清理的, 所以这里定义的 clean 方法即删除项目目录下的 build 文件夹.
  ### 1.2.2、 Moudle-level Gradle: 配置独立 Moudle 的属性
  来看一个全的application的gradle
  ``` gradle
  apply plugin: 'com.android.application'

android {
    /**
     * 设置编译 sdk 和编译工具的版本
     */
    compileSdkVersion 24
    buildToolsVersion "24.0.3"

    /**
     * 关于签名, 请参考 google 官方文档: Sign Your App
     */
    signingConfigs {
        /**
         * As 会自动帮我们使用 debug certificate 进行签名. 这个 debug certificate 每次安装 As 都会变,
         * 因此不适合作为发布之用.
         */
        debug {
        }

        /**
         * 由于 Module-level Build Script(本文件) 也要放在 VCS 中管理, 所以不将密码等信息写在这里.
         * 一般的做法是: 在本机设置环境变量, 然后通过下面代码中演示的这种方式读取.
         * 当然, 最佳实践也指导我们将 `gradle.properties` 排除在 VCS 之外,
         * 此时, 也在该文件中将密码设置为变量, 然后在此读取使用.
         */
        release {
            storeFile file("$System.env.STORE_FILE")
            storePassword "$System.env.STORE_PASSWORD"
            keyAlias "$System.env.KEY_ALIAS"
            keyPassword "$System.env.KEY_PASSWORD"
        }
    }

    /**
     * 为所有的 build variants 设置默认的值. 关于 build variant, 我们后面会用一张图片说明
     */
    defaultConfig {
        applicationId "com.walfud.myapplication"
        minSdkVersion 23
        targetSdkVersion 24
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }

    /**
     * type 默认会有 debug 和 release. 不管你写不写都如此.
     * 通常, 我们在 debug 中保留默认值, release 中开启混淆, 并使用私有的签名
     */
    buildTypes {
        debug {
            // 使用默认值
        }

        release {
            // 混淆
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'

            // 签名
            signingConfig signingConfigs.release
        }
    }

    /**
     * flavor 强调的是不同的版本, 比如付费版和免费版.
     * 在国内, 这个字段更多被用于区分不同的渠道, 即 360 渠道, 小米渠道等等.
     */
    productFlavors {
        m360 {}
        xiaomi {}
    }

    /**
     * 这个选项基本不用.
     * 官方说: 使用 splits 可以比使用 flavor 更加有效创建多 apk.
     * 目前而言, 仅支持 Density 和 ABIs 这两个分类.
     */
    splits {
        // 按屏幕尺寸
        density {
            enable true

            // 默认包含全部分辨率, 这里是剔除一些我们不要的
            exclude "ldpi", "mdpi", "xxxhdpi", "400dpi", "560dpi", "tvdpi"
        }

        // 按架构
        abi {
            enable true

            // 使用 `reset()` 后, 我们就相当于不包含任何架构,
            // 这种情况下我们就可以通过 `include` 指定想要使用的架构
            reset()

            include 'x86', 'armeabi-v7a'
            universalApk true       // 是否同时生成一个包含全部 Architecture 的包
        }
    }
}

/**
 * 这个项目的依赖
 */
dependencies {
    /**
     * `fileTree` 导入 libs 目录下的所有 jar 文件
     */
    compile fileTree(dir: 'libs', include: ['*.jar'])

    /**
     * 想导入本地 aar, 首先需要指明本地 aar 的位置, 如下 `repositories` 中所示, 我们把 aar 放在了
     * Module-level 的 libs 目录下. 然后引用这个文件即可.
     */
    compile(name: 'components', ext: 'aar')
}

/**
 * 配置了去哪里查找这个模块依赖文件
 */
repositories {
    flatDir {
        dirs 'libs'
    }
}
  ```
  
  ### 1.2.3、Gradle Wrapper: 用于统一编译环境, 一般供 CI 使用
  ### 1.2.4、android stuido下的gradle
  - gradle-wrapper.properties
  AS 创建的工程, 在根目录下有 gradlew.bat 和 gradlew.sh 文件. 这两个文件会读取 gradle/wrapper/gradle-wrapper.properties 并使用其中指定的 gradle 版本进行编译. 一般而言这套机制用于 CI 服务器中来保障每次编译都在同样的环境下.
  GRADLE_USER_HOME 默认值是你的 USER_HOME/.gradle (win 下是 c:\Users\\.gradle, linux 下是 ~/.gradle ).
对于 gradlew 而言, 如果上述路径没有找到可执行的 gradle 文件, 则会使用 distributionUrl 中所指定的 url 下载后在执行.

  - gradle.properties
  主要作用配置一些与当前机器 gradle 编译相关的属性. 这些属性是每个编译机器根据自己情况决定的(比如, 分配多大内存), 因此不适合放在 git 中.
  
# 2、gradle的使用应用
主要参考<https://gitbook.cn/books/58a7f671ffb1940d7e41cef6/index.html>    
 <https://juejin.im/post/58eae7e5a22b9d0058a88a56>

## 2.1、android签名和秘钥管理
  项目打包需要签名文件，我们可以手动用选择文件方式打包文件，也可以通过gradle配置好后自动打包，用gradle打包具体分为两种
 ### 2.1.1 文件和密码都放在项目中
 简单方便，但是安全性不好。
 具体配置如下：
 在module中的android模块内加入如下代码
 ```
 signingConfigs {
        release {
            keyAlias 'key0'
            keyPassword '------'
            storeFile file('C:/Users/----/Desktop/tempkeystore/jnirundemo.jks')
            storePassword '------'
        }
    }
 ```
 其中storeFile是存放签名文件的路径，次文件可以放在本地也可以放在项目中去，以上是放在本地中，将文件放在项目中要进行如下更改：
 ```
  storeFile file(getProperty("KSTORE"))
 ```
 getProperty是获取文件路径，其参数“kstore”要在gradle.properties文件中配置好签名文件在项目的路径下，代码如下：
 ```
  KSTORE=key/uschool.keystore
 ```
 运行程序即可。
 ### 2.1.2 文件和密码与项目分离
 将密码和签名文件地址等配置到local.properties文件中，当然也可以配置到别的文件中或者别的地方（环境变量中，还有这种操作，牛）
 git设置忽略的文件包括local.properties文件
 在local.properties的文件中配置如下
 ```
  keystore.path=C\:\\Users\\---\\Desktop\\tempkeystore\\jnirundemo.jks
  keystore.password=------
  keystore.alias=key0
  keystore.alias_password=------
 ```
 在module gradle中的android下代码如下：
 ```
     def keystoreFilepath = ''
    def keystorePSW = ''
    def keystoreAlias = ''
    def keystoreAliasPSW = ''
// default keystore file, PLZ config file path in local.properties
    def keyfile = file('s.keystore.temp')

    Properties properties = new Properties()
// local.properties file in the root director
    properties.load(project.rootProject.file('local.properties').newDataInputStream())
    keystoreFilepath = properties.getProperty("keystore.path")

    if (keystoreFilepath) {
        keystorePSW = properties.getProperty("keystore.password")
        keystoreAlias = properties.getProperty("keystore.alias")
        keystoreAliasPSW = properties.getProperty("keystore.alias_password")
        keyfile = file(keystoreFilepath)
    }

    signingConfigs {
        release {
            keyAlias keystoreAlias
            keyPassword keystoreAliasPSW
            storeFile keyfile
            storePassword keystorePSW
        }
    }
 ```
 
 伸展：以上就是基本的方法，可根据自己的实际需要来选取，也可以在项目中存放签名文件，但是密码设置到本地文件中，这个要自己权衡和取舍。我们在打包的时候都会在buildTypes中设置release和debug选项，不同选项签名设置也会不一样
  ```
  debug {
            signingConfig signingConfigs.debug
        }
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
        }
  ```
  这样打包时候会自动找到不同的签名文件。
## 2.2、自动化多渠道快速打包APK
  参考 <http://godcoder.me/2016/01/03/AndroidStudio%E4%BD%BF%E7%94%A8Gradle%E5%A4%9A%E6%B8%A0%E9%81%93%E6%89%93%E5%8C%85/>
  ### 2.2.1、使用android gradle原生打包
  在Android Gradle中，定义了一个叫Build Variant的概念，直译是构建变体，我喜欢叫它为构件-构建的产物(Apk),一个Build Variant=Build Type+Product Flavor，Build Type就是我们构建的类型，比如release和debug，Product Flavor就是我们构建的渠道，比如baidu，google等等，他们加起来就是baiduRelease，baiduDebug，googleRelease，googleDebug，共有这几种组合的构件产出，Product Flavor也就是我们多渠道构建的基础
  已友盟统计为例子 
  - 在manifest中配置如下
  ```
          <meta-data
            android:name="UMENG_CHANNEL"
            android:value="${UMENG_CHANNEL_VALUE}"/>
  ```
  其中${UMENG_CHANNEL_VALUE}中的值就是你在gradle中自定义配置的值。
  - 在build.gradle设置productFlavors
  ```
  productFlavors {
       wandoujia {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "wandoujia"]
       }

       xiaomi{
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "xiaomi"]
       }

       qq {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "qq"]
       }

       _360 {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "360"]
       }

  }
  ```
  其中[UMENG_CHANNEL_VALUE: “wandoujia”]就是对应${UMENG_CHANNEL_VALUE}的值。
  
  - 执行打包 
  我们可以使用CMD命令，进入到项目所在的目录，直接输入命令,当然Android Studio中的下方底栏中有个命令行工具Terminal，你也可以直接打开，输入上面的命令：
   ```
   gradlew assembleRelease

   ```
   就开始打包了，如果渠道很多的话，时间可能会很长。
  或者有针对性的只打特定渠道包，如：
  ```
  gradlew assembleXiaomiRelease
  ```
  ### 2.2.2、优化
  如果我们需要更多的渠道，只要不停的一直添加即可，这是Android Gradle为我们提供的简便的多渠道方案。但是目前中国的App市场，一个App产品很随意的就有上百个渠道，利用官方的这种方式也可以实现，不过有个致命的问题，就是打包时间。一般一个渠道包需要2-3分钟，那么上百个就要好几个小时，这个太慢了，而且一旦有修改重新打包也不能快速的满足，所以我们有了快速打包的想法。
  
  渠道包一般只是渠道不一样，比如google还是baidu，其余的都是一样的，所以没有必要每次都每个渠道都打包，只需要打一个模版包，然后基于这个版本包修改就可以生成不同的渠道包。

因为每个Apk包都是签名好的，所以我们在修改的时候，如果破坏了签名，就需要重新对修改后的APK签名，才可以发布被安装；如果没有破坏签名，就不需要进行重新签名了。两种方式各有千秋，重新签名的打包耗时要长一些，但是安全，不用重新签名的耗时非常短，但是不是很安全，其他人也可以修改你的渠道信息重新发布，不过目前用处不大。

所以目前大部分采用的是不用修改签名的，比如美团，他们的快速打包方式就是采用这种，利用了在Apk的META-INF目录下添加空文件不用重新签名的原理，非常高效，其大概就是：

  1、利用Android Gradle打一个基本包(母包)
  2、然后基于该包拷贝一个，文件名要能区分出来产品、打包时间、版本、渠道等。
  3、然后对拷贝出来的Apk文件进行修改，在其META-INF目录下新增空文件，但是空文件的文件名要有意义，必须包含能区分渠道的名字比如mtchannel_google。
  4、重复2、3生成我们所需的所有的渠道包Apk，这个可以使用Python这类脚本来做
  5.这样就生成了我们所有发布渠道的Apk包了。
  
  那么我们怎么使用呢，原理也非常简单，我们在Apk启动的时候(Application onCreate)的时候,读取我们写Apk中META-INF目录下的前缀为mtchannel_文件，如果找到的话，把文件名取出来，然后就可以拿到渠道标识(google)了,这里贴一个美团实现的代码，大家可以参考一下：
  ```
  public static String getChannel(Context context) {
        ApplicationInfo appinfo = context.getApplicationInfo();
        String sourceDir = appinfo.sourceDir;
        String ret = "";
        ZipFile zipfile = null;
        try {
            zipfile = new ZipFile(sourceDir);
            Enumeration<?> entries = zipfile.entries();
            while (entries.hasMoreElements()) {
                ZipEntry entry = ((ZipEntry) entries.nextElement());
                String entryName = entry.getName();
                if (entryName.startsWith("mtchannel")) {
                    ret = entryName;
                    break;
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (zipfile != null) {
                try {
                    zipfile.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

        String[] split = ret.split("_");
        if (split != null && split.length >= 2) {
            return ret.substring(split[0].length() + 1);

        } else {
            return "";
        }
    }
  ```
  以上代码逻辑我们可以再优化一下，比如为渠道做个缓存放在SharePreference里，不能总从Apk里读取吧，效率是个问题
  对于Python批处理也很简单，这里给出一段美团的Python代码，大家参考补充
  ```python
  import zipfile
  zipped = zipfile.ZipFile(your_apk, 'a', zipfile.ZIP_DEFLATED)
  empty_channel_file = "META-INF/mtchannel_{channel}".format(channel=your_channel)
  zipped.write(your_empty_file, empty_channel_file)
  ```
  以上是核心实现，我们要做的就是保存一个渠道列表，可以用一个文本文件保存，一行一个渠道，然后使用Python读取，for循环生成不同渠道的Apk包，这个我就不写代码了，大家可以自己试一下。

由此我们可以进行扩充，比如通过Gradle的-P传递参数给Python脚本，Python脚本就可以根据我们传递的参数，打部分渠道包，打特定的渠道包等等。

Android Gradle的技巧还有很多，比如自定义生成的Res资源，自定义生成Java常量等等，他的技巧来源于他的灵活性、可扩展性以及和第三方工具软件的默契配合等，善用他，能提高构建效率，节省成本。
## 2.3、自动生成版本信息
## 2.4、批量控制生成的APK文件名
### 2.4.1、简单的区分release版本和debug版本
在module gradle中的android的buildTypes下：
```
 buildTypes {
        debug {
            resValue "string", "app_name", "APPname测试版"
        }
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
```
这样输出的debug App安装后会多出测试版的字样
### 2.4.2、复杂的有应用名、渠道、日期等的apk
在讲批量修改APK文件名之前，我们讲一下关于Android Gradle插件集成的Task任务问题。普通的Java工程比较简单，因为它有一个有限的任务集合，而且它的属性或者方法都是Java Gradle插件添加的，比较固定，而且我们访问任务以及任务里的方法和属性都比较方便，比如classes这个编译Java源代码的任务，我们通过project.tasks.classes就可以访问它，非常快捷。

但是对于Android工程，就不行了，Android工程相对与Java工程来说，要复杂的多，因为它有很多相同的任务，这些任务的名字都是通过Build Types和Product Flavors 生成的，是动态的创建和生成的，而且时机比较靠后，如果你还像原来一样在某个闭包里通过project.tasks获取一个任务，会提示找不到该任务，因为还没有生成。
既然要修改生成的Apk文件名，那么我们就要修改Android Gradle打包的输出，为了解决上面提到的问题（不限于此），android对象为我们提供了2个属性：
- applicationVariants (仅仅适用于Android应用Gradle插件)
- libraryVariants (仅仅适用于Android库Gradle插件)
- testVariants (以上两种Gradle插件都使用)
以上三个属性返回的都是DomainObjectSet对象集合，里面元素分别是ApplicationVariant、LibraryVariant和TestVariant。这三个元素直译来看是变体，通俗的讲他们就是Android构建的产物，比如ApplicationVariant可以代表google渠道的release包，也可以代表dev开发用的debug包，我们上面提到了，他们基于Build Types和Product Flavors生成的产物。
特别注意的是，访问以上这三种集合都会触发创建所有的任务，这意味着访问这些集合后无须重新配置就会产生，也就是说假如我们通过访问这些集合，修改生成Apk的输出文件名，那么就会自动的触发创建所有任务，此时我们修改后的新的Apk文件名就会起作用，达到可我们修改Apk文件名的目的，因为这些是一个集合，包含我们所有生成的产物，所以我们只需要进行迭代，就可以达到我们批量修改Apk文件名的目的。

具体操作如下：
在module的gradle中定义获取时间的方法：
```
def buildTime() {
    def date = new Date()
    def formattedDate = date.format('yyyyMMdd')
    return formattedDate
}
```
如果gradle3.0以下的，在android目录下添加如下代码：
```
applicationVariants.all { variant ->
        variant.outputs.each { output ->
            if (output.outputFile != null && output.outputFile.name.endsWith('.apk')
                    &&'release'.equals(variant.buildType.name)) {
                def flavorName = variant.flavorName.startsWith("_") ? variant.flavorName.substring(1) : variant.flavorName
                def apkFile = new File(
                        output.outputFile.getParent(),
                        "App_${flavorName}_v${variant.versionName}_${buildTime()}.apk")
                output.outputFile = apkFile
            }
        }
    }
```
如果是gradle3.以上的则改为：
```
    applicationVariants.all{ variant ->
        variant.outputs.all {  output ->
            if (output.outputFile != null && output.outputFile.name.endsWith('.apk')
            &&'release'.equals(variant.buildType.name)){
                def flavorName = variant.flavorName.startsWith("_") ? variant.flavorName.substring(1) : variant.flavorName
                outputFileName = "App_${flavorName}_v${variant.versionName}_${buildTime()}.apk"
            }
        }
    }
```

applicationVariants是一个DomainObjectCollection集合，我们可以通过all方法进行遍历，遍历的每一个variant都是一个生成的产物，针对示例，共有googleRelease和googleDebug两个产物，所以遍历的variant共有googleRelease和googleDebug。

applicationVariants中的variant都是ApplicationVariant，通过查看源代码，可以看到它有一个outputs作为它的输出，每一个ApplicationVariant至少有一个输出，也可以有多个，所以这里的outputs属性是一个List集合，我们再遍历它，如果它的名字是以.apk结尾的话那么就是我们要修改的apk名字了，然后我们就可以根据需求，修改成我们想要的名字。

我这里修改的是以项目名_渠道名_v版本名称_构建日期.apk格式生成的文件名，这样通过文件名就可以了解该apk的基本信息，比如什么渠道，什么版本，什么时候构建的等等，最后生成的示例apk名字为App_google_v1.0_20170217.apk，大家可以运行测试一下，注意buildTime这个我们自定义的返回日期格式的方法.




自动瘦身APK文件
善用AndroidManifest占位符
 
 
