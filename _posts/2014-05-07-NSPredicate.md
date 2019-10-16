---
layout: post
title: "NSPredicate"
date: 2014-05-07
comments: true
categories: iOS
tags: [iOS]
keywords: NSPredicate 筛选
description: NSPredicate数据筛选
---
大家在平常的开发过程中多多少少都会接触到数据筛选，那势必会用到`NSPredicate`

这个类和我上一篇博文中提到的[valueForKeyPath](http://bawn.github.io/2014/05/07/valueForKeyPath/)一样很强大。它的使用主要集中在两个方法中:

NSArray

```objective-c
- (NSArray *)filteredArrayUsingPredicate:(NSPredicate *)predicate;
```

NSMutableArray

```objective-c
- (void)filterUsingPredicate:(NSPredicate *)predicate;
```

还有`NSSet`和`NSMutableSet`也可以用这个类筛选。
下面我就来一一介绍这个类的用法，相信大家看完后会和我一样认为这个类真的很强大。

>## 筛选用法

### 利用成员实例方法

* 筛选出长度大于3的字符串

```objective-c
NSArray *array = @[@"jim", @"cook", @"jobs", @"sdevm"];
NSPredicate *pre = [NSPredicate predicateWithFormat:@"length > 3"];
NSLog(@"%@", [array filteredArrayUsingPredicate:pre]);
```

打印

```
(
    cook,
    jobs,
    sdevm
)
```

`lenght`就是对数组成员执行[xxxx lenght]然后判断返回的NSUInteger值是否大于3。

* NSString其他方法比如`integerValue`


```objective-c
NSArray *array = @[@"2", @"3", @"4", @"5"];
NSPredicate *pre = [NSPredicate predicateWithFormat:@"integerValue >= %@", @3];
NSLog(@"%@", [array filteredArrayUsingPredicate:pre]);
```

如果不想用任何实例方法,想筛选成员本身应该怎么做。这时候就可以用`self`来代替

```objective-c
NSPredicate *pre = [NSPredicate predicateWithFormat:@"self CONTAINS %@", @"3"];
```

`CONTAINS`用法后面会讲到


### 扩展到模型

```objective-c
@interface Test : NSObject
@property (nonatomic, strong) NSString *name;
@property (nonatomic, strong) NSNumber *code;
@end
```
```objective-c
Test *test1 = [[Test alloc]init];
test1.name = @"西湖";
test1.code = @1;

Test *test2 = [[Test alloc]init];
test2.name = @"西溪湿地";
test2.code = @2;

Test *test3 = [[Test alloc]init];
test3.name = @"灵隐寺";
test3.code = @3;

NSArray *array = @[test1, test2, test3];
```
筛选出数组成员`[test code]`方法(code属性的get方法)返回值 >= 2的成员。这里的比较运算符同样也可以使用`==`、`!=`、`<=`、`<`。
```objective-c
NSPredicate *pre = [NSPredicate predicateWithFormat:@"code >= %@", @2];
NSLog(@"%@", [array filteredArrayUsingPredicate:pre]);
```

其实`==`不仅可以用来比较NSNumber对象，还可以用来判断NSString对象是否相同。
```objective-c
NSPredicate *pre = [NSPredicate predicateWithFormat:@"name == %@", @"西湖"];
```
筛选出`name`是"西湖"的对象数组。

#### NSString对象的操作

前面提到`==`比较运算符可以起到`- (BOOL)isEqualToString:(NSString *)aString;`方法的效果，来判断字符串是否相同。那么字符串中包含某个字符串应该如何判断呢，在NSPredicate中可以用`CONTAINS`(大小写都可以)来表示包含关系。
```objective-c
 NSPredicate *pre = [NSPredicate predicateWithFormat:@"name CONTAINS %@", @"湖"];
```
当判断的时候需要忽略大小写可以使用`[cd]`
>[c] 忽略大小写
[d] 忽略重音符号
[cd]既不区分大小写，也不区分发音符号。

使用：
```objective-c
NSPredicate *pre = [NSPredicate predicateWithFormat:@"name CONTAINS[cd] %@", @"abc"];
```
再涉及到一些更复杂的查询语句，比如判断字符串以某个字符串开头或者结尾，通配符的使用。

**BEGINSWITH(已某个字符串开头, begins with)**

```objective-c

NSString *targetString = @"h";
NSPredicate *pre = [NSPredicate predicateWithFormat:@"name BEGINSWITH
%@",targetString];

```

**ENDSWITH(已某个字符串结尾, ends with)**

```objective-c
NSString *targetString = @"ing";
NSPredicate *pre = [NSPredicate predicateWithFormat:@"name ENDSWITH %@",targetString];
```

**通配符 LIKE**
>`*` 代表一个或者多个或者是空，`?` 代表一个字符

```objective-c
Test *test1 = [[Test alloc]init];
test1.name = @"absr";
test1.code = @1;

Test *test2 = [[Test alloc]init];
test2.name = @"asb";
test2.code = @2;

Test *test3 = [[Test alloc]init];
test3.name = @"raskj";
test3.code = @3;

NSArray *array = @[test1, test2, test3];
```

```objective-c
NSPredicate *pre = [NSPredicate predicateWithFormat:@"name LIKE %@", @"?b*"];
NSLog(@"%@", [array filteredArrayUsingPredicate:pre]);
```
筛选结果：只有test1符合。同时`like`也可以接受`[cd]`，比如：
```objective-c
NSPredicate *pre = [NSPredicate predicateWithFormat:@"name LIKE[cd] %@", @"?b*"];
```

#### 关系运算,包括了IN、BETWEEN、AND、OR、NOT

**IN(之中)**

```objective-c
NSPredicate *pre = [NSPredicate predicateWithFormat:@"code IN %@", @[@1, @3]];
```

判断code是否@1或者是@2，也就是是否在数组中。

**OR(或,可以用`||`代替)**

`OR`可以用来代替`IN`达到同样的效果，但是`OR`更灵活。

```objective-c
NSPredicate *pre = [NSPredicate predicateWithFormat:@"code == %@ OR code == %@ ", @1, @3];
```
效果和`IN`一样，但是`OR`可以判断不只一个属性

```objective-c
NSPredicate *pred = [NSPredicate predicateWithFormat:@"code == %@ OR name == %@ ", @1, @"asb"];
```

**BETWEEN(之间)**

通常用于判断NSNumber对象

```objective-c
NSPredicate *pred = [NSPredicate predicateWithFormat:@"code BETWEEN {1, 3}"];
```

判断code是否>=1且<=3


**AND(且,可以用&&代替)**

```objective-c
NSPredicate *pred = [NSPredicate predicateWithFormat:@"code >= %@ AND code <=%@", @1, @3];
```

**NOT(非,可以用!代替)**

`NOT`最常见的用法就是从一个数组中剔除另外一个数组的数据，可能有点绕，举个例子就很明朗了。

```objective-c
NSArray *arrayFilter = @[@"abc1", @"abc2"];
NSArray *arrayContent = @[@"a1", @"abc1", @"abc4", @"abc2"];
NSPredicate *thePredicate = [NSPredicate predicateWithFormat:@"NOT (SELF in %@)", arrayFilter];
NSLog(@"%@",[arrayContent filteredArrayUsingPredicate:thePredicate]);
```

打印

```
(
    a1,
    abc4
)
```
比起循环比较再加到新数组中，简单的不止一两点。


前面提到的都是用 `+ (NSPredicate *)predicateWithFormat:(NSString *)predicateFormat, ...;` 方法创建，还有另一种常用的方法：`+ (NSPredicate*)predicateWithBlock:(BOOL (^)(id evaluatedObject, NSDictionary *bindings))block`，用Block形式创建

```objective-c
NSPredicate *pre = [NSPredicate predicateWithBlock:^BOOL(id evaluatedObject, NSDictionary *bindings) {
    Test *test = (Test *)evaluatedObject;
    if (test.code.integerValue > 2) {
        return YES;
    }
    else{
        return NO;
    }
}];
```

参数`evaluatedObject`表示数组成员，block必须返回YES或者NO，分别表示匹配还是不匹配。请忽略`bindings`参数，具体作用我也没搞清楚。

### 多重筛选

如果需要匹配数个属性的筛选，用`AND`或者`OR`来串联显然有点麻烦，`NSCompoundPredicate`类可以满足我们的需求，它可以将多个`NSPredicate`对象的组合，组合方式可以是`AND`或者`OR`。

```objective-c
	NSPredicate *pre1 = [NSPredicate predicateWithFormat:@"code >= %@", @3];
    NSPredicate *pre2 = [NSPredicate predicateWithFormat:@"code <= %@", @2];
    //以AND形式组合
    NSPredicate *pre = [NSCompoundPredicate andPredicateWithSubpredicates:@[pre1,pre2]];
    //以OR形式组合
    NSPredicate *pre = [NSCompoundPredicate orPredicateWithSubpredicates:@[pre1, pre2]];
```

___

## 匹配用法

其实NSPredicate不仅可以用于筛选，还可以用来判断匹配直接返回是否符合，主要方法是`- (BOOL)evaluateWithObject:(id)object;`，用法：

```objective-c
Test *test1 = [[Test alloc]init];
test1.name = @"absr";
test1.code = @1;

NSPredicate *pres = [NSPredicate predicateWithFormat:@"code == %@", @2];
BOOL match = [pres evaluateWithObject:test1];
```

当然最常用的还是配合配合正则表达式，列举几个常用的正则

是否以a开头以e结尾

```objective-c
NSString *string=@"assdbfe";
NSString *targetString=@"^a.+e$";
NSPredicate *pres = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", targetString];
BOOL match = [pres evaluateWithObject:string];
```

是否是邮箱

```objective-c
NSString *strRegex = @"[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{1,5}";
```

是否是手机号(包含了181、1700、1709、1705号段)

```objective-c
+ (BOOL)isMobileNumber:(NSString *)mobileNum
{
    /**
     *  中国移动：China Mobile
     *
     *  134[0-8],135,136,137,138,139,147,150,151,157,158,159,182,183,187,188,1705
     *
     */
    NSString * CM = @"^1((34[0-8]|(3[5-9]|5[017-9]|8[2378])|47\\d)|705)\\d{7}$";
    /**
     *  中国联通：China Unicom
     *
     *  130,131,132,152,155,156,185,186,1709
     *
     */
    NSString * CU = @"^1((3[0-2]|5[256]|8[56])[0-9]|709)\\d{7}$";
    /**
     *  中国电信：China Telecom
     *
     *  133,1349,153,180,189,181,177,1700
     *
     */
    NSString * CT = @"^1((33|53|77|8[019])[0-9]|349|700)\\d{7}$";
    NSPredicate *regextestcm = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", CM];
    NSPredicate *regextestcu = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", CU];
    NSPredicate *regextestct = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", CT];
    if (
        ([regextestcm evaluateWithObject:mobileNum] == YES)
        || ([regextestcu evaluateWithObject:mobileNum] == YES)
        || ([regextestct evaluateWithObject:mobileNum] == YES)
        ){
        return YES;
    }
    else{
        return NO;
    }
}
```

____

## 坑


在某些情况下，类似于上面例子中的`code`字符串不是很明确，创建的时候就会这样使用

```objective-c
    Test *test1 = [[Test alloc]init];
    test1.name = @"absr";
    test1.code = @1;

    Test *test2 = [[Test alloc]init];
    test2.name = @"asb";
    test2.code = @2;

    Test *test3 = [[Test alloc]init];
    test3.name = @"raskj";
    test3.code = @3;

    NSArray *array = @[test1, test2, test3];

    NSPredicate *pre = [NSPredicate predicateWithFormat:@"%@ == %@", @"code", @2];
    NSLog(@"%@", [array filteredArrayUsingPredicate:pre]);
```
注意NSPredicate对象的初始化方式。执行后你会发现是错误的，打印`pre`发现`"code" == 2`，这说明查找的是`"code"`（注意：这是带双引号的）方法的返回值，这显然行不通。

解决办法：

```objective-c
NSPredicate *pre = [NSPredicate predicateWithFormat:@"%K == %@", @"code", @2];
```
注意：`%K`的`K`必须是大写。



另外对于空字符串的筛选也需要注意，Person 是一个带有 name 字段的类

```objective-c
    Person *person = [[Person alloc] init];
    NSArray *array = @[person];
    NSPredicate *pre = [NSPredicate predicateWithFormat:@"name.length == 0"];
    NSLog(@"%@", [array filteredArrayUsingPredicate:pre]);
```

直接用`name.length == 0` 的筛选条件会莫名其妙的失效，反而 `name == nil` 是可行的，或者直接用

```objective-c
NSPredicate *pre = [NSPredicate predicateWithBlock:^BOOL(Person *_Nullable evaluatedObject, NSDictionary<NSString *,id> * _Nullable bindings) {
        return [evaluatedObject.name length] == 0;
    }];
```

____

## 最后
NSPredicate几乎可以满足所有形式的查询，配合Core Data的数据库查询当然不在话下。NSPredicate用法不只这些，有兴趣的同学可以看下nshipster网站的这篇[博文](http://nshipster.com/nspredicate/)，里面提到了一些我没有涉及到的用法。
