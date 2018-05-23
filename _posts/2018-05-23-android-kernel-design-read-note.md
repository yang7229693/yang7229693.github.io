---
layout: post
title: 读书笔记-深入理解Android内核设计细想
categories: reading-notes
excerpt: 读后感以及一些书摘
tag: [Android, Kernel, Design]
comments: true
image:
  feature: pic-book-1.jpg
---

## 读后感

这本书是在电脑上粗略的翻完了，看的过程中也是选择性的浏览，后面肯定会续上正版书籍。

整体而言，书写的挺不错的，很多地方都有作者自己的思考，例如通过换位思考的方式，去讲解自己该如何设计，但是相反的，太依赖于Android源码。本书挺适合系统级开发者去深入研究，对于应用级开发者，可以选择性阅读，了解一些底层的原理，如果感兴趣，也可以深入了解。

看了这么多Android开发相关的资料，还是觉得谷歌Android开发文档写的最好，如果真能把那个给啃透，我觉得基本上会是大牛级别的。


## 简单书摘

《Anatomy & Physiology of an Android》

分为内核层、硬件抽象层、系统运行库层、应用程序框架层以及应用程序层。

《Android Anatomy and Physiology》
Android采用Linux内核的原因：

* Great memory and progress management；
* Permissions-based security model；
* Proven driver model；
* Support for shared libraries；
* It`s already open source。

Android中使用最多的一种IPC机制是Binder，其次就是UDS（Unix Domain Socket）。

Android中同步机制：

* Mutex：对pthread的简单再封装；
* Condition：条件变量的实现，核心思想是判断“条件是否已经满足”；
* Barrier：同时基于Mutex和Condition实现的一个模型，是填充了“具体条件”的Condition，专门为SufaceFlinger而设计。

内存管理旨在为系统中的所有Task提供稳定可靠的内存分配、释放与保护机制。

Linux的OOM Killer核心思想：按照优先级顺序，从低到高逐步杀掉进程，回收内存。综合考虑了以下几个因素：

* 进程消耗的内存
* 进程占用的CPU时间
* oom_adj（OOM权重）

Android参照Linux的OOM Killer，实现了自己的Low Memory Killer（LMK）。在Android中，修改oom_adj值得方法有：

* 写文件，路径为/proc/<PID>/oom_adj
* android:persistent，通过在Manifest中添加该属性，将程序设置为常驻内存

Anonymous Shared Memory（Ashmem）是Android特有的内存共享机制，可以将指定的物理内存分别映射到各个进程自己的虚拟地址空间中，从而便捷地实现进程间的内存共享。

Linux内核在内存管理上有Slab、Slub和Slob三种机制。

JNI（Java Native Interface）是一种允许运行于JVM的Java程序去调用本地代码的编程框架。在如下三种情况下需要使用到JNI：

* 需要一些平台相关的feature支持，而Java无法满足
* 兼容以前的用其他语言书写的代码库
* 某些关键操作对运行速度要求较高

Handler，MessageQueue，Runnable与Looper

![Runnable，Message，MessageQueue，Looper和Handler的关系简图](/images/runnable message  looper handler关系简图.png)

> Looper不断获取MessageQueue中的一个Message，然后由Handler来处理。

Handler与Thread的关系：

* 每个Thread只对应一个Looper
* 每个Looper只对应一个MessageQueue
* 每个MessageQueue中可以有多个Message
* 每个Message中最多指定一个Handler来处理事件

Binder是Android系统中应用最广泛的IPC机制。类似于TCP/IP中的访问过程，而Service Manager则充当着DNS的作用。

> Service Manager在Binder通信过程中的唯一标志永远都是0。

智能指针的设计目的，就是为了解决C/C++中未初始化、未释放等问题。采用引用计数的方式来进行相关内存的检测。分为强指针和弱指针两种。

> 弱指针的主要使命就是解决循环引用的问题，必须先升级为强指针，才能访问它所指向的目标对象。

进程间的数据传输载体为Parcel，由于采用了虚拟内存机制，两个进程都有自己的独立空间，跨进程传递的地址值是无效的，因此只能传递数据，而Parcel提供了序列化和反序列化操作，可以用于进程间数据得传输。

Bundle就是继承自Parcelable，采用键值对的方式存储数据，并在一定程度上优化了读取效率。

Binder Driver是一个标准的Linux驱动，会将自己注册成一个misc device，并向上层提供一个/dev/binder节点，但是并不对应真实的硬件设备，其运行于内核态，可以提供open、ioctl、mmap等常用的文件操作。

* binder_open，打开Binder驱动，会在/proc系统目录生成各种管理信息，并将它加入Binder的全局管理中。Binder驱动为用户创建一个它自己的binder_proc实体，用户对Binder设备的操作将以这个对象为基础。
* binder_mmap，mmap可以把设备指定的内存块直接映射到应用程序的内存空间中，Binder驱动只需要一次复制，就可以实现两个进程间的数据共享。

![Binder](/images/binder mmap复制操作.png)

* binder_ioctl，承担了Binder驱动的大部分业务，例如读写操作、设置支持的最大线程数、线程退出、获取Binder版本号等。实现了应用进程与Binder驱动之间的命令交互。


ServiceManager本身是一个标准的Binder Server。在init程序解析init.rc时启动。一旦ServiceManager发生问题后重启，其他系统服务如zygote、media、surfaceflinger、drm也会被重新加载。整个Android系统中只允许有一个ServiceManager存在。

ServiceManager的构建包括以下几个步骤：

* 打开Binder设备，进行初始化。
* 调用binder_become_context_manager方法，提升自己权限为Manager。
* 进入循环。

ServiceManager中没有消息队列，从Binder驱动获取消息。主要工作时完成“Binder Server Name”（域名）和“Server Handle”（IP地址）间对应关系的查询而存在的。通过内部维护着的一个svclist列表，来存储对应关系，所有查询和注册都是基于这个表展开的。

ServiceManager提供以下几种服务：

* 注册：当Binder Server创建后，需要将自己的[Binder Server Name，Binder Handle]对应关系告知SM进行备案。
* 查询：应用程序可以向SM发起查询请求，获知某个Binder Server的Handle。
* 其他信息查询：查询SM版本号、当前状态等。

对于设备而言，访问SM的流程如下：

* 打开Binder设备；
* 执行mmap进行内存映射；
* 通过Binder驱动向SM（Handle为0）发送请求；
* 获得结果。

> 每个进程只允许打开一次Binder设备，且只做一次内存映射，所有需要使用Binder驱动的线程共享这一资源。

ActivityManangerService（AMS）是Android提供的一个用于管理Activity（和其他组件）运行状态的系统进程。作用如下：

* 组件状态管理
* 组件状态查询
* Task相关
* 其他辅助功能

ActivityStack是管理当前系统中所有Activity状态的一个数据结构。

> AMS是通过ActivityStack来记录、管理系统中的Activity（和其他组件）状态，并提供查询功能的一个系统服务。

Android的GUI系统是基于OpenGL/EGL实现的。

WindowManagerService主要功能如下：

* 全局的窗口管理（Output）
* 全局的事件管理派发（Input）

Android中窗口的类型可以分为三大类：

* Application Window：普通应用窗口都属于这一类；
* System Window：系统程序所采用的窗口类型，除个别特殊情况，普通应用程序不能创建系统窗口，系统会做权限检查；
* Sub Window：子窗口，附着在其他Window中。

> Android系统中的Ui更新遵循一条规则，只有创建UI元素的线程才能使用它。

动画的本质是通过连续不断地显示若干图像来产生“动”起来的效果。

动画的四要素包括缩放、平移、旋转和透明度，前三者由矩阵Matrix表示，最后一项则由float变量表示，都封装在同一个类中，即Transformation。而Animation是对动画本身的描述，如起点、终点、时长、速率等属性。这两个类是动画的核心，一个是“静态”的描述，一个是“动态”的计算。

ViewRoot是View树的管理者，有一个mView成员变量，指向它所管理的View树的根，ViewRoot的核心任务就是与WindowManagerService进行通信。

Skia的优秀特性如下：

* 跨平台
* 高度优化的渲染器
* 可选的硬件加速
* 动画实现
* 图片解码
* 支持绘制能力
* 支持多种特效

即便View设置了disabled，它同样会消耗点击事件，只是不作出任何回应而已。

Android系统中提供如下几种类型的动画：

* Property Animation
* View Animation
* Drawable Animation
* Window Animation

> 动画的本质是时间和图像的关系。

Android输入事件投递系统的工作流程如下图所示：

![事件处理系统工作流程图](/images/事件处理系统工作流程图.png)

Apk编译过程如下图所示：

![apk编译过程](/images/apk编译过程.png)

签名的apk通过zipalign进行优化，目的是提高程序的加载和运行速度。对apk包中的数据进行边界对齐，从而加快读取和处理过程。

系统只在安装过程中检查证书的有效性，如果程序安装以后证书过期，并不影响使用。当用用程序升级时，系统会比较新旧版本证书是否一致，如果一致升级才能顺利进行。

采用相同签名的程序允许被安排在同一个进程中运行，可以根据特殊权限进行代码和数据得共享。

