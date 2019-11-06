---
title: Kotlin 第一课
date: 2018-08-22 21:19:53
categories: "Kotlin"
tags:
     - Kotlin
---

Kotlin是Jetbrains公司起初用于内部开发的而发起的一个开源项目，这个Jetbrains公司也许你没听过，但是IntelliJ IDEA你一定听过，没错你所用的Android Studio的老妈就是这个公司的产品。在 Google I/O 2017 大会上, 谷歌宣布 Kotlin 成为 Android 开发的官方编程语言.
Kotlin 不仅仅可以用于 Android 应用开发, 代码简洁且表现力强, 与 Java 完全兼容. 同时, Kotlin 还可以用来开发前端 React 应用.
作为一名 Android 开发, 也需要跟上自己的脚步, 学习 Kotlin 是很有必要的. 学习着, 随时准备着用在项目里.

<!-- more -->

## Kotlin 概述

### Kotlin 用于 Android

* **兼容性**: Kotlin 与 JDK 6 完全兼容, 保障了 Kotlin 应用程序可以在较旧的 Android 设备上运行而无任何问题。Kotlin 工具在 Android Studio 中会完全支持，并且兼容 Android 构建系统.

* **性能**: 由于非常相似的字节码结构，Kotlin 应用程序的运行速度与 Java 类似。 随着 Kotlin 对内联函数的支持，使用 lambda 表达式的代码通常比用 Java 写的代码运行得更快。

* **互操作性**: Kotlin 可与 Java 进行 100％ 的互操作，允许在 Kotlin 应用程序中使用所有现有的 Android 库 。这包括注解处理，所以数据绑定与 Dagger 也是一样。

* **占用**: Kotlin 具有非常紧凑的运行时库，可以通过使用 ProGuard 进一步减少。 在实际应用程序中，Kotlin 运行时只增加几百个方法以及 .apk 文件不到 100K 大小。

* **编译时长**: Kotlin 支持高效的增量编译，所以对于清理构建会有额外的开销，增量构建通常与 Java 一样快或者更快。

* **学习曲线**: 对于 Java 开发人员，Kotlin 入门很容易。包含在 Kotlin 插件中的自动 Java 到 Kotlin 的转换器有助于迈出第一步。

### Kotlin 用于 JavaScript

### Kotlin 用于 服务器端

### Kotlin/Native

## Kotlin 特性

* **简洁**: 大大地减少代码量.

例如:
Java:
```
public class User {
  private String id;
  private String name;
  public void setId(long id) {
    this. id = id;
  }
  public void setName(String name) {
    this. name=name;
  }
  public String getId() {
    return id;
  }
  public String getName() {
    return name;
  }
}
```
使用一行代码创建一个包含 getters、 setters、 equals()、 hashCode()、 toString() 以及 copy() 的 POJO:
```
data class Customer(val id: String, val name: String)
```

* **安全**: 避免空指针异常等整个类的错误.

例如:
```
val name: String? = null    // 可控类型
println(name.length())      // 编译错误
```

* **互操作性**: 充分利用 JVM, Android 和 浏览器的现有库.

例如:
```
Flowable
    .fromCallable {
        Thread.sleep(1000) //  模仿高开销的计算
        "Done"
    }
    .subscribeOn(Schedulers.io())
    .observeOn(Schedulers.single())
    .subscribe(::println, Throwable::printStackTrace)
```

* **工具友好**: 可用任何 Java IDE 或者使用命令行构建.
