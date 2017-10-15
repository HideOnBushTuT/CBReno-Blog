---
title: GCD
date: 2017-06-13 15:36:37
tags:
---

## 概念

**多线程编程** 先来看一下下面的代码。

```objective-c
int main() {
  id o = [[MyObject alloc] init];
  [o execBlock];
  return 0;
}
```

上面的代码最终会转化为机器能够识别的指令（二进制代码）。然后这些命令在 CPU 中一句一句的执行。由于一个 CPU 一次只能执行一个命令，不能执行某处分开的并列的两个命令，因此通过 CPU 执行的 CPU 命令就好像一条无分叉的大道，其执行不会出现分歧。这里所说的“一个 CPU 执行的 CPU 命令为一条无分叉路径”即为“线程”。

在单个 CPU的情况下， 使用多线程的程序可以在某个线程和其他线程之间反复的进行上下文切换，因此看上去好像一个 CPU 可以并列执行多个线程。在具有多个 CPU  的情况下，就是真的提供了多个 CPU 并行执行多个线程的任务了。

当然多线程编程也会带来一定的问题：

- 多个线程更新相同的资源导致数据的不一致（数据竞争）
- 多个线程相互等待导致的死锁问题
- 开启多个线程会消耗大量内存

在 iOS 程序启动时，最先执行的线程，就是主线程，主线程用来描绘用户界面、处理触摸屏幕的事件等。 如果在这个线程中进行长时间的操作，就会妨碍主线程的执行，妨碍主线程中的 **RunLoop** 更新，从而导致不能更新用户界面、应用画面长时间停滞等问题。

**GCD** (Grand Central Dispatch)  通过 GCD 开发者只需要定义想要执行的任务并追加到适当的 Dispatch Queue 中， GCD 就能生成必要的线程并计划执行任务。它将线程管理作为系统的一部分来实现的，因此可统一管理，这样就相对来说更加有效率。

## GCD 简单使用

### 重要知识点

- async/sync
  - sync 同步任务，会堵塞当前线程等待任务执行完
  - async 异步任务，不会堵塞当前线程，只要有任务就会继续执行
- Dispatch Queue(列队)
  - Serial Dispatch Queue (等待前一个任务完成后才能继续执行下一个任务)
    - Global Dispatch Queue(系统系统的全局的同步列队)
  - Concurrent Dispatch Queue（不必等待前一个任务执行完成也可以继续执行下一个任务）
    - Main Dispatch Queue(系统主线程，主要绘制用户界面、处理屏幕事件)

> 使用方法： 将指定的任务追加到适当的 Dispatch Queue 中

```swift
//Swift
DispatchQueue.global().async {
	/**
	 *想要执行的任务
	 */
}
```

```objective-c
//Objective-C	
dispatch_async(queue, ^{
  /*
   *想要执行的任务
   */
})
```

### 创建列队

- Objective-C

```objective-c
//Serial Dispatch Queue
dispatch_queue_t serailDispatchQueue = dispatch_queue_create("com.cbreno.gcd.objective.serial, NULL);

dispatch_queue_t serialDispatchQueue = dispatch_queue_create("com.cbreno.gcd.objectivec.serial", DISPATCH_QUEUE_SERIAL);                                                            
```

```objective-c
//Concurrent Dispatch Queue
dispatch_queue_t concurrentDispatchQueue = dispatch_queue_create("com.cbreno.gcd.objectivec.concurrent", DISPATCH_QUEUE_CONCURRENT);
```

- Swift

```swift
//Serial Dispatch Queue 
let serialQueue = DispatchQueue(lable: "com.cbreno.gcd.swift.serial")
```

```swift
//Concurrent Dispatch Queue
let concurrentQueue = DispatchQueue(lable: "com.cbreno.gcd.swift.concurrent", attributes: .concurrent)
```

- Serial Dispatch Queue 一旦生成并且追加处理，系统对于一个 Serial Dispatch Queue 就只生成并使用一个线程。如果过多使用多线程，会消耗大量内存，引起大量的上下文切换，大幅度江都一同的响应性能。
- 为了避免多线程更新相同资源导致数据竞争问题，使用 Serial Dispatch Queue。
- Dispatch Queue 的名称推荐使用应用程序ID的逆序全程域名，这么做的好处是在Xcode 和 Instruments 调试的时候很方便观察。另外应用奔溃时产生的 CrashLog 中很容易找到对应线程。

