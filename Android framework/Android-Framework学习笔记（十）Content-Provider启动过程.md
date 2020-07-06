## query方法到AMS的调用

在上述文章中我用到了Content Provider查询数据库的例子如下：

```
private void query() {
        Cursor cursor = this.getContentResolver().query(mCurrentURI, null, null, null, null);  //1
        showlog("count=" + cursor.getCount());
        cursor.moveToFirst();
        while (!cursor.isAfterLast()) {
            String table = cursor.getString(cursor.getColumnIndex("table_name"));
            String name = cursor.getString(cursor.getColumnIndex("name"));
            String detail = cursor.getString(cursor.getColumnIndex("detail"));
            showlog("table_name:" + table);
            showlog("name: " + name);
            showlog("detail: " + detail);
            cursor.moveToNext();
        }
        cursor.close();
    }

```

要想调用Content Provider，首先需要使用注释1处的getContentResolver方法，如下所示。

frameworks/base/core/Java/android/content/ContextWrapper.java

**ContextWrapper#getContentResolver()**

```
    @Override
    public ContentResolver getContentResolver() {
        return mBase.getContentResolver();
    }

```

[Framework学习（八）Service的启动/绑定过程](https://www.jianshu.com/p/fa6cd0b6f5c3)
这篇文章中我们已经分析了，mBase具体指向就是ContextImpl。

frameworks/base/core/java/android/app/ContextImpl.java

**ContextImpl#getContentResolver()**

```
    private final ApplicationContentResolver mContentResolver;
    ...
    @Override
    public ContentResolver getContentResolver() {
        return mContentResolver;
    }

```

上面代码返回了ApplicationContentResolver类型的mContentResolver对象，ApplicationContentResolver是ContextImpl中的静态内部类，继承自ContentResolver，它在ContextImpl的构造方法中被创建。

当我们调用ContentResolver的insert、query、update、delete等方法时就会启动Content Provider，这里拿query方法来进行举例。query方法的实现在ApplicationContentResolver的父类ContentResolver中。

frameworks/base/core/java/android/content/ContentResolver.java

**ContentResolver#query()**

```
public final @Nullable Cursor query(final @RequiresPermission.Read @NonNull Uri uri,
            @Nullable String[] projection, @Nullable String selection,
            @Nullable String[] selectionArgs, @Nullable String sortOrder,
            @Nullable CancellationSignal cancellationSignal) {
        Preconditions.checkNotNull(uri, "uri");
        IContentProvider unstableProvider = acquireUnstableProvider(uri);  //1
        ...
        try {
           ...
            try {
                qCursor = unstableProvider.query(mPackageName, uri, projection,
                        selection, selectionArgs, sortOrder, remoteCancellationSignal);  //2
            } catch (DeadObjectException e) {
               ...
            }
    ...
   }

```

注释1处通过acquireUnstableProvider方法返回IContentProvider类型的unstableProvider对象。
注释2处调用unstableProvider的query方法。
先看看注释1的方法吧。

**ContentResolver#acquireUnstableProvider()**

```
    public final IContentProvider acquireUnstableProvider(Uri uri) {
        if (!SCHEME_CONTENT.equals(uri.getScheme())) { //1
            return null;
        }
        String auth = uri.getAuthority();
        if (auth != null) {
            return acquireUnstableProvider(mContext, uri.getAuthority());  //2
        }
        return null;
    }

```

注释1处用来检查Uri的scheme是否等于”content”，如果不是则返回null。
注释2处调用了acquireUnstableProvider方法，这是个抽象方法，它的实现在ContentResolver的子类ApplicationContentResolver中。

frameworks/base/core/java/android/app/ContextImpl.java

**ApplicationContentResolver#acquireUnstableProvider()**

```
@Override
protected IContentProvider acquireUnstableProvider(Context c, String auth) {
    return mMainThread.acquireProvider(c,
            ContentProvider.getAuthorityWithoutUserId(auth),
            resolveUserIdFromAuthority(auth), false);
}

```

返回了ActivityThread类型的mMainThread对象的acquireProvider方法。

frameworks/base/core/java/android/app/ActivityThread.java

**ActivityThread#acquireProvider()**

```
public final IContentProvider acquireProvider(
         Context c, String auth, int userId, boolean stable) {
     final IContentProvider provider = acquireExistingProvider(c, auth, userId, stable);  //1
     if (provider != null) {
         return provider;
     }
     IActivityManager.ContentProviderHolder holder = null;
     try {
         holder = ActivityManagerNative.getDefault().getContentProvider(
                 getApplicationThread(), auth, userId, stable);  //2
     } catch (RemoteException ex) {
         throw ex.rethrowFromSystemServer();
     }
     if (holder == null) {
         Slog.e(TAG, "Failed to find provider info for " + auth);
         return null;
     }
     holder = installProvider(c, holder, holder.info,
             true /*noisy*/, holder.noReleaseNeeded, stable);  //3
     return holder.provider;
 }

```

注释1处检查ActivityThread中的ArrayMap类型的mProviderMap中是否有目标ContentProvider存在，有则返回，没有就会在注释2处调用AMP的getContentProvider方法，最终会调用AMS的getContentProvider方法。
注释3处的installProvider方法用来将注释2处返回的ContentProvider相关的数据存储在mProviderMap中，起到缓存的作用，这样使用相同的Content Provider时，就不需要每次都要调用AMS的getContentProvider方法。

## AMS到ActivityThread的调用

frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

**ActivityManagerService#getContentProvider()**

```
@Override
public final ContentProviderHolder getContentProvider(
        IApplicationThread caller, String name, int userId, boolean stable) {
 ...
    return getContentProviderImpl(caller, name, null, stable, userId);
}

```

**ActivityManagerService#getContentProviderImpl()**

```
private ContentProviderHolder getContentProviderImpl(IApplicationThread caller,
            String name, IBinder token, boolean stable, int userId) {
...
       ProcessRecord proc = getProcessRecordLocked(
                                cpi.processName, cpr.appInfo.uid, false); //1
                        if (proc != null && proc.thread != null && !proc.killed) {
                            ...
                            if (!proc.pubProviders.containsKey(cpi.name)) {
                                checkTime(startTime, "getContentProviderImpl: scheduling install");
                                proc.pubProviders.put(cpi.name, cpr);
                                try {
                                    proc.thread.scheduleInstallProvider(cpi); //2
                                } catch (RemoteException e) {
                                }
                            }
                        } else {
                            checkTime(startTime, "getContentProviderImpl: before start process");
                            proc = startProcessLocked(cpi.processName,
                                    cpr.appInfo, false, 0, "content provider",
                                    new ComponentName(cpi.applicationInfo.packageName,
                                            cpi.name), false, false, false); //3
                            checkTime(startTime, "getContentProviderImpl: after start process");
                          ...
                        }
             ...           

}

```

注释1处通过getProcessRecordLocked方法来获取目标ContentProvider的应用程序进程信息，这些信息用ProcessRecord类型的proc来表示，如果该应用进程已经启动就会调用注释2处的代码，否则就会调用注释3的startProcessLocked方法来启动进程。
应用程序进程启动过程请参考Framework学习（六）应用程序进程启动过程这篇文章。

## ActivityThread启动Provider

frameworks/base/services/core/java/com/android/app/ActivityThread.java

**ActivityThread#scheduleInstallProvider()**

```
        @Override
        public void scheduleInstallProvider(ProviderInfo provider) {
            sendMessage(H.INSTALL_PROVIDER, provider);
        }

```

这里的H是ActivityThread的内部类并继承Handler。

## ActivityThread.H

```
 private class H extends Handler {
        public static final int INSTALL_PROVIDER        = 145;
  ...

  public void handleMessage(Message msg) {
            if (DEBUG_MESSAGES) Slog.v(TAG, ">>> handling: " + codeToString(msg.what));
            switch (msg.what) {
                case INSTALL_PROVIDER:
                    handleInstallProvider((ProviderInfo) msg.obj);
                    break;
                 ...
  } 

```

**ActivityThread#handleInstallProvider()**

```
    public void handleInstallProvider(ProviderInfo info) {
        final StrictMode.ThreadPolicy oldPolicy = StrictMode.allowThreadDiskWrites();
        try {
            installContentProviders(mInitialApplication, Lists.newArrayList(info)); //1
        } finally {
            StrictMode.setThreadPolicy(oldPolicy);
        }
    }

```

注释1调用了installContentProviders方法。

**ActivityThread#installContentProviders()**

```
    private void installContentProviders(
            Context context, List<ProviderInfo> providers) {
        final ArrayList<IActivityManager.ContentProviderHolder> results =
            new ArrayList<IActivityManager.ContentProviderHolder>();

        for (ProviderInfo cpi : providers) {  //1
            if (DEBUG_PROVIDER) {
                StringBuilder buf = new StringBuilder(128);
                buf.append("Pub ");
                buf.append(cpi.authority);
                buf.append(": ");
                buf.append(cpi.name);
                Log.i(TAG, buf.toString());
            }
            IActivityManager.ContentProviderHolder cph = installProvider(context, null, cpi,
                    false /*noisy*/, true /*noReleaseNeeded*/, true /*stable*/);  //2
            if (cph != null) {
                cph.noReleaseNeeded = true;
                results.add(cph);
            }
        }

        try {
            ActivityManagerNative.getDefault().publishContentProviders(
                getApplicationThread(), results);  //3
        } catch (RemoteException ex) {
            throw ex.rethrowFromSystemServer();
        }
    }

```

注释1处遍历当前应用程序进程的ProviderInfo列表，得到每个Content Provider的ProviderInfo（存储Content Provider的信息）。
注释2处调用installProvider方法来启动这些Content Provider。
注释3处通过AMS的publishContentProviders方法将这些Content Provider存储在AMS的mProviderMap中，这个mProviderMap在前面提到过，起到缓存的作用，防止每次使用相同的Content Provider时都会调用AMS的getContentProvider方法。

**ActivityThread#installProvider()**

```
private IActivityManager.ContentProviderHolder installProvider(Context context,
           IActivityManager.ContentProviderHolder holder, ProviderInfo info,
           boolean noisy, boolean noReleaseNeeded, boolean stable) {
       ContentProvider localProvider = null;
  ...
               final java.lang.ClassLoader cl = c.getClassLoader();
               localProvider = (ContentProvider)cl.
                   loadClass(info.name).newInstance(); //1
               provider = localProvider.getIContentProvider();
               if (provider == null) {
                 ...
                   return null;
               }
               if (DEBUG_PROVIDER) Slog.v(
                   TAG, "Instantiating local provider " + info.name);
               localProvider.attachInfo(c, info); //2
           } catch (java.lang.Exception e) {
              ...
               }
               return null;
           }
       }
          ...
       return retHolder;
   }

```

注释1处通过反射来创建ContentProvider类型的localProvider对象。
注释2处调用了它的attachInfo方法。

frameworks/base/core/java/android/content/ContentProvider.java

## ContentProvider#attachInfo()

```
public void attachInfo(Context context, ProviderInfo info) {
        attachInfo(context, info, false);
    }

    private void attachInfo(Context context, ProviderInfo info, boolean testing) {
        mNoPerms = testing;

        /*
         * Only allow it to be set once, so after the content service gives
         * this to us clients can't change it.
         */
        if (mContext == null) {
            mContext = context;
            if (context != null) {
                mTransport.mAppOpsManager = (AppOpsManager) context.getSystemService(
                        Context.APP_OPS_SERVICE);
            }
            mMyUid = Process.myUid();
            if (info != null) {
                setReadPermission(info.readPermission);
                setWritePermission(info.writePermission);
                setPathPermissions(info.pathPermissions);
                mExported = info.exported;
                mSingleUser = (info.flags & ProviderInfo.FLAG_SINGLE_USER) != 0;
                setAuthorities(info.authority);
            }
            ContentProvider.this.onCreate();  //1
        }
    }

```

注释1处调用了ContentProvider的onCreate方法，它是一个抽象方法，具体实现在具体Content Provider中。这样Content Provider就启动完毕了。

>原文作者：huaxun66
原文链接：[https://blog.csdn.net/huaxun66/article/details/78194820](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fhuaxun66%2Farticle%2Fdetails%2F78194820)

更多系列教程GitHub学习地址：[https://github.com/Timdk857/Android-Architecture-knowledge-2-](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FTimdk857%2FAndroid-Architecture-knowledge-2-)
