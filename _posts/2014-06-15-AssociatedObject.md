---
layout: post
title: "为类关联对象"
date: 2014-06-15
comments: true
categories: iOS
tags: [iOS]
published: true
keywords: objc_setAssociatedObject objc_getAssociatedObject
description: 关联对象
---

相信大家多多少少都见过`objc_setAssociatedObject`和`objc_getAssociatedObject`这两个方法。

它们到底有什么用呢？如果只是说关联对象，这好像也没体现出来具体的作用，下面我来说一个比较实际作用。
在使用`UIAlertView`的时候有没有觉得很麻烦，点击按钮的逻辑处理必须写在协议方法中，如果一个VC中存在多个`UIAlertView`就会更复杂，这样就必须在协议方法中判断传入`alertView`对象。我们可以利用上述两个方法一劳永逸的解决类似问题

最终得到创建和显示一个`UIAlertView`将会是这样子的

```
	[self showAlertViewWithTitle:@"是否确定结案"
                             message:nil
                   cancelButtonTitle:@"取消"
                   otherButtonTitles:@[@"退回", @"接收"]
                            onCancel:^{  
            NSLog(@"取消按钮点击");
        }onOther:^(NSInteger index) {
            if (index == 0) {
                NSLog(@"其他第一个按钮点击");
            }
            else{
                NSLog(@"其他第二个按钮点击");
            }
        }];
```

可以看到在创建`UIAlertView`的时候已经把每个按钮要处理的逻辑都写好了。

下面开始动手写这个扩展，我们创建一个名叫`Addons`的`UIViewController`扩展

>UIViewController+Addons.h

```
#import <Foundation/Foundation.h>
typedef void(^HandleBlock)(void);
typedef void(^OtherHandleBlock)(NSInteger index);
@interface UIViewController(Addons)<UIAlertViewDelegate>
/**
 *  显示alterView
 *
 *  @param title             title
 *  @param message           message
 *  @param cancelButtonTitle 取消按钮的title
 *  @param otherButtonTitles otherButton的title数组
 *  @param cancelBlock       取消按钮执行的block
 *  @param otherBlock        other按钮执行的block
 */
- (void)showAlertViewWithTitle:(NSString *) title
                      message:(NSString *) message
            cancelButtonTitle:(NSString *) cancelButtonTitle
            otherButtonTitles:(NSArray *) otherButtonTitles
                     onCancel:(CancelBlock) cancelBlock
                      onOther:(OtherHandleBlock) otherBlock;
```

`typedef`两个`block`，第一个用于点击取消按钮执行，第二个用于点击其他按钮执行，因为其他按钮可能有多个，所以加入了`index`参数来判断点击的是哪个按钮。

>UIViewController+Addons.m

```
#import "UIViewController+Addons.h"
#import <objc/runtime.h>
static char kVCActionHandleAlertCancelBlockKey;
static char kVCActionHandleAlertOtherBlockKey;
@implementation UIViewController(Addons)
-(void)showAlertViewWithTitle:(NSString *)title
                      message:(NSString *)message
            cancelButtonTitle:(NSString *)cancelButtonTitle
            otherButtonTitles:(NSArray *)otherButtonTitles
                     onCancel:(CancelBlock)cancelBlock
                      onOther:(OtherHandleBlock)otherBlock{
     UIAlertView *alertView = [[UIAlertView alloc]initWithTitle:title
                                                       message:message
                                                      delegate:self
                      	                   cancelButtonTitle:cancelButtonTitle
                                             otherButtonTitles:nil];
    for(NSString *buttonTitle in otherButtonTitles){
        [alertView addButtonWithTitle:buttonTitle];
    }
    if (cancelBlock) {
        objc_setAssociatedObject(self, &kVCActionHandleAlertCancelBlockKey, cancelBlock, OBJC_ASSOCIATION_COPY_NONATOMIC);
    }
    if (otherBlock) {
        objc_setAssociatedObject(self, &kVCActionHandleAlertOtherBlockKey, otherBlock, OBJC_ASSOCIATION_COPY_NONATOMIC);
    }
    [alertView show];
}
```

```
-(void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex{
	if(buttonIndex == alertView.cancelButtonIndex){
        CancelBlock cancelBlock = objc_getAssociatedObject(self, &kVCActionHandleAlertCancelBlockKey);
        if(cancelBlock){
            cancelBlock();
        }
    }
    else{
        OtherHandleBlock otherBlock = objc_getAssociatedObject(self, &kVCActionHandleAlertOtherBlockKey);
        if(otherBlock){
            otherBlock(buttonIndex - 1);
        }
    }
}
```

记得导入`<objc/runtime.h>`头文件，创建`UIAlertView`，然后把传入的两个`block`关联到`UIViewController`对象上，也就是这步

```
 if (cancelBlock) {
        objc_setAssociatedObject(self, &kVCActionHandleAlertCancelBlockKey, cancelBlock, OBJC_ASSOCIATION_COPY_NONATOMIC);
    }
    if (otherBlock) {
        objc_setAssociatedObject(self, &kVCActionHandleAlertOtherBlockKey, otherBlock, OBJC_ASSOCIATION_COPY_NONATOMIC);
    }

```

需要注意的是这里关联的规则最好用`OBJC_ASSOCIATION_COPY_NONATOMIC`，以便让ARC来管理这个`block`。后面的协议方法也就是把`block`从`self`中取出来执行。

当然还有`UIActionSheet`甚至`UIImagePickerController`也可以这样来扩展
