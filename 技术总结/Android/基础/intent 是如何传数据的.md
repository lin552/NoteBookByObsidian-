---
创建时间: "2025-06-17 09:39:17"
作者: wangxiaoming
tags:
---
#### 一、数据封装：Intent的Bundle的关系、
`Intent` 本身的核心功能是**描述组件间的通信意图**​（如启动 `Activity`、发送广播等），而真正承载数据的载体是它内部的 `Bundle` 对象。`Intent` 通过 `putExtra()` 方法存储的所有键值对，最终都会被封装到 `Bundle` 中。
##### 1.Intent的内部结构
`Intent` 类内部定义了一个 `mExtras` 字段（类型为 `Bundle`），用于存储额外数据：
```java
// AOSP 源码（简化版）
public class Intent implements Parcelable, Cloneable {
    private Bundle mExtras;  // 数据存储的核心容器

    public Intent putExtra(String name, boolean value) {
        if (mExtras == null) {
            mExtras = new Bundle();  // 初始化 Bundle
        }
        mExtras.putBoolean(name, value);  // 存储数据到 Bundle
        return this;
    }

    // 其他 putExtra 方法类似，最终都会操作 mExtras
}
```
所有通过 `putExtra()` 传递的数据（基本类型、`Parcelable`、`Serializable` 等）都会被存入 `mExtras` 这个 `Bundle` 中。
##### 2.Bundle的本质，轻量级键值对容器
`Bundle` 继承自 `BaseBundle`，而 `BaseBundle` 内部通过 ​**类型数组 + 键值索引**​ 的方式存储数据。例如：
- 对于 `int` 类型数据，使用 `int[]` 数组存储值，用 `String[]` 数组存储对应的键；
- 对于 `Parcelable` 类型数据，直接存储对象引用（或通过 `Parcel` 序列化后的数据）。
这种设计使得 `Bundle` 在内存中高效存储小批量数据，同时支持快速通过键查找值。

#### 二、跨进程传输：Binder与Parcel的协作
当 `Intent` 用于启动其他组件（尤其是跨应用或跨进程的组件）时，数据需要通过 Android 的 ​**Binder 进程间通信（`IPC`）机制**​ 传输。此时，`Bundle` 中的数据会被序列化为 `Parcel` 对象，通过 Binder 传递到目标进程，再反序列化为 `Bundle`。
##### 1.序列化：`Bundel` ->`Parcel`
`Bundle` 实现了 `Parcelable` 接口，因此可以通过 `writeToParcel(Parcel dest, int flags)` 方法将数据序列化为 `Parcel`（二进制流）。具体过程如下：
- ​**基本类型**​（如 `int`、`String`）：直接写入 `Parcel` 的对应方法（如 `dest.writeInt()`、`dest.writeString()`）；
- ​`Parcelable` 对象**​：调用其 `writeToParcel()` 方法，将对象数据写入 `Parcel`；
- ​**Serializable 对象**​：通过反射遍历所有字段，逐个写入 `Parcel`（性能较差）。
##### 2.传输：Binder 事务缓冲区
序列化后的 `Parcel` 数据会被放入 ​**Binder 事务缓冲区**​（Binder Transaction Buffer），这是一个内核级别的共享内存区域，用于暂存 `IPC` 传输的数据。Android 对 Binder 事务缓冲区的大小有严格限制（默认约 `1MB`），超过限制会抛出 `TransactionTooLargeException`。

##### 3.反序列化：Parcel → Bundle → Intent
目标进程（如被启动的 `Activity` 所在进程）接收到 Binder 消息后，会从缓冲区中读取 `Parcel` 数据，并通过 `Bundle` 的 `createFromParcel(Parcel source)` 方法反序列化为 `Bundle` 对象。最终，目标组件（如 `Activity`）通过 `getIntent()` 获取到原始 `Intent`，并从 `mExtras` 中提取数据。

#### 三、系统调度：`ActivityManagerService`的角色
`Intent` 数据的传递离不开 Android 系统核心服务 ​`ActivityManagerService（AMS）`​**​ 的调度。以启动 `Activity` 为例，流程如下：
##### 1. 发送方调用 `startActivity(intent)`
应用进程通过 Binder 调用 `AMS` 的 `startActivity()` 方法，传递包含数据的 `Intent`。
##### 2. `AMS` 解析 Intent 并匹配目标组件
`AMS` 根据 `Intent` 的 `action`、`category`、`data` 等信息（显式 Intent 直接通过组件类名匹配），找到需要启动的目标 `Activity` 组件（可能是当前应用或其他应用的）。
##### 3. `AMS` 传递 Intent 到目标进程
如果目标 `Activity` 属于其他进程，`AMS` 会通过 Binder 通知目标进程的 `ActivityThread`（应用进程的主线程），并将 `Intent` 数据（封装在 `Bundle` 中）通过 Binder 传输过去。
##### 4. 目标进程创建组件并传递 Intent
目标进程的 `ActivityThread` 收到消息后，创建目标 `Activity` 实例，并通过 `attach()` 方法将 `Intent` 传递给它（存储在 `Activity` 的 `mIntent` 字段中）。此时，`Activity` 即可通过 `getIntent()` 获取原始 `Intent` 及其数据。

#### 四、关键原理总结
1. ​**数据载体**​：`Intent` 本身不直接存储数据，而是通过内部的 `Bundle` 对象以键值对形式存储；
2. ​**序列化与反序列化**​：`Bundle` 实现 `Parcelable` 接口，数据通过 `Parcel` 序列化后通过 Binder 传输，目标进程反序列化后还原；
3. ​**跨进程限制**​：Binder 事务缓冲区大小限制（约 `1MB`）决定了 `Intent` 传递数据的最大体积；
4. ​**系统调度**​：`AMS` 负责解析 `Intent`、匹配目标组件，并协调跨进程数据传输。

#### 扩展：为什么 `Parcelable` 比 `Serializable` 更高效？
- ​**`Parcelable`**​：Android 专为 `IPC` 设计的序列化方式，通过手动编写 `writeToParcel()` 和 `createFromParcel()` 方法，直接操作内存（如写入基本类型的二进制表示），无需反射，性能接近原生内存复制；
- ​**Serializable**​：Java 原生的序列化方案，通过反射遍历对象所有字段并生成字节流，涉及大量临时对象和类型检查，性能较差（尤其对于复杂对象）。因此，Android 推荐在需要 `IPC` 的场景（如 `Intent` 跨进程传值）中使用 `Parcelable`。