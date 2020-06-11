阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)

Flutter中主要有以下几种布局类的Widget：

* 线性布局Row和Column
* 弹性布局Flex
* 流式布局Wrap、Flow
* 层叠布局Stack、Positioned
线性布局Row和Column
线性布局其实是指沿水平或垂直方向排布子Widget，Flutter中通过Row来实现水平方向的子Widegt布局，通过Column来实现垂直方向的子Widget布局。他们都继承Flex，所以它们有很多相似的属性。
![](https://upload-images.jianshu.io/upload_images/19956127-42c0e45b9cf27ca2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/19956127-ae50b536b28db332.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在前端的Flex布局中，默认存在两根轴：水平的主轴（main axis）和垂直的交叉轴（cross axis）。主轴的开始位置（与边框的交叉点）叫做main start，结束位置叫做main end；交叉轴的开始位置叫做cross start，结束位置叫做cross end。与Flutter中MainAxisAlignment和CrossAxisAlignment类似，分别代表主轴对齐和纵轴对齐。
源码属性解读
```
  Row({
    .....
    MainAxisAlignment mainAxisAlignment = MainAxisAlignment.start,
    MainAxisSize mainAxisSize = MainAxisSize.max,
    CrossAxisAlignment crossAxisAlignment = CrossAxisAlignment.center,
    TextDirection textDirection,
    VerticalDirection verticalDirection = VerticalDirection.down,
    TextBaseline textBaseline,
    List<Widget> children = const <Widget>[],
  })

  Column({
    .....
    MainAxisAlignment mainAxisAlignment = MainAxisAlignment.start,
    MainAxisSize mainAxisSize = MainAxisSize.max,
    CrossAxisAlignment crossAxisAlignment = CrossAxisAlignment.center,
    TextDirection textDirection,
    VerticalDirection verticalDirection = VerticalDirection.down,
    TextBaseline textBaseline,
    List<Widget> children = const <Widget>[],
  }) 
```
textDirection：表示水平方向子widget的布局顺序(是从左往右还是从右往左)，默认为系统当前Locale环境的文本方向(如中文、英语都是从左往右，而阿拉伯语是从右往左)。
主轴方向: Row即为水平方向，Column为垂直方向
mainAxisAlignment 主轴方向，对child起作用

center：将children放置在主轴的中心
start：将children放置在主轴的起点
end：将children放置在主轴的末尾
spaceAround：将主轴方向上的空白区域均分，使children之间的空白区域相等，但是首尾child的靠边间距为空白区域为1/2
spaceBetween：将主轴方向上的空白区域均分，使children之间的空白区域相等，首尾child靠边没有间隙
spaceEvenly：将主轴方向上的空白区域均分，使得children之间的空白区域相等，包括首尾child


mainAxisSize max表示尽可能占多的控件，min会导致控件聚拢在一起
crossAxisAlignment 交叉轴方向，对child起作用

baseline：使children baseline对齐
center：children在交叉轴上居中展示
end：children在交叉轴上末尾展示
start：children在交叉轴上起点处展示
stretch：让children填满交叉轴方向


verticalDirection ，child的放置顺序

VerticalDirection.down，在Row中就是从左边到右边，Column代表从顶部到底部
VerticalDirection.up，相反
Row
示例代码
```
ListView(
      children: <Widget>[
        Row(
          mainAxisAlignment: MainAxisAlignment.start,
          children: <Widget>[
            Text("我是Row的子控件  "),
            Text("MainAxisAlignment.start")
          ],
        ),
        Row(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text("我是Row的子控件  "),
            Text("MainAxisAlignment.center")
          ],
        ),
        Row(
          mainAxisAlignment: MainAxisAlignment.end,
          children: <Widget>[
            Text("我是Row的子控件  "),
            Text("MainAxisAlignment.end")
          ],
        ),
        Row(
          crossAxisAlignment: CrossAxisAlignment.start,
          verticalDirection: VerticalDirection.up,
          children: <Widget>[
            Text(" Hello World ", style: TextStyle(fontSize: 30.0),),
            Text(" I am Jack "),
          ],
      ],
    )
```
代码运行效果
![](https://upload-images.jianshu.io/upload_images/19956127-07b6a2574cff6039.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
前3个Row很简单，只是设置了主轴方向的对齐方式；第四个Row测试的是纵轴的对齐方式，由于两个子Text字体不一样，所以其高度也不同，我们指定了verticalDirection值为VerticalDirection.up，即从低向顶排列，而此时crossAxisAlignment值为CrossAxisAlignment.start表示底对齐。大家可以参考上面Row和Column的主侧轴的示意图，看看布局是不是正确的，还有很多种情况就不一一列举了。
#### Column
Column组件即垂直布局控件，能够将子组件垂直排列。其实Column组件里边的大部分属性都与Row组件一样。
```
/**
 * 垂直布局
 */
class ColumnWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      crossAxisAlignment: CrossAxisAlignment.center,
      children: <Widget>[
        RaisedButton(onPressed: () {}, color: Colors.red, child: Text('红色按钮')),
        RaisedButton(
          onPressed: () {},
          color: Colors.orange,
          child: Text('黄色大按钮'),
        ),
        RaisedButton(onPressed: () {}, color: Colors.pink, child: Text('粉色按钮'))
      ],
    );
  }
}
```
实现效果如下：
![](https://upload-images.jianshu.io/upload_images/19956127-aa3be0a5a4c9a6ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 实际使用
由于篇幅有限，常见现象和解决办法：

1.如果Row里面嵌套Row，或者Column里面再嵌套Column，那么只有对最外面的Row或Column会占用尽可能大的空间，里面Row或Column所占用的空间为实际大小，如果要让里面的Colum或Row占满外部Colum或Row，可以使用Expanded widget
2.如果使用Column发现超范围，可用SingleChildScrollView包裹，scrollDirection属性设置滑动方向
3.使用Column嵌套ListView/GridView的时候，会报异常信息【Viewports expand in the scrolling direction to fill their container...】，这种情况flutter已给出解决办法，将ListView/GridView的 shrinkWrap属性设为true
4.有的时候修改Row/Column的verticalDirection会得到很好的效果，比如需要页面在底部需要几个按键，也可以用Stack来布局，但是相对麻烦，而且有时还需要知道控件的大小，没有verticalDirection方便。

##### 弹性布局

弹性布局允许子widget按照一定比例来分配父容器空间，弹性布局的概念在其UI系统中也都存在，如H5中的弹性盒子布局，Android中的FlexboxLayout。Flutter中的弹性布局主要通过Flex和Expanded来配合实现。

##### Flex

Flex可以沿着水平或垂直方向排列子widget，如果你知道主轴方向，使用Row或Column会方便一些，因为Row和Column都继承自Flex，参数基本相同，所以能使用Flex的地方一定可以使用Row或Column。Flex本身功能是很强大的，它也可以和Expanded配合实现弹性布局，接下来我们只讨论Flex和弹性布局相关的属性(其它属性已经在介绍Row和Column时介绍过了)。

```
Flex({
  ...
  @required this.direction, //弹性布局的方向, Row默认为水平方向，Column默认为垂直方向
  List<Widget> children = const <Widget>[],
})

```

Flex继承自MultiChildRenderObjectWidget，对应的RenderObject为RenderFlex，RenderFlex中实现了其布局算法。

##### Expanded

可以按比例“扩伸”Row、Column和Flex子widget所占用的空间。

```
const Expanded({
  int flex = 1, 
  @required Widget child,
})

```

flex为弹性系数，如果为0或null，则child是没有弹性的，即不会被扩伸占用的空间。如果大于0，所有的Expanded按照其flex的比例来分割主轴的全部空闲空间。下面我们看一个例子：

```
class _home extends StatefulWidget{
  @override
  State<StatefulWidget> createState() {
    // TODO: implement createState
    return _homeState();
  }
}
class _homeState extends State<_home>{
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return new Scaffold(
      appBar: new AppBar(
        title: new Text("title"),
        centerTitle: true,
      ),
      body: _body(),
    );
  }
}

Widget _body(){
  return Column(
    children: <Widget>[
      //Flex的两个子widget按1：2来占据水平空间
      Flex(
        direction: Axis.horizontal,
        children: <Widget>[
          Expanded(
            flex: 1,
            child: Container(
              height: 80.0,
              color: Colors.red,
            ),
          ),
          Expanded(
            flex: 2,
            child: Container(
              height: 80.0,
              color: Colors.green,
            ),
          ),
        ],
      ),
      Padding(
        padding: const EdgeInsets.only(top: 20.0),
        child: SizedBox(
          height: 100.0,
          //Flex的三个子widget，在垂直方向按2：1：1来占用100像素的空间
          child: Flex(
            direction: Axis.vertical,
            children: <Widget>[
              Expanded(
                flex: 2,
                child: Container(
                  color: Colors.red,
                ),
              ),
              Spacer(//Spacer的功能是占用指定比例的空间，实际上它只是Expanded的一个包装
                flex: 1,
              ),
              Expanded(
                flex: 1,
                child: Container(
                  color: Colors.green,
                ),
              ),
            ],
          ),
        ),
      ),
    ],
  );
}

```

![](//upload-images.jianshu.io/upload_images/5439590-9a46997a39298c73.png?imageMogr2/auto-orient/strip|imageView2/2/w/320/format/webp)

### 流式布局

流式布局（Liquid）的特点（也叫"Fluid") 是页面元素的宽度按照屏幕分辨率进行适配调整，但整体布局不变。栅栏系统（网格系统），用户标签等。在Flutter中主要有Wrap和Flow两种Widget实现。

#### Wrap

在介绍Row和Colum时，如果子widget超出屏幕范围，则会报溢出错误，在Flutter中通过Wrap和Flow来支持流式布局，溢出部分则会自动折行。

##### 源码属性解读

```
Wrap({
  ...
  this.direction = Axis.horizontal,
  this.alignment = WrapAlignment.start,
  this.spacing = 0.0,
  this.runAlignment = WrapAlignment.start,
  this.runSpacing = 0.0,
  this.crossAxisAlignment = WrapCrossAlignment.start,
  this.textDirection,
  this.verticalDirection = VerticalDirection.down,
  List<Widget> children = const <Widget>[],
})
```

上述有很多属性和Row的相同，其意义其实也是相同的，这里我就不一一介绍了，主要介绍下不同的属性：

*   spacing：主轴方向子widget的间距
*   runSpacing：纵轴方向的间距
*   runAlignment：纵轴方向的对齐方式

##### 示例代码

```
Wrap(
   spacing: 10.0,
   direction: Axis.horizontal,
   alignment: WrapAlignment.start,
   children: <Widget>[
     _card('关注'),
     _card('推荐'),
     _card('新时代'),
     _card('小视频'),
     _card('党媒推荐'),
     _card('中国新唱将'),
     _card('历史'),
     _card('视频'),
     _card('游戏'),
     _card('头条号'),
     _card('数码'),
   ],
 )

  Widget _card(String title) {
    return Card(child: Text(title),);
  }
}
复制代码
```

##### 运行效果

![](https://upload-images.jianshu.io/upload_images/19956127-bcfc5a7ba8cea809.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 小结

*   使用Wrap可以很轻松的实现流式布局效果
*   Wrap支持设置流式布局是纵向显示或者是横向显示
*   可以使用alignment属性来控制widgets的布局方式

#### Flow

我们一般很少会使用Flow，因为其过于复杂，需要自己实现子widget的位置转换，在很多场景下首先要考虑的是Wrap是否满足需求。Flow主要用于一些需要自定义布局策略或性能要求较高(如动画中)的场景。Flow有如下优点：

*   性能好；Flow是一个对child尺寸以及位置调整非常高效的控件，Flow用转换矩阵（transformation matrices）在对child进行位置调整的时候进行了优化：在Flow定位过后，如果child的尺寸或者位置发生了变化，在FlowDelegate中的paintChildren()方法中调用context.paintChild 进行重绘，而context.paintChild在重绘时使用了转换矩阵（transformation matrices），并没有实际调整Widget位置。
*   灵活；由于我们需要自己实现FlowDelegate的paintChildren()方法，所以我们需要自己计算每一个widget的位置，因此，可以自定义布局策略。 缺点：
*   使用复杂.
*   不能自适应子widget大小，必须通过指定父容器大小或实现TestFlowDelegate的getSize返回固定大小。

##### 示例代码

我们对六个色块进行自定义流式布局：

```
Flow(
  delegate: TestFlowDelegate(margin: EdgeInsets.all(10.0)),
  children: <Widget>[
    new Container(width: 80.0, height:80.0, color: Colors.red,),
    new Container(width: 80.0, height:80.0, color: Colors.green,),
    new Container(width: 80.0, height:80.0, color: Colors.blue,),
    new Container(width: 80.0, height:80.0,  color: Colors.yellow,),
    new Container(width: 80.0, height:80.0, color: Colors.brown,),
    new Container(width: 80.0, height:80.0,  color: Colors.purple,),
  ],
)
```

实现TestFlowDelegate:

```
class TestFlowDelegate extends FlowDelegate {
  EdgeInsets margin = EdgeInsets.zero;
  TestFlowDelegate({this.margin});
  @override
  void paintChildren(FlowPaintingContext context) {
    var x = margin.left;
    var y = margin.top;
    //计算每一个子widget的位置  
    for (int i = 0; i < context.childCount; i++) {
      var w = context.getChildSize(i).width + x + margin.right;
      if (w < context.size.width) {
        context.paintChild(i,
            transform: new Matrix4.translationValues(
                x, y, 0.0));
        x = w + margin.left;
      } else {
        x = margin.left;
        y += context.getChildSize(i).height + margin.top + margin.bottom;
        //绘制子widget(有优化)  
        context.paintChild(i,
            transform: new Matrix4.translationValues(
                x, y, 0.0));
        x += context.getChildSize(i).width + margin.left + margin.right;
      }
    }
  }

  getSize(BoxConstraints constraints){
    //指定Flow的大小  
    return Size(double.infinity,200.0);
  }

  @override
  bool shouldRepaint(FlowDelegate oldDelegate) {
    return oldDelegate != this;
  }
}
```

##### 运行效果

![](https://upload-images.jianshu.io/upload_images/19956127-153455cfc7e2c196.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到我们主要的任务就是实现paintChildren，它的主要任务是确定每个子widget位置。由于Flow不能自适应子widget的大小，我们通过在getSize返回一个固定大小来指定Flow的大小，实现起来还是比较麻烦的。

##### 小结

*   参数简单，不过需要自己定义delegate
*   delegate一般是为了实现child的绘制，就是位置的摆放，不同情况需要定义不同的delegate
*   不同的delegate一般会提供实现的几个方法:
    *   getConstraintsForChild: 设置每个child的布局约束条件，会覆盖已有的方式
    *   getSize：设置控件的尺寸
    *   shouldRelayout：表示是否需要重新布局
*   尽可能的用Wrap，毕竟简单
### 层叠布局

层叠布局和Web中的绝对定位、Android中的Frame布局是相似的，子widget可以根据到父容器四个角的位置来确定本身的位置。绝对定位允许子widget堆叠（按照代码中声明的顺序）。Flutter中使用Stack和Positioned来实现绝对定位，Stack允许子widget堆叠，而Positioned可以给子widget定位（根据Stack的四个角）。

#### Stack

```
Stack({
  this.alignment = AlignmentDirectional.topStart,
  this.textDirection,
  this.fit = StackFit.loose,
  this.overflow = Overflow.clip,
  List<Widget> children = const <Widget>[],
})
```

*   alignment：此参数决定如何去对齐没有定位（没有使用Positioned）或部分定位的子widget。所谓部分定位，在这里特指没有在某一个轴上定位：left、right为横轴，top、bottom为纵轴，只要包含某个轴上的一个定位属性就算在该轴上有定位。
*   textDirection：和Row、Wrap的textDirection功能一样，都用于决定alignment对齐的参考系即：textDirection的值为TextDirection.ltr，则alignment的start代表左，end代表右；textDirection的值为TextDirection.rtl，则alignment的start代表右，end代表左。
*   fit：此参数用于决定没有定位的子widget如何去适应Stack的大小。StackFit.loose表示使用子widget的大小，StackFit.expand表示扩伸到Stack的大小。
*   overflow：此属性决定如何显示超出Stack显示空间的子widget，值为Overflow.clip时，超出部分会被剪裁（隐藏），值为Overflow.visible 时则不会。

##### 下面是我用Stack实现的一个简易的loading

```
class Loading extends StatelessWidget {
  /// ProgressIndicator的padding，决定loading的大小
  final EdgeInsets padding = EdgeInsets.all(30.0);

  /// 文字顶部距菊花的底部的距离
  final double margin = 10.0;

  /// 圆角
  final double cornerRadius = 10.0;

  final Widget _child;
  final bool _isLoading;
  final double opacity;
  final Color color;
  final String text;

  Loading({
    Key key,
    @required child,
    @required isLoading,
    this.text,
    this.opacity = 0.3,
    this.color = Colors.grey,
  })  : assert(child != null),
        assert(isLoading != null),
        _child = child,
        _isLoading = isLoading,
        super(key: key);

  @override
  Widget build(BuildContext context) {
    List<Widget> widgetList = List<Widget>();
    widgetList.add(_child);
    if (_isLoading) {
      final loading = [
        Opacity(
          opacity: opacity,
          child: ModalBarrier(dismissible: false, color: color),
        ),
        _buildProgressIndicator()
      ];
      widgetList.addAll(loading);
    }
    return Stack(
      children: widgetList,
    );
  }

  Widget _buildProgressIndicator() {
    return Center(
      child: Container(
        padding: padding,
        child: Column(
          mainAxisSize: MainAxisSize.min,
          crossAxisAlignment: CrossAxisAlignment.center,
          children: <Widget>[
            CupertinoActivityIndicator(),
            Padding(
                padding: EdgeInsets.only(top: margin),
                child: Text(text ?? '加载中...')),
          ],
        ),
        decoration: BoxDecoration(
            borderRadius: BorderRadius.all(Radius.circular(cornerRadius)),
            color: Colors.white),
      ),
    );
  }
}

```

##### 显示效果

![](https://upload-images.jianshu.io/upload_images/19956127-a4d546cf7efaa348.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

本控件使用Stack封装，你传入的主视图在最下面一层，背景层在中间，最上面一层为菊花和文字loading，用isLoading控制显示

### Positioned

```
const Positioned({
  Key key,
  this.left, 
  this.top,
  this.right,
  this.bottom,
  this.width,
  this.height,
  @required Widget child,
})
```

left、top 、right、 bottom分别代表离Stack左、上、右、底四边的距离。width和height用于指定定位元素的宽度和高度，注意，此处的width、height 和其它地方的意义稍微有点区别，此处用于配合left、top 、right、 bottom来定位widget，举个例子，在水平方向时，你只能指定left、right、width三个属性中的两个，如指定left和width后，right会自动算出(left+width)，如果同时指定三个属性则会报错，垂直方向同理。

##### 示例代码

```
//通过ConstrainedBox来确保Stack占满屏幕
ConstrainedBox(
  constraints: BoxConstraints.expand(),
  child: Stack(
    alignment:Alignment.center , //指定未定位或部分定位widget的对齐方式
    children: <Widget>[
      Container(child: Text("Hello world",style: TextStyle(color: Colors.white)),
        color: Colors.red,
      ),
      Positioned(
        left: 18.0,
        child: Text("I am Jack"),
      ),
      Positioned(
        top: 18.0,
        child: Text("Your friend"),
      )        
    ],
  ),
);
```

##### 运行效果:

![](https://upload-images.jianshu.io/upload_images/19956127-0c3e722e254ea33d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由于第一个子widget Text("Hello world")没有指定定位，并且alignment值为Alignment.center，所以，它会居中显示。第二个子widget Text("I am Jack")只指定了水平方向的定位(left)，所以属于部分定位，即垂直方向上没有定位，那么它在垂直方向对齐方式则会按照alignment指定的对齐方式对齐，即垂直方向居中。对于第三个子widget Text("Your friend")，和第二个Text原理一样，只不过是水平方向没有定位，则水平方向居中。

参考：
https://juejin.im/post/5c2458d6f265da613a541349
https://my.oschina.net/mchp/blog/3123764
https://www.jianshu.com/p/5c2654422486
阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)










