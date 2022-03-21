

# MySQL优化：如何避免回表查询？什么是索引覆盖？索引下推

> ## Excerpt
> 数据库表结构： create table user ( id int primary key, name varchar(20), sex varchar(5), index(name) )engin

---
数据库表结构：

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p></td><td><div><p><code>create</code> <code>table</code> <code>user</code> <code>(</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>id </code><code>int</code> <code>primary</code> <code>key</code><code>,</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>name</code> <code>varchar</code><code>(20),</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>sex </code><code>varchar</code><code>(5),</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>index</code><code>(</code><code>name</code><code>)</code></p><p><code>)engine=innodb;</code></p></div></td></tr></tbody></table>

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p></td><td><div><p><code>select</code> <code>id,</code><code>name</code> <code>where</code> <code>name</code><code>=</code><code>'shenjian'</code></p><p><code>select</code> <code>id,</code><code>name</code><code>,sex </code><code>where</code> <code>name</code><code>=</code><code>'shenjian'</code></p></div></td></tr></tbody></table>

**多查询了一个属性，为何检索过程完全不同？**

　　**什么是回表查询？**

　　**什么是索引覆盖？**

　　**如何实现索引覆盖？**

**哪些场景，可以利用索引覆盖来优化SQL？**

这些，这是今天要分享的内容。

_画外音：本文试验基于MySQL5.6-InnoDB。_

**一、什么是回表查询？**

这先要从InnoDB的索引实现说起，InnoDB有两大类索引：

-   聚集索引(clustered index)
    
-   普通索引(secondary index)
    

**InnoDB聚集索引和普通索引有什么差异？  
**

InnoDB**聚集索引**的叶子节点存储行记录，因此， InnoDB必须要有，且只有一个聚集索引：

（1）如果表定义了PK，则PK就是聚集索引；

（2）如果表没有定义PK，则第一个not NULL unique列是聚集索引；

（3）否则，InnoDB会创建一个隐藏的row-id作为聚集索引；

_画外音：所以PK查询非常快，直接定位行记录。_

InnoDB**普通索引**的叶子节点存储主键值。

　_画外音：注意，不是存储行记录头指针，MyISAM的索引叶子节点存储记录指针。_

举个栗子，不妨设有表：

　　_t(id PK, name KEY, sex, flag);_

_画外音：id是聚集索引，name是普通索引。_

表中有四条记录：

　　_1, shenjian, m, A_

　　_3, zhangsan, m, A_

　　_5, lisi, m, A_

　　_9, wangwu, f, B_

