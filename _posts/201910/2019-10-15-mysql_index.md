---
layout: post
title:  "Mysql 索引原理与Mysql优化"
date:   2018-07-11 12:00:00

categories: mysql
tags: index mysql database
author: "XueGang Zhang"
image: /images/logo.png
comments: true
published: true
---

#### 背景介绍
不可否认的是，MySQL由于其性能高、成本低、可靠性好，已经成为最流行的开源数据库之一，随着MySQL的不断成熟，它也逐渐用于更多大规模网站和应用，国内大部分互联网公司在去IOE的过程中，用MySQL的居多，创业公司也大部分采用了MySQL。互联网的不断发展中，体验成为了企业决胜的一大要素，而体验有一个很重要的衡量指标就是影响速度。响应速度很大程度上瓶颈都在数据库的瓶颈，那极致的优化数据库的性能变得很重要。索引结构在Mysql的查询中起到了重要的作用，那了解其数据结构，并针对其优化变得异常重要。

#### 索引的数据结构

我们知道，数据库查询是数据库最重要的功能之一。我们都希望查询数据的速度能够尽可能的块，因此数据库系统的设计者会从查询算法的角度进行优化。我们知道MYSQL索引是`帮助MySQL高效获取数据的数据结构。它的本质就是数据结构，单独存储在磁盘上，用它来提高数据查询的效率。`，那么我们需要保证他尽可能少的执行磁盘IO操作。数据结构随着应用场景的不断增多，也不断的演化，从最简单的顺序查找、到二分查找、到二叉树的查找。仔细想一下，这些数据结构都有自己的特定应用场景。接下来我们大概说一下这几个数据结构的特点，然后来看看数据库为什么普遍选择B树来作为自己的索引数据结构。


**二叉查找树（Binary Search Tree）**

我们先来看下比较平衡的二叉树（Binary Search Tree）：

<img src="/assets/images/pictures/2019-10-15-mysql_index/BinarySearchTree.png" alt="二叉树" style="zoom:50%" />

对该二叉树的节点进行查找发现深度为1的节点的查找次数为1，深度为2的查找次数为2，深度为n的节点的查找次数为n，因此其平均查找次数为 (1+2+2+3+3+3) / 6 = 2.3次

