---
创建时间: 2025-04-25 18:41:01
作者: wangxiaoming
tags:
  - Kotlin
  - 数据类
---
#### 一、核心特性
1. **自动生成标准方法**​
    - ​**`toString()`**​：输出类名及所有主构造函数属性值（如 `User(name=Alice, age=30)`）
    - ​**`equals()`/`hashCode()`**​：基于主构造函数属性判断对象相等性及哈希值
    - ​**`copy()`**​：创建对象的浅拷贝，支持修改部分属性（如 `user.copy(age = 31)`）
    - ​**`componentN()`**​：按声明顺序生成解构函数（如 `val (name, age) = user`）
2. ​**语法要求**​
    - ​**主构造函数**​：必须至少有一个 `val` 或 `var` 参数，且参数不可省略
    - ​**不可变性**​：属性默认为 `val`（不可变），若需可变需显式声明 `var`
    - ​**限制**​：不能是 `abstract`、`open`、`sealed` 或 `inner` 类
#### 二、使用场景
1. ​**数据传输对象（`DTO`）​**​
    - 用于网络请求或数据库交互，简化数据模型定义：
```kotlin
data class User(val id: Int, val name: String, val email: String)
```
1. ​**状态管理**​
    - 表示 `UI` 或业务逻辑状态（如加载中、错误信息）：
```kotlin
data class LoginState(val isLoading: Boolean, val error: String?)
```
1. ​**集合操作**​
    - 结合 `map`、`filter` 处理对象集合：
```kotlin
val users = listOf(User("Alice", 30), User("Bob", 25))
val names = users.map { it.name } // ["Alice", "Bob"]
```
1. ​**解构声明**​
    - 直接提取属性值：
```kotlin
val user = User("Alice", 30)
val (name, age) = user // 解构赋值
```
1. ​**与 Java 互操作**​
    - 通过 `@JvmStatic` 和 `@JvmField` 暴露静态成员：
```kotlin
data class User(val name: String) {
    @JvmStatic fun staticMethod() {}
    @JvmField var staticField: String = "value"
}
```
#### 三、注意事项与限制
1. ​**主构造函数属性限制**​
    - 自动生成的方法仅基于主构造函数属性，类体中声明的属性不会参与（需手动重写）
```kotlin
data class Person(val name: String) {
    var age: Int = 0 // 不参与 equals()/copy()
}
```
1. ​**继承与实现**​
    - 数据类可继承其他类或实现接口，但需注意方法覆盖规则
```kotlin
open class Animal(val species: String)
data class Dog(val name: String, species: String) : Animal(species)
```
1. ​**深拷贝问题**​
    - `copy()` 是浅拷贝，引用类型属性需手动深拷贝：
```kotlin
data class Company(val employees: MutableList<String>)
val company1 = Company(mutableListOf("Alice"))
val company2 = company1.copy() // employees 共享同一列表
```
1. ​**默认值与空安全**​
    - 主构造函数参数可指定默认值，支持可空类型：
```kotlin
data class User(val name: String, val age: Int = 0, val email: String? = null)
```
#### 四、与普通类对比
|​**特性**​|​**数据类**​|​**普通类**​|
|---|---|---|
|​**`toString()`**​|自动生成（显示属性值）|默认输出类名和哈希值|
|​**`equals()`/`hashCode()`**​|基于主构造函数属性|基于对象引用|
|​**`copy()`**​|自动生成浅拷贝|需手动实现|
|​**解构支持**​|自动生成 `componentN()`|需手动实现|
|​**适用场景**​|纯数据载体|含复杂逻辑的类|
