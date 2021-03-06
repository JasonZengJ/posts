title: Objective-C和Java的简单对比
date: 2015-08-08 01:24:18
tags: Objective-c
---

转自[http://www.coderyi.com/archives/177](http://www.coderyi.com/archives/177)

##Objective-C的一些点

Objective-C通常写作Object-C或者Obj-C，是根据C语言所衍生出来的语言，继承了C语言的特性，是扩充C的面向对象编程语言。

##Java的一些点

Java是一种简单的，跨平台的，面向对象的，分布式的，解释的，健壮的安全的，结构的中立的，可移植的，性能很优异的多线程的，动态的语言。
Java分为三个体系Java SE（J2SE，Java2 Platform Standard Edition，标准版），JavaEE（J2EE，Java 2 Platform, Enterprise Edition，企业版），Java ME（J2ME，Java 2 Platform Micro Edition，微型版）。
Java通过JDBC为多种关系数据库提供统一访问，JDBC是一种用于执行SQL语句的Java API，它由一组用Java语言编写的类和接口组成。

##一些相同点

在与C++对比上，他们有相同的地方，没有运算符重载、类的多继承。
Java 和Objective-C 都区分大小写，并且都是采用驼峰命名法。
Java和Objective-C都有异常处理

##一些对比点

Java没有指针，Objective-C中所有对象都是指针的形式。
Java含有构造方法和析构方法finalize ，对比Objective-C相当于init方法和dealloc方法。
Java是通过set和get方法来访问成员变量，java的成员变量是在类体的变量部分中定义的变量，也称为属性。成员变量又称全局变量，定义在类中，和类的方法处于同一个层次。 Objective-C的属性也是类似概念，它通过@property与@synthesize配对使用来实现属性概念，并且默认实现setter和getter方法。
Java中包是类和接口的集合，这相当于Objective-C中的framework。
Java通过输入流和输出流来读写文件，Objective-C则通过更简单的NSData来实现。
Java是通过JVM来进行垃圾回收的，Objective-C则通过ARC的机制进行自动内存管理。

##Java和Objective-C是如何实现多继承的？

Java中不可以继承多个父类，但是可以实现多个接口，这样就实现了多继承概念。Objective-C则通过Categories和Protocols来提供多继承。

##Java是解释型语言，Objective-C编译型语言

解释型语言在运行程序的时候才翻译，比如解释型BASIC语言，专门有一个解释器能够直接执行BASIC程序，每个语句都是执行的时候才翻译。这样解释型语言每执行一次就要翻译一次，效率比较低。解释型语言，执行速度慢、效率低；依赖解释器、跨平台性好。如Java、BASIC。
编译型语言：把做好的源程序全部编译成二进制代码的可运行程序。然后，可直接运行这个程序。 编译型语言，执行速度快、效率高；依赖编译器、跨平台性差些。如C、C++、Objective-C、Delphi、Pascal，Fortran。
另外Objective-C是动态语言，就是在运行时可以改变代码结构，Java则是静态语言。Java和Objective-C同属于静态类型语言，并非动态类型语言，他们的数据类型是在编译其间确定的或者说运行之前确定。他们也同是强类型语言，一旦一个变量被指定了某个数据类型，如果不经过强制类型转换，那么它就永远是这个数据类型。
关于这点可以此文，编译型语言、解释型语言、静态类型语言、动态类型语言概念与区别

##JVM和GCC

Java是一种解释型语言，它的编译器不是直接编译成机器指令，而只需生成在Java虚拟机（JVM）上运行的目标代码（二进制字节码），就可以在多种平台上不加修改地运行。Java虚拟机在执行字节码时，把字节码解释成具体平台上的机器指令执行。这就是Java的能够“一次编译，到处运行”的原因。
GCC全称GNU编译器套件（GNU Compiler Collection），它把Objective-C编译成机器指令。

