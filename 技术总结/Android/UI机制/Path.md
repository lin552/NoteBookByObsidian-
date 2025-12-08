---
创建时间: "2025-12-04 14:31:43"
作者: wangxiaoming
tags:
---
Path（路径）是 Android 绘图系统中用于定义任意几何形状的核心类，它可以描述直线、曲线、多边形、圆弧等复杂轮廓，配合 Paint（画笔）和 Canvas（画布）即可绘制出丰富图形。Path的强大之处在于灵活性——从简单的矩形到复杂的曲线路径（如波浪线、心形），都能通过它精准定义。

#### 一、Path核心方法
Path通过一系列方法逐步构建路径，就像用笔在纸上画线一样。常用方法如下：

| 方法签名                                                                  | 作用                                              | 示例                                                                   |
| --------------------------------------------------------------------- | ----------------------------------------------- | -------------------------------------------------------------------- |
| `moveTo(float x, float y)`                                            | 移动画笔到指定点（不绘制，作为新路径起点）                           | `path.moveTo(100, 100);`// 移动到 (100,100)                             |
| `lineTo(float x, float y)`                                            | 从当前点绘制直线到目标点                                    | `path.lineTo(200, 200);`// 从当前点画直线到 (200,200)                        |
| `quadTo(float x1, float y1, float x2, float y2)`                      | 绘制二次贝塞尔曲线（控制点 (x1,y1)，终点 (x2,y2)）               | `path.quadTo(150, 50, 200, 200);`// 控制点 (150,50)，终点 (200,200)        |
| `cubicTo(float x1, float y1, float x2, float y2, float x3, float y3)` | 绘制三次贝塞尔曲线（控制点1 (x1,y1)，控制点2 (x2,y2)，终点 (x3,y3)） | `path.cubicTo(150, 50, 250, 50, 200, 200);`// 双控制点曲线                 |
| `arcTo(RectF oval, float startAngle, float sweepAngle)`               | 绘制圆弧（椭圆区域 oval，起始角度 startAngle，扫过角度 sweepAngle） | `path.arcTo(new RectF(100,100,300,300), 0, 90);`// 从0°画90°圆弧         |
| `addRect(RectF rect, Direction dir)`                                  | 添加矩形路径（dir 为顺时针/逆时针，影响填充规则）                     | `path.addRect(new RectF(50,50,200,200), Path.Direction.CW);`// 顺时针矩形 |
| `addCircle(float x, float y, float radius, Direction dir)`            | 添加圆形路径                                          | `path.addCircle(150, 150, 50, Path.Direction.CW);`                   |
| `addPath(Path src)`                                                   | 合并另一个路径                                         | `path.addPath(otherPath);`// 将 otherPath 添加到当前路径                     |
| `close()`                                                             | 闭合路径（从当前点绘制直线回起点）                               | `path.close();`// 形成封闭图形（如三角形）                                       |
#### 二、Path高级功能
##### 1)路径合并（布尔运算）
通过 `op(Path path, Op op)`方法可对两个路径进行**布尔运算**（交集、并集、差集等），类似 Photoshop 的路径操作。
**Op 枚举值**：
- `Path.Op.UNION`：并集（两个路径合并）
- `Path.Op.INTERSECT`：交集（保留重叠部分）
- `Path.Op.DIFFERENCE`：差集（用当前路径减去目标路径）
- `Path.Op.XOR`：异或（保留不重叠部分）

```java
Path circle1 = new Path();
circle1.addCircle(150, 150, 100, Path.Direction.CW); // 圆1

Path circle2 = new Path();
circle2.addCircle(250, 150, 100, Path.Direction.CW); // 圆2

Path intersection = new Path();
intersection.op(circle1, circle2, Path.Op.INTERSECT); // 取交集
```
###### ① 何时用Path.Op优化？
1. **静态路径优先合并**：
    对**不频繁变化的路径**（如背景、固定图标区域），用 `Path.Op`合并后缓存，避免每帧重复计算。
2. **动态路径谨慎使用**：
    对**动画中的动态路径**（如随手势变化的形状），若顶点数量少（<20 个），可合并；若顶点多，优先分开绘制（避免运算耗时超过绘制收益）。
3. **结合 `Path`复用**：
    用 **单个 `Path`对象**​ 存储合并后的结果，避免频繁创建新对象（参考之前的“对象复用”优化）。
4. **性能测试验证**：
    通过 `Systrace`或 `Profile GPU Rendering`工具对比合并前后的帧耗时，确保优化有效。
