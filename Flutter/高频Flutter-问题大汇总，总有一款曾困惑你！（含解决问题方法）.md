## 如何实现Android平台的wrap_content 和match_parent

你可以按照如下方式实现：

1、Width = Wrap_content Height=Wrap_content：

```
Wrap(
  children: <Widget>[your_child])
```

2、Width = Match_parent Height=Match_parent：

```
Container(
        height: double.infinity,
    width: double.infinity,child:your_child)复制代码
```

3、Width = Match_parent ,Height = Wrap_conten：

```
Row(
  mainAxisSize: MainAxisSize.max,
  children: <Widget>[*your_child*],
);
```

4、Width = Wrap_content ,Height = Match_parent：

```
Column(
  mainAxisSize: MainAxisSize.max,
  children: <Widget>[your_child],
);
```

## 如何避免FutureBuilder频繁执行`future`方法

错误用法：

```
@override
Widget build(BuildContext context) {
  return FutureBuilder(
    future: httpCall(),
    builder: (context, snapshot) {

    },
  );
}
```

正确用法：

```
class _ExampleState extends State<Example> {
  Future<int> future;

  @override
  void initState() {
    future = Future.value(42);
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return FutureBuilder(
      future: future,
      builder: (context, snapshot) {

      },
    );
  }
}
```

## 底部导航切换导致重建问题

在使用底部导航时经常会使用如下写法：

```
Widget _currentBody;

@override
Widget build(BuildContext context) {
  return Scaffold(
    body: _currentBody,
    bottomNavigationBar: BottomNavigationBar(
      items: <BottomNavigationBarItem>[
          ...
      ],
      onTap: (index) {
        _bottomNavigationChange(index);
      },
    ),
  );
}

_bottomNavigationChange(int index) {
  switch (index) {
    case 0:
      _currentBody = OnePage();
      break;
    case 1:
      _currentBody = TwoPage();
      break;
    case 2:
      _currentBody = ThreePage();
      break;
  }
  setState(() {});
}
```

此用法导致每次切换时都会重建页面。

解决办法，使用`IndexedStack`：

```
int _currIndex;

@override
Widget build(BuildContext context) {
  return Scaffold(
    body: IndexedStack(
        index: _currIndex,
        children: <Widget>[OnePage(), TwoPage(), ThreePage()],
      ),
    bottomNavigationBar: BottomNavigationBar(
      items: <BottomNavigationBarItem>[
          ...
      ],
      onTap: (index) {
        _bottomNavigationChange(index);
      },
    ),
  );
}

_bottomNavigationChange(int index) {
  setState(() {
      _currIndex = index;
    });
}
```

## TabBar切换导致重建（build）问题

通常情况下，使用TabBarView如下：

```
TabBarView(
  controller: this._tabController,
  children: <Widget>[
    _buildTabView1(),
    _buildTabView2(),
  ],
)
```

此时切换tab时，页面会重建，解决方法设置`PageStorageKey`：

```
var _newsKey = PageStorageKey('news');
var _technologyKey = PageStorageKey('technology');

TabBarView(
  controller: this._tabController,
  children: <Widget>[
    _buildTabView1(_newsKey),
    _buildTabView2(_technologyKey),
  ],
)
```

## Stack 子组件设置了宽高不起作用

在Stack中设置100x100红色盒子，如下：

```
Center(
  child: Container(
    height: 300,
    width: 300,
    color: Colors.blue,
    child: Stack(
      children: <Widget>[
        Positioned.fill(
          child: Container(
            height: 100,
            width: 100,
            color: Colors.red,
          ),
        )
      ],
    ),
  ),
)
```

此时红色盒子充满父组件，解决办法，给红色盒子组件包裹Center、Align或者UnconstrainedBox，代码如下：

```
Positioned.fill(
  child: Align(
    child: Container(
      height: 100,
      width: 100,
      color: Colors.red,
    ),
  ),
)
```

## 如何在State类中获取StatefulWidget控件的属性

```
class Test extends StatefulWidget {
  Test({this.data});
  final int data;
  @override
  State<StatefulWidget> createState() => _TestState();
}

class _TestState extends State<Test>{

}
```

如下，如何在_TestState获取到Test的`data`数据呢：

1.  在_TestState也定义同样的参数，此方式比较麻烦，不推荐。
2.  直接使用`widget.data`（推荐）。

## default value of optional parameter must be constant

上面的异常在类构造函数的时候会经常遇见，如下面的代码就会出现此异常：

```
class BarrageItem extends StatefulWidget {
  BarrageItem(
      { this.text,
      this.duration = Duration(seconds: 3)});
```

异常信息提示：可选参数必须为常量，修改如下：

```
const Duration _kDuration = Duration(seconds: 3);

class BarrageItem extends StatefulWidget {
  BarrageItem(
      {this.text,
      this.duration = _kDuration});
```

定义一个常量，`Dart`中常量通常使用`k`开头，`_`表示私有，只能在当前包内使用，别问我为什么如此命名，问就是源代码中就是如此命名的。

