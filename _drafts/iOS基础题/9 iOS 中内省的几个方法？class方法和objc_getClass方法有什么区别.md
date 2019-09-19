## iOS 中内省的几个方法？class方法和objc_getClass方法有什么区别?

### iOS 中内省的几个方法

- 判断类型:

`-(BOOL) isKindOfClass:`
> 判断是否是这个类或者这个类的子类的实例

`-(BOOL) isMemberOfClass:` 
> 判断是否是这个类的实例

- 判断对象or类是否有这个方法

`-(BOOL) respondsToSelector:`                      
> 判读实例是否有这样方法

`+(BOOL) instancesRespondToSelector:`

> 判断类是否有这个方法


### class方法和objc_getClass方法有什么区别?

object_getClass:获得的是isa的指向 

self.class:当self是实例对象的时候，返回的是类对象，否则则返回自身。



# 参考文章

[class和object_getClass方法区别](https://www.cnblogs.com/lybSkill/p/10186663.html)