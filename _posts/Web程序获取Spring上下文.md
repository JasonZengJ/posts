title: Web程序获取Spring上下文
date: 2014-07-29 00:53:28
tags: Spring
---
获取Spring上下文主要有两类方法：从文件获取和在web容器中获取。

从文件中获取很简单，说下从web容器中获取，在网上找到一种很简单的方法，如下：

建个类，实现spring的ApplicationContextAware 接口，即 

```
public class SpringContextTool implements ApplicationContextAware { 
        private static ApplicationContext context;
        public void setApplicationContext(ApplicationContext acx) {
                context = acx; 
        } 
        public static ApplicationContext getApplicationContext() { 
                return context; 
        } 
}
```

然后把这个类配置到spring-context.xml所有bean的最上面，这样spring在一开始加载配置文件的时候，就把ApplicationContext的实例自动注入：

```
<bean id="contextTool"  class="com.jason.utils.SpringContextTool"></bean>
```

然后在代码中这样调用：

```
SpringContextTool.getApplicationContext().getBean(BeanId); 
```