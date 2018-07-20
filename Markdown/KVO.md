# KVO
---

>>> Key Value Observer
>>> 
>>> 键值观察

### 原理?
观察一个对象时,一个新的类会被动态创建.这个类继承自该对象的原本的类,并重写了被观察属性的 `setter`方法.重写的`setter`方法会负责在调用原`setter`方法之前和之后,通知所有观察对象:值的更改.最后通过`isa`混写(isa-swizzling)把这个对象的`isa`指针(isa 指针告诉 Runtime 系统这个对象的类是什么)指向这个新创建的子类,对象就神奇的变成了新创建的子类的实例.

键值观察通知依赖于`NSObject`的两个方法:`willChangeValueForKey:`和 `didChangevlueForKey:`.在一个被观察属性发生改变之前,`willChangeValueForKey:`一定会被调用,这就会记录旧的值.而当改变发生后,`didChangeValueForKey:`会被调用,继而 `observeValueForKey:ofObject:change:context:`也会被调用.

新类核心代码  

````
- (void)setName:(NSString *)name {
    [self willChangeValueForKey:@"name"];
    [super setValue:aDate forKey:@"name"]; // 注意: super
    [self didChangeValueForKey:@"name"];
}
````

### 关于观察对象的 Set 重写

````
- (void)setName:(NSString *)name {
    [self willChangeValueForKey:@"name"];
    [self setValue:aDate forKey:@"name"];
    [self didChangeValueForKey:@"name"];
}

KVO会被调用两次.
````

### 手动触发KVO?

在目标位置调用`willChangeValueForKey :` & `didChangeValueForKey :`.

### 关于 isa-swizzling
`isa: 一个指向Class的指针.`  
如果观察对象类为 A , KVO 创建的的新类为 B (继承于 A ).  
isa-swizzling: A 的 isa 指向 B 的 Class.

