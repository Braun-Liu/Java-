---
typora-copy-images-to:media
---

# 常见的 OOM 原因及其解决方法

当 JVM 内存严重不足时，就会抛出 java.lang.OutOfMemoryError 错误。本文总结了常见的 OOM 原因及其解决方法，如下图所示。如有遗漏或错误，欢迎补充指正。

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy90TzdORU43d2pyN3hkVWN4a0RENGhEaWFJZHJoUXRkQ01JZGNsUzVpYWdFZWVHVWdDbFlUbW9ESktRNnBuanZtamMxdmNmMWZPYTM3c01saWNiNFNybFl3dy82NDA_d3hfZm10PXBuZyZ0cD13ZWJwJnd4ZnJvbT01Jnd4X2xhenk9MSZ3eF9jbz0x)

1、Java heap space

当堆内存（Heap Space）没有足够空间存放新创建的对象时，就会抛出 `java.lang.OutOfMemoryError:Javaheap space` 错误（根据实际生产经验，可以对程序日志中的 OutOfMemoryError 配置关键字告警，一经发现，立即处理）。

### 原因分析

`Javaheap space` 错误产生的常见原因可以分为以下几类：

1、请求创建一个超大对象，通常是一个大数组。

2、超出预期的访问量/数据量，通常是上游系统请求流量飙升，常见于各类促销/秒杀活动，可以结合业务流量指标排查是否有尖状峰值。

3、过度使用终结器（Finalizer），该对象没有立即被 GC。

4、内存泄漏（Memory Leak），大量对象引用没有释放，JVM 无法对其自动回收，常见于使用了 File 等资源没有回收。

### 解决方案

针对大部分情况，通常只需要通过 `-Xmx` 参数调高 JVM 堆内存空间即可。如果仍然没有解决，可以参考以下情况做进一步处理：

1、如果是超大对象，可以检查其合理性，比如是否一次性查询了数据库全部结果，而没有做结果数限制。

2、如果是业务峰值压力，可以考虑添加机器资源，或者做限流降级。

3、如果是内存泄漏，需要找到持有的对象，修改代码设计，比如关闭没有释放的连接。

## 2、GC overhead limit exceeded

当 Java 进程花费 98% 以上的时间执行 GC，但只恢复了不到 2% 的内存，且该动作连续重复了 5 次，就会抛出 `java.lang.OutOfMemoryError:GC overhead limit exceeded` 错误。简单地说，就是应用程序已经基本耗尽了所有可用内存， GC 也无法回收。

此类问题的原因与解决方案跟 `Javaheap space` 非常类似，可以参考上文。

## 3、Permgen space

该错误表示永久代（Permanent Generation）已用满，通常是因为加载的 class 数目太多或体积太大。

### 原因分析

永久代存储对象主要包括以下几类：

1、加载/缓存到内存中的 class 定义，包括类的名称，字段，方法和字节码；

2、常量池；

3、对象数组/类型数组所关联的 class；

4、JIT 编译器优化后的 class 信息。

PermGen 的使用量与加载到内存的 class 的数量/大小正相关。

### 解决方案

根据 Permgen space 报错的时机，可以采用不同的解决方案，如下所示：

1、程序启动报错，修改 `-XX:MaxPermSize` 启动参数，调大永久代空间。

2、应用重新部署时报错，很可能是没有应用没有重启，导致加载了多份 class 信息，只需重启 JVM 即可解决。

3、运行时报错，应用程序可能会动态创建大量 class，而这些 class 的生命周期很短暂，但是 JVM 默认不会卸载 class，可以设置 `-XX:+CMSClassUnloadingEnabled` 和 `-XX:+UseConcMarkSweepGC`这两个参数允许 JVM 卸载 class。

如果上述方法无法解决，可以通过 jmap 命令 dump 内存对象 `jmap-dump:format=b,file=dump.hprof<process-id>` ，然后利用 Eclipse MAT https://www.eclipse.org/mat 功能逐一分析开销最大的 classloader 和重复 class。

## 4、Metaspace

JDK 1.8 使用 Metaspace 替换了永久代（Permanent Generation），该错误表示 Metaspace 已被用满，通常是因为加载的 class 数目太多或体积太大。

此类问题的原因与解决方法跟 `Permgenspace` 非常类似，可以参考上文。需要特别注意的是调整 Metaspace 空间大小的启动参数为 `-XX:MaxMetaspaceSize`。

## 5、Unable to create new native thread

每个 Java 线程都需要占用一定的内存空间，当 JVM 向底层操作系统请求创建一个新的 native 线程时，如果没有足够的资源分配就会报此类错误。

### 原因分析

JVM 向 OS 请求创建 native 线程失败，就会抛出 `Unableto createnewnativethread`，常见的原因包括以下几类：

1、线程数超过操作系统最大线程数 ulimit 限制；

2、线程数超过 kernel.pid_max（只能重启）；

3、native 内存不足；

该问题发生的常见过程主要包括以下几步：

