---
layout: post
title: "带有简单动画的PageControl"
date: 2015-06-16
comments: true
categories: iOS UIPageControl
tags: [UIPageControl, iOS]
keywords: UIPageControl iOS
publish: true
description: 带有动画的UIPageControl
---

开源一个带有简单动画的PageControl控件，支持Autolayout，地址[GitHub](https://github.com/bawn/LCAnimatedPageControl)。

目前有三种样式可选，包括：

 * LCSquirmPageStyle
 ![image1](/images/LCAnimatedPageControl/demo1.gif)
 * LCScaleColorPageStyle
 ![image2](/images/LCAnimatedPageControl/demo2.gif)
 * LCDepthColorPageStyle
 ![image3](/images/LCAnimatedPageControl/demo3.gif)
 * LCFillColorPageStyle
 ![image3](/images/LCAnimatedPageControl/demo4.gif)


##例子
{% highlight ruby %}

self.pageControl.numberOfPages = 5;指示器的数量
self.pageControl.indicatorMargin = 5.0f;// 指示器之间的间隔，默认是0
self.pageControl.indicatorMultiple = 1.6f;// 指示器的放大倍数，默认是2
self.pageControl.indicatorDiameter = 10.0f;// 指示器的直径
pageControl.pageIndicatorColor = [UIColor grayColor];// 普通状态下的颜色
pageControl.currentPageIndicatorColor = [UIColor redColor];// 当前状态下的颜色
self.pageControl.pageStyle = LCScaleColorPageStyle;// 样式
self.pageControl.sourceScrollView = _collectionView;// 绑定 ScrollView
[self.pageControl prepareShow];// 全部属性设置完后再调用
[self.view addSubview:_pageControl];
{% endhighlight %}

注意，`indicatorMargin`调整的间距是两个指示器都在放大状态下的距离，图示：
![2](/images/LCAnimatedPageControl/indicatorMargin.png)

在 ScaleColorPageStyle 样式下，如果 scrollView 不是滚动到相邻位置的，必须实现以下协议方法，调用`clearIndicators`
{% highlight ruby %}
- (void)scrollViewDidEndScrollingAnimation:(UIScrollView *)scrollView;{
    [self.pageControl clearIndicators];
}
{% endhighlight %}

另外和和原生的`UIPageControl`一样，监听当前显示指示器的位置变化，使用的是`target - action`的形式：

{% highlight ruby %}
[pageControl addTarget:self action:@selector(valueChanged:) forControlEvents:UIControlEventValueChanged];
{% endhighlight %}
