---
title: Kotlin 第七课 -- 属性
date: 2018-08-27 21:07:22
categories: "Kotlin"
tags:
     - Kotlin
     - property
---

# 属性

## var 和 val

用关键字 var 声明的属性为可变属性, 用关键字 val 声明的属性为只读属性.

<!-- more -->

```kotlin
class Address {
    var name: String = ……
}
```

要使用一个属性，只要用名称引用它即可:

```kotlin
fun copyAddress(address: Address): Address {
    val result = Address() // Kotlin 中没有“new”关键字
    result.name = address.name // 将调用访问器
    result.street = address.street
    // ……
    return result
}
```

## Getter 和 Setter

声明一个属性的完整语法是:

```kotlin
var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]
```

其初始器（initializer）、getter 和 setter 都是可选的.
属性类型如果可以从初始器 （或者从其 getter 返回值，如下文所示）中推断出来，也可以省略:

```kotlin
var allByDefault: Int? // 错误：需要显式初始化器，隐含默认 getter 和 setter
var initialized = 1 // 类型 Int、默认 getter 和 setter
```

我们可以编写自定义的访问器:

```kotlin
val isEmpty: Boolean
    get() = this.size == 0
```

```kotlin
var stringRepresentation: String
    get() = this.toString()
    set(value) {
        setDataFromString(value) // 解析字符串并赋值给其他属性
    }
```

注: setter 参数的默认名称是 value，也可以选择一个不同的名称

自 Kotlin 1.1 起，如果可以从 getter 推断出属性类型，则可以省略它：

```kotlin
val isEmpty get() = this.size == 0  // 具有类型 Boolean
```

如果需要改变一个访问器的可见性或者对其注解，但是不需要改变默认的实现， 你可以定义访问器而不定义其实现:

```kotlin
var setterVisibility: String = "abc"
    private set // 此 setter 是私有的并且有默认实现

var setterWithAnnotation: Any? = null
    @Inject set // 用 Inject 注解此 setter
```

## 只读属性和可变属性的区别

1. 只读属性的用 val开始代替var
2. 只读属性不允许 setter

## Field 关键字

在 Kotlin 中，任何时候当你写出“一个变量后边加等于号”这种形式的时候，比如我们定义 var no: Int 变量，当你写出 no = ... 这种形式的时候，这个等于号都会被编译器翻译成调用 setter 方法；而同样，在任何位置引用变量时，只要出现 no 变量的地方都会被编译器翻译成 getter 方法。那么问题就来了，当你在 setter 方法内部写出 no = ... 时，相当于在 setter 方法中调用 setter 方法，形成递归，进而形成死循环，例如：

```kotlin
var no: Int = 100
    get() = field                // 后端变量
    set(value) {
        if (value < 10) {       // 如果传入的值小于 10 返回该值
            field = value
        } else {
            field = -1         // 如果传入的值大于等于 10 返回 -1
        }
    }
```

这段代码按以上这种写法是正确的，因为使用了 field 关键字，但是如果不用 field 关键字会怎么样呢？例如：

```kotlin
var no: Int = 100
    get() = no
    set(value) {
        if (value < 10) {       // 如果传入的值小于 10 返回该值
            no = value
        } else {
            no = -1         // 如果传入的值大于等于 10 返回 -1
        }
    }
```

注意这里我们使用的 Java 的思维写了 getter 和 setter 方法，那么这时，如果将这段代码翻译成 Java 代码会是怎么样呢？如下：

```java
int no = 100;
public int getNo() {
    return getNo();// Kotlin中的get() = no语句中出来了变量no，直接被编译器理解成“调用getter方法”
}

public void setNo(int value) {
    if (value < 10) {
        setNo(value);// Kotlin中出现“no =”这样的字样，直接被编译器理解成“这里要调用setter方法”
    } else {
        setNo(-1);// 在setter方法中调用setter方法，这是不正确的
    }
}
```

翻译成 Java 代码之后就很直观了，在 getter 方法和 setter 方法中都形成了递归调用，显然是不正确的，最终程序会出现内存溢出而异常终止。

注: field 标识符只能用在属性的访问器内.

## 延迟初始化属性与变量

一般地，属性声明为非空类型必须在构造函数中初始化。 然而，这经常不方便。例如：属性可以通过依赖注入来初始化， 或者在单元测试的 setup 方法中初始化。 这种情况下，你不能在构造函数内提供一个非空初始器。 但你仍然想在类体中引用该属性时避免空检查。
为处理这种情况，可以用 lateinit 修饰符标记该属性：

```kotlin
public class MyTest {
    lateinit var subject: TestSubject

    @SetUp fun setup() {
        subject = TestSubject()
    }

    @Test fun test() {
        subject.method()  // 直接解引用
    }
}
```

注:

- 该修饰符只能用于在类体中的属性（不是在主构造函数中声明的 var 属性，并且仅当该属性没有自定义 getter 或 setter 时），而自 Kotlin 1.2 起，也用于顶层属性与局部变量。该属性或变量必须为非空类型，并且不能是原生类型.
- 在初始化前访问一个 lateinit 属性会抛出一个特定异常，该异常明确标识该属性被访问及它没有初始化的事实。

### 检测一个 lateinit var 是否已初始化

```kotlin
if (foo::bar.isInitialized) {
    println(foo.bar)
}
```

## 常量

Kotlin 中的:
    val num = 6
等价于 Java 中的:
    public final int num = 6

但是, Kotlin 中用 val 修饰还不是常量, 只能是不能修改的变量.而常量的定义, 需要在关键字 val 前面加上关键字 const:

```kotlin
const val NUM = 6
```

注: const 只能修饰 val, 不能修饰 var.

### 声明常量的三种方式

- 在顶层声明
- 在 object 修饰的类中声明, 在 Kotlin 中称为对象声明, 相当于 Java 中一种形式的单例类
- 在伴生对象中声明

```kotlin
// 1. 顶层声明
const val NUM_A : String = "顶层声明"

// 2. 在object修饰的类中
object TestConst{
    const val NUM_B = "object修饰的类中"
}

// 3. 伴生对象中
class TestClass{
    companion object {
        const val NUM_C = "伴生对象中声明"
    }
}

fun main(args: Array<String>) {
    println("NUM_A => $NUM_A")
    println("NUM_B => ${TestConst.NUM_B}")
    println("NUM_C => ${TestClass.NUM_C}")
}
```