# 自定义绘制概述

1. 重写绘制方法 onDraw()
2. Canvas绘制类方法，Canvas辅助类方法
3. 使用不同的绘制方法来控制遮盖关系

# 自定义绘制的四个级别

1. Canvas 的 drawXXX()
2. Paint 的完全攻略
3. Canvas 绘制辅助,范围裁切和几何变换
4. 控制绘制顺序

# onDraw()： 一切的开始

```java
Paint paint = new Paint();

@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);

    // 绘制一个圆
    canvas.drawCircle(300, 300, 200, paint);
}
```

## Canvas

Canvas：draw- 打头的方法，如drawCircle() drawBitmap()

## Paint

- Paint.setStyle(Style style) 设置绘制模式
- Paint.setColor(int color) 设置颜色
- Paint.setStrokeWidth(float width) 设置线条宽度
- Paint.setTextSize(float textSize) 设置文字大小
- Paint.setAntiAlias(boolean aa) 设置抗锯齿开关

绘制方法中的 `独有信息` 都是作为参数传递到 `draw-`方法中的，
`公有信息`则是统一放在 `paint` 参数里的

## Path

用于绘制自定义图形。
两类作用：一类是直接描述路径，二类是辅助的设置或计算。

### 直接描述路径

- addXxx();添加子图形

> canvas.drawCircle() == path.AddCircle(x, y, radius, dir) + canvas.drawPath(path, paint);

- xxxTo();画线（直线或曲线）

> lineTo(float x, float y): 绝对坐标
> rLineTo(float x, float y): 相对坐标
> quadTo/rQuadTo: 二次贝塞尔曲线
> cubicTo/rCubicTo: 三次贝塞尔曲线
> moveTo: 移动到目标位置
> arcTo()/addArc():比起Canvas.drawArc()，
少一个`useCenter`参数，只用来画弧形而不画扇形，所以不再需要
多了一个`forceMoveTo`参数，是要「抬一下笔移动过去」,还是「直接拖着笔过去」
> close():封闭当前子图形，Paint.Style == FILL | FILL_AND_STROKE 的时候，会自动封闭

### 辅助的设置或计算

使用场景比较少，这里只讲了其中一个方法：

- Path.setFillType(Path.FillType ft)：设置填充方式

包含四个值：

- EVEN_ODD
- WINDING （默认值）
- INVERSE_EVEN_ODD
- INVERSE_WINDING

使用图例

![FillType](imgs/fill_type.png)

文章练习：https://github.com/BoBoMEe/PracticeDraw1


