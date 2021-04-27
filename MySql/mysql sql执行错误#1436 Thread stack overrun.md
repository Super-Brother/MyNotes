错误原因：

thread_stack太小，默认 128K。



在my.cnf的[mysqld]小节中加入下面的配置：
`thread_stack＝256K`
保存，重启mysql服务即可。