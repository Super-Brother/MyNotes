### 问题

上面的代码想在注册布局放在toolbar下面，按照我的正常写法android:layout_width="match_parent"
 android:layout_height="match_parent"  然后app:layout_constraintTop_toBottomOf="@+id/toolbar"放到toolbar下面但是注册页面还是覆盖了toolbar

### 原因

ConstraintLayout 布局不支持match_parent属性

### 解决方案

把注册布局android:layout_width，android:layout_height设置为0再添加约束，看效果

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.hhmab.linkr.widget.CommonToolbar
        android:id="@+id/toolbar"
        app:layout_constraintTop_toBottomOf="parent"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <android.support.constraint.ConstraintLayout
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/toolbar"
        app:layout_constraintBottom_toBottomOf="parent"
        android:background="@color/white"
        android:paddingLeft="40dp"
        android:paddingRight="40dp">

       ...
    </android.support.constraint.ConstraintLayout>
</android.support.constraint.ConstraintLayout>
```

