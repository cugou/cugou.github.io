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

1. IDA6.6

so里面的函数有的是ARM模式，有的是thumb模式。
so的大多数函数都使用了代码混淆。混淆的大概模式是伪循环+多级跳转，跳转的判断条件都是一些很大的数，一会正一会儿负。这样打乱了原有的代码执行顺序，不但增加了逆向的难度，对使用IDA6.6调试也造成很大影响。
对IDA6.6的影响：F8的时候出现“illegle instruction”错误，调试无法继续。
解决方法：

`
		a. 尽量不要在BL或BLX的地方F8，pc+2条指令的后面下端，然后F9。
		b. 修改PC指向下一条指令，然后手动修改上一条指令对内存或寄存器的改动，然后F8并忽略异常。
		c. 升级IDA，希望下个版本没有这个问题。（坐等土豪放出高版本啊）
`
2. 010Editor
dex文件进行过处理，除了method_code_off指向文件size的**后方**以外，还在Annotation_off上做了手脚，指向了错误的地方，导致010Editor解析错误。当然这个比较好修复，直接改成0就行了。
使用010Editor的作用实际上是查看method_off字段指向哪里，然后可以根据这个值dump相应的内存，修复dex文件。Annotation_off上的手脚只是让010Editor遇到错误停止解析，给查看method_code_off制造一些障碍。

3. dex2jar
这个就比较常规了，还原的dex文件中包含`testdex2jarcrash`函数，并且关键函数都会首先调用这个函数，这样会使dex2jar反编译出错。
解决方法：
`
		在baksmali出来的所有.smali文件中删除`testdex2jarcrash`函数及其调用
		使用其他反编译工具。
`

# 最快脱壳方法

### 步骤一
运行apk，DDMS中观察调试信息（发现ali的apk题目都有很友善的debug_log输出，只是每次都是分析完了回头看才发现，实际上ali在最关键的地方都告诉你了，你要做的就是相信他呵呵）：

![图](/public/img/2015-03-11_android_debug_log.png)

在我的模拟器上，地址47ff2000一定是个关键的地方。下面就是如何查看47ff2000地址处的内容。
1. IDA调试获取。显然这个需要绕过反调试，能正常在调试器中运行程序才行。
2. 直接运行apk，然后在`/proc/pid/task`选择一个线程attach，当然这个线程要没有调用ptrace，TRACE_ME才行。这样就是可以dump整个内存，然后进行离线分析。（这个方法要感谢老白）

### 步骤
找到47ff2000处，发现这个地址开头的是一个odex文件。可以直接dump其中的dex部分，当然也可以dump整个odex，但baksmali和smali在处理odex的时候需要模拟器/system/framework中的内容，比较麻烦一些。

010Editor分析dump出来的dex，发现中断在某个Annotation的分析上，可以将Annotation_off改成0，继续分析，会中断在`method_code_off`上，错误是超出了文件的size。看其值，发现是超出了文件的末尾（不是负值，否则要往上找，那就需要像evilapk400那样手动修补了）。

根据`method_code_off`的提示，重新dump dex，从dex_header开始dump到+0x6000处。
重新分析文件结构，发现正常了。

### 步骤三
baksmali+smali，然后dexdecompile反编译。虽然变量类型会有错误，但程序结构是正确的。
java层的分析见博客的另一篇[java静态分析](http://cugou.github.io/2015/03/11/msc-2015_android_No3_java_static_analysis.html)的文章。

如果这里使用dex2jar,参照前面的“吐槽”里面的方法处理一下。


# 壳分析
