---
创建时间: 2025-05-24 13:40:55
作者: wangxiaoming
tags:
  - 四大组件
  - BroadcastReceiver
---
#### 一、基础概念
1. ​`BroadcastReceiver`是什么？有什么作用？​**​
    - ​**回答**​：`BroadcastReceiver`是`Android`四大组件之一，用于**接收系统或应用发送的全局广播事件**​（如电量变化、网络状态变化、自定义事件）。它支持组件间通信，甚至跨进程通信。
    - ​**关键点**​：
        - 用途：监听系统事件（如开机完成）、应用内事件通知（如登录成功）。
        - 分类：静态注册（清单文件）和动态注册（代码中）。
2. ​**静态注册和动态注册的区别？​**​
    - ​**回答**​：
        - ​**静态注册**​：在`AndroidManifest.xml`中声明，应用未启动时也可接收广播（如开机广播）。但Android 8.0+限制隐式广播的静态注册。
        - ​**动态注册**​：通过`registerReceiver()`注册，需手动解绑（`unregisterReceiver()`），灵活性高，无启动限制。
```java
// 动态注册
IntentFilter filter = new IntentFilter(ConnectivityManager.ACTION_NETWORK_STATE_CHANGED);
registerReceiver(networkReceiver, filter);
```
#### 二、广播类型与生命周期
3. ​**普通广播和有序广播的区别？​**​
    - ​**回答**​：
        - ​**普通广播**​：完全异步，所有接收者同时收到，无法终止。
        - ​**有序广播**​：按优先级顺序同步传递，接收者可通过`abortBroadcast()`终止广播。
    - ​**代码示例**​：
```xml
<!-- 有序广播优先级在清单中声明 -->
<receiver android:name=".MyReceiver">
    <intent-filter android:priority="100">
        <action android:name="com.example.MY_ACTION" />
    </intent-filter>
</receiver>
```
4. ​`BroadcastReceiver`的生命周期是怎样的？​**​
    - ​**回答**​：生命周期仅存在于`onReceive()`方法的执行期间。若在此方法中启动`Activity`或`Service`，需注意可能引发`ANR`（主线程阻塞）。
    - ​**建议**​：
```java
@Override
public void onReceive(Context context, Intent intent) {
    goAsync(); // 延迟处理
    // 异步任务完成后调用result.finish()
}
```
#### 三、权限与安全性
5. ​**如何控制广播的发送和接收权限？​**​
    - ​**回答**​：
        - ​**发送方**​：通过`sendBroadcast(intent, permission)`指定接收方需声明的权限。
        - ​**接收方**​：在清单文件中声明`android:permission`属性。
    - ​**示例**​：
```xml
<!-- 发送方需权限 -->
<uses-permission android:name="android.permission.SEND_SMS" />

<!-- 接收方需权限 -->
<receiver
    android:name=".SmsReceiver"
    android:permission="android.permission.RECEIVE_SMS" />
```
5. ​**如何防止恶意应用监听敏感广播？​**​
    - ​**回答**​：
        - 使用显式Intent（指定包名或`ComponentName`）。
        - 对敏感数据加密，避免通过Intent明文传输。
#### 四、应用场景与最佳实践
7. **常见的使用场景有哪些？​**​
    - ​**回答**​：
        - 监听系统事件（电池、网络、屏幕状态）。
        - 应用内通信（如Activity通知Service停止）。
        - 推送消息的补充（结合本地广播）。
8. ​`LocalBroadcastManager`的作用是什么？​**​
    - ​**回答**​：用于**应用内广播**，避免全局广播的安全性和性能开销。仅限当前应用内组件通信，自动线程切换。
    - ​**示例**​：
```java
LocalBroadcastManager.getInstance(context).sendBroadcast(intent);
```
#### 五、高频陷阱题
9. ​**静态注册的Receiver在Android 8.0+无法接收隐式广播，如何解决？​**​
    - ​**回答**​：
        - 改用显式Intent（指定`ComponentName`）。
        - 使用`JobScheduler`或`WorkManager`替代后台任务。
        - 对于系统事件（如开机），仍需静态注册。
10. ​**在`onReceive()`中启动Activity为何可能导致`ANR`？​**​
    - ​**回答**​：`onReceive()`运行在主线程，若直接启动Activity，可能阻塞后续广播处理。
    - ​**解决方案**​：使用`Intent.FLAG_ACTIVITY_NEW_TASK`并异步处理。
#### 六、进阶问题
11. ​**粘性广播（Sticky Broadcast）为何被废弃？替代方案是什么？​**​
    - ​**回答**​：废弃因安全性和线程问题。替代方案：
        - 使用`LocalBroadcastManager`。
        - 通过`LiveData`或`EventBus`实现组件间通信。
12. ​**如何保证有序广播的优先级？​**​
    
    - ​**回答**​：在清单文件中为接收器设置`android:priority`属性，值越大优先级越高。