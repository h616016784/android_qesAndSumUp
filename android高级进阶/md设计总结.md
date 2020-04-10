# 一、md设计概括
# 二、android提供的md风格组件
## 1、toolbar的使用
  基础参考地址：[Toolbar 开发实践总结](https://www.jianshu.com/p/79604c3ddcae)
  
  基本的title样式都能满足，如果不能的话也可以包裹在其中进行布局
## 2、ConstraintLayout的使用
  基础参考地址：[Android新特性介绍，ConstraintLayout完全解析](https://blog.csdn.net/guolin_blog/article/details/53122387)
  
  此布局为减少布局嵌套很好用。适合零散的多个控件布局
  
# 三、android项目md风格开发设计与总结
## 1、简单搭建md项目
  具体详情不在这里写了，参考地址[官网md的开始](https://material.io/develop/android/docs/getting-started/)
  
  如果是在androidx基础上才能使用，如果不是的话只能一步步来了
## 2、md风格基础风格的搭建
 md的主要由UI regions, surfaces, 和components（基础是layout，color，shape等）
 ### a、主题风格的搭建
 主要参考[goole自定义风格示例](https://github.com/material-components/material-components-android-examples/tree/develop/MaterialThemeBuilder)
 
 还有[md官网的自定义主题风格](https://material.io/design/material-theming/implementing-your-theme.html#)
 
 主题风格由color，Typography，Shape，Icons的基本样式构成。
 
 - color
 由12种颜色样式基本确定：1. Primary 2. Primary Variant 3. Secondary 4. Secondary Variant 5. Background 6. Surface 7. Error 8. On Primary 9. On Secondary 10. On Background 11. On Surface 12. On Error
 
 - Typography
 由13种样式基本确定，由family, font, case, size, and tracking构成：具体得有6种headLine、2种subtitle、2种body、一个button、一个caption、一个overline，利用android得样式化固定到项目种。
 
 - Shape
 由Rounded shapes:分为1. Rounded (0dp), 2. Rounded (4dp), 3. Rounded (16dp), 4. Rounded (24dp)
                  高度通常为 36dp和64dp.
                  
     cards, menus, snackbars, tooltips, dialogs, and buttons通常都是4dp，
 和Cut shapes ：分为1. Cut (0dp), 2. Cut (2dp), 3. Cut (8dp), 3. Cut (12dp)
                高度通常为 36dp和64dp.
  - Icons
  由1. Filled, 2. Sharp, 3. Rounded, 4. Outlined, 5. Two-Tone 五种图形基本构成
 