1、JVM 内部的应用程序请求创建一个新的 Java 线程；

2、JVM native 方法代理了该次请求，并向操作系统请求创建一个 native 线程；

3、操作系统尝试创建一个新的 native 线程，并为其分配内存；

4、如果操作系统的虚拟内存已耗尽，或是受到 32 位进程的地址空间限制，操作系统就会拒绝本次 native 内存分配；

5、JVM 将抛出 `java.lang.OutOfMemoryError:Unableto createnewnativethread` 错误。

### 解决方案

1、升级配置，为机器提供更多的内存；

2、降低 Java Heap Space 大小；

3、修复应用程序的线程泄漏问题；

4、限制线程池大小；

5、使用 -Xss 参数减少线程栈的大小；

6、调高 OS 层面的线程最大数：执行 `ulimia-a` 查看最大线程数限制，使用 `ulimit-u xxx` 调整最大线程数限制。

ulimit -a .... 省略部分内容 ..... max user processes (-u) 16384

## 6、Out of swap space？

该错误表示所有可用的虚拟内存已被耗尽。虚拟内存（Virtual Memory）由物理内存（Physical Memory）和交换空间（Swap Space）两部分组成。当运行时程序请求的虚拟内存溢出时就会报 `Outof swap space?` 错误。

### 原因分析

该错误出现的常见原因包括以下几类：

1、地址空间不足；

2、物理内存已耗光；

3、应用程序的本地内存泄漏（native leak），例如不断申请本地内存，却不释放。

4、执行 `jmap-histo:live<pid>` 命令，强制执行 Full GC；如果几次执行后内存明显下降，则基本确认为 Direct ByteBuffer 问题。

### 解决方案

根据错误原因可以采取如下解决方案：

1、升级地址空间为 64 bit；

2、使用 Arthas 检查是否为 Inflater/Deflater 解压缩问题，如果是，则显式调用 end 方法。

3、Direct ByteBuffer 问题可以通过启动参数 `-XX:MaxDirectMemorySize` 调低阈值。

4、升级服务器配置/隔离部署，避免争用。

## 7、 Kill process or sacrifice child

有一种内核作业（Kernel Job）名为 Out of Memory Killer，它会在可用内存极低的情况下“杀死”（kill）某些进程。OOM Killer 会对所有进程进行打分，然后将评分较低的进程“杀死”，具体的评分规则可以参考 Surviving the Linux OOM Killer。

不同于其他的 OOM 错误， `Killprocessorsacrifice child` 错误不是由 JVM 层面触发的，而是由操作系统层面触发的。

### 原因分析

默认情况下，Linux 内核允许进程申请的内存总量大于系统可用内存，通过这种“错峰复用”的方式可以更有效的利用系统资源。

然而，这种方式也会无可避免地带来一定的“超卖”风险。例如某些进程持续占用系统内存，然后导致其他进程没有可用内存。此时，系统将自动激活 OOM Killer，寻找评分低的进程，并将其“杀死”，释放内存资源。

### 解决方案

1、升级服务器配置/隔离部署，避免争用。

2、OOM Killer 调优。

## 8、Requested array size exceeds VM limit

JVM 限制了数组的最大长度，该错误表示程序请求创建的数组超过最大长度限制。

JVM 在为数组分配内存前，会检查要分配的数据结构在系统中是否可寻址，通常为 `Integer.MAX_VALUE-2`。

此类问题比较罕见，通常需要检查代码，确认业务是否需要创建如此大的数组，是否可以拆分为多个块，分批执行。

## 9、Direct buffer memory

Java 允许应用程序通过 Direct ByteBuffer 直接访问堆外内存，许多高性能程序通过 Direct ByteBuffer 结合内存映射文件（Memory Mapped File）实现高速 IO。

### 原因分析

Direct ByteBuffer 的默认大小为 64 MB，一旦使用超出限制，就会抛出 `Directbuffer memory` 错误。

### 解决方案

1、Java 只能通过 ByteBuffer.allocateDirect 方法使用 Direct ByteBuffer，因此，可以通过 Arthas 等在线诊断工具拦截该方法进行排查。

2、检查是否直接或间接使用了 NIO，如 netty，jetty 等。

3、通过启动参数 `-XX:MaxDirectMemorySize` 调整 Direct ByteBuffer 的上限值。

4、检查 JVM 参数是否有 `-XX:+DisableExplicitGC` 选项，如果有就去掉，因为该参数会使 `System.gc()` 失效。

5、检查堆外内存使用代码，确认是否存在内存泄漏；或者通过反射调用 `sun.misc.Cleaner` 的 `clean()` 方法来主动释放被 Direct ByteBuffer 持有的内存空间。

6、内存容量确实不足，升级配置。

# 排查步骤



## JAVA线上内存溢出问题排查步骤

   一般线上遇到比较头疼的就是OOM内存溢出问题，我们都会先看错误日志，如果错误日志能够定位出哪个类对象导致内存溢出，那么我们只需要针对问题修改bug就好。但是很多时候我们单凭日志无法定位出内存溢出问题，那么我们这时候就需要以下操作来定位问题。

