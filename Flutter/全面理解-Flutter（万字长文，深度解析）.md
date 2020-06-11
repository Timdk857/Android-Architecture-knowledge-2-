当 Flutter 遇见 Web，会有怎样的秘密？

> 在线教育团队（简称：OED）已经将 Flutter 这样技术在业务中落地了，做为 IMWeb 前端团队的我们也要进行一些尝试。本文从前端角度进行 Flutter 开发的概况描述。主要是为了让您了解和感受一下：Flutter to Web 的实例、Flutter 为什么会出现、Flutter 设计实现原理、Flutter 技术特点和优势。

## **前言**

**OED**的客户端团队在 2019 年上半年 ，就已经把 Flutter 落地到**企鹅辅导**的业务中了。今年我们又一起去上海参加了 2019 年谷歌开发者大会，遇见了更多的 Flutter 开发者，这次体验比第一次去的时候感觉熟悉了很多。希望未来有机会把他们邀请来深圳，进行一些 Flutter 的技术分享。**此次开发者大会又恰逢 Flutter to Web 也已经正式合入 Master**，那么，前端同学是否可以趁着这股东风一起参与到 Flutter 的协同开发中呢，我想这问题会困扰着很多人？如果您有好的想法，可以在留言区参与评论。

本文不是一篇 Flutter 详细的学习教程，更像是一个概览，用尽可能平实的语言和对比的思路去描述它。本着依旧从前端同学的角度出发，去理解一项新的技术，但又不限于前端技术本身。希望您能通过这篇文章相对全面的理解 Flutter 这项技术本身。**特别感谢领导的鼓励和支持，让我有机会去学习和理解 Flutter 框架**，因为相对我而言，**OED** 的客户端团队的同学经验会远超于我，**他们已经完整经历了业务从 0 到 1 的过程**，这是一种非常有意思的体验。

## **1、Flutter to Web 案例**

