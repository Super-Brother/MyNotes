# 1 简介

ANR全称：Application Not Responding，也就是应用程序无响应。



# 2 发生条件

Android系统中，ActivityManagerService(简称AMS)和WindowManagerService(简称WMS)会检测App的响应时间，如果App在特定时间无法相应屏幕触摸或键盘输入时间，或者特定事件没有处理完毕，就会出现ANR。

以下四个条件都可以造成ANR发生：

- **InputDispatching Timeout**：**5秒内无法响应屏幕触摸事件或键盘输入事件**
- **BroadcastQueue Timeout** ：在执行前台广播（BroadcastReceiver）的onReceive()函数时10秒没有处理完成，后台为60秒。
- **Service Timeout** ：前台服务20秒内，后台服务在200秒内没有执行完毕。
- **ContentProvider Timeout** ：ContentProvider的publish在10s内没进行完。