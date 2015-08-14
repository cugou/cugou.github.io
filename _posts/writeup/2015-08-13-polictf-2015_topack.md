---
layout: post
title: polictf-2015 topack
category: reverse
tags: ctf
keywords: 
description: 
---

此题略过静态分析过程，只列出其中的要点：

### 关键的“解密--执行--加密”函数**sub_80485e0**

```
	mprotect 修改代码段内存页属性为 RWX。
	获取解密
		密钥：* (int_804A294[arg_0 % 5])  // 数组存放的是密钥的地址，所以最后还要取值。
		解密：[arg_0] ^ 密钥 	// 每次一个 int 单位，循环 arg_1 次
	sub_804859B	重新加密回去
```

### 静态解密脚本：

```python
arg0s = [0x8048AA5, 0x8048655, 0x804869A, 0x80486DE, 0x8048A42, 0x804873A, 0x80489A9, 0x8048813, 0x804890B, 0x80488E4]
arg1s = [0x53, 0x11, 0x11, 0x17, 0x18, 0x36, 0x26, 0x34, 0x27, 0x9]

print "unpack start..."
for k in range(0, len(arg0s)):
	index = arg0s[k] % 5
	key = Dword(Dword(0x804A294 + index*4))
	print "key is " + hex(key)
	for i in range(0, arg1s[k]):
		ret = PatchDword(arg0s[k] + i*4, Dword(arg0s[k] + i*4) ^ key)
		if ret == 0:
			print "error"
			break

	MakeUnkn(arg0s[k], DOUNK_SIMPLE)	# undefine area first can raise posiblity of succession of MakeFunction.
	# two statements do not work correctly, but do it manually in IDA will work fine.
	MakeCode(arg0s[k])
	MakeFunction(arg0s[k])
print "unpack done!"
```

对比脚本运行前和脚本运行后的函数列表，可以看出，很多函数被解密出来。

![图1](/public/img/2015-08-14-polictf-2015-topack-1.PNG)   ![图2](/public/img/2015-08-14-polictf-2015-topack-2.PNG)

但有一个最关键的函数**sub_8048AA5**脚本没有 MakeFunction 成功。这个函数就是验证flage的函数。没关系，我们可以自己在 IDA 手动定义。
使用 IDA 的 graph 功能，画出 sub_8048AA5 的调用树。IDA 很智能的将通过 sub_80485E0 解密后调用的，函数都画出来了，而且智能去掉了这一多余调用层级。

![图3](/public/img/2015-08-14-polictf-2015-topack-3.PNG)

除了最后边的调用分支是调用 sub_80485E0 本身，以及 sub_804859B 是将解密后的函数加密回去以外，其余直接调用的 7 个分支就是验证 flage 的 7 个函数。

1、sub_8048655

		验证是否以`flag{`开头

2、sub_804869A

	 	验证是否以`}`结尾

3、sub_80486DE

		验证是否有`> 0x7F`的字符

4、sub_8048A42

		验证`flag{`后的6个字符。
	sub_804873A
		根据参数arg0 = 1--6，会生成6个不同的字符，这就是flag的第一部分。汇编代码很复杂，但调试很容易知道这些字符为`packer`。

5、sub_80489A9

		验证flage的后11个字符。其中整个flag[0x11]位字符的最小二进制为要为 1。
	sub_8048813
		每个字符都要通过这个函数的验证，返回值为 1，表示正确，为 0，表示错误。
		这个函数非常复杂，我是看不懂具体的算法，只有暴力之。手工当然不是最好的选择了，除非闲着没事。
		暴力方式可以选择四种：
			a、dump出这一段函数的二进制，写一个壳直接调用，但需要重定位 pow 函数的调用。也可以个dump出汇编代码，用汇编重写编译。
			b、PyEmu之类的x86模拟器，但同样需要解决 pow 调用的问题。当然PyEmu可以连库函数的内存也加载进来一起模拟。
			c、直接用IDA的DBG_Hooks，当然需要解决好断点的Add和Del，因为 sub_80485E0 解密后还会加密回去。
			d、由于已经静态解密，可以直接patch掉sub_80485e0的间接调用，改为直接调用。dump出新的ELF文件，就可以使用类似python gdb的方式进行暴力了
					
		我用的第三种方法，暴力flag[0x11]时，有多个符合条件的字符，其他都是唯一解。结果是：`-15-4-k41=-`，flag[0x11]我选择字符`k`。

6、sub_804890B

		验证从flag[0x14]开始往后的字符，算法就是和08048D41处的字节数组进行 xor。结果是：`in-th3-4ssssssssssssss`。

7、sub_80488E4

		验证flag的长度 == 0x21。

七个步骤验证都通过，则组合起来，正确的flag就是：`flag{packer-15-4-k41=-in-th3-4ss}`。
当然由于flag[0x11]存在多个解，这个题目答案不唯一。