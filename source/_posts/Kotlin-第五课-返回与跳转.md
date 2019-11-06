---
title: Kotlin 第五课 -- 返回与跳转
date: 2018-08-26 13:21:19
categories: "Kotlin"
tags:
     - Kotlin
     - return
     - break
     - continue
---

# 返回与跳转: return break continue

```kotlin
val s = person.name ?: return
```

这些表达式的类型是 Nothing 类型
<!-- more -->

## 标签

### break 和 continue

在 Kotlin 中任何表达式都可以用标签（label）来标记。 标签的格式为标识符后跟 @ 符号，例如：abc@、fooBar@

```kotlin
loop@ for (i in 1..100) {
    // ……
}
```

可以用标签限制 break 或者continue:

```kotlin
loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (……) break@loop
    }
}
```

### return

```kotlin
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach {
        if (it == 3) return // 非局部直接返回到 foo() 的调用者
        print(it)
    }
    println("this point is unreachable")
}
```

 如果我们需要从 lambda 表达式中返回，我们必须给它加标签并用以限制 return:

 ```kotlin
 fun foo() {
    listOf(1, 2, 3, 4, 5).forEach lit@{
        if (it == 3) return@lit // 局部返回到该 lambda 表达式的调用者，即 forEach 循环
        print(it)
    }
    print(" done with explicit label")
}
```

现在，它只会从 lambda 表达式中返回。通常情况下使用隐式标签更方便。 该标签与接受该 lambda 的函数同名:

```kotlin
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach {
        if (it == 3) return@forEach // 局部返回到该 lambda 表达式的调用者，即 forEach 循环
        print(it)
    }
    print(" done with implicit label")
}
```

或者，我们用一个匿名函数替代 lambda 表达式。 匿名函数内部的 return 语句将从该匿名函数自身返回:

```kotlin
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach(fun(value: Int) {
        if (value == 3) return  // 局部返回到匿名函数的调用者，即 forEach 循环
        print(value)
    })
    print(" done with anonymous function")
}
```

或者:

```kotlin
fun foo() {
    run loop@{
        listOf(1, 2, 3, 4, 5).forEach {
            if (it == 3) return@loop // 从传入 run 的 lambda 表达式非局部返回
            print(it)
        }
    }
    print(" done with nested loop")
}
```

当要返一个回值的时候，解析器优先选用标签限制的 return，即:

```kotlin
return@a 1 \\ 意为“从标签 @a 返回 1”，而不是“返回一个标签标注的表达式 (@a 1)”
```