下面转换的工程案例 是我们的 **企鹅辅导**APP 里面的业务代码（[详细的操作流程](https://link.zhihu.com/?target=https%3A//github.com/flutter/flutter_web/blob/master/OLD_README.md)）。Flutter 官方文档具有很好的说明，如果您单纯转一个完全不依赖 APP 的项目，您安装完环境并且切换到 Master 版本就可以直接进行了，甚至不需要任何代码修改。转换业务产品中的代码，还需要处理一些奇奇怪怪的问题，相信这对您来说，应该都不是问题。可以在这里**[安装和环境配置](https://link.zhihu.com/?target=https%3A//flutter.cn/docs/get-started/install)** **进行环境安装。Flutter 官网提供了一个** **[案例](https://link.zhihu.com/?target=https%3A//flutter.cn/docs/get-started/codelab)**可以尝试一下。官方给了一个开箱即用的 [开发文档](https://link.zhihu.com/?target=https%3A//flutter.cn/docs/development/ui/widgets-intro) 。

视频

上面的视频里面展示我们 **企鹅辅导** 的第三个 TAB 内的一个上课页的 Flutter 业务实践，以及转换后的 Web 页面。可以明显看到，确实有一局部有些失真，但是用户完全可用。**打包压缩之后，Web 端代码只有 1.4M，Mac 桌面端体验过程中，没有出现卡顿问题。**这里 Web 页面内的渲染是通过 **Canvas 渲染** 和 **DOM** 进行的页面填充。Dart 原本从天生就支持在 Chrome 中使用的，只是在 2015 年不幸夭折，但是，它长期支持转换为 JavaScript。因此，**可以遇见的未来，随着 Flutter 的发展，Dart to Js 业务实践的进化速度，可能会超过 WASM 的业务使用**。

下面是一个处理无限列表的场景，无论是在 Mac，还是在移动端时，依旧会有卡顿的现象，FPS 表现并不理想。

![](https://upload-images.jianshu.io/upload_images/19956127-cb2ca48f48422028.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**首先官方目前还不建议，在产品化中使用 。**但既然已经合入 Master，相信这一天也不会远了。

转换这里需要解决一些问题，整理了一下官方建议和实践的体验：

![](https://upload-images.jianshu.io/upload_images/19956127-3b226f41785180f2.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

业务转换使用的时候，需要把系统依赖解决掉，部分样式问题跟 Flutter 排版组件有关，而系统相关的如本地存储、网络请求需要我们自定义转化方案。例如：客户端使用的是 WNS 协议，而前端需要使用的是 HTTPS。目前看 Flutter to Web 作为业务容灾的策略还是可以的，总是优于 APP Crash 或者 影响范围非常大的 BUG 导致用户无法使用的情况出现。这里 Skia 有个明显的短板，就是 3D 动画，如果您业务对 3D 动画依赖比较强，一段时间内，就不要选择 Flutter 作为业务的技术选型了。

![](https://upload-images.jianshu.io/upload_images/19956127-00ca704ec250b10c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上面简单的罗列了一些前端在 Flutter 的工作范围，**前端定位更多是打辅助的！**当然如果您是全能型开发，也可以全部都做。技术本质上没有边界，局限的只有自己。从前端角度看 Flutter 的开发成本相比于 H5 确实还是多了一些成本，但学起来也不会太难，只是时间的问题。对于团队有人力，并且希望尝试新技术的，完全放在业务中尝试使用。

至于未来是否能确定是 Flutter 这个框架成为行业标杆，还不得而知。但如果想彻底统一全端技术栈，Flutter 今天的设计思路是一个非常有突破性的存在！**应该说，类 Flutter 的自绘引擎方案在未来会有机会大放异彩**。

站在前端的角度上，**我们尝试着在组件化和工程化的方向找到自己在 Flutter 生态中的定位**。并且相比于 IOS 和 Android，Flutter 的 Dart 代码是完全开源的，参与感是会让开发者提升不少的，就类似 linux 和 windows 的感觉。

至于团队是否要参与进去，很多时候是看综合的成本和收益，做与不做，做到什么程度，适合什么时候进行业务跟进，其实，都是要以团队的价值最大化为目标，没有绝对的对与错，结合团队的实际情况量身定制就好。错误的时机进入，也会付出不小的成本，您自己考量。

如下图，横向对比行业开源方案：

![](https://upload-images.jianshu.io/upload_images/19956127-05f58abdb973e467.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

简单对比来看，结合团队的技术实践和能力。站在第三方的角度上。RN 和 Flutter 相对是 2 个比较好的跨平台方案。而且其它方案或多或少都有一些局限。

**那真的放到业务上，又当怎么选择呢？**

回答这个问题确实有些艰难，仁者见仁智者见智。无论是相对成熟的 React Native，还是新贵 Flutter，放眼到整个大前端技术史里面看，都不如 Hybird 影响的深远。但如今它们也都达到了商用的标准。我同时使用过 Hybird 和 RN 作为业务接入方，算是会有一点点经验之谈。

从目前行业的产品，以及社区生态来说，React Native 整体还是胜出 Flutter 一筹。毕竟早出来几年，市场占有率和行业积累还是在的。但是长远来看，技术发展也有它的必然规律，Flutter 的技术理念已经领先了 React Native，作为大公司、或者大前端团队的技术储备和技术选型，科技公司要想在未来在行业有一席之地，使用 Flutter 这样的技术，必然也会是一个趋势。至于开发者对于技术本身，应该也会对 Flutter 保有浓厚的兴趣吧，毕竟技术永远都会向前发展，无论是谁，不进步一样会被淘汰。因此，作为技术开发人员首先保持的就是一颗好奇的心，进而持续成长。

当然，未来可能成功的框架不一定就是 Flutter，但是它设计理念和设计思路是一脉相承的，类 Flutter 框架一样也会出现。 就像 React 出来了，Vue 也会跟着出来了。有的团队使用 Vue，有的团队使用 React。但是，您会发现它们越来越像了。使用 Vue 的也没必要说使用 React 的同学水平不好，不存在的！理念很重要，**有一句话说的很有意思，叫 ~ 思路决定出路！**

## **2、Flutter 技术架构**

### **1）拥有了 RN，为什么又会出现 Flutter**

在谈及 Flutter 之前，我们还是要先简单回顾一下，客户端的上一次技术革新 —— ReactNative（此后简称 RN）。相信非常多的团队都有去落地实践 RN 的机会，很多 APP 的首屏渲染方案都是用 RN 技术栈进行的。我们自己的产品 **企鹅辅导** 和 **腾讯课堂** 内的应用也是一样。

这里简单回顾一下，在有客户端开发的场景下，为什么又出现了 RN ？

*RN 的价值简单来讲就是—— 可接受的页面性能 + 高效开发 + 热更新。*

**更新**：传统的 APP 上架之后，出现了业务 BUG，用户只能去更新 APP，进行 BUG 修复。客户端实现热更新修复 BUG，有多难，可以问问 IOS 的开发同学。大概率猜测，手 Q 和微信，应该还是有方案可以热更新的。但是对很多小厂商这确实是非常艰难的事情。因此，得益于强大的动态化能力 RN 的价值也就完美的体现出来了。

**高效**：一个 APP 发布上线，Android 和 IOS 同时需要开发两个应用，而 RN 只需要一套代码，就可以运行在双平台上，节省很大的人力成本。并且很多业务线有很强的业务运营诉求，可能会存在很短时间内的多次改版和发布的情况出现，客户端开发的人力瓶颈和发布周期的限制，已经很难满足这样的业务场景了。尤其在一些有损发布的情况下，赶着时间点，带着 BUG 上线的场景，在后续进行增量的修复，再这样的情况下，传统客户端的表现，简直就是灾难性的。

**性能**：RN 具有优于 H5 的性能体验。毕竟是通过客户端进行的页面渲染，速度上比 WebView 渲染还是要快不少的。这个在 Weex、Hippy、Plato 上都有所体现，虽然低于 Native 的性能，但是在可接受范围。

PS：这里的表达，不是描述客户端开发不好。只是单纯从业务角度上看待问题，而把合适的技术放在合适的位置是非常重要的，这也是架构师核心价值之一。

回顾了以上三点，我们发现 RN 的出现，有它的必然性。那么回到主题，RN 已经这么优秀了，为什么还要有 Flutter 的存在，*有一次向 Ab 哥请教技术成长的时候，Ab 哥提到了很有意思的一个观点，就是您对一项技术了解的深入程度，取决于是否能认清这项技术的局限。* 就像人一样，他（她）有多少优点，就会存在多少缺点。没发现，不等于不存在，因为一定存在。因此，顺着这个思路，我们简单的看一下 RN 的问题。

首先，看**维护成本**，虽然 RN 是一套代码多端运行。但还是需要 IOS 和 Android 开发帮助我们去一个一个的绘制组件，尤其遇到特殊诉求的时候，还要 case by case 的处理，并且随着 IOS 和 Android 系统本身的迭代和升级，以及框架自身发展的历史包袱，我们可能还需要处理很多与原生系统之间的平台差异，修复各种奇奇怪怪的 BUG，这对业务来说是很大的负担。

其次，对**性能诉求**，无论是产品还是开发同学，对于用户体验的追求，永远都不会停止。RN 存在诸多性能的短板，因此，才会有 Weex 这样的产品出现，去定制化的解决业务场景下的问题。JS 和 Native 的通信，页面的事件监听，复杂动画的渲染和交换成本，都是很大的性能挑战。

最后，在存在更强的业务诉求的时候，人们就不得不去寻找更好的方式去实现。非常存感激的看待谷歌这家公司，都是定位于商业公司，但实际上对世界的影响力上面，公司与公司之间差距还是非常大的。这个课题范围太大，以后有机会可以深度讨论一下。

您看到了上面的描述，为了解决上面这些问题 —— **自绘引擎时代**出现了，以 Flutter 为代表的技术方案会应运而生，相信一定不会只有 Flutter 一项跨平台技术出现的，历史总是惊人的相似。其实想到自绘引擎，我最先想到的是那些游戏引擎。那现在又为什么给出 **自绘引擎** 这样的一个概念呢？H5 是依赖于浏览器渲染，RN 依赖于客户端渲染，而 Flutter 基于 Skia 自己绘制的图形界面。因此，Flutter 才能真正实现跨端！相信在不久的未来，在传统客户端上也能看到 Flutter 的身影，这样才能真正达到多端统一。

最后，我们再简单总结一下有哪些问题：

1、Web 性能差，跟原生 App 存在肉眼可见的差距；

2、React Native 跟 Web 相比，支持的能力非常有限，特定长场景问题，需要三端团队一个一个处理；

3、Web 浏览器的安卓碎片化严重（感谢 X5，腾讯的同学过得相对轻松一些）。

为了解决上面的问题，Flutter 出现了：

**一套代码可以运行在两端；自绘 UI，脱离平台，也可以简单的把它理解为一个浏览器的子集。**

铺垫了这么多，就是为了帮助您回忆起技术发展的脉络和技术趋势，可以更好的理解下面即将要表达的文稿，下面我们正式开始介绍 Flutter。

### **2）Flutter 实现原理**

Flutter 能介绍的技术点其实非常多，这里找了一些具有代表性的技术项，结合自己的理解跟大家分享一下。包括设计思路、渲染方式、UI 的生命周期。因为这几个点，跟 React 技术栈风格非常相似，以这种思考结构去对比介绍，可以帮助大家更好的理解这项技术本身。

Flutter 整体架构设计

![](https://upload-images.jianshu.io/upload_images/19956127-09291d341def1443.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Google 了一下关键词，搜素得到了这张图片，从下向上进行一些描述：

**Embedder**：是操作系统适配层，实现了渲染 Surface 设置，线程设置，以及平台插件等平台相关特性的适配。从这里我们可以看到，Flutter 平台相关特性并不多，这就使得从框架层面保持跨端一致性的成本相对较低。

**Flutter Engine**：这是一个纯 C++实现的 SDK，其中囊括了 Skia 引擎、Dart 运行时、文字排版引擎等。不过说白了，它就是 Dart 的一个运行时，它可以以 JIT、JITSnapshot 或者 AOT 的模式运行 Dart 代码。在代码调用 dart:ui 库时，提供 dart:ui 库中 Native Binding 实现。 不过别忘了，这个运行时还控制着 VSync 信号的传递、GPU 数据的填充等，并且还负责把客户端的事件传递到运行时中的代码。具体的绘制方式，我们放在后面描述。

**Flutter Framework**：这是一个纯 Dart 实现的 SDK，类似于 React 在 JavaScript 中的作用。它实现了一套基础库， 用于处理动画、绘图和手势。并且基于绘图封装了一套 UI 组件库，然后根据 Material 和 Cupertino 两种视觉风格区分开来。这个纯 Dart 实现的 SDK 被封装为了一个叫作 dart:ui 的 Dart 库。我们在使用 Flutter 写 App 的时候，直接导入这个库即可使用组件等功能。

PS：虽然很早知道 Flutter，但实际写 Flutter 时间也比较短暂。引擎源码层面，目前也没有深入的涉猎。了解的方式可以通过自己阅读源码，或者找谷歌、阿里、美团、以及我司的开发者帮忙。从技术角度来了解这些，在需要的阶段，不会成为大家的瓶颈。**毕竟商业世界充满了壁垒，而应用层面的技术本身是开放的。**

### **3）Flutter 应用层语言 Dart**

**简单描述一下 JIT 与 AOT**：

*   JIT
    在运行时即时编译，在开发周期中使用，可以动态下发和执行代码，开发测试效率高，但运行速度和执行性能则会因为运行时即时编译受到影响。
*   AOT
    即提前编译，可以生成被直接执行的二进制代码，运行速度快、执行性能表现好，但每次执行前都需要提前编译，开发测试效率低。

**Dart 是什么**

它的目标在于成为下一代结构化 Web 开发语言。Dart 发布于 2011 年 10 月 Google 的"GOTO 国际软件开发大会"。是一种基于类编程语言（class-based programminglanguage），在所有浏览器都能够有高性能的运行效率。Chrome 浏览器内置了 DartVM，可以直接高效的运行 dart 代码（2015 年被移出）。支持 Dart 代码转成 Javascript，直接在 Javascript引擎上运行。[dart2js](https://link.zhihu.com/?target=https%3A//dart.dev/tools/dart2js)

**Dart 的特点**

*   开发时 JIT，提升开发效率；发布时 AOT，提升性能。
*   不会面对 JS 与 Native 之间交互的问题了。
*   Dart 的内存策略，采用多生代算法（与 Node 有一些类似）。
*   线程模型依旧是单线程 Event Loop 模型，通过 isolate 进行隔离，可以降低开发难度（与 Node 也非常类似）。
*   Dart 的生态，这个跟 Node.js 差距十分明显，npm 还是行业中最活跃的。
*   而静态语法与排版方式，纯前端入门还是有一定成本。

备注：

*   1）TS 可以一定程度上帮助 JS 添加一些静态检测，但本质上依旧是无法达成这样的效果；
*   2) 关于入门成本这个问题，如果您想深入，我相信这都不会成为问题。关键看是否能为业务和团队带来价值。

**Flutter 选择 Dart 的原因**

*   健全的类型系统，同时支持静态类型检查和运行时类型检查。
*   代码体积优化（TreeShaking），编译时只保留运行时需要调用的代码（不允许反射这样的隐式引用），所以庞大的 Widgets 库不会造成发布体积过大。
*   丰富的底层库，Dart 自身提供了非常多的库。多生代无锁垃圾回收器，专门为 UI 框架中常见的大量 Widgets 对象创建和销毁优化。
*   跨平台，iOS 和 Android 共用一套代码。
*   JIT & AOT 运行模式，支持开发时的快速迭代和正式发布后最大程度发挥硬件性能。
*   Native Binding。在 Android 上，v8 的 Native Binding 可以很好地实现，但是 iOS 上的 JavaScriptCore 不可以，所以如果使用 JavaScript，Flutter 基础框架的代码模式就很难统一了。而 Dart 的 Native Binding 可以很好地通过 Dart Lib 实现。

### **4）Flutter 实现思路**

看到了上面的介绍，这里总结一下 Flutter 的实现思路。它开辟了新的设计理念，实现了真正的跨平台的方案，自研 UI 框架，它的渲染引擎是 Skia 图形库来实现的，而开发语言选择了同时支持 JIT 和 AOT 的 Dart。不仅保证了开发效率，同时也提升了执行效率。由于 Flutter 自绘 UI 的实现方式，因此也尽可能的减少了不同平台之间的差异。也保持和原生应用一样的高性能。因此，**Flutter 也是跨平台开发方案中最灵活和彻底的那个，它重写了底层渲染逻辑和上层开发语言的一整套完整解决方案**。

*   彻底跨端：
*   Flutter 构建了一整套包括底层渲染、顶层设计的全套开发套件。
*   这样不仅可以保证视图渲染在 Android 和 IOS 上面的高度一致，也可以保证渲染和交互性能（媲美原生应用）。
*   与现有方案核心区别：
*   类 RN 方案，JS 开发，Native 渲染。数据通信 bridge；
*   Hybird 浏览器渲染 + 原生组件绘制；
*   Flutter 设计自闭环，完成渲染和数据通信。

## **3、Flutter 的 UI 渲染方案**

渲染方案是 Flutter 目前独特的设计形态，就是由于渲染自闭环，才能真正跨平台。谈到 UI 渲染方案，作为前端开发，我们是绕不过现在如火如荼的三大框架的。为什么要谈类 React 方案呢？因为 Flutter 的设计方案，与 React 设计具有一样的思路。在渲染这里我们会谈及控件、渲染原理、以及生命周期。

Flutter 是如何进行页面渲染的呢？传统 Web 是通过浏览器，而 Flutter 是自绘。所谓自绘就是用户界面上 Flutter 自己绘制到界面，无需依赖 IOS 和 Android 原生能力，是通过一个叫做 Skia 引擎进行页面绘图。

### **1）介绍一下 Skia**

Skia 是一个 2D 的绘图引擎库，其前身是一个向量绘图软件，Chrome 和 Android 均采用 Skia 作为绘图引擎。Skia 提供了非常友好的 API，并且在图形转换、文字渲染、位图渲染方面都提供了友好、高效的表现。Skia 是跨平台的，所以可以被嵌入到 Flutter 的 iOS SDK 中，而不用去研究 iOS 闭源的 CoreGraphics / Core Animation。

Skia 是用 C++ 开发的、性能彪悍的 2D 图像绘制引擎，其前身是一个向量绘图软件。Skia 在图形转换、文字渲染、位图渲染方面都表现卓越，并提供了开发者友好的 API。Android 自带了 Skia，所以 Flutter Android SDK 要比 iOS SDK 小很多。正是得益于 Skia 的存在：

*   Flutter 底层的渲染能力得到了统一，不在需要使用做双端适配；
*   通过 OpenGL、GPU，不需要依赖原生的组件渲染框架。
*   Flutter 可以最大限度的抹平平台差异，提升渲染效率和性能。

### **2）Flutter 的渲染流程**

用户可以看到一张图像展示，至少需要三类介质：CPU、GPU 和 显示器。CPU 负责图像的数据计算，GPU 负责图像数据的渲染，而显示器是最终图片展示的载体。CPU 拿到需要上屏的数据做处理和加工，处理完成之后交给 GPU，GPU 在渲染之后将数据放入帧缓冲区，随后随着控制同步信号 (VSync)以周期性的频率，从缓冲区内读出数据，在显示器上进行图像呈现。而且操作系统就是一个无限循环的机制，不停的重复上面的操作，进行显示器的更新.

![](https://upload-images.jianshu.io/upload_images/19956127-94360566800b2ab9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Flutter 的渲染整体流程也是这样的， Dart 进行视图数据的合成，然后交给 Skia 引擎进行处理，处理之后再交给 GPU 进行数据合成，然后准备上屏。当一帧图像绘制完毕后准备绘制下一帧时，显示器会发出一个垂直同步信号（VSync），所以 60Hz 的屏幕就会一秒内发出 60 次这样的信号。

### **3）Flutter 绘制流程**

![](https://upload-images.jianshu.io/upload_images/19956127-ea906d63c91c25f3.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如上图所示，Flutter 渲染流程分为 7 个步骤：

首先是获取到用户的操作，然后你的应用会因此显示一些动画 ；

接着 Flutter 开始构建 Widget 对象。Widget 对象构建完成后进入渲染阶段，这个阶段主要包括三步：

*   布局元素：决定页面元素在屏幕上的位置和大小；
*   绘制阶段：将页面元素绘制成它们应有的样式；
*   合成阶段：按照绘制规则将之前两个步骤的产物组合在一起

最后的光栅化由 Engine 层来完成。

### **4）布局**

布局时 Flutter 深度优先遍历渲染对象树。数据流的传递方式是从上到下传递约束，从下到上传递大小。也就是说，父节点会将自己的约束传递给子节点，子节点根据接收到的约束来计算自己的大小，然后将自己的尺寸返回给父节点。整个过程中，位置信息由父节点来控制，子节点并不关心自己所在的位置，而父节点也不关心子节点具体长什么样子。

![](https://upload-images.jianshu.io/upload_images/19956127-b07f6993df5d0d63.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

为了防止因子节点发生变化而导致的整个控件树重绘，Flutter 加入了一个机制——RelayoutBoundary，在一些特定的情形下 Relayout Boundary 会被自动创建，不需要开发者手动添加。

边界：Flutter 使用边界标记需要重新布局和重新绘制的节点部分，这样就可以避免其他节点被污染或者触发重建。就是控件大小不会影响其他控件时，就没必要重新布局整个控件树。有了这个机制后，无论子树发生什么样的变化，处理范围都只在子树上。

![](https://upload-images.jianshu.io/upload_images/19956127-03706a165eabf7e7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

缓存：要提升性能表现，缓存也是少不了的。在 Flutter 中，几乎所有的 Element 都会具有一个 key，这个 key 是唯一的。当子树重建后，只会刷新 key 不同的部分，而节点数据的复用就是依靠 key 来从缓存中取得。

在确定每个空间的位置和大小之后，就进入绘制阶段。绘制节点的时候也是深度遍历绘制节点树，然后把不同的 RenderObject 绘制到不同的图层上。

### **5）绘制**

在布局完成之后，渲染对象树中的每个节点都有了明确的尺寸和位置。Flutter 会把所有的 Element 绘制到不同的图层上。与布局过程类似，绘制的过程也是深度优先遍历，先绘制父节点，然后绘制子节点。以下图为例：节点 1、节点 2、节点 3、4、5，最好绘制节点 6。

![](https://upload-images.jianshu.io/upload_images/19956127-7619c1d11fc56683.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如上图可以看到一种场景，就是比如视图可能会合并，导致 节点 2 的子节点 5 与它的兄弟节点 6 处于同一个图层，这样会导致当 节点 2 需要重绘的时候，与其无关的节点 6 也会被重绘，带来性能问题。

为了解决上面的问题，Flutter 提出了布局边界的机制 ——重绘边界(Repaint-Boundary)。在重绘边界内，Flutter 会强制切换新的图层，这样可以避免边界内外的互相影响，避免无关内容虽然处于同一个层级导致的不必要的重绘。

![](https://upload-images.jianshu.io/upload_images/19956127-2457eb10721f4345.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

重绘边界的一个典型场景就是 ScrollView。ScorllView 滚动的时候会刷新视图，从而触发内容重绘，而当滚动内容重绘时，一般情况下其它内容是不需要被重绘的。这个时候重绘边界就非常有价值了。

**这里简单理解，就是更精细化的对控件的更新，进行了小范围的控制。在时间复杂度和空间复杂度中进行权衡。**未来我们优化业务，大概率也会优化这里，找到自身业务的平衡点。

### **6）合成和渲染**

最上面已经展示了 Flutter 的 **7 层渲染流水线**（Renderingpipline）的图里。这里主要描述一下对合成和渲染的理解。渲染流水线是由垂直同步信号（Vsync）驱动的。这个概念很类似我们平时说的 FPS 的概念，每秒 60 帧，过低的频率会显得页面很卡。当每一次 Vsync 信号到来以后，Flutter 框架会按照图里的顺序执行一系列动作:**动画（Animate）、构建（Build）、布局（Layout）和绘制（Paint）**

最终生成一个场景（Scene）之后送往底层，由 GPU 绘制到屏幕上。

![](https://upload-images.jianshu.io/upload_images/19956127-c46bd5bfacc85069.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Flutter App 只有在状态发生变化的时候需要触发渲染流水线。当你的 App 无任何状态改变的时候，Flutter 是不需要重新渲染页面的。所以，Vsync 信号需要 Flutter App 去调度。比如，我们在 Widget 内使用了 setState 方法改变了控件的状态。

整个渲染流水线是运行在 UI 线程里的，以 Vsync 信号为驱动，在框架渲染完成之后会输出 Layer Tree。Layer Tree 被送入 Engine，Engine 会把 Layer Tree 调度到 GPU 线程，在 GPU 线程内合成（compsite）Layer Tree，然后由 Skia 2D 渲染引擎渲染后送入 GPU 显示。这里提到 Layer Tree 是因为我们即将要分析的渲染流水线绘制阶段最终输出就是这样的 LayerTree。所以绘制阶段并不是简单的调用 Paint 函数这么简单了，而是很多地方都涉及到 Layer Tree 的管理。

Flutter 只关心向 GPU 提供视图数据，GPU 的 VSync 信号同步到 UI 线程，UI 线程使用 Dart 来构建抽象的视图结构，这份数据结构在 GPU 线程进行图层合成，视图数据提供给 Skia 引擎渲染为 GPU 数据，这些数据通过 OpenGL 或者 Vulkan 提供给 GPU。

这里描述一下合成的概念，所谓合成就是因为我们绘制的页面结构复杂，如果直接交付给绘图引擎去进行图层渲染，可能会出现大量的渲染内容重绘，因此，需要先进性一次图层合成，就是说先把所有的图层根据大小、层级等规则计算出最终的显示效果，将相同的图层合并，简化渲染树，提升渲染效率。

Flutter 会将合成之后的数据，交给 Skia 进行页面二维图层的渲染。

## **4、Widget 控件的更新策略**

在这一个部分我们对比着 React 的设计方式对比着看一下 Flutter 的实现，在 React 中您可以看到三种很重要的名称。JSX、Virtual Dom、真实 Dom，而在 Flutter 中我们依然可以看到对应的三类抽象的数据结构分别是 Widget、Element 和 RenderObject，他们的功能与 React 内三个数据抽象有异曲同工之处。Flutter 绘制界面的基础是 Widget，也就是描述页面的最小模块。

**Flutter 的核心设计思想就是 "一切皆 Widget"：**

前端同学可以把 Widget 理解为 Web Component 的 **组件** 即可。 一种结构化数据的抽象，包含了组件的布局、渲染属性、事件响应信息等。

### **1）Widget 类似 React VM 的 F(x) = Y 中的 x 存在**

Flutter 中的 Widget 是完全不可变的！只要当视图发生变化，Flutter 就会重新创建一个新的 Widget 进行更新。即是 React 也是有一定的数据 Diff 的策略，而这里变更即创建的方式，会带来大量的销毁和重建的过程，是否非常消耗性能？

Widget 对标的是 标识 React 的虚拟 DOM 节点的 数据描述 JSX，不是真实渲染的页面 DOM。只是数据的抽象，不涉及视图渲染。并且 Widget 具有不可变性，也提升了 Widget 本身的复用性。因此并没大量的性能消耗，而 Dart 的作为静态语言的运行速度，也会有着超越 JS 的性能。

### **2）Element 是 Widget 的一个实例化对象**

Element 承载了视图构建的上下文数据，是连接结构化的配置信息到完成最终渲染的桥梁； Element 是一个可变的数据结构， 可以大致理解为 Virtual DOM。可以进行 diff 更新； 可以将真正需要修改的数据同步到 RenderObject 中。最大程度的降低渲染视图的修改，提升渲染效率。

### **3）RenderObject 负责视图渲染的对象**

Flutter 的渲染分为 4 个部分。布局、绘制、合成、渲染，其中 布局和绘制是在 RenderObject 中完成的。 Flutter 采用深度优的方式渲染对象树，确定树中的各个对象的位置和尺寸，并把它绘制到不同图层， 绘制完成之后交给
Skia 在 VSync 信号同步时从渲染树合成位图，然后交给 CPU 进而完成上屏。

### **4）Widget 同样分为有状态 和 无状态组件**

无状态控件 StatelessWidget 类似 React 的 PFC。 有状态控件 StatefulWidget 就是 React 的 组件。 如同 react 组件一样，使用有状态组件是有成本的。正确的评估你的需求，避免使用无意义的有状态组件。

这里比较大的区别，**是 Flutter 直接把 Widget 设计成为了一个不可变的！** 这也导致了技术方案的实现上存在了差异。

### **5）既然看到了 Widget，那一定会有生命周期的存在**

由于篇幅限制，这里就不在详细介绍了，简单描述一下，生命周期，让您有个影响。

**创建：**构造函数 --> initState --> didChangeDependencies --> build

**更新:** 主要由三个方法触发：setState、didChangeDependencies 和 didUpdateWidget。
**销毁:** 系统会调用 diactivate 和 dispose 这两个方法，来移除或销毁组件。

### **这里多了 APP 生命周期的概念，在传统 Web 我们相对关注较少**

从后台切入前台：paused -> inactive -> resumed；
从前台退到后台：resumed-> inactive-> paused；

## **5、学习路线**

![](https://upload-images.jianshu.io/upload_images/19956127-7fa56c9fb2d328d7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

粗略了整理了一下我最近这 2 周体验开发过程中认知的学习范围，这上面除了跟 APP 相关的部分，大部分场景已经通过代码体验和实践过了。这里由于篇幅限制，就不在一一的介绍了。最重要的是关注 Flutter 的官方文档。

其次，找了三个实例，跟您分享，您可以 clone 下来，更细节的体验一下：

布局案例：[https://github.com/yang7229693/flutter-study](https://link.zhihu.com/?target=https%3A//github.com/yang7229693/flutter-study)
代码实例： [https://github.com/nisrulz/flutter-examples](https://link.zhihu.com/?target=https%3A//github.com/nisrulz/flutter-examples)
FlutterDemo: [https://github.com/OpenFlutter/Flutter-Notebook](https://link.zhihu.com/?target=https%3A//github.com/OpenFlutter/Flutter-Notebook)

除了上面列出的这些，还有很多要做，比如 运维、调试、自动化测试、兼容性、客户端 SDK 封装、国际化等等。当然对于开发者来说，工程化、调试、组件化都是非常重要的实践内容。

## **6、引文**

*   [Flutter 官网](https://link.zhihu.com/?target=http%3A//km.oa.com/articles/show/%255Bhttps%253A/flutter.dev%255D%28https%3A/flutter.dev/%29)
*   **[Flutter 中文网](https://link.zhihu.com/?target=https%3A//flutterchina.club/)**
*   **[Flutter 实战](https://link.zhihu.com/?target=http%3A//km.oa.com/articles/show/%255Bhttps%253A/book.flutterchina.club%255D%28https%3A/book.flutterchina.club/%29)**
*   **[闲鱼技术博文](https://link.zhihu.com/?target=https%3A//www.yuque.com/xytech/flutter)**

作者： haigecao，腾讯 CSIG Web 开发工程
# 最后

题外话，我在一线互联网企业工作十余年里，指导过不少同行后辈。帮助很多人得到了学习和成长。

我意识到有很多经验和知识值得分享给大家，也可以通过我们的能力和经验解答大家在IT学习中的很多困惑，所以在工作繁忙的情况下还是坚持各种整理和分享。但苦于知识传播途径有限，很多程序员朋友无法获得正确的资料得到学习提升，故此将并将重要的Android进阶资料包括自定义view、性能优化、MVC与MVP与MVVM三大框架的区别、NDK技术、阿里面试题精编汇总、常见源码分析等录播视频免费分享出来。

【[Android学习PDF+学习视频+面试文档+知识点笔记](https://links.jianshu.com/go?to=https%3A%2F%2Fshimo.im%2Fdocs%2Fw6cyqyXqKRPDGcrr)】

**【Android思维脑图（技能树）】**

知识不体系？这里还有整理出来的Android进阶学习的思维脑图，给大家参考一个方向。

![](https://upload-images.jianshu.io/upload_images/1095900-4bc8337d3615bf27.png?imageMogr2/auto-orient/strip|imageView2/2/w/530/format/webp)


**【Android进阶学习视频】、【全套Android面试秘籍】可以简信我【学习】查看免费领取方式！**


希望我能够用我的力量帮助更多迷茫、困惑的朋友们，帮助大家在IT道路上学习和发展~
