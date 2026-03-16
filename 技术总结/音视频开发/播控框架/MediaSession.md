---
创建时间: "2026-03-16 19:29:33"
作者: wangxiaoming
tags:
---
#### 一、`MediaSession`是什么？
`MediaSession`​ 是 Android 多媒体框架中的一个核心组件（API 级别 21 引入），用于表示一个**媒体播放会话**。

它运行在你自己的播放器 Service 中，负责管理播放状态、元数据、接收并处理外部控制命令（播放、暂停、切歌等），并与系统 UI（锁屏、通知栏）、蓝牙设备、车载系统等进行交互。

可以把 `MediaSession` 想象成一个“**媒体控制中心**”，外界所有的播放控制请求都会经过它，再由你的播放器逻辑去执行。

#### 二、核心作用
- **统一管理媒体控制命令**​
    接收来自通知栏、锁屏、耳机按键、`Google Assistant`、`Wear OS` 等触发的控制事件。
- **维护播放状态**​
    通过 `PlaybackState`告诉外界当前是播放、暂停、缓冲还是停止，以及当前进度、速度等。
- **提供媒体元数据**​
    通过 `MediaMetadata`告诉外界当前播放的是什么歌（标题、歌手、封面等）。
- **支持后台播放与跨进程控制**​
    配合前台服务和 SessionToken，可以让其他 App 或系统组件远程控制你的播放器。
#### 三、核心组成部分
##### 1）`MediaSession对象本身`
- 通过 `new MediaSession(Context context, String tag)`创建。
- 必须调用 `setActive(true)`才能接收控制命令。
- 不再使用时必须调用 `release()`释放资源。
```java
mediaSession = new MediaSession(this, "MyMusicApp");
mediaSession.setActive(true);
```
##### 2) `MediaSession.Callback`
- 这是一个抽象类，你必须继承并重写感兴趣的方法来处理控制命令。
- 常见回调方法：

| 方法名                                            | 触发场景     |
| ---------------------------------------------- | -------- |
| `onPlay()`                                     | 用户按下播放按钮 |
| `onPause()`                                    | 用户按下暂停按钮 |
| `onStop()`                                     | 用户按下停止按钮 |
| `onSkipToNext()`                               | 下一曲      |
| `onSkipToPrevious()`                           | 上一曲      |
| `onSeekTo(long pos)`                           | 拖动进度条    |
| `onCustomAction(String action, Bundle extras)` | 自定义命令    |
```java
mediaSession.setCallback(new MediaSession.Callback() {
    @Override
    public void onPlay() {
        // 启动播放逻辑
        player.play();
        updatePlaybackState(PlaybackState.STATE_PLAYING);
    }

    @Override
    public void onPause() {
        player.pause();
        updatePlaybackState(PlaybackState.STATE_PAUSED);
    }
});
```
##### 3) `PlybackState(播放状态)`
描述当前播放器的状态，包括：
- 播放状态：`STATE_NONE`、`STATE_STOPPED`、`STATE_PLAYING`、`STATE_PAUSED`、`STATE_BUFFERING`
- 当前播放位置（`getCurrentPosition()`）
- 缓冲进度
- 支持的播放控制动作（`actions`位掩码）
```java
PlaybackState state = new PlaybackState.Builder()
    .setState(PlaybackState.STATE_PLAYING, currentPos, 1.0f)
    .setActions(
        PlaybackState.ACTION_PLAY |
        PlaybackState.ACTION_PAUSE |
        PlaybackState.ACTION_SKIP_TO_NEXT |
        PlaybackState.ACTION_SEEK_TO
    )
    .build();

mediaSession.setPlaybackState(state);
```
##### 4)`MediaMetadata`(媒体元数据)
存储当前播放媒体的描述信息，用于显示在锁屏、通知栏、蓝牙设备上。
常用键值：
- METADATA_KEY_TITLE
- METADATA_KEY_ARTIST
- METADATA_KEY_ALBUM
- METADATA_KEY_DURATION
- METADATA_KEY_ALBUM_ART（`Bitmap`）
```java
MediaMetadata metadata = new MediaMetadata.Builder()
    .putString(MediaMetadata.METADATA_KEY_TITLE, "晴天")
    .putString(MediaMetadata.METADATA_KEY_ARTIST, "周杰伦")
    .putBitmap(MediaMetadata.METADATA_KEY_ALBUM_ART, coverBitmap)
    .build();

mediaSession.setMetadata(metadata);
```
##### 5)`SessionToken`
- `MediaSession` 的唯一标识符。
- 外部组件（如 MediaController、通知栏）通过这个 Token 连接到你的 `MediaSession`。
```java
MediaSession.Token token = mediaSession.getSessionToken();
```
#### 四、生命周期与资源管理
**创建与销毁**
```java
@Override
public void onCreate() {
    mediaSession = new MediaSession(this, "PlayerService");
    mediaSession.setCallback(callback);
    mediaSession.setActive(true);
}

@Override
public void onDestroy() {
    mediaSession.release(); // 必须调用！
    mediaSession = null;
}
```
**激活与停用**
- `setActive(true)`：激活会话，开始接收控制命令。
- `setActive(false)`：停用会话，节省资源，但仍保留状态。
#### 五、与播放器的绑定
`MediaSession` 本身不播放音频，它只是一个控制中枢。你需要把它和实际的播放器（如 `MediaPlayer`、`ExoPlayer`）连接起来。
示例思路：
- 在 `onPlay()`中调用 `exoPlayer.play()`
- 在 `onPause()`中调用 `exoPlayer.pause()`
- 在 `onSeekTo()`中调用 `exoPlayer.seekTo(pos)`
- 定时更新 `PlaybackState`中的位置信息

#### 六、与系统组件的协作

| 系统组件                     | 如何与 `MediaSession` 协作                          |
| ------------------------ | ---------------------------------------------- |
| 通知栏                      | 使用 `Notification.MediaStyle`，绑定 `SessionToken` |
| 锁屏界面                     | 自动读取 `MediaSession` 的元数据与控制按钮                  |
| 蓝牙耳机                     | 耳机按键事件会通过 `MediaSession.Callback` 传递           |
| `Google Assistant`       | 语音命令转化为 MediaSession 控制命令                      |
| `Android Auto / Wear OS` | 通过 `MediaBrowser + MediaController` 连接         |
