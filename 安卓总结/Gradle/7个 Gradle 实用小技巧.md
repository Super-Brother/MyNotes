# 1 Gradle依赖树查询

需要查看依赖树，我们常用的查看依赖树的命令为。

```
gradlew app:dependencies
```

官方又推出了Scan工具来帮助我们更加方便地查看依赖树。

在项目根目录位置下运行`gradle build --scan`即可，然后会生成 HTML 格式的分析文件的分析文件。

分析文件会直接上传到Scan官网，命令行最后会给出远程地址。



# 2 使用循环优化Gradle依赖管理

如下所示，我们常常使用ext来管理依赖。

```
dependencies {
    implementation fileTree(include: ['*.jar'], dir: 'libs')
    implementation rootProject.ext.dependencies["appcompat-v7"]
    implementation rootProject.ext.dependencies["cardview-v7"]
    implementation rootProject.ext.dependencies["design"]
    implementation rootProject.ext.dependencies["constraint-layout"]
    annotationProcessor rootProject.ext.dependencies["glide_compiler"]
    ...
}
```

有没有一种好的方式不在 build.gradle 中写这么多的依赖配置？
有，就是使用**循环遍历依赖**。
示例如下，首先添加config.gradle。

```
ext{
    dependencies = [
            // base
            "appcompat-v7": "com.android.support:appcompat-v7:${version["supportLibraryVersion"]}"，
            ...
    ]

    annotationProcessor = [
            "glide_compiler": "com.github.bumptech.glide:compiler:${version["glideVersion"]}",
            ...
    ]

    apiFileDependencies = [
            "launchstarter" :"libs/launchstarter-release-1.0.0.aar"
    ]

    debugImplementationDependencies = [
            "MethodTraceMan": "com.github.zhengcx:MethodTraceMan:1.0.7"
    ]

    ...

    implementationExcludes = [
            "com.android.support.test.espresso:espresso-idling-resource:3.0.2" : [
                    'com.android.support' : 'support-annotations'
            ]
    ]

    ...
}
```

然后在build.gradle中配置如下：

```
apply from config.gradle
...

def implementationDependencies = project.ext.dependencies
def processors = project.ext.annotationProcesso
def implementationExcludes = project.ext.implementationExcludes
dependencies{
    // 处理所有的 xxximplementation 依赖
    implementationDependencies.each { k, v -> implementation v }   
    // 处理 annotationProcessor 依赖
    processors.each { k, v -> annotationProcessor v }
    // 处理所有包含 exclude 的依赖
    implementationExcludes.each { entry ->
        implementation(entry.key) {
            entry.value.each { childEntry ->
                exclude(group: childEntry)
            }
        }
    }
    ...

}
```

这样做的优点在于:
1.后续添加依赖不需要改动build.gradle,直接在config.gradle中添加即可。
2.精简了build.gradle的长度。



# *3* 支持代码提示的Gradle依赖管理

Kotlin + buildSrc：更好的管理Gadle依赖



# *4* Gradle模块化

我们在开发中，引入一些插件时，有时需要在build.gradle中引入一些配置，比如greendao,推送,tinker等。
这些其实是可以封装在相应gradle文件中，然后通过apply from引入。

这样做主要有2个优点：

1.单一职责原则，将greendao的相关配置封装在一个文件里，不与其他文件混淆。
2.精简了build.gradle的代码，同时后续修改数据库相关时不需要修改build.gradle的代码。



# *5* Library模块Gradle代码复用

随着我们项目的越来越大，Library Module也越建越多，每个Module都有自己的build.gradle。
但其实每个build.gradle的内容都差不多，我们能不能将重复的部分封装起来复用？

我们可以做一个 basic 抽取，同样将共有参数/信息提取到basic.gradle 中，每个 moduleapply，这样就是减少了不少代码量。



# *6* 资源文件分包

主要是利用gradle的sourceSets属性。
我们可以将资源文件像代码一样按业务分包,具体操作如下：

**1.新建res_xxx目录 。**
在 main 目录下新建 res_core, res_feed（根据业务模块命名）等目录，在res_core中新建res目录中相同的文件夹如：layout、drawable-xxhdpi、values等。

**2.在gradle中配置res_xx目录。**

```
android {
    //...
    sourceSets {
        main {
            res.srcDirs(
                    'src/main/res',
                    'src/main/res_core',
                    'src/main/res_feed',
            )
        }
    }
}
```

以上就完成了资源文件分包,这样做主要有几点好处。

1.按业务分包查找方便，结构清晰。
2.strings.xml等key-value型文件多人修改可以减少冲突。
3.当删除模块或做组件化改造时资源文件删除或迁移方便，不必像以前一样一个个去找。



# *7* AAR依赖与源码依赖快速切换

当我们的项目中Module越来越多，为了加快编译速度，常常把Module发布成AAR，然后在项目中直接依赖AAR。
但是我们有时候又需要修改AAR，就需要依赖于源码。

所以我们需要一个可以快速地切换依赖AAR与依赖源码的方式。

我们下面举个例子，以retrofit为例。假如我们要修改retrofit的源码，修改步骤如下：
1.首先下载retrofit，可以放到和项目同级的目录,并修改目录名为retrofit-source,以便区分。
2.在settings.gradle文件中添加需要修改的aar库的源码project。

```
include ':retrofit-source'
project(':retrofit-source').projectDir = new File("../retrofit-source")
```

3.替换aar为源码。

build.gradle(android) 脚本中添加替换策略。

```
allprojects {
    repositories {
...
    }

    configurations.all {
        resolutionStrategy {
            dependencySubstitution {
                substitute module( "com.squareup.retrofit2:retrofit") with project(':retofit-source')
            }
        }
    }
}
```

如上几步，就可以比较方便地实现aar依赖与源码依赖间的互换了。
这样做的主要优点在于：
1.不需要修改原有的依赖配置，而是通过全局的配置，利用本地的源码替换掉aar，侵入性低。
2.如果有多个Module依赖于同一个aar，不需要重复修改，只需在根目录build.gradle中修改一处。