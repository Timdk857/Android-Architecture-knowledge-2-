本文通过改造flutter的counter app, 展示不同的状态管理方法的用法.

可以直接去demo地址看代码:
https://github.com/mengdd/counter_state_management
切换分支对应不同的实现方式.

## Contents

*   Flutter中的状态管理
    *   状态分类
    *   状态管理方法概述
*   Counter sample默认实现: StatefulWidget
*   InheritedWidget
*   Scoped Model
*   Provider
*   BLoC
    *   BLoC手动实现
    *   BLoC + InheritedWidget做传递
    *   BLoC rxdart实现
    *   BLoC用库实现
*   rxdart
*   Redux
*   MobX
*   Flutter Hooks
*   Demo说明及感想

## Flutter State Management

Flutter是描述性的(declarative), UI反映状态.

```
UI = f(state)
```

其中`f`代表了build方法.

状态的改变会直接触发UI的重新绘制.

UI reacts to the changes.

相对的, Android, iOS等都是命令式的(imperative), 会有`setText()`之类的方法来改变UI.

### 状态分类

状态分两种:

*   Ephemeral state: 有时也叫UI state或local state. 这种可以包含在单个widget里.
    比如: `PageView`的当前页, 动画的当前进度, `BottomNavigationBar`的当前选中tab.
    这种状态不需要使用复杂的状态管理手段, 只要用一个`StatefulWidget`就可以了.
*   App state: 需要在很多地方共享的状态, 也叫shared state或global state.
    比如: 用户设置, 登录信息, 通知, 购物车, 新闻app中的已读/未读状态等.

这种状态分类其实没有一个清晰的界限.
在简单的app里, 可以用`setState()`来管理所有的状态; 在app需要的时候, tab的index也可能被抽取到外部作为一个需要保存和管理的app state.

### 状态管理方法

