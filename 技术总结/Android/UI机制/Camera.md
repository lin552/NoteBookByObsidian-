---
创建时间: "2025-12-04 14:56:20"
作者: wangxiaoming
tags:
---
`android.graphics.Camera`是 Android 图形系统中用于**实现 3D 变换效果**的核心类，位于 `android.graphics`包下。它通过**虚拟摄像机模型**将 3D 场景投影到 2D 屏幕，生成变换矩阵（`Matrix`），从而为自定义 `View`、`Drawable` 或 `Bitmap` 提供立体旋转、平移等效果。
与系统相机（`android.hardware.Camera`或 `Camera2`）不同，`android.graphics.Camera`不涉及实际拍照功能，仅用于**图形渲染中的 3D 变换**，是自定义动画、立体 UI 效果的关键工具。

#### 一、核心功能与API
##### 1）基础操作
- `save()/ restore()`：保存/恢复摄像机的当前状态（如位置、旋转角度），避免操作冲突。
- `getMatrix(Matrix matrix)`：将摄像机的变换状态（平移、旋转）应用到传入的 Matrix对象中，生成最终的变换矩阵。该矩阵可用于 `Canvas.concat(matrix)`或 `Bitmap.createBitmap()`实现图形变换。
##### 2)3D变换操作
- **平移（`translate(float x, float y, float z)`）**：调整摄像机的位置。例如，`camera.translate(100, 0, 0)`表示摄像机沿 X 轴正方向移动 100 个单位（注意：摄像机移动方向与视图移动方向相反，如摄像机右移等价于视图左移）。
- **旋转（`rotateX(float degrees)`/ `rotateY(float degrees)`/ `rotateZ(float degrees)`）**：绕指定轴旋转摄像机。例如，`camera.rotateY(45)`表示摄像机绕 Y 轴旋转 45 度，对应视图的 3D 旋转效果。
- **位置设置（`setLocation(float x, float y, float z)`）**：直接设置摄像机的绝对位置（默认位置为 `(0, 0, -8)`）。需注意：**摄像机与视图的 Z 轴距离不能为 0**（否则无法拍摄到视图）。