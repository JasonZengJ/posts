title: Objective-c ARC要点
data: 2014-7-16 13:00:00
tag: Objective-c
---

1、只要一个对象有strong 修饰的引用，那么这个对象就不会被释放.

2、若使用weak声明的对象引用，则不会延长对象的生命周期，而且若没有使用strong声明的变量来保存该对象引用，则weak声明的对象引用会自动变成nil，而对象分配的内存空间被回收.比如：

```
NSString * __weak string = [[NSString alloc] initWithFormat:@"First Name: %@", [self firstName]];
NSLog(@"string: %@", string);
```

输出来的结果为:

```
string: (null)
```
而且在编译时就会显示出warning:

**Assigning retained object to weak variable;object will be released after assignment**.

3、strong 等价于 retain，唯一的区别是：在ARC中使用strong,非ARC中使用retain.

4、在ARC中，若不声明(如weak),则默认是strong.

5、ARC不能防止strong reference cycles,因此要充分利用strong,weak等qualifiers来管理对象的生命周期。

6、__unsafe_unretained 声明的引用不会使得引用的对象一直存活，并且在该对象没有被`strong`引用指向的时候与`weak`引用不同，该引用不会被置为`nil`，因此会变为一个野指针，一旦对象被释放后，再使用该引用则会造成crash，加`exception`断点会看到`EXC_BAD_ACCESS`的异常提示。这是因为此时引用的指针指向的内存地址不是一个有效地址（对象被释放，内存被回收）。

7、在方法中传递参数时，要注意函数声明的参数的qualifier与传入的参数的qualifier是否一致，若不一致，则编译器会自动额外创建临时变量来使得传入的参数与函数声明的参数的qualifier一致，代码范例如下：

```
NSError *error;
BOOL OK = [myObject performOperationWithError:&error];
if (!OK) {
    // Report the error.
    // ...
```
因为在ARC中，error默认是用strong 引用即：

```
NSError * __strong e;
```

而`performOperationWithError`函数的参数声明如下：

```
-(BOOL)performOperationWithError:(NSError * __autoreleasing *)error;
```
因此编译器会重写这段代码，重写后的代码如下：

```
NSError * __strong error;
NSError * __autoreleasing tmp = error;
BOOL OK = [myObject performOperationWithError:&tmp];
error = tmp;
if (!OK) {
    // Report the error.
    // ...]
```

8、非ARC中，__block id x; 不会使引用计数＋1，而在ARC中则会使得引用计数＋1.

9、可以通过声明，\_\_unsafe_unretained \__\_block id x来在ARC模式下手动管理引用计数，但是正如\__unsafe_unretained名字的含义所示，它是不安全的，而且容易使得引用指针变成野指针（指向无效地址），在block内使用外部引用时，尤其是使用self，最好先声明一个__weak的临时变量来保存外部引用，然后再在block中使用，比如：

```
MyClass * __weak myClass = self;
[UIView animateWithDuration:0 animations:^(){
   [myClass performMethod];
}];
```
这样可以防止**Strong Reference Cycles**，或者在block结束的时候加入代码：`myClass = nil`这样也可以防止retain cycle.

```
MyClass * __block myClass = self;
[UIView animateWithDuration:0 animations:^(){
   [myClass performMethod];
   myClass = nil;
}];
```

10、在使用ARC之后，strong、weak、autoreleasing声明的变量如果没有指定初始值，那么会默认使用nil进行初始化。比如：

```
- (void)myMethod {
    NSString *name;
    NSLog(@"name: %@", name);
}
```
此时会输出`string: (null)`而不会造成crash.

11、ARC不会管理去Core Foundation或使用Core Foundation的framework如Core Graphics中的对象的生命周期（如CFArrayRef、CFMutableDictionaryRef、CGColorSpaceRef,CGGradientRef等以Ref结尾的对象），要通过调用CFRetain和CFRelease方法来管理。

