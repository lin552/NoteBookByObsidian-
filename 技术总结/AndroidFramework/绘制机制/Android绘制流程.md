---
创建时间: "2025-08-22 11:38:54"
作者: wangxiaoming
tags:
---
Android View 的绘制流程是一个分层级的递归过程，主要由 ​**测量（Measure）、布局（Layout）、绘制（Draw）​**​ 三个核心阶段组成。其触发通常与界面更新需求相关（如 Activity 启动、View 可见性变化、手动调用 `invalidate()`或 `requestLayout()`等）。以下是详细的触发流程和关键步骤：

#### 一、触发条件
View 绘制的触发通常由以下场景引发：
##### 1.界面初始化：
如Activity启动、Fragment加载完成时，系统会触发根View（`DecorView`）的绘制
##### 2.界面更新：
- 调用 `invalidate()`（或 `postInvalidate()`，用于非 `UI` 线程）：标记当前 View 及其父链需要重绘（仅触发 `draw`阶段）。
- 调用 `requestLayout()`：标记 View 树需要重新测量、布局和绘制（触发完整流程）。
##### 3.尺寸/位置变化：
如通过 `LayoutParams`修改 View 大小，或 View 被添加/移除（触发 `requestLayout()`）。
##### 4.动画或交互：
属性动画（如 `ObjectAnimator`）修改 View 属性时，会通过 `invalidate()`触发重绘；触摸反馈等交互也可能触发局部重绘。

#### 二、核心流程：从`ViewRootImpl`到具体View
绘制流程的入口是`ViewRootImpl`（连接Window和`DecorView`的桥梁），它通过`performTraversals()`方法驱动整个View树的测量、布局和绘制。
##### 1.测量阶段（Measure）:确定View的尺寸
测量阶段的目的是计算 View 的实际宽高（`mMeasuredWidth`和 `mMeasuredHeight`），由父 View 向子 View 传递约束条件（通过 `MeasureSpec`）。

**关键步骤：**
•`MeasureSpec` 的生成​：
父 View 根据自身的测量模式（如自身是否被限制大小）和子 View 的布局参数（如 `match_parent`、`wrap_content`、具体数值），为子 View 生成一个 `MeasureSpec`（32 位整数，高 2 位为模式，低 30 位为尺寸）。
模式有三种：
- `UNSPECIFIED`：父 View 不限制子 View 大小（常见于 `ScrollView`）。
- `EXACTLY`：父 View 已确定子 View 的精确大小（如 `match_parent`或具体数值且父 View 有足够空间）。
- `AT_MOST`：子 View 大小不能超过父 View 指定的最大值（如 `wrap_content`）。

• `onMeasure()` 的调用：
父 View 调用子 View 的 `measure(int widthMeasureSpec, int heightMeasureSpec)`方法，子 View 在 `onMeasure()`中根据 `MeasureSpec`计算自身尺寸，并通过 `setMeasuredDimension(measuredWidth, measuredHeight)`保存结果。
> **自定义 View 注意**​：若 View 是 `wrap_content`或 `match_parent`，需在 `onMeasure()`中处理这两种情况（默认实现可能仅处理 `EXACTLY`模式）。

##### 2.布局阶段（Layout）:确定View的位置
布局阶段的目的是确定 View 在父容器中的坐标（`left`、`top`、`right`、`bottom`），由父 View 向子 View 传递位置信息。

关键步骤：
•`onLayout()` 的调用​：
父 View 完成自身布局后（如 `ViewGroup`的 `onLayout()`实现），会遍历子 View 并调用其 `layout(int left, int top, int right, int bottom)`方法，传递子 View 的位置坐标。
子 View 在 `onLayout()`中可进一步调整自身或子 View 的位置（`ViewGroup`需重写此方法处理子 View 布局）。
> ​**注意**​：普通 View（非 `ViewGroup`）的 `onLayout()`通常为空实现，因为无需管理子 View。

##### 3.绘制阶段（Draw）：将View渲染到屏幕
绘制阶段将 View 的内容渲染到 Canvas 上，最终通过 `SurfaceFlinger`合成到屏幕。`View.draw(Canvas canvas)`是核心方法，包含以下步骤：

绘制顺序（由父到子）：
##### 1.绘制背景（Background）:
 绘制 View 的背景（如 `android:background`属性设置的图片或颜色），通过 `drawBackground(canvas)`实现。
##### 2.绘制自身内容
 调用 `onDraw(Canvas canvas)`绘制 View 的核心内容（如文本、图片等）。普通 View 需在此方法中实现自定义绘制逻辑（如 `TextView`绘制文字）。
##### 3.绘制子View（`DispatchDraw`）:
若当前 View 是 `ViewGroup`，会调用 `dispatchDraw(Canvas canvas)`遍历所有子 View，并调用其 `draw()`方法（递归触发子 View 的绘制流程）。普通 View 无此步骤。
##### 4.绘制装饰（Foreground & Scrollbar）:
绘制前景（如 `android:foreground`属性）和滚动条（如 `scrollbars`相关属性），通过 `drawForeground(canvas)`和 `drawScrollbars(canvas)`实现。

#### 三、关键类与协作
- ​`ViewRootImpl`​：驱动整个绘制流程的入口，通过 `performTraversals()`调用 `performMeasure()`、`performLayout()`、`performDraw()`三级方法。
- ​**Choreographer**​：协调 `VSYNC` 信号（屏幕刷新频率，通常 `60Hz`），确保绘制与屏幕刷新同步，避免丢帧。
- ​**Canvas & Paint**​：绘制工具类，`Canvas`提供绘图操作（如 `drawRect()`、`drawText()`），`Paint`定义画笔样式（颜色、粗细、字体等）。

##### 四、优化提示
- **减少无效重绘**​：仅当 View 内容变化时调用 `invalidate()`，避免频繁触发绘制流程。
- ​**避免过度测量/布局**​：自定义 View 时，优化 `onMeasure()`和 `onLayout()`的逻辑（如缓存测量结果）。
- ​**使用硬件加速**​：Android 3.0（API 11）起支持硬件加速，可通过 `android:hardwareAccelerated="true"`开启，提升复杂绘制性能。

总流程图
```
触发条件（如 VSYNC、invalidate()）
       ↓
ViewRootImpl.performTraversals()
       ↓
performMeasure() → 根 View.measure() → 子 View.measure() → ...（递归测量）
       ↓
performLayout() → 根 View.layout() → 子 View.layout() → ...（递归布局）
       ↓
performDraw() → 根 View.draw() → 子 View.draw() → ...（递归绘制）
```