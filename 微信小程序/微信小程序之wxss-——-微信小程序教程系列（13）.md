WXSS（WeiXin Style Sheet）与CSS对应，用于描述页面的样式。



# 特性

内联样式：
组件的 style 接收动态的样式，在运行时会进行解析，请尽量避免将静态的样式写进style中，以免影响渲染速度。

**选择器**
对于常用的选择器，目前支持的选择器有：
![](https://upload-images.jianshu.io/upload_images/19956127-031ceea2ce50f733.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


注：绿色背景色行表示官方文档中没有说明，但经个人亲测后确定也支持的选择器。



目前不支持的选择器有：
![](https://upload-images.jianshu.io/upload_images/19956127-871d8ebc28c33b22.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**注意：**
如之前提到，页面的顶层是节点，所以作用于整个页面的样式或修改顶层节点样式请使用page选择器。
小程序目前不支持Media Query。

**扩展的特性**
![](https://upload-images.jianshu.io/upload_images/19956127-98142135b7252bba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

建议：开发微信小程序时设计师可以用 iPhone6 作为视觉稿的标准。

注意： 由于数值较小时渲染时会存在四舍五入的情况，在较小屏幕上差距会很大，所以要求精确而较小的视图内容需避免使用此单位。



样式导入
用@import语句可以导入外联样式表，@import后跟需要导入的外联样式表的相对路径，用;表示语句结束。

**示例：**

index.wxml：
![](https://upload-images.jianshu.io/upload_images/19956127-282179a6e757e975.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


index.wxss：
![](https://upload-images.jianshu.io/upload_images/19956127-11edd9f8754b42d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**运行：**

选择iphone 4手机查看
![](https://upload-images.jianshu.io/upload_images/19956127-5904d59fafc38ec2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


选择iphone 6 Plus手机查看
![](https://upload-images.jianshu.io/upload_images/19956127-3e3530f9724d3acb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

原文作者：michael_ouyang
原文链接：https://blog.csdn.net/michael_ouyang/article/details/55050052
