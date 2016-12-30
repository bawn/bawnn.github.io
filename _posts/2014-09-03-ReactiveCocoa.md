---
layout: post
title: "ReactiveCocoa"
date: 2015-03-26
comments: true
categories: iOS
tags: [ReactiveCocoa]
keywords: ReactiveCocoa 函数式编程 响应式编程
description: 关于ReactiveCocoa一些使用总结
---
很早就开始关注[ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)了，前段时间决定把它加入到项目中，理由如下:

* 熟悉响应式编程(函数式编程)模式的好时机
* 整个框架经过0.0.1版本到2.3.1的迭代已经相对成熟
* MVVM模式的尝试


从开始了解`ReactiveCocoa`到现在，有时候总感觉没有完全利用好，比如

```
@weakify(self);
[[self.nextButton rac_signalForControlEvents:UIControlEventTouchUpInside] subscribeNext:^(id x) {
     @strongify(self);
     [self performSegueWithIdentifier:@"Captcha" sender:nil];
}];
```
这样单一的信号传递和部署，没有和其他信号有任何的联系，总感觉和`action-target`模式没什么区别，觉得这偏离了响应式编程的本意，或许这样说有点极端。

## 倒计时功能

先说说很常用的验证码倒计时功能，用RAC来实现几乎一气呵成
![验证码](http://7ls0py.com1.z0.glb.clouddn.com/ReactiveCocoa_1.jpg)
点击按钮可以重新开始倒计时

```
  @weakify(self);
  RACSignal *timeSignal = [[[[[RACSignal interval:1.0f onScheduler:[RACScheduler mainThreadScheduler]] take:numberLimit] startWith:@(1)] map:^id(NSDate *date) {
      @strongify(self);
      if (number == 0) {
          [self.timeButton setTitle:@"重新发送" forState:UIControlStateNormal];
          return @YES;
      }
      else{
          self.timeButton.titleLabel.text = [NSString stringWithFormat:@"%d", number--];
          return @NO;
      }
  }] takeUntil:self.rac_willDeallocSignal];
  self.timeButton.rac_command = [[RACCommand alloc]initWithEnabled:timeSignal signalBlock:^RACSignal *(id input) {
      number = numberLimit;
      return timeSignal;
  }];
```

* 利用 `+ (RACSignal *)interval:(NSTimeInterval)interval onScheduler:(RACScheduler *)scheduler` 方法返回一个在主线程上间隔一秒执行一次的signal ，RAC内部是利用 `dispatch_source_set_timer` `dispatch_source_set_event_handler` `dispatch_resume` 这一系列的GCD方法来实现的。注意这个signal 所sendnext的值是当前日期的NSDate对象，不过这个值对于整个功能是没有用处的。
* `- (instancetype)take:(NSUInteger)count` 取前numberLimit次sendnext的值，相当于我们需要倒计时多久
* 通过 `- (instancetype)startWith:(id)value` 修改第一次sendnext的值，并且立即sendnext这个值。理论上对于sendnext的值是不需要处理的，原因是`+ (RACSignal *)interval:(NSTimeInterval)interval onScheduler:(RACScheduler *)scheduler`所返回的signal会停顿1秒才执行，利用startWith来立即开始倒计时罢了，所以说`startWith:`的参数值是什么不重要
* 利用 `- (instancetype)map:(id (^)(id value))block` 修改sendnext最终的值返回@YES或者@NO
* `- (RACSignal *)takeUntil:(RACSignal *)signalTrigger` 意思是当这个VC的即将dealloc的时候停止倒计时，RAC内部其实就是sendCompleted。
* 最后把timeSignal和timeButton.rac_command绑定判断按钮是否可用，并且启动timeSignal。按钮被点击的时候恢复number的值，`return buttonSignal`来重新启动倒计时timeSignal

____

## 代替委托协议模式

如果RAC利用得当，几乎可以抛弃写自定义的委托协议。想象一下：当aVC `presentViewController` 到bVC，然后 `dismissViewControllerAnimated` bVC的时候需要在aVC中做一些事情，这时候通常用委托协议可以解决这种问题，比较麻烦。利用RAC实现就显得非常简洁明了

其中的关键方法是`- (RACSignal *)rac_signalForSelector:(SEL)selector`，基本使用是这样的

```
[[self rac_signalForSelector:@selector(dismiss:)] subscribeNext:^(id x) {
	NSLog(@"%s", __func__);
}];

```

```
- (IBAction)dismiss:(id)sender{
    [self dismissViewControllerAnimated:YES completion:NULL];
}
```
当`dismiss:`方法被执行后信号就会被部署


信号什么时候部署可以由我们来决定，既然需要在aVC中处理一些事，那么就应该想办法在aVC中来部署信号。这时候就需要把信号作为bVC的属性

>bVC.h

```
#import <UIKit/UIKit.h>
@interface RACDelegateViewController : UIViewController
@property (nonatomic, strong) RACSignal *delegateSignal;
@end
```

部署信号
>aVC.m

```
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender{
    if ([segue.destinationViewController isKindOfClass:[UINavigationController class]]) {
        UINavigationController *nav = (UINavigationController *)segue.destinationViewController;
        RACDelegateViewController *delegateVC = (RACDelegateViewController *)nav.topViewController;
        [delegateVC.delegateSignal subscribeNext:^(id x) {
            NSLog(@"%@", x);
        }];
    }
}
```
还有最后一步，生成`delegateSignal`
>bVC.m

```
- (void)awakeFromNib{
    [super awakeFromNib];
    self.delegateSignal = [self rac_signalForSelector:@selector(dismiss:)];
}
```

注意：千万不要在`- (void)viewDidLoad;`去生成`delegateSignal`，这会导致在执行`subscribeNext`的时候`delegateSignal`还是空的。

还不够完美对不对，aVC中接收的数据一直都是UIBarButtonItem对象，想把bVC中的数据传递到aVC中应该如何实现。还必须在信号上做一些调整，利用 `- (RACSignal *)then:(RACSignal * (^)(void))block;` 这个方法功能是忽略接收者所有的`next`，直到信号`complete`返回一个新的RACSignal。看用法：

```
- (void)awakeFromNib{
    [super awakeFromNib];
    @weakify(self);
    self.delegateSignal = [[self rac_signalForSelector:@selector(dismiss:)] then:^RACSignal *{
        @strongify(self);
        return [RACSignal return:self.array];
    }];
}
```

当`[self rac_signalForSelector:@selector(dismiss:)]`信号`sendComplete`的时候，执行一个block，这个block必须返回一个不为空的新的RACSignal。当然还有其他方法，比如：`map`这个信号：

```
- (void)awakeFromNib{
    [super awakeFromNib];
    @weakify(self);
    self.delegateSignal = [[self rac_signalForSelector:@selector(dismiss:)] map:^id(id value) {
        @strongify(self);
        return self.array;
    }];
}
```

对于需要监听协议方法的时候可以使用 `- (RACSignal *)rac_signalForSelector:(SEL)selector fromProtocol:(Protocol *)protocol ` 具体可以看我的[笔记共享](https://www.evernote.com/shard/s147/sh/a01d60f5-36d3-43ad-9715-23fb19e4ae78/c4a9520d5d02767eea89a8c4cf005b4a)

____

## 自动处理数据更新

__注意：此功能只适合与`row`和`section`不变的UITableView以及UICollectionView__

前段时间看了objc中国的两篇文章更轻量的 [View Controllers](http://objccn.io/issue-1-1/)和整洁的 [Table View](http://objccn.io/issue-1-1/) 代码，也试着把 ReactiveCocoa 结合进 UITableView 中，下面是我做的一些尝试

* __KVO数据源数组__

```
- (id)initWithCellIdentifier:(NSString *)aCellIdentifier
          configureCellBlock:(CellConfigureBlock)aConfigureCellBlock{
    self = [super init];
    if (self) {
        _cellIdentifier = aCellIdentifier;
        _configureCellBlock = [aConfigureCellBlock copy];
        _signal = RACObserve(self, dataSource);
    }
    return self;
}
```

* __map数据源得到单一的数据模型__

```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:_cellIdentifier
                                                            forIndexPath:indexPath];
    if (self.configureCellBlock) {
        @weakify(self);
        self.configureCellBlock(cell, [self.signal map:^id(NSArray *array) {
            @strongify(self);
            return [self itemAtIndexPath:indexPath];
        }]);
    }
    return cell;
}
```

* __VC中的用法__

```
self.model = [[DataSourceGeneralModel alloc] initWithCellIdentifier:CelIdentifier
                                                 configureCellBlock:^(TableViewCell *cell, RACSignal *signal) {
        [cell configureCellWithSignal:signal];
    }];
```

* __cell中配置__

```
- (void)configureCellWithSignal:(RACSignal *)signal{
    @weakify(self);
    [signal subscribeNext:^(Model *model) {
        @strongify(self);
        self.titleLabel.text = model.title;
        self.detailLabel.text = model.detail;
    }];
}
```

如果更新数据源数组，UITableView也会得到相应的更新，不用调用`[self.tableView reloadData]`，像这样更新即可

```
self.model.dataSource = @[model1];
```

#### 以上例子可以到[github](https://github.com/bawn/RAC-Demo)下载

____

## 代替dispatch_group功能

**传统的`dispatch_group`创建使用方式**

```

// 创建一个dispatch_group对象
static dispatch_group_t home_request_operation_completion_group() {
    static dispatch_group_t http_request_operation_completion_group;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        http_request_operation_completion_group = dispatch_group_create();
    });
    return http_request_operation_completion_group;
}
// .......
- (void)viewDidLoad{
    [self updateData];
}
- (void)updateProfit{
// 加入一个异步方法
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_group_async(home_request_operation_completion_group(), queue, ^{
    dispatch_group_enter(home_request_operation_completion_group());
        [self update1];
    });
// 加入另一个异步方法
    dispatch_group_async(home_request_operation_completion_group(), queue, ^{
    dispatch_group_enter(home_request_operation_completion_group());
        [self update2];
    });
// 当两个异步方法都执行完的时候调用停止下拉刷新
    dispatch_group_notify(home_request_operation_completion_group(), dispatch_get_main_queue(), ^{
        if ([self.tableView isHeaderRefreshing]) {
            [self.tableView headerEndRefreshing];
        });
};
```

**RAC代替dispatch_group**

```
- (void)updateProfit{
    [[RACSignal zip:@[[self update1], [self update2]]] subscribeNext:^(RACTuple *x) {
        @strongify(self);
        if ([self.tableView isHeaderRefreshing]) {
            [self.tableView headerEndRefreshing];
        });
    }];
```

当然这里需要改写`[self update1]`和`[self update2]`返回RACSignal对象。

`+ (instancetype)zip:(id<NSFastEnumeration>)streams`作用当绑定的多个signal都sendNext后再subscribeNext。

另外关于`combineLatest`和`zip`区别可以看我的[笔记](https://www.evernote.com/l/AJMEYyw-2oxEQ7Pv0uxJ4faAgCyhXgjRnzs)

____

## RAC相关博客

* [唐巧](http://blog.devtang.com/blog/2014/02/11/reactivecocoa-introduction/)
* [无网不剩](http://blog.leezhong.com/ios/2013/06/19/frp-reactivecocoa.html)
* [阿峰](http://hufeng825.github.io/2013/10/13/ios31/)
* [随缘](http://www.cnblogs.com/sunnyxx/category/551817.html)
* [sjpsega](http://sjpsega.com/blog/2014/02/11/yi--ios-7-best-practices-part-1/)
* [teehanlax](http://www.teehanlax.com/blog/reactivecocoa/)
