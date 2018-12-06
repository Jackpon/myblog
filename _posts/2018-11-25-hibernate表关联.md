---
title: Hibernate表关联
categories: 
- SSH
tags:
- SSH
updated: 2018-11-25
---

- 一对一单向外键关联

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
- 一对一双向外键关联

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

---
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
- 小结
一对一的单向和双向关联在数据库的建表是一样的，其主要区别在逻辑不一样，例如单向只能通过A找到B，而双向则A、B可以相互找到；实现也不一样，单向只需在其中一个设置@OneToOne，而双向则需要在另一个设置@OneToOne(mappedBy="xx") 

---


- 一对多单向关联

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
- 多对一单向关联

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
- 小结

	同理，两者在数据库建的表一致，不同的是逻辑。一对多、多对一，都会在多的一方建立字段。

---
- 组件映射

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