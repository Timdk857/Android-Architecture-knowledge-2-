底部3-5个选项的底部导航栏，目前在移动端上是主流布局之一

因此腾讯官方特地做了，可以通过设置，就可以做出了一个底部的导航栏

相关教程：http://blog.csdn.net/michael_ouyang/article/details/55045300

但是通过设置的这个底部的导航栏，功能上比较固定，它必须要设置与它对应的一个页面，而且并不能滑动。

在业务上，有时候会比较限制，并不能完全满足所需。


又例如早前有人拿着UI稿问我，这种广告轮播图的样式，在小程序能不能实现呢？

我当时没有想了下，还不是很确定，因为小程序的轮播图的那几个小点点实在比较普通，样式单一。

因此特意写了一篇自定义轮播图的文章

链接：http://blog.csdn.net/michael_ouyang/article/details/58591232

因此自定义就有这个必要性

### 下面介绍这个仿android fragment可滑动的底部导航栏如何实现

项目最终效果图：

![](https://upload-images.jianshu.io/upload_images/19956127-00625c2ca21b2ad9.gif?imageMogr2/auto-orient/strip)


**wxml：**
```
<swiper current="{{currentTab}}" class="swiper-box" duration="300" style="height:{{winHeight - 51}}px" bindchange="bindChange">
 
  <!-- frag01 -->
  <swiper-item>
    <scroll-view class="hot-box" scroll-y="true" upper-threshold="50" lower-threshold="100" bindscrolltolower="scrolltolower">
 
      <!-- 列表 -->
      <view class="themes-list">
        <view class="themes-list-box" wx:for="{{datalists}}">
          <view class="themes-list-main">
            <view class="themes-list-name">{{item}}</view>
          </view>
        </view>
      </view>
    </scroll-view>
  </swiper-item>
 
  <!-- grag02 -->
  <swiper-item>
    <scroll-view class="hot-box" scroll-y="true" upper-threshold="50" lower-threshold="100" bindscrolltolower="scrolltolower">
 
      <!-- 列表 -->
      <view class="themes-list">
        <view class="themes-list-box" wx:for="{{reslists}}">
          <view class="themes-list-main">
            <view class="themes-list-name">{{item}}</view>
          </view>
        </view>
      </view>
    </scroll-view>
  </swiper-item>
 
  <!-- grag03 -->
  <swiper-item>
    <scroll-view class="hot-box" scroll-y="true" upper-threshold="50" lower-threshold="100" bindscrolltolower="scrolltolower">
 
      <!-- 列表 -->
      <view class="themes-list">
        <view class="themes-list-box" wx:for="{{datalists}}">
          <view class="themes-list-main">
            <view class="themes-list-name">{{item}}</view>
          </view>
        </view>
      </view>
    </scroll-view>
  </swiper-item>
 
  <!-- grag02 -->
  <swiper-item>
    <scroll-view class="hot-box" scroll-y="true" upper-threshold="50" lower-threshold="100" bindscrolltolower="scrolltolower">
 
      <!-- 列表 -->
      <view class="themes-list">
        <view class="themes-list-box" wx:for="{{reslists}}">
          <view class="themes-list-main">
            <view class="themes-list-name">{{item}}</view>
          </view>
        </view>
      </view>
    </scroll-view>
  </swiper-item>
</swiper>
 
<!--tab_top-->
<view class="swiper-tab">
  <view class="swiper-tab-list {{currentTab==0 ? 'active' : ''}}" data-current="0" bindtap="swichNav">
    <view class="swiper-tab-img"><image class="img" src="{{currentTab==0 ? iconlists[0].focus: iconlists[0].normal}}"></image></view>
    <view>frag01</view>
  </view>
  <view class="swiper-tab-list {{currentTab==1 ? 'active' : ''}}" data-current="1" bindtap="swichNav">
     <view class="swiper-tab-img"><image class="img" src="{{currentTab==1 ? iconlists[1].focus: iconlists[1].normal}}"></image></view>
    <view>frag02</view>
  </view>
  <view class="swiper-tab-list {{currentTab==2 ? 'active' : ''}}" data-current="2" bindtap="swichNav">
     <view class="swiper-tab-img"><image class="img" src="{{currentTab==2 ? iconlists[2].focus: iconlists[2].normal}}"></image></view>
    <view>frag03</view>
  </view>
  <view class="swiper-tab-list {{currentTab==3 ? 'active' : ''}}" data-current="3" bindtap="swichNav">
     <view class="swiper-tab-img"><image class="img" src="{{currentTab==3 ? iconlists[3].focus: iconlists[3].normal}}"></image></view>
    <view>frag04</view>
  </view>
</view>
```

**wxss：**
```
/*swiper*/
.swiper-box {
  display: block;
  height: 100%;
  width: 100%;
  overflow: hidden;
}
.hot-box {
  display: block;
  height: 100%;
  font-family: Helvetica;
}
/* list */
.themes-list {
  background: #fff;
  display: block;
  margin-bottom: 20px;
}
.themes-list-box {
  display: block;
  position: relative;
  padding: 16px 20px;
  border-bottom: 1px solid #f2f2f2;
}
.themes-list-main {
  margin-left: 1px;
}
.themes-list-name {
  font-size: 14px;
  color: #444;
  height: 20px;
  line-height: 20px;
  overflow: hidden;
}
/*tab*/
.swiper-tab {
  height: 50px;
  background: #fff;
  display: flex;
  position: relative;
  z-index: 2;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  border-top: 1px solid #ccc;
}
.swiper-tab-list {
  margin: 0 20px;
  padding: 0 4px;
  font-size: 28rpx;
  font-family: Helvetica;
}
.active {
  /*border-bottom: 1px solid #FFCC00;*/
  color: #FFCC00;
}
.swiper-tab-img {
  text-align: center;
}
.img {
  width:23px;
  height: 23px;
}
```

**js：**
```
Page( {
    data: {
        winWidth: 0,
        winHeight: 0,
        currentTab: 0,       
        datalists : [
            "习近平主持中央财经领导小组第十五次会议",
            "李克强打叉的“万里审批图”成历史",
            "新疆自治区举行反恐维稳誓师大会",
            "朝鲜代表团抵达马来西亚处理金正男遇害案",
            "宝马车祸案肇事者二次精神鉴定:案发为精神病状态",
            "朝鲜代表团抵达马来西亚处理金正男遇害案",
            "宝马车祸案肇事者二次精神鉴定:案发为精神病状态",
            "朝鲜代表团抵达马来西亚处理金正男遇害案",
            "宝马车祸案肇事者二次精神鉴定:案发为精神病状态",
            "朝鲜代表团抵达马来西亚处理金正男遇害案",
            "宝马车祸案肇事者二次精神鉴定:案发为精神病状态",
            "砸锅卖铁！索尼是在走向毁灭 还是在奔向新生？"
        ],
        reslists:["hello","thank you for your read","if u feel good","can u give me good？"],
        iconlists:[
            {normal:"../../images/wp.png",focus:"../../images/wpselect.png"},
            {normal:"../../images/ss.png",focus:"../../images/ssselect.png"},
            {normal:"../../images/hc.png",focus:"../../images/hcselect.png"},
            {normal:"../../images/my.png",focus:"../../images/myselect.png"},
        ]
    },
    onLoad: function( options ) {
        var that = this;
        //获取系统信息
        wx.getSystemInfo( {
            success: function( res ) {
                that.setData( {
                    winWidth: res.windowWidth,
                    winHeight: res.windowHeight
                });
            }
        });
    },
    /**
     * 滑动切换tab
     */
    bindChange: function( e ) {
        var that = this;
        that.setData( { currentTab: e.detail.current });
    },
    /**
     * 点击切换tab
     */
    swichNav: function( e ) {
        console.log(e)
        var that = this;
        if( this.data.currentTab === e.currentTarget.dataset.current ) {
            //点击的是同一个，则不操作
            return false;
        } else {
            that.setData( {
                currentTab: e.currentTarget.dataset.current
            })
        }
 
    }
})
```
原文作者：michael_ouyang
原文链接：https://blog.csdn.net/michael_ouyang/article/details/60333699
