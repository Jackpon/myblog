---
title: spring笔记
categories: 
- spring
tags:
- spring
updated: 2019-04-01
---

#### Bean以两种形态存在：
    singletons形式和prototypes形式。
    当bean以singletons形态存在时，BeanFactory只管理一个共享的实例。所有对这个特定bean的
    实例请求，都导致返回这个唯一bean实例的引用。当bean以prototype形态存在时，每次对这个
    bean的实例请求都会导致一个新的实例的创建。当用户需要不受其他用户对象影响的对象或有类似的
    需求时，这是一个较理想的解决办法。Bean默认是以singleton形态存在的，除非你另外显式加以指定。

Webapp的Action不是线程安全的，要求在多线程环境下必须是一个线程对应一个独立的实例，不能使用singleton。
所以，我们在Spring配置Action的Bean时，需要加上属性scope=”prototype”或singleton=”false”。

#### AOP
    拦截器就是一种AOP思想产物；比如说为每个执行类添加日志执行代码，我们可以直接在需要的代码段前后添加，
    但是如果代码段有上百千段，显然效率就不高了，这个时候，我们把日志代码独立开来，在配置文件为有需求的
    对象添加日志代码就行。

#### AOP理解
    AOP是继OOP（面向对象编程）后最伟大的思想了，例如可乐生产流水线，它的基本步骤就是：拿瓶子，装可乐、
    拧瓶盖、贴上包装；但是老板突然说：“我想要为每瓶可乐写上我的名字”，你要怎么办？难道重新制作另一条新的生产线？
    这显然不太可能，而AOP就能解决这种问题，它就相当于在每条可乐生产线切一个口，织入刻字机器，当然可以切入很多个口，
    这样就算老板突然有一天又说“我又不想写上我的名字了，我想换个包装”，那我们只需要撤掉刻字机器，将包装机器换成老板
    要的那样。

#### 底层的实现：
    将日志代码独立开来，继承InvocationHandler，写上你的逻辑代码，并传入有这个打印日志需求的对象userDAO,
    最后调用jdk内置代理对象将两者融合；

```java
    UserDAO userDAO = new UserDAOImpl();
    LogInterceptor li = new LogInterceptor();
    li.setTarget(userDAO);
    UserDAO userDAOProxy = (UserDAO)Proxy.newProxyInstance(userDAO.getClass().getClassLoader(), userDAO.getClass().getInterfaces(), li);
    System.out.println(userDAOProxy.getClass());
    userDAOProxy.delete();
```

#### Aspectj框架实现spring的aop
    导入所需要的包，在配置文件加入aop代码，然后直接在interceptor类上添加注解代码就行