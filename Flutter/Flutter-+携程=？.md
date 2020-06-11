## 目录

*   主要页面，包括首页、搜索、旅拍和我的四个主页面
*   依赖库
*   实际效果

## 主要页面

*   整体框架采用PageView + BottomNavigationBar ，每个页面的状态保存采用AutomaticKeepAliveClientMixin

*   **首页**

    *   全面屏适配，体现在顶部搜索框距离状态栏的距离，项目内笔者采用的是 MediaQueryData.fromWindow(window).padding.top 得到状态栏高度进行适配，当然也可以使用SafeArea来包裹页面。(使用了Scaffold的appbar与bottomNavigationBar是不需要适配的，因为Scaffold框架会自动帮我们完成这些适配工作)
    *   轮播图主要采用的是Swiper控件
    *   列表采用ListView控件，如果数据过多，需要上拉加载建议使用ListView的Builder方法进行服用View
    *   主页整体布局采用了Stack + MediaQuery.removePadding + RefreshIndicator + appBar
    *   通过对Container进行alpha设置实现appBar的颜色渐变
*   **搜索**

    *   语音识别采用百度API，native接入百度语音识别API，这里需要注意build.gradle的设置，由于笔者是通过新建android模块，所以需要仿照主app的build.gradle对fltter引入，才能导入MethodChannel相关类。此处涉及Flutter与native通信，两端方法名需要一致。
    *   语音识别后自动跳转就行搜索，利用ListView显示数据，用到FractionallySizedBox控件撑满屏幕宽度，利用Expand设置权重，个人感觉Expand等价于LinearLayout，flex属性和weight属性类似
*   **旅拍**

    *   TabBar + Flexible+ TabBarView
    *   RefreshIndicator + StaggeredGridView + Stack + Card + PhysicalModel 实现下拉刷新 上拉加载
    *   文字固定宽度 LimitedBox
    *   圆形图片使用 PhysicalModel 圆角设置为控件长/宽一半
*   **我的**

    *   WebView
*   网页加载

    *   所有点击功能采用GestureDetector控件实现，网页使用WebView(利用FlutterWebviewPlugin控件自定义)控件加载
    *   当然也可以利用webview_flutter插件替代上述自定义WebView
*   网络

    *   采用Http get和post请求，json解析
    *   接口在项目内

## 依赖库

*   flutter_swiper: ^1.1.4
*   http: ^0.12.0+4
*   flutter_webview_plugin: ^0.3.10+1
*   flutter_staggered_grid_view: ^0.3.0
*   flutter_splash_screen: ^0.1.0
*   Flutter插件开发 Flutter插件库

## 实际效果

![](https://upload-images.jianshu.io/upload_images/19956127-a3626ffb91f83478.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](https://upload-images.jianshu.io/upload_images/19956127-60e0f169db966e0c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](https://upload-images.jianshu.io/upload_images/19956127-b8a6aa485885fb65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](https://upload-images.jianshu.io/upload_images/19956127-33006e0bdef68723.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**推荐阅读：[2017-2020历年字节跳动Android面试真题解析（累计下载1082万次，持续更新中）](https://www.jianshu.com/p/7f9ade51232e)**


作者：Tenderness4
链接：https://juejin.im/post/5e650b72518825492f771f5a
来源：掘金

