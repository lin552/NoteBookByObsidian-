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

# 查看车机属性
getprop sys.xiaopeng.vin 

# 酷狗杜比 AMP开关
setprop persist.sys.xiaopeng.DOLBY 1 
setprop persist.sys.xiaopeng.AMP 1

# 使用工具打开后排屏
D:\工具\scrcpy-win64-v1.25\scrcpy.exe --display 4
```

#### 二、维诚的遗产
```bash
// 切换语言 adb shell am broadcast -a com.xiaopeng.intent.action.LANGUAGE_CHANGE --ei reboot 0 

// 切换字号 adb shell am broadcast -a com.xiaopeng.xpui.demo.ACTION --es a font --es v 1.08 

// 日夜间切换 // 模拟napa调用业务接口 v1=appid v2=module v3=method v4=param adb shell am broadcast -a NAPA_MOCK --es v1 "Settings" --es v2 "wifi" --es v3 "setPower" --es v4 "true" 

// P档 adb shell vdt rp VCU_CURRENT_GEARLEV 4 

// 挂P挡 adb shell vdt logctrl SIGNALLOST 0 & adb shell vdt rp VCU_CURRENT_GEARLEV 4

// Ready状态切换 adb shell vdt logctrl SIGNALLOST 0 & adb shell vdt rp VCU_EVSYS_READYST 0 0 是低压，1是高压，2是ready 
 
// 测试可见即可说 老车型需要打开全场景语音开关，XOS5需要关闭智能唤醒开关 adb shell am broadcast -a "carspeechservice.ACTION_SEND_TEXT" --es text "你好" 档 挡 

// 蓄电池不足 adb shell vdt logctrl SIGNALLOST 0 adb shell vdt rp VCU_EBS_BATT_SOC 70 

// 清下载数据 adb shell pm clear com.xiaopeng.resourceservice adb shell pm clear com.android.providers.downloads 配置测试数据 adb shell am broadcast -a carconfig.intent.action.mock.car --es car F30R adb shell am broadcast -a carconfig.intent.action.mock.car --es car exit 

// 关闭水印 adb shell setprop settings.language.mask 1 

// 替换本地feature adb disable-verity adb reboot adb root adb remount 

// 切车型配置 adb shell am broadcast -a carconfig.intent.action.mock.car --es car F30R adb shell am broadcast -a carconfig.intent.action.mock.car --es car exit 

// kill adb shell am force-stop com.xiaopeng.autoshow 

// mock 属性 adb shell am broadcast -a carconfig.intent.action.mock.one_config.value --es key getDefaultMirrorReversePos --es value 1,2,3,4,5,6 

// 投屏 scrcpy --display 1 "下电 adb shell vdt rp MCU_IG_DATA 0 上电 adb shell vdt rp MCU_IG_DATA 1" 

// NEDC adb shell vdt rp XPU_NEDC_STATUS 0 H93是用的这个看后排娱乐屏adb shell setprop persist.sys.xiaopeng.SFM 1 

// 自动下电 adb shell vdt rp TBOX_COUNTDOWN 10 0
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