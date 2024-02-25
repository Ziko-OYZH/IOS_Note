# Runtime

###  isa详解

从64bit开始，isa需要进行一次位运算，才能计算出真实地址

```assembly
#if __arm64__
# define ISA_MASK  0X00000000FFFFFFFF8ULL	
#elif __x86_64__
# define ISA_MASK  0x00007ffffffffff8ULL
# endif
```



- 在arm64架构之前，isa就是一个普通的指针，存储着Class、Meta-Class对象的内存地址

- 从arm64架构开始，对isa进行了优化，变成了一个公用体（union）结构，还使用位域来存储更多信息

![image-20240217140440404](/Users/oyzh/Library/Application Support/typora-user-images/image-20240217140440404.png)





###  Class结构

![image-20240217223235356](/Users/oyzh/Library/Application Support/typora-user-images/image-20240217223235356.png)



- class_rw_t里面的methods、properties、protocols是二维数组，是可读可写的，包含了类的初始内容、分类的内容



![image-20240217224811673](/Users/oyzh/Library/Application Support/typora-user-images/image-20240217224811673.png)

- class_ro_t里面的baseMethodList、baseProtocols、ivars、baseProperties是一维数组，是只读的，包含了类的初始内容



![image-20240217224842344](/Users/oyzh/Library/Application Support/typora-user-images/image-20240217224842344.png)



### method_t

- method_t是对方法\函数的封装

![image-20240218010521629](/Users/oyzh/Library/Application Support/typora-user-images/image-20240218010521629.png)



- IMP代表函数的具体实现

![image-20240218010747217](/Users/oyzh/Library/Application Support/typora-user-images/image-20240218010747217.png)



- SEL代表方法\函数名，一般叫做选择器，底层结构跟char *类似
  - 可以通过@selector()和sel_registerName()获得
  - 可以通过sel_getName()和NSStringFromSelector()转成字符串
  - 不同类中相同名字的方法，所对应的方法选择器是相同的



### 方法缓存

- class内部结构有个方法缓存(cache_t)，用**散列表**来缓存**曾经调用过的方法**，可以提高方法的查找速度



![image-20240218174813767](/Users/oyzh/Library/Application Support/typora-user-images/image-20240218174813767.png)





![image-20240220144550292](/Users/oyzh/Library/Application Support/typora-user-images/image-20240220144550292.png)



在扩容时会清掉所有hashTable的缓存





### objc_msgSend

OC的方法调用：消息机制，给方法调用者发送消息



**objc_msgSend的执行流程可以分为3大阶段**

- 消息发送
- 动态方法解析
- 消息转发



如果找不到合适的方法调用，报错如下：

```
unrecongnized selector send to instance
```





![image-20240221162852127](/Users/oyzh/Library/Application Support/typora-user-images/image-20240221162852127.png)







### 动态方法解析

![image-20240222144931709](/Users/oyzh/Library/Application Support/typora-user-images/image-20240222144931709.png)



![image-20240222144944985](/Users/oyzh/Library/Application Support/typora-user-images/image-20240222144944985.png)





### 消息转发

![image-20240222161550245](/Users/oyzh/Library/Application Support/typora-user-images/image-20240222161550245.png)



 

### super

**消息接收者仍然是子对象，只不过从父类的类对象中开始搜索方法**

[super message] 的底层实现

- 消息接收这仍然是子类对象
- 从父类的类对象中寻找方法



**class底层**

**谁调用这个方法，就返回谁的类型**

```objective-c
-(Class)class {
	return object_getClass(self);
}
```



 

![image-20240223011304707](/Users/oyzh/Library/Application Support/typora-user-images/image-20240223011304707.png)

所以输出：

- Student
- Person
- Student
- Person

 





### Runtime API



获取isa指向的Class

```
object_getClass(id_NonNuLL);
```



设置isa指向的Class

```
Class object_setClass(id obj, Class cls) 
```



动态创建一个类（参数：父类，类名，额外的内存空间）

```
Class objc_allocateClassPair(Class superclass, const char *name, size_t extraBytes)
```



拷贝实例变量列表（最后需要调用free释放）

```
Ivar *class_copyIvarList(Class cls, unsigned int *outCount)
```



