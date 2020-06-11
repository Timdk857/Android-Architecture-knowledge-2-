作者：melonstreet，
链接：www.cnblogs.com/QG-whz/p/9636366.html
**下面总结了 JVM 的 4 个问题，看你能顶住么？**

[1、JVM的内存区域是怎么划分的？](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247489332&idx=2&sn=65de5886e13b98116c8432d7d10ae4bc&chksm=eb539202dc241b14010f70edf89dc37c7629b5e2b7add50fd3f58070ecf2c14260196bf147d8&scene=21#wechat_redirect)
[2、OOM可能发生在哪些区域上？](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247489332&idx=2&sn=65de5886e13b98116c8432d7d10ae4bc&chksm=eb539202dc241b14010f70edf89dc37c7629b5e2b7add50fd3f58070ecf2c14260196bf147d8&scene=21#wechat_redirect)
[3、堆内存结构是怎么样的？](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247489332&idx=2&sn=65de5886e13b98116c8432d7d10ae4bc&chksm=eb539202dc241b14010f70edf89dc37c7629b5e2b7add50fd3f58070ecf2c14260196bf147d8&scene=21#wechat_redirect)
[4、常用的性能监控与问题定位工具有哪些？](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247489332&idx=2&sn=65de5886e13b98116c8432d7d10ae4bc&chksm=eb539202dc241b14010f70edf89dc37c7629b5e2b7add50fd3f58070ecf2c14260196bf147d8&scene=21#wechat_redirect)
**1、JVM的内存区域是怎么划分的？**
JVM的内存划分中，有部分区域是线程私有的，有部分是属于整个JVM进程；有些区域会抛出OOM异常，有些则不会，了解JVM的内存区域划分以及特征，是定位线上内存问题的基础。那么JVM内存区域是怎么划分的呢？
首先是程序计数器（Program Counter Register)，在JVM规范中，每个线程都有自己的程序计数器。这是一块比较小的内存空间，存储当前线程正在执行的Java方法的JVM指令地址，即字节码的行号。如果正在执行Native方法，则这个计数器为空。该内存区域是唯一一个在Java虚拟机规范中没有规定任何OOM情况的内存区域。
第二，Java虚拟机栈(Java Virtal Machine Stack)，同样也是属于线程私有区域，每个线程在创建的时候都会创建一个虚拟机栈，生命周期与线程一致，线程退出时，线程的虚拟机栈也回收。虚拟机栈内部保持一个个的栈帧，每次方法调用都会进行压栈，JVM对栈帧的操作只有出栈和压栈两种，方法调用结束时会进行出栈操作。该区域存储着局部变量表，编译时期可知的各种基本类型数据、对象引用、方法出口等信息。
第三，本地方法栈（Native Method Stack）与虚拟机栈类似，本地方法栈是在调用本地方法时使用的栈，每个线程都有一个本地方法栈。
第四，堆（Heap）,几乎所有创建的Java对象实例，都是被直接分配到堆上的。堆被所有的线程所共享，在堆上的区域，会被垃圾回收器做进一步划分，例如新生代、老年代的划分。Java虚拟机在启动的时候，可以使用“Xmx”之类的参数指定堆区域的大小。
第五，方法区（Method Area)。方法区与堆一样，也是所有的线程所共享，存储被虚拟机加载的元（Meta）数据，包括类信息、常量、静态变量、即时编译器编译后的代码等数据。这里需要注意的是运行时常量池也在方法区中。根据Java虚拟机规范的规定，当方法区无法满足内存分配需求时，将抛出OutOfMemoryError异常。由于早期HotSpot JVM的实现，将CG分代收集拓展到了方法区，因此很多人会将方法区称为永久代。Oracle [JDK8](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247483776&idx=1&sn=aa0203cfca67ff450ecaf1d9d147582d&chksm=eb5384b6dc240da057200d0f0edbf82ef41c03ea78df0f51e7476d155a2ea2dcbce4e5d0082c&scene=21#wechat_redirect)中已永久代移除永久代，同时增加了元数据区（Metaspace）。
第六，运行时常量池（Run-Time Constant Pool)，这是方法区的一部分，受到方法区内存的限制，当常量池无法再申请到内存时，会抛出OutOfMemoryError异常。在Class文件中，除了有类的版本、方法、字段、接口等描述信息外，还有一项信息是常量池。每个Class文件的头四个字节称为Magic Number，它的作用是确定这是否是一个可以被虚拟机接受的文件；接着的四个字节存储的是Class文件的版本号。紧挨着版本号之后的，就是常量池入口了。常量池主要存放两大类常量：

*   字面量（Literal），如文本字符串、final常量值
*   符号引用，存放了与编译相关的一些常量，因为Java不像C++那样有连接的过程，因此字段方法这些符号引用在运行期就需要进行转换，以便得到真正的内存入口地址。

class文件中的常量池，也称为静态常量池，JVM虚拟机完成类装载操作后，会把静态常量池加载到内存中，存放在运行时常量池。
第七，直接内存（Direct Memory），直接内存并不属于Java规范规定的属于Java虚拟机运行时数据区的一部分。Java的NIO可以使用Native方法直接在java堆外分配内存，使用DirectByteBuffer对象作为这个堆外内存的引用。下面这张图，反映了运行中的Java进程内存占用情况：![](https://upload-images.jianshu.io/upload_images/19956127-aaa04c323bd1bf2a?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**2、OOM可能发生在哪些区域上？**

根据javadoc的描述，OOM是指JVM的内存不够用了，同时垃圾收集器也无法提供更多的内存。从描述中可以看出，在JVM抛出OutOfMemoryError之前，垃圾收集器一般会出马先尝试回收内存。

从上面分析的Java数据区来看，除了程序计数器不会发生OOM外，哪些区域会发生OOM的情况呢？
第一，堆内存。堆内存不足是最常见的发送OOM的原因之一，如果在堆中没有内存完成对象实例的分配，并且堆无法再扩展时，将抛出OutOfMemoryError异常。当前主流的JVM可以通过-Xmx和-Xms来控制堆内存的大小，发生堆上OOM的可能是存在内存泄露，也可能是堆大小分配不合理。
第二，Java虚拟机栈和本地方法栈，这两个区域的区别不过是虚拟机栈为虚拟机执行Java方法服务，而本地方法栈则为虚拟机使用到的Native方法服务，在内存分配异常上是相同的。在JVM规范中，对Java虚拟机栈规定了两种异常：1.如果线程请求的栈大于所分配的栈大小，则抛出StackOverFlowError错误，比如进行了一个不会停止的递归调用；2\. 如果虚拟机栈是可以动态拓展的，拓展时无法申请到足够的内存，则抛出OutOfMemoryError错误。
第三，直接内存。直接内存虽然不是虚拟机运行时数据区的一部分，但既然是内存，就会受到物理内存的限制。在JDK1.4中引入的NIO使用Native函数库在堆外内存上直接分配内存，但直接内存不足时，也会导致OOM。
第四，方法区。随着Metaspace元数据区的引入，方法区的OOM错误信息也变成了“java.lang.OutOfMemoryError:Metaspace”。对于旧版本的Oracle JDK，由于永久代的大小有限，而JVM对永久代的垃圾回收并不积极，如果往永久代不断写入数据，例如String.Intern()的调用，在永久代占用太多空间导致内存不足，也会出现OOM的问题，对应的错误信为“java.lang.OutOfMemoryError:PermGen space”![](https://upload-images.jianshu.io/upload_images/19956127-99775a55db7af665?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**3、堆内存结构是怎么样的？**可以借助一些工具来了解JVM的内存内容，具体到特定的内存区域，应该用什么工具去定位呢？

*   图形化工具。图形化工具的优点是直观，连接到Java进程后，可以显示堆内存、堆外内存的使用情况，类似的工具有JConsole,VisualVm等。
*   命令行工具。这类工具可以在运行时进行查询，包括jstat，jmap等，可以对堆内存、方法区等进行查看。定位线上问题时也多会使用这些工具。jmap也可以生成堆转储文件（Heap Dump）文件，如果是在linux上，可以将堆转储文件拉到本地来，使用Eclipse MAT进行分析，也可以使用jhap进行分析。

关于内存的监控与诊断，在后面会进行深入了解。现在来看下一个问题：堆内的结构是怎么的呢？[深入浅出 Java 中 JVM 内存管理](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247487556&idx=2&sn=174f51201a53679564d40ade1cc5cb1d&chksm=eb539572dc241c64ffa677116e55ae4723920b53a4e2d6f4835190e11d93a324778262d22763&scene=21#wechat_redirect)，这篇推荐看下。站在垃圾收集器的角度来看，可以把内存分为新生代与老年代。内存的分配规则取决于当前使用的是哪种垃圾收集器的组合，以及内存相关的参数配置。往大的方向说，对象优先分配在新生代的Eden区域，而大对象直接进入老年代。第一, 新生代的Eden区域，对象优先分配在该区域，同时JVM可以为每个线程分配一个私有的缓存区域，称为TLAB（Thread Local Allocation Buffer），避免多线程同时分配内存时需要使用加锁等机制而影响分配速度。TLAB在堆上分配，位于Eden中。TLAB的结构如下：

```
// ThreadLocalAllocBuffer: a descriptor for thread-local storage used by// the threads for allocation.// It is thread-private at any time, but maybe multiplexed over// time across multiple threads. The park()/unpark() pair is// used to make it avaiable for such multiplexing.class ThreadLocalAllocBuffer: public CHeapObj<mtThread> {  friend class VMStructs;private:  HeapWord* _start; // address of TLAB  HeapWord* _top; // address after last allocation  HeapWord* _pf_top; // allocation prefetch watermark  HeapWord* _end; // allocation end (excluding alignment_reserve)  size_t    _desired_size; // desired size (including alignment_reserve)  size_t    _refill_waste_limit; // hold onto tlab if free() is larger than this
```

从本质上来说，TLAB的管理是依靠三个指针：start、end、top。start与end标记了Eden中被该TLAB管理的区域，该区域不会被其他线程分配内存所使用，top是分配指针，开始时指向start的位置，随着内存分配的进行，慢慢向end靠近，当撞上end时触发TLAB refill。因此内存中Eden的结构大体为：![](https://upload-images.jianshu.io/upload_images/19956127-fc53b19035091459?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第二、新生代的Survivor区域。当Eden区域内存不足时会触发Minor GC，也称为新生代GC，在Minor GC存活下来的对象，会被复制到Survivor区域中。我认为Survivor区的作用在于避免过早触发Full GC。如果没有Survivor，Eden区每进行一次Minor GC都把对象直接送到老年代，老年代很快便会内存不足引发Full GC。新生代中有两个Survivor区，我认为两个Survivor的作用在于提高性能，避免内存碎片的出现。在任何时候，总有一个Survivor是empty的，在发生Minor GC时，会将Eden及另一个的Survivor的存活对象拷贝到该empty Survivor中，从而避免内存碎片的产生。新生代的内存结构大体为：![](https://upload-images.jianshu.io/upload_images/19956127-a57cc10af6cc643e?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第三、老年代。老年代放置长生命周期的对象，通常是从Survivor区域拷贝过来的对象，不过当对象过大的时候，无法在新生代中用连续内存的存放，那么这个大对象就会被直接分配在老年代上。一般来说，普通的对象都是分配在TLAB上，较大的对象，直接分配在Eden区上的其他内存区域，而过大的对象，直接分配在老年代上。
第四、永久代。如前面所说，在早起的Hotspot JVM中有老年代的概念，老年代用于存储Java类的元数据、常量池、Intern字符串等。在[JDK8](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247483776&idx=1&sn=aa0203cfca67ff450ecaf1d9d147582d&chksm=eb5384b6dc240da057200d0f0edbf82ef41c03ea78df0f51e7476d155a2ea2dcbce4e5d0082c&scene=21#wechat_redirect)之后，就将老年代移除，而引入元数据区的概念。
第五、Vritual空间。前面说过，可以使用Xms与Xmx来指定堆的最小与最大空间。如果Xms小于Xmx，堆的大小不会直接扩展到上限，而是留着一部分等待内存需求不断增长时，再分配给新生代。Vritual空间便是这部分保留的内存区域。关注微信公众号：Java技术栈，在后台回复：JVM，可以获取我整理的 N 篇最新JVM 教程，都是干货。那么综上所述，可以画出Java堆内的内存结构大体为：![](https://upload-images.jianshu.io/upload_images/19956127-913235be381d9884?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过一些参数，可以来指定上述的堆内存区域的大小：

*   -Xmx value 指定最大的堆大小
*   -Xms value 指定初始的最小堆大小
*   -XX:NewSize = value 指定新生代的大小
*   -XX：NewRatio = value 老年代与新生代的大小比例。默认情况下，这个比例是2，也就是说老年代是新生代的2倍大。老年代过大的时候，Full GC的时间会很长；老年代过小，则很容易触发Full GC，Full GC频率过高，这就是这个参数会造成的影响。
*   -XX：SurvivorRation = value . 设置Eden与Srivivor的大小比例，如果该值为8，代表一个Survivor是Eden的1/8，是整个新生代的1/10。

**4、常用的性能监控与问题定位工具有哪些？**在系统的性能分析中，[CPU](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247491744&idx=3&sn=cddccdabe0d3892c4817dbd03229e647&chksm=eb506596dc27ec80c6335412cb031661ad3afed9dc1793be89ee835a1062e7a3b08f5b8e9c2c&scene=21#wechat_redirect)、内存与IO是主要的关注项。很多时候服务出现问题，在这三者上会体现出现，比如[CPU](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247491744&idx=3&sn=cddccdabe0d3892c4817dbd03229e647&chksm=eb506596dc27ec80c6335412cb031661ad3afed9dc1793be89ee835a1062e7a3b08f5b8e9c2c&scene=21#wechat_redirect)飙升，内存不足发生[OOM](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247489456&idx=1&sn=f24823b901bf4a697b7593737546e548&chksm=eb539286dc241b90529e3aa5efee03dc2cf41c9884c2e015ea0123ead6c02dda605e607d4c15&scene=21#wechat_redirect)等，这时候需要使用对应的工具，来对性能进行监控，对问题进行定位。对于CPU的监控，首先可以使用top命令来进行查看，下面是使用top查看负载的一个截图：![](https://upload-images.jianshu.io/upload_images/19956127-9a734656107d357a?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

load average 代表1分钟、5分钟、15分钟的系统平均负载，从这三个数字，可以判断系统负荷是大还是小。当[CPU](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247491744&idx=3&sn=cddccdabe0d3892c4817dbd03229e647&chksm=eb506596dc27ec80c6335412cb031661ad3afed9dc1793be89ee835a1062e7a3b08f5b8e9c2c&scene=21#wechat_redirect)完全空闲的时候，平均负荷为0；当[CPU](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247491744&idx=3&sn=cddccdabe0d3892c4817dbd03229e647&chksm=eb506596dc27ec80c6335412cb031661ad3afed9dc1793be89ee835a1062e7a3b08f5b8e9c2c&scene=21#wechat_redirect)工作量饱和的时候，平均负荷为1。因此 load average 这三个数值越低，代表系统负荷越小，那么什么时候能看出系统负荷比较重呢？这篇文章（Understanding Linux CPU Load – when should you be worried）里解释得非常通俗。如果电脑里只有一个[CPU](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247491744&idx=3&sn=cddccdabe0d3892c4817dbd03229e647&chksm=eb506596dc27ec80c6335412cb031661ad3afed9dc1793be89ee835a1062e7a3b08f5b8e9c2c&scene=21#wechat_redirect)，把[CPU](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247491744&idx=3&sn=cddccdabe0d3892c4817dbd03229e647&chksm=eb506596dc27ec80c6335412cb031661ad3afed9dc1793be89ee835a1062e7a3b08f5b8e9c2c&scene=21#wechat_redirect)看成一条单行桥，桥上只有一个车道，所有的车都必须从这个桥上通过。那么系统负荷为0，代表桥上一辆车也没有![](https://upload-images.jianshu.io/upload_images/19956127-c91eb5d32ef5d9eb?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

系统负荷0.5，意味着桥上一半路段上有车![](https://upload-images.jianshu.io/upload_images/19956127-c071dba0843e4b40?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

系统负荷1，意味着桥上道路已经被车占满![](https://upload-images.jianshu.io/upload_images/19956127-4bbf51b7d13b72d5?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

系统负荷1.7，代表着在桥上车子已经满了（100%），同时还有70%的车子在等待从桥上通过：![](https://upload-images.jianshu.io/upload_images/19956127-98340e9bee8d733e?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从top命令的截图中可以看到这三个值机器的load average非常低。如果这三个值非常高，比如超过了50%或60%，就应当引起注意。从时间维度上来说，如果发现[CPU](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247491744&idx=3&sn=cddccdabe0d3892c4817dbd03229e647&chksm=eb506596dc27ec80c6335412cb031661ad3afed9dc1793be89ee835a1062e7a3b08f5b8e9c2c&scene=21#wechat_redirect)负荷慢慢升高，也需要警惕。其他的内存、[CPU](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247491744&idx=3&sn=cddccdabe0d3892c4817dbd03229e647&chksm=eb506596dc27ec80c6335412cb031661ad3afed9dc1793be89ee835a1062e7a3b08f5b8e9c2c&scene=21#wechat_redirect)等性能监控工具的使用，以一张脑图来展示：![](https://upload-images.jianshu.io/upload_images/19956127-7578f0983ce6c785?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

具体的使用方式可以参考从一次线上[故障](http://mp.weixin.qq.com/s?__biz=MzI3OTU0MzI4MQ==&mid=2247488104&idx=1&sn=8e59dbd80c530b8d8d877ff6b61a0a20&chksm=eb477ef6dc30f7e0de3c072f190556f2619c453d7cc23750d240ce61c0793c37ba56921c6c90&scene=21#wechat_redirect)思考Java问题定位思路。

**参考**

*   https://www.cnblogs.com/dreamroute/p/5946272.html

*   https://docs.oracle.com/javase/specs/jvms/se9/html/jvms-2.html#jvms-2.5

*   https://www.cnblogs.com/Kidezyq/p/8040338.html

*   https://www.cnblogs.com/baihuitestsoftware/articles/6405580.html

*   https://www.jianshu.com/p/cd85098cca39

*   http://www.ruanyifeng.com/blog/2011/07/linux_load_average_explained.html
<article class="_2rhmJa" deep="5" style="box-sizing: border-box; display: block; font-weight: 400; line-height: 1.8; margin-bottom: 20px; color: rgb(64, 64, 64); font-family: -apple-system, BlinkMacSystemFont, &quot;Apple Color Emoji&quot;, &quot;Segoe UI Emoji&quot;, &quot;Segoe UI Symbol&quot;, &quot;Segoe UI&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei&quot;, &quot;Helvetica Neue&quot;, Helvetica, Arial, sans-serif; font-size: 16px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; text-decoration-style: initial; text-decoration-color: initial;">

# 2020金三银四提前备战完整精编面试解析**下载地址：[https://shimo.im/docs/3Tvytq686Yyv83KX](https://links.jianshu.com/go?to=https%3A%2F%2Fshimo.im%2Fdocs%2F3Tvytq686Yyv83KX)**


