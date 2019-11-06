---
title: 如何使用hide掉的类及接口
date: 2019-03-24 18:35:09
categories: "Android"
tags:
    - hide
    - framework
    - 反射
---

在做蓝牙电话的时候，发现有的类或者api是被 @hide 掉的，例如 BluetoothPbapClient.java 没法调用。
最正确的方法应该是自己修改定制源码，再编译使用。但是如果仅是测试用，可以继续考虑使用 hide 的类及接口，这里做个简单的总结。

# 不能访问的类或者接口

Android 有两种类型的 API 不能通过 SDK 访问。一种是在 com.android.internal 包中的 API(源码路径为framework/base/core/)。另一种是被标记为 @hide 属性的类和接口。

对于internal API来说，从来都没有计划将其开放出来。它就是Android的“内部厨房”，对开发者来说，应该将其视作黑盒。

Hidden API之所以被隐藏，是想阻止开发者使用SDK中那些未完成或不稳定的部分（接口或类），因为其可能会在后续的版本中被修改或者移除。

当使用 Android SDK 进行开发的时候，应用默认引用了 android.jar，它位于 sdk-platform-{android版本}-android.jar。SDK 中默认移除了所有的被@hide标识的接口或者类以及 internal 包下的类。

当应用在设备上运行时，它会加载 framework.jar。简单来说，framework.jar 和 android.jar 等同，但是没有移除 internal API 和 hidden API。

<!-- more -->

# 解决方案1：反射

可以通过反射调到hide的类或者接口，这方面使用之前已经有介绍了：

[反射机制原理及使用](https://mcl-123.github.io/2019/03/15/%E5%8F%8D%E5%B0%84%E6%9C%BA%E5%88%B6%E5%8E%9F%E7%90%86%E5%8F%8A%E4%BD%BF%E7%94%A8/)

# 解决方案2：原始 android.jar

我们需要修改android.jar，这样它才能包含所有的*.class文件（包括internal和hidden API类）。有两种办法：

## 编译源码

Android是一个开源工程。我们可以下载源码并搭建编译环境，这样它就不能移除那些internal和hidden的类了。

## 从模拟器中提取

每个模拟器或真机在运行时都会有一个等同android.jar的东西。我们可以从这里拿到jar文件，提取出原始的.class文件，并拷贝到Android SDK的android.jar中。

具体步骤可以看这里：[android怎样调用@hide和internal API](https://blog.csdn.net/hudan2714/article/details/7853908)

## android-hidden-api

GitHub 上有一个项目：[android-hidden-api](https://github.com/anggrayudi/android-hidden-api)，里面提供了众多版本完整的 android.jar 包

## 总结

在开发中使用隐藏 API 和内部 API 是不推荐的做法，因为在后续版本中很可能大改动。
