在Android系统中，启动四大组件中的任何一个都可以启动应用程序。但绝大部分时候我们是通过点击Launcher图标来启动应用程序。本文依据Android7.0源码，从点击Launcher图标开始，分析应用程序的启动过程，其实就是分析根Activity的启动过程。

## Launcher请求AMS

在 [Framework学习（四）Launcher启动过程](https://links.jianshu.com/go?to=%255Bhttps%3A%2F%2Fwww.jianshu.com%2Fp%2F22462aeef184%255D%28https%3A%2F%2Fwww.jianshu.com%2Fp%2F22462aeef184%29)这篇文章我讲过Launcher启动后会将已安装应用程序的快捷图标显示到界面上，当我们点击应用程序的快捷图标时就会调用Launcher的startActivitySafely方法。

packages/apps/Launcher3/src/com/android/launcher3/Launcher.java

**Launcher#startActivitySafely()**

```
    public boolean startActivitySafely(View v, Intent intent, Object tag) {
        boolean success = false;
        if (mIsSafeModeEnabled && !Utilities.isSystemApp(this, intent)) {
            Toast.makeText(this, R.string.safemode_shortcut_error, Toast.LENGTH_SHORT).show();
            return false;
        }
        try {
            success = startActivity(v, intent, tag); //1
        } catch (ActivityNotFoundException e) {
            Toast.makeText(this, R.string.activity_not_found, Toast.LENGTH_SHORT).show();
            Log.e(TAG, "Unable to launch. tag=" + tag + " intent=" + intent, e);
        }
        return success;
    }

```

注释1调用了startActivity函数。

**Launcher#startActivity()**

```
private boolean startActivity(View v, Intent intent, Object tag) {
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);//1
        try {
          ...
            if (user == null || user.equals(UserHandleCompat.myUserHandle())) {
                StrictMode.VmPolicy oldPolicy = StrictMode.getVmPolicy();
                try {            
                    StrictMode.setVmPolicy(new StrictMode.VmPolicy.Builder().detectAll() .penaltyLog().build());
                    startActivity(intent, optsBundle);//2
                } finally {
                    StrictMode.setVmPolicy(oldPolicy);
                }
            } else {
                launcherApps.startActivityForProfile(intent.getComponent(), user, intent.getSourceBounds(), optsBundle);
            }
            return true;
        } catch (SecurityException e) {      
          ...
        }
        return false;
    }

```

注释1处设置Flag为Intent.FLAG_ACTIVITY_NEW_TASK，这样根Activity会在新的任务栈中启动。
注释2处调用了Activity的startActivity函数。

frameworks/base/core/java/android/app/Activity.java

**Activity#startActivity()**

```
    @Override
    public void startActivity(Intent intent, @Nullable Bundle options) {
        if (options != null) {
            startActivityForResult(intent, -1, options); //1
        } else {
            // Note we want to go through this call for compatibility with
            // applications that may have overridden the method.
            startActivityForResult(intent, -1); //2
        }
    }

```

注释1和2处都会调用startActivityForResult函数，其中第二个参数为-1，表示Launcher不需要知道Activity启动的结果。

**Activity#startActivityForResult**

```
    public void startActivityForResult(@RequiresPermission Intent intent, int requestCode, @Nullable Bundle options) {
        if (mParent == null) {
            Instrumentation.ActivityResult ar =
                mInstrumentation.execStartActivity(this, mMainThread.getApplicationThread(), mToken, this, intent, requestCode, options);  //1
            if (ar != null) {
                mMainThread.sendActivityResult(mToken, mEmbeddedID, requestCode, ar.getResultCode(), ar.getResultData());
            }
...
        } else {
            ...
        }
    }

```

mParent是Activity类型的，表示当前Activity的父类。因为目前根Activity还没有创建出来，因此，mParent == null成立。
注释1调用Instrumentation的execStartActivity方法，Instrumentation主要用来监控应用程序和系统的交互。

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

注释1首先调用ActivityManagerNative的getDefault来获取ActivityManageService（以后简称为AMS)的代理对象，接着调用它的startActivity方法。
这里ActivityManagerNative.getDefault()涉及到Binder进程间通信机制，下面进行一个简短的介绍，这不是本文的重点。了解过的同学可以略过下面这段Binder简析，不想了解的也可以直接理解为返回的是AMS。实际调用的是AMS#startActivity()。

