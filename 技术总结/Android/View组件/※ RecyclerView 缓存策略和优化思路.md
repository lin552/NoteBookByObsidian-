---
创建时间: 2025-03-14T18:39:00
作者: wangxiaoming
tags:
  - Android
  - RecyclerView
---

#### 一、RecyclerView 缓存机制核心策略
##### 1）Scrap 缓存（屏内缓存）
- **mAttachedScrap**：存储当前可见或即将可见的 ViewHolder（例如布局调整时临时分离的视图），无需重新绑定数据
- ​**mChangedScrap**：保存数据已变更但未重新绑定的 ViewHolder，用于动画或局部刷新场景
##### 2)Cache 缓存（离屏缓存）
**mCachedViews**：默认存储最近被移除的 2 个 ViewHolder，按位置或 ID 匹配复用，无需重新绑定数据
。例如快速反向滑动时，可直接复用刚滑出屏幕的视图。
##### 3）自定制缓存 （ViewCacheExtension）
开发者可扩展的缓存层，用于存储高频复用或特定类型的 ViewHolder（如固定尺寸的广告位视图）
##### 4)RecycledViewPool(回收池)
- 按 ViewType 分类存储 ViewHolder，默认每个类型最多存 5 个，复用时会触发 `onBindViewHolder`
    。适用于跨列表共享复用（如多个 Tab 页共用同一类型视图）

**缓存工作流程**：  
当需要新视图时，按优先级依次从 Scrap → Cache → 自定义缓存 → 回收池查找；若未命中，则创建新 ViewHolder 并绑定数据

#### 二、优化思路与实践方案
##### 1)布局优化
- ​**减少嵌套层级**：使用 `ConstraintLayout` 替代多层 `LinearLayout`，测量耗时降低 30%
- ​**固定尺寸标记**：调用 `setHasFixedSize(true)` 避免动态高度导致的重复测量
- ​**预加载机制**：通过 `LayoutManager.setInitialPrefetchItemCount()` 预加载下一屏数据，减少滑动卡顿

##### 2）数据绑定优化
​**DiffUtil 差异更新**：对比新旧数据集差异，仅刷新变更项（减少 90% 无效刷新）
```kotlin
val diffResult = DiffUtil.calculateDiff(MyDiffCallback(oldList, newList))  
diffResult.dispatchUpdatesTo(adapter)  
```
​**异步数据预处理**：在 `onBindViewHolder` 外完成数据格式化、图片 URL 拼接等耗时操作

##### 3）内存与缓存管理
- **动态调整缓存池**：
    - `mCachedViews` 扩容：`recyclerView.setItemViewCacheSize(20)` 提升快速反向滑动的流畅性
    - 共享 `RecycledViewPool`：多个列表实例共享池，减少内存碎片（适用于电商首页多 Tab 列表）
- ​**大图分块加载**：使用 `BitmapRegionDecoder` 实现长图按需加载，避免单张图片内存溢出

##### 4）滚动性能优化
- **滚动任务调度**：通过 `addOnScrollListener` 监听滑动状态，暂停非核心任务（如动画、日志上报）
- ​**轻量化动画**：关闭默认 ItemAnimator 或自定义简单动画（如 Alpha 渐变），减少 GPU 绘制压力

##### 5）图片加载专项优化
​**滑动暂停加载**：结合 Glide 的 `pauseRequests()` 和 `resumeRequests()`，避免快速滑动时触发冗余网络请求
```java
recyclerView.addOnScrollListener(new OnScrollListener() {  
    @Override  
    public void onScrollStateChanged(RecyclerView recyclerView, int newState) {  
        if (newState == SCROLL_STATE_DRAGGING) Glide.with(context).pauseRequests();  
        else Glide.with(context).resumeRequests();  
    }  
});  
```
​**防错位机制**：在 `onBindViewHolder` 中为 ImageView 设置 Tag（图片 URL），异步加载完成后校验 Tag 一致性

#### 三、高级场景优化
- ​**瀑布流布局优化**：通过 `GapWorker` 预计算动态高度，避免布局抖动（StaggeredGridLayoutManager 特有）
- ​**分页加载策略**：结合 `Paging 3.0` 实现数据库与网络数据的分页加载，减少单次数据量对渲染的影响
- ​**视图绑定加速**：使用 `ViewBinding` 或 `DataBinding` 替代 `findViewById`，单次绑定耗时从 1.2ms 降至 0.3ms