1、top下对当前服务器内存有个大致了解

![img](media/abb74bab4dd7af034705d17426a4c61c.png)

top后 shift+M俺内存占用由大到小排序，RES是此进程实际占用内存，%MEM是占服务器总内存的49.8。

2、利用ps命令查看服务pid

```
[root@speedyao java]# ps -aux|grep java
```

 3、利用jstat查看虚拟机gc情况

jstat -gc:util <vmid>  [<interval> [<count>]

vmid：虚拟机进程号

interval:采样时间，默认单位是ms

count：采样条数

```
[root@speedyao java]# jstat -gcutil  17561  1000 10
```

 以上命令代表1秒钟采样1次，总共采样10次。

![img](media/8f8beb62a3086f7be958aaed9a16f777.png)

FULL GC明显大于YOUNG GC次数，并且FULL GC次数很频繁，说明程序有大内存对象，并且一直无法释放。

4、生成dump文件，有两种方式。 一种是利用jmap直接生成dump文件；另一种是利用gcore先生成core文件，再根据core文件利用jmap生成dump文件。

（1）先说第一种，这种比较简单，使用这种方案的时候请注意：JVM会将整个heap的信息dump写入到一个文件，heap如果比较大的话，就会导致这个过程比较耗时，并且执行的过程中为了保证dump的信息是可靠的，所以会暂停应用。

> [root@speedyao java]# jmap -dump:format=b,file=heap.prof 17561 

 format=b：表示生成二进制类型的dump文件

file=：后面写的是输出的dump文件路径

17561：jvm进程id

 

（2）接下来是第二种。这一种在jmap转换core文件的时候比较耗时，并且生成的dump文件用mat打开的时候分析结果不太正确，不太好定位问题。所以我建议使用第一种，虽然会造成服务挂起吧，但是结果总归是正确的。

利用gcore保存服务的内存信息，因为gcore比jmap的dump会快很多，也不对线上服务有大的影响

> [root@speedyao java]# gdb -q  --pid=17561

 ![img](media/5e4d37a2cb67fa3c0273deec174cea6e.png)

 generate-core-file：生成内存对象，生成的文件存储在当前位置，文件格式pid.core

detach：断开与进程的连接

quit：退出

利用jmap将gcore文件转换为java的dump文件，这一步执行的比较慢，可以用nohup执行，以防止误点Ctrl+C导致退出。

> [root@speedyao java]# jmap -dump:format=b,file=heap.prof /usr/bin/java core.17561 

 format=b：表示生成二进制类型的dump文件

file=：后面写的是输出的dump文件路径

/usr/bin/java：java命令路径，可以通过命令which java 获取这个路径

core.17561：表示core文件路径

 

6、利用MAT（eclipse开发的可以下载eclipse插件，idea开发的可以下载单独的MAT压缩包）分析dump文件。当然也可以用jdk自带的jvisualvm.exe来分析dump文件

![img](media/1e988c14bd834410342116f217b3bfb3.png)

上图是概要，阴影部分就是大内存对象类，点击选择 “list Object”、“with incoming references”，就出现下图。下图就是这个对象的信息，RunMian 就是map对象所在的类，这样就能快速定位出哪个类中的哪个对象出现了内存异常。

![img](media/90ad6e02ed162924b18de751b45b0bd0.png)

下图Histogram这个tab是堆内存占比从大到小排序。 

![img](media/d394bab835a9d3f39021785030e70242.png)

以上就是内存问题排查的大致步骤。 

# VisualVM排查

**使用jvisualvm排查一次内存溢出（OOM）过程**



### 内存溢出在开发中或者线上出现的概率很高，造成的直接原因就是系统运行缓慢，或者直接宕机了。

小编在这里模拟下内存溢出的情况以防患于线上出现内存溢出要如何排查问题。题外话（线上出问题你需要生成一个快照（hprof文件），在本地查看问题），当然了还有其他工具调试如阿里的Arthas、还有MAT。我这里只演示jvisualvm。

##### **我使用的jdk版本是jdk1.8.0_05**

先准备一个可以出现内存溢出的代码

![img](media/aHR0cDovL3AxLnBzdGF0cC5jb20vbGFyZ2UvcGdjLWltYWdlL2FlZTE3OTAxYWM0MzQwZmQ5ZWJiODY4NzEzMjMzNGU3)

之后在IDEA中配置VM参数【-Xms2m -Xmx4m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=d:\jvmtest】

![img](media/aHR0cDovL3AzLnBzdGF0cC5jb20vbGFyZ2UvcGdjLWltYWdlLzMzYTRlZTU4NWIwMDQ4MDA5Y2YzYTdhMjExMjEwZDVm)

说明一下：

-Xms 为jvm启动时分配的内存，比如-Xms2m，表示分配2M。

-Xmx 为jvm运行过程中分配的最大内存，比如-Xmx4m，表示jvm进程最多只能够占用4M内存。

-XX:+HeapDumpOnOutOfMemoryError 表示出现OutOfMemoryError异常时，记录快照。

-XX:HeapDumpPath 表示快照的存储位置（这里可以设置文件名字，也可以不设置），不设置名字它会自己生成的。

执行后，会抛出如下异常

![img](media/aHR0cDovL3AxLnBzdGF0cC5jb20vbGFyZ2UvcGdjLWltYWdlLzkzYjViZjg1MDljYjQwZWQ4MmM0MTA3MjYzOWVmOWNi)

看到如下：Heap dump file created [5505387 bytes in 0.042 secs]，那就计算下使用了多少M

![img](media/aHR0cDovL3AxLnBzdGF0cC5jb20vbGFyZ2UvcGdjLWltYWdlLzVmMzI2NTYwNjFmNjQzMTdhY2JiYTQzYjdlMzM1Zjc2)

可以看得出，明显超出了4M。使用jvisualvm.exe来打开生成的hprof文件

![img](media/aHR0cDovL3AzLnBzdGF0cC5jb20vbGFyZ2UvcGdjLWltYWdlLzdlNWUzNWE5YTQzOTQ1OGNhNzQyYjFlNGJjNWM3MTcz)

提示内存溢出了。点击main线程进入

![img](media/aHR0cDovL3AzLnBzdGF0cC5jb20vbGFyZ2UvcGdjLWltYWdlL2YyNjdkNGQyNTE4YzQ3MDI5NjAxNWQwYWQ1NjZiYWI3)

看到了ArrayList，点进去

![img](media/aHR0cDovL3AxLnBzdGF0cC5jb20vbGFyZ2UvcGdjLWltYWdlLzNkMWFhZjdmZWMwODQ1OTliYmQxMTM3N2U2OTg5YjVj)

可以看得到ArrayList的存储大小（33384）。在点击到elementData看看里面存储的什么元素

![img](media/aHR0cDovL3AxLnBzdGF0cC5jb20vbGFyZ2UvcGdjLWltYWdlL2FmMzkwNWMwYmQ3NDRiMmU5NjYxYTU3NGE1ODM2M2Q5)

可以清晰的看得出是TestOOMA$OOMObject这个对象，也就是我上面例子中的对象。另外在类视图中也可以看见这个对象在飙升。

![img](media/aHR0cDovL3AxLnBzdGF0cC5jb20vbGFyZ2UvcGdjLWltYWdlLzVmMjBkN2ViYzJjOTRkZWQ4NTUzMzcwZjZiMDdmMTM0)

也就分析出，内存溢出的原因就是因为疯狂的创建对象造成的





# JAVA 线上故障排查完整套路，从 CPU、磁盘、内存、网络、GC 一条龙！

[后端技术精选](javascript:void(0);) *8月23日*

![img](media/640)

*作者：\*fredal**

*https://fredal.xin/java-error-check*

- CPU
- 磁盘
- 内存
- GC问题
- 网络

线上故障主要会包括cpu、磁盘、内存以及网络问题，而大多数故障可能会包含不止一个层面的问题，所以进行排查时候尽量四个方面依次排查一遍。

同时例如jstack、jmap等工具也是不囿于一个方面的问题的，基本上出问题就是df、free、top 三连，然后依次jstack、jmap伺候，具体问题具体分析即可。

## CPU

一般来讲我们首先会排查cpu方面的问题。cpu异常往往还是比较好定位的。原因包括业务逻辑问题(死循环)、频繁gc以及上下文切换过多。而最常见的往往是业务逻辑(或者框架逻辑)导致的，可以使用jstack来分析对应的堆栈情况。

### 使用jstack分析cpu问题

我们先用ps命令找到对应进程的pid(如果你有好几个目标进程，可以先用top看一下哪个占用比较高)。

接着用top -H -p pid来找到cpu使用率比较高的一些线程

![img](media/640)

然后将占用最高的pid转换为16进制`printf '%x\n' pid`得到nid

![img](media/640)

接着直接在jstack中找到相应的堆栈信息`jstack pid |grep 'nid' -C5 –color`

![img](media/640)

可以看到我们已经找到了nid为0x42的堆栈信息，接着只要仔细分析一番即可。

当然更常见的是我们对整个jstack文件进行分析，通常我们会比较关注WAITING和TIMED_WAITING的部分，BLOCKED就不用说了。我们可以使用命令`cat jstack.log | grep "java.lang.Thread.State" | sort -nr | uniq -c`来对jstack的状态有一个整体的把握，如果WAITING之类的特别多，那么多半是有问题啦。

![img](media/640)

### 频繁gc

当然我们还是会使用jstack来分析问题，但有时候我们可以先确定下gc是不是太频繁，使用jstat -gc pid 1000命令来对gc分代变化情况进行观察，1000表示采样间隔(ms)，S0C/S1C、S0U/S1U、EC/EU、OC/OU、MC/MU分别代表两个Survivor区、Eden区、老年代、元数据区的容量和使用量。YGC/YGT、FGC/FGCT、GCT则代表YoungGc、FullGc的耗时和次数以及总耗时。如果看到gc比较频繁，再针对gc方面做进一步分析。

![img](media/640)

### 上下文切换

针对频繁上下文问题，我们可以使用vmstat命令来进行查看

![img](media/640)

cs(context switch)一列则代表了上下文切换的次数。

如果我们希望对特定的pid进行监控那么可以使用 pidstat -w pid命令，cswch和nvcswch表示自愿及非自愿切换。

![img](media/640)

## 磁盘

磁盘问题和cpu一样是属于比较基础的。首先是磁盘空间方面，我们直接使用df -hl来查看文件系统状态

![img](media/640)

更多时候，磁盘问题还是性能上的问题。我们可以通过iostatiostat -d -k -x来进行分析

![img](media/640)

最后一列%util可以看到每块磁盘写入的程度，而rrqpm/s以及wrqm/s分别表示读写速度，一般就能帮助定位到具体哪块磁盘出现问题了。

另外我们还需要知道是哪个进程在进行读写，一般来说开发自己心里有数，或者用iotop命令来进行定位文件读写的来源。

![img](media/640)

不过这边拿到的是tid，我们要转换成pid，可以通过readlink来找到pidreadlink -f /proc/*/task/tid/../..。

![img](media/640)

找到pid之后就可以看这个进程具体的读写情况cat /proc/pid/io

![img](media/640)

我们还可以通过lsof命令来确定具体的文件读写情况lsof -p pid

![img](media/640)

## 内存

内存问题排查起来相对比CPU麻烦一些，场景也比较多。主要包括OOM、GC问题和堆外内存。一般来讲，我们会先用free命令先来检查一发内存的各种情况。

![img](media/640)

### 堆内内存

内存问题大多还都是堆内内存问题。表象上主要分为OOM和StackOverflow。

#### OOM

JMV中的内存不足，OOM大致可以分为以下几种：

```
Exception in thread "main" java.lang.OutOfMemoryError: unable to create new native thread
```

这个意思是没有足够的内存空间给线程分配java栈，基本上还是线程池代码写的有问题，比如说忘记shutdown，所以说应该首先从代码层面来寻找问题，使用jstack或者jmap。如果一切都正常，JVM方面可以通过指定Xss来减少单个thread stack的大小。

另外也可以在系统层面，可以通过修改/etc/security/limits.confnofile和nproc来增大os对线程的限制

![img](media/640)

```
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
```

这个意思是堆的内存占用已经达到-Xmx设置的最大值，应该是最常见的OOM错误了。解决思路仍然是先应该在代码中找，怀疑存在内存泄漏，通过jstack和jmap去定位问题。如果说一切都正常，才需要通过调整Xmx的值来扩大内存。

```
Caused by: java.lang.OutOfMemoryError: Meta space
```

这个意思是元数据区的内存占用已经达到XX:MaxMetaspaceSize设置的最大值，排查思路和上面的一致，参数方面可以通过XX:MaxPermSize来进行调整(这里就不说1.8以前的永久代了)。

#### Stack Overflow

栈内存溢出，这个大家见到也比较多。

```
Exception in thread "main" java.lang.StackOverflowError
```

表示线程栈需要的内存大于Xss值，同样也是先进行排查，参数方面通过Xss来调整，但调整的太大可能又会引起OOM。

#### 使用JMAP定位代码内存泄漏

上述关于OOM和StackOverflow的代码排查方面，我们一般使用JMAP      jmap -dump:format=b,file=filename pid来导出dump文件

![img](media/640)

通过mat(Eclipse Memory Analysis Tools)导入dump文件进行分析，内存泄漏问题一般我们直接选Leak Suspects即可，mat给出了内存泄漏的建议。另外也可以选择Top Consumers来查看最大对象报告。和线程相关的问题可以选择thread overview进行分析。除此之外就是选择Histogram类概览来自己慢慢分析，大家可以搜搜mat的相关教程。 

![img](media/640)

日常开发中，代码产生内存泄漏是比较常见的事，并且比较隐蔽，需要开发者更加关注细节。比如说每次请求都new对象，导致大量重复创建对象；进行文件流操作但未正确关闭；手动不当触发gc；ByteBuffer缓存分配不合理等都会造成代码OOM。

另一方面，我们可以在启动参数中指定`-XX:+HeapDumpOnOutOfMemoryError`来保存OOM时的dump文件。

搜索Java知音，回复“后端面试”，送你一份面试宝典.pdf

#### gc问题和线程

gc问题除了影响cpu也会影响内存，排查思路也是一致的。一般先使用jstat来查看分代变化情况，比如youngGC或者fullGC次数是不是太多呀；EU、OU等指标增长是不是异常呀等。

线程的话太多而且不被及时gc也会引发oom，大部分就是之前说的unable to create new native thread。除了jstack细细分析dump文件外，我们一般先会看下总体线程，通过pstreee -p pid |wc -l。

![img](media/640)

或者直接通过查看/proc/pid/task的数量即为线程数量。

![img](media/640)

### 堆外内存

如果碰到堆外内存溢出，那可真是太不幸了。首先堆外内存溢出表现就是物理常驻内存增长快，报错的话视使用方式都不确定，如果由于使用Netty导致的，那错误日志里可能会出现OutOfDirectMemoryError错误，如果直接是DirectByteBuffer，那会报`OutOfMemoryError: Direct buffer memory`。

堆外内存溢出往往是和NIO的使用相关，一般我们先通过pmap来查看下进程占用的内存情况pmap -x pid | sort -rn -k3 | head -30，这段意思是查看对应pid倒序前30大的内存段。这边可以再一段时间后再跑一次命令看看内存增长情况，或者和正常机器比较可疑的内存段在哪里。

![img](media/640)

我们如果确定有可疑的内存端，需要通过gdb来分析gdb --batch --pid {pid} -ex "dump memory filename.dump {内存起始地址} {内存起始地址+内存块大小}"

![img](media/640)

获取dump文件后可用heaxdump进行查看hexdump -C filename | less，不过大多数看到的都是二进制乱码。

NMT是Java7U40引入的HotSpot新特性，配合jcmd命令我们就可以看到具体内存组成了。需要在启动参数中加入 `-XX:NativeMemoryTracking=summary` 或者 `-XX:NativeMemoryTracking=detail`，会有略微性能损耗。

一般对于堆外内存缓慢增长直到爆炸的情况来说，可以先设一个基线jcmd pid VM.native_memory baseline。

![img](media/640)

然后等放一段时间后再去看看内存增长的情况，通过jcmd pid VM.native_memory detail.diff(summary.diff)做一下summary或者detail级别的diff。

![img](media/640)

![img](media/640)

可以看到jcmd分析出来的内存十分详细，包括堆内、线程以及gc(所以上述其他内存异常其实都可以用nmt来分析)，这边堆外内存我们重点关注Internal的内存增长，如果增长十分明显的话那就是有问题了。

detail级别的话还会有具体内存段的增长情况，如下图。

![img](media/640)

此外在系统层面，我们还可以使用strace命令来监控内存分配 strace -f -e "brk,mmap,munmap" -p pid

这边内存分配信息主要包括了pid和内存地址。

![img](media/640)

不过其实上面那些操作也很难定位到具体的问题点，关键还是要看错误日志栈，找到可疑的对象，搞清楚它的回收机制，然后去分析对应的对象。比如DirectByteBuffer分配内存的话，是需要full GC或者手动system.gc来进行回收的(所以最好不要使用-XX:+DisableExplicitGC)。

那么其实我们可以跟踪一下DirectByteBuffer对象的内存情况，通过jmap -histo:live pid手动触发fullGC来看看堆外内存有没有被回收。如果被回收了，那么大概率是堆外内存本身分配的太小了，通过-XX:MaxDirectMemorySize进行调整。如果没有什么变化，那就要使用jmap去分析那些不能被gc的对象，以及和DirectByteBuffer之间的引用关系了。

搜索Java知音，回复“后端面试”，送你一份面试宝典.pdf

## GC问题

堆内内存泄漏总是和GC异常相伴。不过GC问题不只是和内存问题相关，还有可能引起CPU负载、网络问题等系列并发症，只是相对来说和内存联系紧密些，所以我们在此单独总结一下GC相关问题。

我们在cpu章介绍了使用jstat来获取当前GC分代变化信息。而更多时候，我们是通过GC日志来排查问题的，在启动参数中加上`-verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps`来开启GC日志。

常见的Young GC、Full GC日志含义在此就不做赘述了。

针对gc日志，我们就能大致推断出youngGC与fullGC是否过于频繁或者耗时过长，从而对症下药。我们下面将对G1垃圾收集器来做分析，这边也建议大家使用G1-XX:+UseG1GC。

### youngGC过频繁

youngGC频繁一般是短周期小对象较多，先考虑是不是Eden区/新生代设置的太小了，看能否通过调整-Xmn、-XX:SurvivorRatio等参数设置来解决问题。如果参数正常，但是young gc频率还是太高，就需要使用Jmap和MAT对dump文件进行进一步排查了。

### youngGC耗时过长

耗时过长问题就要看GC日志里耗时耗在哪一块了。以G1日志为例，可以关注Root Scanning、Object Copy、Ref Proc等阶段。Ref Proc耗时长，就要注意引用相关的对象。

Root Scanning耗时长，就要注意线程数、跨代引用。Object Copy则需要关注对象生存周期。而且耗时分析它需要横向比较，就是和其他项目或者正常时间段的耗时比较。比如说图中的Root Scanning和正常时间段比增长较多，那就是起的线程太多了。

![img](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhq5hUg5NpzQCMvSzUOmdUKlBGLCBlIxyGpXFLqAl0BhyL7mXqva9UajibgN5yKj0tJbnWEDtZkHxiaw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 触发fullGC

G1中更多的还是mixedGC，但mixedGC可以和youngGC思路一样去排查。触发fullGC了一般都会有问题，G1会退化使用Serial收集器来完成垃圾的清理工作，暂停时长达到秒级别，可以说是半跪了。

fullGC的原因可能包括以下这些，以及参数调整方面的一些思路：

- 并发阶段失败：在并发标记阶段，MixGC之前老年代就被填满了，那么这时候G1就会放弃标记周期。这种情况，可能就需要增加堆大小，或者调整并发标记线程数-XX:ConcGCThreads。
- 晋升失败：在GC的时候没有足够的内存供存活/晋升对象使用，所以触发了Full GC。这时候可以通过-XX:G1ReservePercent来增加预留内存百分比，减少-XX:InitiatingHeapOccupancyPercent来提前启动标记，-XX:ConcGCThreads来增加标记线程数也是可以的。
- 大对象分配失败：大对象找不到合适的region空间进行分配，就会进行fullGC，这种情况下可以增大内存或者增大-XX:G1HeapRegionSize。
- 程序主动执行System.gc()：不要随便写就对了。

另外，我们可以在启动参数中配置-XX:HeapDumpPath=/xxx/dump.hprof来dump fullGC相关的文件，并通过jinfo来进行gc前后的dump

```
jinfo -flag +HeapDumpBeforeFullGC pid 
jinfo -flag +HeapDumpAfterFullGC pid
```

这样得到2份dump文件，对比后主要关注被gc掉的问题对象来定位问题。

搜索Java知音，回复“后端面试”，送你一份面试宝典.pdf

## 网络

涉及到网络层面的问题一般都比较复杂，场景多，定位难，成为了大多数开发的噩梦，应该是最复杂的了。这里会举一些例子，并从tcp层、应用层以及工具的使用等方面进行阐述。

### 超时

超时错误大部分处在应用层面，所以这块着重理解概念。超时大体可以分为连接超时和读写超时，某些使用连接池的客户端框架还会存在获取连接超时和空闲连接清理超时。

- 读写超时。readTimeout/writeTimeout，有些框架叫做so_timeout或者socketTimeout，均指的是数据读写超时。注意这边的超时大部分是指逻辑上的超时。soa的超时指的也是读超时。读写超时一般都只针对客户端设置。
- 连接超时。connectionTimeout，客户端通常指与服务端建立连接的最大时间。服务端这边connectionTimeout就有些五花八门了，jetty中表示空闲连接清理时间，tomcat则表示连接维持的最大时间。
- 其他。包括连接获取超时connectionAcquireTimeout和空闲连接清理超时idleConnectionTimeout。多用于使用连接池或队列的客户端或服务端框架。

我们在设置各种超时时间中，需要确认的是尽量保持客户端的超时小于服务端的超时，以保证连接正常结束。

在实际开发中，我们关心最多的应该是接口的读写超时了。

如何设置合理的接口超时是一个问题。如果接口超时设置的过长，那么有可能会过多地占用服务端的tcp连接。而如果接口设置的过短，那么接口超时就会非常频繁。

服务端接口明明rt降低，但客户端仍然一直超时又是另一个问题。这个问题其实很简单，客户端到服务端的链路包括网络传输、排队以及服务处理等，每一个环节都可能是耗时的原因。

### TCP队列溢出

tcp队列溢出是个相对底层的错误，它可能会造成超时、rst等更表层的错误。因此错误也更隐蔽，所以我们单独说一说。

![img](media/640)

如上图所示，这里有两个队列：syns queue(半连接队列）、accept queue（全连接队列）。三次握手，在server收到client的syn后，把消息放到syns queue，回复syn+ack给client，server收到client的ack，如果这时accept queue没满，那就从syns queue拿出暂存的信息放入accept queue中，否则按tcp_abort_on_overflow指示的执行。

tcp_abort_on_overflow 0表示如果三次握手第三步的时候accept queue满了那么server扔掉client发过来的ack。tcp_abort_on_overflow 1则表示第三步的时候如果全连接队列满了，server发送一个rst包给client，表示废掉这个握手过程和这个连接，意味着日志里可能会有很多connection reset / connection reset by peer。

那么在实际开发中，我们怎么能快速定位到tcp队列溢出呢？

netstat命令，执行`netstat -s | egrep "listen|LISTEN"`

![img](https://mmbiz.qpic.cn/mmbiz_jpg/JfTPiahTHJhq5hUg5NpzQCMvSzUOmdUKlMt6nISSp2qKR2j7OdYD3q8KHRWwxJInHMvAn8buicW0CF2tJrNsPiaHw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如上图所示，overflowed表示全连接队列溢出的次数，sockets dropped表示半连接队列溢出的次数。

#### ss命令，执行ss -lnt

![img](media/640)

上面看到Send-Q 表示第三列的listen端口上的全连接队列最大为5，第一列Recv-Q为全连接队列当前使用了多少。

接着我们看看怎么设置全连接、半连接队列大小吧：

全连接队列的大小取决于min(backlog, somaxconn)。backlog是在socket创建的时候传入的，somaxconn是一个os级别的系统参数。而半连接队列的大小取决于max(64, /proc/sys/net/ipv4/tcp_max_syn_backlog)。

在日常开发中，我们往往使用servlet容器作为服务端，所以我们有时候也需要关注容器的连接队列大小。在tomcat中backlog叫做acceptCount，在jetty里面则是acceptQueueSize。

### RST异常

RST包表示连接重置，用于关闭一些无用的连接，通常表示异常关闭，区别于四次挥手。

在实际开发中，我们往往会看到connection reset / connection reset by peer错误，这种情况就是RST包导致的。

#### 端口不存在

如果像不存在的端口发出建立连接SYN请求，那么服务端发现自己并没有这个端口则会直接返回一个RST报文，用于中断连接。

#### 主动代替FIN终止连接

一般来说，正常的连接关闭都是需要通过FIN报文实现，然而我们也可以用RST报文来代替FIN，表示直接终止连接。实际开发中，可设置SO_LINGER数值来控制，这种往往是故意的，来跳过TIMED_WAIT，提供交互效率，不闲就慎用。

**客户端或服务端有一边发生了异常，该方向对端发送RST以告知关闭连接**

我们上面讲的tcp队列溢出发送RST包其实也是属于这一种。这种往往是由于某些原因，一方无法再能正常处理请求连接了(比如程序崩了，队列满了)，从而告知另一方关闭连接。

**接收到的TCP报文不在已知的TCP连接内**

比如，一方机器由于网络实在太差TCP报文失踪了，另一方关闭了该连接，然后过了许久收到了之前失踪的TCP报文，但由于对应的TCP连接已不存在，那么会直接发一个RST包以便开启新的连接。

**一方长期未收到另一方的确认报文，在一定时间或重传次数后发出RST报文**

这种大多也和网络环境相关了，网络环境差可能会导致更多的RST报文。

之前说过RST报文多会导致程序报错，在一个已关闭的连接上读操作会报connection reset，而在一个已关闭的连接上写操作则会报connection reset by peer。通常我们可能还会看到broken pipe错误，这是管道层面的错误，表示对已关闭的管道进行读写，往往是在收到RST，报出connection reset错后继续读写数据报的错，这个在glibc源码注释中也有介绍。

我们在排查故障时候怎么确定有RST包的存在呢？当然是使用tcpdump命令进行抓包，并使用wireshark进行简单分析了。tcpdump -i en0 tcp -w xxx.cap，en0表示监听的网卡。

![img](media/640)

接下来我们通过wireshark打开抓到的包，可能就能看到如下图所示，红色的就表示RST包了。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/JfTPiahTHJhq5hUg5NpzQCMvSzUOmdUKl5r2Dwp6KLQlD021x8zOOn1XvAr3QcZNbbiaPJ44nuOk4ch6vYmZM4ibQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### TIME_WAIT和CLOSE_WAIT

TIME_WAIT和CLOSE_WAIT是啥意思相信大家都知道。

在线上时，我们可以直接用命令`netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'`来查看time-wait和close_wait的数量

用ss命令会更快`ss -ant | awk '{++S[$1]} END {for(a in S) print a, S[a]}'`

### ![img](media/640)

### TIME_WAIT

time_wait的存在一是为了丢失的数据包被后面连接复用，二是为了在2MSL的时间范围内正常关闭连接。它的存在其实会大大减少RST包的出现。

过多的time_wait在短连接频繁的场景比较容易出现。这种情况可以在服务端做一些内核参数调优:

```
#表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭
net.ipv4.tcp_tw_reuse = 1
#表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭
net.ipv4.tcp_tw_recycle = 1
```

当然我们不要忘记在NAT环境下因为时间戳错乱导致数据包被拒绝的坑了，另外的办法就是改小tcp_max_tw_buckets，超过这个数的time_wait都会被干掉，不过这也会导致报time wait bucket table overflow的错。

### CLOSE_WAIT

close_wait往往都是因为应用程序写的有问题，没有在ACK后再次发起FIN报文。close_wait出现的概率甚至比time_wait要更高，后果也更严重。往往是由于某个地方阻塞住了，没有正常关闭连接，从而渐渐地消耗完所有的线程。

想要定位这类问题，最好是通过jstack来分析线程堆栈来排查问题，具体可参考上述章节。这里仅举一个例子。

开发同学说应用上线后CLOSE_WAIT就一直增多，直到挂掉为止，jstack后找到比较可疑的堆栈是大部分线程都卡在了countdownlatch.await方法，找开发同学了解后得知使用了多线程但是确没有catch异常，修改后发现异常仅仅是最简单的升级sdk后常出现的class not found。