---
title: 《第一行代码》(第三版) 之 Kotlin 的总结
date: 2020-05-20 22:38:32
categories: "Kotlin"
tags:
     - Kotlin
---

1. 循环： ..     是左闭右闭
         downTo 是左闭右闭
         until  是左闭右开
2. 类：任何一个非抽象类默认都是不可以被继承的。
3. 构造函数：子类的主构造函数调用父类中的哪个构造函数，在继承的时候通过括号来指定。
            子类在主构造函数中声明成 val 或 var 的参数将自动成为该类的字段，就会与父类同名的字段冲突。因此，同名的参数前面我们不用加任何关键字，让它的作用域仅限定在主构造函数当中。
            子类没有主构造函数只有次构造函数时，继承父类就不需要加括号了。
4. 接口：Kotlin 中允许对接口的定义的函数进行默认实现（JDK 1.8 后也支持）。

5. 函数的可见性修饰符：
    - Java: public、private、protected、default
    - Kotlin: publi、private、protected、internal
    - Java 中默认是 default
    - Kotlin 中默认是 public

    修饰符 | Java |  Kotlin  
    -|-|-
    public | 所有类可见 | 所有类可见 |
    private | 当前类可见 | 当前类可见 |
    protected | 当前类、子类、同一包路径下的类可见 | 当前类、子类可见 |
    default | 同一包路径下的类可见 | 无 |
    internal | 无 | 同一模块中的类可见 |

<!-- more -->

6. 数据类：类名前面声明了 data 关键字，Kotlin 会根据主构造函数中的参数帮自动生成 equals()、hashCode()、toString() 等方法。
7. 单例类：object SingleTon {}
8. 集合：创建：listOf("a", "b") ; mutableListOf("a", "b")
              setOf(); mutableSetOf()
              mapOf("a" to 1, "b" to 2)   (ps: key to value)
        api：maxBy()；map()；filter()；any()；all()；；；
9. Lambda：表达式的语法结构：{ 参数名1：参数类型，参数名2：参数类型 -> 函数体 }。
           当 Lambda 参数是函数的最后一个参数时，可以将 Lambda 表达式移到函数括号的外面；
           当 Lambda 参数是函数的唯一一个参数时，可以将函数的括号省略；
           Lambda 表达式中的参数列表在大多数情况下不必声明参数类型；
           当 Lambda 参数列表中只有一个参数时，也不必声明参数名，可以用 it 关键字代替。
10. 匿名内部类：创建匿名内部类时不能使用 new，而是改用了 object 关键字。
11. Java 函数式 API：如果我们在 Kotlin 代码中调用了一个Java方法，并且该方法接收一个 Java 单抽象方法接口参数，就可以使用函数式 API。
                    如果一个 Java 方法的参数列表中不存在一个以上 Java 单抽象方法接口参数，还可以将参数名进行省略。
                    PS：限定于从 Kotlin 中调用 Java 方法，并且单抽象方法接口也必须用 Java 语言定义。
12. 可空类型：在类名后面加一个问号
13. 判空辅助工具：
    - ?. 当对象不为空时正常调用相应的方法，为空时则什么都不做。
    - ?: 操作符左右各接收一个表达式，如果左边表达式的结果不为空就返回左边表达式的结果，否则就返回右边表达式的结果。
    - !! 非空断言工具，告诉 Kotlin，我非常确信这里的对象不为空，所以你不用来帮我做空指针检查了
