## 概述

上篇文章Android Framework学习笔记（五）应用程序启动过程
我们讲解了应用程序启动过程，但是还有一个遗留知识点，那就是应用程序进程的启动。我们知道应用程序启动之前需要保证应用程序的进程先启动，本文我们就来看看应用程序进程的启动过程。

AMS在启动应用程序时会检查这个应用程序需要的应用程序进程是否存在，不存在就会请求Zygote进程将需要的应用程序进程启动。在Framework学习（二）Zygote进程启动过程这篇文章中，我分析了Zygote会创建一个Server端的Socket，这个Socket用来等待AMS来请求Zygote来创建新的应用程序进程的。我们知道Zygote进程通过fock自身创建应用程序进程，这样应用程序进程就会获得Zygote进程在启动时创建的虚拟机实例。当然，在应用程序创建过程中除了获取虚拟机实例外，还可以获得Binder线程池和消息循环，这样运行在应用进程中的应用程序就可以方便的使用Binder进行进程间通信以及消息处理机制了。

## 发送请求创建应用程序进程

先从上篇文章中ActivityStackSupervisor的startSpecificActivityLocked函数说起：

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

AMS启动应用程序时，会检查这个应用程序需要的应用程序进程是否存在。
注释1处获取当前Activity所在的进程的ProcessRecord，如果进程已经启动了，会执行注释2处的代码。否则执行注释3的代码，通过调用startProcessLocked函数来向Zygote进程发送请求。

frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

**ActivityManagerService#startProcessLocked()**

```
private final void startProcessLocked(ProcessRecord app, String hostingType, String hostingNameStr, String abiOverride, String entryPoint, String[] entryPointArgs) {
      ...
      try {
          try {
              final int userId = UserHandle.getUserId(app.uid);
              AppGlobals.getPackageManager().checkPackageStartable(app.info.packageName, userId);
          } catch (RemoteException e) {
              throw e.rethrowAsRuntimeException();
          }
          int uid = app.uid; //1
          int[] gids = null;
          int mountExternal = Zygote.MOUNT_EXTERNAL_NONE;
          if (!app.isolated) {
            ...
              //2
              if (ArrayUtils.isEmpty(permGids)) {
                  gids = new int[2];
              } else {
                  gids = new int[permGids.length + 2];
                  System.arraycopy(permGids, 0, gids, 2, permGids.length);
              }
              gids[0] = UserHandle.getSharedAppGid(UserHandle.getAppId(uid));
              gids[1] = UserHandle.getUserGid(UserHandle.getUserId(uid));
          }

         ...
          if (entryPoint == null) entryPoint = "android.app.ActivityThread"; //3
          Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "Start proc: " + app.processName);
          checkTime(startTime, "startProcess: asking zygote to start proc");
          //4
          Process.ProcessStartResult startResult = Process.start(entryPoint, app.processName, uid, uid, gids, debugFlags, mountExternal, app.info.targetSdkVersion, app.info.seinfo, requiredAbi, instructionSet, app.info.dataDir, entryPointArgs);
         ...
      } catch (RuntimeException e) {
        ...
      }
  }
 ...
  }

```

注释1处的代码创建应用程序进程的用户ID。
注释2处对用户组ID：gids进行创建和赋值。
注释3处如果entryPoint 为null则赋为”android.app.ActivityThread”。
注释4处调用Process的start函数，将此前得到的应用程序进程用户ID、用户组ID和entryPoint 传进去。

frameworks/base/core/java/android/os/Process.java

**Process#start()**

```
    public static final ProcessStartResult start(final String processClass,
                                  final String niceName,
                                  int uid, int gid, int[] gids,
                                  int debugFlags, int mountExternal,
                                  int targetSdkVersion,
                                  String seInfo,
                                  String abi,
                                  String instructionSet,
                                  String appDataDir,
                                  String[] zygoteArgs) {
        try {
            //1
            return startViaZygote(processClass, niceName, uid, gid, gids, debugFlags, mountExternal, targetSdkVersion, seInfo, abi, instructionSet, appDataDir, zygoteArgs);
        } catch (ZygoteStartFailedEx ex) {
            Log.e(LOG_TAG, "Starting VM process through Zygote failed");
            throw new RuntimeException("Starting VM process through Zygote failed", ex);
        }
    }

```

