## 在运行时创建类的方法objc_allocateClassPair的方法名尾部为什么是pair（成对的意思）？


### objc_allocateClassPair
> Creates a new class and metaclass.

> You can get a pointer to the new metaclass by calling object_getClass(newClass).
To create a new class, start by calling objc_allocateClassPair. Then set the class's attributes with functions like class_addMethod and class_addIvar. When you are done building the class, call objc_registerClassPair. The new class is now ready for use.
Instance methods and instance variables should be added to the class itself. Class methods should be added to the metaclass.



# 参考文章

[详解Objective-C的meta-class](https://blog.csdn.net/windyitian/article/details/19810875)
