---
title: Json
categories:
  - Java
tags:
  - Java
date: 2020-09-22 20:15:39
---

#### Json是什么

定义：JSON (JavaScript Object Notation)是一种轻量级的数据交换格式

作用：数据标记、存储、传输



#### Json的格式

JSON建构于两种结构：

+ “名称/值”对的集合（A collection of name/value pairs）。不同的语言中，它被理解为对象（object），纪录（record），结构（struct），字典（dictionary），哈希表（hash table），有键列表（keyed list），或者关联数组 （associative array）。

+ 值的有序列表（An ordered list of values）。在大部分语言中，它被理解为数组（array）。这些都是常见的数据结构。事实上大分现代计算机语言都以某种形式支持它们。这使得一种数据格式在同样基于这些结构的编程语言之间交换成为可能。



JSON具有以下这些形式：

1. 对象是一个无序的“‘名称/值’对”集合。一个对象以“{”（左括号）开始，“}”（右括号）结束。每个“名称”后跟一个“:”（冒号）；“‘名称/值’ 对”之间使用“,”（逗号）分隔。
![](http://m.qpic.cn/psc?/V13IATxj2uFujC/bqQfVz5yrrGYSXMvKr.cqRdwWXK6nTkMol0CQG.F.90p6s7OD*GjZq2biRGR05VDajsF0HW6qQpWVUlKdOHz1*YMO0V6XLLeAfris*2I.Sc!/b&bo=VgJxAAAAAAADBwc!&rf=viewer_4)

2. 数组是值（value）的有序集合。一个数组以“[”（左中括号）开始，“]”（右中括号）结束。值之间使用“,”（逗号）分隔。
![](http://m.qpic.cn/psc?/V13IATxj2uFujC/bqQfVz5yrrGYSXMvKr.cqZ0Ikyt5IxR1fnojsBS4uVQNRxSBf.2j7qaaMNuMWD3IW4e93kmfzaDH*UBjooulrgOjqRcNVEk.0f3dDadYa*k!/b&bo=VgJxAAAAAAADBwc!&rf=viewer_4)

3. 值（*value*）可以是双引号括起来的字符串（*string*）、数值(number)、 true 、 false 、 null 、对象（object）或者数组（array）。这些结构可以嵌套。
![](http://m.qpic.cn/psc?/V13IATxj2uFujC/bqQfVz5yrrGYSXMvKr.cqQgZdIyQtdUNS8SGEp8FXnUeRYHq6izmcKdtu*iqbJHz8FpGdnmyfLMXyn9gg9Z8Q3geZiO7SrapyvRc0K3dc*U!/b&bo=VgKdAQAAAAADB.o!&rf=viewer_4)

4. 字符串（*string*）是由双引号包围的任意数量Unicode字符的集合，使用反斜线转义。一个字符（character）即一个单独的字符串（character string）。
![](http://m.qpic.cn/psc?/V13IATxj2uFujC/bqQfVz5yrrGYSXMvKr.cqQgZdIyQtdUNS8SGEp8FXnUeRYHq6izmcKdtu*iqbJHz8FpGdnmyfLMXyn9gg9Z8Q3geZiO7SrapyvRc0K3dc*U!/b&bo=VgKdAQAAAAADB.o!&rf=viewer_4)

5. 数值（*number*）也与C或者Java的数值非常相似。除去未曾使用的八进制与十六进制格式。除去一些编码细节。
![](http://m.qpic.cn/psc?/V13IATxj2uFujC/TmEUgtj9EK6.7V8ajmQrEPpv1geVo6vtQv6S2EbWmSSXCQtcgu2xUL0Nu4.WPYbn2.HKcJ9*sEFZOfJmZW8LCOqguHlMr00oDWEXbpWwrVo!/b&bo=VgIKAQAAAAADF20!&rf=viewer_4)



#### Gson的用法

普通使用不再叙述，90%的公司都使用Gson解析

常用注解：

1. ```kotlin
   // 使用此注解的value的序列化和反序列化字段，常用于字段名变更、防止被混淆
   @SerializedName("index")
   ```

2. ```kotlin
   // 用来自己处理那些字段需要序列化、反序列化
   @Expose(serialize = true, deserialize = false) var name: String?
   // 必须使用此方法创建 Gson 才能生效
   GsonBuilder().excludeFieldsWithoutExposeAnnotation().create()
   ```

3. ```kotlin
       /**
        * GsonBuilder().setVersion(1.1).create() 必须setVersion()才能生效，并且 ！= -1.0
        * Since 当前Gson的 version必须小 >= 设置的version 才生效
        * Until 当前Gson的 version必须小 < 设置的version 才生效
        */
   class JsonD(
       var a: Int,
       @Since(1.1) var name: String?, // 当前version必须小 >=1.1 才能解析
       @Until(0.9) var matrix: String? // 当前version必须小 <0.9 才能解析
   )
   ```

4. ```kotlin
   // JsonAdapter 可以作用的Class、Field上，用来做一些特殊字段的转化
   @JsonAdapter(MySpeedAdapter::class) var speed: Int
   
   // 其中的一种用法，自己用来转换成需要的类型等
   class MySpeedAdapter : TypeAdapter<Int>() {
           override fun write(out: JsonWriter?, value: Int?) {
               out?.value("${value}km/s")
           }
   
           override fun read(`in`: JsonReader?): Int {
               var i = 0
               val nextString = `in`?.nextString()
               val pattern = "^\\d+"
               val p = Pattern.compile(pattern)
               val matcher = p.matcher(nextString!!)
               if (matcher.find())
                   i = matcher.group().toInt()
               return i
           }
       }
   ```