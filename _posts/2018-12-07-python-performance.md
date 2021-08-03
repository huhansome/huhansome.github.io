---
title: python性能分析技巧汇总
tags: 流弊技能
keywords: Python，性能分析, Python的性能
description: Python如何分析性能？Python性能分析技巧有哪些？Python性能如何
---

作者|Ujjwal Dalmia 编译|VK 来源|Towards Data Science

![img](https://img-blog.csdnimg.cn/20200829191209213.png)

当我们开始精通编程语言时，我们不仅希望实现最终目标，而且希望使我们的程序高效。

在这个教程中，我们将学习一些Ipython的命令，这些命令可以帮助我们对Python代码进行时间分析。

注意，在本教程中，我建议使用Anaconda。

### 1.分析一行代码

要检查一行python代码的执行时间，请使用**%timeit**。下面是一个简单的例子来了解它的工作原理：

```python
#### magics命令%timeit的简单用法
%timeit [num for num in range(20)]

#### 输出
1.08 µs ± 43 ns per loop (mean ± std. dev. of 7 runs, 1000000 loops each)1234
```

主要注意事项：

- 在要分析的代码行之前使用%timeit
- 它返回代码运行的平均值和标准偏差。在上面的示例中，执行了7次，每次执行对该代码循环100万次(默认行为)。这需要平均1.08微秒和43纳秒的标准偏差。
- 在调用magic命令时，可以自定义运行和循环的数量。示例如下：

```python
#### 在%timeit magic命令中自定义运行和循环数
%timeit -r5 -n100 [num for num in range(20)]

1.01 µs ± 5.75 ns per loop (mean ± std. dev. of 5 runs, 100 loops each)123
```

使用命令选项-r和-n，分别表示执行次数和循环次数，我们将时间配置文件操作定制为执行5次和循环100次。

### 2.分析多行代码

本节向前迈进了一步，并解释了如何分析完整的代码块。通过对%timeit magic命令进行一个小的修改，将单百分比（%）替换为双百分比（%%），就可以分析一个完整的代码块。以下为示例演示，供参考：

```python
#### 使用timeblock%%代码分析
%%timeit -r5 -n1000
for i in range(10):
    n = i**2
    m = i**3
    o = abs(i)

#### 输出
10.5 µs ± 226 ns per loop (mean ± std. dev. of 5 runs, 1000 loops each)12345678
```

可以观察到for循环的平均执行时间为10.5微秒。请注意，命令选项-r和-n分别用于控制执行次数和循环次数。

### 3.代码块中的每一行代码进行时间分析

到目前为止，我们只在分析一行代码或代码块时查看摘要统计信息。如果我们想评估代码块中每一行代码的性能呢？使用**Line_profiler** 。

**Line_profiler** 包可用于对任何函数执行逐行分析。要使用line_profiler软件包，请执行以下步骤：

- 安装—**Line_profiler** 包可以通过简单的调用pip或conda Install来安装。如果使用的是针对Python的anaconda发行版，建议使用conda安装

```python
#### 安装line_profiler软件包
conda install line_profiler1
```

**加载扩展**—一旦安装，你可以使用IPython来加载line_profiler:

```python
#### 加载line_profiler的Ipython扩展
%load_ext line_profiler1
```

**时间分析函数**—加载后，使用以下语法对任何预定义函数进行时间分析

```python
%lprun -f function_name_only function_call_with_arguments
```

**语法细节**：

- 对line_profiler的调用以关键字%lprun开始，后跟命令选项-f
- 命令选项之后是函数名，然后是函数调用

在本练习中，我们将定义一个接受高度（以米为单位）和重量（以磅为单位）列表的函数，并将其分别转换为厘米和千克。

```python
#### 定义函数
def conversion(ht_mtrs, wt_lbs ):
    ht_cms = [ht*100 for ht in ht_mtrs]
    wt_kgs = [wt*.4535 for wt in wt_lbs]

#### 定义高度和重量列表:
ht = [5,5,4,7,6]
wt = [108, 120, 110, 98]

#### 使用line_profiler分析函数
%lprun -f conversion conversion(ht,wt)

---------------------------------------------------------------
#### 输出
Total time: 1.46e-05 s

File: <ipython-input-13-41e195af43a9>

Function: conversion at line 2

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
     2       1        105.0    105.0     71.9      ht_cms = [ht*100 for ht in ht_mtrs]
     3       1         41.0     41.0     28.1      wt_kgs = [wt*.4535 for wt in wt_lbs]1234567891011121314151617181920212223
```

输出详细信息：

以14.6微秒为单位（参考第一行输出）

生成的表有6列：

- 第1列（行#）—代码的行号（请注意，第#1行是故意从输出中省略的，因为它只是函数定义语句）
- 第2列（命中）—调用该行的次数
- 第3列（时间）—在代码行上花费的时间单位数（每个时间单位为14.6微秒）
- 第4列（每次命中平均时间）—第3列除以第2列
- 第5列（%Time）—在所花费的总时间中，花在特定代码行上的时间百分比是多少
- 第6列（内容）—代码行的内容

你可以清楚地注意到，高度从米到厘米的转换几乎占了总时间的72%。

### 结束语

利用每一行代码的执行时间，我们可以部署策略来提高代码的效率。在接下来的3个教程中，我们将分享一些最佳实践来帮助你提高代码的效率。

我希望这篇教程能提供帮助，你能学到一些新东西。

[原文链接](https://towardsdatascience.com/did-you-know-you-can-measure-the-execution-time-of-python-codes-14c3b422d438)

sklearn机器学习中文官方文档： http://sklearn123.com/

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
