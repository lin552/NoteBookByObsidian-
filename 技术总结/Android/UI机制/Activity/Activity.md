---
创建时间: 2025-04-12 17:24:57
作者: wangxiaoming
tags:
  - Activity
---
在Android开发中，​**Activity**是四大组件之一，也是面试和考试中的高频考点。以下是结合搜索结果的**Android Activity核心考点**梳理，涵盖生命周期、状态管理、启动模式、通信机制等关键内容：

#### 一、Activity生命周期
##### 1. ​生命周期方法及调用时机​
- ​**核心方法**​：  
    `onCreate()`（初始化`UI`和数据）→ `onStart()`（可见但不可交互）→ `onResume()`（可交互）→ `onPause()`（失去焦点但可见）→ `onStop()`（不可见）→ `onDestroy()`（销毁）→ `onRestart()`（从不可见到可见）
- ​**关键场景**​：
    - ​**启动Activity**​：`onCreate()` → `onStart()` → `onResume()`。
    - ​**返回键退出**​：`onPause()` → `onStop()` → `onDestroy()`。
    - ​**横竖屏切换**​：默认重建Activity（调用`onSaveInstanceState()`保存状态，`onRestoreInstanceState()`恢复）

##### 2. ​**状态转换与内存管理**​
- ​**四种状态**​：
    - ​**运行状态**​（Active/Running）：栈顶，用户可交互。
    - ​**暂停状态**​（Paused）：部分可见（如对话框覆盖），仍持有资源。
    - ​**停止状态**​（Stopped）：完全不可见，可能被系统回收。
    - ​**销毁状态**​（Killed）：被系统终止，需重新创建
- ​**进程优先级**​：前台进程 > 可见进程 > 服务进程 > 后台进程 > 空进程

#### 二、Activity启动模式
##### 1. ​四种启动模式
| 模式                   | 行为描述                                     | 适用场景           |
| -------------------- | ---------------------------------------- | -------------- |
| ​**standard**​       | 每次启动创建新实例，可叠加到当前任务栈。                     | 普通跳转（如多个详情页）   |
| ​**singleTop**​      | 若栈顶已是该实例，则复用（调用`onNewIntent()`），否则创建新实例。 | 接收通知跳转（避免重复实例） |
| ​**singleTask**      | 全局唯一实例，若存在则清空栈顶以上实例并复用，否则新建。             | 主界面（如微信首页）     |
| ​**singleInstance**​ | 独占任务栈，仅允许一个Activity存在，其他应用跳转时共享该栈。       | 全屏视频、电话拨号界面    |
##### 2. ​Intent标志位​
- `FLAG_ACTIVITY_CLEAR_TOP`：若目标Activity在栈中，清空其上方所有Activity。
- `FLAG_ACTIVITY_SINGLE_TOP`：等效于`singleTop`模式

#### 三、Activity通信机制
##### 1. ​Intent传递数据​
- ​**基本用法**​：`putExtra(key, value)`传递简单数据，`Bundle`传递复杂对象。
- ​**返回数据**​：通过`startActivityForResult()`和`onActivityResult()`实现
##### 2. ​跨进程通信​
- ​**绑定Service**​：通过`bindService()`实现与Service交互。
- ​**广播**​：发送`BroadcastReceiver`接收系统或应用事件

#### 四、任务栈与返回栈管理
- ​**任务栈（Task）​**​：按“后进先出”管理Activity实例，用户按Back键逐层弹出。
- ​**返回栈优化**​：
    - 使用`finish()`主动关闭当前Activity。
    - 通过`moveTaskToBack()`将任务移到后台（类似Home键）

#### 五、配置变更与数据保存
- ​**横竖屏切换**​：
    - 默认重建Activity，需在`AndroidManifest.xml`设置`android:configChanges="orientation|screenSize"`避免重建。
    - 重写`onConfigurationChanged()`动态调整布局
    
- ​**数据持久化**​：
    - 临时保存：`onSaveInstanceState()` → `onRestoreInstanceState()`。
    - 永久保存：`SharedPreferences`、数据库或文件
#### 六、进程与内存优化
- ​**内存泄漏场景**​：
    - 静态变量持有Activity引用。
    - 未注销的广播接收器（`BroadcastReceiver`）。
    - 非静态内部类（如Handler）隐式持有外部Activity引用

- ​**优化方案**​：
    - 使用`WeakReference`弱引用。
    - 在`onDestroy()`中释放资源（如取消网络请求、注销监听）。
