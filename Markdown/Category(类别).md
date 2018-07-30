# Category(类别)
---

>>> 为现有类添加新的方法.

### Category 结构

````
typedef struct objc_category *Category;
struct objc_category {
  char *category_name                          OBJC2_UNAVAILABLE; // 分类名
  char *class_name                             OBJC2_UNAVAILABLE; // 分类所属的类名
  struct objc_method_list *instance_methods    OBJC2_UNAVAILABLE; // 实例方法列表
  struct objc_method_list *class_methods       OBJC2_UNAVAILABLE; // 类方法列表
  struct objc_protocol_list *protocols         OBJC2_UNAVAILABLE; // 分类所实现的协议列表
}
````

### 关于属性&实例变量
类别默认不支持属性&实例变量.  

***如何支持属性&实例变量?***  

````
关联属性
// 赋值
objc_setAssociatedObject(id _Nonnull object, const void * _Nonnull key, id _Nullable value, objc_AssociationPolicy policy);
// 取值
objc_getAssociatedObject(id _Nonnull object, const void * _Nonnull key);
// 注销
objc_removeAssociatedObjects(id _Nonnull object);
````

### 扩展(匿名类别)

````
@interface ViewController ()
@end
````

1. 支持属性.
2. 支持实例变量.