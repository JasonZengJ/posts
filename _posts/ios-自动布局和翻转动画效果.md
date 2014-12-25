title: iOS 自动布局和翻转动画效果
date: 2014-08-08 23:22:47
tags: [iOS,Design]
---
用swift写的一个ios自动布局和翻转效果的小demo，github 地址：[GridPanelDemo](https://github.com/JasonZengJ/GridPanelDemo)

自动布局关键代码块：

```
 self.setTranslatesAutoresizingMaskIntoConstraints(false)//必须设置为false，否则无法实现自动布局

```

翻转效果关键代码块：

```
func flipView(sender:UITapGestureRecognizer) {

    //翻转动画时间    
    var flipDuration = 0.8 
    
    //翻转过度动画效果
    UIView.transitionWithView(sender.view,
        duration: flipDuration,
        options: UIViewAnimationOptions.TransitionFlipFromLeft,
        animations: {() -> Void in
            
    	},
        completion: {(finished:Bool) -> Void in
        
        }
    )
    
    //在翻转到一半时，切换view
    dispatch_after(dispatch_time_t(flipDuration / 2) * 1000, dispatch_get_main_queue(), { () -> Void in
        (sender.view as PanelView).changeView()
    })
}
```

####效果图：
<img src="https://raw.githubusercontent.com/JasonZengJ/GridPanelDemo/master/demo.gif" alt="GridPanelDemo" title="GridPanelDemo" style="display:block;">