---
layout: post
title: 喜闻乐见之Android简介
categories: reading-notes
excerpt: Android简介
tag: [Android, APK， Dalvik, ART]
comments: true
image:
  feature: pic-book-1.jpg
---

# Android简介

本文主要是对Android系统做一个简介，因此很多东西不做深入展开。

## 架构

![Android体系结构图](/images/android-system-architecture.png)

Android是基于Linux内核开发出的一个移动操作系统，系统结构大致可以分为五层。自顶向下分别是系统应用程序、Java API框架、系统运行库、硬件抽象层以及Linux内核。

### Linux内核

Linux内核是Android平台的基础。例如ART依赖于Linux内核层的线程以及内存管理。Android
使用Linux内核，是因为其良好的安全特性以及硬件驱动的支持。

### 硬件抽象层

Android的硬件抽象层，简单来说，就是对Linux内核驱动程序的封装，向上提供接口，屏蔽低层的实现细节。由于Android跟Linux所采用的证书不同，如果把驱动都放在Linux内核层，发布的时候就需要公布源代码，硬件的相关参数和实现就会被公开，这对厂家来说，损失非常大。因此，通过添加这一层，使得商业秘密隐藏起来，减小厂家的损失。单纯从架构角度来说，这一层并不是必须的，只是商业上的一种妥协。

### 系统运行库

这一层包含两部分，核心库以及Android运行环境。这一层提供了一个很关键的模块，Art（早期为Dalvik）虚拟机。

### Java API框架

应用程序框架为应用程序提供了许多更高层的服务，例如Activity Manager、Resource Manager、Notifications Manager等。

### 系统应用程序

包含一系列系统App，例如通讯录、浏览器等等。

## Android启动流程

Android系统的整个启动过程，基本上可以划分为三个阶段：

1. Bootloader引导
2. Kernel启动
3. Android启动

### 引导程序（bootloader）

引导程序是Android操作系统开始运行前的一个小程序，引导程序是运行的第一个程序，它不是Android
操作系统的一部分，是OEM厂商或者运营商`加锁和限制`的地方。

引导程序分两个阶段执行：

1. 检测外部的RAM以及加载对第二阶段有用的程序。
2. 引导程序设置网络、内存等。

### 内核（kernel）

Android内核与桌面linux内核启动的方式差不多。内核启动时，设置缓存、被保护存储器、计划列表，加载驱动。当内核完成系统设置，然后启动init进程。

#### init进程

init是第一个进程，可以说是root进程或者所有进程的父进程。init进程有两个责任：

1. 挂在目录。
2. 运行init.rc脚本。

init.rc文件是由被称为Android初始化语言所编写的，有特定的格式以及规则。init进程创建完毕后，就可以在屏幕上看到Android的Logo了。

#### Zygote进程

Zygote是一个虚拟器进程，是由init进程fork出来的，在系统引导的时候启动。Zygote会预加载以及初始化系统的核心库类，使得虚拟机共享代码、低内存占用以及最小的启动时间成为可能。在Android系统中，所有应用程序进程以及系统服务进程（SystemService）都是由Zygote进程fork出来的，它建立了App运行所需要的环境，是非常重要的进程。在这个阶段可以看到启动动画。

### Android启动

在此过程中，主要是系统服务以及其他服务的创建过程。核心服务如Activity管理器、包管理器以及电池服务等。其他服务如通知管理器、网络连接服务以及设备存储监视服务等。

一旦系统服务在内存中跑起来，Android变完成了引导过程，这个时候就会收到开机启动广播`ACTION_BOOT_COMPLETED`。

## 沙箱机制

Android利用了Linux的基于用户的保护策略（user-based protection），作为鉴定和分离应用程序资源的一种手段。每个Android应用都运行在自己的沙箱内：

