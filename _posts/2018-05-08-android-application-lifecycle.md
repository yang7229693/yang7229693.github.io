---
layout: post
title: 喜闻乐见之Android应用的生命周期
categories: reading-notes
excerpt: 讲述了App的启动流程、Application的生命周期、进程的回收机制
tag: [Android, LifeCycle，Recycle, Progress]
comments: true
image:
  feature: pic-book-1.jpg
---

> 本文主要讲述了App的启动流程、Application的生命周期以及进程的回收机制。

在绝大多数情况下，每一个Android应用都在自己的Linux进程中运行。当需要运行某些代码时，进程就会被创建。进程将保持运行直到不再需要，当其他应用有需要的时候，系统会释放该进程的内存。

一个不常见但很基础的Android特性是，一个应用进程的生命周期并不是由应用本身直接控制的。它是由系统根据正在运行的程序，对用户的重要程度以及所占用的内存，综合去管理的。

## App启动流程

此处讨论的是第一次启动App。在讲解App启动流程的时候，有两点需要知晓：

1. 每一个App都运行在一个独立空间里，也可以称之为沙盒，这意味着它是在独立的线程中，拥有自己的虚拟机实例，被分配一个唯一的用户ID。
2. App由很多不同组建组成，这些组件还可以调用其他App的组建，并没有一个单独的类似于main函数的入口。

总体来说，App启动分为三个步骤，创建进程、绑定Application以及启动Activity。当用户点击Android桌面一个App图标，启动一个应用的时候，整个流程如下所示：

![App Launch Summary](/images/app-launch-summary.jpg)

点击事件通过Binder IPC机制最终被转换成startActivity(intent)，而App相关信息的解析以及处理则在安装的时候就已经完毕。当startActivity的时候，ActivityManagerService（AMS）的操作如下：

* 第一步是intent解析。通过PackageManager的resolveIntent()方法，收集目标intent对象的信息。
* 第二步是权限检查。通过grantUriPermissionLocked()方法，检查用户是否有足够的权限去调用intent的目标组件。
* 第三步是创建新的任务。如果用户有足够的权限，ActivityManagerService就会检查目标Activity是否需要被加载在新的任务中。这个任务是根据Intent Flag来进行创建的。

最后检测进程记录表（ProgressRecord）是否存在。如果不存在，则需要通过ActivityManagerService来创建新的进程。

### 进程创建

ActivityManagerService通过调用startProcessLocked()方法来创建一个新的进程，这个方法会通过socket连接发送参数到Zygote进程。Zygote创建ActivityThread对象并返回新创建进程的pid。ActivityThread通过依次调用Looper.prepareLoop()和Looper.loop()方法开启消息循环。

### 绑定Application

当进程创建完毕后，需要将其与Application绑定起来。ActivityThread通过发送BIND_APPLICATION消息执行绑定，会执行makeApplication()方法，将App的类加载进内存。

### 启动Activity

经过前面两步，系统拥有了与application相关联的进程，应用中相关的类被加载到进程的内存区域。在新创建或者已经存在的进程中启动Activity的步骤是一样的。ActivityThread通过发送LAUNCH_ACTIVITY消息进行启动操作。

大致梳理一下上面的过程，当用户点击应用图标的时候，系统会去检测进程是否存在，如果不存在，则通过Zygote创建进程，创建完进程后，则需要将进程与App进行绑定，将App的资源加载到内存中，当加载完毕，各种条件准备就绪，接下来就是启动Activity了，如此，App的界面就展示出来了。

## Application生命周期

此处讨论的是Application类，而非应用生命周期。Application的作用，用官方文档的话说。

> Base class for maintaining global application state. 

它的作用是维护全局的应用状态的基类。Application类是应用中最先被初始化的类，先于Activity等组件。它的生命周期与Activity有几分相似，都经历了创建销毁过程，只不过它是一个线性的过程，不存在一些恢复过程。

### onCreate

> Called when the application is starting, before any activity, service, or receiver objects (excluding content providers) have been created.

当应用启动的时候调用，除了content providers之外，早于其他任何组件的创建。

### onLowMemory

> This is called when the overall system is running low on memory, and actively running processes should trim their memory usage.

当整个系统内存不足的时候，活跃的进程需要减少它们的内存使用的时候，回调会被调用。系统调用此回调过后，会产生一次GC操作。

### onTerminate

> This method is for use in emulated process environments.It will never be called on a production Android device, where processes are removed by simply killing them; no user code (including this callback) is executed when doing so.

在正式环境的真实设备上，不会被调用。由于系统结束进程采用的是kill的方法，因此不会产生相关的回调。

### onTrimMemory

> Called when the operating system has determined that it is a good time for a process to trim unneeded memory from its process.

当系统检测到应用可以回收不需要的内存时，会产生此回调。例如应用处于后台，系统内存不足的时候。

## 回收机制

当系统出现低内存状况的时候，会根据不同进程的优先级进行内存回收。系统根据进程的状态，将进程分为四个等级。

### Foreground Process

前台进程是优先级最高的进程，用户当前操作交互的必须是前台进程。当一个进程包含以下条件的时候，可以认为是前台进程。

1. 用户正在交互的，处在屏幕最上层的Activity（onResume被调用过后）。
2. 包含一个正在运行的BroadcastReceiver（onReceive正在执行）。
3. 包含一个正在运行它的回调（onCreate、onStart、onDestroy）的service。

