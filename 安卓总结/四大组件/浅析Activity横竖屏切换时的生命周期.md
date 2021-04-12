**1.AndroidManifest没有设置configChanges属性**

竖屏启动：

onCreate -->onStart-->onResume

切换横屏：

onPause -->onSaveInstanceState -->onStop -->onDestroy -->onCreate-->onStart -->

onRestoreInstanceState-->onResume -->onPause -->onStop -->onDestroy    

（Android 6.0 Android 7.0 Android 8.0）

横屏启动：

onCreate -->onStart-->onResume

切换竖屏：

onPause -->onSaveInstanceState -->onStop -->onDestroy -->onCreate-->onStart -->

onRestoreInstanceState-->onResume -->onPause -->onStop -->onDestroy    

（Android 6.0 Android 7.0 Android 8.0）

总结：**没有设置configChanges属性Android 6.0 7.0 8.0 系统手机 表现都是一样的，当前的界面调用onSaveInstanceState走一遍流程，然后重启调用onRestoreInstanceState再走一遍完整流程，最终destory。**



**2.AndroidManifest设置了configChanges  android:configChanges="orientation"**

竖屏启动：

onCreate -->onStart-->onResume

切换横屏：

onPause -->onSaveInstanceState -->onStop -->onDestroy -->onCreate-->onStart -->

onRestoreInstanceState-->onResume -->onPause -->onStop -->onDestroy    

（Android 6.0）

onConfigurationChanged-->onPause -->onSaveInstanceState -->onStop -->onDestroy -->

onCreate-->onStart -->onRestoreInstanceState-->onResume -->onPause -->onStop -->onDestroy    

（Android 7.0）

 onConfigurationChanged  

（Android 8.0）

  

横屏启动：

onCreate -->onStart-->onResume

切换竖屏：

onPause -->onSaveInstanceState -->onStop -->onDestroy -->onCreate-->onStart -->

onRestoreInstanceState--> onResume -->onPause -->onStop -->onDestroy    

（Android 6.0 ）  

onConfigurationChanged-->onPause -->onSaveInstanceState -->onStop -->onDestroy -->

onCreate-->onStart -->onRestoreInstanceState-->onResume -->onPause -->onStop -->onDestroy    

（Android 7.0）

onConfigurationChanged  

（Android 8.0）

总结：**设置了configChanges属性为orientation之后，Android6.0 同没有设置configChanges情况相同，完整的走完了两个生命周期，调用了onSaveInstanceState和onRestoreInstanceState方法；Android 7.0则会先回调onConfigurationChanged方法，剩下的流程跟Android 6.0 保持一致；Android 8.0 系统更是简单，只是回调了onConfigurationChanged方法，并没有走Activity的生命周期方法。**



**4.AndroidManifest设置了configChanges**  

**android:configChanges="orientation|screenSize"** 

竖(横)屏启动：onCreate -->onStart-->onResume

切换横(竖)屏：onConfigurationChanged  （Android 6.0 Android 7.0 Android 8.0）

总结：没有了keyboardHidden跟3是相同的，orientation代表横竖屏切换 screenSize代表屏幕大小发生了改变，

设置了这两项就不会回调Activity的生命周期的方法，只会回调onConfigurationChanged 。



**5.AndroidManifest设置了configChanges**  

**android:configChanges="orientation|keyboardHidden"** 

总结：跟只设置了orientation属性相同，Android6.0 Android7.0会回调生命周期的方法，Android8.0则只回调onConfigurationChanged。说明如果设置了orientation 和 screenSize 都不会走生命周期的方法，keyboardHidden不影响。

1.不设置configChanges属性不会回调onConfigurationChanged，且切屏的时候会回调生命周期方法。

2.只有设置了orientation 和 screenSize 才会保证都不会走生命周期，且切屏只回调onConfigurationChanged。

3.设置orientation，没有设置screenSize，切屏会回调onConfigurationChanged，但是还会走生命周期方法。

注：这里只选择了Android部分系统的手机做测试，由于不同系统的手机品牌也不相同，可能略微会有区别。

   

另：

**代码动态设置横竖屏状态**（onConfigurationChanged当屏幕发生变化的时候回调）

setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE);

setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_PORTRAIT);



**获取屏幕状态**（int ORIENTATION_PORTRAIT = 1; 竖屏  int ORIENTATION_LANDSCAPE = 2; 横屏）

int screenNum = getResources().getConfiguration().orientation;



**configChanges属性**

1. orientation 屏幕在纵向和横向间旋转 

2.keyboardHidden 键盘显示或隐藏 

3.screenSize 屏幕大小改变了 

4.fontScale 用户变更了首选的字体大小 

5.locale 用户选择了不同的语言设定 

6.keyboard 键盘类型变更，例如手机从12键盘切换到全键盘 

7.touchscreen或navigation 键盘或导航方式变化，一般不会发生这样的事件

**常用的包括：orientation keyboardHidden screenSize，设置这三项界面不会走Activity的生命周期，只会回调onConfigurationChanged方法。**



**screenOrientation属性**

1.unspecified 默认值，由系统判断状态自动切换 

2.landscape 横屏 

\3. portrait 竖屏 

4.user 用户当前设置的orientation值 

\5. behind 下一个要显示的Activity的orientation值 

\6. sensor 使用传感器 传感器的方向 

\7. nosensor 不使用传感器 基本等同于unspecified

仅landscape和portrait常用，代表界面默认是横屏或者竖屏，还可以再代码中更改。