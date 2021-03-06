---
author: dashashi
comments: true
date: 2011-08-30 02:39:21+00:00
layout: post
slug: '%e9%99%a4%e6%b3%95%e3%80%81%e4%bd%8d%e8%bf%90%e7%ae%97%e3%80%81%e7%bc%96%e8%af%91%e5%99%a8%e4%bc%98%e5%8c%96%e5%92%8c%e6%97%a0%e7%ac%a6%e5%8f%b7%e6%95%b0%e7%9a%84%e6%81%a9%e6%80%a8%e6%83%85%e4%bb%87'
title: 除法、位运算、编译器优化和无符号数的恩怨情仇（一）
wordpress_id: 1362
categories:
- 写程序
tags:
- 优化
- 位运算
- 编译器
- 除法
---

写程序稍微有点年头的人都知道，一般情况下CPU的位运算比乘除运算是快很多的，比如说k %= 64,完全能够写成 k &= 63来达到加速的目的，k *=2也完全可以写成k<<= 1.在电脑上这东西可能还没有那么严重，因为CPU原生了比较快的一个乘法、除法指令，在写嵌入式程序的时候尤其是不带硬件乘法器、除法器的时候用这些小技巧能够省下不少时间，对于那种一秒钟触发几万、几十万次的操作来说，这样子能够省下<!-- more -->很大的CPU资源。

我们都知道的道理，写编译器的人当然也知道，我一直在想，如果我在程序里面写了一句k %= 64,编译器到底会不会帮我优化成k &= 63,今天突然奇想，准备看一下。

先试了一下最近在用的MDK，由于MDK的优化级别比较高，为了不让我的费语句被编译器优化掉，我写了总共五句话来告诉编译器“这个t变量是实实在在起作用了的，求求你别优化”掉- -|

    
    	int t;
    	t = rand();
    	t %= 64;
    	srand(t);
    	t = rand();


汇编之后结果如下：

    
        11:         int t;
    0x08000430 B508      PUSH     {r3,lr}
        12:         t = rand();
    0x08000432 F7FFFED3  BL.W     0x080001DC
    0x08000436 4604      MOV      r4,r0
        13:         t %= 64;
    0x08000438 4620      MOV      r0,r4
    0x0800043A EA4F71E4  ASR      r1,r4,#31
    0x0800043E EB046191  ADD      r1,r4,r1,LSR #26
    0x08000442 EA4F11A1  ASR      r1,r1,#6
    0x08000446 EBA41481  SUB      r4,r4,r1,LSL #6
        14:         srand(t);
    0x0800044A 4620      MOV      r0,r4
    0x0800044C F002FB88  BL.W     0x08002B60
        15:         t = rand();


发现结果很神奇，编译器既没有简单的把%编译为除法，也没有编译为与运算达到优化的目的，而是有一大堆语句单周期指令，灵光一闪，难道是因为有符号数，所以编译器要考虑的东西比较多？于是把int t改成了unsigned t，再看一下。

    
        11:         unsigned int t;
    0x08000430 B508      PUSH     {r3,lr}
        12:         t = rand();
    0x08000432 F7FFFED3  BL.W     0x080001DC
    0x08000436 4604      MOV      r4,r0
        13:         t %= 64;
    0x08000438 F004043F  AND      r4,r4,#0x3F
        14:         srand(t);
    0x0800043C 4620      MOV      r0,r4
    0x0800043E F002FB89  BL.W     0x08002B54
        15:         t = rand();


果然不出意料，%被优化到只有一句简单的&

接下来试试除法运算，既然都改成unsigned int了，就先试无符号数吧

    
        11:         unsigned int t;
    0x08000430 B508      PUSH     {r3,lr}
        12:         t = rand();
    0x08000432 F7FFFED3  BL.W     0x080001DC
    0x08000436 4604      MOV      r4,r0
        13:         t /= 64;
    0x08000438 EA4F1494  LSR      r4,r4,#6
        14:         srand(t);
    0x0800043C 4620      MOV      r0,r4
    0x0800043E F002FB89  BL.W     0x08002B54
        15:         t = rand();


不出所料，编译器很聪明的优化成了一个逻辑右移，再试试乘法

    
        11:         unsigned int t;
    0x08000430 B508      PUSH     {r3,lr}
        12:         t = rand();
    0x08000432 F7FFFED3  BL.W     0x080001DC
    0x08000436 4604      MOV      r4,r0
        13:         t *= 64;
    0x08000438 EA4F1484  LSL      r4,r4,#6
        14:         srand(t);
    0x0800043C 4620      MOV      r0,r4
    0x0800043E F002FB89  BL.W     0x08002B54
        15:         t = rand();


