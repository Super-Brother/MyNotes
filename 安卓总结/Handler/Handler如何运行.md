##### Handler角色分配

Handler中存在四种角色。

###### Handler

Handler用来向Looper发送消息，在Looper处理到对应的消息时，Handler在对消息进行具体的处理。上层关键API为handleMessage()，由子类自行实现处理逻辑。

###### Looper

Looper运行在目标线程里，不断从消息队列MessageQueue读取消息，分配给Handler处理。Looper起到连接的作用，将来自不同渠道的消息，聚集在目标线程里处理。也因此Looper需要确保线程唯一。

###### MessageQueue

存储消息对象Message，当Looper向MessageQueue获取消息，或Handler向其插入数据时，决定消息如何提取、如何存储。不仅如此，MessageQueue还维护与Native端的连接，也是解决Looper.loop() 阻塞问题的 Java 端的控制器。

###### Message

Message包含具体的消息数据，在成员变量target中保存了用来发送此消息的Handler引用。因此在消息获得这行时机时，能知道具体由哪一个Handler处理。此外静态成员变量sPool，则维护了消息缓存池以复用。