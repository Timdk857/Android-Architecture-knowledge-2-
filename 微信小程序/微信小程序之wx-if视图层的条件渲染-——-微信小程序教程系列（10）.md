![](https://upload-images.jianshu.io/upload_images/19956127-d9afef6be99efec2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
使用wx:if进行视图层的条件渲染

**示例：**
wxml：使用view
```
<!--index.wxml-->

<button bindtap="EventHandle">按钮</button>

<!-- wx:if -->

<view wx:if="{{boolean==true}}">

    <view class="bg_black"></view>

</view>

<view wx:elif="{{boolean==false}}">

    <view class="bg_red"></view>

</view>
```
 
wxss：
```
/**index.wxss**/

.bg_black {

  height: 200rpx;

  background: lightskyblue;

}

.bg_red {

  height: 200rpx;

  background: lightpink;

}
```
 
js：
```
// index.js

Page({

  data: {

    boolean:false

  },

  EventHandle: function(){

    var bol = this.data.boolean;

    this.setData({

      boolean: !bol

    })

  }

})
```
**运行：**
![](https://upload-images.jianshu.io/upload_images/19956127-2658eeef66f3e5c8.gif?imageMogr2/auto-orient/strip)

**续上：**

把上面标注绿色部分的view改成block

**wxml：使用block**
```
<!--index.wxml-->

<button bindtap="EventHandle">按钮</button>

<!-- wx:if -->

<block wx:if="{{boolean==true}}">

    <view class="bg_black"></view>

</block>

<block wx:elif="{{boolean==false}}">

    <view class="bg_red"></view>

</block>
```
运行：
![](https://upload-images.jianshu.io/upload_images/19956127-42428cab327f5ae5.gif?imageMogr2/auto-orient/strip)

**续上：**

增加一个wx:for做列表渲染

**wxml：**
```
<!--index.wxml-->

<button bindtap="EventHandle">按钮</button>

<!-- wx:if -->

<block wx:if="{{boolean==true}}" wx:for="{{arr}}">

    <view class="bg_black">内容：{{item}}</view>

</block>

<block wx:elif="{{boolean==false}}">

    <view class="bg_red">无内容</view>

</block>
```
 
**index.js：**
```
// index.js

Page({

  data: {

    boolean:false,

    arr: [1,2,3]

  },

  EventHandle: function(){

    var bol = this.data.boolean;

    this.setData({

      boolean: !bol

    })

  }

})
```
**运行：**

编辑错误。
![](https://upload-images.jianshu.io/upload_images/19956127-c1fa842b158d6234.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

原因是wx:if不能与wx:for共用在一个组件上！

**续上：**

wx:if和wx:for必须分开使用

**wxml：**
```
<!--index.wxml-->

<button bindtap="EventHandle">按钮</button>

<!-- wx:if -->

<block wx:if="{{boolean==true}}">

    <block wx:for="{{arr}}">

        <view class="bg_black">内容：{{item}}</view>

    </block>

</block>

<block wx:elif="{{boolean==false}}">

    <view class="bg_red">无内容</view>

</block>
```
**wxss：**
```
/**index.wxss**/

.bg_black {

  height: 200rpx;

  background: lightskyblue;

}

.bg_red {

  height: 200rpx;

  background: lightpink;

}
```
**index.js：**
```
// index.js

Page({

  data: {

    boolean:false,

    arr: [1,2,3]

  },

  EventHandle: function(){

    var bol = this.data.boolean;

    this.setData({

      boolean: !bol

    })

  }

})
```
运行：
![](https://upload-images.jianshu.io/upload_images/19956127-ff9da8a88f5260c6.gif?imageMogr2/auto-orient/strip)

原文作者：michael_ouyang
原文链接：https://blog.csdn.net/michael_ouyang/article/details/55049118