**Process#startViaZygote()**

```
private static ProcessStartResult startViaZygote(final String processClass,
                               final String niceName,
                               final int uid, final int gid,
                               final int[] gids,
                               int debugFlags, int mountExternal,
                               int targetSdkVersion,
                               String seInfo,
                               String abi,
                               String instructionSet,
                               String appDataDir,
                               String[] extraArgs)
                               throws ZygoteStartFailedEx {
     synchronized(Process.class) {
         //1
         ArrayList<String> argsForZygote = new ArrayList<String>();
         argsForZygote.add("--runtime-args");
         argsForZygote.add("--setuid=" + uid);
         argsForZygote.add("--setgid=" + gid);
       ...
         if (gids != null && gids.length > 0) {
             StringBuilder sb = new StringBuilder();
             sb.append("--setgroups=");
             int sz = gids.length;
             for (int i = 0; i < sz; i++) {
                 if (i != 0) {
                     sb.append(',');
                 }
                 sb.append(gids[i]);
             }
             argsForZygote.add(sb.toString());
         }
      ...
         argsForZygote.add(processClass);
         if (extraArgs != null) {
             for (String arg : extraArgs) {
                 argsForZygote.add(arg);
             }
         }
         return zygoteSendArgsAndGetResult(openZygoteSocketIfNeeded(abi), argsForZygote);
     }
 }

```

注释1处创建了字符串列表argsForZygote ，并将启动应用进程的启动参数保存在argsForZygote中，例如uid、gid等，这里需要注意processClass = android.app.ActivityThread，后文会用到。
注释2会调用zygoteSendArgsAndGetResult函数，需要注意的是，zygoteSendArgsAndGetResult函数中第一个参数中调用了openZygoteSocketIfNeeded函数，而第二个参数是保存应用进程的启动参数的argsForZygote。

**Process#zygoteSendArgsAndGetResult()**

```
    private static ProcessStartResult zygoteSendArgsAndGetResult(ZygoteState zygoteState, ArrayList<String> args) throws ZygoteStartFailedEx {
        try {
            // Throw early if any of the arguments are malformed. This means we can
            // avoid writing a partial response to the zygote.
            int sz = args.size();
            for (int i = 0; i < sz; i++) {
                if (args.get(i).indexOf('\n') >= 0) {
                    throw new ZygoteStartFailedEx("embedded newlines not allowed");
                }
            }
            final BufferedWriter writer = zygoteState.writer;
            final DataInputStream inputStream = zygoteState.inputStream;

            writer.write(Integer.toString(args.size()));
            writer.newLine();

            for (int i = 0; i < sz; i++) {
                String arg = args.get(i);
                writer.write(arg);
                writer.newLine();
            }

            writer.flush();

            // Should there be a timeout on this?
            ProcessStartResult result = new ProcessStartResult();
            result.pid = inputStream.readInt();
            result.usingWrapper = inputStream.readBoolean();

            if (result.pid < 0) {
                throw new ZygoteStartFailedEx("fork() failed");
            }
            return result;
        } catch (IOException ex) {
            zygoteState.close();
            throw new ZygoteStartFailedEx(ex);
        }
    }

```

zygoteSendArgsAndGetResult函数主要做的就是将传入的应用进程的启动参数argsForZygote写入到ZygoteState中，结合上文我们知道ZygoteState其实是由openZygoteSocketIfNeeded函数返回的，那么我们接着来看openZygoteSocketIfNeeded函数。

**Process#openZygoteSocketIfNeeded()**

