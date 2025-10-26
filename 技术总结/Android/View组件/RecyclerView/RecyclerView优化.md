---
创建时间: "2025-07-29 20:56:39"
作者: wangxiaoming
tags:
---
#### 一、布局优化：固定尺寸&简化层级
##### 1.固定Item尺寸（减少布局计算）
若列表 Item 高度/宽度固定（如聊天列表、商品列表），在 Adapter 初始化时调用 `setHasFixedSize(true)`，避免 `RecyclerView` 重复计算布局。

```kotlin
// 在 Activity/Fragment 中设置
val recyclerView = findViewById<RecyclerView>(R.id.recycler_view)
recyclerView.setHasFixedSize(true) // 关键：声明 Item 尺寸固定
recyclerView.adapter = MyAdapter()
```

##### 2.简化 Item 布局层级（使用 `ConstraintLayout` + merge）
传统嵌套布局（如 `LinearLayout` 套 `RelativeLayout`）会增加测量时间，改用 `ConstraintLayout` 或 `merge` 减少层级。

#### 二、数据更新优化：局部刷新 & `DiffUtil`
##### 1. 使用 `DiffUtil` 计算数据差异（替代 `notifyDataSetChanged`）
`DiffUtil` 可自动对比新旧数据集，生成需要更新的 Item 列表，避免全量刷新。
步骤1：定义`DiffUtil.Callback`
```kotlin
class MyDiffCallback(
    private val oldList: List<Item>,
    private val newList: List<Item>
) : DiffUtil.Callback() {

    // 旧数据集大小
    override fun getOldListSize() = oldList.size

    // 新数据集大小
    override fun getNewListSize() = newList.size

    // 对比两个位置的 Item 是否相同（通常用 id 判断）
    override fun areItemsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean {
        return oldList[oldItemPosition].id == newList[newItemPosition].id
    }

    // 对比相同位置 Item 的内容是否相同（需根据业务字段判断）
    override fun areContentsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean {
        val oldItem = oldList[oldItemPosition]
        val newItem = newList[newItemPosition]
        return oldItem.title == newItem.title 
            && oldItem.content == newItem.content 
            && oldItem.time == newItem.time
    }
}
```
步骤2：在Adapter中应用`DiffUtil`
```kotlin
class MyAdapter : RecyclerView.Adapter<MyAdapter.ViewHolder>() {

    private var dataList: List<Item> = emptyList()

    // 更新数据时使用 DiffUtil
    fun updateData(newList: List<Item>) {
        val diffResult = DiffUtil.calculateDiff(MyDiffCallback(dataList, newList))
        dataList = newList
        diffResult.dispatchUpdatesTo(this) // 关键：通过 dispatchUpdatesTo 触发局部刷新
    }

    // 其他 Adapter 方法（onCreateViewHolder、onBindViewHolder 等）
}
```
**优势**​：相比 `notifyDataSetChanged()`，`DiffUtil` 仅更新变化的 Item，减少无效绘制。

#### 三、内存优化：`ViewHolder`复用&图片加载
##### 1.正确实现 `ViewHolder`（避免重复创建 View）
`ViewHolder` 必须缓存 Item 内部的 View，避免每次 `onBindViewHolder` 都调用 `findViewById`。
```kotlin
class MyAdapter : RecyclerView.Adapter<MyAdapter.ViewHolder>() {

    class ViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        // 缓存 Item 内部的 View（如 TextView、ImageView）
        val titleTv: TextView = itemView.findViewById(R.id.tv_title)
        val contentTv: TextView = itemView.findViewById(R.id.tv_content)
        val timeTv: TextView = itemView.findViewById(R.id.tv_time)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(parent.context).inflate(R.layout.item_view, parent, false)
        return ViewHolder(view) // 复用 ViewHolder
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        val item = dataList[position]
        holder.titleTv.text = item.title
        holder.contentTv.text = item.content
        holder.timeTv.text = item.time
    }
}
```

##### 2.内存与缓存管理
- **动态调整缓存池**：
    - `mCachedViews` 扩容：`recyclerView.setItemViewCacheSize(20)` 提升快速反向滑动的流畅性
    - 共享 `RecycledViewPool`：多个列表实例共享池，减少内存碎片（适用于电商首页多 Tab 列表）
- ​**大图分块加载**：使用 `BitmapRegionDecoder` 实现长图按需加载，避免单张图片内存溢出

