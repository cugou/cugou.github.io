---
layout: post
title: SCTF-2014 misc100 writeup（赛后分析）
category: writeup
tags: reverse
keywords: 
description: 
---

![题目](/public/img/2014-12-13-SCTF-2014_misc100_writeup-1.jpg)

下载文件后，file命令看一下是ELF32程序，strings命令发现程序被UPX加壳了。

```bash
upx -d snake-final.exe
```

脱壳后扔到IDA里面分析，主函数发现调用signal注册了若干个回调函数：

![IDA图一](/public/img/2014-12-13-SCTF-2014_misc100_writeup-2.jpg)

特别是几个38h、32h、34h、36h。实际上是定义了4个控制贪吃蛇行动的游戏按键。但显然就这个程序，按对应的键是产生不出对应的signal信号的。（有队伍使用向进程发送对应signal的方式，间接操控贪吃蛇，游戏成功可以获得flag）。经过进一步分析，发现最重要的回调函数是handler，同时在handler中存在"int 3"反调试，正常执行的时候会产生signal==5的信号，这里的处理函数为nullsub_1。实际上就是忽略，所以我们也可以直接 nop 掉 int 3。当然完全按照本文的静态分析方法，而不使用调试，完全可以忽略 int 3。
进一步分析handler，发现是一个极其复杂的函数。没心思看，找找有没有别的入口。
strings窗口发现"Mission Complete"，xref发现这个字符串引用位置sub_80492E0+14C。
分析sub_80492E0：

![IDA图二](/public/img/2014-12-13-SCTF-2014_misc100_writeup-3.jpg)

这个函数确实是判断游戏成功与否的标志，成功的话，还需要判断一个ebx == 3，这里的ebx实际上是函数的传入参数。但问题是，找到成功的标志，但是后面没有跟着输出flag的地方，而仅仅是调用settimer重置了新的信号量。这样程序的执行流又回到handler里面。
根据目前的信息，寻找带参数3调用sub_80492E0的地方，xref只发现两处8049941、8049FF1。实际上这两处的代码是一样的（经过最后的分析实际上这里往后的代码就是输出flag的地方，但我在这里迷失了很久）。比较奇怪的是这两处都不在IDA识别的handle函数的作用域中。原因是IDA识别出了handler函数的Canary保护机制，以此作为handler函数的开始和结束。
进一步分析handler发现：

![IDA图三](/public/img/2014-12-13-SCTF-2014_misc100_writeup-4.jpg)

原来成功吃到30个食物成功后，是通过C++异常处理机制来调用sub_80492E0( 3 )的。
通过上面查找xref的两个地方，随便选取第二个，看到调用代码：

![IDA图四](/public/img/2014-12-13-SCTF-2014_misc100_writeup-5.jpg)

由于sub_80492E0里面没有显示flag的地方，只是重置settimer，那么猜测很有可能flag也是通过异常处理链来打印的。上面的代码发现，调用sub_80492E0后，重新抛出了一个异常。所以继续观察下面的代码：

![IDA图五](/public/img/2014-12-13-SCTF-2014_misc100_writeup-6.jpg)

这里注意804A039处的一个比较，这是在打印flag前的异常处理链中的最后一道门槛。也是调试时候为什么sub_80492E0已经运行到"Mission Complete"，但还是没有出现flag的原因。

![IDA图六](/public/img/2014-12-13-SCTF-2014_misc100_writeup-7.jpg)

注意这里的一个循环，实际上就是printf "[ebp-50+i] xor 2Ah"。(2Ah是ascii的'\*')

```python
import sys
a='\x7f\x1a\x64\7f\x78\x44\x5e\x50\x67\x7d\x4e\x5f'
for i in range(0, len(a)):
    sys.stdout.write(chr(ord(a[i]) ^ ord('*')))
```
得到：U0N-LRntzMWdu
这个还没有完，继续往下看：

![IDA图七](/public/img/2014-12-13-SCTF-2014_misc100_writeup-8.jpg)

可见flag还有两部分，其中[ebp-84h]就是handler函数开始处的值（说明异常处理实际上还是应该在handler当中的，只是IDA分析的使用因为canary的原因，被排除了）：

![IDA图八](/public/img/2014-12-13-SCTF-2014_misc100_writeup-9.jpg)

最后一部分的数据是804C0C0处的全局数据，三部分的算法是一样的，最后解码得到：
U0NURntzMWduNGxfMXNfZnVubnk6KX0=

```bash
echo 
U0NURntzMWduNGxfMXNfZnVubnk6KX0= | base64 -d
```
得到flag。


后记：
此题看360首发的sctf writeup上，只讲了"Mission Complete"和sub_80492E0，然后就直接base64字符串了，不知道那位大神是如何“看”出来的。
C++的throw--catch的逆向分析是头一回遇到，还是应该写一个简单的C++相应程序，然后逆向研究一下它的结构。这样才知道handler里面canary和异常处理是人为的有意为之，还是编译器本来就是这样处理的。
由于信号量的处理有线程再入的特点，异常也会涉及到跨线程的问题，所以程序流程要完全研究透彻，需要理解的内容非常多。有待进一步学习。
这个题目如果调试，如果跟踪到异常处理的代码，还请大牛不吝赐教。