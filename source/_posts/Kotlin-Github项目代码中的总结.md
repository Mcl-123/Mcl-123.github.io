---
title: Kotlin Github项目代码中的总结
date: 2020-08-20 22:41:53
categories: "Kotlin"
tags:
     - Kotlin
     - 协程
---

本文列举部分 Kotlin Github 项目中代码的实际使用等.

<!-- more -->

## [Kotlin-Coroutine-Use-Cases-on-Android](https://github.com/LukasLechnerDev/Kotlin-Coroutine-Use-Cases-on-Android)


### 三方库

```gradle
    apply plugin: 'kotlin-kapt'

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = JavaVersion.VERSION_1_8.toString()
    }

implementation "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
    implementation 'androidx.core:core-ktx:1.3.1'

    def lifecycle_version = "2.2.0"

    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.7"
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.7"

    implementation 'com.google.android.material:material:1.2.0'

    implementation "androidx.activity:activity-ktx:1.1.0"

    implementation 'androidx.recyclerview:recyclerview:1.1.0'
    implementation 'androidx.cardview:cardview:1.0.0'

    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"
    implementation "androidx.lifecycle:lifecycle-livedata-ktx:$lifecycle_version"

    implementation 'com.squareup.retrofit2:retrofit:2.7.1'
    implementation 'com.squareup.retrofit2:adapter-rxjava2:2.7.1'
    implementation 'com.google.code.gson:gson:2.8.6'
    implementation 'com.squareup.retrofit2:converter-gson:2.7.1'

    implementation 'io.reactivex.rxjava2:rxandroid:2.1.1'
    implementation 'io.reactivex.rxjava2:rxjava:2.2.19'
    implementation 'io.reactivex.rxjava2:rxkotlin:2.4.0'

    implementation "androidx.lifecycle:lifecycle-common-java8:$lifecycle_version"

    def work_manager_version = "2.4.0"
    implementation "androidx.work:work-runtime:$work_manager_version"
    implementation "androidx.work:work-runtime-ktx:$work_manager_version"

    implementation 'com.jakewharton.timber:timber:4.7.1'

    def room_version = "2.2.5"
    implementation "androidx.room:room-runtime:$room_version"
    implementation "androidx.room:room-ktx:$room_version"
    kapt "androidx.room:room-compiler:$room_version"
```

### BaseViewModel 类

```kotlin
open class BaseViewModel<T> : ViewModel() {
    protected val uiState: MutableLiveData<T> = MutableLiveData()
    fun uiState() = uiState
}
```

- BaseViewModel 定义泛型，支持不同页面分别对应的不同的 UiState（定义见下面）
- 定义变量 uiState, 数据类型为 LiveData，且为 protected 修饰，即不可以在 Activity 中修改
- 定义函数 uiState(), 用来在 Activity 中观察

### UiState 类

```kotlin
sealed class UiState {
    object Loading: UiState()
    data class Success(val recentVersions: List<AndroidVersion>) : UiState()
    data class Error(val message: String): UiState()
}
```

- 定义当前页面对应的 UiState 类，同时再定义页面中需要的不用状态，这里分别是单例 Loading、成功的 Success、失败的 Error

### PerformSingleNetworkRequestViewModel 类

```kotlin
viewModelScope.launch {
    try {
        val recentAndroidVersions = mockApi.getRecentAndroidVersions()
        uiState.value = UiState.Success(recentAndroidVersions)
    } catch (exception: Exception) {
        Timber.e(exception)
        uiState.value = UiState.Error("Network Request failed!")
    }
}
```

- viewModelScope.launch {} 创建协程作用域
- 网络请求获取的结果，创建 UiState.Success(recentAndroidVersions) 或者 UiState.Error("Network Request failed!"), 并赋值给 uiState

### PerformSingleNetworkRequestActivity 类

```kotlin
private val viewModel: PerformSingleNetworkRequestViewModel by viewModels()
viewModel.uiState().observe(this, Observer { uiState ->
    render(uiState)
})
```

- 通过 activity-ktx 库中的 viewModels() 代理懒加载 viewModel 对象
- 监听 BaseViewModel 中的 uiState(), 渲染 UI

### Application 类

```kotlin
override fun onCreate() {
    super.onCreate()
    Timber.plant(Timber.DebugTree())
    // Enable Debugging for Kotlin Coroutines in debug builds
    // Prints Coroutine name when logging Thread.currentThread().name
    System.setProperty("kotlinx.coroutines.debug", if (BuildConfig.DEBUG) "on" else "off")
}
```
