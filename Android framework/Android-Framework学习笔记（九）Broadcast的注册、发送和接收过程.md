## 广播的注册

BroadcastReceiver的注册分为两种：静态注册和动态注册。静态注册在应用安装时由PackageManagerService来完成注册过程。这里只介绍BroadcastReceiver的动态注册。

动态注册BroadcastReceiver，需要调用registerReceiver方法，它的实现在ContextWrapper中。

frameworks/base/core/java/android/content/ContextWrapper.java

**ContextWrapper#registerReceiver()**

```
    @Override
    public Intent registerReceiver(
        BroadcastReceiver receiver, IntentFilter filter) {
        return mBase.registerReceiver(receiver, filter);
    }

```

在前面Android Framework学习笔记（八）Service的启动/绑定过程
这篇文章中我们已经分析了，mBase具体指向就是ContextImpl。

frameworks/base/core/java/android/app/ContextImpl.java

**ContextImpl#registerReceiver()**

```
    @Override
    public Intent registerReceiver(BroadcastReceiver receiver, IntentFilter filter) {
        return registerReceiver(receiver, filter, null, null);
    }

    @Override
    public Intent registerReceiver(BroadcastReceiver receiver, IntentFilter filter,
            String broadcastPermission, Handler scheduler) {
        return registerReceiverInternal(receiver, getUserId(),
                filter, broadcastPermission, scheduler, getOuterContext());  //1
    }

```

注释1最终会调用到registerReceiverInternal。

**ContextImpl#registerReceiverInternal()**

```
    private Intent registerReceiverInternal(BroadcastReceiver receiver, int userId,
            IntentFilter filter, String broadcastPermission,
            Handler scheduler, Context context) {
        IIntentReceiver rd = null;
        if (receiver != null) {
            if (mPackageInfo != null && context != null) { //1
                if (scheduler == null) {
                    scheduler = mMainThread.getHandler();
                }
                rd = mPackageInfo.getReceiverDispatcher(
                    receiver, context, scheduler,
                    mMainThread.getInstrumentation(), true); //2
            } else {
                if (scheduler == null) {
                    scheduler = mMainThread.getHandler();
                }
                rd = new LoadedApk.ReceiverDispatcher(
                        receiver, context, scheduler, null, true).getIIntentReceiver(); //3
            }
        }
        try {
            final Intent intent = ActivityManagerNative.getDefault().registerReceiver(
                    mMainThread.getApplicationThread(), mBasePackageName,
                    rd, filter, broadcastPermission, userId); //4
            if (intent != null) {
                intent.setExtrasClassLoader(getClassLoader());
                intent.prepareToEnterProcess();
            }
            return intent;
        } catch (RemoteException e) {
            throw e.rethrowFromSystemServer();
        }
    }

```

注释1处判断如果LoadedApk类型的mPackageInfo不等于null并且context不等null就调用注释2处的代码通过mPackageInfo的getReceiverDispatcher方法获取rd对象，否则就调用注释3处的代码来创建rd对象。
注释2和3的代码的目的都是要获取IIntentReceiver类型的rd对象，IIntentReceiver是一个Binder接口，用于进行跨进程的通信，它的具体实现在LoadedApk.ReceiverDispatcher.InnerReceiver。

frameworks/base/core/java/android/app/LoadedApk.java

**ReceiverDispatcher**

```
static final class ReceiverDispatcher {
      final static class InnerReceiver extends IIntentReceiver.Stub {
          final WeakReference<LoadedApk.ReceiverDispatcher> mDispatcher;
          final LoadedApk.ReceiverDispatcher mStrongRef;
          InnerReceiver(LoadedApk.ReceiverDispatcher rd, boolean strong) {
              mDispatcher = new WeakReference<LoadedApk.ReceiverDispatcher>(rd);
              mStrongRef = strong ? rd : null;
          }
        ...  
       }
      ... 
 }

```

继续来看registerReceiverInternal函数的注释4，其实是通过Binder调用的AMS的registerReceiver方法。

frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

**ActivityManagerService#registerReceiver()**

