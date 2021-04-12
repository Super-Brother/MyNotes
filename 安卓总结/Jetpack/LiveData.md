# 1 LiveData介绍



## **1.1 作用**

LiveData是Jetpack AAC的重要组件，同时也有一个同名抽象类。

> LiveData 是一种可观察的数据存储器类。与常规的可观察类不同，LiveData 具有生命周期感知能力，意指它遵循其他应用组件（如 Activity/Fragment）的生命周期。这种感知能力可确保 LiveData 仅更新处于活跃生命周期状态的应用组件观察者。

拆解开来：

1，LiveData是一个**数据持有者，给源数据包装一层**。

2，源数据使用LiveData包装后，**可以被observer观察，数据有更新时observer可感知**。

3，但 observer的感知，**只发生在（Activity/Fragment）活跃生命周期状态（STARTED、RESUMED）**。

也就是说，LiveData使得 数据的更新 能以观察者模式 被observer感知，且此感知只发生在 LifecycleOwner的活跃生命周期状态。



## **1.2 特点**

使用 LiveData 具有以下优势：

1，确保界面符合数据状态，当生命周期状态变化时，LiveData通知Observer，可以在observer中更新界面。观察者可以在生命周期状态更改时刷新界面，而不是在每次数据变化时刷新界面。

2，不会发生内存泄漏，observer会在LifecycleOwner**状态变为DESTROYED后自动remove**。

3，不会因 Activity 停止而导致崩溃，如果LifecycleOwner生命周期处于非活跃状态，则它不会接收任何 LiveData事件。

4，**不需要手动解除观察**，开发者不需要在onPause或onDestroy方法中解除对LiveData的观察，因为LiveData能感知生命周期状态变化，所以会自动管理所有这些操作。

5，**数据始终保持最新状态**，数据更新时 若LifecycleOwner为非活跃状态，那么会在变为活跃时接收最新数据。例如，曾经在后台的 Activity 会在返回前台后，observer立即接收最新的数据。



# *2* LiveData的使用



## **2.1 基本使用**

gradle依赖在上一篇中已经介绍了。下面来看基本用法：

1，创建LiveData实例，指定源数据类型。

2，创建Observer实例，实现onChanged()方法，用于接收源数据变化并刷新UI。

3，LiveData实例使用observe()方法添加观察者，并传入LifecycleOwner。

4，LiveData实例使用setValue()/postValue()更新源数据 （**子线程要postValue()**）。

举个例子：

```
public class LiveDataTestActivity extends AppCompatActivity{

   private MutableLiveData<String> mLiveData;

   @Override
   protected void onCreate(Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
       setContentView(R.layout.activity_lifecycle_test);

       //liveData基本使用
       mLiveData = new MutableLiveData<>();
       mLiveData.observe(this, new Observer<String>() {
           @Override
           public void onChanged(String s) {
               Log.i(TAG, "onChanged: "+s);
           }
       });
       Log.i(TAG, "onCreate: ");
       mLiveData.setValue("onCreate");//activity是非活跃状态，不会回调onChanged。变为活跃时，value被onStart中的value覆盖
   }
   @Override
   protected void onStart() {
       super.onStart();
       Log.i(TAG, "onStart: ");
       mLiveData.setValue("onStart");//活跃状态，会回调onChanged。并且value会覆盖onCreate、onStop中设置的value
   }
   @Override
   protected void onResume() {
       super.onResume();
       Log.i(TAG, "onResume: ");
       mLiveData.setValue("onResume");//活跃状态，回调onChanged
   }
   @Override
   protected void onPause() {
       super.onPause();
       Log.i(TAG, "onPause: ");
       mLiveData.setValue("onPause");//活跃状态，回调onChanged
   }
   @Override
   protected void onStop() {
       super.onStop();
       Log.i(TAG, "onStop: ");
       mLiveData.setValue("onStop");//非活跃状态，不会回调onChanged。后面变为活跃时，value被onStart中的value覆盖
   }
   @Override
   protected void onDestroy() {
       super.onDestroy();
       Log.i(TAG, "onDestroy: ");
       mLiveData.setValue("onDestroy");//非活跃状态，且此时Observer已被移除，不会回调onChanged
   }
}
```

除了使用observe()方法添加观察者，也可以使用observeForever(Observer) 方法来注册未关联LifecycleOwner的观察者。在这种情况下，观察者会被视为始终处于活跃状态。



## **2.2 扩展使用**

扩展包括两点：

1，自定义LiveData，本身回调方法的覆写：onActive()、onInactive()。

2，**实现LiveData为单例模式，便于在多个Activity、Fragment之间共享数据**。

官方的例子如下：

