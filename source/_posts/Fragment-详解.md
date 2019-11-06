---
title: Fragment 详解
date: 2019-02-26 20:35:42
categories: "Android"
tags:
     - Fragment
     - DialogFragment
---

# Fragment 详解

## 前言

Fragment(碎片)功能，它非常类似于Activity，可以像Activity一样包含布局，Fragment通常是嵌套在Activity中使用的。
Activity一方面需要在布局中为Fragment安排位置，另一方面需要管理好Fragment的生命周期。Activity中有个FragmentManager，其内部维护fragment队列，以及fragment事务的回退栈。

<!-- more -->

![fragment_activity嵌套](/images/fragment_activity.jpg)

## Fragment的生命周期

Fragment必须是依存与Activity而存在的，因此Activity的生命周期会直接影响到Fragment的生命周期。

![fragment生命周期](/images/fragment_lifecycle.jpg)

## Fragment的静态使用

把Fragment当成普通的控件，直接写在Activity的布局文件中。

1. 继承Fragment，重写onCreateView决定Fragemnt的布局

```java
public class TitleFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
        Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_title, container, false);
        return view;
    }
}
```

2. 在Activity 布局文件中中声明此Fragment，就当和普通的View一样

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <fragment
        android:id="@+id/id_fragment_title"
        android:name="com.test.TitleFragment"
        android:layout_width="fill_parent"
        android:layout_height="45dp" />
</RelativeLayout>
```

## Fragment的动态使用

1. 继承Fragment，重写onCreateView决定Fragemnt的布局

同静态使用步骤1.

2. 在Actvity的布局文件，中间使用一个FrameLayout

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <FrameLayout
        android:id="@+id/id_content"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:layout_above="@id/id_ly_bottombar"
        android:layout_below="@id/id_fragment_title" />
</RelativeLayout>
```

3. 在 Activity 的类文件中，动态的添加、更新、以及删除Fragment

```java
private void setDefaultFragment() {
    FragmentManager fm = getSupportFragmentManager();
    FragmentTransaction transaction = fm.beginTransaction();
    titleFragment = new TitleFragment();
    transaction.replace(R.id.id_content, titleFragment);
    transaction.commit();
}
```

## Fragment 常用 API

### Fragment 常用的三个类

Fragment 主要用于定义Fragment
FragmentManager 主要用于在Activity中操作Fragment
FragmentTransaction 保证一些列Fragment操作的原子性

### 获取FragmentManager的方式

```java
getSupportFragmentManager()
```

### 主要的操作都是FragmentTransaction的方法

```java
FragmentTransaction transaction = fm.benginTransatcion();//开启一个事务
transaction.add()
transaction.remove()
transaction.replace()
transaction.hide()
transaction.show()
detach()
attach()
```

1. 我在FragmentA中的EditText填了一些数据，当切换到FragmentB时，如果希望会到A还能看到数据，则适合你的就是hide和show；也就是说，希望保留用户操作的面板，你可以使用hide和show，当然了不要使劲在那new实例，进行下非null判断。

2. 再比如：我不希望保留用户操作，你可以使用remove()，然后add()；或者使用replace()这个和remove,add是相同的效果。

3. remove和detach有一点细微的区别，在不考虑回退栈的情况下，remove会销毁整个Fragment实例，而detach则只是销毁其视图结构，实例并不会被销毁。那么二者怎么取舍使用呢？如果你的当前Activity一直存在，那么在不希望保留用户操作的时候，你可以优先使用detach。

## 管理Fragment回退栈

可以通过Activity维护一个回退栈来保存每次Fragment事务发生的变化。如果你将Fragment任务添加到回退栈，当用户点击后退按钮时，将看到上一次的保存的Fragment。一旦Fragment完全从后退栈中弹出，用户再次点击后退键，则退出当前Activity。

添加一个Fragment事务到回退栈：

```java
FragmentTransaction.addToBackStack(String)
```

1. Activity 添加默认的Fragment

这里不调用 addToBackStack，是考虑到点击back时，不想显示白班

```java
FragmentManager fm = getSupportFragmentManager();
FragmentTransaction tx = fm.beginTransaction();
tx.add(R.id.id_content, new FragmentOne(),"ONE");
tx.commit();
```

2. FragmentOne 中添加点击跳转到 FragmentTwo

