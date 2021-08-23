---
title: React如何使用Cookie，你肯定没试过
tags: 流弊技能
keywords: React, Cookie,云建站,阿里云建站,阿里云,aliyun,网站模板,企业官网定制,搭网站
description: React使用cookie的正取姿势，react保存和使用cookie应该注意什么？
---

Cookie作为数据持久化非常重要的一部分，我们在实际使用中是非常重要的，react设置cookie,持久化保存数据,一般来说,我们使用redux来保存当前
数据即可,但是有一种情况,需要保存信息,那就是登陆的时候,我们需要存储,token,来保持永久化登陆

将其抽取到了common中,通过document.cookie,来获取页面的cookie

```
export default {

    // 获取cookie
    getCookie(key) {
        const name = key + "=";
        const ca = document.cookie.split(';');
        for (let i = 0; i < ca.length; i++) {
            const c = ca[i].trim();
            if (c.indexOf(name) === 0) {
                return c.substring(name.length, c.length);
            }
        }
        return "";
    },
     
    // 设置cookie,默认是30天
    setCookie(key, value) {
        const d = new Date();
        d.setTime(d.getTime() + (30 * 24 * 60 * 60 * 1000));
        const expires = "expires=" + d.toGMTString();
        document.cookie = key + "=" + value + "; " + expires;
    }

}
```
huhansome云服务和云数据库推荐：
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