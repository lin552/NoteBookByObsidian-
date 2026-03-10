---
创建时间: "2026-03-10 10:23:38"
作者: wangxiaoming
tags:
---
#### 一、源码结构
```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          IJKPlayer 架构图                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  🟢 Core Player Framework (必须)                                            │
│     ├─ iplayer/ (jk播放控制、解复用、调度、音视频同步)                           │
│     ├─ ijksdl/ (跨平台渲染抽象：SDL 封装)                                      │     
      └─ Android/iOS 平台适配层 (JNI/Surface/OpenGL)                           │
│                                                                             │
│  🔴 FFmpeg + 第三方库 (必须，编译时链接)                                       │
│     ├─ libavcodec (解码器)                                                   │
│     ├─ libavformat (容器解析)                                                │
│     ├─ libavutil (工具函数)                                                  │
│     ├─ libvpx (VP8/VP9)                                                     │
│     ├─ libaom (AV1)                                                         │
│     ├─ openssl (HTTPS/加密)                                                 │
│     └─ ...                                                                  │
│                                                                             │
│  🟡 平台适配器 (运行时绑定)                                                   │
│     ├─ Android: JNI → Surface + AudioTrack                                  │
│     └─ iOS: JNI → OpenGL / Metal                                            │
│                                                                             │
│  🔵 可选扩展 (需手动编译或配置)                                                │
│     ├─ 硬件加速支持 (MediaCodec、VDA、VideoToolbox)                           │
│     ├─ 自定义滤镜/后处理                                                      │
│     └─ 更多格式支持 (如 FLAC、APE 等)                                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```