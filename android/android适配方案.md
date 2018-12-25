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
   
  
