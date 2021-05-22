1、第一步首先将手机与电脑通过数据线连接，手机开启开发模式，打开USB调试模式。

2、确定电脑是否成功连接手机，电脑快捷键Win+R，输入cmd，输入指令`adb devices`，当连接成功时会出现连接的设备名称（adb指令的安装不是本文的讲述要点，不会的请自行百度，网上教材很多），电脑与手机通过数据线连接成功后，就可以获取apk文件了。

3、执行 `adb shell pm list packages`指令查看该手机所有安装包的包名，找到你要获取的apk的包名（可以根据英文意理解）

4、执行`adb shell pm path + 包名`，例如：adb shell pm path com.cmsz.account，获取该安装apk的路径

5、执行adb pull+安装包路径  输出路径，例如：`adb pull /data/app/com.cmsz.account-2/base.apk C:\Users\Administrator\Desktop`，将该apk文件保存输出到电脑桌面上。

