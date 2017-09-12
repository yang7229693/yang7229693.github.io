---
layout: post
title: Dex方法数优化之Android Support库优化
categories: reading-notes
excerpt: 针对Android Support库进行的优化
tag: [Android, Dex, Method Count, Reduce, 优化]
comments: true
image:
  feature: pic-book-1.jpg
---

查看apk方法数，发现，android support库包含了1.9w+的方法，我们把项目中的support库单独引用了，发现方法数还是没有减少，我们使用gradle工具来查看主包的依赖关系，执行gradle -q lottie:dependencies，查看主包的依赖关系，如下图所示：

![](/images/dex_reduce_2_image1.png)

可以看到整个项目中，对于support库的引用，主要集中在lottie库。下载lottie库源码，查看lottie的依赖关系

![](/images/dex_reduce_2_image2.png)

查看lottie的gradle配置文件

![](/images/dex_reduce_2_image3.png)

接下来，我们查看一下lottie库中对v4、v7 support库中的使用情况，我们全局搜索一下关键词`v4`、`v7`。

![](/images/dex_reduce_2_image4.png)

![](/images/dex_reduce_2_image5.png)

可以看到，针对v7库的使用，仅使用了一个AppCompatImageView控件。而对于v4库中的使用，也大部分集中在support-core-utils库中，因此，我们可以根据实际的使用情况，对lottie引入的support库进行指定引入，对于AppCompatImageView，我们可以直接使用ImageView来代替直接去除v7库。
修改后的gradle配置为

![](/images/dex_reduce_2_image6.png)

去除掉了v7库的引入，对于v4库而言，只是引入了support-annotations、support-core-utils、support-core-ui这三个使用到的库。修改后，再次查看lottie的包依赖关系

![](/images/dex_reduce_2_image7.png)

可以看到，现在只是引用了上面提到的三个v4的子库。但是support-core-utils、support-core-ui中的annotations我们并没有使用，因此可以把这一部分也剔除掉，因此完成版的gradle配置文件修改如下：

![](/images/dex_reduce_2_image8.png)

我们再次查看lottie库的依赖关系

![](/images/dex_reduce_2_image9.png)

可以看到，这个里面包含的库是我们实际中使用到的，由于谷歌对于库只细分到这个粒度，但我们实际上也只是用到库很少一部分模块。下面是优化前后，support占用方法数的一个对比。

![](/images/dex_reduce_2_image10.png)

![](/images/dex_reduce_2_image11.png)

可以看到单纯的针对lottie库，方法数减少了1.3w+，还是相当的可观的。接下来，的工作，则是将lottie库以源文件的形式，引入到项目中，但是我们项目中还有其他地方引入了v4、v7库，对gradle做出修改后。查看app依赖，如下图所示：

![](/images/dex_reduce_2_image12.png)

查看apk的support方法数，如下所示：

![](/images/dex_reduce_2_image13.png)

可以看到，support中方法数从19130 降至 14512，减少了4618，看上面方法数的扫描，其实很有很大的优化空间，将一些控件转化之类的操作，但是这个优化对于后续版本的维护，并不是特别好，例如lottie库的升级，后面同事添加新库以及添加新的代码，都非常容易的将support库给重新引入回来，因此这次的实验只是在本地进行，并没有提交到git，除非我们像QQ或者微信团队，对于第三方库的引入有非常严格的限制，否则的话，这条优化是操作起来比较复杂，但是破坏相当容易的，网上相关的操作介绍不是特别多，因此了解了，大致知道有这么一个方法就可以了吧。