```java
public class FragmentOne extends Fragment implements OnClickListener {
	private Button mBtn;
	@Override
	public View onCreateView(LayoutInflater inflater, ViewGroup container,
			Bundle savedInstanceState) {
		View view = inflater.inflate(R.layout.fragment_one, container, false);
		mBtn = (Button) view.findViewById(R.id.id_fragment_one_btn);
		mBtn.setOnClickListener(this);
		return view;
	}
 
	@Override
	public void onClick(View v) {
		FragmentTwo fTwo = new FragmentTwo();
		FragmentManager fm = getSupportFragmentManager();
		FragmentTransaction tx = fm.beginTransaction();
		tx.replace(R.id.id_content, fTwo, "TWO");
		tx.addToBackStack(null); 
		tx.commit();
	}
}
```

replace是remove和add的合体，并且如果不添加事务到回退栈，前一个Fragment实例会被销毁。这里很明显，我们调用tx.addToBackStack(null);将当前的事务添加到了回退栈，所以FragmentOne实例不会被销毁，但是视图层次依然会被销毁，即会调用onDestoryView和onCreateView

## Fragment与Activity通信

1. 如果你Activity中包含自己管理的Fragment的引用，可以通过引用直接访问所有的Fragment的public方法

2. 如果Activity中未保存任何Fragment的引用，那么没关系，每个Fragment都有一个唯一的TAG或者ID,可以通过getSupportFragmentManager.findFragmentByTag()或者findFragmentById()获得任何Fragment实例，然后进行操作。

3. 在Fragment中可以通过getActivity得到当前绑定的Activity的实例，然后进行操作。

## Fragment与Activity通信的最佳实践

要考虑Fragment的重复使用，所以必须降低Fragment与Activity的耦合，而且Fragment更不应该直接操作别的Fragment，毕竟Fragment操作应该由它的管理者Activity来决定。

1. FragmentOne不和任何Activity耦合，任何Activity都可以使用；并且我们声明了一个接口，来回调其点击事件，想要管理其点击事件的Activity实现此接口就即可。可以看到我们在onClick中首先判断了当前绑定的Activity是否实现了该接口，如果实现了则调用。

1. FragmentOne 中定义接口，来回调点击事件

```java

private FTwoBtnClickListener fTwoBtnClickListener;

public interface FTwoBtnClickListener {
    void onFTwoBtnClick();
}

public void setFTwoBtnClickListener(FTwoBtnClickListener fTwoBtnClickListener) {
    this.fTwoBtnClickListener = fTwoBtnClickListener;
}

mBtn.setOnClickListener(v -> {
    if(fTwoBtnClickListener != null) {
        fTwoBtnClickListener.onFTwoBtnClick();
    }
})
```

2. Activity 中 设置接口

```java
fragmentOne.setFTwoBtnClickListener(new FTwoBtnClickListener() {
    @Override
    public void onFOneBtnClick() {
        if (mFTwo == null) {
            mFTwo = new FragmentTwo();
        }

        FragmentManager fm = getSupportFragmentManager();
        FragmentTransaction tx = fm.beginTransaction();
        tx.replace(R.id.id_content, mFTwo, "TWO");
        tx.addToBackStack(null);
        tx.commit();
    }
})
```

## 处理运行时配置发生变化

运行时配置发生变化，最常见的就是屏幕发生旋转（或者app放后台过长时间），如果你不知道如何处理屏幕变化可以参考：

[Android 屏幕旋转 处理 AsyncTask 和 ProgressDialog 的最佳方案](https://blog.csdn.net/lmj623565791/article/details/37936275)

1. 原本的Activity:

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    requestWindowFeature(Window.FEATURE_NO_TITLE);
    setContentView(R.layout.activity_main);

    mFOne = new FragmentOne();
    FragmentManager fm = getSupportFragmentManager();
    FragmentTransaction tx = fm.beginTransaction();
    tx.add(R.id.id_content, mFOne, "ONE");
    tx.commit();
}
```

每次屏幕旋转时，都会重新创建 FragmentOne, 并叠加显示，显然有问题。

2. 修改后的Activity

2.1 可以这样

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    requestWindowFeature(Window.FEATURE_NO_TITLE);
    setContentView(R.layout.activity_main);
    Log.e(TAG, savedInstanceState+"");
    if(savedInstanceState == null) {
        mFOne = new FragmentOne();
        FragmentManager fm = getSupportFragmentManager();
        FragmentTransaction tx = fm.beginTransaction();
        tx.add(R.id.id_content, mFOne, "ONE");
        tx.commit();
    }
}
```

