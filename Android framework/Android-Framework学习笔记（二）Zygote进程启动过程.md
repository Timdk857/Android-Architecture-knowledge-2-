## Android系统进程启动流程

android系统的Zygote进程是所有android进程的父进程，包括SystemServer和各种应用进程都是通过Zygote进程fork出来的。Zygote（孵化）进程相当于是android系统的根进程，后面所有的进程都是通过这个进程fork出来的，而Zygote进程则是通过linux系统的init进程启动的，也就是说，android系统中各种进程的启动方式：

init进程 ––> Zygote进程 ––> SystemServer进程 ––>各种应用进程

**init进程**
linux的根进程，android系统是基于linux系统的，因此可以算作是整个android操作系统的第一个进程；
**Zygote进程**
android系统的根进程，可以作用Zygote进程fork出SystemServer进程和各种应用进程；
**SystemService进程**
主要是在这个进程中启动系统的各项服务，比如ActivityManagerService，PackageManagerService，WindowManagerService服务等；
**各种应用进程**
启动自己编写的客户端应用时，一般都是重新启动一个应用进程，有自己的虚拟机与运行环境；

## Zygote进程的执行过程

其中Zygote进程由init进程启动，SystemServer进程和应用进程由Zygote进程启动。本文依据7.0源码，主要分析Zygote进程的启动流程。init进程在启动Zygote进程时会调用ZygoteInit#main()。

frameworks/base/core/java/com/android/internal/os/ZygoteInit.java

**ZygoteInit#main()**

```
    public static void main(String argv[]) {
        ...

        try {
            //设置DDMS可用
            RuntimeInit.enableDdms(); //1
            //初始化启动参数
            boolean startSystemServer = false;
            String socketName = "zygote";
            String abiList = null;
            //2
            for (int i = 1; i < argv.length; i++) {
                if ("start-system-server".equals(argv[i])) {
                    startSystemServer = true;
                } else if (argv[i].startsWith(ABI_LIST_ARG)) {
                    abiList = argv[i].substring(ABI_LIST_ARG.length());
                } else if (argv[i].startsWith(SOCKET_NAME_ARG)) {
                    socketName = argv[i].substring(SOCKET_NAME_ARG.length());
                } else {
                    throw new RuntimeException("Unknown command line argument: " + argv[i]);
                }
            }

            if (abiList == null) {
                throw new RuntimeException("No ABI list supplied.");
            }
            //注册Zygote用的Socket
            registerZygoteSocket(socketName); //3
            //预加载类和资源
            preload(); //4
            ...
            //启动SystemServer进程
            if (startSystemServer) {
                startSystemServer(abiList, socketName); //5
            }

            //监听socket,启动新的应用进程  
            runSelectLoop(abiList); //6
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

注释1设置DDMS可用，可以发现DDMS启动的时机还是比较早的，在整个Zygote进程刚刚开始要启动的时候就设置可用了。
注释2的循环主要是解析main方法的参数获取是否需要启动SystemService进程，获取abi列表，获取scoket连接名称 。（这里需要注意的是：android系统中进程之间通讯的方式是Binder，但是有一个例外是SystemService进程与Zygote进程之间是通过Socket的方式进行通讯的）
注释3处通过registerZygoteSocket函数来创建一个Server端的Socket，这个name为”zygote”的Socket用来等待ActivityManagerService来请求Zygote来创建新的应用程序进程。
注释4处用来预加载类和资源。
注释5处用来启动SystemServer进程，这样系统的关键服务也会由SystemServer进程启动起来。
注释6处调用runSelectLoop函数监听socket来等待客户端请求。
由此可见，ZygoteInit的main函数主要做了4件事（注释3、4、5、6），接下来我们对主要的事件一一进行分析。

## 注册Server端的Socket

**ZygoteInit#registerZygoteSocket()**

```
    private static void registerZygoteSocket(String socketName) {
        if (sServerSocket == null) {
            int fileDesc;
            final String fullSocketName = ANDROID_SOCKET_PREFIX + socketName;
            try {
                String env = System.getenv(fullSocketName);
                fileDesc = Integer.parseInt(env);
            } catch (RuntimeException ex) {
                throw new RuntimeException(fullSocketName + " unset or invalid", ex);
            }

            try {
                FileDescriptor fd = new FileDescriptor();
                fd.setInt$(fileDesc);
                //不是使用IP和端口、而是使用fd创建socket
                sServerSocket = new LocalServerSocket(fd); //1
            } catch (IOException ex) {
                throw new RuntimeException(
                        "Error binding to local socket '" + fileDesc + "'", ex);
            }
        }
    }

```

注释1处用来创建LocalServerSocket，也就是服务端的Socket。当Zygote进程将SystemServer进程启动后，就会在这个服务端的Socket上来等待ActivityManagerService请求Zygote进程来创建新的应用程序进程。

## 预加载类和资源

**ZygoteInit#preload()**

```
    static void preload() {
        beginIcuCachePinning();
        preloadClasses(); //加载所需的各种class文件
        preloadResources(); //加载资源文件
        preloadOpenGL(); //初始化OpenGL
        preloadSharedLibraries(); //加载系统Libraries
        preloadTextResources(); //加载文字资源
        // Ask the WebViewFactory to do any initialization that must run in the zygote process,
        // for memory sharing purposes.
        WebViewFactory.prepareWebViewInZygote(); //初始化WebView
        endIcuCachePinning();
        warmUpJcaProviders();
        Log.d(TAG, "end preload");
    }

