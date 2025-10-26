---
创建时间: 2025-07-29 15:09:13
作者: wangxiaoming
tags:
  - ViewPage
---
在 Android 开发中，`ViewPager` 和 `ViewPager2` 都是用于实现左右滑动切换页面（通常是 Fragment 或 View）的核心组件，但 `ViewPager2` 是 `ViewPager` 的升级版本，针对性能、功能和灵活性进行了全面优化。以下是两者的主要区别：

#### 一.底层架构不同

- `ViewPager`：基于 `PagerAdapter`（或其子类如 `FragmentPagerAdapter`、`FragmentStatePagerAdapter`）实现，内部通过维护一个 `View` 缓存池来管理页面。滑动时通过 `instantiateItem()` 和 `destroyItem()` 动态创建/销毁页面，逻辑相对传统。
- ​`ViewPager2`​：基于 `RecyclerView` 实现（本质是 `RecyclerView` 的子类），利用了 `RecyclerView` 成熟的回收复用机制（`Recycler` 和 `ViewHolder` 模式）。这种架构天然支持更高效的页面复用、更灵活的布局管理，以及 `RecyclerView` 原生支持的多种特性（如垂直滑动、嵌套滑动等）。

#### 二.页面管理能力

- `ViewPager`​：
    - 预加载机制固定：默认预加载 1 页（可通过 `setOffscreenPageLimit(int)` 调整，但受限于实现，实际效果有限）。
    - 页面销毁策略：`FragmentPagerAdapter` 仅在 `Fragment` 不可见且不在回退栈时销毁；`FragmentStatePagerAdapter` 会销毁不可见的 `Fragment` 并保存状态，但逻辑较复杂。
    - 滑动冲突：嵌套 `ViewPager` 时容易出现滑动冲突，需要手动处理。
- ​`ViewPager2`：
    - 自动复用更高效：依托 `RecyclerView` 的回收机制，页面复用更智能，内存占用更低。
    - 预加载更灵活：通过 `setOffscreenPageLimit(int)` 可动态调整预加载页数（支持更大范围），且实际复用策略更合理。
    - 生命周期管理更完善：配合 `FragmentStateAdapter` 时，`Fragment` 的生命周期（如 `onResume()`/`onPause()`）与可见性更紧密绑定（例如，仅当前可见页的 `Fragment` 会触发 `onResume()`），避免资源浪费。
    - 嵌套滑动友好：原生支持与父容器或其他滑动组件的嵌套滑动（如 `NestedScrollView`），无需额外处理。

#### 三.功能扩展性

- `ViewPager`​：
    - 仅支持水平滑动（垂直滑动需自定义 `PagerAdapter` 并重写 `getPageWidth()` 等方法，体验较差）。
    - 自定义指示器（Indicator）需手动同步页面位置，逻辑较繁琐。
    - 不支持 `RTL`（从右到左）布局自动适配（需手动反转页面顺序）。
- ​`ViewPager2`：
    - 原生支持垂直滑动：通过 `setOrientation(ViewPager2.ORIENTATION_VERTICAL)` 即可实现垂直方向的滑动切换。
    - 自动支持 `RTL` 布局：当系统语言为阿拉伯语等 `RTL` 语言时，页面会自动从右向左排列，无需额外代码。
    - 更简单的指示器集成：配合 `TabLayout`（通过 `TabLayoutMediator`）可快速实现联动指示器，自动同步选中状态。
    - 支持 `PageTransformer` 自定义动画：虽然 `ViewPager` 也支持，但 `ViewPager2` 的动画触发更流畅（基于 `RecyclerView` 的 `ItemAnimator` 机制）。

#### 四.兼容性与维护
- `ViewPager`​：属于传统组件，存在于 `androidx.viewpager:viewpager` 库中，已停止更新（最后一次发布是 2018 年）。
- ​`ViewPager2`​：属于`AndroidX` 新一代组件（`androidx.viewpager2:viewpager2`），持续维护并集成最新特性（如 `Jetpack Compose` 支持）。官方明确推荐新项目使用 `ViewPager2`，旧项目逐步迁移。

