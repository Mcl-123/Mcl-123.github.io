---
title: Android Daily Interview
date: 2019-11-06 22:21:47
tags:
     - Android
     - interview
categories: "Android"
top: 1
---

以下内容参考 [Android Daily Interview](https://github.com/Moosphan/Android-Daily-Interview)
小聚成河，大聚成江，坚持下来的都是时代的铸就者，共勉之！

<!-- more -->

## 自定义 Handler 时如何有效地避免内存泄漏问题？

原因:
非静态的内部类会持有外部类的引用,若 Activity 退出时, handler 队列中有未处理的 Message, handler 持有 activity, Message 持有 handler, MessageQueen 持有 Message, 则 activity 无法销毁, 就会造成内存泄漏.

解决方案: 
1. 自定义静态 Handler
2. 加一个弱引用
3. ondestroy()时, removecallbacksandmessages去清除Message和Runnable

## Activity 与 Fragment 之间常见的几种通信方式？

1. 接口
2. Bundle
3. 广播
4. EventBus
5. viewModel 

再简单说: 两个 Java 对象相互持有对方的引用,直接回调就行.

## 一般什么情况下会导致内存泄漏问题？

原因: 
1. 资源对象没关闭 ( Cursor、File 等)
2. 接收器、监听器没取消 (广播、EventBus等)
3. 持有 Activity 的 Context (Context 与 单例)
4. 非静态内部类持有外部类的引用所造成的 (handler)
5. 静态类持有非静态类对象 (工具类持有 Activity context)

如何避免:
- 编码规范上:
    1. 资源对象用完一定要关闭，最好加finally
    2. 接收器、监听器使用时候注册和取消成对出现
    3. context使用注意生命周期，如果是静态类引用直接用ApplicationContext
    4. 使用静态内部类;合业务场景，设置软引用，弱引用，确保对象可以在合适的时机回收
    5. 同 3

- 内存监控体系:
    1. 使用ArtHook检测图片尺寸是否超出imageview自身宽高的2倍
    2. 编码阶段Memery Profile看app的内存使用情况，是否存在内存抖动，内存泄漏，结合Mat分析内存泄漏
    3. 使用LeakCannery自动化内存泄漏分析
    4. 上报app使用期间待机内存、重点模块内存、OOM率
    5. 报整体及重点模块的GC次数，GC时间

- 兜底策略:
    低内存状态回调，根据不同的内存等级做一些事情，比如在最严重的等级清空所有的bitmap，关掉所有界面，直接强制把app跳转到主界面，相当于app重新启动了一次一样，这样就避免了系统Kill应用进程，与其让系统kill进程还不如浪费一些用户体验，自己主动回收内存

## LaunchMode 的应用场景？

### Standard 标准模式

系统总是会在目标栈中创建新的activity实例。

for example: 个人中心 -> 粉丝列表 -> 个人中心(粉丝的) -> 粉丝列表(粉丝的)

### SingleTop 栈顶模式

- 如果这个 activity 实例已经存在目标栈的栈顶，系统会调用这个 activity 中的 onNewIntent() 方法，并传递 intent，而不会创建新的 activity 实例；
- 如果不存在这个 activity 实例或者 activity 实例不在栈顶，则 SingleTop 和 Standard 作用是一样的。

for example: 从消息中心点开的详情页面

### SingleTask 栈中模式

- 不会存在多个实例，如果栈中不存在 activity 实例，系统会在新栈的根部创建一个新的 activity；
- 如果这个 activity 实例已经存在，系统会把该 activity 栈上面的 其他 activity 都销毁，并调用该 activity 的 onNewIntent() 方法，而不会创建新的 activity 实例。

for example: app 首页,从任何页面回来,都把上面的 activity 销毁

### SingleInstance

会启用一个新的栈结构，将 Acitvity 放置于这个新的栈结构中，并保证不再有其他 Activity 实例进入该新的栈。

for example: 闹钟提醒页面、打电话页面，当你从正常页面调到闹钟提醒页面，再返回的时候，希望返回之前的页面，且不受影响。

## 哪些情况下会导致oom问题？

简单说：
1. 申请内存的速度超出gc释放内存的速度， 例如大图加载
2. 过多的泄露也会导致溢出

总结：

1、根据java的内存模型会出现内存溢出的内存有堆内存、方法区内存、虚拟机栈内存、native方法区内存；
2、一般说的OOM基本都是针对堆内存；
3、对于堆内存溢出主的根本原因有两种
（1）app进程内存达到上限
（2）手机可用内存不足，这种情况并不是我们app消耗了很多内存，而是整个手机内存不足
4、而我们需要解决的主要是app的内存达到上限
5、对于app内存达到上限只有两种情况
（1）申请内存的速度超出gc释放内存的速度
（2）内存出现泄漏，gc无法回收泄漏的内存，导致可用内存越来越少
6、对于申请内存速度超出gc释放内存的速度主要有2种情况
（1）往内存中加载超大文件
（2）循环创建大量对象
7、一般申请内存的速度超出gc释放内存基本不会出现，内存泄漏才是出现问题的关键所在
8、内存泄漏常见场景
    见问题《一般什么情况下会导致内存泄漏问题？》
9、怎么对内存进行优化呢
三个方向
（1）为应用申请更大内存，把manifest上的largdgeheap设置为true
（2）减少内存的使用
①使用优化后的集合对象，比如SpaseArray；
②使用微信的mmkv替代sharedpreference；
③对于经常打log的地方使用StringBuilder来组拼，替代String拼接
④统一带有缓存的基础库，特别是图片库，如果用了两套不一样的图片加载库就会出现2个图片各自维护一套图片缓存
⑤给ImageView设置合适尺寸的图片，列表页显示缩略图，查看大图显示原图；图片使用软引用，内存不够及时回收
⑥优化业务架构设计，比如省市区数据分批加载，需要加载省就加载省，需要加载市就加载失去，避免一下子加载所有数据
（3）避免内存泄漏
同见问题《一般什么情况下会导致内存泄漏问题？》

## 如何实现多线程中的同步？





