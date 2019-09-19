---
title: iOS开发中遇到的同步机制
date: 2017-05-02 20:39:43
tags:
---

​	今天主要是来讨论下，线程同步机制的问题。说到线程同步，一般人可能会想到**“NSLock”**、**“@synchronized”**、**“GCD信号量”**等等，好吧，其实这是我想到的，然而我要说的是，如果在面试中只答到这么几个可是远远不够的。所以我查找了下资料，这才发现原来ios中线程同步的方法可足足有将近10种，且听我娓娓道来。

<!--more-->

各个锁进行1000000此的加锁解锁的空操作时间如下

```objc
OSSpinLock: 46.15 ms
dispatch_semaphore: 56.50 ms
pthread_mutex: 178.28 ms
NSCondition: 193.38 ms
NSLock: 175.02 ms
pthread_mutex(recursive): 172.56 ms
NSRecursiveLock: 157.44 ms
NSConditionLock: 490.04 ms
@synchronized: 371.17 ms
```

### **NSLock**

​	提到NSLock，首先要提另外一个名词叫NSLocking，这是一个协议，主要就定义了俩个方法，一个叫 lock,一个叫unLock。NSLock其实就是确认了NSLocking协议的一个NSObject对象，那么NSLock如何使用呢？其实很简单，就是在你认为可能会发生多线程访问的地方进行lock 操作，在执行完相应代码之后，执行unlock操作。举个简单例子

   ```objc
   -(void)testLock{// 先简单描述下使用场景，现在有一个线程1，有一个线程2，都要访问字符串name，且线程1访问字符串要耗时3秒，线程2此时需要等待。
   	__block NSString *name = @"成焱";
        NSOperationQueue *queue = [[NSOperationQueue alloc]init];
       queue.maxConcurrentOperationCount = 3;
       
       
       NSLock *lock = [NSLock new];
       
       [queue addOperationWithBlock:^{
           NSLog(@"1 将要上锁");
           [lock lock];
           NSLog(@"1 已上锁，访问资源");
           name = @"哇哈哈";
           sleep(3);
           NSLog(@"1 将要解锁");
           [lock unlock];
           NSLog(@"1 已解锁");
       }];
       
       [queue addOperationWithBlock:^{
           sleep(1);//保证此线程后面的方法后调用
           NSLog(@"2 将要上锁");
           [lock lock];
           NSLog(@"2 已上锁，访问资源");
           name = @"康师傅";
           sleep(2);
           NSLog(@"2 将要解锁");
           [lock unlock];
           NSLog(@"2 已解锁");
           
       }];
   }
   ```
​	***打印结果如下***

   ```objc
   2017-04-24 23:06:59.831694 test[1300:102434] 1 将要上锁
   2017-04-24 23:06:59.831718 test[1300:102434] 1 已上锁，访问资源
   2017-04-24 23:07:00.835992 test[1300:102435] 2 将要上锁
   2017-04-24 23:07:02.836242 test[1300:102434] 1 将要解锁
   2017-04-24 23:07:02.836385 test[1300:102434] 1 已解锁
   2017-04-24 23:07:02.836443 test[1300:102435] 2 已上锁，访问资源
   2017-04-24 23:07:04.841407 test[1300:102435] 2 将要解锁
   2017-04-24 23:07:04.841528 test[1300:102435] 2 已解锁
   ```

   	查看控制台的打印输出很明显的看到了，在线程一访问name时，加锁之后，线程2一直在等待，直到线程1释放锁之后，线程2才会去访问name。

### **@synchronize** 

​	想必但凡是开发过一段时间ios程序的同学，一定会对这个关键字不陌生。这个关键字的字面意思就是“同步”。那么它是如何实现同步的呢？

​	该特性允许传入一个NSObject类型的对象，并执行一个block，形如

```objc
@syncronized(obj){
  // do work
}
```

