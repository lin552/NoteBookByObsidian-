---
创建时间: 2025-04-24 10:33:58
作者: wangxiaoming
tags:
  - SharePreference
---
#### 一、什么是`SharePreference`
- **定义**​：  
    SP（`SharedPreferences`）是 Android 提供的轻量级键值对存储方案，用于保存应用配置、用户偏好等小规模数据（如登录状态、主题设置）。数据以 XML 文件形式存储在 `/data/data/<包名>/shared_prefs/` 目录下
- ​**核心特性**​：
    - ​**键值对存储**​：支持基本类型（String、Int、Boolean 等）。
    - ​**持久化**​：数据在应用关闭后仍保留。
    - ​**轻量级**​：适合小数据量（建议单文件不超过 `1MB`）。
#### 二、如何使用？
##### 基本操作步骤
###### 1）获取SP对象
```java
SharePreference sp = context.getSharePreferences("filename",Content.MODE_PRIVATE);
```
- `filename`：文件名（自动生成 `.xml` 后缀）。
- `MODE_PRIVATE`：私有模式（默认，其他应用不可读写）
##### 2）编辑数据
```java
SharedPreferences.Editor editor = sp.edit();
editor.putString("username", "张三");
editor.putInt("age", 25);
editor.apply(); // 或 commit()
```
##### 3）读取数据
```java
String username = sp.getString("username", "默认值");
```

#### 三、原理与存储机制
- ​**存储位置**​：  
    XML 文件存储在应用私有目录，通过 `Context.getSharedPreferences()` 访问
- ​**读写流程**​：
    - ​**读取**​：首次读取时解析 XML 文件到内存，后续直接访问内存数据。
    - ​**写入**​：通过 `commit()`（同步写入磁盘）或 `apply()`（异步写入磁盘）提交修改
- ​**线程安全**​：
    - 读操作通过锁机制保证线程安全。
    - 写操作通过 `Editor` 的串行化处理避免冲突。

#### 四、使用注意点
1. ​**内存泄漏风险**​：
    - 避免静态持有 `SharedPreferences` 或 `Editor` 引用（如 `static SharedPreferences sp`）。
    - 使用 `Application` 上下文而非 `Activity` 上下文获取 SP 实例
2. ​**性能问题**​：
    - ​**主线程读写大文件**​：可能导致 `ANR`。
    - ​**频繁调用 `commit()`**​：同步写入磁盘阻塞 `UI` 线程。
3. ​**数据类型限制**​：
    - 仅支持基本类型，复杂数据需序列化（如 `JSON` 字符串）。
4. ​**文件模式弃用**​：
    - `MODE_WORLD_READABLE` 和 `MODE_WORLD_WRITEABLE` 已废弃，存在安全风险。

#### 五、可优化点
1. ​**异步写入**​：
    - 优先使用 `apply()` 替代 `commit()`，避免主线程阻塞
2. ​**数据分片**​：
    - 按功能拆分多个 SP 文件（如 `user_prefs.xml`、`config_prefs.xml`）。
3. ​**替代方案**​：
    - ​`MMKV`​：腾讯开源的高性能键值存储库，支持多进程、更快的读写速度。
    - ​`Jetpack DataStore`：基于 Kotlin 协程，支持异步操作和类型安全
4. ​**避免频繁读写**​：
    - 缓存热点数据到内存（如单例对象）。

#### 六、面试考察点
1. ​**SP 和数据库（SQLite）的区别？​**​
    - ​**SP**​：键值对、轻量级、适合小数据（如配置）。
    - ​**SQLite**​：结构化存储、支持复杂查询、适合大数据（如用户数据）
2. ​**`commit()` 和 `apply()` 的区别？​**​
    - `commit()`：同步写入磁盘，返回布尔值表示成功与否，可能阻塞 `UI` 线程。
    - `apply()`：异步写入磁盘，无返回值，通过监听队列确保最终写入
3. ​**SP 如何避免内存泄漏？​**​
    - 使用 `Application` 上下文获取 SP 实例。
    - 避免在匿名内部类（如线程、Handler）中持有 SP 引用
4. ​**SP 是否支持多进程？​**​
    - 默认不支持，需通过 `MODE_MULTI_PROCESS`（已废弃）或改用 `MMKV/DataStore`。

#### 七、适用场景
- **用户偏好设置**​：如主题、语言、通知开关。
- ​**轻量级配置**​：如 API 地址、调试开关。
- ​**临时缓存**​：如登录 Token（需配合加密）。
- ​**不适用场景**​：
    - 大数据（如图片、文件路径）。
    - 复杂数据结构（需序列化）。
    - 高频读写（如日志记录）。