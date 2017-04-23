---
layout:     post
title:      "策略模式"
subtitle:   ""
date:       2017-04-23
author:     "Darcy"
header-img: ""
tags:
- Pattern
image:
  feature: 
  teaser: playing.jpg
  credit: Death to Stock Photo
  creditlink: ""
---

策略模式(Strategy Pattern)：定义一系列算法，将每一个算法封装起来，并让它们可以相互替换。策略模式让算法独立于使用它的客户而变化。

### 鸭子应用

Tom 做了一套很成功的模拟鸭子游戏。游戏里有各种鸭子，会游泳戏水，会呱呱的叫。游戏的内部设计使用了标准的面向对象思想。设计一个鸭子的父类:  `Duck`  ，并让各种鸭子继承次父类。 

<!--more-->

![](https://ww4.sinaimg.cn/large/006tNc79gy1fewfstrpn2j30py0j2taa.jpg)

由于市场需求，程序要带来会飞的鸭子满足客户。Tom 想作为一个面向对象的程序，这有什么困难的？



![](https://ww3.sinaimg.cn/large/006tNc79gy1fewg2ja9wmj30py0m8jt7.jpg)

看起来，一切如此简单。不料，把游戏推向市场的时候，Tom 竟然看到了一只会飞的橡皮鸭子。。。Tom 赶紧把游戏下线，重新审视问题发生的原因，最终得出解决的方案:把橡皮鸭中的 `fly`  方法覆盖掉，就像覆盖 `quack` 方法一样。



![](https://ww2.sinaimg.cn/large/006tNc79gy1fewgjdj8shj313c0u877n.jpg)



可是以后要再加入别的鸭子，比如诱饵鸭，诱饵鸭是木头假鸭，既不会飞又不会叫，又该如何？



### 面向接口

通过前面的案例，Tom 意识到利用继承可能是一个糟糕的设计。每当有新的鸭子子类出现时，他都需要被迫检查是否需要覆盖 `fly`  和  ` quack`  方法，这简直是无穷无尽的噩梦。 

Tom 想到了另外一种实现：把 `fly`  和  ` quack`  方法从父类中取出来，分别放进 `Flyable` 、 `Quackable`  接口中。这样一来只有会飞的鸭子才实现 `Flyable` 接口，同样只有会叫的鸭子才需要实现 `Quackable`  接口。

![](https://ww4.sinaimg.cn/large/006tNc79gy1fewhi59ds2j31ga0tawix.jpg)



你觉得这种设计如何？

看起来似乎并不怎么高明。这么一来重复的代码会变多。会造成代码的无法复用。每个需要飞的鸭子都需要去实现  `fly`  方法。只能算是从一个噩梦跳进了另一个噩梦。



**设计原则**

> 针对接口编程，而不是针对实现编程。


> 找出应用中可能需要变化之处，把他们独立出来，不要和那些不需要变化的代码混在一起。



![](https://ww1.sinaimg.cn/large/006tNc79gy1fewi0fj3hxj317k0kcaek.jpg)



### 实现鸭子的行为

定义两个接口，以及它们对应的实现类来表示具体的鸭子行为：



![](https://ww1.sinaimg.cn/large/006tNc79gy1fewj2fp35yj31kw0dt76h.jpg)



这样的设计，以及可以让飞行和呱呱叫的行为被其他对象所复用了，因为这些行为以及与鸭子类没有关系了。

同时，即使我们新增一些行为，不会影响到既有的行为类，也不会影响到 “使用” 到飞行行为的鸭子类。



### 整合鸭子的行为

鸭子现在会将飞行和呱呱叫的动作 “委托” 给别人处理，而不是使用自定义在 `Duck`  类（或子类）内的呱呱叫和飞行方法。 



![](https://ww3.sinaimg.cn/large/006tNc79gy1fewjqchx1kj31b80kcwk1.jpg)

**设计原则**

> 多用组合，少用继承。

### 代码实现

`Duck`  类 :

```
@interface Duck : NSObject

@property (nonatomic, strong) id<Quackable> quackBehavior;
@property (nonatomic, strong) id<Flyable> flyBehavior;

- (void)swim;
- (void)display;

- (void)performQuack;
- (void)performFly;

@implementation Duck

- (void)swim
{
    NSLog(@"i can swim");
}

- (void)display
{
    NSLog(@"white color");
}

- (void)performQuack
{
    [self.quackBehavior quack];
}

- (void)performFly
{
    [self.flyBehavior fly];
}
```

接口类：

```
@protocol Flyable <NSObject>

@required

- (void)fly;

@protocol Quackable <NSObject>

@required

- (void)quack;

```

行为类：

```
@interface FlyWithWings : NSObject <Flyable>

@implementation FlyWithWings

- (void)fly
{
    NSLog(@"i can fly");
}

@interface FlyNoWay : NSObject <Flyable>

@implementation FlyNoWay

- (void)fly
{
    NSLog(@"i can't fly");
}
```

鸭子子类：

```
@interface MallardDuck : Duck 

@implementation MallardDuck

- (instancetype)init
{
    if (self = [super init]) {
        self.quackBehavior = [[Quack alloc] init];
        self.flyBehavior = [[FlyWithWings alloc] init];
    }
    return self;
}

- (void)display
{
    NSLog(@"green color");
}

```

测试类：

```
    MallardDuck *mallardDuck = [[MallardDuck alloc] init];
    [mallardDuck performQuack];
    [mallardDuck performFly];
    // 运行时动态的改变飞行行为
    mallardDuck.flyBehavior = [[FlyNoWay alloc] init];
    [mallardDuck performFly];
```

结果：

```
2017-04-22 21:35:55.958 StrategyPattern[2412:141348] quack
2017-04-22 21:35:55.958 StrategyPattern[2412:141348] i can fly
2017-04-22 21:35:55.958 StrategyPattern[2412:141348] i can't fly
```



我们把鸭子的行为当成 “一族算法” ，不同的飞行行为或呱呱叫行为是可以随时替换的。通过这种方式我们很好的实现了代码复用，并且做到了在运行时动态的改变某种行为。这就是 `策略模式`  以及 `组合`  的运用所带来的效果。
