微信小程序页面布局方式采用的是`Flex`布局。
`Flex`布局，是W3c在2009年提出的一种新的方案，可以简便，完整，响应式的实现各种页面布局。
Flex布局提供了元素在容器中的对齐，方向以及顺序，甚至他们可以是动态的或者不确定的大小的。
Flex布局的主要特征是能够调整其子元素在不同的屏幕大小中能够用最适合的方法填充合适的空间。

![](https://upload-images.jianshu.io/upload_images/22188-26c5fea609a6d54c.png?imageMogr2/auto-orient/strip%7CimageView2/2)

Flex布局的特点:

*   任意方向的伸缩，向左，向右，向下，向上
*   在样式层可以调换和重排顺序
*   主轴和侧轴方便配置
*   子元素的空间拉伸和填充
*   沿着容器对齐

微信小程序实现了`Flex`布局,简单介绍下`Flex`布局在微信小程序中的使用。

## 伸缩容器

设有`display:flex`或者`display:block`的元素就是一个`flex container`(伸缩容器)，里面的子元素称为`flex item`(伸缩项目)，`flex container`中子元素都是使用`Flex`布局排版。

*   `display:block` 指定为块内容器模式，总是使用新行开始显示，微信小程序的视图容器(view,scroll-view和swiper)默认都是`dispaly:block`。
*   `display:flex`:指定为行内容器模式，在一行内显示子元素，可以使用`flex-wrap`属性指定其是否换行，`flex-wrap`有三个值:*nowrap(不换行)*,*wrap(换行)*,*wrap-reverse(换行第一行在下面)*
    使用`display:block`(默认值)的代码:

    ```
    <view class="flex-row" style="display: block;">
          <view class="flex-view-item">1</view>
          <view class="flex-view-item">2</view>
          <view class="flex-view-item">3</view>
      </view>
    ```

    显示效果:

![](https://upload-images.jianshu.io/upload_images/22188-017d5c26b63d40cb.png?imageMogr2/auto-orient/strip%7CimageView2/2)

改换成`display:flex`的显示效果:

![](https://upload-images.jianshu.io/upload_images/22188-a63bcc3980f93abf.png?imageMogr2/auto-orient/strip%7CimageView2/2)

可以从效果图看到`block`和`flex`的区别，子元素`view`是在换行显示(`block`)还是行内显示(`flex`)。

## 主轴和侧轴

`Flex`布局的伸缩容器可以使用任何方向进行布局。
容器默认有两个轴：*主轴(main axis)*和*侧轴(cross axis)*。
主轴的开始位置为`主轴起点`(main start)，主轴的结束位置为`主轴终点`(main end),而主轴的长度为`主轴长度`(main size)。
同理侧轴的起点为`侧轴起点`(cross start),结束位置为`侧轴终点`(cross end),长度为`侧轴长度`(cross size)。详情见下图:

![](https://upload-images.jianshu.io/upload_images/22188-bbf58812dfcac77d.png?imageMogr2/auto-orient/strip%7CimageView2/2)

注意，`主轴`并不是一定是`从左到右`的，同理`侧轴`也不一定是`从上到下`，主轴的方向使用`flex-direction`属性控制,它有4个可选值:

*   `row` :从左到右的水平方向为主轴
*   `row-reverse`：从右到左的水平方向为主轴
*   `column`:从上到下的垂直方向为主轴
*   `column-reverse`从下到上的垂直方向为主轴

如果水平方向为主轴，那个垂直方向就是侧轴，反之亦然。
四种主轴方向设置的效果图:

![](https://upload-images.jianshu.io/upload_images/22188-c3cae998d57982ef.png?imageMogr2/auto-orient/strip%7CimageView2/2)

图中的实例展示了使用了不同的`flex-direction`值排列方向的区别。
实例代码:

```
<view >
    <view class="flex-row" style="display: flex;flex-direction: row;">
        <view class="flex-view-item">1</view>
        <view class="flex-view-item">2</view>
        <view class="flex-view-item">3</view>
    </view>
    <view class="flex-column" style="display:flex;flex-direction: column;" >
        <view class="flex-view-item">c1</view>
        <view class="flex-view-item">c2</view>
        <view class="flex-view-item">c3</view>
    </view>
</view>
```

运行效果：

![](https://upload-images.jianshu.io/upload_images/22188-1bdb989fea46fdc1.png?imageMogr2/auto-orient/strip%7CimageView2/2)

## 对齐方式

子元素有两种对齐方式：

> `justify-conent` 定义子元素在主轴上面的对齐方式
> `align-items` 定义子元素在侧轴上对齐的方式

`justify-content`有5个可选的对齐方式:

*   `flex-start` 主轴起点对齐(默认值)
*   `flex-end` 主轴结束点对齐
*   `center` 在主轴中居中对齐
*   `space-between` 两端对齐，除了两端的子元素分别靠向两端的容器之外，其他子元素之间的间隔都相等
*   `space-around` 每个子元素之间的距离相等，两端的子元素距离容器的距离也和其它子元素之间的距离相同。
    `justify-content`的对齐方式和主轴的方向有关，下图以`flex-direction`为`row`，主轴方式是`从左到右`,描述`jstify-content`5个值的显示效果:

    ![](https://upload-images.jianshu.io/upload_images/22188-e843b222e9ae5244.png?imageMogr2/auto-orient/strip%7CimageView2/2)

`align-items`表示侧轴上的对齐方式:

*   `stretch` 填充整个容器(默认值)
*   `flex-start` 侧轴的起点对齐
*   `flex-end` 侧轴的终点对齐
*   `center` 在侧轴中居中对齐
*   `baseline` 以子元素的第一行文字对齐

`align-tiems`设置的对齐方式，和侧轴的方向有关，下图以`flex-direction`为`row`,侧轴方向是`从上到下`,描述`align-items`的5个值显示效果:

![](https://upload-images.jianshu.io/upload_images/22188-b9c64a339a543827.png?imageMogr2/auto-orient/strip%7CimageView2/2)

有了主轴和侧轴的方向再加上设置他们的对齐方式，就可以实现大部分的页面布局了。

源代码地址:[https://github.com/jjz/weixin-mina/blob/master/pages/flex/flex.wxml](https://github.com/jjz/weixin-mina/blob/master/pages/flex/flex.wxml)
原文作者：DragonDean 
原文链接：https://www.cnblogs.com/dragondean/p/5922740.html
