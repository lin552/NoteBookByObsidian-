---
创建时间: "2026-02-21 17:51:06"
作者: wangxiaoming
tags:
---
#### 一、新旧焦点获取对比

| 类别           | 旧API（已废弃）                                                                                                     | 新API（推荐）                                                                 |
| ------------ | ------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| 请求焦点方法       | `AudioManager.requestAudioFocus(AudioManager.OnAudioFocusChangeListener l, int streamType, int durationHint)` | `AudioManager.requestAudioFocus(AudioFocusRequest request)`              |
| 焦点请求参数封装     | 分散：`listener`、`streamType`、`durationHint`                                                                     | 集中：`AudioFocusRequest`对象（Builder 模式）                                     |
| 音频属性绑定       | 使用 `streamType`（如 `STREAM_MUSIC`、`STREAM_ALARM`）                                                              | 使用 `AudioAttributes`（`.setAudioAttributes(...)`）                         |
| 焦点变化监听       | 直接传 `OnAudioFocusChangeListener`到方法参数                                                                         | 在 `AudioFocusRequest.Builder`中通过 `.setOnAudioFocusChangeListener(...)`设置 |
| 延迟获取焦点支持     | ❌ 不支持                                                                                                         | ✅ `.setAcceptsDelayedFocusGain(true)`                                    |
| Ducking 行为控制 | 无直接控制，由系统决定                                                                                                   | ✅ `.setWillPauseWhenDucked(true/false)`可配置                               |
| 释放焦点方法       | AudioManager.abandonAudioFocus(OnAudioFocusChangeListener l)                                                  | AudioManager.abandonAudioFocusRequest(AudioFocusRequest request)         |
| 适用最低版本       | 所有版本（但已废弃）                                                                                                    | API 26+（Android 8.0 Oreo）                                                |
| 示例场景         | 老项目兼容                                                                                                         | 新项目或升级项目推荐                                                               |

#### 二、新API参数( `AudioFocusRequest` )


| 方法参数                              | 作用                     | 常见取值/说明                              |
| --------------------------------- | ---------------------- | ------------------------------------ |
| new Builder(`focusGain`)          | 设置初始焦点类型               | AUDIOFOCUS_GAIN/ TRANSIENT/ MAY_DUCK |
| setAudioAttributes()              | 定义音频用途和类型              | USAGE_MEDIA, CONTENT_TYPE_MUSIC等     |
| setOnAudioFocusChangeListener()   | 焦点变化回调                 | 实现 `onAudioFocusChange(focusChange)` |
| setAcceptsDelayedFocusGain()      | 是否接受延迟授予焦点             | true/ false                          |
| setWillPauseWhenDucked()          | ducking 时是否暂停          | true/ false                          |
| build()                           | 生成 `AudioFocusRequest` |                                      |
| requestAudioFocus(request)        | 请求焦点                   | 返回 `GRANTED`或 `FAILED`               |
| abandonAudioFocusRequest(request) | 释放焦点                   |                                      |

##### 1）`AudioFocusRequest.Builder构造方法`
```java
new AudioFocusRequest.Builder(int focusGain)
```
- `focusGain`
表示你**初始请求的音频焦点类型**，取值来自 `AudioManager`的常量：
	- `AUDIOFOCUS_GAIN`：长时间占用焦点，适合音乐/视频播放。
	- `AUDIOFOCUS_GAIN_TRANSIENT`：短暂占用，适合导航语音、通知。
	- `AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK`：短暂占用，可允许其他音频降低音量（ducking）而不是完全停止。
##### 2)`.setAudioAttributes(AudioAttributes attributes)
```java
.setAudioAttributes(new AudioAttributes.Builder()
    .setUsage(AudioAttributes.USAGE_MEDIA)
    .setContentType(AudioAttributes.CONTENT_TYPE_MUSIC)
    .build())
```
- `AudioAttributes​` 是描述音频用途和类型的对象，比旧 API 的 streamType更精确
- `setUsage(...)​` 表示音频的使用场景，例如：
	- USAGE_MEDIA：音乐、电影。
	- USAGE_VOICE_COMMUNICATION：语音通话。
	- USAGE_ALARM：闹钟。
	- USAGE_ASSISTANT：语音助手。
