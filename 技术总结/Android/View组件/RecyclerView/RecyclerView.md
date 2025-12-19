---
创建时间: 2025-04-12 17:24:57
作者: wangxiaoming
tags:
  - Android
  - RecyclerView
---

#### 一、`RecyclerView` 缓存机制核心策略

##### Recycler 四级缓存（从快到慢）​
`RecyclerView` 通过四级缓存查找可复用的 ViewHolder，决定是否触发 `onCreateViewHolder`：
1. `AttachedScrap`（临时缓存）：
    - 存储**正在屏幕上显示的 `ViewHolder`**（如滚动过程中短暂离开屏幕但未回收的），优先级最高。
    - 无需重新绑定数据（除非数据已过期）。

2. `mCachedViews`（一级缓存）：
    - 存储**最近被回收的 `ViewHolder`**（默认缓存 2 个，可按 `ViewType` 配置），复用时不触发 `onBindViewHolder`（直接复用视图状态）。
    - 若缓存满，旧 `ViewHolder` 会被移到 `RecycledViewPool`。
    
3. `ViewCacheExtension`（自定义缓存）：
    - 开发者可扩展的缓存层（极少使用），用于特殊复用逻辑。
    
4. `RecycledViewPool`（二级缓存）
    - 按 `ViewType` 分类缓存的 `ViewHolder` 池（默认每种 `ViewType` 缓存 5 个），复用时会触发 `onBindViewHolder`（需重新绑定数据）。

#### 二、onCreateViewHolder：创建 `ViewHolder` 容器
实例化 `ViewHolder`对象，并通过 `LayoutInflater`加载 item 布局，建立视图与 `ViewHolder` 的关联（将布局中的控件 `findViewById` 绑定到 `ViewHolder` 成员变量）。

| 场景                        | 说明                                                                                                                      |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| 初始加载可见条目                  | ​`RecyclerView` 首次显示时，为屏幕内所有可见 item 创建 ViewHolder（数量 = 可见 item 数）。                                                      |
| 滚动时新条目进入且无复用 `ViewHolder` | 用户滚动列表，新 item 滑入屏幕：  <br>① 优先从回收池（RecycledViewPool）找同类型 `ViewHolder` 复用；  <br>② 若回收池无可用 ViewHolder（如类型不匹配、缓存已耗尽），则触发创建。 |
| 遇到新的 ViewType             | 当 `getItemViewType(position)`返回**新类型值**（如不同布局的 item），即使回收池有其他类型 ViewHolder，也会强制创建新类型的 ViewHolder。                       |
| 回收池容量不足                   | 默认回收池按 ViewType 缓存 5 个 ViewHolder。若同时存在的 ViewHolder 超过容量（如快速滑动大量 item），超出部分会被销毁（而非回收），后续需重新创建。                          |
##### 触发代码
- **设置 Adapter**：`recyclerView.setAdapter(adapter)`（首次加载时触发初始创建）。
- **数据量超过屏幕可见范围**：如屏幕显示 5 个 item，但数据有 10 个，滚动到第 6 个时可能触发创建。
- **主动刷新且复用池失效**：如调用 `adapter.notifyDataSetChanged()`后，所有 ViewHolder 被标记为“无效”，可能重新创建（取决于实现）。
- **动态添加新 `ViewType`：如数据变化导致 `getItemViewType`返回新类型（例如从纯文本变为图文混合）。


#### 三、onBindViewHolder：绑定数据到 `ViewHolder`
将数据集（`List<Data>`）中指定位置的数据，填充到已创建的 `ViewHolder` 视图中（如设置文本、图片、点击事件等）。

| 场景              | 说明                                                                                               |
| --------------- | ------------------------------------------------------------------------------------------------ |
| 初始加载可见条目        | 与 `onCreateViewHolder`同步触发：创建 ViewHolder 后立即绑定数据（每个可见 item 对应一次绑定）。                              |
| 复用已有 ViewHolder | 滚动时，滑出屏幕的旧 ViewHolder 被回收，滑入屏幕的新 item 复用该 ViewHolder → 触发 `onBindViewHolder`更新数据。                |
| 数据更新（局部/全局）     | 调用 `notifyItemChanged(position)`、`notifyDataSetChanged()`等方法时，对应位置的 ViewHolder 需重新绑定最新数据。        |
| 动态增删改数据         | 如 `notifyItemInserted(position)`（插入）、`notifyItemRemoved(position)`（删除）后，受影响位置的 ViewHolder 需重新绑定。 |

##### 触发代码
- **设置 Adapter**：首次加载时，所有可见 item 绑定数据。
- **滚动列表**：复用或新建 ViewHolder 后，均会绑定当前 position 的数据。
- **数据更新通知**：  
    - `notifyDataSetChanged()`：全局刷新，所有可见 ViewHolder 重新绑定；  
    - `notifyItemChanged(position)`：局部刷新，仅指定 position 绑定；  
    - `notifyItemRangeChanged(start, count)`：批量局部刷新。
- **动态修改数据集**：如 `list.add(newData)`后调用 `notifyItemInserted`，新 item 位置触发绑定。

#### 四、不同流程表现
