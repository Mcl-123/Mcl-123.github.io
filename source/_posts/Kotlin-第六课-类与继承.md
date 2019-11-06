---
title: Kotlin 第六课 -- 类与继承
date: 2018-08-26 16:11:02
categories: "Kotlin"
tags:
     - Kotlin
     - class
     - extend
---

# 类

Kotlin 中使用关键字 class 声明类

```kotlin
class Invoice { ... }
```

<!-- more -->

## 构造函数

在 Kotlin 中的一个类可以有一个主构造函数以及一个或多个次构造函数.

### 主构造函数

主构造函数是类头的一部分：它跟在类名（与可选的类型参数）后。

```kotlin
class Person constructor(firstName: String) { ... }
```

如果主构造函数没有任何注解或者可见性修饰符，可以省略这个 constructor 关键字:

```kotlin
class Person(firstName: String) { ... }
```

主构造函数不能包含任何的代码。初始化的代码可以放到以 init 关键字作为前缀的初始化块（initializer blocks）中:

在实例初始化期间，初始化块按照它们出现在类体中的顺序执行，与属性初始化器交织在一起：

```kotlin
class InitOrderDemo(name: String) {
    val firstProperty = "First property: $name".also(::println)

    init {
        println("First initializer block that prints ${name}")
    }

    val secondProperty = "Second property: ${name.length}".also(::println)

    init {
        println("Second initializer block that prints ${name.length}")
    }
}
```

事实上，声明属性以及从主构造函数初始化属性，Kotlin 有简洁的语法：

```kotlin
class Person(val firstName: String, val lastName: String, var age: Int) { …… } 
```

注: 与普通属性一样，主构造函数中声明的属性可以是可变的（var）或只读的（val）.

如果构造函数有注解或可见性修饰符，这个 constructor 关键字是必需的，并且这些修饰符在它前面：

```kotlin
class Customer public @Inject constructor(name: String) { …… }
```

### 次构造函数

前缀有 constructor的次构造函数:

```kotlin
class Person {
    constructor(parent: Person) {
        parent.children.add(this)
    }
}
```

如果类有一个主构造函数，每个次构造函数需要委托给主构造函数， 可以直接委托或者通过别的次构造函数间接委托。委托到同一个类的另一个构造函数用 this 关键字即可：

```kotlin
class Person(val name: String) {
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}
```

初始化块中的代码实际上会成为主构造函数的一部分。委托给主构造函数会作为次构造函数的第一条语句，因此所有初始化块中的代码都会在次构造函数体之前执行。即使该类没有主构造函数，这种委托仍会隐式发生，并且仍会执行初始化块：

```kotlin
class Constructors {
    init {
        println("Init block")   // 先执行
    }

    constructor(i: Int) {
        println("Constructor")  // 后执行
    }
}
```

如果一个非抽象类没有声明任何（主或次）构造函数，它会有一个生成的不带参数的主构造函数。构造函数的可见性是 public.
如果你不希望你的类有一个公有构造函数，你需要声明一个带有非默认可见性的空的主构造函数：

```kotlin
class DontCreateMe private constructor () { ... }
```

注: 在 JVM 上，如果主构造函数的所有的参数都有默认值，编译器会生成 一个额外的无参构造函数，它将使用默认值.

```kotlin
class Customer(val customerName: String = "")
```

## 创建类的实例

要创建一个类的实例，我们就像普通函数一样调用构造函数：

```kotlin
val invoice = Invoice()
val customer = Customer("Joe Smith")
```

注: Kotlin 并没有 new 关键字.

## 类成员

- 构造函数与初始化块
- 函数
- 属性
- 嵌套类与内部类
- 对象声明

## 抽象类

类以及其中的某些成员可以声明为 abstract,  抽象成员在本类中可以不用实现.
我们可以用一个抽象成员覆盖一个非抽象的开放成员:

```kotlin
open class Base {
    open fun f() {}
}

abstract class Derived : Base() {
    override abstract fun f()
}
```

## 嵌套类

我们可以把类嵌套在其他类中:

```kotlin
class Outer {                  // 外部类
    private val bar: Int = 1
    class Nested {             // 嵌套类
        fun foo() = 2
    }
}

fun main(args: Array<String>) {
    val demo = Outer.Nested().foo() // 调用格式：外部类.嵌套类.嵌套类方法/属性
    println(demo)    // == 2
}
```

