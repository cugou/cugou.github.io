---
layout: post
title: 阿里ctf-2014 android 第一、二题
category: android
tags: reverse
keywords: 
description: 
---

# EvilAPK-100
```
APK在执行过程中使用了一个文件作为输入，请问该文件的名称是什么？（不需要路径）
题目下载地址：见附件
```

这个题目继续验证dexdecompile的有效性。

![图一](/public/img/2015-01-16-alictf-2014_android_No1_No2-1.jpg)

这里反编译出来发现问题，两个native函数没有参数。而实际调用情况是：

![图二](/public/img/2015-01-16-alictf-2014_android_No1_No2-2.jpg)

有一个String参数。

所以dexdecompile对于class.dex的反编译还是存在一些问题的。当然有可能其封装的androguard不是最新版本，所以导致这个情况。

题目的答案看一下String v0的赋值就可以了。


---------
# EvilAPK-200
```
请分析static_analysis.apk安装包。
分析出在静态代码中有多少个地方调用了sendSMS方法（不包括该方法本身且flag为数字）
题目下载地址：见附件
```

1、解压缩出classes.dex使用dex2jar反编译成jar包。
    dex2jar.bat classes.dex
    生成classes_dex2jar.jar，用jd-gui打开，搜索-method方法，去掉Declarations，因为题目要求不包含函数本身。

![图三](/public/img/2015-01-16-alictf-2014_android_No1_No2-3.jpg)

可以看到找到18个匹配。

2、使用apktool反编译。
    apktool.jar d 2.apk
    会生成同文件名目录。这个和dexdecomile生成的相似，只是后者的目标文件是classes.dex。
    使用sublime text导入文件夹，搜索

![图四](/public/img/2015-01-16-alictf-2014_android_No1_No2-4.jpg)

比较发现两者有些不一致，因为学习目的，没有参加比赛，掌握方法就行了，结果不是很重要。