**AMS中Binder机制简析**

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

getDefault方法调用了gDefault的get方法，gDefault 是一个Singleton类。注释1处得到名为”activity”的Service代理对象，也就是AMS的代理对象。
注释2处将它封装成ActivityManagerProxy（以后简称为AMP）类型对象，并将它保存到gDefault中，此后调用ActivityManagerNative（以后简称为AMN）的getDefault方法就会直接获得AMS的代理AMP对象。

回到Instrumentation类的execStartActivity方法中，从上面得知就是调用AMP的startActivity，其中AMP是AMN的内部类，代码如下所示。

frameworks/base/core/java/android/app/ActivityManagerNative.java

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

**ActivityManagerService#startActivity()**

```
    @Override
    public final int startActivity(IApplicationThread caller, String callingPackage, Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode, int startFlags, ProfilerInfo profilerInfo, Bundle bOptions) {
        return startActivityAsUser(caller, callingPackage, intent, resolvedType, resultTo, resultWho, requestCode, startFlags, profilerInfo, bOptions, UserHandle.getCallingUserId()); //1
    }

```

AMS中Binder机制暂且分析到这里。

下面看从Launcher到AMS的时序图：

![image](//upload-images.jianshu.io/upload_images/19956127-17d82583da398288.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

## AMS到ActivityThread的调用流程

上面注释1处调用了AMS的startActivityAsUser方法。

**ActivityManagerService#startActivityAsUser()**

```
    @Override
    public final int startActivityAsUser(IApplicationThread caller, String callingPackage, Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode, int startFlags, ProfilerInfo profilerInfo, Bundle bOptions, int userId) {
        enforceNotIsolatedCaller("startActivity");
        userId = mUserController.handleIncomingUser(Binder.getCallingPid(), Binder.getCallingUid(), userId, false, ALLOW_FULL_ONLY, "startActivity", null);
        // TODO: Switch to user app stacks here.
        //1
        return mActivityStarter.startActivityMayWait(caller, -1, callingPackage, intent, resolvedType, null, null, resultTo, resultWho, requestCode, startFlags, profilerInfo, null, null, bOptions, false, userId, null, null);
    }

```

注释1处调用了ActivityStarter的startActivityMayWait方法。

frameworks/base/services/core/java/com/android/server/am/ActivityStarter.java

**ActivityStarter#startActivityMayWait()**

```
final int startActivityMayWait(IApplicationThread caller, int callingUid,String callingPackage, Intent intent, String resolvedType,IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,IBinder resultTo, String resultWho, int requestCode, int startFlags,ProfilerInfo profilerInfo, IActivityManager.WaitResult outResult, Configuration config,Bundle bOptions, boolean ignoreTargetSecurity, int userId,IActivityContainer iContainer, TaskRecord inTask) {
       ...

        doPendingActivityLaunchesLocked(false); //1

       ...
        return err;
    }

```

注释1调用了doPendingActivityLaunchesLocked方法。

**ActivityStarter#doPendingActivityLaunchesLocked()**

```
    final void doPendingActivityLaunchesLocked(boolean doResume) {
        while (!mPendingActivityLaunches.isEmpty()) {
            final PendingActivityLaunch pal = mPendingActivityLaunches.remove(0);
            final boolean resume = doResume && mPendingActivityLaunches.isEmpty();
            try {
                //1
                final int result = startActivityUnchecked(pal.r, pal.sourceRecord, null, null, pal.startFlags, resume, null, null);
                postStartActivityUncheckedProcessing(pal.r, result, mSupervisor.mFocusedStack.mStackId, mSourceRecord, mTargetStack);
            } catch (Exception e) {
                Slog.e(TAG, "Exception during pending activity launch pal=" + pal, e);
                pal.sendErrorResult(e.getMessage());
            }
        }
    }

```

注释1处调用startActivityUnchecked方法。

**ActivityStarter#startActivityUnchecked()**

```
 private int startActivityUnchecked(final ActivityRecord r, ActivityRecord sourceRecord,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            int startFlags, boolean doResume, ActivityOptions options, TaskRecord inTask) {
      ...  
           if (mDoResume) {
                mSupervisor.resumeFocusedStackTopActivityLocked(); //1
            } 
      ... 
        return START_SUCCESS;
    }

```

注释1处调用了ActivityStackSupervisor的resumeFocusedStackTopActivityLocked方法。

frameworks/base/services/core/java/com/android/server/am/ActivityStackSupervisor.java

**ActivityStackSupervisor#resumeFocusedStackTopActivityLocked()**

```
    boolean resumeFocusedStackTopActivityLocked( ActivityStack targetStack, ActivityRecord target, ActivityOptions targetOptions) {
        if (targetStack != null && isFocusedStack(targetStack)) {
            return targetStack.resumeTopActivityUncheckedLocked(target, targetOptions); //1
        }
        final ActivityRecord r = mFocusedStack.topRunningActivityLocked();
        if (r == null || r.state != RESUMED) {
            mFocusedStack.resumeTopActivityUncheckedLocked(null, null);
        }
        return false;
    }

```

注释1处调用了ActivityStack的resumeTopActivityUncheckedLocked方法。

frameworks/base/services/core/java/com/android/server/am/ActivityStack.java

**ActivityStack#resumeTopActivityUncheckedLocked()**

```
    boolean resumeTopActivityUncheckedLocked(ActivityRecord prev, ActivityOptions options) {
        ...
        boolean result = false;
        try {
            // Protect against recursion.
            mStackSupervisor.inResumeTopActivity = true;
            if (mService.mLockScreenShown == ActivityManagerService.LOCK_SCREEN_LEAVING) {
                mService.mLockScreenShown = ActivityManagerService.LOCK_SCREEN_HIDDEN;
                mService.updateSleepIfNeededLocked();
            }
            result = resumeTopActivityInnerLocked(prev, options);  //1
        } finally {
            mStackSupervisor.inResumeTopActivity = false;
        }
        return result;
    }

```

注释1处调用resumeTopActivityInnerLocked函数。

**ActivityStack#resumeTopActivityInnerLocked()**

```
 private boolean resumeTopActivityInnerLocked(ActivityRecord prev, ActivityOptions options) {
        ...
        // If the top activity is the resumed one, nothing to do.
        if (mResumedActivity == next && next.state == ActivityState.RESUMED &&
                    mStackSupervisor.allResumedActivitiesComplete()) {
            ...
            return false;
        }
        if (mResumedActivity != null) {
            if (DEBUG_STATES) Slog.d(TAG_STATES,
                    "resumeTopActivityLocked: Pausing " + mResumedActivity);
            pausing |= startPausingLocked(userLeaving, false, true, dontWaitForPause); //1
        }
        ...
            mStackSupervisor.startSpecificActivityLocked(next, true, false); //2
            if (DEBUG_STACK) mStackSupervisor.validateTopActivitiesLocked();
       ...

```

这个方法里面的内容很多。
注释1主要作用是将mResumedActivity暂停(Launcher任务栈的TopActivity)，即进入onPause状态。
注释2调用了ActivityStackSupervisor的startSpecificActivityLocked函数启动指定的AttivityRecored。

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
注释3处调用AMS的startProcessLocked来启动应用程序进程，注意这里是应用程序进程，只有应用程序进程起来了，才能起应用程序。关于应用程序进程的启动我们可以看Framework学习（六）应用程序进程启动过程这篇文章。

**ActivityStackSupervisor#realStartActivityLocked()**

```
final boolean realStartActivityLocked(ActivityRecord r, ProcessRecord app, boolean andResume, boolean checkConfig) throws RemoteException {
     ...
            //1    
            app.thread.scheduleLaunchActivity(new Intent(r.intent), r.appToken, System.identityHashCode(r), r.info, new Configuration(mService.mConfiguration), new Configuration(task.mOverrideConfig), r.compat, r.launchedFromPackage, task.voiceInteractor, app.repProcState, r.icicle, r.persistentState, results, newIntents, !andResume,mService.isNextTransitionForward(), profilerInfo);

    ...      

        return true;
    }

```

这里的app.thread指的是IApplicationThread，它的实现是ActivityThread的内部类ApplicationThread，其中ApplicationThread继承了ApplicationThreadNative，而ApplicationThreadNative继承了Binder并实现了IApplicationThread接口。

下面看从AMS到ApplicationThread的时序图：

![image](//upload-images.jianshu.io/upload_images/19956127-3cb935fee21618cb.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

**ActivityThread启动Application和Activity**
在应用程序进程启动时会创建ActivityThread实例。ActivityThread作为应用程序进程的核心类，它是如何启动应用程序（Activity）的呢？
根据上文接着查看ApplicationThread的scheduleLaunchActivity方法：

frameworks/base/core/java/android/app/ActivityThread.java

**ApplicationThread#scheduleLaunchActivity()**

```
        @Override
        public final void scheduleLaunchActivity(Intent intent, IBinder token, int ident,ActivityInfo info, Configuration curConfig, Configuration overrideConfig,
CompatibilityInfo compatInfo, String referrer, IVoiceInteractor voiceInteractor,
int procState, Bundle state, PersistableBundle persistentState,List<ResultInfo> pendingResults, List<ReferrerIntent> pendingNewIntents, boolean notResumed, boolean isForward, ProfilerInfo profilerInfo) {

            updateProcessState(procState, false);

            ActivityClientRecord r = new ActivityClientRecord();

            r.token = token;
            r.ident = ident;
            r.intent = intent;
            r.referrer = referrer;
            r.voiceInteractor = voiceInteractor;
            r.activityInfo = info;
            r.compatInfo = compatInfo;
            r.state = state;
            r.persistentState = persistentState;

            r.pendingResults = pendingResults;
            r.pendingIntents = pendingNewIntents;

            r.startsNotResumed = notResumed;
            r.isForward = isForward;

            r.profilerInfo = profilerInfo;

            r.overrideConfig = overrideConfig;
            updatePendingConfiguration(curConfig);

            sendMessage(H.LAUNCH_ACTIVITY, r); //1
        }

```

会将启动Activity的参数封装成ActivityClientRecord。
注释1处sendMessage方法向H类发送类型为LAUNCH_ACTIVITY的消息，并将ActivityClientRecord 传递过去。

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
        public static final int LAUNCH_ACTIVITY         = 100;
        public static final int PAUSE_ACTIVITY          = 101;
  ...

  public void handleMessage(Message msg) {
            if (DEBUG_MESSAGES) Slog.v(TAG, ">>> handling: " + codeToString(msg.what));
            switch (msg.what) {
                case LAUNCH_ACTIVITY: {
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityStart");
                    final ActivityClientRecord r = (ActivityClientRecord) msg.obj;//1
                    r.packageInfo = getPackageInfoNoCheck(
                            r.activityInfo.applicationInfo, r.compatInfo);//2
                    handleLaunchActivity(r, null, "LAUNCH_ACTIVITY");//3
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                } break;
              ...
  }     

```

查看H的handleMessage方法中对LAUNCH_ACTIVITY的处理。
注释1处将传过来的msg的成员变量obj转换为ActivityClientRecord。
在注释2处通过getPackageInfoNoCheck方法获得LoadedApk类型的对象并赋值给ActivityClientRecord的成员变量packageInfo 。应用程序进程要启动Activity时需要将该Activity所属的APK加载进来，而LoadedApk就是用来描述已加载的APK文件。
在注释3处调用handleLaunchActivity方法。

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
            try {
                //3
                ActivityManagerNative.getDefault() .finishActivity(r.token, Activity.RESULT_CANCELED, null,
 Activity.DONT_FINISH_TASK_WITH_ACTIVITY);
            } catch (RemoteException ex) {
                throw ex.rethrowFromSystemServer();
            }
        }
    }

```

注释1处的performLaunchActivity方法用来启动Activity。
注释2处的代码用来执行Activity的onResume方法，将Activity的状态置为Resume。
注释3如果该Activity为null则会通知ActivityManager停止启动Activity。

**ActivityThread#performLaunchActivity()**

```
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
  ...
        ActivityInfo aInfo = r.activityInfo; //1
        if (r.packageInfo == null) {
            r.packageInfo = getPackageInfo(aInfo.applicationInfo, r.compatInfo,Context.CONTEXT_INCLUDE_CODE); //2
        }
        ComponentName component = r.intent.getComponent(); //3
      ...
        Activity activity = null;
        try {
            java.lang.ClassLoader cl = r.packageInfo.getClassLoader();
            activity = mInstrumentation.newActivity(cl, component.getClassName(), r.intent); //4
           ...
            }
        } catch (Exception e) {
         ...
        }
        try {
            Application app = r.packageInfo.makeApplication(false, mInstrumentation); //5
        ...
            if (activity != null) {
                Context appContext = createBaseContextForActivity(r, activity); //6
         ...
                }
                //7
                activity.attach(appContext, this, getInstrumentation(), r.token, r.ident, app, r.intent, r.activityInfo, title, r.parent, r.embeddedID, r.lastNonConfigurationInstances, config, r.referrer, r.voiceInteractor, window);

              ...
                if (r.isPersistable()) {
                    mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState); //8
                } else {
                    mInstrumentation.callActivityOnCreate(activity, r.state);
                }
                ...
                if (!r.activity.mFinished) {
                    activity.performStart(); //9
                    r.stopped = false;
                }
                ...
                mActivities.put(r.token, r); //10
        }
        return activity;
}        

```

注释1处用来获取ActivityInfo。
注释2处获取APK文件的描述类LoadedApk。
注释3处获取要启动的Activity的ComponentName类，ComponentName类中保存了该Activity的包名和类名。
注释4处根据ComponentName中存储的Activity类名，用类加载器通过反射来创建该Activity的实例。
注释5处用来创建Application对象，makeApplication方法内部会调用Application的onCreate方法。该Application对象的唯一作用就是作为参数传递到Activity里，然后在Activity类中可以获得调用getApplication方法来获取Application对象。
注释6处用来创建要启动Activity的上下文环境ContextImpl。
注释7处调用Activity的attach方法初始化Activity，将ContextImpl对象注册到对应的Activity中，之后在Activity类中就可以使用Context的所有功能了。
注释8处会调用Instrumentation的callActivityOnCreate方法来启动Activity。
注释9处用来执行Activity的onStart方法。
注释10处将启动的Activity加入到ActivityThread的成员变量mActivities中，其中mServices是ArrayMap类型。

先来看看注释5方法：
frameworks/base/core/java/android/app/LoadedApk.java

**LoadedApk#makeApplication()**

```
public Application makeApplication(boolean forceDefaultAppClass,
            Instrumentation instrumentation) {
        if (mApplication != null) {  //1
            return mApplication;  
        }

        Application app = null;
        ...
        try {
            ContextImpl appContext = ContextImpl.createAppContext(mActivityThread, this);
            app = mActivityThread.mInstrumentation.newApplication(
                    cl, appClass, appContext); //2
        } catch (Exception e) {
            ...
        }
        ...
        if (instrumentation != null) {
            try {
                instrumentation.callApplicationOnCreate(app); //3
            } catch (Exception e) {
                ...
                }
            }
        }
        return app;
    }

```

注释1判断当前应用是否是第一次创建Application对象，如果不是则直接返回Application对象，否则执行注释2创建第一个Application对象。目的是确保当前应用之创建了一个全局的Application对象。
注释2调用Instrumentation的newApplication()方法创建Application。
注释3调用Instrumentation的callApplicationOnCreate()。

frameworks/base/core/java/android/app/Instrumentation.java

**Instrumentation#newApplication()**

```
public Application newApplication(ClassLoader cl, String className, Context context)
            throws InstantiationException, IllegalAccessException, 
            ClassNotFoundException {
        return newApplication(cl.loadClass(className), context);
    }

    static public Application newApplication(Class<?> clazz, Context context)
            throws InstantiationException, IllegalAccessException, 
            ClassNotFoundException {
        Application app = (Application)clazz.newInstance();  //1
        app.attach(context);  //2
        return app;
    }

```

注释1简单粗暴，通过反射创建一个Application实例。
注释2处调用Application的attach方法初始化Application，将ContextImpl对象注册到对应的Application中，之后在Application类中就可以使用Context的所有功能了。

**Instrumentation#callApplicationOnCreate()**

```
    public void callApplicationOnCreate(Application app) {
        app.onCreate();
    }

```

这样Application的onCreate()方法也得到执行。

现在已经有了Application且它的onCreate()方法也得到执行。但Activity只是被ClassLoader装载了，onCreate()还没有调起来。回到ActivityThread的performLaunchActivity()方法，看看注释8的方法：

**Instrumentation#callActivityOnCreate()**

```
   public void callActivityOnCreate(Activity activity, Bundle icicle) {
        prePerformCreate(activity);
        activity.performCreate(icicle); //1
        postPerformCreate(activity);
    }

```

注释1会调用Activity的performCreate方法。

frameworks/base/core/java/android/app/Activity.java

**Activity#performCreate()**

```
  final void performCreate(Bundle icicle) {
        restoreHasCurrentPermissionRequest(icicle);
        onCreate(icicle); //1
        mActivityTransitionState.readState(icicle);
        performCreateCommon();
    }

```

注释1会调用Activity的onCreate方法，这样Activity就启动了，即应用程序就启动了。

下面看从ApplicationThread到Activity的时序图：

![image](//upload-images.jianshu.io/upload_images/19956127-a6a74ec05f58312f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

## 总体流程

1.Launcher通过Binder机制通知AMS启动一个Activity.
2.AMS使Launcher栈最顶端Activity进入onPause状态.
3.AMS通知Process使用Socket和Zygote进程通信，请求创建一个新进程.
4.Zygote收到Socket请求，fork出一个进程，并调用ActivityThread#main().
5.ActivityThread通过Binder通知AMS启动应用程序.
6.AMS通知ActivityStackSupervisor真正的启动Activity.
7.ActivityStackSupervisor通知ApplicationThread启动Activity.
8.ApplicationThread发消息给ActivityThread，需要启动一个Activity.
9.ActivityThread收到消息之后，通知LoadedApk创建Applicaition，并且调用其onCteate()方法.
10.ActivityThread装载目标Activity类，并调用Activity#attach().
11.ActivityThread通知Instrumentation调用Activity#onCreate().
12.Instrumentation调用Activity#performCreate()，在Activity#performCreate()中调用自身onCreate()方法.
注：其中3、4、5属于应用程序进程启动流程，参考Framework学习（六）应用程序进程启动过程。

Application和Activity#onCreate()方法调用顺序
1.装载Activity
2.装载Application
3.Application#attach()
4.Application#onCreate()
5.Activity#attach()
6.Activity#onCreate()

**推荐阅读：[https://www.jianshu.com/p/14fa0c6ed6e8](https://www.jianshu.com/p/14fa0c6ed6e8)**


>原文作者：huaxun66
原文链接：[https://blog.csdn.net/huaxun66/article/details/78151361](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fhuaxun66%2Farticle%2Fdetails%2F78151361)

**更多系列教程GitHub学习地址：[https://github.com/Timdk857/Android-Architecture-knowledge-2-](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FTimdk857%2FAndroid-Architecture-knowledge-2-)**
