相信绝大部分读者对ActivityManagerService（简称AMS）都有所耳闻。AMS是Android中最核心的服务，主要负责系统中四大组件的启动、切换、调度及应用进程的管理和调度等工作，其职责与操作系统中的进程管理和调度模块相类似，因此它在Android中非常重要。

## AMS启动流程

在Framework学习（三）SystemServer进程启动过程这篇文章我们简单介绍过SystemServer启动AMS的过程。

先来查看SystemServer的main函数：

frameworks/base/services/java/com/android/server/SystemServer.java

**SystemServer#main()**

```
public static void main(String[] args) {
        new SystemServer().run(); 
    }

```

**SystemServer#run()**

```
private void run() {
       ...
           System.loadLibrary("android_servers");//1
       ...
           mSystemServiceManager = new SystemServiceManager(mSystemContext);//2
           LocalServices.addService(SystemServiceManager.class, mSystemServiceManager);
       ...    
        try {
           Trace.traceBegin(Trace.TRACE_TAG_SYSTEM_SERVER, "StartServices");
           startBootstrapServices();//3
           startCoreServices();//4
           startOtherServices();//5
       } catch (Throwable ex) {
           Slog.e("System", "******************************************");
           Slog.e("System", "************ Failure starting system services", ex);
           throw ex;
       } finally {
           Trace.traceEnd(Trace.TRACE_TAG_SYSTEM_SERVER);
       }
       ...
   }

```

注释1处加载了libandroid_servers.so。
注释2处创建SystemServiceManager，它会对系统的服务进行创建、启动和生命周期管理。启动系统的各种服务。
注释3中的startBootstrapServices函数中用SystemServiceManager启动了ActivityManagerService、PowerManagerService、PackageManagerService等服务。
注释4处的函数中则启动了BatteryService、UsageStatsService和WebViewUpdateService。
注释5处的startOtherServices函数中则启动了CameraService、AlarmManagerService、VrManagerService等服务，这些服务的父类为SystemService。

**SystemServer#startBootstrapServices()**

```
private void startBootstrapServices() {
     Installer installer = mSystemServiceManager.startService(Installer.class);
     // Activity manager runs the show.
     mActivityManagerService = mSystemServiceManager.startService(
             ActivityManagerService.Lifecycle.class).getService();//1
     mActivityManagerService.setSystemServiceManager(mSystemServiceManager);
     mActivityManagerService.setInstaller(installer);
   ...

   mActivityManagerService.setSystemProcess(); //2
 }

```

注释1处调用了SystemServiceManager的startService方法，方法的参数是ActivityManagerService.Lifecycle.class。
注释2调用了ActivityManagerService的setSystemProcess方法。

frameworks/base/services/core/java/com/android/server/SystemServiceManager.java

**SystemServiceManager#startService()**

```
@SuppressWarnings("unchecked")
  public <T extends SystemService> T startService(Class<T> serviceClass) {
      try {
         ...
          final T service;
          try {
              Constructor<T> constructor = serviceClass.getConstructor(Context.class);//1
              service = constructor.newInstance(mContext);//2
          } catch (InstantiationException ex) {
            ...
          }
          // Register it.
          mServices.add(service);//3
          // Start it.
          try {
              service.onStart();//4
          } catch (RuntimeException ex) {
              throw new RuntimeException("Failed to start service " + name
                      + ": onStart threw an exception", ex);
          }
          return service;
      } finally {
          Trace.traceEnd(Trace.TRACE_TAG_SYSTEM_SERVER);
      }
  }

```

startService方法传入的参数是Lifecycle.class，Lifecycle继承自SystemService。
注释1处得到传进来的Lifecycle的构造器constructor。
注释2处通过反射来创建Lifecycle类型的service对象。
注释3处将刚创建的service添加到ArrayList类型的mServices对象中来完成注册。
注释4处调用service的onStart方法来启动service，并返回该service。

**Lifecycle**

```
public static final class Lifecycle extends SystemService {
     private final ActivityManagerService mService;
     public Lifecycle(Context context) {
         super(context);
         mService = new ActivityManagerService(context);//1
     }
     @Override
     public void onStart() {
         mService.start();//2
     }
     public ActivityManagerService getService() {
         return mService;//3
     }
 }

```

