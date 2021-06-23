---
title: GCD疑问
date: 2021-06-22 15:20:34
tags:
categories: iOS
---
参考链接：

[GCD 之 函数与队列](https://juejin.cn/post/6949588322574401567)

## 总结
| 函数\队列 | 串行队列 | 并发队列 | 主队列 | 全局并发队列 |
| ----    |  ----   |  ----   |  ----   |  ----   |
| 同步函数 | 顺序执行，不开辟线程 | 顺序执行，不开辟线程 | 死锁(???) | 顺序执行，不开辟线程 |
| 异步函数 | 顺序执行，开辟线程 | 乱序执行，开辟线程 | 顺序执行，不开辟线程 | 乱序执行，开辟线程 |

### 结论疑点：

同步函数+主队列 ---> 一定会死锁？
<!--more-->

验证：

#### 1、dispatch_synch函数执行在主线程

``` C
NSLog(@"testGCD1_同步_主队列:%@", [NSThread currentThread]);
NSLog(@"testGCD1_同步_主队列: 1_%@", [NSThread currentThread]);
dispatch_sync(dispatch_get_main_queue(), ^{
   NSLog(@"testGCD1_同步_主队列: 2_%@", [NSThread currentThread]);
   sleep(2);
});
NSLog(@"testGCD1_同步_主队列: 3_%@", [NSThread currentThread]);
```
输出结果：**死锁**

``` JSON
2021-06-10 14:48:03.115415+0800 [48163:3177135] testGCD1_同步_主队列:<NSThread: 0x6000016291c0>{number = 1, name = main}
2021-06-10 14:48:03.115560+0800 [48163:3177135] testGCD1_同步_主队列: 1_<NSThread: 0x6000016291c0>{number = 1, name = main}
```
#### 2、dispatch_sync函数执行在非主线程

``` C
NSLog(@"testGCD1_同步_主队列:%@", [NSThread currentThread]);
dispatch_async(dispatch_get_global_queue(0, 0), ^{
   NSLog(@"testGCD1_同步_主队列: 1_%@", [NSThread currentThread]);
   dispatch_sync(dispatch_get_main_queue(), ^{
       NSLog(@"testGCD1_同步_主队列: 2_%@", [NSThread currentThread]);
       sleep(2);
   });
   NSLog(@"testGCD1_同步_主队列: 3_%@", [NSThread currentThread]);
});
```
输出结果：**顺序执行**

``` JSON
2021-06-10 14:54:09.405571+0800 [50303:3187114] testGCD1_同步_主队列:<NSThread: 0x6000015d8340>{number = 1, name = main}
2021-06-10 14:54:09.405732+0800 [50303:3187504] testGCD1_同步_主队列: 1_<NSThread: 0x6000015a0640>{number = 10, name = (null)}
2021-06-10 14:54:09.447266+0800 [50303:3187114] testGCD1_同步_主队列: 2_<NSThread: 0x6000015d8340>{number = 1, name = main}
2021-06-10 14:54:11.447552+0800 [50303:3187504] testGCD1_同步_主队列: 3_<NSThread: 0x6000015a0640>{number = 10, name = (null)}
```
总结：

同步函数(主线程)+主队列  ---> **死锁**

同步函数(非主线程)+主队列  ---> **顺序执行，不开辟新线程，非主线程/主线程切换**

| 函数\队列 | 串行队列 | 并发队列 | 主队列 | 全局并发队列 |
| ---- | ---- | ---- | ---- | ---- |
| 同步函数(主线程) | 顺序执行，不开辟线程 | 顺序执行，不开辟线程 | 死锁 | 顺序执行，不开辟线程 |
| 异步函数(主线程) | 顺序执行，开辟线程 | 乱序执行，开辟线程 | 顺序执行，不开辟线程 | 乱序执行，开辟线程 |

