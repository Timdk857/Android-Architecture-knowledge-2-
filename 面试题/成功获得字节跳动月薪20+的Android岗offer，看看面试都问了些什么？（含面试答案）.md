本人年前已经获得offer，已经时间过去蛮久，尽可能多回忆些面试遇到的问题。
这次肺炎疫情的影响，不少小伙伴年初跳槽的计划被搁置。虽然计划被打乱，但是这也给我们留出时间更好的准备面试。随着企业复工时间的延长，不少公司裁员、降薪甚至倒闭，之后只会更难。

**话不多说直接分享面试题解析**
# 一、JAVA&Android

## **1.JVM和DVM有什么区别，以及ART垃圾回收机制？**

1.基于不同位置

Dalvik：基于寄存器，编译和运行都会快一些 

JVM: 基于栈, 编译和运行都会慢些

2.字节码的区别

Dalvik: 执行.dex格式的字节码，是对.class文件进行压缩后产生的,文件变小

JVM: 执行.class格式的字节码

3.运行环境的区别 　 　

Dalvik : 一个应用启动都运行一个单独的虚拟机运行在一个单独的进程中

JVM: 只能运行一个实例, 也就是所有应用都运行在同一个JVM中

但是在Android4.4引入了ART，也是 Android 5.0 及更高版本的默认 Android 运行时。目前google已经不再维护和发布dalvik（DVM）。

1.ART GC 与 Dalvik 的主要区别在于 ART GC 引入了移动垃圾回收器。使用移动 GC 的目的在于通过堆压缩来减少后台应用使用的内存。目前，触发堆压缩的事件是 ActivityManager 进程状态的改变。当应用转到后台运行时，它会通知 ART 已进入不再“感知”卡顿的进程状态。此时 ART 会进行一些操作（例如，压缩和监视器压缩），从而导致应用线程长时间暂停。目前正在使用的两个移动 GC 是同构空间压缩和半空间压缩。

2.与 Dalvik 相比，暂停次数从 2 次减少到 1 次。Dalvik 的第一次暂停主要是为了进行根标记，而这个动作在 ART中已经是让线程自己去标记，然后马上恢复运行，这样就减少了一次暂停。

3.与 Dalvik 类似，ART GC 在清除过程开始之前也会暂停 1 次。两者在这方面的主要差异在于：在此暂停期间，某些 Dalvik 环节在 ART 中并发进行。这些环节包括 java.lang.ref.Reference 处理、系统弱清除（例如，jni 弱全局等）、重新标记非线程根和卡片预清理。在 ART 暂停期间仍进行的阶段包括扫描脏卡片以及重新标记线程根，这些操作有助于缩短暂停时间。

4.相对于 Dalvik，ART GC 改进的最后一个方面是粘性 CMS 回收器增加了 GC 吞吐量。不同于普通的分代 GC，粘性 CMS 不移动。系统会将年轻对象保存在一个分配堆栈（基本上是 java.lang.Object 数组）中，而非为其设置一个专属区域。这样可以避免移动所需的对象以维持低暂停次数，但缺点是容易在堆栈中加入大量复杂对象图像而使堆栈变长

最后说一下回收机制：

1. 先回收与其他Activity 或Service/Intent Receiver 无关的进程(即优先回收独立的Activity)因此建议,我们的一些(耗时)后台操作，最好是作成Service的形式

2.不可见(处于Stopped状态的)Activity

3.Service进程(除非真的没有内存可用时会被销毁)

4.非活动的可见的(Paused状态的)Activity

5.当前正在运行（Active/Running状态的）Activity 

## **2.自定义Handler时如何避免内存泄漏**

一般非静态内部类持有外部类的引用的情况下，造成外部类在使用完成后不能被系统回收内存，从而造成内存泄漏。为了避免这个问题，我们可以自定义的Handler声明为静态内部类形式，然后通过弱引用的方式，让Handler持有外部类的引用，从而可避免内存泄漏问题。

以下是代码实现

```
private WeakReference<MainActivity> activityWeakReference;
private MyHandler myHandler;

static class MyHandler extends Handler {
private MainActivity activity;

MyHandler(WeakReference<MainActivity> ref) {
this.activity = ref.get();
}

@Override
public void handleMessage(Message msg) {
super.handleMessage(msg);
switch (msg.what) {
case 1:
//需要做判空操作
if (activity != null) {
activity.mTextView.setText("new Value");
}
break;
default:
Log.i(TAG, "handleMessage: default ");
break;
}

@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_main);
//在onCreate中初始化
activityWeakReference = new WeakReference<MainActivity>(this);
myHandler = new MyHandler(activityWeakReference);

myHandler.sendEmptyMessage(1);
mTextView = (TextView) findViewById(R.id.tv_test);
}
```

