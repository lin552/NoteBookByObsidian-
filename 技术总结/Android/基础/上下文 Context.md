---
创建时间: 2025-05-07 10:46:51
作者: wangxiaoming
tags:
  - Android
  - Context
---
#### 一、定义与作用
- **本质**​：`Context` 是 Android 应用的核心抽象类，提供访问应用资源、系统服务及组件交互的接口，是连接应用代码与系统环境的桥梁
- ​**核心功能**​：
    - ​**资源访问**​：通过 `getResources()` 获取字符串、布局、图片等资源。
    - ​**组件启动**​：启动 `Activity`、`Service`、发送广播。
    - ​**系统服务**​：获取 `LayoutInflater`、`WindowManager` 等服务。
    - ​**文件操作**​：访问应用私有目录（`getFilesDir()`）和 `SharedPreferences`。
#### 二、生命周期
- ​**Application Context**​：与整个应用进程绑定，生命周期最长。
- ​**Activity Context**​：与 `Activity` 生命周期一致，`UI` 操作需依赖此类型。
- ​**Service Context**​：与 `Service` 生命周期一致，适用于后台任务。
- ​**`BroadcastReceiver Context`**​：仅在广播接收期间存在，不可用于 `UI` 操作
#### 三、Context的类型与区别
|**类型**​|​**生命周期**​|​**适用场景**​|​**限制**​|
|---|---|---|---|
|​**Application**​|应用进程全生命周期|全局资源、单例初始化、后台服务|无法弹窗、无法启动 `Activity`（需标志位）|
|​**Activity**​|与 Activity 绑定|UI 操作、弹窗、主题相关任务|静态持有易内存泄漏|
|​**Service**​|与 Service 绑定|后台任务、跨进程通信|无法直接操作 UI|
|​**BroadcastReceiver**​|广播处理期间|接收广播并触发响应|仅限短暂操作（10 秒内）|
**关键区别**​：
- ​**`UI` 操作**​：仅 `Activity Context` 支持（如 `setContentView()`、`Dialog`）。
- ​**主题与样式**​：`Activity` 的 `Context` 继承自 `ContextThemeWrapper`，支持主题；`Application` 使用系统默认主题
- ​**启动 Activity**​：非 `Activity Context` 需添加 `FLAG_ACTIVITY_NEW_TASK` 标志
#### 四、内存泄漏与注意事项
#### 1）常见泄漏场景
- ​**静态持有 Context**​：如静态变量保存 `Activity Context`，导致无法回收。
```java
public class Singleton {
    private static Context sContext; //内存泄漏风险
    public Singleton(Context context) { sContext = context; }
}
```
- **非静态内部类**​：如 `AsyncTask`、`Handler` 隐式持有外部 `Activity` 引用。
##### 2）解决方案
- **使用 Application Context**​：全局对象优先选择 `getApplicationContext()`。
- ​**弱引用（`WeakReference`）​**​：持有 `Context` 时使用弱引用避免强引用循环。
```kotlin
private static class MyHolder {
    private static WeakReference<Context> sContextRef;
}
```
- **生命周期绑定**​：如 `ViewModel`、`LifecycleObserver` 自动管理引用。
#### 3）最佳实践
- ​**`UI` 操作必用 Activity Context**​：弹窗、主题相关任务需绑定 Activity。
- ​**避免跨组件传递 Context**​：减少生命周期耦合。
- ​**慎用 `getApplicationContext()`**​：`UI` 操作可能引发异常（如 `setContentView()`）。
#### 五、Context的底层机制
##### 1）继承关系
```markdown
Context (抽象类)
├─ ContextWrapper (包装类)
│  ├─ ContextThemeWrapper (支持主题)
│  └─ Application
│     └─ Activity
│     └─ Service
└─ ContextImpl (实际实现类)
```
- **`ContextImpl`**​：真正实现资源加载、服务代理等功能
- ​**`ContextWrapper`**​：通过 `attachBaseContext()` 关联实际 `ContextImpl`。
##### ​2）创建流程​
1. ​**应用启动**​：`ActivityThread` 创建 `Application` 并初始化 `ContextImpl`。
2. ​**Activity/Service 启动**​：创建对应的 `ContextImpl` 并绑定到组件。
#### 六、面试高频问题
1. ​**为何 `Application Context` 不能弹窗？​**​
    - 因无任务栈支持，需添加 `FLAG_ACTIVITY_NEW_TASK` 标志。
2. ​**`Context` 的内存泄漏如何避免？​**​
    - 避免静态持有、使用弱引用、优先 `Application Context`。
3. ​**`Service` 能否更新 `UI`？​**​
    - 不能，需通过 `Broadcast` 或 `LiveData` 通知 `Activity`。
4. ​**`Context` 的实现类有哪些？​**​
    - `Application`、`Activity`、`Service`、`BroadcastReceiver`。