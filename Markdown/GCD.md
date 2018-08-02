# GCD
---

>>> 百度百科  
>>> Grand Central Dispatch(GCD) 是 Apple 开发的一个多核编程的较新的解决方法.它主要用于优化应用程序以支持多核处理器以及其他对称多处理系统.它是一个在线程池模式的基础上执行的并发任务.在 Mac OS X 10.6 雪豹中首次推出,也可在 iOS 4 及以上版本使用.

### 优点

1. GCD 可用于多核的并行运算
2. GCD 会自动利用更多的 CPU 内核(比如双核、四核)
3. GCD 会自动管理线程的生命周期(创建线程、调度任务、销毁线程)
4. 只需要告诉 GCD 想要执行什么任务,不需要编写任何线程管理代码

### 基本概念

````
同步执行(sync)
同步添加任务到指定的队列中,在添加的任务执行结束之前,会一直等待,直到队列里面的任务完成之后再继续执行.
只能在当前线程中执行任务,不具备开启新线程的能力.
````

````
异步执行(async)
异步添加任务到指定的队列中,它不会做任何等待,可以继续执行任务.
可以在新的线程中执行任务,具备开启新线程的能力.
````

````
串行队列(Serial Dispatch Queue)
每次只有一个任务被执行,让任务一个接着一个地执行(只开启一个线程,一个任务执行完毕后,再执行下一个任务).
````

````
并发队列(Concurrent Dispatch Queue)
可以让多个任务并发(同时)执行.(可以开启多个线程,并且同时执行任务).
````

### 组合

**同步执行 + 并发队列** : 没有开启新线程,串行执行任务   
**异步执行 + 并发队列** : 有开启新线程,并发执行任务    
**同步执行 + 串行队列** : 没有开启新线程,串行执行任务   
**异步执行 + 串行队列** : 有开启新线程(1条),串行执行任务    
**同步执行 + 主队列** : 主线程调用(死锁卡住不执行) 其他线程调用(没有开启新线程,串行执行任务)    
**异步执行 + 主队列** : 没有开启新线程,串行执行任务   

### 线程通信

````
Code:  
dispatch_async(dispatch_queue_create("gcd_a", DISPATCH_QUEUE_CONCURRENT), ^{
    NSString *gcd_a = @"gcd_a";
    NSLog(@"gcd_a: %@", [NSThread currentThread]);
    dispatch_async(dispatch_get_main_queue(), ^{
        NSLog(@"gcd_main: %@ (from: %@)", [NSThread currentThread], gcd_a);
    });
});

Log:
2018-07-30 14:21:38.364820+0800 GCD_Test[27876:7958697] gcd_a: <NSThread: 0x608000072d00>{number = 3, name = (null)}
2018-07-30 14:21:38.382934+0800 GCD_Test[27876:7958641] gcd_main: <NSThread: 0x60c00006f740>{number = 1, name = main} (from: gcd_a)
````

### 栅栏

A,B,C,D 是四个(异步+并发).  
保证 A&B 执行结束后执行 C&D.

````
Code:
dispatch_queue_t queue = dispatch_queue_create("gcd_barrier", DISPATCH_QUEUE_CONCURRENT);
dispatch_async(queue, ^{
    dispatch_async(queue, ^{
        NSLog(@"a");
    });
    dispatch_async(queue, ^{
        NSLog(@"b");
    });
    dispatch_barrier_async(queue, ^{
        NSLog(@"barrier");
    });
    dispatch_async(queue, ^{
        NSLog(@"c");
    });
    dispatch_async(queue, ^{
        NSLog(@"d");
    });
});

Log:
2018-07-30 14:30:33.524359+0800 GCD_Test[28067:7975037] b
2018-07-30 14:30:33.524359+0800 GCD_Test[28067:7975038] a
2018-07-30 14:30:33.524547+0800 GCD_Test[28067:7975038] barrier
2018-07-30 14:30:33.524668+0800 GCD_Test[28067:7975038] c
2018-07-30 14:30:33.524669+0800 GCD_Test[28067:7975037] d
````

### 队列组

`dispatch_group_notify`: 监听 group 中任务的完成状态,当所有的任务都执行完成后,追加任务到 group 中,并执行任务.

