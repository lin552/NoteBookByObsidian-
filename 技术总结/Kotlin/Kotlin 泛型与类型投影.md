---
创建时间: 2025-04-25 17:47:31
作者: wangxiaoming
tags:
  - Kotlin
  - 泛型
---
#### 一、泛型基础
##### 1）泛型类与函数
- **声明方式**​：通过尖括号 `<T>` 定义类型参数，支持默认上界 `Any?`（允许 `null`）
```kotlin
class Box<T : Number>(val value: T) //T 必须是 Number 或其子类  
  
fun <T> singletonList(item:T):List<T> //默认上界为Any?  
{  
    return emptyList()  
}
```
- **类型推断**​：Kotlin 支持省略显式类型参数，由编译器自动推断。
```kotlin
val intBox = Box(10) //自动推断 T 为Int
```
##### 2）类型擦除
- ​**运行时限制**​：泛型类型信息在编译后被擦除，无法直接获取具体类型（如 `List<String>` 运行时为 `List`）。
- ​**解决方案**​：通过 `reified` 关键字在 `inline` 函数中保留类型信息。
```kotlin
inline fun <reified T> parseJson(json: String): T { ... }
```
#### 二、型变（Variance）
##### 1）协变（`Covariance`）
- ​**定义**​：允许子类型替代父类型，使用 `out` 修饰符。
```kotlin
interface Producer<out T> {
   fun produce():T //T 仅作为返回值（输出）
}
```
- **场景**​：生产者类（如 `List<out T>`），确保类型安全。
```kotlin
val strings:Producer<String> = Producer()
val arrys:Producer<Any> = strings //允许协变
```
##### 2）逆变（`Contravariance`）
- ​**定义**​：允许父类型替代子类型，使用 `in` 修饰符。
```kotlin
interface Consumer<in T> {
    fun consume(item: T) //T 仅作为入参（输入）
}
```
- **场景**​：消费者类（如 `Comparator<in T>`），支持更灵活的类型适配。
```kotlin
val intConsumer: Consumer<Int> = Consumer()
val anyConsumer: Consumer<Any> = intConsumer //允许逆变
```
##### 3）不变（`Invariance`）
- **默认行为**​：未声明 `in` 或 `out` 时，泛型类型严格匹配。
```kotlin
class Box<T>(val value:T)
val intBox:Box<Int> = Box(10)
val anyBox:Box<Any> = intBox //编译错误，类型不匹配
```
#### 三、类型投影（Type Projections）
##### 1）声明处型变
- ​**作用**​：在类/接口定义时指定泛型参数的协变或逆变行为。
```kotlin
interface List<out T> { //协变，允许子类型替换
   fun get(index:Int):T
}
```
##### 2）使用处型变（类型投影）
- **语法**​：在泛型类型后使用 `in` 或 `out` 修饰，不改变原始类定义。
```kotlin
fun copy(from:Array<out Any>,to:Array<Any>) {
   ...
}
```
- **应用场景**​：
	- ​**只读投影**​：`MutableList<out T>` 限制仅读取操作。
	- ​**只写投影**​：`MutableList<in T>` 限制仅写入操作。
##### 3）星号投影（*）
- ​**含义**​：表示未知类型，等价于 `out Any?`（协变）或 `in Nothing`（逆变）。
```kotlin
fun printList(list: MutableList<*>) { //只能读取元素（类型为Any?）
   println(list[0])
}
```
- ​**限制**​：无法向星号投影的集合添加元素（类型不确定）。
#### 四、核心区别与对比
|​**特性**​|​**协变（out）​**​|​**逆变（in）​**​|​**不变**​|
|---|---|---|---|
|​**类型关系**​|子类可替代父类|父类可替代子类|严格匹配|
|​**使用位置**​|仅输出（返回值）|仅输入（参数）|输入/输出|
|​**典型场景**​|`List<out T>`、`Sequence<out T>`|`Comparator<in T>`|`MutableList<T>`|
#### 五、实际应用与注意事项
##### 1）API设计
​**生产者接口**​：使用 `out` 保证类型安全。
```kotlin
interface Producer<out T> { fun produce(): T }
```
​**消费者接口**​：使用 `in` 扩展兼容性。
```kotlin
interface Consumer<in T> { fun consume(item: T) }
```
##### 2）避免类型擦除问题
​**内联函数 + `reified`**​：在需要运行时类型信息时使用。
```kotlin
inline fun <reified T> createInstance() = T::class.java.newInstance()
```
##### 3）常见错误
- ​**协变类型参数不可变**​：`List<out T>` 无法调用 `add()`。
- ​**逆变类型参数不可读**​：`Comparator<in T>` 无法读取元素。
#### 六、与Java的对比
|**特性**​|​**Kotlin**​|​**Java**​|
|---|---|---|
|​**默认上界**​|`Any?`（允许 `null`）|`Object`（不允许 `null`）|
|​**通配符**​|无，通过型变实现|使用 `? extends` 和 `? super`|
|​**星号投影**​|支持（`*`）|支持（`?`）|
|​**类型擦除**​|与 Java 类似|完全类型擦除|
