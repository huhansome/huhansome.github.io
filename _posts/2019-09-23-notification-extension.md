---
title: iOS Notification Service Extension
tags: 流弊技能
keywords: 云建站,阿里云建站, iOS, Notification,阿里云,aliyun,网站模板,企业官网定制,搭网站
description: iOS推送扩展怎么使用？可以自定义显示，自定义可控性开放
---


iOS10之后的通知具有通知扩展功能，可以在系统受到通知、展示通知时做一些事情。

#### UNNotificationServiceExtension：通知服务扩展，是在收到通知后，展示通知前，做一些事情的。

###### 1. 创建一个UNNotificationServiceExtension

![img](https:////upload-images.jianshu.io/upload_images/12673226-42ddf140254c9485.png?imageMogr2/auto-orient/strip|imageView2/2/w/730/format/webp)

如图创建完可以看到工程中多出一个文件

![img](https:////upload-images.jianshu.io/upload_images/12673226-7ca143bc9b4588a2.png?imageMogr2/auto-orient/strip|imageView2/2/w/274/format/webp)

在NotificationService.m文件中，有两个自动生成的方法：



```cpp
// 系统接到通知后，有最多30秒在这里重写通知内容（在此方法可进行一些网络请求，如上报是否收到通知等操作）
- (void)didReceiveNotificationRequest:(UNNotificationRequest *)request withContentHandler:(void (^)(UNNotificationContent *contentToDeliver))contentHandler;
// 处理过程超时，则收到的通知直接展示出来
- (void)serviceExtensionTimeWillExpire;
```

示例代码：



```objectivec
- (void)didReceiveNotificationRequest:(UNNotificationRequest *)request withContentHandler:(void (^)(UNNotificationContent * _Nonnull))contentHandler {
    self.contentHandler = contentHandler;
    self.bestAttemptContent = [request.content mutableCopy];
    NSLog(@"收到推送消息，可进行处理");
//      这里重写一些东西
    self.bestAttemptContent.title = @"这里是标题";    
    self.bestAttemptContent.subtitle = @"这里是子标题";
    self.bestAttemptContent.body = @"这里是body";
    self.contentHandler(self.bestAttemptContent);
}
```

效果图：


![img](https:////upload-images.jianshu.io/upload_images/12673226-5c7df59516cbbd4a.png?imageMogr2/auto-orient/strip|imageView2/2/w/430/format/webp)


- 如果，你按照教程给自己的app设置好 Notification Service Extension，但是你的扩展程序没有执行。有可能是远程通知的格式问题。
   远程推送消息的格式，如下。



```json
{
  "aps":{
        "category":"SECRET",
        "mutable-content": 1,
        "alert":{
           "title":"Secret Message!",
           "body":"(Encrypted)"
         },
    }
}
```

其中 mutable-content字段不能没有，没有的话，就不会走扩展程序的代码；
 其中alert字段的值不能没有，没有的话，就不会走扩展程序的代码。

- 最重要最容易忽略的一点，必须要设置target的 Deployment Target

  ![img](https:////upload-images.jianshu.io/upload_images/12673226-40b52133b26b865e.png?imageMogr2/auto-orient/strip|imageView2/2/w/658/format/webp)

#### UNNotificationContentExtension：通知内容扩展，是在展示通知时展示一个自定义的用户界面

###### 1. 创建一个UNNotificationContentExtension

![img](https:////upload-images.jianshu.io/upload_images/12673226-078e4ced7a08ef4d.png?imageMogr2/auto-orient/strip|imageView2/2/w/730/format/webp)



如图创建完可以看到工程中多出一个文件



![img](https:////upload-images.jianshu.io/upload_images/12673226-f2703402a9b58e5d.png?imageMogr2/auto-orient/strip|imageView2/2/w/270/format/webp)



设置Info.plist



![img](https:////upload-images.jianshu.io/upload_images/12673226-57f7550d2897f1f8.png?imageMogr2/auto-orient/strip|imageView2/2/w/653/format/webp)


使用的时候，我们参照如下代码：



```objectivec
- (void)didReceiveNotificationRequest:(UNNotificationRequest *)request withContentHandler:(void (^)(UNNotificationContent * _Nonnull))contentHandler {
    self.contentHandler = contentHandler;
    self.bestAttemptContent = [request.content mutableCopy];
    NSLog(@"收到推送消息，可进行处理");
//      这里重写一些东西
        self.bestAttemptContent.title = @"这里是标题";
        self.bestAttemptContent.subtitle = @"这里是子标题";
        self.bestAttemptContent.body = @"这里是body";
    // ！！！！！ 这里是重点！！！！！！！！！！！！
    // 我在这里写死了myNotificationCategory，其实在收到系统推送时，每一个推送内容最好带上一个catagory，跟服务器约定好了，这样方便我们根据categoryIdentifier来自定义不同类型的视图，以及action
    //myNotificationCategory这个值要跟info.plist里面的值一样
    self.bestAttemptContent.categoryIdentifier = @"myNotificationCategory";
    self.contentHandler(self.bestAttemptContent);
}
```

这里为了方便，我在ContentExtension文件夹下直接更改MainInterface.storyboard文件实现自定义界面



![img](https:////upload-images.jianshu.io/upload_images/12673226-a243c481205d1bde.png?imageMogr2/auto-orient/strip|imageView2/2/w/840/format/webp)



运行效果图



![img](https:////upload-images.jianshu.io/upload_images/12673226-c9aaea78437a5604.png?imageMogr2/auto-orient/strip|imageView2/2/w/430/format/webp)


附：以上两个项目运行要记得选择项目，否则是无效的。
 如图：



![img](https:////upload-images.jianshu.io/upload_images/12673226-b5bfcf2fcb8859c9.png?imageMogr2/auto-orient/strip|imageView2/2/w/380/format/webp)



![img](https:////upload-images.jianshu.io/upload_images/12673226-38c9904953657bee.png?imageMogr2/auto-orient/strip|imageView2/2/w/433/format/webp)

huhansome云服务器和云数据库推荐，阿里云服务器限时折扣：
---
[阿里云服务器，限时秒杀价低至102/年，券后96/年](https://www.aliyun.com/minisite/goods?source=5176.11533457&userCode=j6bryttg)

[阿里云企业级服务器1折限时特惠](https://promotion.aliyun.com/ntms/act/enterprise-discount.html?source=5176.11533457&userCode=j6bryttg)

[阿里云建站 速成美站 - 千套模板，价格低至500元](https://ac.aliyun.com/application/webdesign/sumei?source=5176.11533457&userCode=j6bryttg)

[阿里云SaaS商标注册服务低至680元](https://tm.aliyun.com/?userCode=j6bryttg&source=5176.11533457&userCode=j6bryttg)

[CDN爆款产品低至5.5折](https://yqh.aliyun.com/live/cdncarnival?userCode=j6bryttg&source=5176.11533457&userCode=j6bryttg)
<hr>
[腾讯云服务器限时秒杀](https://cloud.tencent.com/act/cps/redirect?redirect=1067&cps_key=49b140a6ab2ad2bbd5cd3a951a0661fc&from=console)

[腾讯云境外服务器低至2折](https://cloud.tencent.com/act/cps/redirect?redirect=1001&cps_key=49b140a6ab2ad2bbd5cd3a951a0661fc&from=console)

[腾讯云数据库](https://cloud.tencent.com/act/cps/redirect?redirect=1003&cps_key=49b140a6ab2ad2bbd5cd3a951a0661fc&from=console)