14. let 函数：主要是配合 ?. 操作符进行辅助判空处理。let 函数可以处理全局变量的判空问题，而 if 判断语句不行。
15. 字符串内嵌表达式："$obj"；"${obj.name}"
16. 函数的参数默认值：可以通过键值对的方式来传参。
17. 标准函数：let、with、run、apply
    - with：接收两个参数，第一个是任意类型的对象，第二个是 Lambda 表达式。
            with 函数会在 Lambda 表达式中提供第一个参数对象的上下文，并将 Lambda 表达式中的最后一行代码最为返回值返回。
            用处：连续调用同一个对象的多个方法时让代码变得精简。
    - run：一定要调用某个对象的 run 函数
           只接收一个参数，Lambda 表达式
           会在 Lambda 表达式中提供对象的上下文，并将 Lambda 表达式中的最后一行代码最为返回值返回。
    - apply：一定要调用某个对象的 apply 函数
             只接收一个参数，Lambda 表达式
             会在 Lambda 表达式中提供对象的上下文，并将 Lambda 表达式中的最后一行代码最为返回值返回。
18. 定义静态方法：
    - companion object {} (PS: 在类中创建有且唯一的伴生类对象，调用的静态方法实际上就是调用的伴生对象的方法)
    - 顶层方法，指的是没有定义在任何类中的方法。
    - 给单例类或者 companion object 中的方法加上 @JvmStatic 注解
19. 延迟初始化：lateinit
               ::adapter.isInitiallized 用来判断 adapter 变量是否已经初始化
20. 密封类：sealed class
        用处：在 when 语句中传入一个密封类变量作为条件，可以确保所有的情况，同时不用写 else
21. 扩展函数：语法结构: fun ClassName.methodName(param1: Int, param2: Int): Int { return 0 }
22. 运算符重载：operator
23. 高阶函数：定义：如果一个函数接收另一个函数作为参数，或者返回值的类型是另一个函数，那么该函数就称为高阶函数。
            函数类型的基本规则：ClassName.(String, Int) -> Unit (PS: 左边的部分用来声明该函数接收什么参数，右边的部分用来声明该函数的返回值是什么类型；加上 ClassName. 表明该函数类型参数是该 ClassName 对象的函数)。
            用处：允许让函数类型的参数来决定函数的执行逻辑。
            函数的引用方式：::plus 表示将 plus() 函数作为参数传递给方法里。
            调用高阶函数的方式：
                - 定义一个与其函数类型参数相匹配的函数
                - Lambda 表达式 (Lambda 表达式中的最后一行代码会自动作为返回值)
                - 匿名内部类
                - 成员引用
            实现原理：Lambda 表达式在底层被转换成了匿名内部类的实现方式
24. 内联函数：在定义高阶函数时加上 inline 关键字的声明
            解决的问题：解决每调用 Lambda 表达式，都会创建一个新的匿名内部类实例造成的额外的内存及性能开销
            工作原理：Kotlin 编译器会将 Lambda 表达式中的函数体替换到调用它的地方
            局限性：内联的函数类型参数在编译时会被进行代码替换，没有真正的参数属性，所以内联的函数类型参数只允许传递给另外一个内联函数
25. noinline 与内联函数的区别：
                - 非内联的函数类型参数可以自由的传递给其他任何函数；内联的函数类型参数只允许传递给另外一个内联函数
                - 非内联函数所引用的 Lambda 表达式中只能进行局部返回；内联函数所引用的 Lambda 表达式中是可以使用 return 关键字来进行函数的返回及局部返回
26. crossinline：
        内联函数可能会遇到的问题：在 Lambda 或者匿名内部类中调用了传入的函数类型参数，因为内联函数所引用的 Lambda 表达式允许使用 return 关键字进行函数返回，而匿名类中调用的函数类型参数，此时是不可能进行外层的函数返回，最多只能对匿名类中的函数调用进行返回，因此会报错
        解决办法：在函数类型参数前面加上 crossinline
        原理：crossinline 关键字就像一个契约，用于保证内联函数的 Lambda 表达式中一定不会使用 return 关键字，但仍可以使用局部返回
