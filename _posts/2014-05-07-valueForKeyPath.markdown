---
layout: post
title: "valueForKeyPath"
date: 2014-05-07
comments: true
categories: iOS
tags: [KVC, valueForKeyPath]
keywords: iOS KVC valueForKeyPath
description: valueForKeyPath方法的强大之处
---
或许大家在平常的开发中`- (id)valueForKeyPath:(NSString *)keyPath`方法用的不多

但是这个方法非常强大，举个例子:

```objective-c
NSArray *array = @[@"name", @"w", @"aa", @"jimsa"];
NSLog(@"%@", [array valueForKeyPath:@"uppercaseString"]);
```

输出

```
(
    NAME,
    W,
    AA,
    JIMSA
)
```

相当于数组中的每个成员执行了`uppercaseString`方法，然后把返回的对象组成一个新数组返回。既然可以用`uppercaseString`方法，那么NSString的其他方法也可以，比如

```
[array valueForKeyPath:@"length"]
```

返回每个字符串长度的组成的数组。只要你能想到的成员实例方法都可以这么用。

如果你觉得这个方法就这么点功能，那就错了。还是举具体的例子

###  对NSNumber数组快速计算数组求和、平均数、最大值、最小值

```
NSArray *array = @[@1, @2, @3, @4, @10];
NSNumber *sum = [array valueForKeyPath:@"@sum.self"];
NSNumber *avg = [array valueForKeyPath:@"@avg.self"];
NSNumber *max = [array valueForKeyPath:@"@max.self"];
NSNumber *min = [array valueForKeyPath:@"@min.self"];
```

或者指定输出类型

```
NSNumber *sum = [array valueForKeyPath:@"@sum.floatValue"];
NSNumber *avg = [array valueForKeyPath:@"@avg.floatValue"];
NSNumber *max = [array valueForKeyPath:@"@max.floatValue"];
NSNumber *min = [array valueForKeyPath:@"@min.floatValue"];
```

这里有一点需要注意，`@sum` 和 `@avg`的集合操作，会先把对象先转换为 double 值，
最后输出包含 double 值的 NSNumber 对象，所以千万不要放这样的错误：

```
NSArray *arr = @[@(0),@(10),@(40)];
NSInteger avg = [[arr valueForKeyPath:@"@avg.self"] integerValue];
NSLog(@"---%ld----",avg);
```
integerValue 操作后直接返回 0 了



### 剔除重复数据

```objective-c
NSArray *array = @[@"name", @"w", @"aa", @"jimsa", @"aa"];
NSLog(@"%@", [array valueForKeyPath:@"@distinctUnionOfObjects.self"]);
```
```
(
    name,
    w,
    jimsa,
    aa
)
```

###  快速取值

```objective-c
    NSArray *array = @[@{@"name" : @"cookeee",
                         @"code" : @1},
                       @{@"name" : @"sswwre",
                         @"code" : @2}];
    NSLog(@"%@", [array valueForKeyPath:@"name"]);
```

直接得到字典中`name`key对应的值组成的数组，显然比循环取值再加入到新数组中方便快捷

```
(
    cookeee,
    jim,
    jim,
    jbos
)
```

###  嵌套使用

对`name`字段对应的值剔除重复数据再取值

```objective-c

NSArray *array = @[@{@"name" : @"cookeee",@"code" : @1},
                    @{@"name": @"jim",@"code" : @2},
                    @{@"name": @"jim",@"code" : @1},
                    @{@"name": @"jbos",@"code" : @1}];

NSLog(@"%@", [array valueForKeyPath:@"@distinctUnionOfObjects.name"]);
```

```
(
    cookeee,
    jim,
    jbos
)
```

对`number`字段对应的值取最大值

```
NSArray *array = @[@{@"number" : @1, @"title" : @""},
                   @{@"number" : @2, @"title" : @""},
                   @{@"number" : @20, @"title" : @""},
                   @{@"number" : @4, @"title" : @""},
                   @{@"number" : @5, @"title" : @""}
];
NSNumber *max = [array valueForKeyPath:@"@max.number"];
```

###  改变UITextfiedl的placeholder的颜色

```
    [searchField setValue:[UIColor whiteColor] forKeyPath:@"_placeholderLabel.textColor"];
```

比起重写`- (void)drawPlaceholderInRect:(CGRect)rect;`要方便很多



### 实用例子

```objective-c
NSArray *array = @[
                   @[@1, @2],
                   @[@1, @2, @3, @4],
                   @[@1, @2, @3]
                   ];
NSNumber *count = [[array valueForKeyPath:@"@unionOfObjects.@count"] valueForKeyPath:@"@sum.self"];
```

计算嵌套数组的总个数，比如这里返回的是 9