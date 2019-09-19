## Autoreleasepool所使用的数据结构是什么？AutoreleasePoolPage结构体了解么？


### Autoreleasepool所使用的数据结构是什么

由AutoreleasePoolPage组成的双向链表

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g74bxyqhr7j30rs0a3di5.jpg)

### AutoreleasePoolPage结构体了解么？

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g74byn1lddj30bk06b3yw.jpg)

- id *next: 指向了下一个能存放autorelease对象地址的区域

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g74c0pialzj30i8085aah.jpg)

- parent: 父节点 指向前一个page
- child: 子节点 指向下一个page 
- POOL_BOUNDARY: 是一个边界对象nil,哨兵对象,用来区别每个page即每个 AutoreleasePoolPage边界
- AutoreleasePool是按线程一一对应的（结构中的thread指针指向当前线程）。
 
- AutoreleasePoolPage每个对象会开辟4096字节内存（也就是虚拟内存一页的大小），除了上面的实例变量所占空间，剩下的空间全部用来储存autorelease对象的地址。

- AutoreleasePoolPage::push()
调用push方法会将一个POOL_BOUNDARY入栈，并且返回其存放的内存地址.
push就是压栈的操作,先加入边界对象,然后添加person1对象,然后是person2对象...以此类推

- AutoreleasePoolPage::pop(ctxt);
调用pop方法时传入一个POOL_BOUNDARY的内存地址，会从最后一个入栈的对象开始发送release消息，直到遇到这个POOL_BOUNDARY(因为是双向链表,所以可以向上寻找)

### Autorelease对象释放时机

> Autorelease对象是在当前的runloop每次运行结束和开始休眠之前，而它能够释放的原因是系统在每个runloop迭代中都加入了自动释放池Push和Pop


# 参考文章

[iOS面试题：Autoreleasepool所使用的数据结构是什么？AutoreleasePoolPage结构体了解么？](https://www.jianshu.com/p/0afda1f23782)
[]()


