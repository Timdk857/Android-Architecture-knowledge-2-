**阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)**
## 使用生命周期感知组件处理生命周期
生命周期感知组件可以在其他组件（例如 activity 和 fragment）的生命周期状态更改时执行操作。这些组件可帮助我们生成更轻量级且更容易维护的代码。

我们通常会在 activity 和 fragment 的生命周期方法中实现自己的业务逻辑。但是，这种方式会导致我们代码组织的不良和出错的增加。但是通过使用生命周期感知组件，我们就可以将依赖组件生命周期的代码移出生命周期方法。

android.arch.lifecycle 包提供的类和接口，使我们可以构建生命周期感知组件 - 这些组件可以根据 activity 和 fragment 的当前生命周期状态自动调整其行为。

Android Framework 中定义的大多数应用程序组件都附加了生命周期。生命周期由操作系统或进程中运行的 Framework 代码管理。它们是Android工作原理的核心，我们的应用程序必须遵循它们，如果不这样做可能会触发内存泄漏甚至应用程序崩溃。

假设我们现在有一个 activity ，用来在手机屏幕上显示设备位置。常见的实现可能如下：
```
class MyLocationListener {
    public MyLocationListener(Context context, Callback callback) {
        // ...
    }

    void start() {
        // connect to system location service
    }

    void stop() {
        // disconnect from system location service
    }
}


class MyActivity extends AppCompatActivity {
    private MyLocationListener myLocationListener;

    @Override
    public void onCreate(...) {
        myLocationListener = new MyLocationListener(this, (location) -> {
            // update UI
        });
    }

    @Override
    public void onStart() {
        super.onStart();
        myLocationListener.start();
        // manage other components that need to respond
        // to the activity lifecycle
    }

    @Override
    public void onStop() {
        super.onStop();
        myLocationListener.stop();
        // manage other components that need to respond
        // to the activity lifecycle
    }
}
```
尽管这个示例看起来很好，但在实际的应用程序中，最终会有太多的回调来管理 UI 和 其他组件以响应生命周期的当前状态。管理多个组件会在生命周期方法中放置大量代码，导致如 onStart() 和 onStop() 这样的方法难以维护。

此外，我们无法保证 MyLocationListener 在 MyActivity 停止之前启动，如果我们需要执行长时间运行的操作（例如在 onStart() 中进行一些 配置检查），这会导致 MyActivity 的 onStop() 完成之前而这个操作才会回调，这就使组件保持活动的时间会长于其所需的时间。
```
class MyActivity extends AppCompatActivity {
    private MyLocationListener myLocationListener;

    public void onCreate(...) {
        myLocationListener = new MyLocationListener(this, location -> {
            // update UI
        });
    }

    @Override
    public void onStart() {
        super.onStart();
        Util.checkUserStatus(result -> {
            // what if this callback is invoked AFTER activity is stopped?
            if (result) {
                myLocationListener.start();
            }
        });
    }

    @Override
    public void onStop() {
        super.onStop();
        myLocationListener.stop();
    }
}
```
android.arch.lifecycle 包提供了类和接口，可以帮助我们以弹性和隔离的方式解决这些问题。

## 生命周期
Lifecycle 是一个持有关于组件（如 activity 或fragment）生命周期状态信息并允许其他对象观察此状态的类。

Lifecycle 使用两个主要枚举来跟踪其关联组件的生命周期状态：

**Event**

从 Framework 和 Lifecycle 类分发生命周期事件 。这些事件映射到 activities 和 fragments 中的回调事件。

**State**