#### 七、常见面试题
1. ​**Activity生命周期有哪些方法？横竖屏切换时生命周期如何变化？​**​
    - 答：
	    ①默认情况：
	    - 旧Activity(竖屏)的生命周期回调 `onPause() -> onStop() ->onDestroy()`
	    - 新Activity(竖屏)的生命周期回调 `onCreate() -> onStart() ->onResume()`
	    
	    ②设置了`configChanges` 配置了"`android:configChanges="orientation`"（或更全面的 `screenSize|orientation`）
	     旧Activity(竖屏)的生命周期回调 `onConfigurationChanged(Configuration newConfig)​`：直接触发此方法，系统通过 `newConfig` 传递新的配置信息（如屏幕方向、尺寸等）。
		    

2. ​**Activity的四种启动模式有何区别？如何选择？​**​
    - 答：对比表格说明行为差异，结合场景（如主界面、通知跳转）选择模式

3. ​**如何实现两个Activity间的数据传递与返回？​**​
    - 答：使用`Intent`传递数据，`startActivityForResult()`接收返回结果

4. ​**Activity被系统回收时如何保存状态？​**​
    - 答：重写`onSaveInstanceState()`保存关键数据，`onRestoreInstanceState()`恢复

5. ​**Activity A 启动 Activity B 的生命周期变化？​**​

①B为全屏
总结：
`A onPause()` 后 B开始`onCreate()->onStart()->onResume()` A 再开始`onStop()`
返回A时， `B onPause()` `A onRestart() -> onStart() -> onResume()` 后 B `onStop()` -> `onDestroy()`


| 时间点 | Activity A的生命周期回调 | Activity B的生命周期回调 | 说明                                                       |
| --- | ----------------- | ----------------- | -------------------------------------------------------- |
| 1   | `onPause()`       | -                 | A失去焦点（界面可能仍可见，但无法交互），开始保存临时状态（如调用`onSaveInstanceState`）。 |
| 2   | -                 | `onCreate()`      | B被创建，初始化布局、数据等（首次启动）。                                    |
| 3   | -                 | `onStart()`       | B可见（但未获取焦点，准备显示）。                                        |
| 4   | `onStop()`        | `onResume()`      | B获取焦点，进入前台可交互状态；A完全不可见（被B覆盖），停止运行。                       |
| 5   | -                 | （B运行中，无回调）        | B处于`resumed`状态，用户与B交互。                                   |
| 6   | `onRestart()`     | `onPause()`       | B被关闭（调用`finish()`），失去焦点；A重新启动（从`onStop()`恢复）。            |
| 7   | `onStart()`       | `onStop()`        | A可见（准备显示），B完全不可见（被A覆盖），停止运行。                             |
| 8   | `onResume()`      | `onDestroy()`     | A回到前台可交互状态；B被销毁，释放资源。                                    |
②B为透明非全屏
总结 ：
A到B：和情况①类似  但A 无`onStop()`
B返回A： 因为A 没`onStop()` 随意A恢复时没有`onRestart()` 直接从`onPause() -> onResume()`

| 时间点 | Activity A的生命周期回调 | Activity B的生命周期回调 | 说明                                         |
| --- | ----------------- | ----------------- | ------------------------------------------ |
| 1   | `onPause()`       | -                 | A失去焦点（界面可能变暗或部分可见），开始保存临时状态。               |
| 2   | -                 | `onCreate()`      | B被创建（布局为透明，可能仅覆盖A的部分区域）。                   |
| 3   | -                 | `onStart()`       | B可见（透明，A的内容仍可见），准备显示。                      |
| 4   | （无`onStop()`）     | `onResume()`      | B获取焦点（可能与A的前台状态共存，具体取决于窗口层级）。              |
| 5   | -                 | （B运行中，无回调）        | B处于`resumed`状态，用户可与B交互（但A的界面仍部分可见）。        |
| 6   | `onPause()`       | `onPause()`       | B被部分遮挡（如用户点击返回键前的瞬间），A重新获取部分焦点。            |
| 7   | `onResume()`      | `onStop()`        | B失去焦点（准备关闭），A恢复前台交互（但仍透明）。                 |
| 8   | -                 | `onDestroy()`     | B被销毁，释放资源。                                 |
| 9   | `onResume()`      | -                 | B完全销毁，A回到前台可交互状态（无`onStop()`/`onStart()`）。 |


③B为透明非全屏
总结：和②完全表现一样

|时间点|Activity A的生命周期回调|Activity B的生命周期回调|说明|
|---|---|---|---|
|1|`onPause()`|-|A失去焦点（界面变暗或部分被遮盖），开始保存临时状态。|
|2|-|`onCreate()`|B被创建（布局为Dialog样式，仅覆盖A的中间区域）。|
|3|-|`onStart()`|B可见（Dialog样式，A的边缘内容仍可见），准备显示。|
|4|（无`onStop()`）|`onResume()`|B获取焦点（模态Dialog会拦截大部分事件，但A的界面仍可见）。|
|5|-|（B运行中，无回调）|B处于`resumed`状态，用户主要与B交互（A的界面被半透明遮盖）。|
|6|`onPause()`|`onPause()`|B被点击关闭（如点击返回键或取消按钮），失去焦点；A重新获取焦点。|
|7|`onResume()`|`onStop()`|B失去焦点（准备关闭），A恢复前台交互（界面逐渐清晰）。|
|8|-|`onDestroy()`|B被销毁，释放资源。|
|9|`onResume()`|-|B完全销毁，A回到前台可交互状态（无`onStop()`/`onStart()`）。|
