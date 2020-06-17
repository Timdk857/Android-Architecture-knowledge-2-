**阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)**
## 概述
ViewModel，从字面上理解的话，它肯定是跟视图(View)以及数据(Model)相关的。正像它字面意思一样，它是负责准备和管理和UI组件(Fragment/Activity)相关的数据类，也就是说ViewModel是用来管理UI相关的数据的，同时ViewModel还可以用来负责UI组件间的通信。

## 之前存在的问题
ViewModel用来存储和管理UI相关的数据，可于将一个Activity或Fragment组件相关的数据逻辑抽象出来，并能适配组件的生命周期，如当屏幕旋转Activity重建后，ViewModel中的数据依然有效。

#### 引入ViewModel之前，存在如下几个问题：

通常Android系统来管理UI controllers（如Activity、Fragment）的生命周期，由系统响应用户交互或者重建组件，用户无法操控。当组件被销毁并重建后，原来组件相关的数据也会丢失，如果数据类型比较简单，同时数据量也不大，可以通过onSaveInstanceState()存储数据，组件重建之后通过onCreate()，从中读取Bundle恢复数据。但如果是大量数据，不方便序列化及反序列化，则上述方法将不适用。
UI controllers经常会发送很多异步请求，有可能会出现UI组件已销毁，而请求还未返回的情况，因此UI controllers需要做额外的工作以防止内存泄露。 
当Activity因为配置变化而销毁重建时，一般数据会重新请求，其实这是一种浪费，最好就是能够保留上次的数据。
UI controllers其实只需要负责展示UI数据、响应用户交互和系统交互即可。但往往开发者会在Activity或Fragment中写许多数据请求和处理的工作，造成UI controllers类代码膨胀，也会导致单元测试难以进行。我们应该遵循职责分离原则，将数据相关的事情从UI controllers中分离出来。
#### ViewModel基本使用
```
public class MyViewModel extends ViewModel {
    private MutableLiveData<List<User>> users;
    public LiveData<List<User>> getUsers() {
        if (users == null) {
            users = new MutableLiveData<List<Users>>();
            loadUsers();
        }
        return users;
    }

    private void loadUsers() {
        // 异步调用获取用户列表
    }
}
```
新的Activity如下：
```
public class MyActivity extends AppCompatActivity {
    public void onCreate(Bundle savedInstanceState) {
        MyViewModel model = ViewModelProviders.of(this).get(MyViewModel.class);
        model.getUsers().observe(this, users -> {
            // 更新 UI
        });
    }
}
```
如果Activity被重新创建了，它会收到被之前Activity创建的相同MyViewModel实例。当所属Activity终止后，框架调用ViewModel的onCleared()方法清除资源。

因为ViewModel在指定的Activity或Fragment实例外存活，它应该永远不能引用一个View，或持有任何包含Activity context引用的类。如果ViewModel需要Application的context（如获取系统服务），可以扩展AndroidViewmodel，并拥有一个构造器接收Application。

#### 在Fragment间共享数据
一个Activity中的多个Fragment相互通讯是很常见的。之前每个Fragment需要定义接口描述，所属Activity将二者捆绑在一起。此外，每个Fragment必须处理其他Fragment未创建或不可见的情况。通过使用ViewModel可以解决这个痛点，这些Fragment可以使用它们的Activity共享ViewModel来处理通讯：
```
public class SharedViewModel extends ViewModel {
    private final MutableLiveData<Item> selected = new MutableLiveData<Item>();

    public void select(Item item) {
        selected.setValue(item);
    }

    public LiveData<Item> getSelected() {
        return selected;
    }
}

public class MasterFragment extends Fragment {
    private SharedViewModel model;
    public void onActivityCreated() {
        model = ViewModelProviders.of(getActivity()).get(SharedViewModel.class);
        itemSelector.setOnClickListener(item -> {
            model.select(item);
        });
    }
}

public class DetailFragment extends LifecycleFragment {
    public void onActivityCreated() {
        SharedViewModel model = ViewModelProviders.of(getActivity()).get(SharedViewModel.class);
        model.getSelected().observe(this, { item ->
           // update UI
        });
    }
}
```
**注意:上面两个Fragment都用到了如下代码来获取ViewModel，getActivity()返回的是同一个宿主Activity，因此两个Fragment之间返回的是同一个SharedViewModel对象。**
```
SharedViewModel model = ViewModelProviders.of(getActivity()).get(SharedViewModel.class);
```
这种方式的好处包括：

