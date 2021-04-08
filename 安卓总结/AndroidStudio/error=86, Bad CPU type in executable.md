最近在维护一个**N**久的项目时，发现在**mac**升级系统为**10.15.5**后（Android Studio 4.0，gradle 2.3.1），编译失败了，报错如下：

`Cannot run program "/Users/xxxx/Android/sdk/build-tools/23.0.1/aapt": error=86, Bad CPU type in executable`

原因是最新版本的**macOS Catalina(10.15.5)**已经不支持**32**位的应用了，只能运行**64**位的应用。

解决方法：升级工程的**buildToolsVersion**，本例中将**23.0.1** 升级成**25.0.3**。