````
Code:
dispatch_group_t group = dispatch_group_create();
dispatch_queue_t queue = dispatch_queue_create("gcd_grout_", DISPATCH_QUEUE_CONCURRENT);
dispatch_group_async(group, queue, ^{
    [NSThread sleepForTimeInterval:3.];
    NSLog(@"a");
});
dispatch_group_async(group, queue, ^{
    [NSThread sleepForTimeInterval:2.];
    NSLog(@"b");
});
dispatch_group_notify(group, queue, ^{
    NSLog(@"notify");
});

Log:
2018-07-30 16:19:53.914820+0800 GCD_Test[29644:8075312] b
2018-07-30 16:19:54.913442+0800 GCD_Test[29644:8075314] a
2018-07-30 16:19:54.913835+0800 GCD_Test[29644:8075314] notify
````

`dispatch_group_wait`: 暂停当前线程(阻塞当前线程),等待指定的 group 中的任务执行完成后,才会往下继续执行.

````
Code:
dispatch_group_t group = dispatch_group_create();
dispatch_queue_t queue = dispatch_queue_create("gcd_grout_", DISPATCH_QUEUE_CONCURRENT);
dispatch_group_async(group, queue, ^{
    [NSThread sleepForTimeInterval:3.];
    NSLog(@"a");
});
dispatch_group_async(group, queue, ^{
    [NSThread sleepForTimeInterval:2.];
    NSLog(@"b");
});
dispatch_group_notify(group, queue, ^{
    NSLog(@"notify");
});
dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
NSLog(@"dispatch_group_wait");

Log:
2018-07-30 16:32:46.546058+0800 GCD_Test[29911:8096807] a
2018-07-30 16:32:46.546271+0800 GCD_Test[29911:8096807] notify
2018-07-30 16:32:46.546274+0800 GCD_Test[29911:8096741] dispatch_group_wait    
````

`dispatch_group_enter`: 标志着一个任务追加到 group,执行一次,相当于 group 中未执行完毕任务数+1.  
`dispatch_group_leave`: 标志着一个任务离开了 group,执行一次,相当于 group 中未执行完毕任务数-1.  
当 group 中未执行完毕任务数为0的时候,才会使dispatch_group_wait解除阻塞,以及执行追加到dispatch_group_notify中的任务.

````
Code:
dispatch_group_t group = dispatch_group_create();
dispatch_queue_t queue = dispatch_queue_create("gcd_grout_", DISPATCH_QUEUE_CONCURRENT);
dispatch_group_enter(group);
dispatch_async(queue, ^{
    [NSThread sleepForTimeInterval:3.];
    NSLog(@"a");
    dispatch_group_leave(group);
});
dispatch_group_enter(group);
dispatch_async(queue, ^{
    [NSThread sleepForTimeInterval:2.];
    NSLog(@"b");
    dispatch_group_leave(group);
});
dispatch_group_notify(group, queue, ^{
    NSLog(@"notify");
});
dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
NSLog(@"dispatch_group_wait");
    
Log:
2018-07-30 16:37:53.217590+0800 GCD_Test[30035:8108341] b
2018-07-30 16:37:54.217441+0800 GCD_Test[30035:8108342] a
2018-07-30 16:37:54.217664+0800 GCD_Test[30035:8108245] dispatch_group_wait
2018-07-30 16:37:54.217676+0800 GCD_Test[30035:8108342] notify
````

### 信号量(dispatch_semaphore)

信号量(Semaphore),有时被称为信号灯,是在多线程环境下使用的一种设施,是可以用来保证两个或多个关键代码段不被并发调用.在进入一个关键代码段之前,线程必须获取一个信号量;一旦该关键代码段完成了,那么该线程必须释放信号量.其它想进入该关键代码段的线程必须等待直到第一个线程释放信号量.为了完成这个过程,需要创建一个信号量 VI(`dispatch_semaphore_create`) .然后将 Acquire Semaphore VI(`dispatch_semaphore_signal`) 以及 Release Semaphore VI(`dispatch_semaphore_wait`) 分别放置在每个关键代码段的首末端。确认这些信号量VI引用的是初始创建的信号量.  

`dispatch_semaphore_create`: 创建一个Semaphore并初始化信号的总量.  
`dispatch_semaphore_signal`: 发送一个信号，让信号总量+1.  
`dispatch_semaphore_wait`: 可以使总信号量-1,当信号总量为0时就会一直等待(阻塞所在线程),否则就可以正常执行.  

