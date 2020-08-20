---
title: Kotlin 协程
date: 2020-07-22 14:12:23
categories: "Kotlin"
tags:
     - Kotlin
     - 协程
---

此文是协程相关的blog\文章\demo, 做总结或简化, 或使用心得等等, 统一放在这里, 方便查找及总结.

<!-- more -->

## 《第一行代码》(第三版) 之 Kotlin 的总结

1. 协程：简单理解为一种轻量级的线程，用来提升并发编程的运行效率。(一套线程框架)
        有点: 方便, 方便在它能在同一个代码块里进行多次的线程切换.
        开启方式：
                - Global.launch {} : 每次创建的都是一个顶层协程，这种协程当应用程序运行结束时跟着一起结束。
                - runBlocking 函数 : 同样会创建一个协程的作用域，但是它可以保证在协程作用域内的所有代码和子代码没有全部执行完成前一直阻塞当前线程 (PS: 只在测试环境使用，生产环境会有性能问题)
                - launch 函数 : 用来创建多个协程，但是首先它必须要在协程的作用域下才能调用；其次它会在当前协程的作用域下创建子协程 （PS: 子协程的特点是 如果外层作用域的协程结束了，该作用域下的所有子协程也会一同结束）
        delay 函数: 非阻塞式的挂起函数, 只会挂起当前协程，并不会影响其他协程 (PS: delay 函数只能在协程作用域或其它挂起函数中调用)
        协程的取消: GlobalScope.launch 或 launch 函数都会返回一个 Job 对象, 只要调用 Job 对象的 cancel() 方法就可以取消协程了
2. 挂起函数: suspend
        可以将任意的函数声明成挂起函数，而挂起函数之间是可以互相调用的；suspend 无法给函数提供协程作用域的。
3. coroutineScope 函数: 用来解决 suspend 挂起函数无法提供协程作用域的问题。
        特点: - 会继承外部的协程作用域并创建一个子作用域, 这样就可以给任意挂起函数提供协程作用域了；
             - 保证其作用域内的所有代码和子协程在全部执行完之前, 会一直阻塞当前协程。
             - 可以在协程作用域或者挂起函数内调用
4. async 函数: 会创建一个新的子协程并返回一个 Deferred 对象, 如果我们想要获取 async 函数代码块的执行结果，只需要调用 Deferred 对象的 await() 方法即可。(PS: async 只能在协程作用域里调用)。
        调用 async 函数之后, 会立刻执行代码块中的代码; 当调用 await() 方法时，如果代码块里的代码还没执行完, 那么 await() 方法会将当前协程阻塞, 直到可以获得 async 函数的执行结果。
5. withContext 函数: 一个挂起函数, 可以理解成 async 函数的一种简化版写法，相当于 async {}.await(), 不同的是 withContext 函数强制要求我们指定一个线程参数。
6. 线程参数:
        - Dispatchers.Default : 默认低并发的线程策略, 当你要执行的代码属于计算密集型任务时
        - Dispatchers.IO : 较高并发线程策略, 执行的代码大多数时间是阻塞和等待中
        - Dispatchers.Main : Android 主线程中执行代码
        PS: 协程作用域构建器中，除了 coroutineScope 函数之外, 其他所有的函数都是可以指定这样一个线程参数的。
7. suspendCoroutiner 函数:
        作用: 简化传统回调机制
        必须在协程作用域或挂起函数中调用
        使用: 接收一个 Lambda 表达式参数, 将当前协程立即挂起, 然后在一个普通线程中执行 Lambda 表达式中的代码。Lambda 表达式的参数列表中会传入一个 Continuation 参数, 调用它的 resume()方法或 resumeWithException() 方法可以让协程恢复。

## 来源扔物线的视频

1. 协程是非阻塞式挂起的是什么意思呢？就是一段程序切到另一个线程执行，不会卡住到主线程。
2. 扔物线对协程的总结:
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
        协程泄露本质就是线程泄露 (同 AsyncTask)

## blog/demo

