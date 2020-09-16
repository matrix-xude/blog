---
title: Java注解
categories:
  - Java
tags:
  - Java
date: 2020-06-14 18:20:08
---

#### 注解 Annotation 是什么  

java中的annotation是 JDK 5.0 后引入的一种注释机制，本身并没有操作代码。注解是元数据的一种形式，提供有关程序但不属于程序本身的数据。简单的说：annotation就是一个标志。



#### 怎么定义Annotation

```java
@interface TargetField{}
```

`@interface`就可以定义一个annotation，在此annotation上面可以添加元注解

annotation 默认实现Annotation接口

```java
package java.lang.annotation; 

public interface Annotation { 

boolean equals(Object obj); 

int hashCode(); 

String toString(); 

Class<? extends Annotation> annotationType(); 

}
```



#### 元注解

在JDK 1.5中提供了4个标准的用来对注解类型进行注解的注解类，我们称之为 meta-annotation（元注解）

+ @Target    注解可以打在什么位置上，10个位置
+ @Retention   注解保留的时间，3个阶段
+ @Documented  注解是否保留到生成的 Doc 文档上
+ @Inherited 注解是否被继承



#### @Target 的位置

在 Annotation 上打上 @Target 可以确保 Annotation 不被使用在错误的位置上。如不打上@Target，则该Annotation可以使用在所有位置

```java
public enum ElementType {
    /** Class, interface (including annotation type), or enum declaration */
    TYPE,

    /** Field declaration (includes enum constants) */
    FIELD,

    /** Method declaration */
    METHOD,

    /** Formal parameter declaration */
    PARAMETER,

    /** Constructor declaration */
    CONSTRUCTOR,

    /** Local variable declaration */
    LOCAL_VARIABLE,

    /** Annotation type declaration */
    ANNOTATION_TYPE,

    /** Package declaration */
    PACKAGE,  // 非常特殊的一个位置，package注解只能使用在 package-info.java文件夹下

    /**
     * Type parameter declaration
     *
     * @since 1.8
     */
    TYPE_PARAMETER,  // 泛型注解

    /**
     * Use of a type
     *
     * @since 1.8
     */
    TYPE_USE  // 所有位置的注解，除了package
}
```



#### @Retention保留级别

3种级别的保留时间

+ RetentionPolicy.SOURCE    标记的注解仅保留在源级别中，并被编译器忽略
+ RetentionPolicy.CLASS   标记的注解在编译时由编译器保留，但 Java 虚拟机(JVM)会忽略
+ RetentionPolicy.RUNTIME 标记的注解由 JVM 保留，因此运行时环境可以使用它

| **级别** | **技术**   | **说明**                                                     |
| -------- | ---------- | ----------------------------------------- |
| 源码     | APT        | 在编译期能够获取注解与注解声明的类包括类中所有成员信一般用于生成额外的辅助类。 |
| 字节码   | 字节码增强 | 在编译出Class后，通过修改Class数据以实现修改代码逻辑目的。对于是否需要修改的区分或者修改为不同逻辑的判断可以使用注解。 |
| 运行时   | 反射       | 在程序运行期间，通过反射技术动态获取注解与其元素，从而完成不同的逻辑判定。 |



##### APT 

RetentionPolicy.SOURCE 注解主要用法之一

APT全称为："Anotation Processor Tools"，意为注解处理器。顾名思义，其用于处理注解。编写好的Java源文件，需要经过 javac 的编译，翻译为虚拟机能够加载解析的字节码Class文件。注解处理器是 javac 自带的一个工具，用来在编译时期扫描处理注解信息。你可以为某些注解注册自己的注解处理器。 注册的注解处理器由 javac调起，并将注解信息传递给注解处理器进行处理。

>注解处理器是对注解应用最为广泛的场景。在Glide、EventBus3、Butterknifer、Tinker、>ARouter等等常用框架中都有注解处理器的身影。但是你可能会发现，这些框架中对注解的定义并>不是 SOURCE 级别，更多的是 CLASS 级别，别忘了：**CLASS包含了SOURCE，RUNTIME 包含 SOURCE 、CLASS。**

APT 入门写法步骤

1. 新建一个java module ，写一个类继承`AbstractProcessor`

   ```java
   public class APTProcessor extends AbstractProcessor {
   
       @Override
       public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
           Messager messager = processingEnv.getMessager();
       	messager.printMessage(Diagnostic.Kind.NOTE,"~~~~~~~~~~~~~~~~~~~~~~~~~~~");
           return false;
       }
   
       @Override
       public SourceVersion getSupportedSourceVersion() {  // 支持的版本，必须让支持到最新版本
           return SourceVersion.latestSupported();
       }
   }
   ```

2. 写上你需要处理的Annotation全类名，否则找到的set为null

   ```java
   @SupportedAnnotationTypes("com.matrix.java02_annotation.AppAnnotation") //通过注解方式添加
   public class APTProcessor extends AbstractProcessor {
   
       @Override
       public Set<String> getSupportedAnnotationTypes() { // 通过方法添加，2个不共存
           HashSet<String> supportTypes = new LinkedHashSet<>();
           supportTypes.add(APTAnnotation.class.getName());
           return supportTypes;
       }
    }
   ```

