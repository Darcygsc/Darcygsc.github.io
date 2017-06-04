---
layout:     post
title:      "复合模式"
subtitle:   ""
date:       2017-06-04
author:     "Darcy"
header-img: ""
tags:
- Pattern
image:
  feature: 
  teaser: horse.jpg
  credit: Death to Stock Photo
  creditlink: ""
---

复合模式( `Compound Pattern` )：在一个解决方案中结合两个或多个模式，以解决一般货重复发生的问题。

## 与鸭子重聚

之前一篇介绍[策略模式](http://chaosgsc.com/Strategy.html)的文章，我们与鸭子深度合作过。现在我们将从头重建我们的鸭子模拟器。

<!--more-->

### 创建会呱呱叫的鸭子

首先我们制造一个接口，这个接口只做一件事：呱呱叫。

```objective-c
@protocol Quackable <NSObject>

// 呱呱叫方法
- (void)quack;

@end
```

让某些鸭子实现这个接口：

```objective-c
@interface MallardDuck () <Quackable>

@end

@implementation MallardDuck

- (void)quack
{
    NSLog(@"Quack");
}

@end
```

```objective-c
@interface RubberDuck () <Quackable>

@end

@implementation RubberDuck

- (void)quack
{
    NSLog(@"Squeak");
}

@end
```

模拟创建鸭子，让鸭子呱呱叫起来：

```objective-c
id<Quackable> mallardDcuk = [[MallardDuck<Quackable> alloc] init];
id<Quackable> rubberDcuk = [[RubberDuck<Quackable> alloc] init];

[self simulate:mallardDcuk];
[self simulate:rubberDcuk];
```

```objective-c
- (void)simulate:(id<Quackable>)duck
{
    [duck quack];
}
```

现在鸭子已经能够正确的发出呱呱叫的声音了：

![](https://ww1.sinaimg.cn/large/006tNbRwgy1fg93f8rd8wj30li022q3j.jpg)


#### 适配器

有水塘的地方，不仅有鸭子，还会有鹅：

```objective-c
@implementation Goose

- (void)honk
{
    NSLog(@"Honk");
}

@end
```

鹅不会呱呱叫，所以鹅没有 `quack` 方法。那么怎样才能让鹅与鸭子掺杂在一起呢？
我们可以把鹅包装进适配器，将鹅适配成鸭子。

```objective-c
@interface GooseAdapter () <Quackable>

@property (nonatomic, strong) Goose *goose;

@end

@implementation GooseAdapter

- (instancetype)initWithGoose:(Goose *)goose
{
    if (self = [super init]) {
        _goose = goose;
    }
    return self;
}

- (void)quack
{
    [_goose honk];
}

@end
```

让适配器对象 `GooseAdapter` 实现 `Quackable` 接口，这样就可以让鹅像鸭子一样了。

```objective-c
id<Quackable> mallardDcuk = [[MallardDuck<Quackable> alloc] init];
id<Quackable> rubberDcuk = [[RubberDuck<Quackable> alloc] init];
Goose *goose = [[Goose alloc] init];
id<Quackable> gooseAdapter = [[GooseAdapter<Quackable> alloc] initWithGoose:goose];

[self simulate:mallardDcuk];
[self simulate:rubberDcuk];
[self simulate:gooseAdapter];
```

![](https://ww4.sinaimg.cn/large/006tNbRwgy1fg9420c8zyj30lm036jsc.jpg)

### 计算呱呱叫的次数

在一群鸭子中，会有多少呱呱叫声？如何在不改变鸭子类的情况下，计算出呱呱叫的次数呢？

#### 装饰者

通过创建一个装饰者，通过把鸭子包装进装饰者对象，给鸭子一些新的行为。这样就不必修改鸭子类的代码。

```objective-c
static int numberOfQuacks;

@interface QuackCounter () <Quackable>

@property (nonatomic, strong) id<Quackable> duck;

@end

@implementation QuackCounter

- (instancetype)initWithGoose:(id<Quackable>)duck
{
    if (self = [super init]) {
        _duck = duck;
    }
    return self;
}

- (void)quack
{
    [_duck quack];
    numberOfQuacks ++;
}

+ (int)getQuacks
{
    return numberOfQuacks;
}

@end
```
让我们看看测试情况：

```objective-c
id<Quackable> mallardDcuk = [[QuackCounter<Quackable> alloc] initWithDuck:[[MallardDuck<Quackable> alloc] init]];
id<Quackable> rubberDcuk = [[QuackCounter<Quackable> alloc] initWithDuck:[[RubberDuck<Quackable> alloc] init]];
Goose *goose = [[Goose alloc] init];
id<Quackable> gooseAdapter = [[GooseAdapter<Quackable> alloc] initWithGoose:goose];

[self simulate:mallardDcuk];
[self simulate:rubberDcuk];
[self simulate:gooseAdapter];

NSLog(@"The ducks quacked , %d times", [QuackCounter getQuacks]);
```
在这里我们并不想统计鹅叫的次数，所以并没有把鹅包装起来，所以结果只是鸭子叫了2次。

![](https://ww1.sinaimg.cn/large/006tNbRwgy1fg94pto0bej30qy04c0u6.jpg)

### 规模化的生产

我们需要创建大量的鸭子，目前的方式显然不能很好的满足我们。对！我们需要一个工厂，来系统化的管理生产。

#### 抽象工厂

`objective-c` 没有抽象类的概念，所以我们在使用抽象工厂模式的时候，使用 `Protocol` ，让工厂对象实现这个协议。

```objective-c
@protocol CreateDuckProtocol <NSObject>

- (id<Quackable>)createMallardDuck;

- (id<Quackable>)createRubberDuck;

@end
```

```objective-c
@interface DuckFactory () <CreateDuckProtocol>

@end

@implementation DuckFactory

- (id<Quackable>)createMallardDuck
{
    return [[QuackCounter<Quackable> alloc] initWithDuck:[[MallardDuck<Quackable> alloc] init]];
}

- (id<Quackable>)createRubberDuck
{
    return [[QuackCounter<Quackable> alloc] initWithDuck:[[RubberDuck<Quackable> alloc] init]];
}

@end
```

```objective-c
id<CreateDuckProtocol> duckFactory = [[DuckFactory<CreateDuckProtocol> alloc] init];

id<Quackable> mallardDcuk = [duckFactory createMallardDuck];
id<Quackable> rubberDcuk = [duckFactory createRubberDuck];
Goose *goose = [[Goose alloc] init];
id<Quackable> gooseAdapter = [[GooseAdapter<Quackable> alloc] initWithGoose:goose];

[self simulate:mallardDcuk];
[self simulate:rubberDcuk];
[self simulate:gooseAdapter];

NSLog(@"The ducks quacked , %d times", [QuackCounter getQuacks]);
```

### 管理鸭子

当我们有自己的鸭子工厂后，我们可以肆无忌惮的的生产鸭子了。但是鸭子多了，如何去有效的管理它们呢？我们想要所有的鸭子分为几个集合，每次传达一个命令都能让整个集合的鸭子听命行事。

#### 组合

组合模式允许我们像对待单个对象一样对待对象的集合。

```objective-c
@interface Flock () <Quackable>

@property (nonatomic, strong) NSMutableArray *array;

@end

@implementation Flock

- (void)add:(id<Quackable>)quacker
{
    [self.array addObject:quacker];
}

- (void)quack
{
    for (id<Quackable> quacker in self.array) {
        [quacker quack];
    }
}

- (NSMutableArray *)array
{
    if (!_array) {
        _array = [NSMutableArray array];
    }
    return _array;
}

@end
```

```objective-c
id<CreateDuckProtocol> duckFactory = [[DuckFactory<CreateDuckProtocol> alloc] init];

id<Quackable> mallardDcuk = [duckFactory createMallardDuck];

Flock<Quackable> *flockOfDuck = [[Flock<Quackable> alloc] init];
[flockOfDuck add:mallardDcuk];

id<Quackable> rubberDcukOne = [duckFactory createRubberDuck];
id<Quackable> rubberDcukTwo = [duckFactory createRubberDuck];
id<Quackable> rubberDcukThree = [duckFactory createRubberDuck];

Flock<Quackable> *flock = [[Flock<Quackable> alloc] init];
[flock add:rubberDcukOne];
[flock add:rubberDcukTwo];
[flock add:rubberDcukThree];

[flockOfDuck add:flock];

NSLog(@"The Whole Ducks:");
[self simulate:flockOfDuck];

NSLog(@"The Rubber Ducks:");
[self simulate:flock];

NSLog(@"The ducks quacked , %d times", [QuackCounter getQuacks]);
```

执行结果：

![](https://ww2.sinaimg.cn/large/006tNbRwgy1fg964d8rs2j30ty08aadz.jpg)

我们成功的对不同的鸭子下发了命令，执行命令的结果和我们想要的一样。

### 模式总结

这个例子中我们使用了四种不同的设计模式。

**适配器**
> 封装对象，并提供不同的接口。

**装饰者**
> 包装一个对象，以提供新的行为。

**抽象工厂**
> 运行客户创建对象的家族，而无需指定他们的具体类。

**组合**
> 客户用一致方法处理对象集合和单个对象。