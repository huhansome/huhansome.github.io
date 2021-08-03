---
title: CentOS7安装Python3.x，Python2和Python3共存
tags: 流弊技能
keywords: 云服务器，CentOS7, Python3，Python3，Linux
description: CentOS7安装Python这么简单？centos7安装Python3一看就会！
---

CentOS7默认是安装了Python2.7.x的，所以在此之上安装Python3.x是可以和Python2共存的，下面我们就说说CentOS7怎么安装Python3.x

首先登录到你的虚拟机或者云服务器，进入到如下的图形界面
![](https://s1.ax1x.com/2020/08/25/dcBh9A.png)
我这里使用的是远程连接工具xshell，可以使用别的远程连接工具，比如CRT。接下来就可以输入 `python -v `查看当前系统中的Python版本，
不出意外你会看到你的系统上默认安装了Python2.7.x，现在我们需要在这个系统上安装Python3.x

我们先安装python可能用到的依赖：

```
 yum install openssl-devel bzip2-devel expat-devel gdbm-devel readline-devel sqlite-devel
```

我们最后其实是通过wget命令是下载Python资源包进行安装，所以我们要先检查wget命令是否有安装
![](https://s1.ax1x.com/2020/08/25/dcr9xI.png)

如果没有wget命令，我们可以输入下面的命令来安装wget

```
yum -y install wget
```

wget 命令安装完成之后就可以用下面的代码下载Python资源包了，下面以Python3.6.5为例

```
wget https://www.python.org/ftp/python/3.6.5/Python-3.6.5.tgz
```
命令执行完后，你就会发现在当前目录下会有一个Python-3.6.5.tgz的文件，接下来需要解压这个文件

```
tar -zxvf Python-3.6.5.tgz
```
解压之后进入到解压后的目录 ` cd Python-3.6.5` 进行编译

```
./configure --prefix=/usr/local/python3
make && make install 
```
如果编译时报错：` configure: error: no acceptable C compiler found in $PATH ` 说明没有安装gcc，安装一下gcc即可：`yum install gcc`

如果看到下面的日志输出，则表示编译完成
![](https://s1.ax1x.com/2020/08/25/dcyT2j.png)
到这里我们虽然编译成功了，但是还不能直接使用，我们还需要下面的设置才行

```
ln -s /usr/local/python3/bin/python3 /usr/bin/python3  //链接Python3
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3   //链接pip3
```
这时候就可以测试一下可用不可用
`python -v`  `pip -v`	`python3 -v` `pip3 -v`

最后刚才下载的Python3的压缩包和解压缩目录也是可以删除的
执行 ` rm -rf Python-3.6.5.tgz Python-3.6.5`即可

**可能遇到的问题**

1. `can't decompress data: zlib not available ` 解决办法 `yum -y install zlib*`

<hr>
下面是云服务器，可以自行选择

[阿里云服务器](https://www.aliyun.com/minisite/goods?source=5176.11533457&userCode=j6bryttg)

[阿里云企业级服务器限时特惠](https://promotion.aliyun.com/ntms/act/enterprise-discount.html?source=5176.11533457&userCode=j6bryttg)

[阿里云建站 速成美站](https://ac.aliyun.com/application/webdesign/sumei?source=5176.11533457&userCode=j6bryttg)

[阿里云SaaS商标注册服务](https://tm.aliyun.com/?userCode=j6bryttg&source=5176.11533457&userCode=j6bryttg)

[CDN爆款产品](https://yqh.aliyun.com/live/cdncarnival?userCode=j6bryttg&source=5176.11533457&userCode=j6bryttg)

---------------------------------------------------------------------------------------------
[腾讯云服务器](https://cloud.tencent.com/act/cps/redirect?redirect=1067&cps_key=49b140a6ab2ad2bbd5cd3a951a0661fc&from=console)

[腾讯云境外服务器](https://cloud.tencent.com/act/cps/redirect?redirect=1001&cps_key=49b140a6ab2ad2bbd5cd3a951a0661fc&from=console)

[腾讯云数据库](https://cloud.tencent.com/act/cps/redirect?redirect=1003&cps_key=49b140a6ab2ad2bbd5cd3a951a0661fc&from=console)

