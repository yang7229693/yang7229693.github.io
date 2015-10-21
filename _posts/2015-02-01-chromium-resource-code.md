---
layout: post
title:  "Chromium获取源码"
date:  2015-02-01
categories: chromium
featured_image: /images/cover.jpg
---

###Chromium获取源码

1、下载 depot_tools

2、把depot-tools加入环境变量path中

3、执行

 	gclient --version

 	gclient —version

 注意，需要2次。

4、创建存放源代码目录，比如

	mkdir ~/chromium
	
	cd ~/chromium

5、获取代码

	fetch --nohooks chromium

6、切换到master分支

	cd src

	git checkout master

获取代码官方文档：

	http://dev.chromium.org/developers/how-tos/get-the-code

###更新代码

	cd  CHROMIUM_DIR

	git pull

	gclient sync

如果想加快速度，可以执行gclient sync —jobs＝16（jobs数目可以随意指定）

###编译代码
更快编译可以加上

	GYP_DEFINES=fastbuild=1 CHROMIUM_DIR/src/build/gyp_chromium

	CHROMIUM_DIR/src/ninja -C out/Debug chrome

编译官方文档：

	https://code.google.com/p/chromium/wiki/MacBuildInstructions

	https://code.google.com/p/chromium/wiki/NinjaBuild


网上大部分都是svn获取的方式，但是这种方式已经被google抛弃掉了，只能使用git的方式，拉取代码的时候需要稳定的VPN，当然有个国外的VPS最好了，用免费的翻墙工具都不太稳定，尝试了很多次都失败了，最后自己买的vpncup的服务，速度最快400多K，下了两天多，断了好几次，总共13G左右吧。