---
title: Android SystemUI 源码分析
date: 2019-01-27 18:32:19
categories: "Android源码"
tags:
    - Android
    - SystemUI
    - 源码
---

因为项目需要，所以对 Android 源码中的 SystemUI 做了点基本分析。

<!-- more -->

## 平台

Android 5.1

## 源代码路径

/frameworks/base/packages/SystemUI/src/com/android/systemui/

## 什么是SystemUI

其实就是Android的系统界面，包括状态栏Status Bar，导航栏Navigation Bar，锁屏界面Keyguard，电源界面PowerUI，最近任务管理Recents 等等。

![SystemUI相关类继承关系](/images/SystemUI相关类继承关系.jpg)

## 启动流程

SystemServer.java:

```java
public static void main（String[] args）{
    new SystemServer().run()
}

private void run() {
    // Start services.
    startBootstrapServices();  // 启动系统启动所需的关键服务
    startCoreServices();    // 启动基本服务
    startOtherServices();   // 启动其他服务
}

private void startOtherServices() {
    mActivityManagerService.systemReady(new Runnable() {
        @Override
        public void run() {
            startSystemUi();
        }
    });
}

static final void startSystemUi(Context context) {
    Intent intent = new Intent();
    intent.setComponent(new ComponentName("com.android.systemui",
                "com.android.systemui.SystemUIService"));
    context.startServiceAsUser(intent, UserHandle.OWNER);
}
```

然后我们进入设置启动 systemui 程序的 SystemUIService.java:

```java
@Override
public void onCreate() {
    super.onCreate();
    ((SystemUIApplication) getApplication()).startServicesIfNeeded();
}
```

SystemUIApplication.java:

```java

private final Class<?>[] SERVICES = new Class[] {
        com.android.systemui.keyguard.KeyguardViewMediator.class,
        com.android.systemui.recent.Recents.class,
        com.android.systemui.volume.VolumeUI.class,
        com.android.systemui.statusbar.SystemBars.class,
        com.android.systemui.usb.StorageNotification.class,
        com.android.systemui.power.PowerUI.class,
        com.android.systemui.media.RingtonePlayer.class
};

// 这里是拿到每个和 SystemUI 相关的类的反射，存到了 service[] 里，然后赋值给cl，紧接着将通过反射将其转化为具体类的对象，存到了mService[i]数组里，最后对象调 start() 方法启动相关类的服务，启动完成后，回调 onBootCompleted( ) 方法

public void startServicesIfNeeded() {
    final int N = SERVICES.length;
    for (int i=0; i<N; i++) {
        Class<?> cl = SERVICES[i];
        mServices[i] = (SystemUI)cl.newInstance();
        mServices[i].mContext = this;
        mServices[i].mComponents = mComponents;
        mServices[i].start();
        if (mBootCompleted) {
            mServices[i].onBootCompleted();
        }
    }
}
```

// 以 SystemBars 的 start() 为例，所以mService[i].start() 先认为是 SystemBars.start()

SystemBars.java:

```java
@Override
public void start() {
    mServiceMonitor = new ServiceMonitor(TAG, DEBUG,
            mContext, Settings.Secure.BAR_SERVICE_COMPONENT, this);
    mServiceMonitor.start();  // will call onNoService if no remote service is found
}

@Override
public void onNoService() {
    createStatusBarFromConfig();  // fallback to using an in-process implementation
}

private void createStatusBarFromConfig() {
    // R.string.config_statusBarComponent: com.android.systemui.statusbar.phone.PhoneStatusBar
    final String clsName = mContext.getString(R.string.config_statusBarComponent);
    Class<?> cls = null;
    cls = mContext.getClassLoader().loadClass(clsName);
    mStatusBar = (BaseStatusBar) cls.newInstance();
    mStatusBar.mContext = mContext;
    mStatusBar.mComponents = mComponents;
    mStatusBar.start();
}
```

PhoneStatusBar.java:

```java
@Override
public void start() {
    super.start(); // calls createAndAddWindows()
}

@Override
public void createAndAddWindows() {
    addStatusBarWindow();
}

private void addStatusBarWindow() {
    makeStatusBarView();
}

// Constructing the view
protected PhoneStatusBarView makeStatusBarView() {
    mStatusBarWindow = (StatusBarWindowView) View.inflate(context,
            R.layout.super_status_bar, null);
}
```

### 通过 super_status_bar.xml 的分析 SystemBars 的大致视图构成了:

