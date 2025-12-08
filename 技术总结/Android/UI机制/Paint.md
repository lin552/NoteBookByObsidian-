---
创建时间: "2025-12-04 14:15:11"
作者: wangxiaoming
tags:
---
Paint（画笔）是 Android 绘图系统的核心类之一，用于定义绘制的样式和行为（如颜色、线条粗细、字体、模糊效果等）。无论是自定义 View、Drawable，还是使用 Canvas绘制图形/文本，Paint都是控制绘制效果的关键。

#### 一、Paint核心属性与方法
##### 1)基础样式属性
| 属性/方法                      | 作用                                        | 示例代码                                                                                                                                                     |
| -------------------------- | ----------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `setAntiAlias(boolean)`    | 开启抗锯齿（边缘更光滑，必开！）                          | `paint.setAntiAlias(true);`                                                                                                                              |
| `setStyle(Style style)`    | 设置图形样式（填充/描边/两者）                          | `paint.setStyle(Paint.Style.FILL);`// 填充（默认）  <br>`paint.setStyle(Paint.Style.STROKE);`// 描边  <br>`paint.setStyle(Paint.Style.FILL_AND_STROKE);`// 填充+描边 |
| `setStrokeWidth(float)`    | 设置描边宽度（仅在 `STROKE`或 `FILL_AND_STROKE`时有效） | `paint.setStrokeWidth(dp2px(2));`// 2dp 描边                                                                                                               |
| `setStrokeCap(Cap cap)`    | 设置线条端点样式（圆头/方头/平头）                        | `paint.setStrokeCap(Paint.Cap.ROUND);`// 圆头端点                                                                                                            |
| `setStrokeJoin(Join join)` | 设置线条拐角样式（圆角/斜角/尖角）                        | `paint.setStrokeJoin(Paint.Join.ROUND);`// 圆角拐角                                                                                                          |
##### 2）颜色和着色
|属性/方法|作用|示例代码|
|---|---|---|
|`setColor(int color)`|设置纯色（如 `Color.RED`、`0xFFFF0000`）|`paint.setColor(Color.parseColor("#FF5722"));`// 橙色|
|`setARGB(int a, int r, int g, int b)`|分别设置透明度（0-255）和 RGB 值|`paint.setARGB(128, 255, 0, 0);`// 半透明红色|
|`setShader(Shader shader)`|设置着色器（渐变/纹理，优先级高于纯色）|`paint.setShader(new LinearGradient(0,0,100,0, Color.RED, Color.BLUE, Shader.TileMode.CLAMP));`// 线性渐变|
##### 3）文本绘制属性
|属性/方法|作用|示例代码|
|---|---|---|
|`setTextSize(float textSize)`|设置字体大小（单位：像素，建议用 `sp2px()`转换）|`paint.setTextSize(sp2px(14));`// 14sp 字体|
|`setTextAlign(Align align)`|设置文本对齐方式（左/中/右，相对于绘制起点）|`paint.setTextAlign(Paint.Align.CENTER);`// 居中对齐|
|`setTypeface(Typeface typeface)`|设置字体（默认/粗体/斜体/自定义字体）|`paint.setTypeface(Typeface.DEFAULT_BOLD);`// 粗体|3
##### 4）特殊效果
|属性/方法|作用|示例代码|
|---|---|---|
|`setMaskFilter(MaskFilter filter)`|设置遮罩滤镜（模糊/浮雕效果）|`paint.setMaskFilter(new BlurMaskFilter(10, BlurMaskFilter.Blur.NORMAL));`// 模糊效果|
|`setShadowLayer(float radius, float dx, float dy, int color)`|设置阴影（半径/偏移量/颜色）|`paint.setShadowLayer(5, 2, 2, Color.GRAY);`// 灰色阴影，偏移(2,2)|
|`setPathEffect(PathEffect effect)`|设置路径效果（虚线/圆角线等）|`paint.setPathEffect(new DashPathEffect(new float[]{10, 5}, 0));`// 虚线（10px实线+5px空白）|
#### 三、Paint与Canvas的配合
Paint需配合 Canvas（画布）使用，Canvas提供绘制方法（如`drawRect`、`drawCircle`），Paint控制绘制样式。
```java
// 1. 创建画笔
Paint paint = new Paint(Paint.ANTI_ALIAS_FLAG); // 构造时直接开启抗锯齿
paint.setColor(Color.RED); // 红色
paint.setStyle(Paint.Style.FILL); // 填充
paint.setStrokeWidth(0); // 填充模式下宽度无效

// 2. 创建画布（假设在自定义View的draw方法中）
Canvas canvas = ...; 

// 3. 绘制图形（圆角矩形）
RectF rect = new RectF(50, 50, 200, 150); // 位置和尺寸
float cornerRadius = 20; // 圆角半径
canvas.drawRoundRect(rect, cornerRadius, cornerRadius, paint); // 用paint控制样式
```
#### 四、高级用法：着色器（Shader）与颜色过滤器（ColorFilter）
##### 1)着色器（Shader）:实现渐变/纹理
Shader是 Paint的高级特性，用于实现非纯色的填充效果，常见子类：
•LinearGradient：线性渐变（如从左到右红→蓝）
•RadialGradient：径向渐变（如从中心向外扩散）
•SweepGradient：扫描渐变（如雷达图效果）
•BitmapShader：用 Bitmap 纹理填充（如马赛克效果）

