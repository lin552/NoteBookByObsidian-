---
创建时间: "2026-03-16 19:29:08"
作者: wangxiaoming
tags:
---
#### 一、什么是`MediaBrowser`
**`MediaBrowser` 是一个客户端组件，用于**连接到一个 `MediaBrowserService` 并浏览其提供的媒体内容层级结构**。
它不仅能获取歌曲列表，还能浏览专辑、歌单、文件夹等层次化内容，非常适合做音乐 App 的“库”功能。
与 `MediaController` 不同，`MediaBrowser` **不能直接控制播放**，它只负责“看内容”。
要播放选中的内容，需要配合 `MediaController` 使用。
可以把 `MediaBrowser` 想象成一个“**文件管理器**”，专门用来查看媒体库的结构，而 `MediaController` 才是“**播放器遥控器**”。
#### 二、核心作用

| 作用                       | 说明                              |
| ------------------------ | ------------------------------- |
| 连接 `MediaBrowserService` | 通过服务组件名建立长连接                    |
| 浏览内容数                    | 获取子目录、歌曲列表、播放列表等                |
| 订阅内容变化                   | 当服务端内容更新时自动收到通知                 |
| 与 `MediaController` 配合播放 | ​选中歌曲后，用 `MediaController` 发起播放 |
#### 三、依赖的服务端：`MediaBrowserService`
`MediaBrowser` 只能连接实现了 **`MediaBrowserService`​ 的服务端。

服务端需要提供：
- 媒体内容的层级结构（通过 `onLoadChildren()`返回）
- 媒体元数据（通过 `MediaMetadata`）
- 可选的 MediaSession（用于播放控制）
```java
public class MusicService extends MediaBrowserService {
    private MediaSession mediaSession;

    @Override
    public BrowserRoot onGetRoot(String clientPackageName, int clientUid, Bundle rootHints) {
        // 返回根目录标识，如果允许连接则返回非空 BrowserRoot
        return new BrowserRoot("root", null);
    }

    @Override
    public void onLoadChildren(String parentId, Result<List<MediaBrowser.MediaItem>> result) {
        // 根据 parentId 加载子项（歌曲或子目录）
        List<MediaBrowser.MediaItem> items = new ArrayList<>();
        // ... 添加 MediaItem ...
        result.sendResult(items);
    }
}
```

#### 四、创建与连接 `MediaBrowser`
```java
MediaBrowser browser = new MediaBrowser(
    context,
    new ComponentName(context, MusicService.class), // 服务端 Service
    connectionCallback,  // 连接回调
    null                 // 可选：Bundle 参数
);

browser.connect(); // 发起连接
```

#### 五、核心功能详解
##### 1）连接回调`ConnectionCallback`
连接过程是异步的，结果通过回调返回：
```java
private final MediaBrowser.ConnectionCallback connectionCallback = new MediaBrowser.ConnectionCallback() {
    @Override
    public void onConnected() {
        // 连接成功！可以获取 SessionToken 并创建 MediaController
        MediaSession.Token token = browser.getSessionToken();
        mediaController = new MediaController(context, token);

        // 订阅根目录的内容
        browser.subscribe(browser.getRoot(), subscriptionCallback);
    }

    @Override
    public void onConnectionFailed() {
        // 连接失败，可能是 Service 未启动或权限不足
    }

    @Override
    public void onConnectionSuspended() {
        // 连接中断（如服务端被杀），需要重新连接
    }
};
```
##### 2) 订阅内容 `SubscriptionCallback`
订阅某个目录后，服务端会通过 `onLoadChildren()`返回该目录下的内容，客户端通过回调接收：
```java
private final MediaBrowser.SubscriptionCallback subscriptionCallback = new MediaBrowser.SubscriptionCallback() {
    @Override
    public void onChildrenLoaded(@NonNull String parentId, List<MediaBrowser.MediaItem> children) {
        // 在这里更新 UI，显示子项列表
        for (MediaBrowser.MediaItem item : children) {
            if (item.isBrowsable()) {
                // 是目录，可继续深入
            } else if (item.isPlayable()) {
                // 是可播放的歌曲
            }
        }
    }

    @Override
    public void onError(String parentId) {
        // 加载内容失败
    }
};
```

##### 3)获取媒体项目 `MediaItem`
MediaItem表示媒体库中的一个项目，可以是：
- **可浏览的（browsable）**：如“我的歌单”、“周杰伦专辑”
- **可播放的（playable）**：如具体的歌曲
```java
for (MediaBrowser.MediaItem item : children) {
    String title = item.getDescription().getTitle().toString();
    Bitmap icon = item.getDescription().getIconBitmap();
    boolean isPlayable = item.isPlayable();
    boolean isBrowsable = item.isBrowsable();
}
```

##### 4)与`MediaController`配合播放
用户点击某首歌后，用 `MediaController` 播放：
```java
// 假设 clickedItem 是用户点击的 MediaItem
String mediaId = clickedItem.getMediaId();
mediaController.getTransportControls().playFromMediaId(mediaId, null);
```
服务端需要在 `MediaSession.Callback`中实现 `onPlayFromMediaId()`来根据 ID 找到并播放对应歌曲。

#### 六、生命周期管理

| 阶段  | 操作                                                     |
| --- | ------------------------------------------------------ |
| 创建  | `new MediaBrowser(...)`                                |
| 连接  | `browser.connect()`                                    |
| 使用  | 在 `onConnected()`中订阅内容、获取 `SessionToken`               |
| 断开  | `browser.disconnect()`（通常在 `onStop()`或 `onDestroy()`中） |
| 释放  | 断开后不再使用，置空引用                                           |
⚠️ 注意：
- 必须在 `onDestroy()`中调用 `disconnect()`，否则可能导致内存泄漏。
- 当 Activity 进入后台时，可以选择断开连接以节省资源。
#### 七、典型使用场景

| 场景                    | 说明                                  |
| --------------------- | ----------------------------------- |
| 音乐 App 的主界面           | 左侧是分类（专辑、歌手），右侧是歌曲列表                |
| `Android Auto` / 车载系统 | 车机界面通过 `MediaBrowser` 浏览手机 App 的音乐库 |
| 第三方播放器 `UI`           | 通用播放器界面可接入任意 `MediaBrowserService`  |
| 智能家居设备                | 音箱等设备通过 `MediaBrowser` 浏览并播放音乐      |
