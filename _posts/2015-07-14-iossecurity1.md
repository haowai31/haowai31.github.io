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

## Hello World #
按照惯例，写一个helloworld的程序，放在手机里执行作为开始。要在MAC上编译一个可以在iOS上执行的C程序，需要编译器，也需要指定SDK，下面分别进行介绍。

### 编译器 #
原来交叉编译环境都是arm-开头，中间应该有关键字llvm-gcc，我找了半天，毛都没有，后面去Xcode更新日志上看到，从Xcode开始编译器都指定为clang，不再使用llvm-gcc，这一头包。后来在StackOverflow上面找到了一种方法，就是执行命令：

	xcrun -f --sdk iphoneos clang

xcrun是Xcode中command line tool的工具，这个命令可以找到Xcode使用的sdk包的地址。而iphoneos则是指定的平台，这样就找到了clang的位置。不过，这里clang是一个通用的编译器，需要指定平台和SDK包才行。

### SDK #
SDK包的位置倒是好找一点，直接find搜所有的SDKs找到相应SDK的位置即可。我要在iPhone执行我的helloworld，那么就选iPhone的SDK，这里SDK的版本是8.4。

### 编译 #
运行命令：
	
	clang --target=arm64-apple-darwin14.4.0 -isysroot $SDKs

这中间target是指定平台，这里我用的iPhone6p，所以平台是arm64，isysroot指定的是SDK的文件，这里是我自己设的变量，需要改成自己的目录。

### 运行 #
通过命令scp放在手机上，用ldid签名然后就可以运行了。
