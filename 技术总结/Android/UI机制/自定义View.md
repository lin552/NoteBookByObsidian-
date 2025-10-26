---
创建时间: "2025-06-14 00:12:14"
作者: wangxiaoming
tags:
---
在 Android 开发中，​**自定义 View**​ 是实现复杂 `UI` 效果的核心能力，也是面试中的高频考点。其核心在于掌握 View 的测量、布局、绘制流程，以及如何通过重写关键方法实现自定义逻辑。以下是自定义 View 的**核心考点**及详细解析：

#### 一、自定义View的分类与选择
根据需求不同，自定义 View 可分为以下几类，需明确其适用场景：

|类型|说明|适用场景|
|---|---|---|
|​**继承 View**​|自定义单个 View（如圆形按钮、进度条）。|需要完全自定义绘制逻辑的组件。|
|​**继承 ViewGroup**​|自定义布局容器（如流式布局、瀑布流）。|需要自定义子 View 排列规则的容器。|
|​**继承已有控件**​|扩展已有控件功能（如自定义 TextView 支持渐变文字）。|在现有控件基础上增加新特性。|
#### 二、核心流程：测量（Measure）、布局（Layout）、绘制（Draw）
自定义 View 的核心是重写这三个阶段的方法，控制其尺寸、位置和外观。
##### 1.测量阶段（`onMeasure`）：确定View的尺寸
`onMeasure(int widthMeasureSpec, int heightMeasureSpec)` 是自定义 View 的第一个关键方法，用于计算 View 的宽度和高度（`mMeasuredWidth` 和 `mMeasuredHeight`）。
###### **`MeasureSpec` 模式**​
`MeasureSpec` 是父 View 传递给子 View 的**尺寸约束**，由两部分组成：
- ​**模式（Mode）​**​：`UNSPECIFIED`（父不限制，子自由设定）、`EXACTLY`（父指定精确尺寸）、`AT_MOST`（子最大不超过父给定的尺寸）。
- ​**尺寸（Size）​**​：父 View 建议的尺寸（如屏幕宽度、布局中指定的 `dp` 值）。
######  **实现逻辑**​
- ​**普通 View**​：根据 `MeasureSpec` 计算自身尺寸（如 `setMeasuredDimension(MeasureSpec.getSize(widthMeasureSpec), ...)`）。
- ​**自定义尺寸**​：若需支持 `wrap_content`，需手动计算最小尺寸（如文本长度、图片尺寸）。

**示例代码（自定义圆形 View）​**​：
```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    int widthSize = MeasureSpec.getSize(widthMeasureSpec);
    int widthMode = MeasureSpec.getMode(widthMeasureSpec);
    int heightSize = MeasureSpec.getSize(heightMeasureSpec);
    int heightMode = MeasureSpec.getMode(heightMeasureSpec);

    // 计算实际宽度：取最小可用尺寸（支持 wrap_content）
    int width;
    if (widthMode == MeasureSpec.EXACTLY) {
        width = widthSize; // 父指定精确尺寸（如 match_parent）
    } else {
        width = Math.min(widthSize, mRadius * 2); // wrap_content 时取直径（半径*2）
    }

    // 同理计算高度（圆形需宽高相等）
    int height;
    if (heightMode == MeasureSpec.EXACTLY) {
        height = heightSize;
    } else {
        height = Math.min(heightSize, mRadius * 2);
    }

    // 若宽高不一致，取较大值保证圆形完整
    int size = Math.max(width, height);
    setMeasuredDimension(size, size); // 最终尺寸
}
```
##### 2.布局阶段（`onLayout`）：确定View的位置
`onLayout(boolean changed, int l, int t, int r, int b)` 用于确定 View 在父容器中的位置（`left`、`top`、`right`、`bottom`）。
- ​**普通 View**​：无需重写（位置由父容器决定）。
- ​`ViewGroup`​：必须重写，为每个子 View 调用 `child.layout(l, t, r, b)` 分配位置。
###### `ViewGroup` 布局逻辑示例（水平排列子 View）
```java
@Override
protected void onLayout(boolean changed, int l, int t, int r, int b) {
    int childCount = getChildCount();
    int currentLeft = l; // 子 View 左边缘起始位置

    for (int i = 0; i < childCount; i++) {
        View child = getChildAt(i);
        int childWidth = child.getMeasuredWidth();
        int childHeight = child.getMeasuredHeight();

        // 子 View 右边缘不超过父容器右边缘
        if (currentLeft + childWidth > r) break;

        // 分配位置（水平排列）
        child.layout(currentLeft, t, currentLeft + childWidth, t + childHeight);
        currentLeft += childWidth; // 下一个子 View 左移
    }
}
```

