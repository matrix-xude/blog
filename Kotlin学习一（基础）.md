---
title:Kotlin学习
categories:
  - Kotlin
tags:
  - Kotlin
date: 2020-05-21 14:27:52

---

# Kotlin学习一（基础）  

***

## 基本类型  
在 Kotlin 中，所有东西都是**对象**，在这个意义上讲我们可以在任何变量上调用成员函数与属性。  

来对比kotlin与java中`int`转为`String`的方法，可以看出2种语言的思想上的不同，kotlin中万物皆对象。  
```kotlin
	val a = 1;
	val s = a.toString()  // 因为Int是对象，所以kotlin中可以直接调用toString方法
```
```java	
	int a = 1;
	String s = String.valueOf(a);
	// a.toString();  java中基本类型不是对象，没有此函数
```
一些类型可以有特殊的内部表示——例如，数字、字符以及布尔值可以在运行时表示为原生类型值，但是对于用户来说，它们看起来就像普通的类。  



先放入官方链接，[Kotlin基础][基础url]，详细学习参考Kotlin中文官网，这里只记录下与其他语言的不同之处。



1. 注意，与一些其他语言不同，Kotlin 中的数字没有隐式拓宽转换:  
```kotlin
	var c = 1 // Int类型
	val d = 1L // Long类型
	// c = d  隐式转换报错
	c = d.toInt() // 显式转换可以
```
2. 可以使用下划线使数字常量更易读：  
```kotlin
	val oneMillion = 1_000_000
	val creditCardNumber = 1234_5678_9012_3456L
	val hexBytes = 0xFF_EC_DE_5E
```
3. 对于位运算，没有特殊字符来表示，而只可用中缀方式调用具名函数:
    这是完整的位运算列表**（只用于`Int`  与 ` Long`)**

|Kotlin中函数|语义|对应java中运算|
|:---:|:---:|:---:|
|`shl(bits)`| – 有符号左移 | `<<`|
|`shr(bits)`| – 有符号右移 | `>>`|
|`ushr(bits)`| – 无符号右移|  `>>>`|
|`and(bits)`|– 位与| `&`|
|`or(bits)`| – 位或 | `|`|
|`xor(bits)`| – 位异或|  `^`|
|`inv()` |– 位非| `~`  |
4. 字符用`Char` 类型表示。它们不能直接当作数字:
字符字面值用单引号括起来: `'1'`。 特殊字符可以用反斜杠转义。 支持这几个转义序列：`\t`、 `\b`、`\n`、`\r`、`\'`、`\"`、`\\` 与 `\$`。 编码其他字符要用 Unicode 转义序列语法：`'\uFF00'`。
```kotlin
	fun check(c: Char) {
		// if (c == 1) { }  编译报错，类型不兼容
        if (c in '1'..'9') {
            throw IllegalArgumentException("Out of range")
        }
        println(c.toInt() - '1'.toInt())
    }
```
***

  


## 包与导入

源文件通常以包声明开头:

```kotlin
/**
 * 包的声明应处于源文件顶部。
 *
 * 目录与包的结构无需匹配：源代码可以在文件系统的任意位置。
 * 如：包名改为 "package com.hetu.kotlinfirst.grammar2" 也可以正常运行，只会提示 "Package directive doesn't match file location"
 * 而java中包名必须与名录匹配
 */
package com.hetu.kotlinfirst.grammar
```



有多个包会默认导入到每个 Kotlin 文件中：

- [kotlin.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/index.html)
- [kotlin.annotation.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/index.html)
- [kotlin.collections.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/index.html)
- [kotlin.comparisons.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.comparisons/index.html) （自 1.1 起）
- [kotlin.io.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.io/index.html)
- [kotlin.ranges.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.ranges/index.html)
- [kotlin.sequences.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.sequences/index.html)
- [kotlin.text.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/index.html)

根据目标平台还会导入额外的包：

- JVM:
  - java.lang.*
  - [kotlin.jvm.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.jvm/index.html)
- JS:
  - [kotlin.js.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.js/index.html)

如果出现名字冲突，可以使用 as 关键字在本地重命名冲突项来消歧义
```kotlin
import org.example.Message // Message 可访问
import org.test.Message as testMessage // testMessage 代表“org.test.Message”
```

***



