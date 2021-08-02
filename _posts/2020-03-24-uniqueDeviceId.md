---
title: android ios 如何获取设备唯一id
tags: android,ios,unique_id, flutter, uniapp
keywords: unique_id,iOS, uniapp插件, android,设备id,设备唯一标识,unique_device_id,flutter插件
description: 设备唯一id如何获取，安卓获取设备唯一标识；iOS设备唯一标识
---

## 前言
设备id，也就是和设备可以一一对应，并且每次获取都是相同的这么一个字符串，在早些年，系统api是开放的，可以直接获取到唯一的设备标识，这些设备标识往往是和设备硬件绑定的，这样一来，设备id就是永远固定的，除非更换了硬件，但是随着现在对于个人隐私保护的重视，系统api已经不能获取到唯一标识，那么我们就只能另外想办法了。获取设备id的目的无非也就以下几点：
>
- 统计，统计DAU等
- 业务需要，这类APP一般都是根据个人喜好来推送内容，恰恰是有违隐私保护政策
- 风险控制，防止薅羊毛等刷注册行为 

随着工信部或应用市场合规整改通知的发布，对于个人隐私保护已经非常严格，相信不少的人都收到以下警告：
>
- 违规收集个人信息
- 强制、频繁、过度索取权限

## 分析
对于iOS，虽然官方api也去掉了，但是还是有identifierForVendor可以将就使用，实在不行那就自己生产一个uuid保存在keychain里，这样就自己生成uuid来实现设备标识；这里要说的是安卓可以用相同的思路来实现，但是由于安卓没有keychain类似的实现，所以只能自己生成后保存到SDK卡，这样才能保证APP卸载之后数据不丢失，但是这样又涉及到获取SDK权限，用户：你TM又想瞎存啥东西？所以我们应该尽量避免需要权限的做法。

安卓唯一id的文章网上已经泛滥了，但是大致也就那么几种方式，在经过筛选之后，我个人觉得如果我们要在不需要权限的情况下获取，那么我们只能尽量保证准确性，很难做到百分百，另外有种思路就是说把能获取到的能标识的数据都获取，然后自己服务器保存起来，每次获取之后对比，有几个数据相同则表示是同一设备，这种方式需要服务器来保存，还需要一个策略来判断，除了繁琐点，其实还是不错的。

借鉴于[is-there-a-unique-android-device-id]（https://stackoverflow.com/questions/2785485/is-there-a-unique-android-device-id），乐观一点话会发现其实通过 `Build.BOARD` `Build.BRAND` `Build.CPU_ABI` 等等参数来生成的序列还是很大程度能标识设备，如果`Build. SERIAL `能获取成功，那么将会很大可能能区分，如果`Build. SERIAL`不能获取成功，那么我们可以暂时使用`Android ID`来代替`Build. SERIAL`，虽然`Android ID`会在用户刷机或者恢复出厂设置时变更，但是毕竟这样做的人不会太多。

基于以上思路，于是安卓代码就长这样：

```Java
	private String getUniquePsuedoID() {
        // If all else fails, if the user does have lower than API 9 (lower
        // than Gingerbread), has reset their device or 'Secure.ANDROID_ID'
        // returns 'null', then simply the ID returned will be solely based
        // off their Android device information. This is where the collisions
        // can happen.
        // Thanks http://www.pocketmagic.net/?p=1662!
        // Try not to use DISPLAY, HOST or ID - these items could change.
        // If there are collisions, there will be overlapping data
        String m_szDevIDShort = "35" + (Build.BOARD.length() % 10) + (Build.BRAND.length() % 10) + (Build.CPU_ABI.length() % 10) + (Build.DEVICE.length() % 10) + (Build.MANUFACTURER.length() % 10) + (Build.MODEL.length() % 10) + (Build.PRODUCT.length() % 10);

        // Thanks to @Roman SL!
        // https://stackoverflow.com/a/4789483/950427
        // Only devices with API >= 9 have android.os.Build.SERIAL
        // http://developer.android.com/reference/android/os/Build.html#SERIAL
        // If a user upgrades software or roots their device, there will be a duplicate entry
        String serial = null;
        try {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                serial = android.os.Build.getSerial();
            } else {
                serial = Build.SERIAL;
            }
            // Go ahead and return the serial for api => 9
            return new UUID(m_szDevIDShort.hashCode(), serial.hashCode()).toString();
        } catch (Exception exception) {
            // String needs to be initialized
            final String androidId = "" + android.provider.Settings.Secure.getString(_flutterPluginBinding.getApplicationContext().getContentResolver(), android.provider.Settings.Secure.ANDROID_ID);
            serial = androidId;
        }

        // if Build.SERIAL get successfuly, the 'id' is unique very likely, if not, we use androidId to guarante 'id' to be unique as possible
        // without any permissions


        // Thanks @Joe!
        // https://stackoverflow.com/a/2853253/950427
        // Finally, combine the values we have found by using the UUID class to create a unique identifier
        return new UUID(m_szDevIDShort.hashCode(), serial.hashCode()).toString();
    }
```
## 源码
考虑到现在跨平台的流行，笔者将实现封装成了uniapp和flutter插件，可以在uniapp开发跨平台APP和flutter开发中使用

- [uniapp插件](https://ext.dcloud.net.cn/plugin?id=2916) https://ext.dcloud.net.cn/plugin?id=2916
- [flutter插件](https://pub.dev/packages/flutter_unique_device_id) https://pub.dev/packages/flutter_unique_device_id
- [项目源码](https://github.com/sharequeue/unique_device_id) https://github.com/sharequeue/unique_device_id
