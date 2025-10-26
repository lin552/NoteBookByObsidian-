---
创建时间: 2025-04-16 19:00:59
作者: wangxiaoming
tags:
  - RecyclerView
  - Behavior
---
#### 一、机制原理
`Behavior` 是 `Android Material Design` 中 `CoordinatorLayout` 的核心组件,本质是通过 `访问者模式` 实现视图的交互逻辑解耦。其核心原理包含：
##### 1）责任链机制
`CoordinatorLayout` 的子视图按布局顺序依次处理事件，排序靠前的视图优先拦截
##### 2）事件分发
- **触摸事件**：通过 `onInterceptTouchEvent` 和 `onTouchEvent` 实现嵌套滑动优先级控制 （如子视图先处理滑动）
- **滚动事件**：通过 `onStartNestedScroll` 建立父子视图的滚动协调关系，`dispatchNestedPreScroll` 实现滚动事件的分发优先级
- **布局测量**：通过重写 `onMeasureChild` 和 `onLayoutChild`实现自定义布局逻辑（如动态调整视图位置）

#### 二、核心方法与参数

| 方法名                      | 作用         | 关键参数说明                                         | 使用场景示例                                 |
| ------------------------ | ---------- | ---------------------------------------------- | -------------------------------------- |
| `onStartNestedScroll`    | 判断是否响应嵌套滑动 | `axes`：滑动方向`（SCROLL_AXIS_VERTICAL/HORIZONTAL）` | 控制仅响应垂直滑动<br>                          |
| `onNestedPreScroll`      | 父视图优先处理滚动  | `dx/dy`：待消耗的滚动距离；`consumed`：父视图已消耗的距离          | 实现顶部导航栏折叠时优先消耗滚动<br>                   |
| `onDependentViewChanged` | 依赖视图变化时的回调 | `dependency`：被依赖的视图对象                          | 实现底部按钮随 `RecyclerView` 滚动隐藏            |
| `layoutDependsOn`        | 声明视图依赖关系   | `parent`：`CoordinatorLayout`；`child`：当前视图      | 实现 FloatingActionButton 依赖 Snackbar 位置 |
| `onMeasureChild`         | 自定义子视图测量逻辑 | parentWidth/HeightMeasureSpec`：父容器的测量规格        | 实现动态调整子视图高度<br>                        |

#### 三、注意事项与优化
##### 1）性能陷阱
- 避免在 `onDependtViewChanged` 中频繁触发布局计算（用 `translationY`替代修改 `LayoutParams`）
- `RecyclerView` 需设置 `setHasFixedSize(true)` 避免嵌套测量风暴
##### 2）事件冲突
- 通过 `nestedScrollingEnabled = false`关闭子视图独立滚动
- 在 `onStartNestedScroll` 中精确控制响应方向
##### 内存优化
- 避免在 `Behavior` 中持有 Context/View 强引用
- 使用 `@Nullable` 注解标记可能为空的参数
#### 四、面试考察重点
##### 1）原理层面
- 解释 `NestedScrolling`机制的事件分发流程（ACTION_DOWN->预滚动->正式滚动）
- `CoordinatorLayout` 如何通过 `Behavior`实现责任链模式
##### 2）实践层面
如何实现吸顶效果(Sticky Header)
处理`RecyclerView`与 `NestedScrollView`的滚动冲突
##### 3）性能优化
- 分析`Behavior`导致卡顿的常见场景（如频繁布局、未复用视图）
- 如何通过 `Merge`标签减少布局层级