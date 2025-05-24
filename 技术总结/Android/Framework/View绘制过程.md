---
创建时间: 2025-05-17 14:24:53
作者: wangxiaoming
tags:
  - View绘制过程
---
#### 一、基础回答：三阶段流程

**核心公式**​：  
​`measure → layout → draw`
（测量 → 布局 → 绘制）
##### **1. Measure（测量）​**​
- ​**目的**​：确定View的宽高（`mMeasuredWidth`和`mMeasuredHeight`）。
- ​**方法**​：`onMeasure(widthMeasureSpec, heightMeasureSpec)`。
- ​**关键点**​：
    - ​`MeasureSpec`：由父容器传递的约束条件（模式 + 尺寸），如`EXACTLY`、`AT_MOST`、`UNSPECIFIED`。
    - ​**递归测量**​：父容器测量所有子View的尺寸（如`LinearLayout`按方向依次测量子View）。
    - ​**自定义View**​：需重写`onMeasure`，调用`setMeasuredDimension()`设置最终尺寸。
```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    int width = getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec);
    int height = getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec);
    setMeasuredDimension(width, height);
}
```
##### ​**2. Layout（布局）​**​
- ​**目的**​：确定View在父容器中的位置（`left`, `top`, `right`, `bottom`）。
- ​**方法**​：`onLayout(boolean changed, int l, int t, int r, int b)`。
- ​**关键点**​：
    - ​`ViewGroup`​：需遍历子View，调用`child.layout()`确定其位置（如`RelativeLayout`按规则定位子View）。
    - ​`LayoutParams`​：子View的布局参数（如`MATCH_PARENT`、`WRAP_CONTENT`）影响布局逻辑。
    - ​自定义`ViewGroup`：需重写`onLayout`，根据业务逻辑计算子View坐标。
##### ​**3. Draw（绘制）​**​
- ​**目的**​：将View内容绘制到屏幕上。
- ​**方法**​：`onDraw(Canvas canvas)`。
- ​**关键点**​：
    - ​**绘制顺序**​：背景 → 自定义内容 → 子View → 滚动条。
    - ​**Canvas操作**​：通过`Canvas`绘制形状、文本、位图等（如`drawCircle()`画圆）。
    - ​**自定义View**​：重写`onDraw`，结合`Paint`对象控制样式（颜色、字体、路径）。
```java
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    Paint paint = new Paint();
    paint.setColor(Color.RED);
    canvas.drawCircle(mCenterX, mCenterY, mRadius, paint);
}
```

#### 二、进阶回答：细节与源码
#### **1. Measure中的`MeasureSpec`**​
- ​**模式解析**​：
    - `EXACTLY`：精确尺寸（如`layout_width="100dp"`）。
    - `AT_MOST`：最大尺寸（如`layout_width="wrap_content"`）。
    - `UNSPECIFIED`：无限制（多见于`AdapterView`的子项）。
- ​**源码关联**​：  
    `MeasureSpec`由父容器的`LayoutParams`和测量模式共同决定，父容器通过`getChildMeasureSpec()`生成。

#### ​**2. Measure与Layout的递归**​
- ​**递归终止条件**​：  
    View的`mPrivateFlags`标记为`MEASURED`，或为叶子节点（无子View的`ViewGroup`）。
- ​**性能优化**​：  
    避免在`onMeasure`中执行耗时操作（如IO），否则可能导致`ANR`。
#### ​**3. Draw中的双缓冲机制**​
- ​**硬件加速**​：  
    Android 3.0+默认启用硬件加速，通过`DisplayList`记录绘制指令，提升性能。
- ​**软件绘制**​：  
    旧版本通过`Canvas`直接绘制到Bitmap，需注意`onDraw`的重复调用问题（如避免在`onDraw`中创建对象）。

#### 三、高阶回答：场景与问题
##### **1. 自定义View的最佳实践**​
- ​**避免过度绘制**​：  
    在`onDraw`中复用`Paint`对象，减少对象创建。
- ​**处理View尺寸变化**​：  
    重写`onSizeChanged()`，在尺寸变化时更新绘制参数（如圆形半径）。
- ​**支持硬件加速**​：  
    使用`Canvas`的硬件绘制方法（如`drawBitmap`），避免依赖`View`层级。
##### ​**2. 常见面试题扩展**​
- ​**Q：View的绘制流程和屏幕刷新（Choreographer）的关系？​**​  
    A：绘制完成后，通过`ViewRootImpl`的`doTraversal()`触发`performTraversals()`，最终由`Choreographer`调度帧刷新（`60Hz`）。
- ​**Q：为什么`onMeasure`可能被多次调用？​**​  
    A：父容器布局变化、尺寸不确定（如`wrap_content`）、或配置变更（如横竖屏切换）。