官方提供了一些options: [Flutter官方文档 options](https://flutter.dev/docs/development/data-and-backend/state-mgmt/options)
目前官方比较推荐的是provider.

各种状态管理方法要解决的几个问题:

*   状态保存哪里?
*   状态如何获取?
*   UI如何更新?
*   如何改变状态?

## Counter Sample默认实现: StatefulWidget

新建Flutter app, 是一个counter app, 自动使用了`StatefulWidget`来管理状态.
对这个简单的app来说, 这是很合理的.

我们对这个app进行一个简单的改造, 再增加一个button用来减数字.
同样的方式, 只需要添加一个方法来做减法就可以了.

这种方法的一个变体是, 用`StatefulBuilder`, 主要好处是少写一些代码.

`StatefulWidget`对简单的Widget内部状态来说是合理的.

对于复杂的状态, 这种方式的缺点:

*   状态属性多了以后, 可能有很多地方都在调用`setState()`.
*   不能把状态和UI分开管理.
*   不利于跨组件/跨页面的状态共享. (如何调用另一个Widget的`setState()`? 把方法通过构造传递过来? No, don't do this!)

千万不要用全局变量法来解决问题.

如果企图用这种方式来管理跨组件的状态, 就难免会用这些Anti patterns:

*   紧耦合. Strongly coupling widgets.
*   全局保存的state. Globally tracking state.
*   从外部调用setState方法. Calling setState from outside.

所以这种方法只适用于local state的管理.

*   代码分支1: [starter-code](https://github.com/mengdd/counter_state_management/blob/starter-code/lib/main.dart).
*   代码分支2: [stateful-builder](https://github.com/mengdd/counter_state_management/tree/stateful-builder/lib/main.dart).

## InheritedWidget

[InheritedWidget](https://api.flutter.dev/flutter/widgets/InheritedWidget-class.html)的主要作用是在Widget树中有效地传递信息.

如果没有`InheritedWidget`, 我们想把一个数据从widget树的上层传到某一个child widget, 要利用途中的每一个构造函数, 一路传递下来.

Flutter中常用的`Theme`, `Style`, `MediaQuery`等就是inherited widget, 所以在程序里的各种地方都可以访问到它们.

`InheritedWidget`也会用在其他状态管理模式中, 作为传递数据的方法.

### InheritedWidget状态管理实现

当用`InheritedWidget`做状态管理时, 基本思想就是把状态提上去.
当子widgets之间需要共享状态, 那么就把状态保存在它们共有的parent中.

首先定义一个`InheritedWidget`的子类, 包含状态数据.
覆写两个方法:

*   提供一个静态方法给child用于获取自己. (命名惯例`of(BuildContext)`).
*   判断是否发生了数据更新.

```
class CounterStateContainer extends InheritedWidget {
  final CounterModel data;

  CounterStateContainer({
    Key key,
    @required Widget child,
    @required this.data,
  }) : super(key: key, child: child);

  @override
  bool updateShouldNotify(CounterStateContainer oldWidget) {
    return data.counter.value != oldWidget.data.counter.value;
  }

  static CounterModel of(BuildContext context) {
    return context
        .dependOnInheritedWidgetOfExactType<CounterStateContainer>()
        .data;
  }
}
```

之后用这个`CounterStateContainer`放在上层, 包含了数据和所有状态相关的widgets.
child widget不论在哪一层都可以方便地获取到状态数据.

```
  Text(
    '${CounterStateContainer.of(context).counter.value}',
  ),
```

代码分支: [inherited-widget](https://github.com/mengdd/counter_state_management/tree/inherited-widget).

### InheritedWidget缺点

`InheritedWidget`解决了访问状态和根据状态更新的问题, 但是改变state却不太行.

*   accessing state
*   updating on change
*   mutating state -> X

首先, 不支持跨页面(route)的状态, 因为widget树变了, 所以需要进行跨页面的数据传递.

其次, `InheritedWidget`它包含的数据是不可变的, 如果想让它跟踪变化的数据:

*   把它包在一个`StatefulWidget`里.
*   在`InheritedWidget`中使用`ValueNotifier`, `ChangeNotifier`或steams.

这个方案也是了解一下, 实际的全局状态管理还是用更成熟的方案.
但是它的原理会被用到其他方案中作为对象传递的方式.

## Scoped Model

scoped model是一个外部package: https://pub.dev/packages/scoped_model
Scoped Model是基于`InheritedWidget`的. 思想仍然是把状态提到上层去, 并且封装了状态改变的通知部分.

### Scoped Model实现

它官方提供例子就是改造counter: https://pub.dev/packages/scoped_model#-example-tab-

*   添加scoped_model依赖.
*   创建数据类, 继承`Model`.

```
import 'package:scoped_model/scoped_model.dart';

class CounterModel extends Model {
  int _counter = 0;

  int get counter => _counter;

  void increment() {
    _counter++;
    notifyListeners();
  }

  void decrement() {
    _counter--;
    notifyListeners();
  }
}
```

其中数据变化的部分会通知listeners, 它们收到通知后会rebuild.

在上层初始化并提供数据类, 用`ScopeModel`.

访问数据有两种方法:

*   用`ScopedModelDescendant`包裹widget.
*   用`ScopedModel.of`静态方法.

使用的时候注意要提供泛型类型, 会帮助我们找到离得最近的上层`ScopedModel`.

```
  ScopedModelDescendant<CounterModel>(
      builder: (context, child, model) {
    return Text(
      model.counter.toString(),
    );
  }),
```

数据改变后, 只有`ScopedModelDescendant`会收到通知, 从而rebuild.

`ScopedModelDescendant`有一个`rebuildOnChange`属性, 这个值默认是true.
对于button来说, 它只是控制改变, 自身并不需要重绘, 可以把这个属性置为false.

```
  ScopedModelDescendant<CounterModel>(
    rebuildOnChange: false,
    builder: (context, child, model) {
      return FloatingActionButton(
        onPressed: model.increment,
        tooltip: 'Increment',
        child: Icon(Icons.add),
      );
    },
  ),
```

scoped model这个库帮我们解决了数据访问和通知的问题, 但是rebuild范围需要自己控制.

*   access state
*   notify other widgets
*   minimal rebuild -> X -> 因为需要开发者自己来决定哪一部分是否需要被重建, 容易被忘记.

代码分支: [scoped-model](https://github.com/mengdd/counter_state_management/tree/scoped-model)

## Provider

Provider是官方文档的例子用的方法.
去年的Google I/O 2019也推荐了这个方法.
和BLoC的流式思想相比, Provider是一个观察者模式, 状态改变时要`notifyListeners()`.

有一个counter版本的sample: https://github.com/flutter/samples/tree/master/provider_counter

Provider的实现在内部还是利用了`InheritedWidget`.
Provider的好处: dispose指定后会自动被调用, 支持`MultiProvider`.

### Provider实现

*   model类继承`ChangeNotifer`, 也可以用`with`.

```
class CounterModel extends ChangeNotifier {
  int value = 0;

  void increment() {
    value++;
    notifyListeners();
  }

  void decrement() {
    value--;
    notifyListeners();
  }
}
```

*   数据提供者: `ChangeNotifierProvider`.

```
void main() => runApp(ChangeNotifierProvider(
      create: (context) => CounterModel(),
      child: MyApp(),
    ));
```

*   数据消费者/操纵者, 有两种方式: `Consumer`包裹, 用`Provider.of`.

```
  Consumer<CounterModel>(
    builder: (context, counter, child) => Text(
      '${counter.value}',
    ),
  ),
```

FAB:

```
  FloatingActionButton(
    onPressed: () =>
        Provider.of<CounterModel>(context, listen: false).increment(),
  ),
```

这里`listen`置为false表明状态变化时并不需要rebuild FAB widget.

### Provider性能相关的实现细节

*   `Consumer`包裹的范围要尽量小.
*   listen变量.
*   child的处理. `Consumer`中builder方法的第三个参数.

可以用于缓存一些并不需要重建的widget:

```
return Consumer<CartModel>(
  builder: (context, cart, child) => Stack(
        children: [
          // Use SomeExpensiveWidget here, without rebuilding every time.
          child,
          Text("Total price: ${cart.totalPrice}"),
        ],
      ),
  // Build the expensive widget here.
  child: SomeExpensiveWidget(),
);
```

代码分支: [provider](https://github.com/mengdd/counter_state_management/tree/provider).

## BLoC

BLoC模式的全称是: business logic component.

所有的交互都是a stream of asynchronous events.
`Widgets + Streams = Reactive`.

BLoC的实现的主要思路: Events in -> BloC -> State out.

Google I/O 2018上推荐的还是这个, 2019就推荐Provider了.
当然也不是说这个模式不好, 架构模式本来也没有对错之分, 只是技术选型不同.

### BLoC手动实现

不添加任何依赖可以手动实现BLoC, 利用:

*   Dart SDK > dart:async > `Stream`.
*   Flutter的`StreamBuilder`: 输入是一个stream, 有一个builder方法, 每次stream中有新值, 就会rebuild.

可以有多个stream, UI只在自己感兴趣的信息发生变化的时候重建.

BLoC中:

*   输入事件: `Sink<Event> input`.
*   输出数据: `Stream<Data> output`.

CounterBloc类:

```
class CounterBloc {
  int _counter = 0;

  final _counterStateController = StreamController<int>();

  StreamSink<int> get _inCounter => _counterStateController.sink;

  Stream<int> get counter => _counterStateController.stream;

  final _counterEventController = StreamController<CounterEvent>();

  Sink<CounterEvent> get counterEventSink => _counterEventController.sink;

  CounterBloc() {
    _counterEventController.stream.listen(_mapEventToState);
  }

  void _mapEventToState(CounterEvent event) {
    if (event is IncrementEvent) {
      _counter++;
    } else if (event is DecrementEvent) {
      _counter--;
    }
    _inCounter.add(_counter);
  }

  void dispose() {
    _counterStateController.close();
    _counterEventController.close();
  }
}
```

有两个`StreamController`, 一个控制state, 一个控制event.

读取状态值要用`StreamBuilder`:

```
  StreamBuilder(
    stream: _bloc.counter,
    initialData: 0,
    builder: (BuildContext context, AsyncSnapshot<int> snapshot) {
      return Text(
        '${snapshot.data}',
      );
    },
  )
```

而改变状态是发送事件:

```
  FloatingActionButton(
    onPressed: () => _bloc.counterEventSink.add(IncrementEvent()),
  ),
```

实现细节:

*   每个屏幕有自己的BLoC.
*   每个BLoC必须有自己的`dispose()`方法. -> BLoC必须和`StatefulWidget`一起使用, 利用其生命周期释放.

代码分支: [bloc](https://github.com/mengdd/counter_state_management/tree/bloc)

### BLoC传递: 用InheritedWidget

手动实现的BLoC模式, 可以结合`InheritedWidget`, 写一个Provider, 用来做BLoC的传递.

代码分支: [bloc-with-provider](https://github.com/mengdd/counter_state_management/tree/bloc-with-provider)

### BLoC rxdart实现

用了rxdart package之后, bloc模块的实现可以这样写:

```
class CounterBloc {
  int _counter = 0;

  final _counterSubject = BehaviorSubject<int>();

  Stream<int> get counter => _counterSubject.stream;

  final _counterEventController = StreamController<CounterEvent>();

  Sink<CounterEvent> get counterEventSink => _counterEventController.sink;

  CounterBloc() {
    _counterEventController.stream.listen(_mapEventToState);
  }

  void _mapEventToState(CounterEvent event) {
    if (event is IncrementEvent) {
      _counter++;
    } else if (event is DecrementEvent) {
      _counter--;
    }
    _counterSubject.add(_counter);
  }

  void dispose() {
    _counterSubject.close();
    _counterEventController.close();
  }
}
```

`BehaviorSubject`也是一种`StreamController`, 它会记住自己最新的值, 每次注册监听, 会立即给你最新的值.

代码分支: [bloc-rxdart](https://github.com/mengdd/counter_state_management/tree/bloc-rxdart).

### BLoC Library

可以用这个package来帮我们简化代码: https://pub.dev/packages/flutter_bloc

自己只需要定义Event和State的类型并传入, 再写一个逻辑转化的方法:

```
class CounterBloc extends Bloc<CounterEvent, CounterState> {
  @override
  CounterState get initialState => CounterState.initial();

  @override
  Stream<CounterState> mapEventToState(CounterEvent event) async* {
    if (event is IncrementEvent) {
      yield CounterState(counter: state.counter + 1);
    } else if (event is DecrementEvent) {
      yield CounterState(counter: state.counter - 1);
    }
  }
}
```

用`BlocProvider`来做bloc的传递, 从而不用在构造函数中一传到底.

访问的时候用`BlocBuilder`或`BlocProvider.of<CounterBloc>(context)`.

```
  BlocBuilder(
    bloc: BlocProvider.of<CounterBloc>(context),
    builder: (BuildContext context, CounterState state) {
      return Text(
        '${state.counter}',
      );
    },
  ),
```

这里bloc参数如果没有指定, 会自动向上寻找.

`BlocBuilder`有一个参数`condition`, 是一个返回bool的函数, 用来精细控制是否需要rebuild.

```
  FloatingActionButton(
    onPressed: () =>
        BlocProvider.of<CounterBloc>(context).add(IncrementEvent()),
  ),
```

代码分支: [bloc-library](https://github.com/mengdd/counter_state_management/tree/bloc-library).

## rxdart

这是个原始版本的流式处理.

和BLoC相比, 没有专门的逻辑模块, 只是改变了数据的形式.

利用rxdart, 把数据做成流:

```
class CounterModel {
  BehaviorSubject _counter = BehaviorSubject.seeded(0);

  get stream$ => _counter.stream;

  int get current => _counter.value;

  increment() {
    _counter.add(current + 1);
  }

  decrement() {
    _counter.add(current - 1);
  }
}
```

获取数据用`StreamBuilder`, 包围的范围尽量小.

```
    StreamBuilder(
      stream: counterModel.stream$,
      builder: (BuildContext context, AsyncSnapshot snapshot) {
        return Text(
          '${snapshot.data}',
        );
      },
    ),
```

Widget dispose的时候会自动解绑.

数据传递的部分还需要进一步处理.

代码分支: [rxdart](https://github.com/mengdd/counter_state_management/tree/rxdart).

## Redux

Redux是前端流行的, 一种单向数据流架构.

概念:

*   `Store`: 用于存储`State`对象, 代表整个应用的状态.
*   `Action`: 事件操作.
*   `Reducer`: 用于处理和分发事件的方法, 根据收到的`Action`, 用一个新的`State`来更新`Store`.
*   `View`: 每次Store接到新的State, `View`就会重建.

`Reducer`是唯一的逻辑处理部分, 它的输入是当前`State`和`Action`, 输出是一个新的`State`.

### Flutter Redux状态管理实现

首先定义好action, state:

```
enum Actions {
  Increment,
  Decrement,
}

class CounterState {
  int _counter;

  int get counter => _counter;

  CounterState(this._counter);
}

```

reducer方法根据action和当前state产生新的state:

```
CounterState reducer(CounterState prev, dynamic action) {
  if (action == Actions.Increment) {
    return new CounterState(prev.counter + 1);
  } else if (action == Actions.Decrement) {
    return new CounterState(prev.counter - 1);
  } else {
    return prev;
  }
}

```

*   数据提供者: `StoreProvider`.
    放在上层:

```
   StoreProvider(
    store: store,
    child: MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(title: 'Flutter Demo Home Page'),
    ),
  );
```

*   数据消费者: `StoreConnector`, 可读可写.

读状态:

```
    StoreConnector<CounterState, String>(
      converter: (store) => store.state.counter.toString(),
      builder: (context, count) {
        return Text(
          '$count',
        );
      },
    )
```

改变状态: 发送action:

```
    StoreConnector<CounterState, VoidCallback>(
      converter: (store) {
        return () => store.dispatch(action.Actions.Increment);
      },
      builder: (context, callback) {
        return FloatingActionButton(
          onPressed: callback,
        );
      },
    ),
```

代码分支: [redux](https://github.com/mengdd/counter_state_management/tree/redux).

## MobX

MobX本来是一个JavaScript的状态管理库, 它迁移到dart的版本: [mobxjs/mobx.dart](https://github.com/mobxjs/mobx.dart).

核心概念:

*   Observables
*   Actions
*   Reactions

### MobX状态管理实现

官网提供了一个counter的指导: https://mobx.netlify.com/getting-started

这个库的实现需要先生成一些代码.
先写类:

```
import 'package:mobx/mobx.dart';

part 'counter.g.dart';

class Counter = _Counter with _$Counter;

abstract class _Counter with Store {
  @observable
  int value = 0;

  @action
  void increment() {
    value++;
  }

  @action
  void decrement() {
    value--;
  }
}
```

运行命令`flutter packages pub run build_runner build`, 生成`counter.g.dart`.

改完之后就不需要再使用`StatefulWidget`了.

找一个合适的地方初始化数据对象并保存:

```
final counter = Counter();
```

读取值的地方用`Observer`包裹:

```
Observer(
  builder: (_) => Text(
    '${counter.value}',
    style: Theme.of(context).textTheme.display1,
  ),
),
```

改变值的地方:

```
  FloatingActionButton(
    onPressed: counter.increment,
    tooltip: 'Increment',
    child: Icon(Icons.add),
  ),
```

代码分支: [mobx](https://github.com/mengdd/counter_state_management/tree/mobx).

## Flutter hooks

React hooks的Flutter实现.
package: https://pub.dev/packages/flutter_hooks

Hooks存在的目的是为了增加widgets之间的代码共享, 取代`StatefulWidget`.

首页的例子是: 对一个使用了`AnimationController`的`StatefulWidget`的简化.

flutter_hooks包中已经内置了一些已经写好的hooks.

### Flutter hooks useState

counter demo一个最简单的改法, 就是将`StatefulWidget`改为`HookWidget`.

在`build`方法里:

```
final counter = useState(0);
```

调用`useState`方法设定一个变量, 并设定初始值, 每次值改变的时候widget会被rebuild.

使用值:

```
  Text(
    '${counter.value}',
  ),
```

改变值:

```
  FloatingActionButton(
    onPressed: () => counter.value++,
  ),
```

实际上是把`StatefulWidget`包装了一下, 在初始化Hook的时候注册了listener, 数据改变的时候调用`setState()`方法.
只是把这些操作藏在hook里, 不需要开发者手动调用而已.

所以本质上还是`StatefulWidget`, 之前解决不了的问题它依然解决不了.

代码分支: [flutter-hooks](https://github.com/mengdd/counter_state_management/tree/flutter-hooks).

## Demo

本文demo地址: https://github.com/mengdd/counter_state_management
每个分支对应一种实现. 切换不同分支查看不同的状态管理方法.

对于代码的说明:
这是counter app用不同的状态管理模式进行的改造.
因为这个demo的逻辑和UI都比较简单, 可能实际上并不需要用上一些复杂的状态管理方法, 有种杀鸡用牛刀的感觉.
只是为了保持简单来突出状态管理的实现, 说明用法.

## 一些自己的感想

老实说, 做了这么多年Android, 各种构架MVP, MVVM, MVI, 目的就是数据和逻辑分离, 逻辑和UI分离,
所以初识Flutter的时候对这种万物皆widget, 一个树里面包含一切的方式有点怀疑, UI逻辑数据写成一堆, 程序功能复杂后, 肯定会越写越乱.

但是了解了它的状态管理之后, 发现Flutter的状态管理就是它的程序构架, 并且也是百家争鸣各取所需.
只是Flutter的构架是服务于Flutter framework的设计思想的, 要遵从利用它, 而不是与之反抗.
爱它如是, 而不是如我所愿.

印证了一些道理:

*   不要只喜欢自己熟悉的东西.
*   了解之后才有发言权.

## 参考

*   [Flutter官方文档](https://flutter.dev/docs/development/data-and-backend/state-mgmt)
*   [Flutter官方文档 options](https://flutter.dev/docs/development/data-and-backend/state-mgmt/options)
*   [Flutter Architecture Samples](http://fluttersamples.com/)
*   [Flutter State Management - The Grand Tour](https://www.youtube.com/watch?v=3tm-R7ymwhc)

### Google I/O

*   [Build reactive mobile apps with Flutter (Google I/O'18)](https://www.youtube.com/watch?v=RS36gBEp8OI)
*   [Pragmatic State Management in Flutter (Google I/O'19)](https://www.youtube.com/watch?v=d_m5csmrf7I)

### InheritedWidget

*   [InheritedWidget](https://api.flutter.dev/flutter/widgets/InheritedWidget-class.html)
*   [Flutter实战 7.2 数据共享（InheritedWidget）](https://book.flutterchina.club/chapter7/inherited_widget.html)

### Scoped Model

*   [scoped_model package](https://pub.dev/packages/scoped_model)

### provider

*   [Flutter guide](https://flutter.dev/docs/development/data-and-backend/state-mgmt/simple)
*   [Flutter samples: provider shopper](https://github.com/flutter/samples/tree/master/provider_shopper)
*   [Flutter实战 7.3 跨组件状态共享（Provider）](https://book.flutterchina.club/chapter7/provider.html)

### Bloc

*   [Build reactive mobile apps in Flutter — companion article](https://medium.com/flutter/build-reactive-mobile-apps-in-flutter-companion-article-13950959e381)
*   [filiph/state_experiments](https://github.com/filiph/state_experiments)
*   [Flutter BLoC Pattern](https://www.youtube.com/watch?v=oxeYeMHVLII&list=PLB6lc7nQ1n4jCBkrirvVGr5b8rC95VAQ5)
*   [Getting Started with the BLoC Pattern](https://www.raywenderlich.com/4074597-getting-started-with-the-bloc-pattern)
*   [Effective BLoC pattern](https://medium.com/flutterpub/effective-bloc-pattern-45c36d76d5fe)

### Redux

*   [Introduction to Redux in Flutter](https://blog.novoda.com/introduction-to-redux-in-flutter/)
*   [flutter redux package](https://pub.dev/packages/flutter_redux)
*   [flutter redux github](https://github.com/brianegan/flutter_redux)

### MobX

*   [mobx github](https://github.com/mobxjs/mobx.dart)

**推荐阅读：[腾讯技术团队整理，万字长文轻松彻底入门 Flutter，秒变大前端](https://www.jianshu.com/p/ca0ef3eb32b1)**
原文作者：[圣骑士wind](https://home.cnblogs.com/u/mengdd/)
原文链接：https://www.cnblogs.com/mengdd/p/flutter-state-management.html
