---
title: PolarDB-X 是如何用15M内存跑1G的TPCH
tags: 流弊技能
keywords: PolarDB-X,PostgreSQL,云建站,阿里云建站,Serverless,阿里云,aliyun,网站模板,企业官网定制,搭网站
description: 在数据时代，过多耗内存的大查询都有可能压垮整个集群，所以其内存管理模块在整个系统中扮演着非常重要的角色。
---

> **简介：** 本文主要对PolarDB-X计算层的内存管理进行分析，这有助于大家有PolarDB-X有更深入的理解。 PolarDB-X内存管理机制的设计，主要为了几类问题： 1. 让用户更容易控制每个查询的内存限制； 2. 预防内存使用不当，导致内存溢出进而引发OOM； 3. 避免查询间由于内存争抢出现相互饿死现象； 4. 避免AP Workload使用过多内存，严重拖慢TP Workload

在数据时代，过多耗内存的大查询都有可能压垮整个集群，所以其内存管理模块在整个系统中扮演着非常重要的角色。

而PolarDB-X 作为一款分布式数据库，其面对的数据可能从TB到GB字节不等，同时又要支持TP和AP Workload，要是在计算过程中内存使用不当，不仅会造成TP和AP相互影响，严重拖慢响应时间，甚至会出现内存雪崩、OOM问题，导致数据库服务不可用。


CPU和MEMORY相对于网络带宽比较昂贵，所以PolarDB-X 代价模型中，一般不会将涉及到大量数据又比较耗内存的计算下推到存储DN，DN层一般不会有比较耗内存的计算。这样还有一个好处，当查询性能不给力的时候，无状态的CN节点做弹性扩容代价相对于DN也低。

鉴于此，所以本文主要对PolarDB-X计算层的内存管理进行分析，这有助于大家有PolarDB-X有更深入的理解。


PolarDB-X内存管理机制的设计，主要为了几类问题：

1. 让用户更容易控制每个查询的内存限制；
2. 预防内存使用不当，导致内存溢出进而引发OOM；
3. 避免查询间由于内存争抢出现相互饿死现象；
4. 避免AP Workload使用过多内存，严重拖慢TP Workload


**业界解决方案**



在计算层遇到的内存问题，业界其他产品也会遇到。这里我们先看下业界对此类问题是如何解决的。

### **PostgreSQL**

