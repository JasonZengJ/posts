title: 从Proxy对象中获取目标对象
date: 2014-07-24 21:43:52
tags: Java
---
Java的代理有两种，一种是JDK自带的叫 JDK dynamic proxy 还有一种叫做CGLib，以下引自[stackoverflow](http://stackoverflow.com/questions/10664182/what-is-the-difference-between-jdk-dynamic-proxy-and-cglib)的回答解释了两者的区别：

**JDK Dynamic proxy can only proxy by interface (so your target class needs to implement an interface, which will also be implemented by the proxy class).**

**CGLIB (and javassist) can create a proxy by subclassing. In this scenario the proxy becomes a subclass of the target class. No need for interfaces.**

**So Java Dynamic proxies can proxy: public class Foo implements iFoo where CGLIB can proxy: public class Foo**

有些情况下，比如在使用hibernate spatial 3.x 连接oracle存储地理空间数据的时候，connection 是被proxy封装了的，因此在连接的时候会出现connection找不到的问题，可通过以下代码从proxy中获取被proxy封装的对象。

```
@SuppressWarnings({"unchecked"})
protected <T> T getTargetObject(Object proxy, Class<T> targetClass) throws Exception {
  if (AopUtils.isJdkDynamicProxy(proxy)) {
    return (T) ((Advised)proxy).getTargetSource().getTarget();
  } else {
    return (T) proxy; // expected to be cglib proxy then, which is simply a specialized class
  }
}
```
