之前的文章Android Framework学习笔记（五）应用程序启动过程
我们讲解过了应用程序（Activity）的启动过程，本篇我们来看看Service的启动/绑定过程。

## Service的启动过程

## ContextImpl请求AMS

要启动Service，我们会调用startService方法，它的实现在ContextWrapper中。

frameworks/base/core/java/android/content/ContextWrapper.java

ContextWrapper#startService()

```
    Context mBase;

    @Override
    public ComponentName startService(Intent service) {
        return mBase.startService(service); //1
    }

```

注释1会调用mBase的startService方法。那么Context类型的mBase代表什么呢？
在Framework学习（五）应用程序启动过程一文讲过应用程序启动过程中会调用ActivityThread的performLaunchActivity()函数。

frameworks/base/core/java/android/app/ActivityThread.java

**ActivityThread#performLaunchActivity()**

```
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
        ...
            if (activity != null) {
                Context appContext = createBaseContextForActivity(r, activity); //1
         ...
                }
                activity.attach(appContext, this, getInstrumentation(), r.token, r.ident, app, r.intent, r.activityInfo, title, r.parent, r.embeddedID, r.lastNonConfigurationInstances, config, r.referrer, r.voiceInteractor, window); //2

              ...
        return activity;
}        

```

注释1用来创建要启动Activity的上下文环境appContext。
注释2调用Activity的attach方法，将Activity与上下文关联起来。

**ActivityThread#createBaseContextForActivity()**

```
private Context createBaseContextForActivity(ActivityClientRecord r, final Activity activity) {
...
    ContextImpl appContext = ContextImpl.createActivityContext(
            this, r.packageInfo, r.token, displayId, r.overrideConfig); //1
    appContext.setOuterContext(activity);
    Context baseContext = appContext;
    ...
    return baseContext;
}

    static ContextImpl createActivityContext(ActivityThread mainThread,
            LoadedApk packageInfo, IBinder activityToken, int displayId,
            Configuration overrideConfiguration) {
        if (packageInfo == null) throw new IllegalArgumentException("packageInfo");
        return new ContextImpl(null, mainThread, packageInfo, activityToken, null, 0,
                null, overrideConfiguration, displayId);
    }

```

注释1处的上下文对象appContext 的具体类型就是ContextImpl 。Activity的attach方法中将ContextImpl赋值给ContextWrapper的成员变量mBase中，因此，mBase具体指向就是ContextImpl 。
具体是不是这样呢？看看Activity的attach方法：

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
        attachBaseContext(context); //1
        ...
}

```

Activity继承ContextThemeWrapper，ContextThemeWrapper继承ContextWrapper，因此注释1执行的是ContextWrapper类attachBaseContext方法。

**ContextWrapper#attachBaseContext()**

```
    protected void attachBaseContext(Context base) {
        if (mBase != null) {
            throw new IllegalStateException("Base context already set");
        }
        mBase = base;
    }

```

果不其然，因此，ContextWrapper中的mBase具体指向就是ContextImpl 。

继续来看mBase.startService方法，就是ContextImpl的startService方法。

frameworks/base/core/java/android/app/ContextImpl.java

**ContextImpl#startService()**

```
    @Override
    public ComponentName startService(Intent service) {
        warnIfCallingFromSystemProcess();
        return startServiceCommon(service, mUser);
    }

```

**ContextImpl#startServiceCommon()**

```
    private ComponentName startServiceCommon(Intent service, UserHandle user) {
        try {
            validateServiceIntent(service);
            service.prepareToLeaveProcess(this);
            //1
            ComponentName cn = ActivityManagerNative.getDefault().startService(
                mMainThread.getApplicationThread(), service, service.resolveTypeIfNeeded(
                            getContentResolver()), getOpPackageName(), user.getIdentifier());
            ...
            return cn;
        } catch (RemoteException e) {
            throw e.rethrowFromSystemServer();
        }
    }

