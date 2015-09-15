---
layout: post
title: 看雪“玩命”大牛2011年的虚拟机题————我的第一个VM保护的破解学习
category: reverse
tags: VM
keywords: 
description: 
---

## 一、提要

### 软件保护的发展目前来说就是虚拟机保护，但网上系统的资料非常少，看雪搜索了一下，发现“玩命”大牛2011年的crackme。于是尝试破解一下。

### 破解过程发现“玩命”大牛的虚拟机特别有意思，由于加了很强的花指令代码混淆，分析指令解析函数表实在太过困难，回头准备好好把大神的文档和代码学习一下。

## 二、快速分析

实际上这个crackme要找到一组用户名对应的序列号还是非常容易的。IDA里面挂脚本对所有call调用hook跑一遍：`username = root`，`serial = 123456`，搜索`123456`发现如下结果。

```
Function call: 0x40480e to sub_40460A (0x40460a)
EAX: 0x00419888 (".3A..............1A.............................................(*<.....(.A.H.A. ........................2A..-<...........A...A...........A...A......")
EBX: 0xf7fffeff ("N/A")
ECX: 0x0012fe38 ("DBEFFCFF-CFCFBDF-F7FFFEFF-78FCFFDF")
EDX: 0x0012fd38 ("123456")
ESI: 0xdbeffcff ("N/A")
EDI: 0x0cfcfbdf ("N/A")
EBP: 0x0012fd18 -> 0x0012ff40 -> 0x0012ff88 -> 0x0012ffc0 ("....gp.|..............T......(.........|pp.|..............C.....Actx ........$.......... ..................")
ESP: 0x0012fd0c -> 0x0012fd38 ("123456")
EIP: 0x0040480e ("........]...U..].......U...}..t-.u.j..54.A.....H...u.V..Q........H.P.{Q..Y..^]...U..].......U........u...a..Y..t..u...6..Y..t.......A.....A...1A.u,..")
EFL: 0x00000296 ("N/A")
arg_00: 0x0012fd38 ("123456")
arg_04: 0x0012fe38 ("DBEFFCFF-CFCFBDF-F7FFFEFF-78FCFFDF")
arg_08: 0x00000000 ("N/A")
```
但“玩命”大神要求写出注册机，这必然要找到序列号算法。

## 三、分析过程

一开始分析入口点`start`函数，结果是熟悉了一下TEB、PEB、DLL loader、基址重定位等windows概念。当然我只是逆向业余爱好者，数据结构都要现查，浪费了不少时间。
这里需要说明的是用直接内存拷贝的方法把`kernel32.dll`的内存映像复制到堆上，在XP下面是可以的，但是win7下，由于各个节之间的空隙没有分配物理内存，所以连续拷贝内存映像会出现内存访问错误。导致程序在win7下不能正常运行。

粗看代码结构，大量的未识别函数，复杂的函数逻辑，十分变态。IDA跑了几遍都没有头绪。从输入点`ReadFile`函数进行栈回溯也会迷失掉。最后只能从头开始一层一层call去找输入点。直到发现在调用过程中，突然出现了栈空间（ESP）的较大变化。此时豁然开朗————难道这个所谓的虚拟机实际上类似沙盒，让受保护的程序在自己的空间里面运行。要实现这样的栈空间（程序空间）的切换，在实现上我的理解是只要进行`Context`上下文的切换就可以了（类似CPU对线程和进程的切换）。当然需要处理地址重定位和IAT，实现上也会比较复杂。

根据这个逻辑，很快找到

```
sub_421610 从虚拟机调用原程序函数
	0x421848	retn	bp condition: Dword(esp) == 0x401230
sub_421849 回到虚拟机
```

没细看这两个函数，函数存在大量的栈数据的交换。这里可以看到栈空间切换后，是直接`retn`到目标函数的。这就让栈回溯找不到北了。在`sub_421610`函数的结尾`0x421848 retn`处下断点，导出所有的调用：

