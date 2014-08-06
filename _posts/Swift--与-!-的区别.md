title: "Swift ? 与 ! 的区别"
date: 2014-08-06 14:28:30
tags: Swift
---
高大上的Swift一出来，果断xcode 6.0 beta1 装上，拿着官方的swift书一边学习一边打代码，然后碰到了这个? 与 !这样新奇的东东，玩了半天百思不得其解，最后google了下，终于了解了它的用法.

这里有一篇比较详细的介绍Swift ? 与 !的区别的文章 [http://vjiazhi.com/kaifa/1075.html](http://vjiazhi.com/kaifa/1075.html) ，虽然我木有仔细看过，不过大致浏览了一下，对于? 与 !用的比较少的人会比较适合，用的多的直接看一下总结会更直白：

###说明：
Optional ( ? 与 ! ) 其实是个enum，里面有None和Some两种类型。其实所谓的nil就是Optional.None, 非nil就是Optional.Some, 然后会通过Some(T)包装（wrap）原始值，这也是为什么在使用Optional的时候要拆包（从enum里取出来原始值）的原因, 也是PlayGround会把Optional值显示为类似{Some "hello world"}的原因。

###? 与 ! 总结如下：

  1. Int? 等价于 Optional<Int>
  
  2. 上述 Int? 类型的变量 oVar 表示 oVar 要嘛包含Int类型的数据，要嘛为nil —— 非Optional Type的变量不能赋值为nil；也就是说，Optional类型允许变量没有值，其它类型如果没有初始化值在使用时会报错 —— 比如通过下标访问数组元素时返回Optional类型，越界时可以返回nil；
  
  3. 在声明一个Optional类型变量时如果没有赋初始值，那么默认为nil；
  
  4. Optional类型变量的真实值是被封装起来的，包装在其枚举值Some中，如上{Some 123}；
  
  5. 使用问号？表示封装（可能有值可能没有），使用感叹号！表示拆封取值（强制认为有值，如果没有则会触发运行时错误），有点类似“Has a value?” —— "I assure it has!"；
  
  6. 由于Optional枚举类型遵循LogicValue协议，所以可以作为逻辑判断条件，当有值时为true，不包含值时为false；