[《“吹上天”的Kotlin协程 要不看下实战？》](https://mp.weixin.qq.com/s?__biz=MzAxMTI4MTkwNQ==&mid=2650829957&idx=1&sn=d3beb0bf0a0473a176735ec0569dbdae&chksm=80b7a01bb7c0290d64d660878370a63e674a1239e2f372f20cefa0a792208c28f8d88769acf9&scene=21#wechat_redirect)

1. Retrofit 2.6 后对 Kotlin 协程也做了支持, 加个 suspend 直接返回结果, 而不是 Call 对象

```kotlin
@GET("banner/json")
suspend fun getBanner(): BaseResult<List<BannerBean>>
```

2. 在 ViewModel 中使用 viewModelScope.launch {} 可以启动一个协程, 并且他会在 ViewModel 销毁的时候，自动取消他自己和在他内部启动的所有协程

3. flow: 在协程中，做异步可以返回一个值，当我们想返回多个值的时候，Flow就开始展现他的作用

    - flow：构建器，他可以发射数据多个数据，用 emit() 来发射
    - flatMapConcat ：这个是在一个流收集完成之后，再收集下一个流
    - onStart：这个看名字估计也能猜出来，就是在发射之前做一些事情，我们可以在这里再 emit()一个数据，他会在flow里边的数据发射之前发射，我们上边的例子，是在OnStart里边打开了等待框
    - flowOn：这个就是指定我们的流运行在那个协程里边
    - onCompletion ：是在所有流都收集完成了，就会触发，我们可以在这里取消等待框再合适不过了
    - catch：这个就是遇到错误的时候会触发，我们我错误处理就是在这里来做了 (只能捕捉上游, 要想捕捉 collect 的话, 得用 try/catch 了)
    - collect：这个就是收集器的意思，我们的结果都在这里来处理。也只有我们调用了这个收集方法，数据才真正的开始发射了，这也是官方说的一句话，流是冷的，就是这个意思

4. Room 使用自己的调度器在后台线程进行查询操作。你不应该再使用 withContext(Dispatchers.IO) 来调用 Room 的 suspend 查询，这只会让你的代码运行的更慢。

5. job.join() // 等待请求的完成，包括其所有子协程, 会阻塞当前线程

6. Flow 是一种类似于序列的冷流, flow 构建器中的代码直到流被收集的时候才运行。

7. 流构建器: - flow{}
            - flowOf
            - .asFlow()

8. flow操作符: map(); filter(); transform(); take(); reduce()

9. flow {...} 构建器中的代码必须遵循上下文保存属性，不允许内部更换 context 来 emit, 但是可以通过 flowOn() 来改变流构建器中的上下文 (即 emit 发生在另一个 context 中, 与 collect 不一样的 context)

10. buffer() 用来缓冲 emit 项

11. conflate() 合并 emit 项  (PS: 有 buffer 效果, 并且当收集的时候, 同事有多个值, 只会收集最新的一个值)

12. collectLatest() 取消耗时的 collect 项, 并重新发射最后的一个值 (PS: 每个 emit 都会执行 collect, 但是如果有新的 emit 出来的话, 就会立刻中断上一个 collect)

13. zip() 组合两个流中的值

14. combine() 和 zip 类似是对两个流中的值做组合, 但是 combine 是每当 两个流中的值有任何一个 emit 时, 都会 collect; 而 zip 是两个流组合成一对, 每都有一个 emit 时, 才会 collect

15. flatMapConcat() 第一个流 emit 一项走到第二个流再走到收集, 等第二个流走后的的 collect 项走完, 再循环走到第一个流的 emit 下一个项

16. flatMapMerge() 把所有的流合并到一个单独的流, 然后按时间顺序收集, 第二个流的耗时不影响第一个流的 emit

17. flatMapLatest() 类似 collectLatest(), 第一个流有新的 emit 项, 就会立即中断上一个 collect

18. launchIn() 单独的协程中执行流, 返回的是一个 job

19. CoroutineExceptionHandler 一个上下文, 用来处理全局异常处理

20. coroutineScope 和 supervisorScope 的 区别: coroutineScope 内部的取消操作是双向传播的(PS: 任何一个子协程异常退出，那么整体都将退出); 而 supervisorScope 同样继承外部作用域的上下文，但其内部的取消操作是单向传播的，只是父协程向子协程传播(子协程出了异常并不会影响父协程以及其他兄弟协程)

21. (取消 + 阻塞式的任务)[https://blog.csdn.net/tigershin/article/details/86482808]: 使用suspendCancellableCoroutine (用来取消 socket 通信等)
