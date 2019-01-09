
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
