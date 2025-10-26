---
创建时间: 2025-04-12 17:24:57
作者: wangxiaoming
tags:
  - Android
  - RecyclerView
---

#### 一、`RecyclerView` 缓存机制核心策略
##### 1）Scrap 缓存（屏内缓存）
- `mAttachedScrap`：存储当前可见或即将可见的` ViewHolder`（例如布局调整时临时分离的视图），无需重新绑定数据
- ​`mChangedScrap`：保存数据已变更但未重新绑定的 `ViewHolder`，用于动画或局部刷新场景
##### 2）Cache 缓存（离屏缓存）
`mCachedViews`：默认存储最近被移除的 2 个 `ViewHolder`，按位置或 ID 匹配复用，无需重新绑定数据
。例如快速反向滑动时，可直接复用刚滑出屏幕的视图。
##### 3）自定制缓存 （`ViewCacheExtension`）
开发者可扩展的缓存层，用于存储高频复用或特定类型的 `ViewHolder`（如固定尺寸的广告位视图）
##### 4）`RecycledViewPool`(回收池)
- 按 `ViewType` 分类存储 `ViewHolder`，默认每个类型最多存 5 个，复用时会触发 `onBindViewHolder`
    。适用于跨列表共享复用（如多个 Tab 页共用同一类型视图）

**缓存工作流程**：  
当需要新视图时，按优先级依次从 Scrap → Cache → 自定义缓存 → 回收池查找；若未命中，则创建新 `ViewHolder` 并绑定数据






