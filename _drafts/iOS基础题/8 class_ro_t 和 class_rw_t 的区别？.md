## class_ro_t 和 class_rw_t 的区别？

```

struct class_rw_t {
    // Be warned that Symbolication knows the layout of this structure.
    uint32_t flags;
    uint32_t version;
 
    const class_ro_t *ro;
 
    method_array_t methods;
    property_array_t properties;
    protocol_array_t protocols;
 
    Class firstSubclass;
    Class nextSiblingClass;
 
    char *demangledName;
 
    ...

```

class_rw_t 中包括

method_array_t 方法数组

property_array_t 属性数组

protocol_array_t 代理数组

class_ro_t



```


struct class_ro_t {
    uint32_t flags;
    uint32_t instanceStart;
    uint32_t instanceSize;
#ifdef __LP64__
    uint32_t reserved;
#endif
 
    const uint8_t * ivarLayout;
    
    const char * name;
    method_list_t * baseMethodList;
    protocol_list_t * baseProtocols;
    const ivar_list_t * ivars;
 
    const uint8_t * weakIvarLayout;
    property_list_t *baseProperties;
 
    method_list_t *baseMethods() const {
        return baseMethodList;
    }

```


class_ro_t 中包括

name 类名

method_list_t 方法列表 

property_list_t 属性列表

protocol_list_t 代理列表

ivar_list_t 成员变量列表

等



### 两者区别
class_rw_t | class_ro_t
---|---
rw readwrite | ro readonly
内部信息可读可写的 | 内部信息只读
内部包含的信息来源时runtime时动态添加的，比如分类中的方法会在运行时添加到method_array_t中| 内部为类编译器生成的信息，不可添加和删除



# 参考文章
[Objective-C class_rw_t class_ro_t](https://blog.csdn.net/wangchuanqi256/article/details/88955164)