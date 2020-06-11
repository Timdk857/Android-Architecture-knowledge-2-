阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)

在开发中通常需要 Flutter 端与原生内容进行交互。Flutter 定义了三种不同的Channel，它们分别是

BasicMessageChannel：用于传递字符串和半结构化的信息
MethodChannel：用于传递方法调用
EventChannel：用于数据流的通信
# BasicMessageChannel
**1.实现插件**
```
public class FlutterPluginBasicTest implements BasicMessageChannel.MessageHandler{

    private static final String TAG = "FlutterPluginBasicTest";
    public static String CHANNEL = "com.mmd.flutterapp/plugin";

    static BasicMessageChannel messageChannel;

    public static void registerWith(PluginRegistry.Registrar registrar) {
        messageChannel = new BasicMessageChannel(registrar.messenger(),CHANNEL,StandardMessageCodec.INSTANCE);
        FlutterPluginBasicTest flutterPluginBasicTest = new FlutterPluginBasicTest();
        messageChannel.setMessageHandler(flutterPluginBasicTest);
    }

    /**
     * java 发起通信
     * @param string
     */
    void sendMessage(String string) {
        messageChannel.send(string, new BasicMessageChannel.Reply() {
            @Override
            public void reply(Object o) {
                Log.d(TAG, "reply: "+0);
            }
        });
    }


    /**
     * Flutter 发起的通信
     * @param o
     * @param reply
     */
    @Override
    public void onMessage(Object o, BasicMessageChannel.Reply reply) {
        Log.d(TAG, "onMessage: "+o);
        reply.reply("ok");
    }
}
```
**2.注册**
 ```FlutterPluginBasicTest.registerWith(this.registrarFor(FlutterPluginBasicTest.CHANNEL));
```
**3.dart 调用**
```
/**
 * 发送
 */
Future<String> sendMessage() async{
  String reply = await messageChannel.send("Flutter send");
  print(reply);
  return reply;
}

/**
 * 接收
 */
void receiveMessage(){
  messageChannel.setMessageHandler((message) async{
    print(message);
    return "is ok";
  });
}
```
# MethodChannel
#### flutter 调用 原生

**1.实现插件**
```
public class FlutterPluginTest implements MethodChannel.MethodCallHandler {

    private static final String TAG = "FlutterPluginTest";

    /**
     * 插件标识
     */
    public static String CHANNEL = "com.mmd.flutterapp/plugin";

    private static String ACTION_LOG = "log";

    private static String LOG_ARGUMENT = "data";

    static MethodChannel channel;

    public static void registerWith(PluginRegistry.Registrar registrar) {
        channel = new MethodChannel(registrar.messenger(), CHANNEL);
        FlutterPluginTest instance = new FlutterPluginTest();
        channel.setMethodCallHandler(instance);
    }

    @Override
    public void onMethodCall(MethodCall methodCall, MethodChannel.Result result) {

        /**
         * 通过 method 判断调用方法
         */
        if (methodCall.method.equals(ACTION_LOG)) {
            /**
             * 解析参数
             */
            String text = methodCall.argument(LOG_ARGUMENT);
            if (TextUtils.isEmpty(text)) {
                /**
                 * 错误返回
                 */
                result.error("Data is Null",null,null);
            }else {
                Log.d(TAG, "onMethodCall: "+text);
                /**
                 * 成功返回
                 */
                result.success("is ok");
            }
        }else {
            result.notImplemented();
        }
    }
}
```
**2.注册插件**
```
public class MainActivity extends FlutterActivity {
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    /**
     * 注册插件
     */
    FlutterPluginTest.registerWith(this.registrarFor(FlutterPluginTest.CHANNEL));
  }
}
```
**3.Flutter 端调用**
```
import 'package:flutter/services.dart';

/**
 * 名称要和Java端一致
 */
const channelName = "com.mmd.flutterapp/plugin";

const methodName = "log";

const MethodChannel channel = MethodChannel(channelName);

Future<String> _testLog() async{
  
  Map<String,String> map = {"data":"Flutter Hello !"};
  
  String result = await channel.invokeMethod(methodName,map);
  
  print(result);
}
```
# EventChannel
#### 原生发送数据到Flutter

**1.实现插件**
```
public class FlutterPluginEventTest implements EventChannel.StreamHandler {

    private static final String TAG = "FlutterPluginEventTest";
    public static String CHANNEL = "com.mmd.flutterapp/plugin";

    static EventChannel channel;

    public static void registerWith(PluginRegistry.Registrar registrar) {
        channel = new EventChannel(registrar.messenger(), CHANNEL);
        FlutterPluginEventTest flutterPluginEventTest = new FlutterPluginEventTest();
        channel.setStreamHandler(flutterPluginEventTest);
    }

    @Override
    public void onListen(Object o, EventChannel.EventSink eventSink) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    try {
                        Thread.sleep(1000);
                        eventSink.success(System.currentTimeMillis());
                    } catch (InterruptedException e) {
                        eventSink.error("error","error",e.getMessage());
                    }
                }
            }
        }).start();

    }

    @Override
    public void onCancel(Object o) {
        Log.i(TAG, "onCancel: "+o);
    }
}
```
**2.注册插件**
```
FlutterPluginEventTest.registerWith(this.registrarFor(FlutterPluginEventTest.CHANNEL));
```
**3.Flutter 接收**
```
import 'dart:async';

import 'package:flutter/services.dart';

/**
 * 名称要和Java端一致
 */
const channelName = "com.mmd.flutterapp/plugin";

const EventChannel eventChannel = EventChannel(channelName);

StreamSubscription _subcription = null;

void init(void onEvent(String value),Function onError){
  if(_subcription == null) {
    _subcription = eventChannel.receiveBroadcastStream().listen(onEvent,onError: onError);
  }
}

void dispose(){
  if(_subcription !=null){
    _subcription.cancel();
  }
}
```
原文作者：MaDeng
原文链接：http://www.mdshi.cn/flutter-yu-yuan-sheng-jiao-hu-android/
阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)