​	网上查询资料获得，这个特性其实是对objc_sync_enter()于objc_sync_exit()的封装，其实际上等价于

```objc
@try{
	  objc_sync_enter(obj);
 }@finally{
 	  objc_sync_exit(obj);
 }
```

​	函数objc_sync_enter()内部实际进行的操作，是对传入的对象，分配递归锁，并存在哈希表中，感兴趣的同学可以参考这篇[blog](http://yulingtianxia.com/blog/2015/11/01/More-than-you-want-to-know-about-synchronized/),在这里我就不展开讨论了。不过下面还是举个简单例子来说明下如何使用这个特性，这里有个地方需要注意就是，***当传入的对象为nil时，将会从代码中移走线程安全***

```objc
- (void)testSynchronized
{
    __block NSString *source = @"资源";   
    dispatch_queue_t global = dispatch_get_global_queue(0, 0);
    
    dispatch_async(global, ^{
        @synchronized (source) {
            NSLog(@"1 将要执行");
            sleep(3);
            NSLog(@"1 执行完毕");
        }
    });
    
    dispatch_async(global, ^{
        sleep(1);//只是为了让这个线程后调用
        @synchronized (source) {
            NSLog(@"2 将要执行");
            sleep(1);
            NSLog(@"2 执行完毕");
        }
    });
    
}
```

​	***打印结果如下***

```objc
2017-04-24 23:43:01.933835 test[1589:154385] 1 将要执行
2017-04-24 23:43:04.938447 test[1589:154385] 1 执行完毕
2017-04-24 23:43:04.938782 test[1589:154386] 2 将要执行
2017-04-24 23:43:05.942177 test[1589:154386] 2 执行完毕
```
### **信号量**

​	GCD的信号量机制，通过消耗信号的方式，控制线程同步

``````objc
- (void)testSemaphore
{
    dispatch_queue_t global = dispatch_get_global_queue(0, 0);
    
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);
    dispatch_async(global, ^{
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
        NSLog(@"1 将要执行");
        sleep(3);
        NSLog(@"1 执行完毕");
        dispatch_semaphore_signal(semaphore);
    });

    dispatch_async(global, ^{
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
        NSLog(@"2 将要执行");
        sleep(3);
        NSLog(@"2 执行完毕");
        dispatch_semaphore_signal(semaphore);
        
    });
}
``````
   	***打印结果如下***

   ```objc
   2017-04-26 22:32:17.563019 test[914:142382] 1 将要执行
   2017-04-26 22:32:20.568142 test[914:142382] 1 执行完毕
   2017-04-26 22:32:20.568336 test[914:142383] 2 将要执行
   2017-04-26 22:32:23.573598 test[914:142383] 2 执行完毕
   ```

### **NSConditionLock**

​	条件锁，当满足某种条件时，才会尝试获取锁，利用该特性，可以人为干预线程执行的依赖顺序，参见如下代码

```objc
- (void)testConditionLock
{
    int condition = 1;
    
    // 当满足条件时，如果锁空闲，可以获取到锁
    NSConditionLock *conditionLock = [[NSConditionLock alloc]initWithCondition:condition];
    
    dispatch_queue_t global = dispatch_get_global_queue(0, 0);
    dispatch_async(global, ^{
        BOOL islocked = [conditionLock tryLockWhenCondition:1];
        NSLog(@"线程1 要执行");
        sleep(2);
        NSLog(@"线程1 执行完毕");
        if (islocked) {
            [conditionLock unlockWithCondition:3];
        }
    });
    dispatch_async(global, ^{
        BOOL isLocked = [conditionLock lockWhenCondition:2 beforeDate:[NSDate dateWithTimeIntervalSinceNow:10]];
        NSLog(@"线程2 要执行");
        sleep(2);
        NSLog(@"线程2 执行完毕");
        if (isLocked) {
            [conditionLock unlockWithCondition:1];
        }
    });
    dispatch_async(global, ^{
        BOOL isLocked = [conditionLock tryLockWhenCondition:3];
        NSLog(@"线程3 要执行");
        sleep(3);
        NSLog(@"线程3 执行完毕");
        if (isLocked) {
            NSLog(@"加锁了");
            [conditionLock unlockWithCondition:10];
        }
    });
    
}
```

