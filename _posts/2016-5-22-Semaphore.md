---
layout:     post
title:      "Dispatch Semaphore 细粒度的排他控制"
subtitle:   ""
date:       2016-05-22
author:     "Chaos"
header-img: "img/post-sad.jpg"
tags:
    - IOS
---
**当并行执行的处理更新数据时，很容易产生数据不一致的情况，有时还会出现程序crash的情况，怎样避免这种情况的产生？我们需要更细粒度的排他控制。**

### Dispatch Semaphore
*****

#### 可变数组中添加元素

不考虑顺序，将100000个数据追加到可变数组中：

```
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    
NSMutableArray *array = [NSMutableArray array];
dispatch_group_t group = dispatch_group_create();
for (NSInteger i = 0; i < 100000; i++) {
        dispatch_group_async(group, queue, ^{
            [array addObject:[NSNumber numberWithInteger:i]];
        });
}
    
dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
NSLog(@"%@",  @([array count]));

```

这里采用的是异步并行队列更新NSMutableArray对象，这样由于内存错误导致应用异常结束的概率非常高：

```
TestSemaphore(7246,0x700000094000) malloc: *** error for object 0x7fb8aa803000: pointer being freed was not allocated
*** set a breakpoint in malloc_error_break to debug
(lldb) 

```

避免这种情况的发送，一般的做法是使用串行队列，或者同步队列：

```
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0); NSMutableArray *array = [[NSMutableArray alloc] init];
for(int i = 0; i< 100000; ++i) {
        dispatch_sync(queue, ^{
            [array addObject:[NSNumber numberWithInt:i]];
        });

}
NSLog(@"%@", @([array count]));
    
```

#### 使用Dispatch Semaphore

Dispatch Semaphore是持有计数的信号，该计数是多线程编程中的计数类型信号。类似于在停车场停车的概念，一开始有N个车位，当N等于0时等待，当N大于0时，停一辆车N减一而无需等待。

通过dispatch_semaphore_create函数生成Dispatch Semaphore:

```
dispatch_semaphore_t semaphore = dispath_semaphore_create(1);

```

参数表示计数的初始值，这里初始化为1，意思就是停车场目前空车位数量为1。

```
dispathc_semaphore_wait(semaphore, DISPATCH_TIMER_FOREVER);

```
dispathc_semaphore_wait函数等待Dispatch Semaphore的计数值达到大于0，当满足条件时，对计数进行减法并从dispatch_semaphore_wait函数返回。第二个参数是等待所需时间，DISPATCH_TIMER_FOREVER意味着永久等待。

所以如果我们使用Dispatch Semaphore对可变数组进行操作:

```
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    /*
     *
     *生成Dispatch Semaphore
     Dispatch Semaphore 的计数初始值设定为“1”
     保证可访问 NSMutableArray 类对象的线程
     同时只能有1个
     *
     */
dispatch_semaphore_t semaphore = dispatch_semaphore_create(1) ;
NSMutableArray *array = [NSMutableArray array];
    for(int i = 0; i< 100000; ++i) {
        dispatch_async(queue, ^{
            /*
             *
             *等待Dispatch Semaphore
             *一直等待，直到Dispatch Semaphore的计数值达到大于等于1
             */
            dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER) ;
            /*
             *由于Dispatch Semaphore的计数值达到大于等于1
             *所以将Dispatch Semaphore的计数值减去1
             *dispatch_semaphore_wait 函数执行返回。
             *即执行到此时的
             *Dispatch Semaphore 的计数值恒为0
             *
             *由于可访问NSMutaleArray类对象的线程
             *只有一个
             *因此可安全地进行更新
             *
             */
            [array addObject:[NSNumber numberWithInt:i]];
            /*
             *
             *排他控制处理结束，
             *所以通过dispatch_semaphore_signal函数
             *将Dispatch Semaphore的计数值加1
             *如果有通过dispatch_semaphore_wait函数
             *等待Dispatch Semaphore的计数值增加的线程，
             ★就由最先等待的线程执行。
             */
            dispatch_semaphore_signal(semaphore);
        });
    }

```

在[iOS实时卡顿监控](http://www.tanhao.me/code/151113.html/)中，为了让计算更精确，需要让子线程更及时的获知主线程NSRunLoop的状态变化，采取了使用dispatch_semaphore_t:

```
static void runLoopObserverCallBack(CFRunLoopObserverRef observer, CFRunLoopActivity activity, void *info)
{
    MyClass *object = (__bridge MyClass*)info;
    
    // 记录状态值
    object->activity = activity;
    
    // 发送信号
    dispatch_semaphore_t semaphore = moniotr->semaphore;
    dispatch_semaphore_signal(semaphore);
}

- (void)registerObserver
{
    CFRunLoopObserverContext context = {0,(__bridge void*)self,NULL,NULL};
    CFRunLoopObserverRef observer = CFRunLoopObserverCreate(kCFAllocatorDefault,
                                                            kCFRunLoopAllActivities,
                                                            YES,
                                                            0,
                                                            &runLoopObserverCallBack,
                                                            &context);
    CFRunLoopAddObserver(CFRunLoopGetMain(), observer, kCFRunLoopCommonModes);
    
    // 创建信号
    semaphore = dispatch_semaphore_create(0);
    
    // 在子线程监控时长
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        while (YES)
        {
            // 假定连续5次超时50ms认为卡顿(当然也包含了单次超时250ms)
            long st = dispatch_semaphore_wait(semaphore, dispatch_time(DISPATCH_TIME_NOW, 50*NSEC_PER_MSEC));
            if (st != 0)
            {
                if (activity==kCFRunLoopBeforeSources || activity==kCFRunLoopAfterWaiting)
                {
                    if (++timeoutCount < 5)
                        continue;
                    
                    NSLog(@"好像有点儿卡哦");
                }
            }
            timeoutCount = 0;
        }
    });
}

```




































