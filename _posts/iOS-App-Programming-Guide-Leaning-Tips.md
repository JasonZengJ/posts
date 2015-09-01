title: iOS App Programming Guide Leaning Tips
date: 2015-09-01 15:18:19
tags: [Objective-c]
---
[App Programming Guide for iOS](https://developer.apple.com/library/prerelease/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40007072-CH1-SW1)

##Tips

* UIApplication是第一个接收到用户交互事件（点击、长按、拖动等）的对象，然后来决定该做什么，比如把事件发给哪个对象处理。
* 一个点击触摸事件，一般会分发给主window 对象，然后该window对象根据点击的区域，来把事件分发给对应的被触摸点击的view，事件由底往上冒泡传递。

  touch event => OS => UIKit Port => Event Queue => Main Run Loop => UIApplication => Core Objects(UIWindow -> UIVew -> UIView ....)
  
*  Remote control事件比如视频、歌曲的播放操作等，由耳机、其它配件产生，最终会发给对应的Responder对象（UIResponder、NSResponder）
* 如果一个View 是通过ViewController管理的，当View不能处理事件时，事件会传递给ViewController，因为ViewController 也是继承自Responder类，所以也在Responder chain 事件响应链中。
* 加速计、陀螺仪、磁力计、定位gps等事件，发送给自定义指定的对象 
* Protocol 与 Categories 中都可以在@interface .. @end 区域内定义 @property，但是不会自动生成相应的getter和setter，需要使用@ synthesize property;生成或自己在 @implementation ... @end 中定义。
* Extension 相当于声明了一个匿名的Cagegories的头文件，Extension的实现需要在原.m文件中。


##App 的执行状态

![](https://developer.apple.com/library/prerelease/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Art/high_level_flow_2x.png)

* Not Running：App没有启动或 应用被系统终止掉了。
* Inactive：App处于打开状态，但是没有接收到事件比如用户点击之类的。
* Active：App处于打开状态，并在处理事件，比如用户滑动、点击、键盘输入等。
* Background：App切换到后台（按Home键，此时App没有被系统终止掉）执行代码,
* Suspended：此时App被切换到后台，但是没有运行任何代码。当处于Background状态的App没有继续在后台执行的时候，会被系统自动转换到Suspended状态.当系统内存不足的时候，会把处于Suspended状态的App终止掉并清除掉App的内存，来给当前正在运行的App提供更多的内存空间。

##App 状态切换时调用的方法

[参考了这里](http://www.cnblogs.com/wayne23/p/3868441.html)

* case 1：应用启动

```
function:[applicationWillFinishLaunching],state:[Inactive]
function:[applicationDidFinishLaunching],state:[Inactive]
function:[applicationDidBecomeActive], state:[Active]
```

* case 2：当按下home键

```
function:[applicationWillResignActive], state:[Active]
function:[applicationDidEnterBackground], state:[Background]
```

* case 3：当双击home键，然后选择返回当前app：

```
function:[applicationWillEnterForeground], state:[Background]
function:[applicationDidBecomeActive], state:[Active]
```

* case 4：当双击home键，然后选择其它app：

```
function:[applicationWillResignActive], state:[Active]
function:[applicationDidEnterBackground], state:[Background]
```

* case 5：当有来电，拒绝接听：

```
function:[applicationWillResignActive], state:[Active]
function:[applicationDidBecomeActive], state:[Active]
```

* case 6：当有来电，接听后并挂断：

```
function:[applicationWillResignActive], state:[Active]
function:[applicationDidEnterBackground], state:[Background]
function:[applicationWillEnterForeground], state:[Background]
function:[applicationDidBecomeActive], state:[Active]
```

* case 7：当有短信或顶部的推送消息，点击查看，或按下锁屏键：

```
function:[applicationWillResignActive], state:[Active]
function:[applicationDidEnterBackground], state:[Background]
```



##App后台运行

* 通过UIApplication的backgroundTimeRemaining 可以获取到App还可以在后台运行多长时间。
* beginBackgroundTaskWithName中最好只做一些后续的资源保存、状态保存等处理，而不是执行需要比较长时间运行的代码，因为超过了过期时间，系统会把应用强行终止掉。
* 后台下载文件、数据请求等需要使用NSURLSession，然后使用 backgroundSessionConfigurationWithIdentifier:来生成后台执行所需的配置NSURLSessionConfiguration。
* 如果下载任务没有完成而系统终止了App的运行，那么系统会自动在后台管理下载任务，（因为系统是用另外的进程来处理后台文件传输的）当传输结束的时候，会launch 你的App，并调用AppDelegate的application:handleEventsForBackgroundURLSession:completionHandler:方法。而如果App是用户自己终止掉的，那么所有要执行的任务都会被取消
* 在项目的Capabilities -> Background Modes中进行配置需要后台执行的类型，同时在info.plist中增加Required background modes选项
* 可以使用UILocalNotification 来发送本地通知.

##App Launch Cycle

![](https://developer.apple.com/library/prerelease/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Art/app_launch_fg_2x.png)

* 若App在5秒内没有启动完成，那么App就会被系统终止掉
* 当用户拉下通知中心的菜单的时候， App也会进入Inactive状态
* applicationDidEnterBackground: 函数的执行最大时间是5秒，若超过五秒后该函数没有执行完，那么App就会被系统Kill掉