很顺利，优化成了左移

对于非2的整数次幂会怎么样，比如48？试试看！

    
        11:         unsigned int t;
    0x08000430 B508      PUSH     {r3,lr}
        12:         t = rand();
    0x08000432 F7FFFED3  BL.W     0x080001DC
    0x08000436 4604      MOV      r4,r0
        13:         t *= 48;
    0x08000438 EB040044  ADD      r0,r4,r4,LSL #1
    0x0800043C EA4F1400  LSL      r4,r0,#4
        14:         srand(t);
    0x08000440 4620      MOV      r0,r4
    0x08000442 F002FB89  BL.W     0x08002B58
        15:         t = rand();


看来编译器十分聪明，通过位运算的办法间接实现了乘法运算，我们知道，48在2进制下面是110000，因为只有2个1，所以可以这么干，来试试1比较多的情况，比如10101010，也就是170

    
        11:         unsigned int t;
    0x08000430 B508      PUSH     {r3,lr}
        12:         t = rand();
    0x08000432 F7FFFED3  BL.W     0x080001DC
    0x08000436 4604      MOV      r4,r0
        13:         t *= 170;
    0x08000438 F04F00AA  MOV      r0,#0xAA
    0x0800043C FB04F400  MUL      r4,r4,r0
        14:         srand(t);
    0x08000440 4620      MOV      r0,r4
    0x08000442 F002FB89  BL.W     0x08002B58
        15:         t = rand();


编译器表示无能为力了，直接给我来了一条mul指令，不过实际上由于stm32是带了单周期硬件乘法器的，所以mul指令反而很快……

接下来试试有符号数的情况

先是除法

    
        11:         int t;
    0x08000430 B508      PUSH     {r3,lr}
        12:         t = rand();
    0x08000432 F7FFFED3  BL.W     0x080001DC
    0x08000436 4604      MOV      r4,r0
        13:         t /= 64;
    0x08000438 4620      MOV      r0,r4
    0x0800043A EA4F71E4  ASR      r1,r4,#31
    0x0800043E EB046191  ADD      r1,r4,r1,LSR #26
    0x08000442 EA4F14A1  ASR      r4,r1,#6
        14:         srand(t);
    0x08000446 4620      MOV      r0,r4
    0x08000448 F002FB88  BL.W     0x08002B5C
        15:         t = rand();


嗯嗯，编译器通过四句单周期指令间接实现了除法，主要还是因为多了负数的情况

看看乘法的情况

    
        11:         int t;
    0x08000430 B508      PUSH     {r3,lr}
        12:         t = rand();
    0x08000432 F7FFFED3  BL.W     0x080001DC
    0x08000436 4604      MOV      r4,r0
        13:         t *= 64;
    0x08000438 EA4F1484  LSL      r4,r4,#6
        14:         srand(t);
    0x0800043C 4620      MOV      r0,r4
    0x0800043E F002FB89  BL.W     0x08002B54
        15:         t = rand();


编译器利用了LSL指令一次性实现了乘法的运算，试试乘以48

    
        11:         int t;
    0x08000430 B508      PUSH     {r3,lr}
        12:         t = rand();
    0x08000432 F7FFFED3  BL.W     0x080001DC
    0x08000436 4604      MOV      r4,r0
        13:         t *= 48;
    0x08000438 EB040044  ADD      r0,r4,r4,LSL #1
    0x0800043C EA4F1400  LSL      r4,r0,#4
        14:         srand(t);
    0x08000440 4620      MOV      r0,r4
    0x08000442 F002FB89  BL.W     0x08002B58
        15:         t = rand();


跟无符号数类似，170也试试

    
        11:         int t;
    0x08000430 B508      PUSH     {r3,lr}
        12:         t = rand();
    0x08000432 F7FFFED3  BL.W     0x080001DC
    0x08000436 4604      MOV      r4,r0
        13:         t *= 170;
    0x08000438 F04F00AA  MOV      r0,#0xAA
    0x0800043C FB04F400  MUL      r4,r4,r0
        14:         srand(t);
    0x08000440 4620      MOV      r0,r4
    0x08000442 F002FB89  BL.W     0x08002B58
        15:         t = rand();


不出意料，它放弃优化了。

结论：为了能够让编译器达到比较好的优化效果，能够用无符号数的地方就别用有符号数。

文章比较长了，Visual Studio 2010中的测试再开一篇吧。