```
private static ZygoteState openZygoteSocketIfNeeded(String abi) throws ZygoteStartFailedEx {
        if (primaryZygoteState == null || primaryZygoteState.isClosed()) {
            try {
                primaryZygoteState = ZygoteState.connect(ZYGOTE_SOCKET); //1
            } catch (IOException ioe) {
                throw new ZygoteStartFailedEx("Error connecting to primary zygote", ioe);
            }
        }

        if (primaryZygoteState.matches(abi)) { 
            return primaryZygoteState;
        }

        // The primary zygote didn't match. Try the secondary.
        if (secondaryZygoteState == null || secondaryZygoteState.isClosed()) {
            try {
            secondaryZygoteState = ZygoteState.connect(SECONDARY_ZYGOTE_SOCKET);  //2
            } catch (IOException ioe) {
                throw new ZygoteStartFailedEx("Error connecting to secondary zygote", ioe);
            }
        }

        if (secondaryZygoteState.matches(abi)) {
            return secondaryZygoteState;
        }

        throw new ZygoteStartFailedEx("Unsupported zygote ABI: " + abi);
    }

```

在之前讲Zygote进程启动过程时我们得知，在Zygote的main函数中会创建name为“zygote”的Server端Socket。
注释1处会调用ZygoteState的connect函数与名称为ZYGOTE_SOCKET的Socket建立连接，这里ZYGOTE_SOCKET的值为“zygote”。
注释2处如果连接name为“zygote”的Socket返回的primaryZygoteState与当前的abi不匹配，则会连接name为“zygote_secondary”的Socket。这两个Socket区别就是：name为”zygote”的Socket是运行在64位Zygote进程中的，而name为“zygote_secondary”的Socket则运行在32位Zygote进程中。既然应用程序进程是通过Zygote进程fock产生的，当要连接Zygote中的Socket时，也需要保证位数的一致。

## 接收请求并创建应用程序进程

Socket进行连接成功并匹配abi后会返回ZygoteState类型对象，我们在分析zygoteSendArgsAndGetResult函数中讲过，会将应用进程的启动参数argsForZygote写入到ZygoteState中，这样Zygote进程就会收到一个创建新的应用程序进程的请求，我们回到ZygoteInit的main函数。

frameworks/base/core/java/com/android/internal/os/ZygoteInit.java

**ZygoteInit#main()**

```
    public static void main(String argv[]) {
        ...
        try {
            String socketName = "zygote";
            /**注册Zygote用的Socket*/
            registerZygoteSocket(socketName); //1
            //预加载类和资源
            preload(); //2
            ...
            //启动SystemServer进程
            if (startSystemServer) {
                startSystemServer(abiList, socketName); //3
            }

            //监听socket,启动新的应用进程  
            runSelectLoop(abiList); //4
            closeServerSocket();
        } catch (MethodAndArgsCaller caller) {
            //通过反射调用SystemServer#main()
            caller.run();
        } catch (RuntimeException ex) {
            Log.e(TAG, "Zygote died with exception", ex);
            closeServerSocket();
            throw ex;
        }
    }

```

注释1处通过registerZygoteSocket函数来创建一个Server端的Socket，这个name为”zygote”的Socket用来等待AMS来请求Zygote来创建新的应用程序进程。
注释2处用来预加载类和资源。
注释3处用来启动SystemServer进程，这样系统的关键服务也会由SystemServer进程启动起来。
注释4处调用runSelectLoop函数来等待AMS的请求。

**ZygoteInit#runSelectLoop()**