其实通过检查onCreate的参数Bundle savedInstanceState就可以判断，当前是否发生Activity的重新创建, 默认的savedInstanceState会存储一些数据，包括Fragment的实例。
所以，只有在savedInstanceState==null时，才进行创建Fragment实例。

2.2 或者可以这样

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    FragmentManager fm = getSupportFragmentManager();
    mContentFragment = (ContentFragment) fm.findFragmentById(R.id.id_fragment_container);
    if(mContentFragment == null ) {
        mContentFragment = new ContentFragment();
        fm.beginTransaction().add(R.id.id_fragment_container,mContentFragment).commit();
    }
}
```

当Activity因为配置发生改变（屏幕旋转）或者内存不足被系统杀死，造成重新创建时，我们的fragment会被保存下来，但是会创建新的FragmentManager，新的FragmentManager会首先会去获取保存下来的fragment队列，重建fragment队列，从而恢复之前的状态。

3. 重新绘制时，Fragment也发生重建

和Activity类似，Fragment也有onSaveInstanceState的方法，在此方法中进行保存数据，然后在onCreate或者onCreateView或者onActivityCreated进行恢复都可以。

## 没有布局的Fragment的作用

没有布局文件Fragment实际上是为了保存，当Activity重启时，保存大量数据准备的。

1. 继承Fragment，声明需保存状态的对象

2. 在 Fragment 创建时，调用 setRetainInstance(true);

3. 在 Activity 中创建 Fragment 实例

4. 当Activity重新启动后，使用FragmentManager对Fragment进行恢复, 并取出数据

Fragment.java:

```java
public class RetainedFragment extends Fragment {
    // 1
    private Bitmap data;
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // 2
        setRetainInstance(true);
    }
    public void setData(Bitmap data) { this.data = data; }
    public Bitmap getData() { return data; }
}
```

Activity.java:

```java
// 3
private RetainedFragment dataFragment;

public void onCreate(Bundle savedInstanceState) {
    // 3
    FragmentManager fm = getSupportFragmentManager();
    dataFragment = (RetainedFragment) fm.findFragmentByTag("data");
    if (dataFragment == null) {
        dataFragment = new RetainedFragment();
        fm.beginTransaction().add(dataFragment, "data").commit();
    }
    data = dataFragment.getData();
    if (data == null) { initData(); }
}

@Override
public void onDestroy() {
    super.onDestroy();
    dataFragment.setData(mBitmap);
}
```

## 使用Fragment创建对话框

[DialogFragmentDemo](https://github.com/Mcl-123/DialogFragmentDemo)

## Fragment Arguments

我们某个按钮触发Activity跳转，需要通过Intent传递参数到目标Activity的Fragment中，那么此Fragment如何获取当前的Intent的值？

Fragment 中：

```java
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    mArgument = getActivity().getIntent().getStringExtra(ARGUMENT);
}
```

存在一个大问题：我们用Fragment的一个很大的原因，就是为了复用。你这么写，相当于这个Fragment已经完全和当前这个宿主Activity绑定了，复用直接废了，所以改用下面的方法：

Fragment 中：

```java
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    Bundle bundle = getArguments();
    if (bundle != null) { mArgument = bundle.getString(ARGUMENT); }
}

public static ContentFragment newInstance(String argument)
{
    Bundle bundle = new Bundle();
    bundle.putString(ARGUMENT, argument);
    ContentFragment contentFragment = new ContentFragment();
    contentFragment.setArguments(bundle);
    return contentFragment;
}
```

注：setArguments方法必须在fragment创建以后，添加给Activity前完成。千万不要，首先调用了add，然后设置arguments

## Fragment的startActivityForResult

在Fragment中存在startActivityForResult（）以及onActivityResult（）方法，但是呢，没有setResult（）方法，用于设置返回的intent，这样我们就需要通过调用getActivity().setResult(ListTitleFragment.REQUEST_DETAIL, intent);

## SingleFragmentActivity

抽象BaseFragmentActivigty类： SingleFragmentActivity.java:

```java
public abstract class SingleFragmentActivity extends FragmentActivity {
	protected abstract Fragment createFragment();
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_single_fragment);
	
		FragmentManager fm = getSupportFragmentManager();
		Fragment fragment =fm.findFragmentById(R.id.id_fragment_container);
		
		if(fragment == null )
		{
			fragment = createFragment() ;
			
			fm.beginTransaction().add(R.id.id_fragment_container,fragment).commit();
		}
	}	
}
```

activity_single_fragment.xml:

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
	android:id="@+id/id_fragment_container"
>
</RelativeLayout>
```

