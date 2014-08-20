title: ios 代码实现部分背景透明
date: 2014-08-18 11:34:54
tags: ios
---
先上挖洞代码代码，然后慢慢讲解.. :P

```
UIView *view = [[UIView alloc] initWithFrame:[UIScreen mainScreen].bounds];
    
    view.backgroundColor = [UIColor colorWithRed:0 green:0 blue:0 alpha:0.6];
    UIBezierPath *maskPath = [UIBezierPath bezierPathWithRect:view.bounds];
    
    // Punch a hole :p...
    [maskPath appendPath:[UIBezierPath bezierPathWithRoundedRect:CGRectMake(100, 100, 100, 100) cornerRadius:10]];
    
    CAShapeLayer *maskLayer = [CAShapeLayer layer];
    
    maskLayer.fillRule = kCAFillRuleEvenOdd;
    maskLayer.path = maskPath.CGPath;
    
    view.layer.mask = maskLayer;
    
    [self.view addSubview:view ];
```


####效果图

<img src="https://raw.githubusercontent.com/JasonZengJ/Images/master/blog/ios_punch_hole.png" width=320/>