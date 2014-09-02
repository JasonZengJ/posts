title: ios 8 新特性和改进
date: 2014-09-01 12:22:55
tags: ios
---

英文原文：[NSHipster i​OS 8](http://nshipster.com/ios8/)


##NSProcessInfo -isOperatingSystemAtLeastVersion

忘记[[UIDevice currentDevice] systemVersion]和NSFoundationVersionNumber吧, 现在可以用NSProcessInfo -isOperatingSystemAtLeastVersion来确定系统版本。

```
import Foundation

let yosemite = NSOperatingSystemVersion(majorVersion: 10, minorVersion: 10, patchVersion: 0)
NSProcessInfo().isOperatingSystemAtLeastVersion(yosemite) // false
```

值得注意的是，在做兼容性测试的时候还是应该使用SomeClass.class或respondsToSelector:。


##新的NSFormatter子类

Foundation中严重缺失的一项功能就是不能处理重量和长度单位转换。在iOS 8和OS X Yosemite中，引进了三个新类NSEnergyFormatter，NSMassFormatter和NSLengthFormatter来弥补这一缺失。

这使得NSFormatter子类的数量翻了一倍, 之前只有NSNumberFormatter，NSDateFormatter和NSByteCountFormatter。

注：虽然这些都是Foundation的子类，但是它们主要都是在HealthKit当中使用


###NSEnergyFormatter

NSEnergyFormatter使用焦作为能量的原始单位，当处理健康信息时，则使用它.

```
let energyFormatter = NSEnergyFormatter()
energyFormatter.forFoodEnergyUse = true

let joules = 10_000.0
println(energyFormatter.stringFromJoules(joules)) // "2.39 Cal"
```

###NSMassFormatter

虽然质量是物质存在的基本单位, 但在HealthKit中，它主要指的是身体重量.

```
let massFormatter = NSMassFormatter()
let kilograms = 60.0
println(massFormatter.stringFromKilograms(kilograms)) // "132 lb"
```

###NSLengthFormatter

我们可以把它想象为MKDistanceFormatter的加强版。

```
let lengthFormatter = NSLengthFormatter()
let meters = 5_000.0
println(lengthFormatter.stringFromMeters(meters)) // "3.107 mi"
```

##CMPedometer

沿着iOS 8的健康路线, CMStepCounter被重新设计了。CMPedometer作为它的改良版本不仅可以即时获取离散的点数据，并且可以同时跟踪脚步和距离，甚至计算总共爬了多少级楼梯。

M7芯片真是功能强大.

```
import CoreMotion

let lengthFormatter = NSLengthFormatter()
let pedometer = CMPedometer()
pedometer.startPedometerUpdatesFromDate(NSDate(), withHandler: { data, error in
    if !error {
        println("Steps Taken: \(data.numberOfSteps)")

        let distance = data.distance.doubleValue
        println("Distance: \(lengthFormatter.stringFromMeters(distance))")

        let time = data.endDate.timeIntervalSinceDate(data.startDate)
        let speed = distance / time
        println("Speed: \(lengthFormatter.stringFromMeters(speed)) / s")
    }
})
```

##CMAltimeter

在支持的设备上，CMAltimeter可以让CMPedometer的floorsAscended，floorsDescended数据更加精准：

```
import CoreMotion

let altimeter = CMAltimeter()
if CMAltimeter.isRelativeAltitudeAvailable() {
    altimeter.startRelativeAltitudeUpdatesToQueue(NSOperationQueue.mainQueue(), withHandler: { data, error in
        if !error {
            println("Relative Altitude: \(data.relativeAltitude)")
        }
    })
}
```

##CLFloor


CLFloor的引入展示了苹果进军室内导航的宏伟计划，楼层信息将扮演着重要的角色。

```
import CoreLocation

class LocationManagerDelegate: NSObject, CLLocationManagerDelegate {
    func locationManager(manager: CLLocationManager!, didUpdateLocations locations: AnyObject[]!) {
        let location: CLLocation? = locations[0] as? CLLocation
        if let floor: CLFloor? = location?.floor {
            println("Current Floor: \(floor?.level)")
        }
    }
}

let manager = CLLocationManager()
manager.delegate = LocationManagerDelegate()
manager.startUpdatingLocation()
```

##HKStatistics

作为一个框架，HealthKit包含着大量的子类和常量。要想全部理解，HKStatistics是一个很好的开始。

HealthKit管理着所有的生理信息，例如：心率，卡路里摄入量，血氧等等，并且通过统一的API聚合在一起。

下面这个例子演示了如何从一天的连续数据中，挖掘和获取单独的数据：

```
import HealthKit

let collection: HKStatisticsCollection? = ...
let statistics: HKStatistics? = collection!.statisticsForDate(NSDate())
for item: AnyObject in statistics!.sources {
    if let source = item as? HKSource {
        if let quantity: HKQuantity = statistics!.sumQuantityForSource(source) {
            if quantity.isCompatibleWithUnit(HKUnit.gramUnitWithMetricPrefix(.Kilo)) {
                let massFormatter = NSMassFormatter()
                let kilograms = quantity.doubleValueForUnit(HKUnit.gramUnitWithMetricPrefix(.Kilo))
                println(massFormatter.stringFromKilograms(kilograms))
            }

            if quantity.isCompatibleWithUnit(HKUnit.meterUnit()) {
                let lengthFormatter = NSLengthFormatter()
                let meters = quantity.doubleValueForUnit(HKUnit.meterUnit())
                println(lengthFormatter.stringFromMeters(meters))
            }

            if quantity.isCompatibleWithUnit(HKUnit.jouleUnit()) {
                let energyFormatter = NSEnergyFormatter()
                let joules = quantity.doubleValueForUnit(HKUnit.jouleUnit())
                println(energyFormatter.stringFromJoules(joules))
            }
        }
    }
}

```

##NSStream +getStreamsToHostWithName

在许多方面，WWDC 2014也是苹果查漏补遗的一年，比如给NSStream添加了新的initializer（再也不用调用CFStreamCreatePairWithSocketToHost了），这就是：+[NSStream getStreamsToHostWithName:port:inputStream:outputStream:]

```
var inputStream: NSInputStream?
var outputStream: NSOutputStream?

NSStream.getStreamsToHostWithName(hostname: "nshipster.com",
                                      port: 5432,
                               inputStream: &inputStream,
                              outputStream: &outputStream)
                                                            
```
`
##NSString -localizedCaseInsensitiveContainsString

这又是一个NSString小而实用的修缮:

```
let string: NSString = "Café"
let substring: NSString = "É"

string.localizedCaseInsensitiveContainsString(substring) // true
```

##CTRubyAnnotationRef

好吧，此Ruby非彼Ruby. . 这是用来给亚洲文字添加注音符号的.

ps:略逗...: P

```
@import CoreText;

NSString *kanji = @"猫";
NSString *hiragana = @"ねこ";

CFStringRef furigana[kCTRubyPositionCount] =
    {(__bridge CFStringRef)hiragana, NULL, NULL, NULL};

CTRubyAnnotationRef ruby =
    CTRubyAnnotationCreate(kCTRubyAlignmentAuto, kCTRubyOverhangAuto, 0.5, furigana);
```

无可否认的是，文档中并没有很清晰的描述具体如何将它整合进入你剩下的CoreText中，但是结果如下：

ねこ

猫

##新的日历识别符

iOS 8和OS X中这些新的日历识别符使得Fundation跟上了CLDR的步伐：

* NSCalendarIdentifierCoptic: 亚历山大日历, 科普特正教使用.
* NSCalendarIdentifierEthiopicAmeteMihret: 埃塞俄比亚日历, Amete Mihret
* NSCalendarIdentifierEthiopicAmeteAlem: 埃塞俄比日历, Amete Alem
* NSCalendarIdentifierIslamicTabular: 一个简单的伊斯兰星历.
* NSCalendarIdentifierIslamicUmmAlQura: 沙特阿拉伯伊斯兰日历.

##NSURLCredentialStorage

自从去年NSURLSession的引入之后，Foundation的URL载入系统并没有太大的改变。然而，NSURLCredentialStorage提供了一些顶层的操作单元，这些操作单元针对某个任务设置和获取credentials都是以异步，非阻塞的方式执行的。

```
import Foundation

let session = NSURLSession()
let task = session.dataTaskWithURL(NSURL(string: "http://nshipster.com"), completionHandler: { data, response, error in
    // ...
})

let protectionSpace = NSURLProtectionSpace()
NSURLCredentialStorage.getCredentialsForProtectionSpace(protectionSpace: protectionSpace, task: task, completionHandler: { credentials in
    // ...
})
```

##kUTTypeToDoItem

看了一下最近新版与老版API的差异，有一个值得注意的是大量新的UTIs常量.其中一个比较闪亮，吸引我眼球的是kUTTypeToDoItem这货

```
import MobileCoreServices

kUTTypeToDoItem // "public.to-do-item"
```

作为一个公共类型，iOS 和 OS X 现在提供了一种通用的方式来在应用之间共享tasks。

UTI (Uniform Type Identifier - 同一类型标识符)

UTI的含义和这个是干什么用的，可以参考官方文档:[Introduction to Uniform Type Identifiers Reference](https://developer.apple.com/library/ios/documentation/Miscellaneous/Reference/UTIRef/Introduction/Introduction.html)

##kCGImageMetadataShouldExcludeGPS

大部分用户完全不知道现在使用手机拍摄的照片里面是带有GPS愿数据的。（难怪用QQ发说说能显示照片拍摄地点！）
由于这个小细节，无数人的隐私被暴露了（FUCK！）

新的图像I/O框架提供了一个方便的新选项，即CGImageDestination: kCGImageMetadataShouldExcludeGPS，该选项
在图像I/O的时候可以选择是否排除地理位置数据。

```
@import UIKit;
@import ImageIO;
@import MobileCoreServices;

UIImage *image = ...;
NSURL *fileURL = [NSURL fileURLWithPath:@"/path/to/output.jpg"];
NSString *UTI = kUTTypeJPEG;
NSDictionary *options = @{
                          (__bridge id)kCGImageDestinationLossyCompressionQuality: @(0.75),
                          (__bridge id)kCGImageMetadataShouldExcludeGPS: @(YES),
                          };

CGImageDestinationRef imageDestinationRef =
CGImageDestinationCreateWithURL((__bridge CFURLRef)fileURL,
                                (__bridge CFStringRef)UTI,
                                1,
                                NULL);

CGImageDestinationAddImage(imageDestinationRef, [image CGImage], (__bridge CFDictionaryRef)options);
CGImageDestinationFinalize(imageDestinationRef);
CFRelease(imageDestinationRef);
```

##WTF_PLATFORM_IOS

\#define WTF_PLATFORM_IOS 从JavaScriptCore中移除了

ps:WTF_PLATFORM_IOS的含义会不会是WHAT_THE_FUCK_PLATFORM_IOS  :P 哈哈

##WKWebView

新增了这个WKWebView，据说要替换UIWebView。

WKWebView 能在应用中提供了Safari级别的网页处理性能，并且通过偏好配置大大的增强了UIWebView

[可以戳这里看看](http://www.hotobear.com/?p=741)

```
import WebKit

let preferences = WKPreferences()
preferences.javaScriptCanOpenWindowsAutomatically = false

let configuration = WKWebViewConfiguration()
configuration.preferences = preferences

let webView = WKWebView(frame: self.view.bounds, configuration: configuration)
let request = NSURLRequest(URL: NSURL(string: "http://nshipster.com"))
webView.loadRequest(request)
```

##NSQualityOfService

ps:看了下，还没开始用，等用用再放上使用代码 ：P

线程这个概念已经在苹果的框架中被系统性的忽略。这对于开发者而言是件好事。

因此在最新的API中对NSOperation做了一些改变，一个新的属性qualityOfService替换了threadPriority。
这些新的语义是的app可以推迟一些不是很重要的线程任务的执行，从而保证持续良好的用户体验。

NSQualityOfService 枚举类型定义了以下常量值：

* UserInteractive：和图形处理相关的任务，比如滚动和动画
* UserInitiated：用户请求的任务，但是不需要精确到毫秒级。例如，如果用户请求打开电子邮件App来查看邮件。
* Utility：周期性的用户请求任务。比如，电子邮件App可能被设置成每五分钟自动检查新邮件。但是在系统资源极度匮乏的时候，将这个周期性的任务推迟几分钟也没有大碍。
* Background：后台任务，用户可能并不会察觉对这些任务。比如，电子邮件App对邮件进行引索以方便搜索。

NSQualityOfService将在iOS 8和OS X Yosemite中广泛的应用，所以留意所有能利用它们的机会。

##LocalAuthentication

最后，最令人期待的iOS 8新功能之一：LocalAuthentication。自从iPhone 5S加入TouchID，开发者就对它的应用前景垂涎三尺。

想象一下，只要有CloudKit和LocalAuthentication，创建新账号的烦恼讲不复存在。只需要扫描一下你的手就搞定了！

LocalAuthentication以LAContext的方式工作，LAContext评估指定的策略，然后返回是否验证成功。整个过程中，用户的生物信息都被安全的储存在硬件当中。

```
LAContext *context = [[LAContext alloc] init];
NSError *error = nil;

if ([context canEvaluatePolicy:LAPolicyDeviceOwnerAuthenticationWithBiometrics
                         error:&error])
{
    [context evaluatePolicy:LAPolicyDeviceOwnerAuthenticationWithBiometrics
            localizedReason:NSLocalizedString(@"...", nil)
                      reply:^(BOOL success, NSError *error) {
        if (success) {
            // ...
        } else {
            NSLog(@"%@", error);
        }
    }];
} else {
    NSLog(@"%@", error);
}
```


在这里可以深入挖掘ios8的新特性和改进 [dive into the iOS 7.1 to 8.0 API diffs](https://developer.apple.com/library/prerelease/ios/releasenotes/General/iOS80APIDiffs/index.html#//apple_ref/doc/uid/TP40014455)

