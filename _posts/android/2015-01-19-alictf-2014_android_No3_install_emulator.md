---
layout: post
title:  阿里ctf-2014 android 第三题——andriod模拟器安装
category: android
tags: reverse
keywords: 
description: 
---

```
EvilAPK-300
  该破解程序jscrack主界面包含两个控件：
  1）URL输入框
  2）进入”按钮“
  要求自己构造一个网页，并把网页对应的URL输入到URL输入框控件，然后，点击”进入”按钮，jscrack会打开webview浏览你的网页，如果jscrack能弹出一个Toast，就证明已经成功破解，同时Toast显示的内容就是这个题目的flag。
```

说实话这道题目学到了许多知识，有些还是强迫去学的。
以往的Andriod题目基本都是算法逆向，偶尔有几个native函数，也是考的算法。这个题目需要搭建andriod模拟器（当然更完整的是搭建一个完整的andriod developer IDE）。
一开始想偷懒，以为随便找个windows的android模拟器就行了，后来发现这些实际上都是x86平台的模拟器。真正支持arm的模拟器这些似乎都不行。
然后又想到使用qemu这款开源的模拟系统，他可以在x86平台上模拟arm指令系统，在这个平台上可以安装arm-linux。
后来发现，实际上不用这么麻烦，android开发者平台集成了一个模拟器，支持x86 atom、arm、mips三种指令系统。实际上这个模拟器底层就是使用的qemu。

### 安装android emulator：
我是经历了一个痛苦的过程，这个过程是伟大的GFW带来的。真想不明白，既然这么痛恨google，为什么还允许google的产品成为绝大多数国人手中的玩物！！！
1、最简单的方法，直接安装andriod studio。
http://developer.android.com/sdk/index.html
2、定制的方法，下载andriod sdk tools。
http://developer.android.com/sdk/index.html#Other
这个tools里面主要有两个重要的工具图形化工具SDK Manager和AVD Manager。前者用来下载各种版本的SKD以及各种platform-tools等待。后者用来生成和管理对应某个SDK版本和API Level的andriod虚拟机。

![图](/public/img/2015-01-19-alictf-2014_android_No3_install_emulator-1.jpg)

![图](/public/img/2015-01-19-alictf-2014_android_No3_install_emulator-2.jpg)

![图](/public/img/2015-01-19-alictf-2014_android_No3_install_emulator-3.jpg)

![图](/public/img/2015-01-19-alictf-2014_android_No3_install_emulator-4.jpg)

当然说着简单，坑爹的是andriod和google都被GFW了，挂代理那个速度还是要有耐心的。直到后来才发现原来android developer在国内有镜像。到底不是专业开发andriod的，这里浪费了很多时间。
http://wear.techbrood.com/


做了一大堆的准备工作后，回到题目上。
这个题目实际上我是不会的，直接参考人家的提示——webView漏洞。
努力学习了google了一遍，发现这个漏洞2011年就有组织发不了一个pdf，2012年有人利用java反射创建类的实例发布进一步利用的poc，到2013年中，才公布出一个有杀伤力的POC，引起广泛关注。
漏洞具体的就不说了。乌云检测网址：
http://drops.wooyun.org/webview.html
这个题目存在一个`SmokeyBear`的`js to java`对象，可以在网页的js脚本中使用这个对象访问java的内置对象。
题目的要求找到Toast很有意思，百度了一下：Toast是andriod里面提供的一个类似MessageBox的一个东东，特点是显示一段时间后会自动关闭。
如何找到java代码里面显示Toast的函数，这个我是没有想明白，不过别人的提示是`SmokeyBear.showToast()`函数。

在服务器建立a.html，输入一下内容：
```javascript
<script type="text/javascript">
SmokeyBear.showToast();
</script>
```

首先将3.apk导入android emulator：
将3.apk拷贝到android-sdk\platform-tools目录下，cmd到这个目录下，执行`adb install 3.apk`
在模拟器中找到`jscrack`应用，打开。输入http://ip/a.html，点击进入。弹出Toast。


最后一点疑问就是，如何从反编译出来的so中找到相应的代码。这样能更加清楚的理解整个webView的调用脉络。因为这个程序反编译出来就是3个native函数，在so中又没有找到对应的export函数。我是个andriod菜鸟，希望有大牛能指点如何静态反编译。

[题目下载](/public/download/2015-01-19-alictf-2014_android_No3_install_emulator-3.rar)