上面的代码结合SystemServiceManager的startService方法来分析，当通过反射来创建Lifecycle实例时，会调用注释1处的方法创建AMS实例，当调用Lifecycle类型的service的onStart方法时，实际上是调用了注释2处AMS的start方法。

在SystemServer的startBootstrapServices方法的注释1处，调用了如下代码：

```
mActivityManagerService = mSystemServiceManager.startService(
               ActivityManagerService.Lifecycle.class).getService();

```

我们知道SystemServiceManager的startService方法最终会返回Lifecycle类型的对象，紧接着又调用了Lifecycle的getService方法，这个方法会返回AMS类型的mService对象，见注释3处，这样AMS实例就会被创建并且返回。

继续来看看SystemServer中startBootstrapServices()方法的注释2：

```
    public void setSystemProcess() {
        try {
            ServiceManager.addService(Context.ACTIVITY_SERVICE, this, true); //1
            ServiceManager.addService(ProcessStats.SERVICE_NAME, mProcessStats);
            ServiceManager.addService("meminfo", new MemBinder(this));
            ServiceManager.addService("gfxinfo", new GraphicsBinder(this));
            ...
}
        } catch (PackageManager.NameNotFoundException e) {
            throw new RuntimeException(
                    "Unable to find android system package", e);
        }
    }

```

注释1中的Context.ACTIVITY_SERVICE=“activity”，注释1就是把ActivityManagerService实例注册到ServiceManager中。ServiceManager用来管理系统中的各种Service，用系统C/S架构中的Binder机制通信，Client端要使用某个Service，则需要先到ServiceManager查询Service的相关信息，然后根据Service的相关信息与Service所在的Server进程建立通讯通路，这样Client端就可以使用Service了。

```
IBinder b = ServiceManager.getService("activity"); //查询

```

## AMS与应用进程启动

在Framework学习（二）Zygote进程启动过程这篇文章中，我们说过Zygote进程会创建一个服务器端Socket，并监听Socket来等待客户端请求创建新的应用程序进程。要启动一个应用程序，首先要保证这个应用程序的进程已经被启动。AMS在启动应用程序时会检查这个应用程序的进程是否存在，不存在就会请求Zygote进程将应用程序进程启动。

在Framework学习（五）应用程序启动过程这篇文章，我们说过应用程序启动时会走到ActivityStackSupervisor的startSpecificActivityLocked函数启动指定的ActivityRecored。

frameworks/base/services/core/java/com/android/server/am/ActivityStackSupervisor.java

**ActivityStackSupervisor#startSpecificActivityLocked()**

```
void startSpecificActivityLocked(ActivityRecord r, boolean andResume, boolean checkConfig) {
        //1
        ProcessRecord app = mService.getProcessRecordLocked(r.processName, r.info.applicationInfo.uid, true);
        r.task.stack.setLaunchTime(r);
        if (app != null && app.thread != null) { 
            try {
                if ((r.info.flags&ActivityInfo.FLAG_MULTIPROCESS) == 0 || !"android".equals(r.info.packageName)) {
                    app.addPackage(r.info.packageName, r.info.applicationInfo.versionCode, mService.mProcessStats);
                }
                realStartActivityLocked(r, app, andResume, checkConfig);  //2
                return;
            } catch (RemoteException e) {
                Slog.w(TAG, "Exception when starting activity "  + r.intent.getComponent().flattenToShortString(), e);
            }
        } 
        //3
        mService.startProcessLocked(r.processName, r.info.applicationInfo, true, 0, "activity", r.intent.getComponent(), false, false, true);
    }

```

**ActivityStackSupervisor#getProcessRecordLocked()**

```
final ProcessRecord getProcessRecordLocked(String processName, int uid, boolean keepIfLarge) {
        if (uid == Process.SYSTEM_UID) {
            // The system gets to run in any process.  If there are multiple
            // processes with the same uid, just pick the first (this
            // should never happen).
            SparseArray<ProcessRecord> procs = mProcessNames.getMap().get(processName);
            if (procs == null) return null;
            ...
        }
        ...
    }

```

