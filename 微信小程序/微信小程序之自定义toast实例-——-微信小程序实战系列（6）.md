微信提供了一个toast的api  wx.showToast()

相关连接：https://mp.weixin.qq.com/debug/wxadoc/dev/api/api-react.html#wxshowtoastobject

 

本来是比较好的，方便使用，但是这个toast会显示出图标，而且不能去除。

假设：我们执行完业务的时候，toast一下，当执行成功的时候，效果还可以接受，如下图：
![](https://upload-images.jianshu.io/upload_images/19956127-cbff9b61ee652d28.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


但是，当执行失败的时候，如下图：

失败了，你还显示个扣扣图案，那到底是成功还是失败？？这肯定是不能接受的。【捂脸】

若是给老板看到这种效果，又是一顿臭骂，程序猿的委屈
![](https://upload-images.jianshu.io/upload_images/19956127-cfcd7450e15ee451.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


下面介绍一个自定义的toast

效果：
![](https://upload-images.jianshu.io/upload_images/19956127-4859494f834da3ef.gif?imageMogr2/auto-orient/strip)

**具体实现：**


**wxml：**
```
<!--按钮-->
<view style="{{isShowToast?'position:fixed;':''}}">
  <view class="btn" bindtap="clickBtn">button</view>
</view>
 
<!--mask-->
<view class="toast_mask" wx:if="{{isShowToast}}"></view>
<!--以下为toast显示的内容-->
<view class="toast_content_box" wx:if="{{isShowToast}}">
  <view class="toast_content">
    <view class="toast_content_text">
      {{toastText}}
    </view>
  </view>
</view>
```
**wxss：**
```
Page {
  background: #fff;
}
/*按钮*/
.btn {
  font-size: 28rpx;
  padding: 15rpx 30rpx;
  width: 100rpx;
  margin: 20rpx;
  text-align: center;
  border-radius: 10rpx;
  border: 1px solid #000;
}
/*mask*/
.toast_mask {
  opacity: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
  position: fixed;
  top: 0;
  left: 0;
  z-index: 888;
}
/*toast*/
.toast_content_box {
  display: flex;
  width: 100%;
  height: 100%;
  justify-content: center;
  align-items: center;
  position: fixed;
  z-index: 999;
}
.toast_content {
  width: 50%;
  padding: 20rpx;
  background: rgba(0, 0, 0, 0.5);
  border-radius: 20rpx;
}
.toast_content_text {
  height: 100%;
  width: 100%;
  color: #fff;
  font-size: 28rpx;
  text-align: center;
}
```
**js：**
```
Page({
  data: {
    //toast默认不显示
    isShowToast: false 
  },
  showToast: function () {
    var _this = this;
    // toast时间
    _this.data.count = parseInt(_this.data.count) ? parseInt(_this.data.count) : 3000;
    // 显示toast
    _this.setData({
      isShowToast: true,
    });
    // 定时器关闭
    setTimeout(function () {
      _this.setData({
        isShowToast: false
      });
    }, _this.data.count);
  },
  /* 点击按钮 */
  clickBtn: function () {
    console.log("你点击了按钮")
    //设置toast时间，toast内容
    this.setData({
      count: 1500,
      toastText: 'Michael’s　Toast'
    });
    this.showToast();
  }
})
```
原文作者：michael_ouyang
原文链接：https://blog.csdn.net/michael_ouyang/article/details/60867679
