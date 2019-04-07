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

            <!-- 一行代码替代下面的映射器和适配器
            <mvc:annotation-driven/> -->
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

---
### 参数传递

### 异常处理

### 上传文件

### json数据交互
导入架包，配置json转换器，走起
#### 交互流程

![]({{ site.url }}/assets/blog_images/json交互.png)

#### 交互实现
请求的是json串：
```java
public @ResponseBody User Test(@RequestBody User user ) throws Exception{
            return user;
    }
```

请求的是key/value串：
```java
public @ResponseBody User Test(User user ) throws Exception{
            return user;
    }
```

### RESTful
对于RESTful架构的理解，参考 [理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful.html?bsh_bid=1717507328)

我摘取其中的一段：
    
    综合上面的解释，我们总结一下什么是RESTful架构：
    （1）每一个URI代表一种资源；
    （2）客户端和服务器之间，传递这种资源的某种表现层；
    （3）客户端通过四个HTTP动词，对服务器端资源进行操作，实现"表现层状态转化"。
#### RESTful的url风格
现实开发中，我们一般只是实现URL地址的REST风格，如下：
```
非REST的url：http://.../queryItems.action?id=001

REST的url：http://.../items/001
```
#### URL模板映射实现RESTful

**注意要配置REST的前端控制器**

controller类配置：
```java
/*
/items/{id}里面的id表示将url这个位置的参数传递到@PathVariable指定名称中；
*/
@RequestMapping("/items/{id}/{type}")
public @ResponseBody Items items(@PathVariable("id") Integer id,
@PathVariable("type") String type,Model model) throws Exception{
        Items items=itemsservice.findById(id);
        return items;
}

```

### 拦截器
    springMVC拦截器针对HandlerMapping进行拦截设置，只有映射成功才使用该拦截器；

#### 拦截器实现
    在 Spring 框架之中，我们要想实现拦截器的功能，主要通过两种途径，
    第一种是实现HandlerInterceptor接口，第二种是实现WebRequestInterceptor接口。

##### HandlerInterceptor 接口
```java
//测试拦截器1
public class HandlerInterceptor1 implements HandlerInterceptor{

    @Override
    public boolean preHandle(HttpServletRequest request,
            HttpServletResponse response, Object handler) throws Exception {

        System.out.println("HandlerInterceptor1....preHandle");

        //false表示拦截，不向下执行；true表示放行
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request,
            HttpServletResponse response, Object handler,
            ModelAndView modelAndView) throws Exception {

        System.out.println("HandlerInterceptor1....postHandle");

    }

    @Override
    public void afterCompletion(HttpServletRequest request,
            HttpServletResponse response, Object handler, Exception ex)
            throws Exception {

        System.out.println("HandlerInterceptor1....afterCompletion");
    }
}

```

    1. preHandle方法：进入Handler方法之前执行。可以用于身份认证、身份授权。比如如果认证没有通过
    表示用户没有登陆，需要此方法拦截不再往下执行（return false），否则就放行（return true）; 

    2. postHandle方法：进入Handler方法之后，返回ModelAndView之前执行。可以看到该方法中有个
    modelAndView的形参。应用场景：从modelAndView出发：将公用的模型数据（比如菜单导航之类的）
    在这里传到视图，也可以在这里同一指定视图;

    3. afterCompletion方法：执行Handler完成之后执行。应用场景：统一异常

#### springmvc拦截器的配置
在springmvc中，拦截器是针对具体的HandlerMapping进行配置的，也就是说如果在某个HandlerMapping中配置拦截，经过该 HandlerMapping映射成功的handler最终使用该拦截器。比如，假设我们在配置文件中配置了的映射器是org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping，那么我们可以这样来配置拦截器：

```xml
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping">
    <property name="interceptors">
        <list>
            <ref bean="handlerInterceptor1"/>
            <ref bean="handlerInterceptor2"/>
        </list>
    </property>
</bean>
<bean id="handlerInterceptor1" class="ssm.intercapter.HandlerInterceptor1"/>
<bean id="handlerInterceptor2" class="ssm.intercapter.HandlerInterceptor2"/>
```

那么在springmvc中，如何配置类似于全局的拦截器呢？上面也说了，springmvc中的拦截器是针对具体的映射器的，为了解决这个问题，springmvc框架将配置的类似全局的拦截器注入到每个HandlerMapping中，这样就可以成为全局的拦截器了。配置如下：
```xml
<!-- 配置拦截器 -->
<mvc:interceptors>
    <!-- 多个拦截器，按顺序执行 -->        
    <mvc:interceptor>
        <mvc:mapping path="/**"/> <!-- 表示拦截所有的url包括子url路径 -->
        <bean class="ssm.interceptor.HandlerInterceptor1"/>
    </mvc:interceptor>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <bean class="ssm.interceptor.HandlerInterceptor2"/>
    </mvc:interceptor>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <bean class="ssm.interceptor.HandlerInterceptor3"/>
    </mvc:interceptor>
</mvc:interceptors>
```

#### springmvc拦截器的执行测试
##### 三个拦截器都放行
我们将三个拦截器的preHandle方法中返回值都改成true，来测试一下拦截器的执行顺序，测试结果如下：

    HandlerInterceptor1….preHandle 
    HandlerInterceptor2….preHandle 
    HandlerInterceptor3….preHandle

    HandlerInterceptor3….postHandle 
    HandlerInterceptor2….postHandle 
    HandlerInterceptor1….postHandle

    HandlerInterceptor3….afterCompletion 
    HandlerInterceptor2….afterCompletion 
    HandlerInterceptor1….afterCompletion

当所有拦截器都放行的时候，preHandle方法是按照配置的顺序执的；而另外两个方法按照配置的顺序逆向执行的。
##### 有一个拦截器不放行
我们将第三个拦截器的preHandle方法中返回值改成false，前两个还是true

    HandlerInterceptor1….preHandle 
    HandlerInterceptor2….preHandle 
    HandlerInterceptor3….preHandle

    HandlerInterceptor2….afterCompletion 
    HandlerInterceptor1….afterCompletion

根据打印的结果做一个总结： 

    1. 由于拦截器1和2放行，所以拦截器3的preHandle才能执行。
    也就是说前面的拦截器放行，后面的拦截器才能执行preHandle。 

    2. 拦截器3不放行，所以其另外两个方法没有被执行。即如果某个拦截器不放行，
    那么它的另外两个方法就不会背执行。 

    3. 只要有一个拦截器不放行，所有拦截器的postHandle方法都不会执行，
    但是只要执行过preHandle并且放行的，就会执行afterCompletion方法。

##### 三个拦截器都不放行
    HandlerInterceptor1….preHandle

很明显，就只执行了第一个拦截器的preHandle方法，因为都不放行，所以没有一个执行postHandle方法和afterCompletion方法。