```

注释1有没有很熟悉？在Framework学习（七）AMS家族一文说过AMS中Binder机制。注释1其实最终调用的是AMS的startService方法。

## AMS到ActivityThread的调用

接着来查看AMS的startService方法。
frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

**ActivityManagerService#startService()**

```
    @Override
    public ComponentName startService(IApplicationThread caller, Intent service,
            String resolvedType, String callingPackage, int userId)
            throws TransactionTooLargeException {
        ...
        synchronized(this) {
            final int callingPid = Binder.getCallingPid();
            final int callingUid = Binder.getCallingUid();
            final long origId = Binder.clearCallingIdentity();
            ComponentName res = mServices.startServiceLocked(caller, service,
                    resolvedType, callingPid, callingUid, callingPackage, userId); //1
            Binder.restoreCallingIdentity(origId);
            return res;
        }
    }

```

注释1处调用mServices的startServiceLocked方法，mServices的类型是ActiveServices。

frameworks/base/services/core/java/com/android/server/am/ActiveServices.java

**ActiveServices#startServiceLocked()**

```
ComponentName startServiceLocked(IApplicationThread caller, Intent service, String resolvedType,
            int callingPid, int callingUid, String callingPackage, final int userId)
            throws TransactionTooLargeException {
      ...
        return startServiceInnerLocked(smap, service, r, callerFg, addToStarting);
    }

```

**ActiveServices#startServiceInnerLocked()**

```
   ComponentName startServiceInnerLocked(ServiceMap smap, Intent service, ServiceRecord r,
            boolean callerFg, boolean addToStarting) throws TransactionTooLargeException {

     ...
        String error = bringUpServiceLocked(r, service.getFlags(), callerFg, false, false);
     ...
        return r.name;
    }

```

**ActiveServices#bringUpServiceLocked()**

```
  private String bringUpServiceLocked(ServiceRecord r, int intentFlags, boolean execInFg,
            boolean whileRestarting, boolean permissionsReviewRequired)
            throws TransactionTooLargeException {
...
  final String procName = r.processName;//1
  ProcessRecord app;
  if (!isolated) {
            app = mAm.getProcessRecordLocked(procName, r.appInfo.uid, false);//2
            if (DEBUG_MU) Slog.v(TAG_MU, "bringUpServiceLocked: appInfo.uid=" + r.appInfo.uid
                        + " app=" + app);
            if (app != null && app.thread != null) {//3
                try {
                    app.addPackage(r.appInfo.packageName, r.appInfo.versionCode,
                    mAm.mProcessStats);
                    realStartServiceLocked(r, app, execInFg);//4
                    return null;
                } catch (TransactionTooLargeException e) {
                    throw e;
                } catch (RemoteException e) {
                    Slog.w(TAG, "Exception when starting service " + r.shortName, e);
                }
            }
        } else {
            app = r.isolatedProc;
        }
 if (app == null && !permissionsReviewRequired) {//5
            if ((app=mAm.startProcessLocked(procName, r.appInfo, true, intentFlags,
                    "service", r.name, false, isolated, false)) == null) {//6
              ...
            }
            if (isolated) {
                r.isolatedProc = app;
            }
        }
 ...     
}

```

和应用程序启动流程基本一致。
注释1处得到ServiceRecord的processName的值赋值给procName ，其中ServiceRecord用来描述Service的android:process属性。
注释2处将procName和Service的uid传入到AMS的getProcessRecordLocked方法中，来查询是否存在一个与Service对应的ProcessRecord类型的对象app，ProcessRecord主要用来记录运行的应用程序进程的信息。
注释5处判断Service对应的app为null则说明用来运行Service的应用程序进程不存在，则调用注释6处的AMS的startProcessLocked方法来创建对应的应用程序进程，具体创建过程参考Framework学习（六）应用程序进程启动过程。
注释3处表示Service的应用程序进程已经存在，则执行注释4的realStartServiceLocked方法。

**ActiveServices#realStartServiceLocked()**

```
private final void realStartServiceLocked(ServiceRecord r,
        ProcessRecord app, boolean execInFg) throws RemoteException {
   ...
    try {
       ...
        //1
        app.thread.scheduleCreateService(r, r.serviceInfo,
                mAm.compatibilityInfoForPackageLocked(r.serviceInfo.applicationInfo),
                app.repProcState);
        r.postNotification();
        created = true;
    } catch (DeadObjectException e) {
      ...
    } 
    ...
}