```
1: 404df0
2: 404df0
3: 401420		cout << 要求:写出注册机
4: 401ff0
5: 402aeb
6: 401420		cout << input username
7: 404df0
8: 401230		cin >> 0x12FD38		username
	这里还有一步计算crc32的trick，没有在原来的代码中，极有可能在虚拟机本身中计算。如果是这样，那么和作者说的main函数都是在虚拟机中运行矛盾啊。
9: 402380
10: 402380
11: 402380
12: 402380
13: 4044ff		sprintf( sNum, "%4X-%4X-%4X-%4X", .....)
14: 401420		cout << input serial number
15: 404df0
16: 401230		cin >> 0x12FD38		serial number
17: 404801		strcmp
18: 401420		cout << sucess | failed
19: 401ff0
20: 402aeb
21: 404329
```

基本上原来的程序逻辑一目了然了。但跟踪`sub_402380`第一次调用的时候，发现其参数竟然不是指向`root`字符串的指针，而是`0x16f4f95b`。由于对crackme.exe使用krypton工具，发现存在`crc32_table`。大胆猜测这个是一个字符串的`crc32`值。验证：

```python
>>> import binascii
>>> hex(binascii.crc32('root'))
'0x16f4f95b'
```

后面三次调用`sub_402380`的参数实际上就是前一次调用的返回值。这样原来程序的流程和算法也就基本明了了。
这里存在一个问题就是输入的用户名`root`是如何变成`0x16f4f95b`的。这里难道是作者留的小trick？

## 四、寻找丢失的CRC32函数

既然CRC32这个函数不是通过`retn`栈空间切换后调用的，那么只有可能是在虚拟机里面执行完后写回到原程序的栈空间中。这里就涉及到两个栈空间的数据交换。原程序栈空间中取出`root`，计算出CRC32值后写回原来的栈空间，作为`sub_402380`的参数。
想着是简单，实际上找到这个过程非常复杂。因为需要解构虚拟机的指令集函数。

寻找的过程非常坑啊，确实就差对整个`crc32_table`下内存断点了。回头想想，既然明确知道算法，从栈地址的内存断点又容易混乱，还是要直接对`crc32_table`下内存断点的。因为后来也是在调试中，发现一个call调用的参数是`crc32_table`中的一个值，才确定一个`xor`指令的模拟函数。

```
VPOPIWCM:0042D6C3     mov     ecx, [edi]
VPOPIWCM:0042D6C5     push    esi
VPOPIWCM:0042D6C6     call    sub_424540
VPOPIWCM:0042D6CB     mov     [edi], eax	eax = 4, edi = 0x15633c	返回用户名的长度
```
xor指令

![pic1](/public/img/2015-09-15-vm_1.png)

or指令

![pic2](/public/img/2015-09-15-vm_2.png)

循环控制

![pic3](/public/img/2015-09-15-vm_3.png)

如果跟入对应的函数，可以看见，一个简单的指令模拟，竟然加了那么多复杂的逻辑跳转和花指令。十分变态。我是对照CRC32的源代码找的。不然以我的功力，根本找不到。

```c
static u_int32 calculate_CRC32 (void *pStart, u_int32 uSize)
{
#define INIT  0xffffffff
#define XOROT 0xffffffff

  u_int32 uCRCValue;
  u_int8 *pData;

  /* init the start value */
  uCRCValue = INIT;
  pData = pStart;

  /* calculate CRC */
  while (uSize --)
  {
	// 这个代码转成相应的汇编逻辑，去虚拟机中找对应的指令模拟。
    uCRCValue = CRC32_Table[(uCRCValue ^ *pData++) & 0xFF] ^ (uCRCValue >> 8);
  }
  /* XOR the output value */
  return uCRCValue ^ XOROT;
}
```

[题目下载](/public/download/2015-09-15-crackme.zip)