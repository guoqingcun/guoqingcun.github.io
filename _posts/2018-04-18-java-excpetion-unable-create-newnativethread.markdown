---
layout: post
title:  "记一次OOM：unable to create new native thread"
date:   2018-05-10 00:24:17 +0800
categories: java
location: 
description: "OOM unable to create new native thread"
---
---

### VirtualMachineError
在进入到本文主题之前，我们先了解下Java虚拟机的的error类:VirtualMachineError。

VirtualMachineError是在Java虚拟机损坏或所需资源耗尽时抛出。而抛出的error具体类包括如下四种
>
- InternalError:底层主机系统软件有利故障或硬件错误
- OutOfMemoryError:Java虚拟机耗尽虚拟内存或物理且GC不能回收足够的空间时的错误
- StackOverflowError:Java虚拟机耗尽了线程的堆栈空间，通常是由于程序执行无数次递归调用所至。
- UnknowError:发生异常或错误，但Java虚拟机无法得知实际的异常或错误。

如上四种皆是VirtualMachineError的子类

### OutOfMemoryError
本文主要说的是OOM(OutOfMemoryError)。首先让我们看看产生OOM(OutOfMemoryError)的几种情况。
>
1. java.lang.OutOfMemoryError:Java heap space
2. java.lang.OutOfMemoryError:Perm space
3. java.lang.OutOfMemoryError:unable to create new native thread
4. java.lang.OutOfMemoryError:Gc overhead limit exceeded

第一种情况是由于程序申请更多堆空间,而堆空间不足导致

第二种情况是由于程序申请永久带空间，空间不足导致

第三种情况是由于创建线程数达到是所限的数量，无法创建新线程所至

第四种情况是由于是在并行或者并发回收器在GC回收时间过长、超过98%的时间用来做GC并且回收了不到2%的堆内存，然后抛出这种异常进行提前预警，用来避免内存过小造成应用不能正常工作。

扯了这么多，都是做为本文的引子，今天主要谈的是第三种情况
> java.lang.OutOfMemoryError:unable to create new native thread

### 排差问题
写本文的原因也是由于本人线上遇到此问题后决定做一个笔录。如前面所述，本错误是由于达到了最大线程数，无法创建新线程导致。那为什么不能生成新的线程呢?
>    具体原因如下两点
-	    1、操作系统不能提供足够物理内存
-		2、线程(linux下是轻量的进程数)已到操作系统设置的上限
		
#### 第一种情况：
当我在日志文件中看到此OOM后，使用free -h查看了下系统内存情况.
```
[xxx@face-tomcat01 ~]# free -h
             total       used       free     shared    buffers     cached
Mem:          3.7G       3.6G       177M       184K       286M       1.4G
-/+ buffers/cache:       1.9G       1.8G
Swap:           0B         0B         0B
```
从上面输出可以得出，生产机器4G内存，使用了3个多G，剩余177M，操作系统cache使用1.4G。	操作系统cache使用的1.4G是用来缓存IO数据，进程内存不足时，可以释放出来优先分配给进程使用。
一个线程默认为1M，也就是说还可以new出177个新线程。所以第一个错误原因可以排除掉。

#### 第二种情况：
那让我们看看是不是第二种情况导致的问题.首先使用```ulimit -a```查看系统对用户使用资源的限制：

>  max user processes              (-u) 1024

如此说明允许用户最大使用```1024```个进程。接下来我们看看jvm线程数的情况,
使用```jstack pid > xxx.log``` 导出运行时线程数据。再用```grep "Thread " xxx.log | wc -l``` 统计线程数量为1014。线程数量一直在1024以下波动。

产生OOM的原因已经找到，具体代码上是哪里引起的，此处略，请在各自的xxx.log中找寻原因。