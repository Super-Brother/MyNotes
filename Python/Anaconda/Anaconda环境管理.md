## Anaconda环境的创建

```
conda create -n py3 python=3.5
```

其中py3表示创建环境的名字，后面python=3.5表示创建的版本。

```
conda create -n py3 python=3.5 numpy pandas
```

这个是在创建环境的时候同时安装包



## Anaconda环境的激活

### 在 OSX/Linux 上

```
# 首次使用这个
source activate py3
# 再次使用下面这个
conda activate py3
```

py3为环境名，上述表示激活py3

### windows下

```
activate py3
```



## Anaconda环境的管理

列出所有环境

```
conda env list
```

删除环境

```
conda env remove -n py3
```

上述表示删除环境名为py3的环境



## Anaconda环境下安装包

```
conda install numpy
```

