---
创建时间: "2026-03-16 20:49:44"
作者: wangxiaoming
tags:
---
#### 一、全程与定位
- **A2DP**: **Advanced Audio Distribution Profile**（高级音频分发配置文件）
- 是 **Bluetooth SIG**​ 定义的**标准蓝牙配置文件（Bluetooth Profile）**之一。
- 位于 **Bluetooth Profiles**​ 层，基于 **L2CAP**​ 和 **RFCOMM**​ 等底层协议，但主要使用 **L2CAP**​ 的 **`AVDTP（Audio/Video Distribution Transport Protocol）`​ 来传输音频。

> 简单说：A2DP 是蓝牙世界里**专门负责“发声音”**的协议。
#### 二、核心作用
- 在**源设备（Source）**和**接收设备（`Sink`）**之间建立**单向、持续的高质量音频流传输通道**。
- 将**压缩的音频数据**从 `Source` 推送到 `Sink`，让 `Sink` 设备播放出来。
- 不负责控制播放、暂停、切歌等逻辑，只管“传声”。
#### 三、技术架构
```bash
┌─────────────────────────────────────────────────────────────┐
│                    A2DP 协议栈架构                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Application Layer (你的 App / 系统媒体服务)                 │
│       │                                                     │
│       ▼                                                     │
│  A2DP Profile (定义角色: Source/Sink, 控制点)                │
│       │                                                     │
│       ▼                                                     │
│  AVDTP (Audio/Video Distribution Transport Protocol)         │
│       │  - 负责信令(Signaling): 发现、配置、打开、关闭流     │
│       │  - 负责数据传输(Data Streaming)                      │
│       ▼                                                     │
│  L2CAP (Logical Link Control and Adaptation Protocol)        │
│       │  - 提供面向连接的、可靠的数据链路层                   │
│       ▼                                                     │
│  Baseband (蓝牙物理层/链路层)                                │
│       │  - 负责跳频、调制解调、数据包传输                    │
│       ▼                                                     │
│  Radio (2.4GHz ISM Band)                                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```
#### 四、关键概念
##### 1）`Source` 和 `Sink`
- **Source（源）**：产生音频流的设备，如手机、电脑、电视。
- **Sink（接收）**：接收并播放音频流的设备，如耳机、音箱、汽车音响。
- 一个设备可以同时是多个配对的 `Sink`，但在一个 `A2DP` 连接中，角色是固定的。
##### 2) 编解码器（Codec）
`A2DP` 不规定音频格式，但定义了**编解码器协商机制**，`Source` 和 `Sink` 会协商一个双方都支持的编解码器。

| 编解码器    | 公司/组织        | 特点                  | 音质/带宽         |
| ------- | ------------ | ------------------- | ------------- |
| SBC     | 蓝牙 SIG (强制)  | 基础编解码器，所有设备必支持      | 中等，~328 kbps  |
| AAC     | Apple/Dolby  | 高效压缩，音质好，苹果设备常用     | 高，~320 kbps   |
| aptX    | Qualcomm     | 低延迟，适合游戏/视频         | 高，~352 kbps   |
| aptX HD | Qualcomm     | 高解析度，24bit/48kHz    | 很高，~576 kbps  |
| LDAC    | ​Sony        | 高解析度，支持 24bit/96kHz | 极高，~990 kbps  |
| LHDC    | Huawei/HiRes | 中国标准，类似 LDAC        | 极高，~900+ kbps |
| HWA     | 中国（盛微/华为）    | 与 LHDC 类似           | 高             |
> 编解码器协商是 A2DP 连接建立过程的一部分，由 AVDTP 的信令消息完成。

##### 3）流状态
- **IDLE**: 无活动流
- **CONFIGURED**: 已配置编解码器等参数
- **OPEN**: 流已打开，可以传输数据
- **STREAMING**: 正在传输音频数据
- **CLOSING**: 正在关闭流
- **ABORTING**: 异常终止

#### 五、工作流程
##### 1)设备发现与配对：
通过蓝牙基础协议（`Baseband`, `LMP`）完成。
##### 2）`A2DP`连接建立
- `Source` 向 `Sink` 发送 `AVDTP` `Discover`请求，发现 `Sink` 支持的 `Capabilities`。
- `Source` 发送 `Get Capabilities`获取 `Sink` 支持的编解码器。
- `Source` 选择双方都支持的编解码器，发送 `Set Configuration`进行配置。
- 发送 Open`请求打开流。
##### 3) 音频流传输
- `Source` 将音频数据（`PCM` 经过编解码器压缩后）通过 `L2CAP` 通道发送给 `Sink`。
- `Sink` 接收数据，解码，通过 `DAC` 转换成模拟信号播放。
##### 4) 流控制与关闭：
- 可以发送 `Suspend`暂停流（不关闭连接）。
- 发送 `Close`关闭流。
- 发送 `Abort`异常终止。

#### 六、应用场景
- 手机/电脑 → 蓝牙耳机/音箱（音乐、视频、游戏音频）
- 电视/机顶盒 → 蓝牙耳机（私人观影）
- 游戏主机 → 蓝牙音箱（增强音效）
- 平板电脑 → 蓝牙音箱（家庭聚会）
- 医疗设备（如助听器） → 手机/电脑
- 工业设备（如噪声监测） → 接收端