##### 3.绘制阶段（`onDraw`）：渲染View的内容
`onDraw(Canvas canvas)` 是自定义 View 的核心方法，通过 `Canvas` 绘制图形、文本或图片。
###### Canvas 的常用绘制方法
|方法|作用|
|---|---|
|`drawRect()`|绘制矩形|
|`drawCircle()`|绘制圆形|
|`drawText()`|绘制文本（需设置 `Paint` 的 `textSize`、`color`）|
|`drawBitmap()`|绘制位图|
|`drawPath()`|绘制路径（如贝塞尔曲线）|
|`drawArc()`|绘制圆弧|
###### 示例代码（绘制圆形进度条）
```java
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);

    // 绘制背景圆环
    mPaint.setColor(Color.LTGRAY);
    mPaint.setStyle(Paint.Style.STROKE);
    mPaint.setStrokeWidth(10);
    canvas.drawCircle(mCenterX, mCenterY, mRadius, mPaint);

    // 绘制进度圆弧（根据进度调整角度）
    mPaint.setColor(Color.BLUE);
    RectF rectF = new RectF(mCenterX - mRadius, mCenterY - mRadius,
                            mCenterX + mRadius, mCenterY + mRadius);
    float sweepAngle = mCurrentProgress * 3.6f; // 进度转角度（100% → 360°）
    canvas.drawArc(rectF, -90, sweepAngle, false, mPaint); // 从顶部开始绘制
}
```
#### 三、自定义属性（Custom Attributes）
通过 XML 属性传递参数（如颜色、尺寸）是自定义 View 的常见需求，需以下步骤：
##### 1.定义属性
在 `res/values/attrs.xml` 中声明自定义属性：
```xml
<resources>
    <declare-styleable name="CircleView">
        <!-- 圆形颜色 -->
        <attr name="circleColor" format="color" />
        <!-- 圆形半径（dp） -->
        <attr name="circleRadius" format="dimension" />
    </declare-styleable>
</resources>
```
#### 2.在布局中使用
在 XML 中引用自定义 View 并设置属性：
```xml
<com.example.CircleView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:circleColor="#FF0000"
    app:circleRadius="50dp" />
```
#### 3.在代码中获取属性
在自定义 View 的构造方法中解析属性：
```java
public CircleView(Context context, AttributeSet attrs) {
    super(context, attrs);
    TypedArray ta = context.obtainStyledAttributes(attrs, R.styleable.CircleView);
    int color = ta.getColor(R.styleable.CircleView_circleColor, Color.RED);
    float radius = ta.getDimension(R.styleable.CircleView_circleRadius, 50);
    ta.recycle(); // 必须回收 TypedArray

    // 应用属性
    mPaint.setColor(color);
    mRadius = dpToPx(radius); // dp 转 px（避免不同设备尺寸差异）
}

// dp 转 px 工具方法
private float dpToPx(float dp) {
    return dp * getResources().getDisplayMetrics().density;
}
```

#### 四、触摸事件处理（Event Handling）
自定义 View 常需响应触摸事件（如滑动、点击），核心方法是 `onTouchEvent(MotionEvent event)`。
###### **1. 触摸事件类型**​
- `ACTION_DOWN`：手指按下。
- `ACTION_MOVE`：手指移动。
- `ACTION_UP`：手指抬起。
###### ​**2. 事件分发逻辑**​
- ​**单点触摸**​：直接处理 `ACTION_DOWN`，后续 `ACTION_MOVE`/`ACTION_UP` 需判断是否为同一手指。
- ​**多点触摸**​：通过 `event.getPointerId()` 区分不同手指，记录每个手指的位置。
###### 示例代码（实现滑动效果）
```java
private float mLastX; // 上次触摸的 X 坐标
private float mCurrentX; // 当前触摸的 X 坐标

@Override
public boolean onTouchEvent(MotionEvent event) {
    switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN:
            mLastX = event.getX();
            return true; // 消费事件，否则后续事件无法接收
        case MotionEvent.ACTION_MOVE:
            mCurrentX = event.getX();
            float deltaX = mCurrentX - mLastX;
            // 移动 View（通过平移 Canvas 实现）
            mOffsetX += deltaX;
            invalidate(); // 触发重绘
            mLastX = mCurrentX;
            return true;
        case MotionEvent.ACTION_UP:
            // 处理抬起逻辑（如复位）
            return true;
    }
    return super.onTouchEvent(event);
}
```

#### 五、性能优化
自定义 View 若使用不当易导致卡顿，需关注以下优化点：
##### **1. 减少 `onDraw` 中的计算**​
- 避免在 `onDraw` 中创建对象（如 `Paint`、`Rect`），应在初始化时创建并复用。
- 避免复杂计算（如数学运算、字符串拼接），可提前计算并缓存结果。
##### ​**2. 避免过度绘制**​
- 过度绘制指同一区域被多次绘制（如父 View 和子 View 重叠绘制）。
- 解决方法：通过开发者选项的「显示过度绘制区域」检查，减少冗余绘制（如隐藏不可见的背景）。
##### ​**3. 使用硬件加速**​
- 硬件加速可提升绘制效率，但部分绘制操作（如 `clipPath`）可能不兼容。
- 强制开启硬件加速：在 `AndroidManifest.xml` 中为 Activity 添加 `android:hardwareAccelerated="true"`。
##### ​**4. 异步绘制**​
- 对耗时操作（如加载大图），可在子线程处理完成后通过 `postInvalidate()` 触发重绘。

#### 六、常见面试题
1. ​**自定义 View 的生命周期？​**​  
    关键方法：`onMeasure()`（测量）、`onLayout()`（布局）、`onDraw()`（绘制）；需注意 `onAttachedToWindow()`（附加到窗口）和 `onDetachedFromWindow()`（从窗口分离）中释放资源（如取消动画）。
    
2. ​**如何实现一个支持 wrap_content 的自定义 View？​**​  
    在 `onMeasure()` 中根据 `MeasureSpec` 的模式（`EXACTLY`/`AT_MOST`）计算合理尺寸，取最小可用值。
    
3. ​**View 的坐标系是怎样的？​**​  
    原点在左上角（`(0,0)`），`x` 轴向右延伸，`y` 轴向下延伸（与数学坐标系相反）。
    
4. ​**`invalidate()` 和 `postInvalidate()` 的区别？​**​  
    `invalidate()` 是 View 的方法，触发主线程重绘；`postInvalidate()` 是 `Runnable` 的方法，可在子线程调用，触发主线程重绘。
    
5. ​**如何处理自定义 `ViewGroup` 中的滑动冲突？​**​  
    重写 `onInterceptTouchEvent()` 判断是否拦截事件（如水平滑动时拦截，垂直滑动时不拦截），或在子 View 中处理 `requestDisallowInterceptTouchEvent(true)` 禁止父容器拦截。