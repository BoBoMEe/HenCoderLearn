# 概述

Paint 的完全功能的掌握，可以做出一些更加细致、炫酷的效果，本篇接上篇

# Paint 相关API

Paint 的 API 大致可以分为 4 类：

- 颜色 类
- 效果 类
- drawText() 相关
- 初始化

# 颜色

Canvas 绘制的内容，有三层对颜色的处理：

1. 基本颜色
2. ColorFilter
3. Xfermode

## 基本颜色

- drawColor/RGB/ARGB()：参数提供
- drawBitmap()：Bitmap参数的像素颜色提供
- 图形和文字的绘制，由Paint 属性设置的颜色（drawText,drawPath）

Paint 设置颜色的方法有两种：
> 1. 通过 Paint.setColor/ARGB() 来 `直接设置颜色`，
> 2. 使用 Shader 来指定着色方案

### 直接设置颜色

- setColor(int color)
- setARGB(int a, int r, int g, int b)

### setShader(Shader shader)

Shader是一个着色器，当设置Shader之后，就不使用 setColor/ARGB() 设置的颜色了。
而是使用 Shader 的方案中的颜色。

### Shader家族

- LinearGradient 线性渐变，
- RadialGradient 辐射渐变
- SweepGradient 扫描渐变
- BitmapShader 用 Bitmap 来着色，用 Bitmap 的像素来作为图形或文字的填充
> 如果想画圆形的Bitmap，不用drawBitmap了，可以使用drawCircle + BitmapShader

- ComposeShader 混合着色器，
> ComposeShader 在硬件加速下是不支持两个相同类型的 Shader 的

### ComposeShader(Shader shaderA, Shader shaderB, PorterDuff.Mode mode)

PorterDuff.Mode 是两个 Shader 的 结合方式，一共有 17 个，分为两类

- Alpha 合成 (Alpha Compositing):12种
- 混合 (Blending)：有5种，类似PhotoShop中的那些混合模式

官方文档介绍：[PorterDuff.Mode](https://developer.android.com/reference/android/graphics/PorterDuff.Mode.html)


## ColorFilter

直接设置颜色后，还可以通过 setColorFilter 来为绘制设置颜色过滤。

通常我们使用的是 ColorFilter 的子类，如
- LightingColorFilter （模拟简单的光照效果）
- PorterDuffColorFilter （使用一个指定的颜色和一种指定的 PorterDuff.Mode 来与绘制对象进行合成）
- ColorMatrixColorFilter （ColorMatrix 来对颜色进行处理）

关于 ColorMatrix 更多介绍：[Android 中的矩阵 (一)：ColorMatrix](https://juejin.im/entry/5801ffbfbf22ec0064d292d9)

## setXfermode

除了设置基础颜色和颜色过滤后，还有最后一层处理颜色的办法，setXfermode选取一个 PorterDuff.Mode 作为绘制内容的颜色处理方案。

综上 PorterDuff.Mode 的使用场景有三：

- ComposeShader：混合两个 Shader
- PorterDuffColorFilter：增加一个单色的 ColorFilter
- Xfermode：设置绘制内容和 View 中已有内容的混合计算方式

> 其中PorterDuffXfermode是Xfermode的唯一子类，其实也有别的子类，但现在都deprecated了。

## Xfermode 注意事项

### Xfermode离屏缓冲

把内容绘制在额外的层上，再把绘制好的内容贴回 View 中，避免出现奇怪的结果了。

使用离屏缓冲有两种方式：

- Canvas.saveLayer()

```java
int saved = canvas.saveLayer(null, null, Canvas.ALL_SAVE_FLAG);

canvas.drawBitmap(rectBitmap, 0, 0, paint); // 画方
paint.setXfermode(xfermode); // 设置 Xfermode
canvas.drawBitmap(circleBitmap, 0, 0, paint); // 画圆
paint.setXfermode(null); // 用完及时清除 Xfermode

canvas.restoreToCount(saved);
```

- View.setLayerType()

```java
 View.setLayerType() //是直接把整个 View 都绘制在离屏缓冲中。
 setLayerType(LAYER_TYPE_HARDWARE) //是使用 GPU 来缓冲，
 setLayerType(LAYER_TYPE_SOFTWARE) //是直接直接用一个 Bitmap 来缓冲
```

### 控制好透明区域

透明区域不要太小，要让它足够覆盖到要和它结合绘制的内容

# 效果类

效果类的 API ，指的就是抗锯齿、填充/轮廓、线条宽度等等这些。

1. 抗锯齿、（setAntiAlias）
2. 填充/轮廓、（setStyle）
3. 线条形状（setStrokeWidth，setStrokeCap，setStrokeJoin，setStrokeMiter）
4. 色彩优化
-  setDither：设置图像的抖动，抖动更多的作用是在图像降低色彩深度绘制时，避免出现大片的色带与色块.
-  setFilterBitmap：设置是否使用双线性过滤来绘制 Bitmap，优化 Bitmap 放大绘制的效果。

5. setPathEffect

-  使用 PathEffect 来给图形的轮廓设置效果，对 Canvas 所有的图形绘制有效。
-  Android 中包含 6 种 PathEffect：

> 四种单一效果：
CornerPathEffect ,DiscretePathEffect,DashPathEffect,PathDashPathEffect

> 两种组合效果：
SumPathEffect,ComposePathEffect

```java
1. Canvas.drawLine() 和 Canvas.drawLines()画直线时，setPathEffect()不支持硬件加速。
2. PathDashPathEffect 对硬件加速的支持也有问题
```

6. setShadowLayer : 绘制内容下面加一层阴影

-  setShadowLayer() 需要关闭硬件加速才能正常绘制阴影
-  如果 shadowColor 是半透明的，阴影的透明度就使用 shadowColor 自己的透明度；而如果 shadowColor 是不透明的，阴影的透明度就使用 paint 的透明度

7. setMaskFilter：与setShadowLayer相反，在绘制层上方的附加效果

比如常见的
- BlurMaskFilter： 高斯模糊效果 MaskFilter,

> BlurMaskFilter(float radius, BlurMaskFilter.Blur style):其中，第三个参数的效果如下所示

- EmbossMaskFilter：浮雕效果的 MaskFilter

![](../images/blur_style.png)

8. 获取绘制的 Path

- getFillPath(Path src, Path dst):通过src计算出实际 Path，然后把结果保存在 dst 里
- getTextPath：获取文字的path，Canvas.drawText实际是将文字转化为path

Note：主要是用于图形和文字的装饰效果的位置计算

![](../images/fill_path.png)


# drawText() 相关

> 下一节

# 初始化类

- reset(): 重置 Paint
- set(Paint src):把 src 的所有属性全部复制过来
- setFlags(int flags):批量设置 flags

文章练习：https://github.com/BoBoMEe/PracticeDraw2










