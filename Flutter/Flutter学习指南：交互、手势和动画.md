阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)

# 手势处理

## 按钮点击

为了获取按钮的点击事件，只需要设置 onPressed 参数就可以了：

```
class TestWidget extends StatelessWidget {  
@override  
Widget build(BuildContext context) {    
return RaisedButton(      
child: Text('click'),      
onPressed: () => debugPrint('clicked'),    
);  
}
}
```

## 任意控件的手势事件

跟 button 不同，大多数的控件没有手势事件监听函数可以设置，为了监听这些控件上的手势事件，我们需要使用另一个控件——GestureDetector（没错，它也是一个控件）：

```
class TestWidget extends StatelessWidget {  
@override  
Widget build(BuildContext context) {    
return GestureDetector(      
child: Text('text'),      
onTap: () => debugPrint('clicked'),    
);  
}
}
```

除了上面代码使用到的 onTap，GestureDetector 还支持许多其他事件：

*   onTapDown：按下

*   onTap：点击动作

*   onTapUp：抬起

*   onTapCancel：前面触发了 onTapDown，但并没有完成一个 onTap 动作

*   onDoubleTap：双击

*   onLongPress：长按

*   onScaleStart, onScaleUpdate, onScaleEnd：缩放

*   onVerticalDragDown, onVerticalDragStart, onVerticalDragUpdate, onVerticalDragEnd, onVerticalDragCancel, onVerticalDragUpdate：在竖直方向上移动

*   onHorizontalDragDown, onHorizontalDragStart, onHorizontalDragUpdate, onHorizontalDragEnd, onHorizontalDragCancel, onHorizontalDragUpdate：在水平方向上移动

*   onPanDown, onPanStart, onPanUpdate, onPanEnd, onPanCancel：拖曳（水平、竖直方向上移动）

如果同时设置了 onVerticalXXX 和 onHorizontalXXX，在一个手势里，只有一个会触发（如果用户首先在水平方向移动，则整个过程只触发 onHorizontalUpdate；竖直方向的类似）

这里要说明的是，onVerticalXXX/onHorizontalXXX 和 onPanXXX 不能同时设置。如果同时需要水平、竖直方向的移动，使用 onPanXXX。

如果读者希望在用户点击的时候能够有个水波纹效果，可以使用 InkWell，它的用法跟 GestureDetector 类似，只是少了拖动相关的手势（毕竟，这个水波纹效果只有在点击的时候才有意义）。

## 原始手势事件监听

GestureDetector 在绝大部分时候都能够满足我们的需求，如果真的满足不了，我们还可以使用最原始的 Listener 控件。

```
class TestWidget extends StatelessWidget {  
@override  
Widget build(BuildContext context) {    
return Listener(      
child: Text('text'),      
onPointerDown: (event) => print('onPointerDown'),      
onPointerUp: (event) => print('onPointerUp'),      
onPointerMove: (event) => print('onPointerMove'),      
onPointerCancel: (event) => print('onPointerCancel'),    
);  
}
}
```

# 在页面间跳转

Flutter 里所有的东西都是 widget，所以，一个页面，也是 widget。为了调整到新的页面，我们可以 push 一个 route 到 Navigator 管理的栈中。

```
Navigator.push(  
context,  
MaterialPageRoute(builder: (_) => SecondScreen())
);
```

需要返回的话，pop 掉就可以了：

```
Navigator.pop(context);
```

下面是完整的例子：