​	***打印结果如下***

```objc
2017-04-26 23:11:56.343532 test[1092:199052] 线程1 要执行
2017-04-26 23:11:56.343546 test[1092:199054] 线程3 要执行
2017-04-26 23:11:58.348745 test[1092:199052] 线程1 执行完毕
2017-04-26 23:11:59.348618 test[1092:199054] 线程3 执行完毕
2017-04-26 23:12:06.348449 test[1092:199053] 线程2 要执行
2017-04-26 23:12:08.353647 test[1092:199053] 线程2 执行完毕
```
### **dispatch_barrier_async()与dispatch_barrier_sync()**

​	GCD提供了线程顺序控制的一个函数，假设有5个任务要执行，需要前俩个并发执行，执行完成之后执行第三个任务，等第三个执行完成才可以执行第四个和第五个任务，这个时候就可以考虑使用dispath_barrier_async()函数，具体dispatch_barrier_asyn与dispatch_barrier_sync有什么区别的话，稍后再说，请看下面这个例子

```objc
- (void)testBarrierAsyncAndSync
{
    /// 创建一个并发执行的队列
    dispatch_queue_t global = dispatch_queue_create("com.demo.chengyan", DISPATCH_QUEUE_CONCURRENT);
    
    dispatch_async(global, ^{
        NSLog(@"任务1");
    });
    
    dispatch_async(global, ^{
        NSLog(@"任务2");
    });
    
    
    dispatch_barrier_async(global, ^{
        sleep(3);
        NSLog(@"任务3");
    });
    NSLog(@"---------------");
    
    dispatch_async(global, ^{
        NSLog(@"任务4");
    });
    
    dispatch_async(global, ^{
        NSLog(@"任务5");
    });
}
```

​	***打印结果如下***

```objc
2017-04-26 23:43:08.653034 test[1214:243421] ---------------
2017-04-26 23:43:08.653138 test[1214:243447] 任务1
2017-04-26 23:43:08.653151 test[1214:243448] 任务2
2017-04-26 23:43:11.654589 test[1214:243447] 任务3
2017-04-26 23:43:11.654715 test[1214:243447] 任务4
2017-04-26 23:43:11.654728 test[1214:243448] 任务5
```

​	可以看到由于是通过async的方式添加到队列中的，所以没有阻塞主线程，-----------------被最先执行了。同时注意到在任务3中，沉睡了3秒，而任务4和5都在等任务3执行完之后，才开始执行的。

​	那么接下来再看下sync的方式执行barrier会怎么样？

```objc
- (void)testBarrierAsyncAndSync
{
    /// 创建一个并发执行的队列
    dispatch_queue_t global = dispatch_queue_create("com.demo.chengyan", DISPATCH_QUEUE_CONCURRENT);
    
    dispatch_async(global, ^{
        NSLog(@"任务1");
    });
    
    dispatch_async(global, ^{
        NSLog(@"任务2");
    });
    
    
    dispatch_barrier_sync(global, ^{
        sleep(3);
        NSLog(@"任务3");
    });
    NSLog(@"---------------");
    
    dispatch_async(global, ^{
        NSLog(@"任务4");
    });
    
    dispatch_async(global, ^{
        NSLog(@"任务5");
    });
}
```

​	***打印结果如下***

```
2017-04-26 23:53:57.275541 test[1249:258152] 任务1
2017-04-26 23:53:57.275542 test[1249:258154] 任务2
2017-04-26 23:54:00.280724 test[1249:258129] 任务3
2017-04-26 23:54:00.280832 test[1249:258129] ---------------
2017-04-26 23:54:00.280944 test[1249:258154] 任务4
2017-04-26 23:54:00.281000 test[1249:258152] 任务5
```