```
private static void runSelectLoop(String abiList) throws MethodAndArgsCaller {
        ArrayList<FileDescriptor> fds = new ArrayList<FileDescriptor>();
        ArrayList<ZygoteConnection> peers = new ArrayList<ZygoteConnection>();

        fds.add(sServerSocket.getFileDescriptor()); //1
        peers.add(null);

        while (true) {
            StructPollfd[] pollFds = new StructPollfd[fds.size()];
            for (int i = 0; i < pollFds.length; ++i) {  //2
                pollFds[i] = new StructPollfd();
                pollFds[i].fd = fds.get(i);
                pollFds[i].events = (short) POLLIN;
            }
            try {
                Os.poll(pollFds, -1);
            } catch (ErrnoException ex) {
                throw new RuntimeException("poll failed", ex);
            }
            for (int i = pollFds.length - 1; i >= 0; --i) {  //3
                if ((pollFds[i].revents & POLLIN) == 0) {
                    continue;
                }
                if (i == 0) {
                    ZygoteConnection newPeer = acceptCommandPeer(abiList);  //4
                    peers.add(newPeer);
                    fds.add(newPeer.getFileDesciptor());
                } else {
                    boolean done = peers.get(i).runOnce();  //5
                    if (done) {
                        peers.remove(i);
                        fds.remove(i);
                    }
                }
            }
        }
    }

    private static ZygoteConnection acceptCommandPeer(String abiList) {
        try {
            return new ZygoteConnection(sServerSocket.accept(), abiList);
        } catch (IOException ex) {
            ...
        }
    }

```

注释1处中的sServerSocket就是我们在registerZygoteSocket函数中创建的服务端Socket，调用sServerSocket.getFileDescriptor()用来获得该Socket的fd字段的值并添加到fd列表fds中。接下来无限循环用来等待AMS请求Zygote进程创建新的应用程序进程。
注释2处通过遍历将fds存储的信息转移到pollFds数组中。
注释3处对pollFds进行遍历。
注释4如果i==0则说明服务端Socket与客户端连接上，也就是当前Zygote进程与AMS建立了连接，则通过acceptCommandPeer函数得到ZygoteConnection类并添加到Socket连接列表peers中，接着将该ZygoteConnection的fd添加到fd列表fds中，以便可以接收到AMS发送过来的请求。
注释5如果i的值大于0，则说明AMS向Zygote进程发送了一个创建应用进程的请求，则调用ZygoteConnection的runOnce函数来创建一个新的应用程序进程。并在成功创建后将这个连接从Socket连接列表peers和fd列表fds中清除。

frameworks/base/core/java/com/android/internal/os/ZygoteConnection.java

**ZygoteConnection#runOnce()**

```
boolean runOnce() throws ZygoteInit.MethodAndArgsCaller {
        String args[];
        Arguments parsedArgs = null;
        FileDescriptor[] descriptors;
        try {
            args = readArgumentList(); //1
            descriptors = mSocket.getAncillaryFileDescriptors();
        } catch (IOException ex) {
            Log.w(TAG, "IOException on command socket " + ex.getMessage());
            closeSocket();
            return true;
        }
...
        try {
            parsedArgs = new Arguments(args);//2
        ...
            //3
            pid = Zygote.forkAndSpecialize(parsedArgs.uid, parsedArgs.gid, parsedArgs.gids,
                    parsedArgs.debugFlags, rlimits, parsedArgs.mountExternal, parsedArgs.seInfo,
                    parsedArgs.niceName, fdsToClose, parsedArgs.instructionSet,
                    parsedArgs.appDataDir);
        } catch (ErrnoException ex) {
          ....
        }
       try { 
            //4
            if (pid == 0) {
                // in child
                IoUtils.closeQuietly(serverPipeFd);
                serverPipeFd = null;
                handleChildProc(parsedArgs, descriptors, childPipeFd, newStderr);
                return true;
            } else {
                // in parent...pid of < 0 means failure
                IoUtils.closeQuietly(childPipeFd);
                childPipeFd = null;
                return handleParentProc(pid, descriptors, serverPipeFd, parsedArgs);
            }
        } finally {
            IoUtils.closeQuietly(childPipeFd);
            IoUtils.closeQuietly(serverPipeFd);
        }
    }

```

