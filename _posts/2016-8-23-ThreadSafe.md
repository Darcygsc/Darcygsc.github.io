---
layout:     post
title:      "重新审视iOS线程安全"
subtitle:   ""
date:       2016-08-23
author:     "Chaos"
header-img: "img/post-sad.jpg"
tags:
	- IOS
image:
  feature: 
  teaser: computer-teaser.jpg
  credit: Death to Stock Photo
  creditlink: ""
---


### Mutable
很多稍微有点iOS 编程经验的人都知道，在多线程的环境下，当我们向一个`NSMutableArray`中增加元素时，会导致程序crash：
![][image-1]
### Immutable
那么如果我们操作的是一个`Immutable`对象呢？ 很多人会觉得不可变对象在线程间传递的是一份只读文件了，那么应该就是线程安全的了。所以在使用`NSArray`的时候，可能会肆无忌惮，而完全忽视了多线程的环境。

<!--more-->

![][image-2]

**这里的不可变，仅仅指的是`Immutable`对象本身。编程过程中，是需要指针指向对象。**

在对对象进行赋值的过程中，其实是在更改指针的指向。在多线程环境下，用一个指针指向一个`Immutable`对象的时候，当指针发生修改时，会出现一些百思不得其解的问题:

\`\`\`objc
self.array = @[@"A",@"B",@"C"];  //线程A
self.array = @[@"A",@"B",@"C",@"D"]; //线程B

[self.array objectAtIndex:4]; //主线程
\`\`\`

---- 
\`\`\`objc
 if (self.key) {  // 主线程
	 [self.dict setObject:@"ops" forKey:self.key];  
 }
 self.key = nil;  // 线程A
\`\`\`



上面两种情况都有可能在多线程的环境中出现问题。解决这个问题，一般我们使用**局部变量**

#### 属性Setter的问题
##### 在MRC时代，系统默认生成的setter像这样：
\`\`\`objc
 - (void)setUserName:(NSString \*)userName {
	 if(\_uesrName != userName) {
		 [userName retain];
		 [_userName release];
		 _userName = userName;
	 }
 }
\`\`\`

假设一种场景，线程1先通过`getter`获取当前`\_userName`，之后线程2通过`setter`调用`\[\[\_userName release]]`，线程1所持有的`\_userName`就变成无效的地址空间了，如果再给这个地址空间发消息就会导致crash，出现多线程不安全的场景。

##### ARC时代的处理，变得复杂很多了：
首先看下Objective-C的[底层源码]对`Setter`的实现：

\`\`\`objc
 static inline void reallySetProperty(id self, SEL \_cmd, id newValue, ptrdiff\_t offset, bool atomic, bool copy, bool mutableCopy)
 {
	 if (offset == 0) {
	     object_setClass(self, newValue);
	     return;
	 }
	
	 id oldValue;
	 id slot = (id) ((char*)self + offset);
	
	 if (copy) {
	     newValue = [newValue copyWithZone:nil];
	 } else if (mutableCopy) {
	     newValue = [newValue mutableCopyWithZone:nil];
	 } else {
	     if (*slot == newValue) return;
	     newValue = objc_retain(newValue);
	 }
	 if (!atomic) {  // 危险区
	     oldValue = *slot;
	     *slot = newValue;
	 } else {
	     spinlock_t& slotlock = PropertyLocks[slot];
	     slotlock.lock();
	     oldValue = *slot;
	     *slot = newValue;        
	     slotlock.unlock();
	 }
	 objc_release(oldValue);
 }
\`\`\`



通过源码可以发现：

> 针对`atomic`的情况，系统会默认加上锁进行保护。所以当我们对属性设置`atomic`的时候，在做赋值操作时候是线程安全的。

> 对象如果是`nonatomic`，所以逻辑会走到上述注释危险区处。还是设想一下多线程对一个属性同时设置的情况，我们首先在线程A处获取到了执行第一步代码后的`oldValue`，然后此时线程切换到了B，B也获得了第一步后的`oldValue`，所以此时就有两处持有`oldValue`。然后无论是线程A或者线程B执行到最后都会执行`objc\_release(oldValue)`。程序走到这里就会crash了。


`nonatomic`的情况比较特殊，在`nonatomic`判断处理之前。苹果分别针对 `offset`、`copy`、`mutableCopy` 做了判断。

苹果在调用`objc\_setProperty`的示例代码是这样的：


\`\`\`objc
 - (void)setValue:(NSString\*)inValue { objc\_setProperty(self, _cmd, offsetof(TestDefs, _value), inValue, YES, YES); }

 - (void)setObject:(id)inObject { objc\_setProperty(self, _cmd, offsetof(TestDefs, _object), inObject, YES, NO); }
\`\`\`



这里`NSString`类型和id类型对`mutableCopy`的复杂是不一样的。差别的原因可能在于`NSString`实现了`NSMutableCopying`协议。

首先对offset进行判断:

\`\`\`objc
 if (offset == 0) {
	     object_setClass(self, newValue);
	     return;
 }
\`\`\`

在调用处，传递给offset的值为`offsetof(TestDefs, \_value)`。

`TestDefs`为结构体:

\`\`\`objc
typedef struct {  
     void \*isa;
	 void *_value;
	 // _object is at the last optimized property offset
	 void *object _attribute__((aligned(64)));
 } TestDefs;
\`\`\`

C语言函数 `offsetof(type, member-designator) `会生成一个类型为 `size\_t `的整型常量，它是一个结构成员相对于结构开头的字节偏移量。

在offset==0的时候，会调用`object\_setClass(self, newValue)`; 然后直接返回。这也是我们即使给属性设置`nonatomic`，并在多线程环境下操作`NSString`、`NSData`的时候不会出现任何问题的原因：

![][image-3]

![][image-4]

对于那些offset != 0 并且实现了`NSCopying` 或 `NSMutableCopying` 的情况，最终都会毫无例外的面临线程不安全的问题。

![][image-5]

那么对于那些offset !=0 又没有实现`NSCopying` 或 `NSMutableCopying`的情况呢？
我们以`NSCache`作为示例来测试：

![][image-6]

发现在循环`500W`次的情况下，有很小的概率出现crash的问题。这里的原因主要是苹果在`reallySetProperty`方法里做了一定程度的优化：

\`\`\`objc
else {
	     // 某些程度的优化
	     if (*slot == newValue) return;
	     newValue = objc_retain(newValue);
	 }
\`\`\`


优化只是减少了crash发生的概率，并不是杜绝了crash的发生。

**如果我们想在多线程环境下，对属性进行赋值。如果是`NSString`或者`NSData`这样每次生成的对象地址都一样的情况，完全不用顾虑大胆使用吧。但是除此以外的其它情况，还是老实的设置`atomic`吧。**


### 总结
尽量避免多线程的设计，多线程是个复杂的问题，即使是经验丰富的程序员，在面对多线程的时候也很容易放错。解决多线程，没有绝对的银弹。































[image-1]:	http://s2.mogucdn.com/p2/170330/1_3j0d7ib65gd27blbh9a5ig61ejhi4_984x146.png
[image-2]:	http://s2.mogucdn.com/p2/170330/1_05a6i8gi198862a0j1ea116k77f7k_410x427.png
[image-3]:	http://s2.mogucdn.com/p2/170330/1_597f6g4bkhh71j2k536f56ijfe5gc_1031x146.png
[image-4]:	http://s2.mogucdn.com/p2/170330/1_78d81kfe91h41827h0a645kha98l9_383x403.png
[image-5]:	http://s2.mogucdn.com/p2/170330/1_550288ed1bk229e14gkcdigl99l8d_976x154.png
[image-6]:	http://s2.mogucdn.com/p2/170330/1_2gl5fhac00e6ajd4ehhik73f8b72f_1013x119.png