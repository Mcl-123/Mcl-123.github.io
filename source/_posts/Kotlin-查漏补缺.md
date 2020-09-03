---
title: Kotlin 查漏补缺
date: 2020-09-02 09:37:27
categories: "Kotlin"
tags:
     - Kotlin
---

本文收集各处看到的 kotlin 小技巧及实用点.

<!-- more -->

## [一些 Kotlin 小技巧及解析](https://mp.weixin.qq.com/s/nxHcqpIqDSLgW1kfKmpREA)

### plus 操作符

利用 plus (+) 和 minus (-) 对 Map 集合做运算

```kotlin
val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)

// plus (+)
println(numbersMap + Pair("four", 4)) // {one=1, two=2, three=3, four=4}
println(numbersMap + Pair("one", 10)) // {one=10, two=2, three=3}
println(numbersMap + Pair("five", 5) + Pair("one", 11)) // {one=11, two=2, three=3, five=5}

// minus (-)
println(numbersMap - "one") // {two=2, three=3}
println(numbersMap - listOf("two", "four")) // {one=1, three=3}
```

## Map 集合的默认值

```kotlin
val map = mapOf(
        "java" to 1,
        "kotlin" to 2,
        "python" to 3
).withDefault { "?" }

println(map.getValue("java")) // 1
println(map.getValue("kotlin")) // 2
println(map.getValue("c++")) // ?
```

注意: withDefault 和 plus 操作符在一起用，有一个 bug:

```kotlin
val newMap = map + mapOf("python" to 3)
println(newMap.getValue("c++")) // 调用 getValue 时抛出异常，异常信息：Key c++ is missing in the map.
```

通过 plus(+) 操作符合并两个 map，返回一个新的 map, 但是忽略了默认值，

## 使用 require 或者 check 函数作为条件检查

```kotlin
// 传统的做法
val age = -1;
if (age <= 0) {
    throw IllegalArgumentException("age must  not be negative")
}

// 使用 require 去检查
require(age > 0) { "age must be negative" }

// 使用 checkNotNull 检查
val name: String? = null
checkNotNull(name){
    "name must not be null"
}
```

## 区分和使用 run, with, let, also, apply

| 函数 | 是否是扩展函数 | 函数参数(this、it) | 返回值(调用本身、最后一行) |
| :---: | :---: | :---: | :---: |
| with | 不是 | this | 最后一行 |
| T.run | 是 | this | 最后一行 |
| T.let | 是 | it | 最后一行 |
| T.also | 是 | it | 调用者本身 |
| T.apply | 是 | this | 调用者本身 |

### 使用 T.also 函数交换两个变量

```kotlin
int a = 1;
int b = 2;

// Java - 中间变量
int temp = a;
a = b;
b = temp;
System.out.println("a = "+a +" b = "+b); // a = 2 b = 1

// Java - 加减运算
a = a + b;
b = a - b;
a = a - b;
System.out.println("a = " + a + " b = " + b); // a = 2 b = 1

// Java - 位运算
a = a ^ b;
b = a ^ b;
a = a ^ b;
System.out.println("a = " + a + " b = " + b); // a = 2 b = 1

// Kotlin
a = b.also { b = a }
println("a = ${a} b = ${b}") // a = 2 b = 1
```

b.also { b = a } 会先将 a 的值 (1) 赋值给 b，此时 b 的值为 1，然后将 b 原始的值（2）赋值给 a，此时 a 的值为 2，实现交换两个变量的目的。

## 三种单例的实现方式

### 使用 Object 实现单例

```kotlin
object WorkSingleton
```

编译后的 java 文件:

```java
public final class WorkSingleton {
   public static final WorkSingleton INSTANCE;

   static {
      WorkSingleton var0 = new WorkSingleton();
      INSTANCE = var0;
   }
}
```

### 使用 by lazy 实现单例

```kotlin
class WorkSingleton private constructor() {
    companion object {
        // 方式一
        val INSTANCE1 by lazy(mode = LazyThreadSafetyMode.SYNCHRONIZED) { WorkSingleton() }

        // 方式二 默认就是 LazyThreadSafetyMode.SYNCHRONIZED，可以省略不写，如下所示
        val INSTANCE2 by lazy { WorkSingleton() }
    }
}
```

### 可接受参数的单例

但是有的时候，希望在单例实例化的时候传递参数，例如:

```kotlin
Singleton.getInstance(context).doSome()
```

第一种和第二种方式都不能满足, 用下面的方式:

```kotlin
class WorkSingleton private constructor(context: Context) {
    init {
        // Init using context argument
    }

    companion object : SingletonHolder<WorkSingleton, Context>(::WorkSingleton)
}

open class SingletonHolder<out T : Any, in A>(creator: (A) -> T) {
    private var creator: ((A) -> T)? = creator
    @Volatile
    private var instance: T? = null

    fun getInstance(arg: A): T {
        if (instance != null) {
            return instance!!
        }
        return synchronized(this) {
            if (instance != null) {
                instance!!
            }
            val created = creator!!(arg)
            instance = created
            creator = null
            instance!!
        }
    }
}
```

并且不限制传入参数的类型，凡是需要传递参数的单例模式，只需将单例类的伴随对象继承于 SingletonHolder，然后传入当前的单例类和参数类型即可，例如：

```kotlin
class FileSingleton private constructor(path: String) {
    companion object : SingletonHolder<FileSingleton, String>(::FileSingleton)
}
```
