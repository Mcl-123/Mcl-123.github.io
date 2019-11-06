---
title: Dagger2 超全使用攻略
date: 2019-10-09 22:20:50
categories: "Android"
tags:
     - Dagger2
---

## 在 Gradle 中的使用

```java
implementation 'com.google.dagger:dagger:2.4'
annotationProcessor 'com.google.dagger:dagger-compiler:2.4'
//java注解
implementation 'org.glassfish:javax.annotation:10.0-b28'
```

## 基本使用

```java
public class MainPresenter implements IPresenter {

    IView view;

    // 被注入对象的构造函数中添加 @Inject (方式2)
    @Inject
    public MainPresenter(IView view) {
        this.view = view;
    }
}
```

```java
// Module
@Module
public class MainModule {

    private IView view;

    public MainModule(IView view) {
        this.view = view;
    }

    // 在 Module 中提供被注入的对象(或构建该对象所需的参数) (方式1(优先))
    // 需要添加 @Provides 注解, 方法名为 provideXXX()
    @Provides
    IView provide() {
        return view;
    }
}
```

```java
// 注入器, 被注入的对象(MainPresenter)和注入的类(MainActivity)的桥梁
@Component(modules = MainModule.class)
public interface MainComponent {
    void inject(MainActivity activity);
}
```

```java
public class MainActivity extends AppCompatActivity implements IView{

    // 使用 @Inject 注入 MainPresenter 对象
    @Inject
    MainPresenter presenter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // -> Make Project 生成相应的注解类
        DaggerMainComponent.builder().mainModule(new MainModule(this)).build()
                .inject(this);

        presenter.getData();
    }
}
```

<!-- more -->

## 生成类流程图

![Dagger2 流程图](/images/dagger2.png)

简单总结:
@Inject 带有此注解的属性或构造方法将参与到依赖注入中，Dagger2会实例化有此注解的类
@Module 带有此注解的类，用来提供依赖，里面定义一些用@Provides注解的以provide开头的方法，这些方法就是所提供的依赖，Dagger2会在该类中寻找实例化某个类所需要的依赖。
@Component 用来将@Inject和@Module联系起来的桥梁，从@Module中获取依赖并将依赖注入给@Inject

## 不同注解的含义

### @Inject

@Inject用于注解构造函数或成员变量

#### 作用于成员变量时

Dagger2根据该注解及成员变量的类型，从moudle中得到相应实例。
注意：成员变量的访问修饰符不能是private

#### 作用于构造函数时

就是Dagger2对于获取对象实例的方式2，比如上面的例子其实可以直接在Presenter的构造函数上注解@Inject，并移除@Provides注解的方法

#### @Inject 不适用的情况

- 不具有构造函数的接口
- 远程引入的三方库中的类无法被添加注解
- 通过建造者模式等方式可配置化的进行构造的对象

(使用@Provides可以处理这些问题)

### @Module

@Module注解用于获取对象实例的类，Dagger2根据该注解知道应该去哪个类里获取对象实例    (上面的方式 1)

### @Provides

@Provides注解用于module类中获取对象实例的方法，Dagger2根据该注解及方法的返回值类型将对象实例注入到对应的引用中

### @Component

@Component注解用于担任连接桥梁的接口，其两端分别是在@Component的参数中指定的modules数组和在方法参数中指定的具体类
注意：方法参数必须是要注入的具体类，而非其父类或接口

注:
方式一的优先级高于方式二：

在供应某实例时，会先通过方式一查找是否存在返回该实例的的方法
如果存在，则获取实例并对外供应
如果不存在，再通过方式二查找是否存在@Inject标注的构造函数
如果存在，则将通过该构造函数构造实例并对外供应
如果不存在，那将报错，因为无法供应所需的实例

### @Singleton

使得对象成为单例只需要同时在@Provides注解的方法和component接口上添加@Singleton这个注解即可。

