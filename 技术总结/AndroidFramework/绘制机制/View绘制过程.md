---
创建时间: 2025-05-17 14:24:53
作者: wangxiaoming
tags:
  - View绘制过程
---
Android 中 `View` 的绘制流程是界面呈现的核心机制，负责将 `UI` 元素从数据转换为屏幕上的像素。整个流程由 ​`ViewRootImpl​` 驱动，从根视图（`DecorView`）开始，通过递归遍历视图树完成测量、布局和绘制。以下是详细流程解析：

#### 一、绘制流程的发起起点
绘制流程的触发始于 ​**Activity 启动后窗口的创建**，核心步骤如下：
1. ​**Activity 初始化**​：  
    Activity 启动时，系统创建 `Window`（默认是 `PhoneWindow`），并为其设置 `DecorView`（窗口的根视图，包含标题栏、内容区等系统装饰）。
2. ​**加载布局文件**​：  
    `DecorView` 通过 `LayoutInflater` 加载 Activity 的布局文件（如 `setContentView(R.layout.activity_main)`），生成用户自定义的 View 树，并将其添加到 `DecorView` 的内容区（`FrameLayout`，id 为 `android.R.id.content`）。
3. ​`ViewRootImpl` 绑定**​：  
    系统创建 `ViewRootImpl`（连接 Window 与视图树的核心类），并通过 `attach()` 方法绑定到 `DecorView`。`ViewRootImpl` 负责协调视图的测量、布局、绘制，并与 Choreographer（帧调度器）协作，确保在 `VSYNC` 信号到来时触发绘制。

#### 二、绘制流程的核心阶段
整个绘制流程可分为三个关键阶段：​**测量（Measure）​**、**布局（Layout）​**、**绘制（Draw）​**，均从根视图 `DecorView` 开始，递归遍历子 View 完成。
##### **1. 测量阶段（Measure）：确定 View 的尺寸**​
测量阶段的目的是计算每个 View 的宽度和高度（`mMeasuredWidth` 和 `mMeasuredHeight`），为布局和绘制提供依据。流程如下：
- ​**触发入口**​：`ViewRootImpl.performTraversals()` → `measureHierarchy()` → `View.measure(int widthMeasureSpec, int heightMeasureSpec)`。
- ​`MeasureSpec` 模式**​：  
    测量时，父 View 会通过 `MeasureSpec` 传递约束条件（由父 View 的尺寸和自身布局模式决定），包含两种关键信息：
    - ​**测量模式（Mode）​**​：`UNSPECIFIED`（父 View 不限制子 View 尺寸，子 View 可自由设定）、`EXACTLY`（父 View 指定精确尺寸，子 View 需适配）、`AT_MOST`（子 View 最大不超过父 View 给定的尺寸）。
    - ​**测量大小（Size）​**​：父 View 建议的尺寸（如屏幕宽度、布局中指定的 `dp` 值）。
- ​**测量过程**​：
    - ​**View（非 `ViewGroup`）​**​：重写 `onMeasure(int widthMeasureSpec, int heightMeasureSpec)`，根据 `MeasureSpec` 计算自身尺寸（如 `setMeasuredDimension(MeasureSpec.getSize(widthMeasureSpec), ...)`)。
    - ​`ViewGroup`**​：除自身测量外，需遍历子 View 调用 `measure()`，并根据自身布局（如 `LinearLayout` 的 `match_parent`/`wrap_content`）调整子 View 的 `MeasureSpec`。

##### ​**2. 布局阶段（Layout）：确定 View 的位置**​
布局阶段的目的是确定每个 View 在屏幕上的坐标（`mLeft`、`mTop`、`mRight`、`mBottom`），形成视图树的空间布局。流程如下：
- ​**触发入口**​：测量完成后，`ViewRootImpl.performTraversals()` → `layout()`。
- ​**布局过程**​：
    - ​**View（非 `ViewGroup`）​**​：默认无布局逻辑（`onLayout()` 为空），位置由父 View 决定。
    - ​`ViewGroup`：重写 `onLayout(boolean changed, int l, int t, int r, int b)`，为每个子 View 计算坐标并调用 `child.layout(l, t, r, b)`。例如：
        - `LinearLayout` 按水平/垂直方向依次排列子 View；
        - `RelativeLayout` 根据 `layout_*` 属性计算子 View 位置。

##### ​**3. 绘制阶段（Draw）：将 View 渲染到屏幕**​
绘制阶段是将 View 的内容（背景、自身、子 View 等）绘制到屏幕的过程，通过 `Canvas` 对象操作。流程如下：
- ​**触发入口**​：布局完成后，`ViewRootImpl.performTraversals()` → `draw()`。
- ​**绘制过程**​：  
    `View.draw(Canvas canvas)` 方法默认按以下顺序绘制内容（可重写 `onDraw()` 自定义绘制逻辑）：
    1. ​**绘制背景**​：调用 `drawBackground(canvas)` 绘制背景色或 `Drawable`。
    2. ​**绘制自己**​：调用 `onDraw(canvas)` 绘制 View 自身内容（如 `TextView` 的文本）。
    3. ​**绘制子 View**​：调用 `dispatchDraw(canvas)` 遍历子 View 并调用其 `draw()` 方法（仅 `ViewGroup` 有效）。
    4. ​**绘制装饰**​：绘制滚动条、轮廓等装饰（如 `View` 的 `onDrawScrollBars()`）。
#### 三、关键类与方法总结
| 类/方法                  | 作用                                                       |
| --------------------- | -------------------------------------------------------- |
| `ViewRootImpl`        | 驱动整个绘制流程，协调测量、布局、绘制，与 Choreographer 交互。                  |
| `performTraversals()` | `ViewRootImpl` 的核心方法，触发 `measure()`、`layout()`、`draw()`。 |
| `MeasureSpec`         | 封装父 View 对子 View 的尺寸约束（模式 + 大小）。                         |
| `onMeasure()`         | 自定义 View 测量逻辑，需调用 `setMeasuredDimension()` 设置尺寸。         |
| `onLayout()`          | 自定义 `ViewGroup` 布局逻辑，设置子 View 的坐标。                       |
| `onDraw(Canvas)`      | 自定义 View 绘制逻辑（如绘制圆形、动画）。                                 |
#### 四、自定义View的注意事项
若需自定义 View（如实现一个圆形进度条），需重写以下方法并处理关键逻辑：
1. ​**`onMeasure()`**​：根据 `MeasureSpec` 计算合理尺寸（如圆形直径取宽高的较小值）。
2. ​**`onLayout()`**​：若为 `ViewGroup`，需为子 View 分配位置；普通 View 可忽略。
3. ​**`onDraw(Canvas)`**​：使用 `Canvas.drawXXX()` 方法绘制内容（如 `drawCircle()`、`drawText()`）。
#### 五、总结
View 的绘制流程可概括为：  
​**Activity 启动 → 创建 `DecorView` → `ViewRootImpl` 绑定 → 触发 `performTraversals()` → 依次执行 measure（测量尺寸）→ layout（确定位置）→ draw（渲染内容）​。  
理解这一流程是自定义 View、优化 `UI` 性能（如减少重复测量/布局）的基础。