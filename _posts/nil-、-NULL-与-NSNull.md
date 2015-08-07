title: nil 、 NULL 与 NSNull
date: 2015-08-08 02:48:33
tags: Objective-c
---

参考文档[http://nshipster.com/nil/](http://nshipster.com/nil/)

* nil是空对象指针，是 void *，如果给基本类型变量赋值nil，编译器会报以下错误：

```
Incompatible pointer to integer conversion initializing 'int' with an expression of type 'void *'
```

* nil（零）在语义上也许与null（空）有区别，但是从技术上来说，它们是等价的，都是空对象指针.

* objective-c 中的 nil与其它语言不一样的是，OC的nil 可以调用方法，只是返回的依然是nil，而其它语言 如 c ++ 可能就会造成crash.

```
    NSString *str = nil;    
    [str stringByAppendingString:@"1"];
    NSLog(@"%@",str); // nil
```

* NSNull：用一个对象来表明空的含义，主要用在Foundation 和 其它一些框架中，用于绕过垃圾回收机制的限制，使得例如NSDictionary 和 NSArray 中可以存放NULL值. 

```
 NSArray *arr = @[[NSNull null]]; // √

 NSArray *arr = @[nil]; // 编译报错：Collection element of type 'void *' is not an Objective-c object.
 
 NSMutableDictionary *mutableDictionary = [NSMutableDictionary dictionary];
 
 mutableDictionary[@"someKey"] = [NSNull null]; // Sets value of NSNull singleton for `someKey`

 NSLog(@"Keys: %@", [mutableDictionary allKeys]); // @[@"someKey"]

```
* NSNull： ＋(NSNull *)null 返回的是一个单例Singleton NSNull对象

![NSNull测试截图](https://raw.githubusercontent.com/JasonZengJ/Images/master/blog/NSNull-singleton.png)