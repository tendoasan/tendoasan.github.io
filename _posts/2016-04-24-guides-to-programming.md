---
layout: post
title: 编程入门指南
---

## 启蒙

1. [笨方法学Python](http://learnpythonthehardway.org/book/)
2. [MIT计算思维和数据科学导论](http://www.xuetangx.com/courses/MITx/6_00_2x/2014_T2/about)：MOOC 是学习编程的一个有效途径。虽然该课程的教学语言为Python，但作为一门优秀的导论课，它强调学习计算机科学领域里的重要概念和范式，而不仅仅是教你特定的语言。如果你不是科班生，这能让你在自学时开阔眼界；课程内容：计算概念，Python编程语言，一些简单的数据结构与算法，测试与调试。
**支线任务**：
	- [Python核心编程](https://book.douban.com/subject/3112503/)

3. [哈佛大学公开课:计算机科学cs50](http://open.163.com/special/opencourse/cs50.html)：同样是导论课，但这门课与MIT的导论课互补。教学语言涉及 C, PHP,  JavaScript + SQL, HTML + CSS，内容的广度与深度十分合理，还能够了解到最新的一些科技成果，可以很好激发学习计算机的兴趣。
**支线任务**：
	- [编码:隐匿在计算机软件和硬件背后的语言](https://book.douban.com/subject/4822685/)
	- [C语言编程:一本全面的C语言入门教程](https://book.douban.com/subject/1786294/)
	- (**可选**)[Hacker's Delight](https://book.douban.com/subject/1784887/)

---

## 入门

1. [计算机程序的构造与解释(SICP)](https://mitpress.mit.edu/sicp/full-text/book/book.html)：在阅读SICP之前，你也许能通过调用几个函数解决一个简单问题。但阅读完SICP之后，你会学会如何将问题抽象并且分解，从而处理更复杂更庞大的问题，这是编程能力巨大的飞跃，这会在本质上改变你思考问题以及用代码解决问题的方式。此外，SICP的教学语言为 Scheme，可以让你初步了解函数式编程。更重要的是，他的语法十分简单，你可以很快学会它，从而把更多的时间用于学习书中的编程思想以及复杂问题的解决之道上。
	*辅助资源*：
	- [Udacity CS212 Design of Computer Program](https://www.udacity.com/course/design-of-computer-programs--cs212)：由Google研究主管Peter Norvig 主讲，教学语言为 Python，内容有一定难度。
	- [How to Design Programs, Second Edition](http://www.ccs.neu.edu/home/matthias/HtDP2e/Draft/index.html)：HtDP的起点比SICP低，书中的内容循循善诱，对初学者很友好，如果觉得完成SICP过于困难，可以考虑先读一读HtDP。
	- [UC Berkeley SICP授课视频](http://webcast.berkeley.edu/playlist#c,d,Computer_Science,EC3E89002AA9B9879E)，以及SICP 的两位作者给 Hewlett-Packard 公司员工培训时的录像，该视频的[中文化项目](https://github.com/DeathKing/Learning-SICP/)。
	- [Composing Programs](http://composingprograms.com/)：一个继承了SICP思想但使用Python作为教学语言的编程导论（其中包含了一些小项目）。
	- [SICP 解题集](http://sicp.readthedocs.org/en/latest/index.html)：对于书后的习题，作为初学者应**尽力并量力**完成。
	- **练手项目**：[Mega Project List](https://github.com/karan/Projects/)
2. [计算机系统要素:从零开始构建现代计算机](https://book.douban.com/subject/1998341/)：这本书会教会你从最基本的 Nand 门开始构建计算机，直到俄罗斯方块在你的计算机上顺利运行。 它会贯穿你的整个编程入门阶段，你入门阶段的目标就是坚持完成这本书的所有项目（包括一个最简的编译器与操作系统）。为了完全搞定这本书，为了继续打好根基。为了将来的厚积薄发，在下面这几个方面你还要做足功课（注意：下面的内容没有绝对意义上的先后顺序）。

### 计算机系统基础

1. [深入理解计算机系统(CSAPP)](https://book.douban.com/subject/5333562/)：这本书只是 CMU的**计算机系统导论**的教材而已。CMU的计算机科学专业相对较偏软件，该书就是从一个程序员的视角观察计算机系统，以**程序在计算机中如何执行**为主线，全面阐述计算机系统内部实现的诸多细节。如果觉得枯燥可以参考如下的一门Coursera上的MOOC（`软硬件接口`）。完成这本书后，你会具备坚实的系统基础，也具有了学习操作系统，编译器，计算机网络等内容的先决条件。当学习更高级的系统内容时，翻阅一下此书的相应章节，同时编程实现其中的例子，一定会对书本上的理论具有更加感性的认识，真正做到经手的代码，从上层设计到底层实现都了然于胸，并能在脑中回放数据在网络->内存->缓存->CPU的流向。
	*辅助资源*：
	- [软硬件接口](https://www.coursera.org/course/hwswinterface)：这门课的内容是 CSAPP 的一个子集，但是最经典的实验部分都移植过来了。
	- [C程序设计语言](https://book.douban.com/subject/1139336/)：回顾一下C语言的知识。

2. [UNIX编程环境](https://book.douban.com/subject/1033144/)：在实践中，开始熟悉命令行界面，配置文件。并且在开发中逐渐脱离之前使用的IDE，学会使用Vim或Emacs（或者最好两者都去尝试）。
3. [UNIX编程艺术](https://book.douban.com/subject/1467587/)
4. [折腾UN*X](http://heather.cs.ucdavis.edu/~matloff/unix.html)

### 数据结构与算法基础

1. [算法导论](https://book.douban.com/subject/1885170/)：读第一遍的时候考虑跳过习题和证明。
	*辅助资源*：
	- [数据结构与算法分析](https://book.douban.com/subject/1139426/)
2. [算法设计与分析：PART1&PART2](https://www.coursera.org/course/algo)： Stanford 开的算法课，不限定语言，两个部分跟下来算法基础基本就有了。
3. [麻省理工学院公开课：算法导论](http://open.163.com/special/opencourse/algorithms.html)
4. **入门阶段还要注意培养使用常规算法解决小规模问题的能力，配合SICP部分可以阅读**：
	- [编程珠玑](https://book.douban.com/subject/3227098/)
	- [程序设计实践](https://book.douban.com/subject/1173548/)

### 编程语言基础

1. [C++ Primer](https://book.douban.com/subject/25708312/)
	*可选进阶*：
	- 高效使用：[Effective C++](https://book.douban.com/subject/1842426/)
	- 深入了解：[深入探索C++对象模型](https://book.douban.com/subject/1091086/)，[C++ Templates](https://book.douban.com/subject/2378124/)
	- 深入反思：[The Design and Evolution of C++](https://book.douban.com/subject/1456860/)，看这本书可以让你选择是成为守夜人还是守日人。
2. **学习资源**：
	- [程序设计语言-实践之路](https://book.douban.com/subject/2152385/)：CMU编程语言原理的教材，程序语言入门书，现在就可以看，会极大扩展你的眼界，拉开你与普通人的差距。
	- [程序设计语言MOOC](https://www.coursera.org/course/proglang)：课堂上你能接触到极端FP（函数式）的SML，中性偏FP的Racket，以及极端OOP（面向对象）的Ruby，并学会问题的FP分解 vs OOP分解、ML的模式匹配、Lisp宏、不变性与可变性、解释器的实现原理等，让你在将来学习新语言时更加轻松并写出更好的程序。
	- [Programming Languages:Building a Web Browser](https://www.udacity.com/course/programming-languages--cs262)：教你写一个简单的浏览器——其实就是一个javascript和html的解释器，完成后的成品还是很有趣的；接下来，试着完成一个之前在SICP部分提到过的项目：用Python完成一个[Scheme Interpreter](http://inst.eecs.berkeley.edu/~cs61a/fa13/proj/scheme/scheme.html)。

---

## 必要的技能

1. **学好英语**，*参考*：[把你的英语用起来](https://book.douban.com/subject/3748247/)
2. [学会提问](https://book.douban.com/subject/1504957/)，学会搜索引擎的[高级搜索](https://support.google.com/websearch/answer/35890?hl=zh-Hans)，去[Stack Overflow](http://stackoverflow.com/)或[知乎](https://www.zhihu.com/)提问前，读读:[what have you tried?](http://mattgemmell.com/what-have-you-tried/)
3. **不要做一匹独狼**：尝试搭建个人网站，像[这样](http://ezyang.com/)，学习[Markdown](https://zh.wikipedia.org/wiki/Markdown)与[LaTex](https://zh.wikipedia.org/wiki/LaTeX)，在Blog上记录自己的想法，订阅编程类博客：[Joel on Software](http://www.joelonsoftware.com/)，[Peter Norving](http://www.norvig.com/index.html)，[Coding Horror](http://blog.codinghorror.com/)

---

## 小结

1. **编程的入门期间你会遇到无数的困难，当你碰壁时试着尝试[费曼技巧](https://www.quora.com/How-can-you-learn-faster/answer/Acaz-Pereira)**：
	- Step 1. 选择你需要理解的概念，用一张白纸写下来；
	- Step 2. 假装你需要向小白解释这一概念，试着将需要阐述的内容写下来；
	- Step 3. 如果卡壳了，回到书本，重新理解原材料，直到能写纸上阐述清楚；
	- Step 4. 用自己的语言重新阐述概念，试着简化语言或者创造一个类推概念来更好地理解它。
2. [程序员必读书单](http://stackoverflow.com/questions/1711/what-is-the-single-most-influential-book-every-programmer-should-read)(前2)：
	- [代码大全](https://book.douban.com/subject/1477390/?i=0)：不管是对于经验丰富的程序员还是对于那些没有受过太多的正规训练的新手程序员，此书都能用来填补自己的知识缺陷。对于入门阶段的新手们，可以重点看看涉及变量名，测试，个人性格的章节。
	- [程序员修炼之道：从小工到专家](https://book.douban.com/subject/1152111/)：程序员入门书，终极书。有人称这本书为代码小全：从 DRY 到 KISS，从做人到做程序员，这本书教给了你一切，你所需的只是遵循书上的指导。

---

## 后记

如果你能设法完成以上的所有任务，恭喜你，你已经真正实现了编程入门。这意味着你在之后更深入的学习中，不会畏惧那些学习新语言的任务，不会畏惧那些**复杂**的API，更不会畏惧学习具体的技术，甚至感觉很容易。当然，为了掌握这些东西你依旧需要大量的练习，腰还是会疼，走路还是会费劲，一口气也上不了5楼。但我能保证你会在思想上有巨大的转变，获得极大的自信，看老师同学和 [CSDN](http://www.csdn.net/) 的眼光会变得非常微妙，虽然只是完成了编程入门，但已经成为了程序员精神世界的高富帅。不，我说错了，即使是高富帅也不会有强力精神力，他也会怀疑自己，觉得自己没钱就什么都不是了。但总之，你遵循指南好好看书，那就会体验**会当凌绝顶**的感觉。

作者：@[萧井陌](https://www.zhihu.com/people/xiao-jing-mo), @[Badger](https://www.zhihu.com/people/badger23)

- 2015年06月07日 v1.4 更新

- 自由转载-非商用-非衍生-保持署名 \| Creative Commons BY-NC-ND 3.0

- **[CoCode](http://cocode.cc/)**：**一个让大家学习、成长、相聚并获得乐趣的技术社区**。

- **答疑邮箱**：xiao.gua@outlook.com  (@[萧井陌](https://www.zhihu.com/people/xiao-jing-mo))

整理：@[tendoasan](https://github.com/tendoasan)
