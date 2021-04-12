项目需求：在Application中判断用户是否登录，如果登录则跳转到主页，如果未登录则跳转到登录页面。

一般通过Intent跳转到Activity的方法：

```
Intent intent = new Intent(this, MainActivity.class); 
startActivity(intent);
```

在Application中通过以上方式跳转到Activity的话，会出现异常：**原因是原有的任务栈已经销毁，因此要判断启动的activity是不是被销毁了。**

```
android.util.AndroidRuntimeException: Calling startActivity() from outside of an Activity  context requires the FLAG_ACTIVITY_NEW_TASK flag. Is this really what you want?
```

解决方法：**为activity开启新的栈，Intent.FLAG_ACTIVITY_NEW_TASK 设置状态**，首先查找是否存在和被启动的Activity具有相同的任务栈，如果有则直接把这个栈整体移到前台，并保持栈中的状态不变，既栈中的activity顺序不变，如果没有，则新建一个栈来存放被启动的Activity。

```
Intent intent = new Intent(this, MainActivity.class);
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK); 
startActivity(intent);
```

