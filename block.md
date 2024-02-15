# block

### block本质

- block本质也是一个OC对象，它内部也有个isa指针
- block是封装了函数调用以及函数调用环境的OC对象

![image-20240210214835487](/Users/oyzh/Library/Application Support/typora-user-images/image-20240210214835487.png)

### **block底层结构如下图所示：**

![image-20240210214856678](/Users/oyzh/Library/Application Support/typora-user-images/image-20240210214856678.png)





### **block的变量捕获机制**

- auto          可以捕获到block内部        访问方式：值传递
- static          可以捕获到block内部        访问方式：指针传递
- 全局变量    不可以捕获到block内部     访问方式：直接访问





block会捕获self，因为会有隐式的两个参数 (Person *self, SEL _cmd)

```objective-c
-(void)test {
	void(^block)(void) = ^{
		NSLog(@"------%p",self);
	};
	block();
}
```





### **block的类型**

- __NSGlobalBlock__ （ _NSConcreteGlobalBlock ）
- __NSStackBlock__ （ _NSConcreteStackBlock ）
- __NSMallocBlock__ （ _NSConcreteMallocBlock ）

![image-20240211001340288](/Users/oyzh/Library/Application Support/typora-user-images/image-20240211001340288.png)

![image-20240211132744622](/Users/oyzh/Library/Application Support/typora-user-images/image-20240211132744622.png)

 ![image-20240211144933419](/Users/oyzh/Library/Application Support/typora-user-images/image-20240211144933419.png)



### block自动复制到堆上

在ARC环境下，编译器会根据情况自动将栈上的block复制到堆上，比如以下情况

- block作为函数返回值时
- 将block赋值给__strong指针时
- block作为Cocoa API中方法名含有usingBlock的方法参数时
- block作为GCD API的方法参数时 



### 对象类型的auto变量



**当block内部访问了对象类型的auto变量时**

- 如果block是在栈上，将不会对auto变量产生强引用（block自身都会在离开scope时被销毁）

- 如果block被拷贝到堆上

  会调用block内部的copy函数

  copy函数内部会调用_Block_object_assign函数

  _Block_object_assign函数会根据auto变量的修饰符（strong、weak、__unsafe_unretained）做出相应的操作，形成强引用（retain）或者弱引用



- 如果block从堆上移除，会调用block内部的dispose函数

1. dispose函数内部会调用_Block_object_dispose函数
2. _Blocks_object_dispose函数会自动释放引用的auto变量（release）



![image-20240213185026493](/Users/oyzh/Library/Application Support/typora-user-images/image-20240213185026493.png)

 



### __block 修饰变量

- __block可以用于解决block内部无法修改auto变量值的问题
- __block不能修饰全局变量、静态变量（static）
- 编译器会把__block变量包装成一个对象





![image-20240214013140990](/Users/oyzh/Library/Application Support/typora-user-images/image-20240214013140990.png)



修改变量：age->__forwarding->age 来修改值





### __block内存管理

**当block被copy到堆时**

- 会调用block内部的copy函数
- copy函数内部会调用_Block_object_assign函数
- _Block_object_assign函数会对__block变量形成强引用（retain）



当block被copy到堆上时，block捕获的__block变量也会被copy到堆上去

![image-20240214151619166](/Users/oyzh/Library/Application Support/typora-user-images/image-20240214151619166.png)

**当block从堆中移除时**

- 会调用block内部的dispose函数
- dispose函数 



### 对象类型的auto变量，__block变量

**当block在栈上时，对它们都不会产生强引用**



**当block拷贝到堆上时，都会通过copy函数来处理它们**

- block变量（假设变量名叫做a）
- _Block_object_assign((void*)&dst->a, (void*)src->a, 8/*BLOCK_FIELD_IS_BYREF*/);



- 对象类型的auto变量（假设变量名叫做p）
- _Block_object_assign((void*)&dst->p, (void*)src->p, 3/*BLOCK_FIELD_IS_OBJECT*/);



**当block从堆上移除时，都会通过dispose函数来释放它们**

- __block变量（假设变量名叫做a）
- _Block_object_dispose((void*)src->a, 8/*BLOCK_FIELD_IS_BYREF*/);



- 对象类型的auto变量（假设变量名叫做p）
- _Block_object_dispose((void*)src->p, 3/*BLOCK_FIELD_IS_OBJECT*/);



![image-20240215133828818](/Users/oyzh/Library/Application Support/typora-user-images/image-20240215133828818.png)





### 被__block修饰的对象类型

![image-20240215164432276](/Users/oyzh/Library/Application Support/typora-user-images/image-20240215164432276.png)



- 当__block变量在栈上时，不会对指向的对象产生强引用



**当__block变量被copy到堆时**

- 会调用__block变量内部的copy函数
- copy函数内部会调用_Block_object_assign函数
- _Block_object_assign函数会根据所指向对象的修饰符（__strong、__weak、__unsafe_unretained）做出相应的操作，形成强引用（retain）或者弱引用（注意：这里仅限于ARC时会retain，MRC时不会retain）



**如果__block变量从堆上移除**

- 会调用__block变量内部的dispose函数
- dispose函数内部会调用_Block_object_dispose函数
- _Block_object_dispose函数会自动释放指向的对象（release）



### 循环引用







**解决方案** 

- 使用__weak

```
Person *person = [[Person alloc] init];
person.age = 10;
__weak Person *weakPerson = person;
```

- 使用_unsafe_unretained

unretained就是不会强引用，而unfase的原因是，block中person指向的类中的_block被销毁后，block中的person不会置为nil，仍然保存着之前的地址，其实如果访问就会出现野指针错误，因此不安全，而使用__weak，就会将block中的person置为nil。



![image-20240215211533442](/Users/oyzh/Library/Application Support/typora-user-images/image-20240215211533442.png)



- 使用__block解决循环引用（必须要调用block）

```objective-c
__block Person *person = [[Person alloc] init];
perosn.age = 10;
person.block = ^{
	NSlog(@"age is %d",perosn.age);
	person = nil;
};
person.block();
```





![image-20240215232914226](/Users/oyzh/Library/Application Support/typora-user-images/image-20240215232914226.png)



 **MRC下解决循环引用**

MRC下不支持（__weak)

- 使用__unsafe_unretained解决

```objective-c
__unsafe_unretained id weakSelf = self;
self.block = ^{
	NSLog("%p",seakSelf);
};
```



- 使用__block解决

```objective-c
__block id weakSelf = self;
self.block = ^{
	printf("%p",seakSelf);
};
```

