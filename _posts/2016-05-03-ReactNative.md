---
layout: post
title: "React Native杂谈"
date: 2016-04-15
comments: true
categories: [React Native]
tags: [react-native]
keywords: [React Native, iOS]
publish: true
description: React Native
---
![image](http://7ls0py.com1.z0.glb.clouddn.com/react_native.png)

和[reactivecocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)一样，[react-native](https://github.com/facebook/react-native)也很早就开始关注了，之所以到前段时间才学习，原因有几点

- 整个项目已经相对成熟
- 国内外社区比较活跃
- JavaScript和React学习成本并没有想象的那么高

对前两个原因我觉得有必要详细说明下，下面的内容我会用RN缩写来代替react-native

RN的更新频率很高，从目前来看差不多半个多月一个版本，从去年的3月底发布一个版本`0.1.0`以来到现在的`0.25.0`为止一共发布了多达82个版本，当然这里面包含很多rc版，可见facebook对RN的重视程度。

其实影响一个项目的流行程度很大一部分取决于其社区的活跃程度，RN社区现在这么活跃也是得益于facebook的推广，比如这两年的F8大会、开源基于Atom的IDE：[nuclide](https://github.com/facebook/nuclide)、第三方库搜索网站[js.coach](https://js.coach/react-native)，以及自家的`Facebook Ads Manager`和`Facebook Groups`项目使用RN开发等，国内的也有不错的交流平台，比如[react native中文社区](http://reactnative.cn/)。另外github上两个RN学习资料、组件等的集合项目[awesome-react-native](https://github.com/jondot/awesome-react-native)和 [react-native-guide](https://github.com/ele828/react-native-guide)都是不错的指引。

我挑了几个大家比较关心的问题一一说下自己的看法和理解

### 学习成本

我想对于学习一项新技能，都会首先去了解需要什么基本条件。对于RN来说，需要的当然就是JavaScript和React基础，但我并没有刻意的去学习JavaScript和React，而是直接上手RN的同时了解和学习这两样。个人认为对于有开发经验的程序员来说，语法并不是问题，即使对于JavaScript这种我认为很"随便"的语言也是一样，所以当你想学习RN但是没有JavaScript和React经验，根本不用担心因为没有基础给你带来多大的阻碍。

### 性能

关于性能问题有很多文章都分析过，我只说自己的主观感受，对于iOS而言，只要不是非常复杂的页面性能基本接近native，不过有时候你可能会在页面切换的时候察觉到略微的卡顿。安卓可能是平台或者RN优化的问题，目前性能表现不是非常理想，不过完全可以接受。这里需要提一下，RN上的动画会对性能造成一定的影响，所以并不建议使用大量动画，官网也有[文档](https://facebook.github.io/react-native/docs/performance.html#content)阐述了性能问题以及出现的原因，如果遇到性能问题，强烈建议仔细阅读。

### 实际应用

完全用RN来写一个商业项目我想会遇到以下问题：

* 多少会涉及到native平台的代码，除非你的项目非常简单，没有涉及到第三方平台(第三方分享、登录)，没有复杂的页面，没有复杂的处理，没有复杂的动画。举个例子，比如APP有清除缓存功能，那么你必须得写两套代码，个人认为无论RN迭代到什么程度，都还是有无法同时满足两个平台的情况，这时候就需要依靠第三方库了。
* 平台之间的UI差异，RN的理念是`write once, run anywhere`，所以你还是需要做好适配平台的准备，这是几乎也是无法避免的，另外可能相同的控件在不同平台上的表现也有差异，我曾经遇到过`TextInput`控件的`placeholder`位置在两个平台下位置不同情况。
* 动画，前面我也提到动画对于性能会存在一定的影响，而且复杂的动画对于RN来说是非常困难的
* RN本身的bug问题，这个问题或许在早期版本尤为突出，一旦遇到只能寄希望于RN团队尽快解决，这种对于商业项目就较为麻烦了

另外学习过程中，我发现比较几个有意思的地方：

1. [ListView](https://facebook.github.io/react-native/docs/listview.html#content)在iOS层是基于UIScrollView封装的
2. [Text](https://facebook.github.io/react-native/docs/text.html#content)在iOS层是UIView，并不是UILabel或者是UITextView，RN是通过Draw方法把文字绘制到视图上的，因为这样会有更好的性能表现，随便提一下iPhone自带的日历应用的也是这样的处理方式，详情：[简书](http://www.jianshu.com/p/1881c12ae33e)
3. [TouchableHighlight](http://reactnative.cn/docs/0.24/touchablehighlight.html#content)等在iOS层实际上是基于`UIGestureRecognizer`的封装并不是UIButton

### 工具

对于初学者，MAC平台推荐使用编辑器：[Atom](https://atom.io/)，配合前面提到的基于Atom的IDE：[nuclide](https://github.com/facebook/nuclide)，再加上几个Atom插件：

* [language-babel](https://github.com/gandm/language-babel)
* [language-javascript-jsx](https://github.com/subtleGradient/language-javascript-jsx)
* [react](https://github.com/orktes/atom-react)

如果觉得自动补全很有必要，可以试试这两个插件[react-snippets](https://atom.io/packages/react-snippets)和[atom-react-native-autocomplete](https://atom.io/packages/atom-react-native-autocomplete)

### 笔记

在用国际版的印象笔记的同学，可以添加我的[RN笔记本](https://www.evernote.com/pub/lingchen621/reactnative)，我会不定期把总结写到上面。