![1.jpg](https://ucc.alicdn.com/pic/developer-ecology/eb910c20f7884db8bd0737cbbede4e84.jpg)



为了面对内存问题，PG的内存管理有两大特点:

1. 内存按照一定的比例，分为四大区域(Shared Buffers、Temp Buffers、Work Mem、Maintenance Work Mem)；
2. 在查询过程算子主要使用Work Mem区域内存，且每个算子都是预先分配固定内存的，但内存不够时，需要与硬盘进行swap；



所以PG在生产中要求业务方合理配置内存比例，配置不当的话会极大的降低系统的性能。

一般有两种方式去指导业务方配置：估计方法与计算方法。

第一种是可以根据业务量的大小和类型，一般语句运行时间，来粗略的估计一下。第二种方式是通过对数据库的监控，数据采集，然后计算其大小。



### **Flink**

**
**

![2.jpg](https://ucc.alicdn.com/pic/developer-ecology/89a77c6fce384e3bb7f9c3db4579689a.jpg)

接下来我们看看作为大数据库实时领域非常流行的计算引擎Flink，他的内存管理特点是：

1. 引入了一套基于字节流的数据结构，在JVM虚拟机上实现了类似C申请和释放内存的池化管理；
2. 内存主要分为三块: 网络内存池、系统预留的内存池、计算内存池。
3. 为每个算子设置一个[min-memory, max-meory] 调度上可以保证每个算子初始化的时候预先分配min memory，在计算过程中不断从计算内存池申请内存，当申请失败或者内存使用超过算子预先定义max memory的时候，主动触发数据落盘。



Flink相对于PG来说，在计算过程中引入了动态管理机制，但这种机制是有限的。主要应该有两点考虑: 

1. 整个计算是动态的，其实是很难权衡哪个算子抢占权重高，如果任意放开，肯定会影响性能；
2. Flink是流水线计算模型，类似PartialAgg算子一定不落盘，所以一旦PartialAgg占用过多内存，导致下游没有内存可用，而PartialAgg又一定会往下游发送数据，这样必然会导致Dead Lock。





### **Spark**



说到计算层的内存管理，就不得不提SPARK。SPARK早在2015年就提出了Project Tungsten。

当然钨丝计划的提出，主要是为了不要让硬件成为Spark性能的瓶颈，无限充分利用硬件资源。而其中内存管理的设计不仅仅可以减少JVM内存释放的开销和垃圾回收的压力，结合落盘能力可以支持海量数据的查询。



![3.jpg](https://ucc.alicdn.com/pic/developer-ecology/0f260f79777e45ec95d2d72450bde321.jpg)



Spark 内存管理和Flink很像的，这里就不展开来说。但是它支持完全动态内存抢占，这种动态内存抢占主要体现在：Excution和Storage内存可以相互强制；Task间可以相互抢占；Task内部可以相互抢占。Spark这里假设了每个Task动态抢占内存的能力相当于，理论值都是1/N (N表示进程内部Task数量)。

Spark其实是可以这么假设的，他是典型的MR模型，只有Map端计算完后，才调度Reduce端，所有在一个进程内部多个Task你可以简单认为都是Map Task。当某个Task使用的内存不足1/2N时，它会等到其他Task释放内存。而Task间最多可以抢占的阈值也是1/N，而Task内算子是可以完全相互动态抢占的。

Spark的计算模型决定了他可以做到不预先为每个Task和算子预先分配内存，随着N的变化，动态调整每个Task的抢占能力。个人觉得这种动态抢占内存的方案，相对于其他计算引擎优越不少。但是由于Spark主要是针对大数据的ETL场景，稳定性至关重要，尽量避免任务重试。所以对每个Task抢占的上界都做了约束(Task Memory < 1/N * Executor Memory)，确保每个Task都有内存执行任务。



### **Presto**

**
**

**![4.jpg](https://ucc.alicdn.com/pic/developer-ecology/9f37d1c0394241c48f811651772cfbe8.jpg)**

Presto是由 Facebook 推出的一个基于Java开发的开源分布式SQL查询引擎。它的特点是:

1. 采用的是逻辑内存池，来管理不同类型的内存需求。
2. Presto整体将内存分为三块: System Pool、Reserved Pool、General Pool。
3. Presto内存申请其实是不预分配的，且抢占是完全随机的，理论上不设定Task抢占的上限，但为了出现某个大查询把内存耗尽的现象，所以后台会有定时轮训的机制，定时轮训和汇总Task使用内存，限制Query使用的最大内存；同时当内存不够用的时候，会对接下来提交的查询做限流，确保系统能够平稳运行。



考虑到Presto是基于逻辑计数方式做内存管理，所以是不太精准的。但逻辑计算的好处是，管理内存成本比较低，这又往往适合对延迟要求特别低的在线计算场景。



## **PolarDB-X内存管理**



业界基本上都会将内存分区域管理，针对于计算层主要区别在于: 内存是否预分配和查询过程中是否支持动态抢占。而PolarDB-X 是存储计算分离的架构，CN节点是无状态的，且主要针对于HTAP场景，所以对实时性要求比较高。

能否充分使用内存，对我们的场景比较关键。所以我们采取类似Presto这种无预分配内存，且支持动态抢占内存方案。除此之外，我们会根据不同的WorkLoad来管理内存，充分满足HTAP场景。

### **统一的内存模型**

PolarDB-X内存也是分区域管理。结合自身的计算特点，按照不同的维度会有不同的划分和使用方式。

- **结合HTAP特性，我们将内存池划分为：**

1. System Memory Pool：系统预留的内存，主要用于分片元数据和数据结构、临时对象;
2. Cache Memory Pool：用于管理Plan Cache和其他LRU Cache的内存；
3. AP Memory Pool: 用于管理WorkLoad是AP的内存;
4. TP Memory Pool: 用于管理WorkLoad是TP的内存;



![5.jpg](https://ucc.alicdn.com/pic/developer-ecology/92a853e12c934063b8e15e653cfbc9b2.jpg)

相对于业界其他产品来说，我们主要划分出了AP和TP的内存区域。这么做是为了做TP/AP Workload在内存使用上的资源隔离。

- **结合我们的计算层次结构，我们按照树形结构去管理内存:**



![6.jpg](https://ucc.alicdn.com/pic/developer-ecology/b6cd58a740b043f2a464a6c8dde799de.jpg)

一个TP Workload的查询，我们会从TP Memory 申请出Query Memory，然后按照我们的计算层次关系，又会划分出Task/Pipeline/Driver/Operator Memory。

这样的好处是，我们可以动态监测不同维度下的内存使用情况。这里唯一要注意的是在Query Memory下，我们额外会创建一个Planner Memory，用于管理查询时和DN交互的数据对象的内存。

- **结合算子对内存申请和释放的行为，我们对一个Operator Memory划分为两个内存块: Reserve和Revoke。**



![7.jpg](https://ucc.alicdn.com/pic/developer-ecology/30991272d7e1462da5e5190d110d8531.jpg)

Reserve Memory顾名思义就是算子一旦申请了内存就一定不会被动释放，直到该算子运行结束，主动释放内存，比如Scan算子，主要和DN交互数据的，一般都采样流式实现，每次运行都从DN获取一批数据，然后发送给下游，该算子一般不支持数据落盘，所以Scan算子都是申请Reseve Memory。

而Revoke Memory表示算子申请内存后，可以通过触发数据落盘的方式被动释放，将申请到的Revoke Memory归还到内存池，以供其他算子使用。比如HashAgg算子，都是申请Revoke Memory，但它申请的内存过多，导致这个查询其他算子无内存可用，就会被框架触发HashAgg数据落盘，将其申请的内存统统归还给内存池。

这里需要注意PolarDB-X对于Reserve和Revoke两块内存并没有按比例严格划分，这两者是可以完全相互抢占的。





### **内存对象**

对于内存管理上，比较重要的实现类就是Memory Pool和MemoryAllocatorCtx。其中Memory Pool提供了申请和释放内存的API。

考虑到内存是按树形结构管理起来的，申请一次内存会涉及到多次函数调用，所以这里封装了MemoryAllocatorCtx对象，用于确保每次申请/释放内存的最小单位是512Kb，避免对MemoryPool相关函数多次调用的开销。且MemoryAllocatorCtx对象会记录上一次内存申请失败的大小，是框架用于判断内存是否不够用的重要标志。



```
interface MemoryPool {``//申请reserve 内存``ListenableFuture<?> allocateReserveMemory(long size);``//尝试着申请reserve 内存``boolean tryAllocateReserveMemory(long size, ListenableFuture<?> allocFuture);``//申请revoke 内存``ListenableFuture<?> allocateRevocableMemory(long size);``//释放reserve 内存``void freeReserveMemory(long size);``//释放revoke 内存``void freeRevocableMemory(long size);``}
```



在内存的使用上，我们采样的是非预分配模型，在计算调度上不需要额外考虑每个算子的使用内存。每个算子都是在执行期间按需去申请释放内存的，但这并不是意味着算子就可以任意去申请内存。

一旦所有的算子使用内存之和超过查询规定的最大内存，或者内存不够用的时候，我们都会阻塞当前查询，为此我们在原有的MemoryPool基础之上派生出了BlockingMemory；同样的为了方便AP和TP Workload基于内存做自适应的资源隔离，我们进一步派生出了AdaptiveMemory。



![8.jpg](https://ucc.alicdn.com/pic/developer-ecology/a3e7e95ed73b4f50bafd327d07b91d70.jpg)

BlockingMemory：内存申请过中，会根据内存是否超过一定的阈值，创建阻塞对象，该对象会被执行线程引用。一般来说全局内存或者Query内存超过一定的阈值(0.8)的时候，就会触发当前算子申请内存失败，并且主动退出执行，这个和Spark算子申请不到内存后，阻塞当前线程的行为是不一致的。

AdaptiveMemory：用于做TP/AP的内存管理，在申请过程中会根据AP和TP的内存占比做一些自适应调整，比如触发限流、query自杀、大查询落盘等操作。



### **动态抢占内存机制**

在内存申请和释放的行为上，我们采样的都是非阻塞模型，这种设计可以很好的和我们的时间片执行框架结合。结合这种设计，PolarDB-X算子都是不预分配内存的，各个算子都是在运行过程中完全动态抢占内存。

这钟动态抢占内存主要体现在: AP Memory 和TP Memory之间、Query Memory之间以及Operator Memory的Revoke和Reserve之间。

而实现这种抢占机制的基本单元依然是Driver，由下图可知，Driver会在运行过程中会根据当前Worload的内存空间、Query Used Memory 和 Operator Used Memory，来判断执行线程是否需要让出执行线程，被内存阻塞。一旦发现内存不够用的话，会主动退出执行队列，加入到阻塞队列，直到内存空间满足一定的条件，才会唤醒该Driver。



![9.jpg](https://ucc.alicdn.com/pic/developer-ecology/33950af2147640b0815c556d549d97e0.jpg)



Driver被内存阻塞主要条件是：

1. 当内存池子(AP/TP Memory Pool)内存不足阈值的0.8之时，申请内存的算子会在下一刻会被暂停执行，等待被触发数据落盘，以到达释放内存的目的。
2. 当前查询申请的总内存超过当前Max Query Memory阈值的0.8之后，申请的算子会被暂停执行等待被被触发数据落盘，以到达释放内存的目的。



当Driver被内存阻塞后，Driver会退出执行线程，将线程资源拱手相让给其他Driver。同时会回调MemoryRevokeService服务，该服务主要是基于内存大小和Pipeline的依赖关系挑选出耗内存的Driver进行标记。



![10.jpg](https://ucc.alicdn.com/pic/developer-ecology/12b26cea1f9a4931b7225f442c6d78fb.jpg)



当内存不够时，CN会基于Used Memory Size对Task做排序，依次对占用大内存的Task打标记；Taks内部则基于依赖关系，对Pipeline做排序，依次对父节点的Pipeline打标；Pipeline内部首先基于Driver Used Memory Siz做排序后，再结合Operator前后依赖关系，来决定Operator的内存释放顺序。

这里唯一需要注意的是MemoryRevokeService只是对需要释放内存的Operator进行打标记，然后唤醒对应的Driver，等待被调度的时候由执行线程池来触发数据落盘。

这里可能会有一个疑问？为什么打标记的同时不立刻触发数据落盘释放内存呢。由于Driver被落盘标记的时候，也有可能正在执行线程池执行，如果这个时候触发Driver执行落盘操作的时候，Driver会有并发安全的问题。所以我们将Driver的落盘动作和执行动作都交给执行线程池统一处理。



![11.jpg](https://ucc.alicdn.com/pic/developer-ecology/d7b2f5a1558f4f2aac8f56ba9da924dc.jpg)



从上图还可知，一旦Driver被MemoryRevokeService唤醒，就调度到执行线程池中，首先做自我巡检，判断当前Driver是否包含SPILL标记，如果是的话，就需要触发Driver上相应算子做数据落盘的动作，这里唯一需要注意的是算子真正做数据落盘的逻辑是完全异步的，不占用执行线程。

一旦Driver开始做异步落盘后，也会主动退出执行线程，直到异步落盘结束后，才会唤醒当前Driver。整个过程确保了执行线程的资源不会被卡住。



### **基于负载的内存隔离**



![1.jpg](https://ucc.alicdn.com/pic/developer-ecology/3663a04bc2324b22be316f3c117b8baf.jpg)

PolarDB-X在HTAP场景下是希望做到AP Workload不影响TP Workload。但当系统承载的都是AP Workload时，AP可以充分利用所有资源；当系统承载的都是TP Worload时，TP可以充分利用所有的资源。这里头我们巧妙了利用了TP Memory的阈值来到达目的。

1. Driver 执行过程中，按需申请内存，如果内存足够，会反复被调度执行；
2. 当Driver被内存阻塞时，判断其Workload；
3. 如果是AP，则通过MemoryRevokeService服务触发数据落盘；如果是TP Workload，则在触发数据落盘的同时，会判断当前TP Memory是否超过TP Memory Pool，若不是，则说明AP Workload使用内存过多，则回调AP Workload做调整；
4. 回调的方式有两种：触发部分AP Worload自杀，且触发Ap Worload 基于令牌桶的方式做滑动限流。



其中系统触发AP Worload自杀的回调机制，是比较危险的，默认是关闭的。而一旦触发AP Worload限流，则令牌桶流入的速率会减半，这样接下来的AP workload请求可能获取不到Token，而只能排队等待。

默认我们将TP Memory Pool的值设置为内存最大可利用资源的80%，表示在没有AP查询时，TP可以使用100%资源，在没有TP查询时，AP可以使用100%资源，当TP和AP Worload比较高时，AP最大只能占用20%的资源。



# **数据测试**

# 基于15MB的内存跑通1g TPCH，这里我们截取了几个比较耗内存的Query统计了落盘文件的数量。



![2.jpg](https://ucc.alicdn.com/pic/developer-ecology/e7bc1623ba304fb6a5c597089e6148f1.jpg)

测试了TPCC和TPCH混跑情况下，AP 对TP的影响。这部分测试也涵盖了CPU的资源隔离，后面我们会单独开一篇谈谈PolarDB-X基于负载的资源隔离技术。在不开启TPCH的情况下，tpmc保持在3.2w-3.4w之间; 再开启TPCH后，tpmc基本可以维持在3w以上，且有抖动10%左右。



![3.jpg](https://ucc.alicdn.com/pic/developer-ecology/d43e8d2bc9df454e9ac614d6adae7b92.jpg)



**总结**



![4.jpg](https://ucc.alicdn.com/pic/developer-ecology/82df6393bf24465b9bc206915381622b.jpg)

从上表对比来看：

1. PolarDB-X算子不预分配内存，而是在计算过程中按需去申请内存，可以充分提高内存利用率；
2. 支持Workload/Query/Operator维度的内存动态抢占，比较适合HTAP场景，可以通过设置不同维度的阈值，尽可能确保在内存上TP Workload不受AP的干扰；
3. 相对于PostgreSQL/Flink/Spark来，PolarDB-X释放内存的时机是Lazy的。就是说当算子内存不够的时候，算子并不是立即落盘释放内存的，而是退出执行线程，由另外一个服务计算出更加耗内存的算子，这种方案可以挑选更加耗内存的查询或者算子做数据落盘，避免对小查询造成影响。但Lazy的方式存在一定的风险，可能在一瞬间内存未及时释放，而运行的查询申请内存过多，导致OOM，但好在PolarDB-X是计算存储分离的架构，计算层是无状态的；
4. PolarDB-X 相对于Presto 动态内存抢占的粒度更加细，将Workload也纳入考虑。但Presto在实现的细节上会考虑大查询间内存的相互影响。而在HTAP场景下，我们更加注重AP对TP的影响，AP内查询间并没有额外的机制去保证内存不受影响。



从业界产品来看，每个产品在内存管理上都有各自的特点，这和产品本身的定位是有一定关系。而PolarDB-X作为一款计算存储分离的HTAP数据库来说，其计算层目前采用的完全动态内存抢占方案，可以做到充分使用内存，避免AP对TP的影响。

<hr>

[阿里云服务器，限时秒杀价低至86/年，券后66/年](https://www.aliyun.com/minisite/goods?source=5176.11533457&userCode=j6bryttg)

[阿里云企业级服务器1折限时特惠](https://promotion.aliyun.com/ntms/act/enterprise-discount.html?source=5176.11533457&userCode=j6bryttg)

[阿里云建站 速成美站 - 千套模板，价格低至500元](https://ac.aliyun.com/application/webdesign/sumei?source=5176.11533457&userCode=j6bryttg)

[阿里云SaaS商标注册服务低至680元](https://tm.aliyun.com/?userCode=j6bryttg&source=5176.11533457&userCode=j6bryttg)

[CDN爆款产品低至5.5折](https://yqh.aliyun.com/live/cdncarnival?userCode=j6bryttg&source=5176.11533457&userCode=j6bryttg)
<hr>
[腾讯云服务器限时秒杀](https://cloud.tencent.com/act/cps/redirect?redirect=1067&cps_key=49b140a6ab2ad2bbd5cd3a951a0661fc&from=console)

[腾讯云境外服务器低至2折](https://cloud.tencent.com/act/cps/redirect?redirect=1001&cps_key=49b140a6ab2ad2bbd5cd3a951a0661fc&from=console)

[腾讯云数据库](https://cloud.tencent.com/act/cps/redirect?redirect=1003&cps_key=49b140a6ab2ad2bbd5cd3a951a0661fc&from=console)