```

## 启动SystemServer进程

**ZygoteInit#startSystemServer()**

```
private static boolean startSystemServer(String abiList, String socketName)
            throws MethodAndArgsCaller, RuntimeException {
        long capabilities = posixCapabilitiesAsBits(
            OsConstants.CAP_IPC_LOCK,
            OsConstants.CAP_KILL,
            OsConstants.CAP_NET_ADMIN,
            OsConstants.CAP_NET_BIND_SERVICE,
            OsConstants.CAP_NET_BROADCAST,
            OsConstants.CAP_NET_RAW,
            OsConstants.CAP_SYS_MODULE,
            OsConstants.CAP_SYS_NICE,
            OsConstants.CAP_SYS_RESOURCE,
            OsConstants.CAP_SYS_TIME,
            OsConstants.CAP_SYS_TTY_CONFIG
        );
        /* Containers run without this capability, so avoid setting it in that case */
        if (!SystemProperties.getBoolean(PROPERTY_RUNNING_IN_CONTAINER, false)) {
            capabilities |= posixCapabilitiesAsBits(OsConstants.CAP_BLOCK_SUSPEND);
        }
        /* Hardcoded command line to start the system server */
        /* 1 */
        String args[] = {
            "--setuid=1000",
            "--setgid=1000",
            "--setgroups=1001,1002,1003,1004,1005,1006,1007,1008,1009,1010,1018,1021,1032,3001,3002,3003,3006,3007,3009,3010",
            "--capabilities=" + capabilities + "," + capabilities,
            "--nice-name=system_server",
            "--runtime-args",
            "com.android.server.SystemServer",
        };
        ZygoteConnection.Arguments parsedArgs = null;

        int pid;

        try {
            /* 2 */
            parsedArgs = new ZygoteConnection.Arguments(args); 
            /** 打开系统调试属性*/
            ZygoteConnection.applyDebuggerSystemProperty(parsedArgs);
            ZygoteConnection.applyInvokeWithSystemProperty(parsedArgs);

            /* 3 */
            // 请求fork SystemServer进程
            pid = Zygote.forkSystemServer(
                    parsedArgs.uid, parsedArgs.gid,
                    parsedArgs.gids,
                    parsedArgs.debugFlags,
                    null,
                    parsedArgs.permittedCapabilities,
                    parsedArgs.effectiveCapabilities);
        } catch (IllegalArgumentException ex) {
            throw new RuntimeException(ex);
        }

        // pid为0表示子进程，即SystemServer进程，从此SystemServer进程与Zygote进程分道扬镳
        if (pid == 0) {
            if (hasSecondZygote(abiList)) {
                waitForSecondaryZygote(socketName);
            }
            handleSystemServerProcess(parsedArgs); //4
        }

        return true;
    }

```

注释1处用来创建args数组，这个数组用来保存SystemServer的启动参数，其中可以看出SystemServer进程的用户id和用户组id被设置为1000；并且拥有用户组1001<sub>1010，1018、1021、1032、3001</sub>3010的权限；进程名为system_server；启动的类名为com.android.server.SystemServer。
注释2处将args数组封装成Arguments对象并供注释3的forkSystemServer函数调用。
注释3处调用Zygote的forkSystemServer，主要通过fork函数在当前进程创建一个子进程(也就是SystemServer进程)，如果返回的pid 为0，也就是表示在新创建的子进程中执行的，则执行注释4处的handleSystemServerProcess，反射调用SystemServer#main()，来启动SystemServer进程。

具体handleSystemServerProcess是如何启动SystemServer进程的，请看Framework学习（三）SyetemServer进程启动过程这篇文章。

## 监听Socket,启动应用进程

**ZygoteInit#runSelectLoop()**

启动SystemServer进程后，最后进入runSelectLoop函数，如下所示。

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

```

注释1处中的sServerSocket就是我们在registerZygoteSocket函数中创建的服务端Socket，调用sServerSocket.getFileDescriptor()用来获得该Socket的fd字段的值并添加到fd列表fds中。接下来无限循环用来等待ActivityManagerService请求Zygote进程创建新的应用程序进程。
注释2处通过遍历将fds存储的信息转移到pollFds数组中。
注释3处对pollFds进行遍历。
注释4如果i==0则说明服务端Socket与客户端连接上，也就是当前Zygote进程与ActivityManagerService建立了连接，则通过acceptCommandPeer函数得到ZygoteConnection类并添加到Socket连接列表peers中，接着将该ZygoteConnection的fd添加到fd列表fds中，以便可以接收到ActivityManagerService发送过来的请求。
注释5如果i的值大于0，则说明ActivityManagerService向Zygote进程发送了一个创建应用进程的请求，则调用ZygoteConnection的runOnce函数来创建一个新的应用程序进程。并在成功创建后将这个连接从Socket连接列表peers和fd列表fds中清除。

## Zygote进程总结

Zygote进程共做了如下几件事：

1.初始化DDMS
2.通过registerZygoteSocket函数创建服务端Socket
3.加载class、resource、OpenGL、WebView等各种资源
4.fork并启动SystemServer进程
5.调用runSelectLoop()一直监听Socket信息
6.fork并启动应用进程

原文作者：huaxun66
原文链接：[https://blog.csdn.net/huaxun66/article/details/78140775](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fhuaxun66%2Farticle%2Fdetails%2F78140775)

更多系列教程GitHub学习地址：[https://github.com/Timdk857/Android-Architecture-knowledge-2-](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FTimdk857%2FAndroid-Architecture-knowledge-2-)