这里有推荐两个比较好的算法的演示平台：[二叉树的构造过程演示](https://visualgo.net/zh/bst)；[二叉树的构造过程演示](https://www.cs.usfca.edu/~galles/visualization/flash.html) 我们可以在这里演示二叉树的构造和删改过程，我们不难发现，虽然在删除方面算法相对比较复杂，但是在查找、插入和删除的时候都比较快。但是它有个比较大的问题，二叉树的任意构造，会出现一种极端情况，如图：

<img src="/assets/images/pictures/2019-10-15-mysql_index/skewedleft.png" alt="二叉树" style="zoom:50%" />

这样二叉树，退化为线性表，导致树的高度过高，从而降低了查询效率。`那么如何解决二叉查找树多次插入新节点而导致的不平衡,是不是让树尽可能平衡就行了，如何让树平衡了这就产生了平衡二叉查找树（AVL Tree）`

**平衡二叉查找树（AVL Tree）**

为了避免插入新节点后导致的不平衡问题，AVL走向了极端，它给自己设定了一个约定，任何一个节点的左子树和右子树的深度之差不得超过1，当然为了实现这个约定，它每次插入删除数据通过不断的left rotation和right rotation来保证平衡。

<img src="/assets/images/pictures/2019-10-15-mysql_index/avl.png" alt="二叉树" style="zoom:50%" />

而旋转是非常耗时的。由此他只适合在插入和删除次数比较少，但是查找多的场景下。`那么如何解决这种旋转的问题，是不是让旋转次数少一点，折中处理下就好了，于是就有了红黑树`

**红黑树**

相对于要求严格的AVL树来说，它的旋转次数变少，为了实现这种旋转次数的减少，它也自己设定了一个约定`通过对任何一条从根到叶子的路径上各个节点着色的方式的限制，红黑树确保没有一条路径会比其它路径长出两倍。`， 我们可以看出它是一种弱平衡的二叉树。它基本长这个样子：

<img src="/assets/images/pictures/2019-10-15-mysql_index/redBlack.png" alt="二叉树" style="zoom:50%" />

因为这篇文章不是讲红黑树的，我们只需要知道，这个数据结构的一个特点，如果要搞懂红黑树的一个基本原理可以参考这个文章：
[五分钟搞懂什么是红黑树（全程图解）](https://www.toutiao.com/i6584714397543825927/?tt_from=weixin_moments&utm_campaign=client_share&wxshare_count=2&from=timeline&share_type=original&timestamp=1544587349&app=news_article&utm_source=weixin_moments&iid=53509037357&utm_medium=toutiao_android&group_id=6584714397543825927&pbid=6633976754150000142)，我们看到红黑树的高度是没有限制的，这样数据量大的时候，发生IO次数可能就会非常多，影响性能。`那么怎么解决IO次数多的问题呢，也很简单，让每一层存储更多的数据就好了，于是就有人想到了平衡N叉查找树`

**平衡多路查找树（B-Tree）/ (B+Tree)**

因为增加了叉数，树的高度就可以控制了，于是就产生了平衡N叉查找树。B树在提高了IO性能的同时并没有解决元素遍历的效率低下的问题，B+树只需要去遍历叶子节点就可以实现整棵树的遍历。而且在数据库中基于范围的查询是非常频繁的，而B树不支持这样的操作或者说效率太低。所以B+树产生了。


#### 执行计划

通过explain命令我们可以查看sql的执行计划，然后我们依次解读一下执行计划的列。

<img src="/assets/images/pictures/2019-10-15-mysql_index/mysqlExplain.png" alt="sql执行计划" style="zoom:50%" />

<table>
  <tr>
    <th>列名</th>
    <th>解读</th>
  </tr>
  <tr>
    <td>id</td>
    <td colspan="2">id用来表示执行顺序，id相同的为一组，先执行id数字大的组，然后执行数字小的组。在id相同的一组内，顺序由上而下执行。子查询和union操作产生新的id，普通的join不会产生新id </td>
  </tr>
  <tr>
    <td>select type</td>
    <td colspan="2">查询类型，如普通查询，子查询，union，物化视图等，对于只写单表的我们，意义不大，不再介绍</td>
  </tr>
  <tr>
    <td rowspan="7">type</td>
    <td>ALL：代表全表扫描（如果有limit，也会显示ALL，其实可能没有扫描全部的数据，扫描部分就停止了），
    </td>
    
  </tr>
  <tr>
    <td>index：代表索引全扫描，它的性能甚至不如ALL，使用这个一般是为了避免排序或者覆盖索引扫描。</td>
  </tr>
</table>



#### 索引匹配的原则

最左前缀匹配原则，非常重要的原则，mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配，比如a = 1 and b = 2 and c > 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整。

=和in可以乱序，比如a = 1 and b = 2 and c = 3 建立(a,b,c)索引可以任意顺序，mysql的查询优化器会帮你优化成索引可以识别的形式。

尽量选择区分度高的列作为索引，区分度的公式是count(distinct col)/count(*)，表示字段不重复的比例，比例越大我们扫描的记录数越少，唯一键的区分度是1，而一些状态、性别字段可能在大数据面前区分度就是0，那可能有人会问，这个比例有什么经验值吗？使用场景不同，这个值也很难确定，一般需要join的字段我们都要求是0.1以上，即平均1条扫描10条记录。

索引列不能参与计算，保持列“干净”，比如from_unixtime(create_time) = ’2014-05-29’就不能使用到索引，原因很简单，b+树中存的都是数据表中的字段值，但进行检索时，需要把所有元素都应用函数才能比较，显然成本太大。所以语句应该写成create_time = unix_timestamp(’2014-05-29’)。

尽量的扩展索引，不要新建索引。比如表中已经有a的索引，现在要加(a,b)的索引，那么只需要修改原来的索引即可。

不要为输出列加索引，`select ip_address from t_user_action_log where name='LiSi' group by action order by create_time`, 这个时候可以考虑增加在 name action create_time 列上，而不是 ip_address。



#### 总结



本文中还参考了如下内容：
    
[mysql官方手册](https://dev.mysql.com/doc/refman/5.6/en/innodb-deadlocks.html)

《高性能Mysql》

Google