在一般状况下，杀死前台进程需要用户交互。当被系统杀死的时候，说明此时系统连该进程所需要的内存都无法满足，是最后才被杀死的。

### Visible Process

可见进程是当前用户关心但是杀掉它会显著的影响用户体验的进程。当包含如下情况时，该进程可以被当做可见进程。

1. 包含一个对用户可见但不在前台（onPause被调用过后）的Activity。
2. 包含一个通过startForeground启动，正在运行的作为前台service的进程。
3. 包含一些用户可感知的特定需求的service，例如动态壁纸、输入服务等。

可见进程一般不会被销毁，除非是为了保证所有前台进程的运行，而不得不杀死可见进程。

### Service Process

服务进程是包含用startService所创建service的进程。这类进程对用户不是直接可见，但是用户会关心的，例如后台上传服务等等。所以系统会尽量维持它们的运行，除非系统内存不足以维持前台进程和可见进程的运行需要。

当service运行了很长的时间，例如超过30分钟，系统就会对其降级，以使该进程会被更容易的回收。

### Cached Process

缓存进程是当前不被需要的进程，因此系统可以在任何需要内存的时候，释放掉它们的内存。

这类进程通常包含一个或多个当前不可见（onStop被调用）的Activity实例。当系统杀掉这类进程的时候，不会影响用户的体验。

### 不一致的地方

关于进程优先级，一般网上给出的前三种跟此处所列一致，不同之处是最后两种为后台进程以及空进程，而谷歌文档上，直接被归到缓存进程了。这个本身没有什么冲突，本质是一样的，谷歌根据进程对用户的重要程度划分的优先级，记住这个大方向就没啥问题了。

### 管理

关于进程的管理，首先需要知道Low Memory Killer（LMK）这个概念。它是基于Linux的Out of Memory Mechanism（OOM机制）改进而来的。LMK是一个内核层组件，是一个进程杀手。它的主要作用是在系统低内存状态时，释放掉那些不太重要进程的内存，让系统更加流畅。OOM只有当系统内存不足的时候才会启动检查，而LMK则不仅是在应用程序分配内存发现内存不足时启动检查，它也会定时地进行检查。

每一个进程根据其重要性，都包含一个oom_adj值，oom_adj的大小和进程的类型以及进程被调度的次序有关，AMS会去动态的更新oom_adj。当系统处在低内存状态时，LMK会根据oom_adj大小，杀死相关的进程。oom_adj值得范围是-17~15，一个进程的oom_adj值越高，它被杀死的概率就越大。

整个过程就是AMS更新oom_adj值，LMK去挑选并杀死进程。

## 问题

> ActivityManagerService的主要功能包括哪些？

可以看到，在安装App以及启动App的过程中，都有AMS的大量参与，它的主要功能包括以下几部分：

1. 统一调度各应用程序的Activity
2. 内存管理
3. 进程管理

> App的安装过程是怎样的？

Apk其实就是一个压缩包，系统安装App的过程，其实就是资源的解析、拷贝以及验证等过程。

PackageManagerService会将App中的Manifest信息解析出来，并持久化，当用户点击桌面icon的时候，系统就会知道该启动哪个组件。

PMS安装App，最后底层调用的也是adb命令来执行的。

大致来说，整个流程是，解析apk文件，执行安装过程，最后更新UI。

> onLowMemory与onTrimMemory区别？

onLowMemory与onTrimMemory都是可以进行内存回收操作的地方，两者不同之处有以下几点：

1. 两者API level不同，onLowMemory在API level 1就被添加了，而onTrimMemory是在API level 14中被添加的。当然，对于现在最低版本都是从十几开始支持的，完全可以直接使用onTrimMemory。
2. 两者的触发时机不同，onLowMemory是在系统出现低内存状况时被触发，而onTrimMemory则是在置于后台而内存不足时被触发。

对于API 14以上的条件下，onTrimMemory在`TRIM_MEMORY_COMPLETE`级别跟onLowMemory可以等同。


## 参考

1. [Processes and Application Lifecycle](https://developer.android.com/guide/components/activities/process-lifecycle?hl=zh-cn)
2. [Android Application Launch](http://multi-core-dump.blogspot.com/2010/04/android-application-launch.html)
3. [Android Application Launch Part 2](http://multi-core-dump.blogspot.com/2010/04/android-application-launch-part-2.html)
4. [Application](https://developer.android.com/reference/android/app/Application)
5. [Android application and activity life cycle - Tutorial](http://www.vogella.com/tutorials/AndroidLifeCycle/article.html)
6. [Android系统APP安装流程解析](https://www.2cto.com/kf/201711/696423.html)
7. [android内核剖析学习笔记：AMS（ActivityManagerService）内部原理和工作机制](https://blog.csdn.net/u013149325/article/details/41827013)
8. [Android源码解析之（十二）-->系统启动并解析Manifest的流程](https://blog.csdn.net/qq_23547831/article/details/51203482)
9. [Android源码解析之（十三）-->apk安装流程](https://blog.csdn.net/qq_23547831/article/details/51210682)
10. [How Android manages background processes?](https://github.com/mcgill-cpslab/mobile-research/wiki/How-Android-manages-background-processes%3F)
11. [Android Low Memory Killer](http://www.programering.com/a/MjNzADMwATE.html)
12. [Android low memory killer 机制](https://my.oschina.net/wolfcs/blog/288259)
13. [How to Tweak Android Low Memory Killer to Your Needs](http://www.droidviews.com/tweak-android-low-memory-killer-needs/)