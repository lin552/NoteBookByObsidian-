---
创建时间: "2025-12-04 13:57:05"
作者: wangxiaoming
tags:
---
#### 一、基础概念
`Drawable`是 Android 中用于**绘制图形/图像**的抽象基类（类似 iOS 的 `CALayer`），比 `View`更轻量，专注于**2D 渲染**。它可以直接作为 `View`的背景（`setBackground()`）、`ImageView`的图片（`setImageDrawable()`），或独立绘制到 `Canvas`。

**核心特点**：
- 轻量级：不包含布局逻辑，仅负责绘制。
- 可复用：同一 Drawable 实例可设置给多个 View。
- 支持动画：可通过 `ValueAnimator`动态修改属性实现动画。
##### 1）常见Drawable子类
| 类名                 | 用途                | 示例场景                |
| ------------------ | ----------------- | ------------------- |
| `BitmapDrawable`   | 显示图片              | ImageView 加载本地/网络图片 |
| `ShapeDrawable`    | 绘制几何图形（矩形、圆形等）    | 按钮背景、标签形状           |
| `GradientDrawable` | 渐变图形（线性/径向渐变）     | 进度条、卡片背景            |
| `VectorDrawable`   | 矢量图（XML 定义，缩放不失真） | 图标、简单插画             |
#### 二、自定义Drawable
##### 1)onBoundsChange(Rect bounds) :处理尺寸变化
当 Drawable 被设置给 View 时，系统会调用此方法通知尺寸变化（`bounds`即绘制区域）。需在此处记录宽高，并初始化依赖尺寸的变量（如路径、缓存）。
```java
@Override
protected void onBoundsChange(Rect r) {
    super.onBoundsChange(r);
    width = r.width();   // 绘制区域宽度
    height = r.height(); // 绘制区域高度
    bgPath = createBgPath(width, height); // 初始化背景路径（避免每帧重建）
}
```

##### 3)draw(Canvas canvas):核心绘制逻辑
在此方法中通过 `Canvas`和 `Paint`绘制图形。这是自定义 Drawable 的**灵魂**，需根据需求组织绘制顺序（背景→中间层→前景）。
```java
@Override
public void draw(Canvas canvas) {
    // 1. 保存画布状态（避免后续绘制影响）
    canvas.save();
    
    // 2. 平移画布到绘制中心（方便计算坐标）
    canvas.translate(width/2f, height/2f);
    
    // 3. 绘制背景（渐变/纯色）
    drawBg(canvas);
    
    // 4. 绘制3D卡片层（含多层阴影和位图）
    draw3DCard(canvas);
    
    // 5. 恢复画布状态
    canvas.restore();
}
```
##### 4)状态管理
需实现以下方法以支持透明度、颜色过滤等特性：
- `setAlpha(int alpha)`：设置整体透明度（0-255）。
- `setColorFilter(ColorFilter filter)`：设置颜色过滤（如 tint 着色）。
- `getOpacity()`：返回透明度类型（如 `PixelFormat.TRANSLUCENT`半透明）。

```java
@Override
public void setAlpha(int alpha) {
    this.alpha = alpha; // 保存透明度
    paint.setAlpha(alpha); // 应用到画笔
    invalidateSelf(); // 触发重绘
}

@Override
public int getOpacity() {
    return PixelFormat.TRANSLUCENT; // 半透明（根据实际需求调整）
}
```
##### 5)动画支持
通过 `ValueAnimator`动态修改 Drawable 的属性（如位置、颜色、透明度），实现动画效果。例如 `MusicRecommendDrawable`中的入场动画：
```java
ValueAnimator animator = ValueAnimator.ofFloat(100, 0); // 从100px偏移到0
animator.addUpdateListener(animation -> {
    initOffset1 = (float) animation.getAnimatedValue(); // 更新偏移量
    invalidateSelf(); // 重绘
});
animator.start();
```
#### 三、自定义复杂Drawable注意事项
##### 1）性能第一：避免高频创建对象
- **不要在 `draw()`中创建对象**（如 `new Paint()`、`new Path()`），应在构造函数或 `onBoundsChange()`中初始化。
- **复用 Bitmap**：如需临时绘制（如阴影），用 `BitmapPool`复用 Bitmap，避免频繁 `createBitmap()`/`recycle()`（易引发 GC 卡顿）。
##### 2）绘制顺序：从后往前
遵循 **“背景→中间层→前景”**​ 顺序，类似 Photoshop 图层叠加：
```java
void draw(Canvas canvas) {
    drawBg(canvas);       // 最底层（背景）
    drawShadow(canvas);   // 中间层（阴影）
    drawCard(canvas);     // 最上层（卡片主体）
}
```
##### 3)离屏缓存（Offscreen Buffer）合理使用
`canvas.saveLayerAlpha()`可创建离屏缓冲（类似“临时画布”），用于复杂混合效果（如阴影、模糊），但过度使用会消耗双倍内存。仅在必要时使用（如需要 PorterDuff.Mode混合模式时）。
##### 4)硬件加速兼容性
部分 API（如 `BlurMaskFilter`、`setShadowLayer`）在硬件加速下可能失效，需：
- 检测硬件加速状态：`canvas.isHardwareAccelerated()`
- 必要时关闭硬件加速（在 View 中设置 `android:layerType="software"`）
##### 5）状态同步与缓存
- **标记“脏数据”**：当属性变化时（如颜色、位图），调用 `invalidateSelf()`触发重绘。
- **缓存静态内容**：对不变的绘制结果（如背景）缓存到 Bitmap，减少重复计算（见前文“智能缓存机制”）。

