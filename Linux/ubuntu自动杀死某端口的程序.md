在我们写sh文件的时候，经常需要把原来端口的进程杀死，这里使用了一个run.sh来代替手工的杀死过程，核心还是lsof命令和kill命令：

```shell
pid=$(sudo lsof -i:1080 |grep LISTEN| awk '{print $2}')
if ! ["x$pid" = "x"]; then
    echo "killing process on port 1080..."
    kill -9 $pid
fi
```

我这个例子是杀死1080端口的程序.