---
title: 用flutter写一个抖音是什么体验？
tags: flutter,douyin,抖音
keywords: flutter, douyin, 抖音, android,跨平台,flutter demo,用flutter写抖音,flutter插件
description: 我竟然用flutter写了一个抖音，flutter最强demo参考
---

不知不觉flutter已经诞生好几年了，在这场轰轰烈烈的跨平台革命风暴中，`weex` `React-Native` `flutter` 这三者算是跨平台三巨头，weex给人的感觉就是简单轻便，React-Native被人们贴上了‘笨重’的标签，开始的flutter被认为问题多，开源库少；而如今的flutter已经变得非常强大，开源库也丰富了起来，在如今的形势下，怎么能不尝试一下。

其实最开始我比较好奇的是flutter插件是怎么实现和原生交互的，原来是[platform-channels](https://flutterchina.club/platform-channels/#example)这玩意，通过在`didFinishLaunchingWithOptions`注册plugin，注册一个MethodChannel，MethodChannel的name要唯一，以后调用的时候通过这个名字获取到channel来调用方法，同时定义了handler来处理解码后的消息；channel定义有三种：`BasicMessageChannel `  `MethodChannel `  `EventChannel ` handler和channel是一一对应的；dart调用原生，那么会把参数、方法名等封装成MethodCall对象，然后由`StandardMethodCodec `这玩意来封装成二进制，然后通过`BinaryMessages `发送平台消息；再往后就调到了C++: Window client --> RuntimeDelegate --> PlatformViewAndroid/PlatformViewIOS；最后FlutterResult（也就是callback）是怎么正确回调的? 全局一个字典pending_responses_保存了response，key是responseId value就是response，这样就能正确的回调了原理和[WebViewJavascriptBridge](https://github.com/marcuswestin/WebViewJavascriptBridge)是一样的；

基于对flutter插件原理分析，自己也尝试着写了一个，实现了无需权限获取设备标识：[flutter_unique_device_id](https://pub.dev/packages/flutter_unique_device_id) ；再不说点抖音好像就跑题了。。。

也是出于对flutter的实践，打算用flutter写一个抖音，但是如果你一点flutter都没学过，那么这不是一件容易的事情，你会发现我该怎么开始？tabbarcontroller该怎么实现？tabbar该怎么自定义？tableview加header在flutter里怎么实现？我要实现动画该怎么写？网络请求用什么库？json解析怎么解析？等等一系列的问题，在开始你会发现你就是在使用flutter的三方库组装APP而已，各种各样的三方库太多了，很多的功能可以说都有开源库写好了，和原生开发一样的是一个百家齐放的状态，等到对flutter有一定熟练度后，就能大肆的自定义，随便操刀了！

抖音的tabbar其实很像`BottomAppBar`的效果，但是有不完全一样，其实flutter里还没有可以直接使用的和抖音一样的，没办法那就只能自己写，还有抖音个人中心的那个滑动放大效果，在原生里就是一个tableview header滑动放大，但是没想到在flutter里实现这么复杂，用了`NestedScrollView `还是不行，还得自己监听滑动，去实现发大和自动复位的动画效果。flutter在某些极少数时候实现是没有原生方便的。现在douyin-flutter项目已经实现了很多功能，但是还在完善中。源码地址[douyin-flutter](https://github.com/sharequeue/douyin-flutter)； 文章一写起来就收不住，暂时就写这些吧~

