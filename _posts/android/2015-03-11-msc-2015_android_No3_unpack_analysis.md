---
layout: post
title: MSC-2015移动安全挑战赛 第三题 壳分析
category: android
tags: reverse
keywords: 
description: 
---

# 先吐槽一下这个壳：
如果你使用IDA6.6、010Editor、dex2jar等工具，那么这个壳可以说是量身打造的。

1、 IDA6.6

so里面的函数有的是ARM模式，有的是thumb模式。
so的大多数函数都使用了代码混淆。混淆的大概模式是伪循环+多级跳转，跳转的判断条件都是一些很大的数，一会正一会儿负。这样打乱了原有的代码执行顺序，不但增加了逆向的难度，对使用IDA6.6调试也造成很大影响。
对IDA6.6的影响：F8的时候出现“illegle instruction”错误，调试无法继续。
解决方法：


	a. 尽量不要在BL或BLX的地方F8，pc+2条指令的后面下端，然后F9。
	b. 修改PC指向下一条指令，然后手动修改上一条指令对内存或寄存器的改动，然后F8并忽略异常。
	c. 升级IDA，希望下个版本没有这个问题。（坐等土豪放出高版本啊）


2、 010Editor

dex文件进行过处理，除了method_code_off指向文件size的**后方**以外，还在Annotation_off上做了手脚，指向了错误的地方，导致010Editor解析错误。当然这个比较好修复，直接改成0就行了。
使用010Editor的作用实际上是查看method_off字段指向哪里，然后可以根据这个值dump相应的内存，修复dex文件。Annotation_off上的手脚只是让010Editor遇到错误停止解析，给查看method_code_off制造一些障碍。

3、 dex2jar

这个就比较常规了，还原的dex文件中包含`testdex2jarcrash`函数，并且关键函数都会首先调用这个函数，这样会使dex2jar反编译出错。
解决方法：


	在baksmali出来的所有.smali文件中删除`testdex2jarcrash`函数及其调用
	使用其他反编译工具。


# 最快脱壳方法

### 步骤一
运行apk，DDMS中观察调试信息（发现ali的apk题目都有很友善的debug_log输出，只是每次都是分析完了回头看才发现，实际上ali在最关键的地方都告诉你了，你要做的就是相信他呵呵）：

![图](/public/img/2015-03-11_android_debug_log.png)

在我的模拟器上，地址47ff2000一定是个关键的地方。下面就是如何查看47ff2000地址处的内容。
1. IDA调试获取。显然这个需要绕过反调试，能正常在调试器中运行程序才行。
2. 直接运行apk，然后在`/proc/pid/task`选择一个线程attach，当然这个线程要没有调用ptrace，TRACE_ME才行。这样就是可以dump整个内存，然后进行离线分析。（这个方法要感谢老白）

### 步骤二
找到47ff2000处，发现这个地址开头的是一个odex文件。可以直接dump其中的dex部分，当然也可以dump整个odex，但baksmali和smali在处理odex的时候需要模拟器/system/framework中的内容，比较麻烦一些。

010Editor分析dump出来的dex，发现中断在某个Annotation的分析上，可以将Annotation_off改成0，继续分析，会中断在`method_code_off`上，错误是超出了文件的size。看其值，发现是超出了文件的末尾（不是负值，否则要往上找，那就需要像evilapk400那样手动修补了）。

根据`method_code_off`的提示，重新dump dex，从dex_header开始dump到+0x6000处。
重新分析文件结构，发现正常了。

