<meta charset="utf-8">

## Window、WindowManager和WMS

Window是一个抽象类，具体的实现类为PhoneWindow，它是对View进行管理的。
WindowManager是一个接口类，继承自父接口ViewManager，它是用来管理Window的，它的具体实现类为WindowManagerImpl。
WMS是WindowManager进行窗口管理的具体实施者，如果我们想要对Window进行添加和删除就可以使用WindowManager，具体的工作都是由WMS来处理的，WindowManager和WMS通过Binder来进行跨进程通信，WMS作为系统服务有很多API是不会暴露给WindowManager的，这一点与ActivityManager和AMS的关系有些类似。
Window、WindowManager和WMS的关系可以用下图来表示：

![](//upload-images.jianshu.io/upload_images/19956127-0e3cfc26d124fd37.png?imageMogr2/auto-orient/strip|imageView2/2/w/418/format/webp)

Window包含了View并对View进行管理，Window用虚线来表示是因为Window是一个抽象概念，并不是真实存在，Window的实体其实也是View。WindowManager用来管理Window，而WindowManager所提供的功能最终会由WMS来进行处理。

## WindowManager结构

WindowManager是一个接口类，继承自接口ViewManager，ViewManager中定义了三个方法，分别用来添加、更新和删除View：

frameworks/base/core/java/android/view/ViewManager.java

```
public interface ViewManager {
    public void addView(View view, ViewGroup.LayoutParams params);
    public void updateViewLayout(View view, ViewGroup.LayoutParams params);
    public void removeView(View view);
}

```

WindowManager也继承了这些方法，同时又加入很多功能，包括Window的类型和层级相关的常量、内部类以及一些方法，其中有两个方法是根据Window的特性加入的。

frameworks/base/core/java/android/view/WindowManager.java

```
//得到WindowManager所管理的屏幕（Display）
public Display getDefaultDisplay();
//完成传入的View相关的销毁工作
public void removeViewImmediate(View view);

```

可以看到这些方法传入的参数都是View，说明WindowManager具体管理的是以View形式存在的Window。

WindowManager除了增加方法外，还定义了一个静态内部类LayoutParams，这就是Window的属性，了解Window的属性能够更好的理解WMS的内部原理，下面来看看Window的属性。

## Window的属性

Window的属性有很多种，与应用开发最密切的有三种，它们分别是Type(Window的类型)、Flag(Window的标志)和SoftInputMode（软键盘相关模式），下面分别介绍这三种Window的属性。

### Window的类型

Window的类型有很多种，比如应用程序窗口、系统错误窗口、输入法窗口、PopupWindow、Toast、Dialog等等。总的来说分为三大类：Application Window（应用程序窗口）、Sub Windwow（子窗口）、System Window（系统窗口），每个大类又包含了很多种类型，它们都定义在WindowManager的静态内部类LayoutParams中。

**（1）应用程序窗口**

Activity就是一个典型的应用程序窗口，应用程序窗口包含的类型：

```
public static final int FIRST_APPLICATION_WINDOW = 1;//应用程序窗口类型初始值
public static final int TYPE_BASE_APPLICATION   = 1;//窗口的基础值，其他的窗口值要大于这个值
public static final int TYPE_APPLICATION        = 2;//普通的应用程序窗口类型
public static final int TYPE_APPLICATION_STARTING = 3;//应用程序启动窗口类型，用于系统在应用程序窗口启动前显示的窗口。
public static final int TYPE_DRAWN_APPLICATION = 4;
public static final int LAST_APPLICATION_WINDOW = 99;//应用程序窗口类型结束值

```

应用程序窗口的初始值是1，结束值是99，也就是说应用程序窗口的Type值范围为1到99，这个数值的大小涉及到窗口的层级，一般情况下，Type值越大则Z-Oder排序越靠前，就越靠近用户，在频幕越上层。

**（2）子窗口**

子窗口，顾名思义，它不能独立的存在，需要附着在其他窗口才可以，PopupWindow就属于子窗口。子窗口的类型定义如下：

```
public static final int FIRST_SUB_WINDOW = 1000;//子窗口类型初始值
 public static final int TYPE_APPLICATION_PANEL = FIRST_SUB_WINDOW;
 public static final int TYPE_APPLICATION_MEDIA = FIRST_SUB_WINDOW + 1;
 public static final int TYPE_APPLICATION_SUB_PANEL = FIRST_SUB_WINDOW + 2;
 public static final int TYPE_APPLICATION_ATTACHED_DIALOG = FIRST_SUB_WINDOW + 3;
 public static final int TYPE_APPLICATION_MEDIA_OVERLAY  = FIRST_SUB_WINDOW + 4; 
 public static final int TYPE_APPLICATION_ABOVE_SUB_PANEL = FIRST_SUB_WINDOW + 5;
 public static final int LAST_SUB_WINDOW = 1999;//子窗口类型结束值

```

子窗口的Type值范围为1000到1999。

**（3）系统窗口**

Toast、输入法窗口、系统音量条窗口、系统错误窗口都属于系统窗口。系统窗口的类型定义如下：

```
public static final int FIRST_SYSTEM_WINDOW     = 2000;//系统窗口类型初始值
 public static final int TYPE_STATUS_BAR         = FIRST_SYSTEM_WINDOW;//系统状态栏窗口
 public static final int TYPE_SEARCH_BAR         = FIRST_SYSTEM_WINDOW+1;//搜索条窗口
 public static final int TYPE_PHONE              = FIRST_SYSTEM_WINDOW+2;//通话窗口
 public static final int TYPE_SYSTEM_ALERT       = FIRST_SYSTEM_WINDOW+3;//系统ALERT窗口
 public static final int TYPE_KEYGUARD           = FIRST_SYSTEM_WINDOW+4;//锁屏窗口
 public static final int TYPE_TOAST              = FIRST_SYSTEM_WINDOW+5;//TOAST窗口
 ...

 public static final int LAST_SYSTEM_WINDOW      = 2999;//系统窗口类型结束值

```

系统窗口的Type值范围为2000到2999。

## Window的标志

Window的标志也就是Flag，用于控制Window的显示，同样被定义在WindowManager的内部类LayoutParams中，一共有20多个，这里我们给出几个比较常用的。

```
// 只要窗口可见，就允许在开启状态的屏幕上锁屏. 
public static final int FLAG_ALLOW_LOCK_WHILE_SCREEN_ON     = 0x00000001;
//窗口不能获得输入焦点，设置该标志的同时，FLAG_NOT_TOUCH_MODAL也会被设置
public static final int FLAG_NOT_FOCUSABLE      = 0x00000008;
//窗口不接收任何触摸事件
public static final int FLAG_NOT_TOUCHABLE      = 0x00000010;
//在该窗口区域外的触摸事件传递给其他的Window,而自己只会处理窗口区域内的触摸事件
public static final int FLAG_NOT_TOUCH_MODAL    = 0x00000020;
//只要窗口可见，屏幕就会一直亮着
public static final int FLAG_KEEP_SCREEN_ON     = 0x00000080;
//允许窗口超过屏幕之外
public static final int FLAG_LAYOUT_NO_LIMITS   = 0x00000200;
//隐藏所有的屏幕装饰窗口，比如在游戏、播放器中的全屏显示
public static final int FLAG_FULLSCREEN      = 0x00000400;
//窗口可以在锁屏的窗口之上显示
public static final int FLAG_SHOW_WHEN_LOCKED = 0x00080000;
//当用户的脸贴近屏幕时（比如打电话），不会去响应此事件
public static final int FLAG_IGNORE_CHEEK_PRESSES    = 0x00008000;
//窗口显示时将屏幕点亮
public static final int FLAG_TURN_SCREEN_ON = 0x00200000;

```

设置Window的Flag有三种方法，第一种是通过Window的addFlags方法：

```
Window mWindow =getWindow(); 
mWindow.addFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN);

```

第二种通过Window的setFlags方法，其实内部还是会调用setFlags方法:

```
Window mWindow =getWindow();            
mWindow.setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,WindowManager.LayoutParams.FLAG_FULLSCREEN);

```

第三种则是给LayoutParams设置Flag，并通过WindowManager的addView方法进行添加：

```
WindowManager.LayoutParams mWindowLayoutParams = new WindowManager.LayoutParams();
mWindowLayoutParams.flags = WindowManager.LayoutParams.FLAG_FULLSCREEN;
WindowManager mWindowManager =(WindowManager)getSystemService(Context.WINDOW_SERVICE);  
TextView mTextView=new TextView(this);
mWindowManager.addView(mTextView, mWindowLayoutParams);

```

## 软键盘相关模式

窗口和窗口的叠加是非常常见的场景，但如果其中的窗口是软键盘窗口，可能就会出现一些问题，比如典型的用户登录界面，默认的情况弹出的软键盘窗口可能会盖住输入框，这样用户体验会非常糟糕。

为了使得软键盘窗口能够按照期望来显示，WindowManager的静态内部类LayoutParams中还定义了软键盘相关模式，这里给出常用的几个：

```
//没有指定状态,系统会选择一个合适的状态或依赖于主题的设置
public static final int SOFT_INPUT_STATE_UNSPECIFIED = 0;
//不会改变软键盘状态
public static final int SOFT_INPUT_STATE_UNCHANGED = 1;
//当用户进入该窗口时，软键盘默认隐藏
public static final int SOFT_INPUT_STATE_HIDDEN = 2;
//当窗口获取焦点时，软键盘总是被隐藏
public static final int SOFT_INPUT_STATE_ALWAYS_HIDDEN = 3;
//当软键盘弹出时，窗口会调整大小
public static final int SOFT_INPUT_ADJUST_RESIZE = 0x10;
//当软键盘弹出时，窗口不需要调整大小，要确保输入焦点是可见的
public static final int SOFT_INPUT_ADJUST_PAN = 0x20;

```

从上面给出的SoftInputMode ，可以发现，它们与AndroidManifest中Activity的属性android:windowSoftInputMode是对应的。因此，除了在AndroidMainfest中为Activity设置android:windowSoftInputMode以外还可以在Java代码中为Window设置SoftInputMode：

```
getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_RESIZE);

```

## Window的添加过程

WindowManager对Window的操作最终都是交由WMS来进行处理。窗口的操作分为两大部分，一部分是WindowManager处理部分，另一部分是WMS处理部分。我们知道Window分为三大类，分别是：Application Window（应用程序窗口）、Sub Windwow（子窗口）和System Window（系统窗口），对于不同类型的窗口添加过程会有所不同，但是对于WMS处理部分，添加的过程基本上是一样的。

![](//upload-images.jianshu.io/upload_images/19956127-e191da2437d4cae3.png?imageMogr2/auto-orient/strip|imageView2/2/w/412/format/webp)

无论是哪种窗口，它的添加过程在WMS处理部分中基本是类似的，只不过会在权限和窗口显示次序等方面会有些不同。但是在WindowManager处理部分会有所不同，这里以最典型的应用程序窗口Activity为例。

Activity在启动过程中，会调用ActivityThread的handleLaunchActivity()方法，具体可以参考Framework学习（五）应用程序启动过程这篇文章。

frameworks/base/core/java/android/app/ActivityThread.java

**ActivityThread#handleLaunchActivity()**

```
 private void handleLaunchActivity(ActivityClientRecord r, Intent customIntent, String reason) {
      ...
        Activity a = performLaunchActivity(r, customIntent); //1
        if (a != null) {
            r.createdConfig = new Configuration(mConfiguration);
            reportSizeConfigurations(r);
            Bundle oldState = r.state;
            //2
            handleResumeActivity(r.token, false, r.isForward, !r.activity.mFinished && !r.startsNotResumed, r.lastProcessedSeq, reason); 

            if (!r.activity.mFinished && r.startsNotResumed) {      
                performPauseActivityIfNeeded(r, reason);
                if (r.isPreHoneycomb()) {
                    r.state = oldState;
                }
            }
        } else {
           ...
        }
    }

```

注释1处的performLaunchActivity方法用来启动Activity。
注释2处的代码用来添加Activity的window窗口。

**ActivityThread#performLaunchActivity()**

```
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
      ...
        //1
        activity.attach(appContext, this, getInstrumentation(), r.token, r.ident, app, r.intent, r.activityInfo, title, r.parent, r.embeddedID, r.lastNonConfigurationInstances, config, r.referrer, r.voiceInteractor, window);
     ...           
        return activity;
}    

```

注释1调用Activity的attach方法。

frameworks/base/core/java/android/app/Activity.java

**Activity#attach()**

```
final void attach(Context context, ActivityThread aThread,
            Instrumentation instr, IBinder token, int ident,
            Application application, Intent intent, ActivityInfo info,
            CharSequence title, Activity parent, String id,
            NonConfigurationInstances lastNonConfigurationInstances,
            Configuration config, String referrer, IVoiceInteractor voiceInteractor,
            Window window) {
        attachBaseContext(context);
        mFragments.attachHost(null /*parent*/);
        mWindow = new PhoneWindow(this, window); //1
        ...
        //2
        mWindow.setWindowManager(
                (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
                mToken, mComponent.flattenToString(),
                (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);
      ...
}      

```

注释1处创建了PhoneWindow。
注释2处调用了PhoneWindow的setWindowManager方法，这个方法的具体的实现在PhoneWindow的父类Window中。

frameworks/base/core/java/android/view/Window.java

**Window#setWindowManager()**

```
public void setWindowManager(WindowManager wm, IBinder appToken, String appName,
        boolean hardwareAccelerated) {
    mAppToken = appToken;
    mAppName = appName;
    mHardwareAccelerated = hardwareAccelerated
            || SystemProperties.getBoolean(PROPERTY_HARDWARE_UI, false);
    if (wm == null) {
        wm = (WindowManager)mContext.getSystemService(Context.WINDOW_SERVICE);//1
    }
    mWindowManager = ((WindowManagerImpl)wm).createLocalWindowManager(this);//2
}

```

注释1处如果传入的WindowManager为null，就会调用Context的getSystemService方法，并传入服务的名称Context.WINDOW_SERVICE（”window”），Context的实现类是ContextImpl，具体的实现在ContextImpl中。

frameworks/base/core/java/android/app/ContextImpl.java

**ContextImpl#getSystemService()**

```
    @Override
    public Object getSystemService(String name) {
        return SystemServiceRegistry.getSystemService(this, name);
    }

```

frameworks/base/core/java/android/app/SystemServiceRegistry.java

**SystemServiceRegistry#getSystemService()**

```
    private static final HashMap<String, ServiceFetcher<?>> SYSTEM_SERVICE_FETCHERS = new HashMap<String, ServiceFetcher<?>>();
    ...
    public static Object getSystemService(ContextImpl ctx, String name) {
        ServiceFetcher<?> fetcher = SYSTEM_SERVICE_FETCHERS.get(name);
        return fetcher != null ? fetcher.getService(ctx) : null;
    }

```

SYSTEM_SERVICE_FETCHERS是一个HashMap类型的数据，它是用来存储服务的，这里是取得地方，那存的地方在哪里呢？找啊找，终于在SystemServiceRegistry 的静态代码块中找到了。

```
final class SystemServiceRegistry {
...
 private SystemServiceRegistry() { }
 static {
 ...
           registerService(Context.WINDOW_SERVICE, WindowManager.class,
                new CachedServiceFetcher<WindowManager>() {
            @Override
            public WindowManager createService(ContextImpl ctx) {
                return new WindowManagerImpl(ctx);
            }});
...
 }
}

```

SystemServiceRegistry#registerService()

```
    private static <T> void registerService(String serviceName, Class<T> serviceClass,
            ServiceFetcher<T> serviceFetcher) {
        SYSTEM_SERVICE_NAMES.put(serviceClass, serviceName);
        SYSTEM_SERVICE_FETCHERS.put(serviceName, serviceFetcher); //1
    }

```

注释1处就是将创建的服务添加到SYSTEM_SERVICE_FETCHERS中的。
从上面代码可以看出，传入的Context.WINDOW_SERVICE对应的就是WindowManagerImpl实例，因此得出结论，Context的getSystemService方法得到的是WindowManagerImpl实例。我们再回到Window的setWindowManager方法，在注释1处得到WindowManagerImpl实例后转为WindowManager类型，在注释2处调用了WindowManagerImpl的createLocalWindowManager方法。

frameworks/base/core/java/android/view/WindowManagerImpl

**WindowManagerImpl#createLocalWindowManager()**

```
public WindowManagerImpl createLocalWindowManager(Window parentWindow) {
       return new WindowManagerImpl(mContext, parentWindow);
   }

```

createLocalWindowManager方法同样也是创建WindowManagerImpl，不同的是这次创建WindowManagerImpl时将创建它的Window作为参数传了进来，这样WindowManagerImpl就持有了Window的引用，就可以对Window进行操作。比如
在Window中添加View，来查看WindowManagerImpl的addView方法。

**WindowManagerImpl#addView()**

```
    @Override
    public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
        applyDefaultToken(params);
        mGlobal.addView(view, params, mContext.getDisplay(), mParentWindow); //1
    }

```

注释1处调用了WindowManagerGlobal的addView方法，其中最后一个参数mParentWindow就是Window，可以看出WindowManagerImpl虽然是WindowManager的实现类，但是却没有实现什么功能，而是将功能实现委托给了WindowManagerGlobal。

来查看WindowManagerImpl中如何定义的WindowManagerGlobal：

```
public final class WindowManagerImpl implements WindowManager {
    private final WindowManagerGlobal mGlobal = WindowManagerGlobal.getInstance(); //1
    private final Context mContext;
    private final Window mParentWindow; //2
...
  private WindowManagerImpl(Context context, Window parentWindow) {
        mContext = context;
        mParentWindow = parentWindow;
    }
 ...   
}

```

注释1可以看出WindowManagerGlobal是一个单例，说明在一个进程中只有一个WindowManagerGlobal实例。
注释2处说明WindowManagerImpl可能会实现多个Window，也就是说在一个进程中WindowManagerImpl可能会有多个实例。

Window和WindowManager的关系如下图所示。

![](//upload-images.jianshu.io/upload_images/19956127-327d1aa0441dce1e.png?imageMogr2/auto-orient/strip|imageView2/2/w/872/format/webp)

PhoneWindow继承自Window，Window通过setWindowManager方法与WindowManager发生关联。WindowManager继承自接口ViewManager，WindowManagerImpl是WindowManager接口的实现类，但是具体的功能都会委托给WindowManagerGlobal来实现。

再次回到最初的ActivityThread#handleLaunchActivity()方法。注释2处调用了handleResumeActivity。

**ActivityThread#handleResumeActivity()**

```
 final void handleResumeActivity(IBinder token,
            boolean clearHide, boolean isForward, boolean reallyResume, int seq, String reason) {
       ...   
         r = performResumeActivity(token, clearHide, reason);//1           
  ...
 if (r.window == null && !a.mFinished && willBeVisible) {
                r.window = r.activity.getWindow();
                View decor = r.window.getDecorView();
                decor.setVisibility(View.INVISIBLE);
                ViewManager wm = a.getWindowManager();//2
                WindowManager.LayoutParams l = r.window.getAttributes();
                a.mDecor = decor;
                l.type = WindowManager.LayoutParams.TYPE_BASE_APPLICATION;
                l.softInputMode |= forwardBit;
                if (r.mPreserveWindow) {
                    a.mWindowAdded = true;
                    r.mPreserveWindow = false;
                    ViewRootImpl impl = decor.getViewRootImpl();
                    if (impl != null) {
                        impl.notifyChildRebuilt();
                    }
                }
                if (a.mVisibleFromClient && !a.mWindowAdded) {
                    a.mWindowAdded = true;
                    wm.addView(decor, l);//3
                }
...                
}

```

注释1处的performResumeActivity方法最终会调用Activity的onResume方法。
注释2处得到ViewManager类型的wm对象。
注释3处调用了wm的addView方法，而addView方法的实现则是在WindowManagerImpl中，最终调用的是WindowManagerGlobal的addView方法，唯一需要注意的是addView的第一个参数是DecorView。

frameworks/base/core/java/android/view/WindowManagerGlobal.java

**WindowManagerGlobal#addView()**

```
    public void addView(View view, ViewGroup.LayoutParams params,
            Display display, Window parentWindow) {
        /**参数检查*/
        if (view == null) {
            throw new IllegalArgumentException("view must not be null");
        }
        if (display == null) {
            throw new IllegalArgumentException("display must not be null");
        }
        if (!(params instanceof WindowManager.LayoutParams)) {
            throw new IllegalArgumentException("Params must be WindowManager.LayoutParams");
        }

        final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams) params;
        if (parentWindow != null) {
            parentWindow.adjustLayoutParamsForSubWindow(wparams); //1
        } else {
            ...
        }

        ViewRootImpl root;
        View panelParentView = null;

        ...

            root = new ViewRootImpl(view.getContext(), display); //2

            view.setLayoutParams(wparams);

            mViews.add(view);
            mRoots.add(root); //3
            mParams.add(wparams);
        }

        try {
            root.setView(view, wparams, panelParentView); //4
        } catch (RuntimeException e) {
            ...
            throw e;
        }
    }

```

首先会对参数view、params和display进行检查。
注释1处，如果当前窗口要作为子窗口，就会根据父窗口对子窗口的WindowManager.LayoutParams类型的wparams对象进行相应调整。
注释2处创建了ViewRootImp并赋值给root.
注释3处将root存入到ArrayList<\ViewRootImpl>类型的mRoots中，除了mRoots，mViews和mParams也是ArrayList类型的，分别用于存储窗口的view对象和WindowManager.LayoutParams类型的wparams对象。
注释4处调用了ViewRootImpl的setView方法。

**ViewRootImpl身负了很多职责：**

View树的根并管理View树
触发View的测量、布局和绘制
输入事件的中转站
管理Surface
负责与WMS进行进程间通信
frameworks/base/core/java/android/view/ViewRootImpl.java

**ViewRootImpl#setView()**

```
public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView) {
        synchronized (this) {
           ...
                try {
                    mOrigWindowType = mWindowAttributes.type;
                    mAttachInfo.mRecomputeGlobalAttributes = true;
                    collectViewAttributes();
                    //1
                    res = mWindowSession.addToDisplay(mWindow, mSeq, mWindowAttributes,
                            getHostVisibility(), mDisplay.getDisplayId(),
                            mAttachInfo.mContentInsets, mAttachInfo.mStableInsets,
                            mAttachInfo.mOutsets, mInputChannel);
                } 
                ...
    }

```

注释1处mWindowSession是IWindowSession对象。在创建ViewRootImpl对象时被实例化。

```
public ViewRootImpl(Context context, Display display) {
        mWindowSession = WindowManagerGlobal.getWindowSession();
        ...
    }

```

frameworks/base/core/Java/Android/view/WindowManagerGlobal.java

**WindowManagerGlobal#getWindowSession()**

```
public static IWindowSession getWindowSession() {
        synchronized (WindowManagerGlobal.class) {
            if (sWindowSession == null) {
                try {
                    InputMethodManager imm = InputMethodManager.getInstance();
                    IWindowManager windowManager = getWindowManagerService(); //1
                    sWindowSession = windowManager.openSession(  //2
                            new IWindowSessionCallback.Stub() {
                                @Override
                                public void onAnimatorScaleChanged(float scale) {
                                    ValueAnimator.setDurationScale(scale);
                                }
                            },
                            imm.getClient(), imm.getInputContext());
                } catch (RemoteException e) {
                    Log.e(TAG, "Failed to open window session", e);
                }
            }
            return sWindowSession;
        }
    }

```

注释1这里getWindowManagerService()通过AIDL返回WindowManagerService实例。
注释2调用WindowManagerService#openSession()。

frameworks/base/services/java/com/android/server/wm/WindowManagerService.java

**WindowManagerService#openSession()**

```
public IWindowSession openSession(IWindowSessionCallback callback, IInputMethodClient client,
            IInputContext inputContext) {
        if (client == null) throw new IllegalArgumentException("null client");
        if (inputContext == null) throw new IllegalArgumentException("null inputContext");
        Session session = new Session(this, callback, client, inputContext);
        return session;
    }

```

返回一个Session对象。也就是说在ViewRootImpl#setView()中调用的是Session#addToDisplay()。

继续回到ViewRootImpl#setView()方法，总结一下：
注释1主要就是调用了mWindowSession的addToDisplay方法。
mWindowSession是IWindowSession类型的，它是一个Binder对象，用于进行进程间通信，IWindowSession是Client端的代理，它的Server端的实现为Session，此前包含ViewRootImpl在内的代码逻辑都是运行在本地进程的，而Session的addToDisplay方法则运行在WMS所在的进程。

frameworks/base/services/core/java/com/android/server/wm/Session.java

**Session#addToDisplay()**

```
@Override
    public int addToDisplay(IWindow window, int seq, WindowManager.LayoutParams attrs,
            int viewVisibility, int displayId, Rect outContentInsets, Rect outStableInsets,
            Rect outOutsets, InputChannel outInputChannel) {
        return mService.addWindow(this, window, seq, attrs, viewVisibility, displayId,
                outContentInsets, outStableInsets, outOutsets, outInputChannel);
    }

```

addToDisplay方法中会调用了WMS的addWindow方法，并将自身也就是Session，作为参数传了进去，每个应用程序进程都会对应一个Session，WMS会用ArrayList来保存这些Session。这样剩下的工作就交给WMS来处理，在WMS中会为这个添加的窗口分配Surface，并确定窗口显示次序，可见负责显示界面的是画布Surface，而不是窗口本身。WMS会将它所管理的Surface交由SurfaceFlinger处理，SurfaceFlinger会将这些Surface混合并绘制到屏幕上。

后面是WMS中的处理了，接下来我会有专门的文章分析WMS中添加window的流程。

下面是本文的时序图：

![](//upload-images.jianshu.io/upload_images/19956127-fc0efe3b33c556a3.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

>原文作者：huaxun66
原文链接：[https://blog.csdn.net/huaxun66/article/details/78354893](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fhuaxun66%2Farticle%2Fdetails%2F78354893)


**更多系列教程GitHub学习地址：[https://github.com/Timdk857/Android-Architecture-knowledge-2-](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FTimdk857%2FAndroid-Architecture-knowledge-2-)**
