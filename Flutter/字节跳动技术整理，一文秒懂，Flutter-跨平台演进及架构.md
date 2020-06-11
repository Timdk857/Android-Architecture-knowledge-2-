## 一、移动跨平台技术演进[](http://gityuan.com/flutter/#一移动跨平台技术演进)

#### 1\. 引言[](http://gityuan.com/flutter/#1-引言)

移动互联网发展十余年，伴随着 Android、iOS 等智能手机的不断普及，移动端已逐步取代 PC 端，成为兵家必争之地。正所谓“得移动端者得天下”，移动端已成为互联网领域最大的流量分发入口，一大批互联网公司正是在这大趋势下崛起。

#### 2\. 为什么需要跨平台技术[](http://gityuan.com/flutter/#2-为什么需要跨平台技术)

伴随着移动互联网的高速发展，公司间竞争越来越激烈，如何将好想法快速落地、快速试错，成为备受关注的问题。提升研发效率、缩短研发周期，保障产品快速试错并能快速迭代新功能，让新产品新功能以最快的速度同时抵达 Android、iOS 等多端用户。

众所周知，Android 应用采用 Java 或 Kotlin 编写，iOS 应用采用 Objective-C 或 Swift 编写，Web 端采用 HTML /CSS/JavaScript 编写。当需要开发支持多端的应用，每一端都需要独立研发、测试，一直到上线，以及后续的维护工作，工作量成倍增涨，势必延长研发周期。

为了解决多端独立开发的问题，跨平台技术便应运而生，各大互联网公司为此都投入大量人力，于是出现了各种跨平台技术框架，**面对移动领域的跨平台技术方案的层出不穷，又该如何做技术选型呢？**

#### 3\. 移动端技术选型[](http://gityuan.com/flutter/#3-移动端技术选型)

作为移动端的跨端技术方案，所关注无外乎以下这4个方面：研发效率、动态性、多端一致性、性能体验。

