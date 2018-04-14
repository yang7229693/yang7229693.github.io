---
layout: post
title: 喜闻乐见之Activity的launchMode
categories: reading-notes
excerpt: Activity启动方式
tag: [Android, Activity, 启动方式, launchMode]
comments: true
image:
  feature: pic-book-1.jpg
---

> launchMode，通俗点说，就是定义了Activity应该如何被launch的。那么这几种模式的区别以及应用场景，会有何不同呢？谷歌是基于什么原因设计这几种模式的呢？这几种模式背后的工作原理是什么呢？

## 任务和返回栈

在讲解launchMode之前，先说说任务（Task）和返回栈（Back Stack，有些译作回退栈、任务栈）这两个概念。

> A task is a collection of activities that users interact with when performing a certain job. The activities are arranged in a stack—the back stack—in the order in which each activity is opened.

任务是指当完成一个特定的工作时与用户交互的一系列Activity。这些Activity按照打开的顺序存放在一个栈中，即返回栈。

通过定义可以知道，Activity会被按照打开的顺序存放。不难猜想，这种存放方式，是为了方便回退操作，也就不难解释为什么要用栈去存放。

当用户点击启动app的时候，这个app的返回栈就会跑到前台，如果这个返回栈不存在的话，就会创建一个。当前Activity启动另一个Activity的时候，新的Activity就会入栈，在栈顶。如果用户点击返回按钮，当前的Activity就会出栈并销毁，之前的Activity就会被resume，如果栈为空，就会被销毁掉。栈中的Activity永远都不会被重新排序。

返回栈根据是否在前台，可以分为在前台显示的返回栈，和置于后台的返回栈。其中，置于后台的返回栈中所有的Activity都处于stop状态，用户可以手动的去切换前后台的返回栈状态。

> 当系统内存不足时，系统会优先销毁处于后台的Activity。那么问题来了。后台销毁Activity的优先级是怎样的呢？是将一个返回栈中的Activity都销毁了过后，再去销毁另一个，还是说，只是单纯的按照Activity来销毁回收呢？

### 任务管理

任务以及返回栈的管理，可以通过一系列的参数设置来进行，包括我们本文讲解的launchMode，也是任务管理的一种方式。

#### taskAffinity

TaskAffinity即任务相关性，标识一个Activity所需要的返回栈的名字。默认情况下是包名。设置了相同taskAffinity属性的Activity会被放进同一个栈中。一个返回栈的相关性（affinity）是由这个栈的根Activity的相关性（affinity）决定的。

> taskAffinity属性主要与singleTask或allowTaskReparenting结合使用，在其他情况下，这个属性没有作用。这是为什么呢？


#### allowTaskReparenting

它的主要作用是Activity的迁移，从一个栈迁移到另一个栈，这个迁移跟Activity的taskAffinity有关。

#### clearTaskOnLaunch

这个属性用来清除回退栈中除了根Activity的所有Activity，只对根Activity起作用。当设置为true时，每次重新进入app，只会看到根Activity。

#### finishOnTaskLaunch

这个属性与clearTaskOnLaunch相反，它是将本Activity移除出去，而不影响其他的Activity。

#### alwaysRetainTaskState

这个属性的作用是保存返回栈的状态，只对根Activity起作用。正常情况下，系统清理一个返回栈，会将根Activity之上的所有Activity都清除掉。设置该属性后，系统会保存当前的状态。

## 启动模式

启动模式主要的作用是什么呢？根据上面对任务及返回栈的介绍，它的作用是定义，一个新的Activity实例如何与当前的任务相关联。它本身是任务的管理方式。

启动模式有两种定义方式，manifest里定义和intent flag的方式。一种是类似配置式的，一种是代码层面的。可以大致推测，肯定是带么层面的优先级高一些，但是代码方式劣处就是不启动Activity就无法设置。Android中这种一般提供动态以及静态方式的，套路都大致相同，一些区别各种优劣等。

其中manifest设置与intent flag中都包含对方没有的方式。这也是两者的一个区别。

### launchMode

此处的launchMode专指Activity的launchMode属性。其中有四种方式，这四种方式想必大家也都很清楚了，在这里我不详细展开了。

#### standard

最常见的一种模式，Activity的默认模式，每次启动该模式的Activity，都会被重新创建，可以从属不同的任务，也可以在一个任务中被创建多次。

它的应用场景特别广泛， 一般不是特殊需求的话，都会去使用这种模式。