主要作用:

1. 保持线程同步,将异步执行任务转换为同步执行任务.
2. 保证线程安全,为线程加锁.  

***线程同步***

````
Code:
dispatch_queue_t queue = dispatch_queue_create("gcd_semaphore_", DISPATCH_QUEUE_CONCURRENT);
dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
dispatch_async(queue, ^{
    [NSThread sleepForTimeInterval:2.];
    NSLog(@"a");
    dispatch_semaphore_signal(semaphore);
});
NSLog(@"semaphore: <-wait");
dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
NSLog(@"semaphore: wait->");

Log:
2018-07-30 16:55:40.787978+0800 GCD_Test[30404:8144004] semaphore: <-wait
2018-07-30 16:55:42.792843+0800 GCD_Test[30404:8144061] a
2018-07-30 16:55:42.793268+0800 GCD_Test[30404:8144004] semaphore: wait->
````

***线程安全***

````
Code:
static dispatch_semaphore_t semaphore;
@interface ViewController ()
@property (nonatomic, assign) NSInteger semaphoreNumber;
@end
- (void)semaphore {
    self.semaphoreNumber = 10;
    semaphore = dispatch_semaphore_create(1);
    dispatch_queue_t queue_1 = dispatch_queue_create("gcd_semaphore_safe_1", DISPATCH_QUEUE_SERIAL);
    dispatch_queue_t queue_2 = dispatch_queue_create("gcd_semaphore_safe_1", DISPATCH_QUEUE_SERIAL);
    dispatch_async(queue_1, ^{
        [self semaphoreSafe];
    });
    dispatch_async(queue_2, ^{
        [self semaphoreSafe];
    });
}
- (void)semaphoreSafe {
    while (1) {
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
        if (self.semaphoreNumber == 0) {
            NSLog(@"end");
            break;
        }
        NSLog(@"%ld", self.semaphoreNumber);
        self.semaphoreNumber --;
        dispatch_semaphore_signal(semaphore);
    }
}

Log:
2018-07-30 17:09:02.980911+0800 GCD_Test[30848:8173853] 10
2018-07-30 17:09:02.981243+0800 GCD_Test[30848:8173863] 9
2018-07-30 17:09:02.981429+0800 GCD_Test[30848:8173853] 8
2018-07-30 17:09:02.981772+0800 GCD_Test[30848:8173863] 7
2018-07-30 17:09:02.982402+0800 GCD_Test[30848:8173853] 6
2018-07-30 17:09:02.982674+0800 GCD_Test[30848:8173863] 5
2018-07-30 17:09:02.982981+0800 GCD_Test[30848:8173853] 4
2018-07-30 17:09:02.983141+0800 GCD_Test[30848:8173863] 3
2018-07-30 17:09:02.983272+0800 GCD_Test[30848:8173853] 2
2018-07-30 17:09:02.983387+0800 GCD_Test[30848:8173863] 1
2018-07-30 17:09:02.983503+0800 GCD_Test[30848:8173853] end
````

### 延迟执行

`dispatch_after`函数并不是在指定时间之后才开始执行处理,而是在指定时间之后将任务追加到主队列中(也就是说延迟时间并不是绝对准确的).

````
Code:
NSLog(@"a");
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    NSLog(@"after");
});
    
Log:
2018-07-30 17:12:58.426891+0800 GCD_Test[30962:8182111] a
2018-07-30 17:12:59.518203+0800 GCD_Test[30962:8182111] after
````

### 一次性执行

`dispatch_once`函数能保证某段代码在程序运行过程中只被执行1次,并且即使在多线程的环境下,`dispatch_once`也可以保证线程安全.

````
Code:
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
    NSLog(@"once");
});

Log:
2018-07-30 17:17:25.182846+0800 GCD_Test[31088:8191960] once
````

### 快速迭代

串行队列中使用`dispatch_apply`,那么就和 for 循环一样,按顺序同步执行.    
并发队列进行异步执行,遍历 0~9 这 10 个数字`dispatch_apply`可以在多个线程中同时**异步**遍历多个数字(非顺序).    
无论是在串行队列,还是异步队列中,`dispatch_apply`都会等待全部任务执行完毕,这点就像是同步操作,也像是队列组中的`dispatch_group_wait`方法.  

