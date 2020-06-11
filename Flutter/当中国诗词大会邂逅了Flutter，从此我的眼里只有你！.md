### 前言

Flutter是Google推出的跨平台的解决方案，Slogan是“Design beautiful apps”，国内也有知名企业在使用和推广，例如阿里、美团都有在尝试。

个人对其中的一些特性，比如JIT、Material Design、快速开发等很感兴趣，于是决定尝试一下。

### 诗词汇

于是诞生了诗词汇APP，首先看一下是个什么样的APP。

![](https://upload-images.jianshu.io/upload_images/15038736-01c04a047b8dacdc?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来我们一步步从不同方面说说Flutter的开发。

### 开始

FLutter可以在Windows、Linux、Mac上进行开发，开发工具可以使用VS Code、Android Studio、IDEA等，本文推荐使用Android Studio，主要在于Android Studio提供了FLutter Inspector工具，可以实时审查元素，解决界面的显示适配问题。

#### 搭建开发环境

搭建环境的主要步骤：

1.  下载SDK，[下载地址](https://flutter.io/docs/get-started/install)。

2.  配置PATH，如果使用Mac或者Linux系统，一定要将`bin`目录添加到系统PATH。

3.  配置依赖源镜像，这一步很重要，并且需要将脚本放到启动shell中。

```

export PUB_HOSTED_URL=https://pub.flutter-io.cn

export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn

```

1.  执行`flutter doctor`，这一步耗时会很长，需要耐心等耐。

2.  安卓开发工具及插件，[配置编辑器](https://flutter.io/docs/get-started/editor)。

#### 配置编辑器

主要是给编辑器安装相应的插件。

VS Code安装[flutter插件](https://marketplace.visualstudio.com/items?itemName=Dart-Code.flutter)，Android Studio和IDEA需要安装Flutter和Darter插件。

其中Android Studio和IDEA基本一样，跟VS Code的主要区别在于：

1.  VS Code提供了更好的代码提示功能

2.  IDEA提供了Flutter Inspector，可实时审查页面元素

可根据个人喜好、习惯选择使用。

#### 推荐网站

在安装、配置过程中，可参考以下中文资料：

[Flutter中国](https://flutter-io.cn/)

[Flutter中文文档](http://doc.flutter-dev.cn/docs/)

### 主要技术点

#### Dart

Flutter项目的开发语言是Dart，Dart 是由 Google 开发的一种面向对象语言，可以编译成 ARM 和 x86 代码直接运行在 iOS、Android 设备上。

推荐先学习Dart语言[官方教程](https://www.dartlang.org/guides/language/language-tour)，对Dart有初步了解之后再进行Flutter的学习和开发。

#### 界面开发

终于可以进入Flutter本身了。

##### Widget

Flutter中页面所有元素都是Widget，又分为StatelessWidget和StatefulWidget。

顾名思义，StatelessWidget 就是指无可变状态的 Widget，这类 Widget 的状态只由创建 Widget 时传入的参数决定，一旦创建，其状态、在页面上的展示效果也就不再改变。

而 StatefulWidget 内部则存在着可变状态。当通过setState改变这些状态时，Flutter 会重新渲染该 Widget。

##### 布局

在实际开发中，主要使用了Row、Column、Container、Expanded、Stack等。

Row、Column提供了水平、垂直方向的布局，Stack提供了堆叠方式的布局，各种容器有不同的特性，可根据实际页面需求选择搭配不同的布局。

推荐学习 [官方文档](https://flutter.io/docs) 及 [国内维护的中文翻译](https://flutterchina.club/docs/)。

#### 主要插件

话题切回到诗词汇APP，本APP收集了4000余位诗人的30多万首诗词，提供了古诗词的查询、收藏、朗诵功能，并且实现了初步的社区功能。

项目目录结构如下：

![image](https://upload-images.jianshu.io/upload_images/15038736-41bee798c8cfa7a9?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

开发这个APP大概用了一个月的业余时间，每天抽出一两个小时，这样折算为工作日，大概是两个星期左右，开发效率还是很高的。

下面跟大家分享一下主要功能及所使用的一些插件。

##### 切换主题

为了实现实时切换主题颜色，使用了状态管理插件。

[flutter_redux](https://pub.flutter-io.cn/packages/flutter_redux) 。

##### 极光推送

在国内厂商中，极光是少有的对Flutter提供了技术支持的，这里给极光大大的👍。

[jpush_flutter](https://pub.flutter-io.cn/packages/jpush_flutter)

##### QQ

QQ的Flutter插件提供了基本的登录、分享功能。

[flutter_qq](https://pub.flutter-io.cn/packages/flutter_qq)

##### 微信

微信的Flutter插件提供的功能稍微丰富，包含了支付、登录、分享、启动小程序的功能。

[fluwx](https://pub.flutter-io.cn/packages/fluwx)

##### 事件总线Event Bus

大名鼎鼎的event_bus也提供了对Flutter的支持。

[event_bus](https://pub.flutter-io.cn/packages/event_bus)

##### 音频

录音及播放音频也有很好的支持。

[audio_recorder](https://pub.flutter-io.cn/packages/audio_recorder)

[audioplayer2](https://pub.flutter-io.cn/packages/audioplayer2)

##### 其它

其它诸如加载HTML、Toast提示、图片选择器、图片加载等也有较好的插件支持。

可在 [官方插件库](https://pub.flutter-io.cn/) 查询相关的插件。

### 坑

#### 安装、升级

FLutter的安装、升级会经常遇到卡死的问题，主要原因就是使用了Google的源，但是莫名的，即使使用了上网、设置了国内镜像后，也会遇到同样的问题。只能通过反复的`flutter doctor` 或 `flutter upgrade`直到解决问题。

#### 开发

由于笔者最近一段时间Android项目做得较多，习惯了Android的XML布局方式，对于在代码中编写页面的形式一开始还有些不习惯，但是在按照官方例子实践了几个页面后，用代码写页面的优势就体现出来了。

在页面已经设计好的情况下，开发的时候脑海中就构思出一个Widget树，从根节点到每一个节点一级一级嵌套下去，自然而然的布局就写好了。

#### Dialog弹出框

使用Dialog的时候，弹出Dialog的Context及Dialog本身都会压入栈中，所以让Dialog消失的方法是`Navigator.of(ctx).pop()`，这样的设计既不同于Android也不同于iOS，也许跟Flutter本身所有元素都是Widget的设计有关。

#### 编译

在编译Android版本的时候很顺畅，没有遇到任何问题。但是在编译iOS版本的时候，遇到了很多问题，直到现在也没有解决。

问题在于使用了`audio_recorder`和`flutter_qq`两个插件，而这两个插件一个要求编译选项需要设置`!use_framework`，一个要求不能设置，造成了冲突，在实际编译中一直编译不通过。

### 结语

开发结束，最终打包了Release版本的APK，安装到手机后，发现惊喜。

**竟然如丝般顺滑，这是我始料未及的，转场效果、页面相应速度不输原生APP。**

总而言之，个人对Flutter的前景相当看好，毕竟是Google大厂出品，并且项目本身的迭代速度很快，目前已经是0.11版本，期望在不远的将来发布正式的1.0版本，更期望国内厂商加大对Flutter的支持力度。

**顺便推广一下笔者的诗词汇APP，欢迎大家 [下载试用](https://android.myapp.com/myapp/detail.htm?apkName=com.liuzb.allpoems) ，或者访问 [诗词汇](https://52gushici.cn/) 体验下Flutter如丝般的顺滑。^_^**

**推荐阅读：[37岁老码农现身说法：那些年，我走过的弯路](https://www.jianshu.com/p/8bab31550431)**
**[疫情结束后，会影响程序员年后找工作吗？](https://www.jianshu.com/p/3a040c531456)**


原文作者：[Game_over](https://link.zhihu.com/?target=https%3A//home.cnblogs.com/u/game-over/)

原文链接：[两个星期，用Flutter撸个APP - Game_over - 博客园](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/game-over/p/9998392.html)

来源：博客园
