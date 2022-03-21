---
created: 2021-12-15T15:52:20 (UTC +08:00)
tags: [后端,Java]
source: https://juejin.cn/post/7031185816495849509
author: JavaGieGie
---

# 使用 JOIN 实现表关联，面试挂了！ - 掘金

> ## Excerpt
> 面试中被问到使用什么实现表关联，回答：JOIN，面试官：卒，到嘴的offer飞了，下次再也不这么回答了

---
「这是我参与11月更文挑战的第15天，活动详情查看：[2021最后一次更文挑战](https://juejin.cn/post/7023643374569816095 "https://juejin.cn/post/7023643374569816095")」

### 前言

记得有一次去面试，面试官问了一道关于数据库的问题，问题大致内容如下：

> 面试官：你平时经常写sql吗？
> 
> 我：肯定啊，哪天不得写个几千行。
> 
> 面试官：那你知道多表数据之间有关联，你是怎么查数据的呢？
> 
> 我：JOIN啊。
> 
> 面试官：好用吗？
> 
> 我：贼好用，我特别是LEFT JOIN，我用的贼熟练，天天写。
> 
> 面试官：emm......，那你回去等通知吧。

不知道大家有没有遇到过这种问题，或者说大家平时是如何处理多表关联的数据的，是不是也用JOIN语句来处理呢。

我们暂且不直接下定义说JOIN语句有啥问题，先来看一个简单的例子。

### 测试数据准备

这里我使用的是mysql数据库，由于测试需要我们先创建两张表t/t2，语句比较简单

```
CREATE TABLE `t` (
  `id` int(11) NOT NULL,
  `a` int(11) NOT NULL,
  `b` int(11) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `a` (`a`),
  KEY `b` (`b`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

CREATE TABLE `t2` (
  `id` bigint(19) NOT NULL,
  `a` int(11) DEFAULT NULL,
  `c` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
复制代码
```

再使用函数创建一些测试数据，我这里在 t 表中创建**一千万条**，在 t2 表中创建**一万条。**

```
CREATE DEFINER=`root`@`localhost` PROCEDURE `testdata`()
begin
 declare item int;
 set item=1;
 while(item<=10000000)do
 insert into t values(item, item, item);
 set item=item+1;
 end while;
end
复制代码
```

### JOIN测试

先写一句小学生都会的语句测一下，只是这里我们使用了**straight\_join**，目的是让MySQL 使用固定的连接方式执行查询，这样优化器只会按照我们指定的方式去 join。这样就固定了表t 是驱动表（外层表），t2 是被驱动表（内层表）：

```
select t.*,t2.* from t  straight_join t2 on t.a = t2.a 
复制代码
```

emm......不知道等了多久之后，还是没有结束，于是乎我就加了一个limit限制条件，只查询10条数据

```
select t.*,t2.* from t straight_join t2 on t.a = t2.a limit 10000,10
复制代码
```

..............................十分钟过去了，又是一次漫长的等待，放弃等待，直接停止。

我们将两张表反过来试一下

```
select t.*,t2.* from t2 straight_join t on t.a = t2.a limit 1000,10
复制代码
```

![[0da2500e4d174599b09622e1c9ac4fa1~tplv-k3u1fbpfcp-watermark.awebp]]

速度很麻溜，仅仅用了0.01秒，初步判断和驱动表、被驱动表的数量大小有关。

### 问题分析

想要搞明白为什么消耗这么长时间，就要先理解这个查询过程是怎么实现的，然后才能得出结论，JOIN究竟能不能被使用。

首先看一下执行计划

![[e39b7ddcda404d03a9a00b7929631e30~tplv-k3u1fbpfcp-watermark.awebp]]

其中比较重要的三个参数：

-   type：判断是全表扫描还是索引扫描，这里我们是 _ALL_ ，也就是全表扫描；
-   rows：找到最终结果的过程中，需要扫描的行数，越少越好；
-   Extra：不适合其他列显示的额外信息，本例中关注 _Block Nested Loop_，本文后半段会继续分析。
-   filtered：返回结果的行数占读取行数的百分比，越大越好。

**整个过程JOIN可以理解为下面两步：**

0.  将表t 全部数据加载到内存（join\_buffer，默认大小256k）中；
1.  扫描表t2，将 表t2 中的每一行数据取出，和join\_buffer中的数据对比，找到满足条件 _t.a = t2.a_的行，形成结果集。

![[4f527067e1bd4a088e44c9083ace9e07~tplv-k3u1fbpfcp-watermark.awebp]] 上述过程比较简单，对应JAVA中实现方式就是两层for循环，第一层遍历表t 中数据，将每一行数据分别和表t2的数据进行比较，最终得到结果集，所有的比较过程是在内存join\_buffer中进行比较，该算法被称为 _Block Nested-Loop。_

**时间复杂度：**

-   扫描行数：M+N，M驱动表数据，N被驱动表数据。
-   时间复杂度：M\*N。

从时间复杂度就可以看出这里非常消耗内存，例如本例中的比较次数就高达一千亿次，非常耗时。

### 初步优化

对数据库有过一点了解的小伙伴肯定问了，表t2 中为啥不给 字段a 加索引呢，加了不就快了吗。

没毛病，先不说原理是啥，我们就先尝试加一下索引，然后看看查询同样的sql耗时多少。

![[fa67fcf97642423daf6b66df4b7a0cc7~tplv-k3u1fbpfcp-watermark.awebp]]

oh my god，这快的可不是一点两点，现在查询一次仅仅耗时0.04s，简直快到飞起

再来看一下执行计划：

![[b0968a8bbfc5471da9e421546e1d7018~tplv-k3u1fbpfcp-watermark.awebp]]

此时表t依旧为全表扫描，不同的是 表t2 中的type是ref，也就是说此时我们是用索引，再来看一下整个执行过程：

0.  从表t 中读取一条数据R；
1.  取出数据R中的a字段的值，到表t2中查找；
2.  **根据索引a， 匹配满足条件 _t.a = t2.a_的行，形成结果集；**
3.  重复上述步骤，直至 表t 中的数据全部遍历完成。

可以发现，上面步骤还是双层嵌套，第一层遍历表t，分别取出表t中的数据去表t2中查找满足条件的数据，最终形成结果集，因为又用到了被驱动表的索引，这种算法被叫做_Index Nested-Loop Join_ ，简称**NLJ。**

![[708353ab152c4142a45118ca779affb4~tplv-k3u1fbpfcp-watermark.awebp]]

**那加了索引为什么会变得很快呢：** 通过驱动表匹配条件直接与内层表索引进行匹配，避免和被驱动表的每条记录进行比较， 从而利用索引的查询减少了对内层表的匹配次数，极大的提升了 join的性能。

**时间复杂度：**

-   单次查询消耗的时间复杂度为：_1 + 2_log2N，N为被驱动表t2 的行数。
-   总时间复杂度：M \* (1 + 2_log2M)_ ，M驱动表数据，N被驱动表数据，log2M为树的查找复杂度，因为是普通索引，所以查找当前树的全部字段时，需要再走一遍主键索引，所以查找一行数据复杂度为 1+2\*log2M。

### MySQL中JOIN算法对比分析

在MySQL的实现中，Nested-Loop Join有3种实现的算法：

-   **Simple Nested-Loop Join**

该算法就是嵌套循环，依次读取驱动表中的数据，对于每次取出的数据，再和被驱动表进行比较，比如本例中我们驱动表数据（10000000）\* 被驱动表数据（10000） = 100000000000 亿次的时间复杂度，**当然这种算法没有数据库会采用的**。

-   **Block Nested-Loop Join**

该算法就是我们没有新增索引时，MySQL的InnoDB引擎 就会使用这种算法，具体过程如上，时间复杂度和上述算法相同，都是M \* N，但是引用buffer\_join进行了优化。

-   **Index Nested-Loop Join**

该算法是增加索引后，MySQL会使用该算法，具体过程上面也已经介绍过。

### JOIN能不能用

如果我们使用 Index Nested-Loop Join 算法，其实是没有多大影响的，完全满足绝大多数场景的使用。

如果使用 Block Nested-Loop Join 算法，就会导致大量的扫描行数，尤其像本例中这种上千万，乃至上亿的数据量，这样会导致被驱动表扫描过多次，占用大量的系统资源和缓存资源，非常不建议使用。

至于开头提到的面试官的问题，其实是真的发生过的，面试官难道想得到的答案是在内存中处理表关联的数据，在sql执行时只查询单表吗，其实这种时间复杂度并没有减少，反而会增加sql执行次数，不知道大家是怎么使用的呢？

_下一章我们就要探讨一下，当使用JOIN时如何对其进行优化？以及本例中的缓存（join\_buffer）是什么，如何优化？以及其他提升查询效率的有效手段。_