​	可以看到由于sync的方式，阻塞了主线程的操作，导致主线程后面的打印必须要等任务3完成之后才会执行。所以尽量不要用这种方式在主线程调用。防止卡到ui

### **GCD的串行队列实际上也是可以起到锁的作用（略）**

### **os_unfair_lock（系统非公平锁）**

​	在IOS10，MacOS10.12之后，苹果新提供的锁，用来替代OSSPinLock,根据官方文档说明，该锁解决了OSSPinLock的优先级反转问题，主要是通过该锁上携带的值以及它持有线程的所有权信息，系统可以以此做出相应的策略，来解决优先级反转的问题。就像它的名字一样，这是个非公平锁。

​	使用此锁，需要注意的是

>1. ***unlock和lock操作必须得在同一个线程中，如果在不同的线程中解锁，将会导致线程直接crash。***
>
>2. ***该锁决不能通过shared或者mutiplay_mapped memory的方式，在多线程或者多进程中访问。因为该锁的实现，依赖于该锁的值和所在的进程。***
>
>   ***该锁主要是为了替代废弃的OSSPinLock，但是它在争夺资源的时候，不是靠自旋，而是在内核上等待唤醒。***
>
>3. ***Mac系统开发要在<u>10.12</u>之后只用，iOS需要在<u>iOS10</u>以后才能使用***

​       	最后要说明的是，此锁就像它的名字一样，是非公平的，具体啥意思呢？举个例子，解开锁的消费者，存在一种可能立即又重新获得了锁，导致那些在休眠中等待的不能被唤醒。这可能是处于性能的考虑，但是确实有一种可能导致等待者处于饥饿状态。

​	下面的代码简要演示下如何使用该锁

```objc
#import <os/lock.h>
- (void)testUnfairlock
{
    os_unfair_lock_t unfairlock = &OS_UNFAIR_LOCK_INIT;
    
    dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
    
    dispatch_async(queue, ^{
        sleep(1);
        os_unfair_lock_lock(unfairlock);
        NSLog(@"线程1 将要执行");
        sleep(3);
        NSLog(@"线程1 执行结束");
        os_unfair_lock_unlock(unfairlock);
    });
    
    dispatch_async(dispatch_get_main_queue(), ^{
        os_unfair_lock_lock(unfairlock);
        NSLog(@"线程2 将要执行");
        sleep(2);
        NSLog(@"线程2 执行结束");
        os_unfair_lock_unlock(unfairlock);
    });
    
}
```

​	***打印结果如下***

```objc
2017-04-28 00:14:11.692782 test[892:121163] 线程2 将要执行
2017-04-28 00:14:13.698007 test[892:121163] 线程2 执行结束
2017-04-28 00:14:13.698230 test[892:121192] 线程1 将要执行
2017-04-28 00:14:16.700588 test[892:121192] 线程1 执行结束
```
### **pthread_mutex_t（互斥锁）**

​	互斥锁和自旋锁的区别，主要就在于，当获取锁失败之后，自旋锁会一直轮询，而互斥锁会轮询大概一秒之后，进入休眠，等待唤醒，此外互斥锁，是有队列概念的，有一个等待队列，依次唤醒等待者。

​	互斥锁和NSLock的区别则在于互斥锁trylock返回正确时返回0，错误时返回错误值，而NSLock则只会返回NO和YES。

​	此外互斥锁也是性能比较高的锁。使用方式的话，如下