- `setContentType(...)​` 表示内容的类型，例如：
	- CONTENT_TYPE_MUSIC：音乐。
	- CONTENT_TYPE_SPEECH：演讲/语音。
	- CONTENT_TYPE_MOVIE：电影。
	- CONTENT_TYPE_SONIFICATION：提示音/音效。
##### 3)`.setOnAudioFocusChangeListener(OnAudioFocusChangeListener listener)`
```java
.setOnAudioFocusChangeListener(afChangeListener)
```
- listener​ 是一个回调接口，当系统改变你的音频焦点时会被调用，参数是 focusChange值，如：
	- AUDIOFOCUS_GAIN
	- AUDIOFOCUS_LOSS
	- AUDIOFOCUS_LOSS_TRANSIENT
	- AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK
##### 4）`.setAcceptsDelayedFocusGain(boolean accepts)`
```java
.setAcceptsDelayedFocusGain(true)
```
- **意义**：如果你的焦点请求不能被立即授予（比如当前有其他高优先级音频），是否允许系统在将来授予焦点。
- **`true`**：系统可以在稍后调用 `onAudioFocusChange(AUDIOFOCUS_GAIN)`给你焦点。
- **`false`**：如果当前无法获得焦点，就直接返回 `AUDIOFOCUS_REQUEST_FAILED`。
##### 5)`.setWillPauseWhenDucked(boolean willPause)`
```java
.setWillPauseWhenDucked(true)
```
- **意义**：当进入 `AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK`状态时，你是否选择**暂停播放**而不是降低音量。
- **`true`**：ducking 时暂停。
- **`false`**：ducking 时降低音量继续播放。
##### 6)`.build`
- 完成所有配置，生成 `AudioFocusRequest`对象，用于 `requestAudioFocus()`。
##### 7) `AudioManager.requestAudioFocus(AudioFocusRequest request)`
```java
int result = audioManager.requestAudioFocus(focusRequest);
```
- request：前面 Builder构建的 AudioFocusRequest对象。
- 返回值：
	- AUDIOFOCUS_REQUEST_GRANTED：成功获得焦点。
	- AUDIOFOCUS_REQUEST_FAILED：未获得焦点。
##### 8)`AudioManager.abandonAudioFocusRequest(AudioFocusRequest request)`
- request：之前请求焦点的同一个 AudioFocusRequest对象。
- 作用：告诉系统你不再需要焦点，可以释放相关资源，并允许其他应用获取焦点。
#### 二、旧API核心方法
##### 1）请求焦点
```java
int requestAudioFocus(AudioManager.OnAudioFocusChangeListener listener, 
                      int streamType, 
                      int durationHint)
```
###### 参数意义

| 参数           | 类型                           | 作用                                                                                                                                                                                     |
| ------------ | ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| listener     | `OnAudioFocusChangeListener` | 焦点变化时的回调，当系统改变你的焦点状态时会触发 `onAudioFocusChange(int focusChange)`。                                                                                                                        |
| streamType   | <br>int                      | 音频流类型，用来告诉系统**你打算用哪一类音频流**播放，这会影响音量控制和音频路由（扬声器/耳机/蓝牙等）。常用值：`STREAM_MUSIC`、`STREAM_RING`、`STREAM_ALARM`。                                                                                |
| durationHint | int                          | 你期望持有焦点的时长类型，系统根据这个决定其他应用的行为。常用值：  <br>- `AUDIOFOCUS_GAIN`：长时间持有，适合音乐/视频。  <br>- `AUDIOFOCUS_GAIN_TRANSIENT`：短暂持有，适合导航语音。  <br>- `AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK`：短暂持有，允许其他音频降低音量。 |
###### 返回值
- AUDIOFOCUS_REQUEST_GRANTED：成功获得焦点。
- AUDIOFOCUS_REQUEST_FAILED：未能获得焦点。
##### 2）释放焦点
```java
void abandonAudioFocus(AudioManager.OnAudioFocusChangeListener listener)
```
###### 参数意义

| 参数       | 类型                           | 作用                                                         |
| -------- | ---------------------------- | ---------------------------------------------------------- |
| listener | `OnAudioFocusChangeListener` | 必须与之前 `requestAudioFocus`时使用的同一个监听器实例，这样系统才能找到对应的焦点请求并释放它。 |
