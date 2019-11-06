---
title: POJO
date: 2018-09-05 21:10:18
categories: "Java"
tags:
    - java
    - pojo
---

# POJO

Plain Ordinary Java Object
简单地 Java 对象, 实际上就是普通的 JavaBeans, 是为了避免和 EJB 混淆所创造的简称.

<!-- more -->

JavaBean 是一种JAVA语言写成的可重用组件。它的方法命名，构造及行为必须符合特定的约定：

- 这个类必须有一个公共的缺省构造函数
- 这个类的属性使用getter和setter来访问，其他方法遵从标准命名规范
- 这个类应是可序列化的

因为这些要求主要是靠约定而不是靠实现接口，所以许多开发者把JavaBean看作遵从特定命名约定的POJO。

简而言之，当一个Pojo可序列化，有一个无参的构造函数，使用getter和setter方法来访问属性时，他就是一个JavaBean。

简单理解:

POJO 是 DO/DTO/BO/VO 的统称, 但禁止被命名成 xxxPOJO 而已.

- 数据对象: xxxDO
- 数据传输对象: xxxDTO
- 展示对象: xxxVO
- 业务对象: xxxBO