3. 注册你的Processor，就像Android xml中注册activity一样，需要注册Processor才能找到它

   切换到Project目录下，在main目录下新建 

   **resource -> META-INF.services -> javax.annotation.processing.Processor**

   一个字符都不能错，这个固定的路径写法，gradle 3.0之后不支持自动生成，所以最好手动写，改文件下放上你的需要给javac处理的processor，例如 `com.matrix.annotation.APTProcessor`

4. Android工程的gradle下引入你的注解处理器

   ```groovy
   annotationProcessor project(':annotation')
   ```

   gradle 3.0 后必须这么引入，否则会报异常



##### IDE语法检查

RetentionPolicy.SOURCE 注解主要用法之二（Android 程序中的语法检查，不是java中的类）

```java
public static void main(String[] args) {
    setBehavior(222);
}

private static final int BEHAVIOR_SHOW = 1;
private static final int BEHAVIOR_HINT = 2;
private static final int BEHAVIOR_RESUME = 3;

/**
 * 我想要的是传入上面3种 behavior 模式之一，但是int可以随便传
 * @param behavior
 */
private static void setBehavior(int behavior){

}
```

现在使用Android中的 IntDef 限制

```java
@IntDef({Test3.BEHAVIOR_SHOW, Test3.BEHAVIOR_HINT, Test3.BEHAVIOR_RESUME})
@Target({ElementType.PARAMETER, ElementType.FIELD})
@Retention(RetentionPolicy.SOURCE)
@interface Test3Behavior {
}


public static void main(String[] args) {
    // setBehavior(1);  不能再随便传入值了
    setBehavior(BEHAVIOR_SHOW);
}

@Test3Behavior // 也可以不加此注解，加了就必须写到 @IntDef 上，强制使用
private static final int BEHAVIOR_SHOW = 1;
@Test3Behavior
private static final int BEHAVIOR_HINT = 2;
@Test3Behavior
private static final int BEHAVIOR_RESUME = 3;

/**
 * 我想要的是传入上面3种 behavior 模式之一，但是int可以随便传
 *
 * @param behavior
 */
private static void setBehavior(@Test3Behavior int behavior) {

}
```



##### 字节码插桩

这里暂时不写，RetentionPolicy.CLASS的应用



##### 反射

RetentionPolicy.RUNTIME 时的应用

比较简单，这里说下反射+泛型

Gson中序列化泛型类时必须用到TypeToken才能拿到泛型的类型

```java
Type type = new TypeToken<Level1<Level3>>() {
}.getType();
Level1<Level3> level1 = gson.fromJson(json, type);
System.out.println(type);
```

自己写一个简单的Type拿泛型试试

```java
class MyType<T> {

    private Type type;

    MyType() {
        // 获取泛型
        Type genericSuperclass = getClass().getGenericSuperclass();
        // 如果有泛型，可以强转成 ParameterizedType
        ParameterizedType parameterizedType = (ParameterizedType) genericSuperclass;

        // 泛型可以定义很多个  Type<T,K...>
        Type[] actualTypeArguments = parameterizedType.getActualTypeArguments();
        type = actualTypeArguments[0];
    }

    public Type getType() {
        return type;
    }
}
```

```java
Type myType = new MyType<Level1<Level2<Level3>>>().getType(); //报错
System.out.println(myType);
Type myType2 = new MyType<Level1<Level2<Level3>>>() {}.getType(); // 正常
System.out.println(myType2);
```

**少一个{}就会失败？**为什么？

因为只有子类才能拿到泛型类型T,本身的类泛型擦除都会被变为object



写2个空类ASM看下就明白了

```java
public class GenericFather<T> {
    T t;
}

// signature <T:Ljava/lang/Object;>Ljava/lang/Object;  泛型被擦除了
// declaration: com/matrix/annotation/GenericFather<T>
public class com/matrix/annotation/GenericFather {

  // compiled from: GenericFather.java

  // access flags 0x0
  // signature TT;
  // declaration: T
  Ljava/lang/Object; t

  // access flags 0x1
  public <init>()V
   L0
    LINENUMBER 3 L0
    ALOAD 0
    INVOKESPECIAL java/lang/Object.<init> ()V
    RETURN
   L1
    LOCALVARIABLE this Lcom/matrix/annotation/GenericFather; L0 L1 0
    // signature Lcom/matrix/annotation/GenericFather<TT;>;
    // declaration: com.matrix.annotation.GenericFather<T>
    MAXSTACK = 1
    MAXLOCALS = 1
}
```

```java
public class GenericSon extends GenericFather<String> {

}

// signature Lcom/matrix/annotation/GenericFather<Ljava/lang/String;>;  泛型类型被拿到了
// declaration: com/matrix/annotation/GenericSon extends com.matrix.annotation.GenericFather<java.lang.String>
public class com/matrix/annotation/GenericSon extends com/matrix/annotation/GenericFather  {

  // compiled from: GenericSon.java

  // access flags 0x1
  public <init>()V
   L0
    LINENUMBER 3 L0
    ALOAD 0
    INVOKESPECIAL com/matrix/annotation/GenericFather.<init> ()V
    RETURN
   L1
    LOCALVARIABLE this Lcom/matrix/annotation/GenericSon; L0 L1 0
    MAXSTACK = 1
    MAXLOCALS = 1
}
```

