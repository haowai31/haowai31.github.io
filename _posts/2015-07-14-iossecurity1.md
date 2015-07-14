---
layout: post
title: iOS安全实验记录（一）：搭建环境与所需工具
category: ios安全
date: 2015-07-14
---

会持续更新iOS相关的安全实验，具体多少更不清楚，有好的主题就继续往下做，包括但不限于以下几个方面（这是已经做过和很感兴趣的）：

1. 逆向某些App。
2. Hook与注入实验。
3. ROP攻击实验。
4. 越狱开发。

有兴趣探讨实验或提供实验主题请联系**qinsky31@gmail.com**。

<!-- more -->

今天，主要讲一下我这里搭建的环境，以及使用工具。

### Theos #
这是一款越狱开发的工具包，与iOSOpenDev不同，这个工具包并不那么的**完善**，而逆向工程中很多东西无法自动化，所以使用这个不怎么完善的工具包可以方便对整个逆向的过程理解更加清晰明确。

### ldid #
签名用，在MAC上放一个也可以，在iPhone上放一个也可以。

### CydiaSubstrate #
著名的越狱开发的基础包，主要提供了三种功能，使用简单方便：Hook，Loader，Safemode。Hook和Loader都不陌生，可以对目标进程进行Hook和注入。Safemode是CydiaSubstrate引入的专为越狱开发而做的，因为Loader使用的dylib载入第三方lib的，如果第三方lib（也就是自己的lib）出错的话，会让被注入的进程崩溃。而这个safemode可以识别SIGTRAP、SIGABRT、SIGILL、SIGBUS、SIGSEGV、SIGSYS六个信号，一旦被触发就进入safemode，禁用所有第三方lib，方便调试。

### Cycript #
一个可以进行运行在MTerminal的脚本语言，很方便的就注入进程，查看某函数的运行结果，某个中间结果都非常方便。

### LLDB和debugserver #
由于笔者研究方向与LLVM相关，所以老老实实的用lldb进行调试了。

### OpenSSH #
方便手机远程调试。

### usbmuxd #
一个将USB协议抽象成为TCP协议的服务，可以使用USB插口进行调试。

### MTerminal #
iOS上的终端。

### syslogd #
记录系统日志。进行越狱开发的时候syslog是不可或缺的帮手。

### otool #
不用安装，可以查看程序依赖的动态库。


