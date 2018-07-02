---
layout: post
status: publish
published: true
title: "StarterWare How to build the first example LEDBlink "
author: Dillon Peng
date: 2018年 07月 02日 星期一 17:06:04 CST
comments: []
---

  有一块AM3354的开发板，提供了原理图，u-boot可执行文件MLO和u-boot.img（不提供源代码和裸板开发，裸板开发可理解为基于TI的StareWare开发小应用)，部分linux驱动源码，要想移植U-boot到这个板子上，基本思路是，在TI的CCS里编译一个StarterWare小例子，如题，让板子上的一个LED闪烁，然后到TI找AM335X的u-boot SDK源码，根据板子的硬件资源一步步调整，直到可以装载和运行Linux。
