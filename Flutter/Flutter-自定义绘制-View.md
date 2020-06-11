阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)

在 Flutter 中自定义 View 有两种方式:

1.  组合已有控件
2.  自定义绘制

## 如何自定义绘制

有两个类做这件事情:

*   `CustomPaint` :会在绘制阶段提供一个 `Canvas` 画布
*   `CustomPainter` : 具体的画笔, 可配置画笔的颜色,路径等

```
CustomPaint(
  painter: Sky(),
  child: Center(
    child: Text(
      'Once upon a time...',
      style: const TextStyle(
        fontSize: 40.0,
        fontWeight: FontWeight.w900,
        color: Color(0xFFFFFFFF),
      ),
    ),
  ),
)

```

当在绘制阶段时, `CustomPaint` 首先会调用 `painter` 在画布上进行绘制, 然后再绘制它的 `child` 控件, `child` 绘制完成之后会调用 `foregroundPainter` 进行绘制. 画布的坐标系和 `CustomPaint` 的坐标系匹配. `CustomPaint` 有个 `Size` 属性标识将绘制多大的区域, 绘制时这个 `Size` 属性将会传递到 `CustomPainter` 的 `paint` 方法中, 具体的绘制就在`paint` 方法中进行, `void paint(Canvas canvas, Size size);` 的方法签名中的两个参数:

*   canvas: 用于绘制的画布, **注意: 该画布是应用所有控件都在使用的, 所以通过这个画布其实是可以绘制充满屏幕的内容的**
*   size: 表示应该绘制的区域大小, **每次绘制都应该限制在这个区域内, 否则可能会覆盖了其他控件的绘制结果**

## 实例一

绘制一个矩形和圆角:

![](//upload-images.jianshu.io/upload_images/2833342-f87f23d205ea8d9f.png?imageMogr2/auto-orient/strip|imageView2/2/w/400/format/webp)

```
Widget build(BuildContext context) {
  return CustomPaint(
    painter: ColorPainter(),
    size: Size(300, 200),
  );
}

```

```
class ColorPainter extends CustomPainter {

  final red = Color.fromRGBO(255, 0, 0, 1);
  final blue = Color.fromRGBO(0, 0, 255, 1);

  @override
  void paint(Canvas canvas, Size size) {  
    final paint = Paint();
    final rect = Rect.fromLTRB(0.0, 0.0, size.width, size.height);
    // 注意这一句
    canvas.clipRect(rect);
    paint.color = blue;
    canvas.drawRect(rect, paint);
    paint.color = red;
    canvas.drawCircle(Offset.zero, size.height, paint);
  }

  @override
  bool shouldRepaint(CustomPainter oldDelegate) {
    return false;
  }
}

```

*   定义了绘制区域大小, 为 `CustomPaint` 中的 `size` 属性

```
    final rect = Rect.fromLTRB(0.0, 0.0, size.width, size.height);

```

*   绘制矩形区域

```
    canvas.drawRect(rect, paint);

```

*   绘制圆形
    *   圆心偏移量为: 0, 也就是 `CustomPaint` 的原点
    *   半径为区域的高度

```
    canvas.drawCircle(Offset.zero, size.height, paint);

```

**最需要注意**的地方我觉得是 `canvas.clipRect(rect);` 这句. 这句表示只绘制给定的区域中的内容. 如果不写这句, 效果就是这样:

![](//upload-images.jianshu.io/upload_images/2833342-5cd3de322b9fd499.png?imageMogr2/auto-orient/strip|imageView2/2/w/400/format/webp)

> 为什么会这样呢 ?

其实这就是 `Size` 这个参数的重要性, 因为 `Canvas` 是被所有控件公有的, 如果我们绘制时不指定区域大小, 那在进行一些形状的绘制时就会出现超出区域的问题.

作者：stefanJi
链接：https://www.jianshu.com/p/0e0c45f776b9
阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)
