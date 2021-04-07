###### 安装apk

```
adb install **/**.apk
```



###### 查看设备列表

```
adb devices
```



###### 多设备安装apk

```
adb -s 设备id install **/**.apk
```



###### 拉取调试设备的文件

```
adb pull /sdcard/DCIM/Camera/IMG_20170902_111856.jpg IMG_20170902_111856.jpg
adb -s 设备ID pull /sdcard/DCIM/Camera/IMG_20170902_111856.jpg IMG_20170902_111856.jpg
```



###### 往调试设备传文件

```
adb push **/** **/**
adb -s 设备id push **/** **/**
```