```
public class StockLiveData extends LiveData<BigDecimal> {
    private static StockLiveData sInstance; //单实例
    private StockManager stockManager;

    private SimplePriceListener listener = new SimplePriceListener() {
        @Override
        public void onPriceChanged(BigDecimal price) {
            setValue(price);//监听到股价变化 使用setValue(price) 告知所有活跃观察者
        }
    };

//获取单例
    @MainThread
    public static StockLiveData get(String symbol) {
        if (sInstance == null) {
            sInstance = new StockLiveData(symbol);
        }
        return sInstance;
    }

    private StockLiveData(String symbol) {
        stockManager = new StockManager(symbol);
    }

  //活跃的观察者（LifecycleOwner）数量从 0 变为 1 时调用
    @Override
    protected void onActive() {
        stockManager.requestPriceUpdates(listener);//开始观察股价更新
    }

  //活跃的观察者（LifecycleOwner）数量从 1 变为 0 时调用。这不代表没有观察者了，可能是全都不活跃了。可以使用hasObservers()检查是否有观察者。
    @Override
    protected void onInactive() {
        stockManager.removeUpdates(listener);//移除股价更新的观察
    }
}    
```

只有当 存在活跃的观察者（LifecycleOwner）时 才会连接到 股价更新服务 监听股价变化。使用如下：

```
public class MyFragment extends Fragment {
    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        //获取StockLiveData单实例，添加观察者，更新UI
        StockLiveData.get(symbol).observe(getViewLifecycleOwner(), price -> {
            // Update the UI.
        });
    }
}
```

由于StockLiveData是单实例模式，那么多个LifycycleOwner（Activity、Fragment）间就可以共享数据了。



## **2.3 高级用法**

如果希望在将 LiveData 对象分派给观察者之前对存储在其中的值进行更改，或者需要根据另一个实例的值返回不同的 LiveData 实例，可以使用LiveData中提供的Transformations类。

### **2.3.1 数据修改 - Transformations.map**

```
//Integer类型的liveData1
MutableLiveData<Integer> liveData1 = new MutableLiveData<>();
//转换成String类型的liveDataMap
LiveData<String> liveDataMap = Transformations.map(liveData1, new Function<Integer, String>() {
    @Override
    public String apply(Integer input) {
        String s = input + " + Transformations.map";
        Log.i(TAG, "apply: " + s);
        return s;
    }
});
liveDataMap.observe(this, new Observer<String>() {
    @Override
    public void onChanged(String s) {
        Log.i(TAG, "onChanged1: "+s);
    }
});

liveData1.setValue(100);
```

使用很简单：原本的liveData1 没有添加观察者，而是使用Transformations.map()方法 对liveData1的数据进行的修改 生成了新的liveDataMap，liveDataMap添加观察者，最后liveData1设置数据 。

### **2.3.2 数据切换 - Transformations.switchMap**

如果想要根据某个值 切换观察不同LiveData数据，则可以使用Transformations.switchMap()方法。

```
//两个liveData，由liveDataSwitch决定 返回哪个livaData数据
MutableLiveData<String> liveData3 = new MutableLiveData<>();
MutableLiveData<String> liveData4 = new MutableLiveData<>();

//切换条件LiveData，liveDataSwitch的value 是切换条件
MutableLiveData<Boolean> liveDataSwitch = new MutableLiveData<>();

//liveDataSwitchMap由switchMap()方法生成，用于添加观察者
LiveData<String> liveDataSwitchMap = Transformations.switchMap(liveDataSwitch, new Function<Boolean, LiveData<String>>() {
    @Override
    public LiveData<String> apply(Boolean input) {
    //这里是具体切换逻辑：根据liveDataSwitch的value返回哪个liveData
        if (input) {
            return liveData3;
        }
        return liveData4;
    }
});

liveDataSwitchMap.observe(this, new Observer<String>() {
    @Override
    public void onChanged(String s) {
        Log.i(TAG, "onChanged2: " + s);
    }
});

boolean switchValue = true;
liveDataSwitch.setValue(switchValue);//设置切换条件值

liveData3.setValue("liveData3");
liveData4.setValue("liveData4");
```

### **2.3.3 观察多个数据 - MediatorLiveData**

MediatorLiveData 是 LiveData 的子类，允许合并多个 LiveData 源。**只要任何原始的 LiveData 源对象发生更改，就会触发MediatorLiveData 对象的观察者。**

```
MediatorLiveData<String> mediatorLiveData = new MediatorLiveData<>();

MutableLiveData<String> liveData5 = new MutableLiveData<>();
MutableLiveData<String> liveData6 = new MutableLiveData<>();

//添加 源 LiveData
mediatorLiveData.addSource(liveData5, new Observer<String>() {
    @Override
    public void onChanged(String s) {
        Log.i(TAG, "onChanged3: " + s);
        mediatorLiveData.setValue(s);
    }
});
//添加 源 LiveData
mediatorLiveData.addSource(liveData6, new Observer<String>() {
    @Override
    public void onChanged(String s) {
        Log.i(TAG, "onChanged4: " + s);
        mediatorLiveData.setValue(s);
    }
});

//添加观察
mediatorLiveData.observe(this, new Observer<String>() {
    @Override
    public void onChanged(String s) {
        Log.i(TAG, "onChanged5: "+s);
        //无论liveData5、liveData6更新，都可以接收到
    }
});

liveData5.setValue("liveData5");
//liveData6.setValue("liveData6");
```

