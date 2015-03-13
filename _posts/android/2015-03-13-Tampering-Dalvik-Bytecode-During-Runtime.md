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

[图](/public/img/2015-03-13_delta_zip_encrypt.png)
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

[图](/public/img/2015-03-13_delta_file_encrypt.png)
[图](/public/img/2015-03-13_delta_direntry_encrypt.png)


# 第二步：