#### 五.适配器(Adapter)的差异
- `ViewPager`​：依赖 `PagerAdapter` 或其子类（如 `FragmentPagerAdapter`），需要手动管理 `View` 或 `Fragment` 的创建、缓存和销毁逻辑。例如，`FragmentPagerAdapter` 适用于少量固定 `Fragment`（如引导页），而 `FragmentStatePagerAdapter` 适用于动态或大量 `Fragment`（如新闻分页）。
- ​`ViewPager2`​：依赖 `RecyclerView.Adapter`（或其子类），推荐使用 `FragmentStateAdapter`（专门为 `Fragment` 设计）。`FragmentStateAdapter` 会自动管理 `Fragment` 的生命周期，并与 `ViewPager2` 的滑动事件深度绑定，代码更简洁。

#### 六.Fragment生命周期表现
##### 1.`ViewPager`（以 `FragmentPagerAdapter`/`FragmentStatePagerAdapter` 为例）​​
`ViewPager` 对 `Fragment` 的生命周期管理较为“保守”，核心逻辑是 ​**尽可能保留 `Fragment` 实例以减少重建开销**，但会导致生命周期与可见性不完全同步：

- ​**当前可见页（Active Page）​**​：  
    `Fragment` 处于完整生命周期状态（`onCreate` → `onStart` → `onResume`），视图正常显示。
    
- ​**相邻预加载页（Adjacent Pages，通常 1 页）​**​：  
    `ViewPager` 默认预加载 1 页（通过 `setOffscreenPageLimit(1)`），这些页的 `Fragment` 会被实例化，但 ​**视图未附加到窗口**​（即 `onCreateView` 未被调用）。此时 `Fragment` 的生命周期停留在 `onAttach` → `onCreate` → `onCreateView`（可能未执行）→ `onActivityCreated` → `onStart`，但 ​**不会触发 `onResume`**​（因为视图未显示）。  
    当滑动到这些页时，`Fragment` 才会触发 `onResume`；滑离时，`Fragment` 的 `onPause` 不会立即触发（视图被 `detach`，但实例保留）。
    
- ​**非预加载页（超出预加载范围的页）​**​：  
    当滑动到更远的页面时，`ViewPager` 会销毁这些页的 `Fragment` 视图（调用 `onDestroyView`），但 ​**保留 `Fragment` 实例**​（通过 `FragmentManager` 缓存）。再次滑回时，直接复用缓存的实例（仅重新创建视图）。
    
- ​**极端情况（页面过多）​**​：  
    若页面数量超过 `FragmentManager` 的缓存限制（默认无明确限制，但受内存约束），`FragmentPagerAdapter` 会销毁不可见的 `Fragment` 实例（调用 `onDestroy`），而 `FragmentStatePagerAdapter` 会更激进地销毁并保存状态（类似 `Activity` 的 `onSaveInstanceState`）。

##### 2.`ViewPager2`（基于 `FragmentStateAdapter`）
`ViewPager2` 基于 `RecyclerView` 的回收机制，​**严格将 `Fragment` 的生命周期与可见性绑定**，仅在 `Fragment` 可见或即将可见时激活其生命周期，不可见时主动释放资源：

- ​**当前可见页（Active Page）​**​：  
    `Fragment` 处于完整生命周期状态（`onCreate` → `onStart` → `onResume`），视图正常显示。
    
- ​**预加载页（Adjacent Pages，默认 1 页）​**​：  
    `ViewPager2` 默认预加载 1 页（可通过 `setOffscreenPageLimit(int)` 调整），这些页的 `Fragment` 会被实例化，但 ​**仅在预加载时触发 `onCreate` 和 `onCreateView`**，不会主动调用 `onResume`（除非用户滑动到该页）。  
    当用户滑动到预加载页时，`Fragment` 触发 `onResume`；滑离时，触发 `onPause` → `onStop`（若继续滑离到更远页面，可能进一步销毁视图）。
    
- ​**非预加载页（超出预加载范围的页）​**​：  
    当滑动到更远的页面时，`ViewPager2` 会销毁这些页的 `Fragment` 实例（调用 `onDestroy`），并释放相关资源。再次滑回时，重新创建 `Fragment` 实例（类似 `RecyclerView` 的 `ViewHolder` 复用逻辑）。
    
- ​**可见性驱动的生命周期**​：  
    `ViewPager2` 通过 `FragmentVisibilityListener` 或内部机制精确跟踪 `Fragment` 的可见性（如进入屏幕可见区域、离开屏幕不可见区域）。例如：
    
    - 当 `Fragment` 完全离开屏幕（不可见）时，触发 `onPause` → `onStop` → `onDestroy`（除非被缓存）。
    - 当 `Fragment` 进入屏幕可见区域时，触发 `onCreate`（首次）→ `onStart` → `onResume`。