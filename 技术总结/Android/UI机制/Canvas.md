---
创建时间: "2025-06-19 19:13:51"
作者: wangxiaoming
tags:
---
在 Android 开发中，`Canvas` 和**矩阵变换**是图形绘制与视图渲染的核心技术，广泛用于自定义 View、图片处理、动画效果等场景。以下是其核心考点及详细解析：

#### 一、Canvas的核心概念与考点
`Canvas` 是 Android 图形绘制的核心类，负责管理绘图状态（如颜色、画笔、变换矩阵）并执行具体的绘制操作。其核心考点包括：
##### **1. Canvas 的基本作用**​
- ​**绘图上下文**​：封装了绘制所需的全部状态（如当前画笔 `Paint`、变换矩阵、裁剪区域等）。
- ​**绘制操作入口**​：通过 `drawXXX()` 系列方法（如 `drawBitmap()`、`drawRect()`、`drawText()`）将图形渲染到目标（如 `Bitmap` 或屏幕）。
##### 2.Canvas的关键方法
| 方法                                                                        | 作用                            | 考点                                                |
| ------------------------------------------------------------------------- | ----------------------------- | ------------------------------------------------- |
| `Canvas(Bitmap bitmap)`                                                   | 基于 `Bitmap` 创建 Canvas（用于离屏绘制） | 离屏绘制的核心工具（如缓存复杂 View 的绘制结果）。                      |
| `save()`/`restore()`                                                      | 保存/恢复绘图状态（通过栈管理）              | 避免变换或样式修改影响后续绘制（必考点：状态栈的使用）。                      |
| `drawBitmap(Bitmap, x, y, Paint)`                                         | 绘制位图到指定坐标                     | 参数含义（`x,y` 是位图左上角的位置）、`Paint` 的过滤效果（如抗锯齿）。        |
| `drawRect(float left, float top, float right, float bottom, Paint paint)` | 绘制矩形区域                        | 坐标系的理解（左上角为原点，`right` > `left`，`bottom` > `top`）。 |
| `drawText(String text, float x, float y, Paint paint)`                    | 绘制文本                          | 文本基线（`y` 是文本基线的位置）、`Paint` 的字体与对齐方式。              |
##### 3.Canvas的坐标系
- **原点**​：默认在 View 的左上角（`(0,0)`）。
- ​**方向**​：`x` 轴向右延伸，`y` 轴向下延伸（与数学坐标系相反）。
- ​**变换影响**​：旋转、缩放等操作会改变坐标系的基准（如旋转后，`x` 轴方向可能变为斜向）。

#### 二、矩阵变换的核心考点
矩阵变换是 `Canvas` 实现几何效果（平移、旋转、缩放等）的底层机制。Android 中通过 `Matrix` 类表示变换矩阵，其核心考点包括：
##### 1.矩阵的数学基础
​**变换矩阵**​：`3x3` 矩阵（仿射变换矩阵），形式为：
```
[ a, b, c ]   // 控制 x 轴变换（缩放、倾斜、平移）
[ d, e, f ]   // 控制 y 轴变换（缩放、倾斜、平移）
[ 0, 0, 1 ]   // 固定值（仿射变换保持平行性）
```
- ​**变换顺序**​：矩阵乘法满足结合律但不满足交换律（`AB ≠ BA`），因此变换顺序直接影响最终效果（如先平移后旋转 ≠ 先旋转后平移）。
##### 2.常见变化类型与矩阵
|变换类型|数学表达式|效果描述|关键方法（Matrix）|
|---|---|---|---|
|​**平移（Translate）​**​|`x' = x + tx`  <br>`y' = y + ty`|将图形沿 `x`/`y` 轴移动指定距离|`postTranslate(tx, ty)`|
|​**旋转（Rotate）​**​|`x' = x·cosθ - y·sinθ`  <br>`y' = x·sinθ + y·cosθ`|绕原点（或指定点）旋转 `θ` 角度（弧度制）|`postRotate(deg, px, py)`|
|​**缩放（Scale）​**​|`x' = x·sx`  <br>`y' = y·sy`|沿 `x`/`y` 轴缩放（`sx/sy` 为缩放因子，1.0 为原始大小，2.0 为放大 1 倍）|`postScale(sx, sy, px, py)`|
|​**错切（Skew）​**​|`x' = x + y·skewX`  <br>`y' = y + x·skewY`|沿 `x`/`y` 轴倾斜（`skewX`/`skewY` 为倾斜因子）|`postSkew(skewX, skewY)`|
##### 3.Matrix的操作模式
`Matrix` 提供两种变换应用方式，需重点区分：
- ​**`post()` 方法**​：将当前变换**追加**到现有变换之后（矩阵右乘，变换顺序为从右到左）。  
    示例：`matrix.postTranslate(10, 20).postRotate(30)` 等价于先旋转 30°，再平移 (10,20)。
