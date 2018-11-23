---
layout:     post                    # 使用的布局（不需要改）
title:      Kotlin Study（-）               # 标题 
subtitle:   Hello World, Hello Blog #副标题
date:       2016-02-29             # 时间
author:     fantianwen                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - git
---

# 快速上手

## function定义

### 具备返回类型的方法

```
fun sumDemo2(a: Int, b: Int): Int {
    ...
    return ...
}
```

如果方法体重只有一行，那么也可以这样写：
	
```
fun sumDemo(a: Int, b: Int): Int = a + b
```
### 推断类型（inferred）返回的方法

```
fun sumDemo1(a: Int, b: Int) = a + b
```
	
### 如果不能直接写成一行，那么方法默认需要一个返回类型
### 没有返回类型的方法（在java中使用void修饰的方法）

```
fun funDemo(str: String): Unit {
    println("this is a demo for kotlin" + str)
}
```
	
这里，`Unit`也可以省略

<!-- more -->

## 变量定义

- 只读的变量定义（read-only）：

```
val a=1
a+=1
```
上面的片段在编译的时候会`Error:(24, 5) Kotlin: Val cannot be reassigned`
	
表明`val`只能声明`read-only`的变量
	
### 可变变量的定义
 
```
var a=1
a+=1
```	
	
### 模板的使用

```
val strings: ArrayList<String> = arrayListOf("1", "2")
println("${strings[0]}")
```
	
注意其中`ArrayList`的定义
	
### 简化的`if else`

```
fun ifDemo(a: Int, b: Int): Int = if (a > b) a else b
```	
	
其实和“三元运算符”也没差
	
### nullable的值和`null`的检查

```
fun nullDemo(str: String): Int? {
    return null
}
```	
	
如上，这段代码会返回字符串`null`
	
如果希望在判断出“null”的时候抛出异常
	
```
fun checkNull(str: String?): String {
return str ?: throw NullPointerException("nollllll")
}
```
	
这样设置参数的时候，其实就已经默认该方法在传入`null`的时候编译器不做检查，并且在判断出`null`的时候抛出异常
	
### 类型检查和类型自动装换

	使用`is`和`!is`进行类型检查，如果一个可变的参数的类型已经是确定了的，那么就没有必要显示的进行转化
	
	
### `for`循环

```
fun forLoopTest(obj: Any): Unit {
	
    if (obj is Iterable<*>) {
        for (o in obj) {
            print("==" + o)
        }
    } else {
        print("${obj} is not iterable")
    }
	
}
```	
	
### `when`
	
可以使用`when`代替`switch  case`语句
	
```
fun whenDemo(obj: Any): Unit {
    when (obj) {
        1 -> println("one")
        3 -> println("three")
        "1" -> {
            val nn = obj + "this is amazing!"
            println(nn)
        }
        else -> {
            println("unknown")
        }
    }
}
```	
	
这个比`switch  case`好用

### `range`
	
使用`..`生产序列
	
```
val ra = 1..5

for (r in ra) {
   println(r)
}
```
	
### 使用`in`检查一个`collection`中是否包括某个值
	

### 拓展类的方法

```
fun String.fa() {
println(this + "oh! FA...")
}
```	

### return的值的类型

能够直接返回`Exception`
	
		
# 基础

## `===`和`==`

- `===`：验证`identity`
- `==`：验证`equality`

较小的类型不能向较大的类型进行隐式的转换（如：Byte->Int）
但是每种类型的对象均有下面的方法：

	 
	 — toByte(): Byte	 — toShort(): Short — toInt(): Int	 — toLong(): Long	 — toFloat(): Float 
	 — toDouble(): Double 
	 — toChar(): Char
	 
	
## 三种loop具备iterable的对象的方法
	
- 使用value直接loop
	
```
val array0: Array<String> = arrayOf("1", "2", "3", "4", "5")
for (value in array0) {
   println("item is ${value}")
}
```
	
- 使用index进行loop
	
```
val array1: Array<String> = arrayOf("this", "is", "fucking", "awesome")
for (i in array1.indices) {
   println("${i}'s position's value is \"${array1[i]}\"")
}
```
	
- 类似于collection（Java）的遍历方法

```
val array2: Array<String> = arrayOf("a", "b", "c", "d", "e", "f")
for ((index, value) in array2.withIndex()) {
   println("${index}'s position's value is ${value}")
}
```
	
> 使用`$`符号的时候，如果只有一个参数，那么我们只需要`$value`这样就行了

## 返回和跳转
	
- break
	
```java
loopDemo@ for (i in 1..5) {
   println("this is value ${i}")
   if (i == 3) {
       break@loopDemo
   }
}
```
	
给需要break的方法体之前赋予一个别名`name`，并且在该别名的后后面加上`@`，在需要break的时候，直接`break@name`即可
	
	
	

	
	
	

	
	



