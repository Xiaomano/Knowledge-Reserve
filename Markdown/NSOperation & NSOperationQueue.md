### NSOperation & NSOperationQueue
---

>>> NSOperation、NSOperationQueue 是苹果提供给我们的一套多线程解决方案.实际上 NSOperation、NSOperationQueue 是基于 GCD 更高一层的封装,完全面向对象.但是比 GCD 更简单易用、代码可读性也更高.

***为什么要使用 NSOperation、NSOperationQueue?***  

1. 可添加完成的代码块,在操作完成后执行.
2. 添加操作之间的依赖关系,方便的控制执行顺序.
3. 设定操作执行的优先级.
4. 可以很方便的取消一个操作的执行.
5. 使用 KVO 观察对操作执行状态的更改: isExecuteing、isFinished、isCancelled.

***操作 (Operation)***  

1. 执行操作的意思,换句话说就是你在线程中执行的那段代码.
2. 在 GCD 中是放在 block 中的.在 NSOperation 中,我们使用 NSOperation 子类 NSInvocationOperation、NSBlockOperation,或者自定义子类来封装操作.

***操作队列 (Operation Queues)***

1. 这里的队列指操作队列,即用来存放操作的队列.不同于 GCD 中的调度队列 FIFO（先进先出）的原则.NSOperationQueue 对于添加到队列中的操作,首先进入准备就绪的状态(就绪状态取决于操作之间的依赖关系),然后进入就绪状态的操作的开始执行顺序(非结束执行顺序)由操作之间相对的优先级决定(优先级是操作对象自身的属性).
2. 操作队列通过设置最大并发操作数(maxConcurrentOperationCount)来控制并发、串行.
3. NSOperationQueue 为我们提供了两种不同类型的队列:主队列和自定义队列.主队列运行在主线程之上,而自定义队列在后台执行.

### NSInvocationOperation

在没有使用 NSOperationQueue、在主线程中单独使用使用子类 NSInvocationOperation 执行一个操作的情况下,操作是在当前线程执行的,并没有开启新线程.

````
Code:
- (void)startInvocationOperation {
    NSInvocationOperation *op = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(test) object:nil];
    [op start];
}
- (void)test {
    NSLog(@"a");
    NSLog(@"%@", [NSThread currentThread]);
}

Log:
2018-07-31 10:19:41.938352+0800 NSOperation_Test[46133:9001499] a
2018-07-31 10:19:41.938602+0800 NSOperation_Test[46133:9001499] <NSThread: 0x6080000665c0>{number = 1, name = main}
````

在其他线程中单独使用子类 NSInvocationOperation,操作是在当前调用的其他线程执行的,并没有开启新线程.

````
Code:
[NSThread detachNewThreadSelector:@selector(startInvocationOperation) toTarget:self withObject:nil];
...

Log:
2018-07-31 10:22:45.482398+0800 NSOperation_Test[46256:9008347] a
2018-07-31 10:22:45.482691+0800 NSOperation_Test[46256:9008347] <NSThread: 0x600000076b40>{number = 3, name = (null)}
````

### NSBlockOperation

在没有使用 NSOperationQueue、在主线程中单独使用 NSBlockOperation 执行一个操作的情况下,操作是在当前线程执行的,并没有开启新线程.  

NSBlockOperation 提供了一个方法 `addExecutionBlock:`,通过 `addExecutionBlock:` 就可以为 NSBlockOperation 添加额外的操作.这些操作(包括 blockOperationWithBlock 中的操作)可以在不同的线程中同时(并发)执行.只有当所有相关的操作已经完成执行时,才视为完成.

如果添加的操作多的话,`blockOperationWithBlock:` 中的操作也可能会在其他线程(非当前线程)中执行,这是由系统决定的,并不是说添加到 `blockOperationWithBlock:` 中的操作一定会在当前线程中执行.

````
Code:
NSBlockOperation *op = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"a");
    NSLog(@"%@", [NSThread currentThread]);
}];
[op addExecutionBlock:^{
    NSLog(@"b");
    NSLog(@"%@", [NSThread currentThread]);
}];
[op addExecutionBlock:^{
    NSLog(@"c");
    NSLog(@"%@", [NSThread currentThread]);
}];
[op addExecutionBlock:^{
    NSLog(@"d");
    NSLog(@"%@", [NSThread currentThread]);
}];
[op addExecutionBlock:^{
    NSLog(@"e");
    NSLog(@"%@", [NSThread currentThread]);
}];
[op addExecutionBlock:^{
    NSLog(@"f");
    NSLog(@"%@", [NSThread currentThread]);
}];
[op addExecutionBlock:^{
    NSLog(@"g");
    NSLog(@"%@", [NSThread currentThread]);
}];
[op addExecutionBlock:^{
    NSLog(@"h");
    NSLog(@"%@", [NSThread currentThread]);
}];
[op addExecutionBlock:^{
    NSLog(@"i");
    NSLog(@"%@", [NSThread currentThread]);
}];
[op addExecutionBlock:^{
    NSLog(@"j");
    NSLog(@"%@", [NSThread currentThread]);
}];
[op addExecutionBlock:^{
    NSLog(@"k");
    NSLog(@"%@", [NSThread currentThread]);
}];
[op start];

