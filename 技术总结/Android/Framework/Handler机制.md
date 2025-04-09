---
创建时间: 2025-03-14T18:39:00
作者: wangxiaoming
tags:
  - Android
  - Handler
---
#### 一、Handler解决了什么问题

**合理调度的交互方案：** 主线程处理UI操作，子线程执行耗时任务，Handler提供了主线程和子线程交互的方案(基于MessageQueue 和 Looper 且支持延时任务 postDelayed)
#### 二、Handler核心优势

##### 1)异步消息处理
通过 `sendMessage()` 或 `post(Runnable)` 将任务封装为消息，由目标线程的 Looper 按顺序处理，避免并发冲突。
##### 2)线程安全与消息队列
消息队列通过 ​**synchronized 锁** 保证多线程插入的安全性，Looper 的循环机制确保消息按时间顺序处理。
##### 3)定时与延时任务
`postDelayed()` 方法通过计算消息的 ​**执行时间戳（`when`）​**，实现精准的延时任务调度
##### 4)资源复用
​**Message 池机制**​（`Message.obtain()`）复用消息对象，减少内存分配和 GC 频率
#### 三、使用注意事项
##### 1）内存泄露
- ​**原因**：Handler 持有 Activity 的隐式引用，若 Activity 销毁时未移除未处理消息，会导致内存泄漏
- ​**解决**：
    - 使用 ​**静态内部类 + WeakReference** 弱引用
    - 在 `onDestroy()` 中调用 `handler.removeCallbacksAndMessages(null)` 移除消息