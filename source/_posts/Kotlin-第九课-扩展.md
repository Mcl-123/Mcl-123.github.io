---
title: Kotlin 第九课 -- 扩展
date: 2018-08-29 20:45:56
categories: "Kotlin"
tags:
     - Kotlin
     - 伴生对象
---

# 扩展

Kotlin 可以对一个类的属性和方法进行扩展，且不需要继承或使用 Decorator 模式.
扩展是一种静态行为，对被扩展的类代码本身不会造成任何影响.

<!-- more -->

## 扩展函数

扩展函数可以在已有类中添加新的方法，不会对原类做修改，扩展函数定义形式:

```kotlin
fun receiverType.functionName(params){
    // ...
}
```

- receiverType：表示函数的接收者，也就是函数扩展的对象
- functionName：扩展函数的名称
- params：扩展函数的参数，可以为NULL

MutableList<Int> 添加一个swap 函数:

```kotlin
fun MutableList<Int>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // “this”对应该列表
    this[index1] = this[index2]
    this[index2] = tmp
}
```

注: 这个 this 关键字在扩展函数内部对应到接收者对象（传过来的在点符号前的对象）.

使用泛型:

```kotlin
fun <T> MutableList<T>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // “this”对应该列表
    this[index1] = this[index2]
    this[index2] = tmp
}
```

注: 为了在接收者类型表达式中使用泛型，我们要在函数名前声明泛型参数.

### 扩展是静态的

扩展不能真正的修改他们所扩展的类.通过定义一个扩展，你并没有在一个类中插入新成员， 仅仅是可以通过该类型的变量用点表达式去调用这个新函数.
在调用扩展函数时，具体被调用的的是哪一个函数，由调用函数的的对象表达式来决定的，而不是动态的类型决定的:

```kotlin
open class C

class D: C()

fun C.foo() = "c"

fun D.foo() = "d"

fun printFoo(c: C) {
    println(c.foo())
}

printFoo(D())   // "c"
```

这个例子会输出 "c"，因为调用的扩展函数只取决于参数 c 的声明类型，该类型是 C 类.

如果一个类定义有一个成员函数与一个扩展函数，而这两个函数又有相同的接收者类型、相同的名字并且都适用给定的参数，这种情况总是取成员函数。 例如：

```kotlin
class C {
    fun foo() { println("member") }
}

fun C.foo() { println("extension") }    

C().foo()     // “member”
```

当然，扩展函数重载同样名字但不同签名成员函数也完全可以：

```kotlin
class C {
    fun foo() { println("member") }
}

fun C.foo(i: Int) { println("extension") }

C().foo(1)      // "extension"
```

### 可空接收者

可以为可空的接收者类型定义扩展.

```kotlin
fun Any?.toString(): String {
    if (this == null) return "null"
    // 空检测之后，“this”会自动转换为非空类型，所以下面的 toString()
    // 解析为 Any 类的成员函数
    return toString()
}
```

## 扩展属性

与函数类似，Kotlin 支持扩展属性：

```kotlin
val <T> List<T>.lastIndex: Int
    get() = size - 1
```

扩展属性允许定义在类或者kotlin文件中，不允许定义在函数中.初始化属性因为属性没有后端字段（backing field），所以不允许被初始化，只能由显式提供的 getter/setter 定义。

```kotlin
val Foo.bar = 1 // 错误：扩展属性不能有初始化器
```

注: 扩展属性只能被声明为 val.

## 伴生对象的扩展

如果一个类定义有一个伴生对象 ，你也可以为伴生对象定义扩展函数与属性：

```kotlin
class MyClass {
    companion object { }  // 将被称为 "Companion"
}

fun MyClass.Companion.foo() {
    println("伴随对象的扩展函数")
}

val MyClass.Companion.no: Int
    get() = 10

fun main(args: Array<String>) {
    println("no:${MyClass.no}")
    MyClass.foo()
}
```

## 扩展的作用域

大多数时候我们在顶层定义扩展，即直接在包里：

