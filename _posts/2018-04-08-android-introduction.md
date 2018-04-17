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

![Android体系结构图](/images/android-system-architecture.jpg)

Android系统体系结构分为四层。自顶向下分别是应用程序、应用程序框架、系统运行库以及Linux内核。

其中，蓝色的代表Java程序，黄色的为运行在Java上的虚拟机，绿色部分为C/C++编写的程序库，红色部分为内核代码。可以看出，Android本身是基于Linux内核而开发出的一个移动操作系统。

### Linux内核

这一层为Android系统提供一系列硬件驱动，但是谷歌对其中做了部分修改，最主要的两部分为：Binder以及电源管理。

### 系统运行库

这一层包含两部分，核心库以及Android运行环境。这一层提供了一个很关键的模块，Dalvik虚拟机。

Dalvik虚拟机采用了Linux的很多核心的特性，例如内存管理以及多线程。它最初不是为Java设计的，并不能直接运行Java的bytecode指令，而是运行Dalvik executable，简称dx。为此Android提供了dx工具，用来将Java bytecode转换为dx。

### 应用程序框架

应用程序框架为应用程序提供了许多更高层的服务，例如Activity Manager、Resource Manager、Notifications Manager等。

### 应用程序

包含一系列App，例如通讯录、浏览器等等。

## 沙箱机制

每个Android应用都运行在自己的沙箱内：

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
* ART有着更高的GC效率。

