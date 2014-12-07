---
layout:     post
title:      "iOS中如何挖出一个透明的图形"
subtitle:   "挖出透明的三角形，圆形，以及五角星"
date:       2014-11-30 12:00:00
author:     "ButtFly"
---

1.__准备工作__  

准备一个 _CGContextRef_ 来构建我们的“画布”。得到一个 _CGContextRef_ 的方式有很多，最常用的有两种：在 _UIView_ 的子类的 _-drawRect:_ 方法中使用 _UIGraphicsGetCurrentContext()_ 得到；还有一种就可以在任何的地方使用，那就是使用 _UIGraphicsBeginImageContext()_ 和 _UIGraphicsEndImageContext()_ 来开始一个画布，并且在他们中间使用 _UIGraphicsGetCurrentContext()_ 得到这个画布进行我们的创作。

```
UIGraphicsBeginImageContext(self.view.bounds.size); //开始画布

CGContextRef contextRef = UIGraphicsGetCurrentContext(); //得到画布

UIGraphicsEndImageContext(); //结束画布
```

认识 _UIBezierPath_ 这个是 _UIKit_ 中给予我们非常方便的制作 _CGPath_ 的类。这里就不对它进行讲解了，不知道的可以在其他地方进行查阅。那么第一步我们先画出一个绿色的背景：设定路径为包含整个视图的矩形，对其填充绿色。当然一切从简，就直接写在 _-viewDidLoad:_ 里面了

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    
    UIGraphicsBeginImageContext(self.view.bounds.size);
    CGContextRef contextRef = UIGraphicsGetCurrentContext();
    UIBezierPath * path = [UIBezierPath bezierPathWithRect:self.view.bounds];
    [[UIColor greenColor] set];
    CGContextAddPath(contextRef, path.CGPath);
    CGContextFillPath(contextRef);
    
    /**这部分代码是在视图的view的layer上面加上背景为根据我们画布创建的layer*/
    UIImage * image = UIGraphicsGetImageFromCurrentImageContext();
    CALayer * layer = [CALayer layer];
    layer.backgroundColor = [[UIColor colorWithPatternImage:image] CGColor];
    layer.frame = self.view.bounds;
    [self.view.layer addSublayer:layer];
    /***/
    UIGraphicsEndImageContext();
}

```

另外，我们要认识下一个东西，那就是 _CGContextClearRect()_ 它的作用就是清理一个矩形区域出来。因此我们要得到中间通明的矩形的话，我们就只需要用它清理一个矩形出来就行。为了方便，我们将视图设置上背景色：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    
    UIGraphicsBeginImageContext(self.view.bounds.size);
    CGContextRef contextRef = UIGraphicsGetCurrentContext();
    UIBezierPath * path = [UIBezierPath bezierPathWithRect:self.view.bounds];
    [[UIColor greenColor] set];
    CGContextAddPath(contextRef, path.CGPath);
    CGContextFillPath(contextRef);
    
    CGContextClearRect(contextRef, CGRectInset(self.view.bounds, 50.0f, 50.0f));
    
    /**这部分代码是在视图的view的layer上面加上背景为根据我们画布创建的layer*/
    UIImage * image = UIGraphicsGetImageFromCurrentImageContext();
    CALayer * layer = [CALayer layer];
    layer.backgroundColor = [[UIColor colorWithPatternImage:image] CGColor];
    layer.frame = self.view.bounds;
    [self.view.layer addSublayer:layer];
    /***/
    UIGraphicsEndImageContext();
}
```

试试看你得到了什么。