## 内部类

内部类使用 inner 关键字来表示。
内部类会带有一个对外部类的对象的引用，所以内部类可以访问外部类成员属性和成员函数.

```kotlin
class Outer {
    private val bar: Int = 1
    var v = "成员属性"
    /**嵌套内部类**/
    inner class Inner {
        fun foo() = bar  // 访问外部类成员
        fun innerTest() {
            var o = this@Outer //获取外部类的成员变量
            println("内部类可以引用外部类的成员，例如：" + o.v)
        }
    }
}

fun main(args: Array<String>) {
    val demo = Outer().Inner().foo()
    println(demo) //   1
    val demo2 = Outer().Inner().innerTest()   
    println(demo2)   // 内部类可以引用外部类的成员，例如：成员属性
}
```

注: 为了消除歧义，要访问来自外部作用域的 this，我们使用this@label，其中 @label 是一个 代指 this 来源的标签。

### 嵌套类和内部类在使用时的区别

#### 创建对象的区别

```kotlin
var demo = Outter.Nested()// 嵌套类，Outter后边没有括号
var demo = Outter().Inner();// 内部类，Outter后边有括号
```

也就是说，要想构造内部类的对象，必须先构造外部类的对象，而嵌套类则不需要.

#### 引用外部类的成员变量的方式不同

先来看嵌套类：

```kotlin
class Outer {                  // 外部类
    private val bar: Int = 1
    class Nested {             // 嵌套类
        var ot: Outer = Outer()
        println(ot.bar) // 嵌套类可以引用外部类私有变量，但要先创建外部类的实例，不能直接引用
        fun foo() = 2
    }
}
```

再来看一下内部类:

```kotlin
class Outer {
    private val bar: Int = 1
    var v = "成员属性"
    /**嵌套内部类**/
    inner class Inner {
        fun foo() = bar  // 访问外部类成员
        fun innerTest() {
            var o = this@Outer //获取外部类的成员变量
            println("内部类可以引用外部类的成员，例如：" + o.v)
        }
    }
}
```

可以看来内部类可以直接通过 this@ 外部类名 的形式引用外部类的成员变量，不需要创建外部类对象；


## 匿名内部类

使用对象表达式来创建匿名内部类：

```kotlin
class Test {
    var v = "成员属性"

    fun setInterFace(test: TestInterFace) {
        test.test()
    }
}

/**
 * 定义接口
 */
interface TestInterFace {
    fun test()
}

fun main(args: Array<String>) {
    var test = Test()

    /**
     * 采用对象表达式来创建接口对象，即匿名内部类的实例。
     */
    test.setInterFace(object : TestInterFace {
        override fun test() {
            println("对象表达式创建匿名内部类的实例")
        }
    })
}
```

注: 特别注意这里的 object : TestInterFace，这个 object 是 Kotlin 的关键字，要实现匿名内部类，就必须使用 object 关键字，不能随意替换其它单词.

## 类的修饰符

类的修饰符包括 classModifier 和_accessModifier_

### classModifier: 类属性修饰符，标示类本身特性

- abstract    // 抽象类  
- final       // 类不可继承，默认属性
- enum        // 枚举类
- open        // 类可继承，类默认是final的
- annotation  // 注解类

### accessModifier: 访问权限修饰符

- private    // 仅在同一个文件中可见
- protected  // 同一个文件中或子类可见
- public     // 所有调用的地方都可见
- internal   // 同一个模块中可见

# 继承

在 Kotlin 中所有类都有一个共同的父类 Any，这对于没有父类声明的类是默认父类：

```kotlin
class Example // 从 Any 隐式继承
```

注: Any 并不是 java.lang.Object；尤其是，它除了 equals()、hashCode() 与 toString() 外没有任何成员

## 继承父类

要继承一个父类，我们把类型放到类头的冒号之后：

```kotlin
open class Base(p: Int)
class Derived(p: Int) : Base(p)
```

注:

- 默认情况下，在 Kotlin 中所有的类都是 final. 类上的 open 标注与 Java 中 final 相反，它允许其他类从这个类继承.
- 如果子类有主构造函数, 它的父类则必须在父类的主构造函数里立即初始化.
- 如果子类没有主构造函数, 则子类的每个次构造函数必须使用 super 关键字初始化父类构造函数, 或者调用另一个构造函数. 不同的次构造函数可以调用父类的不同的构造函数.

