---
layout: post
title: 喜闻乐见之Activity生命周期
categories: reading-notes
excerpt: Activity生命周期解析
tag: [Android, Activity, 生命周期, LifeCycle]
comments: true
image:
  feature: pic-book-1.jpg
---

> Activity的生命周期，对于Android开发者来说，再熟悉不过了。但是我们接触到的资料，绝大部分都只是谈了一些表面上的东西，例如各个回调的顺序等等。本文试图换个角度来讲解，也希望对各位读者有所帮助。


### 生命周期

首先附上一张大家都熟悉的不能再熟悉的图了

![](/images/activity_lifecycle_1.png)

对于各个流程的回调，想必大家早已熟记于心了，对于单个Activity来说，完全没问题，复杂点的，不就是转屏嘛？能有啥，好的，下面几个问题，麻烦大家先思考下。

1. FirstActivity启动了SecondActivity，整个流程是怎样的？
2. setContentView如果放在onStart或者onResume中，会有什么问题吗？
3. onPause中可以保存状态，为什么还要onSaveInstanceState，onCreate中有恢复机制，为什么还需要onRestoreInstanceState？
4. 如何判断一个Activity真正可见

### 来自官方

在回答这些问题之前，先来回顾下Activity的各个阶段，下面的英文部分出自[Google Android官方文档](https://developer.android.com/reference/android/app/Activity.html?hl=zh-cn)。

##### onCreate

> Called when the activity is first created. <font color='red'>This is where you should do all of your normal static set up: create views, bind data to lists, etc.</font> This method also provides you with a Bundle containing the activity's previously frozen state, if there was one. Always followed by onStart().

直接看重点，onCreate是用来干啥的，创建view、绑定data的地方。是初始化Activity的地方，setContentView以及获取控件都应该放在这里去做。

##### onStart

> Called when the activity is <font color='red'>becoming visible to the user.</font>
Followed by onResume() if the activity comes to the foreground, or onStop() if it becomes hidden.

onPause会被调用，是在Activity正在对用户变得可见的时候。也就是说这个时候，对于用户来说不是真正的可见，也不可去响应用户的输入。

##### onResume

> Called when the activity <font color='red'>will start interacting with the user.</font> At this point your activity is at the top of the activity stack, with user input going to it.
Always followed by onPause().

onResume是在将要可以产生交互的时候被调用的，也就是说，还不能响应用户的输入操作。在这个阶段，Activity已经是处在栈顶了。

##### onPause

> Called when the system is about to start resuming a previous activity. <font color='red'>This is typically used to commit unsaved changes to persistent data, stop animations and other things that may be consuming CPU, etc.</font> Implementations of this method <font color='red'>must be very quick</font>  because the next activity will not be resumed until this method returns.
Followed by either onResume() if the activity returns back to the front, or onStop() if it becomes invisible to the user.

onPause可以用来干啥，停止动画或者那些占用CPU操作的地方，但是在onPause里面进行的操作，不能够太久，应该very quick，因为下一个Activity需要等onPause结束后才会被调起。这个very quick的时间，应该是多少了？这个在ActivityManagerService中有定义，不能超过500ms。如果在onPause里面处理时间超过5秒的话，会不会出现ANR呢？


    // How long we wait until giving up on the last activity to pause.  This
    // is short because it directly impacts the responsiveness of starting the
    // next activity.
    static final int PAUSE_TIMEOUT = 500;
{: .notice}
    
##### onStop

> Called when the activity is <font color='red'>no longer visible to the user</font>, because another activity has been resumed and is covering this one. This may happen either because a new activity is being started, an existing one is being brought in front of this one, or this one is being destroyed.
Followed by either onRestart() if this activity is coming back to interact with the user, or onDestroy() if this activity is going away.

onStop的描述很简单，Activity对用户不可见时调用，但是文档后面Killable一栏显示的是YES，也就是说，onStop有可能会被强制结束掉而不完整的执行。

##### onDestroy

> The final call you receive before your activity is destroyed. This can happen either because the activity is finishing (someone called finish() on it, or because the system is temporarily destroying this instance of the activity to save space. <font color='red'>You can distinguish between these two scenarios with the isFinishing() method.</font>

onDestroy是在Activity要被释放掉时调用的，但是这个被释放，有主动的（手动去调用finish()）和被动的（系统回收），可以通过isFinishing来区分这两种场景。

##### onSaveInstanceState

> This method is called <font color='red'>before an activity may be killed</font> so that when it comes back some time in the future it can restore its state. 

> <font color='red'>The default implementation takes care of most of the UI per-instance state</font> for you by calling onSaveInstanceState() on each view in the hierarchy that has an id, and by saving the id of the currently focused view (all of which is restored by the default implementation of onRestoreInstanceState(Bundle)). 

> If called, <font color='red'>this method will occur after onStop() for applications targeting platforms starting with P. For applications targeting earlier platform versions this method will occur before onStop() and there are no guarantees about whether it will occur before or after onPause().</font>

从官网的三段介绍，可以看出几点

首先，onSaveInstanceState的调用时机，它是在当前Activity可能会被杀死的时候才会触发，如果用户主动的去杀死当前Activity，这个方法是不会被调用的。

其次，它的作用主要是用于存储UI层面的状态，不同于onPause。

最后，这个方法的调用时机，在不同的系统中不同，从Android P开始，它是在onStop之后调用的，在之前的系统中，则是在onStop之前调用的。但是是否发生在onPause前后，则看具体情况。

##### onRestoreInstanceState

> This method is called after onStart() when the activity is being re-initialized from a previously saved state, given here in savedInstanceState. Most implementations will simply use onCreate(Bundle) to restore their state, but it is sometimes convenient to do it here after all of the initialization has been done or to allow subclasses to decide whether to use your default implementation. <font color='red'>The default implementation of this method performs a restore of any view state that had previously been frozen by onSaveInstanceState(Bundle).</font>

> This method is called <font color='red'>between onStart() and onPostCreate(Bundle).</font>

可以看出，这个方法的主要作用，是恢复在onSaveInstanceState中保存的状态。它的调用时机在onStart和onPostCreate之间。

### 回到问题

读了谷歌官方文档，是否会发现一些平时开发中没有注意到的点呢？接下来我们回到上面的几个问题。

> FirstActivity启动了SecondActivity，整个流程是怎样的？

在正常情况下，按照文档说的，首先会去调用FirstActivity的onPause方法，在onPause方法结束完毕后，然后调用SecondActivity的onCreate、onStart、onResume，最后是FirstActivity的onStop。这个是根据文档来推断的，我们实际跑一下程序。

    I/FirstActivity: =====FirstActivity=====onPause
    I/FirstActivity: =====FirstActivity=====onWindowFocusChanged
    I/SecondActivity: =====SecondActivity=====onCreate
    I/SecondActivity: =====SecondActivity=====onStart
    I/SecondActivity: =====SecondActivity=====onResume
    I/SecondActivity: =====SecondActivity=====onWindowFocusChanged
{: .notice}


从打印的log可以看出，跟推断的一样，首先调用的是FirstActivity的onPause，然后才是resume SecondActivity。

> setContentView如果放在onStart或者onResume中，会有什么问题吗？

从官方文档来看，也只是说应该把setContentView放在onCreate中，并没有说必须放在这里，所以应该也不会有显示以及调用的问题吧。还是跑一下程序看看，分别将setContentView以及设置点击事件放在onStart以及onResume中，跑了下程序，没有出现显示问题。但是这个仅仅是显示上的问题，会不会存在效率的问题呢？我们来打印一下时间，以调用onCreate到onWindowFocusChanged之间的时间，作为Activity加载时间，来进行对比

    I/SecondActivity: =====SecondActivity=====Load Time:56
    I/SecondActivity: =====SecondActivity=====Load Time:57
    I/SecondActivity: =====SecondActivity=====Load Time:57
{: .notice}


三次时间几乎一样的，也就是说，如果只是单丛初次启动的效率来说，在三个地方去进行setContentView是没有任何差别的。但是为甚么官方说应该放在onCreate里面去处理了，这是因为，onCreate在正常状况下，只会被调用一次，而onStart以及onResume都会被调用多次，放在这里面去做的话，在onResume的过程，会增加额外的耗时。

另外，由于onRestoreInstanceState是在onStart之后才调用的，如果将setContentView放在onResume的话，可能会产生问题。

>  onPause中可以保存状态，为什么还要onSaveInstanceState，onCreate中有恢复机制，为什么还需要onRestoreInstanceState？

根据官方文档来看，onPause主要的作用是停止动画以及一些耗CPU的操作，可以用于保存状态。onSaveInstanceState主要作用是对UI进行状态保存。两者的侧重点不同，onPause也可以保存UI的状态，但是，onPause设计的主要目的是和onStart配对，停止耗时操作。

onCreate中也可以恢复状态，但是onRestoreInstanceState的触发时间滞后于onCreate，在onStart之后执行，它设计的目的是用来恢复onSaveInstanceState中保存的状态。

onPause以及onCreate可以干这些事情，但是它们当初不是设计出来，专门干这个事情的。


> 如何判断一个Activity真正可见?

对于这个问题，其实没有严格的说法，onResume以及onWindowFocusChanged中都可以做判断，但是官方文档上说，onWindowFocusChanged是Activity对用户是否可见最好的指示器。

### 扩展之外


好了，上面几个问题都回答了，谷歌官方文档写的非常的详细。那么，问题又来了

> 谷歌为什么要设计生命周期中的这几种状态呢？

从谷歌的官方文档可以看出，onStart，是通过是否可见这种状态来作为区分，onResume则是通过是否可以交互来区分，onPause设计的是与onResume相对应，onStop与onStart相对应，onCreate则是与onDestroy对应。

可以看出，谷歌在设计Activity的生命周期时，主要的依据是，是否可见以及是否可交互。那么这两种状态对于Activity来说有哪些影响呢？

仔细想想，不难看出，在`正常状况下`，Activity的生命周期也是根据这两个因素来运作的。失去焦点时，流程会走向onPause，获取焦点，则会走向onResume。完全不可见时，会走向onStop，后面又可见时，会去调取onStart。`正常状况下`的状态变换，都是围绕着这两个因素来流动的。

### 最后的话

本文只是单纯的从Activity生命周期的角度，来分析一些问题，并对一些现状做了解释。当然，我所表述的有可能会有问题，欢迎大家指正。

平时接触到的书籍以及网上的资料，包括日常的开发中，这些问题的分析解答少之又少。这些问题也一直困扰着我，因此我才写了这篇文章，就最常见的东西，换个角度去看问题。我所参考的资料简单的不能再简单了，就是谷歌文档，一些结论也是依据文档中的描述所推导的。

最后感谢大家的阅读。


