---
layout: single
title: Swift如何实现dispatch_once
date: 2020-05-20
category: 技术
tags: [iOS, swift, GCD]
---

*对于使用过Objective-C的开发者来说，`dispatch_once`已经很熟悉了，用于保证某段程序只需要初始化一次的时候使用，但是现在用Swift开发之后发现没有这个方法了，查阅相关资料，发现可以自己实现类似的功能，代码如下：*
```
public extension DispatchQueue {
    private static var _onceTracker = [String]()

    class func once(file: String = #file, function: String = #function, line: Int = #line, block: () -> Void) {
        let token = "\(file):\(function):\(line)"
        once(token: token, block: block)
    }

    class func once(token: String, block: () -> Void) {
        objc_sync_enter(self)
        defer {
            objc_sync_exit(self)
        }
        guard !_onceTracker.contains(token) else { return }
        _onceTracker.append(token)
        block()
    }
}
```

代码很简单，其中挑选两个关键点分析：
## defer
它提供了一种延迟调用的方法，保证在当前代码块返回前执行某段代码，常用于资源的释放，数据库关闭等。文章代码主要用于锁的释放。
如果没有defer，你的代码可能是这样的
```
let db = DataBase()
func myFunc() {
    db.open()
    // do something
    if condition1 {
        // do something
        db.close()
        return
    }
    if condition2 {
        // do something
        db.close()
        return
    }
    // do something
    db.close()
    return
}
```
如果某个判断条件没有关闭数据库，可能就会发生异常了，但是有了defer之后，使得代码变得更加安全：
```
let db = DataBase()
func myFunc() {
    db.open()
    defer {
        db.close()
    }
    // do something
    if condition1 {
        // do something
        return
    }
    if condition2 {
        // do something
        return
    }
    // do something
    return
}
```

## objc_sync_enter和objc_sync_exit
对于这两个方法大家可能比较陌生，但是提到`@sychronized`关键字大家就比较熟悉了，主要用于多线程对某个对象加锁，其实`@sychronized`底层就是调用的`objc_sync_enter`和`objc_sync_exit`来对对象进行多线程控制，并且这是一个递归锁，具体的底层实现代码可以参考以下链接[https://github.com/opensource-apple/objc4/blob/master/runtime/objc-sync.mm]

参考文章：
* https://onevcat.com/2018/11/defer/
* https://nshipster.cn/guard-and-defer/
