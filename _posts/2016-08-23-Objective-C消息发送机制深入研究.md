---
layout: single
title: Objective-C消息发送机制深入研究
date: 2016-08-23

---

### objc_msgSend方法
在OC里面，直到运行时消息才绑定到方法的实现，编译器将以下表达式

`[receiver message]`

转换为调用objc_msgSende的一个方法

`objc_msgSend(receiver, selector, arg1, arg2, ...)`

查找方法的顺序

* 通过`isa`指针找到对应的class，先从class的方法缓存表里面查找。
* 如果找不到再去的selector分配表里面查找方法。
* 如果找不到对应的方法，通过`super_class`指针找到父类，再从父类的分配表里面查找方法，就这样一直找到`NSObject` 

### 动态添加方法

可以通过实现` resolveInstanceMethod:` 或者 `resolveClassMethod:` 方法分别为实例方法或者类方法动态添加实现。

```
void dynamicMethodIMP(id self, SEL _cmd) {
    // implementation ....
}

@implementation MyClass
+ (BOOL)resolveInstanceMethod:(SEL)aSEL
{
    if (aSEL == @selector(resolveThisMethodDynamically)) {
          class_addMethod([self class], aSEL, (IMP) dynamicMethodIMP, "v@:");
          return YES;
    }
    return [super resolveInstanceMethod:aSEL];
}
@end
```

如果以上的动态方法也没有添加，不用担心，还有机会动态转发消息。

### 动态转发消息
举个例子，假设小明不想做家庭作业，而是让他的家长代替他做，我们将这种行为模拟出来

```
- (void)forwardInvocation:(NSInvocation *)anInvocation {
    Parent *parent = [[Parent alloc] init];
    if ([parent respondsToSelector:anInvocation.selector]) {
        [anInvocation invokeWithTarget:parent];
    } else {
        [super forwardInvocation:anInvocation];
    }
}

- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
    Parent *parent = [[Parent alloc] init];
    if (aSelector == @selector(doSomeHomeWork)) {
        return [parent methodSignatureForSelector:aSelector];
    }
    return nil;
}
```