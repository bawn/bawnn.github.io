---
layout: post
title: "新大陆：Texture"
date: 2016-12-12
comments: true
categories: [Texture]
tags: [iOS]
keywords: [Texture, iOS]
publish: true
description: Texture
---

![image](http://lc.yardwill.top/textture-logo.png)



APP性能的优化，一直都是任重而道远，对于如今需要承载更多信息的APP来说更是突出，值得庆幸的苹果在这方面做得至少比安卓让开发者省心。UIKit 控件虽然在大多数情况下都能满足用户对于流畅性的需求，但有时候还是难以达到理想效果。

[AsyncDisplayKit](https://github.com/facebook/AsyncDisplayKit)(以下简称ASDK) 的出现至少又给了开发者一个不错的选择。毕竟[Paper](https://en.wikipedia.org/wiki/Facebook_Paper)(虽然 Facebook 已经关闭了这个应用)当年有着炫酷的效果的同时依然保持较好的流畅性也得益于 ASDK 的加入。在[Paper](https://en.wikipedia.org/wiki/Facebook_Paper)发布的几个月后 Facebook 就干脆从中剥离出来成为一个独立的库，就在前两天 ASDK 刚好发布了 2.0 版本。目前据我所知国内比较知名有 [轻芒阅读](http://www.wandoujia.com/yilan?utm_source=homepage&utm_campaign=routine&utm_medium=internal&utm_content=header)(豌豆荚一览) 、 [即刻](http://www.ruguoapp.com/) 、 [Yep](https://itunes.apple.com/cn/app/id983891256?mt=8&l=cn) 、[小红书](https://www.xiaohongshu.com/)，[皮皮虾](https://itunes.apple.com/cn/app/%E7%9A%AE%E7%9A%AE%E8%99%BE-%E4%BB%8A%E6%97%A5%E5%A4%B4%E6%9D%A1%E5%AE%98%E6%96%B9%E7%88%86%E7%AC%91%E7%A4%BE%E5%8C%BA/id1393912676) 在用ASDK。

**目前  [AsyncDisplayKit](https://github.com/facebook/AsyncDisplayKit) 已经从 facebook 迁移至 TextureGroup 新的项目地址是  [Texture](https://github.com/TextureGroup/Texture)**

## 控件

Texture 几乎涵盖了常用的控件，下面是 `Texture` 和 `UIKit` 的对应关系，有些封装可以说非常良心。

**Nodes:**

| Texture              | UIKit                                |
| :------------------- | :----------------------------------- |
| ASDisplayNode        | UIView                               |
| ASCellNode           | UITableViewCell/UICollectionViewCell |
| ASTextNode           | UILabel                              |
| ASImageNode          | UIImageView                          |
| ASNetworkImageNode   | UIImageView                          |
| ASVideoNode          | AVPlayerLayer                        |
| ASControlNode        | UIControl                            |
| ASScrollNode         | UIScrollView                         |
| ASControlNode        | UIControl                            |
| ASEditableTextNode   | UITextView                           |
| ASMultiplexImageNode | UIImageView                          |

**Node Containers**

| Texture          | UIKit            |
| :--------------- | :--------------- |
| ASViewController | UIViewController |
| ASTableNode      | UITableView      |
| ASCollectionNode | UICollectionView |
| ASPagerNode      | UICollectionView |

子父类关系：

* ASDisplayNode
  * ASCellNode
    * ASTextCellNode
  * ASCollectionNode
    * ASPagerNode
  * ASControlNode
    * ASButtonNode
    * ASImageNode
      * ASMapNode
      * ASMultiplexImageNode
      * ASNetworkImageNode
        * ASVideoNode
    * ASTextNode
    * ASTextNode2
  * ASEditableTextNode
  * ASScrollNode
  * ASTableNode
  * ASVideoPlayerNode

### ASDisplayNode：



作用同等于`UIView`，是所有 Node 的父类，同时 `ASDisplayNode` 有一个`view`属性，所以`ASDisplayNode`及其子类都可以通过这个`view`来添加`UIKit`控件。

`ASDisplayNode` 中添加 `UIKit`

```
UIView *otherView = [[UIView alloc] init];
otherView.frame = ...;
[node.view addSubview:otherView];
```
或

```
ASDisplayNode *gradientNode = [[ASDisplayNode alloc] initWithViewBlock:^UIView * _Nonnull{
    UIView *view = [[UIView alloc] init];
    return view;
}];
```
第二种的初始化最终生成的就是 block 返回的 `UIKit` 对象，但外部表现出来的是 `ASDisplayNode`。这样子的好处在于布局，关于布局，后面会讲到。

`UIKit` 中添加 `ASDisplayNode`

```
ASImageNode *imageNode = [[ASImageNode alloc] init];
imageNode.image = [UIImage imageNamed:@"iconShowMore"];
imageNode.frame = ...;
[self addSubnode:imageNode];
self.imageNode = imageNode;
```

### ASCellNode：

作用同等于 `UITableViewCell` 或 `UICollectionViewCell`，自带 `indexPath` 属性，有些时候很有用。

### ASTextNode

作用同等于`UILabel`，和 `UILabel` 不同的是 `ASTextNode` 必须通过 `attributedText` 添加文字。

### ASTextNode2

在 ASTextNode 基础修复了一些 Bug

### ASImageNode



作用同等于 `UIImageView`，但是只能设置静态图片，如果需要使用网络图片，请使用 `ASNetworkImageNode`。

### ASNetworkImageNode


作用同等于 `UIImageView`，如果使用网络图片请使用此类，`Texture` 用的是第三方的图片加载库[PINRemoteImage](https://github.com/pinterest/PINRemoteImage)，`ASNetworkImageNode` 其实并不支持 gif，如果需要显示 gif 推荐使用[FLAnimatedImage](https://github.com/Flipboard/FLAnimatedImage)。

### ASButtonNode



作用同等于 `UIButton`，需要注意的是下面这个两个属性

```
@property (nonatomic, assign) CGFloat contentSpacing;// 设置图片和文字的间距
@property (nonatomic, assign) ASButtonNodeImageAlignment imageAlignment;// 图片和文字的排列方式，
```
简直要抱头痛哭一下😭，`imageAlignment` 可以设置两个值：

```
ASButtonNodeImageAlignmentBeginning, // 图片在前，文字在后
ASButtonNodeImageAlignmentEnd// 文字在前，图片在后
```

### ASTableNode


作用同等于 `UITableView`，但是实现上并没有采用 `UITableView` 的重用机制，而是通过用户滚动对需要显示的视图进行`add` 和 不需要的进行`remove` 的操作（我猜的）。另外重要的一点：`ASTableNode` 并没有像 `UITableView` 一样提供一个`-tableView:heightForRowAtIndexPath:`协议方法来决定每个 Cell 的高度，而是由 `ASCellNode` 本身决定。**这样带来的另外一个好处是，动态高度的实现可谓是易如反掌**，具体可以看官方 Demo 中的 [Kittens](https://github.com/facebook/AsyncDisplayKit/tree/master/examples/Kittens)。



## 布局

阅读 [Texture 布局篇](https://bawn.github.io/2017/12/Texture-Layout/)


## 缺点

当然 Texture 并不是完美的，在项目中使用后总结了以下缺点：

1. 虽然和 UIKit 可以混用，但有些组件不得不以 Texture 的形式再封装一套
2. 由于底层实现方式不一样，某些第三方库不再适用，比如[DZNEmptyDataSet](https://github.com/dzenbot/DZNEmptyDataSet)
3. 因为渲染、图片解码、布局等都不在主线程，多多少少会带来例如：线程安全、死锁、调试困难等问题
4. 布局系统学习成本略高，布局繁琐
5. 入侵性强，难以剥离
6. [闪烁问题](https://juejin.im/post/5987cc536fb9a03c4b374bec)
7. 不支持 Interface Builder
8. 目前还不太完善，比如[[Crash] ASDN::Mutex::~Mutex()](https://github.com/TextureGroup/Texture/issues/136)（16年8月1日被提出，直到17年9月1日才修复）


## 其他

**刷新列表**



无论是 `ASTableNode` 还是 `ASCollectionNode` 当列表中已经有数据显示了，调用 `reloadData` 你会发现列表会闪一下。最常见的案例是上拉加载更多获取到新数据后调用 `reloadData` 刷新列表用户体验会比较差，事实上官方文档在 [[Batch Fetching API](http://asyncdisplaykit.org/docs/batch-fetching-api.html)] 给出了解决办法：

```objective-c
- (void)tableNode:(ASTableNode *)tableNode willBeginBatchFetchWithContext:(ASBatchContext *)context 
{
  // Fetch data most of the time asynchronoulsy from an API or local database
  NSArray *newPhotos = [SomeSource getNewPhotos];

  // Insert data into table or collection node
  [self insertNewRowsInTableNode:newPhotos];

  // Decide if it's still necessary to trigger more batch fetches in the future
  _stillDataToFetch = ...;

  // Properly finish the batch fetch
  [context completeBatchFetching:YES];
}
```

获取新数据后直接插入到列表中，而不是刷新整个列表，比如：

```objective-c
- (void)insertSections:(NSIndexSet *)sections withRowAnimation:(UITableViewRowAnimation)animation;
```

和

```objective-c
- (void)insertRowsAtIndexPaths:(NSArray<NSIndexPath *> *)indexPaths withRowAnimation:(UITableViewRowAnimation)animation;
```

**加载数据**

细心的同学可能发现了前面提到内容中的就有相关的方法：

```
- (void)tableNode:(ASTableNode *)tableNode willBeginBatchFetchWithContext:(ASBatchContext *)context;
```

现在绝大多数APP加载更多数据的方法都是通过下拉到列表底部再去请求数据然后添加到列表中，但是 Texture 提供了另外一种更“合理”的方式，原文是这样描述的：

> By default, as a user is scrolling, when they approach the point in the table or collection where they are 2 “screens” away from the end of the current content, the table will try to fetch more data.

当列表滚到到距离底部还有两个屏幕高度请求新的数据，这个阈值是可以调整的。一旦距离底部达到两个屏幕的高度的时候，就会调用前面提到的方法。所以用起来大概是这样的：

```
- (void)tableNode:(ASTableNode *)tableNode willBeginBatchFetchWithContext:(ASBatchContext *)context{
    [context beginBatchFetching];
    [listApi startWithBlockSuccess:^(HQHomeListApi *request) {
        @strongify(self);
        NSArray *array = [request responseJSONObject];
        [self.dataSourceArray addObjectsFromArray:array];
        [self.tableNode insertSections:[NSIndexSet indexSetWithIndexesInRange:rang] withRowAnimation:UITableViewRowAnimationNone];
        [self updateHavMore:array];
        [context completeBatchFetching:YES];
    } failure:NULL];
}

- (BOOL)shouldBatchFetchForTableNode:(ASTableNode *)tableNode{
    return self.haveMore;
}
```

`shouldBatchFetchForTableNode` 用来控制是否需要获取更多数据。这种方式优点在于在网络状况好的情况下用户都不会感受到已经加载了其他数据并显示，缺点在于网络状况不好的情况下用于即使列表已经下拉到底部也没有任何提示。

