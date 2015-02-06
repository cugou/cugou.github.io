---
layout: post
title:  阿里ctf-2014 android 第三题——so动态调试及破解加固
category: android
tags: reverse
keywords: 
description: 
---

通过做题来学习android逆向是一个比较不错的方法。
虽然有投机取巧的方法解决这题，但是对这个题目的深入研究，学习到了dex的动态调试、破解加固等技术。
要感谢各位android大牛对此题的详细介绍。我只是照着做了一遍，对有些不明确的地方自己演练了一下，并对android的so调试进行了一下简单的归纳。

## 从零开始进行android的so代码调试：
### 一、工具准备：
jre——java runtime enveroment    //下面的一切工具都要这个支持
jdk——java development kits    //需要用到里面的jdb程序
Android SDK Manager    //包括adb、ddms等各种工具，及各种SDK版本下载管理
Android AVD Manager    //用来建立各种SDK版本和相应API级别的android虚拟机

为了方便，将包含以上各种实用工具的目录如sdk\platform-tools，sdk\tools等路径加入Path环境变量。
新建一个虚拟机：见我的前一篇文章
    我使用的是SDK 4.0.3、API level 15。
    安装apk：adb install 3.apk

### 二、调试
这个还是参考大牛的文章。我这里只把我参照大牛文章后自己做的截图贴出来。

![图](/public/img/2015-01-22-alictf-2014_android_No3_so_debug-1.jpg)

![图](/public/img/2015-01-22-alictf-2014_android_No3_so_debug-2.jpg)

![图](/public/img/2015-01-22-alictf-2014_android_No3_so_debug-3.jpg)

![图](/public/img/2015-01-22-alictf-2014_android_No3_so_debug-4.jpg)

![图](/public/img/2015-01-22-alictf-2014_android_No3_so_debug-5.jpg)

![图](/public/img/2015-01-22-alictf-2014_android_No3_so_debug-6.jpg)

![图](/public/img/2015-01-22-alictf-2014_android_No3_so_debug-7.jpg)

### 三、分析
因为不会使用idc脚本，所以我的dump内存方法是利用Edit菜单，导出rawdata。（看来是要好好学习一下IDA的高级功能了）
导出的dex，使用dex2jar转换成jar包，jd-gui查看，没有看到完整的内容。
使用dexdecompile完整反编译出了具体的内容。这个有可能是dump出的是odex格式的，dex2jar可能只支持dex格式。
直接找`addJavascriptInterface`函数，发现其导出的接口对象名是加密的。
当然就这个题目来说，接口对象名可以使用wooyun的那个链接来枚举出来，或者再次利用上面的方法调试出来。或者根本不用管直接找`Toast.MakeText`函数里面的参数，就是答案了。（见上图）

作为一个喜欢刨根问底的人，自然那个translate.so和decrypt_native函数也要研究一番。一方面是强化一下so的调试技术，另一个方面是锻炼一下ARM指令集的逆向能力。