#### singleTop

如果在当前任务的栈顶，系统会调用Activity的onNewIntent()方法而不是重新创建一个新的实例。当用户点击返回键时，当前Activity会被出栈，而不是会退到onNewIntent()之前的状态。

它的应用场景也有一些。例如搜索页面，每次打开，搜索一些结果，点击详情页面，然后继续搜索。在比方说，通过通知打开的页面，如果该页面存在，则更新，如果不存在，则创建。

#### singleTask

该模式只允许系统中存在一个该Activity的实例，如果当前实例不存在，则创建，如果已经存在，则将该实例之上的Activity全部出栈，走onNewIntent()。

singleTask适合作为程序入口点，当通过其他方式调用app时候，不会反复创建主页面。例如一般情况下的MainActivity，其他app调用的时候。

#### singleInstance

这种模式与singleTask十分类似，区别在于，持有该Activity的任务中只能包含一个Activity即它本身。

singleInstance适合需要与程序分离开的页面，例如闹钟的响铃界面，与闹钟的设置相分离。再例如系统的拨号界面。

### Intent flags

此处讨论的是通过代码方式进行设置，常见的有如下三种方式。

#### FLAG_ACTIVITY_NEW_TASK

使用一个新的返回栈来启动Activity，跟上面讨论的singleTask类似

#### FLAG_ACTIVITY_SINGLE_TOP

跟上面讨论的singleTop类似

#### FLAG_ACTIVITY_CLEAR_TOP

这种方式是上面讨论的launchMode中不存在的，它与singleTop的区别是，当已存在该实例了，会将它之上的Activity都出栈。

它经常与FLAG_ACTIVITY_NEW_TASK组合使用，可以达到singleTask的作用。

## 回到问题

> 几种模式的区别以及应用场景，会有何不同呢？

答案见上面关于launchMode

> 谷歌是基于什么原因设计这几种模式的呢？

关于这个问题，我们先倒着来推理，即从使用场景去考虑，一般状况下，我们打开一个页面，不在意是否是唯一，这个是最常见的需求，因此有了standard模式，这种也是默认的模式。当我们用搜索页面，当最顶层是搜索页面的时候，我不希望再打开一个搜索页面，于是有了singleTop模式。当从其他App调用我们的app的时候，我只希望只显示一个主页面时，于是有了singleTask。关于singleInstance模式，则是希望与当前的页面分离。

但是，我觉得谷歌并不能列举出所有的场景，例如，我希望打开一个页面，记录当前的路径，例如a->b->c，这种场景下，四种模式里面没有包含。

如果从正面去推导的话，几种启动模式是任务及返回栈的管理。根据在栈中的状态，大致可以分为如下几类：

1. 最常见的出栈入栈（standard）
2. 当前栈中唯一（singleTask）
3. 全局唯一（singleInstance）
4. 栈顶唯一（singleTop）

是不是很明晰了，有没有其他的出现形式？肯定有的，例如栈底唯一，栈中唯一。但是这种方式可以等同于当前栈中唯一啊。

我们是否可以推导出，谷歌是根据唯一性，来将启动模式分为这几种呢？intent flags则作为辅助的一些操作，例如部分出栈等等。当然这些也只是我的推测，不一定准确，哈哈。

> 这几种模式背后的工作原理是什么呢？

见任务及返回栈

> 内存回收的方式，是以Activity还是以任务作为基准回收？

目前已知的状况时，如果返回栈置于后台，当内存不足的时候，如果不设置alwaysRetainTaskState属性的话，会将除了根Activity的所有Activity销毁掉。可以确定是以返回栈为基准来进行回收。

> taskAffinity属性为什么与singleTask一起使用才生效？

可以将Activity的launchMode根据是否在栈中唯一分为两类

1. standard、singleTop
2. singleTask、singleInstance

第一类因为其唯一性，肯定是与taskAffinity不兼容的。singleInstance创建的栈中只能包含本身，默认情况下都会单独创建一个栈，指定与否都会单独创建，因此设置没有意义。而singleTask则是当前栈中唯一，适合作为根Activity，创建一个新的栈，这也是为什么taskAffinity只能对根Activity起作用的缘故。

## 最后

我写的内容不一定正确，一些问题的解释也是根据我看到的资料来推到出来的，例如Activity为什么会有四种启动模式，如果大家有准确地答案，希望告知。另外，文中错误的地方，也希望指正。

最后，感谢大家的浏览，希望对您有所帮助。