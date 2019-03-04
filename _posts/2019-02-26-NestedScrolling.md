---
layout: post
title: "嵌套滚动"
date: 2019-02-26
comments: true
categories: [iOS]
tags: [嵌套滚动]
keywords: [嵌套滚动]
publish: true
## description: 嵌套滚动
---


本文要讨论的是类似于[即刻](https://www.ruguoapp.com/)、[淘票票](https://www.taopiaopiao.com/)首页，[抖音](https://www.douyin.com/)、[简书](https://www.jianshu.com/)个人主页这样的嵌套滚动效果，事实上网上已经有很多的相关的文章，比如：



- [嵌套UIScrollview的滑动冲突解决方案](https://www.jianshu.com/p/040772693872)

- [iOS 嵌套UIScrollview的滑动冲突另一种解决方案](https://www.jianshu.com/p/df01610b4e73)

- [多层 UIScrollView 嵌套滚动解决方案](https://jiar.me/article/Multi-tier-UIScrollView-nested-scrolling-solution)



而且绝大多数的文章都是从如何解决手势冲突出发给出相应的解决方案，原因是他们大多数都采用了三级 Scrollview 的解决方案，如下图

![image1](http://lc.yardwill.top/NestedScrolling-1.png)

​    

- 蓝色视图：一级 ScrollView

- 红色视图：HeaderView

- 绿色视图：MenuView

- 橘色视图：二级 ScrollView

- 黑色、深黑、浅黑：三级 ScrollView



可以看到三级 ScrollView 和 一级 ScrollView都需要在纵向滚动，所以重点要解决的就是这里的滚动冲突，具体的细节我就不再赘述，大家还可以参考[HGPersonalCenter](https://github.com/ArchLL/HGPersonalCenter)这个项目，里面有详细的注释。



之所以在前面给出了四个例子，是因为淘票票和简书采用的是上面提到的方案，而抖音和即刻两个则不是，并且即刻在体验上更完美，这个后面会讲到。简单粗暴的用越狱手机+[Reveal](https://revealapp.com/)验证下淘票票

![image2](http://lc.yardwill.top/NestedScrolling-2.png)





- 上层的 MVNestTableView：一级 ScrollView

- 中间的 UIScrollView：二级 ScrollView

- 下层的 MVNestTableView：三级 ScrollView



当然通过点击状态栏看也可以粗略判断实现方式，比如淘票票在点击状态栏后视图只会滚动到子 ScrollView 的顶部而不是最外面 ScrollView 的，简书虽然滚动到最外层的顶部但效果明显不够自然，原因就是三级 ScrollView 在纵向没有延伸到顶部。



抖音和即刻在点击状态栏返回到顶部的效果非常自然，所以有理由相信它们在实现上不同于上述方案，那么抖音和即刻的实现方式具体有什么不同？同样的用[Reveal](https://revealapp.com/)看下即刻的视图结构

![image3](http://lc.yardwill.top/NestedScrolling-3.png)



从整体结构上来看即刻只有二级 ScrollView，所以在纵向上 ChildScrollView 会完全接管手势，横向滚动时又由 MainScrollView 控制，这样子带来的好处在于无需关心手势冲突问题，但要实现前面提到的效果还必须处理是以下问题：



- HeaderView 和 MenuView 的位置需要根据 ChildScrollView 的滚动而改变

- 在切换的 Tab 的时候需要同步下一个 ChildScrollView 的 offset

- ChildScrollView 必须在顶部留出 HeaderView 和 MenuView 高度总和的空白区域

- HeaderView 不能拦截滚动手势



在这里就不给出具体的实现细节，文章后面最后有通过两种方案实现的开源库，欢迎 Star。

前面提到的即刻和抖音采用的都是这种二级 ScrollView 的方案，但即刻在体验上更好，比如抖音的个人主页如果手指开始滚动的地方有可交互的控件（Tab栏），那么这时候滑动是会失效的，还有在切换Tab后将视图下拉滚动到顶部然后返回到之前的Tab页，抖音是直接返回到了原始的位置而即刻还是能保留之前进度。



**头部滚动失效解决方案**

即刻为了达到完美的效果，在每个 ChildScrollView 顶部都添加了 HeaderView 和 MenuView，这样子作为一个整体，即使开始触摸的地方有可交互控件也可以上下滚动。然后在左右滑动的时又让ChildScrollView 内的 HeaderView 和 MenuView 隐藏，当停止滚动的时让原本在外层 ScrollView 内的 HeaderView 和 MenuView 显示。



**保留进度解决方案**

关于保留进度首先要做的就是判断当前 ChildScrollView 是不是处于一种特殊状态，这种状态就是 offset.y的值是否大于 HeaderView 的偏移量，然后再通过判断 ChildScrollView 当前的滚动方向，来决定是否要调整 HeaderView 和 MenuView 的位置。



对比两个方案最终的实现各有优缺点

#### 方案一

优点：

- 无障碍配合使用第三方下拉刷新库

- ChildViewController 无需额外设置



缺点：

* 实现较复杂

- 滚动有细微的停顿感

- 切换Tab不能保留进度

- 点击状态栏不能返回到顶部



#### 方案二

优点：

- 实现简单

- 滚动无停顿感

- 切换Tab可保留进度

- 点击状态栏可返回到顶部



缺点

- ChildViewController 需要额外的设置（ChildScrollView 必须在顶部留出 HeaderView 和 MenuView 高度）

- 下拉刷新只能在 ChildViewController 内实现



这里要提的是，由于方案二中 MainScrollView 并不会在纵向有滚动，所以下拉刷新必须放在 ChildViewController 内实现，但又因为 HeaderView 和 MenuView 需要根据 ChildScrollView 的偏移而移动，在配合[MJRefresh](https://github.com/CoderMJLee/MJRefresh)时它们的偏移有明显的Bug（在本文发布前我并没深究解决方案），或许即刻也是因为这个原因而采用上面提到的解决办法。



### 方案一开源库：[Aquaman](https://github.com/bawn/Aquaman)

### 方案二开源库：[Shazam](https://github.com/bawn/Shazam)



上面两个解决方案中的 MenuView 都设计成了交由开发者实现，因为即使集成各种样式的也难满足设计上的千奇百怪的要求，参考我的Demo就能很快实现一个自己想要的效果。