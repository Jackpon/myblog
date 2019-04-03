---
title: 第一章 小试牛刀
categories: 
- Linux_Shell
tags:
- Linux_Shell_脚本攻略
updated: 2018-06-11
---

​	

#### 前言
  ​	本系列文章是基于《Linux Shell 脚本攻略》 的知识点以及个人的一些想法构成，旨在写成简单明了的shell教程，也希望通过 Learning by doing 学会shell基本操作，由于水平有限，如果写错也在所难免，希望读者指正。

#### 目标
  ​	学完本章你将对shell有基本的了解，学会编写简单的shell脚本，例如对文件的操作，还将理解鼎鼎有名的Fork炸弹；以及编写一个简单的程序，实现直至文件下载成功才停止运行，这对于现实工作应该是非常有用的小技巧了。

  ---

  

#### 简介
  ​	什么是 shell ？你可以理解为，shell就是一门用于高效处理文件的脚本语言，当然远不止于此；shell是什么并不重要，重要的是我们可以用它来做什么。直接上代码吧，Learning  by  doing！

  ```shell
  #输出Hello World
  vim test.sh	#建立test脚本
  
  #这句的意思就是告诉系统，我将调用bash程序，也就是shell
  #!/bin/bash		
  echo "Hello World"
  
  #保存退出
  chmod +x test.sh #改变test权限，使其可执行
  ./test.sh	#你将在终端看到Hello World
  
  #课后题，测试下以下代码区别
  num=123
  echo "$num"
  echo '$num'
  ```
  写完上面的程序，基本上也对shell有个印象，接下来就跟学习其他语言差不多了，让我们来了解shell 的基本知识。

#### 基础

- 定义一个*变量*：
  ```shell
  num=1	#跟python很类似，直接定义就行，不需要写类型，注意“=”
  declare -i num	#当然也可以用这种方式
  str='hello'	#字符串，用双引号也可以
  #数组
  array=(1 2 3)	#数组，元素之间用空格隔开
  declare -a array=([1]=1 [2]=2 [hello]=3)	#“索引-值”数组
  ${array[*]}		#表示访问全部元素
  ${#array[*]}	#表示数组元素个数
  #我们还可以这么操作
  array=(${array[*]} 'world') #向数组添加元素
  ```

  

- *if语句*
  ```shell
  a=1
  b=2
  if [ $a -lt $b ] && [ $a -eq 1 ]
  #引用变量需要用"$"符号，可以写成 $a ，也可以 ${a},
  #复杂语句建议用后者,“and/or”==>"&&/||"
  then
  	echo "a<b" 
  elif [ $a -eq $b ]
  then
  	echo "a=b"
  else
  	echo "a>b"
  fi
  
  
  #比较符号
  -lt #less then		<
  -le #less equal 		<=
  -gt #greater then 	>
  -ge #greater equal 	>=
  -eq #equal 			=
  -ne #not equal 		!=
  ```
  可以看到，样式跟其他语言很不一样，“(  )”改成“[  ]”；而且两边还要有空格隔开；
  if 后要紧随 then，结束要用 fi ;
  

- *for 和 while 循环*

```shell
a=(1 2 3)
for i in ${a[*]}
	do
		echo $i
	done
	
i=1
while [ $i -le 5 ]
	do
		echo $i
		let i++
	done
```

可以看到，两个循环语句都始于do，终于done；让数字做加减可以用 let 语句；



- *数学运算*

```shell
a=1
b=2
c=$a+$b	#这种操作是错误的，以下才是正确操作
let c=$a+$b
((c=$a+$b))
c=$(($a+$b))	#这些方法只能用于整数，不能用于浮点数
#bc是一个用于数学运算的高级工具，或许我们可以试试
```



- *重定向*
```shell
echo "Hello world" > 1.txt		#将字符串覆盖写入到文件
echo "Who are you?" >> 1.txt	#将字符串追加写入到文件
cat 1.txt 2.txt > 3.txt		    #读取文件1和2的内容，并将其写入到文件3
```



- *别名*
```shell
alias install='sudo apt install '	#为命令起别名，下次直接install就行了
```



- *时间*

```shell
date		#直接获取时间
date +%s 	#获取纪元时
#有时候我们要统计下程序运行多久，我们可以这样
#!/bin/bash
start=$(date +%s)
i=1
while [ $i -le 10 ]
do
	echo $i
	let i++
	sleep 1
done
end=$(date +%s)
diff=$(($end-$start))
echo "Time: $diff s"
```

- *函数*
```shell
hello(){
    echo "hello"
}
hello	#直接调用函数名字就行
#如果有参数，又该怎么做
hello(){
    echo "hello $1,welcome to $2"
}
hello Jack BeiJing
```
是的，在shell中 ‘$1’ 就是代表第一个参数，以此类推；shell 也可以编写递归函数，其中最典型的案例就是Fork炸弹，其本质其实就是调用本身，几何增长地产生进程，最终导致服务器崩溃！所以不要轻易尝试。

---

#### 拓展应用
- *Fork炸弹*
```shell
:(){ :|:& };:		#史上最简的Fork炸弹就是这样啦
```
看起来很像外星文，其实只要这样分解，就很容易看得懂了；
```shell
#调用函数本身，然后利用管道调用一个新进程，&就是我们linux用于将进程放在后台的操作
:(){
	:|:&	
};
:
```
防范其实也简单，因为该程序其实就是产生大量进程导致系统崩溃，那么我们只需要限制最大进程数就行。
```shell
ulimit -u 50 	#设置最大进程数为 50
#当然还可以在配置 /etc/security/limits.conf 来限制最大进程数
```

- *下载文件*
```shell
#!/bin/bash
repeat(){
	while true
	do
		$@ && return 	# $@ 代表传递的语句,如果执行成功就返回退出
	done
}
```
由于true是作为 /bin 的一个二进制文件来实现的，这就意味着每执行一次，就要产生一个进程；
所以我们可以改为shell内建的 “ : ” ；我们还应该添加延时操作，不然服务器会任务你在发送垃圾信息，可能把你拉黑。
```shell
#最终版
repeat(){ while :; do $@ && return ; sleep 30; done }
#调用
repeat wget -c http://www.example.com/software.tar.gz
```

- *批处理文件名*
```shell
#!/bin/bash
f="format" #format为修改后的格式，例如mp4
cd path #path为要修改的文件目录
for file in $(ls *)
do
	name=$(ls $file | cut -d . -f 1) #获取文件名
	mv $file $name.$f
done
```