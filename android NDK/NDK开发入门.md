
NDK入门
========
自已一直很想学习关于NDK开发方面的知识，现在终于有时间了，自己学习的同时也记录下笔记以待以后查看方便。
主要参考<http://wiki.jikexueyuan.com/project/jni-ndk-developer-guide/envirment.html>
<https://developer.android.com/ndk/guides/concepts?hl=zh-cn>
<https://juejin.im/post/595da4e25188250d8b65ddbf>

入门知识我首先参照官网的顺序学习的 参考<https://developer.android.com/ndk/guides/?hl=zh-cn>
# 1、简介
NDK 产生的背景
Android 平台从诞生起，就已经支持 C、C++ 开发。众所周知，Android 的 SDK 基于 Java 实现，这意味着基于Android SDK 进行开发的第三方应用都必须使用 Java 语言。但这并不等同于“第三方应用只能使用 Java”。在Android SDK 首次发布时，Google 就宣称其虚拟机 Dalvik 支持 JNI 编程方式，也就是第三方应用完全可以通过 JNI 调用自己的 C 动态库，即在 Android 平台上，“Java+C”的编程方式是一直都可以实现的。

不过，Google 也表示，使用原生 SDK 编程相比 Dalvik 虚拟机也有一些劣势，Android SDK 文档里，找不到任何 JNI 方面的帮助。即使第三方应用开发者使用 JNI 完成了自己的 C 动态链接库（so）开发，但是 so 如何和应用程序一起打包成 apk 并发布？这里面也存在技术障碍。比如程序更加复杂，兼容性难以保障，无法访问Framework API，Debug 难度更大等。开发者需要自行斟酌使用。

于是 NDK 就应运而生了。NDK 全称是 Native Development Kit。

NDK 的发布，使“Java+C”的开发方式终于转正，成为官方支持的开发方式。NDK 将是 Android 平台支持 C 开发的开端。

为什么使用 NDK
  代码的保护。由于 apk 的 java 层代码很容易被反编译，而 C/C++ 库反汇难度较大。
  可以方便地使用现存的开源库。大部分现存的开源库都是用 C/C++ 代码编写的。
  提高程序的执行效率。将要求高性能的应用逻辑使用 C 开发，从而提高应用程序的执行效率。
  便于移植。用 C/C++ 写得库可以方便在其他的嵌入式平台上再次使用。
NDK 简介
NDK 是一系列工具的集合

NDK 提供了一系列的工具，帮助开发者快速开发 C（或C++）的动态库，并能自动将 so 和 java 应用一起打包成 apk。这些工具对开发者的帮助是巨大的。

NDK 集成了交叉编译器，并提供了相应的 mk 文件隔离 CPU、平台、ABI 等差异，开发人员只需要简单修改 mk 文件（指出“哪些文件需要编译”、“编译特性要求”等），就可以创建出 so。

NDK 可以自动地将 so 和 Java 应用一起打包，极大地减轻了开发人员的打包工作。

NDK 提供了一份稳定、功能有限的 API 头文件声明

Google 明确声明该 API 是稳定的，在后续所有版本中都稳定支持当前发布的 API。从该版本的 NDK 中看出，这些 API 支持的功能非常有限，包含有：C 标准库（libc）、标准数学库（libm）、压缩库（libz）、Log 库（liblog）。

# 2、基本知识
官网主要提供建立和运行 NDK 所需的信息。
官方文档分别从以下几个方面介绍了 NDK

 - NDK 的基础概念
 - 如何编译 NDK 项目
 - ABI 是什么以及不同 CPU 指令集支持哪些 ABI
 - 如何使用您自己及其他预建的库
 
 ## 2.1、NDK的基本概念
  Android NDK 是一组允许您将 C 或 C++（“原生代码”）嵌入到 Android 应用中的工具。 能够在 Android 应用中使用原生代码对于想执行以下一项或多项操作的开发者特别有用：

  - 在平台之间移植其应用。
  - 重复使用现有库，或者提供其自己的库供重复使用。
  - 在某些情况下提高性能，特别是像游戏这种计算密集型应用。
  
 ### 2.1.1 主要组件
  - ndk-build：ndk-build 脚本用于在 NDK 中心启动构建脚本。这些脚本：自动探测您的开发系统和应用项目文件以确定要构建的内容；生成二进制文件；将二进制文件复制到应用的项目路径。
  如需了解详细信息，请参阅 <https://developer.android.com/ndk/guides/ndk-build?hl=zh-cn>
  
  - Java：Android 构建过程从 Java 来源生成 .dex (Dalvik EXecutable) 文件，这些文件是 Android OS 在 Dalvik 虚拟机（“DVM”）中运行的文件。 即使您的应用根本未包含任何 Java 源代码，构建过程仍会生成原生组件在其中运行的 .dex 可执行文件。
开发 Java 组件时，使用 native 关键字指示以原生代码形式实现的方法。 例如，以下函数声明向编译器告知实现在原生库中：