##### 2)路径变换（平移/旋转/缩放）
通过 `transform(Matrix matrix)`方法可对路径应用矩阵变换（平移、旋转、缩放、倾斜），实现路径的动态变形。

```java
Matrix matrix = new Matrix();
matrix.postTranslate(100, 50); // 向右平移100px，向下平移50px
path.transform(matrix); // 应用变换
```

##### 3)路径测量（PathMeasure）
PathMeasure用于测量路径的长度、获取路径上的点坐标，常用于路径动画（如沿着路径移动的小球）。
核心方法：
•`setPath(Path path, boolean forceClosed)`：绑定路径（forceClosed 是否强制闭合路径）
•`getLength()`：获取路径总长度
•`getPosTan(float distance, float[] pos, float[] tan)`：获取路径上指定距离处的坐标（pos）和切线方向（tan）

```java
PathMeasure measure = new PathMeasure(path, false); // 绑定路径（不强制闭合）
float length = measure.getLength(); // 路径总长度
float[] pos = new float[2]; // 存储坐标
float[] tan = new float[2]; // 存储切线方向（可选）
measure.getPosTan(length * 0.5f, pos, tan); // 获取50%位置的坐标
Log.d("Path", "中点坐标：(" + pos[0] + "," + pos[1] + ")");
```

##### 4)填充规则（FillType）
当路径自相交（如五角星）或有孔洞时，`FillType`决定**哪些区域需要填充**。常用两种规则：

|规则|说明|适用场景|
|---|---|---|
|`Path.FillType.WINDING`（非零环绕数）|默认规则，根据路径方向（顺时针/逆时针）计算环绕次数，非零则填充|大多数封闭图形|
|`Path.FillType.EVEN_ODD`（奇偶规则）|从点向外引射线，穿过路径奇数次则填充，偶数次则不填充|自相交图形（如镂空图案）|
```java
path.setFillType(Path.FillType.EVEN_ODD); // 使用奇偶规则
```

#### 四、Path与Canvas的配合绘制
Path本身不绘制图形，需通过 Canvas的绘制方法将其呈现，常用方法：

|Canvas 方法|作用|
|---|---|
|`drawPath(Path path, Paint paint)`|绘制路径轮廓（描边）|
|`drawPath(Path path, Paint paint)`（配合 `Paint.Style.FILL`）|填充路径内部|
|`clipPath(Path path)`|用路径裁剪画布（仅显示路径内区域）|
```java
// 1. 创建路径（心形）
Path heartPath = new Path();
heartPath.moveTo(150, 100);
heartPath.cubicTo(100, 50, 50, 100, 100, 150); // 左半边曲线
heartPath.cubicTo(150, 200, 200, 150, 150, 100); // 右半边曲线
heartPath.close();

// 2. 创建画笔（填充红色）
Paint heartPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
heartPaint.setColor(Color.RED);
heartPaint.setStyle(Paint.Style.FILL);

// 3. 绘制路径
canvas.drawPath(heartPath, heartPaint);
```

#### 五、Path性能与注意事项
##### 1）避免频繁修改Path
Path的修改（如 lineTo、quadTo）会触发内部数据结构重建，高频修改（如每帧动画）可能导致性能问题。优化方案：
•静态路径（不变的形状）在 `onBoundsChange()`中初始化一次，复用实例；
•动态路径（如动画中的变形）用 PathMeasure或矩阵变换替代频繁重建。

##### 2)路径缓存
对复杂路径（如文字轮廓、图标），可预先生成 `Path`并缓存，避免重复计算。例如：
```java
private Path mCachedPath; // 缓存路径

@Override
protected void onBoundsChange(Rect bounds) {
    super.onBoundsChange(bounds);
    if (mCachedPath == null) {
        mCachedPath = createComplexPath(bounds.width(), bounds.height()); // 预生成路径
    }
}

private Path createComplexPath(int w, int h) {
    Path path = new Path();
    // 构建复杂路径（如波浪线、星形）
    return path;
}

@Override
public void draw(Canvas canvas) {
    canvas.drawPath(mCachedPath, mPaint); // 复用缓存路径
}
```

##### 3)硬件加速兼容性
Path的大部分方法（如 lineTo、quadTo）在硬件加速下完全支持，但部分高级操作（如 op()布尔运算）可能在低版本系统（API < 19）有性能问题。建议：
•复杂路径优先用软件绘制（View.setLayerType(LAYER_TYPE_SOFTWARE, null)）；
•测试不同设备的渲染效果，避免兼容性问题。

