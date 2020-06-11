
阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)

先附上几张截图

![](//upload-images.jianshu.io/upload_images/697635-38e14c479edde374.png?imageMogr2/auto-orient/strip|imageView2/2/w/1080/format/webp)

![](//upload-images.jianshu.io/upload_images/697635-0017b13b5937f788.png?imageMogr2/auto-orient/strip|imageView2/2/w/1080/format/webp)

没有设计，自己怎么想怎么弄的，调用了[干活集中营](http://gank.io/api)的api，这个号称客户端最多的网站，由于已经过了两个多月才来写这篇博客，由于flutter的更新，现在最新的是6.0dev版本，可能会影响部分功能的使用，但应该不会有很大的改动，我就开发的过程谈谈，flutter的一些优缺点。

如果有面向对象的编程经验，入门还是很简单的，并且，官网也很细心的为我们讲解了一些概念，因为我是做android的我看得比较多的是这一篇[flutter-for-android](https://flutter.io/flutter-for-android/),还有针对于[iOS devs](https://flutter.io/flutter-for-ios/)，[React Native devs](https://flutter.io/flutter-for-react-native/)，[Xamarin.Forms devs](https://flutter.io/flutter-for-xamarin-forms/)，可以看出google的野心不小，不过我认为Flutter会降低移动端的开发成本，Flutter UI相比android原生更细腻一些，还原度会更高一些，毕竟做过android开发就知道碎片化的问题有多麻烦。

> step by step

### 环境安装

文档：[https://flutter.io/get-started/install/](https://flutter.io/get-started/install/)，按照步骤一步步基本就没什么问题了，但需要注意的是有一行小字**Note:** *If you’re in China, please read [this wiki article](https://github.com/flutter/flutter/wiki/Using-Flutter-in-China) first*. 我自己安装过程还是挺顺利的，这里不做过多描述。

### Flutter Gallery编译

由于目前资料比较少，Flutter Gallery在Flutter工程目录下，可以说是比较齐全的资料，虽然有文档，哪有一个Demo来的爽，直接看效果，事半功倍。我在这个过程还是花费了一番功夫，整个过程不是很顺利，只是因为Flutter版本不匹配，这里我就说一个较快的方法

*   下载Flutter 这里下载最新版本[https://flutter.io/sdk-archive](https://flutter.io/sdk-archive)
*   解压Flutter到你的电脑，xxxx\flutter_windows_v0.x.x-dev\flutter\bin 配置到环境变量，这样就可以在任何地方使用flutter命令了
*   找到xx\flutter_windows_v0.x.x-dev\flutter\examples\flutter_gallery，这里包含了Flutter Gallery的源代码
*   直接运行 flutter build apk 命令，在xx\build\app\outputs\apk\release目录下可以找到编译好的apk
*   发送到android手机或者模拟器就可以看到运行好的Demo
    整个过程在命令行完成的，当前编译版本截图：

    ![](//upload-images.jianshu.io/upload_images/697635-c6bca5c8efc2a465.png?imageMogr2/auto-orient/strip|imageView2/2/w/1080/format/webp)

    因为同一目录下的Flutter Gallery版本是匹配的，所以比较顺利，我最开始是从github上直接下载master的源码，所以遇到了很多问题。

Flutter Gallery apk下载: [https://fir.im/ts78](https://fir.im/ts78)

### 开发工具

官网上提供了两种编辑器的插件Android Studio 和VS Code，这里我还是选择比较熟悉的Android Studio，安装插件的方式也很简单，[https://flutter.io/get-started/editor/](https://flutter.io/get-started/editor/)，和安装一般的插件是一样的，安装完成后重启，就能找打，新建flutter 工程的选项

![](//upload-images.jianshu.io/upload_images/697635-bf5f4d7cf73f1d40.png?imageMogr2/auto-orient/strip|imageView2/2/w/454/format/webp)

工程新建完成后，一个基础的工程就出来了，第一次新建工程有点慢，一次就成功了，直接运行到模拟器就ok了。

### 开发阶段

新建完成的Flutter工程包含了一些目录，其中比较显眼的就是android 和ios目录，但实际上我没在这两个目录下写代码（以后肯定会写），其实只要看了一些Demo大致就晓得，几乎所有的代码都在lib下面，依赖管理通过 pubspec.yaml，我并不想讲太多关于代码的事情，因为我在前面编译好了一个Demo（Flutter Gallery）我写代码的时候基本上是照葫芦画瓢，文档和Demo都有，那就慢慢研究吧，没有捷径可走。

### Flutter的优点

在开头的时候我介绍了一些，但都是比较官方的，下面是结合自己的开发体验

*   编译很快，hotReload果然名不虚传
*   扩展性很强，一切皆widget，可以轻松实现一些复杂的效果
*   动画很赞，Hero动画可以很轻松的实现过度动画，其他动画api也非常灵活可配置
*   换主题很赞，可快速全局切换主题
*   调试模式很赞，Flutter Gallery就可以进行性能分析，slow motion（慢动作）等等，

### Flutter的缺点

Flutter还处于Beta版本，肯定是有些原因的

*   开源库太少，尽管[https://pub.dartlang.org/](https://pub.dartlang.org/)已经提供了大量的插件，但相对于其他语言来说，远远不够，资料还是太少了
*   一些控件在debug模式和release模式下表现有些细微的差异
*   json解析太麻烦，因为习惯了使用GsonFormat之类的插件，不过这应该不是太大的问题，有时间，也可以尝试着写一个
*   gif循环播放造成的oom问题，我查阅了issue，好像是skia图像引擎的问题，其实加载大量图片也容易出现oom，没有出现像Picasso和Glide之类的图像框架

以上仅仅是在开发的过程中碰到的

### Flutter_Gank 项目介绍

终于说了一点和标题相关的了，不然有人要说我标题党了，其实做完之后感觉也没什么特色，就列一些用到了哪些知识点吧

*   网络请求插件：[https://github.com/flutterchina/dio](https://github.com/flutterchina/dio)
*   图片加载插件：[https://github.com/renefloor/flutter_cached_network_image](https://github.com/renefloor/flutter_cached_network_image)
*   瀑布流插件：[https://github.com/letsar/flutter_staggered_grid_view](https://github.com/letsar/flutter_staggered_grid_view)
*   共享参数插件：[https://github.com/flutter/plugins/tree/master/packages/shared_preferences](https://github.com/flutter/plugins/tree/master/packages/shared_preferences)
*   网络状态插件：[https://github.com/flutter/plugins/tree/master/packages/connectivity](https://github.com/flutter/plugins/tree/master/packages/connectivity)
*   跳转外部浏览器插件：[https://github.com/flutter/plugins/tree/master/packages/url_launcher](https://github.com/flutter/plugins/tree/master/packages/url_launcher)
    还有一些基础控件ListView，TabBar，TabBarView，Button，Text，TextField，SnakeBar……的使用

开过过程中还遇到了一些小坑，比如，

*   侧滑抽屉Drawer可以通过 _globalKey.currentState.openDrawer()打开却没有提供Close，或者Hide之类的方法关闭，需要通过Navigator.pop(context)来隐藏菜单 。
*   做双击退出的时候调用Navigator.pop(context)会出现黑屏，需要调用SystemNavigator.pop()才能完全退出。

作者：dongjunkun
链接：https://www.jianshu.com/p/1031f30dbb2e
阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)