```xml
<com.android.systemui.statusbar.phone.StatusBarWindowView>
    <com.android.systemui.statusbar.BackDropView android:id="@+id/backdrop">
        <ImageView android:id="@+id/backdrop_back" />
        <ImageView android:id="@+id/backdrop_front" />
    </com.android.systemui.statusbar.BackDropView>
    <com.android.systemui.statusbar.ScrimView android:id="@+id/scrim_behind" />
    <!-- 系统状态栏的布局文件 -->
    <include layout="@layout/status_bar" />
    <FrameLayout android:id="@+id/brightness_mirror">
        <FrameLayout>
            <!-- 下拉的通知窗口的布局文件 -->
            <include layout="@layout/quick_settings_brightness_dialog" />
        </FrameLayout>
    </FrameLayout>
    <com.android.systemui.statusbar.phone.PanelHolder android:id="@+id/panel_holder">
        <include layout="@layout/status_bar_expanded" />
    </com.android.systemui.statusbar.phone.PanelHolder>
    <com.android.systemui.statusbar.ScrimView android:id="@+id/scrim_in_front" />
</com.android.systemui.statusbar.phone.StatusBarWindowView>
```

![super_status_bar](/images/super_status_bar.jpg)

#### status_bar

PhoneStatusBarView 即为手机最上方的状态栏，主要用于显示系统状态，通知等，主要包括 notification icons 和 status bar icons。status_bar.xml 即对应状态栏的视图如下:

```xml
<com.android.systemui.statusbar.phone.PhoneStatusBarView android:id="@+id/status_bar">
    <ImageView android:id="@+id/notification_lights_out" />
    <LinearLayout android:id="@+id/status_bar_contents">
        <ImageView android:id="@+id/status_bar_weather_image" />
        <TextView android:id="@+id/status_bar_weather_text" />
        <TextView android:id="@+id/status_bar_temperature" />
        <TextView android:id="@+id/status_bar_location" />
        <com.android.systemui.statusbar.AlphaOptimizedFrameLayout android:id="@+id/notification_icon_area">
            <!-- The alpha of this area is both controlled from PhoneStatusBarTransitions and
                 PhoneStatusBar (DISABLE_NOTIFICATION_ICONS), so we need two views here. -->
            <com.android.keyguard.AlphaOptimizedLinearLayout android:id="@+id/notification_icon_area_inner">
                <com.android.systemui.statusbar.StatusBarIconView android:id="@+id/moreIcon" />
                <com.android.systemui.statusbar.phone.IconMerger android:id="@+id/notificationIcons" />
            </com.android.keyguard.AlphaOptimizedLinearLayout>
        </com.android.systemui.statusbar.AlphaOptimizedFrameLayout>
        <com.android.keyguard.AlphaOptimizedLinearLayout android:id="@+id/system_icon_area">
            <include layout="@layout/system_icons" />
            <com.android.systemui.statusbar.policy.Clock android:id="@+id/clock" />
        </com.android.keyguard.AlphaOptimizedLinearLayout>
    </LinearLayout>
    <ViewStub android:id="@+id/ticker_stub" />
</com.android.systemui.statusbar.phone.PhoneStatusBarView>
```

![status_bar](/images/status_bar.jpg)

#### PanelHolder

PanelHolder是用户下拉 status bar 后得到的 view。它主要包含 QuickSettings 和 Notification panel 两个部分。 

status_bar_expanded.xml:

```xml
<com.android.systemui.statusbar.phone.NotificationPanelView>
    <include layout="@layout/carrier_label" />
    <include layout="@layout/keyguard_status_view" />
    <TextView android:id="@+id/emergency_calls_only"
        android:textAppearance="@style/TextAppearance.StatusBar.Expanded.Network.EmergencyOnly" />
    <com.android.systemui.statusbar.phone.NotificationsQuickSettingsContainer android:id="@+id/notification_container_parent">
        <com.android.systemui.statusbar.phone.ObservableScrollView android:id="@+id/scroll_view">
            <LinearLayout>
                <include layout="@layout/qs_panel" />
                <View android:id="@+id/reserve_notification_space" />
                <View />
            </LinearLayout>
        </com.android.systemui.statusbar.phone.ObservableScrollView>
        <com.android.systemui.statusbar.stack.NotificationStackScrollLayout android:id="@+id/notification_stack_scroller" />
        <ViewStub android:id="@+id/keyguard_user_switcher" android:layout="@layout/keyguard_user_switcher" />
        <include layout="@layout/keyguard_status_bar" />
    </com.android.systemui.statusbar.phone.NotificationsQuickSettingsContainer>
    <include layout="@layout/keyguard_bottom_area" />
    <include layout="@layout/status_bar_expanded_header" />
    <com.android.systemui.statusbar.AlphaOptimizedView android:id="@+id/qs_navbar_scrim" />
</com.android.systemui.statusbar.phone.NotificationPanelView><!-- end of sliding panel -->
```

![status_bar_expanded](/images/status_bar_expanded.jpg)

## SystemUI启动流程图

![SystemUI启动流程图1](/images/SystemUI启动流程图1.jpg)
![SystemUI启动流程图2](/images/SystemUI启动流程图2.jpg)