27. Any 是 Kotlin 中所有类的共同基类，相当于 Java 中的 Object
28. vararg 对应的就是 Java 中的可变参数列表
29. 在 Kotlin 中使用 A to B 这样的语法结构会创建一个 Pair 对象
30. Kotlin 中有 Smart Cast 功能，适用于 when 及 if
31. 泛型：
        两种定义方式:
                - 定义泛型类
                - 定义泛型方法
                （PS：语法结构都是 <T> ）
        泛型的上界：<T : Number>
                  默认情况下，所有的泛型都是可以指定成可空类型的，这是因为在不手动指定上界的时候，泛型的上界默认是 Any?；
                  想要泛型的类型不为空，只需要将泛型的上界手动指定成 Any 就行了。
32. 类委托：by
        委托模式的意义：我们只是让大部分的方法实现调用辅助对象中的方法，少部分的方法实现由自己来重写，甚至加入一些自己独有的方法。
        委托模式的弊端：遇到接口中待实现的方法太多
        解决方案：通过类委托的功能来解决
        核心：将一个类的具体实现委托给另一个类去完成
33. 委托属性
        核心：将一个属性的具体实现委托给另一个类去完成
        by lazy { }
                by 是 Kotlin 中的关键字，lazy 在这里只是一个高阶函数，在 lazy 函数中会创建并返回一个 Delegate 对象，然后当我们调用 p 属性时，其实调用的是 Delecate 对象的 getValue() 方法，然后 getValue() 方法中又会调用 lazy 函数传入的 Lambda 表达式，并且调用 p 属性后得到的值就是 Lambda 表达式中最后一行代码
34. infix 函数：允许我们将函数调用时的小数点、括号等计算机相关的语法去掉
        例如：A.to(B) 等价成 A to B
        两点限制：1. infix 函数是不能定义成顶层函数的，它必须是某个类的成员函数，可以使用扩展函数的方式将它定义到某个类当中；
                 2. infix 函数必须接收且只能接收一个参数，至于参数类型没有限制；
35. 对泛型进行实化：
        Java 中不支持 T.class
        泛型功能的实现：Java 泛型功能是通过类型擦除机制来实现的，也就是说泛型对于类型的约束只在编译时期存在，运行的时候仍然会按照 JDK1.5 之前的机制来运行，JVM 是识别不出来我们在代码中指定的泛型类型。
        泛型实化的条件：1.该函数必须是内联函数
                       2.在声明泛型的地方必须加上 reified 关键字
        泛型实化：inline fun <reified T> getGenericType() {}
        泛型实化的用处：允许我们在泛型函数当中获得泛型的实际类型，这也使得类似于 T::class.java ; a is T 这样的语法成为了可能。
36. 泛型的协变：
        先说一个约定：一个泛型类或者泛型接口中的方法，它的参数列表是接收数据的地方，称为 in 位置；它的返回值是输出数据的地方，称为 out 位置；
        协变的定义：假如定义了一个 MyClass<T> 的泛型类，其中 A 是 B 的子类型，同时 MyClass<A> 又是 MyClass<B> 的子类型，那么我们就称 MyClass 在 T 这个泛型上是协变的。
        实现的方式：如果一个泛型类在其泛型类型的数据上是只读的话，那么它是没有类型转换的安全隐患的；所以需要让 MyClass<T> 类中的所有方法都不能接收 T 类型的参数，即 T 只能出现在 out 位置上，而不能出现在 in 位置上。
37. 泛型的逆变：
        逆变的定义：假如定义了一个 MyClass<T> 的泛型类，其中 A 是 B 的子类型，同时 MyClass<B> 又是 MyClass<A> 的子类型，那么我们就称 MyClass 在 T 这个泛型上是逆变的。
        实现的方式：在泛型 T 的声明前面加上了一个 in 关键字。这就意味着 T 只能出现在 in 位置上，而不能出现在 out 位置上。
