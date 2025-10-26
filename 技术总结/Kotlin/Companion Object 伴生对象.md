---
创建时间: 2025-04-25 18:30:15
作者: wangxiaoming
tags:
  - Kotlin
  - 伴生对象
---
#### 一、核心概念
1. ​**定义与作用**​
    - 伴生对象是 Kotlin 中替代 Java 静态成员的机制，通过 `companion object` 声明，与类绑定且唯一。
    - ​**核心目的**​：提供类级别的属性和方法（类似静态成员），同时支持单例模式、工厂方法等高级功能
2. ​**语法与初始化**​
    - ​**初始化时机**​：首次通过类名访问伴生对象成员时加载（线程安全）
    - ​**命名**​：可自定义名称（如 `companion object Factory`），默认名称为 `Companion`
```kotlin
class MyClass {
    companion object {
        const val TAG = "MyClass"  // 常量
        fun create() = MyClass()   // 工厂方法
    }
}
```
#### 二、核心特性
1. **类级访问**​
    - 通过类名直接调用成员（如 `MyClass.create()`），无需实例化类
    - ​**Java 互操作**​：需使用 `@JvmStatic` 注解暴露为静态成员，`@JvmField` 暴露为静态字段
2. ​**单例性**​
    - 每个类的伴生对象是唯一实例，生命周期与类同步
    - ​**与 `object` 声明的区别**​：`object` 是独立单例，伴生对象与类强关联
3. ​**访问外部类成员**​
    - 可访问外部类的私有成员（如构造函数、属性），用于工厂模式或内部工具方法
```kotlin
class Connection private constructor() {
    companion object {
        fun create() = Connection()  // 访问私有构造函数
    }
}
```
#### 三、使用场景
##### 1）工厂模式
- 隐藏构造函数细节，提供灵活的实例化方式：
```kotlin
class User private constructor(val name: String) {
    companion object {
        fun fromJson(json: String): User { ... }
    }
}
```
##### 2）常量定义
- 替代 Java 静态常量，集中管理配置值
```kotlin
class Constants {
    companion object {
        const val MAX_SIZE = 1024
    }
}
```
##### 3）接口实现
- 伴生对象可实现接口，增强扩展性：
```kotlin
interface JsonFactory<T> { fun fromJson(json: String): T }
class Data {
    companion object : JsonFactory<Data> {
        override fun fromJson(json: String): Data { ... }
    }
}
```
##### 4）Java互操作
- 通过注解暴露静态成员，便于 Java 调用：
```kotlin
class KotlinClass {
    companion object {
        @JvmStatic fun staticMethod() {}
        @JvmField var staticField: String = "value"
    }
}
```
#### 四、与Java的对比
| **特性**​    | ​**Kotlin 伴生对象**​          | ​**Java 静态成员**​  |
| ---------- | -------------------------- | ---------------- |
| ​**语法**​   | `companion object { ... }` | `static { ... }` |
| ​**初始化**​  | 延迟加载（首次访问时）                | 类加载时立即初始化        |
| ​**访问控制**​ | 可访问外部类私有成员                 | 无法直接访问外部类私有成员    |
| ​**单例性**​  | 与类强关联，唯一实例                 | 需手动实现单例模式        |
| ​**接口实现**​ | 可直接实现接口                    | 需通过匿名内部类实现       |
#### 五、注意事项
1. ​**避免滥用**​
    - 仅用于类级别逻辑，实例方法仍需通过对象调用。
2. ​**线程安全**​
    - 伴生对象初始化是线程安全的，但内部逻辑需自行保证并发安全。
3. ​**与 `object` 的区别**​
    - `object` 是独立单例，伴生对象需绑定类，两者可结合使用：
```kotlin
object GlobalConfig { ... }
class AppConfig {
    companion object : GlobalConfig { ... }
}
```