---
layout: post
title: App Slicing分析
categories: iOS
excerpt: 苹果iOS9中的App Slicing原理分析
tag: [iOS, iOS9, app slicing, app thinning, bitcode, odr, On-Demand Resources]
comments: true
image:
  feature: pic-ios-2.jpg
---

{% include _toc.html %}

####关于App Thinning

苹果在iOS9中引入了不少新的特性，对于目前越做越大的APP而言，App Thinning的引入无疑是雪中送炭，从iOS9的推送包从原来的三四G减小到一点几G，可以看到，App Thinning对于程序体积的减小效果相当的明显。

App Thinning主要包括三部分，[苹果官网有详细的介绍](https://developer.apple.com/library/prerelease/ios/documentation/IDEs/Conceptual/AppDistributionGuide/AppThinning/AppThinning.html)，主要包括三部分

	App Slicing
	Bitcode
	ODR(On-Demand Resources)
{: .notice}
其中Bitcode是需要各种库的支持，在XCode 7+中的`Build Settings`中`Enable Bitcode`即可，ODR一般用于游戏中多关卡场景较多，在此不做过多的阐述。

下面主要说下Slicing，就字面意思就是分割了，苹果最终想要实现的是针对不同设备来提供不同的安装包，去除不必要的各种资源文件来达到减小安装包大小的效果。盗用一张苹果官网的图

![](/images/app_thinning.jpg)

原理很简单，只需要注意只有在iOS9+的设备上才会支持。

####App Slicing分析

App Slicing的实现过程，官网也都有，我按照官网的内容简单的总结一下吧

	1. 在asset catalog中指定目标设备，并提供多种分辨率版本的图片（1x 2x 3x）
	2. App打包，导出不同设备的variant进行测试
	3. 上传到iTunes Connect
{: .notice}
其实Slicing是个体力活吧，把应用中的资源整理一番，然后对相关地方的引用稍作修改即可。这样说起来比较抽象吧，直接看我弄的一个小demo的结果吧，其中我把bitcode选项关掉了。

![](/images/app_slicing_1.jpg)

接下来我用这个简单的例子走一遍流程，希望可以对大家有所帮助吧。

我新建了一个简单的程序，内容很简单，在页面上展示五张图片，其中也包括各尺寸的icon以及launch图片，都放在Assets.xcassets中。


![](/images/app_slicing_2.jpg)

Archive程序，选择`Export...`，`Save for Development Deployment`，如图

![](/images/app_thinning_3.jpg)

我们选择`Export for specific devices`下的`All compatible device varients`，导出所有设备的variant，可以看到导出的文件夹内包括三个文件，

	App Thinning Size Report.txt
	app-thinning.plist
	Apps
{: .notice}
其中Apps文件夹下面包含着的就是各设备对应的variant，如图

![](/images/app_thinning_4.jpg)

可以看到各设备对应程序包大小有所不同，接下来我们来分析一下造成这些包大小不同的原因，初步猜测应该是资源文件不同造成的，对于plus设备，应该只包含一套3x图，对于5、5s、6、6等设备，应该只包含一套2x图，对于all的ipa，应该包含1x、2x、3x总共三套图。

上面的只是猜测，我们接下来只是分析下6s和all的ipa包内的资源有所不同，解压后会发现一个很令人匪夷所思的问题，如下图

![](/images/app_thinning_5.jpg)

所有的解压包中都包含了全尺寸的icon和launch image资源，这两部分是根据iOS版本来显示的，因此苹果不会对这些资源进行Slicing处理的。

对比不同variant，可以发现，主要有两个部分不同，也正是这两块造成了包大小的差异

	执行文件
	Assets.car
{: .notice}
对于`执行文件`，使用lipo命令可以看到，主要是包含的处理器架构不同导致的执行文件大小不同，对于6s来说，只包含了arm64架构，对于all，包含了armv7和arm64.

对于`Assets.car`，就其名称就可以猜测出，是资源文件的打包，使用工具对其提取资源，6s中包含了一套2x图，符合猜测，对于all，提取出的资源文件包含了1x、2x、3x三套资源与我们猜测的一致。

最后还有一个问题，对于上传到iTunes Connect，我们是否需要去关注Slicing的过程呢，答案是`否`，我们upload到苹果的服务器，苹果会在服务器端帮助我们对包进行Slicing操作，对应于不同设备的App Store，包的大小就会不同了。


####总结

苹果的App Slicing技术可以有效的减小安装包的大小，相比较Bitcode需要各种库的支持，它的实现最为简单，只需要对资源进行整理即可，苹果会自行的拆分成多个variant给用户，我们在对现有项目进行Slicing操作的时候，只需要注意下资源的引用是否正确即可。

如果有什么地方不正确的地方，欢迎大家补充哈，谢看～


####补充

xcode6生成的ipa中，`默认只支持armv7和arm64`。不再构建armv7s指令集代码，因此对于打包中iphone5设备对应的架构则显示为armv7。

然后附一张[设备对应系统架构图](http://static1.squarespace.com/static/51adfbd9e4b095d664d9b869/t/5596a861e4b0eb5f837cf243/1435936865957/iOS_Support_Matrix_v3_2.pdf)

