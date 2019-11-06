---
title: BroadcastReceiver 静态注册与动态注册
date: 2019-02-21 21:51:51
categories: "Android"
tags:
    - Android
    - BroadcastReceiver
---

上篇文章提到了 使用自定义uri用于外部启动 app，而我们也可以使用广播来启动外部app，这篇文章简单的介绍下android广播的使用。

# BroadcastReceiver 静态注册与动态注册

## 接收广播

```java
public class MyReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        //  UI 线程，不建议过多操作

        // 注：启动Activity在这些类中是可以的，但是需要创建一个新的task。一般情况不推荐, 见Context 干货：(https://www.jianshu.com/p/881acfafd18d)
        // activityIntent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
    }
}
```

### 静态注册

```xml
<receiver android:name=".MyReceiver">
    <intent-filter>
        <action android:name="music"/>
    </intent-filter>
</receiver>
```

### 动态注册

```java
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    IntentFilter filter=new IntentFilter("music");
    mBroadcast =new MyReceiver();
    registerReceiver(mBroadcast,filter);
}

protected void onDestroy() {
    super.onDestroy();
    unregisterReceiver(mBroadcast);
}
```

## 发送广播

```java
Intent intent = new Intent("music");
// 8.0后，静态注册广播接收者时，需 setComponent()
intent.setComponent(new ComponentName("com.example.pvwav.broadcastreceiver", "com.example.pvwav.broadcastreceiver.MyReceiver"));
sendBroadcast(intent);
```
