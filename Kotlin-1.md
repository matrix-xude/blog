---
title: Kotlin基础学习1
categories:
  - Android
  - Kotlin
tags:
  - Kotlin
date: 2020-05-22 04:48:49
---

本文只讲述`Kotlin`与`Java`区别大的地方，如需要详细了解细节，可到`Kotlin`官网查询
[Kotlin官网](https://kotlinlang.org/docs/reference/ )
[Kotlin中文网](https://www.kotlincn.net/docs/reference/)

此外，github上还有使用架构的项目集合，有很多Kotlin语言写的项目，并综合了jetpack，可以给时间充裕的朋友提供很大帮助。
[GitHub项目集合](https://github.com/android/architecture-components-samples)         关键字：**architecture**   

##### 关键字 `val` `var`

```kotlin
val a: Int = 1;  // 只读 val -> value 值
var b: Int = 2;  // 可读可写 var -> variable 易变的
```

可以用 Tools -> Kotlin -> Show Kotlin ByteCode 拿到编译后的语言，再 decompile,得到反编译后的文件

```java
public final class MyClass {
	private final int a = 1;  // 对应 a，成为了 final 但是 val 和 final 还是有点区别
	private int b = 2;

	public final int getA() {  // A没有set 方法
		return this.a;
	}

	public final int getB() {
		return this.b;
	}

    public final void setB(int var1) {
        this.b = var1;
    }
}
```



##### 自动类型推断

```kotlin
var c = 1L // 自动推断
//c = 1 错误，自动推断只能一次
```



##### `？`关键字

表示可以为null, Int?  不等于  Int

```kotlin
var d: Int? = 1
var e: Int = 2
// d = e  不能赋值，为什么？
```

继续 decompile, 可以看出 Int?是Integer类型，而Int则是int基本类型

```java
@Nullable
private Integer d = 1;
private int e = 2;
```

**Tip:遇到不理解的可以多使用 decompile,比如在遇到泛型问题的时候也可以通过反编译找答案**



##### 默认值问题

```kotlin
// var f: Int 不能通过检查，kotlin中所有变量都没有默认值
// java中的成员变量有默认值，而局部变量没有默认值
```



##### 字符串模板

${ 可以执行代码 }

```kotlin
private val price = 45.3
private val str1 = "《星空的琴弦》价格是：$price, 打特价后价格为：${price - 10}"
// 《星空的琴弦》价格是：45.3, 打特价后价格为：35.3

private val str2 = """  // 无缩进
    第一行 
       第二行
随便来
""".trimIndent()
```



##### 顶部函数

```kotlin
// Unit 等于java中的void，可以省略
fun function(): Unit {

}

// 参数在（）中，返回值在后面
fun function2(i: Int): String {
    return ""
}

// 函数体只有一行，并且能自动推断出返回类型
fun add(x: Int, y: Int) = x + y
```

decompile之后看到对应了java中的static函数，java中函数只能定义在类中

```java
public static final void function() {
}

@NotNull
public static final String function2(int i) {
	return "";
}

public static final int add(int x, int y) {
	return x + y;
}
```



##### 构造函数

```kotlin
// ()是主构造函数,可以用val这种变量，可以给默认的参数
class User(val id: Int = 0) {

    var name: String

    init {  // 主构造函数调用，初始化代码块，对应java中的{}代码块
        name = ""
    }

    init {  // 可以写多init,类似java中可以写多个代码块

    }

    constructor(id: Int, name: String) : this(id) { // 次构造函数，必须调用主构造函数 this()
        this.name = name
    }
}
```



##### 属性、字段、幕后字段

```kotlin
class Person {

    // 此处id是一个“属性”，java中成员变量id是一个“字段”
    // 属性 + get + set = 字段
    var id: Int = 0
        get() = 0
        set(value) {
            field = value  // 幕后字段
        }

    private var name2: String = name
    var name: String // 不保存状态
        get() = ""
        set(value) {
            name2 = name  // 这里保存在name2上，没有定义保存在幕后字段上
        }
}
```

```kotlin
fun main() {
    val person = Person()
    person.name = "星空"  // 用name给私有的name2
}
```

何时会生成幕后字段？
对于var|val属性
Kotlin自动生成了   setter |getter                                       产生幕后字段
重写了                     setter |getter                                       不产生幕后字段
在重写了的             setter |getter  中使用了field               产生幕后字段



##### `lateinit`

```kotlin
lateinit var name: String  // 延迟初始化，只能修饰var,如果没有初始化调用直接调用会报错
// lateinit val id = 0; // val不能延迟加载
// lateinit var price: Double = 1.0 // java中对应的八种基本类型不能延迟加载
```

```kotlin
fun isNameInitialize(): Boolean {  // 只能在内部中判断name是否初始化，外部拿不到
    return ::name.isInitialized
}
```



##### `data`关键字

被data修饰的类用decompile后，不但会生成字段、get、set方法，还会生成析构函数

```kotlin
data class Basic4(var id: Int = 0, val name: String = "", var describe: String?) {

}

fun main() {
    val basic4 = Basic4(11, "doIt", "just do it")

    // 析构函数用法
    val (id2: Int, _, desc: String?) = basic4  // 赋值回 id2,desc  _ 用来站位
    println("$id2 $desc")
}
```