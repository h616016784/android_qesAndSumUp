
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

