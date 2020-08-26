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

implementation fileTree(dir: "libs", include: ["*.jar"])
implementation 'androidx.appcompat:appcompat:1.2.0'
implementation 'androidx.constraintlayout:constraintlayout:2.0.0'

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

implementation 'com.squareup.retrofit2:retrofit:2.9.0'
implementation 'com.squareup.retrofit2:adapter-rxjava2:2.7.2'
implementation 'com.google.code.gson:gson:2.8.6'
implementation 'com.squareup.retrofit2:converter-gson:2.9.0'

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

### 同时执行多个网络请求

```kotlin
val oreoFeaturesDeferred = viewModelScope.async { mockApi.getAndroidVersionFeatures(27) }
val pieFeaturesDeferred = viewModelScope.async { mockApi.getAndroidVersionFeatures(28) }
val android10FeaturesDeferred = viewModelScope.async { mockApi.getAndroidVersionFeatures(29) }
viewModelScope.launch {
    try {
        val versionFeatures =
            awaitAll(oreoFeaturesDeferred, pieFeaturesDeferred, android10FeaturesDeferred)
        uiState.value = UiState.Success(versionFeatures)
    } catch (exception: Exception) {
        uiState.value = UiState.Error("Network Request failed")
    }
}

/*

Alternatively:

viewModelScope.launch {
    try {
        // we need to wrap this code with a coroutineScope block
        // otherwise the app would crash on unsuccessful network requests
        coroutineScope {
            val oreoFeaturesDeferred = async { mockApi.getAndroidVersionFeatures(27) }
            val pieFeaturesDeferred = async { mockApi.getAndroidVersionFeatures(28) }
            val android10FeaturesDeferred = async { mockApi.getAndroidVersionFeatures(29) }

            val oreoFeatures = oreoFeaturesDeferred.await()
            val pieFeatures = pieFeaturesDeferred.await()
            val android10Features = android10FeaturesDeferred.await()

            val versionFeatures = listOf(oreoFeatures, pieFeatures, android10Features)

            // other alternative: (but slightly different behavior when a deferred fails, see docs)
            // val versionFeatures = awaitAll(oreoFeaturesDeferred, pieFeaturesDeferred, android10FeaturesDeferred)

            uiState.value = UiState.Success(versionFeatures)
        }

    } catch (exception: Exception) {
        uiState.value = UiState.Error("Network Request failed")
    }
}*/
```

通过 awaitAll() 同时执行多个请求

### 给请求设置超时时间

```kotlin
try {
    val recentVersions = withTimeout(timeout) {
        api.getRecentAndroidVersions()
    }
    uiState.value = UiState.Success(recentVersions)
} catch (timeoutCancellationException: TimeoutCancellationException) {
    uiState.value = UiState.Error("Network Request timed out!")
} catch (exception: Exception) {
    uiState.value = UiState.Error("Network Request failed!")
}

or:

try {
    val recentVersions = withTimeoutOrNull(timeout) {
        api.getRecentAndroidVersions()
    }
    if (recentVersions != null) {
        uiState.value = UiState.Success(recentVersions)
    } else {
        uiState.value = UiState.Error("Network Request timed out!")
    }
} catch (exception: Exception) {
    uiState.value = UiState.Error("Network Request failed!")
}
```

通过 withTimeout(timeout) 或者 withTimeoutOrNull(timeout), 设置超时时间

### 重试网络请求

```kotlin
// retry with exponential backoff
// inspired by https://stackoverflow.com/questions/46872242/how-to-exponential-backoff-retry-on-kotlin-coroutines
private suspend fun <T> retry(
    times: Int,
    initialDelayMillis: Long = 100,
    maxDelayMillis: Long = 1000,
    factor: Double = 2.0,
    block: suspend () -> T
): T {
    var currentDelay = initialDelayMillis
    repeat(times) {
        try {
            return block()
        } catch (exception: Exception) {
            Timber.e(exception)
        }
        delay(currentDelay)
        currentDelay = (currentDelay * factor).toLong().coerceAtMost(maxDelayMillis)
    }
    return block() // last attempt
}

viewModelScope.launch {
    val numberOfRetries = 2
    try {
        retry(times = numberOfRetries) {
            val recentVersions = api.getRecentAndroidVersions()
            uiState.value = UiState.Success(recentVersions)
        }
    } catch (e: Exception) {
        uiState.value = UiState.Error("Network Request failed")
    }
}
```

### 执行可变数量的网络请求

```kotlin
val versionFeatures = recentVersions.map { androidVersion ->
                        async { mockApi.getAndroidVersionFeature(androidVersion.apiLevel) }
                    }.awaitAll()
```

### Room + 协程

#### Entity

```kotlin
@Entity(tableName = "androidversions")
data class AndroidVersionEntity(@PrimaryKey val apiLevel: Int, val name: String)
```

#### Dao

```kotlin
@Dao
interface AndroidVersionDao {

    @Query("SELECT * FROM androidversions")
    suspend fun getAndroidVersions(): List<AndroidVersionEntity>

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(androidVersionEntity: AndroidVersionEntity)

    @Query("DELETE FROM androidversions")
    suspend fun clear()
}
```

#### Database (单例)

