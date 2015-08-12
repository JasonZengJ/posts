title: 实现滑动的时候可以定时改变UI
date: 2015-08-12 14:10:38
tags: Objective-c
---

###实现代码

```
......

@property(nonatomic,strong)UILabel *label;

......

	NSTimer *timer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(timerData) userInfo:nil repeats:YES];
	[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
	[timer fire];
 
 ......
 
 - (void)timerData
{
    self.time ++;
    self.label.text = [NSString stringWithFormat:@"%ld",_time];
    NSLog(@"%ld",self.time);
    
}   
    
```

###为什么scrollView滑动的时候，NSTimer 无法触发

* 当滑动界面时，系统为了更好地处理UI事件和滚动的平滑显示，主线程runloop会暂时停止处理一些其它事件，此时runloop的mode是UITrackingRunLoopMode，而NSTimer默认运行在NSDefaultRunLoopMode的runloop上，所以这时主线程中运行的NSTimer就会被暂停。

* 改变NSTimer运行的mode（mode可以看成事件类型），不使用缺省的NSDefaultRunLoopMode，而是改用NSRunLoopCommonModes（能被NSDefaultRunLoopMode和UITrackingRunLoopMode的runloop运行），这样主线程就会继续处理NSTimer事件了

###Runloop的NSDefaultRunLoopMode vs NSRunLoopCommonModes vs UITrackingRunLoopMode

参考资料：[NSDefaultRunLoopMode vs NSRunLoopCommonModes](http://stackoverflow.com/questions/7222449/nsdefaultrunloopmode-vs-nsrunloopcommonmodes)

* Runloop:是一种能够让系统唤醒睡眠中的线程来管理异步事件的机制。这样线程不需要不断的轮询有没有事件到来，而是当有新事件到来的时候，runloop会主动唤醒相应的线程来进行处理，提高了cpu的使用效率。
* 除了NSURLConnection生成的线程，因为该线程只处理来自外部网络调用的数据事件，而一般的线程可能做一些其它操作，比如数据处理，导入等，又要处理UI事件。
* run loop mode 主要是系统为了更好地优先处理一些事件，比如UI，保证滚动的平滑等，然后把其它一些事件延后处理。
* UITrackingRunLoopMode 优先于 NSDefaultRunLoopMode，NSRunLoopCommonModes包含UITrackingRunLoopMode、NSDefaultRunLoopMode，也就是说配置了NSRunLoopCommonModes的在两种模式的RunLoop 中都不会被延后处理
* 当用户进行点击之类对UI进行操作而产生事件时，主线程的run loop 转为UITrackingRunLoopMode

