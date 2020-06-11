阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)

`Flutter`默认是单线程任务处理的，如果不开启新的线程，任务默认在主线程中处理。

### 事件队列

和iOS应用很像，在`Dart`的线程中也存在事件循环和消息队列的概念，但在`Dart`中线程叫做`isolate`。应用程序启动后，开始执行`main`函数并运行`main isolate`。

每个`isolate`包含一个事件循环以及两个事件队列，`event loop`事件循环，以及`event queue`和`microtask queue`事件队列，`event`和`microtask`队列有点类似iOS的`source0`和`source1`。

*   event queue：负责处理I/O事件、绘制事件、手势事件、接收其他`isolate`消息等外部事件。
*   microtask queue：可以自己向`isolate`内部添加事件，事件的优先级比`event queue`高。

![](//upload-images.jianshu.io/upload_images/270478-3d931404e40911ee.png?imageMogr2/auto-orient/strip|imageView2/2/w/471/format/webp)

这两个队列也是有优先级的，当`isolate`开始执行后，会先处理`microtask`的事件，当`microtask`队列中没有事件后，才会处理`event`队列中的事件，并按照这个顺序反复执行。但需要注意的是，当执行`microtask`事件时，会阻塞`event`队列的事件执行，这样就会导致渲染、手势响应等`event`事件响应延时。为了保证渲染和手势响应，应该尽量将耗时操作放在`event`队列中。

### async、await

在异步调用中有三个关键词，`async`、`await`、`Future`，其中`async`和`await`需要一起使用。在`Dart`中可以通过`async`和`await`进行异步操作，`async`表示开启一个异步操作，也可以返回一个`Future`结果。如果没有返回值，则默认返回一个返回值为`null`的`Future`。

`async`、`await`本质上就是`Dart`对异步操作的一个语法糖，可以减少异步调用的嵌套调用，并且由`async`修饰后返回一个`Future`，外界可以以链式调用的方式调用。这个语法是`JS`的`ES7`标准中推出的，`Dart`的设计和`JS`相同。

下面封装了一个网络请求的异步操作，并且将请求后的`Response`类型的`Future`返回给外界，外界可以通过`await`调用这个请求，并获取返回数据。从代码中可以看到，即便直接返回一个字符串，`Dart`也会对其进行包装并成为一个`Future`。

```
Future<Response> dataReqeust() async {
    String requestURL = 'https://jsonplaceholder.typicode.com/posts';
    Client client = Client();
    Future<Response> response = client.get(requestURL);
    return response;
}

Future<String> loadData() async {
    Response response = await dataReqeust();
    return response.body;
}

```

在代码示例中，执行到`loadData`方法时，会同步进入方法内部进行执行，当执行到`await`时就会停止`async`内部的执行，从而继续执行外面的代码。当`await`有返回后，会继续从`await`的位置继续执行。所以`await`的操作，不会影响后面代码的执行。

下面是一个代码示例，通过`async`开启一个异步操作，通过`await`等待请求或其他操作的执行，并接收返回值。当数据发生改变时，调用`setState`方法并更新数据源，`Flutter`会更新对应的`Widget`节点视图。

```
class _SampleAppPageState extends State<SampleAppPage> {
  List widgets = [];

  @override
  void initState() {
    super.initState();
    loadData();
  }

  loadData() async {
    String dataURL = "https://jsonplaceholder.typicode.com/posts";
    http.Response response = await http.get(dataURL);
    setState(() {
      widgets = json.decode(response.body);
    });
  }
}

```

### Future

`Future`就是延时操作的一个封装，可以将异步任务封装为`Future`对象。获取到`Future`对象后，最简单的方法就是用`await`修饰，并等待返回结果继续向下执行。正如上面`async、await`中讲到的，使用`await`修饰时需要配合`async`一起使用。

在`Dart`中，和时间相关的操作基本都和`Future`有关，例如延时操作、异步操作等。下面是一个很简单的延时操作，通过`Future`的`delayed`方法实现。

```
loadData() {
    // DateTime.now()，获取当前时间
    DateTime now = DateTime.now();
    print('request begin $now');
    Future.delayed(Duration(seconds: 1), (){
      now = DateTime.now();
      print('request response $now');
    });
}

```

`Dart`还支持对`Future`的链式调用，通过追加一个或多个`then`方法来实现，这个特性非常实用。例如一个延时操作完成后，会调用`then`方法，并且可以传递一个参数给`then`。调用方式是链式调用，也就代表可以进行很多层的处理。这有点类似于iOS的`RAC`框架，链式调用进行信号处理。

```
Future.delayed(Duration(seconds: 1), (){
  int age = 18;
  return age;
}).then((onValue){
  onValue++;
  print('age $onValue');
});

```

### 协程

如果想要了解`async`、`await`的原理，就要先了解协程的概念，`async`、`await`本质上就是协程的一种语法糖。协程，也叫作`coroutine`，是一种比线程更小的单元。如果从单元大小来说，基本可以理解为进程->线程->协程。

#### 任务调度

在弄懂协程之前，首先要明白并发和并行的概念，并发指的是由系统来管理多个IO的切换，并交由CPU去处理。并行指的是多核CPU在同一时间里执行多个任务。

并发的实现由非阻塞操作+事件通知来完成，事件通知也叫做“中断”。操作过程分为两种，一种是CPU对IO进行操作，在操作完成后发起中断告诉IO操作完成。另一种是IO发起中断，告诉CPU可以进行操作。

线程本质上也是依赖于中断来进行调度的，线程还有一种叫做“阻塞式中断”，就是在执行IO操作时将线程阻塞，等待执行完成后再继续执行。但线程的消耗是很大的，并不适合大量并发操作的处理，而通过单线程并发可以进行大量并发操作。当多核CPU出现后，单个线程就无法很好的利用多核CPU的优势了，所以又引入了线程池的概念，通过线程池来管理大量线程。

#### 协程

在程序执行过程中，离开当前的调用位置有两种方式，继续调用其他函数和`return`返回离开当前函数。但是执行`return`时，当前函数在调用栈中的局部变量、形参等状态则会被销毁。

协程分为无线协程和有线协程，无线协程在离开当前调用位置时，会将当前变量放在堆区，当再次回到当前位置时，还会继续从堆区中获取到变量。所以，一般在执行当前函数时就会将变量直接分配到堆区，而`async`、`await`就属于无线协程的一种。有线协程则会将变量继续保存在栈区，在回到指针指向的离开位置时，会继续从栈中取出调用。

#### async、await原理

以`async`、`await`为例，协程在执行时，执行到`async`则表示进入一个协程，会同步执行`async`的代码块。`async`的代码块本质上也相当于一个函数，并且有自己的上下文环境。当执行到`await`时，则表示有任务需要等待，CPU则去调度执行其他IO，也就是后面的代码或其他协程代码。过一段时间CPU就会轮训一次，看某个协程是否任务已经处理完成，有返回结果可以被继续执行，如果可以被继续执行的话，则会沿着上次离开时指针指向的位置继续执行，也就是`await`标志的位置。

由于并没有开启新的线程，只是进行IO中断改变CPU调度，所以网络请求这样的异步操作可以使用`async`、`await`，但如果是执行大量耗时同步操作的话，应该使用`isolate`开辟新的线程去执行。

如果用协程和iOS的`dispatch_async`进行对比，可以发现二者是比较相似的。从结构定义来看，协程需要将当前`await`的代码块相关的变量进行存储，`dispatch_async`也可以通过`block`来实现临时变量的存储能力。

我之前还在想一个问题，苹果为什么不引入协程的特性呢？后来想了一下，`await`和`dispatch_async`都可以简单理解为异步操作，OC的线程是基于`Runloop`实现的，`Dart`本质上也是有事件循环的，而且二者都有自己的事件队列，只是队列数量和分类不同。

我觉得当执行到`await`时，保存当前的上下文，并将当前位置标记为待处理任务，用一个指针指向当前位置，并将待处理任务放入当前`isolate`的队列中。在每个事件循环时都去询问这个任务，如果需要进行处理，就恢复上下文进行任务处理。

#### Promise

这里想提一下`JS`里的`Promise`语法，在iOS中会出现很多`if`判断或者其他的嵌套调用，而`Promise`可以把之前横向的嵌套调用，改成纵向链式调用。如果能把`Promise`引入到OC里，可以让代码看起来更简洁，直观。

### isolate

`isolate`是`Dart`平台对线程的实现方案，但和普通`Thread`不同的是，`isolate`拥有独立的内存，`isolate`由线程和独立内存构成。正是由于`isolate`线程之间的内存不共享，所以`isolate`线程之间并不存在资源抢夺的问题，所以也不需要锁。

通过`isolate`可以很好的利用多核CPU，来进行大量耗时任务的处理。`isolate`线程之间的通信主要通过`port`来进行，这个`port`消息传递的过程是异步的。通过`Dart`源码也可以看出，实例化一个`isolate`的过程包括，实例化`isolate`结构体、在堆中分配线程内存、配置`port`等过程。

`isolate`看起来其实和进程比较相似，之前请教阿里架构师宗心问题时，宗心也说过“`isolate`的整体模型我自己的理解其实更像进程，而`async`、`await`更像是线程”。如果对比一下`isolate`和进程的定义，会发现确实`isolate`很像是进程。

#### 代码示例

下面是一个`isolate`的例子，例子中新创建了一个`isolate`，并且绑定了一个方法进行网络请求和数据解析的处理，并通过`port`将处理好的数据返回给调用方。

```
loadData() async {
    // 通过spawn新建一个isolate，并绑定静态方法
    ReceivePort receivePort =ReceivePort();
    await Isolate.spawn(dataLoader, receivePort.sendPort);

    // 获取新isolate的监听port
    SendPort sendPort = await receivePort.first;
    // 调用sendReceive自定义方法
    List dataList = await sendReceive(sendPort, 'https://jsonplaceholder.typicode.com/posts');
    print('dataList $dataList');
}

// isolate的绑定方法
static dataLoader(SendPort sendPort) async{
    // 创建监听port，并将sendPort传给外界用来调用
    ReceivePort receivePort =ReceivePort();
    sendPort.send(receivePort.sendPort);

    // 监听外界调用
    await for (var msg in receivePort) {
      String requestURL =msg[0];
      SendPort callbackPort =msg[1];

      Client client = Client();
      Response response = await client.get(requestURL);
      List dataList = json.decode(response.body);
      // 回调返回值给调用者
      callbackPort.send(dataList);
    }    
}

// 创建自己的监听port，并且向新isolate发送消息
Future sendReceive(SendPort sendPort, String url) {
    ReceivePort receivePort =ReceivePort();
    sendPort.send([url, receivePort.sendPort]);
    // 接收到返回值，返回给调用者
    return receivePort.first;
}

```

`isolate`和iOS中的线程还不太一样，`isolate`的线程更偏底层。当生成一个`isolate`后，其内存是各自独立的，相互之间并不能进行访问。但`isolate`提供了基于`port`的消息机制，通过建立通信双方的`sendPort`和`receiveport`，进行相互的消息传递，在`Dart`中叫做消息传递。

从上面例子中可以看出，在进行`isolate`消息传递的过程中，本质上就是进行`port`的传递。将`port`传递给其他`isolate`，其他`isolate`通过`port`拿到`sendPort`，向调用方发送消息来进行相互的消息传递。

### Embedder

正如其名，`Embedder`是一个嵌入层，将`Flutter`嵌入到各个平台上。`Embedder`负责范围包括原生平台插件、线程管理、事件循环等。

![](//upload-images.jianshu.io/upload_images/270478-8314eb1ff5e12fb2.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

`Embedder`中存在四个`Runner`，四个`Runner`分别如下。其中每个`Flutter Engine`各自对应一个`UI Runner`、`GPU Runner`、`IO Runner`，但所有`Engine`共享一个`Platform Runner`。

![](//upload-images.jianshu.io/upload_images/270478-2408ba7779231f90.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/731/format/webp)

`Runner`和`isolate`并不是一码事，彼此相互独立。以iOS平台为例，`Runner`的实现就是`CFRunLoop`，以一个事件循环的方式不断处理任务。并且`Runner`不只处理`Engine`的任务，还有`Native Plugin`带来的原生平台的任务。而`isolate`则由`Dart VM`进行管理，和原生平台线程并无关系。

#### Platform Runner

`Platform Runner`和iOS平台的`Main Thread`非常相似，在`Flutter`中除耗时操作外，所有任务都应该放在`Platform`中，`Flutter`中的很多API并不是线程安全的，放在其他线程中可能会导致一些bug。

但例如IO之类的耗时操作，应该放在其他线程中完成，否则会影响`Platform`的正常执行，甚至于被`watchdog`干掉。但需要注意的是，由于`Embedder Runner`的机制，`Platform`被阻塞后并不会导致页面卡顿。

不只是`Flutter Engine`的代码在`Platform`中执行，`Native Plugin`的任务也会派发到`Platform`中执行。实际上，在原生侧的代码运行在`Platform Runner`中，而`Flutter`侧的代码运行在`Root Isolate`中，如果在`Platform`中执行耗时代码，则会卡原生平台的主线程。

#### UI Runner

`UI Runner`负责为`Flutter Engine`执行`Root Isolate`的代码，除此之外，也处理来自`Native Plugin`的任务。`Root Isolate`为了处理自身事件，绑定了很多函数方法。程序启动时，`Flutter Engine`会为`Root`绑定`UI Runner`的处理函数，使`Root Isolate`具备提交渲染帧的能力。

当`Root Isolate`向`Engine`提交一次渲染帧时，`Engine`会等待下次vsync，当下次vsync到来时，由`Root Isolate`对`Widgets`进行布局操作，并生成页面的显示信息的描述，并将信息交给`Engine`去处理。

由于对`widgets`进行`layout`并生成`layer tree`是`UI Runner`进行的，如果在`UI Runner`中进行大量耗时处理，会影响页面的显示，所以应该将耗时操作交给其他`isolate`处理，例如来自`Native Plugin`的事件。

![](//upload-images.jianshu.io/upload_images/270478-56e71a5d9ae4b175.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

#### GPU Runner

`GPU Runner`并不直接负责渲染操作，其负责GPU相关的管理和调度。当`layer tree`信息到来时，`GPU Runner`将其提交给指定的渲染平台，渲染平台是Skia配置的，不同平台可能有不同的实现。

`GPU Runner`相对比较独立，除了`Embedder`外其他线程均不可向其提交渲染信息。

![](//upload-images.jianshu.io/upload_images/270478-14ab5a0cebfcb505.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

#### IO Runner

一些`GPU Runner`中比较耗时的操作，就放在`IO Runner`中进行处理，例如图片读取、解压、渲染等操作。但是只有`GPU Runner`才能对GPU提交渲染信息，为了保证`IO Runner`也具备这个能力，所以`IO Runner`会引用`GPU Runner`的`context`，这样就具备向GPU提交渲染信息的能力。

作者：刘小壮
链接：https://www.jianshu.com/p/54da18ed1a9e
阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)
