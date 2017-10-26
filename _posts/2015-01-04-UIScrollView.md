---
layout: post
title: "UIScrollView相对布局"
date: 2015-01-04
comments: true
categories: iOS
tags: [iOS]
keywords: UIScrollView autolayout
description: UIScrollView相对布局，包含视频教程
---

UIScrollView在IB中的相对布局一直是个令人头疼的事情，大家所遇到的不外乎下面两个问题

1. 如何正确确定contentSize大小
2. 如何设计超过一屏大小的界面

首先必须知道的一点是使用autolayout后，contentSize无需手动设置，系统会通过加入到UIScrollView的内容来确定contentSize的大小，先来看一个简单的例子。

### 如何正确确定contentSize大小

1. 拖一个`UIScrollView`到VC上，设置其上下左右的约束相对于父视图为0
![image](/images/UIScrollView/UIScrollView-1.png)

2. 拖一个`UILabel`到`UIScrollView`中心上，同样需要设置上下左右的约束
![image](/images/UIScrollView/UIScrollView-2.png)

3. 最终得到的结果:
![image](/images/UIScrollView/UIScrollView-4.png)

__为什么label的布局需要确定设定上下左右的约束？__

前面提到的，系统会通过加入到UIScrollView的内容来确定`contentSize`的大小。设想一下如果去除掉向右和向下的约束，那么UIScrollView为了能包含这个label，contentSize所要达到的条件是：

* 横向至少 139 + 42=181 像素的宽度
* 纵向至少 273 + 21=294 像素的高度

也就是`contentSize`的值至少是{181, 294}，当然这个值还可以是{200, 300}等等，所以说`contentSize`的值是不确定的。而且当去除这任意一个约束的时候，IB也会给出contentSize不确定的警告。

> warning: Ambiguous Layout: Scrollable content size is ambiguous for "Scroll View".

__另外需要注意的是，对于固定行数和固定text值的label，高宽是确定的，不用显性的去设置。__

再来看增加了向下和向右的约束的时，contentSize所要达到的条件是：

* 横向: 139 + 139 + 42 = 320
* 纵向: 274 + 273 + 21 = 568

`contentSize`这时候的值是确定的为`{320, 568}`，所以在布局UIScrollView内容视图的时候，始终要记住一点：必须让UIScrollView知道其contentSize大小，而不是一个不确定的值，这样才算完成UIScrollView布局。

再回头看UIScrollView的布局：相对于其父视图的上下左右约束都是0。这就说明它的宽高其实决定于设备，3.5寸屏幕大小的设备是`{320,480}`，4.0寸屏幕大小的设备是`{320,568}`.....那么如果在iPhone 4上运行，UIScrollView纵向需要滚动，因为纵向需要的像素大小(568) > 480。在iPhone 5上运行，UIScrollView纵向不需要滚动，因为纵向需要的像素大小(568) = 568。

实际运行情况具体如何？在iPhone 5(7.1固件)模拟器上运行会发现，UIScrollView竟然可以纵向滚动，这是为什么。还记得iOS 7给UIScrollView及其子类预留的额外64个像素的滚动区域吗，就是这个原因导致UIScrollView的contentSize值的高度会相对减少64像素。那么纵向需要的像素大小(568) > 568 - 64，所以UIScrollView才会在纵向滚动。可以试着将label向上的约束值改为`274 - 64 = 210`或在IB中取消viewcontroller的`Adjust Scroll View Insets`就会无法滚动。

### 如何设计超过一屏大小的界面

简单的布局两屏宽度的UI，我也上传了具体操作视频到[土豆](http://www.tudou.com/programs/view/RK6niTubChI)

1. 设置viewcontroller的宽度
![image](/images/UIScrollView/UIScrollView-5.png)
选择Size设置为Freeform，这样一来就可以在超过两屏宽度的视图上布局
2. 拖两个view到UIScrollView上，分别设置好颜色，view1代表红色view，view2代表黑色view
![image](/images/UIScrollView/UIScrollView-6.png)
3. view1设置左、上、下约束为0，向右相对于view2属于设置0，view2设置右、上、下约束为0
![image](/images/UIScrollView/UIScrollView-7.png)
先无视警告，到这步contentSize的宽高其实都是未确定的，因为view1和view2的宽高都是不确定的。
4. 选中view1和UIScrollView约束选择相同的宽高，选择view2执行相同的约束，最后Update Frames
![image](/images/UIScrollView/UIScrollView-8.png)
横向：0 + view1的宽度 + 0 + view2的宽度 = 两倍的UIScrollView宽度
纵向：0 + view1的高度 + 0 = UIScrollView高度 或 0 + view2的高度 + 0 = UIScrollView高度
5. 将viewcontroller的size重新设置为Fixed
