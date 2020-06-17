**阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)**

#### 前言

本篇文章主要讲解LiveData工作的原理，如果还不知道LiveData如何用的话，[请参考官方文档](https://developer.android.com/topic/libraries/architecture/livedata)。
LiveData的讲解涉及到了Lifecycle的知识，如果你还不了解LifeCycle，[请参考文档LifeCycle介绍](https://developer.android.com/reference/android/arch/lifecycle/Lifecycle)。

#### 介绍

LiveData是一个数据持有类，它可以通过添加观察者被其他组件观察其变更。不同于普通的观察者，它最重要的特性就是遵从应用程序的生命周期，如在Activity中如果数据更新了但Activity已经是destroy状态，LivaeData就不会通知Activity(observer)。当然。LiveData的优点还有很多，如不会造成内存泄漏等。

LiveData通常会配合ViewModel来使用，ViewModel负责触发数据的更新，更新会通知到LiveData，然后LiveData再通知活跃状态的观察者。

#### 原理分析

下面直接看代码：

```
public class UserProfileViewModel extends ViewModel {
    private String userId;
    private MutableLiveData<User> user;
    private UserRepository userRepo;

    public void init(String userId) {
        this.userId = userId;
        userRepo = new UserRepository();
        user = userRepo.getUser(userId);
    }

    public void refresh(String userId) {
        user = userRepo.getUser(userId);
    }

    public MutableLiveData<User> getUser() {
        return user;
    }

}

```

上面UserProfileViewModel内部持有 UserRepository 中 MutableLiveData<User>的引用，并且提供了获取 MutableLiveData 的方法 getUser()，UserRepository 负责从网络或数据库中获取数据并封装成 MutableLiveData 然后提供给 ViewModel。

我们在 UserProfileFragment 中为 MutableLiveData<User> 注册观察者，如下：

```
@Override
public void onActivityCreated(@Nullable Bundle savedInstanceState) {
    super.onActivityCreated(savedInstanceState);
    String userId = getArguments().getString(UID_KEY);
    viewModel = ViewModelProviders.of(this).get(UserProfileViewModel.class);
    viewModel.init(userId);
    //标注1
    viewModel.getUser().observe(UserProfileFragment.this, new Observer<User>() {
        @Override
        public void onChanged(@Nullable User user) {
            if (user != null) {
                tvUser.setText(user.toString());
            }
        }
    });
}

```

看标注1处，viewModel.getUser()获取到 MutableLiveData<User> 也就是我们的 LiveData，然后调用 LiveData的observer方法，并把UserProfileFragment作为参数传递进去。observer() 方法就是我们分析的入口了，接下来我们看LiveData的observer()方法都做了什么：

```
@MainThread
public void observe(@NonNull LifecycleOwner owner, @NonNull Observer<T> observer) {
    //标注1
    if (owner.getLifecycle().getCurrentState() == DESTROYED) {
        // ignore
        return;
    }
    //标注2
    LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer);
    ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);
    if (existing != null && !existing.isAttachedTo(owner)) {
        throw new IllegalArgumentException("Cannot add the same observer"
                + " with different lifecycles");
    }
    if (existing != null) {
        return;
    }
    owner.getLifecycle().addObserver(wrapper);
}

```

可以看到，UserProfileFragment 是作为 LifeCycleOwner 参数传进来的，如果你的support包版本大于等于26.1.0，support包中的 Fragment 会默认继承自 LifecycleOwner，而 LifecycleOwner 可获取到该组件的 LifeCycle，也就知道了 UserProfileFragment 组件的生命周期（在这里默认大家已经了解过LifeCycle了）。

看标注1处，如果我们的 UserProfileFragment 组件已经是destroy状态的话，将直接返回，不会被加入观察者行列。如果不是destroy状态，就到标注2处，新建一个 LifecycleBoundObserver 将我们的 LifecycleOwner 和 observer保存起来，然后调用 mObservers.putIfAbsent(observer, wrapper) 将observer和wrapper分别作为key和value存入Map中，putIfAbsent()方法会判断如果 value 已经能够存在，就返回，否则返回null。
如果返回existing为null，说明以前没有添加过这个观察者，就将 LifecycleBoundObserver 作为 owner 生命周期的观察者，也就是作为 UserProfileFragment 生命周期的观察者。

我们看下LifecycleBoundObserver 源码：

```
class LifecycleBoundObserver extends ObserverWrapper implements GenericLifecycleObserver {
    @NonNull final LifecycleOwner mOwner;

    LifecycleBoundObserver(@NonNull LifecycleOwner owner, Observer<T> observer) {
        super(observer);
        mOwner = owner;
    }

    @Override
    boolean shouldBeActive() {
        return mOwner.getLifecycle().getCurrentState().isAtLeast(STARTED);
    }

    @Override
    public void onStateChanged(LifecycleOwner source, Lifecycle.Event event) {
        if (mOwner.getLifecycle().getCurrentState() == DESTROYED) {
            removeObserver(mObserver);
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

代码并不多，LifecycleBoundObserver 继承自 ObserverWrapper 并实现了 GenericLifecycleObserver接口，而 GenericLifecycleObserver 接口又继承自 LifecycleObserver 接口，那么根据 Lifecycle 的特性，实现了LifecycleObserver接口并且加入 LifecycleOwner 的观察者里就可以感知或主动获取 LifecycleOwner 的状态。

好了，看完了观察者，那么我们的LiveData什么时候会通知观察者呢？不用想，肯定是数据更新的时候，而数据的更新是我们代码自己控制的，如请求网络返回User信息后，我们会主动将User放入MutableLiveData中，这里我在UserRepository中直接模拟网络请求如下：

```
public class UserRepository {
    final MutableLiveData<User> data = new MutableLiveData<>();

    public MutableLiveData<User> getUser(final String userId) {
        if ("xiasm".equals(userId)) {
            data.setValue(new User(userId, "夏胜明"));
        } else if ("123456".equals(userId)) {
            data.setValue(new User(userId, "哈哈哈"));
        } else {
            data.setValue(new User(userId, "unknow"));
        }
        return data;
    }
}

```

当调用getUser()方法的时候，我们调用MutableLiveData的setValue()方法将数据放入LiveData中，这里MutableLiveData实际上就是继承自LiveData，没有什么特别：

```
public class MutableLiveData<T> extends LiveData<T> {
    @Override
    public void postValue(T value) {
        super.postValue(value);
    }

    @Override
    public void setValue(T value) {
        super.setValue(value);
    }
}

```

setValue()在放入User的时候必须在主线程，否则会报错，而postValue则没有这个检查，而是会把数据传入到主线程。我们直接看setValue()方法：

```
@MainThread
protected void setValue(T value) {
    assertMainThread("setValue");
    mVersion++;
    mData = value;
    dispatchingValue(null);
}

```

首先调用assertMainThread()检查是否在主线程，接着将要更新的数据赋给mData，然后调用 dispatchingValue()方法并传入null，将数据分发给各个观察者，如我们的 UserProfileFragment。看 dispatchingValue()方法实现：

```
private void dispatchingValue(@Nullable ObserverWrapper initiator) {
    if (mDispatchingValue) {
        mDispatchInvalidated = true;
        return;
    }
    mDispatchingValue = true;
    do {
        mDispatchInvalidated = false;
        //标注1
        if (initiator != null) {
            considerNotify(initiator);
            initiator = null;
        } else {
            //标注2
            for (Iterator<Map.Entry<Observer<T>, ObserverWrapper>> iterator =
                    mObservers.iteratorWithAdditions(); iterator.hasNext(); ) {
                considerNotify(iterator.next().getValue());
                if (mDispatchInvalidated) {
                    break;
                }
            }
        }
    } while (mDispatchInvalidated);
    mDispatchingValue = false;
}

```

从标注1可以看出，dispatchingValue()参数传null和不传null的区别就是如果传null将会通知所有的观察者，反之仅仅通知传入的观察者。我们直接看标注2，通知所有的观察者通过遍历 mObservers ，将所有的 ObserverWrapper 拿到，实际上就是我们上面提到的 LifecycleBoundObserver，通知观察者调用considerNotify()方法，这个方法就是通知的具体实现了。

```
private void considerNotify(ObserverWrapper observer) {
    if (!observer.mActive) {
        return;
    }
    // Check latest state b4 dispatch. Maybe it changed state but we didn't get the event yet.
    //
    // we still first check observer.active to keep it as the entrance for events. So even if
    // the observer moved to an active state, if we've not received that event, we better not
    // notify for a more predictable notification order.
    if (!observer.shouldBeActive()) {
        observer.activeStateChanged(false);
        return;
    }
    if (observer.mLastVersion >= mVersion) {
        return;
    }
    observer.mLastVersion = mVersion;
    //noinspection unchecked
    observer.mObserver.onChanged((T) mData);
}

```

如果观察者不是活跃状态，将不会通知此观察者，看最后一行，observer.mObserver.onChanged((T) mData)，observer.mObserver就是我们调用LiveData的observer()方法传入的 Observer，然后调用 Observer 的 onChanged((T) mData)方法，将保存的数据mData传入，也就实现了更新。在看下我们实现的Observer：

```
viewModel.getUser().observe(UserProfileFragment.this, new Observer<User>() {
    @Override
    public void onChanged(@Nullable User user) {
        if (user != null) {
            tvUser.setText(user.toString());
        }
    }
});

```

如果哪个控件要根据user的变更而及时更新，就在onChanged()方法里处理就可以了。到这里，LiveData已经能够分析完了，其实LiveData的实现还是要依赖于Lifecycle。

原文链接：https://www.jianshu.com/p/21bb6c0f8a5a
**阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)**
