---
layout: post
title:  "Chromium八宫格排序解析"
date:  2015-05-01
categories: Chromium
featured_image: /images/chromium_cover.jpg
---

###Chromium八宫格排序解析



Chrome的新标签页所展示的八宫格，主要是为了方便人们快捷访问一些站点，但是经常会有人产生这样的疑问，为什么我经常访问的网站有时候会被访问没多久的网站所挤掉呢？这会不会是Chrome的系统bug呢？难道Chrome不是单纯的按照网站的访问次数来进行排序的吗？

首先，我们来看下Chrome中存放八宫格展示数据的文件

	/Users/*/Library/Application Support/Google/Chrome/Default/Top Sites
	
这个文件是一个SQLite数据库文件，通过一些工具（此处用到的是SQLiteManager），可以看到里面包含两张表，其中**thumbnails**这张表格中存储的就是八宫格所展示的数据。
![](/images/new-tab-sort-1.png)

八宫格中所显示的顺序主要是通过**url_rank**这个数值来决定的，数值越小越靠前，为**-1**的则是已经删除的站点。然而这个url_rank值是如何生成的呢？我们接下来看一下另一个文件
	
	/Users/*/Library/Application Support/Google/Chrome/Default/History

这个文件同样是一个SQLite数据库文件，这个数据库包含9张表，对于八宫格排序相关的表格是**segments_usage**
![](/images/new-tab-sort-2.png)

可以看到segments_usage表里面有一个**visit_count**，大家或许会认为，八宫格中的站点排序是根据访问次数来的吧，但是这紧紧猜对了一半，排序算法跟访问次数有关，但不全部依赖于它。好了，关子卖了这么多了，接下来就一起来看一下chromium中八宫格站点排序算法吧。

    days_ago = (now - timeslot).InDays();
    day_visits_score = 1.0 + log(visit_count));
    recency_boost = 1.0 + (2.0 * (1.0 / (1.0 + days_ago/7.0)));
    score = recency_boost * day_visits_score;
    
看起来很简单吧，其中**days_ago**是入库时间距离现在的天数，通过这个值来计算出一个**recency_boost**值，然后根据访问次数**visit_count**计算出的**day_visits_score**，与recency_boost一起计算出score，这个score值就是八宫格中站点排序的依据。

这个排序算法很简单，但是设计的却很巧妙，访问次数造成的影响不是线性的，距离近的站点优先级要高一些，同时兼顾了访问次数和时间，但是使用过chrome的人会发现一个问题，通过点击八宫格上的站点，访问次序却不会发生改变，这是因为谷歌对增加访问次数的方式做了限制，只有通过手动输入和书签栏跳转的站点，访问次数才会增加，这其中又增加了人为的因素，谷歌认为主动去跳转到一个站点，这样子会增加这个站点的比重，通过时间、访问次数、访问方式这三个，来确保八宫格站点的排序比较合适吧，但也仅仅是合适，这个里面肯定也会有不合理的地方，例如一些不知情的用户通过八宫格上快捷方式不断去访问一个站点，但是这个站点的排序可能会被一个偶尔通过地址栏输入的站点所挤掉。但是这种方式是用户不愿意看到的，这个算法在时间、访问次数、访问方式上或许折中的比较恰当，但是绝大部分用户可能不会看重访问方式的差异，这个是否可以改进呢，改进过后是否会有新的问题存在呢？