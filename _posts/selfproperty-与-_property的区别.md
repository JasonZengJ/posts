title: self.property 与 _property的区别
date: 2014-08-05 13:49:32
tags: Objective-c
---

1、self.property = value 等价于 **[self setProperty:value]**，self.property 等价于 **[self property]**

2、\_property = value 等价于 property = value 相当于将值直接写入property中，没有方法调用， \_property 等价于 property

3、在访问速度上，\_property 要比 self.property快，毕竟 \_property 不需要通过方法调用来赋值或取值，而是直接访问内存

4、Key-Value Observing(KVO) 通知不支持 _property 的方式,只支持 self.property 的方式。因为通过使用 @property PropertyType [*]property 来定义 property 的时候，系统会自动生成两个方法和里面的代码：

```

@property PropertyType [*]property

...... //其它代码

- (void)setProperty:(PropertyType [*])property 
{
   _property = property;
   ......
   NSObject *observer = [observers objectForkey:@"property"];
   if (observer){
      [observer observeValueForKeyPath:@"property" ofObject:self change:change context:context];
   }
}

- (PropertyType [*])property
{
    ......
    return _property;
    
}
```
因此，当添加了 observer 之后，使用 self.property = value 的方式来改变属性值时会在 setProperty 方法里面获取 observer 对象，如果 observer 对象存在，则调用 **[observer observeValueForKeyPath:@"property" ofObject:self change:change context:context]** 方法来通知 observer 属性 property 的值改变了。此时，若使用 _property = value 来改变属性值的话，就不会调用 setProperty: 方法，因此也不会触发 KVO 通知了。

5、在类的初始化方法中，比如init开头的方法，要使用 _property 的方式来访问属性，因为子类可以覆盖父类属性的 getter 和 setter 方法，这样在初始化的时候可能会出现问题。

```
- (void)setLastName:(NSString *)lastName 
{
   if(![lastName isEqualToString:@"value"]) {
     [NSException raise:NSInvalidArgumentException format:@"Last name must be Smith"];
   }
   self.lastName = lastName;
}
```
比如以上代码，若父类实例化的时候，默认把lastName初始化为@"",那么在子类执行setter方法的时候，就会抛出异常.

6、如果一个属性实现懒加载，那么要使用 self.property 即 **[self property]**，否则 property 不会被初始化，属性的懒加载代码范例如下：

```
- (PropertyType [*])property
{
   if(!_property) {
      _property = [PropertyType new];
   }
   return property;
}
```

参考自 《Effective Objective-c 2.0》










