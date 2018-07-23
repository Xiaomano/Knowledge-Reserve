# Singleton(单例)
---
>>> 单例模式是一种常用的软件设计模式。在它的核心结构中只包含一个被称为单例的特殊类。通过单例模式可以保证系统中一个类只有一个实例而且该实例易于外界访问，从而方便对实例个数的控制并节约系统资源。如果希望在系统中某个类的对象只能存在一个，单例模式是最好的解决方案。

### Step
1. 创建类方法,返回对象实例.以shared,default,current开头
2. 创建一个全局变量用来保存对象的引用
3. 判断对象是否存在,若不存在,创建对象

### Code
#### 非线程安全

````
static UserHelper * helper = nil;
+ (UserHelper *)sharedUserHelper {
    if (helper == nil) {
        helper = [[UserHelper alloc] init];
    }
    return helper;
}
````

#### 线程安全
````
// 一:
static UserHelper * helper = nil;
+ (UserHelper *)sharedUserHelper {
    @synchronized(self) {
        if (helper == nil) {
            helper = [[UserHelper alloc] init];
        }
    }
    return helper;
}

// 二:
static UserHelper * helper = nil;
+ (void)initialize {
    if ([self class] == [UserHelper class]) {
        helper = [[UserHelper alloc] init];
    }
}

// 三: 苹果推荐
static UserHelper * helper = nil;
+ (UserHelper *)sharedUserHelper {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        helper = [[UserHelper alloc] init];
    });
    return helper;
}

````