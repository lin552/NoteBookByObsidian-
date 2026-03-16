---
创建时间: "2026-03-16 19:29:21"
作者: wangxiaoming
tags:
---
#### 一、`MediaController`是什么?

**`MediaController`是一个客户端组件，用于**连接并控制一个已经存在的 `MediaSession`**。
它本身不播放媒体，也不管理播放逻辑，它的作用是：
- 向 `MediaSession` 发送控制命令（播放、暂停、切歌、拖动进度条等）
- 监听 `MediaSession` 的状态变化（播放状态、元数据、队列变化等）
- 在 UI 层（Activity/Fragment）或系统组件（通知栏、锁屏）中实现对播放器的远程控制
可以把 `MediaController` 想象成一个“**万能遥控器**”，你拿着它就能控制远处的 MediaSession（运行在 Service 中）。

#### 二、核心作用

| 作用                  | 说明                                                 |
| ------------------- | -------------------------------------------------- |
| 发送控制命令              | ​如 `play()`、`pause()`、`skipToNext()`、`seekTo()`等   |
| 监听状态变化              | ​实时获取播放状态、当前歌曲信息、进度等，用于更新 UI                       |
| 桥接 `UI` 与 `Service` | ​让 `Activity` 能够安全地控制后台播放器                         |
| 跨进程控制               | 即使 `MediaSession` 运行在另一个进程，只要有 `SessionToken`，就能控制 |
#### 三、创建方式
`MediaController` 必须通过 `MediaSession` 的 `SessionToken` 来构造：
```java
MediaController controller = new MediaController(context, sessionToken);
```
这个 `sessionToken`通常由服务端 `MediaSession` 通过 `getSessionToken()`获取，并通过 `Intent/Binder/AIDL` 等方式传递给客户端。
#### 四、核心功能详解
##### 1) 发送控制命令
控制命令通过 `MediaController.getTransportControls()`获取的对象发送：
```java
MediaController.TransportControls controls = controller.getTransportControls();

controls.play();
controls.pause();
controls.skipToNext();
controls.seekTo(30000); // 跳转到 30 秒
controls.setShuffleMode(PlaybackState.SHUFFLE_MODE_ALL);
```
- 这些命令是**异步**发送给 `MediaSession` 的，执行结果需要通过 Callback 监听。
- 不要在 UI 线程直接执行耗时操作。
##### 2）监听`MediaSession`状态变化
通过注册 `MediaController.Callback`来接收服务端状态变化：
```java
controller.registerCallback(new MediaController.Callback() {
    @Override
    public void onPlaybackStateChanged(PlaybackState state) {
        // 更新播放/暂停按钮图标
        int currentState = state.getState();
        if (currentState == PlaybackState.STATE_PLAYING) {
            btnPlay.setImageResource(R.drawable.ic_pause);
        } else {
            btnPlay.setImageResource(R.drawable.ic_play);
        }
    }

    @Override
    public void onMetadataChanged(MediaMetadata metadata) {
        // 更新歌曲标题、艺术家、封面
        tvTitle.setText(metadata.getString(MediaMetadata.METADATA_KEY_TITLE));
        tvArtist.setText(metadata.getString(MediaMetadata.METADATA_KEY_ARTIST));
        ivCover.setImageBitmap(metadata.getBitmap(MediaMetadata.METADATA_KEY_ALBUM_ART));
    }

    @Override
    public void onQueueChanged(List<MediaSession.QueueItem> queue) {
        // 更新播放列表 UI
    }
});
```
- 在 `onDestroy()`或 `onStop()`中一定要调用 `controller.unregisterCallback()`防止内存泄漏。
- 当 `MediaSession` 被释放或断开连接时，`Callback` 的 `onSessionDestroyed()`会被调用。
#### 五、与UI结合的典型场景
##### 1）场景1：Actvity控制后台播放器
```java
public class PlayerActivity extends AppCompatActivity {
    private MediaController mediaController;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_player);

        // 假设从 Intent 中获取了 SessionToken
        MediaSession.Token token = getIntent().getParcelableExtra("session_token");
        mediaController = new MediaController(this, token);

        // 设置控制按钮点击事件
        findViewById(R.id.btn_play).setOnClickListener(v -> {
            mediaController.getTransportControls().play();
        });

        findViewById(R.id.btn_pause).setOnClickListener(v -> {
            mediaController.getTransportControls().pause();
        });

        // 注册回调更新 UI
        mediaController.registerCallback(new MediaController.Callback() {
            @Override
            public void onPlaybackStateChanged(PlaybackState state) {
                // 更新 UI
            }
        });
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mediaController.unregisterCallback(callback);
    }
}
```
##### 2）场景2：通知栏控制（`Notification.MediaStyle`）
```java
MediaController controller = new MediaController(context, sessionToken);
MediaController.TransportControls controls = controller.getTransportControls();

Notification notification = new NotificationCompat.Builder(context, CHANNEL_ID)
    .setContentTitle("Song Title")
    .setSmallIcon(R.drawable.ic_music_note)
    .setLargeIcon(albumArtBitmap)
    .setStyle(new androidx.media.app.NotificationCompat.MediaStyle()
        .setMediaSession(sessionToken))
    .addAction(R.drawable.ic_prev, "Prev", controls.getActionIntent(MediaSession.ACTION_FLAG_PREVIOUS))
    .addAction(R.drawable.ic_play, "Play", controls.getActionIntent(MediaSession.ACTION_FLAG_PLAY))
    .addAction(R.drawable.ic_next, "Next", controls.getActionIntent(MediaSession.ACTION_FLAG_NEXT))
    .build();

startForeground(NOTIFICATION_ID, notification);
```
#### 六、生命周期管理

| 阶段   | 操作                                                              |
| ---- | --------------------------------------------------------------- |
| 创建   | 在获取到 `SessionToken` 后，用 `new MediaController(context, token)`创建 |
| 注册回调 | 调用 `registerCallback()`监听状态变化                                   |
| 使用   | 调用 `getTransportControls()`发送命令                                 |
| 销毁   | 调用 `unregisterCallback()`，并将引用置空                                |

❗ 如果不注销 Callback，可能会导致：
- 内存泄漏
- 崩溃（因为 `MediaController` 持有 Context 引用）
#### 七、与`MediaBrowser`的区别

| 特性      | `MediaController` | `MediaBrowser`           |
| ------- | ----------------- | ------------------------ |
| 主要用途    | 控制播放、监听状态         | 浏览媒体内容（歌单、专辑、歌曲）         |
| 是否可控制播放 | ✅ 是               | ❌ 否（需配合 MediaController） |
| 是否可浏览内容 | ❌ 否               | ✅ 是                      |
| 使用场景    | 通知栏、锁屏、UI 控制      | 音乐库浏览、`Android Auto`     |
