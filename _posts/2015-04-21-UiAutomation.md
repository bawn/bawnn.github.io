---
layout: post
title: "UI自动化测试"
date: 2015-04-21
comments: true
categories: iOS
tags: [UI, iOS, UiAutomation]
keywords: UI自动化测试 iOS
publish: false
description: iOS UI自动化测试
---

秉着想偷懒的原则和测试这块一直存在的诟病，空闲的时把苹果提供的UIAutomation研究了一番，心想这样就可以坐等APP自己跑完所有流程然后输出 carsh 报告。但是想象很丰满，现实很骨感，UiAutomation 并没有想象中那么的完美。<br>

## 基本介绍

`⌘ + I` 打开Instruments，选择 UiAutomation，基本界面就是这样

![image](/images/UiAutomation/tool.png)
**功能区域介绍:**<br>
① 开始、结束测试按钮，选择设备和项目菜单<br>
② JS脚本编辑区，Trace Log 和 Editor Log显示区<br>
③ 自动化测试时间线<br>
④ 其他菜单页面<br>
⑤ 脚本或是Log选择菜单<br>
⑥ 自动生成JS脚本代码的开始、结束和停止按钮<br>

看一个简单的测试例子，APP的界面上有一个UITextField，称之为`A元素`，Navigationbar有个rightItem，称之为`B元素`，点击rightItem会push到另一个VC
![image](/images/UiAutomation/screen.png)

在④中选择一个空的脚本写入下列代码，对 JS 不了解的也不用当心，自动化测试的 JS 代码非常简单。


```
var target = UIATarget.localTarget();
var app = target.frontMostApp();
var window = app.mainWindow();
// 打印元素树
app.logElementTree();
```

`⌘ + R`跑一下，②区域应该会自动切换到Log界面
![image](/images/UiAutomation/log.png)

Log打印出来的是当前界面的元素（UIAElement）树，同层级的元素会被包含到数组中，模拟用户操作其实就是对元素的操作，那么获取到元素才是关键。`logElementTree()`这个函数非常有用，在页面切换的时候记得要再次调用，以便找到你想要的元素<br>

**补充脚本：**


```
var target = UIATarget.localTarget();
var app = target.frontMostApp();
var window = app.mainWindow();
app.logElementTree();
var textField = window.textFields()[0];
// 在 A 元素中输入122
textField.setValue("122");
// 停顿1秒
target.delay(1);
// 获取 B 元素
var rightButton = window.navigationBar().buttons()['Button'];
// 点击 B 元素
rightButton.tap();
```

运行后的效果是：A 元素被输入了122的字符——>然后 B 元素被点击——>APP 进入了一个页面

**代码解读：**

* 获取 A 元素：`window.textFields()[0]`，界面上只有一个textField，那么当然是textFields数组的第一个元素
* 获取 B 元素：`window.navigationBar().buttons()['Button']`，由于 B 元素是在navigationBar上，所以需要先获取navigationBar，再从buttons数组中获取 B 元素，通过 B 元素`name`属性的值`Button`(默认值)获取。

元素的`name`值也可以手动设置，比如设置 A 元素的`name`值为`textField`，注意：不要设置元素的可访问性（isAccessibilityElement）为NO
![image](/images/UiAutomation/label.png)
或者用代码设置

```
self.aView.accessibilityLabel = @"textField";
```

那么就可以通过下面这种方式获取

```
var textField = window.textFields()['textField'];
```

