---
title: 苹果加码隐私保护，但Facebook不会是最大受害者
tags: 科技解读
keywords: Apple，iOS 14,facebook，广告，隐私保护
description: iOS 14隐私加强，facebook这种广告公司会不会成为最大受害者？
---


本文来自[ “界面新闻” ](https://www.jiemian.com/article/4990761.html)，作者 姜菁玲

苹果将在9月17日正式推送iOS 14更新，从已经收到公测版推送的图片来看，苹果将在一直强调的隐私保护方面继续加码——

“新的‘隐私’功能提高了App访问位置、照片、麦克风和相机的方式对用户的透明度，并让用户可以更好地控制。”

在此次更新中，此前被热议的IDFA功能并没有出现。9月初，苹果曾宣布将针对IDFA的隐私新规推迟至2021年初发布，以帮助开发者预留更多时间应对IDFA更改。

IDFA（Identifier For Advertising）是苹果分配给每个设备的广告标识符，相当于“一台手机的身份证”，广告商可以通过IDFA对准用户和用户产生的一系列行为，实现用户追踪和广告效果监测。

苹果在6月份的WWDC上首次透露了将对IDFA做出的更改——IDFA分享功能将由默认开启的状态变为默认关闭，商家想要获得用户的IDFA，需要明确向用户弹窗示意并请求许可，否则商家所获得的的将只有一串无效0值。

默认关闭的IDFA会让广告商收到的数据量大大减少。由于IDFA在精准推送和广告效果评估上的意义，苹果此举被认为是对数字广告业务的一次重大打击。Facebook此前表示，苹果公司对此进行的变更，将导致Facebook数字网络广告业务下降50％以上。

但值得注意的是，以Facebook为代表的的大型数字广告平台可能并不会是IDFA取消的最大受害者，大公司自身在精准画像上的能力远远超过中小企业，而对IDFA强依赖的中小广告商则会遭到致命的打击。广告资源将进一步向头部企业集中，IDFA的默认取消为广告行业带来的很可能是一轮新的马太效应。

## IDFA如何影响Facebook

2013年，为了权衡用户隐私与广告商追踪用户的需求，苹果对唯一设备ID UDID进行封装，生成了广告标识符IDFA。

广告商可以利用IDFA对齐设备信息与用户行为，相当于“一台手机的身份证”，进而实现对用户的精准画像和广告效果评估。

“比如说，我们在网络上形成的所有数字路径，包括我们的购买记录、浏览器搜索纪录等等都可以通过IDFA被关联起来，广告商就可以对你进行画像，当你要买车的时候，给你推宝马、奔驰还是五菱之光就会更加精准。”一位从事游戏业务的开发者告诉界面新闻记者。

对于广告效果评估来说，广告主需要借助IDFA来判断——用户的转化（下载、点击、购买等行为）是否是由于该广告产生，从而对广告的内容和投放渠道进行效果评估和优化。

如果没有IDFA，商家只能通过自己内部的账号体系去识别用户及用户行为，不能实现跨应用的信息读取以及广告归因。例如，淘宝只能根据用户在淘宝应用内发生的行为进行数据分析和画像，但由于多应用之间的账号体系不互通，所以就不能读取其他例如微博、浏览器上的用户数据。

这也是Facebook、谷歌等数字广告平台产生焦虑的原因。Facebook的Audience Network业务主要帮助广告主进行广告定位和投放，它的基本模式是通过对用户建立画像并且定位追踪，然后精准投放广告。

长期以来，Audience Network业务使用IDFA来定位用户，当用户启动使用了 Audience Network 广告功能的应用并使用 Facebook 账号登录时，设备上的独一识别码（iOS 中的 IDFA 或安卓端的 unique ID）就会与该账号联系起来。那么当用户在同一设备上打开另一个使用该广告服务的应用时，Facebook 就会收到来自设备的信号然后根据你在社交网络上留下的相关信息推广告。

比如，你在Facebook上赞了某品牌的商品，那么当你在使用有关的第三方应用时就有可能看见该品牌向用户投放的广告内容，与Facebook的其它移动广告服务一样，这些内容可能以横幅、弹窗或者原生的形式出现。

从广告主的角度看，以游戏行业为例，对于小成本团队而言，做好游戏，然后在Facebook投钱买精准广告，等待效果评估结果，作为游戏测试是一个常规路径。如果IDFA对于Facebook的用户画像和效果评估产生重大影响，那么吸引他们的进入Audience Network的必要性就会减弱。

TapResearch最近的一项调查表明，如果App产品对IDFA的追踪请求在用户打开产品时弹出，那么用户选择允许的概率甚至将远低于30%，这意味着IDFA在用户的大规模追踪方面将受到很大影响。

在7月份的最新财报电话会议上，Facebook也警告投资者，IDFA可能会证明是“具有挑战性的逆风”。在近日的博客中，Facebook表示，整个Audience Network的应用发布商收入或将下降50％以上。

## Facebook不会是最受伤害的公司

但即使这样，在App专项治理专家何延哲看来，Facebook并不是IDFA默认关闭的最大受害者，最受影响的应该是中小广告商。

回归IDFA发挥作用的两大场景，一是广告精准触达，二是广告效果归因。

精准触达方面，一位游戏行业开发者告诉界面新闻记者，IDFA实际上只是用户画像的一个维度，主要作用在于对齐设备和用户行为，对于像Facebook、腾讯这类的大企业而言，没有IDFA，企业内部所覆盖场景所产生的数据也足够对用户进行精准画像，例如腾讯的朋友圈广告、淘宝的猜你喜欢。

“而对于IDFA强依赖的中小企业来说，他们并没有足够的数据可以支撑IDFA手段之外的用户画像。”何延哲表示。

在广告效果归因上，苹果目前提供的官方解决方案是一种新的隐私安全框架——SKAdNetwork，其核心就是给每个广告生成类似于标签的 Ad Signature 并一直伴随到用户点击、安装结束，让广告平台能够在不获取 IDFA、不收集用户信息的前提下，对用户的点击和安装应用行为进行追踪，但其能够获得的数据种类则被大幅度缩减，仅提供了APP安装时间、广告组、具体的广告，以及运营商等非常有限的信息。

在这之上，Facebook表示，“正在制定短期和长期战略，以通过这些变化为发布商提供支持”，以便通过没有IDFA的Audience Network实现个性化广告。该过程将涉及开发不依赖IDFA的跟踪手段，但现在就Facebook的工作细节分享细节还为时过早。”

对于大型企业而言，在苹果新框架之内或者之外，重新开发出可以实现目的的广告效果归因技术，只是时间问题。而对于中小广告商来说，技术、资金和人才的限制让他们只能在设备商的协议之下进行产品创新。

因此，总体而言，对于Facebook、谷歌等大型企业而言，苹果默认关闭IDFA，是对其既有运营模式的一次突然冲击，但是只要有足够的缓冲时间，业务仍然有恢复的可能性，但对于中小型的广告商而言，在平台新政面前的还手之力显得非常薄弱。

更进一步，随着未来一段时间内，大型企业本身在数据资源和技术上的优势，大型企业与中小企业在精准画像和广告归因上的能力差距可能越来越大。出于服务稳定和ROI优化的目的，广告主大概率也会向大型企业更加集中，一场新的变革所带来的的马太效应即将被加剧。

## IDFA关闭会改变什么，不会改变什么

“IDFA（关闭）不会改变你要收到广告这个事实，只是顶多你收到的广告不那么精准了。”一位主要面向海外市场的手游开发者向界面新闻记者分析。

何延哲表示，如果把精准推送当成一个问题的话，IDFA的默认关闭确实会在这个问题上起一定作用，甚至可以说是有点釜底抽薪、硬着陆的感觉。

但是，在收到广告的量一定的前提下，精准推送是否是一个问题还需要讨论。“如果你是一个希望买车的用户，相比于给你推房子的广告，我认为车子的广告至少还有点用。”上述游戏行业开发者认为。

“毋庸置疑，IDFA的默认关闭对于用户隐私保护确实是一个进步。虽然之前用户也可以自由选择主动关闭，但是这种（要求开发者主动申请的）方式，让用户对于自己的数据有了更高自由度的选择权。”何延哲认为。

个人隐私与数据价值坐落在天平两端，这背后分别代表了用户诉求和企业诉求，往何处偏离，都会产生问题，“解决方案是需要寻求平衡”。

隐私受到保护是每个人的基本权利，商业公司对用户过度精准的画像和跟踪，让人不适，造成信息茧方，且存在潜在危害。“从隐私的角度来看，对于App治理来说，用户隐私安全是底线。精准需要一个度，我们向企业鼓励间接画像的做法，即对群体进行画像，而不是对于个人画像。”何延哲表示。

“合规是一件必须要做的事，但合规到某种程度，也是一种门槛，会扼杀创新。”何延哲表示，多年来的发展，让互联网广告的模式已经比较成熟，个人享受互联网服务其实应该付出成本，对于一部分人来说，这些成本用金钱可以承担，但对于另一部分人而言，可能承担不起。

每次新事件的产生，都在试探商业价值和个人隐私二者的平衡点。IDFA默认关闭，是加在隐私一端的砝码，是个人隐私保护的一种进步，“但从整个商业生态来说，它会对数字经济产生怎样的影响，有待观察。”何延哲称。