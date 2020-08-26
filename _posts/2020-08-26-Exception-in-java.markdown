---
layout: post
title:  "JAVA中的异常分类"
date:   2020-08-26 17:03:36 +0530
categories: JAVA 异常
---
![plainwhite theme preview](/assets/images/exceptionInJava.png)
**Error**
1．总是不可控制的(unchecked)。
2．经常用来用于表示系统级错误或低层资源的错误。
3．如何可能的话，应该在系统级被捕捉。

Exception：
1．可以是可被控制(checked) 或不可控制的(unchecked)。
2．表示一个由程序员导致的错误或者其他外部原因(非程序原因)导致的错误。
3．应该在应用程序级被处理。


从逻辑的角度来说，checked exceptions和runtime exception是有不同的使用目的：
	1. checked exception用来指示一种调用方能够直接处理的异常情况。
	2. 而runtime exception则用来指示一种调用方本身无法处理或恢复的程序错误。

从设计者角度来看：checked异常为调用方可处理的：
	* 调用方能够进行处理的异常；
	* 虽然无法处理，但必须引起重视，需要保留现场的异常，如SQLException（保留现场也可以视为一种处理）。

unchecked异常为调用方不可处理的，需要自己进行处理的异常：
	1. 可预测的，通过代码检查可以在发生前避免的异常，如空指针、数组越界等（即使捕获也无法处理）；
	2. 难以预测、无法由调用方处理但又必须处理的异常，通过重试或降级处理（已不是调用方处理）；
	3. 由框架或系统产生且会自行处理的异常，程序无须关心的异常。

但是，不幸的是，由于Java使用者水平的参差不齐，大量的unchecked exception该被设计成了checked exception，而对于真正的checked exception，又有太多被catch了之后啥都不作就悄无声息了。尤其是不声不响吞噬exception的行为，不但达不到设计者本来的要求（进行恢复处理），甚至问题更大(连 unchecked exception那种最后报错的效果都没了）。

Java中RuntimeException这个类名起的并不恰当，因为任何异常都是运行时出现的。（在编译时出现的错误并不是异常，换句话说，异常就是为了解决程序运行时出现的的错误）。 

参考：阿里JAVA开发手册