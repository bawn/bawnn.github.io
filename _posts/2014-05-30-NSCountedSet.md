---
layout: post
title: "NSCountedSet"
date: 2014-05-30
comments: true
categories: iOS
tags: [iOS]
published: true
keywords: NSCountedSet
description: 统计数组中重复对象的个数
---

之前群里有人讨论计算老虎机中奖等级的问题：老虎机有四列，每列四个图案，如果四个相同就是等级1，三个相同就是等级2，以此类推。

说是用`if else`太麻烦，有没有什么比较快捷高效的方法，我首先想到的是用`KVC`中剔除重复数据的办法。比如:

```
NSArray *array = @[@2, @2, @2, @1];
NSArray *result = [array valueForKeyPath:@"@distinctUnionOfObjects.self"];
NSLog(@"%@", result);
```

得到数组的个数为2，刚好是等级2.看起来很合理，如果数组是`NSArray *array = @[@2, @1, @2, @1];`这时候计算出来的数组个数是也是2，但是正确结果应该是等级3。后来我想起一个几乎很少用到的一个类:`NSCountedSet`

```
NSArray *array = @[@1, @2, @2, @1];
NSCountedSet *set = [[NSCountedSet alloc]initWithArray:array];

[set enumerateObjectsUsingBlock:^(id obj, BOOL *stop) {
	NSLog(@"%@ => %d", obj, [set countForObject:obj]);
}];
```

打印

```
2014-05-29 00:35:22.741 CoreData[3235:60b] 1 => 2
2014-05-29 00:35:22.742 CoreData[3235:60b] 2 => 2
```

可能你会发现这个类的父类是`NSMutableSet`。纳尼？不是说`NSMutableSet`是不可以储存重复对象的吗。其实`NSCountedSet`也是不能储存重复的对象的，查看Apple文档中对这个类的描述有这么一句：

>Each distinct object inserted into an NSCountedSet object has a counter associated with it.
插入NSCountedSet对象的每个不同的对象都有一个与之相关的计数器(google翻译)

也就是说如果遇到重复对象的加入，这个对象的计数器就会+1。所以可以到这个类有个名叫`- (NSUInteger)countForObject:(id)object;`的方法来统计重复对象的个数。