```kotlin
class MyView : View {
    constructor(ctx: Context) : super(ctx)

    constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs)
}
```

### 重写

在父类中，使用fun声明函数时，此函数默认为final修饰，不能被子类重写。
如果允许子类重写该函数，那么就要手动添加 open 修饰它, 子类重写方法使用 override 关键词：

```kotlin
open class Base {
    open fun v() { ... }
    fun nv() { ... }
}
class Derived() : Base() {
    override fun v() { ... }
}
```

标记为 override 的成员本身是开放的，也就是说，它可以在子类中覆盖。如果你想禁止再次覆盖，使用 final 关键字：

```kotlin
open class AnotherDerived() : Base() {
    final override fun v() { ... }
}
```

### 属性重写

属性重写同方法重写一样, 需使用 override 关键字，属性必须具有兼容类型，每一个声明的属性都可以通过初始化程序或者getter方法被重写：

```kotlin
open class Foo {
    open val x: Int get() { …… }
}
class Bar1 : Foo() {
    override val x: Int = ……
}
```

注: 可以用一个 var 属性覆盖一个 val 属性，但反之则不行. 
因为一个 val 属性本质上声明了一个 getter 方法，而将其覆盖为 var 只是在子类中额外声明一个 setter 方法; 反过来, 父类 var 声明了 getter 和 setter 方法, 子类 val 不能有 setter 方法, 即无法重写父类的 setter 方法, 相当于缩小了父类相应属性的使用范围, 是不允许的.

可以在主构造函数中使用 override 关键字作为属性声明的一部分:

```kotlin
interface Foo {
    val count: Int  // 接口的成员变量默认是 open 的
}

class Bar1(override val count: Int) : Foo
```

注: 子类继承父类时，默认不能有跟父类同名的变量，除非父类中该变量为 private，或者父类中该变量为 open 并且子类用 override 关键字重写

### 子类初始化顺序

在构建派生类的新势力的过程中, 第一步是完成其父类的初始化, 第二步才是完成子类的初始化

```kotlin
open class Base(val name: String) {

    init { println("Initializing Base") }   // 2.

    open val size: Int = 
    name.length.also { println("Initializing size in Base: $it") }  // 3.
}

class Derived(
    name: String,
    val lastName: String
) : Base(name.capitalize().also { println("Argument for Base: $it") }) {    // 1.

    init { println("Initializing Derived") }    // 4

    override val size: Int =
    (super.size + lastName.length).also { println("Initializing size in Derived: $it") }    // 5.
}
```

## 子类调用父类函数及属性

子类可以使用 super 关键字调用其父类的函数与属性:

```kotlin
open class Foo {
    open fun f() { println("Foo.f()") }
    open val x: Int get() = 1
}

class Bar : Foo() {
    override fun f() { 
        super.f()
        println("Bar.f()") 
    }

    override val x: Int get() = super.x + 1
}
```

在一个内部类中访问外部类的超类，可以通过由外部类名限定的 super 关键字来实现：super@Outer：

```kotlin
class Bar : Foo() {
    override fun f() { /* …… */ }
    override val x: Int get() = 0

    inner class Baz {
        fun g() {
            super@Bar.f() // 调用 Foo 实现的 f()
            println(super@Bar.x) // 使用 Foo 实现的 x 的 getter
        }
    }
}
```

### 重写规则

在 Kotlin 中，实现继承由下述规则规定：如果一个类从它的直接父类继承相同成员的多个实现， 它必须覆盖这个成员并提供其自己的实现（也许用继承来的其中之一）.
为了表示采用从哪个父类继承的实现，我们使用由尖括号中父类名限定的 super，如 super<Base>：

```kotlin
open class A {
    open fun f() { print("A") }
    fun a() { print("a") }
}

interface B {
    fun f() { print("B") } // 接口成员默认就是“open”的
    fun b() { print("b") }
}

class C() : A(), B {
    // 编译器要求覆盖 f()：
    override fun f() {
        super<A>.f() // 调用 A.f()
        super<B>.f() // 调用 B.f()
    }
}
```

同时继承 A 与 B 没问题，并且 a() 与 b() 也没问题因为 C 只继承了每个函数的一个实现。 但是 f() 由 C 继承了两个实现，所以我们必须在 C 中覆盖 f() 并且提供我们自己的实现来消除歧义.
