# Category

一个类只有一个类对象

分类的类方法和对象方法是在运行时进行合并（通过Runtime，动态合并）



 

```
xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc Person+Test.m
```

  生成对应的cpp文件，并找到_category_t结构体

```c++
struct _category_t {
	const char *name;
	struct _class_t *cls;
	const struct _method_list_t *instance_methods;
	const struct _method_list_t *class_methods;
	const struct _protocol_list_t *protocols;
	const struct _prop_list_t *properties;
};
```



在Runtime源码中，有函数remothodizeClass(cls) 和 remothodizeClass(cls->ISA())，可以看出既传入了类对象，也穿入了元类对象。



**最后面参与编译的分类放在最前面,最优先调用最后编译的分类的方法**



类扩展在编译时就已经合并到类中

而分类则是在运行时动态合并。



**load方法什么时候调用，能否继承？**

**+load**方法会在Runtime加载类、分类的时候调用



**在调用load方法时，原来类和分类都有load方法，为什么分类不会覆盖原来类的load方法？**

指针指向load方法的内存地址进行调用(直接调用)

**而常规方法分类会覆盖原来类是因为此处方法调用是通过消息机制，也就是objc_msgsend**



**load方法调用顺序**

1.先调用类的load

- 按照编译先后顺序调用（先编译，先调用）
- 调用子类的load方法之前会先调用父类的load

 

2.再调用分类的load

- 按照编译先后顺序调用（先编译，先调用）

 



**initialize方法**

initialize方法会在类第一次接受到消息时调用

 initialize方法只调用一个



**initialize调用顺序**

- 先调用父类的+initialize，再调用子类的+initialize
- （先初始化父类，再初始化子类，每个类只会初始化一次） 





 **initialize和load的区别**

- initialize是通过objc_msgSend进行调用的，所以具有以下特点
- 如果子类没有实现initialize，会调用父类的initialize（所以父类的initialize可能会被调用多次）
- 如果分类实现了initialize，就会覆盖类本身的initialize调用
- 而load是根据函数地址直接调用
- load是在Runtime加载类、分类的时候调用（只会调用1次）
- initialize是类在第一次接收到消息的时候调用，每一个类只会initialize一次（父类的initialize可能会被调用一次，这是因为有的子类自身没有initialize方法，所以就调用父类的）
- 





**在分类中可以添加属性**

如果是在类声明中添加这些属性：

- 生成下划线变量（成员变量）
- 生成set、get方法声明
- 生成set、get方法实现



如果是在分类中添加：

- 只生成set、get方法的声明





**关联对象**

通过关联对象向分类中添加属性

```objective-c
 objc_setAssociatedObject(id  _Nonnull object, const void * _Nonnull key, id  _Nullable value, objc_AssociationPolicy policy)
```



![image-20240207000707733](/Users/oyzh/Library/Application Support/typora-user-images/image-20240207000707733.png)



```objective-c
//
//  Person+Test.m
//  Category的成员变量
//
//  Created by oyzh on 2024/2/2.
//

#import "Person+Test.h"
#import <objc/runtime.h>

@implementation Person (Test)

static const char NameKey;
static const char WeightKey;

-(void)setName:(NSString *)name {
    objc_setAssociatedObject(self, &NameKey, name, OBJC_ASSOCIATION_COPY_NONATOMIC);
    
}

-(NSString *)name {
    return objc_getAssociatedObject(self, &NameKey);
}

-(void)setWeight:(int)weight {
    objc_setAssociatedObject(self, &WeightKey, @(weight), OBJC_ASSOCIATION_COPY_NONATOMIC);
}

-(int)weight {
    return [objc_getAssociatedObject(self, &WeightKey) intValue];
}
@end
```



**get方法的隐式参数**

get方法有两个隐式参数：self 和  __cmd(_ _cmd == @selector(name))





**关联对象的原理**

实现关联对象技术的核心对象有

- AssociationsManager

- AssociationsHashMap

-  ObjectAssociationMap

- ObjcAssociation

```
objc_setAssociatedObject(id  _Nonnull object, const void * _Nonnull key, id  _Nullable value, objc_AssociationPolicy policy)
```

其中 value 和 policy 存储在 objcAssociation中



- 关联对象并不是存储在被关联对象本身内存中
- 关联对象存储在全局统一的一个AssociationsHashMap（AssociationsManager）中 
- 设置关联对象为nil，就相当于移除关联对象









关联对象

block

Runtime

Runloop

多线程

内存管理

性能优化

架构设计