注释1处调用readArgumentList函数来获取应用程序进程的启动参数。注释2处将readArgumentList函数返回的字符串封装到Arguments对象parsedArgs中。
注释3处调用Zygote的forkAndSpecialize函数来创建应用程序进程，参数为parsedArgs中存储的应用进程启动参数，返回值为pid。
注释4处forkAndSpecialize函数主要是通过fork当前进程来创建一个子进程的，如果pid等于0，则说明是在新创建的子进程中执行的，就会调用handleChildProc函数来启动这个子进程也就是应用程序进程。

ZygoteConnection#handleChildProc()

```
private void handleChildProc(Arguments parsedArgs, FileDescriptor[] descriptors, FileDescriptor pipeFd, PrintStream newStderr)
           throws ZygoteInit.MethodAndArgsCaller {
     ...
                   //1
                   if (parsedArgs.invokeWith != null) {
            WrapperInit.execApplication(parsedArgs.invokeWith,
                    parsedArgs.niceName, parsedArgs.targetSdkVersion,
                    VMRuntime.getCurrentInstructionSet(),
                    pipeFd, parsedArgs.remainingArgs);
        } else {
            RuntimeInit.zygoteInit(parsedArgs.targetSdkVersion,
                    parsedArgs.remainingArgs, null /* classLoader */);
        }
       }
   }

```

注释1处由于parsedArgs.invokeWith属性默认为null，最后调用RuntimeInit.zygoteInit函数。

frameworks/base/core/java/com/android/internal/os/RuntimeInit.java

**RuntimeInit#zygoteInit()**

```
    public static final void zygoteInit(int targetSdkVersion, String[] argv, ClassLoader classLoader)
            throws ZygoteInit.MethodAndArgsCaller {
        if (DEBUG) Slog.d(TAG, "RuntimeInit: Starting application from zygote");

        Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "RuntimeInit");
        redirectLogStreams();

        commonInit();
        nativeZygoteInit(); //1
        applicationInit(targetSdkVersion, argv, classLoader); //2
    }

```

注释1处会在新创建的应用程序进程中创建Binder线程池。
注释2处调用了applicationInit函数。

**RuntimeInit#applicationInit()**

```
private static void applicationInit(int targetSdkVersion, String[] argv, ClassLoader classLoader)
            throws ZygoteInit.MethodAndArgsCaller {
        ...

        // 初始化虚拟机环境
        VMRuntime.getRuntime().setTargetHeapUtilization(0.75f);
        VMRuntime.getRuntime().setTargetSdkVersion(targetSdkVersion);

        final Arguments args;
        try {
            args = new Arguments(argv);
        } catch (IllegalArgumentException ex) {
            Slog.e(TAG, ex.getMessage());
            // let the process exit
            return;
        }

        // Remaining arguments are passed to the start class's static main
        invokeStaticMain(args.startClass, args.startArgs, classLoader); //1
    }

```

注释1处applicationInit函数中主要调用了invokeStaticMain函数，需要注意的是第一个参数args.startClass，这里指的就是前面赋值的android.app.ActivityThread。

**RuntimeInit#invokeStaticMain()**

```
private static void invokeStaticMain(String className, String[] argv, ClassLoader classLoader) throws ZygoteInit.MethodAndArgsCaller {
        Class<?> cl;

        try {
            cl = Class.forName(className, true, classLoader); //1
        } catch (ClassNotFoundException ex) {
            throw new RuntimeException("Missing class when invoking static main " + className, ex);
        }

        Method m;
        try {
            // 获取main方法
            m = cl.getMethod("main", new Class[] { String[].class }); //2
        } catch (NoSuchMethodException ex) {
            throw new RuntimeException("Missing static main on " + className, ex);
        } catch (SecurityException ex) {
            throw new RuntimeException("Problem getting static main on " + className, ex);
        }
        // 判断修饰符
        int modifiers = m.getModifiers(); //3
        if (! (Modifier.isStatic(modifiers) && Modifier.isPublic(modifiers))) {
            throw new RuntimeException("Main method is not public and static on " + className);
        }

        /*
         * This throw gets caught in ZygoteInit.main(), which responds
         * by invoking the exception's run() method. This arrangement
         * clears up all the stack frames that were required in setting
         * up the process.
         */
        throw new ZygoteInit.MethodAndArgsCaller(m, argv); //4
    }

```