[![[885859-20190729184808306-758660222.png]]](https://img2018.cnblogs.com/blog/885859/201907/885859-20190729184808306-758660222.png)

两个B+树索引分别如上图：

　　（1）id为PK，聚集索引，叶子节点存储行记录；

　　（2）name为KEY，普通索引，叶子节点存储PK值，即id；

既然从普通索引无法直接定位行记录，那**普通索引的查询过程是怎么样的呢？**

通常情况下，需要扫码两遍索引树。

例如：

<table><tbody><tr><td><p>1</p></td><td><div><p><code>select</code> <code>* </code><code>from</code> <code>t </code><code>where</code> <code>name</code><code>=</code><code>'lisi'</code><code>;　</code></p></div></td></tr></tbody></table>

**是如何执行的呢？**

[![[885859-20190729184911699-676257427.png]]](https://img2018.cnblogs.com/blog/885859/201907/885859-20190729184911699-676257427.png)

如**粉红色**路径，需要扫码两遍索引树：

（1）先通过普通索引定位到主键值id=5；

（2）在通过聚集索引定位到行记录；

这就是所谓的**回表查询**，先定位主键值，再定位行记录，它的性能较扫一遍索引树更低。

**二、什么是索引覆盖****(Covering index)****？**

额，楼主并没有在MySQL的官网找到这个概念。

_画外音：治学严谨吧？_

借用一下SQL-Server官网的说法。

 [![[885859-20190729184925229-1323143660.png]]](https://img2018.cnblogs.com/blog/885859/201907/885859-20190729184925229-1323143660.png)

MySQL官网，类似的说法出现在explain查询计划优化章节，即explain的输出结果Extra字段为**Using index**时，能够触发索引覆盖。

[![[885859-20190729184944177-1702731545.png]]](https://img2018.cnblogs.com/blog/885859/201907/885859-20190729184944177-1702731545.png)

不管是SQL-Server官网，还是MySQL官网，都表达了：只需要在一棵索引树上就能获取SQL所需的所有列数据，无需回表，速度更快。

**三、如何实现索引覆盖？**

常见的方法是：将被查询的字段，建立到联合索引里去。

仍是之前中的例子：

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p></td><td><div><p><code>create</code> <code>table</code> <code>user</code> <code>(</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>id </code><code>int</code> <code>primary</code> <code>key</code><code>,</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>name</code> <code>varchar</code><code>(20),</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>sex </code><code>varchar</code><code>(5),</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>index</code><code>(</code><code>name</code><code>)</code></p><p><code>)engine=innodb;</code></p></div></td></tr></tbody></table>

第一个SQL语句：　　

 [![[885859-20190729185028557-1703422478.png]]](https://img2018.cnblogs.com/blog/885859/201907/885859-20190729185028557-1703422478.png)

<table><tbody><tr><td><p>1</p></td><td><div><p><code>select</code> <code>id,</code><code>name</code> <code>from</code> <code>user</code> <code>where</code> <code>name</code><code>=</code><code>'shenjian'</code><code>;　</code></p></div></td></tr></tbody></table>

能够命中name索引，索引叶子节点存储了主键id，通过name的索引树即可获取id和name，无需回表，符合索引覆盖，效率较高。

_画外音，Extra：__Using index__。_

第二个SQL语句：                 

[![[885859-20190729185053070-767208274.png]]](https://img2018.cnblogs.com/blog/885859/201907/885859-20190729185053070-767208274.png)

<table><tbody><tr><td><p>1</p></td><td><div><p><code>select</code> <code>id,</code><code>name</code><code>,sex </code><code>from</code> <code>user</code> <code>where</code> <code>name</code><code>=</code><code>'shenjian'</code><code>;</code></p></div></td></tr></tbody></table>

能够命中name索引，索引叶子节点存储了主键id，但sex字段必须回表查询才能获取到，不符合索引覆盖，需要再次通过id值扫码聚集索引获取sex字段，效率会降低。

_画外音，Extra：__Using index condition__。_

如果把(name)单列索引升级为联合索引(name, sex)就不同了。

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p></td><td><div><p><code>create</code> <code>table</code> <code>user</code> <code>(</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>id </code><code>int</code> <code>primary</code> <code>key</code><code>,</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>name</code> <code>varchar</code><code>(20),</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>sex </code><code>varchar</code><code>(5),</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>index</code><code>(</code><code>name</code><code>, sex)</code></p><p><code>)engine=innodb;</code></p></div></td></tr></tbody></table>

[![[885859-20190729185140811-2063536201.png]]](https://img2018.cnblogs.com/blog/885859/201907/885859-20190729185140811-2063536201.png)

可以看到：

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p></td><td><div><p><code>select</code> <code>id,</code><code>name</code> <code>... </code><code>where</code> <code>name</code><code>=</code><code>'shenjian'</code><code>;</code></p><p><code>select</code> <code>id,</code><code>name</code><code>,sex ... </code><code>where</code> <code>name</code><code>=</code><code>'shenjian'</code><code>;</code></p></div></td></tr></tbody></table>

都能够命中索引覆盖，无需回表。

_画外音，Extra：__Using index__。_

**四、哪些场景可以利用索引覆盖来优化SQL？**

**场景1：全表count查询优化**

[![[885859-20190729185205243-1779249721.png]]](https://img2018.cnblogs.com/blog/885859/201907/885859-20190729185205243-1779249721.png)

原表为：

_user(PK id, name, sex)；_

直接：

<table><tbody><tr><td><p>1</p></td><td><div><p><code>select</code> <code>count</code><code>(</code><code>name</code><code>) </code><code>from</code> <code>user</code><code>;</code></p></div></td></tr></tbody></table>

不能利用索引覆盖。

添加索引：

<table><tbody><tr><td><p>1</p></td><td><div><p><code>alter</code> <code>table</code> <code>user</code> <code>add</code> <code>key</code><code>(</code><code>name</code><code>);</code></p></div></td></tr></tbody></table>

就能够利用索引覆盖提效。

**场景2：列查询回表优化**

<table><tbody><tr><td><p>1</p></td><td><div><p><code>select</code> <code>id,</code><code>name</code><code>,sex ... </code><code>where</code> <code>name</code><code>=</code><code>'shenjian'</code><code>;</code></p></div></td></tr></tbody></table>

这个例子不再赘述，将单列索引(name)升级为联合索引(name, sex)，即可避免回表。

**场景3：分页查询**

<table><tbody><tr><td><p>1</p></td><td><div><p><code>select</code> <code>id,</code><code>name</code><code>,sex ... </code><code>order</code> <code>by</code> <code>name</code> <code>limit 500,100;</code></p></div></td></tr></tbody></table>

将单列索引(name)升级为联合索引(name, sex)，也可以避免回表。

**InnoDB聚集索引普通索引**，**回表**，**索引覆盖**

出处：[如何避免回表查询？什么是索引覆盖？](https://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651962609&idx=1&sn=46e59691257188d33a91648640bcffa5&chksm=bd2d092d8a5a803baea59510259b28f0669dbb72b6a5e90a465205e9497e5173d13e3bb51b19&mpshare=1&scene=1&srcid=&sharer_sharetime=1564396837343&sharer_shareid=7cd5f6d8b77d171f90b241828891a85f&key=abd60b96b5d1f2e52ca45314fb2c95a67fad7a457fe265562eb51a1c026389d3f28c52359f96e920368ab44a5d08ebcbbe2ded474be2ba70731ed8b5dcc5dd68cc0eceb4989a74fb04e5055c78af8d38&ascene=1&uin=MTAwMjA4NTM0Mw%3D%3D&devicetype=Windows+7&version=62060739&lang=zh_CN&pass_ticket=tXA4xc7SZYamLpGZz5B6JwJa1ZRvZ4bRlmzFhXwEKeOfloPLulU0O80gsIQUiONb "如何避免回表查询？什么是索引覆盖？")