38. 协程：简单理解为一种轻量级的线程，用来提升并发编程的运行效率。(一套线程框架)
        有点: 方便, 方便在它能在同一个代码块里进行多次的线程切换.
        开启方式：
                - Global.launch {} : 每次创建的都是一个顶层协程，这种协程当应用程序运行结束时跟着一起结束。
                - runBlocking 函数 : 同样会创建一个协程的作用域，但是它可以保证在协程作用域内的所有代码和子代码没有全部执行完成前一直阻塞当前线程 (PS: 只在测试环境使用，生产环境会有性能问题)
                - launch 函数 : 用来创建多个协程，但是首先它必须要在协程的作用域下才能调用；其次它会在当前协程的作用域下创建子协程 （PS: 子协程的特点是 如果外层作用域的协程结束了，该作用域下的所有子协程也会一同结束）
        delay 函数: 非阻塞式的挂起函数, 只会挂起当前协程，并不会影响其他协程 (PS: delay 函数只能在协程作用域或其它挂起函数中调用)
        协程的取消: GlobalScope.launch 或 launch 函数都会返回一个 Job 对象, 只要调用 Job 对象的 cancel() 方法就可以取消协程了
39. 挂起函数: suspend
        可以将任意的函数声明成挂起函数，而挂起函数之间是可以互相调用的；suspend 无法给函数提供协程作用域的。
40. coroutineScope 函数: 用来解决 suspend 挂起函数无法提供协程作用域的问题。
        特点: - 会继承外部的协程作用域并创建一个子作用域, 这样就可以给任意挂起函数提供协程作用域了；
             - 保证其作用域内的所有代码和子协程在全部执行完之前, 会一直阻塞当前协程。
             - 可以在协程作用域或者挂起函数内调用
41. async 函数: 会创建一个新的子协程并返回一个 Deferred 对象, 如果我们想要获取 async 函数代码块的执行结果，只需要调用 Deferred 对象的 await() 方法即可。(PS: async 只能在协程作用域里调用)。
        调用 async 函数之后, 会立刻执行代码块中的代码; 当调用 await() 方法时，如果代码块里的代码还没执行完, 那么 await() 方法会将当前协程阻塞, 直到可以获得 async 函数的执行结果。
42. withContext 函数: 一个挂起函数, 可以理解成 async 函数的一种简化版写法，相当于 async {}.await(), 不同的是 withContext 函数强制要求我们指定一个线程参数。
43. 线程参数:
        - Dispatchers.Default : 默认低并发的线程策略, 当你要执行的代码属于计算密集型任务时
        - Dispatchers.IO : 较高并发线程策略, 执行的代码大多数时间是阻塞和等待中
        - Dispatchers.Main : Android 主线程中执行代码
        PS: 协程作用域构建器中，除了 coroutineScope 函数之外, 其他所有的函数都是可以指定这样一个线程参数的。
44. suspendCoroutiner 函数:
        作用: 简化传统回调机制
        必须在协程作用域或挂起函数中调用
        使用: 接收一个 Lambda 表达式参数, 将当前协程立即挂起, 然后在一个普通线程中执行 Lambda 表达式中的代码。Lambda 表达式的参数列表中会传入一个 Continuation 参数, 调用它的 resume()方法或 resumeWithException() 方法可以让协程恢复。
45. DSL: 领域特定语言
        infix 函数构建出的特有语法结构就属于 DSL
46. ViewModel
        作用：帮 Activity 分担一部分工作，专门用于存放与界面相关的数据；将与界面相关的变量存放在 ViewModel 中，即使旋转手机屏幕，界面上显示的数据也不会丢失。
        使用：ViewModelProviders.of(xxxActivity).get(xxxViewModel::class.java)
        向 ViewModel 传递参数：借助 ViewModelProvider.Factory
47. Lifecycles
        解决问题：编写 Android 应用程序时，需要感知 Activity 生命周期的情况。
        使用：lifecycle.addObserver(xxObserver);
              MyObserver 中，在方法上使用 @OnLifecycleEvent 注解，传入生命周期事件（ON_CREATE、ON_START、ON_RESUME、ON_PAUSE、ON_STOP、ON_DESTROY、ON_ANY）;
              MyObserver 虽然能够感知到 Activity 的生命周期发生变化，却没法主动获取当前的生命周期状态，因此可以在 MyObserver 的构造函数中传入 lifecycle 对象，通过 lifecycle.currentState 返回生命周期状态，包括 INITIALIZED、DESTROYED、CREATED、STARTED、RESUMED
