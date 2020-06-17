## 面向标准化开发已成现实

金三银四，相信有不少读者在抓紧机会面试。

Android 市场已今非昔比。在过去，迫于招人的压力，应试者只需了解四大组件、视图、网络请求，即可谋得一份满意的工作。

现如今，Jetpack 架构组件 及 标准化开发模式 的确立，意味着 Android 开发已步入成熟阶段：

> 许多 样板代码 不再需要开发者手写，而是可以通过模版工具 自动生成，在取缔繁杂耗时的重复工作的同时，**避免因人工操作的疏忽，而造成难以排查、不可预期的错误**。

这十分符合企业的利益，因而面试官在招人的时候，也更加看重应试者对 架构组件 —— 至少是 MVVM 的理解程度。🧐

像“解耦”等 含糊其辞的说法，已经不能够被面试官所认可，稍微对 MVVM 有一点经验的面试官都会请你举例说明，好证明你确实对 MVVM 有着正确、深入的理解，能够自然而然地写出标准化、规范化的代码，能够迅速适应 各家公司自制的 自动化模版工具。

## 本文的目标

本人拥有 3 年的 移动端架构 践行和设计经验，领导团队重构的中大型项目多达十数个，对 Jetpack MVVM 架构在确立规范化、标准化 开发模式 以减少不可预期的错误 所作的努力，有着深入的理解。

因而本文的目标，就是结合前几期我们分别 深入浅出 介绍过的 Lifecycle、LiveData、ViewModel、DataBinding，来融汇贯通地演绎一下：

**作为 应用开发骨架 的 标准化状态管理框架**，究竟为 快速开发过程中 减少不可预期的错误 做了哪些努力。

不同于 东拼西凑、人云亦云、徒添困扰 的网文，愿意将 标准化开发模式的 **深度思考知识** 和 **实战反思经验** 无保留地分享，全网仅此一家。**这样的文章可以说是 看一篇、少一篇**，因此，就算不去 hold 住面试官，也请务必跟随本文的脚步，无障碍地将 Jetpack MVVM 过一遍！😉

## 文章目录一览

*   前言
*   面向标准化开发已成现实
*   本文的目标
*   Jetpack Lifecycle
    *   Lifecycle 存在前的混沌世界
    *   Lifecycle 为什么能解决上述这些问题？
*   Jetpack LiveData
    *   LiveData 存在前的混沌世界
    *   LiveData 为什么能解决上述这些问题？
    *   **LiveData 有个坑需要注意**
*   Jetpack ViewModel
    *   ViewModel 存在前的混沌世界
    *   ViewModel 为什么能做到这几点？
*   Jetpack DataBinding
    *   DataBinding 存在前的混沌世界
    *   DataBinding 就是来解决这些问题
*   综上

## Jetpack Lifecycle

> **Lifecycle 的存在，主要是为了解决 生命周期管理 的一致性问题**

### Lifecycle 存在前的混沌世界

在 Lifecycle 面市前，生命周期管理 纯靠手工维持，这样就容易滋生大量的一致性问题。

例如跨页面共享的 GpsManager 组件，在每个依赖它的 Activity 的 onResume 和 onPause 中都需要 **手工 激活、解绑 和 叫停**。

那么 **随着 Activity 的增多，这种手工操作 埋下的一致性隐患 就会指数级增长**：

> 一方面，凡是手工维持的，开发者容易疏忽，特别是工作交接给其他同事时，同事并不能及时注意到这些细节。

> 另一方面，分散的代码不利于修改，日后除了激活、叫停，若有其他操作需要补充（例如状态监听），那么每个 Activity 都需要额外书写一遍。

