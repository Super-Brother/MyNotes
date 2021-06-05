扫描枪蓝牙连接或者断开的时候，会导致当前所在的`activity onDestroy -> onCreate`

即activity被销毁又重建，类似屏幕旋转导致activity重建

解决办法： 在xml中配置：

```
android:configChanges=“orientation|keyboard|keyboardHidden”
```