通过 Lifecycle 对象来跟踪组件的当前状态 。
![](https://upload-images.jianshu.io/upload_images/19956127-8135ce224007f36a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

将 State 视为图形的节点，将 Event 视为这些节点之间的边缘。

一个类可以通过向组件的方法添加注解来监视其生命周期状态。我们可以通过调用 Lifecycle 类的 addObserver() 方法来添加观察者，并传递观察者的实例，如以下示例所示：
```
public class MyObserver implements LifecycleObserver {
    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    public void connectListener() {
        ...
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    public void disconnectListener() {
        ...
    }
}

myLifecycleOwner.getLifecycle().addObserver(new MyObserver());
```
在上面的示例中，myLifecycleOwner 对象实现了 LifecycleOwner 接口，这个在下一节中进行说明。

## LifecycleOwner
LifecycleOwner 是一个接口，它只有一个 getLifecycle() 方法，表明实现LifecycleOwner接口的类有一个 Lifecycle 。如果你正在尝试管理整个应用进程的生命周期，可以看下 ProcessLifecycleOwner。

LifecycleOwner 接口从个别类（例如Fragment 和 AppCompatActivity）中抽象出了 Lifecycle 的所有权，并允许编写与其一起使用的组件。任何自定义类都可以实现 LifecycleOwner 接口。

实现了 LifecycleObserver 的组件与实现了 LifecycleOwner 的组件可以无缝工作，因为所有者可以提供生命周期，观察者可以注册观察。

例如位置跟踪的例子，我们可以使 MyLocationListener类实现 LifecycleObserver ，然后在 activity 生命周期的 onCreate() 方法中初始化它，这意味着响应生命周期状态变化的逻辑可以被声明在 MyLocationListener 而不是 activity，当把逻辑代码置于在各个组件中后，就能使 activities 和 fragments 更易于管理。
```
class MyActivity extends AppCompatActivity {
    private MyLocationListener myLocationListener;

    public void onCreate(...) {
        myLocationListener = new MyLocationListener(this, getLifecycle(), location -> {
            // update UI
        });
        Util.checkUserStatus(result -> {
            if (result) {
                myLocationListener.enable();
            }
        });
  }
}
```
一个常见的用例是，如果 Lifecycle 现在不处于激活状态，则应避免调用某些回调 。例如：如果回调在 Activity 状态被保存到 onSaveInstanceState() 后运行 Fragment Transaction，就会触发崩溃，因此我们会避免调用该回调。

为了简化这个用例， Lifecycle 类允许其他对象查询当前状态。
```
class MyLocationListener implements LifecycleObserver {
    private boolean enabled = false;
    public MyLocationListener(Context context, Lifecycle lifecycle, Callback callback) {
       ...
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    void start() {
        if (enabled) {
           // connect
        }
    }

    public void enable() {
        enabled = true;
        if (lifecycle.getCurrentState().isAtLeast(STARTED)) {
            // connect if not connected
        }
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    void stop() {
        // disconnect if connected
    }
}
```
通过这种实现，我们的LocationListener类完全可以感知生命周期。如果我们需要从另一个 activity 或 fragment 使用LocationListener，我们只需要初始化它。所有设置和卸载操作都由LocationListener类本身管理。

如果一个库提供了需要使用Android生命周期的类，Google 建议我们使用生命周期感知组件。我们的客户端可以轻松地集成这些组件，而无需在客户端进行手动管理生命周期。

## 实现自定义LifecycleOwner
在 Support Library 26.1.0 及更高版本的 Fragments 和 Activities 已实现了 LifecycleOwner 接口。

如果你想让自定义的类成为 LifecycleOwner，可以使用 LifecycleRegistry 类，但是你需要将生命周期事件转发到此类，如以下代码所示：
```
public class MyActivity extends Activity implements LifecycleOwner {
    private LifecycleRegistry mLifecycleRegistry;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        mLifecycleRegistry = new LifecycleRegistry(this);
        mLifecycleRegistry.markState(Lifecycle.State.CREATED);
    }

    @Override
    public void onStart() {
        super.onStart();
        mLifecycleRegistry.markState(Lifecycle.State.STARTED);
    }

    @NonNull
    @Override
    public Lifecycle getLifecycle() {
        return mLifecycleRegistry;
    }
}
```
## 生命周期感知组件的最佳实践
尽可能精简我们的 UI 控制器（Activity 和 Fragment）。它们不应该尝试获取自己的数据， 相反，应该使用 ViewModel 来执行此操作，并观察 LiveData 对象以将数据更改反映到视图上。

编写数据驱动的UI，UI控制器负责在数据更改时更新视图，或者将用户操作通知给 ViewModel 。

把我们的数据逻辑放在 ViewModel 类中。 ViewModel 应该作为 UI 控制器和应用程序其余部分之间的桥接器。不过 ViewModel 是没有获取数据（例如从网络）的职责的。相反， ViewModel 应该调用合适的组件（例如 Repository组件）来获取数据，然后将结果提供给 UI 控制器。

使用 Data Binding 在 视图 和 UI 控制器之间维持一个干净的界面。这会使我们的视图更具声明性，并最大限度地减少在 Activity 和 Fragment 中编写所需的更新代码。如果你更喜欢使用Java 编程语言执行此操作，那就使用 Butter Knife 之类的库来避免样板代码，使代码具有更好的抽象。

如果你的 UI 很复杂，可以考虑创建一个 presenter 类来处理UI修改。这可能是一项蛋疼的任务，但是它可以让我们的 UI 组件更容易测试。

避免在你的 ViewModel 中引用 View 或 Activity 的 context 。如果 ViewModel 的存活时间超过这个activity（如在 configuration 更改的情况下），导致垃圾收集器未正确处理 activity 造成内存泄露。

## 生命周期感知组件的用例
生命周期感知组件可以让我们在各种情况下更轻松地管理生命周期。比如以下场景：

在粗粒度和细粒度的位置更新之间切换。使用生命周期感知组件可在我们的位置应用程序可见时启用细粒度位置更新，并在应用程序位于后台时切换到粗粒度更新。LiveData 是一个生命周期感知组件，它允许我们的应用在用户更改位置时自动更新 UI。

停止和开始视频缓冲。使用生命周期感知组件可以尽快开启视频缓冲，但推迟播放直到应用程序完全启动。我们还可以使用生命周期感知组件在销毁应用程序时终止缓冲。

开启和停止网络连接。使用生命周期感知组件在应用程序处于前台时启用网络数据的实时更新（流式传输），并在应用程序进入后台时自动暂停。

暂停和恢复动画绘制。当应用程序在后台时，使用生命周期感知组件暂停绘制动画内容，并在应用程序位于前台后恢复绘制内容。

## 处理 ON_STOP 事件
如果一个 Lifecycle 属于一个 AppCompatActivity 或 Fragment，当 AppCompatActivity 或 Fragment 的 onSaveInstanceState() 方法被调用时， 这个 Lifecycle 的状态就会变为 CREATED ，它的 ON_STOP 事件也会被分发 。

当一个 AppCompatActivity 或 Fragment 的状态被保存到 onSaveInstanceState() 后，它的UI在 ON_START 被调用之前被认为是不可变的 。保存状态后尝试修改UI 可能会导致应用程序的导航状态不一致，这就是为什么如果应用程序在保存状态后运行 FragmentTransaction ，FragmentManager 会抛出异常的原因，详细原因请查看 commit() 。

如果观察者的关联 Lifecycle 不是STARTED 状态， LiveData 会通过避免调用其观察者来防止这种边缘情况开箱即用。在背后，它会确保在调用它的观察者之前调用 isAtLeast() 。

蛋疼的是，AppCompatActivity 的 onStop() 方法会在 onSaveInstanceState() 之后调用，这就导致程序留下了不允许更改UI状态但 Lifecycle 又还没转变为 CREATED 状态的间隔。

为了防止出现此问题， Lifecycle 类的 beta2 版本以及更低版本将 CREATED 状态标记为不调度事件，即使在系统调用 onStop() 之前未调度事件，任何检查当前状态的代码都会获得实际值。

悲催的是这个解决方案有两个重大问题：

在API 23 和更低级别，Android系统实际上保存了 activity 的状态，即使它被另一个 activity 部分覆盖。换句话说，Android系统调用 onSaveInstanceState() 但不一定要调用 onStop() 。这会创建一个潜在的长间隔，即使无法修改其UI状态，观察者仍然- - 认为生命周期处于活跃状态。

任何类想要向 LiveData 类暴露类似行为 都必须实现 Lifecycle beta 2 版本 和更低版本提供的解决方法 。

## 注意
为了使此流程更简单并提供与旧版本的更好兼容性，从1.0.0-rc1版本开始，在调用 onSaveInstanceState() 时 Lifecycle 对象被标记为 CREATED 并且调度 ON_STOP ，而无须等待调用 onStop() 方法。这可能不会影响我们的代码，但我们需要注意这一点，因为它与 API 26及更低级别的Activity 类中的调用顺序不匹配。

原文链接：https://blog.csdn.net/guiying712/article/details/81176039
**阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)**
