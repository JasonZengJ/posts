title: javascript比较运算符==与===的区别
date: 2014-07-29 00:54:17
tags: Javascript
---
===操作符：

要是两个值类型不同，返回false

要是两个值都是number类型，并且数值相同，返回true

要是两个值都是stirng，并且两个值的String内容相同，返回true

要是两个值都是true或者都是false，返回true

要是两个值都是指向相同的Object，Arraya或者function，返回true

要是两个值都是null或者都是undefined，返回true

``` 
function test() {
    console.log(null === undefined);
    console.log( 1   === 1);
    console.log("1"  === 1);
    var obj = {};
    var obj1 = obj;
    console.log(obj === obj1);
}
```
             
==操作符：

如果两个值具有相同类型，会进行===比较，返回===的比较值

如果两个值不具有相同类型，也有可能返回true

如果一个值是null另一个值是undefined，返回true

如果一个值是string另个是number，会把string转换成number再进行比较

如果一个值是true，会把它转成1再比较，false会转成0

如果一个值是Object，另一个是number或者string，会把Object利用 valueOf()或者toString()转换成原始类型再进行比较

```
function test() {
    console.log(null == undefined);
    console.log( 1   == 1);
    console.log("1"  == 1);
    console.log(nul  == undefined);
    var obj = {str:str};
    var obj1 = obj;
    console.log("2"  == 2 == 1);
    console.log(obj  == "str:str");
}
```                    
