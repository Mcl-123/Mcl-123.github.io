---
title: Kotlin 第二课 -- 基本数据类型
date: 2018-08-23 20:08:03
categories: "Kotlin"
tags:
     - Kotlin
     - Data Type
---

# 基本类型

数字、字符、布尔值、数组与字符串

<!-- more -->

## 数字

| Typt   | Bit Width   |
| :----: | :---------: |
| Double | 64          |
| Float  | 32          |
| Long   | 64          |
| Int    | 32          |
| Short  | 16          |
| Byte   |  8          |

注: Kotlin 处理数字与 Java 相近, 但不完全相同. 例如: Kotlin 没有对于数字的隐式拓宽转换, 而 Java 中有 int 隐式转换成 long

### 字面常亮

* 十进制: 123
    Long 类型用大写 L 标记: 123L
* 十六进制: 0x0F
* 二进制: 0b00001011

注:

1. 不支持八进制

2. kotlin 同样支持浮点数表示法:

     * 默认 double: 123.5
     * Float 用 F 或 f 标记: 123.5f

3. 支持数字字面值的下划线:

    ```kotlin
    val oneMillion = 1_000_000
    ```

### 表示方式

在 Java 平台数字是物理存储为 JVM 的原生类型，除非我们需要一个可空的引用（如 Int?）或泛型。 后者情况下会把数字装箱
需注意, 数字装箱保留了相等性, 不保留同一性(同 Java)

```kotlin
val a: Int = 10000
println(a == a) // 输出“true”
println(a === a) // 输出“true”
val boxedA: Int? = a
val anotherBoxedA: Int? = a
println(boxedA == anotherBoxedA) // 输出“true”
println(boxedA === anotherBoxedA) // ！！！输出“false”！！！
```

### 显式转换

由于不同的表示方式，较小类型并不是较大类型的子类型。 如果它们是的话，就会出现下述问题：

```kotlin
// 假想的代码，实际上并不能编译：
val a: Int? = 1 // 一个装箱的 Int (java.lang.Integer)
val b: Long? = a // 隐式转换产生一个装箱的 Long (java.lang.Long)
print(b == a) // 惊！这将输出“false”鉴于 Long 的 equals() 会检测另一个是否也为 Long
```

所以, 实际结果是这样:

```kotlin
val b: Byte = 1 // OK, 字面值是静态检测的
val i: Int = b // 错误
```

同时我们可以这样显式转换来拓宽数字:

```kotlin
val i: Int = b.toInt() // OK：显式拓宽
print(i)
```

每个数字类型支持如下转换:

```kotlin
toByte(): Byte
toShort(): Short
toInt(): Int
toLong(): Long
toFloat(): Float
toDouble(): Double
toChar(): Char
```

虽然没有隐式转换, 但是类型可以从上下文中推断出来:

```kotlin
val l = 1L + 3 // Long + Int => Long
```

### 智能转换

```kotlin
fun hasPrefix(x: Any) = when(x) {
    is String -> x.startsWith("prefix")
    else -> false
}
```

### 运算

对于位运算，没有特殊字符来表示，而只可用中缀方式调用命名函数，例如:

```kotlin
val x = (1 shl 2) and 0x000FF000
```

完整的位运算列表(只用于 Int 和 Long):

-. shl(bits) – 有符号左移 (Java 的 <<)
-. shr(bits) – 有符号右移 (Java 的 >>)
-. ushr(bits) – 无符号右移 (Java 的 >>>)
-. and(bits) – 位与
-. or(bits) – 位或
-. xor(bits) – 位异或
-. inv() – 位非

### 浮点数比较

-. 相等性比较: a == b 与 a != b
-. 比较操作符：a < b、 a > b、 a <= b、 a >= b
-. 区间实例以及区间检测：a..b、 x in a..b、 x !in a..b

## 字符

字符用 Char 类型表示。它们不能直接当作数字

```kotlin
fun check(c: Char) {
    if (c == 1) { // 错误：类型不兼容

    }
}
```

## 布尔

布尔用 Boolean 类型表示，它有两个值：true 与 false

## 数组

数组在 Kotlin 中使用 Array 类来表示，它定义了 get 与 set 函数（按照运算符重载约定这会转变为 []）以及 size 属性，以及一些其他有用的成员函数:

```kotlin
class Array<T> private constructor() {
    val size: Int
    operator fun get(index: Int): T
    operator fun set(index: Int, value: T): Unit

    operator fun iterator(): Iterator<T>
    // ……
}
```

创建数组的三种方式:

* arrayOf(1, 2, 3) 创建了 array [1, 2, 3]

* arrayOfNulls() 可以用于创建一个指定大小的、所有元素都为空的数组

* 创建一个 Array<String> 初始化为 ["0", "1", "4", "9", "16"]

```kotlin
val asc = Array(5, { i -> (i * i).toString() })
```

注:
Kotlin 不让我们把 Array<String> 赋值给 Array<Any>，以防止可能的运行时失败（但是你可以使用 Array<out Any>）

## 字符串

字符串用 String 类型表示。字符串是不可变的。 字符串的元素——字符可以使用索引运算符访问: s[i]。 可以用 for 循环迭代字符串:

```kotlin
val str = "abcd"
    for (c in str) {
        println(c)
    }
}
```

可以用 + 操作符连接字符串。这也适用于连接字符串与其他类型的值， 只要表达式中的第一个元素是字符串

```kotlin
val s = 2+ "abc" + 1
```

### 字符串字面值

#### 转移字符串

```kotlin
val s = "Hello, world!\n"
```

#### 原始字符串

```kotlin
val text = """
    for (c in "foo")
        print(c)
"""
```

注: 可以通过 trimMargin() 函数去除前导空格

### 字符串模板

模板表达式以美元符（$）开头，由一个简单的名字构成:

```kotlin
val i = 10
println("i = $i") // 输出“i = 10”
```

或者用花括号括起来的任意表达式:

```kotlin
val s = "abc"
println("$s.length is ${s.length}") // 输出“abc.length is 3”
```

如果你需要在原始字符串中表示字面值 $ 字符（它不支持反斜杠转义），你可以用下列语法：

```kotlin
println("${'$'}9.99") // 输出“$9.99"
```