```

注释1调用了app.thread的scheduleCreateService方法。其中app.thread是IApplicationThread类型的，它的实现是ActivityThread的内部类ApplicationThread，其中ApplicationThread继承了ApplicationThreadNative，而ApplicationThreadNative继承了Binder并实现了IApplicationThread接口。

ActivityThread启动Service
frameworks/base/core/java/android/app/ActivityThread.java

**ApplicationThread#scheduleCreateService()**

```
        public final void scheduleCreateService(IBinder token,
                ServiceInfo info, CompatibilityInfo compatInfo, int processState) {
            updateProcessState(processState, false);
            CreateServiceData s = new CreateServiceData(); //1
            s.token = token;
            s.info = info;
            s.compatInfo = compatInfo;

            sendMessage(H.CREATE_SERVICE, s); //2
        }

```

注释1将要启动的信息封装成CreateServiceData对象。
注释2将CreateServiceData对象通过sendMessage方法向H发送CREATE_SERVICE消息。

**ApplicationThread#sendMessage()**

```
    private void sendMessage(int what, Object obj) {
        sendMessage(what, obj, 0, 0, false);
    }

    private void sendMessage(int what, Object obj, int arg1, int arg2, boolean async) {
        if (DEBUG_MESSAGES) Slog.v(
            TAG, "SCHEDULE " + what + " " + mH.codeToString(what)
            + ": " + arg1 + " / " + obj);
        Message msg = Message.obtain();
        msg.what = what;
        msg.obj = obj;
        msg.arg1 = arg1;
        msg.arg2 = arg2;
        if (async) {
            msg.setAsynchronous(true);
        }
        mH.sendMessage(msg);
    }

