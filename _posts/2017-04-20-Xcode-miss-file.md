---
title: Xcode miss file
tags: 码农世界
description: Xcode miss file 报错解决办法，xcode错误解决办法
---

![warning.png](http://upload-images.jianshu.io/upload_images/1406127-f2abec293d41e37a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



相信在开发中，经常会发现这样的警告，而且一出现就是大量的，造成原因：更改了项目目录文件名（或者修改了项目名），造成了文件的路径改变。总结了一些解决办法：

1. 打开Xcode->Prefrences->Source Control  去掉Enable Source Control前面的勾，这样的话Xcode就不能为我们检测文件的变化，只能靠SVN、git去管理了。

2. 进入终端，如果使用SVN管理，则svn delete命令 加上miss的文件去删除就好了，若是使用git做版本管理git rm加上miss的文件就好（注意，需要进入到相应的目录）


***
点滴积累从我做起