```kotlin
package foo.bar

fun Baz.goo() { …… }
```

要使用所定义包之外的一个扩展，我们需要在调用方导入它：

```kotlin
package com.example.usage

import foo.bar.goo // 导入所有名为“goo”的扩展
// 或者
import foo.bar.*   // 从“foo.bar”导入一切

fun usage(baz: Baz) {
    baz.goo()
}
```

## 扩展声明为成员

在一个类内部你可以为另一个类声明扩展。
在这个扩展中，有个多个隐含的接受者 —— 其中的对象成员可以无需通过限定符访问，扩展方法定义所在类的实例称为分发接受者，而扩展方法的目标类型的实例称为扩展接受者。

```kotlin
cclass D {
    fun bar() { println("D bar") }
}

class C {
    fun baz() { println("C baz") }

    fun D.foo() {
        bar()   // 调用 D.bar
        baz()   // 调用 C.baz
    }

    fun caller(d: D) {
        d.foo()   // 调用扩展函数
    }
}

fun main(args: Array<String>) {
    val c: C = C()
    val d: D = D()
    c.caller(d)     // "D bar"  "C baz"

}
```

上例中,在 C 类内创建了 D 类的扩展。此时，C 被成为分发接受者，而 D 为扩展接受者。从上例中，可以清楚的看到，在扩展函数中，可以调用派发接收者的成员函数。

假如在调用某一个函数，而该函数在分发接受者和扩展接受者均存在，则以扩展接收者优先，要引用分发接收者的成员你可以使用限定的 this 语法:

```kotlin
class D {
    fun bar() { println("D bar") }
}

class C {
    fun bar() { println("C bar") }  // 与 D 类 的 bar 同名

    fun D.foo() {
        bar()         // 调用 D.bar()，扩展接收者优先
        this@C.bar()  // 调用 C.bar()
    }

    fun caller(d: D) {
        d.foo()   // 调用扩展函数
    }
}

fun main(args: Array<String>) {
    val c: C = C()
    val d: D = D()
    c.caller(d)        //  "D bar"  "C bar"
}
```

以成员的形式定义的扩展函数, 可以声明为 open , 而且可以在子类中覆盖. 也就是说, 在这类扩展函数的派 发过程中, 针对分发接受者是虚拟的(virtual), 但针对扩展接受者仍然是静态的

```kotlin
open class D {
}

class D1 : D() {
}

open class C {
    open fun D.foo() {
        println("D.foo in C")
    }

    open fun D1.foo() {
        println("D1.foo in C")
    }

    fun caller(d: D) {
        d.foo()   // 调用扩展函数
    }
}

class C1 : C() {
    override fun D.foo() {
        println("D.foo in C1")
    }

    override fun D1.foo() {
        println("D1.foo in C1")
    }
}


fun main(args: Array<String>) {
    C().caller(D())   // 输出 "D.foo in C"
    C1().caller(D())  // 输出 "D.foo in C1" —— 分发接收者虚拟解析
    C().caller(D1())  // 输出 "D.foo in C" —— 扩展接收者静态解析
}
```

## 扩展的意义

在Java中，我们将类命名为“*Utils”：FileUtils、StringUtils 等，著名的 java.util.Collections 也属于同一种命名方式。 关于这些 Utils-类的不愉快的部分是代码写成这样:

```java
// Java
Collections.swap(list, Collections.binarySearch(list,
    Collections.max(otherList)),
    Collections.max(list));
```

这些类名总是碍手碍脚的，我们可以通过静态导入达到这样效果：

```java
// Java
swap(list, binarySearch(list, max(otherList)), max(list));
```

这会变得好一点，但是我们并没有从 IDE 强大的自动补全功能中得到帮助。如果能这样就更好了：

```java
// Java
list.swap(list.binarySearch(otherList.max()), list.max());
```

但是我们不希望在 List 类内实现这些所有可能的方法，对吧？这时候扩展将会帮助我们.