```java
// 创建线性渐变（起点(0,0)，终点(width,0)，颜色红→蓝，平铺模式CLAMP）
LinearGradient gradient = new LinearGradient(
    0, 0, width, 0,
    new int[]{Color.RED, Color.BLUE},
    null, // 颜色分布比例（null表示均匀分布）
    Shader.TileMode.CLAMP // 超出渐变区域时拉伸边缘色
);
paint.setShader(gradient); // 应用到画笔
canvas.drawRect(0, 0, width, height, paint); // 绘制渐变矩形
```

##### 2)颜色过滤器（ColorFilter）:修改颜色
ColorFilter用于动态修改绘制内容的颜色，常见子类：
•LightingColorFilter：模拟光照效果（如变暗/变亮）
•PorterDuffColorFilter：使用 `PorterDuff` 混合模式（如 tint 着色）
•ColorMatrixColorFilter：通过矩阵精确调整 RGBA 通道

```java
// 创建颜色过滤器（源色为黑色，混合模式为SRC_IN，目标色为蓝色）
ColorFilter colorFilter = new PorterDuffColorFilter(Color.BLUE, PorterDuff.Mode.SRC_IN);
paint.setColorFilter(colorFilter); 
canvas.drawBitmap(iconBitmap, x, y, paint); // 图标会被染成蓝色
```
#### 五、使用注意事项
##### 1)避免频繁创建Paint对象
Paint对象创建成本较低，但在 draw()等高频方法中反复创建会导致内存抖动（GC 频繁）。建议在构造函数或 `onBoundsChange()`中初始化，复用同一个实例。
```java
private Paint mPaint; // 成员变量复用

public MyDrawable() {
    mPaint = new Paint(Paint.ANTI_ALIAS_FLAG); // 构造时初始化
    mPaint.setColor(Color.RED);
}

@Override
public void draw(Canvas canvas) {
    canvas.drawRect(rect, mPaint); // 复用mPaint
}
```

##### 2)硬件加速兼容性
部分 `Paint`效果在**硬件加速**下可能失效（如 `BlurMaskFilter`、`setShadowLayer`），表现为效果不显示或异常。解决方法：
- 检测硬件加速状态：`canvas.isHardwareAccelerated()`
- 局部关闭硬件加速：在 View 中设置 `android:layerType="software"`（仅对该 View 生效）

##### 3）抗锯齿必开
Paint.ANTI_ALIAS_FLAG是绘制平滑图形的基础，几乎所有场景下都应开启（除非刻意追求像素风格）。构造时可简写：
```java
Paint paint = new Paint(Paint.ANTI_ALIAS_FLAG); // 等价于 setAntiAlias(true)
```

##### 4)清除效果用null
当需要移除 `Paint`的某个效果（如渐变、阴影）时，用 `null`重置：
```java
paint.setShader(null); // 清除渐变
paint.setMaskFilter(null); // 清除模糊效果
paint.setColorFilter(null); // 清除颜色过滤
```