>![](https://upload-images.jianshu.io/upload_images/19956127-1f23d8f39c8dcb7c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.  研发效率：最大化代码复用，减少多端差异的适配工作量，降低开发成本，专注业务开发，实现“write once，run everywhere”的终极目标。效率提升是贯穿整个业务的生命周期线，即便业务上线后，可持续降低后续的维护成本，加快新feature的迭代速度，这是一个持续的效率收益。当然，这里不得不说，任何一门新技术在开发启动学习阶段会有一些成本，但上手后的收益是长期的。
2.  动态化：突破渠道的更新频率，可快速迭代新功能，这一点不仅是跨平台技术的诉求，也是Native技术必备的杀手锏，这也是评估跨端技术的一个重要考核点。
3.  多端一致性：好产品在多端UI设计上，往往是整体风格统一，所以业务方采用原生各自独立开发完成后，还需额外花不少时间来修改UI以保证多端一致性；可见，各端独立实现开发方式，带来的效率滞后，不仅仅是Android和iOS各开发一份代码的工作量，还有双端UI的一致性对齐的工作。
4.  性能体验：一般地，跨端技术方案拥有以上多重优势，但在性能方面比原生流畅更差些。牺牲部分体验换来效率提升，这一点也是情理之中，试想一下，跨平台技术方案同时兼得这4点，那么原生技术恐怕已退出历史舞台，早已是跨平台技术的天下，所以往往跨平台技术的性能优劣便成为核心指标。

#### 4\. 跨平台技术划分[](http://gityuan.com/flutter/#4-跨平台技术划分)

对研发效率和体验的不断追逐，移动端的跨平台技术方框架层出不穷，然则天下武功众多，万变不离其宗，从其核心本质来划分，可大致分为以下三大类：

>![](https://upload-images.jianshu.io/upload_images/19956127-0f49f91dfe26fc66.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.  Web技术：主要依赖于WebView的技术，功能支持受限，性能体验很差，比如PhoneGap、Cordova、小程序。
2.  原生渲染：使用JavaScript作为编程语言，通过中间层转化为原生控件来渲染UI界面，比如React Native、Weex。
3.  自渲染技术：自行实现一套渲染框架，可通过调用skia等方式完成自渲染，而不依赖于原生控件，比如Flutter、Unity。

#### 5\. 跨平台技术演进[](http://gityuan.com/flutter/#5-跨平台技术演进)

跨平台技术，一直以来是每一个有追求的开发者所追逐的梦想，同时也是守旧者的噩梦，跨平台的多端一体化方案势必颠覆现有的原生各端独立开发模式，接下来列举众多的跨平台技术中最为关键的几个技术方案的演进阶段。

>![](https://upload-images.jianshu.io/upload_images/19956127-bc7e97d4ca979e15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从上图可以看出，技术演进过程大致分以下三个阶段： 第一阶段，采用WebView技术绘制界面的Hybrid混合开发技术，通过JS Bridge 将系统部分能力暴露给 JS 调用，其缺点是性能较差，功能受限，扩展性差，不适合交互复杂的场景，比如Cordova。 第二阶段，针对WebView界面性能等问题，于是绘制交还原生渲染，仅仅通过JS调用原生控件，相比WebView技术性能体验更好，这是目前绝大部分跨平台框架的设计思路，比如React Native、Weex。另外，最近小程序也比较火，第一和第二阶段的融合，依然采用WebView作为渲染容器，通过限制Web技术栈的子集，规范化组件使用，并逐步引入原生控件代表WebView渲染，以提升性能。 第三阶段，虽然通过桥接技术使用原生控件解决了功能受限问题，提升性能体验，但相比原生体验差距还是比较大，以及处理平台差异性非常耗费人力。于是Flutter提出自带渲染引擎的解决方案，尽可能减少不同平台间的差异性, 同时媲美原生的高性能体验，因此业界对 Flutter有着极高的关注度。

**面对现有的如此多跨平台方案，为何当下最火的跨平台技术是Flutter，有哪些优势呢？**

RN、Weex均使用JavaScript作为编程语言，JavaScript作为前端开发语言，在跨平台开发中可谓大放异彩，利用web技术不仅能开发出网站，也可以开发手机端web应用和移动端应用程序，似有一统三界(Android、iOS、Web)的趋势，这就是大家常说的“大前端”时代。这些技术方案流畅度不太好，平台一致性较差，至今还没能大面积取代原生开发。

Flutter是以Dart语言编写，开发体验更接近客户端，从大家使用反馈来看也是如此，Flutter开发环境这一套的流程对于前端开发来说并不太友好。Flutter的定位同样是多端一体化，但是以客户端为首，先磨平Android和iOS双端开发体验，再逐步向Web端渗透，从Flutter规划的Roadmap也能看出，Flutter for web目前仍处于预览版，Flutter客户端方向都已经如火如荼上线了不少应用。

在此之前，大家常说“大前端”，对于Flutter技术，在笔者看来称之为“大移动端”更贴切，Flutter的UI框架优先支持客户端(Android/iOS)应用的同时，然后再适配Web端。移动互联网时代，不少公司都制定“移动优先”的战略，甚至只开发移动端，没有Web端。移动互联网的时代造就“大移动端”，Flutter作为一款能做到媲美原生的高性能跨平台技术方案，或许一统天下。

在跨平台技术领域，只要挑战在，技术就不会停滞，伴随着技术不断演进与革新，终将走向美好。

#### 6\. Flutter技术优势[](http://gityuan.com/flutter/#6-flutter技术优势)

Flutter是彻底的跨平台方案，既没有采用webView，也没有采用JS桥接原生控件，而是自行实现一套UI框架，在引擎底层通过Skia渲染到屏幕。对于UI之外所需要使用的移动设备自身提供的服务，比如相机、定位、屏幕触摸等，则采用Platform Channels跟原生系统通信的方式来实现。

对于Flutter优势，回到前面讲到移动端技术选型的4要素，研发效率、动态性、多端一致性、性能体验，分别对应下面这一组词语。

>![](https://upload-images.jianshu.io/upload_images/19956127-e77f80d10b3b9c98.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.  高效率：采用dart语言编写代码，虽然刚开始上手需要点时间，但熟练后效率比较高。一套代码适用多个平台(Android、iOS、Web)，以及高效的Hot Reload能快速辅助调试；
2.  动态化：2017年3月苹果下发警告邮件，禁止JSPatch等 iOS App热更新方案，从此iOS动态化成为一个不宜公开讨论的话题。同样地，Flutter引擎在某一个官方版本对动态化做过一些尝试，但后续基于风险考虑移除，当然并没有阻碍大家对技术的探索，这里不方便展开讨论；
3.  高一致性：实现UI像素级的控制，Flutter渲染引擎依靠跨平台Skia图形库来实现，仅依赖系统图形绘制相关的接口，比如未来Android会支持vulkan，iOS会支持metal，这些都是通过skia封装调用。可最大程度上保证不同平台的体验一致性，见下图所示。

>![](https://upload-images.jianshu.io/upload_images/19956127-2346f27bdad55dd0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.  高性能：渲染性能优于现有的各种跨平台框架，可媲美原生性能的跨平台技术方案，Dart代码执行效率比JS高，通过AOT编译成平台原生代码，渲染采用自渲染skia方案，既不需要JS Bridge桥接，也不需要Art虚拟机参与。再从渲染原理来看看Flutter的高性能的底气在哪里。

>![](https://upload-images.jianshu.io/upload_images/19956127-923e10294da59eef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

图解：

*   Android原生框架，通过调用Java Framework层，再调用到skia来渲染界面；
*   其他跨平台方案(如RN)，通过JSBridge中间层来将JS写的APP转换成相应的原生渲染逻辑，可见比Native代码增加了更多逻辑，性能逊色差于原生框架；
*   Flutter框架，APP通过调用Dart Framework层，再直接调用到skia来渲染界面，并没有经过原生Framework过程，可见其渲染性能并不会弱于Native技术，这是一个性能上限很高的跨平台技术。

当然，不得不说目前的Flutter确实不够尽善尽美，会存在一些不够尽善尽美之处，比如生态不够健全，包体积问题，但其该方案的上限比较高，想象空间比较大，相信更多开发者参与进来，经过更多打磨，未来会做得更好。

#### 7\. 业界发展近况[](http://gityuan.com/flutter/#7-业界发展近况)

2017年5月Google I/O大会正式对外公布Flutter，到2018年12月发布Flutter1.0，引发全球大量的开发者和企业开始研究Flutter。StackOverflow 2019年的全球开发者文件调查中，Flutter被评选为最受开发者欢迎的框架之一，超过了TensorFlow和Node.js。

>![](https://upload-images.jianshu.io/upload_images/19956127-a06149b3030b6854.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

到目前，全球越来越多的公司已经在大家耳熟能详的知名APP中使用Flutter技术并落地，尤其国内知名互联网公司对Flutter投入度很大，社区也是非常活跃。

>![](https://upload-images.jianshu.io/upload_images/19956127-016816a9d0d1e12f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 8\. Flutter未来趋势[](http://gityuan.com/flutter/#8-flutter未来趋势)

目前Flutter主要在移动端Android/iOS双端跨端，Flutter 的愿景是成为一个多端运行的 UI 框架，能够支持不仅仅是移动端，还包括Web、桌面、甚至嵌入式设备。在2019 Google I/O 开发者大会上推出的使用 Flutter 开发 Web 应用的框架，同年9月发布Flutter 1.9，并将Flutter web合入Flutter主仓库。

>![](https://upload-images.jianshu.io/upload_images/19956127-4a04fb11c708ebc8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从架构图看，Flutter采用同一个Dart Framework层来统一Flutter C++引擎和Web引擎，最终可以运行在Android，iOS，Browser上，从Flutter引擎代码不难看出Flutter也是支持Fuchsia操作系统。

Fuchsia是Google内部正在开发的一款新的操作系统，采用Flutter作为系统默认的UI框架，也就是说Flutter天然支持Fuchsia，这无疑让Flutter在众多的跨平台方案更有优势。

从Fuchsia技术架构来看，内核层zircon的基础LK是专为嵌入式应用中小型系统设计的内核，代码简洁，适合嵌入式设备和高性能设备，比如IOT、移动可穿戴设备等，目前这些领域还没有标准化级别的垄断者。以及在框架层中有着语音交互、云端以及智能化等模块，由此笔者揣测未来Fuchsia率先应用在音控等智能嵌入式设备。

>![](https://upload-images.jianshu.io/upload_images/19956127-5dfc8146538bfd70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

目前大家普遍比较看好的未来两个技术就是5G和IoT时代。对于5G的需求，很大程度上是因为移动互联网发展到“IoT时代”的阶段。这个发展阶段，全球上网设备的数量可能会达到500亿个。随着5G+IOT时代的到来，现在大家比较关注的Flutter包大小也同样不再是一个问题，或许Flutter技术的生命期比客户端更长，或许Fuchsia正在驰骋IOT疆场，你所掌握的Flutter技术栈可以无缝迁移，一次弯道超车的机会。

到此，介绍完跨平台技术演进以及Flutter的优势。看到这，相信你可能对Flutter技术有一定兴趣，为了能让大家快速了解Flutter内部原理而不枯燥，Gityuan通过一系列图来帮大家从整体架构来快速理解Flutter。

## 二、Flutter引擎架构[](http://gityuan.com/flutter/#二flutter引擎架构)

#### 1\. Flutter技术架构[](http://gityuan.com/flutter/#1-flutter技术架构)

先来看看Flutter整体的技术架构，分为四层，从上之下依次是Dart APP，Dart Framework， C++ Engine，Platform。

>![](https://upload-images.jianshu.io/upload_images/19956127-fdae9989a01cf32b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Flutter架构最核心的便是Framework（框架）和Engine（引擎）：

*   Flutter Framework层：用Dart编写，封装整个Flutter架构的核心功能，包括Widget、动画、绘制、手势等功能，有Material（Android风格UI）和Cupertino（iOS风格）的UI界面， 可构建Widget控件以及实现UI布局。
*   Flutter Engine层：用C++编写，用于高质量移动应用的轻量级运行时环境，实现了Flutter的核心库，包括Dart虚拟机、动画和图形、文字渲染、通信通道、事件通知、插件架构等。引擎渲染采用的是2D图形渲染库Skia，虚拟机采用的是面向对象语言Dart VM，并将它们托管到Flutter的嵌入层。shell实现了平台相关的代码，比如跟屏幕键盘IME和系统应用生命周期事件的交互。不同平台有不同的shell，比如Android和iOS的shell。

#### 2\. Flutter编译产物[](http://gityuan.com/flutter/#2-flutter编译产物)

看完Flutter内部架构，或许你好奇，Flutter不用Android/iOS的本地语言技术开发，Dart编写完的代码如何让不同系统可以识别，最终编译后得到的产物是什么呢？

>![](https://upload-images.jianshu.io/upload_images/19956127-79ad3332a3e509b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Flutter产物分为Dart业务代码和Engine代码各自生成的产物，图中的Dart Code包含开发者编写的业务代码，Engine Code是引擎代码，如果并没有定制化引擎，则无需重新编译引擎代码。

一份Dart代码，可编译生成双端产物，实现跨平台的能力。经过编译工具处理后可生成双端产物，图中便是release模式的编译产物，Android产物是由vm、isolate各自的指令段和数据段以及flutter.jar组成的app.apk，iOS产物是由App.framework和Flutter.framework组成的Runner.app。

这个过程涉及frontend_server、gen_snapshot、xcrun、ninja编译工具。frontend_server前端编译器会进行词法分析、语法分析以及相关全局转换等工作，将dart代码转换为AST(抽象语法树)，并生成app.dill格式的dart kernel。gen_snapshot经过CHA、内联等一系列执行流的优化，根据中间代码生成优化后的FlowGraph对象，再转换为具体相应系统架构（arm/arm64等）的二进制指令。

#### 3\. Flutter引擎启动[](http://gityuan.com/flutter/#3-flutter引擎启动)

既然了解了Flutter的编译产物，那你或许又好奇，Flutter这台引擎如何发动的，怎么跟Native衔接呢？

>![](https://upload-images.jianshu.io/upload_images/19956127-f8cd15af3a1ab04b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里以Android为例，熟悉Android的开发者，应该都了解APP启动过程，会执行Application和Activity的onCreate()方法，FlutterApplication和FlutterActivity的onCreate()方法正是连接Native和Flutter的枢纽。

*   FlutterApplication.java的onCreate过程主要完成初始化配置、加载引擎libflutter.so、注册JNI方法；
*   FlutterActivity.java的onCreate过程，通过FlutterJNI的AttachJNI()方法来初始化引擎Engine、Dart虚拟机、Isolate、taskRunner等对象。再经过层层处理最终调用main.dart中main()方法，执行runApp(Widget app)来处理整个Dart业务代码。

Flutter引擎启动中会创建有4个TaskRunner以及创建虚拟机，分别来看看它们的工作原理。

#### 4\. TaskRunner工作原理[](http://gityuan.com/flutter/#4-taskrunner工作原理)

Flutter引擎启动过程，会创建UI/GPU/IO这3个线程，会为这些线程依次创建MessageLoop对象，启动后处于epoll_wait等待状态。对于Flutter的消息机制跟Android原生的消息机制有很多相似之处，都有消息(或者任务)、消息队列(或任务队列)以及Looper；有一点不同的是Android有一个Handler类，用于发送消息以及执行回调方法，相对应Flutter中有着相近功能的便是TaskRunner。

>![](https://upload-images.jianshu.io/upload_images/19956127-adee87aea350a9a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上图是从源码中提炼而来的任务处理流程，比官方流程图更容易理解一些复杂流程的时序问题，后续会专门讲解个中原由。Flutter的任务队列处理机制跟Android的消息队列处理相通，只不过Flutter分为Task和MicroTask两种类型，引擎和Dart虚拟机的事件以及Future都属于Task，Dart层执行scheduleMicrotask()所产生的属于Microtask。

每次Flutter引擎在消费任务时调用FlushTasks()方法，遍历整个延迟任务队列delayed_tasks_，将已到期的任务加入task队列，然后开始处理任务。

*   Step 1: 检查task，当task队列不为空，先执行一个task；
*   Step 2: 检查microTask，当microTask不为空，则执行microTask；不断循环Step 2 直到microTask队列为空，再回到执行Step 1；

可简单理解为先处理完所有的Microtask，然后再处理Task。因为scheduleMicrotask()方法的调用自身就处于一个Task，执行完当前的task，也就意味着马上执行该Microtask。

了解了其工作机制，再来看看这4个Task Runner的具体工作内容。

*   Platform Task Runner：运行在Android或者iOS的主线程，尽管阻塞该线程并不会影响Flutter渲染管道，平台线程建议不要执行耗时操作；否则可能触发watchdog来结束该应用。比如Android、iOS都是使用平台线程来传递用户输入事件，一旦平台线程被阻塞则会引起手势事件丢失。
*   UI Task Runner: 运行在ui线程，比如1.ui，用于引擎执行root isolate中的所有Dart代码，执行渲染与处理Vsync信号，将widget转换生成Layer Tree。除了渲染之外，还有处理Native Plugins消息、Timers、Microtasks等工作；
*   GPU Task Runner：运行在gpu线程，比如1.gpu，用于将Layer Tree转换为具体GPU指令，执行设备GPU相关的skia调用，转换相应平台的绘制方式，比如OpenGL, vulkan, metal等。每一帧的绘制需要UI Runner和GPU Runner配合完成，任何一个环节延迟都可能导致掉帧；
*   IO Task Runner：运行在io线程，比如1.io，前3个Task Runner都不允许执行耗时操作，该Runner用于将图片从磁盘读取出来，解压转换为GPU可识别的格式后，再上传给GPU线程。为了能访问GPU，IO Runner跟GPU Runner的Context在同一个ShareGroup。比如ui.image通过异步调用让IO Runner来异步加载图片，该线程不能执行其他耗时操作，否则可能会影响图片加载的性能。

#### 5\. Dart虚拟机工作[](http://gityuan.com/flutter/#5-dart虚拟机工作)

Flutter引擎启动会创建Dart虚拟机以及Root Isolate。DartVM自身也拥有自己的Isolate，完全由虚拟机自己管理的，Flutter引擎也无法直接访问。Dart的UI相关操作，是由Root Isolate通过Dart的C++调用，或者是发送消息通知的方式，将UI渲染相关的任务提交到UIRunner执行，这样就可以跟Flutter引擎相关模块进行交互。

何为Isolate，从字面上理解是“隔离”，isolate之间是逻辑隔离的。Isolate中的代码也是按顺序执行，因为Dart没有共享内存的并发，没有竞争的可能性，故不需要加锁，也没有死锁风险。对于Dart程序的并发则需要依赖多个isolate来实现。

>![](https://upload-images.jianshu.io/upload_images/19956127-7c028467f3827d76.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

图解：

*   isolate堆是运该isolate中代码分配的所有对象的GC管理的内存存储；
*   vm isolate是一个伪isolate，里面包含不可变对象，比如null，true，false；
*   isolate堆能引用vm isolate堆中的对象，但vm isolate不能引用isolate堆；
*   isolate彼此之间不能相互引用；
*   每个isolate都有一个执行dart代码的Mutator thread，一个处理虚拟机内部任务(比如GC, JIT等)的helper thread； 可见，isolate是拥有内存堆和控制线程，虚拟机中可以有很多isolate，但彼此之间内存不共享，无法直接访问，只能通过dart特有的Port端口通信；isolate除了拥有一个mutator控制线程，还有一些其他辅助线程，比如后台JIT编译线程、GC清理/并发标记线程；

#### 6\. Widget架构概览[](http://gityuan.com/flutter/#6-widget架构概览)

Flutter引擎启动后执行Dart业务，是通过runApp(Widget app)方法，那Widget又是什么呢？

>![](https://upload-images.jianshu.io/upload_images/19956127-623481d275339c4b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Widget是所有Flutter应用程序的基石，Widget可以是一个按钮，一种字体或者颜色，一个布局属性等，在Flutter的UI世界可谓是“万物皆Widget”。常见的Widget子类为StatelessWidget(无状态)和StatefulWidget(有状态)；

*   StatelessWidget：内部没有保存状态，UI界面创建后不会发生改变；
*   StatefulWidget：内部有保存状态，当状态发生改变，调用setState()方法会触发StatefulWidget的UI发生更新，对于自定义继承自StatefulWidget的子类，必须要重写createState()方法。

**三棵树**

>![](https://upload-images.jianshu.io/upload_images/19956127-41bcb8fb44b88708.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

图解：

*   Widget是为Element描述需要的配置， 负责创建Element，决定Element是否需要更新。Flutter Framework通过差分算法比对Widget树前后的变化，决定Element的State是否改变。当重建Widget树后并未发生改变， 则Element不会触发重绘，则就是Widget树的重建并不一定会触发Element树的重建。
*   Element表示Widget配置树的特定位置的一个实例，同时持有Widget和RenderObject，负责管理Widget配置和RenderObject渲染。Element状态由Flutter Framework管理， 开发人员只需更改Widget即可。
*   RenderObject表示渲染树的一个对象，负责真正的渲染工作，比如测量大小、位置、绘制等都由RenderObject完成。

可见，开发者通过Widget配置，Framework通过比对Widget配置来更新Element，最后调度RenderObject Tree完成布局排列和绘制。

#### 7\. 渲染原理[](http://gityuan.com/flutter/#7-渲染原理)

Dart的UI采用Widget来实现，最终转换为RenderObject，那界面又是如何渲染的呢？

>![](https://upload-images.jianshu.io/upload_images/19956127-0fd36bf5c7d7a0e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

渲染过程，UI线程完成布局、绘制操作，生成Layer Tree；GPU线程执行合成并光栅化后交给GPU来处理，其中几个关键步骤：

*   Animate: 遍历_transientCallbacks，执行动画回调方法；
*   Build: 对于dirty的元素会执行build构造，没有dirty元素则不会执行，对应于buildScope()
*   Layout: 计算渲染对象的大小和位置，对应于flushLayout()，这个过程可能会嵌套再调用build操作；
*   Compositing bits: 更新具有脏合成位的任何渲染对象， 对应于flushCompositingBits()；
*   Paint: 将绘制命令记录到Layer， 对应于flushPaint()；
*   Compositing: 将Compositing bits发送给GPU， 对应于compositeFrame()；

GPU线程通过skia向GPU硬件绘制一帧的数据，GPU将帧信息保存到FrameBuffer里面，然后视频控制器会根据VSync信号从FrameBuffer取帧数据传递给显示器，从而显示出最终的画面。

#### 8\. Platform Channels[](http://gityuan.com/flutter/#8-platform-channels)

Flutter框架提供了UI的控件支持，对于APP除了UI还有其他依赖于Native平台的支持，比如调用Camera的功能，该怎么办呢？为此，Flutter通过提供Platform Channel的功能，使得Dart代码具备与Native交互的能力。

>![](https://upload-images.jianshu.io/upload_images/19956127-b01f1506c7585eec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Platform Channel用于Flutter与Native之间的消息传递，整个过程的消息与响应是异步执行，不会阻塞用户界面。Flutter引擎框架已完成桥接的通道，这样开发者只需在Native层编写定制的Android/iOS代码，即可在Dart代码中直接调用，这也就是Flutter Plugin插件的一种形式。

## 三、Flutter源码解读[](http://gityuan.com/flutter/#三flutter源码解读)

笔者(Gityuan)之前一直从事于Android操作系统底层研发工作，今年刚接触Flutter，Flutter作为一门全新的跨平台技术框架，不断深究会发现这是一个小型系统，涉及到的技术很广：

*   编译技术如何将dart代码转换为AST(抽象语法树)，如何汇编转换为机器码，打包产物是什么？
*   Flutter这台引擎如何发动的，怎么跟Native原生系统衔接运行，如何识别产物并加载到内存？
*   引擎启动后，TaskRunner如何分发任务，跟原生系统消息机制有什么关系？
*   Dart虚拟机如何管理内存，跟isolate又有什么关系？
*   开发者编写的Widget控件如何渲染到屏幕上？
*   Flutter如何通过plugin支持移动设备提供的服务？

这些疑问在前面都逐一简要解答，如果仅仅是用Flutter做业务开发，并不需要掌握这么深度技术，不过，知其然知其所以然，能让你游刃有余。

#### 启动篇[](http://gityuan.com/flutter/#启动篇)

*   [深入理解Flutter引擎启动](http://gityuan.com/2019/06/22/flutter_booting/)
*   [深入理解Dart虚拟机启动](http://gityuan.com/2019/06/23/dart-vm/)
*   [深入理解Flutter应用启动](http://gityuan.com/2019/06/29/flutter_run_app/)

#### 通信篇[](http://gityuan.com/flutter/#通信篇)

*   [深入理解Flutter消息机制](http://gityuan.com/2019/07/20/flutter_message_loop/)
*   [深入理解Flutter的Platform Channel机制](http://gityuan.com/2019/08/10/flutter_channel/)
*   [深入理解Flutter异步Future机制](http://gityuan.com/2019/07/21/flutter_future/)
*   [深入理解Flutter的Isolate创建过程](http://gityuan.com/2019/07/27/flutter-isolate/)

#### 渲染篇[](http://gityuan.com/flutter/#渲染篇)

*   [Flutter渲染机制—UI线程](http://gityuan.com/2019/06/15/flutter_ui_draw/)
*   [Flutter渲染机制—GPU线程](http://gityuan.com/2019/06/16/flutter_gpu_draw/)
*   [深入理解setState更新机制](http://gityuan.com/2019/07/06/flutter_set_state/)
*   [深入理解Flutter动画原理](http://gityuan.com/2019/07/13/flutter_animator/)

后续笔者将持续研究与梳理Flutter内部机制的文章。

## 四、结束语[](http://gityuan.com/flutter/#四结束语)

本文分为三部分全方位逐级深入的分析Flutter技术，先讲述跨平台技术的过去与未来，再解读Flutter架构设计原理，最后剖析Flutter内部源码，后续将持续更新更多剖析技术深度细节以及实战经验，以彻底揭秘更多Flutter技术。

科技不断在进步，技术不断发展，移动跨平台技术几乎从Android、iOS诞生不久便出现，已发展快10年。时至今日，兼具跨端高效率与高性能体验的Flutter力压群雄，崭露头角，已然成为当下最热门的移动端新技术，全球越来越多的公司在Flutter技术布局并落地产品应用，社区也非常活跃。

随着5G+IOT时代的到来，Fuchsia系统或许发力IOT新战场，你所掌握的Flutter技术栈可以无缝迁移，这是一次弯道超车的机会。即便Fuchsia落败，相信只要深扎Flutter系统技术的精髓，其他任何的移动端新技术都可以轻松快速地掌握。

最后，用一句话来结束本次分享，“有时候，你选择一个方向，不是因为它一定会成为未来，而是它有可能成为不一样的未来。”

题外话，我在一线互联网企业工作十余年里，指导过不少同行后辈。帮助很多人得到了学习和成长。

我意识到有很多经验和知识值得分享给大家，也可以通过我们的能力和经验解答大家在IT学习中的很多困惑，所以在工作繁忙的情况下还是坚持各种整理和分享。但苦于知识传播途径有限，很多程序员朋友无法获得正确的资料得到学习提升，故此将并将重要的Android进阶资料包括自定义view、性能优化、MVC与MVP与MVVM三大框架的区别、NDK技术、阿里面试题精编汇总、常见源码分析等学习资料免费分享出来。
【[Android学习PDF+学习视频+面试文档+知识点笔记](https://links.jianshu.com/go?to=https%3A%2F%2Fshimo.im%2Fdocs%2FQ6V8xPVxHpkrtRtD)】

**【Android思维脑图（技能树）】**

知识不体系？这里还有整理出来的Android进阶学习的思维脑图，给大家参考一个方向。

> ![](https://upload-images.jianshu.io/upload_images/1095900-4bc8337d3615bf27.png?imageMogr2/auto-orient/strip|imageView2/2/w/530/format/webp)

**需要的朋友，可以点击关注+转发+简信“学习”前往免费领取！**

希望我能够用我的力量帮助更多迷茫、困惑的朋友们，帮助大家在IT道路上学习和发展~

原文作者：Gityuan
原文链接：http://gityuan.com/flutter/