1.Activity不需要做任何事情，也不需要知道通讯的事情
2.Fragment不需要知道彼此，除了SharedViewModel进行联系。如果它们(Fragment)其中一个消失了，其余的仍然能够像往常一样工作。
3.每个Fragment有自己的生命周期，而且不会受其它Fragment生命周期的影响。事实上，一个Fragment替换另一个Fragment，UI的工作也不会受到任何影响。
#### ViewModel的生命周期
ViewModel对象的范围由获取ViewModel时传递至ViewModelProvider的Lifecycle所决定。ViewModel始终处在内存中，直到Lifecycle永久地离开—对于Activity来说，是当它终止（finish）的时候，对于Fragment来说，是当它分离（detached）的时候。 
 ![](https://upload-images.jianshu.io/upload_images/19956127-3f03922ab943aba3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上图左侧为Activity的生命周期过程，期间有一个旋转屏幕的操作；右侧则为ViewModel的生命周期过程。

一般通过如下代码初始化ViewModel：
```
viewModel = ViewModelProviders.of(this).get(UserProfileViewModel.class);
```
this参数一般为Activity或Fragment，因此ViewModelProvider可以获取组件的生命周期。

Activity在生命周期中可能会触发多次onCreate()，而ViewModel则只会在第一次onCreate()时创建，然后直到最后Activity销毁。

#### ViewModel相关类图
![](https://upload-images.jianshu.io/upload_images/19956127-c3814feecf9d1720.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

ViewModelProviders是ViewModel工具类，该类提供了通过Fragment和Activity得到ViewModel的方法，而具体实现又是由ViewModelProvider实现的。

ViewModelProvider是实现ViewModel创建、获取的工具类。在ViewModelProvider中定义了一个创建ViewModel的接口类——Factory。ViewModelProvider中有个ViewModelStore对象，用于存储ViewModel对象。

ViewModelStore是存储ViewModel的类，具体实现是通过HashMap来保存ViewModle对象。

ViewModel是个抽象类，里面只定义了一个onCleared()方法，该方法在ViewModel不在被使用时调用。ViewModel有一个子类AndroidViewModel，这个类是便于要在ViewModel中使用Context对象，因为我们前面提到不能在ViewModel中持有Activity的引用。

ViewModelStores是ViewModelStore的工厂方法类，它会关联HolderFragment，HolderFragment有个嵌套类——HolderFragmentManager。

#### ViewModel相关时序图
追溯创建一个ViewModel的源码，会察觉需要的步骤有点多。下面以在Fragment中得到ViewModel对象为例看下整个过程的时序图。 
![](https://upload-images.jianshu.io/upload_images/19956127-7571dacd95f6f3f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

时序图看起来比较复杂，但是它只描述了两个过程：

得到ViewModel对象。
HolderFragment被销毁时，ViewModel收到onCleared()通知。
## ViewModel相关源码分析
#### ViewModelProviders类的具体实现：
```
public class ViewModelProviders {

    private static Application checkApplication(Activity activity) {
        Application application = activity.getApplication();
        if (application == null) {
            throw new IllegalStateException("Your activity/fragment is not yet attached to "
                    + "Application. You can't request ViewModel before onCreate call.");
        }
        return application;
    }

    private static Activity checkActivity(Fragment fragment) {
        Activity activity = fragment.getActivity();
        if (activity == null) {
            throw new IllegalStateException("Can't create ViewModelProvider for detached fragment");
        }
        return activity;
    }

    @NonNull
    @MainThread
    public static ViewModelProvider of(@NonNull Fragment fragment) {
        ViewModelProvider.AndroidViewModelFactory factory =
                ViewModelProvider.AndroidViewModelFactory.getInstance(
                        checkApplication(checkActivity(fragment)));
        return new ViewModelProvider(ViewModelStores.of(fragment), factory);
    }

    @NonNull
    @MainThread
    public static ViewModelProvider of(@NonNull FragmentActivity activity) {
        ViewModelProvider.AndroidViewModelFactory factory =
                ViewModelProvider.AndroidViewModelFactory.getInstance(
                        checkApplication(activity));
        return new ViewModelProvider(ViewModelStores.of(activity), factory);
    }

    @NonNull
    @MainThread
    public static ViewModelProvider of(@NonNull Fragment fragment, @NonNull Factory factory) {
        checkApplication(checkActivity(fragment));
        return new ViewModelProvider(ViewModelStores.of(fragment), factory);
    }

    @NonNull
    @MainThread
    public static ViewModelProvider of(@NonNull FragmentActivity activity,
            @NonNull Factory factory) {
        checkApplication(activity);
        return new ViewModelProvider(ViewModelStores.of(activity), factory);
    }
```
ViewModelProviders提供了四个of()方法，四个方法功能类似，其中of(FragmentActivity activity, Factory factory)和of(Fragment fragment, Factory factory)提供了自定义创建ViewModel的方法。 
1. 判断Fragment的是否Attached to Activity，Activity的Application对象是否为空。 
2. 创建ViewModel对象看似很简单，一行代码搞定。
```
new ViewModelProvider(ViewModelStores.of(fragment), factory)
```
先看看ViewModelStores.of()方法:
```
    @NonNull
    @MainThread
    public static ViewModelStore of(@NonNull Fragment fragment) {
        if (fragment instanceof ViewModelStoreOwner) {
            return ((ViewModelStoreOwner) fragment).getViewModelStore();
        }
        return holderFragmentFor(fragment).getViewModelStore();
    }
```
继续深入发现其实是实现了一个接口：
```
public interface ViewModelStoreOwner {
    @NonNull
    ViewModelStore getViewModelStore();
}
```
holderFragmentFor()是HolderFragment的静态方法，HolderFragment继承自Fragment。我们先看holderFragment()方法的具体实现
```
    @RestrictTo(RestrictTo.Scope.LIBRARY_GROUP)
    public static HolderFragment holderFragmentFor(Fragment fragment) {
        return sHolderFragmentManager.holderFragmentFor(fragment);
    }

    @NonNull
    @Override
    public ViewModelStore getViewModelStore() {
        return mViewModelStore;
    }
```
继续看HolderFragmentManager.holderFragmentFor()方法的具体实现
```
HolderFragment holderFragmentFor(Fragment parentFragment) {
            FragmentManager fm = parentFragment.getChildFragmentManager();
            HolderFragment holder = findHolderFragment(fm);
            if (holder != null) {
                return holder;
            }
            holder = mNotCommittedFragmentHolders.get(parentFragment);
            if (holder != null) {
                return holder;
            }

            parentFragment.getFragmentManager()
                    .registerFragmentLifecycleCallbacks(mParentDestroyedCallback, false);
            holder = createHolderFragment(fm);
            mNotCommittedFragmentHolders.put(parentFragment, holder);
            return holder;
        }

private FragmentLifecycleCallbacks mParentDestroyedCallback =
    new FragmentLifecycleCallbacks() {
        @Override
        public void onFragmentDestroyed(FragmentManager fm, Fragment parentFragment) {
            super.onFragmentDestroyed(fm, parentFragment);
            HolderFragment fragment = mNotCommittedFragmentHolders.remove(
                    parentFragment);
            if (fragment != null) {
                Log.e(LOG_TAG, "Failed to save a ViewModel for " + parentFragment);
            }
        }
    };

private static HolderFragment createHolderFragment(FragmentManager fragmentManager) {
    HolderFragment holder = new HolderFragment(); // 创建HolderFragment对象
    fragmentManager.beginTransaction().add(holder, HOLDER_TAG).commitAllowingStateLoss();
    return holder;
}

public HolderFragment() {
    //这个是关键，这就使得Activity被recreate时，Fragment的onDestroy()和onCreate()不会被调用
    setRetainInstance(true); 
}
```
setRetainInstance(boolean) 是Fragment中的一个方法。将这个方法设置为true就可以使当前Fragment在Activity重建时存活下来, 如果不设置或者设置为 false, 当前 Fragment 会在 Activity 重建时同样发生重建, 以至于被新建的对象所替代。

在setRetainInstance(boolean)为true的 Fragment 中放一个专门用于存储ViewModel的Map, 自然Map中所有的ViewModel都会幸免于Activity重建，让Activity, Fragment都绑定一个这样的Fragment, 将ViewModel存放到这个 Fragment 的 Map 中, ViewModel 组件就这样实现了。

到此为止，我们已经得到了ViewStore对象，前面我们在创建ViewModelProvider对象是通过这行代码实现的new ViewModelProvider(ViewModelStores.of(fragment), sDefaultFactory)现在再看下ViewModelProvider的构造方法
```
public ViewModelProvider(@NonNull ViewModelStore store, @NonNull Factory factory) {
        mFactory = factory;
        this.mViewModelStore = store;
    }
```
现在就可以通过ViewModelProvider.get()方法得到ViewModel对象，继续看下该方法的具体实现
```
    @NonNull
    @MainThread
    public <T extends ViewModel> T get(@NonNull String key, @NonNull Class<T> modelClass) {
        ViewModel viewModel = mViewModelStore.get(key); //从缓存中查找是否有已有ViewModel对象。

        if (modelClass.isInstance(viewModel)) {
            //noinspection unchecked
            return (T) viewModel;
        } else {
            //noinspection StatementWithEmptyBody
            if (viewModel != null) {
                // TODO: log a warning.
            }
        }

        viewModel = mFactory.create(modelClass); //创建ViewModel对象，然后缓存起来。
        mViewModelStore.put(key, viewModel);
        //noinspection unchecked
        return (T) viewModel;
    }
```
ViewModelProvider.get()方法比较简单，注释中都写明了。最后我们看下ViewModelStore类的具体实现
```
public class ViewModelStore {

    private final HashMap<String, ViewModel> mMap = new HashMap<>();

    final void put(String key, ViewModel viewModel) {
        ViewModel oldViewModel = mMap.get(key);
        if (oldViewModel != null) {
            oldViewModel.onCleared();
        }
        mMap.put(key, viewModel);
    }

    final ViewModel get(String key) {
        return mMap.get(key);
    }

    public final void clear() {
        for (ViewModel vm : mMap.values()) {
            vm.onCleared();
        }
        mMap.clear();
    }
}
```
ViewModelStore是缓存ViewModel的类，put()、get()方法用于存取ViewModel对象，另外提供了clear()方法用于清空缓存的ViewModel对象，在该方法中会调用ViewModel.onCleared()方法通知ViewModel对象不再被使用。

#### ViewModel收到onCleared()通知 
HolderFragment的onDestroy()方法
```
    @Override
    public void onDestroy() {
        super.onDestroy();
        mViewModelStore.clear();
    }
```
在onDestroy()方法中调用了ViewModelStore.clear()方法，我们知道在该方法中会调用ViewModel的onCleared()方法。在你看了HolderFragment源码后，或许你会有个疑问，mViewModelStore保存的ViewModel对象是在哪里添加的呢？ 细心的话，你会发现在ViewModelProvider的构造方法中，已经将HolderFragment中的ViwModelStore对象mViewModelStore的引用传递给了ViewModelProvider中的mViewModelStore，而在ViewModelProvider.get()方法中会向mViewModelStore添加ViewModel对象。

## 总结
ViewModel职责是为Activity或Fragment管理、请求数据，具体数据请求逻辑不应该写在ViewModel中，否则ViewModel的职责会变得太重，此处需要一个引入一个Repository，负责数据请求相关工作。具体请参考 Android架构组件。
ViewModel可以用于Activity内不同Fragment的交互，也可以用作Fragment之间一种解耦方式。
ViewModel也可以负责处理部分Activity/Fragment与应用其他模块的交互。
ViewModel生命周期（以Activity为例）起始于Activity第一次onCreate()，结束于Activity最终finish时。

原文链接：https://blog.csdn.net/qq_24442769/article/details/79426609
**阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)**
