原文作者：[腾讯技术](https://zhuanlan.zhihu.com/tencent-TEG)
原文链接：[https://zhuanlan.zhihu.com/p/90836859](https://zhuanlan.zhihu.com/p/90836859)
来源：知乎
> 本文真对 Flutter 的技术特性，做了一些略全面的入门级的介绍，如果你听说过Flutter，想去了解它，但是又不想去翻厚厚的API，那么本文就是为你准备的。

随着纯客户端到Hybrid技术，到RN&Weex，再到如今的Flutter技术，客户端实现技术不断前进。 在之前的一个APP项目中，因为历史原因当时选择了weex，随着使用的不断深入，我们逐渐发现了weex的渲染性能问题已经成为一个隐患和瓶颈。 而Flutter技术的不断成熟和流行，Flutter的良好的跨平台性和高性能优点，不断吸引着我们。

（本文包含以下内容，阅读完需要约18分钟）

*   1.Flutter是啥玩意儿？
*   2.移动端跨平台技术对比
    2.1 H5+原生APP
    2.2 RN&Weex
    2.3 Flutter
*   3.Dart语言
*   4.环境配置
*   5.Hello World
    5.1 创建项目
    5.2 项目结构
    5.3 启动模拟器
    5.4 启动项目APP
    5.5 简化版的Hello World
    5.6 给页面加上状态
    5.7 小结一下
*   6.路由
    6.1 单个页面的跳转
    6.2 更多页面跳转使用路由表
    6.3 路由传参
*   7.widget
    7.1 Text
    7.2 Button
    7.3 Container
    7.4 Image
*   8.布局
    8.1 Row & Column & Center 行列轴布局
    8.2 Align 角定位布局
    8.3 Stack & Positioned 绝对定位
    8.4 Flex & Expanded 流式布局
*   9.动画
    9.1 简单动画：淡入淡出
    9.2 复杂一些的动画：放大缩小
*   10.http请求
    10.1 HttpClient
    10.2 http
    10.3 Dio
*   11.吐吐槽
    11.1 墙
    11.2 组件过度设计
    11.3 嵌套太多不适应
    11.4 布局修改会导致嵌套关系修改
    11.5 Dart语言升级
    11.6 不能热更新
*   12.结语

## **1.Flutter是啥玩意儿？**

Flutter是谷歌的移动UI框架，可以快速在iOS和Android上构建高质量的原生用户界面。

*   具有**跨平台**开发特性，支持IOS、Android、Web三端。
*   **热重载**特性大大提高了开发效率
*   **自绘UI引擎**和编译成原生代码的方式，使得系统的运行时的**高性能**成为了可能
*   使用**Dart语言**，目前已经支持同时编译成Web端代码，

到底值不值得跟进Flutter技术呢？ 还是看下Flutter，Weex，ReactNative的搜索指数对比，大概就知道这个行业趋势了。

![](https://upload-images.jianshu.io/upload_images/19956127-c4ffa7360e126aea.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

蓝色是Flutter，可以看出上升势头非常强劲。**苦逼的前端就是这样，你不跟潮流，潮流就会把你抛弃。**

## **2.移动端跨平台技术对比**

为啥会有Flutter这种东西? 他的原理是什么？ 他是怎么做到高性能的？ 要明白这些问题，我们不得不从几种移动端跨平台技术的对比讲起。

### **2.1 H5+原生APP**

![](https://upload-images.jianshu.io/upload_images/19956127-8ced3b54688dc203.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

技术门槛最低，接入速度最快，热更新最方便的，自然就是H5方式。APP中提供一个Webview使用H5页面的Http直连。APP和H5可以相互独立开发，JS使用Bridge与原生进行数据通信，显示界面依赖Webview的浏览器渲染。 但是带来的问题也很明显，因为是需要远程直连，那么初次打开H5页面，会有瞬间的白屏，并且Webview本身会有至少几十M的内存消耗。

当然，作为前端开发人员，在H5方式可以使用SPA单页面、懒加载、离线H5等各种前端优化手段进行性能优化，以使得H5的表现更接近原生。但是首次的瞬间白屏和内存，Bridge的通信效率低下，始终是被技术框架给局限住了。

### **2.2 RN&Weex**

![](https://upload-images.jianshu.io/upload_images/19956127-57998a5b2a58d08a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由于H5的那些弊端，爱折腾的前端工程师，祭出了RN、Weex两个大杀器， 使用原生去解析RN、Weex的显示配置，显示层、逻辑层都直接与原生数据通信。 因为抛弃了浏览器，自然渲染性能、执行性能都提升了一大截。

但是，每次遇到显示的变更，JS都还会通过Bridge和原生转一道再做渲染的调整，所以Bridge就最后成为了性能的瓶颈。在实际项目中，特别是做一些大量复杂动画处理的时候，由于渲染部分需要频繁通信，性能问题变得尤为突出。 有兴趣的同学可以去看看[BindingX](https://link.zhihu.com/?target=https%3A//github.com/alibaba/bindingx)，里面有关于动画中数据通信效率低下导致动画帧率低下的详细说明。

### **2.3 Flutter**

![](https://upload-images.jianshu.io/upload_images/19956127-5406a557f694a0b4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

不得不佩服Google开发人员的想象力，为了达到极致性能，Flutter更前进了一步，Flutter代码编译完成以后，直接就是原生代码，并且使用自绘UI引擎原生方式做渲染。 Flutter依赖一个[Skia 2D图形化引擎](https://link.zhihu.com/?target=https%3A//github.com/google/skia)。Skia也是Android平台和Chrome的底层渲染引擎，所以性能方面完全不用担心。因为使用Dart做AOT编译成原生，自然也比使用解释性的JS在V8引擎中执行性能更快，并且因为去掉Bridge，没有了繁琐的数据通信和交互，性能就更前进了一步。

## **3.Dart语言**

学习Flutter，得先了解Dart。Dart语言曾经雄心勃勃的要替换Javascript, 但是发布的时机正好遇到JS的飞速发展，于是就逐渐沉寂，直到配合Flutter的发布，才又重新焕发了生机。

在最近2019年9月的一次[Google开发者大会中](https://link.zhihu.com/?target=https%3A//developers.googleblog.com/2019/09/flutter-news-from-gdd-china-flutter1.9.html)，伴随着Flutter1.9的发布，目前的[Dart也同时更新到了2.5版本](https://link.zhihu.com/?target=https%3A//medium.com/dartlang/announcing-dart-2-5-super-charged-development-328822024970)， 提供了机器学习和对C跨平台调用的能力。总体来说，Dart语法，对于前端同学，上手还是很容易的，风格很像。

![](https://upload-images.jianshu.io/upload_images/19956127-4591d121b40e3a9b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

关于Dart语法，请移步传送门：[https://dart.dev/samples](https://link.zhihu.com/?target=https%3A//dart.dev/samples)

## **4.环境配置**

无论学什么新语言，首先都是环境配置。由于Flutter出自Google，所以有一定门槛，如果在公司内安装，你还需要一个方便的代理切换工具, 比如：[Proxifier](https://link.zhihu.com/?target=https%3A//www.proxifier.com/) 。

安装教程，参照官网：[https://flutter.dev/docs/get-started/install](https://link.zhihu.com/?target=https%3A//flutter.dev/docs/get-started/install)

Flutter支持多种编辑器如：Android Studio , XCode。 但是既然作为支持跨双端的开发，个人还是推荐使用 [VSCode](https://link.zhihu.com/?target=https%3A//code.visualstudio.com/)。

VSCode安装完成后，需要安装Flutter插件，和Dart插件. 在扩展窗口里，搜索**Flutter**，和**Dart**，点击“Install”即可，非常方便。

![](https://upload-images.jianshu.io/upload_images/19956127-3b4717c9dc0c5033.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果安装不上去，记得开启下代理。

## **5.Hello World**

作为一个伟大的程序员，第一行代码总是从Hello World开始。^_^

### **5.1 创建项目：**

**方法1：直接使用命令创建：**

```
flutter create projectname

```

**方法2：使用VSCode创建:**

View -> Command Palette -> Flutter:New Project 即可

![](https://upload-images.jianshu.io/upload_images/19956127-f9f82e84b5c0eced.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/19956127-c3743c8795d033b2.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 注意请先打开代理，否则你的创建进度，会一直被卡住。

### **5.2 项目结构**

将项目先拖入VSCode，看下目录结构。自动创建完成的项目中，我们看到已经自带了Android，IOS相关的运行环境。

入口主文件是main.dart. 可以打开来先熟悉下，暂时不了解没关系，后面再讲。

还有一个重要的文件是pubspec.yaml ，是项目的配置文件，这个后续也会做修改。

![](https://upload-images.jianshu.io/upload_images/19956127-66a8a83b2b73daac.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### **5.3 启动模拟器**

点击VSCode右下角的模拟器，启动模拟器。(VSCode会自动找到Android环境、IOS环境下的模拟器，以及真机环境)

![](https://upload-images.jianshu.io/upload_images/19956127-613861b78fd05414.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### **5.4 启动项目APP**

![](https://upload-images.jianshu.io/upload_images/19956127-25b1afbfe19c04e9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选中Main.dart， 点击**Debug-> Start Debugging** , 项目就会启动调试，并在模拟器里运行。

![](https://upload-images.jianshu.io/upload_images/19956127-eb731c054ba362f3.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### **5.5 简化版的Hello World**

讲道理，Flutter一上来就用StatefulWidget做一个自增的Demo，其实是对新手不太友好。 我还是喜欢循序渐进，先删掉那些复杂的自增逻辑，我们基于StatelessWidget 只做一个最简单的静态页面显示。（什么是StatefulWidget 和StatelessWidget？后面会说）

**main.dart**

```
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatelessWidget{
   @override
  Widget build(BuildContext context) {
       return Scaffold(
            appBar: AppBar(
              title: Text("我是Title"),
            ),
            body: Center(
                    child: Text(
                        'Hello World',
                    )
            )
      );
  }  
}

```

在上面的代码中，可以清楚看到，最简单的页面的层级关系：

**MaterialApp -> MyHomePage -> Scaffold -> body ->** **Center -> Text**

> **Scaffold是啥？**他是Flutter的页面脚手架，你可以当HTML页面一样去理解，不同的是，他除了Body以外，还提供appBar顶部TitleBar、bottomNavigationBar底部导航栏等属性。

显示效果：

![](https://upload-images.jianshu.io/upload_images/19956127-ab00cfd73b4250cc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这是最简单的页面，没有交互，只有显示，但是实际业务场景中，是不太可能都是这种页面的，页面上的数据一般都是来自接口返回，然后再在页面上进行动态的渲染。 此时，就需要使用使用带状态的StatefulWidget了

### **5.6 给页面加上状态**

给自己一个需求，按钮点击时，修改页面上显示的文字“Hello World” 变成“You Click Me”

```
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget{
  @override
  MyHomePageState createState() => MyHomePageState();
}

class MyHomePageState extends State<MyHomePage>{
   var msg="Hello World"; //msg默认文字
   @override
   Widget build(BuildContext context) {
       return Scaffold(
            appBar: AppBar(
              title: Text("我是Title"),
            ),
            body: Center(
                      child:Column(
                              children:<Widget>[
                                  Text(msg), //根据变量值，显示文字
                                  FlatButton(
                                      color: Colors.blue,
                                      textColor: Colors.white,
                                      //点击按钮，修改msg的文字
                                      onPressed: () {
                                        setState(() {
                                          this.msg="You Click ME";
                                        });
                                      },
                                      child: Text(
                                        "Click ME",
                                        style: TextStyle(fontSize: 20.0),
                                      ),
                                  )
                              ]
                      )
                  )
      );
  }  

}

```

执行效果：

![](https://upload-images.jianshu.io/upload_images/19956127-39762f5678bd7337.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上面最关键的一段代码就是这个：

```
 onPressed: () {
         setState(() {
                this.msg="You Click ME";
          });
 },

```

相信写过小程序的同学，对这个 setState 还是很眼熟的 ^_^

### **5.7 小结一下**

> **StatelessWidget**：无状态变更，UI静态固化的Widget， 页面渲染性能更高。
> **StatefulWidget**：因状态变更可以导致UI变更的的Widget，涉及到数据渲染场景，都使用StatefulWidget。

为啥要分两个？ StatelessWidget拥有的功能，StatefulWidget都有了啊？

答案只有一个：**性能、性能、性能**

在StatefulWidget里，因为要维护状态，他的生命周期比StatelessWidget更复杂，每次执行setState，都会触发
window.scheduleFrame() 导致整个页面的widget被刷新，性能就会降低。

使用过小程序的同学在这点上应该有体会，在[小程序的官方文档](https://link.zhihu.com/?target=https%3A//developers.weixin.qq.com/miniprogram/dev/framework/performance/tips.html)中，会强烈建议减少setData的使用频率，以避免性能的下降。 只不过flutter更是激进，推出了StatelessWidget，并直接在该Widget里砍掉了setState的使用。

**页面结构关系如下：**

![](https://upload-images.jianshu.io/upload_images/19956127-824fad639d5f04f2.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## **6.路由**

实际的项目，是有多个不同的页面的，页面之间的跳转，就要用到路由了。 我们增加一个list页面，点击Home页的“Click Me”按钮，跳转到列表页list。

### **6.1 单个页面的跳转**

**增加list.dart**

```
import 'package:flutter/material.dart';

class ListPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    //定义列表widget的list
    List<Widget> list=<Widget>[];

    //Demo数据定义
    var data=[
      {"id":1,"title":"测试数据AAA","subtitle":"ASDFASDFASDF"},
      {"id":2,"title":"测试数据bbb","subtitle":"ASDFASDFASDF"},
      {"id":3,"title":"测试数据ccc","subtitle":"ASDFASDFASDF"},
      {"id":4,"title":"测试数据eee","subtitle":"ASDFASDFASDF"},
    ];

    //根据Demo数据，构造列表ListTile组件list
    for (var item in data) {
      print(item["title"]);

      list.add( ListTile( 
          title: Text(item["title"],style: TextStyle(fontSize: 18.0) ),
          subtitle: Text(item["subtitle"]),
          leading:  Icon( Icons.fastfood, color:Colors.orange ),
          trailing: Icon(Icons.keyboard_arrow_right)
      ));
    }

    //返回整个页面
    return Scaffold(
      appBar: AppBar(
        title: Text("List Page"),
      ),
      body: Center(
        child: ListView(
          children: list,
        )
      ),
    );
  }
}

```

在main.dart增加list页面的引入

```
import 'list.dart';

```

修改Home页的按钮事件，增加Navigator.push跳转

```
FlatButton(
          color: Colors.blue,textColor: Colors.white,
          onPressed: () {    
                       Navigator.push(context, MaterialPageRoute(builder:(context) {
                                return  ListPage();
                       }));
              },
           child: Text("Click ME",style: TextStyle(fontSize: 20.0) ),
    )

```

核心方法就是：**Navigator.push(context,MaterialPageRoute)**

跳转示例：

![](https://upload-images.jianshu.io/upload_images/19956127-858871c259a354d4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### **6.2 更多页面跳转使用路由表**

在MaterialApp中，有一个属性是routes，我们可以对路由进行命名，这样跳转的时候，只需要使用对应的路由名字即可，如：**Navigator.pushNamed(context, RouterName)**。点击两个不同的按钮，分别跳转到ListPage，和Page2去。

**Main.dart修改一下如下**：

```
import 'package:flutter/material.dart';
import 'list.dart';
import 'page2.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      //路由表定义
      routes:{
        "ListPage":(context)=> ListPage(),
        "Page2":(context)=> Page2(),
      },
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget{
  @override
  MyHomePageState createState() => MyHomePageState();
}

class MyHomePageState extends State<MyHomePage>{
   @override
   Widget build(BuildContext context) {
       return Scaffold(
            appBar: AppBar(
              title: Text("我是Title"),
            ),
            body: Center(
                      child:Column(
                              children:<Widget>[
                                  RaisedButton(
                                      child: Text("Clikc to ListPage" ),
                                      onPressed: () {
                                        //根据命名路由做跳转
                                         Navigator.pushNamed(context, "ListPage");
                                      },
                                  ),
                                   RaisedButton(
                                      child: Text("Click to Page2" ),
                                      onPressed: () {
                                          //根据命名路由做跳转
                                         Navigator.pushNamed(context, "Page2");
                                      },
                                  )

                              ]
                      )
                  )
      );
  }  

}

```

示例：

![](https://upload-images.jianshu.io/upload_images/19956127-316168b74c8dd61e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当我们有了路由以后，就可以开始在一个项目里用不同的页面，去学习不同的功能了。

### **6.3 路由传参**

列表页跳转到详情页，需要路由传参，这个在flutter体系里，又是怎么做的呢？

首先，在main.dart里，增加详情页DedailPage的路由配置

```
//路由表定义
      routes:{
        "ListPage":(context)=> ListPage(),
        "Page2":(context)=> Page2(),
        "DetailPage":(context)=> DetailPage(), //增加详情页的路由配置
      },

```

并修改ListPage里ListTile的点击事件，增加路由跳转传参，这里是将整个item数据对象传递

```
ListTile( 
          title: Text(item["title"],style: TextStyle(fontSize: 18.0) ),
          subtitle: Text(item["subtitle"]),
          leading:  Icon( Icons.fastfood, color:Colors.orange ),
          trailing: Icon(Icons.keyboard_arrow_right),
          onTap:(){
            //点击的时候，进行路由跳转传参
             Navigator.pushNamed(context, "DetailPage", arguments:item);
          },
      )

```

详情页DetailPage里，获取传参并显示

```
import 'package:flutter/material.dart';
class DetailPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
     //获取路由传参
     final Map args = ModalRoute.of(context).settings.arguments;

    return Scaffold(
      appBar: AppBar(
        title: Text("Detail Page"),
      ),
      body: 
        new Column(
          children: <Widget>[
             Text("我是Detail页面"),
             Text("id:${args['id']}" ),
             Text("id:${args['title']}"),
             Text("id:${args['subtitle']}")
          ],
        )
      );
  }
}

```

Demo效果：

![](https://upload-images.jianshu.io/upload_images/19956127-b9579d5f2b7f6b7e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## **7.widget**

Flutter提供了很多默认的组件，而每个组件的都继承自widget 。 在Flutter眼里：**一切都是widget**。 这句看起来是不是很熟悉？ 还记得在webpack里，一切都是module吗？ 类似的还有java的一切都是对象。貌似任何一个技术，最后都是用哲学作为指导思想。

widget，作为可视化的UI组件，包含了显示UI、功能交互两部分。大的widget，也可以由多个小的widget组合而成。

常用的widget组件：

### **7.1 Text**

**Demo:**

```
Text(
         "Hello world",
         style: TextStyle(
                      fontSize: 50,
                      fontWeight: FontWeight.bold,
                      color:Color(0xFF0000ff)
                  )
    ),

```

Text的样式，来自另一个widget：TextStyle。 而TextStyle里的color，又是另一个widget Color的实例。

如果用flutter的缩进的方法，看起来确实有点丑陋，习惯写CSS的前端同学，可以看看下面的风格：

```
Text( "Hello world", style: TextStyle( fontSize: 50,fontWeight: FontWeight.bold,color:Color(0xFF0000ff) ) )

```

写成一行，是不是就顺眼多了？这算前端恶习吗？^_^

![](https://upload-images.jianshu.io/upload_images/19956127-03c954fb6a1a497f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### **7.2 Button**

对于flutter来说，Button就提供了很多种，我们来看看他们的区别：

> **RaisedButton**: 凸起的按钮
> **FlatButton**：扁平化按钮
> **OutlineButton**：带边框按钮
> **IconButton**：带图标按钮

按钮测试页dart:

```
import 'package:flutter/material.dart';

class ButtonPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {

    return Scaffold(
      appBar: AppBar(
        title: Text("Button Page"),
      ),
      body: Column(
        children: <Widget>[
             RaisedButton(
                  child: Text("我是 RaiseButton" ),
                  onPressed: () {},
              ),
               FlatButton(
                  child: Text("我是 FlatButton" ),
                  color: Colors.blue,
                  onPressed: () {},
              ),
              OutlineButton(
                  child: Text("我是 OutlineButton" ),
                  textColor: Colors.blue,
                  onPressed: () {},
              ),
              IconButton(
                  icon: Icon(Icons.add),
                  onPressed: () {},
              )  
        ]
      )
    );
  }
}

```

Demo:

![](https://upload-images.jianshu.io/upload_images/19956127-f29490bc886b2dd3.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

项目中要用哪个，就各取所需吧~

### **7.3 Container**

Container是非常常用的一个widget，他一般是用作一个容器。我们先来看看他的基础属性，顺便可以想想他像HTML里的啥？

**基础属性：width，height，color，child**

```
body: Center(
        child: Container(
           color: Colors.blue,
           width: 200,
           height: 200,
           child: Text("Hello Container ",style:TextStyle(fontSize: 20,color: Colors.white)),
        )
      )

```

![](https://upload-images.jianshu.io/upload_images/19956127-c3008490e4d87cdc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**Padding**

我们也可以不设置宽高，用**padding**在内部撑开增加留白：

```
Container(
           color: Colors.blue,
           padding: EdgeInsets.all(30),
           child: Text("Hello Container ",style:TextStyle(fontSize: 20,color: Colors.white)),

        )

```

![](https://upload-images.jianshu.io/upload_images/19956127-fe6017a8d3ac8283.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**Margin**

我们还可以使用**margin**，在容器的外部撑开增加偏移量，

```
Container(
           color: Colors.blue,
           padding: EdgeInsets.all(30),
           margin: EdgeInsets.only(left: 150,top: 0,right: 0,bottom: 0),
           child: Text("Hello Container ",style:TextStyle(fontSize: 20,color: Colors.white)),
        )

```

![](https://upload-images.jianshu.io/upload_images/19956127-1944f67b549b007f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**Transform**

我们还可以给这个矩形，使用tansform做一些变化，比如，旋转一个角度

```
Container(
           color: Colors.blue,
           padding: EdgeInsets.all(30),
           child: Text("Hello Container ",style:TextStyle(fontSize: 20,color: Colors.white)),
           transform: Matrix4.rotationZ(0.5)
        )

```

![](https://upload-images.jianshu.io/upload_images/19956127-d00a8564ba2ae4b7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看到这里，好多前端同学要说了，好熟悉啊。 对，他就是很像Html里的一个东西：**DIV**，你确实可以对应的去加强理解。

### **7.4 Image**

**网络图片加载**

使用NetworkImage，可以做网络图片的加载：

```
child:Image(
          image: NetworkImage("https://mat1.gtimg.com/pingjs/ext2020/qqindex2018/dist/img/qq_logo_2x.png"),
           width: 200.0,
        )  

```

![](https://upload-images.jianshu.io/upload_images/19956127-8176f5125ecf053a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**本地图片加载**

加载本地图片，就稍微复杂一些，首先要把图片的路径配置，加入到之前说过的pubspec.yaml配置文件里去:

![](https://upload-images.jianshu.io/upload_images/19956127-d15f23f3f65b62d3.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

加载本地图片时使用AssetImage：

```
child:Image(
               image: AssetImage("assets/images/logo.png"),
                width: 200.0,
            )      

```

也可以使用简写：

```
 Image.asset("assets/images/logo.png",width:200.0)

```

> flutter提供的组件很多，这里就不一一举例说明，有兴趣的还是建议大家去看API：[https://api.flutter.dev/](https://link.zhihu.com/?target=https%3A//api.flutter.dev/)

## **8.布局**

我们已经了解了这么多组件，那么怎么绘制一个完整的页面呢？ 这就到了页面布局的部分了。

### **8.1 Row & Column & Center 行列轴布局**

字面意义也很好理解，行布局、列布局、居中布局，这些布局对于Flutter来说，也都是一个个的widget。

区别在于，row、column 是有多个children的widget， 而Center是只有 1个child的 widget。

```
 Row(
     children:<Widget>[]
 ) 

 Column(
     children:<Widget>[]
 )    

 Center(
      child:Text("Hello")
 ）

```

### **8.2 Align 角定位布局**

我们常常在Container里，需要显示的内容在左上角，左下角，右上角，右下角。 在html时代，使用CSS可以很容易的实现，但是flutter里，必须依赖Align 这个定位的Widget

右下角定位示例：

```
 child: Container(
           color: Colors.blue,
           width: 300,
           height: 200,
           child: Align(
                      alignment: Alignment.bottomRight,
                      child:Text("Hello Align ",style:TextStyle(fontSize: 20,color: Colors.white)),
                  )
        )

```

显示效果：

![](https://upload-images.jianshu.io/upload_images/19956127-8d8d01fec8fd19d5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Alignment提供了多种定位供选择，还算是很贴心的。

![](https://upload-images.jianshu.io/upload_images/19956127-ee3e8f6adc8f6d8a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### **8.3 Stack & Positioned 绝对定位**

当然还有绝对定位的需求，这在css里，使用position：absolute就搞定了，但是在flutter里，需要借助stack+ positioned两个widget一起组合使用。

> **Stack**: 支持元素堆叠
> **Positioned**：支持绝对定位

```
child:Stack(
              children: <Widget>[
                  Image.network("https://ossweb-img.qq.com/upload/adw/image/20191022/627bdf586e0de8a29d5a48b86700a790.jpeg"),
                  Positioned(
                    top: 20,
                    right: 10,
                    child:Image.asset("assets/images/logo.png",width:200.0)
                  )
              ],
            )

```

![](https://upload-images.jianshu.io/upload_images/19956127-537742e180e77648.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### **8.4 Flex & Expanded 流式布局**

Flex流式布局作为前端同学都熟悉，之前讲过的Row，Column，其实都是继承自Flex，也属于流式布局。

> 如果轴向不确定，使用Flex，通过修改`direction的值设定轴向`
> 如果轴向已确定，使用Row，Column，布局更简洁，更有语义化

Flex测试页：

```
class FlexPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {

    return Scaffold(
      appBar: AppBar(
        title: Text("Flex Page"),
      ),
      body:  Flex(
          direction: Axis.horizontal,
          children: <Widget>[
            Container(
              width: 30,
              height: 100,
              color: Colors.blue,
            ),
            Expanded(
              flex: 1,
              child: Container(
                height: 100.0,
                color: Colors.red,
              ),
            ),
            Expanded(
              flex: 1,
              child: Container(
                height: 100.0,
                color: Colors.green,
              ),
            ),
          ],
        ),
    );
  }
}

```

示例中，轴向横向排列，最左边一个固定宽度的Container，右边两个Expanded，各自占剩下的宽度的一半。

![](https://upload-images.jianshu.io/upload_images/19956127-0c14dc90d0f48fe3.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## **9.动画**

Flutter既然说了，一切都是Widget，包括动画实现，也是一个Widget。 我们还是看一个示例

### **9.1 简单动画：淡入淡出：**

使用flutter提供的现成的Widget：

```
import 'package:flutter/material.dart';

class AnimatePage extends StatefulWidget {
  _AnimatePage  createState()=> _AnimatePage();
} 

class _AnimatePage extends State<AnimatePage> {
  bool _visible=true;
  @override
  Widget build(BuildContext context) {

    return Scaffold(
      appBar: AppBar(
        title: Text("Animate Page"),
      ),
      body: 
          Center(

            child: Column(
                  children: <Widget>[
                      AnimatedOpacity(
                        opacity: _visible ? 1.0:0.0,
                        duration: Duration(milliseconds: 1000),
                        child: Image.asset("assets/images/logo.png"),
                      ),

                      RaisedButton(
                        child: Text("显示隐藏"),
                        onPressed: (){
                          setState(() {
                            _visible=!_visible;
                          });
                         },
                      ),

                  ],
                ),
          )    
      );

  }
}

```

其中的AnimatedOpacity就是动画透明度变化的的Widget，而被透明度控制变化的Image则是AnimatedOpacity的子元素。这个和以往前端写动画的方式，就完全不一样了，需要改变一下思维方式。

Demo效果

![](https://upload-images.jianshu.io/upload_images/19956127-cdf202313faf0c79.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### **9.2 复杂一些的动画：放大缩小**

当写复杂一些动画的时候，没有对应的widget组件，就需要自己使用Animation，和AnimationController，以及Tween来组合。

> **Animation**: 保存动画的值和状态
> **AnimationController**: 控制动画，包含：启动forward()、停止stop()、反向播放reverse()等方法
> **Tween**: 提供begin，end作为动画变化的取值范围
> **Curve**：设置动画使用曲线变化，如非匀速动画，先加速，后减速等的设定。

动画示例：

```
class AnimatePage2 extends StatefulWidget {
  _AnimatePage  createState()=> _AnimatePage();
} 

class _AnimatePage extends State<AnimatePage2>  with SingleTickerProviderStateMixin {

  Animation<double> animation;
  AnimationController controller;

  initState() {
    super.initState();
    controller =  AnimationController(duration:  Duration(seconds: 3), vsync: this);

     //使用弹性曲线，数据变化从0到300
     animation = CurvedAnimation(parent: controller, curve: Curves.bounceIn);
     animation = Tween(begin: 0.0, end: 300.0).animate(animation)
      ..addListener(() {
        setState(() {
        });
      });

    //启动动画(正向执行)
    controller.forward();
  }

  @override
  Widget build(BuildContext context) {

    return Scaffold(
      appBar: AppBar(
        title: Text("Animate Page"),
      ),
      body: 
          Center(
              child: Image.asset(
                  "assets/images/logo.png",
                  width: animation.value, 
                  height: animation.value
              ),
            )  
      );   
  }

  dispose() {
    //路由销毁时需要释放动画资源
    controller.dispose();
    super.dispose();
  }

}

```

很重要的一点，**在路由销毁的时候，需要释放动画资源，否则容易导致内存泄漏**。

显示Demo：

![image](https://upload-images.jianshu.io/upload_images/19956127-39efd107435d7da8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## **10.http请求**

做业务逻辑，总离不开http请求，接下来，就来看下flutter的http请求是如何做的。

### **10.1 HttpClient**

httpClient在 dart:io库中，不需要引入第三方库就可以使用，示例代码如下：

**使用示例**

```
import 'dart:convert';
import 'dart:io';

Future _getByHttpClient() async{
    //接口地址
    const url="https://www.demo.com/api";

    //定义httpClient
    HttpClient client = new HttpClient();
    //定义request
    HttpClientRequest request = await client.getUrl(Uri.parse(url));
    //定义reponse
    HttpClientResponse response = await request.close();
    //respinse返回的数据，是字符串
    String responseBody = await response.transform(utf8.decoder).join();
    //关闭httpClient
    client.close();
    //字符串需要转化为JSON
    var json= jsonDecode(responseBody);
    return json;

} 

```

总的看起来，代码还是挺繁琐的，使用起来并不方便。

### **10.2 http**

这是Dart.dev提供的第三方类库，地址：[https://pub.dev/packages/http](https://link.zhihu.com/?target=https%3A//pub.dev/packages/http)

需要先在pubspec.yaml里添加类库应用

```
dependencies:
  flutter:
    sdk: flutter
  json_annotation: ^2.0.0
  http: ^0.12.0+2

```

**使用示例：**

```
Future _getByDartHttp() async {
  // 接口地址
 const url="https://www.demo.com/api";//获取接口的返回值
 final response = await http.get(url);
 //接口的返回值转化为JSON
 var json = jsonDecode(response.body); 
 return json;
}

```

这种写法，比上面的httpClient简洁了许多。

### **Dio**

国内使用最广泛的，还是flutterchina在github上提供的Dio第三方库，目前Star达到了5800多个。

官网地址：[https://github.com/flutterchina/dio](https://link.zhihu.com/?target=https%3A//github.com/flutterchina/dio)

使用Dio，因为是第三方库，所以同样要先在 pubspec.yaml 添加第三方库引用。

```
dependencies:
  flutter:
    sdk: flutter
  json_annotation: ^2.0.0
  dio: 2.1.16

```

使用示例：

```
import 'package:dio/dio.dart';

Future _getByDio() async{

      // 接口地址
      const url="https://www.demo.com/api";

      //定义 Dio实例
      Dio dio = new Dio();
      //获取dio返回的Response
      Response response = await dio.get(url);
      //返回值转化为JSON
      var json=jsonDecode(response.data);
      return json;
}

```

接口调用也是比httpclient简单很多，可能由于fluterchina在他的官方教程里，极力推荐这个dio库，所以目前这个第三方库的使用情况最为广泛。和Dart.dev的http不同的是，他需要new一个Dio的实例，在创建实例的时候，还可以传入更多的扩展配置参数。

```
BaseOptions options = new BaseOptions(
    baseUrl: "https://www.xx.com/api",
    connectTimeout: 5000,
    receiveTimeout: 3000,
);
Dio dio = new Dio(options);

```

## **11.吐吐槽**

学习Flutter的过程中，其实还是有很多坎坷和需要吐槽的地方。

### **11.1 墙**

因为有墙在，所以在配置flutter，或者下载flutter插件和第三方库的时候，需要墙内外来回切换。

### **11.2 组件过度设计**

提供的各种widget组件很多，但是真正核心的组件、常用的组件，也就哪些。 比如Flex 和column、row的关系，比如，Tween 与IntTween，ColorTween，SizeTween等20多个Tween子类之间的关系，你需要花很大的精力，去看每个具体子类的实现差别。

### **11.3 嵌套太多不适应**

因为嵌套层级很多，而且布局、动画、功能都在一起，第一次上手Flutter和Dart，这种嵌套关系让人很晕菜，这个只能去慢慢克服。 另外，多开发自定义的组件，可以让嵌套关系看起来清晰一些。

### **11.4 布局修改会导致嵌套关系修改**

前端的html+css分离世界里，不改变嵌套关系，修改CSS就可以调整布局。 但是在Flutter里因为布局也是嵌套关系，这就导致必须去改变嵌套关系。 要让嵌套更简单变动影响更小，页面拆分成子组件变得尤为重要。

### **11.5 Dart语言升级**

没错，语言升级也会导致学习的困扰，外面的资料新旧都有，比如有些是 new Text() ,有些直接是Text() ，新手上路会很晕菜。 其实这都是Dart语言升级导致的，记住Dart升级2.X以后，都不使用new了。感兴趣的可以自己去看下Dart的升级变更说明。

### **11.6 不能热更新**

年中的时候，Google官方宣布flutter暂不官方支持热更新，但是闲鱼团队已经有了自己的热更新方案。 关于热更新，只能静观其变了。 性能、开发效率、热更新，总是要有取舍的。即使是闲鱼团队，热更新也是付出了一点点性能下降的代价的，这是你选择flutter的初衷吗？还是那句话：权衡得失。

## **12.结语**

随着 9 月谷歌发布 Flutter1.9 以及flutter for web，Flutter的组件化思路，使得一份代码跨三端变成可能，相信Flutter的未来会更加广阔。

这不是一篇教程，只是在学习Flutter过程中的一点体验和经历，也因为时间关系，研究并不深入，如有疏漏，还请不吝赐教。
## 学习分享，共勉
题外话，毕竟我在三星小米工作多年，深知技术改革和创新的方向，Flutter作为跨平台开发技术、Flutter以其美观、快速、高效、开放等优势迅速俘获人心，但很多FLutter兴趣爱好者进阶学习确实资料，今天我把我搜集和整理的这份学习资料分享给有需要的人，若有关Flutter学习进阶可以与我在Flutter跨平台开发终极之选交流群一起讨论交流。下载地址：https://shimo.im/docs/yTD3t8Pjq3XJtGv8
![](https://upload-images.jianshu.io/upload_images/19956127-10e159838da0bd01.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**下载地址：https://shimo.im/docs/yTD3t8Pjq3XJtGv8**