注释1处通过反射来获得android.app.ActivityThread类。
注释2处来获得ActivityThread的main函数。
注释3判断修饰符，必须是static而且必须是public类型。
注释4将找到的main函数传入到MethodAndArgsCaller异常中并抛出该异常。这个异常在ZygoteInit#main()方法中捕获。这么做的作用是清除应用程序进程创建过程的调用栈。

**ZygoteInit#main()**

```
public static void main(String argv[]) {
    try {
        ...
        startSystemServer(abiList, socketName);
        ...
    } catch (MethodAndArgsCaller caller) {
        caller.run(); //1
    }
}

```

在注释1处调用了MethodAndArgsCaller的run函数。

**MethodAndArgsCaller**

```
public static class MethodAndArgsCaller extends Exception
            implements Runnable {
        /** method to call */
        private final Method mMethod;

        /** argument array */
        private final String[] mArgs;

        public MethodAndArgsCaller(Method method, String[] args) {
            mMethod = method;
            mArgs = args;
        }

        public void run() {
            try {
                mMethod.invoke(null, new Object[] { mArgs }); //1
            } catch (IllegalAccessException ex) {
                throw new RuntimeException(ex);
            } catch (InvocationTargetException ex) {
                Throwable cause = ex.getCause();
                if (cause instanceof RuntimeException) {
                    throw (RuntimeException) cause;
                } else if (cause instanceof Error) {
                    throw (Error) cause;
                }
                throw new RuntimeException(ex);
            }
        }
    }

```

注释1处通过反射调用了android.app.ActivityThread#main(String[] args)。至此，Zygote进程fork出ActivityThread进程，并成功调用ActivityThread#main()。

frameworks/base/core/java/android/app/ActivityThread.java

**ActivityThread#main()**

```
public static void main(String[] args) {
        Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");
        SamplingProfilerIntegration.start();
...
        Looper.prepareMainLooper();//1
        ActivityThread thread = new ActivityThread();//2
        thread.attach(false); //3
        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler(); //4
        }
        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        Looper.loop();//5
        throw new RuntimeException("Main thread loop unexpectedly exited");
    }

```

注释1处在当前应用程序进程中创建消息循环。
注释2处创建ActivityThread实例。
注释4处从ActivityThread获取handler，这样就将ActivityThread关联到Looper和MessageQueue了。
注释5处调用Looper的loop，使得Looper开始工作，开始处理消息。可以看出，系统在应用程序进程启动完成后，就会创建一个消息循环，用来方便的使用Android的异步消息处理机制。
关于Android的异步消息处理机制大家可以参考我这篇文章：Android 异步消息处理机制：Looper、Handler、Message。这里就不多说了。

