---
title: Struts2入门 > 影视网demo
categories: 
- Struts2
tags:
- Struts2
updated: 2018-09-09
---

​	

###  影视网demo 

 - **目的**
 	以项目为驱动，学习Struts2的基本知识点。
 - **项目介绍**
 	影视网demo网站，前端布局boostrap+js，后端Struts2，数据库mysql+redis；
 	实现功能：用户登录注册、我喜欢；
 	主要知识点：拦截器，比如用户在未登录状态下点击“喜欢”，将会被拦截请求，通过判断来进一步后续操作。
 	![在这里插入图片描述](https://img-blog.csdn.net/20180925160004409)
 

### Just code it
 	
 - **struts.xml** 
 	

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 2.1//EN" "http://struts.apache.org/dtds/struts-2.1.dtd">
<struts>
<!-- 实时更新到服务器 -->
 <constant name="struts.devMode" value="true" />
	<package name="movies" extends="struts-default">
		<!-- 拦截器 -->
		<interceptors>
		<interceptor name="loginStatus" class="com.jackpon.interceptors.loginStatusInterceptor"></interceptor>
			<!-- 拦截栈 -->
			<interceptor-stack name="iloginStatus">
				<!-- defaultStack一定要写，不然无法传参数 -->
				<interceptor-ref name="defaultStack" />
				<interceptor-ref name="loginStatus" />
			</interceptor-stack>
		</interceptors>
		<action name="like-add" class="com.jackpon.actions.likeAction" method="add">
				<result>./WEB-APP/like-success.jsp</result>
				<!-- 调用拦截器 -->
				<interceptor-ref name="iloginStatus"></interceptor-ref>
				<result name="login">./WEB-APP/like-error.jsp</result>
		</action>
		<action name="like-ilist" class="com.jackpon.actions.likeAction" method="ilist">
				<result>./WEB-APP/ilist.jsp</result>
		</action>
		<!-- 通配符匹配action，命名规范要做好才有用，比如页面最好唯一，下面就不是很好 -->
		<action name="*-*" class="com.jackpon.actions.{1}Action" method="{2}">
			<result>./WEB-APP/{1}-success.jsp</result>
			<result name="error">./WEB-APP/{1}-error.jsp</result>
		</action>
	</package>
</struts>    
```

 - **项目结构**
 	![在这里插入图片描述](https://img-blog.csdn.net/20180925161625419)
 	action、javabean、数据库DB、拦截器interceptors、服务service要单独建包，需要注意的是构建数据库的语句要写在文件夹mysql里，方便他人使用，引用外部的框架放在WEB-INF>lib;
 - **构建Model层**
 	model也称bean，在此demo中，我们只需要构建用户User和电影Movies，由于在注册时，有确认密码这项，我们还需要建立UserDTO，它的作用就是User的爸爸（UserDTO包含User），User就可以从UserDTO获取所需要的属性值；由于项目比较小，我们可以同时建立数据库，bean与数据库的名字最好一致；
 - **数据库服务**
 	DB.java用于启动mysql服务
```
package com.jackpon.DB;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class DB implements DBConfig{
	public static Connection createConn() {
		Connection conn = null;
		try {
			Class.forName(driver);
			conn = DriverManager.getConnection(url, user, passwd);
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} catch (SQLException e) {
			e.printStackTrace();
		}
		return conn;
	}
	
	public static PreparedStatement prepare(Connection conn, String sql) {
		PreparedStatement ps = null;
		try {
			ps = conn.prepareStatement(sql);
		} catch (SQLException e) {
			e.printStackTrace();
		}
		return ps;
	}
	
	public static void close(Connection conn) {
		
		try {
			conn.close();
			conn = null;	//将其设置为null，系统就会自动回收
		} catch (SQLException e) {
			e.printStackTrace();
		}
	}
	public static void close(Statement stmt) {
		try {
			stmt.close();
			stmt = null;
		} catch (SQLException e) {
			e.printStackTrace();
		}
	}
	public static void close(ResultSet rs) {
		try {
			rs.close();
			rs = null;
		} catch (SQLException e) {
			e.printStackTrace();
		}
	}
}

```
调用DB服务，以UserService.add()为例

```
public String add(User user) {
	 	Connection conn= (Connection) DB.createConn();
		String sql = "insert into user(name,password) values (?, ?)";
		PreparedStatement ps = (PreparedStatement) DB.prepare(conn, sql);
		try {
			ps.setString(1, user.getName());
			ps.setString(2, user.getPassword());
			ps.executeUpdate();
		} catch (SQLException e) {
			e.printStackTrace();
			return ERROR;
		}finally {
			DB.close(ps);
			DB.close(conn);
		}
		return SUCCESS;
	}
```

 - **拦截器**

```
package com.jackpon.interceptors;

import java.util.Map;

import com.opensymphony.xwork2.Action;
import com.opensymphony.xwork2.ActionContext;
import com.opensymphony.xwork2.ActionInvocation;
import com.opensymphony.xwork2.interceptor.AbstractInterceptor;

public class loginStatusInterceptor extends AbstractInterceptor{

	@Override
	public String intercept(ActionInvocation invocation) throws Exception {
		Map session=invocation.getInvocationContext().getSession();
        if (session.get("username")==null) {
			return Action.LOGIN;
		}else {
			//表示执行成功，继续下一个拦截器或action
			return invocation.invoke();
		}
	}

}
```
[movies](https://github.com/Jackpon/Struts2Demos/tree/master/movies)