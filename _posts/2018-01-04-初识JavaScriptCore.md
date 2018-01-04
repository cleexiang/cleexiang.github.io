---

layout: single
title: 初识JavaScriptCore
date: 2018-01-04

---

### 基本概念

#### <a name="https://developer.apple.com/documentation/javascriptcore/jsvirtualmachine">JSVirtualMachine</a>

> JS虚拟运行环境，它有两个作用：一是提供了JS的多线程运行环境，为了并发执行JS，你需要提供多个JSVirtualMachine，二是负责管理JS和OC或者Swift之间桥接的对象（也就是内存管理）。不同JSVirtualMachine中的JSValue对象不能被互相传递（参考图一）。

#### <a name="https://developer.apple.com/documentation/javascriptcore/jscontext">JSContext</a>

> 执行JS代码的环境，通过Objective-C或者Swift代码运行JS代码，不但能接收从JS中计算的值，而且提供了从原生对象，方法，函数对JS的调用能力。一个JSVirtualMachine中可以有多个JSContext，不同JSContext中的JSValue对象可以互相传递（参考图一）。

#### <a name="https://developer.apple.com/documentation/javascriptcore/jsvalue">JSValue</a>

> JSValue可以表示任何的原始数据类型，它既可以用来转换基本数值（如number，string），也可以用来创建js对象，用于包装native代码中的class，method，block对象（参考图二）。每一个JSValue对象关联着一个JSContext，并且强引用持有JSContext。

#### <a name="https://developer.apple.com/documentation/javascriptcore/jsmanagedvalue"> JSManagedValue </a>

> JSManagedValue封装了JSValue对象，通过“条件持有”的方式提供自动内存管理。注意：不要在native对象中持有non-managed的JSValue对象，否则会造成循环引用。

#### <a name="https://developer.apple.com/documentation/javascriptcore/jsexport"> JSExport</a>

> JSExport是一个协议，实现该协议可以将Objective-C类中的方法，属性导出成JS代码。

图一：对象传递规则
![p1](https://koenig-media.raywenderlich.com/uploads/2016/02/javascriptcore.png)

图二：JavaScript和Native之间的对象转换规则

Native  | JavaScript
------------- | -------------
 nil         |     undefined
NSNull       |        null
NSString      |       string
NSNumber      |   number, boolean
NSDictionary    |   Object object
NSArray       |    Array object
NSDate       |     Date object
NSBlock (1)   |   Function object (1)
  id (2)     |   Wrapper object (2)
Class (3)    | Constructor object (3)

#### 未完待续。。。