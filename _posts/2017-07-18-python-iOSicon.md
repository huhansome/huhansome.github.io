---
title: python图片处理：切图
tags: 人生苦短，就用python
description: Python图片处理代码，Python实现自动化切图
---

继爬[取科技新闻](https://huhansome.github.io/_posts/2017-06-26-python-get-news/)之后，可以说大大的感觉到python的好处，不管是做些偷懒的事还是其他（至于是什么事就不吐露了），某次巧合之中，iOS程序需要更换APPicon，但是偏偏美工同学繁忙，怎么办，写个代码自己切图。


主要使用PIL 图片处理库，这个库的功能太过复杂，包括切片、旋转、滤镜、输出文字、调色板，在这里主要利用一点皮毛功能。PIL 的安装

windows安装PIL，pillow取代PIL
- pip install pillow

在Debian/Ubuntu Linux下直接通过apt安装：
- sudo apt-get install pillow
 
 Mac和其他版本的Linux可以直接使用easy_install或pip安装

- pip install pillow

```python
iconSize = [20,29,40,60,76]
#[83.5,1024] ipad pro, appstore

def iosIcon():
    current_path = os.path.abspath(os.curdir)
    image_path = current_path + '/' + 'bigImage/icon.jpg'
    image_path1 = current_path + '/' + 'bigImage/icon.png'

    if(not(os.path.exists(image_path)) and not(os.path.exists(image_path1))):
        print "请将图片按照规则放于指定目录下"
        return
    img = None
    if os.path.exists(image_path):
        img = Image.open(image_path)
    elif os.path.exists(image_path1):
        img = Image.open(image_path1)
    else:
        pass

    if img is None:
        print "读取图片失败"
        return

    icon_path = current_path + '/' + 'iOSIcon'
    for size in iconSize:
        new1 = img.resize((size, size))
        new2 = img.resize((size * 2, size * 2))
        new3 = img.resize((size * 3, size * 3))

        #写入到目录下
        # new1.save(r'/Users/hcd/Desktop/PythonPic/ico.png')
        new1.save(icon_path + '/' + str(size) + 'x' + str(size) + '@1x.png')
        new2.save(icon_path + '/' + str(size * 2) + 'x' + str(size * 2) + '@2x.png')
        new3.save(icon_path + '/' + str(size * 3) + 'x' + str(size * 3) + '@3x.png')

    #针对iPad Pro和App Store出个图   size为整数，浮点数报错
    # ipadPro = img.resize((83.5,83.5))
    # ipadPro.save(icon_path + '/' + 'ipadPro' + '83.5x83.5@2x.png')

    appStore = img.resize((1024, 1024))
    appStore.save(icon_path + '/' + 'appStore' + '1024x1024@1x.png')

    print "iOS切图完成~"
```

其中ipadPro那个size是浮点数，代码报错了，这就尴尬了，好像resize()的参数只能是整数