关于元素的可执行方法，比如`tap()`，可以查看苹果的[官方文档](https://developer.apple.com/library/ios/documentation/DeveloperTools/Reference/UIAutomationRef/)。

**以及元素数组包括：**

```
1. buttons()
2. images()
3. scrollViews()
4. textFields()
5. webViews()
6. segmentedControls()
7. sliders()
8. staticTexts()
9. switches()
10. tabBar()
11. tableViews()
12. textViews()
13. toolbar()
14. toolbars()
15. secureTextFields() // 加密的UITextField
...
```

苹果另外也提供一个辅助检查功能工具，可以方便查看元素的信息<br>打开设置（Settings）-- 通用（General）-- 辅助功能（Accessibility）-- 辅助功能检查器（Accessibility Inspector）

![image](/images/UiAutomation/accessibility.png)


## 操作脚本录制


在④中创建一个新的脚本，在⑥中点击中间的红色按钮，代表开始录制用户操作并转换为JS代码，点击右侧的按钮，停止录制，点击左侧按钮，执行脚本。这时候你肯定在想，既然有这功能，还需要写什么脚本，真是这样吗？<br>下面录制的一个同样的操作：点击 A 元素——>在键盘上输入112——>点击 B 元素

```
var target = UIATarget.localTarget();
target.frontMostApp().mainWindow().textFields()[0].textFields()[0].tap();
target.frontMostApp().keyboard().typeString("112");
target.frontMostApp().navigationBar().buttons()["Button"].tap();
```

这种方式生成的代码会有个问题，在各个操作之间都不会生成停顿代码，也就是`target.delay();`。如果对于那种夸页面的复杂操作，这种方式录制的脚本，在重新执行的时候有可能会报错。

举个例子：点击一个按钮后`present`到另一个页面，然后在另一个界面上点击上面的textfield，那么大概的脚本应该是这样：

```
var target = UIATarget.localTarget();
target.frontMostApp().mainWindow().buttons()[0].tap();
target.frontMostApp().mainWindow().textFields()[0].tap();
```

当执行到`target.frontMostApp().mainWindow().textFields()[0].tap();`这时候界面其实还处于前一个，那么获取`textfield`必然会有问题，所以如果遇到此类问题，不妨各个操作之间加入`target.delay();`试试。<br>另外，缺少逻辑判断，比如当情况一点击这个按钮，情况二点击另一个。

所以想完全依靠这种方式，偷懒不用写脚本，基本上行不通。

## 常用操作

苹果[官方API文档](https://developer.apple.com/library/ios/documentation/DeveloperTools/Reference/UIAutomationRef/)

* 屏幕点击

```
// 单击
UIATarget.localTarget().tap({x:100, y:200});
// 双击
UIATarget.localTarget().doubleTap({x:100, y:200});
// 双指点击
UIATarget.localTarget().twoFingerTap({x:100, y:200});
```

* 缩放

```
// 放大
UIATarget.localTarget().pinchOpenFromToForDuration({x:20, y:200},{x:300, y:200},2);
// 缩小
UIATarget.localTarget().pinchCloseFromToForDuration({x:20, y:200}, {x:300, y:200},2);
```

* 拖拽与划动

```
// 拖拽
UIATarget.localTarget().pinchOpenFromToForDuration({x:20, y:200},{x:300, y:200},2);
// 划动
UIATarget.localTarget().pinchCloseFromToForDuration({x:20, y:200}, {x:300, y:200},2);
```

* 打印

```
UIALogger.logStart("xxx");
UIALogger.logFail("xxx");
UIALogger.logDebug("xxx");
```

## 命令行执行

使用命令行的原因是，Instruments工具跑测试实在太慢了，命令如下

```
instruments -t /Applications/Xcode.app/Contents/Applications/Instruments.app/Contents/PlugIns/AutomationInstrument.xrplugin/Contents/Resources/Automation.tracetemplate -w "iPhone 5s (8.3 Simulator)" /Users/xxxx/Library/Developer/CoreSimulator/Devices/A35F991E-425E-4F41-B76C-B7C176A06C36/data/Containers/Data/Application/324E3D2F-A0BC-4E96-97DF-97E791AB10A8/xxxx.app -e UIASCRIPT /Users/xxxx/Desktop/untitled.js -e UIARESULTSPATH /Users/xxxx/Desktop/tmp/
```

需要自行修改的部分

```
// 测试设备
-w "iPhone 5s (8.3 Simulator)"
```

```
// 项目目录，如果目录中没有xxxx.app也没关系
/Users/xxxx/Library/Developer/CoreSimulator/Devices/A35F991E-425E-4F41-B76C-B7C176A06C36/data/Containers/Data/Application/324E3D2F-A0BC-4E96-97DF-97E791AB10A8/xxxx.app
```

```
// 脚本目录
-e UIASCRIPT /Users/xxxx/Desktop/untitled.js
```

```
// 测试报告输出目录
-e UIARESULTSPATH /Users/xxxx/Desktop/tmp/
```

## 最后

之所以说 UiAutomation 并没有想象中那么的完美的原因有下面几点：

1. JS脚本调试较为麻烦
2. UiAutomation 目前存在不少bug，比如手动设置`accessibilityLabel`无效、莫名其妙的报错，再跑一次又好了
3. 不够完善，比如不能判断一个button是否可用（对应enabled属性），UIAlterView无法手动处理（网上提供的方法都无效），多份JS脚本，不能设置完成之后自动跑下一份
4. 输出的测试报告比较简单，只有简单的log和截图，截图的规则也不太清楚（可能有提供截图的API）

另外推荐一个开源的测试库[tuneup_js](https://github.com/alexvollmer/tuneup_js)，以及[文档](http://www.tuneupjs.org/installation.html)。

#### 注意：提交项目的时候若遇到Error itms-90035错误请尝试去掉[tuneup_js](https://github.com/alexvollmer/tuneup_js)库。

__参考：[知平软件](http://www.cnblogs.com/vowei/archive/2012/08/10/2631949.html#3105924)__
