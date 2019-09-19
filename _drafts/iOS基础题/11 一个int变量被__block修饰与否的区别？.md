## 一个int变量被__block修饰与否的区别？

没有修饰，被block捕获，是值拷贝。
使用__block修饰,会生成一个结构体，复制int的引用地址。达到修改数据。


# 参考文章
[iOS Block 详解](https://imlifengfeng.github.io/article/457/)