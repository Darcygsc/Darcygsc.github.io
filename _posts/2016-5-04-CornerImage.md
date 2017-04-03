---
layout:     post
title:      "圆角图片的几种实现方式 "
subtitle:   ""
date:       2016-05-04
author:     "Chaos"
header-img: "img/post-sad.jpg"
tags:
    - IOS
image:
  feature: 
  teaser: 03234_lagoadajansen_2880x1800.jpg
  credit: Death to Stock Photo
  creditlink: ""

---
**方形显得过于棱角分明，给用户带来的感觉冷淡不够友好，所以很多图片的展示往往倾向于采用圆角：不管是iPhone上的圆角矩形应用图标，还是QQ上的圆角用户头像。**

### 实现圆角的方式
*****

#### 切个Layer层

这是最简单工作量最少的方式了，只需要我们使用layer的两个属性，简单的两行代码就能搞定圆角的实现：

```
imageView.layer.cornerRadius = 10.0f;  
imageView.layer.masksToBounds = YES;  

```

但事情很少能有完美的解决方案，当这两个属性同时使用时，就会造成离屏渲染问题。由于这样处理的渲染机制是GPU在当前屏幕缓冲区外新开辟一个渲染缓冲区进行工作，这会给我们带来额外的性能损耗，如果在屏幕中充满了这样的圆角图片，当我们在滚蛋的时候，会触发缓冲区的频繁合并和上下文的的频繁切换，性能的代价会宏观地表现在用户体验上----掉帧。所以这种方式只适合在屏幕上圆角图片屈指可数的时候使用。

#### 图片本身带圆角

这种方式针对开发者是最友好的，因为图片本身已经有了圆角的效果，所以自然开发者不需要做任何额外的工作，就跟一张普通的图片一样不会不会增加性能损耗问题。上帝是公平的，当上帝为程序猿打开一扇窗的时候，就将设计师的一扇门了。是的，设计师需要为每张图片切圆角，工作量大大的增加。

#### 通过画图实现圆角

在展示很多张圆角图片的时候，往往我们既不想带来界面卡顿问题，也不想给设计师带来更多的工作量。这个时候就需要我们自己考虑绘制圆角图片了。

既然我们要避免让GPU触发离屏，那么只能通过CPU的方式去做，虽然CPU对图形的处理能力不及GPU，但由于这种处理的难度不大，且代价肯定远小于上下文切换。

这里采用的方式是，**通过UIBezierPath，对图片进行切角。并且保证这部分操作是在子线程CPU内完成，之后我们只需要拿到处理完成的新的UIImage对象，再到主线程交给CPU显示到屏幕上即可。**

```
UIImage *image = nil;
    UIGraphicsBeginImageContextWithOptions(self.originImageView.bounds.size, NO, [UIScreen mainScreen].scale);
    CGContextRef currnetContext = UIGraphicsGetCurrentContext();
    if (currnetContext) {
        CGContextAddPath(currnetContext, [UIBezierPath bezierPathWithRoundedRect:self.originImageView.bounds cornerRadius:self.cornerRadius].CGPath);
        CGContextClip(currnetContext);
        [self.originImageView.layer renderInContext:currnetContext];
        image = UIGraphicsGetImageFromCurrentImageContext();
        UIGraphicsEndImageContext();
    }
    
    if ([image isKindOfClass:[UIImage class]]) {
        image.aliCornerRadius = YES;
        self.originImageView.image = image;
    }
```

在UITableView这样可以滚动的视图中，UIImageView的内容可能由于cell的重用而被重新设置，这样画出来的圆角效果就会因此而消失，所以我们需要对imageView的属性进行监听，一旦发生变化即对改变后的新值再次作切角处理，再次赋值给imageView显示。这里我们监听imageView的image和contentMode属性。

```
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSString*, id> *)change context:(void *)context {
    if ([keyPath isEqualToString:@"image"]) {
        UIImage *newImage = change[@"new"];
        if (![newImage isKindOfClass:[UIImage class]] || newImage.aliCornerRadius) {
            return;
        }
        [self updateImageView];
    }
    if ([keyPath isEqualToString:@"contentMode"]) {
        self.originImageView.image = self.originImage;
    }
}
```



