48. LiveData
        遇到的问题：我们一直使用的都是在 Activity 中手动获取到 ViewModel 中的数据这种交互方式，但是 ViewModel 却无法将数据的变化主动通知给 Activity。
        解决方案：LiveData 可以包含任何类型的数据，并在数据发生变化时通知给观察者;
                 MutableLiveData 是一种可变的 LiveData, 有三种读写数据的方法：getValue()、setValue()、postValue();
                 任何 LiveData 都可以调用它的 observe() 方法来观察数据的变化；observer() 接收两个参数，第一个参数是 LifecycleOwner 对象，第二个参数是 Observer 接口，当 LiveData 中包含的数据发生变化时，就会回调到这里。
        推荐：永远只暴露不可变的 LiveData 给外部。
        map() 方法：
                作用：将实际包含数据的 LiveData 和 仅用于观察数据的 LiveData 进行转换。
                使用：调用 Transformations 的 map() 方法，接收两个参数，第一个参数是原始的包含数据的 LiveData，第二个参数是一个转换函数，在转换函数里编写具体的转换逻辑，转换函数中返回的是用于观察的 LiveData 数据所包含的数据类型。
        switchMap() 方法：
                遇到的问题：很可能 ViewModel 中的某个 LiveData 对象是调用另外的方法获取的，这样每次调用方法返回的都是一个新的 LiveData 对象，如果直接观察的话，观察的一直都是老的 LiveData，所以无法观察到数据的变化。
                作用：将调用另外的方法获取的 LiveData 转换成另外一个可观察的 LiveData。
                使用：将触发数据变化的值封装成一个 LiveData，然后调用 Transformations 的 switchMap() 方法，接收两个参数，第一个参数是触发数据变化的值的LiveData，第二个参数是转换函数，转换函数中返回的是一个 LiveData 对象。
                工作原理：将转换函数中返回的 LiveData 对象转换成另一个可观察的 LiveData 对象。
        注意: - LiveData 内部不会判断即将设置的数据和原有数据是否相同；
              -当 Activity 不可见时，如果 LiveData 中的数据发生了变化，是不会通知给观察者的。只有当 Activity 可见时，才会将数据通知给观察者（通过 Lifecycles 组件实现该细节）；
              - 如果 Activity 不可见时，LiveData 发生了多次数据变化，当 Activity 恢复可见时，只有最新的那份数据会通知观察者，前面的数据直接丢弃掉。
49. ViewModel + LiveData
        架构思路：
                1. 将 Activity 上触发数据变化的值在 ViewModel 中封装成一个包含数据(变化的值)的 private LiveData，再在 ViewModel 中定义一个方法，更新这个包含数据 LiveData 的 value；
                2. 再在 ViewModel 中定义一个 public 的仅用于观察的 LiveData (没有 set()), 这个仅用于观察的 LiveData 可以通过定义的 get() 获取，亦或者通过 Transformations 的 map() 或者 switchMap() 将 包含数据的 private LiveData 进行处理等逻辑后获取的新的用于观察的 LiveData
                3. 在 Activity 中观察该用于观察的 LiveData 的数据变化，更新 UI
50. Room
        ORM：对象关系映射；简单说：使用的编程语言是面向对象语言，数据库是关系型数据库，这两者之间建立一种映射关系，就是 ORM。
        ORM 框架好处：用面向对象的思维来和数据库进行交互，绝大多数情况下不用和 SQL 语句打交道了；也不用担心操作数据库的逻辑会让项目的整体代码变得混乱。
        使用：
            - Entity：封装数据的实体类
                (良好的数据库编程建议：给每个实体类都添加一个 id 字段，并将这个字段设为主键)
            - Dao：对数据库的各项操作进行封装
                (Dao 要做的事情就是覆盖所有的业务需求，使得业务永远只需要与 Dao 层进行交互，而不必和底层数据库打交道)
            - Database：定义数据库中的关键信息
                (定义3个部分内容, 数据库的版本号、包含哪些实体类、提供 Dao 层的访问实例)

