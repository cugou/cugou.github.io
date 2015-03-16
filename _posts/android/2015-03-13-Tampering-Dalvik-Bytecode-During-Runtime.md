---
layout: post
title: 运行时篡改dalvik字节码 delta.apk练习
category: android
tags: reverse
keywords: 
description: 
---

# 资料和题目

[Android Security Analysis Challenge: Tampering Dalvik Bytecode During Runtime](https://bluebox.com/technical/android-security-analysis-challenge-tampering-dalvik-bytecode-during-runtime/)
[题目下载](https://github.com/blueboxsecurity/DalvikBytecodeTampering/raw/master/delta.apk)

# 第一步：zip伪加密

![图](/public/img/2015-03-13_delta_zip_encrypt.png)

winrar打开apk可以看到有文件被打 * 号，说明是加密的，解压缩提示需要输入密码。

`
原理：dalvik加载apk的时候没有检查文件头，而加解密工具是都会检查的。
`

去除伪加密需要了解zip压缩文件的格式，当然不了解也没什么关系：

1. 可以使用作者提供的[解压缩脚本](https://github.com/blueboxsecurity/DalvikBytecodeTampering/blob/master/unpack.py)，秒杀一切伪加密。
2. 使用强大的010Editor，进行手工修改。

就这个delta.apk来说，需要使用`ZIPTemplateADV.bt`这个模板。打开后可以发现实际上zip文件是扁平化的，压缩文件被组织成`struct ZIPFILERECORD`结构的数组，同样下面跟着`struct ZIPDIRENTRY`用来组织文件的目录结构。除了两个数组，没有标识整个文件的单独的头。

对于上图显示的加密，可以看到`ZIPFILERECORD.flagType`和`ZIPDIRENTRY.flagType`都显示`encrypt`。但手动修改只有针对`ZIPDIRENTRY.flagType`才能成功，也就是只需要去除目录的加密标志位就可以了。

`ushort flagType`其中的第0为标识文件是否加密，改成0就可以了。
具体方法当然使用010Editor的`replace`功能，搜索替换字节序列。（从magic number开始到flagType）

![图](/public/img/2015-03-13_delta_file_encrypt.png)
![图](/public/img/2015-03-13_delta_direntry_encrypt.png)


# 第二步：分析代码替换（动态调试）

java层代码是调用`String.add`函数，显然在native方法`search`里面应该就是替换这个函数，至于替换成什么，调试获得。

![图](/public/img/2015-03-13_delta_func_search.png)

```c
//这个调用返回当前系统的内存页大小，一般都为4k。
sysconf(_SC_PAGESIZE)	//#define _SC_PAGESIZE 0x27
```

函数的搜索原理是基于2013年5月2日HES2013上Xavier ‘xEU’ Martin的pdf里面对android加载classes.dex到内存地址的描述

`It'll be aligned on _SC_PAGESIZE, at offset 0x28`

实际上加载到`_SC_PAGESIZE`边界上的是odex文件，0x28是odex header的长度。当然odex在dex_off处包含dex文件。

接下来的代码是查找dex文件中`String.add`函数的`method_code_off.insns`所指向的data段中的位置——也就是实际代码的位置。然后调用`mprotect`将该内存页设置为`RW`，最后拷贝libnet.so中的`inject`函数代码到这个位置，实现运行时函数代码层的替换。

![图](/public/img/2015-03-13_delta_debug_memcpy.png)

![pic](/public/img/2015-03-13_delta_code_off.png)

图中需要说明的是我的模拟器上搜索到的odex的加载基址是`0x48000000`，所以dex文件的地址是`0x48000028`。可以据此转换成相对地址理解文件结构。

# 第三步、其他分析

![pic](/public/img/2015-03-13_delta_func_findmagic.png)

这里有个非常有趣的地方，`memcmp`比较后返回0才是相等。后面用来判断返回值的算法很有意思。

```asm
NEGS	R3, R0
ADCS	R0, R3
```

查询ARM手册关于NEGS指令的说明（该指令只在thumb模式下才有，而且是个伪指令，可以用`0-num`的算术指令来实现。

但问题是，按照这个说法，只有`R0 > 0`的时候，执行NEGS，标志寄存器的C = 1。ADCS执行的结果才不为0。
但调试的结果却是，只有`R0 == 0`的时候，NEGS才会将标志寄存器的C = 1。

这个和判断字符串是否相等的条件是等价的。但和手册说明有混淆，所以关键还是这个NEGS对标志寄存器的影响没有讲清楚啊。