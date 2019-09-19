## 有哪些场景是NSOperation比GCD更容易实现的？（或是NSOperation优于GCD的几点，知道多少说多少）

GCD是基于c的底层api，NSOperation属于object-c类。ios 首先引入的是NSOperation，IOS4之后引入了GCD和NSOperationQueue并且其内部是用gcd实现的。

相对于GCD：
1，NSOperation拥有更多的函数可用，具体查看api。
2，在NSOperationQueue中，可以建立各个NSOperation之间的依赖关系。
3，有kvo，可以监测operation是否正在执行（isExecuted）、是否结束（isFinished），是否取消（isCanceld）。
4，NSOperationQueue可以方便的管理并发、NSOperation之间的优先级。
GCD主要与block结合使用。代码简洁高效。
  GCD也可以实现复杂的多线程应用，主要是建立个个线程时间的依赖关系这类的情况，但是需要自己实现相比NSOperation要复杂。
具体使用哪个，依需求而定。 从个人使用的感觉来看，比较合适的用法是：除了依赖关系尽量使用GCD，因为苹果专门为GCD做了性能上面的优化。

