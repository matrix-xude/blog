---
title: Java泛型
categories:
  - Java
tags:
  - Java
date: 2020-05-21 14:27:52

---

#### 什么是泛型

Java泛型（Generics）是JDK 1.5 引入的一个新特性，泛型提供了编译时类型安全检测机制，该机制允许程序员在编译时检测到非法的类型。泛型的本质是**参数化类型**，也就是说所操作的数据类型被指定为一个参数。泛型不存在于JVM虚拟机。

#### 泛型的作用

1. 代码更健壮 (只要编译期没有警告，那么运行期就不会出现 ClassCastException)

2. 代码更简洁 (不用强转)

3. 代码更灵活，复用

   

#### 参数化类型

泛型的实际作用就是把类型当参数一样传递
<数据类型>只能是引用类型(泛型的副作用)
举个例子：
`Plate<T>`中的 ”T” 称为**类型参数**
`Plate<Banana>`中的”Banana”称为**实际类型参数**
`“Plate<T>”` 整个成为**泛型类型**
`“Plate<Banana>”`整个称为**参数化的类型 ParameterizedType**



#### 泛型使用的地方

1. 接口上使用泛型

   ```java
   interface GenericsInterface<T> {
   
       // 此处只是接收一个泛型当参数，不是方法泛型
       void add(T t);
   }
   ```

2. 类上使用泛型

   ```java
   class GenericsClass<T> {
   
       private T t;
   
   }
   ```

3. 方法上使用泛型

   ```java
   /**
    * 泛型方法
    * @param <T> 在可见修饰符 与 返回值 之间定义方法接收的泛型
    */
   public <T> T getMiddle(T... ts) {
       return ts[ts.length/2];
   }
   ```

**Tips:**除了以上3个地方，其它都不可以使用泛型



#### 泛型类的多继承

```java
/**
 * 测试泛型类的继承 和 实现
 * 类上参数只能 extends 不能 super
 */
interface A {
}

interface B {
}

class C {
}

class D {
}

// 泛型中必须先继承类，再继承接口
/*class E<T extends A & B & C> {
}*/

// 泛型中只能单继承类，可以多继承接口
/*class F<T extends C & D & A & B> {
}*/

class G<T extends C & A & B>{
}
```



#### 泛型的实质

这里要用到ASM工具查看 class -> Bytecode 转换后剩下什么，bytecode是在编译成 .class文件后，编译成 .dex文件前插入

