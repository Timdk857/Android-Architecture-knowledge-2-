阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)

## Flutter页面-基础Widget

在Flutter中，几乎所有的对象都是一个 Widget，与原生开发中的控件不同的是，Flutter中的 widget的概念更广泛，它不仅可以表示UI元素，也可以表示一些功能性的组件如：用于手势检测的 GestureDetector widget、用于应用主题数据传递的 Theme等等。由于Flutter主要就是用于构建用户界面的，所以，在大多数时候，可以认为widget就是一个控件，不必纠结于概念。

Widget的功能是“描述一个UI元素的配置数据”，Widget其实并不是表示最终绘制在设备屏幕上的显示元素，而只是显示元素的一个配置数据。实际上，Flutter中真正代表屏幕上显示元素的类是 Element，也就是说Widget只是描述 Element的一个配置。一个Widget可以对应多个 Element，这是因为同一个Widget对象可以被添加到UI树的不同部分，而真正渲染时，UI树的每一个节点都会对应一个 Element对象。
## Widget

StatelessWidget和 StatefulWidget是 flutter的基础组件，日常开发中自定义 Widget都是选择继承这两者之一。也是在往后的开放中，我们最多接触的Widget：
StatelessWidget：无状态的，展示信息，面向那些始终不变的UI控件；
StatefulWidget：有状态的，可以通过改变状态使得 UI 发生变化，可以包含用户交互(比如弹出一个 dialog)。

在实际使用中，Stateless与Stateful的选择需要取决于这个 Widget 是有状态还是无状态，简单来说看界面是否需要更新。
## StatelessWidget

StatelessWidget用于不需要维护状态的场景，它通常在 build方法中通过嵌套其它Widget来构建UI，在构建过程中会递归的构建其嵌套的Widget。
BuildContext表示构建widget的上下文，它是操作widget在树中位置的一个句柄，它包含了一些查找、遍历当前Widget树的一些方法。每一个widget都有一个自己的context对象。
```
import'package:flutter/material.dart';

void main()=> runApp(StatelessApp());

class StatelessApp extends StatelessWidget{
  
///在build方法中通过嵌套其它Widget来构建UI，在构建过程中会递归的构建其嵌套的Widget
  
@override
  
Widget build(BuildContext context){
    
//嵌套 MaterialApp：封装了应用程序实现Material Design所需要的一些widget
    
returnMaterialApp(title:"Widget演示",//标题,显示在recent时候的标题
        
//主页面
        
//Scaffold : Material Design布局结构的基本实现。
     home:Scaffold(
          //ToolBar/ActionBar
          appBar:AppBar(title:Text("Widget")),
          body:Text("Hello,Flutter!"),
        )
    );
  }
}
```
**Material Design:**

一种设计语言，Material Design 于2014年的 Google I/O 首次亮相，是谷歌推出的全新的设计语言。说白了，就是一种设计风格。
## StatefulWidget

StatefulWidget是动态的，添加了一个新的接口 createState()用于创建和Stateful widget相关的状态 State，它在Stateful widget的生命周期中可能会被多次调用。

