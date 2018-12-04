---
title: Bean的装配
categories: 
- spring
tags:
- spring
updated: 2018-12-04
---

- **Bean为什么要装配 ？**

我的理解是，不装配bean，每次都要实例化（new）对象，很显然与控制反转（IOC）的设计模式不符，装配bean能够使对象抽象出来。
- **三种主要的装配机制**

提供了三种主要的装配机制：

    - 在XML中进行显式配置；
    - 在Java中进行显式配置；
    - 隐式的bean发现机制和自动装配；
---
- **自动化装配**

Spring从两个角度来实现自动化装配：

组件扫描（component scanning）：Spring会自动发现应用上下文
中所创建的bean。

自动装配（autowiring）：Spring自动满足bean之间的依赖。

```java
/*
首先我们设计一个会唱歌的人
@Component 这个简单的注解表明该类会作为组件类，
并告知Spring要为这个类创建bean
*/
@Component 
public class Person {
	public void singing(String str) {
		System.out.println(str);
	}
}
---
/*
组件扫描默认是不启用的。我们还需要显式配置一下Spring，
从而命令它去寻找带有@Component注解的类，并为其创建bean。
java配置
*/
@Configuration  //注解表明这个类是一个配置类
@ComponentScan  // 启用组件扫描
public class PersonConfig {

}
//xml配置
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN//EN" "http://www.springframework.org/dtd/spring-beans.dtd">
<beans>
  <bean id="person" class="po.Person">
  </bean>
</beans>

---
public class PersonTest {

	@Test
	public void test() {
	//1. 声明Spring上下文，采用java配置类
        ApplicationContext ac = new AnnotationConfigApplicationContext(PersonConfig.class);
        //2. 声明Spring应用上下文，采用xml配置
        //ApplicationContext ac = new ClassPathXmlApplicationContext("Person.xml");
        //通过Spring上下文获取Bean，在这里Spring通过自动扫描发现了Person的实现，并自动创建bean。
        Person person=ac.getBean(Person.class);
        person.singing("beautiful in white");
	}

}
```
如果没有其他配置的话，@ComponentScan默认会扫描与配置类相同的包；

- 为组件扫描的bean命名

默认情况下，spring会为我们将组件的第一个字母改为小写，例如将Person命名为person
- 设置组件扫描的基础包

spring为我们提供了两种方式：
```java
@ComponentScan(basePackageClasses={})
@ComponentScan(basePackages={})
```
- **通过为bean添加注解实现自动装配**

    简单来说，自动装配就是让Spring自动满足bean依赖的一种方法，在
满足依赖的过程中，会在Spring应用上下文中寻找匹配某个bean需求
的其他bean。为了声明要进行自动装配，我们可以借助Spring的
@Autowired注解。

    如果现在歌手说，你叫我唱歌，怎么不给我麦克风啊？让我们将上面的实验添加个麦克风类；
```java
//建立MicroPhone类
@Component
public class MicroPhone {
	public void singingByMicroPhone(String str){
		System.out.println(str);
	}
}
---
//更改Person类，将MicroPhone添加进去
@Component
public class Person {
    //Autowired就是spring为该东西实例化
	@Autowired
	private MicroPhone microPhone;
	public void singing(String str) {
		microPhone.singingByMicroPhone(str);
	}
}

```
@Autowired可以在变量、构造器、setter()上注解

---
- **在XML中进行显式配置**

XML显式配置有两种方式，构造器和setter();
```java
//建立有名字和年龄的Person类
public class Person {
	private int age;
	private String name;
	public Person(String name,int age){
		this.name=name;
		this.age=age;
	}
	public int getAge() {
		return age;
	}
	public String getName() {
		return name;
	}
	public void setAge(int age) {
		this.age = age;
	}
	public void setName(String name) {
		this.name = name;
	}
}
---
//针对构造器和setter()的xml配置如下：
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN//EN" "http://www.springframework.org/dtd/spring-beans.dtd">
<beans>
<!-- setter -->
  <!-- <bean id="person" class="po.Person">
        <property name="name">
            <value>jack</value>
        </property>    
        <property name="age">
            <value>23</value>
        </property>
   </bean>
    <bean id="psl" class="service.PersonServiceImpl">
    	<property name="p" ref="person"/>
   </bean> -->
   
   <!-- 构造器 -->
   <bean id="person" class="po.Person">
        <constructor-arg index="0" value="jack" />
        <constructor-arg index="1" value="23" />
   </bean>
    <bean id="psl" class="service.PersonServiceImpl">
    	<constructor-arg index="0" ref="person"/>
   </bean>
</beans>

//name是方法中参数名字，ref是注入的对象
<beans default-lazy-init="true"></beans>
```
`lazy-init="true"`表示延时加载，只有用到时才会加载，默认是false


