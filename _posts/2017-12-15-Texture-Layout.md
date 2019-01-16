---
layout: post
title: "Texture 布局篇"
date: 2017-12-15
comments: true
categories: [Texture]
tags: [iOS]
keywords: [Texture, iOS]
publish: true
description: Texture
---

![image](http://lc.yardwill.top/textture-logo.png)

`Texture` 拥有自己的一套成熟布局方案，虽然学习成本略高，但至少比原生的 `AutoLayout` 写起来舒服，重点是性能远好于 `AutoLayout` ，`Texture` 文档上也指出了这套布局方案的的优点：

* **Fast**: As fast as manual layout code and significantly faster than Auto Layout
* **Asynchronous & Concurrent**: Layouts can be computed on background threads so user interactions are not interrupted.
* **Declarative**: Layouts are declared with immutable data structures. This makes layout code easier to develop, document, code review, test, debug, profile, and maintain.
* **Cacheable**: Layout results are immutable data structures so they can be precomputed in the background and cached to increase user perceived performance.
* **Extensible**: Easy to share code between classes.

首先这套布局都是基于 `Texture` 组件的，所以当遇到要使用原生控件时，通过用 block 的方式包装一个原生组件再合适不过了，例如:

```
ASDisplayNode *animationImageNode = [[ASDisplayNode alloc] initWithViewBlock:^UIView * _Nonnull{
    FLAnimatedImageView *animationImageView = [[FLAnimatedImageView alloc] init];
    animationImageView.layer.cornerRadius = 2.0f;
    animationImageView.clipsToBounds = YES;
    return animationImageView;
}];
[self addSubnode:animationImageNode];
self.animationImageNode = animationImageNode;
```


`ASDisplayNode` 在初始化之后会检查是否有子视图，如果有就会调用

```
- (ASLayoutSpec *)layoutSpecThatFits:(ASSizeRange)constrainedSize
```

方法进行布局，所以对视图进行布局需要重写这个方法。看一个例子：

```objective-c
- (ASLayoutSpec *)layoutSpecThatFits:(ASSizeRange)constrainedSize{
    ASInsetLayoutSpec *inset = [ASInsetLayoutSpec insetLayoutSpecWithInsets:UIEdgeInsetsZero child:_childNode];
    return insetLayout;
}
```
`_childNode` 相对于父视图边距都为 0，也就是`AutoLayout`中 `top` `bottom` `left` `right` 都为 0。

```
-----------------------------父视图----------------------------
|  -------------------------_childNode---------------------  |
|  |                                                      |  |
|  |                                                      |  |
|  ---------------------------  ---------------------------  |
--------------------------------------------------------------
```

可以看到`layoutSpecThatFits:`方法返回的必须是 `ASLayoutSpec`, `ASInsetLayoutSpec` 是它的子类之一，下面是所有的子类及其关系：

* ASLayoutSpec
  * ASAbsoluteLayoutSpec   // 绝对布局
  * ASBackgroundLayoutSpec // 背景布局
  * ASInsetLayoutSpec      // 边距布局
  * ASOverlayLayoutSpec    // 覆盖布局
  * ASRatioLayoutSpec      // 比例布局
  * ASRelativeLayoutSpec   // 顶点布局
    * ASCenterLayoutSpec   // 居中布局
  * ASStackLayoutSpec      // 盒子布局
  * ASWrapperLayoutSpec    // 填充布局
  * ASCornerLayoutSpec  // 角标布局

___

### ASAbsoluteLayoutSpec

使用方法和原生的绝对布局类似

```
- (ASLayoutSpec *)layoutSpecThatFits:(ASSizeRange)constrainedSize{
  self.childNode.style.layoutPosition = CGPointMake(100, 100);
  self.childNode.style.preferredLayoutSize = ASLayoutSizeMake(ASDimensionMake(100), ASDimensionMake(100));

  ASAbsoluteLayoutSpec *absoluteLayout = [ASAbsoluteLayoutSpec absoluteLayoutSpecWithChildren:@[self.childNode]];
  return absoluteLayout;
}
```
值得提的是：ASAbsoluteLayoutSpec 一般情况都会通过 `ASOverlayLayoutSpec` 或 `ASOverlayLayoutSpec` 着陆，因为只有上述两种布局才能保留 ASAbsoluteLayoutSpec 绝对布局的事实。举个例子当视图中只有一个控件需要用的是 `ASAbsoluteLayoutSpec` 布局，而其他控件布局用的是 `ASStackLayoutSpec`（后面会介绍），那么一旦 absoluteLayout 被加入到 `ASStackLayoutSpec` 也就失去它原本的布局的意义。

```
ASOverlayLayoutSpec *contentLayout = [ASOverlayLayoutSpec overlayLayoutSpecWithChild:stackLayout overlay:absoluteLayout];
```



不过官方文档明确指出应该尽量少用这种布局方式:

>Absolute layouts are less flexible and harder to maintain than other types of layouts.

___

### ASBackgroundLayoutSpec

```
- (ASLayoutSpec *)layoutSpecThatFits:(ASSizeRange)constrainedSize{
  ASBackgroundLayoutSpec *backgroundLayout = [ASBackgroundLayoutSpec backgroundLayoutSpecWithChild:self.childNodeB background:self.childNodeA];
  return backgroundLayout;
}
```

把`childNodeA` 做为 `childNodeB` 的背景，也就是 `childNodeB` 在上层，**要注意的是 **`ASBackgroundLayoutSpec` 事实上根本不会改变视图的层级关系，比如：

```
ASDisplayNode *childNodeB = [[ASDisplayNode alloc] init];
childNodeB.backgroundColor = [UIColor blueColor];
[self addSubnode:childNodeB];
self.childNodeB = childNodeB;

ASDisplayNode *childNodeA = [[ASDisplayNode alloc] init];
childNodeA.backgroundColor = [UIColor redColor];
[self addSubnode:childNodeA];
self.childNodeA = childNodeA;
```

那么即使使用上面的布局方式，`childNodeB` 依然在下层。

___

### ASInsetLayoutSpec

比较常用的一个类，看图应该能一目了然（图片来自于[官方文档](http://localhost:4000/2016/12/AsyncDisplayKit/)）
![image](http://lc.yardwill.top/ASInsetLayoutSpec-diagram.png)

```
- (ASLayoutSpec *)layoutSpecThatFits:(ASSizeRange)constrainedSize{
    ASInsetLayoutSpec *inset = [ASInsetLayoutSpec insetLayoutSpecWithInsets:UIEdgeInsetsZero child:_childNode];
    return insetLayout;
}
```

`_childNode` 相对于父视图边距都为 0，相当于填充整个父视图。它和之后会说到的
`ASOverlayLayoutSpec` 实际上更多的用来组合两个 `Element` 而已。

___

### ASOverlayLayoutSpec

参考 `ASBackgroundLayoutSpec`

___

### ASRatioLayoutSpec
![image](http://lc.yardwill.top/ASRatioLayoutSpec-diagram.png)（图片来自于[官方文档](http://localhost:4000/2016/12/AsyncDisplayKit/)）

也是比较常用的一个类，作用是设置自身的高宽比，例如设置正方形的视图

```
- (ASLayoutSpec *)layoutSpecThatFits:(ASSizeRange)constrainedSize{
    ASRatioLayoutSpec *ratioLayout = [ASRatioLayoutSpec ratioLayoutSpecWithRatio:1.0f child:self.childNodeA];
    return ratioLayout;
}
```

___

### ASRelativeLayoutSpec

把它称为**顶点布局**可能有点不恰当，实际上它可以把视图布局在：`左上`、`左下`、`右上`、`右下`四个顶点以外，还可以设置成居中布局。

![image](http://lc.yardwill.top/ASRelativeLayoutSpec.jpg)

```
- (ASLayoutSpec *)layoutSpecThatFits:(ASSizeRange)constrainedSize{
    self.childNodeA.style.preferredSize = CGSizeMake(100, 100);
    ASRelativeLayoutSpec *relativeLayout = [ASRelativeLayoutSpec relativePositionLayoutSpecWithHorizontalPosition:ASRelativeLayoutSpecPositionEnd verticalPosition:ASRelativeLayoutSpecPositionStart sizingOption:ASRelativeLayoutSpecSizingOptionDefault child:self.childNodeA];
    return relativeLayout;
}
```
上面的例子就是把 `childNodeA` 显示在右上角。

___

### ASCenterLayoutSpec

绝大多数情况下用来居中显示视图

```
- (ASLayoutSpec *)layoutSpecThatFits:(ASSizeRange)constrainedSize{
    self.childNodeA.style.preferredSize = CGSizeMake(100, 100);
    ASCenterLayoutSpec *relativeLayout = [ASCenterLayoutSpec centerLayoutSpecWithCenteringOptions:ASCenterLayoutSpecCenteringXY sizingOptions:ASCenterLayoutSpecSizingOptionDefault child:self.childNodeA];
    return relativeLayout;
}
```

___

### ASStackLayoutSpec

可以说这是**最常用的类**，而且相对于其他类来说在功能上是最接近于 `AutoLayout` 的。
之所以称之为**盒子布局**是因为它和 CSS 中 `Flexbox` 很相似，关于 `Flexbox` 的可以看下阮一峰的这篇[文章](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html?utm_source=tuicool)。

先看一个例子：

```
- (ASLayoutSpec *)layoutSpecThatFits:(ASSizeRange)constrainedSize{
    self.childNodeA.style.preferredSize = CGSizeMake(100, 100);
    self.childNodeB.style.preferredSize = CGSizeMake(200, 200);
    ASStackLayoutSpec *stackLayout = [ASStackLayoutSpec stackLayoutSpecWithDirection:ASStackLayoutDirectionVertical
                                                                             spacing:12
                                                                      justifyContent:ASStackLayoutJustifyContentStart
                                                                          alignItems:ASStackLayoutAlignItemsStart
                                                                            children:@[self.childNodeA, self.childNodeB]];
    return stackLayout;
}
```
简单的说明下各个参数的作用：

1. `direction`：主轴的方向，有两个可选值：
* 纵向：`ASStackLayoutDirectionVertical`
* 横向：`ASStackLayoutDirectionHorizontal`
2. `spacing`: 主轴上视图排列的间距，比如有四个视图，那么它们之间的存在三个间距值都应该是`spacing`
3. `justifyContent`: 主轴上的排列方式，有五个可选值：
* `ASStackLayoutJustifyContentStart` 从前往后排列
* `ASStackLayoutJustifyContentCenter` 居中排列
    * `ASStackLayoutJustifyContentEnd` 从后往前排列
    * `ASStackLayoutJustifyContentSpaceBetween` 间隔排列，两端无间隔
    * `ASStackLayoutJustifyContentSpaceAround` 间隔排列，两端有间隔
4. `alignItems`: 交叉轴上的排列方式，有五个可选值：
* `ASStackLayoutAlignItemsStart` 从前往后排列
* `ASStackLayoutAlignItemsEnd` 从后往前排列
    * `ASStackLayoutAlignItemsCenter` 居中排列
    * `ASStackLayoutAlignItemsStretch` 拉伸排列
    * `ASStackLayoutAlignItemsBaselineFirst` 以第一个文字元素基线排列（主轴是横向才可用）
    * `ASStackLayoutAlignItemsBaselineLast` 以最后一个文字元素基线排列（主轴是横向才可用）
5. `children`: 包含的视图。数组内元素顺序同样代表着布局时排列的顺序，所以需要注意

**主轴的方向设置尤为重要**，如果主轴设置的是 `ASStackLayoutDirectionVertical`, 那么 `justifyContent` 各个参数的意义就是：

* `ASStackLayoutJustifyContentStart` 从上往下排列
* `ASStackLayoutJustifyContentCenter` 居中排列
* `ASStackLayoutJustifyContentEnd` 从下往上排列
* `ASStackLayoutJustifyContentSpaceBetween` 间隔排列，两端无间隔
* `ASStackLayoutJustifyContentSpaceAround` 间隔排列，两端有间隔

`alignItems` 就是：

* `ASStackLayoutAlignItemsStart` 从左往右排列
* `ASStackLayoutAlignItemsEnd` 从右往左排列
* `ASStackLayoutAlignItemsCenter` 居中排列
* `ASStackLayoutAlignItemsStretch` 拉伸排列
* `ASStackLayoutAlignItemsBaselineFirst` 无效
* `ASStackLayoutAlignItemsBaselineLast` 无效

对于子视图间距不一样的布局方法，后面实战中会讲到。

___

### ASWrapperLayoutSpec

填充整个视图

```
- (ASLayoutSpec *)layoutSpecThatFits:(ASSizeRange)constrainedSize{    
    ASWrapperLayoutSpec *wrapperLayout = [ASWrapperLayoutSpec wrapperWithLayoutElement:self.childNodeA];
    return wrapperLayout;
}
```



---

### ASCornerLayoutSpec

顾名思义  ASCornerLayoutSpec 适用于类似于角标的布局

```swift
override func layoutSpecThatFits(_ constrainedSize: ASSizeRange) -> ASLayoutSpec
{
  let cornerSpec = ASCornerLayoutSpec(child: avatarNode, corner: badgeNode, location: .topRight)
  cornerSpec.offset = CGPoint(x: -3, y: 3)
}
```

**最需要注意的是**`offset`是控件的Center的偏移



## 布局实战

### 案例一
![image](http://lc.yardwill.top/ASDKDemo1.png)

简单的文件覆盖在图片上，文字居中。

```
- (ASLayoutSpec *)layoutSpecThatFits:(ASSizeRange)constrainedSize{
    ASWrapperLayoutSpec *wrapperLayout = [ASWrapperLayoutSpec wrapperWithLayoutElement:self.coverImageNode];

    ASCenterLayoutSpec *centerSpec = [ASCenterLayoutSpec centerLayoutSpecWithCenteringOptions:ASCenterLayoutSpecCenteringXY sizingOptions:ASCenterLayoutSpecSizingOptionDefault child:self.textNode];
    ASOverlayLayoutSpec *overSpec = [ASOverlayLayoutSpec overlayLayoutSpecWithChild:wrapperLayout overlay:centerSpec];
    return overSpec;
}
```
1. `ASWrapperLayoutSpec` 把图片铺满整个视图
2. `ASCenterLayoutSpec` 把文字居中显示
3. `ASOverlayLayoutSpec` 把文字覆盖到图片上

注意第三步就是之前提到的 `ASOverlayLayoutSpec`/`ASBackgroundLayoutSpec` 的作用：用于组合两个 `Element`。

### 案例二

![image](http://lc.yardwill.top/ASDKDemo21.png)

这个是[轻芒阅读](http://www.wandoujia.com/yilan?utm_source=homepage&utm_campaign=routine&utm_medium=internal&utm_content=header)(豌豆荚一览) APP 内 AppSo 频道 Cell 的布局，应该也是比较典型的布局之一。为了方便理解先给各个元素定一下名称，从上至下，从左往右分别是：

* coverImageNode // 大图
* titleNode // 标题
* subTitleNode // 副标题
* dateTextNode // 发布时间
* shareImageNode // 分享图标
* shareNumberNode // 分享数量
* likeImageNode // 喜欢图标
* likeNumberNode // 喜欢数量

```objective-c
- (ASLayoutSpec *)layoutSpecThatFits:(ASSizeRange)constrainedSize{
    
    
    self.shareImageNode.style.preferredSize = CGSizeMake(15, 15);
    self.likeImageNode.style.preferredSize = CGSizeMake(15, 15);
    
    ASStackLayoutSpec *likeLayout = [ASStackLayoutSpec horizontalStackLayoutSpec];
    likeLayout.spacing = 4.0;
    likeLayout.justifyContent = ASStackLayoutJustifyContentStart;
    likeLayout.alignItems = ASStackLayoutAlignItemsCenter;
    likeLayout.children = @[self.likeImageNode, self.likeNumberNode];
    
    ASStackLayoutSpec *shareLayout = [ASStackLayoutSpec horizontalStackLayoutSpec];
    shareLayout.spacing = 4.0;
    shareLayout.justifyContent = ASStackLayoutJustifyContentStart;
    shareLayout.alignItems = ASStackLayoutAlignItemsCenter;
    shareLayout.children = @[self.shareImageNode, self.shareNumberNode];
    
    ASStackLayoutSpec *otherLayout = [ASStackLayoutSpec horizontalStackLayoutSpec];
    otherLayout.spacing = 12.0;
    otherLayout.justifyContent = ASStackLayoutJustifyContentStart;
    otherLayout.alignItems = ASStackLayoutAlignItemsCenter;
    otherLayout.children = @[likeLayout, shareLayout];
    
    ASStackLayoutSpec *bottomLayout = [ASStackLayoutSpec horizontalStackLayoutSpec];
    bottomLayout.justifyContent = ASStackLayoutJustifyContentSpaceBetween;
    bottomLayout.alignItems = ASStackLayoutAlignItemsCenter;
    bottomLayout.children = @[self.dateTextNode, otherLayout];
    
    self.titleNode.style.spacingBefore = 12.0f;
    
    self.subTitleNode.style.spacingBefore = 16.0f;
    self.subTitleNode.style.spacingAfter = 20.0f;
    
    ASRatioLayoutSpec *rationLayout = [ASRatioLayoutSpec ratioLayoutSpecWithRatio:0.5 child:self.coverImageNode];
    
    ASStackLayoutSpec *contentLayout = [ASStackLayoutSpec verticalStackLayoutSpec];
    contentLayout.justifyContent = ASStackLayoutJustifyContentStart;
    contentLayout.alignItems = ASStackLayoutAlignItemsStretch;
    contentLayout.children = @[
                               rationLayout,
                               self.titleNode,
                               self.subTitleNode,
                               bottomLayout
                               ];
    
    ASInsetLayoutSpec *insetLayout = [ASInsetLayoutSpec insetLayoutSpecWithInsets:UIEdgeInsetsMake(16, 16, 16, 16) child:contentLayout];
    return insetLayout;
}
```

下面详细解释下布局，不过首先要明确的是，Texture 的这套布局方式遵守**从里到外**的布局原则，使用起来才会得心应手。

1. 根据布局的原则，首先利用 `ASStackLayoutSpec` 布局 `分享图标` 和 `分享数量`、 `喜欢图标` 和 `喜欢数量`。
2. 还是通过 `ASStackLayoutSpec` 包装第一步的两个的布局得到 `otherLayout` 布局对象。
3. 依然是 `ASStackLayoutSpec` 包装`otherLayout`和 `发布时间`。注意这里设置横向的排列方式 `ASStackLayoutJustifyContentSpaceBetween`已到达两端布局的目的，最终返回 `bottomLayout`。
4. 由于 `大图` 是网络图片，对于 Cell 来说，子视图的布局必能能决定其高度（Cell 宽度是默认等于 TableNode 的宽度），所以这里必须设置 `大图` 的高度，`ASRatioLayoutSpec` 设置了图片的高宽比。
5. 接下来布局应该就是 `大图`、`标题`、`副标题`、`bottomLayout` 的一个纵向布局，可以发现这里的视图间距并不相同，这时候 `spacingBefore` 和 `spacingAfter` 就会很有用，它们用来分别设置元素在主轴上的前后间距。`self.titleNode.style.spacingBefore = 12.0f;` 意思就是 `标题` 相对于 `大图` 间距为 12。
6. 最后通过一个 `ASInsetLayoutSpec` 设置一个边距。


可以看到不仅是 `Node`，`ASLayoutSpec` 本身也可以作为布局元素，这是因为只要是遵守了 `<ASLayoutElement>` 协议的对象都可以作为布局元素。



### 案例三

![image](http://lc.yardwill.top/Texture-1.jpg)



```swift
    override func layoutSpecThatFits(_ constrainedSize: ASSizeRange) -> ASLayoutSpec {
        
        self.node1.style.preferredSize = CGSize(width: constrainedSize.max.width, height: 136)
        
        
        self.node2.style.preferredSize = CGSize(width: 58, height: 25)
        self.node2.style.layoutPosition = CGPoint(x: 14.0, y: 95.0)
        
        self.node3.style.height = ASDimensionMake(37.0)
        self.node4.style.preferredSize = CGSize(width: 80, height: 20)
        self.node5.style.preferredSize = CGSize(width: 80, height: 20)
        
        self.node4.style.spacingBefore = 14.0
        self.node5.style.spacingAfter = 14.0
        
        let absoluteLayout = ASAbsoluteLayoutSpec(children: [self.node2])
        
        let overlyLayout = ASOverlayLayoutSpec(child: self.node1, overlay: absoluteLayout)
        
        let insetLayout = ASInsetLayoutSpec(insets: UIEdgeInsetsMake(0, 14, 0, 14), child: self.node3)
        insetLayout.style.spacingBefore = 13.0
        insetLayout.style.spacingAfter = 25.0
        
        
        let bottomLayout = ASStackLayoutSpec.horizontal()
        bottomLayout.justifyContent = .spaceBetween
        bottomLayout.alignItems = .start
        bottomLayout.children = [self.node4, self.node5]
        bottomLayout.style.spacingAfter = 10.0
//        bottomLayout.style.width = ASDimensionMake(constrainedSize.max.width)
        
        
        let stackLayout = ASStackLayoutSpec.vertical()
        stackLayout.justifyContent = .start
        stackLayout.alignItems = .stretch
        stackLayout.children = [overlyLayout, insetLayout, bottomLayout]
        
        return stackLayout
    }
```



为了演示 ASAbsoluteLayoutSpec 的使用，这里 node3 我们用 ASAbsoluteLayoutSpec 布局。

接下来说下要点：

1. node 和 layoutSpec 都可以设置 style 属性，因为它们都准守 ASLayoutElement 协议
2. 当 spaceBetween 没有达到两端对齐的效果，尝试设置当前 layoutSpec 的 `width`（如注释）或它的上一级布局对象的 alignItems，在例子中就是 `stackLayout.alignItems = .stretch`
3. ASAbsoluteLayoutSpec 必须有落点（除非是只有绝对布局），例子中 ASAbsoluteLayoutSpec 着落点就在 ASOverlayLayoutSpec



### 案例四

![image](http://lc.yardwill.top/Texture-3.jpg)



此案例主要为了演示 `flexGrow` 的用法，先介绍下 flexGrow 的作用（来自于简书[九彩拼盘](http://www.jianshu.com/p/0642dfe0e571)）

>该属性来设置，当父元素的宽度大于所有子元素的宽度的和时（即父元素会有剩余空间），子元素如何分配父元素的剩余空间。
>
>flex-grow的默认值为0，意思是该元素不索取父元素的剩余空间，如果值大于0，表示索取。值越大，索取的越厉害。举个例子:
>
>父元素宽400px，有两子元素：A和B。A宽为100px，B宽为200px，则空余空间为 400-（100+200）= 100px。
>
>如果A，B都不索取剩余空间，则有100px的空余空间。
>
>如果A索取剩余空间:设置flex-grow为1，B不索取。则最终A的大小为 自身宽度（100px）+ 剩余空间的宽度（100px）= 200px
>
>如果A，B都设索取剩余空间，A设置flex-grow为1，B设置flex-grow为2。则最终A的大小为 自身宽度（100px）+ A获得的剩余空间的宽度（100px * (1/(1+2))）,最终B的大小为 自身宽度（200px）+ B获得的剩余空间的宽度（100px * (2/(1+2))）



```swift
     override func layoutSpecThatFits(_ constrainedSize: ASSizeRange) -> ASLayoutSpec {
        
        self.node1.style.height = ASDimensionMake(20.0)
        var imageLayoutArray = [ASLayoutElement]()
        
        [self.node2, self.node3, self.node4].forEach { (node) in
            let layout = ASRatioLayoutSpec(ratio: 2.0/3.0, child: node)
            layout.style.flexGrow = 1 // 相当于宽度相等
            imageLayoutArray.append(layout)
        }
        
        let imageLayout = ASStackLayoutSpec.horizontal()
        imageLayout.justifyContent = .start
        imageLayout.alignItems = .start
        imageLayout.spacing = 14.0
        imageLayout.children = imageLayoutArray
        
        let contentLayout = ASStackLayoutSpec.vertical()
        contentLayout.justifyContent = .start
        contentLayout.alignItems = .stretch
        contentLayout.spacing = 22.0
        contentLayout.children = [self.node1, imageLayout]
        
        return ASInsetLayoutSpec(insets: UIEdgeInsetsMake(22.0, 16.0, 22.0, 16.0), child: contentLayout)
    }
```

在这个案例中 node2、node3、node4 的宽度的总和小于父元素的宽度，所以为了达到宽度相同只需要设置三者的 flexGrow 相同就行（都为1），再通过 ASRatioLayoutSpec 固定各自的宽高比，那么对于这个三个控件来说最终的宽度是确定的。



### 案例四

![image](http://lc.yardwill.top/Texture-2.jpg)



此案例主要为了演示 `flexShrink` 的用法，同样还来自于简书[九彩拼盘](http://www.jianshu.com/p/0642dfe0e571)关于 flexShrink 的介绍

> 该属性来设置，当父元素的宽度小于所有子元素的宽度的和时（即子元素会超出父元素），子元素如何缩小自己的宽度的。
>
> flex-shrink的默认值为1，当父元素的宽度小于所有子元素的宽度的和时，子元素的宽度会减小。值越大，减小的越厉害。如果值为0，表示不减小。
>
> 举个例子:父元素宽400px，有两子元素：A和B。A宽为200px，B宽为300px。则A，B总共超出父元素的宽度为(200+300)- 400 = 100px。
>
> 如果A，B都不减小宽度，即都设置flex-shrink为0，则会有100px的宽度超出父元素。如果A不减小宽度:设置flex-shrink为0，B减小。则最终B的大小为 自身宽度(300px)- 总共超出父元素的宽度(100px)= 200px如果A，B都减小宽度，A设置flex-shirk为3，B设置flex-shirk为2。则最终A的大小为 自身宽度(200px)- A减小的宽度(100px * (200px * 3/(200 * 3 + 300 * 2))) = 150px,最终B的大小为 自身宽度(300px)- B减小的宽度(100px * (300px * 2/(200 * 3 + 300 * 2))) = 250px

目前关于该属性最常见还是用于对文本的宽度限制，在上图中 textNode 和 displayNode 是两端对齐，而且需要限制文本的最大宽度，这时候设置 `flexShrink` 是最方便的。

```swift
override func layoutSpecThatFits(_ constrainedSize: ASSizeRange) -> ASLayoutSpec {
        self.displayNode.style.preferredSize = CGSize(width: 42.0, height: 18.0)
        self.textNode.style.flexShrink = 1
        
        let contentLayout = ASStackLayoutSpec.horizontal()
        contentLayout.justifyContent = .spaceBetween
        contentLayout.alignItems = .start
        contentLayout.children = [self.textNode, self.displayNode]
        
        let insetLayout = ASInsetLayoutSpec(insets: UIEdgeInsetsMake(16.0, 16.0, 16.0, 16.0), child: contentLayout)
        
        return insetLayout
        
    }
```

**随便提一下的是如果 ASTextNode 出现莫名的文本截断问题，可以用 ASTextNode2 代替。**

### 案例五

还算比较典型的例子

![image](http://lc.yardwill.top/ASDKDemo3.png)

```swift
override func layoutSpecThatFits(_ constrainedSize: ASSizeRange) -> ASLayoutSpec {

        let otherLayout = ASInsetLayoutSpec(insets: UIEdgeInsetsMake(10.0, 10.0, CGFloat(Float.infinity), CGFloat(Float.infinity)), child: topLeftNode)
        
        let contentLayout = ASOverlayLayoutSpec(child: coverImageNode, overlay: otherLayout)
        return contentLayout
    }
```

利用 ASInsetLayoutSpec 是最好的解决方案，值得注意的是对于红色控件只需要设置向上和向左的间距，那么其他方向的可以用 `CGFloat(Float.infinity)` 代替，并不需要给出具体数值。
