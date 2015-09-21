---
layout: post
title: 2015-09-21-CSAW-CTF2015-Forensics150-pcapin writeup
category: writeup
tags: Forensics
keywords: 
description: 
---

这题就一个pcapin.pcap包，打开来只有一个完整的tcp 7179服务端口的会话：

![pic](/public/img/2015-09-21-pcapin-1.PNG)

根据题目的提示

```
We have extracted a pcap file from a network where attackers were present. 
We know they were using some kind of file transfer protocol on TCP port 7179. 
We're not sure what file or files were transferred and we need you to investigate.
We do not believe any strong cryptography was employed.

Hint: The file you are looking for is a png
```

很容易知道，第一个请求应该是类似列目录，第二个应该是从返回的文件列表中选择一个下载。根据这个思路，搜索第二个请求和第一个应答的相同部分`8f 95 88 9e c7 89 87 9e`。很显然是一个文件名，密钥的话那么多重复出现的`e9 f9`，试试知道下载的文件名是`flag.png`。
那么第二个那么一长串的返回数据就是这个文件的内容了。
PNG文件格式的标识：
1、文件开始字节流：`89 50 4E 47 0D 0A 1A 0A`
2、文件结束字节流：`00 00 00 00 49 45 4E 44 AE 42 60 82`

第二部分内容`00 d4 xx xx 07 32 00 1c`这种内容重复出现，而下一次出现的位置正好间隔`d4`个字节。每一块这8个字节后的4个字节，很明显是块序号了。这里走了点弯路，以为会是bt协议，块号后面跟着20个字节的sha1。实际上没那么复杂。
根据这些信息可以定义数据传输块的头部结构(需要注意的是网络字节顺序和主机字节顺序的区别）：

```c
struct block_head {
	unsigned short size;
	unsigned short key;		
	unsigned int magic;
	unsigned short serial;
	byte pad[2];
}
```

这里实际上给出了头部的完整结构。还有两个地方需要说明一下：第二个`key`怎么来的；块头部结构的长度为什么正好是那么长。
要解释这个问题就只能尝试。前面PNG文件格式可以用来识别的就是文件开头和结尾的固定字节流。那么重点查看00号块和最后一个1b号块。

![pic](/public/img/2015-09-21-pcapin-2.PNG)

图中看到最后一个块很多`4c 6c`，用这个来对最后一块进行`xor`，在中间偏后的位置找到PNG文件的结束标志流。
现在回到第一个块，这里花费了很多时间进行尝试，最后发现里面的`50 3f`有点意思。`xor`后找到了PNG开头的固定标识字节流。从而可以确定数据块控制头结构的长度。

剩下的问题就是对块进行解密的密钥在哪里。很明显，每个块的加密密钥是不同的。这里只有到每个块头部不断变化的两个字节考察了。猜了许久，突然脑洞打开。

```
key[0] = xx[1] - 0x17
key[1] = xx[0] -6
```

这里之所以交叉，还是网络字节顺序和主机字节顺序相反导致的。

将第二个应答的字节dump出来保存成`a.png`。解密代码如下：

```python
o = open('b.png', 'wb')
f = open('a.png', 'rb')
a = f.read()
b = []
for i in range(0, (5939/0xd4)):
	b.append(a[i*0xd4:(i+1)*0xd4:])

#print len(b)

for i in range(0, len(b)-1):
	key[0] = ord(b[i][3]) - 0x17
	if key[0] < 0:
		key[0] = key[0] + 256
	key[1] = ord(b[i][2]) - 0x6
	if key[1] < 0:
		key[1] = key[1] + 256
	print key[0], key[1]
	for j in range(12, 0xd4):
		o.write(chr(ord(b[i][j]) ^ key[j%2]))

for j in range(12, 0xb5):	// 最后一个块单独列出来，正好凑到文件结尾。
	o.write(chr(ord(b[len(b)-1][j]) ^ key1[j%2]))

o.close()
f.close()

#s1mp!3_n37w0rk_c4@1l3nge
```

[题目下载](/public/download/pcapin_73c7fb6024b5e6eec22f5a7dcf2f5d82.pcap)