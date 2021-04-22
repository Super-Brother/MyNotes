卸载要点

在ubuntu上卸载anaconda的步骤 ：

1）删除整个anaconda目录：
  由于Anaconda的安装文件都包含在一个目录中，所以直接将该目录删除即可。到包含整个anaconda目录的文件夹下，删除整个Anaconda目录：

```
rm -rf anaconda文件夹名
```

2）建议——清理下.bashrc中的Anaconda路径：


  1.到根目录下，打开终端并输入：

```
sudo gedit ~/.bashrc
```


  2.在.bashrc文件末尾用#号注释掉之前添加的路径(或直接删除)：

```
#export PATH=/home/lq/anaconda3/bin:$PATH
```


   保存并关闭文件

   保存并关闭文件

  3.使其立即生效，在终端执行：

```
source ~/.bashrc
```

  4.关闭终端，然后再重启一个新的终端，这一步很重要，不然在原终端上还是绑定有anaconda.