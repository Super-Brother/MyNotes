在project的build.gralde里面加入

```
classpath 'com.hujiang.aspectjx:gradle-android-plugin-aspectjx:2.0.10'
```

在app的build.gradle里面加入

```
apply plugin: 'android-aspectjx'

api 'org.aspectj:aspectjrt:1.9.5'

aspectjx {

enabled true

exclude 'com.google','com.taobao'

}
```

