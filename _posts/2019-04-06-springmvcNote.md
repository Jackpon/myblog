---
title: springMVC入门就是这么简单
categories: 
- spring
tags:
- springMVC
updated: 2019-04-06
---

SSM就是SpringMVC-Spring-Mybatis的简称，是web-service-dao的典型代表；

### 何为SpringMVC
    SpringMVC的作用基本就是Struts2的作用，也就是控制器；

### SpringMVC如何体现MVC
    Model-View-controller；
    在struts2中，service、dao就是Model，action就是controller；
    而在springMVC，controller就是controller；

### 理解SpringMVC原理

![]({{ site.url }}/assets/blog_images/SpringMVC工作原理.jpg)

### 入门配置
#### 在web.xml配置前端拦截器
```xml
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- contextConfigLocation配置springmvc加载的配置文件(配置处理器映射器、适配器等等)
      若不配置，默认加载WEB-INF/servlet名称-servlet(springmvc-servlet.xml)
    -->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:config/*.xml</param-value>
    </init-param>
</servlet>
<!--
    第一种:*.action,访问以.action三结尾，由DispatcherServlet进行解析
    第二种:/,所有访问的地址由DispatcherServlet进行解析，对静态文件的解析需要配置不让DispatcherServlet进行解析，
            使用此种方式和实现RESTful风格的url
    第三种:/*,这样配置不对，使用这种配置，最终要转发到一个jsp页面时，仍然会由DispatcherServlet解析jsp地址，
            不能根据jsp页面找到handler，会报错
    -->
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>

```
注意web.xml有一个顺序的问题，你如果把他写在下面，就可以在上面添加一些别的servlet，那么request就会先被别的servlet拦截。

#### 在springMVC.xml配置
```xml
<!-- 可以扫描controller、service、...
    这里让扫描controller，指定controller的包
    采用扫描就不用配置handler了，当然controller里的handler要注解
     -->
<context:component-scan base-package="com.jackpon.controller"></context:component-scan>
<!-- 处理器映射器
    将bean的name作为url进行查找，需要在配置Handler时指定beanname(就是url)
    所有的映射器都实现了HandlerMapping接口
     -->
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>

<!--下面这个是一个处理器适配器，所有的处理适配器都要实现HandlerAdapter接口-->
<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
<!-- 视图解析器
    解析jsp,默认使用jstl,classpath下要有jstl的包
    -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/jsp"/>
    <property name="suffix" value=".jsp"/>
</bean> 
```

#### 最后一步，编写Handler

需要说明的是，handler跟controller就是一回事
```java
@Controller
public class ItemsController3 {

    @RequestMapping("/queryItems")
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        //调用service查找数据库，查询商品列表，这里使用静态数据模拟
        List<Items> itemsList = new ArrayList<Items>();

        //向list中填充静态数据
        Items items_1 = new Items();
        items_1.setName("联想笔记本");
        items_1.setPrice(6000f);
        items_1.setDetail("X Carbon");

        Items items_2 = new Items();
        items_2.setName("苹果手机");
        items_2.setPrice(5000f);
        items_2.setDetail("iphone7");

        itemsList.add(items_1);
        itemsList.add(items_2);

        //返回ModelAndView
        ModelAndView modelAndView = new ModelAndView();
        //相当于request的setAttribute方法,在jsp页面中通过itemsList取数据
        modelAndView.addObject("itemsList", itemsList);

        //指定视图,在springMVC.xml配置中有改路径的前后缀
        modelAndView.setViewName("/items/itemsList");

        return modelAndView;
    }
}

```
