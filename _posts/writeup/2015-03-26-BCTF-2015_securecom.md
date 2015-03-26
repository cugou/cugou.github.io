---
layout: post
title: BCTF-2015 Securecom
category: reverse
tags: ctf
keywords: 
description: 
---

一开始因为程序不能在我的虚拟机里运行，所以不能动态调试，去观察pUnk到底的虚函数表内容，静态分析pUnk都是0，所以...
在sclient中发现有lpVtbl的调用，直接在sserver中搜索字符，找到以下代码.


```asm
.text:0000000140001000 ; =============== S U B R O U T I N E =======================================
.text:0000000140001000
.text:0000000140001000
.text:0000000140001000 sub_140001000   proc near               ; DATA XREF: .rdata:0000000140002160o
.text:0000000140001000                                         ; .pdata:ExceptionDiro
.text:0000000140001000                 sub     rsp, 28h
.text:0000000140001004                 lea     rcx, pUnk
.text:000000014000100B                 call    sub_140001030
.text:0000000140001010                 lea     rax, off_1400021F0
.text:0000000140001017                 mov     cs:pUnk.lpVtbl, rax
.text:000000014000101E                 add     rsp, 28h
.text:0000000140001022                 retn
.text:0000000140001022 sub_140001000   endp
```

发现IUnkown结构的lpVtbl内容:

```asm
.rdata:00000001400021F0 off_1400021F0   dq offset sub_140001130 ;QueryInterface
.rdata:00000001400021F8                 dq offset sub_140001040 ;AddRef
.rdata:0000000140002200                 dq offset sub_140001040 ;Release
.rdata:0000000140002208                 dq offset sub_140001090
.rdata:0000000140002210                 dq offset sub_140001290
.rdata:0000000140002218                 dq offset unk_140002588
```