![](https://upload-images.jianshu.io/upload_images/19956127-5075843a635d262e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Lifecycle 为什么能解决上述这些问题？

Lifecycle 通过 模板方法模式 和 观察者模式，将生命周期管理的复杂操作，全部在作为 LifecycleOwner 的基类中（例如视图控制器的基类）封装好，默默地在背后为开发者运筹帷幄，

开发者因而得以在视图控制器（子类）中只需一句 `getLifecycle().addObserver(GpsManager.getInstance)` ，优雅地完成 第三方组件在自己内部 对 LifecycleOwner 生命周期的感知。

![](https://upload-images.jianshu.io/upload_images/19956127-3eb0af6d6e6c2bdf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


除了解决一致性问题，这样做还 **顺带地提供了其他 2 个好处**：

> **1.规避 为监听状态 而 注入视图控制器 的做法**

当需要监听状态时，以往我们的做法是 通过方法手工注入 Activity 等参数，这埋下了内存泄漏的隐患 —— 因为团队中的新手容易因这是个 Activity，而在日后误将其依赖给组件中的其他成员。

现如今，我们可以直接在组件内部 点到为止 地监听 LifecycleOwner 的状态，从而规避这种不恰当的使用。

> **2.规避 为追溯事故来源 而 注入视图控制器 的做法**

当发生事故时，以往我们若想在组件中 **追溯事故来源**，同样不得不从方法中直接注入 Activity 等，这同样埋下了内存泄漏的隐患。现如今组件因实现了 DefaultLifecycleObserver，而得以通过生命周期回调方法中的 LifecycleOwner 参数，**在方法作用域中 即可得知事故来源**，无需更多带有隐患的操作。

> 如果这么说还不理解的话，可具体参考我在 [《为你还原一个真实的 Jetpack Lifecycle》](https://xiaozhuanlan.com/topic/3684721950) 中提供的 GpsManager 案例，本文不再累述。

## Jetpack LiveData

> LiveData 的存在，主要是为了帮助 **新手老手 都能不假思索地遵循 通过唯一可信源分发状态 的标准化开发理念**，从而使在快速开发过程中 难以追溯、难以排查、不可预期 的问题所发生的概率降低到最小。

### LiveData 存在前的混沌世界

在 LiveData 面市前，我们分发状态，多是通过 EventBus 或 Java Interface 来完成的。不管你是用于网络请求回调的情况，还是跨页面通信的情况。

那这造成了什么问题呢？首先，EventBus 只是纯粹的 Bus，它 **缺乏上述提到的 标准化开发理念 的约束，那么人们在使用这个框架时，容易因 去中心化 地滥用，而造成 诸如 毫无防备地收到 预期外的 不明来源的推送、拿到过时的数据 及 事件源追溯复杂度 为 n² 的局面**。

并且，**EventBus 本身缺乏 Lifecycle 的加持，存在生命周期管理的一致性问题。这是 EventBus 的硬伤**，也是我拒绝使用 EventBus 的最主要因素。

> 对上述状况不理解的，可具体参考我在 [《LiveData 鲜为人知的 身世背景 和 独特使命》](https://xiaozhuanlan.com/topic/0168753249) 中提供的 播放器状态全局通知 的案例

### LiveData 为什么能解决上述这些问题？

首先，**LiveData 是在 Google 希望确立 标准化、规范化 的开发模式 —— 这样一种背景下诞生的**，因而为了达成这个艰巨的 **使命**，Google 十分克制地将其设计为，仅支持状态的输入和监听，从而，它不得不 **在单例的配合下，承上启下地完成 状态 从 唯一可信源 到 视图控制器 的输送**。

> （ViewModel 姑且也算是一种单例，一种工厂模式实现的伪单例。**唯一可信源是指 生命周期独立于 视图控制器的 数据组件**，通常是 单例 或共享 ViewModel）

**这使得任何一次状态推送，都可预期、都能方便地追溯来源，而不至于在 事件追溯复杂度为 n² 的迷宫中白费时间。**（即，无论是从哪个视图控制器发起的 对某个共享状态改变的请求，状态最终的改变 都由 作为唯一可信源的 单例或 SharedViewModel 来一对多地通知改变）

![](https://upload-images.jianshu.io/upload_images/19956127-6df80ab32755a288.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


并且，这种承上启下的方式，使得单向依赖成为可能：单例无需通过 Java Interface 回调通知视图控制器，从而规避了视图控制器 被生命周期更长的单例 依赖 所埋下的内存泄漏的隐患。

### LiveData 有个坑需要注意

不过，LiveData 的设计有个坑，这里我顺带提一下。

为了在视图控制器发生重建后，能够 自动倒灌 所观察的 LiveData 的最后一次数据，**LiveData 被设计为粘性事件**。

> —— 我姑且认为这是个拓展性不佳的设计，甚至可以说是一个 bug，

因为 MVVM 是一个整体，既然 **ViewModel 支持共享作用域，并且官方文档都承认了通过 共享 ViewModel 来实现跨页面通信的需求**，

那么基于 “开闭原则”，LiveData 理应提供一个与 MutableLiveData 平级的底层支持，专门用于非粘性的事件通信的情况，否则直接在跨页面通信中使用 MutableLiveData **必造成 事件回调的一致性问题 及 难以预期的错误**。

关于非粘性 LiveData 的实现，网上存在通过 “事件包装类”（只适合 kotlin 的情况） 和 “反射干预 LastVersion” （适用于 Java 的情况）两种方式来解决：

> [juejin.im/post/5b2b1b…](https://juejin.im/post/5b2b1b2cf265da5952314b63)

> [blog.csdn.net/geyuecang/a…](https://blog.csdn.net/geyuecang/article/details/89028283)

无论是使用哪一种实现，我都建议 遵循传统 LiveData 所遵循的开发理念，通过唯一可信源分发状态，来方便事件源头的追溯。对于 “去中心化” 的 Bus 方式，我拒绝在项目中这样使用。

> （具体我会在未来开源的最佳实践项目中 展示 UnPeekLiveData 的使用）

## Jetpack ViewModel

> ViewModel 的存在，主要是为了解决 状态管理 和 页面通信 的问题。

### ViewModel 存在前的混沌世界

ViewModel 的本职工作是 **状态托管** 和 **状态管理的分治**，也即当视图控制器重建时，

> 对于轻量的状态，可以通过视图控制器基类的 saveInstanceState 机制，以序列化的方式完成存储和恢复。

> 对于重量级的状态，例如通过网络请求得到的 List，可以通过生命周期长于视图控制器的 ViewModel 持有，从而得以直接从 ViewModel 恢复，而不是以效率较低的序列化方式。

在 Jetpack ViewModel 面市之前，MVP 的 Presenter 和 MVVM - Clean 的 ViewModel 都不具备状态管理分治的能力。

Presenter 和 Clean ViewModel 的生命周期都与视图控制器同生共死，因而它们顶多是为 DataBinding 提供状态的托管，而无法实现状态的分治。

到了 Jetpack 这一版，ViewModel 以精妙的设计，达成了状态管理，以及可共享的作用域。

### ViewModel 为什么能做到这几点？

其实这版主要是基于 **工厂模式**，使得 ViewModel **被 LifecycleOwner 所持有、通过 ViewModelProvider 来引用**，

所以 **它既类似于单例：** —— 当被作为 LifecycleOwner 的 Activity 持有时，能够脱离 Activity 旗下 Fragment 的生命周期，从而实现作用域共享，

**实际上又不是单例：** —— 生命周期跟随 作为 LifecycleOwner 的视图控制器，当 Owner（Activity 或 Fragment）被销毁时，它也被 clear。

> 此外，出于对视图控制器重建的考虑，Google 在视图控制器基类中通过 retain 机制对 ViewModel 进行了保留。

> 因此，对于 作用域共享 和 视图重建 的情况，状态因完好地被保留，而得以被视图控制器在恢复时直接使用。

再者，由于存在 共享作用域的考虑，所以 ViewModel 本身也承担了跨页面通信（例如事件回调）的职责。前面在介绍 LiveData 时，对于 LiveData 在事件通信时粘性设计的问题已经介绍过了，这里不再累述。

> 截至 2020.2.1，ViewModel 在 Fragment 中的 retain 设计已发生剧变，具体缘由可参考我在 [《有了 Jetpack ViewModel . . . 真的可以为所欲为！》](https://xiaozhuanlan.com/topic/6257931840) 文末及评论区的最新补充。

## Jetpack DataBinding

> DataBinding 的存在，主要是为了解决 视图调用 的一致性问题。

### DataBinding 存在前的混沌世界

在 DataBinding 面市前，我们若要改变视图的状态，首先就要引用该视图，例如 textView.setText()，

这造成什么问题呢？

> 当页面存在横、竖布局，且两种布局的控件存在差异，例如横屏存在 textView 控件，而竖屏没有，那么我们就不得不在视图控制器中为 textView 做判空处理，这就造成了一致性问题 —— 容易疏忽而忘记判空，毕竟页面多达数十个、每个页面的控件也无数。

那怎么办呢？

### DataBinding 就是来解决这些问题

通过在布局中与可观察的数据发生绑定，那么当该数据被 set 新的内容时，控件也将得到通知和刷新。

换言之，在使用 DataBinding 后，唯一的改变是，你无需手工调用视图来 set 新状态，你只需 set 数据本身。

因而，**DataBinding 并非许多人不假思索认为的，将 UI 逻辑搬到 XML 中写 从而难以调试 —— 事实根本不是这样的：**

**DataBinding 只负责绑定数据、负责作为 UI 逻辑末端的状态的改变**（也即它是一个不可再分的原子操作，本来就不需要调试），原本在视图控制器中 UI 逻辑怎么写，现在还是怎么写，只不过不再需要 textView.setText(xxx)，而是直接 xxx.set()。

所以在 DataBinding 的帮助下，好处总共有多少个呢？

> 1.规避了视图状态的 一致性问题 —— 无需手工判空。

> 2.规避了视图状态的 一致性问题，乃至无需视图调用，从而完全不用编写 findViewById。

> 3.就算要调用视图，也不用 findViewById，而是直接通过 binding 来引用。

> 4.先前的 UI 逻辑基本不用改动，改的只是作为末端的状态改变的方式。

……

此外，**DataBinding 有个大杀器就是，能为控件提供自定义属性的 BindingAdapter**，它不仅可以解决 圆角 Drawable 复用的问题（你懂得），还可以实现 imageView 直接绑定 url 等需求，总之，没有它办不到的，只有你想不到的，DataBinding 的好处等着你挖掘。😉

关于 DataBinding 的注意事项，以及屡试不爽的排坑技巧，可具体参考 [《从 被误解 到 真香 的 Jetpack DataBinding！》](https://xiaozhuanlan.com/topic/9816742350)，这里不做累述。

## 综上

Lifecycle 的存在，主要是为了解决 **生命周期管理 的一致性问题**。

LiveData 的存在，主要是为了帮助 新手老手 都能不假思索地 **遵循 通过唯一可信源分发状态 的标准化开发理念**，从而在快速开发过程中 规避一系列 **难以追溯、难以排查、不可预期** 的问题。

ViewModel 的存在，主要是为了解决 **状态管理 和 页面通信 的问题**。

DataBinding 的存在，主要是为了解决 **视图调用 的一致性问题**。

它们的存在 大都是为了 在软件工程的背景下 解决一致性的问题、将容易出错的操作在后台封装好，**方便使用者快速、稳定、不产生预期外错误地编码**。

这样说，你理解了吗？😉

[GitHub : Jetpack-MVVM-Best-Practice](https://github.com/KunMinX/Jetpack-MVVM-Best-Practice)

作者：KunMinX
链接：https://juejin.im/post/5dafc49b6fb9a04e17209922