```objc
- (void)testPthreadMutex
{
    static pthread_mutex_t plock;
    pthread_mutex_init(&plock, NULL);
    
    dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
    
    dispatch_async(queue, ^{
        sleep(1);
        pthread_mutex_lock(&plock);
        NSLog(@"线程1 将要执行");
        sleep(3);
        NSLog(@"线程1 执行结束");
        pthread_mutex_unlock(&plock);
    });
    
    dispatch_async(dispatch_get_main_queue(), ^{
        int code = pthread_mutex_lock(&plock);
        NSLog(@"线程2 将要执行,code = %d",code);
        sleep(2);
        NSLog(@"线程2 执行结束");
        pthread_mutex_unlock(&plock);
    });
    
//    pthread_mutex_destroy(&plock);
   
}
```

​	***执行结果如下***

```objc
2017-04-28 00:14:11.692782 test[892:121163] 线程2 将要执行
2017-04-28 00:14:13.698007 test[892:121163] 线程2 执行结束
2017-04-28 00:14:13.698230 test[892:121192] 线程1 将要执行
2017-04-28 00:14:16.700588 test[892:121192] 线程1 执行结束
```



### **NSRecursiveLock(递归锁)**

​	此锁和NSLock的区别主要在于内部实现原理的不同，NSLock内部封装的pthread_mutex_t类型为PTHREAD_MUTEX_TIMED_NP，而NSRecursiveLock的类型为PTHREAD_MUTEX_RECURSIVE_NP，此外还有PTHREAD_MUTEX_ERRORCHECK_NP（检错锁）、PTHREAD_MUTEX_ADAPTIVE_NP（适应锁）。

​	递归锁，允许同一个线程对同一个锁成功获得多次，并通过多次unlock解锁。如果是不同线程请求，则在加锁线程解锁时重新竞争。

​	该锁的使用场景主要是在递归函数内部调用。

​	按照惯例还是看个代码来说明下如何使用。

```objc
- (void)testRecursiveLock
{
    NSRecursiveLock *recursive = [[NSRecursiveLock alloc]init];
//
    dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
    
    dispatch_async(queue, ^{
        
        [self getFactorial:10 cursive:recursive];
        
    });


}

- (int)getFactorial:(int)n cursive:(NSRecursiveLock *)lock
{
    int result = 0;
    NSLog(@"加锁");
    [lock lock];
    
    if (n <= 0) {
        result = 1;
    }else{
        result = [self getFactorial:n-1 cursive:lock] * n;
    }
    
    NSLog(@"result =%d",result);
    
    [lock unlock];
    NSLog(@"解锁");
    return result;
}
```

​	***打印结果如下***

```objc
2017-05-02 00:43:08.241134 test[1194:551965] 加锁
2017-05-02 00:43:08.241902 test[1194:551965] 加锁
2017-05-02 00:43:08.241920 test[1194:551965] 加锁
2017-05-02 00:43:08.241934 test[1194:551965] 加锁
2017-05-02 00:43:08.241942 test[1194:551965] 加锁
2017-05-02 00:43:08.241950 test[1194:551965] 加锁
2017-05-02 00:43:08.241957 test[1194:551965] 加锁
2017-05-02 00:43:08.241974 test[1194:551965] 加锁
2017-05-02 00:43:08.241991 test[1194:551965] 加锁
2017-05-02 00:43:08.241998 test[1194:551965] 加锁
2017-05-02 00:43:08.242005 test[1194:551965] 加锁
2017-05-02 00:43:08.242023 test[1194:551965] result =1
2017-05-02 00:43:08.242030 test[1194:551965] 解锁
2017-05-02 00:43:08.242036 test[1194:551965] result =1
2017-05-02 00:43:08.242053 test[1194:551965] 解锁
2017-05-02 00:43:08.242095 test[1194:551965] result =2
2017-05-02 00:43:08.242110 test[1194:551965] 解锁
2017-05-02 00:43:08.242128 test[1194:551965] result =6
2017-05-02 00:43:08.242136 test[1194:551965] 解锁
2017-05-02 00:43:08.242143 test[1194:551965] result =24
2017-05-02 00:43:08.242151 test[1194:551965] 解锁
2017-05-02 00:43:08.242293 test[1194:551965] result =120
2017-05-02 00:43:08.242309 test[1194:551965] 解锁
2017-05-02 00:43:08.242317 test[1194:551965] result =720
2017-05-02 00:43:08.242324 test[1194:551965] 解锁
2017-05-02 00:43:08.242331 test[1194:551965] result =5040
2017-05-02 00:43:08.242338 test[1194:551965] 解锁
2017-05-02 00:43:08.242344 test[1194:551965] result =40320
2017-05-02 00:43:08.242351 test[1194:551965] 解锁
2017-05-02 00:43:08.242357 test[1194:551965] result =362880
2017-05-02 00:43:08.242365 test[1194:551965] 解锁
2017-05-02 00:43:08.242371 test[1194:551965] result =3628800
2017-05-02 00:43:08.242378 test[1194:551965] 解锁
```

