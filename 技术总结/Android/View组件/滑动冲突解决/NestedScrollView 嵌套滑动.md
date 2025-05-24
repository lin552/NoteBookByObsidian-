---
创建时间: 2025-04-16 12:18:34
作者: wangxiaoming
tags:
  - NestedScrollView
---
#### 一、`NestedScrollView`的定义与核心功能
`NestedScrollView` （通常指 `androidx.core.widget.NestedScrollView`）是Android中一种支持 **嵌套滑动** 的滚动容器，继承自 `ScrollView`。它通过实现 `NestedScrollingParent`接口，解决了传统 `ScrollView` 无法在嵌套滚动场景下协调父空间与子控件滑动行为的核心问题
#### 二、与`ScrollView` 区别
| ​**特性**​     | ​**NestedScrollView**​             | ​**ScrollView**​            |
| ------------ | ---------------------------------- | --------------------------- |
| ​**嵌套滑动支持**​ | ✅ 支持父子控件协同滚动（如内嵌 RecyclerView）<br> | ❌ 不支持，易引发滚动冲突（如子控件截获事件）<br> |
| ​**事件分发机制**​ | 嵌套滑动机制：父控件和子控件可共享滑动事件<br>          | 传统事件分发：子控件消费后父控件无法处理<br>    |
| ​**性能优化**​   | 更适合复杂嵌套场景（需合理设计）<br>               | 仅适合简单滚动布局<br>               |
| ​**版本兼容性**​  | 需依赖 AndroidX 库，低版本需兼容处理<br>        | 无依赖，全版本兼容                   |
#### 三、工作原理
##### 1）嵌套滑动机制
通过 `NestedScrollingParent`和 `NestedScrollingChild`接口协调滑动事件。子控件（如`RecyclerView`）滑动前会询问父控件（`NestedScrollView`）是否消费部分滑动距离，剩余距离再由子控件处理
##### 2）事件分发流程
###### ① `Pre-Scroll`
子控件滑动前将增量传递给父控件
###### ② 消费阶段
父控件处理部分增量，剩余由子控件处理
###### ③ Post-Scroll
未消费的增量再次传递给父控件

#### 四、使用方式
基础代码示例
```java
<androidx.core.widget.NestedScrollView
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fillViewport="true">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <!-- 子控件（如 RecyclerView、TextView） -->
    </LinearLayout>
</androidx.core.widget.NestedScrollView>
```
关键点：
- 仅允许 **一个直接子控件** （通常 `LinearLayout`等容器）
- 内嵌 `RecyclerView` 时需注意复用优化 （避免全部一次性渲染）
#### 五、注意事项
- **性能问题**：避免多层嵌套（如 `NestedScrollView` 内再套 `NestedScrollView`），可能导致卡顿
- **滑动冲突**：若子控件（如 `ViewPager`）需要横向滚动，需自定义事件拦截逻辑
- **内存泄漏**：在 `onDestroy`中移除滚动监听
- **布局优化**：动态计算子控件高度，避免过度测量（如通过 `onGlobalLayout` 监听）

#### 六、优化建议
- **复用机制**：优先使用 `RecyclerView` 替代 `ListView`，利用 `ViewHolder`复用
- **硬件加速**：开启 `android.layerType="hardware"` 提升复杂布局渲染性能
- **分页加载**：大数据场景下分页加载，减少一次性渲染压力

#### 七、面试考察点
##### 1）嵌套滑动机制
- 解释 `NestedScrollingParent` 和 `NestedScrollingChild` 的作用
- 对比传统事件分发与嵌套滑动的差异
##### 2）性能优化
- 如何避免`NestedScrollView + RecyclerView`的复用失效？
- 分析滚动卡顿的可能原因及解决方案
##### 3）实际应用
- 举例说明 `NestedScrollView`的使用场景（如吸顶效果、复杂表单）
##### 4）源码理解
- 描述 `onNestedPreScroll` 和 `onNestedScroll` 的执行流程