##### 3. 图片加载优化（Glide 异步加载 + 缓存）
滑动时加载大图会导致卡顿，使用 Glide 自动管理内存缓存和磁盘缓存，并在滑动时暂停加载。
```kotlin
override fun onBindViewHolder(holder: ViewHolder, position: Int) {
    val item = dataList[position]
    
    // 加载网络图片（自动压缩、缓存）
    Glide.with(holder.itemView.context)
        .load(item.imageUrl) // 图片 URL
        .placeholder(R.drawable.placeholder) // 占位图（加载中）
        .error(R.drawable.error) // 错误图（加载失败）
        .override(200, 200) // 限制图片尺寸（避免内存溢出）
        .into(holder.ivImage)

    // 滑动时暂停加载（可选优化）
    (holder.itemView.context as? AppCompatActivity)?.lifecycle?.addObserver(object : LifecycleEventObserver {
        override fun onStateChanged(source: LifecycleOwner, event: Lifecycle.Event) {
            if (event == Lifecycle.Event.ON_PAUSE) {
                Glide.with(holder.itemView.context).pauseRequests() // 暂停加载
            } else if (event == Lifecycle.Event.ON_RESUME) {
                Glide.with(holder.itemView.context).resumeRequests() // 恢复加载
            }
        }
    })
}
```

#### 四、混动性能优化：预加载&分页加载
##### 1.预加载（`Pre-fetch`）优化
`RecyclerView` 支持预加载相邻 Item，减少滑动时的空白等待。可通过 `LayoutManager` 或自定义 `ItemPrefetchCallback` 实现。
​示例：为 `LinearLayoutManager` 启用预加载
```kotlin
val layoutManager = LinearLayoutManager(context)
layoutManager.setItemPrefetchEnabled(true) // 启用预加载（默认已启用）
layoutManager.setInitialPrefetchItemCount(4) // 初始预加载数量（根据需求调整）

recyclerView.layoutManager = layoutManager
```

##### 2.滚动性能优化
- **滚动任务调度**：通过 `addOnScrollListener` 监听滑动状态，暂停非核心任务（如动画、日志上报）
- ​**轻量化动画**：关闭默认 `ItemAnimator` 或自定义简单动画（如 Alpha 渐变），减少 GPU 绘制压力

##### 3.分页加载（Paging 3 库）
数据量过大时，使用 `Paging 3` 实现分页加载，仅加载当前可见数据，避免内存溢出。
```kotlin
class MyPagingSource(
    private val apiService: ApiService,
    private val query: String
) : PagingSource<Int, Item>() {

    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, Item> {
        val page = params.key ?: 1 // 默认加载第一页
        return try {
            val response = apiService.getItems(query, page, params.loadSize) // 调用接口获取分页数据
            val nextPage = if (response.items.isNotEmpty()) page + 1 else null
            LoadResult.Page(
                data = response.items,
                prevKey = if (page == 1) null else page - 1,
                nextKey = nextPage
            )
        } catch (e: Exception) {
            LoadResult.Error(e)
        }
    }

    override fun getRefreshKey(state: PagingState<Int, Item>): Int? {
        return state.anchorPosition?.let { anchorPosition ->
            state.closestPageToPosition(anchorPosition)?.prevKey?.plus(1)
                ?: state.closestPageToPosition(anchorPosition)?.nextKey?.minus(1)
        }
    }
}
```

#### 五、其他优化：避免冗余操作
##### 1.避免在 `onBindViewHolder` 中做耗时操作
```kotlin
// 在数据更新时预处理（如在 ViewModel 或 Repository 中）
fun preprocessData(rawList: List<RawItem>): List<Item> {
    return rawList.map { rawItem ->
        val parsedData = JsonParser.parse(rawItem.rawJson) // 预处理耗时操作
        Item(id = rawItem.id, parsedData = parsedData)
    }
}

// Adapter 直接使用预处理后的数据
override fun onBindViewHolder(holder: ViewHolder, position: Int) {
    val item = dataList[position] // 已预处理
    holder.bind(item.parsedData) // 无需耗时操作
}
```

#### 六、高级场景优化
- ​**瀑布流布局优化**：通过 `GapWorker` 预计算动态高度，避免布局抖动（`StaggeredGridLayoutManager` 特有）
- ​**分页加载策略**：结合 `Paging 3.0` 实现数据库与网络数据的分页加载，减少单次数据量对渲染的影响
- ​**视图绑定加速**：使用 `ViewBinding` 或 `DataBinding` 替代 `findViewById`，单次绑定耗时从 `1.2ms` 降至 `0.3ms`