需要引用此MyModule的项目的setting.gradle文件中做如下修改：

```
include ':app' 
include ':MyModule' 
project(':MyModule').projectDir = new File("../Library", 'MyModule')
```

app的build.gradle文件像往常一样在dependencies中编译即可：

```
compile project(':MyModule')
```