- ​**`pre()` 方法**​：将当前变换**插入**到现有变换之前（矩阵左乘，变换顺序为从左到右）。  
    示例：`matrix.preRotate(30).preTranslate(10, 20)` 等价于先平移 (10,20)，再旋转 30°。

##### 4.Canvas与Matrix的协作
通过 `Canvas.setMatrix(Matrix)` 或 `Canvas.concat(Matrix)` 应用变换：
- `setMatrix(Matrix)`：直接覆盖 Canvas 当前的变换矩阵（谨慎使用，可能丢失原有状态）。
- `concat(Matrix)`：将当前变换矩阵与传入的矩阵**相乘**​（等价于 `post()`，保留原有变换）。

#### 三、实际开发中的高频考点
##### 1.实现镜像效果（水平/垂直翻转）
- ​**水平镜像**​：`Matrix.setScale(-1, 1)`，并通过 `postTranslate(width, 0)` 调整位置（避免内容被裁剪）。
- ​**垂直镜像**​：`Matrix.setScale(1, -1)`，并通过 `postTranslate(0, height)` 调整位置。
示例代码：
```kotlin
// 水平镜像绘制 Bitmap
val matrix = Matrix().apply {
    setScale(-1f, 1f) // 水平翻转
    postTranslate(bitmap.width.toFloat(), 0f) // 平移避免裁剪
}
canvas.drawBitmap(bitmap, matrix, null)
```
##### 2.图片旋转并保持中心点
直接旋转会导致图片偏移，需先将原点移至中心，旋转后再移回：
```kotlin
val centerX = bitmap.width / 2f
val centerY = bitmap.height / 2f
val matrix = Matrix().apply {
    postTranslate(-centerX, -centerY) // 移至原点
    postRotate(30f)                   // 旋转 30°
    postTranslate(centerX, centerY)   // 移回原位
}
canvas.drawBitmap(bitmap, matrix, null)
```
#### 3.组合变换（如旋转+缩放）
需注意变换顺序对结果的影响：
```kotlin
// 先缩放后旋转（缩放会影响旋转的中心）
val matrix = Matrix().apply {
    postScale(2f, 2f)   // 放大 2 倍
    postRotate(45f)     // 旋转 45°（基于放大后的坐标系）
}

// 先旋转后缩放（旋转后的坐标会被缩放）
val matrix = Matrix().apply {
    postRotate(45f)     // 旋转 45°
    postScale(2f, 2f)   // 放大 2 倍（基于旋转后的坐标系）
}
```
##### 4.性能优化：避免重复计算
- ​**缓存变换后的 Bitmap**​：对静态内容的变换（如固定的图标旋转），可提前生成变换后的 `Bitmap` 并缓存，避免每次绘制重复计算。
- ​**减少矩阵操作**​：复杂的变换（如多次旋转+缩放）可通过合并矩阵减少计算量（`Matrix` 的 `post()` 方法会自动合并）。
#### 四、总结：核心考点清单
|考点分类|具体内容|
|---|---|
|Canvas 基础|绘制方法（`drawBitmap`、`drawRect`、`drawText`）、状态管理（`save()`/`restore()`）|
|矩阵变换原理|仿射变换矩阵的数学表示、变换顺序（`pre()`/`post()` 的区别）|
|常见变换实现|平移、旋转、缩放、错切的具体代码实现|
|实际应用场景|镜像效果、图片旋转居中、组合变换（旋转+缩放）|
|性能优化|缓存变换后的 Bitmap、减少重复矩阵计算|
