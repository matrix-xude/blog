---
title: 序列化
categories:
  - Java
tags:
  - Java
date: 2020-09-20 20:15:39
---

#### 什么序列化

1. 由于在系统底层，数据的传输形式是简单的字节序列形式传递，即在底层，系统不认识对象，只认识字节序列，而为了达到进程通讯的目的，需要先将数据序列化，而序列化就是将对象转化字节序列的过程。相反地，当字节序列被运到相应的进程的时候，进程为了识别这些数据，就要将其序列化，即把字节序列转化为对象。

2. 无论是在进程间通信、本地数据存储又或者是网络数据传输都离不开序列化的支持。而针对不同场景选择合适的序列化方案对于应用的性能有着极大的影响。

3. 从广义上讲，数据序列化就是将数据结构或者是对象转换成我们可以存储或者传输的数据格式的一个过程，在序列化的过程中，数据结构或者对象将其状态信息写入到临时或者持久性的存储区中，而在对应的反序列化过程中，则可以说是生成的数据被还原成数据结构或对象的过程。

4. 这样来说，数据序列化相当于是将我们原先的对象序列化概念做出了扩展，在对象序列化和反序列化中，我们熟知的有两种方法，其一是Java语言中提供的Serializable接口，其二是Android提供的Parcelable接口。而在这里，因为我们对这个概念做出了扩展，因此也需要考虑几种专门针对数据结构进行序列化的方法，如现在那些个开放API一般返回的数据都是JSON格式的，又或者是我们Android原生的SQLite数据库来实现数据的本地存储，从广义上来说，这些都可以算做是数据的序列化。

**序列化** --> 将数据结构或对象转换成二进制串的过程
**反序列化** --> 将在序列化过程中所生成的二进制串转换成数据结构或者对象的过程



#### 如何实现序列化

 常用的Object序列化类：ObjectInputStream` 和 `ObjectOutputStream`

1. 必须实现 `Serializable` 接口才能使用，否则报错
2. 不能是内部类，找不到`Serializable`接口
3. 如果成员变量中有普通Object类，也需要实现`Serializable`接口
4. String默认实现了`Serializable`接口，是特殊的类



#### `serialVersionUID`是什么

1. 实现了`Serializable` 的类，如果不自定义此字段，则默认生成一个
2. 当此字段修改了，反序列化会失败
3. 可以用来当做版本控制



#### 实现Serializable接口中的4个特殊方法

可以特殊处理某些类

```kotlin
class FixedStar : Serializable {
    companion object {
        const val serialVersionUID = 1L
    }

    var name: String? = null
    var lifecycle: Int? = null
    var burst: String? = null

    override fun toString(): String {
        return "FixedStar(name=$name, lifecycle=$lifecycle 亿年, burst=$burst)"
    }

    // 以下4个是通过反射调用的方法，用来在序列化的时候做自己的处理
    private fun readObject(ois: ObjectInputStream) {
        println("readObject")
        name = ois.readObject() as String
        lifecycle = ois.readInt()
        burst = ois.readObject() as String
    }

    private fun writeObject(oos: ObjectOutputStream) {
        println("writeObject")
        oos.writeObject(name)
        oos.writeInt(lifecycle!!)
        oos.writeObject(burst)
    }

    private fun readResolve(): Any {
        println("readResolve")
        return FixedStar().apply {
            name = "${this@FixedStar.name}_readResolve"
            lifecycle = this@FixedStar.lifecycle!! + 3
            burst = "${this@FixedStar.burst}_readResolve"
        }
    }

    private fun writeReplace(): Any {
        println("writeReplace")
        return FixedStar().apply {
            name = "${this@FixedStar.name}_writeReplace"
            lifecycle = this@FixedStar.lifecycle
            burst = "${this@FixedStar.burst}_writeReplace"
        }
    }
}

// 输出结果
writeReplace
writeObject
readObject
readResolve
FixedStar(name=北极星_writeReplace_readResolve, lifecycle=25 亿年, burst=永不爆发_writeReplace_readResolve)
```




#### Externalizable

```java
void writeExternal(ObjectOutput out) throws IOException;
void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;
```

通过覆写上面2个方法自己决定需要序列化哪些对象



#### 序列化的底层实现

[存储的详细格式参考](https://blog.csdn.net/fuhao_ma/article/details/102969349)

```kotlin
data class Deep(var index: Int, var name: String) : Serializable {
    companion object {
        const val serialVersionUID = 1L
    }
}

val deep = Deep(1, "fff")
val str = "aaa"
```

以下是对**deep**进行序列化到本地文件的十六查看（Hex）
00000000: aced 0005 7372 0025 636f 6d2e 7878 642e  ....sr.%com.xxd.
00000010: 7468 7265 6164 2e65 6e63 6f64 652e 456e  thread.encode.En
00000020: 636f 6465 4465 6570 2444 6565 7000 0000  codeDeep$Deep...
00000030: 0000 0000 0102 0002 4900 0569 6e64 6578  ........I..index
00000040: 4c00 046e 616d 6574 0012 4c6a 6176 612f  L..namet..Ljava/
00000050: 6c61 6e67 2f53 7472 696e 673b 7870 0000  lang/String;xp..
00000060: 0001 7400 0366 6666                      ..t..fff

**str**(Hex)
00000000: aced 0005 7400 0361 6161                 ....t..aaa



#### 底层实现的一些特殊字段

具体可以查看`ObjectStreamConstants`类的常量

73 ：代表一个新对象的开始，下面必须接对象所属类的描述

72 ：代表一个新类的开始，下接类名长度、`serialVersionUID`、02（|操作判断该类实现了哪些方法）、

​			03（类中域的个数）、然后是每个域引用的描述（比较复杂）、78（类描述数据块的结束）

78 : 类描述的结束（会开始父类描述，直到没有父类为止）

70 ：接在类（72）描述后表示没有父类; 在赋值的时候表示为null

74 ： 代表NEW STINRG的开始，`java.io.ObjectStreamConstants.TC_STRING 对应这个常量`，先接一个长度，再接数据

71 ： 地址引用类型，一般用来引用之前描述过的类型

域的结构：

1. 域的类型（1个字节，如：4c -> 引用类型、49-> int类型、5a->boolean类型）
2. key的长度（2个字节）
3. key的名字（对应上面的长度）
4. 如果是4c类型则开始描述引用的数据，一般有 74（Ljava.lang.String; Lcom/mfh/ser/Men;）和 71（**00 7E 00 01**：代表引用地址，上面已经出现过String类型，所以引用地址指向了他）2种描述



#### 反序列化创建类

包名必须相同、`serialVersionUID `必须相同，（不是通过构造函数创建，所以私有化构造函数不中）

Java中创建对象的四种方法 
1. 用new语句创建对zhi象，这是dao最常见的创建对象的方法。
2. 运用反射手段,调用java.lang.Class或者java.lang.reflect.Constructor类的newInstance()实例方法。
3. 调用对象的clone()方法。
4. 运用反序列化手段，调用java.io.ObjectInputStream对象的 readObject()方法。