（**Transformations也是对MediatorLiveData的使用**。）



# *3* 源码分析



## **3.1 添加观察者**

LiveData原理是观察者模式，下面就先从LiveData.observe()方法看起：

```
/**
 * 添加观察者. 事件在主线程分发. 如果LiveData已经有数据，将直接分发给observer。
 * 观察者只在LifecycleOwner活跃时接受事件，如果变为DESTROYED状态，observer自动移除。
 * 当数据在非活跃时更新，observer不会接收到。变为活跃时 将自动接收前面最新的数据。 
 * LifecycleOwner非DESTROYED状态时，LiveData持有observer和 owner的强引用，DESTROYED状态时自动移除引用。
 * @param owner    控制observer的LifecycleOwner
 * @param observer 接收事件的observer
 */
@MainThread
public void observe(@NonNull LifecycleOwner owner, @NonNull Observer<? super T> observer) {
    assertMainThread("observe");
    if (owner.getLifecycle().getCurrentState() == DESTROYED) {
        // LifecycleOwner是DESTROYED状态，直接忽略
        return;
    }
    //使用LifecycleOwner、observer 组装成LifecycleBoundObserver，添加到mObservers中
    LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer);
    ObserverWrapper existing = mObservers中.putIfAbsent(observer, wrapper);
    if (existing != null && !existing.isAttachedTo(owner)) {
    //!existing.isAttachedTo(owner)说明已经添加到mObservers中的observer指定的owner不是传进来的owner，就会报错“不能添加同一个observer却不同LifecycleOwner”
        throw new IllegalArgumentException("Cannot add the same observer"
                + " with different lifecycles");
    }
    if (existing != null) {
        return;//这里说明已经添加到mObservers中,且owner就是传进来的owner
    }
    owner.getLifecycle().addObserver(wrapper);
}
```

另外，再看observeForever方法：

```
@MainThread
public void observeForever(@NonNull Observer<? super T> observer) {
    assertMainThread("observeForever");
    AlwaysActiveObserver wrapper = new AlwaysActiveObserver(observer);
    ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);
    if (existing instanceof LiveData.LifecycleBoundObserver) {
        throw new IllegalArgumentException("Cannot add the same observer"
                + " with different lifecycles");
    }
    if (existing != null) {
        return;
    }
    wrapper.activeStateChanged(true);
}
```

和observe()类似，只不过 会认为**观察者一直是活跃状态**，且不会自动移除观察者。



## **3.2 事件回调**

LiveData添加了观察者LifecycleBoundObserver，接着看如何进行回调的：

```
class LifecycleBoundObserver extends ObserverWrapper implements LifecycleEventObserver {
    @NonNull
    final LifecycleOwner mOwner;

    LifecycleBoundObserver(@NonNull LifecycleOwner owner, Observer<? super T> observer) {
        super(observer);
        mOwner = owner;
    }

    @Override
    boolean shouldBeActive() { //至少是STARTED状态
        return mOwner.getLifecycle().getCurrentState().isAtLeast(STARTED);
    }

    @Override
    public void onStateChanged(@NonNull LifecycleOwner source,
            @NonNull Lifecycle.Event event) {
        if (mOwner.getLifecycle().getCurrentState() == DESTROYED) {
            removeObserver(mObserver);//LifecycleOwner变成DESTROYED状态，则移除观察者
            return;
        }
        activeStateChanged(shouldBeActive());
    }

    @Override
    boolean isAttachedTo(LifecycleOwner owner) {
        return mOwner == owner;
    }

    @Override
    void detachObserver() {
        mOwner.getLifecycle().removeObserver(this);
    }
}
```



## **3.3 数据更新**

LivaData数据更新可以使用setValue(value)、postValue(value)，区别在于postValue(value)用于 子线程:

```
//LivaData.java
    private final Runnable mPostValueRunnable = new Runnable() {
        @SuppressWarnings("unchecked")
        @Override
        public void run() {
            Object newValue;
            synchronized (mDataLock) {
                newValue = mPendingData;
                mPendingData = NOT_SET;
            }
            setValue((T) newValue); //也是走到setValue方法
        }
    };

    protected void postValue(T value) {
        boolean postTask;
        synchronized (mDataLock) {
            postTask = mPendingData == NOT_SET;
            mPendingData = value;
        }
        if (!postTask) {
            return;
        }
        ArchTaskExecutor.getInstance().postToMainThread(mPostValueRunnable);//抛到主线程
    }
```

postValue方法把Runable对象mPostValueRunnable抛到主线程，其run方法中还是使用的setValue()，继续看：

```
@MainThread
protected void setValue(T value) {
    assertMainThread("setValue");
    mVersion++;
    mData = value;
    dispatchingValue(null);
}
```



## **3.4 Transformations原理**

