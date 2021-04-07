原因：

view.setVisibility(View.GONE)之后，就停止了对View的渲染，所以就看不到关闭的动画效果了。

解决方法：

监听动画执行结束事件，在动画结束之后再设置view.setVisibility(View.GONE)。

