---
layout: post
title: "UIStatusBarStyle"
date: 2014-07-29
comments: true
categories: iOS
tags: [UIStatusBarStyle]
published: true
keywords: UIStatusBarStyle
---
在iOS中调整状态栏的样式有两种方式:

#### 1. 在plist中添加 View controller-based status bar appearance 设置为 YES ,然后通过以下方法设置样式

```
- (UIStatusBarStyle)preferredStatusBarStyle{
    return UIStatusBarStyleDefault;
}
```
但是这个方法通常会莫名其妙的不执行，细心的话就会发现，只要在使用了 `UINavigationController` ，而且 `navigationBar` 没有被隐藏的条件下，它的 `rootController` 及之后 `push` 的 `controller` 的 `preferredStatusBarStyle` 方法不会被调用。原因是 `UINavigationController` 会根据自己 `navigationBar` 的 `barStyle` ，来决定 `StatusBarStyle`

解决办法：扩展 `UINavigationController`  ([venj博客](http://cocoa.venj.me/blog/view-controller-based-status-bar-style-and-uinavigationcontroller/))

> UINavigationController+StatusBar.h

```
#import <UIKit/UIKit.h>
@interface UINavigationController (StatusBar)
- (UIStatusBarStyle)preferredStatusBarStyle;
@end
```

>UINavigationController+StatusBar.m

```
#import "UINavigationController+StatusBar.h"
@implementation UINavigationController (StatusBar)
- (UIStatusBarStyle)preferredStatusBarStyle{
    return [self.topViewController preferredStatusBarStyle];
}
@end
```

导入`"UINavigationController+StatusBar.h"`然后调用`- (UIStatusBarStyle)preferredStatusBarStyle`即可

#### 2. 在plist中添加View controller-based status bar appearance设置为NO,然后通过以下方法设置样式

```
[[UIApplication sharedApplication] setStatusBarStyle:UIStatusBarStyleLightContent];
```

这样的话可以随意改变状态栏的样式，无论是否使用了 `UINavigationController` 。但是这个方法有个缺陷:如果在push到一个新VC的时候设置了状态栏为 `UIStatusBarStyleLightContent` ，pop回来的时候状态栏还是处于那个状态，这时候想设置回 `UIStatusBarStyleDefault` 必须再调用一次。也就是说此方法必须手动调用。