注释1处获取当前Activity所在的进程的ProcessRecord，如果进程已经启动了，会执行注释2处的代码。否则执行注释3的代码。
注释2处调用realStartActivityLocked来启动应用程序。
注释3处调用AMS的startProcessLocked来启动应用程序进程。关于应用程序进程的启动我们可以看Framework学习（六）应用程序进程启动过程这篇文章。

## AMS家族

ActivityManager是一个和AMS相关联的类，它主要是对运行中的Activity进行管理，这些管理工作并不是由ActivityManager自己来处理，而是交由AMS来处理，ActivityManager中的方法会通过ActivityManagerNative（以后简称AMN）的getDefault方法来得到ActivityManagerProxy(以后简称AMP)，通过AMP就可以和AMN进行通信，而AMN是一个抽象类，它会将功能交由它的子类AMS来处理，因此，AMP就是AMS的代理类。AMS作为系统核心服务，很多API是不会暴露给ActivityManager的，因此ActivityManager并不算是AMS家族一份子。

下面以Activity启动的例子看一下AMS家族的调用。Framework学习（五）应用程序启动过程一文中我们简单讲过AMS中Binder机制，本文我们继续总结一下。

Activity启动过程中会调用Instrumentation的execStartActivity方法。

frameworks/base/core/java/android/app/Instrumentation.java

**Instrumentation#execStartActivity()**

```
public ActivityResult execStartActivity(Context who, IBinder contextThread, IBinder token, Activity target, Intent intent, int requestCode, Bundle options) {
        ...
        try {
            intent.migrateExtraStreamToClipData();
            intent.prepareToLeaveProcess(who);
            //1
            int result = ActivityManagerNative.getDefault().startActivity(whoThread, who.getBasePackageName(), intent, intent.resolveTypeIfNeeded(who.getContentResolver()), token, target != null ? target.mEmbeddedID : null, requestCode, 0, null, options);
            checkStartActivityResult(result, intent);
        } catch (RemoteException e) {
            throw new RuntimeException("Failure from system", e);
        }
        return null;
    }

```

注释1调用ActivityManagerNative的getDefault就是获取AMS的代理对象AMP的，接着调用它的startActivity方法。

frameworks/base/core/java/android/app/ActivityManagerNative.java

**ActivityManagerNative.getDefault()**

```
 static public IActivityManager getDefault() {
        return gDefault.get();
    }

    private static final Singleton<IActivityManager> gDefault = new Singleton<IActivityManager>() {
        protected IActivityManager create() {
            IBinder b = ServiceManager.getService("activity"); //1
            IActivityManager am = asInterface(b); //2
            return am;
        }
    };

    static public IActivityManager asInterface(IBinder obj) {
        if (obj == null) {
            return null;
        }
        IActivityManager in =
            (IActivityManager)obj.queryLocalInterface(descriptor);
        if (in != null) {
            return in;
        }

        return new ActivityManagerProxy(obj);
    }

```

getDefault方法调用了gDefault的get方法，gDefault 是一个Singleton类。注释1处从ServiceManager得到名为”activity”的Service代理对象，也就是AMS的代理对象。
注释2处将它封装成AMP类型对象，并将它保存到gDefault中，此后调用AMN的getDefault方法就会直接获得AMS的代理AMP对象。

**ActivityManagerProxy**

```
class ActivityManagerProxy implements IActivityManager
{
    public ActivityManagerProxy(IBinder remote)
    {
        mRemote = remote;
    }

    public IBinder asBinder()
    {
        return mRemote;
    }
    ...
}

```

AMP的构造方法中将AMS的引用赋值给变量mRemote ，这样在AMP中就可以使用AMS了。
其中IActivityManager是一个接口，AMN和AMP都实现了这个接口，用于实现代理模式和Binder通信。

再回到Instrumentation的execStartActivity方法，来查看AMP的startActivity方法，AMP是AMN的内部类，代码如下所示。

**ActivityManagerProxy#startActivity()**

