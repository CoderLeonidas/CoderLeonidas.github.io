Mac OS下的IPC


Mac OS下的IPC方式种类很多，大约有下面几种。 
1. Mach API 
2. CFMessagePort 
3. Distributed Objects (DO) NSDistributedNotificationCenter
4. Apple events 
5. UNIX domain sockets 
6. Internet sockets 或者 XPC(NSConnection)
7. Memory Mapping 



还有Pipe，NSTask，消息队列，远程过程调用，通知，信号量和锁设置之类

 

第1种太底层，很少有人用苹果也不推荐 .
第2，3，4种是苹果提供的较为高层的通讯机制 .
第5，6种大家应该都知道，用sockets方法离散度，移植性更好。5和6有一些区别，5使用unix文件系统作为通讯媒介，可以使用unix文件权限系统做通讯限制，它的另外一个名字就叫ipc sokets，6可以做机器间通讯做本地ipc也可以，不过要额外做一些努力。 
第7种适合数据比较大的情况.
 
1. 假如你的进程间通讯不频繁，只是发送一些异步信号，DO是很好的选择，也就是NSDistributedNotificationCenter.
2. 如果你的进程间通讯频繁，但数据量不大，需要响应度高，domain sockets很好.
3. 最后如果进程间通讯频繁，数据流巨大例如音频，图片等，需要real-time级别的,  选择Memory Mapping.
————————————————
版权声明：本文为CSDN博主「problc」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/problc/java/article/details/41253015