![](https://upload-images.jianshu.io/upload_images/19956127-1736b6e71d272e36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



除了打开一个页面，Flutter 也支持从页面返回数据：

```
Navigator.pop(context, 'message from second screen');
```

由于打开页面是异步的，页面的结果通过一个 Future 来返回：

```
onPressed: () async {  
// Navigator.push 会返回一个 Future<T>，如果你对这里使用的 await不太熟悉，可以参考  
// https://www.dartlang.org/guides/language/language-tour#asynchrony-support  
var msg = await Navigator.push(    
context,    
MaterialPageRoute(builder: (_) => SecondScreen())  
);  
debugPrint('msg = $msg');}
```

我们还可以在 MaterialApp 里设置好每个 route 对应的页面，然后使用 Navigator.pushNamed(context, routeName) 来打开它们：

```
MaterialApp(  
// 从名字叫做 '/' 的 route 开始（也就是 home）  
initialRoute: '/',  
routes: {    
'/': (context) => HomeScreen(),    
'/about': (context) => AboutScreen(),  
},
);
```

接下来，我们通过实现一个 echo 客户端的前端页面来综合运用前面所学的知识（逻辑部分我们留到下一篇文章再补充）。

# echo 客户端

## 消息输入页

这一节我们来实现一个用户输入的页面。UI 很简单，就是一个文本框和一个按钮。

![](https://upload-images.jianshu.io/upload_images/19956127-08685a1fbc89dc05.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/19956127-3e017e5e3ca61426.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里的按钮本应该使用 RaisedButton 或 FlatButton。为了演示如何监听手势事件，我们这里故意自己用 Container 做了一个按钮，然后通过 InkWell 监听手势事件。InkWell 除了上面展示的几个事件外，还带有一个水波纹效果。如果不需要这个水波纹效果，读者也可以使用 GestureDetector。

## 消息列表页面

我们的 echo 客户端共有两个页面，一个用于展示所有的消息，另一个页面用户输入消息，后者在上一小节我们已经写好了。下面，我们来实现用于展示消息的页面。

### 页面间跳转

我们的页面包含一个列表和一个按钮，列表用于展示信息，按钮则用来打开上一节我们所实现的 AddMessageScreen。这里我们先添加一个按钮并实现页面间的跳转。

```
// 这是我们的消息展示页面class MessageListScreen extends StatelessWidget {  
@override  
Widget build(BuildContext context) {    
return Scaffold(      
appBar: AppBar(        
title: Text('Echo client'),      
),      
floatingActionButton: FloatingActionButton(        
onPressed: () {          
// push 一个新的 route 到 Navigator 管理的栈中，以此来打开一个页面          
Navigator.push(              
context,              
MaterialPageRoute(builder: (_) => AddMessageScreen())          
);        
},        
tooltip: 'Add message',        
child: Icon(Icons.add),      
)    
);  
}
}
```

在消息的输入页面，我们点击 Send 按钮后就返回：

```
onTap: () {  
debugPrint('send: ${editController.text}');  
Navigator.pop(context);
}
```

最后，我们加入一些骨架代码，实现一个完整的应用：

```
void main() {  
runApp(MyApp());
}

class MyApp extends StatelessWidget {  

@override  
Widget build(BuildContext context) {    
return MaterialApp(      
title: 'Flutter UX demo',      
home: MessageListScreen(),    
);  
}
}
```

但是，上面代码所提供的功能还不够，我们需要从 AddMessageScreen 中返回一个消息。

首先我们对数据建模：

```
class Message {  
final String msg;  
final int timestamp;  

Message(this.msg, this.timestamp);  
@override  
String toString() {    
return 'Message{msg: $msg, timestamp: $timestamp}';  
 }
}
```

下面是返回数据和接收数据的代码：

```
onTap: () {  
debugPrint('send: ${editController.text}');  
final msg = Message(    
editController.text,    
DateTime.now().millisecondsSinceEpoch  
);  
Navigator.pop(context, msg);
},

floatingActionButton: FloatingActionButton(  
onPressed: () async {    
final result = await Navigator.push(        
context,        
MaterialPageRoute(builder: (_) => AddMessageScreen())    
);    
debugPrint('result = $result');  },  // ...)
```

### 把数据展示到 ListView
![](https://upload-images.jianshu.io/upload_images/19956127-5e3706b5fcf742c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这段代码里唯一的新知识就是给 MessageList 的 key 参数，我们下面先看看如何使用他，然后再说明它的作用：

![](https://upload-images.jianshu.io/upload_images/19956127-27f05a1de4fe00ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

引入一个 GlobalKey 的原因在于，MessageListScreen 需要把从 AddMessageScreen 返回的数据放到 _MessageListState 中，而我们无法从 MessageList 拿到这个 state。

GlobalKey 的是应用全局唯一的 key，把这个 key 设置给 MessageList 后，我们就能够通过这个 key 拿到对应的 statefulWidget 的 state。

现在，整体的效果是这个样子的：

![message-list](https://upload-images.jianshu.io/upload_images/19956127-8222bff32992538f?imageMogr2/auto-orient/strip "message-list")

如果你遇到了麻烦，在 Github 上找到所有的代码：

```
git clone https://github.com/Jekton/flutter_demo.git 
cd flutter_demo 
git checkout ux-basic
```

# 动画
Flutter 动画的核心是 Animation，Animation 接受一个时钟信号（vsync），转换为 T 值输出。它控制着动画的进度和状态，但不参与图像的绘制。最基本的 Animation 是 AnimationController，它输出 [0, 1] 之间的值。

## 使用内置的 Widget 完成动画
为了使用动画，我们可以用 Flutter 提供的 AnimatedContainer、FadeTransition、ScaleTransition 和 RotationTransition 等 Widget 来完成。

下面我们就来演示如何使用 ScaleTransition：
![](https://upload-images.jianshu.io/upload_images/19956127-f5d22c1031b1a8cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


AnimationController 的输出是线性的。非线性的效果可以使用 CurveAnimation 来实现：

![](https://upload-images.jianshu.io/upload_images/19956127-932c6d6de6761654.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当然，我们还可以组合不同的动画：

```
class _AnimWidgetState extends State<AnimWidget>    
 with SingleTickerProviderStateMixin {  // ...  

@override  
Widget build(BuildContext context) {    
var scaled = ScaleTransition(      
child: FlutterLogo(size: 200.0),      
scale: curve,    
);    
return FadeTransition(      
child: scaled,      
opacity: curve,    
);  
}
}
```

更多的动画控件，读者可以参考 https://flutter.io/widgets/animation/。

## 自定义动画效果

上一节我们使用 Flutter 内置的 Widget 来实现动画。他们虽然能够完成日常开发的大部分需求，但总有一些时候不太适用。这时我们就得自己实现动画效果了。

前面我们说，AnimationController 的输出在 [0, 1] 之间，这往往对我们需要实现的动画效果不太方便。为了将数值从 [0, 1] 映射到目标空间，可以使用 Tween：

```
animationValue = Tween(begin: 0.0, end: 200.0).animate(controller)    // 每一帧都会触发 listener 回调   
..addListener(() {     
// animationValue.value 随着动画的进行不断地变化。我们利用这个值来实现      
// 动画效果      
print('value = ${animationValue.value}');    
});
```

下面我们来画一个小圆点，让它往复不断地在正弦曲线上运动。

![](https://upload-images.jianshu.io/upload_images/19956127-54ffb39a8219d2cc?imageMogr2/auto-orient/strip)

先来实现小圆点沿着曲线运动的效果：
![](https://upload-images.jianshu.io/upload_images/19956127-3c5bb84969527cbe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/19956127-dcdf137fff52e3c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上面的动画中，我们只是对位置做出了改变，下面我们将在位置变化的同时，也让小圆点从红到蓝进行颜色的变化。

```
class _AnimationState extends State<AnimationDemoView>    
with SingleTickerProviderStateMixin {  
// ...  
Animation<Color> color;  
void _initState() {    
// ...    
color = ColorTween(begin: Colors.red, end: Colors.blue).animate(controller);    
controller.forward();  
}  
@override  
Widget build(BuildContext context) {    
// ...    
final color = this.color == null ? Colors.red : this.color.value;    
return Container(      
// 我们根据动画的进度设置圆点的位置      
margin: EdgeInsets.only(left: marginLeft, top: marginTop),      
// 画一个小圆点      
child: Container(        
decoration: BoxDecoration(            
color: color, borderRadius: BorderRadius.circular(7.5)),        
width: 15.0,        
height: 15.0,      
),    
);  
}
}
```

在 GitHub 上，可以找到所有的代码：

```
git clone https://github.com/Jekton/flutter_demo.git
cd flutter_demo
git checkout sin-curve
```

在这个例子中，我们还可以加多一些效果，比方说让小圆点在运动的过程中大小也不断变化、使用 CurveAnimation 改变它运动的速度，这些就留给读者作为练习吧。
原创： 水晶虾饺 
原文链接：https://mp.weixin.qq.com/s/653zL1YvuMwysKu907EQ-g
原文转自：[玉刚说]微信公众号

阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)
