---
layout: post
title: Dex方法数优化
categories: reading-notes
excerpt: Dex方法数的优化方案整理
tag: [Android, Dex, Method Count, Reduce, 优化]
comments: true
image:
  feature: pic-book-1.jpg
---

Android开发做了有段时间了，最近接到的一项工作是优化Dex方法数，这个的起因是我们的集成环境，有个方法数预警，超过6w每次编译就会提示，这个看着很烦，况且，我们的APP框架什么的比较简单，因此这次就单纯的从减少Dex方法数进行优化。

Dex方法数优化，这个本质上可以看作apk包大小的缩减，但又有其特殊性，例如不能够使用压缩来进行方法数的减少。我的思路是，先分析自己apk，找出方法数占用较多的模块，查看其他产品的方法数大小，针对占用较多的模块，进行优化操作。apk打包使用的是gradle，方法数分析工具使用的是dex-method-counts（https://github.com/mihaip/dex-method-counts）。

直接就是干了，根据编译出来的release包，查看其包含的方法数，为5.6w左右，其中android包下方法数为23473，support库占用方法数为19130，v4包含9862个方法，v7包含6267个方法。

com下包含的方法数为2.6w左右：

    iflytek：2402（讯飞库）
    qtranslator包下：5892（我们自己的代码）
    sina weibo：1038（微博分享库）
    okhttp：1582（http库）
    tpush：2044（信鸽push库）
    bugly：1686（crash上报库）
    smtt：2359（tbs库）
    tinker：1588（热更新库）
    taf:1755（taf库）
    jce生成文件：1369（jce生成文件）
    glide:2410（图片库）
    gson:1034（json解析库）
    pulltorefresh:405（下拉刷新）
    async:2831（socket库）
    sqlcipher:703（数据库加密库）
    okio:499（okhttp需要的库）
    wtlogin_sdk:1865（第三方登录库）
    greendao:953（数据库操作库）

可以看到，主要是各种第三方库占用了大量的方法数，大致从以下几个点进行优化，当然一些优化点只是提出阶段，并没有实际去执行，并不是方案不可行，只是现阶段时间比较紧，我觉得投入产出比不是很高。


>优化点1：删除无用代码，删除未使用的import。使用lint工具扫描

这一部删除的代码非常有限，但是也有优化的必要。无用代码放在项目中，终究会带来诸多不便。

![](/images/dex_reduce_image1.png)

方法数变化：56243->55995 (减少248)

>优化点2：删除不使用的以及现阶段不会用到的第三方库

删掉没有使用到的讯飞库，方法数变化：55995->53366（2629）

![](/images/dex_reduce_image2.png)

删掉现阶段没有使用到的信鸽库，方法数变化：53366->51227（2139）

![](/images/dex_reduce_image3.png)

>优化点3：使用最新的第三方库

sqlcipher库从3.4.0->3.5.7  apk大小减少19.35m->16.79m（2.56m）

>优化点4：support库优化

v4、v7包含了1.9w+的方法数，然而代码中我们只是用了少部分模块，因此没有必要全量引入，查看QQ Android版本的方法数，support包含的方法数只有5000+，微信的support也只有9000+，这一块儿优化空间很大。

去除design库，单独引入recycle模块：51227->48412（2815）

![](/images/dex_reduce_image4.png)

>优化点5：替代占用方法数较多的第三方库

app中图片不是一种很强的需求，glide可以找第三方比较轻量级的库替代。
http和socket使用了okHttp和async两个库，后续看是否能找到一个较小的库能替代。


>优化点6：裁剪第三方库

我们引入的第三方库，很多都是只使用很少一部分功能，完全可以进行大量的裁剪工作。

>优化点7：使用Json或者FlatBuffers

jce引入了很多东西，客户端添加接口也异常繁琐，建议后续逐步放弃jce这套机制，转入Json或者FlatBuffers。


整个优化过程下来，apk包的减小最为明显，虽然主要是去减dex方法数。安装包大小（统计的rdm下的qqtranslator_*_android_release.apk包）从19.35m，降到16.39m，减小了2.96m，安装包大小减少了15.3%。dex方法数从56243减少到48412，减少了7831，方法数减少了13.9%。
 
    引入新的第三方库时候，横向对比，综合大小、性能等多方面纬度进行考虑。
    针对简单的功能，引入一个较大的库，这种做法不太适合，建议自行实现。
    如果只是使用第三库的一小部分的功能，建议进行裁剪后使用。
    方法数增多无法去规避，用谷歌官方的multidex插件即可解决（现阶段我们没有插件等复杂处理情况下），因此方法数问题，不是很需要特别去关注，我们应当更多的去关注apk包大小的变化。

针对dex方法数的减少，在这一轮修改能够做的非常有限，我们可以尝试去往apk大小优化的方向去解决问题，dex方法数始终是一个无法规避的问题，apk大小的优化，还有很多可以尝试的方案，建议后续版本可以进行一些调研实现。

手机QQ Android版本 support 方法数：

![](/images/dex_reduce_image5.png)

微信android版本support方法数：

![](/images/dex_reduce_image6.png)

手机QQ Android版本方法数：

![](/images/dex_reduce_image7.png)