如果你看过[Framework学习（三）SyetemServer进程启动过程](https://www.jianshu.com/p/f7515f031865)
这篇文章，你肯定会发现创建应用程序进程和创建SyetemServer进程的步骤如此类似，其实他们都是通过Zygote进程fork自身来实现的，当然步骤差不多了。唯一区别是一个最终调用到SyetemServer的main函数，另一个最终调用到ActivityThread的main函数。

之前说过：在Android系统中，启动四大组件中的任何一个都可以启动应用程序。但实际上应用程序入口方法只有ActivityThread#main()一个。接着看ActivityThread#main()方法。
注释3处调用了ActivityThread#attach(false)。

**ActivityThread#attach()**

```
    final ApplicationThread mAppThread = new ApplicationThread();

    private void attach(boolean system) {
        sCurrentActivityThread = this;
        mSystemThread = system;
        if (!system) {
            ...
            final IActivityManager mgr = ActivityManagerNative.getDefault();
            try {
                mgr.attachApplication(mAppThread); //1
            } catch (RemoteException ex) {
                // Ignore
            }
            ...
        }
        ...
    }

```

前面文章分析过AMS中的Binder机制：ActivityManagerNative.getDefault()，返回的其实是AMS。注释1其实调用的是AMS的attachApplication方法。

frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

**ActivityManagerService#attachApplication()**

```
public final void attachApplication(IApplicationThread thread) {
        synchronized (this) {
            int callingPid = Binder.getCallingPid();
            final long origId = Binder.clearCallingIdentity();
            attachApplicationLocked(thread, callingPid); //1
            Binder.restoreCallingIdentity(origId);
        }
    }

```

注释1调用了两个参数的attachApplicationLocked()方法

**ActivityManagerService#attachApplicationLocked()**

```
    private final boolean attachApplicationLocked(IApplicationThread thread,int pid) {
        ...
        if (normalMode) {
            try {
                if (mStackSupervisor.attachApplicationLocked(app)) { //1
                    didSomething = true;
                }
            } catch (Exception e) {
                Slog.wtf(TAG, "Exception thrown launching activities in " + app, e);
                badApp = true;
            }
        }
        ...
    }

```

注释1调用ActivityStackSupervisor的attachApplicationLocked方法。

frameworks/base/services/core/java/com/android/server/am/ActivityStackSupervisor.java

**ActivityStackSupervisor#attachApplicationLocked()**

```
boolean attachApplicationLocked(ProcessRecord app) throws RemoteException {
        final String processName = app.processName;
        boolean didSomething = false;
        for (int displayNdx = mActivityDisplays.size() - 1; displayNdx >= 0; --displayNdx) {
            ArrayList<ActivityStack> stacks = mActivityDisplays.valueAt(displayNdx).mStacks;
            for (int stackNdx = stacks.size() - 1; stackNdx >= 0; --stackNdx) {
                final ActivityStack stack = stacks.get(stackNdx);
                if (!isFrontStack(stack)) {
                    continue;
                }
                ActivityRecord hr = stack.topRunningActivityLocked(null);
                if (hr != null) {
                    if (hr.app == null && app.uid == hr.info.applicationInfo.uid
                            && processName.equals(hr.processName)) {
                        try {
                            if (realStartActivityLocked(hr, app, true, true)) {
                                didSomething = true;
                            }
                        } catch (RemoteException e) {
                            ...
                        }
                    }
                }
            }
        }
        ...
        return didSomething;
    }

```

首先遍历所有stack，之后找到目前被置于前台的stack。之后通过topRunningActivityLocked()获取最上面的Activity。最后调用realStartActivityLocked()方法来真正的启动目标Activity。
之后就是应用程序启动过程了，具体请看Framework学习（五）应用程序启动过程这篇文章。

下面看看应用程序进程启动时序图：

![image](//upload-images.jianshu.io/upload_images/19956127-6d6f620286ac6623.png?imageMogr2/auto-orient/strip|imageView2/2/w/836/format/webp)

## 总体流程

1.AMS通知Process使用Socket和Zygote进程通信，请求创建一个新进程.
2.Zygote收到Socket请求，fork出一个进程，会在新的进程中创建Binder线程池并反射调用ActivityThread#main().
3.ActivityThread通过Binder通知AMS启动应用程序.

**推荐阅读：[做了六年Android，终于熬出头了，15K到31K全靠这份高级面试题+解析](https://www.jianshu.com/p/14fa0c6ed6e8)**

>原文作者：huaxun66
原文链接：[https://blog.csdn.net/huaxun66/article/details/78155674](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fhuaxun66%2Farticle%2Fdetails%2F78155674)

更多系列教程GitHub学习地址：[https://github.com/Timdk857/Android-Architecture-knowledge-2-](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FTimdk857%2FAndroid-Architecture-knowledge-2-)
