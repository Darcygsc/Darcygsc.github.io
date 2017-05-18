---
layout:     post
title:      "单例模式"
subtitle:   ""
date:       2017-05-15
author:     "Darcy"
header-img: ""
tags:
- Pattern
image:
  feature: 
  teaser: trees.jpg
  credit: Death to Stock Photo
  creditlink: ""
---

单例模式( `Singleton Pattern` )：确保一个类只有一个实例，并提供一个全局访问点。

### 实现( `Java`)

- 利用一个静态变量来记录 `Singleton` 类的唯一实例。

```
private static Singleton uniqueInstace;

```

- 把构造方法声明为私有的，只有 `Singleton` 类内部才可以访问。

```
private Singleton(){}

```

- 用 `sharedInstance` 方法实例化对象，并返回这个实例。

```
public static Singleton getInstance(){
  if (uniqueInstace == null){
  	uniqueInstace = new Singleton();
  }
  return uniqueInstace;
}
```

在多线程环境下，当两个线程同时进入 `if` 语句内时，就会产生两个 `Singleton` 对象。

#### 处理多线程

只要把 `getInstance()` 变成同步 (`synchronized`) 方法，多线程问题就可以迎刃而解了。

```
public static synchronized Singleton getInstance(){
  if (uniqueInstace == null){
  	uniqueInstace = new Singleton();
  }
  return uniqueInstace;
}
```

这种方式可以顺利的解决多线程问题，但是每次执行方法都会进行同步操作，同步会大大降低性能，如何避免这种问题呢？

- 饿汉式

```
public class Singleton{
    private static final Singleton instance = new Singleton();
    private Singleton(){}
    public static Singleton getInstance(){
        return instance;
    }
}
```

然而这种方式又会引入另外一个问题：它不是一种懒加载模式（`lazy initialization`），单例会在加载类后一开始就被初始化，即使客户端没有调用 `getInstance()` 方法。

- 双重检查加锁

```
public static Singleton getSingleton() {
    if (instance == null) {                         
        synchronized (Singleton.class) {
            if (instance == null) {                 
                instance = new Singleton();
            }
        }
    }
    return instance ;
}
```

看起来好像完美了，但是它还有问题。` instance = new Singleton();` 并不是一个原子操作，`JVM` 中这段代码大概做了下面 3 件事情：

1.给 instance 分配内存
2.调用 Singleton 的构造函数来初始化成员变量
3.将instance对象指向分配的内存空间（执行完这步 instance 就为非 null 了）

`JVM` 的即时编译器中存在指令重排序的优化，2和3的执行顺序是不能确定的。如果 3 执行完毕、2 未执行之前，被另一个线程抢占了，那么就会返回一个为初始化的 `instance`。

- `volatile` 关键字

```
public class Singleton {
    private volatile static Singleton instance; 
    private Singleton (){}
    public static Singleton getSingleton() {
        if (instance == null) {                         
            synchronized (Singleton.class) {
                if (instance == null) {       
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

`volatile` 可以禁止指令重排序优化。在 `volatile` 变量的赋值操作后面会有一个内存屏障（生成的汇编代码上），读操作不会被重排序到内存屏障之前。

- 静态内部类

```
public class Singleton {  
    private static class SingletonHolder {  
        private static final Singleton INSTANCE = new Singleton();  
    }  
    private Singleton (){}  
    public static final Singleton getInstance() {  
        return SingletonHolder.INSTANCE; 
    }  
}
```
由于内部类 `SingletonHolder` 是私有的，除了 `getInstance()` 之外没有办法访问它，因此它是懒汉式的；同时读取实例的时候没有同步操作，自然而然不会带来性能问题。

### 实现( iOS)

苹果官方的 `Simple Code` 的实现像这样：

```
static MyGizmoClass *sharedGizmoManager = nil; 
+ (MyGizmoClass*)sharedManager
{
    @synchronized(self) {
        if (sharedGizmoManager == nil) {
            sharedGizmoManager = [[self alloc] init]; // assignment not done here
        }
    }
    return sharedGizmoManager;
}

+ (id)allocWithZone:(NSZone *)zone
{
    @synchronized(self) {
        if (sharedGizmoManager == nil) {
            sharedGizmoManager = [super allocWithZone:zone];
            return sharedGizmoManager;  // assignment and return on first allocation
        }
    }
    return nil; //on subsequent allocation attempts return nil
}

- (id)copyWithZone:(NSZone *)zone
{
    return self;
}

- (id)retain
{
    return self;
}

- (unsigned)retainCount
{
    return UINT_MAX;  //denotes an object that cannot be released
}

- (void)release
{
    //do nothing
}

- (id)autorelease
{
    return self;
}
```

重写 `allocWithZone` 方法，用来保证直接使用 `alloc` 和 `init` 试图获得一个新实力的时候不产生一个新实例。

`Objective－C` 中没有绝对的私有方法，可以使用 `NS_UNAVAILABLE` 将一些方法设置为不可用。

```
- (instancetype)init NS_UNAVAILABLE;

+ (instancetype)new NS_UNAVAILABLE;

+ (instancetype)alloc NS_UNAVAILABLE;
```

#### dispatch_once

在 `ARC` 时代，更多的是使用苹果官方推出的 `GCD` 来完成单例模式：
```
+ (Singleton *)sharedInstance{
    static Singleton *instance = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        instance = [[self alloc] init];
    });
    return instance;
}
```

`dispatch_once` 函数有两个参数，第一个参数 `predicate` 用来保证执行一次，第二个参数是要执行一次的任务 `block`。`dispatch_once` 函数已经为我们处理好了多线程的安全问题。