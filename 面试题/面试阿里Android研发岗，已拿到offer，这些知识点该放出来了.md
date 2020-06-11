## 目录：

**1.Android基础**

**2.网络**

**3.Java 基础&数据结构&设计模式**

**4.Android 性能优化&Framework**

**5.Android 模块化&热修复&热更新&打包&混淆&压缩**

**6.音视频&FFmpeg&播放器**

**7.项目&HR**

![](https://upload-images.jianshu.io/upload_images/19956127-58abbd96b2ec4357.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 1.Android基础

1、什么是ANR 如何避免它？

答：在Android 上，如果你的应用程序有一段时间响应不够灵敏，系统会向用户显示一个对话框，这个对话框称作应用程序无响应（ANR：Application Not Responding）对话框。用户可以选择让程序继续运行，但是，他们在使用你的应用程序时，并不希望每次都要处理这个对话框。因此，在程序里对响应性能的设计很重要，这样，系统不会显示ANR 给用户。不同的组件发生ANR 的时间不一样，主线程（Activity、Service）是5 秒，BroadCastReceiver 是10 秒。解决方案：将所有耗时操作，比如访问网络，Socket 通信，查询大量SQL 语句，复杂逻辑计算等都放在子线程中去，然后通过handler.sendMessage、runonUITread、AsyncTask 等方式更新UI。无论如何都要确保用户界面操作的流畅度。如果耗时操作需要让用户等待，那么可以在界面上显示进度条。

2、View的绘制流程；自定义View如何考虑机型适配；自定义View的事件3、分发机制；View和ViewGroup分别有哪些事件分发相关的回调方法；自定义View如何提供获取View属性的接口；

3、Art和Dalvik对比；虚拟机原理，如何自己设计一个虚拟机(内存管理，类加载，双亲委派)；JVM内存模型及类加载机制；内存对象的循环引用及避免；

4、ddms 和 traceView；

5、内存回收机制与GC算法(各种算法的优缺点以及应用场景)；GC原理时机以及GC对象；内存泄露场景及解决方法；

6、四大组件及生命周期；ContentProvider的权限管理(读写分离，权限控制-精确到表级，URL控制)；Activity的四种启动模式对比；Activity状态保存于恢复；

7、什么是AIDL 以及如何使用；

8、请解释下在单线程模型中Message、Handler、Message Queue、Looper之间的关系；

9、Fragment生命周期；Fragment状态保存startActivityForResult是哪个类的方法，在什么情况下使用，如果在Adapter中使用应该如何解耦；

10、AsyncTask原理及不足；ntentService原理；

11、Activity 怎么和Service 绑定，怎么在Activity 中启动自己对应的Service；

12、请描述一下Service 的生命周期；

13、AstncTask+HttpClient与AsyncHttpClient有什么区别；

14、如何保证一个后台服务不被杀死；比较省电的方式是什么；

15、如何通过广播拦截和abort一条短信；广播是否可以请求网络；广播引起anr的时间限制；

16、进程间通信，AIDL；

17、事件分发中的onTouch 和onTouchEvent 有什么区别，又该如何使用？

18、说说ContentProvider、ContentResolver、ContentObserver 之间的关系；

19**、**请介绍下ContentProvider 是如何实现数据共享的；

20、Handler机制及底层实现；

21、Binder机制及底层实现；

22、ListView 中图片错位的问题是如何产生的；

23、在manifest 和代码中如何注册和使用BroadcastReceiver；

24、说说Activity、Intent、Service 是什么关系；

25、ApplicationContext和ActivityContext的区别；

26、一张Bitmap所占内存以及内存占用的计算；

27、Serializable 和Parcelable 的区别；

28、请描述一下BroadcastReceiver；

29、请描述一下Android 的事件分发机制；

30、请介绍一下NDK；

31、什么是NDK库，如何在jni中注册native函数，有几种注册方式；

32、AsyncTask 如何使用；

33、对于应用更新这块是如何做的？(灰度，强制更新，分区域更新)；

34、混合开发，RN，weex，H5，小程序(做Android的了解一些前端js等还是很有好处的)；

35、什么情况下会导致内存泄露；

36、如何对Android 应用进行性能分析以及优化；

37、说一款你认为当前比较火的应用并设计(直播APP)；

38、OOM的避免异常及解决方法；

39、屏幕适配的处理技巧都有哪些；

40、两个Activity 之间跳转时必然会执行的是哪几个方法？

答：一般情况下比如说有两个activity,分别叫A,B,当在A 里面激活B 组件的时候, A 会调用onPause()方法,然后B 调用onCreate() ,onStart(), onResume()。
这个时候B 覆盖了窗体, A 会调用onStop()方法. 如果B 是个透明的,或者是对话框的样式, 就不会调用A 的
onStop()方法。

## 2.网络

#### 网络协议模型

**应用层**：负责处理特定的应用程序细节 HTTP、FTP、DNS

**传输层**：为两台主机提供端到端的基础通信 TCP、UDP

**网络层**：控制分组传输、路由选择等 IP

**链路层**：操作系统设备驱动程序、网卡相关接口

#### TCP 和 UDP 区别

TCP 连接；可靠；有序；面向字节流；速度慢；较重量；全双工；适用于文件传输、浏览器等

*   全双工：A 给 B 发消息的同时，B 也能给 A 发
*   半双工：A 给 B 发消息的同时，B 不能给 A 发

UDP 无连接；不可靠；无序；面向报文；速度快；轻量；适用于即时通讯、视频通话等

#### TCP 三次握手

A：你能听到吗？ B：我能听到，你能听到吗？ A：我能听到，开始吧

A 和 B 两方都要能确保：我说的话，你能听到；你说的话，我能听到。所以需要三次握手

#### TCP 四次挥手

A：我说完了 B：我知道了，等一下，我可能还没说完 B：我也说完了 A：我知道了，结束吧

B 收到 A 结束的消息后 B 可能还没说完，没法立即回复结束标示，只能等说完后再告诉 A ：我说完了。

#### POST 和 GET 区别

Get 参数放在 url 中；Post 参数放在 request Body 中 Get 可能不安全，因为参数放在 url 中

#### HTTPS

HTTP 是超文本传输协议，明文传输；HTTPS 使用 SSL 协议对 HTTP 传输数据进行了加密

HTTP 默认 80 端口；HTTPS 默认 443 端口

优点：安全 缺点：费时、SSL 证书收费，加密能力还是有限的，但是比 HTTP 强多了

## 3.Java基础&数据结构&设计模式

1、集合类以及集合框架；HashMap与HashTable实现原理，线程安全性，hash冲突及处理算法；ConcurrentHashMap；

2、进程和线程的区别；

3、Java的并发、多线程、线程模型；

4、什么是线程池，如何使用?

答：线程池就是事先将多个线程对象放到一个容器中，当使用的时候就不用new 线程而是直接去池中拿线程即可，节省了开辟子线程的时间，提高的代码执行效率。

5、数据一致性如何保证；Synchronized关键字，类锁，方法锁，重入锁；

6、Java中实现多态的机制是什么；

7、如何将一个Java对象序列化到文件里；

8、说说你对Java反射的理解；

答：Java 中的反射首先是能够获取到Java 中要反射类的字节码， 获取字节码有三种方法，
1.Class.forName(className)

2.类名.class

3.this.getClass()。

然后将字节码中的方法，变量，构造函数等映射成相应的Method、Filed、Constructor 等类，这些类提供了丰富的方法可以被我们所使用。

4、同步的方法；多进程开发以及多进程应用场景；

5、在Java中wait和seelp方法的不同；

答：最大的不同是在等待时wait 会释放锁，而sleep 一直持有锁。wait 通常被用于线程间交互，sleep 通常被用于暂停执行。

synchronized 和volatile 关键字的作用；

答：1）保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。

2）禁止进行指令重排序。

volatile 本质是在告诉jvm 当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取；synchronized 则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。
1.volatile 仅能使用在变量级别；synchronized 则可以使用在变量、方法、和类级别的
2.volatile 仅能实现变量的修改可见性，并不能保证原子性；synchronized 则可以保证变量的修改可见性和原子性
3.volatile 不会造成线程的阻塞；synchronized 可能会造成线程的阻塞。
4.volatile 标记的变量不会被编译器优化；synchronized 标记的变量可以被编译器优化

6、服务器只提供数据接收接口，在多线程或多进程条件下，如何保证数据的有序到达；

7、ThreadLocal原理，实现及如何保证Local属性；

8、String StringBuilder StringBuffer对比；

9、你所知道的设计模式有哪些；

答：Java 中一般认为有23 种设计模式，我们不需要所有的都会，但是其中常用的几种设计模式应该去掌握。下面列出了所有的设计模式。需要掌握的设计模式我单独列出来了，当然能掌握的越多越好。
总体来说设计模式分为三大类：
创建型模式，共五种：工厂方法模式、抽象工厂模式、单例模式、建造者模式、原型模式。
结构型模式，共七种：适配器模式、装饰器模式、代理模式、外观模式、桥接模式、组合模式、享元模式。
行为型模式，共十一种：策略模式、模板方法模式、观察者模式、迭代子模式、责任链模式、命令模式、备忘录模式、状态模式、访问者模式、中介者模式、解释器模式。

10、Java如何调用c、c++语言；

11、接口与回调；回调的原理；写一个回调demo；

12、泛型原理，举例说明；解析与分派；

13、抽象类与接口的区别；应用场景；抽象类是否可以没有方法和属性；

14、静态属性和静态方法是否可以被继承？是否可以被重写？以及原因？

15、修改对象A的equals方法的签名，那么使用HashMap存放这个对象实例的时候，会调用哪个equals方法；

16、说说你对泛型的了解；

17、Java的异常体系；

18、如何控制某个方法允许并发访问线程的个数；

19、动态代理的区别，什么场景使用；

**数据结构与算法**

1、堆和栈在内存中的区别是什么(数据结构方面以及实际实现方面)；

2、最快的排序算法是哪个？给阿里2万多名员工按年龄排序应该选择哪个算法？堆和树的区别；写出快排代码；链表逆序代码；

3、求1000以内的水仙花数以及40亿以内的水仙花数；

4、子串包含问题(KMP 算法)写代码实现；

5、万亿级别的两个URL文件A和B，如何求出A和B的差集C,(Bit映射->hash分组->多文件读写效率->磁盘寻址以及应用层面对寻址的优化)

6蚁群算法与蒙特卡洛算法；

7、写出你所知道的排序算法及时空复杂度，稳定性；

8、百度POI中如何试下查找最近的商家功能(坐标镜像+R树)。

## 4.Android性能优化&Framwork

#### Activity 启动模式

*   standard 标准模式
*   singleTop 栈顶复用模式， 推送点击消息界面
*   singleTask 栈内复用模式， 首页
*   singleInstance 单例模式，单独位于一个任务栈中 拨打电话界面
    细节： taskAffinity：任务相关性，用于指定任务栈名称，默认为应用包名 allowTaskReparenting：允许转移任务栈

#### View 工作原理

*   DecorView (FrameLayout) LinearLayout titlebar Content 调用 setContentView 设置的 View

ViewRoot 的 performTraversals 方法调用触发开始 View 的绘制，然后会依次调用:

*   performMeasure：遍历 View 的 measure 测量尺寸
*   performLayout：遍历 View 的 layout 确定位置
*   performDraw：遍历 View 的 draw 绘制

#### 事件分发机制

*   一个 MotionEvent 产生后，按 Activity -> Window -> decorView -> View 顺序传递，View 传递过程就是事件分发，主要依赖三个方法:
*   dispatchTouchEvent：用于分发事件，只要接受到点击事件就会被调用，返回结果表示是否消耗了当前事件
*   onInterceptTouchEvent：用于判断是否拦截事件，当 ViewGroup 确定要拦截事件后，该事件序列都不会再触发调用此 ViewGroup 的 onIntercept
*   onTouchEvent：用于处理事件，返回结果表示是否处理了当前事件，未处理则传递给父容器处理
*   细节： 一个事件序列只能被一个 View 拦截且消耗 View 没有 onIntercept 方法，直接调用 onTouchEvent 处理 OnTouchListener 优先级比 OnTouchEvent 高，onClickListener 优先级最低 requestDisallowInterceptTouchEvent 可以屏蔽父容器 onIntercet 方法的调用

#### Window 、 WindowManager、WMS、SurfaceFlinger

*   **Window**：抽象概念不是实际存在的，而是以 View 的形式存在，通过 PhoneWindow 实现
*   **WindowManager**：外界访问 Window 的入口，内部与 WMS 交互是个 IPC 过程
*   **WMS**：管理窗口 Surface 的布局和次序，作为系统级服务单独运行在一个进程
*   **SurfaceFlinger**：将 WMS 维护的窗口按一定次序混合后显示到屏幕上

#### View 动画、帧动画及属性动画

**View 动画：**

*   作用对象是 View，可用 xml 定义，建议 xml 实现比较易读
*   支持四种效果：平移、缩放、旋转、透明度

**帧动画：**

*   通过 AnimationDrawable 实现，容易 OOM

**属性动画：**

*   可作用于任何对象，可用 xml 定义，Android 3 引入，建议代码实现比较灵活
*   包括 ObjectAnimator、ValuetAnimator、AnimatorSet
*   时间插值器：根据时间流逝的百分比计算当前属性改变的百分比
*   系统预置匀速、加速、减速等插值器
*   类型估值器：根据当前属性改变的百分比计算改变后的属性值
*   系统预置整型、浮点、色值等类型估值器
*   使用注意事项：
*   避免使用帧动画，容易OOM
*   界面销毁时停止动画，避免内存泄漏
*   开启硬件加速，提高动画流畅性 ，硬件加速：
*   将 cpu 一部分工作分担给 gpu ，使用 gpu 完成绘制工作
*   从工作分摊和绘制机制两个方面优化了绘制速度

#### Handler、MessageQueue、Looper

*   Handler：开发直接接触的类，内部持有 MessageQueue 和 Looper
*   MessageQueue：消息队列，内部通过单链表存储消息
*   Looper：内部持有 MessageQueue，循环查看是否有新消息，有就处理，没就阻塞
*   如何实现阻塞：通过 nativePollOnce 方法，基于 Linux epoll 事件管理机制
*   为什么主线程不会因为 Looper 阻塞：系统每 16ms 会发送一个刷新 UI 消息唤醒

#### MVC、MVP、MVVM

*   MVP：Model：处理数据；View：控制视图；Presenter：分离 Activity 和 Model
*   MVVM：Model：处理获取保存数据；View：控制视图；ViewModel：数据容器 使用 Jetpack 组件架构的 LiveData、ViewModel 便捷实现 MVVM

#### Serializable、Parcelable

*   Serializable ：Java 序列化方式，适用于存储和网络传输，serialVersionUID 用于确定反序列化和类版本是否一致，不一致时反序列化回失败
*   Parcelable ：Android 序列化方式，适用于组件通信数据传递，性能高，因为不像 Serializable 一样有大量反射操作，频繁 GC

#### Binder

*   Android 进程间通信的中流砥柱，基于客户端-服务端通信方式
*   使用 mmap 一次数据拷贝实现 IPC，传统 IPC：用户A空间->内核->用户B空间；mmap 将内核与用户B空间映射，实现直接从用户A空间->用户B空间
*   BinderPool 可避免创建多 Service

#### IPC 方式

*   Intent extras、Bundle：要求传递数据能被序列化，实现 Parcelable、Serializable ，适用于四大组件通信
*   文件共享：适用于交换简单的数据实时性不高的场景
*   AIDL：AIDL 接口实质上是系统提供给我们可以方便实现 BInder 的工具 Android Interface Definition Language，可实现跨进程调用方法 服务端：将暴漏给客户端的接口声明在 AIDL 文件中，创建 Service 实现 AIDL 接口并监听客户端连接请求 客户端：绑定服务端 Service ，绑定成功后拿到服务端 Binder 对象转为 AIDL 接口调用 RemoteCallbackList 实现跨进程接口监听，同个 Binder 对象做 key 存储客户端注册的 listener 监听 Binder 断开：1.Binder.linkToDeath 设置死亡代理；2\. onServiceDisconnected 回调
*   Messenger：基于 AIDL 实现，服务端串行处理，主要用于传递消息，适用于低并发一对多通信
*   ContentProvider：基于 Binder 实现，适用于一对多进程间数据共享
*   Socket：TCP、UDP，适用于网络数据交换

#### Android 系统启动流程

*   按电源键 -> 加载引导程序 BootLoader 到 RAM -> 执行 BootLoader 程序启动内核 -> 启动 init 进程 -> 启动 Zygote 和各种守护进程 ->
*   启动 System Server 服务进程开启 AMS、WMS 等 -> 启动 Launcher 应用进程

#### App 启动流程

Launcher 中点击一个应用图标 -> 通过 AMS 查找应用进程，若不存在就通过 Zygote 进程 fork

#### 进程保活

*   进程优先级：1.前台进程 ；2.可见进程；3.服务进程；4.后台进程；5.空进程
*   进程被 kill 场景：1.切到后台内存不足时被杀；2.切到后台厂商省电机制杀死；3.用户主动清理
*   保活方式： 1.Activity 提权：挂一个 1像素 Activity 将进程优先级提高到前台进程 2.Service 提权：启动一个前台服务（API>18会有正在运行通知栏） 3.广播拉活 4.Service 拉活 5.JobScheduler 定时任务拉活 6.双进程拉活

#### 网络优化及检测

*   速度：1.GZIP 压缩（okhttp 自动支持）；2.Protocol Buffer 替代 json；3.优化图片/文件流量；4.IP 直连省去 DNS 解析时间
*   成功率：1.失败重试策略；
*   流量：1.GZIP 压缩（okhttp 自动支持）；2.Protocol Buffer 替代 json；3.优化图片/文件流量；5.文件下载断点续传 ；6.缓存
*   协议层的优化，比如更优的 http 版本等
*   监控：Charles 抓包、Network Monitor 监控流量

#### UI卡顿优化

*   减少布局层级及控件复杂度，避免过度绘制
*   使用 include、merge、viewstub
*   优化绘制过程，避免在 Draw 中频繁创建对象、做耗时操作

#### 内存泄漏场景及规避

1.静态变量、单例强引跟生命周期相关的数据或资源，包括 EventBus 2.游标、IO 流等资源忘记主动释放 3.界面相关动画在界面销毁时及时暂停 4.内部类持有外部类引用导致的内存泄漏

*   handler 内部类内存泄漏规避：1.使用静态内部类+弱引用 2.界面销毁时清空消息队列
*   检测：Android Studio Profiler

#### LeakCanary 原理

*   通过弱引用和引用队列监控对象是否被回收
*   比如 Activity 销毁时开始监控此对象，检测到未被回收则主动 gc ，然后继续监控

#### OOM 场景及规避

*   加载大图：减小图片
*   内存泄漏：规避内存泄漏

## 5.Android 模块化&热修复&热更新&打包&混淆&压缩

#### Dalvik 和 ART

*   Dalvik 谷歌设计专用于 Android 平台的 Java 虚拟机，可直接运行 .dex 文件，适合内存和处理速度有限的系统 JVM 指令集是基于栈的；Dalvik 指令集是基于寄存器的，代码执行效率更优
*   ART Dalvik 每次运行都要将字节码转换成机器码；ART 在应用安装时就会转换成机器码，执行速度更快 ART 存储机器码占用空间更大，空间换时间

#### APK 打包流程

1.aapt 打包资源文件生成 R.java 文件；aidl 生成 java 文件 2.将 java 文件编译为 class 文件 3.将工程及第三方的 class 文件转换成 dex 文件 4.将 dex 文件、so、编译过的资源、原始资源等打包成 apk 文件 5.签名 6.资源文件对齐，减少运行时内存

#### App 安装过程

*   首先要解压 APK，资源、so等放到应用目录
*   Dalvik 会将 dex 处理成 ODEX ；ART 会将 dex 处理成 OAT；
*   OAT 包含 dex 和安装时编译的机器码

#### 组件化路由实现

ARoute：通过 APT 解析 @Route 等注解，结合 JavaPoet 生成路由表，即路由与 Activity 的映射关系

## 6.音视频&FFmpeg&播放器

#### FFmpeg

基于命令方式实现了一个音视频编辑 App： https://github.com/yhaolpz/FFmpegCmd

集成编译了 AAC、MP3、H264 编码器

#### 播放器原理

视频播放原理：（mp4、flv）-> 解封装 -> （mp3/aac、h264/h265）-> 解码 -> （pcm、yuv）-> 音视频同步 -> 渲染播放

音视频同步：

*   选择参考时钟源：音频时间戳、视频时间戳和外部时间三者选择一个作为参考时钟源（一般选择音频，因为人对音频更敏感，ijk 默认也是音频）
*   通过等待或丢帧将视频流与参考时钟源对齐，实现同步

#### IjkPlayer 原理

集成了 MediaPlayer、ExoPlayer 和 IjkPlayer 三种实现，其中 IjkPlayer 基于 FFmpeg 的 ffplay

音频输出方式：AudioTrack、OpenSL ES；视频输出方式：NativeWindow、OpenGL ES

## 
7.项目&HR

1\. 项目开发中遇到的最大的一个难题和挑战，你是如何解决的。（95% 会问到）

2\. 说说你开发最大的优势点（95% 会问到）

3\. 你为什么会离开上家公司

4\. 你的缺点是什么？

5\. 你能给公司带来什么效益？

6\. 你对未来的职业规划？

## 文末

对于程序员来说，要学习的知识内容、技术有太多太多，要想不被环境淘汰就只有不断提升自己，**从来都是我们去适应环境，而不是环境来适应我们！**

近期我们搜集了 N 套阿里、腾讯、美团、网易等公司 19 年的面试题，把技术点梳理成一份大而全的“Android高级工程师”面试题库（实际上比预期多花了不少精力），包含标准答案解析，由于篇幅有限，这里以图片的形式给大家展示一部分。

![](https://upload-images.jianshu.io/upload_images/20273755-44cc6917185adc0e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

**这份资料尤其适合：**

1.近期想跳槽，要面试的Android程序员，查漏补缺，以便尽快弥补短板；

2.想了解“一线互联网公司”最新技术要求，对比找出自身的长处和弱点所在，评估自己在现有市场上的竞争力如何；

3.做了几年Android开发，但还没形成系统的Android知识体系，缺乏清晰的提升方向和学习路径的程序员。

相信它会给大家带来很多收获。**++++维信，（壹叁贰零叁壹陆叁陆零玖）就可以免费领取了**

除面试资料外，这里还整理了一份最近刚录制的视频——BAT大牛解密Android面试，对于面试，是个不错的补充。

![](https://upload-images.jianshu.io/upload_images/20273755-f0eaab7f9183509b.png?imageMogr2/auto-orient/strip|imageView2/2/w/902/format/webp)

视频围绕“BAT大牛解密Android面试？”的主题，内容由浅入深，同时，对于开源框架相关面试问题也作出重点解读。

**视频具体内容如下：**

*   第1章 课程介绍
*   第2章 一线互联网公司初中高Android开发工程师的技能要求
*   第3章 Android基础相关面试题
*   第4章 异步消息处理机制相关面试问题
*   第5章 View相关面试问题
*   第6章 Android项目构建相关面试问题
*   第7章 开源框架相关面试问题
*   第8章 Android异常与性能优化相关面试问题
*   第9章 热门前沿知识相关面试问题

**需要获取更全面的面试资料，或专题视频，++++维信：（壹叁贰零叁壹陆叁陆零玖）。前往免费领取！**