参考博文：[http://blog.csdn.net/ucxiii/article/details/50972747](https://link.zhihu.com/?target=https%3A//links.jianshu.com/go%3Fto%3Dhttp%253A%252F%252Fblog.csdn.net%252Fucxiii%252Farticle%252Fdetails%252F50972747)

## 3.Android IPC:Binder原理
1.在Activity和Service进行通讯的时候，用到了Binder。 
o1.当属于同个进程我们可以继承Binder然后在Activity中对Service进行操作
o2.当不属于同个进程，那么要用到AIDL让系统给我们创建一个Binder，然后在Activity中对远端的Service进行操作。
2.系统给我们生成的Binder： 
o1.Stub类中有:接口方法的id，有该Binder的标识，有asInterface(IBinder)(让我们在Activity中获取实现了Binder的接口，接口的实现在Service里，同进程时候返回Stub否则返回Proxy)，有onTransact()这个方法是在不同进程的时候让Proxy在Activity进行远端调用实现Activity操作Service
o2.Proxy类是代理，在Activity端，其中有:IBinder mRemote(这就是远端的Binder)，两个接口的实现方法不过是代理最终还是要在远端的onTransact()中进行实际操作。
3.哪一端的Binder是副本，该端就可以被另一端进行操作，因为Binder本体在定义的时候可以操作本端的东西。所以可以在Activity端传入本端的Binder，让Service端对其进行操作称为Listener，可以用RemoteCallbackList这个容器来装Listener，防止Listener因为经历过序列化而产生的问题。
4.当Activity端向远端进行调用的时候，当前线程会挂起，当方法处理完毕才会唤醒。
5.如果一个AIDL就用一个Service太奢侈，所以可以使用Binder池的方式，建立一个AIDL其中的方法是返回IBinder，然后根据方法中传入的参数返回具体的AIDL。
6.IPC的方式有：Bundle（在Intent启动的时候传入，不过是一次性的），文件共享(对于SharedPreference是特例，因为其在内存中会有缓存)，使用Messenger(其底层用的也是AIDL，同理要操作哪端，就在哪端定义Messenger)，AIDL，ContentProvider(在本进程中继承实现一个ContentProvider，在增删改查方法中调用本进程的SQLite，在其他进程中查询)，Socket
## 4.描述一次跨进程通讯
1.client、proxy、serviceManager、BinderDriver、impl、service
2.client发起一个请求service信息的Binder请求到BinderDriver中，serviceManager发现BinderDiriver中有自己的请求 然后将clinet请求的service的数据返回给client这样完成了一次Binder通讯
3.clinet获取的service信息就是该service的proxy，此时调用proxy的方法，proxy将请求发送到BinderDriver中，此时service的 Binder线程池循环发现有自己的请求，然后用impl就处理这个请求最后返回，这样完成了第二次Binder通讯
4.中间client可挂起，也可以不挂起，有一个关键字oneway可以解决这个

# 二、算法题
## 1.lc里最长上升子序列的变形题。
## 2.实现输入英文单词联想的功能
## 3.矩阵旋转，要求空间复杂度O(1)
## 4.无序的数组的中位数。要求时间复杂度尽可能的小

#三、计算机网络

## 1.**tcp 怎么保证数据包有序**

 1. 主机每次发送数据时，TCP就给每个数据包分配一个序列号并且在一个特定的时间内等待接收主机对分配的这个序列号进行确认，
 2. 如果发送主机在一个特定时间内没有收到接收主机的确认，则发送主机会重传此数据包。
 3. 接收主机利用序列号对接收的数据进行确认，以便检测对方发送的数据是否有丢失或者乱序等，
4. 接收主机一旦收到已经顺序化的数据，它就将这些数据按正确的顺序重组成数据流并传递到高层进行处理。

## 2.**tcp 和 udp 的异同**

TCP是面向流的可靠数据传输连接
UDP是面向数据包的不可靠无连接

## 3.**tcp 怎么保证可靠性**

差错检验机制，反馈机制，重传机制，引入序号，滑动窗口协议，选择重传

## 4.**tcp 中 拥塞避免 和 流量控制 机制**

拥塞避免和流量控制这两种机制很像，但是流量控制是由接收方的接受能力也就是接收窗口所决定的，如果接收窗口够大，以动态调整发送窗口的大小调整发送速度
拥塞避免主要由网络情况所限制，网络情况良好，则加大发送速率，网络状态差（冗余ACK和丢包）则降低发送速率（慢启动，拥塞控制，快恢复，快重传）RENO，BBR

## 5.**tcp 四次挥手的详细解释**

 tcp四次挥手其实可以分为两个阶段
第一：
客户端至服务器的半双工连接关闭
客户端向服务器发送FIN信号，进入FIN_WAIT1的状态，等待服务器的ACK信号 
收到服务器的ACK后，进入FIN_WAIT2
第二：
服务器至客户端的半双工连接关闭
客户端收到服务器发来的FIN后，发送ACK，并进入TIME_WAIT，等待2msl，若无异常，则客户端认为连接成功关闭
服务器收到客户端发来的ACK后，关闭连接

**四次挥手之后为什么还要等待2msl**

MSL是报文最大生存时间
1是因为有可能客户端发往服务器的ACK丢失，服务器并不知道客户端已经确认关闭，这时候客户端的关闭会导致服务器端无法正常关闭
2是为了保证连接中的报文都已经传递。假如短时间关闭又重新实现一个TCP还连到了同个端口上，旧连接中尚未消失的数据就会被认为是新连接的数据。

**浏览器从输入网址到显示出网页的全过程**

1.输入网址或者ip。
2.如果输入的是网址，首先要查找域名的ip地址
第一步会在浏览器缓存中查找，如果没有，转至查询系统缓存，如果还是没有，发送请求给路由器，路由器首先会在自身的缓存中查找，如果还是没有，向ips发出请求，查询ips中的dns缓存，如果还是没有递归向上查询直至根服务器。
.浏览器与ip机器之间建立TCP连接（三次握手）（HTTP）或者在TCP上进一步建立SSL/TLS连接（HTTPS）
接下来就是发送HTTP报文啥的了
GET，POST，DELETE，PUT。

**滑动窗口机制的原理和理解**

GBN协议，回退N步协议，这是对停等协议的改进，因为停等协议的传输效率非常低下。每次可发送的数据为N，基数为base，小于base的数据已经发送并且确认，base是最小的已发送未确认的报文序号。在接收端同样也有一个接收窗口，（解释）GBN采用的是累计确认方式，这时候说一下选择重传机制。再说一下TCP中既不是GBN也不是SR，而是GBN和SR的综合体。
N的大小必须报文序列编号的一半，否则接收端对报文的确认可能发生混淆

**Https 原理和实现**

**cookie和session的区别是什么**

由于HTTP协议是无状态的协议，所以服务端需要记录用户的状态时，就需要用某种机制来识具体的用户
cookie存在本地的上的
session是存在服务器上的
通俗讲，Cookie是访问某些网站以后在本地存储的一些网站相关的信息，下次再访问的时候减少一些步骤。另外一个更准确的<u style="text-decoration: none; border-bottom: 1px dashed grey;">[说法](https://link.zhihu.com/?target=http%3A//www.lai18.com/content/407204.html)</u>是：Cookies是服务器在本地机器上存储的小段文本并随每一个请求发送至同一个服务器，是一种在客户端保持状态的方案。
Session是存在服务器的一种用来存放用户数据的类HashTable结构。
二者都用来保持用户的状态，cookie可更改，对服务器来说并不安全，服务器常见做法有这两种
1.把session加密后放入浏览器的cookie中，浏览器重连后将加密的session发给服务器
2.cookie中存储着session的id，浏览器重连时只需要发送session_id'即可

## 四、操作系统

**进程和线程的区别**

**进程就是包换上下文切换的程序执行时间总和** = **CPU加载上下文+CPU执行+CPU保存上下文**

**线程是什么呢？**

进程的颗粒度太大，每次都要有上下的调入，保存，调出。如果我们把进程比喻为一个运行在电脑上的软件，那么一个软件的执行不可能是一条逻辑执行的，必定有多个分支和多个程序段，就好比要实现程序A，实际分成 a，b，c等多个块组合而成。那么这里具体的执行就可能变成：
程序A得到CPU =》CPU加载上下文，开始执行程序A的a小段，然后执行A的b小段，然后再执行A的c小段，最后CPU保存A的上下文。
这里a，b，c的执行是共享了A的上下文，CPU在执行的时候没有进行上下文切换的。这**里的a，b，c就是线程，也就是说线程是共享了进程的上下文环境，的更为细小的CPU时间段。
# 五、设计模式
简述一个设计模式的概念，并简要谈谈framework层哪些地方用到了什么设计模式
单例模式：单例模式是一种对象创建模式，它用于产生一个对象的具体实例，它可以确保系统中一个类只产生一个实例。

适配器模式：将一个接口转换成客户希望的另一个接口，适配器模式使接口不兼容的那些类可以一起工作，其别名为包装器(Wrapper)

装饰模式：动态地给一个对象增加一些额外的职责，就增加对象功能来说，装饰模式比生成子类实现更为灵活。装饰模式是一种对象结构型模式。

使用场景：

1.在不影响其他对象的情况下，以动态、透明的方式给单个对象添加职责。
2.当不能采用继承的方式对系统进行扩展或者采用继承不利于系统扩展和维护时可以使用装饰模式。

优点：

1.对于扩展一个对象的功能，装饰模式比继承更加灵活，不会导致类的个数急剧增加。
2.可以通过一种动态地方式来扩展一个对象的功能。
3.可以对一个对象进行多次装饰，通过使用不同的具体装饰类以及这些装饰类的排列组合。

实际运用：

Android中Context类的实现

外观模式： 主要目的在于让外部减少与子系统内部多个模块的交互，从而让外部能够更简单得使用子系统。它负责把客户端的请求转发给子系统内部的各个模块进行处理。

使用场景：

1.当你要为一个复杂的子系统提供一个简单的接口时
2.客户程序与抽象类的实现部分之前存在着很大的依赖性
3.当你需要构建一个层次结构的子系统时

组合模式：将对象以树形结构组织起来，以达成”部分--整体”的层次结构，使得客户端对单个对象和组合对象的使用具有一致性。

使用场景：

1.需要表示一个对象整体或部分 层次
2.让客户端能够忽略不同对象层次的变化

优点：

1.高层模块调用简单
2.节点自由增加

模板模式：是通过一个算法骨架，而将算法中的步骤延迟到子类，这样子类就可以复写这些步骤的实现来实现特定的算法。它的使用场景：

1.多个子类有公有的方法，并且逻辑基本相同
2.重要、复杂的算法，可以把核心算法设计为模板方法
3.重构时，模板方法模式是一个经常使用的模式

观察者模式：定义对象之间一种一对多依赖关系，使得每当一个对象状态发生改变时，其相关依赖对象皆得到通知并被自动更新。其使用场景：

1.一个抽象模型有两个方面，其中一个方面依赖于另一个方面
2.一个对象的改变将导致一个或多个其他对象也 发生改变
3.需要在 系统中创建一个 触发链

具体应用：

比如回调模式中，实现了抽象类/接口的实例实现了父类提供的抽象方法后，将该方法交还给父类来处理

Listview中的notifyDataSetChanged
RxJava中的观察者模式
**责任链模式: **是一个请求有多个对象来处理，这些对象是一条链，但具体由哪个对象来处理，根据条件判断来确定，如果不能处理会传递给该链条中的下一个对象，直到有对象处理它为止。

使用场景：

1.有多个对象可以处理同一个请求，具体哪个对象处理该请求待运行时再确定
2.在不明确指定接收者的情况下，向多个对象中的一个提交一个请求。

实际运用：

Try...catch语句
OrderedBroadcast
MotionEvent:actionDwon actionMove actionUp
事件分发机制三个重要方法:dispatchTouchEvent. onInterceptTouchEvent. onTouchEvent
策略模式： 定义一系列的算法，把它们一个个封装起来，并且使他们可互相替换。本模式使得算法可独立于使用它的客户而变化。策略模式的使用场景:一个类定义了多种行为，并且这些行为在这个类的方法中以多个条件语句的形式出现，那么可以使用策略模式避免在类中使用大量的条件语句。

使用场景：

一个类定义了多种行为，并且这些行为在这个类的方法中以多个条件语句的形式出现，那么可以使用策略模式避免在类中使用大量的条件语句。

优点：

1.上下文Context和具体策略ConcreateStrategy是松耦合关系
2.策略模式满足 开-闭原则

具体应用：

HttpStack
Volley框架

# 最后
最后我想说：有些东西你不仅要懂，而且要能够很好地表达出来，能够让面试官认可你的理解，例如Handler机制，这个是面试必问之题。有些晦涩的点，或许它只活在面试当中，实际工作当中你压根不会用到它，但是你要知道它是什么东西。

面试：如果不准备充分的面试，完全是浪费时间，更是对自己的不负责！
最后在这里小编分享一份自己收录整理上述技术体系图相关的几十套腾讯、头条、阿里、美团等公司19年的面试题，把技术点整理成了视频和PDF（实际上比预期多花了不少精力），包含知识脉络 + 诸多细节，由于篇幅有限，这里以图片的形式给大家展示一部分。
**需要的小伙伴可以直接++++维信（壹叁贰零叁壹陆叁陆零玖 就可以获取了）**
![](https://upload-images.jianshu.io/upload_images/19956127-f2eb811be82e8d41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/19956127-06efb67f8fbb6a27.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
需要的小伙伴可以直接++++维信（壹叁贰零叁壹陆叁陆零玖 就可以获取了）
