
JNI开发
=====
# 1、运行一个简单的JNIdemo，用ndk-build
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
# 2、运行一个JNIdemo，使用cmake
主要参考<https://juejin.im/post/5a30faaf518825293b50501f> ，
<https://developer.android.com/studio/projects/add-native-code?hl=zh-cn> 
这种方法配置相对简单。
当通过CMake来对应用程序增加C/C++的支持时，对于应用程序的开发者，只需要关注以下三个方面：
 - C/C++源文件
 - CMakeList.txt脚本
 - 在模块级别的build.gradle中通过externalNativeBuild/cmake进行配置
## 2.1、创建并运行支持c++库的项目
### 2.1.1、基本步骤
 - 在新建项目之前，我们需要通过SDK Manager安装一些必要的组件：NDK、CMake、LLDB
 - 创建一个全新的工程，这里需要记得勾选include C++ Support选项
 - 在最后一步选择C++的标准，Toolchain Default表示使用默认的CMake配置
 - 与传统的项目相比：
  该模块所对应的build.gradle需要在里面指定CMakeList.txt所在的路径，也就是下面externalNativeBuild对应的选项。
  当新建工程时，它会生成一个native-lib.cpp的事例文件
  app/目录下多了CMakeList.txt文件
 - 运行程序
 
 注：我在创建工程时，遇到error build异常，是因为高版本的cmake（3.10.2）不能运行在android stdio（3.0）上，我删除掉了D:\ProgramFiles\Android\AppData\Local\Android\sdk\cmake这个路径上的一个高版本文件，运行就正常了。
### 2.1.2、原理
- 首先，在构建时，通过build.gradle中path所指定的路径，找到CMakeList.txt，解析其中的内容。
- 按照脚本中的命令，将src/main/cpp/native-lib.cpp编译到共享的对象库中，并将其命名为libnative-lib.so，随后打包到APK中。
- 当应用运行时，首先会执行MainActivity的static代码块的内容，使用System.loadLibrary()加载原生库。
- 在onCreate()函数中，调用原生库的函数得到字符串并展示。
## 2.2、在现有的项目中添加C/C++代码
- 创建的时候，不勾选nclude C++ Support选项
- 定义接口
```java
 public class NativeCalculator {

    private static final String SELF_LIB_NAME = "calculator";

    static {
        System.loadLibrary(SELF_LIB_NAME);
    }

    public native int addition(int a, int b);

    public native int subtraction(int a, int b);
}

```
- 在模块根目录下的src/main/新建一个文件夹cpp，在其中新增一个calculator.cpp文件,文件内容如下：
```
#include <jni.h>
```
- 在模块根目录下新建一个CMakeLists.txt文件：
```
cmake_minimum_required(VERSION 3.4.1)

add_library(calculator SHARED src/main/cpp/calculator.cpp)
```
- 我们需要让Gradle脚本确定CMakeLists.txt所在的位置，我们可以在CMakeLists.txt上点击右键，之后选择Link C++ Project with Gradle
```
    externalNativeBuild {
        cmake {
            path 'CMakeLists.txt'
        }
    }
```
- 实现 C++ ,文件的全部代码如下：
```
#include <jni.h>

extern "C"
JNIEXPORT jint JNICALL
Java_com_hhl_jnicmaketemp_NativeCalculator_addition(JNIEnv *env, jobject instance, jint a, jint b) {
    return a + b;
}

extern "C"
JNIEXPORT jint JNICALL
Java_com_hhl_jnicmaketemp_NativeCalculator_subtraction(JNIEnv *env, jobject instance,  jint a, jint b) {
    return a - b;
}
```
- 调用本地代码
```
Log.d("Calculator", "11 + 12 =" + (mNativeCalculator.addition(11,12)));
        Log.d("Calculator", "11 - 12 =" + (mNativeCalculator.subtraction(11,12)));

```
运行打印输出

## 2.3、cmake的常用语法学习
主要参考<https://juejin.im/post/5b9879976fb9a05d330aa206#heading-13>  
可以参考中文文档<https://www.zybuluo.com/khan-lau/note/254724> 
### 2.3.1、基本操作
 - CMake 编译可执行文件
 在 cpp 的同一目录下创建 CMakeLists.txt 文件，内容如下：
 ```cmake
  # 指定 CMake 使用版本
  cmake_minimum_required(VERSION 3.9)
  # 工程名
  project(HelloCMake)
  # 编译可执行文件
  add_executable(HelloCMake main.cpp )
```
其中，通过 cmake_minimum_required 方法指定 CMake 使用版本，通过 project 指定工程名。
而 add_executable 就是指定最后编译的可执行文件名称和需要编译的 cpp 文件，如果工程很大，有多个 cpp 文件，那么都要把它们添加进来。

 - CMake 编译静态库和动态库
 ```cmake
 cmake_minimum_required(VERSION 3.12)
# 指定编译的库和文件，SHARED 编译动态库
add_library(share_lib SHARED lib.cpp)
# STATIC 编译静态库
# add_library(share_lib STATIC lib.cpp)

 ```
 add_library 指定要编译的库的名称，以及动态库还是静态库，还有要编译的文件。
 
 ## 2.3.2、CMake 基本语法
  参考官网手册<https://www.zybuluo.com/khan-lau/note/254724>
# 3、深入理解和学习JNI
## 3.1、同时编译多文件和依赖的第三方库文件
- 同时编译多文件
 参考<https://blog.csdn.net/educast/article/details/12843319>  
 
 在Android.mk中 LOCAL_SRC_FILES :=a.cpp b.cpp
 
- 引入第三方 so，.a 包(自己暂时没用到)
如果编译有问题有可能是引用的.so文件不是对应的包内的，或者ndk配置ABI时没有指定对cpu架构。
 重新编写 mk 文件(根据实际情况会有所修改)：
 ```mk
   LOCAL_PATH := $(call my-dir)
  #引入第三方 so 
  include $(CLEAR_VARS)
  LOCAL_MODULE    := vvw
  LOCAL_SRC_FILES := libvvw.so
  LOCAL_EXPORT_C_INCLUDES := include
  include $(PREBUILT_SHARED_LIBRARY)


  include $(CLEAR_VARS)
  LOCAL_MODULE    := jniutils
  LOCAL_SRC_FILES := jniutils.cpp
  LOCAL_LDLIBS :=-llog

  #引入第三方编译模块
  LOCAL_SHARED_LIBRARIES := \
  vvw

  include $(BUILD_SHARED_LIBRARY)
 ```
 与前面相比，多了一个第三方模块的引入，如果导入的工程报错，可以试着 APP_ABI 为 x86。
 
## 3.2、androidstudio引用第三方c++动态库
参考<https://blog.csdn.net/xy_kok/article/details/72885943>

交叉编译？？？
先学习这个<https://juejin.im/post/5b631145e51d45162679f09e>
