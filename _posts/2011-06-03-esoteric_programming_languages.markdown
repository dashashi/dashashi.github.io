---
author: dashashi
comments: true
date: 2011-06-03 05:01:26+00:00
layout: post
slug: '%e8%bd%acbt%e9%9b%b7%e4%ba%ba%e7%9a%84%e7%a8%8b%e5%ba%8f%e8%af%ad%e8%a8%80'
title: '[转]BT雷人的程序语言'
wordpress_id: 1222
categories:
- 未分类
tags:
- C++
- google
- php
- RP
- 程序语言
- 雷人
---

这个世界从来都不会缺少另类的东西，人类自然世界如此，计算机世界也一样。编程语言方面，看过本站《[6个变态的C语言Hello World程序](http://coolshell.cn/?p=914)》的朋友们一定对BT和另类不会陌生，但那都是些小儿科，真正的BT和另类要是从语言级上来完成。让我们来看看其中一个比较另类的语言BrainFuck。看到这个程序语言的名字，请不要以为这是一个搞笑的语言，这是一个“严肃事情”，请大家用“最虔诚的态度”来阅读本文。


#### BF语言介绍<!-- more -->


**Brainfuck**，是一种极小化的计算机语言，它是由Urban Müller在1993年创建的。由于“绿王八”的原因，这种语言有时被称为**brainf**k**或**brainf*****，甚至被简称为**BF**。这种 语言，是一种按照“Turing complete（完整图灵机）”思想设计的语言，它的主要设计思路是：用最小的概念实现一种“简单”的语言，BrainF**k 语言只有八种符号，所有的操作都由这八种符号的组合来完成。

BF基于一个简单的机器模型，除了八个指令，这个机器还包括：一个以字节为单位、被初始化为零的数组、一个指向该数组的指针(初始时指向数组的第一个字节)、以及用于输入输出的两个字节流。

下面是这八种指令的描述，其中每个指令由一个字符标识：


<table border="0" >
<tbody >
<tr >
字符
含义
</tr>
<tr >

<td >`>`
</td>

<td >指针加一
</td>
</tr>
<tr >

<td >`<`
</td>

<td >指针减一
</td>
</tr>
<tr >

<td >`+`
</td>

<td >指针指向的字节的值加一
</td>
</tr>
<tr >

<td >`-`
</td>

<td >指针指向的字节的值减一
</td>
</tr>
<tr >

<td >`.`
</td>

<td >输出指针指向的单元内容（ASCII码）
</td>
</tr>
<tr >

<td >`,`
</td>

<td >输入内容到指针指向的单元（ASCII码）
</td>
</tr>
<tr >

<td >`[`
</td>

<td >如果指针指向的单元值为零，向后跳转到对应的`]`指令的次一指令处
</td>
</tr>
<tr >

<td >`]`
</td>

<td >如果指针指向的单元值不为零，向前跳转到对应的`[`指令的次一指令处
</td>
</tr>
</tbody>
</table>
（按照更节省时间的简单说法，`]`也可以说成“向后跳转到对应的`[`状态”。这两解释是一样的。）

（第三种同价的说法，`[`意思是"向前跳转到对应的`]`“，`]`意思是”向后跳转到对应的`[`指令的次一指令处，如果指针指向的字节非零。"）

Brainfuck程序可以用下面的替换方法翻译成C语言（假设`ptr`是`char*`类型）：
<table border="0" >
<tbody >
<tr >
Brainfuck
C
</tr>
<tr >

<td align="center" >`>`
</td>

<td >`++ptr;`
</td>
</tr>
<tr >

<td align="center" >`<`
</td>

<td >`--ptr;`
</td>
</tr>
<tr >

<td align="center" >`+`
</td>

<td >`++*ptr;`
</td>
</tr>
<tr >

<td align="center" >`-`
</td>

<td >`--*ptr;`
</td>
</tr>
<tr >

<td align="center" >`.`
</td>

<td >`putchar(*ptr);`
</td>
</tr>
<tr >

<td align="center" >`,`
</td>

<td >`*ptr =getchar();`
</td>
</tr>
<tr >

<td align="center" >`[`
</td>

<td >`while (*ptr) {`
</td>
</tr>
<tr >

<td align="center" >`]`
</td>

<td >`}`
</td>
</tr>
</tbody>
</table>


#### BF解释器


因为 BrainFuck 只有八种指令，并且没有关键字，也不允许自定义标识符，因此它的编译器实现起来非常简单，初学 C 语言不久的人都可以自己编出来，尽管在座的各位每人都可以自己编一个，不过为了引起大家的兴趣，我这里还是给出大家一个官方发布的版本。这个程序只有短短 50 多行，并且完全由 ANSI C 写成，因此你随便找个 C 语言编译器，把它编译一下。






<table cellpadding="0" cellspacing="0" border="0" >
<tbody >
<tr >

<td >


1




2




3




4




5




6




7




8




9




10




11




12




13




14




15




16




17




18




19




20




21




22




23




24




25




26




27




28




29




30




31




32




33




34




35




36




37




38




39




40




41




42




43




44




45




46




47




48




49




50




51




52




53




54




55




56




57




58

</td>

<td >





`#include <stdio.h>;`




`int` `p, r, q;`




`char` `a[5000], f[5000], b, o, *s=f;`




`void` `interpret(``char` `*c)`




`{`




` ``char` `*d;`




` ``r++;`




` ``while``( *c ) {`




` ``//if(strchr("<>;+-,.[]\n",*c))printf("%c",*c);`




` ``switch``(o=1,*c++) {`




` ``case` `'<'``: p--; ``break``;`




` ``case` `'>;'``: p++; ``break``;`




` ``case` `'+'``: a[p]++; ``break``;`




` ``case` `'-'``: a[p]--; ``break``;`




` ``case` `'.'``: ``putchar``(a[p]); ``fflush``(stdout); ``break``;`




` ``case` `','``: a[p]=``getchar``();``fflush``(stdout); ``break``;`




` ``case` `'['``:`




` ``for``( b=1,d=c; b && *c; c++ )`




` ``b+=*c==``'['``, b-=*c==``']'``;`




` ``if``(!b) {`




` ``c[-1]=0;`




` ``while``( a[p] )`




` ``interpret(d);`




` ``c[-1]=``']'``;`




` ``break``;`




` ``}`




` ``case` `']'``:`




` ``puts``(``"UNBALANCED BRACKETS"``), ``exit``(0);`




` ``case` `'#'``:`




` ``if``(q>;2)`




` ``printf``(``"%2d %2d %2d %2d %2d %2d %2d %2d %2d %2d\n%*s\n"``,`




` ``*a,a[1],a[2],a[3],a[4],a[5],a[6],a[7],a[8],a[9],3*p+2,``"^"``);`




` ``break``;`




` ``default``: o=0;`




` ``}`




` ``if``( p<0 || p>;100)`




` ``puts``(``"RANGE ERROR"``), ``exit``(0);`




` ``}`




` ``r--;`




` ``//        chkabort();`




`}`




`main(``int` `argc,``char` `*argv[])`




`{`




` ``FILE` `*z;`




` ``q=argc;`




` ``if``(z=``fopen``(argv[1],``"r"``)) {`




` ``while``( (b=``getc``(z))>;0 )`




` ``*s++=b;`




` ``*s=0;`




` ``interpret(f);`




` ``}`




`}`




</td>
</tr>
</tbody>
</table>






当然，如果你觉得用C语言来实现BrainFuck语言的解释器是对BrainFuck这种语言的一种侮辱的话，我们的BrainFuck社区是绝对不能容忍你有这种想法的。因为我们有一个使用100%纯brainfuck写成的一个编译器**awib**：[http://code.google.com/p/awib/](http://code.google.com/p/awib/)


#### Hello World



    
    ++++++++++[>+++++++>++++++++++>+++>+<<<<-]
    >++.>+.+++++++..+++.>++.<<+++++++++++++++.
    >.+++.------.--------.>+.>.


怎么？看不懂吗？下面是解释：

    
    +++ +++ +++ +           initialize counter (cell #0) to 10
    [                       use loop to set the next four cells to 70/100/30/10
        > +++ +++ +             add  7 to cell #1
        > +++ +++ +++ +         add 10 to cell #2
        > +++                   add  3 to cell #3
        > +                     add  1 to cell #4
        <<< < -                 decrement counter (cell #0)
    ]
    >++ .                   print 'H'
    >+.                     print 'e'
    +++ +++ +.              print 'l'
    .                       print 'l'
    +++ .                   print 'o'
    >++ .                   print ' '
    <<+ +++ +++ +++ +++ ++. print 'W'
    >.                      print 'o'
    +++ .                   print 'r'
    --- --- .               print 'l'
    --- --- --.             print 'd'
    >+.                     print '!'
    >.                      print '\n'


**相关链接**：



	
  * BF的官网：[http://www.muppetlabs.com/~breadbox/bf/](http://www.muppetlabs.com/~breadbox/bf/)。

	
  * BF的Wikipedia：[http://en.wikipedia.org/wiki/Brainfuck](http://en.wikipedia.org/wiki/Brainfuck)。




#### 其它另类语言


如果你要觉得BF已经很BT了，那么你就错了，下面这些程序语言更BT。

**WhiteSpace语言**

这是一种只用空白字符（空格，TAB和回车）编程的语言，而其它可见字符统统为注释。下面是它的一个示例：



什么？你什么也没有看见，这就对了，因为这正是这门语言的独特之处。访问下面这个链接查看[Hello,World示例](http://compsoc.dur.ac.uk/whitespace/hworld.ws)。记得按Ctrl+A来查看程序。

官网：[http://compsoc.dur.ac.uk/whitespace/index.php](http://compsoc.dur.ac.uk/whitespace/index.php)。

**LOLCODE语言**

LOLCODE是一种建立在高度缩写的网络英语之上的编程语言，一般来说如果一个人能理解这种网络英语就能在未经训练的情况下读懂LOLCODE程序源代码。下面是其Hello,World例程：

    
    HAI
    CAN HAS STDIO?
    VISIBLE "HAI WORLD!"
    KTHXBYE


翻译成中文就是：

    
    嗨
    我可以用 STDIO 么？
    显示一下 “HAI WORLD!”
    谢谢啊，再见




官网：[http://lolcode.com/](http://lolcode.com/)

**中文编程语言**

不要以为只有老外才那么BT，咱们中国也有自己的BT编程语言。

**中文Basic**
<table border="0" >
<tbody >
<tr >

<td >中文指令
</td>

<td >
</td>

<td >对应于的Applesoft BASIC
</td>
</tr>
<tr >

<td >10 卜=0
</td>

<td >
</td>

<td >10 Y=0
</td>
</tr>
<tr >

<td >20 入 水, 火
</td>

<td >
</td>

<td >20 INPUT E, F
</td>
</tr>
<tr >

<td >30 從 日 = 水 到 火
</td>

<td >
</td>

<td >30 FOR A = E TO F
</td>
</tr>
<tr >

<td >40 卜 = 卜+對數(日)
</td>

<td >
</td>

<td >40 Y = Y + LOG (A)
</td>
</tr>
<tr >

<td >50 下一 日
</td>

<td >
</td>

<td >50 NEXT A
</td>
</tr>
<tr >

<td >60 印 卜
</td>

<td >
</td>

<td >60 PRINT Y
</td>
</tr>
</tbody>
</table>
官网无法访问了，只能看看Wikipedia了：[http://en.wikipedia.org/wiki/Chinese_BASIC](http://en.wikipedia.org/wiki/Chinese_BASIC)

**中蟒语言（中文Python）**

下面的程序是不是很Cool？

    
    #!/usr/local/bin/cpython
    回答 = 读入('你认为中文程式语言有存在价值吗 ? (有/没有)')
    如 回答 == '有':
    写 '好吧, 让我们一起努力!'
    不然 回答 == '没有':
    写 '好吧,中文并没有作为程式语言的价值.'
    否则:
    写 '请认真考虑后再回答.'


官网：[http://www.chinesepython.org/](http://www.chinesepython.org/cgi_bin/cgb.cgi/home.html)

差不多了，该结束了，再次说明，这是一篇很严肃的文章。

(**全文完**)


