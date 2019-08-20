## 严格的数据类型
java中可以数据类型不同的数据赋值，java处于睁一只眼闭一只眼的情况，例如：  

```java
int a = 10;
// 这种情况，java是支持的，因为不存在精度丢失的情况
long b = a;
```
**Kotlin严格的数据类型**
```java
    val a:Int = 10
    val b:Long = a.toLong()
```
> 为何要这样定义呢？我的觉得还是和Kotlin中的`==`有关系，`==`对应的是调用了`equals`方法。
```java
class MyInt : IntIterator() {
    val statrt = 0
    val end = 100

    var value = statrt

    override fun hasNext(): Boolean {
        return value <= end
    }

    override fun nextInt(): Int {
        return value++
    }

    /**
     * 只要重写equals方法，就可以实现对象的==比较
     */
    override fun equals(other: Any?): Boolean {
        if (other is MyInt){
           return other.value == this.value
        }
        return false
    }
}

```
==这里是不是可以联想到Kotlin的`String`字符串，为何直接可以使用``==``做字符串相同比较呢？==

## 聪明的空类型判断和转换
#### 空类型预判
> 再java的编程世界中，最常见的就是`NullPointerException`,kottlin的设计师为我们想到了这个。再编码的过程中规避这个异常（当然不代表100%规避）

**kotlin中把变量分为了可null和不可null的类型**

```java
  /**
     * 定义可空字符串
     */
    val str: String? = null
    val noNStr: String = "我是不可空字符串"

    // 在后续的使用过程中，可null的对象，都会有一个判断

    // 字段名加?代表，如果str为null就返回null
    val length = str?.length

    // 字段名加!!代表强制str不可能为null，这样如果也会执行去长度方法。如果str为null就会抛异常
    val length2 = str!!.length

```
> 小插曲，我当时学习kotlin的时候。回顾下kotlin中？的运用之处

运用方式 | 解释
---|---
val str: String? = null |定义变量的时候，代表可null类型
str?.length| 属性方法调用时候，如果对象为null，不调用方法，直接返回null
?: |表达式，如果对象为null直接调用，后面的表达式
str?.length
> 这里我当时?:和str?.length没分清楚，其实这2个根本就是2个东西，不要把他混在一起哦。我就犯了这个错误，导致出错。例如我们经常的语句，对象为null返回可以写成：
```java
    str?:return
```
```
    /**
     * 我们经常写的adapter长度方法
     */
    fun getSize():Int{
        return datas?.size ?: 0
    }
```
#### 智能类型转换
> java中判断一个对象类型，使用关键词`instanceof`。Kotlin中使用`is`，你以为就是缩短了关键词这么简单？

```
class Data{
    val value=100
}

fun main() {
    /**
     * 我定义的data是Any类型（所有的类都是Any类型，类似java的Object。注意：Kotlin没有基本数据类型这一说了）
     */
    val data: Any = Data()
    // 此时我调用data.value属性是没有的，我也访问不了value属性
    if (data is Data){
        // 这里就自动类型转换了，我可以调用到value属性了
        data.value
    }
}
```
==不要小瞧这一个小的细节，在后续的高阶函数有很大的优势==
## val与var
> kotlin中定义变量需要使用`val`与`var`关键词

- val修饰，定义不可变变量。类似java中final修饰（并不是完全相同）
- var修饰，定义可变比那里。

**val定义变量是不可以修饰的，必须要在定义的时候初始化或者委托给另外方法。  
kotlin中val的变量都有1个隐藏的方法`get`方法，当获取本变量的值的时候，就会调用**

```
    /**
     * 例如这个方法,可以修改值,如果内容为空,就展示另外一个内容
     */
    val str: String = ""
        get() {
            // 对于field就（幕后字段），他就是修改当前变量字段值
            return if (field.isEmpty()) {
                "我不能为null"
            } else {
                field
            }
        }
```
**var定义变量是可修改的值，对应有一个set和get方法**

```java
 var str2: String = ""
        //当获取值的时候调用
        get() {
            return field
        }
        // 当设置新值的时候调用
        set(value) {
            // 如果设置的值时候new,不允许修改
            if (value != "new") {
                field = value
            }
        }
```
==补充下：==
> kotlin中的val和java中的final修饰的变量有何不同呢？其实java中final中修饰的变量，在编译器就已经知道了改变量的值是多少。所以在引用的时候就会把值自动改成改变量的值（编译器替换）。kotlin的val变量是不会替换的。如果想要替换使用const修饰

## is关键词
> java中我们经常使用instanceof判断类型，kotlin中不存在词关键词了。与之替换的是`is`关键词

```
fun main() {
    val d = 0
    if (d is Number){
        println("这家伙是个数")
    }
}
```
## as关键词和安全的类型转换
> java中有强制类型转换。kotlin中的关键词变成了`as`。

```
fun main() {
    val d = 0
    /**
     * 使用关键词as,当然这样转换肯定会抛出异常(int到double使用toDouble()方法)
     * class java.lang.Integer cannot be cast to class java.lang.Double
     */
    val double = d as Double
}
```
#### kotlin中独有安全类型转换

```
fun main() {
    val d = 0
    /**
     * 使用关键词as?,理论上会抛出以下异常
     * class java.lang.Integer cannot be cast to class java.lang.Double
     * 但是使用的as?,如果转换异常,会返回null(再配合?:语法,简直是天衣无缝),不会抛出异常
     */
    val defaultValue = 0.0
    val double = d as? Double ?: defaultValue
    println(double)
}
```

## 延迟加载
> 签名说过，kotlin定义变量。将变量分成了2中类型，可null和不可null的。但是有特殊情况，我们知道这个变量是不可为null的。但是初始化这个变量，还需要其他运行时候产生的条件。

```java
/**
     * 延迟初始化 lateinit var
     * 个人意见,能规避吊这种写法,就规避掉.打破了kotlin的非空安全机制
     */
    lateinit var value: String
    fun test(){
        value = ""
    }
```
# 总结
> 在学习kotlin中，发现kotlin相比较java来说，去其糟粕取其精华的操作。  
在数据类型这块，kotlin对比java做的更加的好。相比来说java的强类型语言，kotlin给我的感觉越来越像javaScript语言靠近。智能类型推倒，智能类型转换，都是相当不错的设计方案。  
koltin在经常出现的null指针设计理念还是很惊艳的，将这种异常大部分暴露在编译期。  
kotlin没有java的基本数据类型的概念了。kotlin都将java的基本类型，进行了包装（类似java的Integer等）。  
