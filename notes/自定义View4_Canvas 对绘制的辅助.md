# 概述

绘制辅助分为如下几个方面：

- 范围裁切
- 几何变换

# 范围裁切

分为`clipRect()` 和 `clipPath()`,裁切方法之后的绘制代码，都会被限制在裁切范围内。

## clipRect

注意：及时`Canvas.save()` 和 `Canvas.restore()` 来保存和恢复绘制范围。

## clipPath

将`Rect`换成了 `Path`，能裁切的形状更多

# 几何变换

- Canvas 二维变化
- Matrix 二维变换
- Camera 三维变换

## Canvas

- Canvas.translate
- Canvas.rotate
- Canvas.scale
- Canvas.skew


## Matrix

### 一般变换：

1. 创建 Matrix
2. pre/postTranslate/Rotate/Scale/Skew:设置几何变换
3. Canvas.setMatrix(matrix)/Canvas.concat(matrix):把 `Matrix` 应用到 `Canvas`

> setMatrix 是用新的 `Matrix` 直接替换 `Canvas` 当前的变换矩阵
> concat 基于 Canvas 当前的变换，叠加上 Matrix 中的变换

### 自定义变换

使用方法：Matrix.setPolyToPoly，多点映射来直接设置变换，让绘制内容任意地扭曲。

## Camera

- 旋转
- 平移
- 移动相机

### 旋转

> 需要注意的是，相机的操作都是基于原点的，因此需要操作对称需要配合 `Canvas.translate()`
> Canvas 的几何变换顺序是反的,把移动到中心的代码写在下面，把从中心移动回来的代码写在上面.

### 平移

使用的是 `Camera.translate`

### 移动相机

使用的是 `Camera.setLocation`,可以避免「糊脸」效果

> 需要注意的是，这里的参数是 inch
> Camera中，相机的默认位置是 `(0, 0, -8)(英寸)`，即 `(0, 0, -576)（像素)`