![](http://m.qpic.cn/psc?/V13IATxj2uFujC/ENmuKd2PHQoigBx2P9ktWYTVhu1miRQ7NtvwB9wTYDba6hZFkHj7mPKprvI3tCqJqpAwhk8ocE8H5rQRKLJdSg!!/mnull&bo=FAKMAQAAAAADB7k!&rf=photolist&t=5)

**Tips:** 在Android Studio 中搜索 bytecode plugins，找到对应插件装好，再点击类既可以 show bytecode outline。ASM工具在做插桩技术（如：无埋点统计、AOP编程）的时候作用很大，此处不介绍，就用来看bytecode码

就查看上面的类 `G` 的bytecode码

```java
class com/matrix/generics/G {  // 此处的G类已经没有任何泛型信息了

  // compiled from: GenericsTest1.java

  // access flags 0x0
  <init>()V
   L0
    LINENUMBER 27 L0
    ALOAD 0
    INVOKESPECIAL java/lang/Object.<init> ()V
    RETURN
   L1
    LOCALVARIABLE this Lcom/matrix/generics/G; L0 L1 0
    // signature Lcom/matrix/generics/G<TT;>;
    // declaration: com.matrix.generics.G<T>
    MAXSTACK = 1
    MAXLOCALS = 1

  // access flags 0x0
  // signature (TT;)V
  // declaration: void add(T)
  add(Lcom/matrix/generics/C;)V  // add方法参数T,被转换成了第一个继承的类
   L0
    LINENUMBER 31 L0
    RETURN
   L1
    LOCALVARIABLE this Lcom/matrix/generics/G; L0 L1 0
    // signature Lcom/matrix/generics/G<TT;>;
    // declaration: com.matrix.generics.G<T>
    LOCALVARIABLE t Lcom/matrix/generics/C; L0 L1 1
    // signature TT;
    // declaration: T
    MAXSTACK = 0
    MAXLOCALS = 2
}
```

可以清晰的看到，G类中类上的泛型信息已经被擦除，而 add方法中的参数 T ，被转换成第一个继承的类，可以得知泛型是在编译过程中检测的，而编译后不存在于JVM中

下面是一个很好的例子说明：

```java
void test(){
    List<C> list1 = new ArrayList<>();
    List<D> list2 = new ArrayList<>();

    // list1 = list2; 这样赋值是被禁止的，因为 list1 与 list2 不是同类型，也不是继承关系

    if (list1.getClass() == list2.getClass()){
        System.out.println("泛型被擦除了");
    }
}
```

> Task :generics:G.main()
> 泛型被擦除了



##### 泛型的桥方法

桥方法是泛型覆盖产生的

当一个方法覆盖了父类的方法，而编译后擦除了类型，等于是生成了2个方法

为了满足多态性，所以出现了桥法

```java
/**
 * 测试桥方法的生成，必须是覆盖父类的方法
 * @param <T>
 */
class BridgeFather<T extends Number> {

    public void add(T t) {

    }
}

class BridgeSon extends BridgeFather<Integer> {

    @Override
    public void add(Integer str) {

    }
}
```

```java
 // access flags 0x0
  <init>()V
   L0
    LINENUMBER 18 L0
    ALOAD 0
    INVOKESPECIAL com/matrix/generics/BridgeFather.<init> ()V
    RETURN
   L1
    LOCALVARIABLE this Lcom/matrix/generics/BridgeSon; L0 L1 0
    MAXSTACK = 1
    MAXLOCALS = 1

  // access flags 0x1
  public add(Ljava/lang/Integer;)V  // 子类中实例化了泛型参数为Integer
   L0
    LINENUMBER 23 L0
    RETURN
   L1
    LOCALVARIABLE this Lcom/matrix/generics/BridgeSon; L0 L1 0
    LOCALVARIABLE str Ljava/lang/Integer; L0 L1 1
    MAXSTACK = 0
    MAXLOCALS = 2

  // access flags 0x1041
  public synthetic bridge add(Ljava/lang/Number;)V  // 被擦除后父类泛型参数为Number
   L0
    LINENUMBER 18 L0
    ALOAD 0
    ALOAD 1
    CHECKCAST java/lang/Integer  // 检测泛型的强转
    INVOKEVIRTUAL com/matrix/generics/BridgeSon.add (Ljava/lang/Integer;)V
    RETURN
   L1
    LOCALVARIABLE this Lcom/matrix/generics/BridgeSon; L0 L1 0
    MAXSTACK = 2
    MAXLOCALS = 2
```



#### 泛型擦除

Java的泛型是JDK5新引入的特性，为了向下兼容，虚拟机其实是不支持泛型，所以Java实现的是一种伪泛型机制，也就是说Java在编译期擦除了所有的泛型信息，这样Java就不需要产生新的类型到字节码，所有的泛型类型最终都是一种原始类型，在Java运行时根本就不存在泛型信息。

Java编译器具体是如何擦除泛型的？

1. 检查泛型类型，获取目标类型
2. 擦除类型变量，并替换为限定类型
   如果泛型类型的类型变量没有限定(<T>),则用Object作为原始类型
   如果有限定(<T extends XClass>),则用XClass作为原始类型
   如果有多个限定(T extends XClass1&XClass2),则使用第一个边界XClass1作为原始类
3. 在必要时插入类型转换以保持类型安全
4. 生成桥方法以在扩展时保持多态性



#### 泛型被擦除之后为何还能看到，还能反射调用？

由于Java泛型的实现机制，使用了泛型的代码在运行期间相关的泛型参数的类型会被擦除，我们无法在运行期间获知泛型参数的具体类型

但是在编译java源代码成 class文件中还是保存了泛型相关的信息，，这些信息被保存在class字节码常量池中，使用了泛型的代码处会生成一个signature签名字段，通过签名signature字段指明这个常量池的地址。

```java
public class Test4 extends Test3<String> {

   static void test(){
       ParameterizedType genericSuperclass = (ParameterizedType) Test4.class.getGenericSuperclass(); // 获取泛型
        for (Type type: genericSuperclass.getActualTypeArguments()){
            System.out.println(type.getTypeName());  // 打印泛型的名字
        }
    }

    public static void main(String[] args) {
        test();
    }
}
```

> Task :generics:Test4.main()
> java.lang.String



#### 泛型类的继承关系

带泛型的类，实际泛型参数不同，它们之间**没有继承关系**，哪怕实际类型参数有继承关系

```java
class Fruit {
    void eat() {
    }
}

class Apple extends Fruit {
}

class Plate<T extends Fruit> {
    void add(T fruit){};
}
```

```java
Plate<Apple> plate1 = new Plate<>();
Plate<Fruit> plate2 = new Plate<>();

// plate2 = plate1; 没有继承关系
```



#### 通配符 ？

为了使得带泛型的类满足继承关系，出现了通配符

泛型中的问号符“？”名为“通配符”

+ 受上下限控制的通配符
+ 通配符匹配

通配符的适用范围： (不能用在类上)
+ 参数类型
+ 字段类型
+ 局部变量类型
+ 返回值类型（但返回一个具体类型的值更好）

```java
Plate< ? extends Apple> plate3 = new Plate<>();
plate3 = plate1; // 这样就能把上面的不能转型的类转型了
// 但是不能赋值了
// plate3.add(new Banana());
// plate3.add(new Fruit());
```

这里就涉及到了**PESC原则** （Producer extends Consumer super）

1. 如果你只需要从集合中获得类型T , 使用< ? extends T>通配符

2. 如果你只需要将类型T放到集合中, 使用< ? super T>通配符

3. 如果你既要获取又要放置元素，则不使用任何通配符。例如List<Apple>
   PECS即 ， 为了便于记忆。

为何要PECS原则？
	提升了API的灵活性

<?>  既不能存也不能取

Plate<?>  非限定通配符 是一个泛型类型 ? 未知
等价于 Plate<? extends Object>



下面3张图可以很好的说明通配符泛型类的继承关系

![](http://m.qpic.cn/psc?/V13IATxj2uFujC/ENmuKd2PHQoigBx2P9ktWaaQfgir6GX3YEd40Dl3.N1BncOI6piN8QqsyW8zOq6CuKmz4GM.ypWYsGYPTxRTEA!!/mnull&bo=OgRdAgAAAAADB0M!&rf=photolist&t=5)

![](http://m.qpic.cn/psc?/V13IATxj2uFujC/ENmuKd2PHQoigBx2P9ktWbQB0RHBtsJVLGRfRz1yCxonmFySWYrOWyNuzWn4BTuZ*EWVdbiMNZ2lqVdX25PUyA!!/mnull&bo=EwT.AQAAAAADB8o!&rf=photolist&t=5)

![](http://m.qpic.cn/psc?/V13IATxj2uFujC/ENmuKd2PHQoigBx2P9ktWUxKHUhsC08YpklR.jhwd89dafMqt4vJ3m0lik7D0XP*SwUpxG7k652OMDdm6Kn7Ww!!/mnull&bo=0gMuAgAAAAADB98!&rf=photolist&t=5)