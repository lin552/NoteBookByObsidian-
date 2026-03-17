---
创建时间: "2026-03-16 21:05:29"
作者: wangxiaoming
tags:
---
#### 一、全称与定位

- **AVRCP**: **Audio/Video Remote Control Profile**（音视频远程控制配置文件）
- 同样是 **Bluetooth SIG**​ 的标准配置文件。
- 位于 **Bluetooth Profiles**​ 层，基于 **L2CAP**​ 的 `AVCTP（Audio/Video Control Transport Protocol）` 传输控制命令。

> 简单说：AVRCP 是蓝牙世界里**专门负责“发命令”**的协议，控制音视频的播放。

#### 二、核心作用
- 在**控制器（`Controller`）和**目标设备（`Target`）**之间建立**双向控制命令通道**。
- 让 `Controller` 设备（如耳机、车载中控、智能手表）能够**远程控制**​ `Target` 设备（如手机、电脑、电视）的媒体播放。
- 可以传输**媒体元数据**（歌曲名、艺术家、专辑封面）和**控制事件**（播放、暂停、切歌、快进、音量）。

#### 三、技术架构
```bash
┌─────────────────────────────────────────────────────────────┐
│                    AVRCP 协议栈架构                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Application Layer (你的 App / 系统媒体服务)                  │
│       │                                                     │
│       ▼                                                     │
│  AVRCP Profile (定义角色: Controller/Target)                 │
│       │                                                     │
│       ▼                                                     │
│  AVCTP (Audio/Video Control Transport Protocol)             │
│       │  - 负责控制命令的封装、传输、响应                       │
│       ▼                                                     │
│  L2CAP (Logical Link Control and Adaptation Protocol)       │
│       │  - 提供面向连接的、可靠的数据链路层                      │
│       ▼                                                     │
│  Baseband (蓝牙物理层/链路层)                                 │
│       │  - 负责跳频、调制解调、数据包传输                       │
│       ▼                                                     │
│  Radio (2.4GHz ISM Band)                                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```
#### 四、关键概念
##### 1）`Controller` 和 `Target`
- **Controller（控制器）**：发送控制命令的设备，如耳机按键、车载中控、智能手表。
- **Target（目标）**：执行控制命令的设备，如手机、电脑、电视。
- 一个设备可以同时是多个配对的 `Controller` 或 `Target`，但在一个 `AVRCP` 连接中，角色是固定的。

##### 2)  控制命令集
`AVRCP` 定义了一组**标准控制事件（PDU，Protocol Data Unit）**：

| 命令                     | 功能                 | 方向  |
| ---------------------- | ------------------ | --- |
| **PLAY**​              | 播放                 | C→T |
| **PAUSE**​             | 暂停                 | C→T |
| STOP                   | 停止                 | C→T |
| NEXT TRACK             | 下一曲                | C→T |
| PREVIOUS TRACK         | 上一曲                | C→T |
| VOLUME UP              | 音量增加               | C→T |
| VOLUME DOWN            | 音量减少               | C→T |
| GET CAPABILITIES       | 获取目标能力             | C→T |
| GET ELEMENT ATTRIBUTES | 获取当前媒体元数据          | C→T |
| SET_ABSOLUTE_VOLUME    | 设置绝对音量             | C→T |
| BROWSE                 | ​浏览媒体库（AVRCP 1.4+） | C→T |
##### 3）元数据（Metadata）
AVRCP 1.3+ 支持传输丰富的媒体信息，通过 `GetElementAttributes`命令获取：
- 标题（Title）
- 艺术家（Artist）
- 专辑（Album）
- 曲目编号（Track Number）
- 总时长（Total Duration）
- 封面（Cover Art，需 AVRCP 1.5+ 及支持 GATT 传输图片）

> 在 Android 中，这些元数据会填充到 `MediaMetadata`中，供锁屏、通知栏、车载系统使用。
##### 4) 版本差异
- **AVRCP 1.0**: 基本控制（Play/Pause/Stop/Next/Prev）
- **AVRCP 1.3**: 增加元数据、支持绝对音量
- **AVRCP 1.4**: 增加媒体库浏览（Browse）、支持更多属性
- **AVRCP 1.5/1.6**: 增加高分辨率封面、更丰富的控制、与 Bluetooth 5.x 结合
#### 五、工作流程
##### 1）`A2DP`连接建立（可选但常见）
先有 A2DP 传声，再建 AVRCP 控制。
##### 2）`AVRCP` 连接建立：
- Controller 向 Target 发送 `Connect`请求。
- 建立 AVCTP 通道。
##### 3) 能力协商
`Controller` 发送 `GetCapabilities`了解 `Target` 支持哪些命令和事件。
##### 4) 控制命令交互
用户按耳机按钮 → `Controller` 发送 `PLAY/PAUSE` 等命令 → `Target` 执行并可能返回响应。
##### 5) 元数据获取
`Controller` 发送 `GetElementAttributes`→ `Target` 返回当前歌曲信息。
##### 6) 媒体库浏览
`Controller` 发送 `Browse`请求 → `Target` 返回目录结构、歌曲列表。

#### 六、音乐场景

- 蓝牙耳机/音箱的播放/暂停/切歌按钮
- 车载中控屏控制手机/USB 设备上的音乐/视频
- 智能手表控制手机音乐
- 蓝牙遥控器控制电视/投影仪
- 家庭影院系统遥控器
- 会议系统控制演示设备
- 无障碍设备（蓝牙开关控制手机媒体）
- 智能电视/机顶盒的手机遥控 App