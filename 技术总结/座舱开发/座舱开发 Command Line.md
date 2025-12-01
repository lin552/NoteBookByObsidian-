---
创建时间: "2025-10-28 10:52:59"
作者: wangxiaoming
tags:
---
#### 一、ADB常用命令
```bash
# 1.adb相关

# 强制安装apk
adb install -r -t -d

# 快捷输入文本
adb shell input text "xxxx"

# 避免debug时出现ANR
adb shell settings put secure anr_show_background 1 

# 切换主题
adb shell am broadcast -a com.xiaopeng.intent.action.SWITCH_DAYNIGHT

# 开启点击反馈
adb shell settings put system show_touches 1

# 开启调试模式
adb shell setprop debug.layout false
adb shell setprop debug.hwui.show_dirty_regions false
adb shell setprop debug.hwui.profile false

# 启动Service
adb shell am startservice -n com.xiaopeng.caraccount/com.xiaopeng.caraccount.login.LoginFlowService  -a "com.xiaopeng.xvs.account.ACTION_ACCOUNT_SHOW_DYNAMIC_DIALOG" --es type remove --es group_id group4 --ez display_state true

# 2.系统命令

# 查看应用进程
ps grep -i musicradio
# 查看aicabin服务
ps -ef | grep -i aicabin

# 杀死进程多个
kill -9 18579 & kill -9 18701 && kil1 -9 19019 杀死进程多个

# 查看车机ROM
cat /system/build.prop | grep -i software
```

#### 二、座舱调试码
```json
# 进入出厂设置入口
*#9387*151#* 

# 进入调试模式
*#9387*141#*

# 查看ROM版本
*#9925*111#*

# 开启预发环境
*#9387*133#*


```