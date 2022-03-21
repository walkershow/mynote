---
created: 2021-12-15T16:08:41 (UTC +08:00)
tags: [MySQL,Java,后端]
source: https://juejin.cn/post/6984275678761844743
author: JavaFish
---

# 面试官：数据库自增 ID 用完了会咋样？ - 掘金

> ## Excerpt
> 01 前言 哈喽，好久没更新啦。因为最近在面试。用了两周时间准备，在 3 天之内拿了 5 个 offer，最后选择了广州某互联网行业独角兽 offer，昨天刚入职。这几天刚好整理下在面试中被问到有意思

---
哈喽，好久没更新啦。因为最近在面试。用了两周时间准备，在 3 天之内拿了 5 个 offer，最后选择了广州某互联网行业独角兽 offer，昨天刚入职。这几天刚好整理下在面试中被问到有意思的问题，也借此机会跟大家分享下。

这家企业的面试官有点意思，一面是个同龄小哥，一起聊了两个小时（聊到我嘴都干了）。他问了我一个有意（keng）思（b）问题：

> 数据库中的自增 ID 用完了该怎么办？

这个问题其实可以分为**有主键 & 无主键**两种情况回答。

国际惯例，先上张脑图：

![[b76301881bbf484a9624e30a780f7b13~tplv-k3u1fbpfcp-watermark.awebp]]

## 1.1 往期精彩

[MySQL 查询语句是怎么执行的？](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FlRY7b9iS_xDDuyKNQKUWSg "https://mp.weixin.qq.com/s/lRY7b9iS_xDDuyKNQKUWSg")

[MySQL 索引](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FZDM_ttWCstw0mUwGtUEciw "https://mp.weixin.qq.com/s/ZDM_ttWCstw0mUwGtUEciw")

[MySQL 日志](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FyG2pQW7qkTPF4TLBk8qgwQ "https://mp.weixin.qq.com/s/yG2pQW7qkTPF4TLBk8qgwQ")

[MySQL 事务与 MVCC](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2Fl62CAZ55ZU9f9fsLOQR71A "https://mp.weixin.qq.com/s/l62CAZ55ZU9f9fsLOQR71A")

[MySQL 的锁机制](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FcuD8QiadO64VcSncpY18KQ "https://mp.weixin.qq.com/s/cuD8QiadO64VcSncpY18KQ")

[MySQL 字符串怎么设计索引？](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2F71eMW6ejKX-E3zfU79jNQA "https://mp.weixin.qq.com/s/71eMW6ejKX-E3zfU79jNQA")

如果你的表有主键，并且把主键设置为自增。

在 MySQL 中，一般会把主键设置成 int 型。而 MySQL 中 int 型占用 4 个字节，作为有符号位的话范围就是 \[-2^31,2^31-1\]，也就是\[-2147483648,2147483647\]；无符号位的话最大值就是 2^32-1，也就是 4294967295。

下面以有符号位创建一张表：

```
CREATE TABLE IF NOT EXISTS `t`(
   `id` INT(11) NOT NULL AUTO_INCREMENT,
   `url` VARCHAR(64) NOT NULL,
   PRIMARY KEY ( `id` )
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
复制代码
```

插入一个 id 为最大值 2147483647 的值，如下图所示：

![[ab2cfa1b1011489093f6e8c571cf7173~tplv-k3u1fbpfcp-watermark.awebp]]

如果此时继续下面的插入语句：

```
INSERT INTO t (url) VALUES ('wwww.javafish.top/article/erwt/spring')
复制代码
```

结果就会造成主键冲突：

![[24e3f425bd354d3a8a8145165af4e038~tplv-k3u1fbpfcp-watermark.awebp]]

## 2.1 解决方案

虽说 int 4 个字节，最大数据量能存储 21 亿。你可能会觉得这么大的容量，应该不至于用完。但是互联网时代，每天都产生大量的数据，这是很有可能达到的。

所以，我们的解决方案是：**把主键类型改为 bigint，也就是 8 个字节**。这样能存储的最大数据量就是 2^64-1，我也数不清有多少了。反正在你有生之年应该是够用的。

