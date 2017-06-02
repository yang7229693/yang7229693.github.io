---
layout: post
title: 代码大全-读书笔记-第三部分
categories: reading-notes
excerpt: 代码大全第三部分读书笔记
tag: [代码大全, code complete, 读书笔记, 观后感]
comments: true
image:
  feature: pic-book-1.jpg
---

《代码大全》读书笔记目录

1. [第一部分 打好基础](http://www.whysodiao.com/reading-notes/code-complete-note-part-1/)
2. [第二部分 创建高质量代码](http://www.whysodiao.com/reading-notes/code-complete-note-part-2/)
3. [第三部分 变量](http://www.whysodiao.com/reading-notes/code-complete-note-part-3/)
4. [第四部分 语句](http://www.whysodiao.com/reading-notes/code-complete-note-part-4/)
5. [第五部分 代码改善](http://www.whysodiao.com/reading-notes/code-complete-note-part-5/)
6. [第六部分 系统考虑](http://www.whysodiao.com/reading-notes/code-complete-note-part-6/)
7. [第七部分 软件工艺](http://www.whysodiao.com/reading-notes/code-complete-note-part-7/)

第三部分主要是讲解变量相关的内容,围绕着使用变量的一般事项,变量命名以及数据类型等部分来展开的,这一部分的东西相比较前两部分,是比较基础也比较常见的内容.

#### 读书笔记

就近原则:把相关的操作放在一起,本质上还是便于阅读.

在可能的情况下使用`final`或者`const`.

攻击窗口（window of vulnerability）:介于同一变量多个引用点之间的代码. 因为一些新的代码可能会增加在窗口中,不当的修改这个变量,增加潜在的风险.

跨度（span）:衡量一个变量的不同引用点的靠近程度的一种方法. 把变量的引用点集中起来的主要好处是提高程序的`可读性`.

存活时间（live time）:一个变量存在期间所跨越的语句总数. 

保持较短的存活时间好处:

    减少攻击窗口
    能对自己代码有更准确的认识
    减少初始化错误的可能
    使代码更具可读性
    把相关代码片段重构为单独的子程序更容易
{: .notice}

如果用跨度和生存时间来考察全局变量,就会发现其跨度和生存时间都很长,避免使用全局变量.

减小作用域的一般原则:

    在循环开始之前再去初始化该循环里使用的变量
    直到变量即将被使用时再为其赋值
    把相关语句放到一起
    把相关语句组提取成单独的子程序
    开始时采用最严格的可见性,然后根据需要扩展变量的作用域
{: .notice}    


>“方便性”和“智力可管理性”两种概念的区别,归根结底是侧重写程序还是读程序之间的区别

应该避免使用具有隐含意义的比那辆,这种滥用在技术领域里被称为“混合耦合（hybrid coupling）”

>为变量命名时最重要的考虑事项是,该名字要完全,准确地描述出该变量所代表的的事物.一个好记的命名反映的通常都是问题,而不是解决方案.

当变量名的平均长度在10到16个字符的时候,调试程序所需花费的力气是最小的（Gorla, Benander 1990）

较长的名字适用于很少用到的变量或者全局变量,较短的名字则适用于局部变量或者循环变量.

好的变量名是提高程序可读性的一项关键要素,代码的阅读次数远远多于编写的次数.

#### 后感

这部分的内容比较基础,作者主要从人的角度来去讲解变量相关的知识点,如何让冰冷的代码对人更友好,也是很多计算机学者专家研究的方向.

写出来的代码,避免不了维护,所以也就避免不了阅读,如果产生歧义,就可能造成潜在的风险,增加不必要的成本,而以前的老计算机科学家,他们就会去研究这些在我们现在看来,感觉很常识性的一些东西,他们会从科学的角度去统计分析这么做所产生的效果,这也是他们为什么会被称为科学家,而我们只是程序员的缘故.

