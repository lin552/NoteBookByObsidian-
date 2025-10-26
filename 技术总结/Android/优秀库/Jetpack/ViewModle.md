---
创建时间: 2025-04-21 11:34:36
作者: wangxiaoming
tags:
  - Jetpack
  - ViewModel
---
#### 一、`ViewModel`是什么？
`ViewModel` 是 `Android Jetpack` 架构组件的核心成员，旨在**管理 `UI` 相关数据**并感知其生命周期。它通过以下特性优化开发流程：
- ​**生命周期感知**​：与 Activity/Fragment 生命周期绑定，确保数据在配置更改（如屏幕旋转）时不被销毁
- ​**数据持久化**​：独立于 `UI` 层存储数据，避免因重建导致的数据丢失
- ​**解耦与可测试性**​：分离业务逻辑与`UI`，便于单元测试和维护

#### 二、核心原理
1. ​**存储机制**​
    - `​ViewModelStore`：每个 Activity/Fragment 持有 `ViewModelStore` 对象，用于存储关联的 `ViewModel` 实例
    - ​`ViewModelProvider`​：通过工厂模式（`Factory`）创建 `ViewModel`，确保同一作用域内单例复用
2. ​**生命周期绑定**​
    - ​`Lifecycle` 感知**​：通过 `LifecycleOwner` 监听生命周期状态，仅在活跃状态（如 `STARTED`、`RESUMED`）更新 `UI`
    - ​**自动清理**​：当 Activity/Fragment 完全销毁（`onDestroy`）时，调用 `onCleared()` 释放资源，避免内存泄漏
3. ​**数据恢复**​
    - ​**配置更改保留**​：屏幕旋转时，`ViewModel` 实例由系统保留，数据通过 `ViewModelStore` 恢复

#### 三、基础与高级用法
##### 1）基础使用
```kotlin
//Activity/Fragment 中初始化
val viewModel = ViewModelProvider(this).get(MyViewModel::class.java)

//ViewModel 定义
class MyViewModel : ViewModel() {
    private val _data = MutableLiveData<String>()
    val data:LiveData<String> = _data
    fun updataData(value:String) {
        _data.value = value
    }
}
```
##### 2)高级功能
- **跨 Fragment 通信**​：同一 Activity 下的多个 Fragment 共享 `ViewModel` 实例
- ​**协程支持**​：结合 `viewModelScope` 管理异步任务，自动取消避免泄漏
- ​**数据转换**​：使用 `Transformations.map`/`switchMap` 处理数据流

#### 四、Activity 和 Fragment `ViewModel`区别
|**特性**​|​**Activity 作用域的 ViewModel**​|​**Fragment 作用域的 ViewModel**​|
|---|---|---|
|​**生命周期绑定对象**​|绑定到 Activity 的生命周期|绑定到 Fragment 的生命周期|
|​**数据共享范围**​|可被同一 Activity 下的所有 Fragment 共享|仅属于当前 Fragment（或通过子 Fragment 共享）|
|​**销毁时机**​|Activity 销毁时（如配置变更外的正常退出）|Fragment 销毁时（或从回退栈弹出时）|
|​**典型使用场景**​|跨 Fragment 共享数据、全局状态管理|Fragment 内部私有数据管理|
##### 1. 生命周期绑定的差异

`ViewModel` 的核心特性是**在配置变更（如屏幕旋转）时保留数据**，其生命周期由 `ViewModelStoreOwner` 决定。
- ​**Activity 作用域的 `ViewModel`：通过 `ViewModelProvider(activity)` 获取，生命周期与 Activity 绑定。即使 Activity 因配置变更重建（如旋转屏幕），`ViewModel` 实例会被保留，直到 Activity 真正销毁（如调用 `finish()`）。
- ​**Fragment 作用域的 `ViewModel`：通过 `ViewModelProvider(fragment)` 获取，生命周期与 Fragment 绑定。Fragment 被销毁（如从 Activity 中移除且未被回退栈保留）时，`ViewModel` 会被清除；若 Fragment 被添加到回退栈（如 `addToBackStack()`），则其 `ViewModel` 会保留到 Fragment 从回退栈弹出。

