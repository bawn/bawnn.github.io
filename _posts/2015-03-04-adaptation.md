---
layout: post
title: "关于适配"
date: 2015-04-03
comments: true
categories: iOS
tags: [适配, iOS]
keywords: 适配 iOS iOS适配
description: iOS开发适配屏幕
---

在苹果正式发布iPhone 6和iPhone 6 plus之后，对适配问题一直比较困扰，虽然项目一直用的是Autolayout，看完网上各种适配的文章后更是一团糟。

当时总结了下关于适配的几个点：

1. 添加iPhone 6和iPhone 6 plus的启动图，或者使用xib作为启动视图，避免自动放大的适配
2. 使用Autolayout，当然使用绝对布局并不是完全不可取。
3. 添加@3x的切图，或者使用矢量图（推荐）

需要补充说明的是，使用PDF矢量图时，Xcode会在编译的时候把它处理成三张png图片，分别是`@1x` `@2x` `@3x` 也就说最终的包里面并不是PDF文件，而是这三张图片。

做完这些这不就完事了吗，但后来发现一个问题

![image](/images/Adaptation/gesture.jpg)

“请连接圆点 并记住路径”这个`Label`当初设计的是向上约束为20，这在 `320 x 480` 和 `320 x 568` 分辨率下基本没什么问题，但在iPhone 6和iPhone 6 plus就会显示的整个Label太靠顶部，不够美观。下面的Button也有同样的问题，太靠底部。

基于上面的问题，我跳入了 `Size Classes` 这个坑，为什么这么说。看下图
![image](/images/Adaptation/sizeclasses.jpg)

`Any height`和`Compact width`模式下对应的是iPhone 6 plus以外的设备，`Regular height`和`Compact width`模式下对应是所有iPhone 设备纵向的情况。没有单独的iPhone 6 plus的情况，这样根本解决不了问题。**这里我想说的是，如果的APP只支持iPhone设备，并且只适用于竖屏，就完全没有必要使用`Size Classes`。**

再回头再想想，除了上面提到的三点，还需要解决什么？

#### 其实只有一个问题：特殊界面的处理

先不要急着吐槽，请往下看

回到刚才那个问题，如何解决？

* 方法1：判断不同设备并设置不同的约束值，适用于手写代码
* 方法2：在xib/storyboard中做几套布局，当然只自适用于 xib/storyboard
* 方法3：约束值使用相对于屏幕高度的百分比，适用于手写和 xib/storyboard

重点来说下方法3，手写代码可以轻松做到这点。但在 xib/storyboard 中是没办法设置约束值相对于另一个view的高度的比例。解决办法：请看下图
![image](/images/Adaptation/gesture_20.png)
比如设计师是用iPhone 5/5s来设计的，向上Label向上约束要求是20，那么在 xib/storyboard 上可以在Label上面加一个透明的View设置高度是20。这时候需要做的是

1. 设置Label水平居中，向上相对于View约束为0
2. 设置View，左、右、上相对于父视图为0
3. 设置View的高度相对于父视图也就是self.view的高度为20:568，这里技巧是选择View按住Control键连接到self.view选择 `Equal Heights` ，然后再去详情中设置Multiplier的值
![image](/images/Adaptation/height.png)
4. 最后 `Update Frames` 即可
![image](/images/Adaptation/update.png)

这样一来，不论在什么屏幕上都会有比较好的布局效果。

还有另一种布局方式，把Label作为View的子视图，并上下水平居中，同样的设置View的高度比例，这种方式适用于控件较多的页面。
![image](/images/Adaptation/other.jpg)

不过利用比例布局的方式，可能没有单独判断并设置约束那么完美，但在复杂的界面布局时就较为有优势。目前项目中首页的适配效果：
![image](/images/Adaptation/header.png)
以及其中一个布局的示例
![image](/images/Adaptation/demo.png)

**另外一个例子：**知乎上这个 [问题](http://www.zhihu.com/question/25308946?sort=created) 其中 [刘炜](http://www.zhihu.com/people/imneway) 的回答，关于 profile 页面的适配。手动设置ImageView的大小，也可以使用上述设置比例的方法解决。