* Android操作系统是一种多用户Linux系统，其中的每个应用都是一个不同的用户；
* 默认情况下，系统会为每个应用分配一个唯一的Linux用户ID（该ID仅由系统使用，应用并不知晓）。系统为应用中的所有文件设置权限，使得只有分配给该应用的用户ID才能访问这些文件；
* 每个进程都具有自己的虚拟机 (VM)，因此应用代码是在与其他应用隔离的环境中运行；
* 默认情况下，每个应用都在其自己的Linux进程内运行。Android会在需要执行任何应用组件时启动该进程，然后在不再需要该进程或系统必须为其他应用恢复内存时关闭该进程。

Android系统可以通过这种方式实现`最小权限原则`。也就是说，默认情况下，每个应用都只能访问执行其工作所需的组件，而不能访问其他组件。 这样便营造出一个非常安全的环境，在这个环境中，应用无法访问系统中其未获得权限的部分。

## APK介绍

### 简介

APK即Android Application Package，Android
应用程序包。是Android系统中的应用程序包的文件格式。APK文件可以看作应用代码、资源、证书以及manifest文件的容器，它基于zip格式将这些资源打包。它的MIME type为application/vnd.android.package-archive。

### 结构

一个APK文件通常包含以下文件：

```
META-INFO文件夹：
	MANIFEST.MF: 清单文件。
	CERT.RSA: 保存着该应用程序的证书和授权信息。
	CERT.SF: 保存着SHA-1信息资源列表。
lib：包含不同CPU架构的库。
res：资源文件夹。
assets：可被AssetManager访问的资源文件夹，不会被特殊处理。
AndroidManifest.xml：manifest文件。
classes.dex：classes文件通过DEX编译后的文件格式，在Dalvik虚拟机上运行的主要代码部分。
resources.arsc：包含预编译的应用资源，例如XML等。

```
{: .notice}

### 构建工具

针对一般的应用开发，在Eclipse年代，构建主要是使用Ant。对于Android Studio年代，则是Gradle，它是AS默认的构建工具，结合了Ant以及Maven的优点，简洁并且灵活性高。

## Dalvik以及ART

谷歌在Android L上直接删除Dalvik，使用了ART作为替代，那么这两者有什么区别呢？

### Dalvik

Dalvik主要是完成对象生命周期管理、堆栈管理、线程管理、安全和异常管理、垃圾回收等等重要功能。Dalvik还负责进程隔离，每一个Android应用在底层都会对应一个独立的Dalvik虚拟机实例，其代码在虚拟机的解释下得以执行。

Dalvik是一个基于寄存器，运行dex格式文件的虚拟机。它不同于JVM的栈架构，也不能运行Java的字节码。从Android
2.2开始，Dalvik支持JIT。app每次运行的时候，Dalvik会将dex文件编译为机器码，这种实时编译的方式是其与ART最主要的区别。

### ART

ART是一种在Android操作系统上的运行环境，ART能够在第一次安装的时候，把应用程序的字节码转换为机器码。采用了预编译（AOT,Ahead-Of-Time）技术。在安装的时候，ART使用dex2oat工具来编译app。与Dalvik相比，主要的改进是减少了字节码到机器码的翻译过程。

### 区别

* ART采用预编译（AOT,Ahead-Of-Time）技术，启动运行会更快。
* 对于同一个app，ART所需要的存储空间比Dalvik大，安装的时间也会变长。
* ART不用实时编译，因此对于CPU的使用减少了很多，续航能力更强一些。
* ART有着更高的GC效率，更友好的debug支持。

## 参考


1. [Android启动过程深入解析](http://blog.jobbole.com/67931/)
2. [Android系统启动流程](https://www.jianshu.com/p/2ca0f6c974c9)
3. [Platform Architecture](https://developer.android.com/guide/platform/)
4. [System and kernel security](https://source.android.com/security/overview/kernel-security)
5. [Android硬件抽象层（HAL）概要介绍和学习计划](https://blog.csdn.net/luoshengyang/article/details/6567257)

