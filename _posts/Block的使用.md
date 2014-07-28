title: Block的使用
date: 2014-07-25 13:10:52
tags: Objective-c
---


本地变量：

```
     returnType (^blockName)(parameterTypes) = ^returnType(parameterTypes){ ... };
```

作为属性：

```
     @property  (nonatomic, copy) returnType (^blockName)(parameterTypes);
```

作为方法参数：

```     
     - (void)someMethodThatTakesABlock:(returnType (^)(parameterTypes))blockName;
```

C语言的typedef：

```
     typedef returnType (^TypeName)(parameterTypes);
     TypeName blockName = ^returnType(parameters){ ... };
```

参考自 [fuckingblockssyntax](http://fuckingblocksyntax.com/)