### **NSCondition**

​	NSCondition 的对象实际上作为一个锁和一个线程检查器：锁主要为了当检测条件时保护数据源，执行条件引发的任务；线程检查器主要是根据条件决定是否继续运行线程，即线程是否被阻塞。

​	使用方式主要包括，lock，unlock, wait, signal,四个方法，分别指获取锁、放开锁、等待信号、发送信号。同样用一段示例代码来看下它的用法

```objc
- (void)testCondition
{
    NSCondition *condition = [NSCondition new];
    NSMutableArray *ops = [NSMutableArray array];
    
    dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
    
    dispatch_async(queue, ^{
        [condition lock];
        NSLog(@"1 将要上锁");
        while (ops.count == 0) {
            NSLog(@"1 等待");
            [condition wait];
        }
        NSLog(@"1 移除第一个元素");
        [ops removeObjectAtIndex:0];
        
        NSLog(@"1 将要解锁");
        [condition unlock];
    });
    
    dispatch_async(queue, ^{
        NSLog(@"2 将要上锁");
        [condition lock];
        NSLog(@"2 生产一个对象");
        [ops addObject:[NSObject new]];
        NSLog(@"2 发送信号");
        [condition signal];
        NSLog(@"2 将要解锁");
        [condition unlock];
    });
}
```

​	***打印结果如下***

```objc
2017-05-02 01:04:28.327316 test[1240:582713] 2 将要上锁
2017-05-02 01:04:28.327329 test[1240:582712] 1 将要上锁
2017-05-02 01:04:28.328121 test[1240:582712] 1 等待
2017-05-02 01:04:28.328149 test[1240:582713] 2 生产一个对象
2017-05-02 01:04:28.328166 test[1240:582713] 2 发送信号
2017-05-02 01:04:28.328182 test[1240:582713] 2 将要解锁
2017-05-02 01:04:28.328233 test[1240:582712] 1 移除第一个元素
2017-05-02 01:04:28.328261 test[1240:582712] 1 将要解锁
```

至此，ios里面的大部分同步方法我们已经基本了解了，剩下的就是在实践中选择合适的方法进行应用了。

### 附录

​	如果有想查看DEMO的同学，可以点击[它](https://github.com/TripleFireMan/SynDemo.git)来下载DEMO，查看。

### 参考资料

---

* [iOS开发中的8种锁](http://www.jianshu.com/p/8b8a01dd6356)
* [深入理解ios开发中的锁](http://blog.csdn.net/super_man_ww/article/details/52753802)
* [不再安全的OSSPinLock](http://blog.ibireme.com/2016/01/16/spinlock_is_unsafe_in_ios/)
* [Threading Programming Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Multithreading/ThreadSafety/ThreadSafety.html#//apple_ref/doc/uid/10000057i-CH8-SW1)
* [Linux线程-互斥锁pthread_mutex_t](http://blog.csdn.net/zmxiangde_88/article/details/7998458)
* [iOS NSCondition详解](http://www.jianshu.com/p/5d20c15ae690)