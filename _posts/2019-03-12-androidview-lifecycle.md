---
title: android View生命周期详解
tags: 流弊技能
keywords: 安卓view生命周期,阿里云建站,阿里云,aliyun,安卓开发,android,网站模板,企业官网定制,搭网站
description: 安卓view生命周期到底怎么样的，看这篇就够了
---

view的生命周期概念有点模糊,记录下方便以后查阅

第一步,构造 Constructors()

View在代码中被创建时调用第一种构造方法，View从layout中加载出来时会被调用第二种构造方法，其中XML中的属性也会被解析。

第二步, onFinishInflate()

该方法当View及其子View从XML文件中加载完成后触发调用。通常是在Activity中的onCreate方法调用后调用。不过这个用到比较少.

第三步. onVisibilityChanged()

此方法当被添加到viewgroup中的时候,也会被调用.不过调用非常频繁,所以也就没有用他了,不过记录一下.当前View或其父容器的可见性改变时被调用。

如果View状态不可见或者GONE，该方法会第一个被调用。

第四步. onAttachedToWindow() 

当view被添加到容器中在window中的时候会被调用。在Activity第一次执行完onResume方法后被调用。

第五步. onMeasure()

此方法确定View以及其子View尺寸大小时被调用。一般在此处重写测量方法

第六步, onSizeChanged()

当大小被改变的时候,比如原先是1000宽度,现在修改成500,则会调用此方法. 该方法在Measure方法之后且测量大小与之前不一样的时候被调用。

第七步 . onLayout()

当为viewgroup的时候.当前View需要为其子View分配尺寸和位置时会被调用。

第八步. onDraw(Canvas) (重点)

用于View渲染内容的细节。自定义view的核心方法也是在于此处 

第九步 . onWindowFocusChanged()

该方法也可能在绘制过程中被调用，具体是在包含当前View的Window获得或失去焦点时被调用。此时可以设置代码中定义的View的一些LayoutParameter。

如果View进入了销毁阶段，肯定是会被调用的。

第十步 onWindowVisibilityChanged()

该方法同上，具体是在包含当前View的Window可见性改变时被调用。

第十一步 . onDetachedFromWindow() 

当View离开附着的窗口时触发，比如在Activity调用onDestroy方法时View就会离开窗口。和一开始的AttachedToWindow相对，都只会被调用一次。.这个方法比较核心,如果是bitmap类.可以进行回收.

 

例子: 自动回收的imageviewbitmap

``` Java
public class RecycleImageView extends ImageView {
    private Bitmap mBitmap;

    public RecycleImageView(Context context) {
        super(context);
    }
     
    public RecycleImageView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
     
    public RecycleImageView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }
     
    @Override
    public void setImageBitmap(Bitmap bm) {
        this.mBitmap = bm;
        super.setImageBitmap(bm);
    }
     
    @Override
    protected void onDetachedFromWindow() {
        if (mBitmap!= null) {
            mBitmap.recycle();
            mBitmap = null;
        }
        super.onDetachedFromWindow();
    }
}
```

云服务器推荐，huhansome云服务器，限时折扣，稳定划算：
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
