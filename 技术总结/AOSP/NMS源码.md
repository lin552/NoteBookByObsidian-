---
创建时间: "2025-08-13 18:00:08"
作者: wangxiaoming
tags:
---
#### 一、消息接收与分发流程
##### 1.系统广播接收（`BroadcastReceiver`）
- ​**触发点**​：系统接收到厂商通道（如 `FCM`、华为 Push）下发的广播。
- **关键类**​：
	- `com.android.server.NotificationManagerService`（`NMS`）：系统级通知服务。
	- `com.android.server.SystemService`：系统服务基类。
- ​**源码路径**​：`frameworks/base/services/core/java/com/android/server/notification/NotificationManagerService.java`

##### 2.Binder 通知处理
- ​**流程**​：
	- 厂商通道通过 Binder 调用 `NMS` 的 `enqueueNotificationWithTag()`方法。
	- `NMS` 通过 Binder 驱动将消息传递至系统服务进程（`system_server`）。
- ​**关键代码**​：
```java
// frameworks/base/core/java/android/app/INotificationManager.aidl
interface INotificationManager {
    void enqueueNotificationWithTag(String pkg, String opPkg, String tag, int id,
                                    in Notification notification, int userId);
}
```
- 调用链：厂商 `SDK` → Binder 驱动 → `NMS` 的 Binder 线程池处理。

#### 二、消息处理与优先级判定
##### 3. ​**消息合法性校验**​
- ​**关键步骤**​：
	- ​**权限检查**​：验证应用是否具备 `POST_NOTIFICATIONS`权限（Android 13+）。
	- ​**渠道有效性**​：检查通知渠道（`NotificationChannel`）是否被用户禁用。
 - **资源校验**​：验证图标、声音等资源是否合法（如防止内存泄漏）。
 - 源码位置**​：
    `NotificationManagerService.enqueueNotificationInternal()`
    
##### 4. ​**进程优先级调整**​
- **策略**​：
	- **前台服务关联**​：若消息来自前台服务（`startForeground()`），`NMS` 提升进程优先级至 `OOM_ADJ=0`。
	- ​**实时性标记**​：通过 `Notification.FLAG_INSISTENT`或 `setPriority()`提升优先级。
- ​**源码实现**​：
```java
// frameworks/base/services/core/java/com/android/server/notification/NotificationManagerService.java
private void enqueueNotificationInternal(...) {
    // 提升前台服务进程优先级
    if (isAppForeground) {
        mAm.setProcessImportant(r.pkg, r.uid, true);
    }
}
```

#### 三、通知构建与状态栏更新
##### 5.通知对象构造
- 关键类
	- `StatusBarNotification`：封装通知元数据（包名、图标、优先级等）。
	- `NotificationRecord`：记录通知状态（如超时时间、分组信息）。
- 源码路径：
`frameworks/base/core/java/android/service/notification/StatusBarNotification.java`

##### 6.状态栏`UI`更新
- 流程
	1. `NMS` 通过 Binder 通知 `SystemUI` 进程（`com.android.systemui`）。
	2. `SystemUI` 的 `NotificationListener`接收消息并触发 `UI` 渲染。
- 关键代码：
```java
// frameworks/base/packages/SystemUI/src/com/android/systemui/statusbar/NotificationListener.java
@Override
public void onNotificationPosted(StatusBarNotification sbn, RankingMap rankingMap) {
    mMainHandler.post(() -> {
        // 触发通知栏更新
        mNotificationPanelViewController.updateNotifications();
    });
}
```

#### 四、多线程与锁屏场景优化
##### 7.跨进程通信机制
- Binder优化：
	- 共享内存：通过 `ParcelFileDescriptor`传递大负载数据（如图片）。
	- 异步队列：`NMS`使用`HandlerThread`处理高并发通知请求，避免主线程阻塞。
- 源码位置：
`frameworks/base/services/core/java/com/android/server/notification/NotificationManagerService.java`

##### 8.锁屏通知处理
- 策略：
	- **预览模式**​：根据 `Notification.visibility`控制锁屏内容可见性。
	- **紧急通知**​：通过 `setFullScreenIntent()`触发唤醒屏幕。
- 源码实现：
```java
// frameworks/base/core/java/android/app/Notification.java
public void setFullScreenIntent(PendingIntent intent, boolean highPriority) {
    mFullScreenIntent = intent;
    mFlags |= highPriority ? FLAG_HIGH_PRIORITY : 0;
}
```

#### 五、核心源码模块关系
|**模块**​|​**关键类/接口**​|​**职责**​|
|---|---|---|
|​**消息接收层**​|`INotificationManager.aidl`|Binder 接口定义|
|​**服务层**​|`NotificationManagerService`|通知入队、优先级管理、进程调度|
|​**UI 层**​|`StatusBarController`|状态栏布局构建、动画控制|
|​**厂商扩展层**​|`VendorPushService`（如华为 `PushKit`）|定制化推送协议实现|
#### 六、调试与优化建立
1. ​**日志跟踪**​：
	 - 使用 `adb logcat -b all`查看 `NMS` 日志（关键字 `NotificationService`）。
	 - 监控 Binder 事务：`adb shell dumpsys activity binder_transactions`。
2. ​**性能优化**​：
    - **批量处理**​：合并高频通知（`NotificationManager.notify()`的 `tag`参数）。
    - ​**异步加载**​：图片资源使用 `Glide`或 `Picasso`异步解码。
3. **兼容性适配**​：
    - **Android 13+​**​：动态申请 `POST_NOTIFICATIONS`权限。
    - ​**厂商定制**​：针对华为/小米等定制 ROM 调用其专属 API（如 `HMS Core`）。