### Main Dispatch Queue/Global Dispatch Queue

- Main Dispatch Queue

```objective-c
dispatch_queue_t mainQueue = dispatch_get_main_queue();
```

```swift
//Swift中这是一个 DispatchQueue 类型的变量
let mainQueue = DispatchQueue.main
```

Main Dispatch Queue 是主线程中执行的 Dispatch Queue ，是一个 Serial Dispatch Queue。追加到 Main Dispatch Queue 的处理在主线程的 RunLoop  中执行。主线程主要处理用户界面的渲染、触摸事件等， 所以需要长时间的操作，最好在别的线程中执行，执行完后再在主线程中刷新UI等操作。

- Global Dispatch Queue 

```objective-c
//Objective-C
//DISPATCH_QUEUE_PRIORITY_HIGH
dispatch_queue_t globalDispatchQueueHigh = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0);

//DISPATCH_QUEUE_PRIORITY_DEFAULT
dispatch_queue_t globalDispatchQueueHigh = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

//DISPATCH_QUEUE_PRIORITY_LOW
dispatch_queue_t globalDispatchQueueHigh = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0);

//DISPATCH_QUEUE_PRIORITY_BACKGROUND
dispatch_queue_t globalDispatchQueueHigh = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0);
```

```swift
//Swift
//userInitiated
let globalQueue = DispatchQueue.global(qos: DispatchQoS.QoSClass.userInitiated)

//default
let globalQueue = DispatchQueue.global(qos: DispatchQoS.QoSClass.default)

//utility
let globalQueue = DispatchQueue.global(qos: DispatchQoS.QoSClass.utility)

//background
let globalQueue = DispatchQueue.global(qos: DispatchQoS.QoSClass.background)
```

```swift
 //在Swift中优先级是一个 enum 类型
   public enum QoSClass {

        case background

        case utility

        case `default`

        case userInitiated

        case userInteractive

        case unspecified

        public init?(rawValue: qos_class_t)

        public var rawValue: qos_class_t { get }
    }
```

- 我们可以直接使用系统提供的这两个Dispatch Queue 帮助我们完成一般操作，像下面一样：

```objective-c
//Objective-C
//Global Dispatch Queue
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0) ^{
  //处理长时间的操作
      .....
  //使用Main Dispatch Queue 在主线程中操作
  dispatch_async(dispatch_get_main_queue(), ^{
    //只能在主线程中执行的操作
  });
});
```

```swift
//Swift
//Global Dispatch Queue
DispatchQueue.global().async {
//处理长时间的操作
	.....
  //使用Main Dispatch Queue 在主线程中操作
	DispatchQueue.main.async {
	//只能在主线程中执行的操作
	}
}
```

- Dispatch After
  当需要延迟执行的情况就要用到 Dispatch After，在指定的时间后将指定的 Block 追加到指定的Dispatch Queue 中。

```objective-c
//Objective-C
dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW, 3ull * NSEC_PER_SEC);
dispatch_after(time, dispatch_get_main_queue(), ^{
  ..........
});
```

- 第一个参数是指定需要延迟的时间，是dispatch_time_t 类型的值。

  dispatch_time 方法的第一个参数是 dispatch_time 类型的有两种值：

```objective-c
 //A somewhat abstract representation of time; where zero means "now" and
  #define DISPATCH_TIME_NOW (0ull) 
  //DISPATCH_TIME_FOREVER means "infinity" and every value in between is an
  #define DISPATCH_TIME_FOREVER (~0ull)
```

  dispatch_time 方法的第二个参数有以下几种类型：

```objective-c
  #define NSEC_PER_SEC 1000000000ull
  #define NSEC_PER_MSEC 1000000ull
  #define USEC_PER_SEC 1000000ull
  #define NSEC_PER_USEC 1000ull  
```

  `  ull` 表示 `unsigned long long ` 。

- 第二个参数是需要追加处理的 Dispatch Queue。
- 第三个参数是需要追加的 Block。

