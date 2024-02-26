# Runloop

运行循环：在程序运行过程中循环做一些事情



**应用范畴**

- 定时器、PerformSelector
- GCD Async Main Queue
- 事件响应、手势识别、界面刷新
- 网络请求
- Autoreleasepool



**基本作用**

- 保持程序的持续运行
- 处理App中的各种事件（比如触摸事件、定时器事件）
- 节省CPU资源，提高程序性能





### 获取RunLoop对象

**iOS中有2套API来访问和使用RunLoop**

- Foundation：NSRunLoop
- Core Foundation：CFRunLoopRef



获取runloop对象

```objective-c
NSRunLoop *runloop = [NSRunLoop currentRunLoop];
```

```c
CFRunLoopRef runloop2 = CFRunLoopGetCurrent();
```





### RunLoop与线程

- 每条线程都有唯一与之对应的RunLoop对象
- RunLoop保存在一个全局的Dictionary里，线程作为Key，RunLoop作为value

```
runloops[thread] = runloop;
```

- 线程刚创建时并没有RunLoop对象，RunLoop会在第一次获取它时创建

- RunLoop会在线程结束时销毁
- 主线程的RunLoop已经自动获取（创建），子线程默认没有开启RunLoop



### RunLoop相关的类

- CFRunLoopRef
- CFRunLoopModeRef
- CFRunLoopSourceRef
- CFRunLoopTimerRef
- CFRunLoopObserverRef



- CFRunLoopMOdeRef代表RunLoop的运行模式

- 一个RunLoop包含若干个Mode，每个Mode又包含若干个Source0/Source1/Timer/Observer
- RunLoop启动时只能选择其中一个Mode，作为currentMode
- 如果需要切换Mode，只能退出当前loop，再重新选择一个Mode进入
- 不同组的Source0/Source1/Timer/Observer能分隔开来，互不影响

- 如果一个Mode没有Source0/Source1/Timer/ObserverRunLoop会立马退出



### 两种常见的Mode

- kcfRunLoopDefaultMode：App的默认模式，通常主线程是在这个模式下运行

- UITrackingRunLoopMode：界面跟踪 Mode，用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他 Mode 影响



### RunLoop的运行逻辑

**Source0**

- 触摸事件处理
- performSelector:onThread:



**Source1**

- 基于Port的线程间通信
- 系统事件捕捉



**Timers**

- NSTimer
- performSelector:withObject:afterDelay:



**observers**

- 用于监听RunLoop的状态
- UI刷新（BeforeWaiting）
- Autorelease pool（BeforeWaiting）













### RunLoop的运行逻辑

![image-20240226175910792](/Users/oyzh/Library/Application Support/typora-user-images/image-20240226175910792.png)





### RunLoop休眠的实现原理

![image-20240226175934589](/Users/oyzh/Library/Application Support/typora-user-images/image-20240226175934589.png)



### RunLoop的状态

![image-20240226180556061](/Users/oyzh/Library/Application Support/typora-user-images/image-20240226180556061.png)





### RunLoop在实际开发中的应用

- 控制线程生命周期（线程保活）
- 解决NSTimer在滑动时停止工作的问题
- 监控应用卡顿
- 性能优化



