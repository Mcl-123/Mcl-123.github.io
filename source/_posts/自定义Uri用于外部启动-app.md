---
title: 自定义Uri用于外部启动 app
date: 2019-02-20 22:12:33
categories: "Android"
tags:
    - Android
    - uri
---

项目有个需求，通过语音打开相应的app，例如我说：“去新街口”，就打开高德地图。讯飞语音demo中就有这样的例子，而它就是通过uir实现的。所以在这儿对该实现做了个总结，以便随时之需。

<!-- more -->

# 自定义Uri用于外部启动 app

## 新建用于外部启动的Activity

应用 A：SchemeURL
activity: SecondActivity

```java
<activity
    android:name=".SecondActivity">
    <intent-filter>
        <action android:name="android.jackie.schemeurl.activity" />
        <category android:name="android.intent.category.DEFAULT" />
        <data
            android:scheme="jackie"
            android:host="test.uri.activity" />
    </intent-filter>
</activity>
```

## 新建启动外部应用的应用

应用 B：StartSchemeURL

```java
Uri uri = Uri.parse("jackie://test.uri.activity?action=123"); // action 为传递的数据
Intent intent = new Intent("android.jackie.schemeurl.activity");
intent.setData(uri);
startActivity(intent);
```

## 通过Uri来传递参数

应用 A：SchemeURL
activity: SecondActivity

```java
Intent intent = getIntent();
if (null != intent) {
    Uri uri = intent.getData();
    if (uri == null) {
        return;
    }
    String acionData = uri.getQueryParameter("action");
}
```
