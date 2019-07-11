---
layout: post
title: "关于埋点"
date: 2019-01-11
comments: true
categories: [iOS]
tags: [埋点]
keywords: [埋点]
publish: true
description: 埋点
---

这篇文章主要介绍在本人在公司项目上埋点实现的一些心得，在项目早期我们完全依赖于第三方的无痕埋点技术，客户端开发人员只需要做一些简单的工作就能满足 BI 部门对数据的需求。但随着业务增长，对数据的准确性和精细化的要求越来越高，之后不得不转向手动埋点（基于[神策](https://www.sensorsdata.cn)）。

很多时候我们需要根据具体业务来选择适合的埋点方案，在 [火球买手](https://itunes.apple.com/cn/app/%E7%81%AB%E7%90%83%E4%B9%B0%E6%89%8B-%E5%B8%AE%E4%BD%A0%E9%80%89%E5%A5%BD%E8%B4%A7/id1070842761?mt=8) 这个项目上 BI 部门对埋点数据要求可以总结为一句话：『从哪里来到哪里去』，比如在 Timeline 中点击一篇文章进入详情页，那么 Timeline 就是『从哪里来』，详情页就是『到哪里去』，当然『从哪里来』不只需要一个维度定位，有时候需要两三个维度才能定位。

下面举一些具体的例子，首先『频道主页』是项目中比较常见的页面，它对应的 Model 是 Channel，然后当任何点击进入的频道主页的事件触发后都需要上报以下数据

```
{
    module_name
    page_name
    channel_name
    channel_id
}
```

`page_name`指的是当前的 ViewController 名称，`module_name`主要用于区分同一个页面内的不同入口，这样子就能确定『从哪里来』，`channel_name` 和 `channel_id` 数据来自于 Channel，至于『到哪里去』这里就用埋点的 key 来表明，比如是 ChannelClick。

频道主页在 APP 中入口众多，即使在不考虑埋点的情况下，一个统一的入口也是必要的

```swift
extension UIViewController {
    func pushToChannelDetailController(_ id: String?) {
        // ...
    }
}
```

显然这样的方法根本无法满足埋点上的需求，改造一下：

```swift
extension UIViewController {
    func pushToChannelDetailController(_ model: Channel?) {
        // ...
        let value = [
            "module_name" : model.module_name,
            "channel_id" : model._id,
            "channel_name": model.name,
            "page_name": self.pageName
        ]
        SensorsAnalyticsSDK.sharedInstance()?.track(key, withProperties: value)
    }
}
```

大多数情况下入口函数只接受一个具象参数是行不通的，因为随着项目的开发业务的迭代总有一些其他的模型被加入，它们同样带有能够跳转至频道主页的 id 属性，这个后面会提到。然后是 pageName：

```swift
extension UIViewController {
    var pageName: String {
        switch self {
			case is ChannelDetailController:
            	return "频道主页"
        }
    }
}
```

最后在具体的跳转处设置 module_name 

```swift
@objc func buttonAction(_ sender: Any) {
    model.module_name = "Header"
    pushToChannelDetailController(model)
}
```

整体看下来虽然可以应付点击进入频道主页的埋点，但是还是存在以下问题

#### 入口函数不够抽象

在实际开发中接收不同的数据模型跳转到同一个页面的情况应该不少见，并且入口函数也是因为埋点把参数从相对抽象的 String 替换成了具象的 Channel，所以抽象 model 是首先要做的。无论接受什么类型参数，实际的跳转只需要用到 id 这一个字段，那么让一个只有带有 id 属性的 protocol 去约束模型再合适不过了。

```swift
protocol CommonModelType {
    var id: String { get }
}
```

然后让 Channle 遵守这个协议，利用 extension 是为了看起来更解耦

```swift
extension Channel: CommonModelType{}
```

这时候入口函数就是这样

```swift
func pushToChannellDetail(_ model: CommonModelType?)
```

可以接受任何有 id 属性的模型，为了更抽象，甚至可以让 String 也遵守这个协议

```swift
extension String: CommonModelType {
    var id: String {
        return self
    }
}
```



#### 数据提供方式不够优雅

埋点数据除了 pageName 不属于 model 以外，其他都属于 model 本身的属性（module_name 属于额外添加），所以和参数一样，同样用 protocol 约束 Channel ，让 Channel 拥有一个直接用于提供数据的属性。

```swift
protocol AnalyticsModelType {
    var analytics: [String: Any] { get }
}
```

让 Channel 同时遵守这两个协议，并且添加 analytics 属性

```swift
extension Channel: CommonModelType, AnalyticsModelType {
    var analytics: [String : Any] {
        let value = [
            "module_name" : module_name,
            "channel_id" : id,
            "channel_name": name
        ]
        return value
    }
}
```

最后完整的入口函数是这样的

```swift
extension UIViewController {

    func pushToChannellDetail(_ model: CommonModelType?) {
        guard let model = model else {
            return
        }
        let viewController = ChannelDetailViewController()
        viewController.id = model.id
        navigationController?.pushViewController(viewController, animated: true)
        
        guard let value = model as? AnalyticsModelType else {
            return
        }
        var properties = value.analytics
        properties["page_name"] = pageName
        SensorsAnalyticsSDK.sharedInstance.track(key: "ChannelClick", properties: properties)
    }
}
```



## 未完待续