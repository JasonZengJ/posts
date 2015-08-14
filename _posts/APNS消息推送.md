title: APNS消息推送
date: 2015-08-14 18:34:36
tags: Objective-c
---
网上好多教程，就不重复造轮子了，自己又从头到尾实践了一遍，遇到的坑也填好了，结局很美好～

* 首先得买一个开发者账号😄

* [iPhone 消息推送(Push Notification)的实现-------搭建 APNS 环境](http://1679554191.iteye.com/blog/1717531)

* iOS部分代码：

```
- (void)registerRemoteNotification
{
    float version=  [[UIDevice currentDevice].systemVersion floatValue];
    if (version >= 8.0) {
        [[UIApplication sharedApplication] registerUserNotificationSettings:[UIUserNotificationSettings settingsForTypes:(UIUserNotificationTypeSound | UIUserNotificationTypeAlert | UIUserNotificationTypeBadge) categories:nil]];
        [[UIApplication sharedApplication] registerForRemoteNotifications];
    } else {
        [[UIApplication sharedApplication] registerForRemoteNotificationTypes:UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound];

    }
    
}

#pragma mark - -- Notification

- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
    NSString *token = [[[[deviceToken description]
       stringByReplacingOccurrencesOfString: @"<" withString: @""]
      stringByReplacingOccurrencesOfString: @">" withString: @""]
     stringByReplacingOccurrencesOfString: @" " withString: @""] ;
    
    DLog(@"%@", token);
}
```

* bunddle identifier(可在plist文件里面修改) 一定要跟appId一致！

* [为什么收不到deviceToken？](http://stackoverflow.com/questions/24426090/get-device-token-in-ios-8)

* [Mac OS下搭建APNS推送服务器(Apache+PHP)](http://handy-wang.github.io/blog/2013/12/19/mac-osxia-da-jian-apnstui-song-fu-wu-qi/)

* [stream\_socket_client(): Unable to set private key file 报错的解决方案](http://stackoverflow.com/questions/1481443/apple-push-notification-service)

* R站的APNS教程详解[Apple Push Notification Services in iOS 6 Tutorial: Part 1/2](http://www.raywenderlich.com/32960/apple-push-notification-services-in-ios-6-tutorial-part-1)

* 消息长度限制:256字节 