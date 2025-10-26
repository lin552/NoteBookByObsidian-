---
创建时间: 2025-04-25 17:21:54
作者: wangxiaoming
tags:
  - Kotlin
  - 内联函数
---
#### 一、核心概念
1. ​**定义**​  
    通过 `inline` 关键字修饰的函数，编译时将其代码直接插入调用处，消除高阶函数（如 `Lambda`）的额外开销
2. ​**作用**​
    - ​**减少内存分配**​：避免创建匿名内部类（`Lambda` 的 `JVM` 实现）。
    - ​**提升性能**​：消除函数调用栈帧的创建与销毁。
    - ​**支持非局部返回**​：Lambda 中可直接使用 `return` 退出外层函数
#### 二、工作原理
##### 1）代码替换
编译器将内联函数的代码及 Lambda 表达式替换到调用位置，例如：
```kotlin
inline fun test(block: () -> Unit) { block() }
test { println("Hello") }
```

反编译后等效为：
```kotlin
System.out.println("Hello"); // 直接插入代码，无 Lambda 对象
```
##### 2）`Lambda`内联优化
- 默认内联所有函数类型参数。
- 使用 `noinline` 限制特定参数内联：
```kotlin
inline fun test(noinline block: () -> Unit) { ... }
```
#### 三、适用场景
1. **高频调用的轻量函数**​  
    如集合操作函数 `map`、`filter`，减少中间对象创建
2. ​**非局部返回**​  
    在 Lambda 中直接退出外层函数：
	```kotlin
	inline fun loop() {
	    for (i in 1..10) {
	        if (i == 5) return // 直接退出 loop()
	    }
	}	```
3. ​**与协程结合**​  
    优化挂起函数调用链的性能，如 `withContext`
#### 四、注意事项
1. ​**代码膨胀风险**​  
    内联函数体较大时，多次调用会导致代码冗余，建议函数体简洁
2. ​**递归函数限制**​  
    递归函数无法内联（编译器报错）
3. ​**类型擦除问题**​  
    泛型内联函数可通过 `reified` 关键字获取具体类型：
	```kotlin
	inline fun <reified T> parseJson(json: String): T { ... }
	```
4. ​**跨模块内联**​  
    跨模块的内联函数需确保编译依赖关系正确。
#### 五、与非内联函数对比
|​**特性**​|​**内联函数**​|​**普通函数**​|
|---|---|---|
|​**内存分配**​|无 Lambda 对象创建|每次调用生成匿名类实例|
|​**性能**​|更高（减少栈帧操作）|较低（存在虚拟调用开销）|
|​**非局部返回**​|支持|不支持（需 `return@label`）|
|​**适用场景**​|轻量级、高频操作|复杂逻辑、大函数体|
#### 六、实际案例
##### 1）优化集合操作
```kotlin
//todo 类型转换不对，待研究
inline fun <T> List<T>.filterEven(): List<T> {
    return this.filter { it % 2 == 0 }
}
// 编译后直接内联 filter 逻辑，避免额外对象
```
##### 2）避免Lambada对象
```kotlin
/**  
 * 避免Lambda对象  
 */  
//非内联 (创建 Function0 对象)  
fun log(message:String,block:() -> Unit) {  
    block()  
}  
  
//内联(直接插入代码)  
inline fun logInline(message: String,block: () -> Unit){  
    block()  
}
```
##### 3）协程上下文切换
```kotlin
/**  
 * 协程上下文切换  
 */  
inline suspend fun <T> withContext(  
    context:CoroutineContext,  
    block:suspend CoroutineScope.() -> T  
): T {  
    return suspendCoroutine {   
          
    }  
}
```
#### 七、面试考察点
1. **内联与宏的区别**​  
    Kotlin 内联是编译时代码替换，类似 C/C++ 宏，但更安全（类型检查、作用域管理）。
2. ​**内联对泛型的影响**​  
    `reified` 关键字使泛型类型在运行时可用，解决类型擦除问题
3. ​**性能调优实践**​  
    如何通过内联减少内存占用，或通过 `noinline` 控制内联范围。