### 步骤三
baksmali+smali，然后dexdecompile反编译。虽然变量类型会有错误，但程序结构是正确的。
java层的分析见博客的另一篇[java静态分析](http://cugou.github.io/2015/03/11/msc-2015_android_No3_java_static_analysis.html)的文章。

如果这里使用dex2jar,参照前面的“吐槽”里面的方法处理一下。


# 壳分析

就我的菜鸟水平来说，这个壳还是比较夸张的。主要进行了以下几个方面的加强：

	1. 所有字符串都是加密的，运行时解密。
	2. 对系统函数调用进行了封装，多了一个中间层。重建的C函数调用表顺序是乱的，打散存放在.bss段，而且还有冗余。JNI native函数也是动态获取
	3. 对C++类和函数进行了隐藏处理，IDA不能自动识别，但个别具有构造函数特征的函数还是可以看出是一个类函数。
	4. 大部分功能函数进行了混淆，混淆算法还算简单，用一系列大数进行比较和交叉跳转来混淆原来的程序执行流程。
	5. 关键解密dex文件的流程放到了线程中。
	6. 主线程和关键线程都使用了ptrace进行反调试。并会创建专门的反调试线程，监视进程是否被调试。
	7. 关键的函数调用链中间使用了.data.rel.ro中调用表，所以静态分析的函数的`xref to`很难找到源头。
	
### 一、绕过反调试

可以在JNI_onLoad上下断，F8到`111BC:		BL sub_86A28`，发现一旦执行这个函数就会出错。进去看发现了`ptrace`调用。（进入没有隐藏ptrace，而是直接使用了导入表）

这里可以在libc.so的`ptrace`上下断，结合静态分析对`ptrace`交叉引用，大致判断一下哪些地方存在反调试。（因为ptrace没有隐藏，所以可以使用这种方法）

当然我这里直接`PC+4`跳过了这个函数（主要是简单撸了一下sub_86A28，除了ptrace调用和创建反调试监视线程，没有特别的地方），运气很好（因为反调试还存在于后续的某个关键线程中，只是这个线程没有被IDA附加，可以正常运行），程序可以正常运行起来。

到此，可以结合前面讲到的快速脱壳，查看和dump出相应内存的dex文件了。

### 二、C函数的调用表（中间层）分析

libc.so里的函数调用会提供很多程序功能的猜测。一般静态分析一下程序调用了哪些C函数，可以大致猜测出程序中某些函数的关键功能，给逆向提供很好的方向指引。（这也是为什么壳要隐藏C函数调用的原因）
当然，不去分析调用表也是可以的，F8到相应调用的时候，IDA会给出调用的C函数名称。只是这种方法太过局部和耗时。

程序是在JNI_onLoad的开始的循环中，调用了10+9个函数来建立这个C函数的调用表的。这里ali还算降低难度的，都放到了“可见”的地方。但这19个函数地址哪里来？ELF里面也存在PE中的TLS，会在main之前调用。很多windows下的pe程序，这里是反调试或者壳入口的最佳地点。

`
------------------------------------------------------------

sub_19108
关注内存地址95ad0开始的区域
解码函数字符串unlink，dword_95ad4 = dlsym(-1, "unlink");

------------------------------------------------------------

sub_19e88
dword_95b00 = dlsym(-1, "perror")

-------------------------------------------------------------

sub_1f920
dword_95b28 = dlsym(-1, "mprotect")
dword_95b2c = dlsym(-1, "fopen")
dword_95b30 = dlsym(-1, "fgetln")
dword_95b34 = dlsym(-1, "sscanf")
dword_95b38 = dlsym(-1, "free")

--------------------------------------------------------------

sub_201e8
dword_95fd0 = dlsym(-1, "dlsym")
dword_95fd4 = dlsym(-1, "access")
dword_95fd8 = dlsym(-1, "system")

--------------------------------------------------------------

sub_207d0
dword_960c8 = dlsym(-1, "access")
dword_960cc = dlsym(-1, "open")
dword_960d0 = dlsym(-1, "sprintf")
dword_960d4 = dlsym(-1, "system")
dword_960d8 = dlsym(-1, "close")

---------------------------------------------------------------

sub_20c2c
dword_96170 = dlsym(-1, "free")
dword_96174 = dlsym(-1, "strdup")
dword_96178 = dlsym(-1, "pthread_create")
dword_9617c = dlsym(-1, "pthread_join")

----------------------------------------------------------------

sub_226f8
dword_961c4 = dlsym(-1, "snprintf")
dword_961c8 = dlsym(-1, "fopen")
dword_961cc = dlsym(-1, "fgets")
dword_961d0 = dlsym(-1, "strstr")
dword_961d4 = dlsym(-1, "strtok")
dword_961d8 = dlsym(-1, "fclose")

----------------------------------------------------------------

sub_23504
dword_96268 = dlsym(-1, "fopen")
dword_9626c = dlsym(-1, "fgets")
dword_96270 = dlsym(-1, "strstr")
dword_96274 = dlsym(-1, "sscanf")
dword_96278 = dlsym(-1, "fclose")
dword_9627c = dlsym(-1, "sprintf")
dword_96280 = dlsym(-1, "free")
dword_96284 = dlsym(-1, "pthread_create")

-----------------------------------------------------------------

sub_32d08
dword_9634c = dlsym(-1, "sscanf")
dword_96350 = dlsym(-1, "mprotect")
dword_96354 = dlsym(-1, "mmap")
dword_96358 = dlsym(-1, "munmap")
dword_9635c = dlsym(-1, "free")

------------------------------------------------------------------
第二组9个函数
------------------------------------------------------------------

sub_343e4
dword_96400 = dlsym(-1, "write")

--------------------------------------------------------------------

sub_351cc
dword_96470 = dlsym(-1, "open")
dword_96474 = dlsym(-1, "write")
dword_96478 = dlsym(-1, "close")
dword_9647c = dlsym(-1, "read")

--------------------------------------------------------------------

sub_37500
dword_965a8 = dlsym(-1, "pthread_create")
dword_965ac = dlsym(-1, "read")

--------------------------------------------------------------------

sub_398e0
dword_965f0 = dlsym(-1, "strdup")
dword_965f4 = dlsym(-1, "munmap")
dword_965f8 = dlsym(-1, "free")
dword_965fc = dlsym(-1, "access")
dword_96600 = dlsym(-1, "open")
dword_96604 = dlsym(-1, "fstat")
dword_96608 = dlsym(-1, "mmap")
dword_9660c = dlsym(-1, "close")
dword_96610 = dlsym(-1, "write")

-------------------------------------------------------------------

sub_3aa20
dword_966a0 = dlsym(-1, "calloc")
dword_966a4 = dlsym(-1, "free")
dword_966a8 = dlsym(-1, "write")

--------------------------------------------------------------------

sub_3bba4
dword_966fc = dlsym(-1, "calloc")
dword_96700 = dlsym(-1, "crc32")
dword_96704 = dlsym(-1, "free")

--------------------------------------------------------------------

sub_3d8c8
dword_9679c = dlsym(-1, "gettimeofday")
dword_967a0 = dlsym(-1, "access")

--------------------------------------------------------------------

sub_485cc
dword_96808 = dlsym(-1, "strcmp")
dword_9680c = dlsym(-1, "free")
dword_96810 = dlsym(-1, "inflateInit2_")
dword_96814 = dlsym(-1, "crc32")
dword_96818 = dlsym(-1, "inflate")
dword_9681c = dlsym(-1, "inflateEnd")

---------------------------------------------------------------------

sub_491bc
dword_96898 = dlsym(-1, "fopen")
dword_9689c = dlsym(-1, "fread")
dword_968a0 = dlsym(-1, "fwrite")
dword_968a4 = dlsym(-1, "ftell")
dword_968a8 = dlsym(-1, "fseek")
dword_968ac = dlsym(-1, "fclose")

---------------------------------------------------------------------
`

### 三、寻找attachBaseContext

反编译SubApplication，可以看到有attachBaseContext函数的声明，显然做过evilapk400的就知道这个函数的作用。找这个很简单，JNI_onLoad中sub_86A38这个函数里面调用了RegisterNatives，注册了函数的对应关系。attachBaseContext对应sub_11B18。

### 四、attachBaseContext分析

一个大函数，进行了代码混淆，这些主要增加的分析的时间和工作量。关键函数在`sub_86A68`。从attachBaseContext开始到pcreate_thread的调用流程：

`
attachBaseContext -> sub_86A68 -> sub_86A6C -> sub_1BC68 -> 1A8E8 ->(这一步是通过 .data.rel.ro:934c8 表间接调用的，有可能是C++类的vtable) sub_20374 -> sub_1A864 -> sub_86C58 -> sub_20B98 -> pcreate_thread
`

线程调用参数和同步：

```
//param_95AB4 -> /data/data/crackme.a3/lib/libmobisecz.so
pcreate_thread(NULL, 0, sub_20AE5, param_95AB4, 959A0, threadid_95AB0)
pthread_join (*threadid_95AB0)
```

线程是带参数启动的，libmobisecz.so就是解密dex的关键文件了。
因为时间关系，怎么解密的就不再详细分析了。要完美还原出C++的类来是个难点，以我的菜鸟水平还是不能胜任的。通过分析，掌握了几个以前不知道的知识点，这就够了。呵呵

[题目及分析下载](/public/download/2015-03-11-msc-2015_android_AliCrackme_3.rar)