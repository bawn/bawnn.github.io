---
layout: post
title: "MagicalRecord"
date: 2014-12-15
comments: true
categories: iOS
tags: [iOS]
published: ture
keywords: MagicalRecord
description: MagicalRecord配合Mantle
---

![image](http://lc.yardwill.top/bang.png)

在开始之前，我们先创建一个名为MemberManaged的实体

![image](http://lc.yardwill.top/entity.png)

>MemberManaged.h

```
@interface MemberManaged : NSManagedObject
@property (nonatomic, retain) NSString * memberID;
@property (nonatomic, retain) NSString * mobilePhone;
@property (nonatomic, retain) NSDate * createDate;
@property (nonatomic, retain) NSNumber * goldNumber;
@property (nonatomic, retain) NSNumber * age;
@property (nonatomic, retain) NSNumber * isVip;
@property (nonatomic, retain) NSString * url;
```
后续的例子都是以此实体进行数据库的操作

## 快速入门

### 配置

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    [MagicalRecord setupAutoMigratingCoreDataStack];
    // ...
    return YES;
}
- (void)applicationWillTerminate:(UIApplication *)application
{
    [MagicalRecord cleanUp];
}
```

### 查找数据

```
//返回MemberManaged表中的第一条数据
MemberManaged *memberManaged = [MemberManaged MR_findFirst];
```

```
//返回MemberManaged表中的所有数据
NSArray *array = [[MemberManaged MR_findAll];
```

```
//键值条件查找，返回符合条件的所有数据
NSArray *array = [MemberManaged MR_findByAttribute:@"memberID" withValue:@"1"];
```

```
//按指定字段排序
NSArray *array = [MemberManaged MR_findAllSortedBy:@"age" ascending:YES];
```

```
//自定义NSPredicate查找，返回符合条件的所有数据
NSPredicate *pre = [NSPredicate predicateWithFormat:@"age > 18"];
MemberManaged *memberManaged = [MemberManaged MR_findAllWithPredicate:pre];
```

对于NSPredicate不熟悉的同学可以看我之前写的介绍NSPredicate的[博文](http://bawn.github.io/2014/05/07/NSPredicate/)，关于其他的查找方法我就不一一介绍了。

### 插入数据
```
    [MagicalRecord saveWithBlock:^(NSManagedObjectContext *localContext) {
        MemberManaged *memberManaged = [MemberManaged MR_createInContext:localContext];
        memberManaged.memberID = @"1";
        memberManaged.mobilePhone = @"xxxxxxxx";
        memberManaged.createDate = [NSDate date];
        memberManaged.goldNumber = @2;
        memberManaged.age = @18;
        memberManaged.url = @"http://bawn.github.io/";
        memberManaged.isVip = @YES;

    } completion:^(BOOL success, NSError *error) {
        // ...
    }];
```

### 删除数据

```
	//删除单条数据
   [MagicalRecord saveWithBlock:^(NSManagedObjectContext *localContext) {
        MemberManaged *member = [MemberManaged MR_findFirstInContext:localContext];
        [member MR_deleteEntity];
    } completion:^(BOOL success, NSError *error) {
        // ...
    }];
```

```
	//删除表
    [MagicalRecord saveWithBlock:^(NSManagedObjectContext *localContext) {
        [MemberManaged MR_truncateAllInContext:localContext];
    } completion:^(BOOL success, NSError *error) {
        // ...
    }];
```

### 更新数据
```
    MemberManaged *member = [MemberManaged MR_findFirst];
    [MagicalRecord saveWithBlock:^(NSManagedObjectContext *localContext) {
        MemberManaged *localMember = [member MR_inContext:localContext];
        localMember.age = @22;
    } completion:^(BOOL success, NSError *error) {
        // ...
    }];
```



## 配合Mantle

### 基本转换

上一篇 [博文](http://bawn.github.io/2014/12/11/Mantle/) 中提到Mantle的 [MTLManagedObjectAdapter](https://github.com/Mantle/MTLManagedObjectAdapter) 类，2.0版本开发者已把此类作为一个单独 repo 从 Mantle 剥离出来，这个类有个名叫`MTLManagedObjectSerializing`的协议，此协议有两个必须实现的方法:

```
//返回此类对应的实体类名
+ (NSString *)managedObjectEntityName;
```

```
//返回此类和实体属性的映射关系
+ (NSDictionary *)managedObjectKeysByPropertyKey;
```

另外由于Member类和MemberManaged类的url字段的类型不一致，需要实现另一个协议方法，实现`NSUrl`-->`NSString`（age和isVip字段不需要转换，不要问我为什么）

```
//属性值转换
+ (NSValueTransformer *)entityAttributeTransformerForKey:(NSString *)key;
```

先看具体实现

>Member.h

```
@interface Member : MTLModel<MTLJSONSerializing, MTLManagedObjectSerializing>
@property (nonatomic, retain) NSString   * memberID;
@property (nonatomic, retain) NSString   * mobilePhone;
@property (nonatomic, retain) NSDate     * createDate;
@property (nonatomic, retain) NSNumber   *goldNumber;
@property (nonatomic, assign) NSUInteger age;
@property (nonatomic, assign) BOOL       isVip;
@property (nonatomic, retain) NSURL      *url;
```

>Member.m

```
//表示Member类对应的实体类是MemberManaged
+ (NSString *)managedObjectEntityName{
    return @"MemberManaged";
}
```

```
//表示Member类向MemberManaged类转换的字段映射，也是需要写全的
+ (NSDictionary *)managedObjectKeysByPropertyKey{
    return @{
             @"memberID" : @"memberID",
             @"mobilePhone" : @"mobilePhone",
             @"createDate" : @"createDate",
             @"goldNumber" : @"goldNumber",
             @"age" : @"age",
             @"isVip" : @"isVip",
             @"url" : @"url"
             };
}
```

具体运用：

```
    NSDictionary *dic = @{
						  @"id" : @"2",
                          @"phone" : @"xxxxxxxx",
                          @"date" : @"2014-09-09",
                          @"goldNumber" : @2,
                          @"age" : @"18",
                          @"url" : @"http://bawn.github.io/",
                          @"isVip" : NSNull.null
                          };
    Member *member = [MTLJSONAdapter modelOfClass:[Member class] fromJSONDictionary:dic error:nil];

    [MagicalRecord saveWithBlock:^(NSManagedObjectContext *localContext) {
        [MTLManagedObjectAdapter managedObjectFromModel:member insertingIntoContext:localContext error:nil];
    } completion:^(BOOL success, NSError *error) {
        NSLog(@"%lu", (unsigned long)[MemberManaged MR_findAll].count);
    }];
```


1. `Member *member = [MTLJSONAdapter modelOfClass:[Member class] fromJSONDictionary:dic error:nil];` 完成从NSDictionary-->Member转换，并返回Member实例
2. `[MTLManagedObjectAdapter managedObjectFromModel:member insertingIntoContext:localContext error:nil];` 完成Member-->MemberManaged转换，返回MemberManaged实例，但是我们并不需要。
3. 配合MagicalRecord储存方法`+ (void) saveWithBlock:(void(^)(NSManagedObjectContext *localContext))block;`

**注意：对于MemberManaged类，我们并不需要对它做任何的处理。**


### 唯一性检查

同样是实现`<MTLManagedObjectSerializing>`中的一个方法：

```
+ (NSSet *)propertyKeysForManagedObjectUniquing{
    return [NSSet setWithObject:@"memberID"];
}
```

表示当插入新数据的时候，对比需要插入的这条数据的memberID字段的值是否和数据库中的有相同。如果有相同就覆盖更新这条数据，如果没有就新增。这样带来的方便之处显而易见。

更新数据

```

- (IBAction)updateData:(id)sender{
    NSDictionary *dic = @{
						  @"id" : @"2",
                          @"phone" : @"xxxxxxxx",
                          @"date" : @"2015-12-09",
                          @"goldNumber" : @2,
                          @"age" : @"19",
                          @"url" : @"http://bawn.github.io/",
                          @"isVip" : NSNull.null
                          };
    Member *member = [MTLJSONAdapter modelOfClass:[Member class] fromJSONDictionary:dic error:nil];

    [MagicalRecord saveWithBlock:^(NSManagedObjectContext *localContext) {
        [MTLManagedObjectAdapter managedObjectFromModel:member insertingIntoContext:localContext error:nil];
    } completion:^(BOOL success, NSError *error) {
        NSLog(@"%lu", (unsigned long)[MemberManaged MR_findAll].count);
    }];
}
```

数据库中只有一条数据，因为插入的数据的memberID都是2。

## 总结

MagicalRecord配合Mantle使用至少让代码看起来简洁了不少，再也不用为Core Data复杂的API而烦恼，也不用再写if/else来做字段的转换。

Demo地址：[MagicalRecord-Mantle](https://github.com/bawn/MagicalRecord-Mantle)
