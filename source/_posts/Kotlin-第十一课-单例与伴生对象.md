---
title: Kotlin 第十一课 -- 单例与伴生对象
date: 2018-08-30 00:17:51
categories: "Kotlin"
tags:
     - Kotlin
     - 单例
     - 伴生对象
---

# 单例

Kotlin 中没有 static 关键字, 如果需要实现单例, 可以使用关键字 object 声明一个对象, 对象的构造器不能提供构造参数.
在第一次使用的时候会被初始化, 可用于提供常量或共享不可变对象.

<!-- more -->

## 对象声明

```kotlin
object DataProviderManager {
    fun registerDataProvider(provider: DataProvider) {
        // ……
    }
    val allDataProviders: Collection<DataProvider>
        get() = // ……
}
```

如需引用该对象，我们直接使用其名称即可：

```kotlin
DataProviderManager.registerDataProvider(……)
```

这些对象可以有超类型：

```kotlin
object DefaultListener : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) { …… }

    override fun mouseEntered(e: MouseEvent) { …… }
}
```

注: 对象声明不能在局部作用域（即直接嵌套在函数内部），但是它们可以嵌套到其他对象声明或非内部类中.

# 伴生对象

类内部的对象声明可以用 companion 关键字标记：

```kotlin
class MyClass {
    companion object Factory {
        fun create(): MyClass = MyClass()
    }
}
```

该伴生对象的成员可通过只使用类名作为限定符来调用：

```kotlin
val instance = MyClass.create()
```

可以省略伴生对象的名称，在这种情况下将使用名称 Companion：

```kotlin
class MyClass {
    companion object { }
}

val x = MyClass.Companion
```

注: 即使伴生对象的成员看起来像其他语言的静态成员，在运行时他们仍然是真实对象的实例成员，而且，例如还可以实现接口：

```kotlin
interface Factory<T> {
    fun create(): T
}

class MyClass {
    companion object : Factory<MyClass> {
        override fun create(): MyClass = MyClass()
    }
}
```

注: 在 JVM 平台，如果使用 @JvmStatic 注解，你可以将伴生对象的成员生成为真正的静态方法和字段.

```kotlin
class StaticTest{
    companion object{
        const  val s= ""
        @JvmStatic fun test() = print(s)
    }
}
```

数据块中的属性或者方法可以使用类.成员的方式调用（调用方式和static成员一致），但是他们在运行时依然是实体的实例成员:

```kotlin
class Singleton2 private constructor(){
    companion object{
        fun get(): Singleton2{
            return Inner2.instance
        }
    }

    private object  Inner2{
        val instance: Singleton2    = Singleton2()
    }
}
```

如上: 我们使用伴生对象可以更好的和单例结合，这样在调用单例的时候可以在我们想要获得单例实例的时候再出初始化单例，保证了线程安全.

