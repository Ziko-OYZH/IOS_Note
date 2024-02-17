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