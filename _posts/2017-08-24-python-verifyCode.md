---
title: Python实现动态验证码功能
tags: 人生苦短，就用python
description: Python如何生成验证码；Python生成网站验证码，python图片处理：绘制
---

[上次切图](https://huhansome.github.io/_posts/2017-07-18-python-iOSicon/)用到了强大的[PIL](http://www.pythonware.com/products/pil/)库，这次继续玩一玩PIL。在某些网站经常会看到验证码图片，于是就想到自己写一个看看，python写这种验证码好像可以写的很炫的样子。

这次主要用到PIL的图片生成功能，顺带用了一下模糊滤镜。

```python
#!/usr/bin/python
# -*- coding:utf-8 -*-

from PIL import  Image, ImageDraw, ImageFont, ImageFilter
import random
from random import randint

#随机字母: 48-57 97-122 65-90
def randomChar():
    while(True):
        c = randint(48, 122)
        if checkValid(c):
            return chr(c)

def checkValid(c):
    number = (c >= 48) and (c <= 57)
    charLow = (c >= 97) and (c <= 122)
    charUpper = (c >= 65) and (c <= 90)
    if (number or charLow or charUpper):
        return True
    else:
        return False

# 随机颜色1:
def backgroundColor():
    return (random.randint(64, 255), random.randint(64, 255), random.randint(64, 255))

# 随机颜色2:
def fontColor():
    return (random.randint(32, 127), random.randint(32, 127), random.randint(32, 127))

#图片的大小
width = 60 * 4
height = 60
image = Image.new('RGB', (width, height), (255, 255, 255))
# 创建Font对象: font路径问题会导致can't open resource 错误,我是将字体放在当前目录的
font = ImageFont.truetype('arial.ttf', 36)
# 创建Draw对象:
draw = ImageDraw.Draw(image)
# 填充每个像素:
for x in range(width):
    for y in range(height):
        draw.point((x, y), fill= backgroundColor())
# 输出文字:
for t in range(4):
    draw.text((60 * t + 10, 10), randomChar(),font = font, fill = fontColor())
# 模糊:
image = image.filter(ImageFilter.BLUR)
image.save('yzm.jpg', 'jpeg');

```
最后看看效果&nbsp; ![验证码](https://github.com/CheaterHu/CheaterHu.github.io/blob/master/myImages/yzm.jpg?raw=true)
