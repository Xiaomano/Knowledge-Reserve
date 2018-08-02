# atomic & nonatomic
---

>>> Atomic is really commonly confused with being thread-safe, and that is not correct. You need to guarantee your thread safety other ways.

eg .

线程 `A` 调了 `getter`,与此同时线程 `B` 、线程 `C` 都调了 `setter`.最后线程 `A` `getter` 到的值有3种可能:  
可能是 `B`、`C` `setter` 之前原始的值.  
可能是 `B` `setter` 的值.  
可能是 `C` `setter` 的值.  
所以 `atomic` 并不能保证对象的线程安全.  

1. `atomic` 和 `nonatomic` 用来决定编译器生成的 `getter` 和 `setter` 是否为原子操作.
2. `atomic` 系统生成的 `getter`/`setter` 会保证 `get`/`set` 操作的完整性,不受其他线程影响. `getter` 能得到一个完好无损的对象(可以保证数据的完整性),但这个对象在多线程的情况下是不能确定的,例如上面的例子.
3. `nonatomic` 没有(`atomic`)保证, `nonatomic`返回你的对象可能就不是完整的value.因此,在多线程的环境下原子操作是非常必要的,否则有可能会引起错误的结果.但仅仅使用 `atomic` 并不会使得对象线程安全。
4. `nonatomic` 的速度要比 `atomic` 的快.

### nonatomic (setter & getter)

````
- (void)setName:(NSString *)name {
    if (_name != name) {
        _name = name;
    }
}

- (NSString *)name {
    return _name;
}
````

### atomic (setter & getter)

````
- (void)setName:(NSString *)name {
    @synchronized(self) {
        if (_name != name) {
            _name = name;
        }
    }
}

- (NSString *)name {
    @synchronized(self) {
        return _name;
    }
}
````