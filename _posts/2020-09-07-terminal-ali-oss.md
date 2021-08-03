---
title: 单行命令完玩转阿里云OSS
tags: 流弊技能
keywords: OSS，阿里云，云存储，阿里云存储
description: 阿里云oss用命令行操作？这也太简单了吧！
---


### 1、第一步

下载并配置好osscmd
config --host=[oss.aliyuncs.com] --id=[accessid] --key=[accesskey]

### 2、批量上传文件

#### 上传文件夹

```
find $dirname | sed 's/\.\///g' | xargs -I {} osscmd put {} oss://$bucket-name/{}
```

如果是上传当前目录将&dollar;dirname 替换成"."
&dollar;dirname 要上传文件名称, 当然要在这个目录上层运行此命令, 可以利用grep -v 过滤掉一些无需上传的文件, 当然osscmd也有上传文件夹功能

#### 批量上传一批文件

```
cat filelist | xargs -I {} osscmd put {} oss://$bucket-name/$dirname/{}
```

filelist 文件名列表

### 3、 文件Size 统计

#### 列出不同大小文件个数

```
osscmd listallobject oss://$bucket-name/$dirname 2> /dev/null | grep "oss://" | awk '{print $3}'  | sort | uniq -c
```

样例输出
1 0.00B
3 1011.50KB
1 115.23KB
1 1167.00B
1 11.96KB
1 12.08KB
3 12.84MB
1 1355.46KB
1 17.28MB
1 18.03MB
2 209.49KB

#### 统计出OSS Bucket 文件夹不同大小级别文件个数 （B KB MB GB）

```
osscmd listallobject oss://$bucket-name/$dirname 2> /dev/null | grep "oss://" | awk '{print $3}' | rev | cut -c 1-2 | rev | sort | uniq -c | sed 's/0B/B/g'
```

样例输出

```
  8  B
 30 KB
 10 MB
```

#### 统计文件夹大小

```
osscmd listallobject oss://$bucket-name/$dirname 2> /dev/null | grep "oss://" | awk '{print $3}' | sed 's/MB/*1024^2/g' | sed 's/GB/*1024^3/g' | sed 's/KB/*1024/g' | tr -d 'B'  | tr '\n' '+' | awk '{print "("$1"0)"}'  | bc
```

算出来的结果是字节数，转换成其他单位需自行转换,如果结果有小数请不要奇怪

### 3、签名

#### 给文件夹下所有文件进行签名

```
osscmd listallobject oss://$bucket-name/$dirname 2> /dev/null | grep "oss://"  | awk '{print $4}' | xargs -I {} osscmd sign {} 2> /dev/null | grep http
```

### 4、批量删除文件

```
osscmd  listallobject oss://$bucket-name/$dirname 2> /dev/null | grep "oss://"  | awk '{print $4}' | xargs -I {} osscmd rm {} 
```

批量删除文件下下的所有文件，或者制定前缀的文件, 更多功能请见OSS的lifecycle功能
当然删除文件列表也是OK的

```
cat filelist | xargs -I {} osscmd rm {}
```

### 福利部分

[阿里云服务器](https://www.aliyun.com/minisite/goods?source=5176.11533457&userCode=j6bryttg)

[阿里云企业级服务器限时特惠](https://promotion.aliyun.com/ntms/act/enterprise-discount.html?source=5176.11533457&userCode=j6bryttg)

[阿里云建站 速成美站](https://ac.aliyun.com/application/webdesign/sumei?source=5176.11533457&userCode=j6bryttg)

[阿里云SaaS商标注册服务](https://tm.aliyun.com/?userCode=j6bryttg&source=5176.11533457&userCode=j6bryttg)

[CDN爆款产品](https://yqh.aliyun.com/live/cdncarnival?userCode=j6bryttg&source=5176.11533457&userCode=j6bryttg)

---------------------------------------------------------------------------------------------
[腾讯云服务器](https://cloud.tencent.com/act/cps/redirect?redirect=1067&cps_key=49b140a6ab2ad2bbd5cd3a951a0661fc&from=console)

[腾讯云境外服务器](https://cloud.tencent.com/act/cps/redirect?redirect=1001&cps_key=49b140a6ab2ad2bbd5cd3a951a0661fc&from=console)

[腾讯云数据库](https://cloud.tencent.com/act/cps/redirect?redirect=1003&cps_key=49b140a6ab2ad2bbd5cd3a951a0661fc&from=console)