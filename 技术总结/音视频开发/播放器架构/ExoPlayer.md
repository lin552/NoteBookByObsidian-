---
创建时间: "2026-03-10 10:14:56"
作者: wangxiaoming
tags:
---
#### 一、源码结构
```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          ExoPlayer 架构图                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  🟢 Core Library (必须)                                                     │
│     ├─ MediaSource (数据源抽象)                                              │
│     ├─ Renderer (音频/视频/字幕渲染器)                                        │
│     ├─ TrackSelector (轨道选择)                                              │
│     └─ AudioSink / VideoSink (输出终点)                                      │
│                                                                             │
│  🔵 Extensions (可选扩展)                                                    │
│     ├─ audio (音频增强/重采样)                                                │
│     ├─ drm (Widevine、PlayReady 等 DRM 支持)                                 │
│     ├─ ffmpeg (FFmpeg 音频解码扩展)                                           │
│     ├─ text (字幕渲染)                                                       │
│     └─ ...                                                                  │
│                                                                             │
│  🟡 UI Components (可选 UI)                                                 │
│     ├─ PlayerView (播放器视图)                                               │
│     ├─ PlaybackControlView (控制栏)                                          │
│     └─ ...                                                                  │
│                                                                             │
│  🔴 依赖库：Android MediaCodec、OpenGL ES、AudioTrack 等系统 API              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```
#### 二、核心库（Core Library）
这是 `ExoPlayer` 的**骨架**，没有它播放器根本跑不起来。

包含：
- `MediaSource`：数据源抽象（从网络、本地、文件、HLS、DASH 等读取媒体）
- `Renderer`：渲染器（音频、视频、字幕等）
- `TrackSelector`：轨道选择器（选音轨、字幕轨）
- `AudioSink / VideoSink`：音频/视频输出终点（送系统播放器或自定义硬件）
- `Player`：播放控制接口（播放、暂停、seek、监听状态）
- `DataSource`：数据读取抽象（HTTP、文件、Asset、Cache）
- `Extractor`：解封装器（MP4、FLAC、MP3、AAC、MKV 等）
- `Util / C`：工具类、JNI 支持

#### 三、扩展库（Extensions）
这些是**增强功能**，不是必须的，但能提供额外格式、协议、硬件支持。

| 扩展名          | 功能说明                                                                    |
| ------------ | ----------------------------------------------------------------------- |
| `ffmpeg`     | 用 `FFmpeg` 解码音频（支持 `MP3、FLAC、AC3、E-AC3、DTS` 等）                          |
| `cast`       | 支持 `Chromecast` 投屏                                                      |
| `ima`        | 集成 `Google IMA` 广告 `SDK`                                                |
| `mediacodec` | 封装 `Android MediaCodec`（硬解）                                             |
| `opus`       | `Opus` 音频解码支持                                                           |
| `av1`        | `AV1` 视频解码支持（需系统支持）                                                     |
| `core`       | 你截图中的这个，其实是“扩展核心”——它提供了一些通用的扩展基类或工具，比如 `FfmpegAudioRenderer`就是在这个扩展里实现的 |
