---
title: mybatis笔记
categories: 
- 数据库
tags:
- mybatis
updated: 2019-04-06
---

### mybatis和hibernate的区别
    hibernate是标准的ORM框架，也就是对象关系映射，它无需程序员编写sql语句，
    可以像操纵对象一样来对数据库操作；
    mybatis可以说是半标准化的ORM框架，它不像hibernate那样完全将数据库对象化，
    而是专注于sql语句的编写，因此相对于hibernate，它可以优化sql性能，入门门槛低，
    当然它也有其映射（输入映射和输出映射）；

### mybatis工作原理

![]({{ site.url }}/assets/blog_images/mybatis工作流程.jpg)

### 实际操作
    按流程图来说，无非就是配置SqlMapConfig.xml文件和编写mapper接口；

#### 配置SqlMapConfig
    添加mapper映射就OK了，如果结合Spring，那完全就自动扫描了

#### 重点放在mapper映射接口
    实现mapper接口和mapper文件xml，注意要放在同一包下；

##### mapper接口

```java
public interface UserMapper {
    User login(String username);
}
```
##### mapper xml文件

```xml
<!-- namespace要对应接口文件 -->
<mapper namespace="com.jackpon.UserMapper">
    <!-- id要和接口的方法一样 -->
    <select id="login" parameterType="com.jackpon.pojo.User" resultType="com.jackpon.pojo.User">
        select * from tb_user where username = #{username}
    </select>
</mapper>
```

### 为实体类定义别名

```xml
<!-- 
在conf.xml文件中<configuration></configuration>标签中添加如下配置：
 -->
<typeAliases>
    <typeAlias type="com.jackpon.pojo.User" alias="User"/>
</typeAliases>

---
<mapper namespace="com.jackpon.UserMapper">
    <!-- id要和接口的方法一样 -->
    <select id="login" parameterType="User" resultType="User">
        select * from tb_user where username = #{username}
    </select>
</mapper>
```