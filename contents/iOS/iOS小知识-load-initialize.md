---
title: iOS小知识-load()&initialize()
date: 2017-07-02 00:26:48
tags:
---

## 前言
之前在面试中被问到关于 `load` 和 `initialize`，虽然之前在网上有看到过相关的资料，但是却没有仔细看过，也没有实际得在项目中使用过，所以被问到就一脸闷逼。。最近在看一些基础的知识，对于这两个方法也有点了解，所以做个整理。

## 介绍

这两个方法都是NSObject下用来初始化类的类方法。

```swift
//Swift
class func initialize()
//Objective-C
+ (void)load;
在类接受第一个消息前初始化这个类		
```

```swift
//Swift
class func load()
//Objective-C
+ (void)initialize;
每当类或者分类被添加到Objective-C运行时的时候调用；实现此方法在加载的时候执行类的特定的行为。	
```
## load()
对于加入运行时的每个类以及分类来说，一定会调用这个方法，而且仅仅调用一次。
初始化的顺序如下：
注意：
*  一个类的 `+load` 方法会在它所有的父类的 `+load` 方法后调用。
*  一个分类的 `+load` 方法会在类本身的 `+load` 方法之后调用。
*  `+load` 方法不遵循继承规则，  如果某个类本身没有实现 `+load` 方法，不管其父类有无实现 `+load` 方法，系统都不会调用。
*  `+load` 方法务必实现得精简些，尽量减少其所需要执行的操作，整个程序可能因为 `+load` 方法而堵塞。
*  `+load` 方法真正的用途是在调试程序。


问题：
load 方法的问题在于，在执行子类 load 方法之前， 必定会先执行所有父类的 load 方法， 如果还依赖了其他程序库，那么其他程序库里相关类对的 load 方法也必定会先执行。然而，根据某个给定的程序库，却无法判断出其中个各类的载入顺序。所以，在 load 方法中使用其他类是不安全的。

> **重要**
> 对于自定义的 load 方法的实现的 Swift 类桥接到 Objective-C 是不会自动调用的。

### 扩展

我们知道OC 中有一个东西叫: Method Swizzling。我们可以通过它来做埋点的工作，当时这只是它的用法之一，它还有很多用法，这里就不一一说明了。使用Method Swizzling 来做埋点（用户行为统计 统计用户点击事件以及页面跳转等操作，上传到服务器做数据分析等操作）可以使得埋点代码与业务代码分离，可复用性好。这里需要用到 `Runtime ` 方面的知识，这里就不细讲。在使用 `Method Swizzling` 的时候方法交换的操作是在 `load` 方法中进行的，在分类中重写 `load` 方法，而且**Swizzling should always be done in +load.**

> There are two methods that are automatically invoked by the Objective-C runtime for each class. `+load` is sent when the class is initially loaded, while `+initialize` is called just before the application calls its first method on that class or an instance of that class. Both are optional, and are executed only if the method is implemented.
>
> Because method swizzling affects global state, it is important to minimize the possibility of race conditions. `+load` is guaranteed to be loaded during class initialization, which provides a modicum of consistency for changing system-wide behavior. By contrast, `+initialize` provides no such guarantee of when it will be executed—in fact, it may *never* be called, if that class is never messaged directly by the app.

`load `  方法总是会在类初始化的时候被调用，这给改变系统范围的操作提供了保障，然而 `initialize`  方法却不能保证一定会被调用，它可能永远不会被调用如果类一个不被使用（懒加载）。所以 `Swizzling`总是应该写在 `load ` 方法里。

代码如下：