Log:
2018-07-31 10:32:34.492492+0800 NSOperation_Test[46432:9024607] b
2018-07-31 10:32:34.492496+0800 NSOperation_Test[46432:9024621] h
2018-07-31 10:32:34.492492+0800 NSOperation_Test[46432:9024608] a
2018-07-31 10:32:34.492492+0800 NSOperation_Test[46432:9024553] f
2018-07-31 10:32:34.492492+0800 NSOperation_Test[46432:9024606] e
2018-07-31 10:32:34.492501+0800 NSOperation_Test[46432:9024620] g
2018-07-31 10:32:34.492492+0800 NSOperation_Test[46432:9024605] c
2018-07-31 10:32:34.492492+0800 NSOperation_Test[46432:9024604] d
2018-07-31 10:32:34.492785+0800 NSOperation_Test[46432:9024553] <NSThread: 0x604000063a00>{number = 1, name = main}
2018-07-31 10:32:34.492798+0800 NSOperation_Test[46432:9024621] <NSThread: 0x60c00007cc80>{number = 4, name = (null)}
2018-07-31 10:32:34.492801+0800 NSOperation_Test[46432:9024607] <NSThread: 0x604000073f40>{number = 3, name = (null)}
2018-07-31 10:32:34.492803+0800 NSOperation_Test[46432:9024608] <NSThread: 0x60800007d680>{number = 5, name = (null)}
2018-07-31 10:32:34.492834+0800 NSOperation_Test[46432:9024606] <NSThread: 0x604000073a80>{number = 6, name = (null)}
2018-07-31 10:32:34.492891+0800 NSOperat2018-07-31 10:32:34.492934+0800 NSOperation_Test[46432:9024604] <NSThread: 0x60c00007c940>{number = 9, name = (null)}
ion_Test[46432:9024605] <NSThread: 0x604000073c00>{number = 8, name = (null)}
2018-07-31 10:32:34.492907+0800 NSOperation_Test[46432:9024620] <NSThread: 0x60000007ab80>{number = 7, name = (null)}
2018-07-31 10:32:34.493084+0800 NSOperation_Test[46432:9024553] i
2018-07-31 10:32:34.493299+0800 NSOperation_Test[46432:9024621] j
2018-07-31 10:32:34.493480+0800 NSOperation_Test[46432:9024607] k
2018-07-31 10:32:34.494388+0800 NSOperation_Test[46432:9024553] <NSThread: 0x604000063a00>{number = 1, name = main}
2018-07-31 10:32:34.494558+0800 NSOperation_Test[46432:9024621] <NSThread: 0x60c00007cc80>{number = 4, name = (null)}
2018-07-31 10:32:34.494677+0800 NSOperation_Test[46432:9024607] <NSThread: 0x604000073f40>{number = 3, name = (null)}
````

### 自定义NSOperation

通过重写 main 或者 start 方法来定义自己的 NSOperation 对象.

```` objc
Code:

@interface PPOperation : NSOperation
@end

@implementation PPOperation
- (void)main {
    NSLog(@"PPOperation");
    NSLog(@"%@", [NSThread currentThread]);
}
@end

````

````
Code:
PPOperation *op = [[PPOperation alloc] init];
[op start];
    
Log:
2018-07-31 10:37:11.277230+0800 NSOperation_Test[46556:9033876] PPOperation
2018-07-31 10:37:11.277386+0800 NSOperation_Test[46556:9033876] <NSThread: 0x600000077c40>{number = 1, name = main}
````

### NSOperationQueue Create

**主队列**

凡是添加到主队列中的操作,都会放到主线程中执行.

````
NSOperationQueue *queue = [NSOperationQueue mainQueue];
````

**自定义队列**

添加到这种队列中的操作,就会自动放到子线程中执行.

````
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
````

### NSOperationQueue add NSOperation

