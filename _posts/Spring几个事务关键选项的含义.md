title: Spring几个事物关键选项的含义
date: 2014-07-28 13:10:52
tags: Spring 
---
1、readonly 只读事务选项，是为了优化数据库和驱动性能所设定的，如不安排相应的数据库锁等，减少轻事务对数据库的压力，但不会保证事务的一致性，适合没有对数据库进行修改操作的应用。同时，也是hibernate二级缓存的类型，设定readonly ＝ true，那么hibernate会开启readonly缓存

2、propagation 事务传播选项，指定事务传播的规则，有以下几个选项：

* REQUIRED 如果已经存在事务，则在当前事务中运行，若没有事务，则新建一个事务。
* SUPPORTS 如果已经存在事务，则在当前事务中运行，若没有事务，则以非事务的方式运行。MANDATORY 如果已经存在事务，则在当前事务中运行，若没有事务，则抛出异常。
* REQUIRES_NEW 总是创建新事务，若已开启一个事务，则将那个事务挂起。
* NOT_SUPPORTED 总是以非事务方式运行，若已开启一个事务，则将那个事务挂起。
* NEVER 总是以非事务方式运行，若已开启一个事务，则抛出异常。
* NESTED 若已存在事务，则在该事务的嵌套事务中运行，否则将创建一个新事务。

以下是使用spring的AOP声明式开启事务的配置代码：

```
<bean id="txManager" class="org.springframework.orm.hibernate4.HibernateTransactionManager">
    <property name="sessionFactory" ref="sessionFactory"></property>
</bean>
<aop:config>
    <aop:advisor pointcut="execution(* com.sdmsystem..*.*(..))" advice-ref="txAdvice"/>
</aop:config>
<tx:advice id="txAdvice" transaction-manager="txManager">
    <tx:attributes>
        <tx:method name="get*" read-only="true"/>
        <tx:method name="query*" read-only="true"/>
        <tx:method name="find*" read-only="true"/>
        <tx:method name="load*" read-only="true"/>
        <tx:method name="list*" read-only="true"/>
        <tx:method name="validate*" read-only="true"/>
        <tx:method name="import*" propagation="REQUIRED"/>
        <tx:method name="save*" propagation="REQUIRED"/>
        <tx:method name="execut*" propagation="REQUIRED"/>
        <tx:method name="register*" propagation="REQUIRED"/>
    </tx:attributes>
 
</tx:advice>
```