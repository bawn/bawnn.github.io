---
layout: post
title: "字节跳动旗下产品简单分析"
date: 2019-04-23
comments: true
categories: [iOS]
tags: [字节跳动]
keywords: [字节跳动]
publish: true
## description: 字节跳动
---



这篇文章主要通过[Reveal](<https://revealapp.com/>)、[AppSight](https://www.appsight.io/)、ipa包三方面简单的分析字条跳动旗下几款比较有代表性的产品。首先基本信息如下：

| 产品     | 版本  | 主要开发语言 | 第三方库                                                     | Interface Builder         |
| -------- | ----- | ------------ | ------------------------------------------------------------ | ------------------------- |
| 今日头条 | 7.1.7 | Objective-C  | [AppSight](<https://www.appsight.io/app/%E4%BB%8A%E6%97%A5%E5%A4%B4%E6%9D%A1>) | Storyboard                |
| 皮皮虾   | 1.7.2 | Objective-C  | [AppSight](<https://www.appsight.io/app/%E7%9A%AE%E7%9A%AE%E8%99%BE-%E4%BB%8A%E6%97%A5%E5%A4%B4%E6%9D%A1%E5%AE%98%E6%96%B9%E7%88%86%E7%AC%91%E7%A4%BE%E5%8C%BA>) | /                         |
| 抖音     | 5.8.0 | Objective-C  | [AppSight](<https://www.appsight.io/app/%E6%8A%96%E9%9F%B3%E7%9F%AD%E8%A7%86%E9%A2%91>) | Storyboard(Launch Screen) |
| 飞书     | 2.6.3 | Swift        | /                                                            | Storyboard(Launch Screen) |



### 今日头条

部分页面结构：

 * 首页：UICollectionView + UITableView (没想到吧)。
 * 新闻详情页：WKWebView(内容) + Native(评论区)，外面再包了一层 UIScrollView，想进一步了解参考[iOS新闻类App内容页技术探索](<https://dequan1331.github.io/hybrid-page-kit.html>)。

从第三方库的引入来看，[GDataXML-HTML](<https://github.com/graetzer/GDataXML-HTML>)、[libwebp](<https://developers.google.com/speed/webp/>)、[YYKit](<https://github.com/ibireme/YYKit>), [TTTAttributedLabel](<https://github.com/TTTAttributedLabel/TTTAttributedLabel>) 这些应该算是新闻类的常规库，因为它们同时也出现在了[网易新闻](<https://www.appsight.io/app/%E7%BD%91%E6%98%93%E6%96%B0%E9%97%BB>)中。在数据传输方面，头条并没有用传统的 JSON 格式，而是采用 [protobuf](<https://github.com/protocolbuffers/protobuf>) 的形式，gif 图片显示还是热门的[FLAnimatedImage](<https://github.com/Flipboard/FLAnimatedImage>)。还有在"首页"、"西瓜视频"、"小视频"、"新闻的评论模块"等采用的都是绝对布局，这应该是出于性能方面的考量。

另外头条还引入了[react-native](<https://github.com/facebook/react-native>)（ 0.55.4），在好奇会在哪些页面用到的时候，我在 ipa 包中发现有个`assets/react-native-feedcell/components/`的目录，里面包含了两个文件夹：`interest-tags`、`weather` ，在用 Reveal 看过首页的"我的频道"和搜索中"天气"页面后，发现并不是用 react-native 所写，本以为头条是和 airbnb 一样放弃了 react-native 只不过没有移除而已，最终在"关注感兴趣的人"页面得到了证实（多层的 RCTView 嵌套）。

![TT-1](http://lc.yardwill.top/TT-1.jpg)

在 ipa 包内同时还发现了一个`flutter_assets`目录，以及 `Frameworks/Flutter.framework`，所以头条在用 Flutter 的事实没跑了，`packages` 下的资源表明了 Flutter 应该用在小视频相关页面。还有一些可以说的东西：

* WCDB.framework：<https://github.com/Tencent/wcdb>
* yw_1222.jpg（1k大小）：应该是阿里百川的安全图片
* vconsole.js：https://github.com/WechatFE/vConsole
* pause_to_play_list：[lottie-ios](https://github.com/airbnb/lottie-ios) 的资源文件，同时我也在看到 LOTAnimationView 的类，但是为什么 AppSight 上没有被收录到
* Assets.car 中的 `*_night` 图片：这应该用于夜间模式的图片，头条在夜间模式采用的是继承方案，所以有 `SSThemedView`、`SSThemedLabel`、`SSThemedButton` 等这样子的基础类，难道[DKNightVersion](<https://github.com/draveness/DKNightVersion>)的解决方案不能满足头条的需求。



### 皮皮虾

相比于今日头条，皮皮虾作为 18 年的项目，显得保守了一些，虽然引入了 Swift 但我在目前没有在视图方面找到任何的使用痕迹，而且也没有在跨平台上有任何的尝试，当然这只是从我所能获得信息中判断，也不排除皮皮虾使用的是内涵段子老项目的可能。从第三方库的使用看，比较特别的是引入了 [AsyncDisplayKit](<https://github.com/facebookarchive/AsyncDisplayKit>)([Texture](<https://github.com/TextureGroup/Texture>))，但是只出现在首页的"图片" tab 页面内，另外在 keywindow 上有一个常驻的 MPVolumeView 视图，不知道具体拿来做什么用。

![TT-2](http://lc.yardwill.top/TT-2.jpg)



### 抖音

部分页面结构：

- 首页：UIPageViewController + UITableView
- 频道详情页：UIScrollView + UICollectionView，关于这个页面实现的技术实现可以看到我的另外一篇[博客](<https://bawn.github.io/2019/02/NestedScrolling/>)

单纯从 Reveal 和 ipa 两个方面并没有挖掘出抖音其他有意思的地方，不过奇怪的是在 appsight 上国内版最近更新时间是 2017.4.28，海外版稍微新一点，更新时间是 2017.12.11，我在这里猜测应该是抖音这种量级的产品根本不需要再依靠第三方库快速开发，也就是说可能在 2017.4.28 左右的时间，所有功能都不在依赖第三方库开发。



### 飞书(lark)

飞书可能是字节跳动旗下最新的项目之一，在 App Store 上显示的是八个月前推出的 1.9.1 版本，作为 18 年的项目飞书不仅在开发语言上选择了 Swift，兼容性上也只兼容到 iOS 10.0。

首先从**页面结构**说起，飞书在整体的页面结构上采用的是

```
keywindow.rootViewController --> UINavigationController --> UITabBarController
```

这种形式好处在于 APP 内的所有页面都只在同一个栈上，便于管理，当然前提条件是 tabbar 不需要常驻。飞书在 webview 优化有以下几个方面：

#### webview 池

在应用内的 keywindow 上始终有两个隐藏的 webview，一旦有需要使用 webview 就把 keywindow 上 webview 移除，添加到需要用的 viewcontroller 上，同时再初始化一个 webview 添加到 keywindow 上。

![TT-3](http://lc.yardwill.top/TT-3.jpg)



#### 离线包

事实上对于飞书这种的对 webview 有重度依赖的应用，在 webview 优化形式肯定不止是加快启动速度这么简单。通过对飞书的沙盒文件分析，在 DocsSDK/ResourceService/docs_channel/eesz 文件夹内发现一些用于 webview 优化的离线包文件，包括



* 2b8654440ea92cd9ed66fb5211abd21c.md5：用于校验各个文件的完整性
* current_revision：版本比较文件，决定是否需要更新离线包
* resource/bear：用于本地加载的一些js、css 文件，文件夹命名让我想起了笔记应用[bear](https://bear.app/)
* template/bear：html 模板文件



所以关于离线包的下载整个流程是这样的

![TT-4](http://lc.yardwill.top/TT-4.png)

之后再通过下面这种方式加载 html 模板文件，所涉及到 js、css 文件都会通过本地获取，不需要再从服务端拉取，大大加快了网页的加载速度。

```swift
let url = URL(fileURLWithPath: "")
let requset = URLRequest(url: url)
webView.load(requset)
```



#### 请求拦截

但进一步观察会发现 html 模板文件内的 css、js 引用并不是通过本地加载的方式

```html
 <script src="//s3.pstatp.com/toutiao/monitor/sdk/slardar.js"></script>
    <script>window.Slardar && window.Slardar.install({ sampleRate: 1, bid: "docs_mobile", pid: "index", ignoreAjax: [/mcs\.snssdk\.com/], ignoreStatic: [] })</script>
    <script src="//s3.pstatp.com/eesz/resource/bear/js/vendors~mobile_app~mobile_update.8c813a80b906f70858fb.js"></script>
    <script src="//s3.pstatp.com/eesz/resource/bear/js/mobile_app~mobile_update.9c09912f9c786c0914b6.js"></script>
    <script src="//s3.pstatp.com/eesz/resource/bear/js/mobile_app.7fd12a95844c9b77cc0e.js"></script>
```

这就有点奇怪，因为这样子的话 resource/bear 文件夹内的资源文件就失去了意义，在查阅其他资料后，我猜测飞书对 webview 的请求都做了拦截，当需要获取的js、css甚至是图片资源的时候直接给与本地的缓存文件，也就是 resource/bear 内的文件。