````
Code:
NSOperationQueue *opQueue = [[NSOperationQueue alloc] init];
NSInvocationOperation *op_invocation = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(test) object:nil];
NSBlockOperation *op_Block = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"b");
    NSLog(@"%@", [NSThread currentThread]);
}];
PPOperation *op_pp = [[PPOperation alloc] init];
[opQueue addOperation:op_invocation];
[opQueue addOperation:op_Block];
[opQueue addOperation:op_pp];
[opQueue addOperationWithBlock:^{
    NSLog(@"c");
    NSLog(@"%@", [NSThread currentThread]);
}];

Log:
2018-07-31 10:50:16.939014+0800 NSOperation_Test[46808:9057917] PPOperation
2018-07-31 10:50:16.939014+0800 NSOperation_Test[46808:9057916] c
2018-07-31 10:50:16.939014+0800 NSOperation_Test[46808:9057919] b
2018-07-31 10:50:16.939026+0800 NSOperation_Test[46808:9057915] a
2018-07-31 10:50:16.939359+0800 NSOperation_Test[46808:9057919] <NSThread: 0x60c000462ec0>{number = 5, name = (null)}
2018-07-31 10:50:16.939369+0800 NSOperation_Test[46808:9057917] <NSThread: 0x604000075a80>{number = 3, name = (null)}
2018-07-31 10:50:16.939370+0800 NSOperation_Test[46808:9057916] <NSThread: 0x608000263fc0>{number = 4, name = (null)}
2018-07-31 10:50:16.939481+0800 NSOperation_Test[46808:9057915] <NSThread: 0x604000074280>{number = 6, name = (null)}
````

### NSOperationQueue -> 串行 & 并发

关键属性 maxConcurrentOperationCount,叫做最大并发操作数.用来控制一个特定队列中可以有多少个操作同时参与并发执行.

1. maxConcurrentOperationCount 默认情况下为-1,表示不进行限制,可进行并发执行.
2. maxConcurrentOperationCount 为1时,队列为串行队列.只能串行执行.
3. maxConcurrentOperationCount 大于1时,队列为并发队列.操作并发执行,当然这个值不应超过系统限制,即使自己设置一个很大的值,系统也会自动调整为系统设定的默认最大值。

### NSOperation Dependency

````
添加依赖,使当前操作依赖于操作 op 的完成.
- (void)addDependency:(NSOperation *)op; 
移除依赖，取消当前操作对操作 op 的依赖.
- (void)removeDependency:(NSOperation *)op; 
在当前操作开始执行之前完成执行的所有操作对象数组.
@property (readonly, copy) NSArray<NSOperation *> *dependencies; 
````

````
Code:
NSOperationQueue *opQueue = [[NSOperationQueue alloc] init];
NSBlockOperation *op_1 = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"op_1");
    NSLog(@"%@", [NSThread currentThread]);
}];
NSBlockOperation *op_2 = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"op_2");
    NSLog(@"%@", [NSThread currentThread]);
}];
NSBlockOperation *op_3 = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"op_3");
    NSLog(@"%@", [NSThread currentThread]);
}];
NSBlockOperation *op_4 = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"op_4");
    NSLog(@"%@", [NSThread currentThread]);
}];
[op_4 addDependency:op_3];
[op_3 addDependency:op_2];
[op_2 addDependency:op_1];
[opQueue addOperation:op_4];
[opQueue addOperation:op_3];
[opQueue addOperation:op_2];
[opQueue addOperation:op_1];
    
Log:
2018-07-31 11:05:48.288734+0800 NSOperation_Test[47089:9083647] op_1
2018-07-31 11:05:48.289038+0800 NSOperation_Test[47089:9083647] <NSThread: 0x60c000470480>{number = 3, name = (null)}
2018-07-31 11:05:48.289195+0800 NSOperation_Test[47089:9083643] op_2
2018-07-31 11:05:48.289479+0800 NSOperation_Test[47089:9083643] <NSThread: 0x608000078340>{number = 4, name = (null)}
2018-07-31 11:05:48.289626+0800 NSOperation_Test[47089:9083643] op_3
2018-07-31 11:05:48.289720+0800 NSOperation_Test[47089:9083643] <NSThread: 0x608000078340>{number = 4, name = (null)}
2018-07-31 11:05:48.289847+0800 NSOperation_Test[47089:9083643] op_4
2018-07-31 11:05:48.289964+0800 NSOperation_Test[47089:9083643] <NSThread: 0x608000078340>{number = 4, name = (null)}
````

### NSOperation Priority