PS：**单表 21 亿的数据量显然不现实，一般来说数据量达到 500 万就该分表了**。

另一种情况就是**建表时没设置主键**。这种情况，InnoDB 会自动帮你创建一个不可见的、长度为 6 字节的 row\_id，默认是无符号的，所以最大长度是 2^48-1。

实际上 InnoDB 维护了一个全局的 dictsys.row\_id，所以**未定义主键的表都共享该 row\_id，并不是单表独享**。每次插入一条数据，都把全局 row\_id 当成主键 id，然后全局 row\_id 加 1。

**这种情况的数据库自增 ID 用完会发生什么呢？**

1、创建一张无显示设置主键的表 t：

```
CREATE TABLE IF NOT EXISTS `t`(
   `age` int(4) NOT NULL
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
复制代码
```

2、通过 `ps -ef|grep mysql` 命令获取 mysql 的进程 ID，然后执行命令，通过 gdb 先把 row\_id 修改为 1。PS：没有 gdb 的，百度安装下

```
sudo gdb -p 16111 -ex 'p dict_sys->row_id=1' -batch
复制代码
```

出现下图就是没错的：

![[83d5c0d1e7e84ea1a7547ed345ec2a4a~tplv-k3u1fbpfcp-watermark.awebp]]

3、插入三条数据：

```
insert into t(age) values(1);
insert into t(age) values(2);
insert into t(age) values(3);
复制代码
```

此时的数据库数据：

![[1dea703138f243d4bf68e2f1624841c5~tplv-k3u1fbpfcp-watermark.awebp]]

4、gdb 把 row\_id 修改为最大值：281474976710656

```
sudo gdb -p 16111 -ex 'p dict_sys->row_id=281474976710656' -batch
复制代码
```

5、再插入三条数据：

```
insert into t(age) values(4);
insert into t(age) values(5);
insert into t(age) values(6);
复制代码
```

此事的数据库数据：

![[5a2d87b779a84fd3b475b77056adbc47~tplv-k3u1fbpfcp-watermark.awebp]]

分析：

-   刚开始设置 row\_id 为 1，插入三条数据 1、2、3 的 row\_id 也理应是 1、2、3；这是没问题的。
    
-   接着设置 row\_id 为最大值，紧跟着插入三条数据。这时的数据库结果是：4、5、6、3；你会发现 1、2 被覆盖了。
    
-   row\_id 达到后最大值后插入的值 4、5、6 的 row\_id 分别是 0、1、2；由于 row\_id 为 1、2 的值已存在，所以后者的值 5、6 会覆盖掉 row\_id 为 1、2 的值。
    

结论：row\_id 达到最大值后会从 0 重新开始算；前面插入的数据就会被后插入的数据覆盖，且不会报错。

数据库自增主键用完后分两种情况：

-   有主键，**报主键冲突**
-   无主键，InnDB 会自动生成一个全局的row\_id。它到达最大值后会从 0 开始算，遇到 row\_id 一样时，**新数据覆盖旧数据**。所以，我们还是尽量**给表设置主键**。

为什么我说这是个有意（keng）思（b）问题？

我的回答除了以上解决方法外，还提到在业务开发中，我们不会等到主键用完那天就已经分库分表了，基本不会遇到这种情况。

这时，面试官可能会问你分库分表咋处理，如果你不会就不要主动提了，点到即止。

-   blog.csdn.net/weixin\_39640090/article/details/113227742
-   blog.csdn.net/qq\_35393693/article/details/100059966
-   time.geekbang.org/column/article/69862

如果看到这里，喜欢这篇文章的话，请帮点个**好看**。

初次见面，也不知道送你们啥。干脆就送**几百本电子书**和**2021最新面试资料**吧。微信搜索**JavaFish**回复**电子书**送你 1000+ 本编程电子书；回复**面试**送点面试题；回复**1024**送你一套完整的 java 视频教程。

面试题都是有答案的，详细如下所示：有需要的就来拿吧，**绝对免费，无套路获取**。

![[992fd2eb75114a839aa6403640191c84~tplv-k3u1fbpfcp-watermark.awebp]]