````
Code:
dispatch_queue_t queue = dispatch_queue_create("gcd_apply_", DISPATCH_QUEUE_CONCURRENT);
dispatch_apply(10, queue, ^(size_t index) {
    NSLog(@"%ld", index);
});
NSLog(@"end");

Log:
2018-07-30 17:23:35.204647+0800 GCD_Test[31246:8204611] 6
2018-07-30 17:23:35.204645+0800 GCD_Test[31246:8204600] 1
2018-07-30 17:23:35.204644+0800 GCD_Test[31246:8204601] 2
2018-07-30 17:23:35.204645+0800 GCD_Test[31246:8204603] 3
2018-07-30 17:23:35.204644+0800 GCD_Test[31246:8204599] 4
2018-07-30 17:23:35.204644+0800 GCD_Test[31246:8204557] 5
2018-07-30 17:23:35.204644+0800 GCD_Test[31246:8204602] 0
2018-07-30 17:23:35.204657+0800 GCD_Test[31246:8204612] 7
2018-07-30 17:23:35.204814+0800 GCD_Test[31246:8204600] 8
2018-07-30 17:23:35.204821+0800 GCD_Test[31246:8204601] 9
2018-07-30 17:23:35.205319+0800 GCD_Test[31246:8204557] end
````

### 任务取消

#### 取消未执行的任务

***方法一***

````
Code:
dispatch_queue_t queue = dispatch_queue_create("gcd_cancle_", DISPATCH_QUEUE_CONCURRENT);
dispatch_block_t block1 = dispatch_block_create(0, ^{
    sleep(1);
    NSLog(@"block1");
});
dispatch_block_t block2 = dispatch_block_create(0, ^{
    NSLog(@"block2");
});
dispatch_block_t block3 = dispatch_block_create(0, ^{
    NSLog(@"block3");
});
dispatch_async(queue, block1);
dispatch_async(queue, block2);
dispatch_async(queue, block3);
dispatch_block_cancel(block3);

Log:
2018-07-30 17:36:54.686663+0800 GCD_Test[31592:8230196] block2
2018-07-30 17:36:55.688053+0800 GCD_Test[31592:8230194] block1
````

***方法二***

````
Code:
static BOOL isCancle; // 全局变量
isCancle = NO;
dispatch_queue_t queue = dispatch_queue_create("gcd_cancle_", DISPATCH_QUEUE_CONCURRENT);
dispatch_block_t block1 = dispatch_block_create(0, ^{
    if (!isCancle) {
        NSLog(@"block1");
        isCancle = YES;
    }
});
dispatch_block_t block2 = dispatch_block_create(0, ^{
    sleep(2);
    if (!isCancle) {
        NSLog(@"block2");
    }
});
dispatch_block_t block3 = dispatch_block_create(0, ^{
    sleep(3);
    if (!isCancle) {
        NSLog(@"block3");
    }
});
dispatch_async(queue, block1);
dispatch_async(queue, block2);
dispatch_async(queue, block3);

Log:
2018-07-30 19:02:34.201776+0800 GCD_Test[33056:8318483] block1
````

#### 取消正在执行的任务

````
Code:
static BOOL isCancle; // 全局变量
dispatch_async(dispatch_get_global_queue(0, 0), ^{
    for (NSInteger i = 0; i < 10000; i++) {
        NSLog(@"正在执行第%ld次...", i);
        sleep(1);
        if (isCancle == YES) {
            NSLog(@"终止!");
            return;
        }
    };
});
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(5 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
   NSLog(@"发送终止信息...");
   isCancle = YES;
                   
});

Log:
2018-07-30 19:03:05.988849+0800 GCD_Test[33087:8320254] 正在执行第0次...
2018-07-30 19:03:06.990976+0800 GCD_Test[33087:8320254] 正在执行第1次...
2018-07-30 19:03:07.992931+0800 GCD_Test[33087:8320254] 正在执行第2次...
2018-07-30 19:03:08.995507+0800 GCD_Test[33087:8320254] 正在执行第3次...
2018-07-30 19:03:09.998326+0800 GCD_Test[33087:8320254] 正在执行第4次...
2018-07-30 19:03:10.988747+0800 GCD_Test[33087:8320216] 发送终止信息...
2018-07-30 19:03:11.002765+0800 GCD_Test[33087:8320254] 终止!
````