```swift
//Swift
let delay = DispatchTime.now() + DispatchTimeInterval.seconds(60)
//let delay = DispatchTime.now() + 3.0
DispatchQueue.main.asyncAfter(deadline: delay) { 
	//需要延迟执行的任务
}
```

- 在定义延迟时间的时候可以直接在 DispatchTime.now() 方法后面加上需要的延迟的时间，是因为苹果定义了 `+` 方法，使得它可以直接操作。

```swift
//Swift
public func +(time: DispatchTime, seconds: Double) -> DispatchTime
```

当然还定义了一些其他的方法：

```swift
//Swift
public func +(time: DispatchWallTime, seconds: Double) -> DispatchWallTime

public func +(time: DispatchTime, interval: DispatchTimeInterval) -> DispatchTime

public func +(time: DispatchWallTime, interval: DispatchTimeInterval) -> DispatchWallTime

public func -(time: DispatchTime, interval: DispatchTimeInterval) -> DispatchTime

public func -(time: DispatchTime, seconds: Double) -> DispatchTime

public func -(time: DispatchWallTime, interval: DispatchTimeInterval) -> DispatchWallTime

public func -(time: DispatchWallTime, seconds: Double) -> DispatchWallTime

public func <(a: DispatchTime, b: DispatchTime) -> Bool

public func <(a: DispatchWallTime, b: DispatchWallTime) -> Bool

public func ==(a: DispatchTime, b: DispatchTime) -> Bool

public func ==(a: DispatchQoS, b: DispatchQoS) -> Bool

public func ==(a: DispatchWallTime, b: DispatchWallTime) -> Bool
```

### Dispatch Group

如果有这样的需求，需要等待所有的线程中的任务都执行完成后，做统一的处理。通常为多个网络请求操作，需要等待所有的网络请求完成后，解析数据，统一刷新UI。这时候我们就可以通过 Dispatch queue 来完成我们的操作。当然在使用一个 Serial Dispatch Queue 的时候只要将所有的任务追加到其中并且在最后处理结果即可实现。但是在使用 Concurrent Dispatch Queue 的时候就使用 Dispatch Group 比较方便。

```objective-c
//Objective-C
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_group_t group = dispatch_group_create();
    
    //追加任务
dispatch_group_async(group, queue, ^{
	NSLog(@"bloack0");
});
dispatch_group_async(group, queue, ^{
	NSLog(@"bloack1");
});
dispatch_group_async(group, queue, ^{
	NSLog(@"bloack2");
});
    
    //在所有任务执行完成后，通知。。。
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
	NSLog(@"done");
});
```

```swift
//Swift
 let group = DispatchGroup()
        
let queue = DispatchQueue.global()
        
	queue.async(group: group) { 
		print("TEST0")
	}
	queue.async(group: group) {
		print("TEST1")
	}
	queue.async(group: group) {
		print("TEST2")
	}
        
	group.notify(queue: DispatchQueue.main) { 
		print("done")
	}
```

### Dispatch barrier

在一些情况下，当我们在处理一些事情的时候，要处理另外一些事情的时候，需要停止当前做的事情的时候，我们可能需要 Dispatch barrier 来帮助我们完成任务。

```objective-c
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    
    dispatch_async(queue, ^{
        NSLog(@"read0");
    });
    
    dispatch_async(queue, ^{
        NSLog(@"read1");
    });
    
    //在读取数据的过程中，执行写操作，就要停止读操作，只执行写操作，这样不会造成数据冲突的问题。
    dispatch_barrier_async(queue, ^{
        NSLog(@"write");
    });
    
    dispatch_async(queue, ^{
        NSLog(@"read2");
    });
    
    dispatch_async(queue, ^{
        NSLog(@"read3");
    });
```

### 相关资料

[Swift 3必看：从使用场景了解GCD新API(卓同学)](http://www.jianshu.com/p/fc78dab5736f)
[Concurrent Programming With GCD in Swift 3](https://developer.apple.com/videos/play/wwdc2016/720/)
[关于iOS多线程，你看我就够了](http://www.jianshu.com/p/0b0d9b1f1f19)

#### 这篇大部分是对《Objective-C 高级编程 iOS 与 OS X 多线程和内存管理》这本书的总结。 暂时写这么多了。。之后再补充吧。。