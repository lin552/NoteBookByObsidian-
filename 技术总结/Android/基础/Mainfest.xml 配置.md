---
创建时间: "2025-10-28 10:51:46"
作者: wangxiaoming
tags:
---
#### 一、Activity 配置

configChanges = 被配置的会被Activity自行处理变更，会执行onConfigurationChanged（）
1.mcc 当系统检测到用于更新MCC的SIM卡时
2.mnc 当系统检测用于更新MNC的SIM卡时
3.locale 语言区域更改
4.touchscreen 触摸屏模式更改
5.keyboard 键盘类型更改
6.keyboardHidden 键盘无障碍功能更改
7.navigation 导航类型的TA更改
8.screenLayout 屏幕布局的更改
9.fontScale 字体缩放比例的更改
10.uiMode 界面模式的更改
11.orientation 屏幕反向的更改，例如用户旋转设备时
12.screenSize 当前可用屏幕尺寸的更改
13.smallScreenSize 实体屏幕尺寸更改（切换到外部显示器）

windowSoftInputMode =
1.adjustNothing 不通过调整或平移activity主窗口的尺寸为软键盘腾出空间
2.stateAlwaysHidden 当窗口获得输入焦点时，会显示软键盘