```
    public Intent registerReceiver(IApplicationThread caller, String callerPackage,
            IIntentReceiver receiver, IntentFilter filter, String permission, int userId) {

        ArrayList<Intent> stickyIntents = null;
        ProcessRecord callerApp = null;
        int callingUid;
        int callingPid;
        synchronized(this) {
            ...
            Iterator<String> actions = filter.actionsIterator(); //1
            if (actions == null) {
                ArrayList<String> noAction = new ArrayList<String>(1);
                noAction.add(null);
                actions = noAction.iterator();
            }

            // Collect stickies of users
            int[] userIds = { UserHandle.USER_ALL, UserHandle.getUserId(callingUid) };
            while (actions.hasNext()) {
                String action = actions.next();
                //2
                for (int id : userIds) {
                    ArrayMap<String, ArrayList<Intent>> stickies = mStickyBroadcasts.get(id);
                    if (stickies != null) {
                        ArrayList<Intent> intents = stickies.get(action);
                        if (intents != null) {
                            if (stickyIntents == null) {
                                stickyIntents = new ArrayList<Intent>();
                            }
                            stickyIntents.addAll(intents); 
                        }
                    }
                }
            }
        }

        ArrayList<Intent> allSticky = null;
        if (stickyIntents != null) {
            final ContentResolver resolver = mContext.getContentResolver();
            // Look for any matching sticky broadcasts...
            for (int i = 0, N = stickyIntents.size(); i < N; i++) {
                Intent intent = stickyIntents.get(i);
                // If intent has scheme "content", it will need to acccess
                // provider that needs to lock mProviderMap in ActivityThread
                // and also it may need to wait application response, so we
                // cannot lock ActivityManagerService here.
                if (filter.match(resolver, intent, true, TAG) >= 0) {
                    if (allSticky == null) {
                        allSticky = new ArrayList<Intent>();
                    }
                    allSticky.add(intent); //3
                }
            }
        }

        // The first sticky in the list is returned directly back to the client.
        Intent sticky = allSticky != null ? allSticky.get(0) : null;
        if (DEBUG_BROADCAST) Slog.v(TAG_BROADCAST, "Register receiver " + filter + ": " + sticky);
        if (receiver == null) {
            return sticky;
        }

        synchronized (this) {
            if (callerApp != null && (callerApp.thread == null
                    || callerApp.thread.asBinder() != caller.asBinder())) {
                // Original caller already died
                return null;
            }
            ReceiverList rl = mRegisteredReceivers.get(receiver.asBinder()); //4
            if (rl == null) {
                rl = new ReceiverList(this, callerApp, callingPid, callingUid,
                        userId, receiver);
                if (rl.app != null) {
                    rl.app.receivers.add(rl);
                } else {
                    try {
                        receiver.asBinder().linkToDeath(rl, 0);
                    } catch (RemoteException e) {
                        return sticky;
                    }
                    rl.linkedToDeath = true;
                }
                mRegisteredReceivers.put(receiver.asBinder(), rl);
            }
            ...
            BroadcastFilter bf = new BroadcastFilter(filter, rl, callerPackage,
                    permission, callingUid, userId); //5
            rl.add(bf); //6
            if (!bf.debugCheck()) {
                Slog.w(TAG, "==> For Dynamic broadcast");
            }
            mReceiverResolver.addFilter(bf); //7

            // Enqueue broadcasts for all existing stickies that match
            // this filter.
            if (allSticky != null) {
                ArrayList receivers = new ArrayList();
                receivers.add(bf);

                final int stickyCount = allSticky.size();
                for (int i = 0; i < stickyCount; i++) {
                    Intent intent = allSticky.get(i);
                    BroadcastQueue queue = broadcastQueueForIntent(intent);
                    BroadcastRecord r = new BroadcastRecord(queue, intent, null,
                            null, -1, -1, null, null, AppOpsManager.OP_NONE, null, receivers,
                            null, 0, null, null, false, true, true, -1);
                    queue.enqueueParallelBroadcastLocked(r);
                    queue.scheduleBroadcastsLocked();
                }
            }

            return sticky;
        }
    }

```