```

这里mH指的是H，它是ActivityThread的内部类并继承Handler。

**ActivityThread.H**

```
 private class H extends Handler {
        public static final int CREATE_SERVICE          = 114;
  ...

  public void handleMessage(Message msg) {
            if (DEBUG_MESSAGES) Slog.v(TAG, ">>> handling: " + codeToString(msg.what));
            switch (msg.what) {
                case CREATE_SERVICE:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, ("serviceCreate: " + String.valueOf(msg.obj)));
                    handleCreateService((CreateServiceData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                 ...
  } 

```

**ActivityThread#handleCreateService()**

```
    private void handleCreateService(CreateServiceData data) {

        LoadedApk packageInfo = getPackageInfoNoCheck(
                data.info.applicationInfo, data.compatInfo);  //1
        Service service = null;
        try {
            java.lang.ClassLoader cl = packageInfo.getClassLoader();  //2
            service = (Service) cl.loadClass(data.info.name).newInstance();  //3
        } catch (Exception e) {
            ...
        }

        try {
            ContextImpl context = ContextImpl.createAppContext(this, packageInfo);  //4
            context.setOuterContext(service);

            Application app = packageInfo.makeApplication(false, mInstrumentation);  //5
            service.attach(context, this, data.info.name, data.token, app,
                    ActivityManagerNative.getDefault());  //6
            service.onCreate();  //7
            mServices.put(data.token, service);  //8
            ...
        }
    }

```

注释1处获取要启动Service的应用程序的LoadedApk，LoadedApk是一个APK文件的描述类。
注释2处通过调用LoadedApk的getClassLoader方法来获取类加载器。
注释3处根据CreateServiceData对象中存储的Service信息，通过反射创建service实例。
注释4处创建Service的上下文环境ContextImpl对象。
注释5处获取当前应用的Application对象，该对象的唯一作用就是作为参数传递到Service里，然后在Service类中可以获得调用getApplication方法来获取Application对象。
注释6处调用Service的attach方法初始化Service，将ContextImpl对象注册到对应的Service中，之后在Service类中就可以使用Context的所有功能了。
注释7处调用Service的onCreate方法，这样Service就启动了。
注释8处将启动的Service加入到ActivityThread的成员变量mServices中，其中mServices是ArrayMap类型。

最后看看Service的启动时序图：

![image](//upload-images.jianshu.io/upload_images/19956127-27206b815743150e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

## Service的绑定过程

Service的绑定过程和Service的启动过程很多内容有重复，这里简单跟一下流程。

## ContextImpl请求AMS

frameworks/base/core/java/android/content/ContextWrapper.java

**ContextWrapper#bindService()**

```
@Override
 public boolean bindService(Intent service, ServiceConnection conn,
         int flags) {
     return mBase.bindService(service, conn, flags);
 }

```

frameworks/base/core/java/android/app/ContextImpl.java

**ContextImpl#bindService()**

```
@Override
  public boolean bindService(Intent service, ServiceConnection conn,
          int flags) {
      warnIfCallingFromSystemProcess();
      return bindServiceCommon(service, conn, flags, mMainThread.getHandler(),
              Process.myUserHandle());
  }

```

**ContextImpl#bindServiceCommon()**

```
private boolean bindServiceCommon(Intent service, ServiceConnection conn, int flags, Handler
        handler, UserHandle user) {
    IServiceConnection sd;
    if (conn == null) {
        throw new IllegalArgumentException("connection is null");
    }
    if (mPackageInfo != null) {
        sd = mPackageInfo.getServiceDispatcher(conn, getOuterContext(), handler, flags);//1
    } else {
        throw new RuntimeException("Not supported in system context");
    }
    validateServiceIntent(service);
    try {
     ...
        //2
        int res = ActivityManagerNative.getDefault().bindService(
            mMainThread.getApplicationThread(), getActivityToken(), service,
            service.resolveTypeIfNeeded(getContentResolver()),
            sd, flags, getOpPackageName(), user.getIdentifier());
      ...
    } catch (RemoteException e) {
        throw e.rethrowFromSystemServer();
    }
}

```

注释1处调用了LoadedApk类型的对象mPackageInfo的getServiceDispatcher方法，它的主要作用是将ServiceConnection封装为IServiceConnection类型的对象sd，从IServiceConnection的名字我们就能得知它实现了Binder机制，这样Service的绑定就支持了跨进程通信。
注释2处我们又看见了熟悉的代码，最终会调用AMS的bindService方法。

AMS到ActivityThread的调用
接着来查看AMS的bindService方法。
frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

**ActivityManagerService#bindService()**

```
    public int bindService(IApplicationThread caller, IBinder token, Intent service,
            String resolvedType, IServiceConnection connection, int flags, String callingPackage,
            int userId) throws TransactionTooLargeException {

        ...
        synchronized(this) {
            return mServices.bindServiceLocked(caller, token, service,
                    resolvedType, connection, flags, callingPackage, userId);
        }
    }

```

frameworks/base/services/core/java/com/android/server/am/ActiveServices.java

**ActiveServices#bindServiceLocked()**

```
int bindServiceLocked(IApplicationThread caller, IBinder token, Intent service,
            String resolvedType, final IServiceConnection connection, int flags,
            String callingPackage, final int userId) throws TransactionTooLargeException {
 ...
 if ((flags&Context.BIND_AUTO_CREATE) != 0) {
                s.lastActivity = SystemClock.uptimeMillis();
                //1
                if (bringUpServiceLocked(s, service.getFlags(), callerFg, false,
                        permissionsReviewRequired) != null) {
                    return 0;
                }
            }
          ...
            if (s.app != null && b.intent.received) { //2
                try {
                    c.conn.connected(s.name, b.intent.binder); //3
                } catch (Exception e) {
                ...
                }
                if (b.intent.apps.size() == 1 && b.intent.doRebind) { //4
                    requestServiceBindingLocked(s, b.intent, callerFg, true); //5
                }
            } else if (!b.intent.requested) { //6
                requestServiceBindingLocked(s, b.intent, callerFg, false); //7
            }
            getServiceMap(s.userId).ensureNotStartingBackground(s);
        } finally {
            Binder.restoreCallingIdentity(origId);
        }
        return 1;
}

```

注释1处会调用bringUpServiceLocked方法，在bringUpServiceLocked方法中又会调用realStartServiceLocked方法，最终由ActivityThread来调用Service的onCreate方法启动Service，这一过程上面已经介绍了。
在注释2处s.app != null 表示Service已经运行，其中s是ServiceRecord类型对象，app是ProcessRecord类型对象。b.intent.received表示当前应用程序进程的Client端已经接收到绑定Service时返回的Binder，这样应用程序进程的Client端就可以通过Binder来获取要绑定的Service的访问接口。
注释3处调用c.conn的connected方法，其中c.conn指的是IServiceConnection，它的具体实现为ServiceDispatcher.InnerConnection，其中ServiceDispatcher是LoadedApk的内部类，InnerConnection的connected方法内部会调用H的post方法向主线程发送消息，从而解决当前应用程序进程和Service跨进程通信的问题，在后面会详细介绍这一过程。
注释4处如果当前应用程序进程的Client端第一次与Service进行绑定的，并且Service已经调用过onUnBind方法，则需要调用注释5的代码。
注释6处如果应用程序进程的Client端没有发送过绑定Service的请求，则会调用注释7的代码，注释7和注释5的代码区别就是最后一个参数rebind为false，表示不是重新绑定。
接着我们查看注释7的requestServiceBindingLocked方法。

**ActiveServices#requestServiceBindingLocked()**

```
private final boolean requestServiceBindingLocked(ServiceRecord r, IntentBindRecord i,
        boolean execInFg, boolean rebind) throws TransactionTooLargeException {
   ...
    if ((!i.requested || rebind) && i.apps.size() > 0) {//1
        try {
            bumpServiceExecutingLocked(r, execInFg, "bind");
            r.app.forceProcessStateUpTo(ActivityManager.PROCESS_STATE_SERVICE);
            r.app.thread.scheduleBindService(r, i.intent.getIntent(), rebind,
                    r.app.repProcState);//2
           ...
        } 
        ...
    }
    return true;
}

```

注释1处i.requested表示是否发送过绑定Service的请求，从前面的代码得知是没有发送过，因此，!i.requested为true。从前面的代码得知rebind值为false，那么(!i.requested || rebind)的值为true。如果IntentBindRecord中的应用程序进程记录大于0，则会调用注释2的代码，r.app.thread的类型为IApplicationThread，它的实现我们已经很熟悉了，是ActivityThread的内部类ApplicationThread。

## ActivityThread绑定Service

frameworks/base/core/java/android/app/ActivityThread.java

**ApplicationThread#scheduleBindService()**

```
public final void scheduleBindService(IBinder token, Intent intent,
              boolean rebind, int processState) {
          updateProcessState(processState, false);
          BindServiceData s = new BindServiceData();
          s.token = token;
          s.intent = intent;
          s.rebind = rebind;
          if (DEBUG_SERVICE)
              Slog.v(TAG, "scheduleBindService token=" + token + " intent=" + intent + " uid="
                      + Binder.getCallingUid() + " pid=" + Binder.getCallingPid());
          sendMessage(H.BIND_SERVICE, s);
      }

```

## ActivityThread.H

```
public void handleMessage(Message msg) {
          if (DEBUG_MESSAGES) Slog.v(TAG, ">>> handling: " + codeToString(msg.what));
          switch (msg.what) {
          ...
              case BIND_SERVICE:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "serviceBind");
                    handleBindService((BindServiceData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
          ...
           }
        ...
        }
     ...   
}

```

**ActivityThread#handleBindService()**

```
private void handleBindService(BindServiceData data) {
       Service s = mServices.get(data.token); //1
       if (DEBUG_SERVICE)
           Slog.v(TAG, "handleBindService s=" + s + " rebind=" + data.rebind);
       if (s != null) {
           try {
               data.intent.setExtrasClassLoader(s.getClassLoader());
               data.intent.prepareToEnterProcess();
               try {
                   if (!data.rebind) { //2
                       IBinder binder = s.onBind(data.intent); //3
                       ActivityManagerNative.getDefault().publishService(
                               data.token, data.intent, binder); //4
                   } else {
                       s.onRebind(data.intent); //5
                       ActivityManagerNative.getDefault().serviceDoneExecuting(
                               data.token, SERVICE_DONE_EXECUTING_ANON, 0, 0);
                   }
                   ensureJitEnabled();
               } 
               ...
           } 
           ...
       }
   }

```

注释1处获取要绑定的Service 。
注释2处如果BindServiceData的成员变量rebind的值为false，这样会调用注释3处的代码来调用Service的onBind方法，这样Service处于绑定状态了。如果rebind的值为true就会调用注释5处的Service的onRebind方法，结合前文的bindServiceLocked方法的注释4，我们得知如果当前应用程序进程的Client端第一次与Service进行绑定，并且Service已经调用过onUnBind方法，则会调用Service的onRebind方法。
注释4的代码，实际上是调用AMS的publishService方法。

frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

**ActivityManagerService#publishService()**

```
    public void publishService(IBinder token, Intent intent, IBinder service) {
        ...
        synchronized(this) {
            if (!(token instanceof ServiceRecord)) {
                throw new IllegalArgumentException("Invalid service token");
            }
            mServices.publishServiceLocked((ServiceRecord)token, intent, service);
        }
    }

```

frameworks/base/services/core/java/com/android/server/am/ActiveServices.java

**ActiveServices#publishServiceLocked()**

```
    void publishServiceLocked(ServiceRecord r, Intent intent, IBinder service) {

        try {
            ...
                    for (int conni=r.connections.size()-1; conni>=0; conni--) {
                        ArrayList<ConnectionRecord> clist = r.connections.valueAt(conni);
                        for (int i=0; i<clist.size(); i++) {
                            ConnectionRecord c = clist.get(i);
                            if (!filter.equals(c.binding.intent.intent)) {
                                continue;
                            }
                            if (DEBUG_SERVICE) Slog.v(TAG_SERVICE, "Publishing to: " + c);
                            try {
                                c.conn.connected(r.name, service); //1
                            } catch (Exception e) {
                                ...
                            }
                        }
                    }
                }

                serviceDoneExecutingLocked(r, mDestroyingServices.contains(r), false);
            }
        } finally {
            Binder.restoreCallingIdentity(origId);
        }
    }