````
// 优先级的取值
typedef NS_ENUM(NSInteger, NSOperationQueuePriority) {
    NSOperationQueuePriorityVeryLow = -8L,
    NSOperationQueuePriorityLow = -4L,
    NSOperationQueuePriorityNormal = 0,
    NSOperationQueuePriorityHigh = 4,
    NSOperationQueuePriorityVeryHigh = 8
};
````
NSOperation 提供了queuePriority(优先级)属性,queuePriority属性适用于同一操作队列中的操作,不适用于不同操作队列中的操作.默认情况下,所有新创建的操作对象优先级都是NSOperationQueuePriorityNormal.但是我们可以通过`setQueuePriority:`方法来改变当前操作在同一队列中的执行优先级.

1. queuePriority 属性决定了进入准备就绪状态下的操作之间的开始执行顺序.并且,优先级不能取代依赖关系.
2. 如果一个队列中既包含高优先级操作,又包含低优先级操作,并且两个操作都已经准备就绪,那么队列先执行高优先级操作.
3. 如果一个队列中既包含了准备就绪状态的操作,又包含了未准备就绪的操作,未准备就绪的操作优先级比准备就绪的操作优先级高.那么,虽然准备就绪的操作优先级低,也会优先执行.优先级不能取代依赖关系.如果要控制操作间的启动顺序,则必须使用依赖关系.

### 线程间的通信

````
Code:
NSOperationQueue *opQueue = [[NSOperationQueue alloc]init];
    [opQueue addOperationWithBlock:^{
        NSLog(@"a");
        NSLog(@"%@", [NSThread currentThread]);
        [[NSOperationQueue mainQueue] addOperationWithBlock:^{
            NSLog(@"b");
            NSLog(@"%@", [NSThread currentThread]);
        }];
    }];
    
Log:
2018-07-31 11:14:37.937430+0800 NSOperation_Test[47283:9100390] a
2018-07-31 11:14:37.937920+0800 NSOperation_Test[47283:9100390] <NSThread: 0x600000260800>{number = 3, name = (null)}
2018-07-31 11:14:37.960435+0800 NSOperation_Test[47283:9100348] b
2018-07-31 11:14:37.960653+0800 NSOperation_Test[47283:9100348] <NSThread: 0x60c000071e80>{number = 1, name = main}
````

### 线程同步&线程安全

略 (锁🔐).

### NSOperation 常用属性和方法

1. `- (void)cancel;` 可取消操作,实质是标记 isCancelled 状态.
2. `- (BOOL)isFinished;` 判断操作是否已经结束.
3. `- (BOOL)isCancelled;` 判断操作是否已经标记为取消.
4. `- (BOOL)isExecuting;` 判断操作是否正在在运行.
5. `- (BOOL)isReady;` 判断操作是否处于准备就绪状态,这个值和操作的依赖关系相关.
6. `- (void)waitUntilFinished;` 阻塞当前线程,直到该操作结束.可用于线程执行顺序的同步.
7. `- (void)setCompletionBlock:(void (^)(void))block;` completionBlock 会在当前操作执行完毕时执行 completionBlock.
8. `- (void)addDependency:(NSOperation *)op;` 添加依赖,使当前操作依赖于操作 op 的完成.
9. `- (void)removeDependency:(NSOperation *)op;` 移除依赖,取消当前操作对操作 op 的依赖.
10. `@property (readonly, copy) NSArray<NSOperation *> *dependencies;` 在当前操作开始执行之前完成执行的所有操作对象数组.

### NSOperationQueue 常用属性和方法

1. `- (void)cancelAllOperations;` 可以取消队列的所有操作.
2. `- (BOOL)isSuspended;` 判断队列是否处于暂停状态. YES 为暂停状态, NO 为恢复状态.
3. `- (void)setSuspended:(BOOL)b;` 可设置操作的暂停和恢复, YES 代表暂停队列, NO 代表恢复队列.
4. `- (void)waitUntilAllOperationsAreFinished;` 阻塞当前线程,直到队列中的操作全部执行完毕.
5. `- (void)addOperationWithBlock:(void (^)(void))block;` 向队列中添加一个 NSBlockOperation 类型操作对象.
6. `- (void)addOperations:(NSArray *)ops waitUntilFinished:(BOOL)wait;` 向队列中添加操作数组,wait 标志是否阻塞当前线程直到所有操作结束.
7. `- (NSArray *)operations;` 当前在队列中的操作数组(某个操作执行结束后会自动从这个数组清除).
8. `- (NSUInteger)operationCount;` 当前队列中的操作数.
9. `+ (id)currentQueue;` 获取当前队列,如果当前线程不是在 NSOperationQueue 上运行则返回 nil.
10. `+ (id)mainQueue;` 获取主队列.

### 取消正在执行的线程

重写 main 方法.