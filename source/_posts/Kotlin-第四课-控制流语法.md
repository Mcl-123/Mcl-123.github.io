---
title: Kotlin 第四课 -- 控制流语法
date: 2018-08-25 22:10:42
categories: "Kotlin"
tags:
     - Kotlin
     - Control Flow
---

# 控制流: If, When, For, While

<!-- more -->

## If

在 Kotlin 中，if是一个表达式，即它会返回一个值. 因此, 就不需要三元运算符了.

```kotlin
// 传统用法
var max = a
if (a < b) max = b

// With else
var max: Int
if (a > b) {
    max = a
} else {
    max = b
}

// 作为表达式
val max = if (a > b) a else b
```

if的分支可以是代码块，最后的表达式作为该块的值：

```kotlin
val max = if (a > b) {
    print("Choose a")
    a
} else {
    print("Choose b")
    b
}
```

注: 如果你使用 if 作为表达式而不是语句（例如：返回它的值或者把它赋给变量），该表达式需要有 else 分支.

## When

- when 取代了 switch 操作符

```kotlin
when (x) {
    1 -> print("x == 1")
    2 -> print("x == 2")
    else -> { // 注意这个块
        print("x is neither 1 nor 2")
    }
}
```

注: 像 if 一样，每一个分支可以是一个代码块，它的值是块中最后的表达式的值;
如果 when 作为一个表达式使用，则必须有 else 分支， 除非编译器能够检测出所有的可能情况都已经覆盖了［例如，对于 枚举（enum）类条目与密封（sealed）类子类型］.

- 多个分支条件放一起

```kotlin
when (x) {
    0, 1 -> print("x == 0 or x == 1")
    else -> print("otherwise")
}
```

- 检测一个值在（in）或者不在（!in）一个区间或者集合中

```kotlin
when (x) {
    parseInt(s) -> print("s encodes x")
    else -> print("s does not encode x")
}
```

- 检测一个值是（is）或者不是（!is）一个特定类型的值

```kotlin
when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}
```

- 也可以用来取代 if-else if链

```kotlin
when {
    x.isOdd() -> print("x is odd")
    x.isEven() -> print("x is even")
    else -> print("x is funny")
}
```

## For

- for 循环可以对任何提供迭代器（iterator）的对象进行遍历

```kotlin
for (item in collection) print(item)
```

- 在数字区间上迭代

```kotlin
for (i in 1..3) {
    println(i)
}
for (i in 6 downTo 0 step 2) {
    println(i)
}
```

- 通过索引遍历一个数组或者一个 list

```kotlin
for (i in array.indices) {
    println(array[i])
}
```

- 用库函数 withIndex

```kotlin
for ((index, value) in array.withIndex()) {
    println("the element at $index is $value")
}
```

## While

正常使用

```kotlin
while (x > 0) {
    x--
}

do {
    val y = retrieveData()
} while (y != null) // y 在此处可见
```

## 循环中的 Break 和 Continue

正常使用
