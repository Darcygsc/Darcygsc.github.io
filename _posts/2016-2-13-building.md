---
layout:     post
title:      "如何将代码写的更好看 "
subtitle:   ""
date:       2016-02-13
author:     "Chaos"
header-img: "img/post-sad.jpg"
tags:
    - IOS
---
**一款好的应用应该从内而外都给人以美的感受。不管是产品设计还是编码结构，如果能提供简约明了的直观感受，那么必将吸引更多的人去欣赏。**

## 项目的目录结构

一般来说比较常规的有两种结构：

1.主目录按照业务分类，内目录按照模块分类(主目录按照MVC架构分类，内部根据项目模块分类)

```
优点：相对比较快定位到业务。
缺点：模块相关的类比较分散，寻找相关文件需要来回切换。
```

2.主目录按照模块分类，内目录按照业务分类

```
优点：对模块的类集中化，方便管理与开发。
缺点：模块之间有共用的类时，不太好归类。
```

两种方式其实都OK，更多细节上的修改需要针对具体业务，目前我们采用的是第二种方式。

## 命名

命名作为编程世界的两大难题之一，困扰着很多程序猿。

### 命名的三个原则

```
Expressive, clear, non-ambiguious
表现力，清晰，无歧义
```

#### 类的命名

建议所有的类名都加上前缀。项目大的时候，可能会存在多个大的模块，每个模块有独立的前缀，这样前缀就可以作为“命名空间”来使用，一方面避免命名冲突，另一方面看到类名就知道具体对应的模块。

#### 变量的命名

一般来说首字母小写，之后每隔单词首字母大写。在容易产生歧义的地方，必须指明类型，比如我们在做一个用户头像控件的时候，若果我们使用
下面这这种命名方式，其他人根本不会知道具体的业务是什么。

```
UIImage        *image;  
UIImageView    *imageView;  
NSURL          *url;
```
所以我们需要指明类型，这样其他人就能够一目了然。

```
UIImage        *avatarImage;  
UIImageView    *avatarImageView;  
NSURL          *avatarURL;
```

#### 方法的命名

方法命名的原则：命名需要清楚的指明方法的用途，参数的意义。
首字母应该小写，当方法含有多个参数的时候，如果方法的参数都是receiver 的属性，那么参数之间的链接不需要使用 and 或者 with来链接。

```
- (int)runModalForDirectory:(NSString *)path file:(NSString *) name types:(NSArray *)fileTypes;
```

如果你的方法里的参数来自截然不同的行为，那么可以用 and 或者 with 来链接。

```
- (BOOL)openFile:(NSString *)fullPath withApplication:(NSString *)appName andDeactivate:(BOOL)flag;
```

#### 图片资源的命名

图片的命名不要出现中文，图片的命名通用的方法一般有两种。

```
1.与声明变量类似，都采用驼峰命名法。(loginButtonIcon)
2.全部小写，单词与单词之间采用_链接。(login_button_icon)
```
更倾向于第二种命名方式，因为第一种方式可能会与变量名混淆。

#### 代理的命名

代理有着非常严格的命名规范。这里我们看下苹果官方的命名方式。

```
@protocol UITableViewDelegate<NSObject, UIScrollViewDelegate>
```

以需要发出消息的类名拼接**Delegate**方式命名代理，再看代理方法的命名如何。

```
- (void)tableView:(UITableView *)tableView willDisplayCell:(UITableViewCell *)cell forRowAtIndexPath:(NSIndexPath *)indexPath;
```

首先以发出消息的类名作为开始，之后是需要用到的参数，用will或者did来描述时间发送的状态。

当代理的参数只有一个并且是发出者本身的时候，那么需要将动作拼接到发出者之后。

```
- (void)scrollViewDidScroll:(UIScrollView *)scrollView;  
```

#### Block的命名

一般Block的写法会使代码看起来很长，所以会采用typedef的方式减少代码量，比如著名网络框架AFNetworking定义Block的方式。

```
typedef void (^AFNetworkReachabilityStatusBlock)(AFNetworkReachabilityStatus status);
```

```
@property (readwrite, nonatomic, copy) AFNetworkReachabilityStatusBlock networkReachabilityStatusBlock;
```

#### 通知的命名

通知是一种特殊的通信方式，一个通知对应一个通知名，所以通知名需要定义成一个常量。

```
NSString * const AFNetworkingReachabilityNotificationStatusItem = @"AFNetworkingReachabilityNotificationStatusItem";
```

然后在.h文件中暴露出去，给需要订阅的地方使用。

```
FOUNDATION_EXPORT NSString * const AFNetworkingReachabilityNotificationStatusItem;
```

#### 枚举的命名

苹果官方推荐我们使用**typedef NS_ENUM**方式去定义通知。

```
typedef NS_ENUM(NSInteger, UITableViewStyle) {
    UITableViewStylePlain,          // regular table view
    UITableViewStyleGrouped         // preferences style table view
};
```

枚举的成员名是枚举名拼接上枚举的状态。

## 常量的声明

```
#define KNumber 1
```

推荐使用类型常量，少使用define预处理指令。使用预处理指令有这么几个缺点：
1.定义出来的常量没有类型信息。
2.预处理过程会将碰到的所有KNumber替换为1，不做类型检查，不会报编译错误。

而是用类型常量恰巧能解决这两个问题。所以推荐的写法是:

```
static const NSInteger KNumber = 1;
```

而对于NSString类型的常量，如通知名。推荐使用：

```
NSString * const AFNetworkingReachabilityNotificationStatusItem = @"AFNetworkingReachabilityNotificationStatusItem";
```

需要注意的是，如果常量不是全局的，那么请在前面加上static修饰。
否则可能会产生duplicate symbol的错误。

如果需要声明全局的常量，那么需要在.h 文件中加入。

```
FOUNDATION_EXPORT NSString * const AFNetworkingReachabilityNotificationStatusItem;
```

## 类内部代码划分

一般来说一个类会有很多方法，如果方法写的杂乱无章，会给方法的寻找带来困难，所以需要我们对方法进行划分。类似如下：

```
#pragma mark - LifeCycle

#pragma mark - DataSource && Delegate

#pragma mark - Events Response

#pragma mark - Private Method

#pragma mark - Public Method

#pragma mark - Getter && Setter
```

**只有将代码写的漂亮，之后别人才愿意review你的代码，让别人在看你代码的时候也是一种享受。**