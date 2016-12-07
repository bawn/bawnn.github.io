---
layout: post
title: "使用TestFlight"
date: 2015-12-25
comments: true
categories: iOS
tags: [iOS, TestFlight]
keywords: TestFlight iOS
publish: true
description: 使用TestFlight
---

TestFlight 可以很方便的给测试人员发放 APP 测试，这篇博客主要介绍如何使用 TestFlight 的内部测试。

#### 上传APP

首先需要上传 APP 到 itunesconnect，这个和发布 APP 到 itunesconnect 审核上线没什么区别，所以需要发布证书和Provisioning，但并不需要是 Release，也可以是Debug，毕竟我们只需要测试。

#### 发送邀请

进入 itunesconnect 选择 APP，点击上方的 TestFlight 菜单，选择`内部测试`，然后点击右侧的`选择版本进行测试`选择一个你要的版本进行测试。![image](/images/TestFlight/TestFlight1.jpg)

当选择完版本后，点击下方`内部测试员 `右侧的 ＋ 号，但是会发现只能添加当前开发者的邮箱账号，添加后再次点击 ＋ 号，按照提示跳转到`用户和职能`板块，

![image](/images/TestFlight/TestFlight2.png)

这时候顶部有三个菜单分别是`iTunes Connect 用户` `TestFlight Beta 版测试员` `沙箱技术测试员`，一般凭自觉，在都会认为在`TestFlight Beta 版测试员`菜单下的`内部测试员`添加测试账号较为合理，但是这里什么都做不了。

我们需要点击` iTunes Connect 用户`菜单在这里添加用户，点击下方用户右侧的  ＋ 号，添加用户（职能随便选择），被添加的用户会收到一份邮件，点击邮件里面的确认链接即可，添加完会在这里显示用户。

![image](/images/TestFlight/TestFlight3.png)

因为在`iTunes Connect 用户`添加过的用户才可以被邀请到测试APP，所以需要返回到`我的APP`板块，选择你要测试的APP，点击上方的`TestFlight`菜单，这时候`内部测试员`右侧的加号就可以添加刚才所添加的用户了，选择完测试人员后点击完成，别忘记点击`储存`。

![image](/images/TestFlight/TestFlight4.jpg)

#### 验证

点击储存完后，每个人还会收到一封邀请测试的邮件，先别急着验证邮件，在这之前还需要一个步骤：在手机上安装[TestFlight](https://itunes.apple.com/us/app/testflight/id899247664?mt=8)，并登陆。完成之后如果在手机上打开验证邮件并点击里面的`Start Testing`就可以跳转到`TestFlight`APP上安装了。

如果是在 PC 端验证的话，你会收到一个验证码，这时候就需要在 `TestFlight`内点击`Redeem`按钮 输入验证码就可以安装APP了。
![image](http://7ls0py.com1.z0.glb.clouddn.com/thumb_IMG_0503_1024.jpg)

#### 测试内容

简单点说就是测试人员需要知道这次更新修改了什么，以便更好的进行测试。在 TestFlight 菜单下，你会看到有个`构建版本XXX`![image](/images/TestFlight/TestFlight5.jpg)点击就会进入填写测试内容、App 描述的界面，填写完别忘记点击储存，这时候回到 TestFlight APP中的点击进入刚才的测试APP详情页就可以看到`What to Test`里面的测试内容。



### 最后

每次上传完 APP 后，会大概延迟十几分钟客户端的才会收到 TestFlight 的更新通知，也就是说不用手动选择新版本进行测试。
