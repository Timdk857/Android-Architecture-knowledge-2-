作者：Focusing

链接：https://juejin.im/post/5c984e926fb9a070c975a9b4
![](https://upload-images.jianshu.io/upload_images/19956127-cf210f7e823990c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**1、如何进行单元测试，如何保证App稳定 ？**

参考回答：要测试Android应用程序，通常会创建以下类型自动单元测试：

*   本地测试：只在本地机器JVM上运行，以最小化执行时间，这种单元测试不依赖于Android框架，或者即使有依赖，也很方便使用模拟框架来模拟依赖，以达到隔离Android依赖的目的，模拟框架如Google推荐的Mockito；

*   Android官网-建立本地单元测试（*https://developer.android.com/training/testing/unit-testing/local-unit-tests.html*）

*   检测测试：真机或模拟器上运行的单元测试，由于需要跑到设备上，比较慢，这些测试可以访问仪器（Android系统）信息，比如被测应用程序的上下文，一般地，依赖不太方便通过模拟框架模拟时采用这种方式；

*   Android官网-建立仪表单元测试（*https://developer.android.com/training/testing/unit-testing/instrumented-unit-tests.html*）

注意：单元测试不适合测试复杂的UI交互事件

推荐文章：Android 单元测试只看这一篇就够了（*https://juejin.im/post/5b57e3fbf265da0f47352618*）

App的稳定主要决定于整体的系统架构设计，同时也不可忽略代码编程的细节规范，正所谓“千里之堤，溃于蚁穴”，一旦考虑不周，看似无关紧要的代码片段可能会带来整体软件系统的崩溃，所以上线之前除了自己本地化测试之外还需要进行Monkey压力测试。

少部分面试官可能会延伸，如Gradle自动化测试、机型适配测试等

 **2、Android中如何查看一个对象的回收情况 ？** 

参考回答：首先要了解Java四种引用类型的场景和使用（强引用、软引用、弱引用、虛引用）

举个场景例子：SoftReference对象是用来保存软引用的，但它同时也是一个Java对象，所以当软引用对象被回收之后，虽然这个SoftReference对象的get方法返回null，但SoftReference对象本身并不是null，而此时这个SoftReference对象已经不再具有存在的价值，需要一个适当的清除机制，避免大量SoftReference对象带来的内存泄露。

因此，Java提供ReferenceQueue来处理引用对象的回收情况。当SoftReference所引用的对象被GC后，JVM会先将softReference对象添加到ReferenceQueue这个队列中。当我们调用ReferenceQueue的poll()方法，如果这个队列中不是空队列，那么将返回并移除前面添加的那个Reference对象。

![](https://upload-images.jianshu.io/upload_images/19956127-e2d18f5120c86916?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

推荐文章：Java中的四种引用类型：强引用、软引用、弱引用和虚引用（*https://segmentfault.com/a/1190000015282652#articleHeader3*）

**3、Apk的大小如何压缩 ？**

参考回答：一个完整APK包含以下目录（将APK文件拖到Android Studio）：

*   META-INF/：包含CERT.SF和CERT.RSA签名文件以及MANIFEST.MF 清单文件。

*   assets/：包含应用可以使用AssetManager对象检索的应用资源。

*   res/：包含未编译到的资源 resources.arsc。

*   lib/：包含特定于处理器软件层的编译代码。该目录包含了每种平台的子目录，像armeabi，armeabi-v7a， arm64-v8a，x86，x86_64，和mips。

*   resources.arsc：包含已编译的资源。该文件包含res/values/ 文件夹所有配置中的XML内容。打包工具提取此XML内容，将其编译为二进制格式，并将内容归档。此内容包括语言字符串和样式，以及直接包含在**resources.arsc*8文件中的内容路径 ，例如布局文件和图像。

*   classes.dex：包含以Dalvik / ART虚拟机可理解的DEX文件格式编译的类。

*   AndroidManifest.xml：包含核心Android清单文件。该文件列出应用程序的名称，版本，访问权限和引用的库文件。该文件使用Android的二进制XML格式。

![](https://upload-images.jianshu.io/upload_images/19956127-3b4b7ffe078eab8d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

*   lib、class.dex和res占用了超过90%的空间，所以这三块是优化Apk大小的重点（实际情况不唯一）

**减少res，压缩图文文件**：图片文件压缩是针对jpg和png格式的图片。我们通常会放置多套不同分辨率的图片以适配不同的屏幕，这里可以进行适当的删减。在实际使用中，只保留一到两套就足够了（保留一套的话建议保留xxhdpi，两套的话就加上hdpi），然后再对剩余的图片进行压缩(jpg采用优图压缩，png尝试采用pngquant压缩)

**减少dex文件大小**：

*   添加资源混淆

![](https://upload-images.jianshu.io/upload_images/19956127-e49b440e0286e241?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

*   shrinkResources为true表示移除未引用资源，和代码压缩协同工作。

*   minifyEnabled为true表示通过ProGuard启用代码压缩，配合proguardFiles的配置对代码进行混淆并移除未使用的代码。

*   代码混淆在压缩apk的同时，也提升了安全性。

推荐文章：Android混淆最佳实践（*https://www.jianshu.com/p/cba8ca7fc36d*）

**减少lib文件大小**：由于引用了很多第三方库，lib文件夹占用的空间通常都很大，特别是有so库的情况下。很多so库会同时引入armeabi、armeabi-v7a和x86这几种类型，这里可以只保留armeabi或armeabi-v7a的其中一个就可以了，实际上微信等主流app都是这么做的。

只需在build.gradle直接配置即可，NDK配置同理

![](https://upload-images.jianshu.io/upload_images/19956127-10d1d27bb5f2b5c2?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

推荐文章：APK瘦身（*https://www.jianshu.com/p/5921e9561f5f*）

 **4、如何通过Gradle配置多渠道包？**

参考回答：首先要了解设置多渠道的原因。在安装包中添加不同的标识，配合自动化埋点，应用在请求网络的时候携带渠道信息，方便后台做运营统计，比如说统计我们的应用在不同应用市场的下载量等信息

这里以友盟统计为例：

*   首先在manifest.xml文件中设置动态渠道变量：

![](https://upload-images.jianshu.io/upload_images/19956127-4f970d5576b05eb0?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

*   接着在app目录下的build.gradle中配置productFlavors，也就是配置打包的渠道：

![](https://upload-images.jianshu.io/upload_images/19956127-0b9f46fc63d2d7e3?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

最后在编辑器下方的Teminal输出命令行：

*   执行./gradlew assembleRelease ，将会打出所有渠道的release包；

*   执行./gradlew assembleVIVO，将会打出VIVO渠道的release和debug版的包；

*   执行./gradlew assembleVIVORelease将生成VIVO的release包。

推荐文章：美团Android自动化之旅—Walle生成渠道包（*https://github.com/Meituan-Dianping/walle*）

 **5、插件化原理分析** 

参考回答：插件化是指将 APK 分为宿主和插件的部分。把需要实现的模块或功能当做一个独立的提取出来，在 APP 运行时，我们可以动态的载入或者替换插件部分，减少宿主的规模

*   宿主： 就是当前运行的APP。

*   插件： 相对于插件化技术来说，就是要加载运行的apk类文件。

而热修复则是从修复bug的角度出发，强调的是在不需要二次安装应用的前提下修复已知的bug。

![](https://upload-images.jianshu.io/upload_images/19956127-46a0890e4b0b0418?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

**类加载机制**：Android中常用的两种类加载器，DexClassLoader和PathClassLoader，它们都继承于BaseDexClassLoader，两者区别在于PathClassLoader只能加载内部存储目录的dex/jar/apk文件。DexClassLoader支持加载指定目录(不限于内部)的dex/jar/apk文件

**插件通信**：通过给插件apk生成相应的DexClassLoader便可以访问其中的类，可分为单DexClassLoader和多DexClassLoader两种结构。

*   若使用多ClassLoader机制，主工程引用插件中类需要先通过插件的ClassLoader加载该类再通过反射调用其方法。插件化框架一般会通过统一的入口去管理对各个插件中类的访问，并且做一定的限制。

*   若使用单ClassLoader机制，主工程则可以直接通过类名去访问插件中的类。该方式有个弊端，若两个不同的插件工程引用了一个库的不同版本，则程序可能会出错。

**资源加载**：原理在于通过反射将插件apk的路径加入AssetManager中并创建Resource对象加载资源，有两种处理方式：

*   合并式：addAssetPath时加入所有插件和主工程的路径；由于AssetManager中加入了所有插件和主工程的路径，因此生成的Resource可以同时访问插件和主工程的资源。但是由于主工程和各个插件都是独立编译的，生成的资源id会存在相同的情况，在访问时会产生资源冲突。

*   独立式：各个插件只添加自己apk路径，各个插件的资源是互相隔离的，不过如果想要实现资源的共享，必须拿到对应的Resource对象。

推荐文章：

Android动态加载技术 简单易懂的介绍方式（*https://segmentfault.com/a/1190000004062866*）

深入理解Android插件化技术（*https://yq.aliyun.com/articles/361233*）

为什么要做热更新（*https://www.cnblogs.com/baiqiantao/p/9160806.html*）

 **6、组件化原理**

参考回答：引入组件化的原因：项目随着需求的增加规模变得越来越大，规模的增大导致了各种业务错中复杂的交织在一起, 每个业务模块之间，代码没有约束，带来了代码边界的模糊，代码冲突时有发生, 更改一个小问题可能引起一些新的问题, 牵一发而动全身，增加一个新需求，需要熟悉相关的代码逻辑，增加开发时间

*   避免重复造轮子，可以节省开发和维护的成本。

*   可以通过组件和模块为业务基准合理地安排人力，提高开发效率。

*   不同的项目可以共用一个组件或模块，确保整体技术方案的统一性。

*   为未来插件化共用同一套底层模型做准备。

组件化开发流程就是把一个功能完整的App或模块拆分成多个子模块（Module），每个子模块可以独立编译运行，也可以任意组合成另一个新的 App或模块，每个模块即不相互依赖但又可以相互交互，但是最终发布的时候是将这些组件合并统一成一个apk，遇到某些特殊情况甚至可以升级或者降级

举个简单的模型例子：

![](https://upload-images.jianshu.io/upload_images/19956127-ee4c33d13ac1572c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

![](https://upload-images.jianshu.io/upload_images/19956127-6c65027365bbc333?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

App是主application，ModuleA和ModuleB是两个业务模块（相对独立，互不影响），Library是基础模块，包含所有模块需要的依赖库，以及一些工具类：如网络访问、时间工具等。

**注意：**提供给各业务模块的基础组件，需要根据具体情况拆分成 aar 或者 library，像登录，基础网络层这样较为稳定的组件，一般直接打包成 aar，减少编译耗时。而像自定义 View 组件，由于随着版本迭代会有较多变化，就直接以源码形式抽离成 Library

推荐文章：干货 | 从智行 Android 项目看组件化架构实践（*https://mp.weixin.qq.com/s?__biz=MjM5MDI3MjA5MQ==&mid=2697268363&idx=1&sn=3db2dce36a912936961c671dd1f71c78*）

 **7、跨组件通信** 

**跨组件通信场景**：

*   第一种是组件之间的页面跳转 (Activity 到 Activity, Fragment 到 Fragment, Activity 到 Fragment, Fragment 到 Activity) 以及跳转时的数据传递 (基础数据类型和可序列化的自定义类类型)。

*   第二种是组件之间的自定义类和自定义方法的调用(组件向外提供服务)。

**跨组件通信方案分析**：第一种组件之间的页面跳转实现简单，跳转时想传递不同类型的数据提供有相应的 API即可。

第二种组件之间的自定义类和自定义方法的调用要稍微复杂点，需要 ARouter 配合架构中的 公共服务(CommonService) 实现：

*   提供服务的业务模块：

*   在公共服务(CommonService) 中声明 Service 接口 (含有需要被调用的自定义方法), 然后在自己的模块中实现这个 Service 接口, 再通过 ARouter API 暴露实现类。

*   使用服务的业务模块：通过 ARouter 的 API 拿到这个 Service 接口(多态持有, 实际持有实现类), 即可调用 Service 接口中声明的自定义方法, 这样就可以达到模块之间的交互。

*   此外，可以使用 AndroidEventBus 其独有的 Tag, 可以在开发时更容易定位发送事件和接受事件的代码, 如果以组件名来作为 Tag 的前缀进行分组, 也可以更好的统一管理和查看每个组件的事件, 当然也不建议大家过多使用 EventBus。

**如何管理过多的路由表？**

*   RouterHub 存在于基础库, 可以被看作是所有组件都需要遵守的通讯协议, 里面不仅可以放路由地址常量, 还可以放跨组件传递数据时命名的各种 Key 值, 再配以适当注释, 任何组件开发人员不需要事先沟通只要依赖了这个协议, 就知道了各自该怎样协同工作, 既提高了效率又降低了出错风险, 约定的东西自然要比口头上说强。

*   Tips: 如果您觉得把每个路由地址都写在基础库的 RouterHub 中, 太麻烦了, 也可以在每个组件内部建立一个私有 RouterHub, 将不需要跨组件的路由地址放入私有 RouterHub 中管理, 只将需要跨组件的路由地址放入基础库的公有 RouterHub 中管理, 如果您不需要集中管理所有路由地址的话, 这也是比较推荐的一种方式。

**ARouter路由原理**：ARouter维护了一个路由表Warehouse，其中保存着全部的模块跳转关系，ARouter路由跳转实际上还是调用了startActivity的跳转，使用了原生的Framework机制，只是通过apt注解的形式制造出跳转规则，并人为地拦截跳转和设置跳转条件。

常见的组件化方案如下：

![](https://upload-images.jianshu.io/upload_images/19956127-ef7db9373ee85326?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

 **8、组件化中路由、埋点的实现** 

参考回答：因为在组件化中，各个业务模块之间是各自独立的, 并不会存在相互依赖的关系, 所以一个业务模块是访问不了其他业务模块的代码的, 如果想从 A 业务模块的 A 页面跳转到 B 业务模块的 B 页面, 光靠模块自身是不能实现的，这就需要一种跨组件通信方案—— 路由（Router）

路由主要有以下两种场景:

*   第一种是组件之间的页面跳转 (Activity 到 Activity, Fragment 到 Fragment, Activity 到 Fragment, Fragment 到 Activity) 以及跳转时的数据传递 (基础数据类型和可序列化的自定义类类型)

*   第二种是组件之间的自定义类和自定义方法的调用(组件向外提供服务)

其原理在于将分布在不同组件module中的某些类按照一定规则生成映射表（数据结构通常是Map，Key为一个字符串，Value为类或对象），然后在需要用到的时候从映射表中根据字符串从映射表中取出类或对象，本质上是类的查找。

![](https://upload-images.jianshu.io/upload_images/19956127-325cc9d18e4d0028?imageMogr2/auto-orient/strip)

埋点则是在应用中特定的流程收集一些信息，用来跟踪应用使用的状况：

*   代码埋点：在某个事件发生时调用SDK里面相应的接口发送埋点数据，百度统计、友盟、TalkingData、Sensors Analytics等第三方数据统计服务商大都采用这种方案

*   全埋点：全埋点指的是将Web页面/App内产生的所有的、满足某个条件的行为，全部上报到后台服务器

*   可视化埋点：通过可视化工具（例如Mixpanel）配置采集节点，在Android端自动解析配置并上报埋点数据，从而实现所谓的自动埋点

*   无埋点：它并不是真正的不需要埋点，而是Android端自动采集全部事件并上报埋点数据，在后端数据计算时过滤出有用数据

推荐文章：安卓组件化开源方案实现（*https://juejin.im/post/5a7ab8846fb9a0634514a2f5*）

 **9、Hook以及插桩技术** 

参考回答：Hook是一种用于改变API执行结果的技术，能够将系统的API函数执行重定向（应用的触发事件和后台逻辑处理是根据事件流程一步步地向下执行。而Hook的意思，就是在事件传送到终点前截获并监控事件的传输，像个钩子钩上事件一样，并且能够在钩上事件时，处理一些自己特定的事件，例如逆向破解App）

![image.gif](https://upload-images.jianshu.io/upload_images/19956127-2246df399c9e5adc.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

Android 中的 Hook 机制，大致有两个方式：

*   要 root 权限，直接 Hook 系统，可以干掉所有的 App。

*   无 root 权限，但是只能 Hook 自身app，对系统其它 App 无能为力。

插桩是以静态的方式修改第三方的代码，也就是从编译阶段，对源代码（中间代码）进行编译，而后重新打包，是静态的篡改； 而Hook则不需要再编译阶段修改第三方的源码或中间代码，是在运行时通过反射的方式修改调用，是一种动态的篡改

推荐文章：

Android插件化原理解析——Hook机制之动态代理（*http://weishu.me/2016/01/28/understand-plugin-framework-proxy-hook/*）

android 插桩基本概念（*https://blog.csdn.net/fei20121106/article/details/51879047*）

Android逆向之旅（*http://www.520monkey.com/*）

 10、Android的签名机制？

参考回答：Android的签名机制包含有消息摘要、数字签名和数字证书

*   **消息摘要：**在消息数据上，执行一个单向的 Hash 函数，生成一个固定长度的Hash值

*   **数字签名：**一种以电子形式存储消息签名的方法，一个完整的数字签名方案应该由两部分组成：签名算法和验证算法

*   **数字证书：**一个经证书授权（Certificate Authentication）中心数字签名的包含公钥拥有者信息以及公钥的文件

推荐文章：一篇文章看明白 Android v1 & v2 签名机制（*https://blog.csdn.net/freekiteyu/article/details/84849651*）

 **11、v3签名key和v2还有v1有什么区别** 

参考回答：在v1版本的签名中，签名以文件的形式存在于apk包中，这个版本的apk包就是一个标准的zip包，V2和V1的差别是V2是对整个zip包进行签名，而且在zip包中增加了一个apk signature block，里面保存签名信息。

![](https://upload-images.jianshu.io/upload_images/19956127-cded71524dc3c664?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

**v2版本签名**块（APK Signing Block）本身又主要分成三部分:

*   **SignerData（签名者数据）**：主要包括签名者的证书，整个APK完整性校验hash，以及一些必要信息

*   **Signature（签名）**：开发者对SignerData部分数据的签名数据

*   **PublicKey（公钥）**：用于验签的公钥数据

**v3版本签名**块也分成同样的三部分，与v2不同的是在SignerData部分，v3新增了attr块，其中是由更小的level块组成。每个level块中可以存储一个证书信息。前一个level块证书验证下一个level证书，以此类推。最后一个level块的证书，要符合SignerData中本身的证书，即用来签名整个APK的公钥所属于的证书

![](https://upload-images.jianshu.io/upload_images/19956127-8ae60ea4b41c3c3c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

推荐文章：

APK 签名方案 v3（*https://source.android.google.cn/security/apksigning/v3*）

Android P v3签名新特性（*https://xuanxuanblingbling.github.io/ctf/android/2018/12/30/signature/*）

 **12、Android5.0~10.0之间大的变化** 

Android **5.0**新特性：

*   MaterialDesign设计风格

*   支持64位ART虚拟机（5.0推出的ART虚拟机，在5.0之前都是Dalvik。他们的区别是：Dalvik,每次运行,字节码都需要通过即时编译器转换成机器码(JIT)。ART,第一次安装应用的时候,字节码就会预先编译成机器码(AOT)）

*   通知详情可以用户自己设计

Android **6.0**新特性

*   动态权限管理

*   支持快速充电的切换

*   支持文件夹拖拽应用

*   相机新增专业模式

Android **7.0**新特性

*   多窗口支持

*   V2签名

*   增强的Java8语言模式

*   夜间模式

Android** 8.0（O）**新特性

*   优化通知：通知渠道 (Notification Channel) 通知标志 休眠 通知超时 通知设置 通知清除

*   画中画模式：清单中Activity设置android:supportsPictureInPicture

*   后台限制

*   自动填充框架

*   系统优化等等优化很多

Android **9.0（P）**新特性

*   室内WIFI定位

*   “刘海”屏幕支持

*   安全增强等等优化很多

Android **10.0（Q）**新特性

*   夜间模式：包括手机上的所有应用都可以为其设置暗黑模式。

*   桌面模式：提供类似于PC的体验，但是远远不能代替PC。

*   屏幕录制：通过长按“电源”菜单中的"屏幕快照"来开启。

推荐文章：Android Developers 官方文档（*https://developer.android.com/guide/topics/manifest/uses-sdk-element.html#ApiLevels*）

 **13、说下Measurepec这个类** 

参考回答：作用：通过宽测量值widthMeasureSpec和高测量值heightMeasureSpec决定View的大小

组成：一个32位int值，高2位代表SpecMode(测量模式)，低30位代表SpecSize( 某种测量模式下的规格大小)。

三种模式：

*   **UNSPECIFIED**：父容器不对View有任何限制，要多大有多大。常用于系统内部。

*   **EXACTLY(精确模式)**：父视图为子视图指定一个确切的尺寸SpecSize。对应LyaoutParams中的match_parent或具体数值。

*   **AT_MOST(最大模式)**：父容器为子视图指定一个最大尺寸SpecSize，View的大小不能大于这个值。对应LayoutParams中的wrap_content。

决定因素：值由子View的布局参数LayoutParams和父容器的MeasureSpec值共同决定。具体规则见下图：

![](https://upload-images.jianshu.io/upload_images/19956127-ad61d1cff58d6023?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

**14、请例举Android中常用布局类型,并简述其用法以及排版效率**

参考回答：Android中常用布局分为传统布局和新型布局

传统布局（编写XML代码、代码生成）：

*   框架布局（FrameLayout）：

*   线性布局（LinearLayout）：

*   绝对布局（AbsoluteLayout）：

*   相对布局（RelativeLayout）：

*   表格布局（TableLayout）：

新型布局（可视化拖拽控件、编写XML代码、代码生成）：约束布局（ConstrainLayout）：

![](https://upload-images.jianshu.io/upload_images/19956127-2ea00f836199023a?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

注：图片出自Carson_Ho的Android：常用布局介绍&属性设置大全（*https://blog.csdn.net/carson_ho/article/details/51719519*）

对于嵌套多层View而言，其排版效率：LinearLayout = FrameLayout >> RelativeLayout

 **15、区别Animation和Animator的用法，概述其原理** 

**动画的种类：**前者只有透明度，旋转，平移，伸缩4种属性，而对于后者，只要是该控件的属性，且有setter该属性的方法就都可以对该属性执行一种动态变化的效果。

**可操作的对象：**前者只能对UI组件执行动画，但属性动画几乎可以对任何对象执行动画（不管它是否显示在屏幕上）。

**动画播放顺序：**在Animator中，AnimatorSet正是通过playTogether()、playSequentially()、animSet.play().with()、before()、after()这些方法来控制多个动画协同工作，从而做到对动画播放顺序的精确控制

![](https://upload-images.jianshu.io/upload_images/19956127-5642bb34c4a5ecec?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

 **16、使用过什么图片加载库？**

Glide的源码设计哪里很微妙？

参考回答：图片加载库：Fresco、Glide、Picasso等

Glide的设计微妙在于：

*   **Glide的生命周期绑定**：可以控制图片的加载状态与当前页面的生命周期同步，使整个加载过程随着页面的状态而启动/恢复，停止，销毁

*   **Glide的缓存设计**：通过（三级缓存，Lru算法，Bitmap复用）对Resource进行缓存设计

*   **Glide的完整加载过程**：采用Engine引擎类暴露了一系列方法供Request操作

推荐文章：Glide 源码分析（*https://user-gold-cdn.xitu.io/2019/4/24/16a4ec49c3af1f5c*）

 **17、如何绕过9.0限制？**

参考回答：

![](https://upload-images.jianshu.io/upload_images/19956127-e49eb1c24efbd363?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

 **18、用过哪些网络加载库？**

OkHttp、Retrofit实现原理？

参考回答：网络加载库：OkHttp、Retrofit、xUtils、Volley等

推荐文章：

Android OkHttp源码解析入门教程（一）（*https://juejin.im/post/5c46822c6fb9a049ea394510*）

Android OkHttp源码解析入门教程（二）（*https://juejin.im/post/5c4682d2f265da6130752a1d*）

 **19、对于应用更新这块是如何做的？**

（灰度，强制更新、分区域更新）

**内部更新**：

*   通过接口获取线上版本号，versionCode

*   比较线上的versionCode 和本地的versionCode，弹出更新窗口

*   下载APK文件（文件下载）

*   安装APK

**灰度更新**：

*   找单一渠道投放特别版本。

*   做升级平台的改造，允许针对部分用户推送升级通知甚至版本强制升级。

*   开放单独的下载入口。

*   是两个版本的代码都打到app包里，然后在app端植入测试框架，用来控制显示哪个版本。测试框架负责与服务器端api通信，由服务器端控制app上A/B版本的分布，可以实现指定的一组用户看到A版本，其它用户看到B版本。服务端会有相应的报表来显示A/B版本的数量和效果对比。最后可以由服务端的后台来控制，全部用户在线切换到A或者B版本~

*   无论哪种方法都需要做好版本管理工作，分配特别的版本号以示区别。 当然，既然是做灰度，数据监控（常规数据、新特性数据、主要业务数据）还是要做到位，该打的数据桩要打。 还有，灰度版最好有收回的能力，一般就是强制升级下一个正式版。

**强制更新：**一般的处理就是进入应用就弹窗通知用户有版本更新，弹窗可以没有取消按钮并不能取消。这样用户就只能选择更新或者关闭应用了，当然也可以添加取消按钮，但是如果用户选择取消则直接退出应用。

**增量更新：**二进制差分工具bsdiff是相应的补丁合成工具，根据两个不同版本的二进制文件，生成补丁文件.patch文件。通过bspatch使旧的apk文件与不定文件合成新的apk。 注意通过apk文件的md5值进行区分版本。

 **20、会用Kotlin、Fultter吗？ 谈谈你的理解**

*   Kotlin是一种具有类型推断的跨平台，静态类型的通用编程语言。Kotlin旨在与Java完全互操作，其标准库的JVM版本依赖于Java类库，但类型推断允许其语法更简洁。

*   Flutter是由Google创建的开源移动应用程序开发框架。它用于开发Android和iOS的应用程序，以及为Google Fuchsia创建应用程序的主要方法

*   关于kotlin的重要性，相信大家在日常开发可以体会到，应用到实际开发中，需要避免语法糖（例如单列模式、空值判断、高阶函数等）

*   至于Flutter，目前Google官方文档还不完善，市面上采用此语言编写的项目较少，如需要具体深入，请参考闲鱼和官方文档

## 最后

其实Android开发的知识点就那么多，面试问来问去还是那么点东西。所以面试没有其他的诀窍，只看你对这些知识点准备的充分程度。so，出去面试时先看看自己复习到了哪个阶段就好。

上面分享的**腾讯、头条、阿里、美团、字节跳动等公司2019-2020年的Android中高级面试题**，这只是Android全套面试真题解析的小部分！
![2020Android春季面试专题解析](https://upload-images.jianshu.io/upload_images/19956127-d4aef70bf0e0f241.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

博主还把这些技术点整理成了视频和PDF（实际上比预期多花了不少精力），包含**知识脉络 + 诸多细节**，由于篇幅有限，上面只是以图片的形式给大家展示一部分。

【[Android学习PDF+学习视频+面试文档+知识点笔记]([https://shimo.im/docs/Q6V8xPVxHpkrtRtD/read](https://shimo.im/docs/Q6V8xPVxHpkrtRtD/read)
)】

**【Android思维脑图（技能树）】**

知识不体系？这里还有整理出来的Android进阶学习的思维脑图，给大家参考一个方向。

![](https://upload-images.jianshu.io/upload_images/1095900-4bc8337d3615bf27.png?imageMogr2/auto-orient/strip|imageView2/2/w/530/format/webp)

**【Android高级架构视频学习资源】**

**Android部分精讲视频领取学习后更加是如虎添翼！**进军BATJ大厂等（备战）！现在都说互联网寒冬，其实无非就是你上错了车，且穿的少（技能），要是你上对车，自身技术能力够强，公司换掉的代价大，怎么可能会被裁掉，都是淘汰末端的业务Curd而已！现如今市场上初级程序员泛滥，这套教程针对Android开发工程师1-6年的人员、正处于瓶颈期，想要年后突破自己涨薪的，进阶Android中高级、架构师对你更是如鱼得水，赶快领取吧！

**【Android进阶学习视频】、【全套Android面试秘籍】可以简信我【学习】查看免费领取方式！**


