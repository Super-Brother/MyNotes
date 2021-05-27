2 写一个选择器，里面是自己要设置的打开和关掉的按钮图标
selector_radio.xml

```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@mipmap/radio_on" android:state_checked="true"></item>
    <item android:drawable="@mipmap/radio_off"></item>
</selector>
```

3 RadioButton的重要属性
// 屏蔽默认的按钮图标
`android:button=”@null”`
// 设置自己在按钮图标
`android:drawableLeft=”@drawable/selector_radio”`
// 图标的内边距是5
android:drawablePadding=”5dp”
// 单选框是选中状态
`android:checked=”true”`