参考微软的说明：[IUnknown interface](https://msdn.microsoft.com/en-us/library/ms680509\(v=vs.85\).aspx)  [INTERFACE IUnknown 的定义](http://web.mit.edu/cygwin/OldFiles/cygwin_v1.3.2/usr/include/w32api/unknwn.h)

根据定义IUnkown的vTable最开始的三个函数是
```c
#define INTERFACE IUnknown
DECLARE_INTERFACE(IUnknown)
{
	STDMETHOD(QueryInterface)(THIS_ REFIID,PVOID*) PURE;
	STDMETHOD_(ULONG,AddRef)(THIS) PURE;
	STDMETHOD_(ULONG,Release)(THIS) PURE;
};
```

```
基础知识：
win_x64参数传递顺序： RCX、RDX、R8 和 R9 中(按从左至右的顺序)
```

转了一圈由于对COM不熟悉，所以静态分析基本摸不到头脑（最后调试了才发现，这种迷茫实际上是IDA的F5带来的）。没办法必须调试，这里需要解决两个问题：
1、程序运行的问题：在我的win7虚拟机里运行所缺少msvcr120.dll，下载了该dll又出现0xc000007b的错误。此处一顿折腾，什么directx修复...最后发现使用360软件管家安装“编程开发--微软常用运行库合集2015”，程序就能运行了。这坑爹的微软啊，就算我装了VS2012，还要补全VS2010的开发库。此处感谢360。
2、64位windows程序的调试问题：OD至今还没有推出64位版本，据说快了。这里要感谢Hitcon2014的冠军队伍0x0的writup推荐的x64dbg。与OD实在是太相似了。

```asm
.rdata:00000001400021F0 off_1400021F0   dq offset sub_140001130 
.rdata:00000001400021F8                 dq offset sub_140001040
.rdata:0000000140002200                 dq offset sub_140001040
.rdata:0000000140002208                 dq offset sub_140001090
.rdata:0000000140002210                 dq offset sub_140001290
.rdata:0000000140002218                 dq offset unk_140002588
.rdata:0000000140002220 off_140002220   dq offset sub_1400011A0 
.rdata:0000000140002228                 dq offset unknown_libname_1 
.rdata:0000000140002230                 dq offset sub_140001240
.rdata:0000000140002238                 dq offset sub_140001210
.rdata:0000000140002240                 dq offset sub_1400012E0
.rdata:0000000140002248                 dq offset sub_1400012A0
.rdata:0000000140002250                 dq offset sub_1400012C0
.rdata:0000000140002258                 dq offset sub_140001060
.rdata:0000000140002260                 dq offset sub_140001290
.rdata:0000000140002268                 dq offset sub_140001290
.rdata:0000000140002270                 dq offset sub_140001290
.rdata:0000000140002278                 dq offset sub_140001290
.rdata:0000000140002280                 dq offset sub_140001290
.rdata:0000000140002288                 dq offset sub_140001290
```

通过调试发现`sub_140001130(QueryInterface)`调用了以`off_140002220`位基址的vTable，很明显上面的这段内存就是`sserver.exe`中定义的继承了IUnknown接口的类的完整vTable。

分析sclient的main函数：

```c
......
.text:000000014000113D loc_14000113D:                          ; CODE XREF: main+12Fj
.text:000000014000113D                 lea     rcx, [rbp+210h+Buffer] ; Buffer
.text:0000000140001141                 call    cs:gets
.text:0000000140001147                 test    rax, rax
.text:000000014000114A                 jz      loc_14000141D
.text:0000000140001150                 mov     ecx, esi        ;这个esi是个trick
.text:0000000140001152                 mov     [rsp+2F0h+Src], 5000h
.text:000000014000115B                 mov     [rsp+2F0h+var_2A8], 4000h
.text:0000000140001164                 mov     r12d, esi		;这个esi是个trick
.text:0000000140001167                 call    cs:CoTaskMemAlloc
.text:000000014000116D                 lea     rdx, [rbp+210h+Buffer] ; Src
.text:0000000140001171                 mov     rcx, rax        ; Dst
.text:0000000140001174                 mov     r8d, esi        ; Size
.text:0000000140001177                 mov     r14, rax
.text:000000014000117A                 call    memcpy
.text:000000014000117F                 lea     rcx, [rsp+2F0h+PerformanceCount] ; lpPerformanceCount
.text:0000000140001184                 call    cs:QueryPerformanceCounter
.text:000000014000118A                 mov     r10, [rdi]
.text:000000014000118D                 xor     r9d, r9d
.text:0000000140001190                 mov     r8d, esi
.text:0000000140001193                 mov     rdx, r14
.text:0000000140001196                 mov     rcx, rdi
.text:0000000140001199                 call    qword ptr [r10+20h] ;这里对应sseerver的off_140002220+0x20
.text:000000014000119D                 lea     rcx, [rsp+2F0h+var_2B8] ; lpPerformanceCount
.text:00000001400011A2                 mov     ebx, eax
.text:00000001400011A4                 call    cs:QueryPerformanceCounter
.text:00000001400011AA                 test    ebx, ebx
.text:00000001400011AC                 js      loc_14000141D
.text:00000001400011B2                 lea     rcx, [rsp+2F0h+Frequency] ; lpFrequency
.text:00000001400011B7                 call    cs:QueryPerformanceFrequency
.text:00000001400011BD                 mov     eax, dword ptr [rsp+2F0h+var_2B8]
.text:00000001400011C1                 movsd   xmm6, cs:qword_140002220
.text:00000001400011C9                 xorps   xmm1, xmm1
.text:00000001400011CC                 xorps   xmm0, xmm0
.text:00000001400011CF                 sub     eax, dword ptr [rsp+2F0h+PerformanceCount]
.text:00000001400011D3                 cvtsi2ss xmm1, rax
.text:00000001400011D8                 mov     eax, dword ptr [rsp+2F0h+Frequency]
.text:00000001400011DC                 cvtsi2ss xmm0, rax
.text:00000001400011E1                 divss   xmm1, xmm0
.text:00000001400011E5                 cvtps2pd xmm1, xmm1
.text:00000001400011E8                 comisd  xmm1, xmm6
.text:00000001400011EC                 ja      loc_14000141D
.text:00000001400011F2                 lea     rcx, [rsp+2F0h+PerformanceCount] ; lpPerformanceCount
.text:00000001400011F7                 call    cs:QueryPerformanceCounter
.text:00000001400011FD                 mov     rax, [rdi]
.text:0000000140001200                 mov     r9, r14
.text:0000000140001203                 mov     edx, 6000h
.text:0000000140001208                 mov     r8d, 3E8h
.text:000000014000120E                 mov     rcx, rdi
.text:0000000140001211                 call    qword ptr [rax+28h] ;这里对应sseerver的off_140002220+0x28
......
```

重要的是搞清楚sclient通过COM接口调用sserver中的函数的对应关系，实际上sclient中的函数调用最终都会被sserver的`QueryInterface`导向到正确的vTable项（偏移量是相同的），也就是调用了正确的函数了。（搞清楚了调用关系，实际上静态分析也能给出答案了）

这里IDA的F5，会解析数据结构，失去了汇编中很明显的偏移量，使得vTable不那么直接，所以静态分析的时候迷失了很久，直到调试看到汇编调用，才恍然大悟。

跟踪Buffer可以发现被传递给了sserver的`sub_1400012E0`：

```c
signed __int64 __fastcall sub_1400012E0(__int64 a1, __int64 a2, int a3, __int64 a4)
{
  signed __int64 result; // rax@3

  if ( a2 || !a3 )
  {
    if ( a3 != 8 || *(_QWORD *)a2 != 4993454379719937875i64 )	;这个值就是sclient的gets获取后传递过来比较的。
    {
      result = 2147500037i64;
    }
    else
    {
      *(_QWORD *)(a1 + 16) = 1i64;
      if ( a4 )
        *(_DWORD *)a4 = 8;
      result = 0i64;
    }
  }
  else
  {
    result = 2147942487i64;
  }
  return result;
}
```

这里有个小trick，比较的是64位的long型，而从sclient的gets传递过来，是一个字符串（当然表述把准确，COM传递的是字节数组，而不是字符串，所以没有\0，长度参数就很重要了），两者比较有个little-ending的问题。

经过解析，`4993454379719937875`十进制大数实际上相当于字符串`SOSIMPLE`。
重新运行sclient，输入`SOSIMPLE`，但还是没有结果。这里存在另一个trick——ESI，代表Buffer的大小，如果不带参数运行sclient，这个值被设置成了6，改成8就行了。（如上面所说，长度参数很重要，和C字符串有\0结尾的习惯很不相同）

`QueryPerformanceCounter`用来反调试，调试的时候一路改改，走到printf就行了。

BCTF{gZ4KVR1GFLH6ujwORT}

[题目下载](/public/download/2015-03-26-bctf-securecom.tar.xz)