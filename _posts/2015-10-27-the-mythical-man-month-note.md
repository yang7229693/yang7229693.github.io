---
layout: post
title:  "《人月神话》读书笔记"
date:  2015-10-27
categories: reading notes
featured_image: /images/notes_cover.jpg
---

###《人月神话》读书笔记


今年看的书比以往都要多，决定从现在开始对看过的书籍做下笔记，方便以后回顾，以后也会抽出时间对今年之前看过的书籍进行一个回顾，整理出一些笔记。

---

在众多软件项目中，`缺乏合理的时间进度`是造成项目滞后的最主要原因。导致这种普遍性灾难的原因有以下五点：

	1. 对估算技术缺乏有效的研究
	2. 错误的将进度与工作量相互混淆
	3. 不会有耐心持续的对项目进行估算
	4. 对进度缺少跟踪和监督
	5. 当出现进度偏移时，增加人力


编程人员都是乐观主义者，我们会对进度安排做假设：`一切都将运作良好，每一项人物仅仅花费它所“应该”话费的时间。`

第二个谬误的思考方式是在估计和进度安排中使用的工作量单位：人月。

	人数和时间的互换仅仅使用于以下情况：某个人物可以分解给参与人员，并且他们之间不需要相互的交流。
	
沟通所增加的负担主要由两部分组成，`培训和相互的交流`

`系统测试进度`的安排常常是编程中最不合理的部分。

对于软件人物的进度安排，作者多年的`经验法则`：

	1/3 计划
	1/6 编码
	1/4 构建测试和早期系统测试
	1/4 系统测试，所有的构件已完成
	
`Brooks法则`

	向进度落后的项目中增加人手，只会使进度更加落后
	

项目的时间依赖于顺序上的限制，人员的数量依赖于单个子任务的数量。

效率高和效率低的实施者之间具体差别非常大，经常达到了`数量级`的水平。

需要写作沟通的人员的数量影响着开发成本，因为成本的主要组成部分是相互的沟通和交流，以及更正沟通不当所引起的不良结果。

整个系统必须具备`概念上的完整性`，要有一个系统结构师从上至下的进行所有的设计。

为了反映一系列`连贯`的设计思路，宁可省略一些不规则的特性和改进，也不提倡独立和无法整合的系统，哪怕它们其实包含着许多很好的设计。

功能与理解上复杂程度的比值才是系统设计的最终测试标准。

对于给定级别的功能，能用`最简洁`和`直接`的方式来指明事情的系统是最好的。

不能与系统基本概念进行整合的良好想法和特色，最好放到一边，不予考虑。如果出现了很多非常重要但不兼容的构想，就应该抛弃原来的设计，对不同基本概念进行合并，在合并后的系统上重新开始。

结构师只能建议，不能支配。

项目所有的文档都必须是该结构的一部分。这包括目的、外部规格说明、接口说明、技术标准、内部说明和管理备忘录。

减少交流的方法是`人力划分`和`限定职责范围`。

`交流和组织`的技能需要管理者仔细考虑，相关经验的积累和能力的提高同软件技术本身一样重要。

培养开发人员`从系统整体出发`、`面向用户的态度`是软件编程管理人员最重要的职能。

项目经理可以做两件事来帮助团队取得良好的时间-空间折衷：

	1. 确保他们在编程技能上的得到培训，而不仅仅是依赖他们自己掌握的知识和先前的经验。
	2. 认识到编程需要技术的积累，需要开发很多公共单元构建。
	
数据的表现形式是编程的根本。

项目经理的基本职责是使每个人都向着相同的方向前进，所以他的主要工作是`沟通`，而不是做出决定。

对于一个广泛使用的程序，其维护总成本通常是开发成本的40％或更多。


程序维护中的一个基本问题是
	
	缺陷修复总会以（20－50）%的机率引入新的bug。
	
系统软件开发是减少混乱度（减少熵）的过程，所以它本身是处于亚稳态的。软件维护是提高混乱度（增加熵）的过程，即使是最熟练的软件维护工作，也只是放缓了系统退化到非稳态的进程。

流程图被鼓吹的程度远大于它们的实际作用。

不是因为软件发展的太慢，而是因为`计算机硬件发展的太快`。

软件开发中困难的部分是规格化、设计和测试这些概念上的结构，而不是对概念进行表达和对实现逼真程度进行验证。

现代软件系统中无法规避的内在特性
	
	复杂度、一致性、可变形和不可见性。
	
如何培养杰出的设计人员：

	1. 尽可能早的、有系统的识别顶级的设计人员
	2. 为设计人员指派一名职业导师，负责他们技术方面的成长，仔细的为他们规划职业生涯
	3. 为每个方面制定和维护一份职业计划，包括与设计大师的、经过仔细挑选的学习过程、正式的高级教育以及短期的课程，所有这些都穿插在设计和技术领导能力的培养安排中
	4. 为成长的设计人员提供相互交流和学习的机会
	

####读书感悟
作者是在1975年写这本书的，今年是第40个年头，里面的一些通用性的东西并未发生太多改变，无怪乎作者在20周年纪念版序中说不对原内容做任何修订，作者在书中强调的非常多的一点：
	
	文档对于项目的重要性

另一个就是度量单位`人月`的神话性。我看的是20周年纪念版，书后面有很大一部分章节是在讨论一个问题，未来十年没有任何单独的软件工程进展可以使软件生产率有数量级的提高（1986年的版本）。

如果把这个时间限制取消掉，未来是否会出现能提高数量级的生产率的软件工程呢？

我想这个答案肯定是 `会`

读这本书并没有豁然开朗的感觉，或许本身没有大型项目的管理经验，也或许里面提到的大部分东西平时早有耳濡目染，对于想看这本书但是时间比较紧凑的人来说，推荐只看书中“《人月神话》的观点：是或非？”这一章节，可以看出一些简略吧。

希望以后有机会走上管理岗位再来review下这本书籍，相信到时候肯定会有不同的看法和见解吧～


