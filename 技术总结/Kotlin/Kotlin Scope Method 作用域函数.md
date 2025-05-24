---
创建时间: 2025-04-25 19:51:34
作者: wangxiaoming
tags:
  - Kotlin
  - ScopeMethod
---
#### 一、作用域函数概览
Kotlin 提供 6 个作用域函数，用于在对象上下文中执行代码块，​**简化代码并提升可读性**。它们的核心差异在于 ​**对象引用方式**​ 和 ​**返回值**​：

|函数|类型|对象引用方式|返回值|是否扩展函数|
|---|---|---|---|---|
|`let`|扩展函数|`it`|Lambda 结果|是|
|`run`|扩展函数|`this`|Lambda 结果|是|
|`run`|顶层函数|无|Lambda 结果|是|
|`with`|顶层函数|`this`|Lambda 结果|是|
|`apply`|扩展函数|`this`|上下文对象本身|是|
|`also`|扩展函数|`it`|上下文对象本身|是|
#### 二、每个函数的实现与场景
##### 1）`let`
- ​**实现**​：
```kotlin
inline fun <T, R> T.let(block: (T) -> R): R {
    return block(this)
}
```
- ​**特点**​：
    - 通过 `it` 引用上下文对象（不可省略）。
    - 返回 Lambda 结果，​**不保留原对象**。
- ​**适用场景**​：
    - ​**非空处理**​：安全调用可空对象。
    - ​**变量引入**​：在 Lambda 中引入局部变量。
    - ​**链式操作**​：对对象进行临时转换或计算。
```kotlin
val length = "Hello".let { 
    println("String: $it") 
    it.length 
} // 输出 "String: Hello"，返回 5
```
##### 2）`run(扩展函数)`
- 实现：
```kotlin
inline fun <T,R> T.run(block: T.() -> R):R {
    return block()
}
```
- ​**特点**​：
    - 通过 `this` 引用上下文对象（可省略）。
    - 返回 Lambda 结果，​**不保留原对象**。
- ​**适用场景**​：
    - ​**对象初始化**​：在对象创建后执行初始化逻辑。
    - ​**复杂计算**​：在对象上下文中执行多步计算并返回结果。
```kotlin
val config = mutableMapOf<String, String>().run {
    put("key1", "value1")
    put("key2", "value2")
    this // 返回配置 Map
}
```
##### 3）`with(顶层函数)`
- ​**实现**​：
```kotlin
inline fun <T, R> with(receiver: T, block: T.() -> R): R {
    return receiver.block()
}
```
- ​**特点**​：
    - 需显式传入接收者对象，无扩展函数特性。
    - 通过 `this` 引用接收者对象。
- ​**适用场景**​：
    - ​**临时作用域**​：在非对象上下文中执行代码块。
    - ​**代码块隔离**​：避免变量名冲突。
```kotlin
val result = with(StringBuilder()) {
    append("Hello")
    append(" World")
    toString()
} // 返回 "Hello World"
```
##### 4）`apply`
- **实现：**
```kotlin
inline fun <T>T.apply(block:T.() -> Unit):T {
    block()
    return this
}
```
- **特点**​：
    - 通过 `this` 引用上下文对象（可省略）。
    - ​**返回原对象**，适合链式调用。
- ​**适用场景**​：
    - ​**对象初始化**​：快速设置多个属性。
    - ​**副作用操作**​：记录日志或触发事件。
```kotlin
val user = User().apply {
    name = "Alice"
    age = 30
    println("User created: $name, $age")
}
```
##### 5）`also`
- 实现：
```kotlin
inline fun <T> T.also(block: (T) -> Unit): T {
    block(this)
    return this
}
```
- **特点**​：
    - 通过 `it` 引用上下文对象（不可省略）。
    - ​**返回原对象**，适合链式调用。
- ​**适用场景**​：
    - ​**附加操作**​：在对象操作链中插入日志或验证。
    - ​**不可变对象处理**​：避免修改原对象。
```kotlin
val list = mutableListOf(1, 2, 3).also {
    println("List before modification: $it")
    it.add(4)
}
```
#### 三、选择指南
|​**需求**​|​**推荐函数**​|​**原因**​|
|---|---|---|
|需要返回对象本身|`apply` / `also`|链式操作，保留上下文对象|
|需要返回计算结果|`let` / `run`|结果导向，适合赋值或条件判断|
|无需对象引用，仅执行代码块|`with`|隔离作用域，避免变量冲突|
|简化可空对象操作|`let`|安全调用 `obj?.let { ... }`|
|作为函数返回值|`run` / `with`|直接返回 Lambda 结果|
#### 四、性能与注意事项
1. ​**内联优化**​：所有作用域函数默认 `inline`，避免 Lambda 包装开销。
2. ​**避免滥用**​：过度使用会降低代码可读性，建议在密集操作同一对象时使用。
3. ​**空安全**​：`let` 和 `also` 可与安全调用结合（`obj?.let { ... }`）。

#### 五、扩展场景示例
##### 1）集合操作
```kotlin
val numbers = listOf(1,2,3,4,5)
val evenSquares = numbers.filter { it % 2 == 0}
                      .map {it * it}
                      .also { println("Result: $it")}
```
##### 2）资源管理
```kotlin
File("data.txt").inputStream().use { input ->
    input.bufferedReader().use { reader ->
        reader.readText()
    }
}
```