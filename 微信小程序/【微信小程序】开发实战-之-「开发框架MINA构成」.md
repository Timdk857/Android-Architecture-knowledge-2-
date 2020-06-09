**小程序开发框架的目标是通过尽可能简单、高效的方式让开发者可以在微信中开发具有原生 APP 体验的服务。**

微信团队为小程序提供的框架命名为MINA。MINA框架通过封装微信客户端提供的文件系统、网络通信、任务管理、数据安全等基础功能，对上层提供一整套JavaScript API，让开发者方便的使用微信客户端提供的各种基础功能与能力，快速构建应用。

### MINA框架

微信小程序的框架示意图如下所示：

 ![](https://upload-images.jianshu.io/upload_images/19956127-9cd6fb39b2ab1924.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### MINA框架主要分为两大部分：

第一部分页面视图层，开发者使用WXML文件来搭建页面的基本视图结构（WXML是类似于HTML标签的语言和一系列基础组件），使用WXSS文件来控制页面的表现样式。

第二部分AppService应用逻辑层，是MINA框架的服务中心，通过微信客户端启动异步线程单独加载运行，页面渲染所需的数据、页面交互处理逻辑都在其中实现。MINA框架中的AppService使用JavaScript来编写交互逻辑、网络请求、数据处理，但不能使用JavaScript中的DOM操作。小程序中的各个页面可以通过AppService实现数据管理、网络通信、生命周期管理和页面路由。

MINA框架为页面组件提供了一系列事件监听相关的属性（比如bindtap、bindtouchstart等），来与AppService中的事件处理函数绑定在一起，来实现页面向AppService层同步用户交互数据。MINA框架同时提供了很多方法将AppService中的数据与页面进行单向绑定（注意数据的绑定方向是单向的），当AppService中的数据变更时，会主动触发对应页面组件的重新渲染。

**框架的核心是一个响应式的数据绑定系统，它能让数据与视图很简单的保持同步**。只需要在逻辑层修改数据，视图层就会做相应的更新。示例如下：
```
<!--页面视图层代码-->  
<view class="apptitle">
    <text class="app-avatar">欢迎使用{{appname}}</text>
    <button bindtap="changeAppname">更换名称 </button>
</view>

//AppService应用逻辑层代码
//初始数据
page({
    data:{
        appname:'易投票'
  },
    changeAppname:function(e){
        this.setData({
        appname:'我的小程序'
    })
  }
})
```
![图1：初始名称](https://upload-images.jianshu.io/upload_images/19956127-e4608803dbcf6cbe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![图2：点击按钮“更换名称”以后](https://upload-images.jianshu.io/upload_images/19956127-370f3e032584ee83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

示例中数据是如何更新的呢？首先，开发者通过框架将AppService应用逻辑层数据中的appname与页面视图层名为appname的变更进行了绑定，页面在刚打开的时候会显示“欢迎使用 易投票。然后，当点击按钮“更换名称”之后，视图层会发送changeAppname的tap事件给逻辑层，逻辑层找到事件函数changeAppname。最后，逻辑层changeAppname函数执行了setData操作，将对象appname的值改变为“我的小程序”，因为该对象已经在视图层绑定，所以视图层会显示为图2的名称了。

小程序的MINA框架有着接近原生App的运行速度，在框架层面做了大量的优化，在重功能上（page或tab切换、多媒体、网络连接等）上使用接近于native的组件继承，对安卓和ios端做出了高度一致的呈现，还有近乎完备的开发、调试工具。

### 目录结构

典型的小程序目录结构非常简洁，一般一个项目包含两个目录（pages和utils）三个文件（app.js、app.json、app.wxss）。pages目录下包括程序所需的各个页面，一个页面对应一个目录，包含2至4个文件（.js、.wxml、.json及.wxss）。utils目录则包含一些公共的js代码文件。当然，我们还可以添加其他的公共目录，如用来存放本地图片资源的images目录。

 ![](https://upload-images.jianshu.io/upload_images/19956127-93279c70b2164c2d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 逻辑层

小程序的逻辑层就是所有.js脚本文件的集合。小程序在逻辑层处理数据并发送至视图层，同时接受视图层发回的事件请求。

MINA框架的逻辑层是由JavaScript编写，在此基础上，微信团队做出了一些优化，以便更高效的开发小程序，这些优化包括：

1、增加app方法用来注册程序，增加page方法用来注册页面；

2、提供丰富的API接口；

3、页面的作用域相对独立，并拥有了模块化的能力；

简单概括，逻辑层就是各个页面的.js脚本文件。

需要注意的是，小程序的逻辑层由js编写，但并不是在浏览器中运行的，所以JavaScript在Web中的一些能力都不能使用，比如 dom、window等，这也是我们开发过程中要克服的阻碍。

### 视图层

对于微信小程序而言，视图层就是所有的.wxml（WeiXin Markup language）文件与.wxss（WeiXin Style Sheet）文件的集合：.wxml用于描述页面结构而.wxss用于描述页面样式。

视图层以给定的样式来展现数据并反馈事件给逻辑层，而数据展现是以组件来进行的。组件（Component）是视图的基本组成单元。

### 数据层

数据层包括临时数据或缓存、文件存储、网络存储与调用。

#### 1、页面临时数据或缓存

在页面page()中，我们要使用setData函数来将数据从逻辑层发送到视图层，同时改变对应的this.data的值。this在小程序中一般指调用页面，广泛情况下指的是包含它的函数作为方法被调用时所属的对象。直接修改this.data是无效的，无法改变页面的状态，还会造成数据的不一致。单次设置的数据有一个大小限制，不能超过1024KB，避免一次性设置过多的数据。

setData()函数的参数接受一个对象。以key,value的形式表示，将this.data中的key对应的值改变为value。key可以非常灵活，包括以数据路径的形式表示，如array[0].title，并且无需在this.data中预定义。

#### 2、文件存储（本地存储）

使用微信提供的现成数据API接口，如：

wx.getStorage：获取本地数据缓存

wx.setStorage：设置本地数据缓存

wx.clearStorage：清理本地数据缓存

#### 3、网络存储与调用

上传或下载文件的API接口，如：

wx.request：发起网络请求

wx.uploadFile：上传文件

wx.downloadFile：下载文件

#### 调用URL的API接口如下：

wx.navigateTo：保留当前页面，跳转到应用内的某个页面。但是不能跳到 tabbar 页面。可返回原页面。

wx.redirectTo：关闭当前页面，跳转到应用内的某个页面。但是不允许跳转到 tabbar 页面。不可返回原页面。

 以上就是微信小程序框架的相关描述，微信团队一直在不断的优化框架能力，及时的关注官方提供的[小程序开发者文档](https://developers.weixin.qq.com/miniprogram/dev/index.html?t=19041716)，了解小程序的最新能力及优化点。
原文作者：DreamGo
原文链接：https://www.cnblogs.com/idreamo/p/10867241.html