```

注释1处的代码，我在前面介绍过，c.conn指的是IServiceConnection，它的具体实现为ServiceDispatcher.InnerConnection，其中ServiceDispatcher是LoadedApk的内部类。

frameworks/base/core/java/android/app/LoadedApk.java

**ServiceDispatcher**

```
    static final class ServiceDispatcher {
        ...

        private static class InnerConnection extends IServiceConnection.Stub {
            final WeakReference<LoadedApk.ServiceDispatcher> mDispatcher;

            InnerConnection(LoadedApk.ServiceDispatcher sd) {
                mDispatcher = new WeakReference<LoadedApk.ServiceDispatcher>(sd);
            }

            public void connected(ComponentName name, IBinder service) throws RemoteException {
                LoadedApk.ServiceDispatcher sd = mDispatcher.get();
                if (sd != null) {
                    sd.connected(name, service); //1
                } 
            }
        }
        ...
}

```

注释1处调用了ServiceDispatcher类型的sd对象的connected方法。

**ServiceDispatcher#connected()**

```
        public void connected(ComponentName name, IBinder service) {
            if (mActivityThread != null) {
                mActivityThread.post(new RunConnection(name, service, 0));  //1
            } else {
                doConnected(name, service);
            }
        }

```

注释1处调用Handler类型对象mActivityThread的post方法，mActivityThread实际上指向的是H。因此，通过调用H的post方法将RunConnection对象的内容运行在主线程中。来看看RunConnection的定义。

**RunConnection**

```
        private final class RunConnection implements Runnable {
            RunConnection(ComponentName name, IBinder service, int command) {
                mName = name;
                mService = service;
                mCommand = command;
            }

            public void run() {
                if (mCommand == 0) {
                    doConnected(mName, mService);  //1
                } else if (mCommand == 1) {
                    doDeath(mName, mService);
                }
            }

            final ComponentName mName;
            final IBinder mService;
            final int mCommand;
        }

```

注释1中调用了doConnected方法。

**ServiceDispatcher#doConnected()**

```
       public void doConnected(ComponentName name, IBinder service) {
            ServiceDispatcher.ConnectionInfo old;
            ServiceDispatcher.ConnectionInfo info;

            synchronized (this) {

            old = mActiveConnections.get(name);

            ...
            // If there was an old service, it is not disconnected.
            if (old != null) {
                mConnection.onServiceDisconnected(name);
            }
            // If there is a new service, it is now connected.
            if (service != null) {
                mConnection.onServiceConnected(name, service); //1
            }
        }

```

注释1处调用了ServiceConnection类型的对象mConnection的onServiceConnected方法，这样在客户端中实现了ServiceConnection接口的类的onServiceConnected方法就会被执行。

最后看看Service的启动时序图：

![](//upload-images.jianshu.io/upload_images/19956127-1d1618471f52864d.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

>原文作者：huaxun66
原文链接：[https://blog.csdn.net/huaxun66/article/details/78182265](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fhuaxun66%2Farticle%2Fdetails%2F78182265)

**更多系列教程GitHub学习地址：[https://github.com/Timdk857/Android-Architecture-knowledge-2-](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FTimdk857%2FAndroid-Architecture-knowledge-2-)**
