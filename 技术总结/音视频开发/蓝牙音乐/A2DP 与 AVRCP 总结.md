---
创建时间: "2026-03-16 21:21:40"
作者: wangxiaoming
tags:
---
#### 一、对比总结

| 特性            | A2DP                                   | AVRCP                                |
| ------------- | -------------------------------------- | ------------------------------------ |
| 全称            | `​Advanced Audio Distribution Profile` | `Audio/Video Remote Control Profile` |
| 主要作用          | 传输高质量音频流                               | 远程控制音视频播放                            |
| 数据方向          | 单向（`Source → Sink`）                    | 双向（`Controller ↔ Target`）            |
| 是否传输音频        | ​✅ 是                                   | ❌ 否                                  |
| 是否传输控制命令      | ❌ 否                                    | ✅ 是                                  |
| 是否传输元数据       | ❌ 否                                    | ✅ 是（1.3+）                            |
| **是否传输媒体库**​  | ❌ 否                                    | ✅ 是（1.4+）                            |
| 协议基础          | `AVDTP over L2CAP`                     | `AVCTP over L2CAP`                   |
| 编解码器          | 需要（`SBC, AAC, aptX, LDAC...`）          | 不需要                                  |
| 常见角色          | `​Source / Sink`                       | `Controller / Target`                |
| 典型场景          | 音乐/视频/游戏音频输出                           | 耳机按钮/车载控制/智能手表/遥控器                   |
| 是否必须一起用       | 否，可独立                                  | 否，可独立，但常与 `A2DP` 配合                  |
| 在 `Android` 中 | ​系统自动管理音频路由                            | 通过 `MediaSession` 接收控制（或系统处理）        |