ContentActivity.java:

```java

public class ContentActivity extends SingleFragmentActivity {
	private ContentFragment mContentFragment;
 
	@Override
	protected Fragment createFragment()
	{
		String title = getIntent().getStringExtra(ContentFragment.ARGUMENT);
 
		mContentFragment = ContentFragment.newInstance(title);
		return mContentFragment;
	}
}
```

ContentFragment.java

```java
public class ContentFragment extends Fragment
{
 
	private String mArgument;
	public static final String ARGUMENT = "argument";
	public static final String RESPONSE = "response";
 
	@Override
	public void onCreate(Bundle savedInstanceState)
	{
		super.onCreate(savedInstanceState);
		Bundle bundle = getArguments();
		if (bundle != null)
		{
			mArgument = bundle.getString(ARGUMENT);
			Intent intent = new Intent();
			intent.putExtra(RESPONSE, "good");
			getActivity().setResult(ListTitleFragment.REQUEST_DETAIL, intent);
		}
 
	}
 
	public static ContentFragment newInstance(String argument)
	{
		Bundle bundle = new Bundle();
		bundle.putString(ARGUMENT, argument);
		ContentFragment contentFragment = new ContentFragment();
		contentFragment.setArguments(bundle);
		return contentFragment;
	}
 
	@Override
	public View onCreateView(LayoutInflater inflater, ViewGroup container,
			Bundle savedInstanceState)
	{
		Random random = new Random();
		TextView tv = new TextView(getActivity());
		tv.setText(mArgument);
		tv.setGravity(Gravity.CENTER);
		tv.setBackgroundColor(Color.argb(random.nextInt(100),
				random.nextInt(255), random.nextInt(255), random.nextInt(255)));
		return tv;
	}
}
```

## FragmentPagerAdapter与FragmentStatePagerAdapter

FragmentPagerAdapter：对于不再需要的fragment，选择调用detach方法，仅销毁视图，并不会销毁fragment实例。

FragmentStatePagerAdapter：会销毁不再需要的fragment，当当前事务提交以后，会彻底的将fragmeng从当前Activity的FragmentManager中移除，state标明，销毁时，会将其onSaveInstanceState(Bundle outState)中的bundle信息保存下来，当用户切换回来，可以通过该bundle恢复生成新的fragment，也就是说，你可以在onSaveInstanceState(Bundle outState)方法中保存一些数据，在onCreate中进行恢复创建。

## Fragment间的数据传递

情况：两个Fragment在同一个Activity中：例如，点击当前Fragment中按钮，弹出一个对话框（DialogFragment），在对话框中的操作需要返回给触发的Fragment中

ContentFragment.java

```java
// 打开 DialogFragment
tv.setOnClickListener(new OnClickListener() {
    @Override
    public void onClick(View v)
    {
        EvaluateDialog dialog = new EvaluateDialog();
        //注意setTargetFragment
        dialog.setTargetFragment(ContentFragment.this, REQUEST_EVALUATE);
        dialog.show(getSupportFragmentManager(), EVALUATE_DIALOG);
    }
});

//接收返回回来的数据
@Override
public void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    if (requestCode == REQUEST_EVALUATE)
    {
        String evaluate = data
                .getStringExtra(EvaluateDialog.RESPONSE_EVALUATE);
        Toast.makeText(getActivity(), evaluate, Toast.LENGTH_SHORT).show();
        Intent intent = new Intent();
        intent.putExtra(RESPONSE, evaluate);
        getActivity().setResult(Activity.REQUEST_OK, intent);
    }
}
```

EvaluateDialog.java:

```java
// 设置返回数据
protected void setResult(int which) {
    // 判断是否设置了targetFragment
    if (getTargetFragment() == null)
        return;

    Intent intent = new Intent();
    intent.putExtra(RESPONSE_EVALUATE, mEvaluteVals[which]);
    getTargetFragment().onActivityResult(ContentFragment.REQUEST_EVALUATE,
            Activity.RESULT_OK, intent);

}
```