public native int add(int  x, int  y);

  - 原生共享库：NDK 从原生源代码构建这些库或 .so 文件。
  注意：如果两个库使用相同的签名实现各自的方法，就会发生关联错误。 在 C 语言中，“签名”只表示方法名称。在 C++ 中，“签名”不仅表示方法名称，还表示其参数名称和类型。
  
  - 原生静态库：NDK 也可构建静态库或 .a 文件，您可以关联到其他库。
  
  - Java 原生接口 (JNI)：JNI 是 Java 和 C++ 组件用以互相沟通的接口。 如需了解相关信息，请查阅 Java 原生接口规范。
  
  - 应用二进制界面 (ABI)：ABI 可以非常精确地定义应用的机器代码在运行时如何与系统交互。 NDK 根据这些定义构建 .so 文件。 不同的 ABI 对应不同的架构：NDK 包含对 ARMEABI（默认）、MIPS 和 x86 的 ABI 支持。
  如需了解详细信息，请参阅 ABI 管理<https://developer.android.com/ndk/guides/abis?hl=zh-cn>
  
  - 清单：如果您要编写没有 Java 组件的应用，必须在清单中声明 NativeActivity 类。原生 Activity 和应用在“使用 native_activity.h 接口”下提供了如何执行此操作的详细信息。
  
下面两个项目仅在使用 ndk-build 脚本构建时以及使用 ndk-gdb 脚本调试时才需要。

  - Android.mk：必须在 jni 文件夹内创建 Android.mk 配置文件。 ndk-build 脚本将查看此文件，其中定义了模块及其名称、要编译的源文件、版本标志以及要链接的库。
  - Application.mk：此文件枚举并描述您的应用需要的模块。 这些信息包括：用于针对特定平台进行编译的 ABI;工具链;要包含的标准库（静态和动态 STLport 或默认系统）。
  
  那 CMake 又是什么呢。脱离 Android 开发来看，c/c++ 的编译文件在不同平台是不一样的。Unix 下会使用 makefile 文件编译，Windows 下会使用 project 文件编译。而 CMake 则是一个跨平台的编译工具，它并不会直接编译出对象，而是根据自定义的语言规则（CMakeLists.txt）生成 对应 makefile 或 project 文件，然后再调用底层的编译。
  在Android Studio 2.2 之后，工具中增加了 CMake 的支持，你可以这么认为，在 Android Studio 2.2 之后你有2种选择来编译你写的 c/c++ 代码。一个是 ndk-build + Android.mk + Application.mk 组合，另一个是 CMake + CMakeLists.txt 组合。这2个组合与Android代码和c/c++代码无关，只是不同的构建脚本和构建命令。本篇文章主要会描述后者的组合。（也是Android现在主推的）

 ### 2.1.2 基本流程
 - 设计应用，确定要在 Java 中实现的部分，以及要以原生代码形式实现的部分。
 注：虽然可以完全避免 Java，但您可能发现，Android Java 框架对于包括控制显示和 UI 在内的任务很有用。
 - 像创建任何其他 Android 项目一样创建一个 Android 应用项目。
如果要编写纯原生应用，请在 AndroidManifest.xml 中声明 NativeActivity 类。 如需了解详细信息，请参阅原生 Activity 和应用。
 - 在“JNI”目录中创建一个描述原生库的 Android.mk 文件，包括名称、标志、链接库和要编译的源文件。
 - 或者，也可以创建一个配置目标 ABI、 工具链、发行/调试模式和 STL 的 Application.mk 文件。对于其中任何您未指明的项目，将分别使用以下默认值：
ABI：armeabi
工具链：GCC 4.8
模式：发行
STL：系统
 - 将原生来源置于项目的 jni 目录下。
 - 使用 ndk-build 编译原生（.so、.a）库。
 - 构建 Java 组件，生成可执行 .dex 文件。
 - 将所有内容封装到一个 APK 文件中，包含 .so、.dex 以及应用运行所需的其他文件。
 
 android还提供了帮助程序类 NativeActivity，可用于写入完全原生的 Activity。
 详细可参照<https://developer.android.com/ndk/guides/concepts?hl=zh-cn>
 
 ## 2.2 设置NDK
 ### 2.2.1、安装NDK
  - 下载android studio及ndk <https://developer.android.com/ndk/downloads/?hl=zh-cn>
  - 将您的 PATH 环境变量更新为包含 NDK 的目录的位置。
  (如果用android studio直接下载就没有上面操作，都是默认生成的)
  

  
 ## 2.3、ABI 是什么以及不同 CPU 指令集支持哪些 ABI
 ABI（Application binary interface）应用程序二进制接口。不同的CPU 与指令集的每种组合都有定义的 ABI (应用程序二进制接口)，一段程序只有遵循这个接口规范才能在该 CPU 上运行，所以同样的程序代码为了兼容多个不同的CPU，需要为不同的 ABI 构建不同的库文件。当然对于CPU来说，不同的架构并不意味着一定互不兼容。

 - armeabi设备只兼容armeabi；
 - armeabi-v7a设备兼容armeabi-v7a、armeabi；
 - arm64-v8a设备兼容arm64-v8a、armeabi-v7a、armeabi；
 - X86设备兼容X86、armeabi；
 - X86_64设备兼容X86_64、X86、armeabi；
 - mips64设备兼容mips64、mips；
 - mips只兼容mips；

具体的兼容问题可以参见这篇文章。Android SO文件的兼容和适配
当我们开发 Android 应用的时候，由于 Java 代码运行在虚拟机上，所以我们从来没有关心过这方面的问题。但是当我们开发或者使用原生代码时就需要了解不同 ABI 以及为自己的程序选择接入不同 ABI 的库。（库越多，包越大，所以要有选择）
下面我们来看下一共有哪些 ABI 以及对应的指令集


 ## 2.4、NDK的构建和调试
 
   这里参考<https://www.jianshu.com/p/b4a4cd12d528>来使用 CMake 来构建ndk项目
   
   也可以参考android官网 <https://developer.android.com/studio/projects/add-native-code?hl=zh-cn>
   
   不过我自己学习的ndk-build配置的，更接近底层配置的，参考<https://developer.android.com/ndk/guides/build?hl=zh-cn>
  