当State被改变时，可以手动调用其 setState()方法通知Flutter framework状态发生改变，Flutter framework在收到消息后，会重新调用其 build方法重新构建widget树，从而达到更新UI的目的。
![](https://upload-images.jianshu.io/upload_images/19956127-90ff3f225953b2b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## State生命周期

State的生命周期为:
![](https://upload-images.jianshu.io/upload_images/19956127-831f7a1205708365.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
State类除了 build之外还提供了很多方法能够让我们重写，这些方法会在不同的状态下由Flutter调起执行，所以这些方法我们就称之为生命周期方法。在这里我们用statefulwidget点击按钮后移除子statefulwidget。
![](https://upload-images.jianshu.io/upload_images/19956127-0a35615833a5f122.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/19956127-b121c33e56c10bba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/19956127-92a9dd2ae44d100f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
执行的输出结果显示为:

运行到显示
![](https://upload-images.jianshu.io/upload_images/19956127-e045ee53e704daf7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击按钮会移除Child
![](https://upload-images.jianshu.io/upload_images/19956127-f6fbf5bbdcd8d851.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
将MyApp的代码由 child:isShowChild?Child():Text("演示移除Child")，改为 child:Child()，点击按钮时
![](https://upload-images.jianshu.io/upload_images/19956127-8d9ab846e81d86bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 基础widget

## 文本显示

**Text**

Text是展示单一格式的文本Widget(Android TextView)。
![](https://upload-images.jianshu.io/upload_images/19956127-3932a98f63457f26.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在使用 Text显示文字时候，可能需要对文字设置各种不同的样式，类似Android的 android:textColor/Size等

在Flutter中也拥有类似的属性
![](https://upload-images.jianshu.io/upload_images/19956127-613d51e2a229b182.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/19956127-664b39e058a722d2.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**RichText**

如果需要显示更为丰富样式的文本(比如一段文本中文字不同颜色),可以使用 RichText或者 Text.rich
![](https://upload-images.jianshu.io/upload_images/19956127-e3fe6c330cc17352.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/19956127-99c5841774bc6fdf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**DefaultTextStyle**

在widget树中，文本的样式默认是可以被继承的，因此，如果在widget树的某一个节点处设置一个默认的文本样式，那么该节点的子树中所有文本都会默认使用这个样式。相当于在Android中定义 Theme
![](https://upload-images.jianshu.io/upload_images/19956127-d6158ecc777d6bba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**FlutterLogo**

这个Widget用于显示Flutter的logo......
![](https://upload-images.jianshu.io/upload_images/19956127-b007c955137fe517.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/19956127-773df6d91999c1ac.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**Icon**

主要用于显示内置图标的 Widget
![](https://upload-images.jianshu.io/upload_images/19956127-06640af149d93ffe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/19956127-86bec7fcba355ffa.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**Image**

显示图片的 Widget。图片常用的格式主要有bmp,jpg,png,gif,webp等，Android中并不是天生支持gif和webp动图，但是这一特性在flutter中被很好的支持了。
![](https://upload-images.jianshu.io/upload_images/19956127-6f2d96dad940c562.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**Iamge.asset**

在工程目录下创建目录，如：assets，将图片放入此目录。打开项目根目录：pubspec.yaml
![](https://upload-images.jianshu.io/upload_images/19956127-4e4676043b3fd2a3.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/19956127-18b6855da1bfc8b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**Image.file**

在sd卡中放入一张图片。然后利用path_provider库获取sd卡根目录(Dart库版本可以在：https://pub.dartlang.org/packages查询)。
![](https://upload-images.jianshu.io/upload_images/19956127-c30852fd1d184599.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**注意权限**
![](https://upload-images.jianshu.io/upload_images/19956127-93c64f602ac96fd3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**Image.network**

直接给网络地址即可。

Flutter 1.0，加载https时候经常出现证书错误。必须断开AS打开app
**Image.memory**
![](https://upload-images.jianshu.io/upload_images/19956127-2cc5e20792467635.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**CircleAvatar**

主要用来显示用户的头像，任何图片都会被剪切为圆形。
![](https://upload-images.jianshu.io/upload_images/19956127-1b7dd91a5a989ef2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**FadeInImage**

当使用默认 Image widget显示图片时，您可能会注意到它们在加载完成后会直接显示到屏幕上。这可能会让用户产生视觉突兀。如果最初显示一个占位符，然后在图像加载完显示时淡入，我们可以使用 FadeInImage来达到这个目的！
![](https://upload-images.jianshu.io/upload_images/19956127-d1f53469e7c2157c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 按钮

Material widget库中提供了多种按钮Widget如RaisedButton、FlatButton、OutlineButton等，它们都是直接或间接对RawMaterialButton的包装定制，所以他们大多数属性都和 RawMaterialButton一样。所有Material 库中的按钮都有如下相同点：

按下时都会有“水波动画”。
有一个 onPressed属性来设置点击回调，当按钮按下时会执行该回调，如果不提供该回调则按钮会处于禁用状态，禁用状态不响应用户点击。
**RaisedButton**

"漂浮"按钮，它默认带有阴影和灰色背景
![](https://upload-images.jianshu.io/upload_images/19956127-b3b1c1baeba25822.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**FlatButton**

扁平按钮，默认背景透明并不带阴影
![](https://upload-images.jianshu.io/upload_images/19956127-0c9e8403931a1c5b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**OutlineButton**

默认有一个边框，不带阴影且背景透明。
![](https://upload-images.jianshu.io/upload_images/19956127-b26ade0f5b3c5414.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**IconButton**

可点击的Icon
![](https://upload-images.jianshu.io/upload_images/19956127-cc360257091dfcb1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
按钮外观可以通过其属性来定义，不同按钮属性大同小异
![](https://upload-images.jianshu.io/upload_images/19956127-182bd2cbeb894a85.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
而 RaisedButton，默认配置有阴影，因此在配置 RaisedButton 时，拥有一系列 elevation 属性的配置
![](https://upload-images.jianshu.io/upload_images/19956127-5539046dc1dfebe9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##输入框
![](https://upload-images.jianshu.io/upload_images/19956127-5355d6ad3cd5ad13.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/19956127-5157cfc2a6e0f976.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这个效果非常的“系统”，我们可能大多数情况下需要将下划线更换为矩形边框，这时候可能就需要组合widget来完成:
![](https://upload-images.jianshu.io/upload_images/19956127-ca3aebf08d854072.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/19956127-545a5a1242de76a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 焦点控制

FocusNode: 与Widget绑定，代表了这个Widget的焦点

FocusScope: 焦点控制范围

FocusScopeNode：控制焦点

![](https://upload-images.jianshu.io/upload_images/19956127-6ce47d99f895c8d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/19956127-a6ed9f4853daed6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 获取输入内容

获取输入内容有两种方式：

定义两个变量，用于保存用户名和密码，然后在onChange触发时，各自保存一下输入内容。
通过controller直接获取。
onChange获得输入内容:
![](https://upload-images.jianshu.io/upload_images/19956127-00da698c8aa2037f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
controller获取:

定义一个controller：
![](https://upload-images.jianshu.io/upload_images/19956127-ffbf575932d77c5b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后设置输入框controller：
![](https://upload-images.jianshu.io/upload_images/19956127-40ec731d1ce5ce48.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
通过controller获取输入框内容
```
debugPrint(_unameController.text)
```
## TextFormField

TextFormField比 TextField多了一些属性，其中 validator用于设置验证回调。在单独使用时与 TextField没有太大的区别。当结合 From，利用 From可以对输入框进行分组，然后进行一些统一操作(验证)
![](https://upload-images.jianshu.io/upload_images/19956127-85d62f5c1db80300.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/19956127-f094bc10b96724ed.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
原文作者：潇湘夜雨
原文链接：https://mp.weixin.qq.com/s/CdJDenLMs8eDR0drA4_ffA
阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)


































