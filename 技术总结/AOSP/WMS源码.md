---
创建时间: 2025-07-29 21:56:20
作者: wangxiaoming
tags:
  - WMS
---
Android 的 ​`WindowManagerService`（`WMS`）​**​ 是系统图形显示和窗口管理的核心服务，其源码内容复杂且高度模块化。以下从架构设计、核心模块、关键流程及源码实现角度解析其内容：

#### 一、架构设计

`WMS` 的架构围绕 ​**窗口层级管理**​ 和 ​**显示合成**​ 展开，核心设计包括：
##### 1.容器树结构
- 以 `RootWindowContainer`为根节点，通过树形结构管理所有窗口容器（如 `DisplayContent`、`TaskDisplayArea`、`WindowState`），实现窗口的层级排序和显示区域划分
- **`DisplayContent`​：管理单个物理/逻辑屏幕的窗口集合。
- ​`TaskDisplayArea`**​：管理 Activity 任务栈（`ActivityStack`）的窗口。
- ​`WindowState`：表示具体窗口实例，包含窗口属性、位置、Surface 等信息
##### 2.核心类与接口
- ​`WindowManagerService`：主类，负责窗口创建、销毁、布局、输入事件分发等核心逻辑。
- ​`WindowState`​：窗口状态对象，保存窗口属性（如 Z-order、尺寸、透明度和 Surface 信息）。
- `WindowToken`：窗口令牌，用于标识窗口集合（如 Activity 的主窗口和子窗口）。
- ​`DisplayContent`​：管理屏幕上的窗口布局和 Surface 分配。
- `InputManagerService（IMS）​`：协作处理输入事件分发

#### 二、核心模块与功能
##### 1.窗口管理
- **窗口生命周期**​：通过 `addWindow()`、`removeWindow()`等方法管理窗口创建与销毁，涉及权限校验（如 `mPolicy.checkAddPermission()`）和 Surface 分配
- ​**层级与 Z-order**​：通过 `computeWindowLayers()`计算窗口层级，确保系统窗口（如状态栏）优先绘制
##### 2.输入事件分发
- **事件路由**​：输入事件由 `InputManagerService`捕获后，`WMS` 根据窗口的 Z-order 和焦点状态（`mFocusedWindow`）确定目标窗口，通过 `deliverPointerEvent()`分发
- **焦点管理**​：动态调整焦点窗口（如点击事件触发焦点切换）
##### 3.动画与显示
- ​`WindowAnimator`​：管理窗口动画（如 Activity 切换动画），通过 `Choreographer`协调帧刷新
- **Surface 合成**​：与 `SurfaceFlinger` 协作，通过 `SurfaceControl`分配图形缓冲区，实现硬件加速渲染

##### 4.多窗口支持
- **窗口类型**​：支持应用窗口（`TYPE_APPLICATION`）、系统窗口（`TYPE_SYSTEM_ALERT`）、子窗口（`TYPE_APPLICATION_PANEL`）等，通过 `DisplayArea`分组管理
- **分屏与自由模式**​：通过调整 `TaskDisplayArea`的布局策略实现多窗口布局

#### 三、关键源码流程
##### 1.`WMS` 启动流程
- `SystemServer `初始化**​：在 `SystemServer.startOtherServices()`中调用 `WindowManagerService.main()`创建 `WMS` 实例
- ​**初始化 Policy**​：通过 `PhoneWindowManager`加载窗口管理策略（如系统栏位置、权限规则）
- **Display 配置**​：调用 `displayReady()`完成屏幕分辨率、方向等参数的初始化

##### 2.窗口添加流程
1. ​**应用层调用**​：通过 `WindowManager.addView()`触发 Binder 调用，传递至 `WMS` 的 `addWindow()`方法
2. ​**权限校验**​：检查窗口类型和权限（如 `mPolicy.checkAddPermission()`）
3. **创建 `WindowState`​：实例化窗口对象，绑定 Surface 并加入 `mWindowMap`
4. ​**层级计算**​：更新 `DisplayContent`的窗口层级树，触发 `SurfaceFlinger` 合成

##### 3.输入事件处理
 - **事件捕获**​：`InputManager` 捕获触摸事件后，调用 `WMS.processPointerEvent()`。
- ​**焦点判断**​：根据窗口的坐标和 Z-order 确定目标窗口，通过 Binder 回调应用进程处理

#### 四、源码关键路径
##### 核心类路径 ：`frameworks/base/services/core/java/com/android/server/wm/`
- `WindowManagerService.java`：主服务类。
- `WindowState.java`：窗口状态管理。
- `DisplayContent.java`：屏幕内容管理。
- `WindowToken.java`：窗口令牌管理。
###### Surface 控制：
`frameworks/native/libs/gui/SurfaceFlinger.cpp`：与 `SurfaceFlinger` 交互的底层代码。