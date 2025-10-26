---
创建时间: "2025-07-28 13:23:12"
作者: wangxiaoming
tags:
---
Kotlin ​**Flow**​ 是 Kotlin 协程（`Coroutines`）生态中用于处理**异步数据流**的核心组件，主要用于建模“随时间发射多个值的异步序列”。它解决了传统异步编程中处理连续数据流（如实时更新、分页加载、事件流）的痛点，提供了声明式、结构化的流处理能力。

#### 一、Flow的核心特性
Flow 的设计围绕“异步数据流”展开，核心特性包括：
##### 1.冷流（Cold Stream）
Flow 是“冷”的：​**只有在被收集（`collect`）时才会开始发射数据**。如果没有任何协程收集它，Flow 不会执行任何发射逻辑。这意味着 Flow 的执行是“按需触发”的，避免了不必要的资源消耗。
```kotlin
val flow = flow {
    println("开始发射数据")
    emit(1)
    emit(2)
}

// 此时 Flow 尚未执行（未收集）
flow.collect { value -> 
    println("收到：$value") 
} 
// 输出：
// 开始发射数据
// 收到：1
// 收到：2
```

##### 2.异步发射与收集
Flow 的发射（`emit`）和收集（`collect`）必须在**协程作用域**中进行（通过 `CoroutineScope` 或挂起函数），天然支持异步操作。发射端可以在协程中暂停、恢复或取消，收集端也可以异步处理每个值。

##### 3.背压（`Backpressure`）处理
当数据发射速度快于收集速度时，Flow 提供了多种策略处理背压（如缓冲、丢弃旧数据、最新数据优先等），避免下游处理不过来导致的阻塞或内存溢出。

##### 4.取消传播
Flow 的生命周期与协程作用域绑定。如果收集 Flow 的协程被取消（如父作用域取消、生命周期结束），Flow 的发射会立即终止，释放资源（结构化并发的核心优势）。

#### 二、Flow的构建与发射
Flow 通过**构建器函数**创建，常见的构建方式包括：

##### 1.`flow` 构建器（最常用）
通过 `flow { ... }` 挂起函数构建 Flow，内部使用 `emit` 发射值。
```kotlin
fun simpleFlow(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100) // 模拟耗时操作
        emit(i)    // 发射值
    }
}
```

##### 2.`flowOf` 构建器
用于发射固定数量的已知值（非动态生成）。
```kotlin
val fixedFlow = flowOf(1, 2, 3, 4) // 发射 1,2,3,4
```

##### 3.`channelFlow` 构建器
更灵活的构建方式，允许在协程中使用 `send` 发送值（支持在非挂起上下文中发送，但需注意协程作用域）。
```kotlin
fun channelBasedFlow(): Flow<Int> = channelFlow {
    coroutineScope {
        launch { send(1) } // 在子协程中发送
        launch { send(2) }
    }
    send(3) // 主协程发送
}
```

##### 4.其他便携函数
如 `asFlow()` 可将集合、序列等转换为 Flow：
```kotlin
(1..3).asFlow() // 等价于 flowOf(1,2,3)
```

#### 三、Flow的操作符
Flow 提供了丰富的操作符，用于转换、组合、过滤或处理数据流，主要分为以下几类：

##### 1.转换操作符（Transformation）
对每个发射的值进行处理，生成新的流。

- `map`：将每个值映射为另一个值（同步转换）。
```kotlin
flowOf(1, 2, 3).map { it * 2 } // 发射 2,4,6
```

- `flatMap`：将每个值转换为一个新的 Flow，最终合并所有子 Flow 的结果（支持异步转换）。
```kotlin
flowOf(1, 2).flatMap { num ->
    flow { 
        delay(100) 
        emit(num * 10) 
    }
} // 发射 10, 20（间隔约 100ms）
```

- `transform`：更灵活的转换，可发射多个值或修改上下文。
```kotlin
flowOf(1, 2).transform { value ->
    emit("开始处理 $value")
    emit(value * 3)
} // 发射 "开始处理 1", 3, "开始处理 2", 6
```

##### 2.组合操作符（Combination）
合并多个 Flow 的结果。

- `zip`：按顺序配对两个 Flow 的值（一一对应），直到其中一个 Flow 结束。
```kotlin
val flow1 = flowOf(1, 2, 3)
val flow2 = flowOf("A", "B")
flow1.zip(flow2) { num, str -> "$num$str" } // 发射 "1A", "2B"
```

- `combine`：实时合并两个 Flow 的最新值（每当任一 Flow 发射新值时触发）。
```kotlin
val flowA = MutableStateFlow(0)
val flowB = MutableStateFlow("")
combine(flowA, flowB) { a, b -> "A=$a, B=$b" }
    .collect { println(it) } // 初始发射 "A=0, B="

flowA.value = 1  // 触发发射 "A=1, B="
flowB.value = "X" // 触发发射 "A=1, B=X"
```

- `merge`：合并多个 Flow 的发射顺序（按发射时间交织）。
```kotlin
val flowX = flowOf(1, 3)
val flowY = flowOf(2, 4)
merge(flowX, flowY).collect { println(it) } // 可能发射 1,2,3,4 或 1,2,4,3（取决于调度）
```

##### 3.终端操作符（Terminal）
触发 Flow 的收集，并返回最终结果。
- `collect`：最基础的收集操作，逐个处理发射的值（挂起函数）。
```kotlin
simpleFlow().collect { value -> 
    println("处理：$value") 
}
```

`first`：收集第一个值并返回。
```kotlin
val firstValue = flowOf(1, 2, 3).first() // 1
```

