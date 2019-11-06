---
title: NavigationBar 分析
date: 2019-02-24 23:15:10
categories: "Android源码"
tags:
    - Android
    - NavigationBar
    - 源码
---

# NavigationBar 分析

## 说明

Android手机可分为有导航栏以及没导航栏两种，一般有物理按键的机器不会带有导航栏，而没有物理按键的机器则基本会带。这里主要是分析导航栏导航栏的加载以及按键实现。

<!-- more -->

## 导航栏的加载以及按键实现

导航栏是属于系统界面的一部分，也就是SystemUI的一部分。

StatusBar.java：

makeStatusBarView():

```java
boolean showNav = mWindowManagerService.hasNavigationBar();
if (showNav) {
    createNavigationBar();
}
```

createNavigationBar():

```java
mNavigationBarView = NavigationBarFragment.create(mContext, (tag, fragment) -> {
            mNavigationBar = (NavigationBarFragment) fragment;
            if (mLightBarController != null) {
                mNavigationBar.setLightBarController(mLightBarController);
            }
            mNavigationBar.setCurrentSysuiVisibility(mSystemUiVisibility);
        });
```

NavigationBarFragment.java:

onCreate():

```java
inflater.inflate(R.layout.navigation_bar, container, false)
```

navigation_bar.xml:

```xml
<com.android.systemui.statusbar.phone.NavigationBarView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:systemui="http://schemas.android.com/apk/res-auto"
    android:layout_height="match_parent"
    android:layout_width="match_parent"
    android:background="@drawable/system_bar_background">

    <com.android.systemui.statusbar.phone.NavigationBarInflaterView
        android:id="@+id/navigation_inflater"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</com.android.systemui.statusbar.phone.NavigationBarView>
```

NavigationBarView.java: // mRecentIcon

onFinishInflate():

```java
mNavigationInflaterView = (NavigationBarInflaterView) findViewById(
                R.id.navigation_inflater);
```

NavigationBarInflaterView.java:

onFinishInflate():

```java
inflateLayout(getDefaultLayout());
```

inflateButtons():

```java
for (int i = 0; i < buttons.length; i++) {
    inflateButton(buttons[i], parent, landscape);
}
```

inflateButton():

```java
View v = createView(buttonSpec, parent, inflater, landscape);
```

createView():

```java
if (HOME.equals(button)) {
    v = inflater.inflate(R.layout.home, parent, false);
} else if (BACK.equals(button)) {
    v = inflater.inflate(R.layout.back, parent, false);
} else if (RECENT.equals(button)) {
    v = inflater.inflate(R.layout.recent_apps, parent, false);
}

```

home.xml:

```xml
<com.android.systemui.statusbar.policy.KeyButtonView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:systemui="http://schemas.android.com/apk/res-auto"
    android:id="@+id/home"
    android:layout_width="@dimen/navigation_key_width"
    android:layout_height="match_parent"
    android:layout_weight="0"
    systemui:keyCode="3"
    android:scaleType="center"
    android:contentDescription="@string/accessibility_home"
    android:paddingStart="@dimen/navigation_key_padding"
    android:paddingEnd="@dimen/navigation_key_padding"
    />
```

KeyButtonView.java:

KeyButtonView():

```java
// id为home以及id为back的KeyButtonView就分别通过systemui:keyCode属性对其进行了设置，设置的值分别是3和4
mCode = a.getInteger(R.styleable.KeyButtonView_keyCode, 0);
```

onTouchEvent():

```java
// 当mCode为0的时候，KeyButtonView并不会调用sendEvent方法
switch (action) {
    case MotionEvent.ACTION_DOWN:
        mDownTime = SystemClock.uptimeMillis();
        mLongClicked = false;
        setPressed(true);
        if (mCode != 0) {
            sendEvent(KeyEvent.ACTION_DOWN, 0, mDownTime);
        } else {
            // Provide the same haptic feedback that the system offers for virtual keys.
            performHapticFeedback(HapticFeedbackConstants.VIRTUAL_KEY);
        }
        break;
    case MotionEvent.ACTION_CANCEL:
        setPressed(false);
        if (mCode != 0) {
            sendEvent(KeyEvent.ACTION_UP, KeyEvent.FLAG_CANCELED);
        }
        break;
    case MotionEvent.ACTION_UP:
        if (mCode != 0) {
            if (doIt) {
                sendEvent(KeyEvent.ACTION_UP, 0);
                sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);
            } else {
                sendEvent(KeyEvent.ACTION_UP, KeyEvent.FLAG_CANCELED);
            }
        } else {
            // no key code, just a regular ImageView
            if (doIt && mOnClickListener != null) {
                mOnClickListener.onClick(this);
                sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);
            }
        }
        break;
}
```

sendEvent():

```java
mMetricsLogger.write(new LogMaker(MetricsEvent.ACTION_NAV_BUTTON_EVENT)
        .setType(MetricsEvent.TYPE_ACTION)
        .setSubtype(mCode)
        .addTaggedData(MetricsEvent.FIELD_NAV_ACTION, action)
        .addTaggedData(MetricsEvent.FIELD_FLAGS, flags));
final int repeatCount = (flags & KeyEvent.FLAG_LONG_PRESS) != 0 ? 1 : 0;
final KeyEvent ev = new KeyEvent(mDownTime, when, action, mCode, repeatCount,
        0, KeyCharacterMap.VIRTUAL_KEYBOARD, 0,
        flags | KeyEvent.FLAG_FROM_SYSTEM | KeyEvent.FLAG_VIRTUAL_HARD_KEY,
        InputDevice.SOURCE_KEYBOARD);
// 构建出一个对应的keyCode的KeyEvent ev，然后调用InputManager的injectInputEvent模拟发送来实现与物理按键相同的功能
InputManager.getInstance().injectInputEvent(ev,
        InputManager.INJECT_INPUT_EVENT_MODE_ASYNC);
```

NavigationBarFragment.java:

onViewCreated():

```java
prepareNavigationBarView();
```

prepareNavigationBarView():

```java
ButtonDispatcher recentsButton = mNavigationBarView.getRecentsButton();
recentsButton.setOnClickListener(this::onRecentsClick);
```

onRecentsClick():

```java
private void onRecentsClick(View v) {
    mStatusBar.awakenDreams();
    mCommandQueue.toggleRecentApps();
}
```

## App实现隐藏导航栏(沉浸式)

MainActivity.java:

```java
@Override
public void onWindowFocusChanged(boolean hasFocus) {
    super.onWindowFocusChanged(hasFocus);
    if (hasFocus) {
        View decorView = getWindow().getDecorView();
        decorView.setSystemUiVisibility(
                View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                        | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                        | View.SYSTEM_UI_FLAG_HIDE_NAVIGATION
                        | View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY);
    }
}
```