12、如果要在Objective-C 和 Core Foundation对象之间进行类型转换，那么需要告诉编译器由谁来管理对象的生命周期，通过使用以下关键字来声明：

<font color="red">
注：ownership 指的是该对象的生命周期，是由ARC控制，还是由你自己控制.
</font>

* __bridge 在Objective-C 和 Core Foundation 进行指针类型转换，但不会转换ownership.
* __bridge_retained 或 CFBridgingRetain 将Objective-C指针类型转换成Core Foundation指针类型，此时会把对象的ownership转给你，然后你需要通过使用CFRelaese或CFRetain来管理对象的生命周期.
* __bridge_transfer 或 CFBridgingRelease 将非Objective-C指针类型转换成Objective-C类型，此时会把对象的ownership转给ARC，由ARC来管理对象的生命周期.

代码范例如下:

```
- (void)logFirstNameOfPerson:(ABRecordRef)person {
 
    NSString *name = (NSString *)ABRecordCopyValue(person, kABPersonFirstNameProperty);
    NSLog(@"Person's first name: %@", name);
    [name release];
}
```

因为使用了Core Foundation 中的类( 如`ABRecordRef` )，因此要变成下面这样

```
- (void)logFirstNameOfPerson:(ABRecordRef)person {

    NSString *name = (NSString *)CFBridgingRelease(ABRecordCopyValue(person, kABPersonFirstNameProperty));
    NSLog(@"Person's first name: %@", name);
}
```
以下是Core Foundation 内存管理示例：

```
- (void)drawRect:(CGRect)rect {
    CGContextRef ctx = UIGraphicsGetCurrentContext();
    CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceGray();
    CGFloat locations[2] = {0.0, 1.0};
    NSMutableArray *colors = [NSMutableArray arrayWithObject:(id)[[UIColor darkGrayColor] CGColor]];
    [colors addObject:(id)[[UIColor lightGrayColor] CGColor]];
    CGGradientRef gradient = CGGradientCreateWithColors(colorSpace, (__bridge CFArrayRef)colors, locations);
    CGColorSpaceRelease(colorSpace); <font color="green"> // 释放 Core Foundation 对象.</font>
    CGPoint startPoint = CGPointMake(0.0, 0.0);
    CGPoint endPoint = CGPointMake(CGRectGetMaxX(self.bounds), CGRectGetMaxY(self.bounds));
    CGContextDrawLinearGradient(ctx, gradient, startPoint, endPoint,
                                kCGGradientDrawsBeforeStartLocation | kCGGradientDrawsAfterEndLocation);
    CGGradientRelease(gradient); <font color="green"> // 释放 Core Foundation 对象.</font>
}
```

13、针对文件启动或关闭ARC.  使用命令-fobjc-arc 或 -fno-objc-arc.

![文件启动或关闭ARC示例图](https://raw.githubusercontent.com/JasonZengJ/Images/master/blog/fno-objc-arc.png)

####ARC强制规则：（以下在编译时就会报错）
1、不能明确的调用dealloc，也不能实现和调用retain, release, retainCount,autorelease等方法，也禁止使用@selector(retain), @selector(release).

2、可以继成实现dealloc方法，但不能在方法中调用[super dealloc].

3、仍然可以使用Core Fundation的CFRetain, CFRelease等相关方法.

4、不能使用NSAllocateObject、 NSDeallocateObject.

5、不能使用C结构的指针，如结构体之类.

6、不能使用NSAutoreleasePool，因为ARC提供了@autoreleasepool 块来管理.

7、不能使用NSZone，因为在现在的Objective-c运行环境中会被忽略.

8、getter和setter不能使用new开头，

```
// Won't work:
@property NSString *newTitle;

// Works:
@property (getter=theNewTitle) NSString *newTitle;
```

参考文档 [Transitioning to ARC Release Notes](https://developer.apple.com/library/mac/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html)