## 如何移除debug模式下右上角“DEBUG”标识

```
MaterialApp(
 debugShowCheckedModeBanner: false
)
```

## 如何使用16进制的颜色值

下面的用法是无法显示颜色的：

```
Color(0xb74093)
```

因为Color的构造函数是`ARGB`，所以需要加上透明度，正确用法：

```
Color(0xFFb74093)
```

`FF`表示完全不透明。

## 如何改变应用程序的icon和名称

链接：[blog.csdn.net/mengks1987/…](https://blog.csdn.net/mengks1987/article/details/95306508)

## 如何给TextField设置初始值

```
class _FooState extends State<Foo> {
  TextEditingController _controller;

  @override
  void initState() {
    super.initState();
    _controller = new TextEditingController(text: '初始值');
  }

  @override
  Widget build(BuildContext context) {
    return TextField(
          controller: _controller,
        );
  }
}
```

## Scaffold.of() called with a context that does not contain a Scaffold

Scaffold.of()中的context没有包含在Scaffold中，如下代码就会报此异常：

```
class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('老孟'),
      ),
      body: Center(
        child: RaisedButton(
          color: Colors.pink,
          textColor: Colors.white,
          onPressed: _displaySnackBar(context),
          child: Text('show SnackBar'),
        ),
      ),
    );
  }
}

_displaySnackBar(BuildContext context) {
  final snackBar = SnackBar(content: Text('老孟'));
  Scaffold.of(context).showSnackBar(snackBar);
}
```

注意此时的context是HomePage的，HomePage并没有包含在Scaffold中，所以并不是调用在Scaffold中就可以，而是看context，修改如下：

```
_scaffoldKey.currentState.showSnackBar(snackbar); 
```

或者：

```
Scaffold(
    appBar: AppBar(
        title: Text('老孟'),
    ),
    body: Builder(
        builder: (context) => 
            Center(
            child: RaisedButton(
            color: Colors.pink,
            textColor: Colors.white,
            onPressed: () => _displaySnackBar(context),
            child: Text('老孟'),
            ),
        ),
    ),
);
```

## Waiting for another flutter command to release the startup lock

在执行`flutter`命令时经常遇到上面的问题，

解决办法一：

1、Mac或者Linux在终端执行如下命令：

```
killall -9 dart
```

2、Window执行如下命令：

```
taskkill /F /IM dart.exe
```

解决办法二：

删除flutter SDK的目录下`/bin/cache/lockfile`文件。

## 无法调用`setState`

不能在StatelessWidget控件中调用了，需要在StatefulWidget中调用。

## 设置当前控件大小为父控件大小的百分比

1、使用`FractionallySizedBox`控件

2、获取父控件的大小并乘以百分比：

```
MediaQuery.of(context).size.width * 0.5
```

## Row直接包裹TextField异常：BoxConstraints forces an infinite width

解决方法：

```
Row(
    children: <Widget>[
        Flexible(
            child: new TextField(),
        ),
  ],
),
```

## TextField 动态获取焦点和失去焦点

获取焦点：

```
FocusScope.of(context).requestFocus(_focusNode);复制代码
```

`_focusNode`为TextField的focusNode：

```
_focusNode = FocusNode();

TextField(
    focusNode: _focusNode,
    ...
)
```

失去焦点：

```
_focusNode.unfocus();
```

## 如何判断当前平台

```
import 'dart:io' show Platform;

if (Platform.isAndroid) {
  // Android-specific code
} else if (Platform.isIOS) {
  // iOS-specific code
}
```

平台类型包括：

```
Platform.isAndroid
Platform.isFuchsia
Platform.isIOS
Platform.isLinux
Platform.isMacOS
Platform.isWindows
```

## Android无法访问http

其实这本身不是Flutter的问题，但在开发中经常遇到，在Android Pie版本及以上和IOS 系统上默认禁止访问http，主要是为了安全考虑。

Android解决办法：

在`./android/app/src/main/AndroidManifest.xml`配置文件中application标签里面设置networkSecurityConfig属性:

```
<?xml version="1.0" encoding="utf-8"?>
<manifest ... >
    <application android:networkSecurityConfig="@xml/network_security_config">
         <!-- ... -->
    </application>
</manifest>
```

在`./android/app/src/main/res`目录下创建xml文件夹（已存在不用创建），在xml文件夹下创建network_security_config.xml文件，内容如下：

```
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="true">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
</network-security-config>
```

## IOS无法访问http

在`./ios/Runner/Info.plist`文件中添加如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    ...
    <key>NSAppTransportSecurity</key>
    <dict>
        <key>NSAllowsArbitraryLoads</key>
        <true/>
    </dict>
</dict>
</plist>
```

作者：老孟程序员
链接：https://juejin.im/post/5e984f26e51d45470a4ac7b0

**推荐阅读：[2017-2020历年字节跳动Android面试真题解析（累计下载1082万次，持续更新中）](https://www.jianshu.com/p/7f9ade51232e)**

