---
layout: post
title: Sublime Text2插件SFTP破解
category: reverse
tags: reverse
keywords: 
description: 
---

插件没有使用限制，只是每10次使用会弹一个购买对话框。请大家还是尊重正版，多多支持开发者吧。
破解很简单，当一个乐趣吧

最新写用python的flask框架写一个web应用程序（virtualenv环境）。服务器使用无图形界面的ubuntu server，这个带来了编辑上的难题。
vim实在是太难配置的如同sublime text这样顺手了。虽然使用git，可以实现本地开发和服务器的同步（参考本笔记的相关文章）。但需要运行几个命令还要输入密码，对于频繁的只修改一两个单词的编辑部署需求，显然过于繁琐了。
幸好发现还有sftp这种sublime text的神奇插件，可以支持本地sublime text2直接打开远程服务器上的文件，编辑后能直接Ctrl+s同步到服务器。

## 1、找到sftp插件文件的安装位置：
windows下，sublime text2的插件都安装在 C:\Users\d\AppData\Roaming\Sublime Text 2\Packages\ 目录下。
C:\Users\d\AppData\Roaming\Sublime Text 2\Packages\SFTP\sftp 目录下就是该插件的执行脚本，都是pyc文件。

## 2、找到关键代码文件：
根据弹出对话框的提示，找到关键词“purchase”。
strings *.pyc | findstr purchase 或者 strings *.pyc | grep purchase

发现只有一条输出，只有一个文件的一处地方存在关键点。一个一个文件找，很容易定位到commands.pyc文件。

## 3、反编译commands.pyc
找个能反编译python2.7的就行，我机器安装的是python2.7。
对抗反编译，去年ctf上的一个python的题目，前面多了2字节的nop，导致没有现成的python语句能表示（python指令长度一般3字节），最终反编译出错。但貌似这种方法，需要对后面的代码进行大范围的地址重定位。对于这种大好几千行的文件，太坑了点。

## 4、代码分析：
第56行开始：
```python
class SftpCommand(object):
    connections = {}
    identifiers = {}
    usage = {}
    remote_roots = {}
    remote_time_offsets = {}

    @classmethod
    def setup_elements(cls, config):
        if not hasattr(SftpCommand, 'elements'):
            SftpCommand.elements = [0,
             config.get('email'),
             'sftp_flags',
             'ssh_key_file',
             'psftp']
            SftpCommand.elements.append('Sublime SFTP\n\nThanks for trying out Sublime SFTP. It is free to try, but a license must be purchased for continued use.\n\nPlease visit http://sublime.wbond.net/sftp for details.')
            SftpCommand.elements.append(config.get('product_key'))    #这个值经过分析config.pyc的源代码，实际上就是从SFTP.sublime-settings配置文件中读取。
            key = SftpCommand.elements[1]
            if isinstance(key, str_cls):
                key = key.encode('utf-8')
            for element in SftpCommand.elements[2:-2]:
                key = hmac.new(element.encode('utf-8'), key).digest()

            SftpCommand.elements[1] = key
```

第240行开始：
```python
                    if self.action not in ('list', 'listr', 'llist', 'llistr', 'cwd'):
                        SftpCommand.elements[0] += 1

                        def zip_to_i(element):
                            if isinstance(element, int):
                                return element
                            return ord(element)

                        if sys.version_info >= (3,):
                            key_prefix = bytes([ zip_to_i(x) ^ zip_to_i(y) for x, y in izip(SftpCommand.elements[1], cycle('22')) ])
                        else:
                            key_prefix = ''.join([ chr(zip_to_i(x) ^ zip_to_i(y)) for x, y in izip(SftpCommand.elements[1], cycle('22')) ])
                        key_prefix = binascii.hexlify(key_prefix)[:30]
                        clen = 6
                        chunks = [ key_prefix[(i - 1) * clen:i * clen] for i in xrange(1, int(len(key_prefix) / clen + 1)) ]
                        key_prefix = '-'.join(chunks).decode('utf-8')

                        #2016-04-11 added by snake. debug out key
                        f = open('M:\\aaaaa.txt', 'w')
                        f.write(key_prefix)
                        f.close()
                        #debug out key

                        #每10此弹一次对话框
                        if SftpCommand.elements[0] > 0 and SftpCommand.elements[0] % 10 == 0 and key_prefix != SftpCommand.elements[-1]:

                            def reg():
                                if int(sublime.version()) >= 2190:    #不是所有的sublime text版本支持对话框
                                    if sublime.ok_cancel_dialog(SftpCommand.elements[-2], 'Buy Now'):
                                        sublime.active_window().run_command('open_url', {'url': 'http://wbond.net/sublime_packages/sftp/buy'})
                                else:
                                    sublime.error_message(SftpCommand.elements[-2])

                            sublime.set_timeout(reg, 1)
                            do_show = True
                    self.do_operation(do_show)
```

## 5、crack it
实际上能完美反编译，并找到关键地方，直接去除就可以了。但完美破解是要生成注册码，实际上也不难。
```python
import hmac, binascii
from itertools import izip, cycle

def zip_to_i(element):
    if isinstance(element, int):
        return element
    return ord(element)

elements = ['sftp_flags', 'ssh_key_file', 'psftp']
key = 'snake@snake.org'.encode('utf-8')        #使用snake@snake.org进行注册
for element in elements:
    key = hmac.new(element.encode('utf-8'), key).digest()

key_prefix = ''.join([ chr(zip_to_i(x) ^ zip_to_i(y)) for x, y in izip(key, cycle('22')) ])
key_prefix = binascii.hexlify(key_prefix)[:30]
clen = 6
chunks = [ key_prefix[(i - 1) * clen:i * clen] for i in xrange(1, int(len(key_prefix) / clen + 1)) ]
key_prefix = '-'.join(chunks).decode('utf-8')
print key_prefix
```

在sublime text2中 Preferences--Package Settings--SFTP--Settings-Default 打开 C:\Users\d\AppData\Roaming\Sublime Text 2\Packages\SFTP.sublime-settings 文件，输入email和product_key的键值对。