## 控制流

 **If 表达式**

 在 Kotlin 中，*if*是一个表达式，即它会返回一个值。 因此就不需要三元运算符（条件 ? 然后 : 否则），因为普通的 *if* 就能胜任这个角色。

 ```kotlin
 // 作为表达式
 val max = if (a > b) a else b
 ```
 *if* 的分支可以是代码块，最后的表达式作为该块的值：

 ```kotlin
 val max = if (a > b) {
     print("Choose a")
     a
 } else {
     print("Choose b")
     b
 }
 ```

 如果你使用 *if* 作为表达式而不是语句（例如：返回它的值或者把它赋给变量），该表达式需要有 `else` 分支。




 **When 表达式**

 *when* 取代了类 C 语言的 switch 操作符。其最简单的形式如下：

 ```kotlin
 when (x) {
     1 -> print("x == 1")
     2 -> print("x == 2")
    else -> { // 注意这个块
         print("x is neither 1 nor 2")
     }
 }
 ```

 *when* 将它的参数与所有的分支条件顺序比较，直到某个分支满足条件。 *when* 既可以被当做表达式使用也可以被当做语句使用。如果它被当做表达式， 符合条件的分支的值就是整个表达式的值，如果当做语句使用， 则忽略个别分支的值。（像 *if* 一样，每一个分支可以是一个代码块，它的值是块中最后的表达式的值。）

 如果其他分支都不满足条件将会求值 *else* 分支。 如果 *when* 作为一个表达式使用，则必须有 *else* 分支， 除非编译器能够检测出所有的可能情况都已经覆盖了［例如，对于 [枚举（*enum*）类](https://www.kotlincn.net/docs/reference/enum-classes.html)条目与[密封（*sealed*）类](https://www.kotlincn.net/docs/reference/sealed-classes.html)子类型］。

 如果很多分支需要用相同的方式处理，则可以把多个分支条件放在一起，用逗号分隔：

 ```kotlin
 when (x) {
     0, 1 -> print("x == 0 or x == 1")
     else -> print("otherwise")
 }
 ```

 自 Kotlin 1.3 起，可以使用以下语法将 *when* 的主语（subject，译注：指 `when` 所判断的表达式）捕获到变量中：

 ```kotlin
 fun Request.getBody() =
         when (val response = executeRequest()) {
             is Success -> response.body
             is HttpError -> throw HttpException(response.status)
         }
 ```



 **For 循环**

 *for* 循环可以对任何提供迭代器（iterator）的对象进行遍历，这相当于像 C# 这样的语言中的 `foreach` 循环。语法如下：

 ```kotlin
 for (item in collection) print(item)
 
 for (item: Int in ints) {
 // 循环体可以是一个代码块
 }
 ```

 如上所述，*for* 可以循环遍历任何提供了迭代器的对象。即：

 + 一个成员函数或者扩展函数   `iterator()`，它的返回类型
 + 一个成员函数或者扩展函数 `next()`，并且
 + 一个成员函数或者扩展函数 `hasNext()` 返回 `Boolean`。
 + 三个函数都需要标记为 `operator`。

 如需在数字区间上迭代，请使用[区间表达式](https://www.kotlincn.net/docs/reference/ranges.html):

 ```kotlin
 for (i in 1..3) {
     println(i)
 }
 for (i in 6 downTo 0 step 2) {
     println(i)
 }
 ```

 如果你想要通过索引遍历一个数组或者一个 list,或者你可以用库函数 `withIndex`：

 ```kotlin
 for (i in array.indices) {
     println(array[i])
 }
 
 for ((index, value) in array.withIndex()) {
     println("the element at $index is $value")
 }
 ```



 **While 循环**

 *while* 与 *do*..*while* 照常使用

 ```kotlin
 while (x > 0) {
     x--
 }
 
 do {
   val y = retrieveData()
 } while (y != null) // y 在此处可见
 ```

****



## 返回与跳转

Kotlin 有三种结构化跳转表达式：

- *return*。默认从最直接包围它的函数或者[匿名函数](https://www.kotlincn.net/docs/reference/lambdas.html#匿名函数)返回。
- *break*。终止最直接包围它的循环。
- *continue*。继续下一次最直接包围它的循环。

所有这些表达式都可以用作更大表达式的一部分：

```kotlin
val s = person.name ?: return  // 如果name为null，直接返回
```

在 Kotlin 中任何表达式都可以用标签（*label*）来标记。 标签的格式为标识符后跟 `@` 符号，例如：`abc@`、`fooBar@`都是有效的标签（参见[语法](https://www.kotlincn.net/docs/reference/grammar.html#label)）。 要为一个表达式加标签，我们只要在其前加标签即可。

因为label不常用，所以只附上链接 [label使用](https://www.kotlincn.net/docs/reference/returns.html#break-%E4%B8%8E-continue-%E6%A0%87%E7%AD%BE)

****

[基础url]:https://www.kotlincn.net/docs/reference/basic-types.html#%E5%9F%BA%E6%9C%AC%E7%B1%BB%E5%9E%8B

