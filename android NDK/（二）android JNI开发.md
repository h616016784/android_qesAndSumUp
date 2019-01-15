
JNI开发
=====
# 1、运行一个简单的JNIdemo
 主要参照<http://www.jcodecraeer.com/a/anzhuokaifa/2017/0401/7769.html>  
 <https://juejin.im/entry/5a9d4ad05188255569187646>
网上的步骤有的会报：
Flag android.useDeprecatedNdk is no longer supported and will be removed in the next version of Android Studio. Please switch to a supported build system. 这个错误，主要是android studio3.0以后useDeprecatedNdk废弃了，只能用ndk-build或者cmakelist，这里参考如下地址解决方案：
 <https://juejin.im/post/5a45f032518825390e34820a>
 ## 1.1 环境配置  
  主要需要配置的就是NDK（Native Development Kit），现在Android studio很便利，可以一键下载： file → setting → 选择NDK → 确定应用下载
 ## 1.2 配置gradle
 - 新建一个项目，打开gradle.properties，添加：android.useDeprecatedNdk=true
 - 打开local.properties，添加：ndk.dir=NDK的路径
 - 在原有的jni目录下，添加Android.mk,Application.mk两个文件，内容参照（一 NDK入门）写
 - 最后打开app内build.gradle，在android下面添加ndk配置：   
    ``` gradle
      externalNativeBuild {
          ndkBuild {
              //指定 Android.mk 的路径
              path "src/main/jni/Android.mk"
          }
      } ```
  也可以指定输出ABI，在defaultConfig下面配置
   ```gradle
    ndk {
            abiFilters 'armeabi', 'armeabi-v7a', 'arm64-v8a', 'x86', 'x86_64'
        }
   ```
   
  也可以指定.so的生成路径,在android下，但是没有生效（不止为啥）
   ```gradle
    sourceSets {
        main{
            jni.srcDirs=[] //禁用as自动生成mk
            jniLibs.srcDirs = ['libs'] //这里设置 so 生成的位置
        }
    }
   ```
      
 ## 1.3 新建java访问c层的接口类
 ``` java
   public class JniUtils {
      public static native String getJniString();
    }
   ```
  getJniString()方法即要与C层的交互的函数
## 1.4 生成头文件
  "make-project"编译完成
  打开终端，运行
  cd app/build/intermediates/classes/debug/
    javah com.lilei.testjni.JniUtils

 运行成功之后打开app/build/intermediates/classes/debug/ 即可找到编译出的头文件"com_lilei_testjni_JniUtils.h"，不难发现头文件名是有原报名+类名组成
 
## 1.5 创建jni开发的文件夹
 - 点击app文件夹，New → Folder → JNI Folder, 选择在main文件夹下即可，生成成功后main目录下会出现一个jni的文件夹
 - 找到刚才生成到头文件，复制到jni文件夹下（记得关闭刚才使用的终端，否则无法复制）
 - 头文件有了，现在在jni目录下创建一个C++文件用于开发使用，命名与头文件相同
 - 编写C++文件中定义函数的代码
    ```c++
    #include "com_lilei_testjni_JniUtils.h"
    JNIEXPORT jstring JNICALL Java_com_lilei_testjni_JniUtils_getJniString
            (JNIEnv *env, jclass) {
        // new 一个字符串，返回Hello World
        return env -> NewStringUTF("Hello World");
    }
    ```
## 1.6 java层加载so
  回到JniUtils，加上
  ```java
  static {
    System.loadLibrary("JNISample");
  }
  ```
## 1.7 运行Run
  调用jni的函数JniUtils.getJniString(),如果有输出则调用成功。
  
  其中生成的.so文件在app\build\intermediates\ndkbuild中查看，如果指定.so的路径则在相应路径上查看。
  
以上只是简单的demo，按照上面的步骤可以生成并运行.so动态库，以上只是在android studio上配置ndk-build来运行，已可以直接用java的命令来生成c++文件，并用命令行调用ndk-bulid来生成.so文件，再用android studio来引用，基本步骤如下：
 - 编写声明了 native 方法的 Java 类
 - 将 Java 源代码编译成 class 字节码文件
 - 用 javah -jni 命令生成.h头文件（javah 是 jdk 自带的一个命令，-jni 参数表示将 class 中用native 声明的函数生成 JNI 规则的函数）
 - 用本地代码实现.h头文件中的函数
 - 将本地代码编译成动态库（Windows：\*.dll，linux/unix：\*.so，mac os x：\*.jnilib）
 - 拷贝动态库至 本地库搜索目录下，并运行 Java 程序
android studio2.2后又出了cmake来简化配置，很好用。

# 2、深入理解和学习JNI
## 2.1、同时编译多文件和依赖的第三方库文件
- 同时编译多文件
 参考<https://blog.csdn.net/educast/article/details/12843319> 
 LOCAL_SRC_FILES :=a.cpp b.cpp
 