```kotlin
@Database(entities = [AndroidVersionEntity::class], version = 1, exportSchema = false)
abstract class AndroidVersionDatabase : RoomDatabase() {

    abstract fun androidVersionDao(): AndroidVersionDao

    companion object {
        private var INSTANCE: AndroidVersionDatabase? = null

        fun getInstance(context: Context): AndroidVersionDatabase {
            if (INSTANCE == null) {
                synchronized(AndroidVersionDatabase::class) {
                    INSTANCE = buildRoomDb(context)
                }
            }
            return INSTANCE!!
        }

        private fun buildRoomDb(context: Context) =
            Room.databaseBuilder(
                context.applicationContext,
                AndroidVersionDatabase::class.java,
                "androidversions.db"
            ).build()
    }
}
```

#### ViewModelFactory (想在 ViewModel 中传入 database)

```kotlin
class ViewModelFactory(private val api: MockApi, private val database: AndroidVersionDao) :
    ViewModelProvider.Factory {

    override fun <T : ViewModel?> create(modelClass: Class<T>): T {
        return modelClass.getConstructor(MockApi::class.java, AndroidVersionDao::class.java)
            .newInstance(api, database)
    }
}
```

#### ViewModel

```kotlin
class RoomAndCoroutinesViewModel(
    private val api: MockApi,
    private val database: AndroidVersionDao
) : BaseViewModel<UiState>() {}
```

#### RoomAndCoroutinesActivity

```kotlin
private val viewModel: RoomAndCoroutinesViewModel by viewModels {
    ViewModelFactory(
        mockApi(),
        AndroidVersionDatabase.getInstance(applicationContext).androidVersionDao()
    )
}
```

### 协程的调试

#### Application

```kotlin
// Enable Debugging for Kotlin Coroutines in debug builds
// Prints Coroutine name when logging Thread.currentThread().name
System.setProperty("kotlinx.coroutines.debug", if (BuildConfig.DEBUG) "on" else "off")
```

#### Extensions

```kotlin
fun addCoroutineDebugInfo(message: String) = "[${Thread.currentThread().name}] $message"
```

### 使计算协程代码可取消

如果协程正在执行计算任务，并且没有检查取消的话，那么它是不能被取消的。
但是有两种方法来使执行计算的代码可以被取消。第一种方法是定期调用挂起函数来检查取消， 对于这种目的 yield 是一个好的选择。 另一种方法是显式的检查取消状态。

```kotlin
// factorial of n (n!) = 1 * 2 * 3 * 4 * ... * n
private suspend fun calculateFactorialOf(number: Int): BigInteger =
    withContext(defaultDispatcher) {
        var factorial = BigInteger.ONE
        for (i in 1..number) {

            // yield enables cooperative cancellations
            // alternatives:
            // - ensureActive()
            // - isActive() - possible to do clean up tasks with
            yield()

            factorial = factorial.multiply(BigInteger.valueOf(i.toLong()))
        }
        factorial
    }

calculationJob = viewModelScope.launch {
        try {
            var result: BigInteger = BigInteger.ZERO
            val computationDuration = measureTimeMillis {
                result = calculateFactorialOf(factorialOf)
            }
            var resultString = ""
            val stringConversionDuration = measureTimeMillis {
                resultString = convertToString(result)
            }
            uiState.value =
                UiState.Success(resultString, computationDuration, stringConversionDuration)
        } catch (exception: Exception) {
            uiState.value = if (exception is CancellationException) {
                UiState.Error("Calculation was cancelled")
            } else {
                UiState.Error("Error while calculating result")
            }
        }
    }

calculationJob?.cancel()
```

### 异常处理

#### 通过 try/catch 处理异常

```kotlin
val exceptionHandler = CoroutineExceptionHandler { _, exception ->
    uiState.value = UiState.Error("Network Request failed!! $exception")
}

uiState.value = UiState.Loading
viewModelScope.launch(exceptionHandler) {
    api.getAndroidVersionFeatures(27)
}
```

#### 通过 CoroutineExceptionHandler 处理异常

```kotlin
val exceptionHandler = CoroutineExceptionHandler { _, exception ->
    uiState.value = UiState.Error("Network Request failed!! $exception")
}

uiState.value = UiState.Loading
viewModelScope.launch(exceptionHandler) {
    api.getAndroidVersionFeatures(27)
}
```

#### 使用 supervisorScope, 在存在失败的协程的情况下不会取消其同级协程的情况

```kotlin
viewModelScope.launch {
    supervisorScope {
        val oreoFeaturesDeferred = async { api.getAndroidVersionFeatures(27) }
        val pieFeaturesDeferred = async { api.getAndroidVersionFeatures(28) }

        val oreoFeatures = try {
            oreoFeaturesDeferred.await()
        } catch (e: Exception) {
            Timber.e("Error loading oreo features")
            null
        }

        val pieFeatures = try {
            pieFeaturesDeferred.await()
        } catch (e: Exception) {
            Timber.e("Error loading pie features")
            null
        }

        val versionFeatures = listOfNotNull(oreoFeatures, pieFeatures)
        uiState.value = UiState.Success(versionFeatures)
    }
}
```

#### 使用 runCatching, 在存在失败的协程的情况下不会取消其同级协程的情况

```kotlin
viewModelScope.launch {
    supervisorScope {
        val oreoFeaturesDeferred = async { api.getAndroidVersionFeatures(27) }
        val pieFeaturesDeferred = async { api.getAndroidVersionFeatures(28) }

        val versionFeatures = listOf(
            oreoFeaturesDeferred,
            pieFeaturesDeferred
        ).mapNotNull { deferred ->
            runCatching {
                deferred.await()
            }.onFailure {
                Timber.e("Failure loading features of an Android Version")
            }.getOrNull()
        }

        uiState.value = UiState.Success(versionFeatures)
    }
}
```