以下是自己发现的问题:

51. Kotlin 中 使用反射类时, 使用 Class.forName(xxx) 时, 需要传进完整的包名, 如下:

```kotlin
val forName = Class.forName(Printer2::class.java.name)
val forName = Class.forName("Printer2") // 这样不正确, 找不到 Printer2 类
```
52. data class 对象可能会绕过 Kotlin 的空指针检查，也可能未执行父类构造方法就把对象构造出来， 如下：

```kotlin
定义一个父类: 
public class People {
    public People(){
        System.out.println("people cons");
    }
}
写一个 Bean, 接收服务器返回的数据：
data class Person(var name: String, var age: Int) : People()
然后构造该对象: 
val gson = Gson()
val person = gson.fromJson<Person>("{\"age\":\"12\"}", Person::class.java)
println(person.name )   // 打印 null， 没有走父类的构造函数
```

原因在 Gson 的源码中: 

Gson 创建对象，一般都会走到以下三个方法里：
- newDefaultConstructor         // 尝试获取无参的构造函数，如果找到，则通过 newInstance 反射的方式构建对象
- newDefaultImplementationConstructor   // 这个方法里面都是一些集合类相关对象的逻辑，直接跳过
- newUnsafeAllocator    // 在没有找到无参的构造方法后，通过sun.misc.Unsafe构造了一个对象

newu 如何不安全的创建一个对象：

```java
public static UnsafeAllocator create() {
// try JVM
// public class Unsafe {
//   public Object allocateInstance(Class<?> type);
// }
try {
  Class<?> unsafeClass = Class.forName("sun.misc.Unsafe");
  Field f = unsafeClass.getDeclaredField("theUnsafe");
  f.setAccessible(true);
  final Object unsafe = f.get(null);
  final Method allocateInstance = unsafeClass.getMethod("allocateInstance", Class.class);
  return new UnsafeAllocator() {
    @Override
    @SuppressWarnings("unchecked")
    public <T> T newInstance(Class<T> c) throws Exception {
      assertInstantiable(c);
      return (T) allocateInstance.invoke(unsafe, c);
    }
  };
} catch (Exception ignored) {
}
// try dalvikvm, post-gingerbread use ObjectStreamClass
// try dalvikvm, pre-gingerbread , ObjectInputStream
}
```

所以我们Person没有提供默认的构造方法，Gson在没有找到默认构造方法时，它就直接通过Unsafe的方法，绕过了构造方法，直接构建了一个对象。
53. 协程是非阻塞式挂起的是什么意思呢？就是一段程序切到另一个线程执行，不会卡住到主线程。
54. 扔物线对协程的总结:
        定义: 协程是一套协程框架.
              协程是一套处理并发事件的方案, 同时也是方案中的核心组件. (参考 Adapter)
              协程和线程一个层级.
              协程中没有线程, 也不存在并行.
              Kotlin 协程不需要关心并行/并发, 同时 Kotlin for Java 不属于广义的协程, 它是基于 Java 线程的, 所以不是什么轻量级线程.
        使用: - launch() 开启一段协程
             - 需要后台工作的函数加上 Suspend, 在内部调用其他的 Suspend 函数切换线程, 一般都是用 withContext
             - 主函数调用时, 一条线写下来
             PS: 耗时操作都放在 Suspend 函数中, 主函数不用关心耗时操作的内部线程切换.
        Suspend: 不是线程切换, 只是标记和提醒; Suspend 函数里的协程切换, 不会影响调用函数的线程.

