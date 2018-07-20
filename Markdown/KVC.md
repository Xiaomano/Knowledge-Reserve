# KVC
---

>>> Key Value Coding
>>> 
>>> 键值编码

### 关于基本数据类型
`setValue:forKey:`value无法传入基本数据类型.  
`valueForKey:`返回值是id类型.

### 关于下划线
KVC中不管你的成员变量是否加下划线,你用KVC取值和赋值时传入的属性名都可以不带下划线.

### 关于 set & get 方法
KVC赋值会调用`set:`.  
KVC取值会调用`get:`.

### 关于KVC取值和赋值顺序
1. 用KVC取值或赋值时,会优先找这个属性对应的getter或setter方法来对这个属性赋值.
2. 如果找不到,则会查找带下划线的属性,如果找到则赋值.
3. 如果依然找不到,则会查找不带下划线的属性,如果找到则赋值.
4. 如果还是找不到,则报错.

### 关于属性的属性
赋值属性的属性:`setValue:forKeyPath:`   
取值属性的属性:`valueForKeyPath:`

### 关于不存在KEY
`- (void)setValue:(id)value forUndefinedKey:(NSString *)key;`