## 被weak修饰的对象在被释放的时候会发生什么？是如何实现的？知道sideTable么？里面的结构可以画出来么？

### 被weak修饰的对象在被释放的时候会发生什么？
会被置为nil.

### weak如何实现？

Runtime维护了一个weak表，用于存储指向某个对象的所有weak指针。

weak表其实是一个hash（哈希）表，Key是所指对象的地址，Value是weak指针的地址（这个地址的值是所指对象的地址）数组。

1、初始化时：runtime会调用objc_initWeak函数，初始化一个新的weak指针指向对象的地址。

2、添加引用时：objc_initWeak函数会调用 objc_storeWeak() 函数， objc_storeWeak() 的作用是更新指针指向，创建对应的弱引用表。

3、释放时，调用clearDeallocating函数。clearDeallocating函数首先根据对象地址获取所有weak指针地址的数组，然后遍历这个数组把其中的数据设为nil，最后把这个entry从weak表中删除，最后清理对象的记录。


### sideTable是什么
SideTable 这个结构体主要用于管理对象的引用计数和 weak 表。在 NSObject.mm 中有其数据结构:
```
struct SideTable {
    spinlock_t slock;//保证原子操作的自旋锁
    RefcountMap refcnts;//引用计数的 hash 表
    weak_table_t weak_table;//weak 引用全局 hash 表

    SideTable() {
        memset(&weak_table, 0, sizeof(weak_table));
    }

    ~SideTable() {
        _objc_fatal("Do not delete SideTable.");
    }

    void lock() { slock.lock(); }
    void unlock() { slock.unlock(); }
    void forceReset() { slock.forceReset(); }

    // Address-ordered lock discipline for a pair of side tables.

    template<HaveOld, HaveNew>
    static void lockTwo(SideTable *lock1, SideTable *lock2);
    template<HaveOld, HaveNew>
    static void unlockTwo(SideTable *lock1, SideTable *lock2);
};
```

### 里面的结构是怎样的？



# 参考文章
[iOS 中weak的实现](https://www.jianshu.com/p/dce1bec2199e)

[iOS 底层解析weak的实现原理（包含weak对象的初始化，引用，释放的分析）](http://www.cocoachina.com/articles/18962)

[weak的生命周期：具体实现方法](http://www.cocoachina.com/articles/11990)