```java
@Singleton
@Component(modules = {AppModule.class})
public interface AppComponent {
    void inject(App app);
}
```

方式 1:

```java
@Module
public class AppModule {
    @Singleton
    @Provides
    public FactoryA providesFactoryA() {
        return new FactoryA();
    }
}
```

或者方式 2:

```java
@Singleton
public class FactoryA{
    @Inject
    public FactoryA() {
    }
}
```

源码:

```java
@Scope
@Documented
@Retention(RUNTIME)
public @interface Singleton {}
```

注意: @Singleton 并不代表单例, 真正起作用的事 @Scope 这个注解.
可以使用任意的被 @Scope 所注解的注解,来替代 @Singleton ,都可以实现单例.
@Scope 是通过 DoubleCheck 缓存获取到的对象实例实现 Component 的组件的生命周期内的对象的单例性.

### @Binds

当注入抽象类时使用 @Binds

```java
public abstract class Factory {}
public class FactoryA extends Factory {}
public class FactoryB extends Factory {}
```

```java
@Module
public abstract class AppModule {
    @Singleton
    @Binds
    public abstract Factory bindsFactory(FactoryB factoryB);

    @Singleton
    @Provides
    public static FactoryA providesFactoryA() {
        return new FactoryA();
    }

    @Singleton
    @Provides
    public static FactoryB providesFactoryB() {
        return new FactoryB();
    }
}
```

注:

1. @Binds注解用于module中的抽象方法，这个方法的参数应该是具体实现类，返回值应该是抽象类，它高速Dagger2在自动生成的代码中注入抽象类引用对象时，应该使用哪一个具体实现类作为实例被获取
2. 在使用抽象类作为module时，获取实例对象的方法只能使用static静态方法

### @Named

```java
@Binds
public abstract Factory bindsFactory(FactoryA factoryA);

@Binds
public abstract Factory bindsFactory(FactoryB factoryB);
```

以上代码, 返回多个同一类的实例时, Dagger2 无法仅仅通过返回类型来判断是调用哪个方法创建实例时,就需要使用 @Named 了

```java
@Module
public abstract class AppModule {
    @Binds
    @Named("factoryA")
    public abstract Factory bindsFactory(FactoryA factoryA);

    @Binds
    @Named("factoryB")
    public abstract Factory bindsFactory(FactoryB factoryB);
}
```

使用时：
```java
@Inject
@Named("factoryA")
Factory factory;
```

### dependencies

将App中的单例对象同样注入到MainActivity中, 需使用 dependencies

```java
@ActivityScope
@Component(modules = {MainModule.class}, dependencies = {AppComponent.class})
public interface MainComponent {
    void inject(MainActivity activity);
}
```

1. 这里依赖之后会产生一个小问题，就是存在依赖关系的组件的作用域标记不能相同，因为在AppComponent中我们标记了@Singleton，因此在这里我们自定义了新的@Scope，即@ActivityScope

```java
@Scope
@Documented
@Retention(RetentionPolicy.RUNTIME)
public @interface ActivityScope {}
```

2. 并且,在构建 MainComponent 实例时,需要传进 AppComponent 实例

```java
public class MainActivity extends AppCompatActivity {
    @Inject
    Factory mFactory;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        MainComponent mainComponent = DaggerMainComponent
                .builder()
                //这里传入appComponent实例，我们可以通过application获取到
                .appComponent(((App) getApplication()).getAppComponent())
                .build()
                .inject(this);
    }
}
```

3. 最后，我们需要给AppComponent接口添加一个方法，用来提供其所持有的单例对象

```java
@Singleton
@Component(modules = {AppModule.class})
public interface AppComponent {
    void inject(App app);

    Factory factory();
}
```

### Lazy

懒加载机制, 需要它在真正被使用时才被实例化，那么你可以使用Lazy，这里以Product为例，代码修改如下即可

```java
@Inject
Lazy<Product> mProduct;

Product product = mProduct.get();
```
