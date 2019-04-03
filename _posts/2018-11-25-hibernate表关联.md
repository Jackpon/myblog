---
title: Hibernate相关
categories: 
- SSH
tags:
- SSH
updated: 2018-11-25
---

### hibernate核心接口
    session：负责被持久化对象CRUD操作
    sessionFactory:负责初始化hibernate，创建session对象
    configuration:负责配置并启动hibernate，创建SessionFactory
    Transaction:负责事物相关的操作
    Query和Criteria接口：负责执行各种数据库查询

### hibernate工作原理：
    1.通过Configuration config = new Configuration().configure();//读取并解析hibernate.cfg.xml配置文件
    2.由hibernate.cfg.xml中的<mapping resource="com/xx/User.hbm.xml"/>读取并解析映射信息
    3.通过SessionFactory sf = config.buildSessionFactory();//创建SessionFactory
    4.Session session = sf.openSession();//打开Sesssion
    5.Transaction tx = session.beginTransaction();//创建并启动事务Transation
    6.persistent operate操作数据，持久化操作
    7.tx.commit();//提交事务
    8.关闭Session
    9.关闭SesstionFactory
### Hibernate缓存包括两大类
    Hibernate一级缓存又称为“Session的缓存”，它是内置的，意思就是说，只要你使用hibernate就必须使用session缓存。由于Session对象的生命周期通常对应一个数据库事务或者一个应用事务，因此它的缓存是事务范围的缓存。在第一级缓存中，持久化类的每个实例都具有唯一的OID。 
    Hibernate二级缓存又称为“SessionFactory的缓存”，由于SessionFactory对象的生命周期和应用程序的整个过程对应，因此Hibernate二级缓存是进程范围或者集群范围的缓存，有可能出现并发问题，因此需要采用适当的并发访问策略，该策略为被缓存的数据提供了事务隔离级别。第二级缓存是可选的，是一个可配置的插件，在默认情况下，SessionFactory不会启用这个插件。
---

### Hibernate表关联

#### 一对一单向外键关联

```java
@Entity
public class Husband {
	private int id;
	private String name;
	private Wife wife;
	@Id
	@GeneratedValue
	public int getId() {
		return id;
	}
	public String getName() {
		return name;
	}
	@OneToOne
	@JoinColumn(name="wifeId")
	public Wife getWife() {
		return wife;
	}
}
```
#### 一对一双向外键关联

```java
@Entity
public class Husband {
	private int id;
	private String name;
	private Wife wife;
	@Id
	@GeneratedValue
	public int getId() {
		return id;
	}
	public String getName() {
		return name;
	}
	@OneToOne
	@JoinColumn(name="wifeId")
	public Wife getWife() {
		return wife;
	}
}

@Entity
public class Wife {
	private Husband husband;
	private int id;
	private String name;
	@OneToOne(mappedBy="wife") //双向必设mappedBy，不然会有冗余字段，会在另一方生成字段wife
	public Husband getHusband() {
		return husband;
	}
	@Id
	@GeneratedValue
	public int getId() {
		return id;
	}
	public String getName() {
		return name;
	}
}
```

#### 小结
一对一的单向和双向关联在数据库的建表是一样的，其主要区别在逻辑不一样，例如单向只能通过A找到B，而双向则A、B可以相互找到；实现也不一样，单向只需在其中一个设置@OneToOne，而双向则需要在另一个设置@OneToOne(mappedBy="xx") 

---

#### 一对多单向关联

```java
@Entity
public class Husband {
	private int id;
	private String name;
	private Set<Wife> wifeGroup=new HashSet<Wife>();
	@Id
	@GeneratedValue
	public int getId() {
		return id;
	}
	public String getName() {
		return name;
	}
	@OneToMany
	@JoinColumn(name="husbandGroup")//会在wife表建立字段husbandGroup
	public Set<Wife> getWifeGroup() {
		return wifeGroup;
	}
}
```
#### 多对一单向关联

在多的一方配置@ManyToOne

```java
@Entity
public class Wife {
	private Husband husband;
	private int id;
	private String name;
	@ManyToOne
	public Husband getHusband() {
		return husband;
	}
	@Id
	@GeneratedValue
	public int getId() {
		return id;
	}
	public String getName() {
		return name;
	}
}
```
#### 小结

	同理，两者在数据库建的表一致，不同的是逻辑。一对多、多对一，都会在多的一方建立字段。

---

#### 组件映射
	@Embedded
 
```java
@Entity
public class Husband {
	private int id;
	private String name;
	private Wife wife;//普通的bean
	@Id
	@GeneratedValue
	public int getId() {
		return id;
	}
	public String getName() {
		return name;
	}
	@Embedded
	public Wife getWife() {
		return wife;
	}
}
```