注释1处根据传入的IntentFilter类型的filter得到actions列表。
注释2处根据actions列表和userIds（userIds可以理解为应用程序的uid）得到所有的粘性广播的intent，并传入到stickyIntents中。
注释3处将这些粘性广播的intent存入到allSticky列表中，从这里可以看出粘性广播是存储在AMS中的。
注释4处获取ReceiverList列表，ReceiverList继承自ArrayList，用来存储广播接收者。
注释5处创建BroadcastFilter并传入此前创建的ReceiverList，BroadcastFilter用来描述注册的广播接收者。
注释6通过add方法将注释5创建的BroadcastFilter添加到ReceiverList中。
注释7处将BroadcastFilter添加到mReceiverResolver中，这样当AMS接收到广播时就可以从mReceiverResolver中找到对应的广播接收者了。

时序图：

![image](//upload-images.jianshu.io/upload_images/19956127-84095c0beab939c6.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

## 广播的发送

广播有很多类型：无序广播（普通广播）、有序广播、粘性广播和APP内部广播，这里以普通广播为例，来讲解广播的发送过程。
要发送普通广播需要调用sendBroadcast方法，它的实现同样在ContextWrapper中：

frameworks/base/core/java/android/content/ContextWrapper.java

**ContextWrapper#sendBroadcast()**

```
    @Override
    public void sendBroadcast(Intent intent) {
        mBase.sendBroadcast(intent);
    }

```

frameworks/base/core/java/android/app/ContextImpl.java

**ContextImpl#sendBroadcast()**

```
    @Override
    public void sendBroadcast(Intent intent) {
        warnIfCallingFromSystemProcess();
        String resolvedType = intent.resolveTypeIfNeeded(getContentResolver());
        try {
            intent.prepareToLeaveProcess(this);
            //1
            ActivityManagerNative.getDefault().broadcastIntent(
                    mMainThread.getApplicationThread(), intent, resolvedType, null,
                    Activity.RESULT_OK, null, null, null, AppOpsManager.OP_NONE, null, false, false,
                    getUserId());
        } catch (RemoteException e) {
            throw e.rethrowFromSystemServer();
        }
    }

```

注释1处最终会调用AMS的broadcastIntent方法。

frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

**ActivityManagerService#broadcastIntent()**

```
    public final int broadcastIntent(IApplicationThread caller,
            Intent intent, String resolvedType, IIntentReceiver resultTo,
            int resultCode, String resultData, Bundle resultExtras,
            String[] requiredPermissions, int appOp, Bundle bOptions,
            boolean serialized, boolean sticky, int userId) {
        synchronized(this) {
            intent = verifyBroadcastLocked(intent); //1

            final ProcessRecord callerApp = getRecordForAppLocked(caller);
            final int callingPid = Binder.getCallingPid();
            final int callingUid = Binder.getCallingUid();
            final long origId = Binder.clearCallingIdentity();
            //2
            int res = broadcastIntentLocked(callerApp,
                    callerApp != null ? callerApp.info.packageName : null,
                    intent, resolvedType, resultTo, resultCode, resultData, resultExtras,
                    requiredPermissions, appOp, bOptions, serialized, sticky,
                    callingPid, callingUid, userId);
            Binder.restoreCallingIdentity(origId);
            return res;
        }
    }

```

注释1处通过verifyBroadcastLocked方法验证广播是否合法。
注释2处调用broadcastIntentLocked方法。

先看看注释1的verifyBroadcastLocked方法：

**ActivityManagerService#verifyBroadcastLocked()**

```
final Intent verifyBroadcastLocked(Intent intent) {
        // Refuse possible leaked file descriptors
        if (intent != null && intent.hasFileDescriptors() == true) { //1
            throw new IllegalArgumentException("File descriptors passed in Intent");
        }

        int flags = intent.getFlags(); //2

        if (!mProcessesReady) {
            // if the caller really truly claims to know what they're doing, go
            // ahead and allow the broadcast without launching any receivers
            if ((flags&Intent.FLAG_RECEIVER_REGISTERED_ONLY_BEFORE_BOOT) != 0) { //3
                // This will be turned into a FLAG_RECEIVER_REGISTERED_ONLY later on if needed.
            } else if ((flags&Intent.FLAG_RECEIVER_REGISTERED_ONLY) == 0) {  //4
                Slog.e(TAG, "Attempt to launch receivers of broadcast intent " + intent
                        + " before boot completion");
                throw new IllegalStateException("Cannot broadcast before boot completed");
            }
        }

        if ((flags&Intent.FLAG_RECEIVER_BOOT_UPGRADE) != 0) {
            throw new IllegalArgumentException(
                    "Can't use FLAG_RECEIVER_BOOT_UPGRADE here");
        }

        return intent;
    }

```

注释1处验证intent是否不为null并且有文件描述符。
注释2处获得intent中的flag。
注释3处如果系统正在启动过程中，判断如果flag设置为FLAG_RECEIVER_REGISTERED_ONLY_BEFORE_BOOT（启动检查时只接受动态注册的广播接收者）则不做处理，如果不是则在注释4处判断如果flag没有设置为FLAG_RECEIVER_REGISTERED_ONLY（只接受动态注册的广播接收者）则会抛出异常。

继续来看broadcastIntent的注释2。

**ActivityManagerService#broadcastIntentLocked()**

```
final int broadcastIntentLocked(ProcessRecord callerApp,
            String callerPackage, Intent intent, String resolvedType,
            IIntentReceiver resultTo, int resultCode, String resultData,
            Bundle resultExtras, String[] requiredPermissions, int appOp, Bundle bOptions,
            boolean ordered, boolean sticky, int callingPid, int callingUid, int userId) {
  ...
        // Figure out who all will receive this broadcast.
        List receivers = null;
        ...
        if (intent.getComponent() == null) {
            if (userId == UserHandle.USER_ALL && callingUid == Process.SHELL_UID) {
                // Query one target user at a time, excluding shell-restricted users
                for (int i = 0; i < users.length; i++) {
                    ...
                    List<BroadcastFilter> registeredReceiversForUser =
                            mReceiverResolver.queryIntent(intent,
                                    resolvedType, false, users[i]);  //1
                    if (registeredReceivers == null) {
                        registeredReceivers = registeredReceiversForUser;
                    } else if (registeredReceiversForUser != null) {
                        registeredReceivers.addAll(registeredReceiversForUser);
                    }
                }
            } else {
                registeredReceivers = mReceiverResolver.queryIntent(intent,
                        resolvedType, false, userId);
            }
        }

        ...

        // Merge into one list.
        int ir = 0;
        if (receivers != null) {
           ...

            int NT = receivers != null ? receivers.size() : 0;
            int it = 0;
            ResolveInfo curt = null;
            BroadcastFilter curr = null;
            //2
            while (it < NT && ir < NR) {
                if (curt == null) {
                    curt = (ResolveInfo)receivers.get(it);
                }
                if (curr == null) {
                    curr = registeredReceivers.get(ir);
                }
                if (curr.getPriority() >= curt.priority) {
                    // Insert this broadcast record into the final list.
                    receivers.add(it, curr);
                    ir++;
                    curr = null;
                    it++;
                    NT++;
                } else {
                    // Skip to the next ResolveInfo in the final list.
                    it++;
                    curt = null;
                }
            }
        }
        while (ir < NR) {
            if (receivers == null) {
                receivers = new ArrayList();
            }
            receivers.add(registeredReceivers.get(ir));
            ir++;
        }

        if ((receivers != null && receivers.size() > 0)
                || resultTo != null) {
            BroadcastQueue queue = broadcastQueueForIntent(intent);
            //3
            BroadcastRecord r = new BroadcastRecord(queue, intent, callerApp,
                    callerPackage, callingPid, callingUid, resolvedType,
                    requiredPermissions, appOp, brOptions, receivers, resultTo, resultCode,
                    resultData, resultExtras, ordered, sticky, false, userId);

            if (DEBUG_BROADCAST) Slog.v(TAG_BROADCAST, "Enqueueing ordered broadcast " + r
                    + ": prev had " + queue.mOrderedBroadcasts.size());
            if (DEBUG_BROADCAST) Slog.i(TAG_BROADCAST,
                    "Enqueueing broadcast " + r.intent.getAction());

            boolean replaced = replacePending && queue.replaceOrderedBroadcastLocked(r);
            if (!replaced) {
                queue.enqueueOrderedBroadcastLocked(r);
                queue.scheduleBroadcastsLocked(); //4
            }
        } else {
            ...
        }
        return ActivityManager.BROADCAST_SUCCESS;
}

```

代码很长，我们这里只捡重点看。
注释1通过mReceiverResolver查询到所有符合条件的广播接收器，这些接收器是在广播注册的时候放到mReceiverResolver中的，具体可以参考广播注册过程。
注释2将注释1获得的所有广播接收器按照优先级高低存储到receivers列表中，这样receivers列表包含了所有的广播接收者（无序广播和有序广播）。
注释3处创建BroadcastRecord对象并将receivers传进去。
注释4处调用BroadcastQueue的scheduleBroadcastsLocked方法。

frameworks/base/services/core/java/com/android/server/am/BroadcastQueue.java

**BroadcastQueue#scheduleBroadcastsLocked()**

```
    public void scheduleBroadcastsLocked() {
        if (mBroadcastsScheduled) {
            return;
        }
        mHandler.sendMessage(mHandler.obtainMessage(BROADCAST_INTENT_MSG, this)); //1
        mBroadcastsScheduled = true;
    }

```

注释1处向BroadcastHandler类型的mHandler对象发送了BROADCAST_INTENT_MSG类型的消息，这个消息在BroadcastHandler的handleMessage方法中进行处理。

**BroadcastQueue#BroadcastHandler**

```
private final class BroadcastHandler extends Handler {
        public BroadcastHandler(Looper looper) {
            super(looper, null, true);
        }

        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case BROADCAST_INTENT_MSG: {
                    if (DEBUG_BROADCAST) Slog.v(
                            TAG_BROADCAST, "Received BROADCAST_INTENT_MSG");
                    processNextBroadcast(true);
                } break;
                ...
            }
        }
    }

```

注释1执行processNextBroadcast方法。processNextBroadcast方法对无序广播和有序广播分别进行处理，旨在将广播发送给广播接收者，下面给出processNextBroadcast方法中对无序广播的处理部分。

**BroadcastQueue#processNextBroadcast()**

```
final void processNextBroadcast(boolean fromMsg) {
...
          if (fromMsg) {
              mBroadcastsScheduled = false;//1
          }
          // First, deliver any non-serialized broadcasts right away.
            while (mParallelBroadcasts.size() > 0) { //2
                r = mParallelBroadcasts.remove(0); //3
                r.dispatchTime = SystemClock.uptimeMillis();
                r.dispatchClockTime = System.currentTimeMillis();
                final int N = r.receivers.size();
                if (DEBUG_BROADCAST_LIGHT) Slog.v(TAG_BROADCAST, "Processing parallel broadcast ["
                        + mQueueName + "] " + r);
                for (int i=0; i<N; i++) {
                    Object target = r.receivers.get(i);
                    if (DEBUG_BROADCAST)  Slog.v(TAG_BROADCAST,
                            "Delivering non-ordered on [" + mQueueName + "] to registered "
                            + target + ": " + r);
                    deliverToRegisteredReceiverLocked(r, (BroadcastFilter)target, false, i); //4
                }
                addBroadcastToHistoryLocked(r);
                if (DEBUG_BROADCAST_LIGHT) Slog.v(TAG_BROADCAST, "Done with parallel broadcast ["
                        + mQueueName + "] " + r);
            }
            ...
}

```

从前面的代码我们得知fromMsg的值为true，因此注释1处会将mBroadcastsScheduled 设置为flase，表示对于此前发来的BROADCAST_INTENT_MSG类型的消息已经处理了。
注释2处的mParallelBroadcasts列表用来存储无序广播，通过while循环将mParallelBroadcasts列表中的无序广播发送给对应的广播接收者。
注释3处获取每一个mParallelBroadcasts列表中存储的BroadcastRecord类型的r对象。
注释4处将这些r对象描述的广播发送给对应的广播接收者。

**BroadcastQueue#deliverToRegisteredReceiverLocked**

```
    private void deliverToRegisteredReceiverLocked(BroadcastRecord r,
            BroadcastFilter filter, boolean ordered, int index) {
        ...
        try {
            if (filter.receiverList.app != null && filter.receiverList.app.inFullBackup) {
                // Skip delivery if full backup in progress
                // If it's an ordered broadcast, we need to continue to the next receiver.
                if (ordered) {
                    skipReceiverLocked(r);
                }
            } else {
                performReceiveLocked(filter.receiverList.app, filter.receiverList.receiver,
                        new Intent(r.intent), r.resultCode, r.resultData,
                        r.resultExtras, r.ordered, r.initialSticky, r.userId);  //1
            }
            if (ordered) {
                r.state = BroadcastRecord.CALL_DONE_RECEIVE;
            }
        } catch (RemoteException e) {
            ...
        }
    }

```

省略了很多代码，这些代码是用来检查广播发送者和广播接收者的权限。如果通过了权限的检查，则会调用注释1处的performReceiveLocked方法。

**BroadcastQueue#performReceiveLocked**

```
    void performReceiveLocked(ProcessRecord app, IIntentReceiver receiver,
            Intent intent, int resultCode, String data, Bundle extras,
            boolean ordered, boolean sticky, int sendingUser) throws RemoteException {
        // Send the intent to the receiver asynchronously using one-way binder calls.
        if (app != null) {  //1
            if (app.thread != null) {  //2
                // If we have an app thread, do the call through that so it is
                // correctly ordered with other one-way calls.
                try {
                    app.thread.scheduleRegisteredReceiver(receiver, intent, resultCode,
                            data, extras, ordered, sticky, sendingUser, app.repProcState);  //3
                } catch (RemoteException ex) {
                    ...
                }
            } else {
                // Application has died. Receiver doesn't exist.
                throw new RemoteException("app.thread must not be null");
            }
        } else {
            receiver.performReceive(intent, resultCode, data, extras, ordered,
                    sticky, sendingUser);
        }
    }

```

注释1和2处的代码表示如果广播接收者所在的应用程序进程存在并且正在运行。
注释3处的代码，表示用广播接收者所在的应用程序进程来接收广播，这里app.thread指的是ActivityThread的内部类ApplicationThread。

时序图：

![image](//upload-images.jianshu.io/upload_images/19956127-a2f01fb27ccbc66a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

## 广播的接收

frameworks/base/core/java/android/app/ActivityThread.java

**ApplicationThread#scheduleRegisteredReceiver()**

```
        public void scheduleRegisteredReceiver(IIntentReceiver receiver, Intent intent,
                int resultCode, String dataStr, Bundle extras, boolean ordered,
                boolean sticky, int sendingUser, int processState) throws RemoteException {
            updateProcessState(processState, false);
            receiver.performReceive(intent, resultCode, dataStr, extras, ordered,
                    sticky, sendingUser);  //1
        }

```

注释1处调用了IIntentReceiver类型的对象receiver的performReceive方法，这里实现receiver的类为LoadedApk.ReceiverDispatcher.InnerReceiver。

frameworks/base/core/java/android/app/LoadedApk.java

**ReceiverDispatcher**

```
static final class ReceiverDispatcher {

        final static class InnerReceiver extends IIntentReceiver.Stub {
            final WeakReference<LoadedApk.ReceiverDispatcher> mDispatcher;
            final LoadedApk.ReceiverDispatcher mStrongRef;

            InnerReceiver(LoadedApk.ReceiverDispatcher rd, boolean strong) {
                mDispatcher = new WeakReference<LoadedApk.ReceiverDispatcher>(rd);
                mStrongRef = strong ? rd : null;
            }

            @Override
            public void performReceive(Intent intent, int resultCode, String data,
                    Bundle extras, boolean ordered, boolean sticky, int sendingUser) {
                final LoadedApk.ReceiverDispatcher rd;
                ...
                if (rd != null) {
                    rd.performReceive(intent, resultCode, data, extras,
                            ordered, sticky, sendingUser);  //1
                } else {
                    ...
                }
            }
        }

```

注释1处调用了ReceiverDispatcher类型的rd对象的performReceive方法。

**ReceiverDispatcher#performReceive()**

```
        public void performReceive(Intent intent, int resultCode, String data,
                Bundle extras, boolean ordered, boolean sticky, int sendingUser) {
            final Args args = new Args(intent, resultCode, data, extras, ordered,
                    sticky, sendingUser);  //1
            if (intent == null) {
                Log.wtf(TAG, "Null intent received");
            } else {
                if (ActivityThread.DEBUG_BROADCAST) {
                    int seq = intent.getIntExtra("seq", -1);
                    Slog.i(ActivityThread.TAG, "Enqueueing broadcast " + intent.getAction()
                            + " seq=" + seq + " to " + mReceiver);
                }
            }
            if (intent == null || !mActivityThread.post(args)) {  //2
                if (mRegistered && ordered) {
                    IActivityManager mgr = ActivityManagerNative.getDefault();
                    if (ActivityThread.DEBUG_BROADCAST) Slog.i(ActivityThread.TAG,
                            "Finishing sync broadcast to " + mReceiver);
                    args.sendFinished(mgr);
                }
            }
        }

```

注释1处将广播的intent等信息封装为Args对象。
注释2处调用mActivityThread的post方法并传入了Args对象。这个mActivityThread是一个Handler对象，具体指向的就是H，注释2处的代码就是将Args对象通过H发送到主线程的消息队列中。Args继承自Runnable，这个消息最终会在Args的run方法执行。

**LoadedApk#Args**

```
        final class Args extends BroadcastReceiver.PendingResult implements Runnable {
            private Intent mCurIntent;
            private final boolean mOrdered;
            private boolean mDispatched;

            public Args(Intent intent, int resultCode, String resultData, Bundle resultExtras,
                    boolean ordered, boolean sticky, int sendingUser) {
                super(resultCode, resultData, resultExtras,
                        mRegistered ? TYPE_REGISTERED : TYPE_UNREGISTERED, ordered,
                        sticky, mIIntentReceiver.asBinder(), sendingUser, intent.getFlags());
                mCurIntent = intent;
                mOrdered = ordered;
            }

            public void run() {
                final BroadcastReceiver receiver = mReceiver;

                final IActivityManager mgr = ActivityManagerNative.getDefault();
                final Intent intent = mCurIntent;

                try {
                    ClassLoader cl =  mReceiver.getClass().getClassLoader();
                    intent.setExtrasClassLoader(cl);
                    intent.prepareToEnterProcess();
                    setExtrasClassLoader(cl);
                    receiver.setPendingResult(this);
                    receiver.onReceive(mContext, intent); //1
                } catch (Exception e) {
                    ...
                }
            }
        }

```

注释1处执行了广播接收者的onReceive方法，这样注册的广播接收者就收到了广播并得到了intent。

时序图：

![image](//upload-images.jianshu.io/upload_images/19956127-9461cd225a635c04.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

**推荐阅读：[做了六年Android，终于熬出头了，15K到31K全靠这份高级面试题+解析](https://www.jianshu.com/p/14fa0c6ed6e8)**


>原文作者：huaxun66
原文链接：[https://blog.csdn.net/huaxun66/article/details/78190002](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fhuaxun66%2Farticle%2Fdetails%2F78190002)

更多系列教程GitHub学习地址：[https://github.com/Timdk857/Android-Architecture-knowledge-2-](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FTimdk857%2FAndroid-Architecture-knowledge-2-)

