---

layout: single
title: 初识JavaScriptCore
date: 2018-01-04

---

### 背景
*由于近半年来的技术主要围绕着Cordova，ReactNative等跨平台技术，一方面为了深入理解其中的运行原理，一方面也是因为对大红大紫的前端技术越来越感兴趣，于是对iOS系统上的JavaScript相关的技术进行了一些更深入的研究*

### JavaScriptCore中的基本概念

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

#### 基本用法

##### Objective-C调用Javascript

```
func objcToJs(_ number1: Int, _ number2: Int) -> Int32 {
    // 声明一个JSContext对象
    guard let ctx = JSContext() else { return 0}
    // 定义一个js函数，这里的js代码也可以从js文件中读取
    let js = "function add(a, b) { return a + b }"
    _ = ctx.evaluateScript(js)
    // 获取add函数对象
    let jsValue = ctx.objectForKeyedSubscript("add")
    // 调用函数add()
    guard let result = jsValue?.call(withArguments: [number1, number2]).toInt32() else { return 0 }
    return result
}
```

##### Javascript调用Objective-C

```
func jsToObjc() {
    // 声明一个JSContext对象
    guard let ctx = JSContext() else { return }
    // 定义一个兼容OC Block的swift闭包
    let addfunction: @convention(block) (_ title: String) -> Void = { (title) in
        print("called from " + title)
    }
    let callNativeBlock = unsafeBitCast(addfunction, to: AnyObject.self)
    // 将Native的方法设置到Context中，
    ctx.setObject(callNativeBlock, forKeyedSubscript: "callNativeBlock" as NSCopying & NSObjectProtocol);
    //执行js代码，调用callNativeBlock方法
    ctx.evaluateScript("callNativeBlock('js')")
}
```

##### JSExport的使用

```
@objc protocol MovieJSExports: JSExport {
    var title: String {get set}
    var imageUrl: String{get set}
    
    static func movieWith(title: String, imageUrl: String) -> Movie
}

class Movie: NSObject, MovieJSExports {
  
  dynamic var title: String
  dynamic var imageUrl: String
  
  init(title: String, imageUrl: String) {
     self.title = title
     self.imageUrl = imageUrl
  }
    
  class func movieWith(title: String, imageUrl: String) -> Movie {
     return Movie(title: title, imageUrl: imageUrl)
  }
}
	
```

#### 总结
****

在对JavaScriptCore进一步研究之后，对这个神奇的框架有了更多更深的认识：

* JavaScriptCore组件在整个Webkit框架中独立存在（除开JavaScriptCore，还有负责Dom渲染，CSS解析等组件），在之前，我们如果要做到Native和JS之间的交互，大多只能通过WebView中提供的方法来处理，这样最大的问题就是必须依赖WebView，然而JavaScriptCore的出现，让JS的运行环境完全独立出来，进而催生了ReactNative、JSPatch这些优秀开源框架的诞生。

#### 未完待续。。。