```objective-c
#import <objc/runtime.h>

@implementation UIViewController (Tracking)
//重写load 方法，Swizzling应该总是写在load方法里
+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [self class];
		//获取两个selector
        SEL originalSelector = @selector(viewWillAppear:);
        SEL swizzledSelector = @selector(xxx_viewWillAppear:);
		
      	//通过selector获取方法
        Method originalMethod = class_getInstanceMethod(class, originalSelector);
        Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);

        // When swizzling a class method, use the following:
        // Class class = object_getClass((id)self);
        // ...
        // Method originalMethod = class_getClassMethod(class, originalSelector);
        // Method swizzledMethod = class_getClassMethod(class, swizzledSelector);
		
      	//这里的操作是判断类中是否已经有需要替换的方法的实现。
        BOOL didAddMethod =
            class_addMethod(class,
                originalSelector,
                method_getImplementation(swizzledMethod),
                method_getTypeEncoding(swizzledMethod));
		//如果类中已经有了需要替换的方法的实现，那么就将需要将替换原方法实现的实现替换为原方法的实现(解释起来听绕口的。。233333)，没有就直接交换两个方法的实现。
        if (didAddMethod) {
            class_replaceMethod(class,
                swizzledSelector,
                method_getImplementation(originalMethod),
                method_getTypeEncoding(originalMethod));
        } else {
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
    });
}

#pragma mark - Method Swizzling
//这里的方法来替换原来viewWillAppear，在这个方法里写上埋点的代码
- (void)xxx_viewWillAppear:(BOOL)animated {
    [self xxx_viewWillAppear:animated];
    NSLog(@"viewWillAppear: %@", self);
}

@end
```

参考资料：

* [Method Swizzling](http://nshipster.com/method-swizzling/)
* [通过 Method Swizzling 来实现埋点](http://www.jianshu.com/p/0497afdad36d)

## initialize()

运行时系统会在这个类或者是继承于这个类的子类接受第一条消息之前向这个类发送 `initialize()` 方法。父类在子类之前接受这个消息。

运行时系统通过一种线程安全的方式向类发送`initialize`消息。也就是说，`initialize`方法是在发送给类的第一个消息的第一个线程中执行的，其他的线程只有等到`initialize` 方法执行完成后，才能继续向这个类发送消息。

父类实现的`initialize` 方法可以会被多次调用如果子类没有实现 `initialize` 方法——运行时系统会调用继承的方法实现或者是子类明确调用了 `[super initialize]`. 如果你想保护你自己避免调用多次，你可以按照下面的方法来实现你的`initialize` 方法:

```objective-c
+ (void)initialize {
  if (self == [ClassName self]) {
    // ... do the initialization ...
  }
}
```

因为`initialize()`是以堵塞的方式调用，所以限制类所需要做的事情就很有必要。尤其是任何需要用到锁的代码在其他类的`initialize` 中调用可能造成死锁。因此，你不能依靠 `initialize` 方法来做复杂的初始化，应该在使其变得简单，初始化类的本地变量。

> **特别需要注意的点**
>
> `initialize()` 每个类只会调用一次。如果你给类或者分类做一些独立的初始化，那么你应该去实现 `load()`  方法。

## 总结
* 在加载阶段，如果类实现了 `load()` 方法， 那么系统就会调用它。分类也可以定义此方法，类中的 `load()` 方法比分类中的先调用。与其他方法不同，`load()` 方法不参与重写机制。
* 首次使用某个类之前， 系统会向其发送 `initialize` 消息。此方法遵循普通的重写规则，所以需要在里面判断当前是哪个类需要初始化。
* `load()`、 `initialize()` 方法都需要实现的精简一些，对于程序的相应能力有好处，也避免造成死锁等不必要的麻烦。
* 在 `initialize()` 方法中可以初始化类的一些本地变量。

*本文是对于Effective Objective-C 2.0 这本书的部分总结，参考了苹果官方文档以及一些大神的博客。如果不对的地方请指出。蟹蟹。欢迎交流。*

参考资料：

* [神经病院Objective-C Runtime出院第三天——如何正确使用Runtime](https://halfrost.com/how_to_use_runtime/)

* [iOS动态性(二)可复用而且高度解耦的用户统计埋点实现](http://www.jianshu.com/p/0497afdad36d)

* [Method Swizzling](http://nshipster.com/method-swizzling/)

* [Apple Develop Documendation](https://developer.apple.com/documentation/objectivec/nsobject)

  ​