---
layout: post
title: 代码大全-读书笔记-第四部分
categories: reading-notes
excerpt: 代码大全第四部分读书笔记
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


第四部分主要是语句相关的讲解,作者从代码组织层面,条件语句,控制循环,表驱动,一般控制问题等方面,讲解了语句过程中应该注意的事项.

#### 读书笔记

专家们认为自上而下的顺序对提高可读性最有帮助.

如果你所使用的语言支持,就把if else语句串替换成其他结构.

带退出循环结构要比其他循环结构更接近于人类思考迭代型控制.

把嵌套限制在`3层以内`,研究表明,当嵌套超出3层以后,程序员对循环的理解能力会极大地降低.（Yourdon 1986）

goto是一个充满争议的语句,如果能用的很好的话,就去使用,一般不建议使用.

表驱动法是一种编程模式（scheme）--从表里面查找信息而不使用逻辑语句.

使用表驱动法必须解决的两个问题:

    怎样从表中查询条目
    在表里面存些什么
{: .notice}

表里面查询记录的方法:

    直接访问
    索引访问
    阶梯访问
{: .notice}

用决策表代替复杂的条件判断

程序的复杂度在很大程度上决定了理解程序所需要花费的精力.

应用程序的复杂度是由它的控制流来定义的（McCabe 1976）.

将复杂度降低到最低水平是编写高质量代码的关键.

#### 后感

这一部分也是蛮基础性的一些东西,上一部分从语句的部分---变量,来讲解如何写出易于阅读的代码,以及为什么要这么做,这一部分则是通过语句层面,从语句的组织顺序,结构化编程,控制语句的使用等方面,讲解该如何写出易于理解的高质量代码.

不由得感叹一下,现在的编程语言已经比较高级了,但是最基本的一些东西没有变化,编程语言越来越傻瓜,让编程人员考虑的东西越来越少,最后埋没在业务之中.而我们应该跳出业务,去理解本质,被业务而忙是无意义的忙,我觉得对自己来说几乎是没有太多价值的.

