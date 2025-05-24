---
创建时间: 2025-04-25 18:56:08
作者: wangxiaoming
tags:
  - Kotlin
  - 密封类
---
#### 一、核心特性
1. ​**受限继承**​
    - 密封类的子类必须与密封类定义在 ​**同一文件**​ 或 ​**嵌套在密封类内部**​（Kotlin 1.1+ 允许子类在同级包中）
    - ​**目的**​：确保类型集合的封闭性，避免外部随意扩展。
2. ​**抽象性与不可实例化**​
    - 密封类默认是抽象的，不能直接实例化。
    - 构造函数默认为 `private`，子类可通过 `protected` 或 `internal` 访问
3. ​**与 `when` 表达式深度集成**​
    - 使用 `when` 匹配密封类子类时，若覆盖所有分支，​**无需 `else` 子句**​（编译器强制检查）
```kotlin
sealed class Result {  
    data class Success(val data: String = "") : Result()  // 参数带默认值  
    data class Error(val message: String = "") : Result()  // 参数带默认值  
    data object Loading : Result()  // 无参数单例  
}  
  
fun handleResult(result: Result) {  
    when (result) {  
        is Result.Success -> { /* 处理成功逻辑 */ }  
        is Result.Error -> { /* 处理错误逻辑 */ }  
        Result.Loading -> {  
        }  
    }  
}
```
#### 二、与枚举（`Enum`）的对比
|**特性**​|​**密封类**​|​**枚举**​|
|---|---|---|
|​**实例数量**​|子类可有多个实例（含状态）|每个枚举常量仅一个实例|
|​**状态支持**​|子类可包含属性和方法（如数据类）|无状态|
|​**扩展性**​|子类可嵌套或定义在同级包|所有枚举常量必须在同一文件|
|​**类型安全**​|编译期强制覆盖所有子类分支|依赖 `when` 的 `else` 分支|
#### 三、实际应用示例
#####  1）网络请求结果处理
```kotlin
/**  
 * 网络请求结果处理  
 */  
sealed class NetworkResult<out T>{  
    data class Success<out T>(val data:T) :NetworkResult<T>()  
    data class Error(val code:Int,val message: String) :NetworkResult<Nothing>()  
    data object Loading:NetworkResult<Nothing>()  
}  
  
fun handleResponse(response:NetworkResult<String>){  
    when(response){  
        is NetworkResult.Success -> {  
  
        }  
        is NetworkResult.Error -> {  
  
        }  
        NetworkResult.Loading -> {  
  
        }  
    }  
}
```
##### 2）类安全的`JSON`解析
```kotlin
todo
```
#### 四、注意事项
1. ​**子类限制**​
    - 子类可以是 `data class`、`object` 或普通类，但需在密封类定义的 ​**同一模块**​ 内
    - 若子类在密封类外部，需使用 `@JvmSynthetic` 或 `@JvmOverloads` 控制可见性。
2. ​**避免滥用**​
    - 仅用于类型有限且需要严格匹配的场景（如状态、结果）。
    - 复杂逻辑建议结合接口或抽象类实现。
3. ​**与 `sealed interface` 的区别**​
    - Kotlin 1.5+ 支持密封接口（`sealed interface`），功能类似密封类，但更轻量，适合接口层级结构
