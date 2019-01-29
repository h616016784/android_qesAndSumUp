
gradle使用总结
========
查看了网络上的很多文章，大多都是从基本语法说起，感觉看完了也是云里雾里的，到自己真的上手写的时候还是搞不定。这里从实际应用出发，总结gradle的具体应用。
参考<http://www.jcodecraeer.com/a/anzhuokaifa/Android_Studio/2015/0810/3281.html> 
  <https://juejin.im/entry/57fde5da8ac2470058d1b986>
  
# 1、基础知识
这里知识简单的罗列出需要的基础知识，有时间的可以仔细查看学习，最起码粗略的查看一番，对基本的语法有了解。
# 1.1、Groovy语言
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
 
 # 1.2、android studio中的gradle文件
 ![gradle文件]( https://github.com/h616016784/android_qesAndSumUp/raw/master/pic/gradlefile.png )
 
  标准的 As 项目中, 包含三大部分:
  # 1.2.1、 Top-level Gradle:用于配置所有的 Module 属性
  - buildscript
  简单来说, buildscript 就是配置构建脚本所需要用到的 class 的 path. 其内部会把 configuration closure 传递给 ScriptHandler (你不用关心这是什么) 然后实现设置 repositories 和 dependencies.
一般而言, 这个 script block 都写在开头, 声明这个脚本本身所需要的依赖.
  - allprojects
  这个 configuration closure 里的内容会被包括当前的 project 以及其所有 sub-project 执行. 你在这里配置的 repositories 会在上述地方都生效.

这里的写法意味着, 所有的 project 都会在 jcenter() 中寻找依赖. 除此之外, 还可以指定 flatDir, maven, ivy 等. 可以参考 RepositoryHandler. 
<https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.dsl.RepositoryHandler.html> 

  - task clean
  这是定义了一个新的 task, 叫做 clean. 其类型是 Delete. 实际上, Android Plugin 内置了 clean 方法, 该方法位于 module 中. Module 中内置的 clean 方法只会清理 Module 中的文件并删除 Module 中的 build 目录, 但是工程根目录下的 build 文件是没人清理的, 所以这里定义的 clean 方法即删除项目目录下的 build 文件夹.
  # 1.2.2、 Moudle-level Gradle: 配置独立 Moudle 的属性
  
  # 1.2.3、Gradle Wrapper: 用于统一编译环境, 一般供 CI 使用
 
 
 