```
public int startActivity(IApplicationThread caller, String callingPackage, Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode, int startFlags, ProfilerInfo profilerInfo, Bundle options) throws RemoteException {
        Parcel data = Parcel.obtain();
        Parcel reply = Parcel.obtain();
        data.writeInterfaceToken(IActivityManager.descriptor);
        data.writeStrongBinder(caller != null ? caller.asBinder() : null);
        data.writeString(callingPackage);
        intent.writeToParcel(data, 0);
        data.writeString(resolvedType);
        data.writeStrongBinder(resultTo);
        data.writeString(resultWho);
        data.writeInt(requestCode);
        data.writeInt(startFlags);
        if (profilerInfo != null) {
            data.writeInt(1);
            profilerInfo.writeToParcel(data, Parcelable.PARCELABLE_WRITE_RETURN_VALUE);
        } else {
            data.writeInt(0);
        }
        if (options != null) {
            data.writeInt(1);
            options.writeToParcel(data, 0);
        } else {
            data.writeInt(0);
        }
        mRemote.transact(START_ACTIVITY_TRANSACTION, data, reply, 0); //1
        reply.readException();
        int result = reply.readInt();
        reply.recycle();
        data.recycle();
        return result;
    }

```

首先会将传入的参数写入到Parcel类型的data中。在注释1处通过IBinder对象mRemote向AMN发送一个START_ACTIVITY_TRANSACTION类型的进程间通信请求。那么服务端AMN就会从Binder线程池中读取我们客户端发来的数据，最终会调用AMN的onTransact方法中执行。

**ActivityManagerNative#onTransact()**

```
@Override
    public boolean onTransact(int code, Parcel data, Parcel reply, int flags) throws RemoteException {
        switch (code) {
        case START_ACTIVITY_TRANSACTION:
        {
    ...
            int result = startActivity(app, callingPackage, intent, resolvedType, resultTo, resultWho, requestCode, startFlags, profilerInfo, options);  //1
            reply.writeNoException();
            reply.writeInt(result);
            return true;
        }
    } 

```

因为AMS继承了AMN，服务端真正的实现是在AMS中，注释1最终会调用AMS的startActivity方法。

frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

**ActivityManagerService#startActivity()**

```
@Override
public final int startActivity(IApplicationThread caller, String callingPackage,
        Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
        int startFlags, ProfilerInfo profilerInfo, Bundle bOptions) {
    return startActivityAsUser(caller, callingPackage, intent, resolvedType, resultTo,
            resultWho, requestCode, startFlags, profilerInfo, bOptions,
            UserHandle.getCallingUserId());
}

```

上面的调用过程是一个典型的Binder通信机制，Instrumentation和AMS通过Binder进程间通信实现交互。

在Activity的启动过程中提到了AMP、AMN和AMS，它们共同组成了AMS家族的主要部分：

![image](//upload-images.jianshu.io/upload_images/19956127-9fbd271fbae2a5f1.png?imageMogr2/auto-orient/strip|imageView2/2/w/951/format/webp)

AMP是AMN的内部类，它们都实现了IActivityManager接口，这样它们就可以实现代理模式，具体来讲是远程代理：AMP和AMN是运行在两个进程的，AMP是Client端，AMN则是Server端，而Server端中具体的功能都是由AMN的子类AMS来实现的，因此，AMP就是AMS在Client端的代理类。AMN又实现了Binder类，这样AMP就可以和AMS通过Binder来进行进程间通信。

ActivityManager通过AMN的getDefault方法得到AMP，通过AMP就可以和AMN进行通信，也就是间接的与AMS进行通信。除了ActivityManager，其他想要与AMS进行通信的类都需要通过AMP，如下图所示。

![image](//upload-images.jianshu.io/upload_images/19956127-985caa21243eb49b.png?imageMogr2/auto-orient/strip|imageView2/2/w/877/format/webp)

**推荐阅读：[做了六年Android，终于熬出头了，15K到31K全靠这份高级面试题+解析](https://www.jianshu.com/p/14fa0c6ed6e8)**

>原文作者：huaxun66
原文链接：[https://blog.csdn.net/huaxun66/article/details/78169337](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fhuaxun66%2Farticle%2Fdetails%2F78169337)

更多系列教程GitHub学习地址：[https://github.com/Timdk857/Android-Architecture-knowledge-2-](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FTimdk857%2FAndroid-Architecture-knowledge-2-)

