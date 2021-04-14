## Google官方提供的分页加载解决方案

##### **什么是Paging**

Paging组件是Google新推出的分页组件，可以轻松帮助开发者实现RecyclerView中分页预加载以达到无限滑动的效果



##### **如何引入Paging**

```
implementation 'android.arch.paging:runtime:1.0.0'
```



##### **Paging工作原理图示**

下图是官方提供的原理图

- DataSource:数据源提供者，数据的改变会驱动列表的更新，因此，数据源是很重要的。
- PageList:核心类，它从数据源取出数据，同时，它负责页面初始化数据+分页数据什么时候加载，以何种方式加载。
- PagedListAdapter:列表适配器，通过DiffUtil差分异定向更新列表数据。

> 一共3种DataSource可选，取决于你的数据是以何种方式分页加载
> ItemKeyedDataSource：基于cursor实现，数据容量可动态自增
> PageKeyedDataSource：基于页码实现，数据容量可动态自增
> PositionalDataSource：数据容量固定，基于index加载特定范围的数据

![img](https://mmbiz.qpic.cn/mmbiz_png/v1LbPPWiaSt63y6EnvBawfqxA8cDVsBFZnGpialibmTE0ic4QHFvHQfF9GycGJnEYibm9IzZxfkic0nPb3yvXwbaSweQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



##### **Paging工作原理代码分析**

**1.** Paging的设计和传统的分页加载不大一样，它巧妙的融合LiveData的能力
首先看下如何使用Paging触发页面初始化数据加载的。

```
//1.构建PagedList.Config对象,用以声明以何种方式分页
PagedList.Config config = new PagedList.Config.Builder()
                .setPageSize(10)指定每次分页加载的条目数量
                .setInitialLoadSizeHint(12)指定初始化数据加载的条目数量
                //.setPrefetchDistance(10)指定提前分页预加载的条目数量，默认和pageSize相等
                //setMaxSize(100) 指定数据源最大可加载的条目数量
                //setEnablePlaceholders(false)指定未加载出来的条目是否用占位符替代,必须和setMaxSize搭配使用才有效
                .build();//最后构造出一个PagedList.Config对象,用以指定PagedList以何种方式分页

//2.创建数据源工厂类,用来创建数据提供者
DataSource.Factory factory = new DataSource.Factory() {
        @NonNull
        @Override
        public DataSource create() {
           return new ItemKeyedDataSource();
        }
    };

//3.构建一个能够触发加载页面初始化数据的LiveData对象,并且把上面创建的DataSource.Factory和PagedList.Config 传递进去
LiveData<PagedList<T>> pageData = new LivePagedListBuilder(factory, config)
                .setInitialLoadKey(0)//设置初始化数据加载的key(任意类型)
                //.setFetchExecutor()//指定使用哪个线程池进行异步工作
                .setBoundaryCallback(callback)//指定pagedList的第一条、最后一条，被加载到列表之上的边界回调callback。
                .build();//最后通过build方法 构建出LiveData对象。请注意它的 泛型是<PagedList<T>>

//4.最后我们只需要拿到前面构建出来的LiveData对象注册一个Observer观察者,仅仅如此就可以触发页面初始化数据的加载了
mViewModel.getPageData().observe(this, pagedList -> submitList(pagedList));

那么问题来了，LiveData和Paging数据加载是怎么配合的呢？怎么就产生火花了呢？ 
```

**2.** 那先来看一下Paging是如何利用LiveData能力的，首先要看下ComputableLiveData这个类，它并不是LiveData的子类，但它利用了LiveData的onActive方法被激活的时机。

> 这也就解释了为么这段代码可以触发Paging的初始化数据的加载逻辑
> mViewModel.getPageData().observe(this, pagedList -> submitList(pagedList));

```
public abstract class ComputableLiveData<T> {
public ComputableLiveData(@NonNull Executor executor) {
        mExecutor = executor;
       //在构造函数中创建了一个LiveData对象。并且复写了它的onActive方法。至关重要，看过我LiveData源码分析的同学应该知道，
       //该方法当且仅当有第一个Observer被注册到LiveData的时候，会被调用。
       //而当onActive被调用的时候，它使用线程池执行了RefreshRunnable，实际上是触发了下面的compute方法
        mLiveData = new LiveData<T>() {
            @Override
            protected void onActive() {
                mExecutor.execute(mRefreshRunnable);
            }
        };
    }

//一旦调用了该方法也便会触发下面的compute方法
 public void invalidate() {
   ArchTaskExecutor.getInstance().executeOnMainThread(mInvalidationRunnable)
  }
//虚方法，在LivePagedListBuilder中有唯一实现
 protected abstract T compute();

//获取在构造函数中创建的LiveData对象
 public LiveData<T> getLiveData() { return mLiveData;}
}
```

3.再来看一下上面的LivePagedListBuilder#build()方法又是是如何使用ComputableLiveData类的

噢，原来build()方法直接调用了内部方法create()并返回了LiveData对象

```
 private static <Key, Value> LiveData<PagedList<Value>> create(
            @Nullable final Key initialLoadKey,
            @NonNull final PagedList.Config config,
            @Nullable final PagedList.BoundaryCallback boundaryCallback,
            @NonNull final DataSource.Factory<Key, Value> dataSourceFactory,
            @NonNull final Executor notifyExecutor,
            @NonNull final Executor fetchExecutor) {

       //该方法直接new 了一个 ComputableLiveData，并复写了它的`compute`方法,至关重要。
        return new ComputableLiveData<PagedList<Value>>(fetchExecutor) {

            private final DataSource.InvalidatedCallback mCallback =
                    new DataSource.InvalidatedCallback() {
                        @Override
                        public void onInvalidated() {
                           // 一旦监听到DataSource被置为无效,则调用ComputableLiveData#invalidate方法。
                          //也就是会触发下面的compute方法
                            invalidate();
                        }
                    };


            @Override
            protected PagedList<Value> compute() {

                    //通过上面配置的dataSourceFactory创建一个数据源提供者
                    mDataSource = dataSourceFactory.create();
                   //并且于此同时会给mDataSource注册一个回调,用以监听该DataSource被置为无效的事件,
                   //一旦DataSource被置为无效,则不能在提供数据,但会再次触发该方法(compute)再次创建一个新的mDataSource，重新执行下面的逻辑
                    mDataSource.addInvalidatedCallback(mCallback);

                   //compute方法是ComputableLiveData 的方法
                  //会通过PagedList.Builder的build方法构造出一个PagedList，在跟下去就知道怎么触发初始化数据的加载了，来继续看下面
                    PagedList  mList = new PagedList.Builder<>(mDataSource, config)
                            .setNotifyExecutor(notifyExecutor)
                            .setFetchExecutor(fetchExecutor)
                            .setBoundaryCallback(boundaryCallback)
                            .setInitialKey(initializeKey)
                            .build();
                return mList;
            }
        }.getLiveData();
    }
```

**4.** 触发初始化数据的加载

```
ContiguousPagedList(
            @NonNull ContiguousDataSource<K, V> dataSource,
            @NonNull Executor mainThreadExecutor,
            @NonNull Executor backgroundThreadExecutor,
            @Nullable BoundaryCallback<V> boundaryCallback,
            @NonNull Config config,
            final @Nullable K key,
            int lastLoad) {
        super(new PagedStorage<V>(), mainThreadExecutor, backgroundThreadExecutor,
                boundaryCallback, config);
        mDataSource = dataSource;
        mLastLoad = lastLoad;

// 如果当前datasource已经被置为无效了，则不会触发初始化数据加载的逻辑
        if (mDataSource.isInvalid()) {
            detach();
        } else {
           //上面的build方法调用了之后，经过一堆判断后便会走到这里，mDataSource.dispatchLoadInitial遍触发了我们上面配置的DataSource的oadInitial方法，也就是触发了页面初始化数据的加载了。
           //其中mReceiver参数请注意,这是接收网络数据加载成功之后的callback
           // PageResult.Receiver是内部类,它会判断本次分页回来的数据是初始化数据，还是分页数据，用以确定分页的状态
          //而数据最终都是会被存储到PagedStorage<T>这个类中。它实际上是一个按页存储数据的ArrayList
            mDataSource.dispatchLoadInitial(key,
                    mConfig.initialLoadSizeHint,
                    mConfig.pageSize,
                    mConfig.enablePlaceholders,
                    mMainThreadExecutor,
                    mReceiver);
```

**5.** 触发分页数据的加载

分页数据加载的是在PagedListAdapter#getItem()方法中触发的,接着又会调用到ContiguousPagedList#loadAroundInternal方法。

该方法中会计算列表当前滑动状态下,列表后面还需要追加几条Item，列表前面还需要向前追加几条Item(Paging能向前向后分页加载数据,强大吧)。

```
public class ContiguousPagedList{
protected void loadAroundInternal(int index) {
        int prependItems = getPrependItemsRequested(mConfig.prefetchDistance, index,
                mStorage.getLeadingNullCount());
        int appendItems = getAppendItemsRequested(mConfig.prefetchDistance, index,
                mStorage.getLeadingNullCount() + mStorage.getStorageCount());

        mPrependItemsRequested = Math.max(prependItems, mPrependItemsRequested);
        if (mPrependItemsRequested > 0) {
           //计算之后，如果需要向前追加的Item数量大于0,  schedulePrepend则会触发DataSource的LoadBefore方法
            schedulePrepend();
        }

        mAppendItemsRequested = Math.max(appendItems, mAppendItemsRequested);
        if (mAppendItemsRequested > 0) {
            //计算之后，如果需要向后追加的Item数量大于0,  scheduleAppend则会触发DataSource的LoadAfter方法
            scheduleAppend();
        }
    }
}
```



##### **列表数据差分异增量更新**

如今,给列表RecyclerView设置Adapter,需要使用PagedListAdapter，并且要求传递一个DiffUtil.ItemCallback用以做列表新旧数据的差分异计算。如此便能使用Paging提供的列表数据差量更新能力了。

```
class UserAdapter extends PagedListAdapter<User,ViewHolder> {

    public void UserAdapter(){
           super( new DiffUtil.ItemCallback<User>() {
           @Override
          public boolean areItemsTheSame(@NonNull User oldUser, @NonNull User newUser) {
               return oldUser.getId() == newUser.getId();
          }
          @Override
           public boolean areContentsTheSame( @NonNull User oldUser, {@literal @}NonNull User newUser) {
               return oldUser.equals(newUser);
           }
       })
     }
}
```



##### **UML图解Paging工作流程**

Paging的设计、设计到的类十分的复杂，一篇文章难以概括其全部。下面用一张图描述Paging的工作原理

![img](https://mmbiz.qpic.cn/mmbiz_png/v1LbPPWiaSt63y6EnvBawfqxA8cDVsBFZtCdyJLIkA5XzaGrjLkZ6XPlicv7Wbyjd0Wuic3WXwDdz7UP4Ey2VSqfw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



##### **Paging美中不足**

虽然Paging的设计十分优秀，能力什么强悍，但现阶段还有些美中不足，没有合适的解决方案

\1. PagedList并不支持列表数据的增删改
\2. Paging一旦有一次分页失败，便再也不会继续分页了
\3. Paging如何先展示缓存数据再展示网络数据
\4. Paging如果先添加了HeaderView，再展示加载的网络数据，列表定位会有问题