##### 2. 数据共享能力的差异
- ​**Activity 作用域的 `ViewModel`：适合作为「跨 Fragment 的共享数据容器」。同一 Activity 下的所有 Fragment 都可以通过 `ViewModelProvider(activity)` 获取同一个 `ViewModel` 实例，从而实现数据共享。  
    ​**示例**​：一个 Activity 包含 `OrderListFragment` 和 `OrderDetailFragment`，两者需要共享选中的订单数据，此时用 Activity 作用域的 `ViewModel` 存储订单信息，两个 Fragment 可以直接读写同一实例。
    
- ​**Fragment 作用域的 `ViewModel`：数据仅属于当前 Fragment，其他 Fragment 无法直接访问（除非通过接口回调或公共存储）。适合管理 Fragment 内部的私有状态（如列表的加载状态、表单输入内容等）。

##### 3.获取方式的差异
在代码中，通过 `ViewModelProvider` 指定作用域：

- ​**获取 Activity 作用域的 `ViewModel`：
```kotlin
// 在 Fragment 中获取 Activity 的 ViewModel
val activityViewModel: ActivityViewModel by viewModels({ requireActivity() })
```
（Kotlin 使用 `viewModels(ownerProducer = { ... })` 委托；Java 需手动创建 `ViewModelProvider(activity)`）

- ​**获取 Fragment 作用域的 `ViewModel`（默认方式）：
```kotlin
// 在 Fragment 中获取自身的 ViewModel（默认作用域为当前 Fragment）
val fragmentViewModel: FragmentViewModel by viewModels()
```

##### 4.生命周期行为的细节差异
- **配置变更时的表现**​：两者都会在配置变更（如旋转屏幕）时保留实例，但 Activity 作用域的 ViewModel 会存活更久——即使 Activity 因配置变更重建，其 ViewModel 仍会被新 Activity 实例复用。
- ​**Fragment 回退栈的影响**​：若 Fragment 被添加到回退栈（如导航时返回），其作用域的 `ViewModel` 不会被销毁，直到 Fragment 从回退栈中弹出；而 Activity 作用域的 `ViewModel` 仅在 Activity 真正销毁时才会被清除。

#### 五、注意事项与优化
1. ​**避免内存泄漏**​
    - ​**禁止持有 `UI` 引用**​：`ViewModel` 不应直接引用 Activity/Fragment 或 View
    - ​**使用 Application Context**​：若需 Context，通过 `AndroidViewModel` 获取 Application 级别 Context
2. ​**性能优化**​
    - ​**合理分页**​：大数据集分页加载，避免内存溢出
    - ​**结合 Repository**​：将网络/数据库操作封装到 Repository 层，`ViewModel` 仅负责逻辑调度
3. ​**数据恢复增强**​
    - ​**`SavedStateHandle`**​：通过 `SavedStateViewModelFactory` 保存关键数据，应对进程终止场景

#### 六、面试高频考点
1. ​**`ViewModel` 与 MVP 的 Presenter 区别**​
    - `ViewModel` 自动管理生命周期，无需手动释放资源；Presenter 需处理生命周期和内存泄漏
2. ​**屏幕旋转时数据恢复原理**​
    - `ViewModel` 实例存储在 `ViewModelStore` 中，由系统在配置更改时保留并重新绑定到新 Activity
3. ​`LiveData` 与 `ViewModel` 协同机制**​
    - `ViewModel` 持有 `LiveData`，`UI` 通过 `observe()` 监听数据变化，实现响应式更新
4. ​`ViewModel` 的线程安全性**​
    - 数据更新需在主线程调用 `setValue()`，子线程使用 `postValue()`

#### 七、适用场景
1. ​**数据持久化需求**​
    - 配置更改频繁的场景（如横竖屏切换、分屏模式）
2. ​**复杂 `UI` 状态管理**​
    - 表单输入、多步骤流程等需要维护中间状态的场景
3. ​**跨组件通信**​
    - Activity 与多个 Fragment 共享数据（如购物车总价）
4. ​`MVVM` 架构实现**​
    - 作为 Model 与 View 的桥梁，结合 `Data Binding` 或 `LiveData` 实现双向绑定