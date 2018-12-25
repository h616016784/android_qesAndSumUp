适配方案
============
# 1、屏幕适配
  ## 1.1、屏幕适配原理
  android手机会根据不同的手机像素密度（dpi）来加载对应文件的资源（这些资源包括drawable、valuse、layout等）。
  不明白的可上网查询、至于dp dip dpi sp等的区别请网上自行搜索。
  ## 1.2、适配方案
  ### 1.2.1 布局适配
    1、 使用密度无关像素指定尺寸 dp
    2、使用相对布局或线性布局，不要使用绝对布局（但相对布局性能有不好，所以要权衡）
    3、 使用wrap_content、match_parent、权重（与性能权衡）
    4、使用minWidth、minHeight、lines等属性
  为了保证界面的对齐、数据显示完整等等的原因。
    5、在不同文件夹中的适配
      布局别名，res/layout的名称不同（最小宽度限定符）
      dimens使用，res/values的名称不同
      Strings使用，res/values的名称不同
    以上并不是最好的方案，各有优缺点，今日头条的方案网上都说好 <https://mp.weixin.qq.com/s/d9QCoBP6kV9VSWvVldVVwA>
    网上许多代码都是集成字头条的，也要鸿神的方案 <https://blog.csdn.net/lmj623565791/article/details/45460089>
  ### 1.2.2 图片适配
    1、 LOGO 图标
      建议按官方标准准备好各个图标；
    2、普通图片和图标
     一般我们只需xhdpi，特殊的也可以用其他的密度
    3、自动拉伸位图：Nine-Patch的图片类型
    4、动画、自定义view、shape（主要是shape）
    5、ImageView的ScaleType适配
    
# 2、权限适配
  
   
  
