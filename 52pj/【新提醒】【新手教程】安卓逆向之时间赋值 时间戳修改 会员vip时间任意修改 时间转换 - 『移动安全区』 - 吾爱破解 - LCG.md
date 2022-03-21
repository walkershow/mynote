---
created: 2021-12-13T10:56:06 (UTC +08:00)
tags: [【新手教程】安卓逆向之时间赋值 时间戳修改 会员vip时间任意修改 时间转换]
source: https://www.52pojie.cn/
author: 芽衣
---

# 【新提醒】【新手教程】安卓逆向之时间赋值 时间戳修改 会员vip时间任意修改 时间转换 - 『移动安全区』 - 吾爱破解 - LCG - LSG |安卓破解|病毒分析|www.52pojie.cn

> ## Excerpt
> 不适合加固、混淆的app，对于新手来说难度过高安卓时间修改对于破解一些本地验证的app很好用，在线的没辙。一般来说大公司的app是在线验证的，因为这个貌似很消耗 ... 【新手教程】安卓逆向之时间赋值 时间戳修改 会员vip时间任意修改 时间转换 ,吾爱破解 - LCG - LSG |安卓破解|病毒分析|www.52pojie.cn

---
_本帖最后由 417788939 于 2020-7-13 17:21 编辑_

不适合加固、混淆的app，对于新手来说难度过高

  
安卓时间修改对于[破解](https://www.52pojie.cn/)一些本地验证的app很好用，在线的没辙。一般来说大公司的app是在线验证的，因为这个貌似很消耗服务器资源，比如pixiv，有钱就是为所欲为，当然你也是 ![[105559ugy2z1x4u34niu3k.png]]   
有不少软件有年费会员或者终身会员什么的，可以对关键地方进行赋值破解。本帖主要说明时间在smali里面的表达方法。

找到老巢  
![[4.gif]]  
反编译后一般搜索的方法名关键字有vip、isvip、endtime、starttime、getvip、userinfo、viptime等；字符串关键字有到期、剩余、开通、续费、isvip、非会员等，他也有可能加短杠【\_】混在里面。最常见的是找res字符串的ID，这个用的比较多。具体怎么找，这个每个app逻辑都不一样，而且大多需要跳转，而且如果不熟悉英文……![[16.gif]]

找到vip计算时间的老巢之后，接下来就是赋值了。

进行赋值破解  
![[4.gif]]  
在线时间戳转换：[http://www.beijing-time.org/shijianchuo/](http://www.beijing-time.org/shijianchuo/)

案例：[https://www.52pojie.cn/thread-1218307-1-1.html](https://www.52pojie.cn/thread-1218307-1-1.html)、[https://www.52pojie.cn/thread-1211527-1-1.html](https://www.52pojie.cn/thread-1211527-1-1.html)、[https://www.52pojie.cn/thread-1193093-1-1.html](https://www.52pojie.cn/thread-1193093-1-1.html)

> Unix时间戳简介：安卓里面的时间戳叫Unix时间戳，是一种时间表示方式，定义为从格林威治时间1970年01月01日00时00分00秒起至现在的总秒数。有些apk打开以后里面的文件最后修改时间是1970年的，就是因为这个原因。大多数smali的时间戳要转换成16进制的毫秒才能使用。

对某一个方法的时间赋值，比如【const-wide v0, 0x3bb2b0c6018L】，会员到期时间就是2099年12月31日。那么3bb2b0c6018怎么来的呢？也就是（2099年12月31日-1970年1月1日）×365天×24小时×60分钟×60秒×1000毫秒，转换成16进制就大概是那个数了。那个单位L是long的意思，数值类型。

当然数值不是越大越好，如果涉及到运算就不能随意赋值了。

 ![[115445yi2hlp2eyf6d64zl.png]] 

比如这款彩云天气app，对const-wide v6, 0x11840ad80L这个数值改大以后，本来是送2个月会员的，现在变成了20多天到期，而且重新登录也是20多天。改小时间越来越长，最终变成永久会员。如果把下面的判断if-ltz v5, :cond\_1删掉他也是永久会员。【cmp-long vx, vy, vz。比较vy和vz的long值并在vx存入int型返回值（比较操作，如果第一个操作数大于第二个操作数返回正值；如果两者相等，返回0；如果第一个操作数小于第二个操作数，返回负值。）】

再比如[https://www.52pojie.cn/thread-1218307-1-1.html](https://www.52pojie.cn/thread-1218307-1-1.html)

\[Asm\] _纯文本查看_ _复制代码_

1

2

3

`const-wide/16 v1,0x3e8`

`mul``-long p0, p0, v1`

计算的是中间这个p0的值，v1是乘数。原本v1是1000，改大以后倍率高了到期时间就会变得很恐怖。一般获取时间后会有计算代码，常用于数值转换。比如分钟转换小时。

获取手机系统时间固定smali代码（1970-01-01开始）：

1.  invoke-static {}, Ljava/lang/System;->currentTimeMillis()J  
    
2.    
    
3.  move-result-wide v×

_复制代码_

  
以后看见这行代码就知道他在获取系统时间了。然后下面一般会紧跟着计算代码，比如cmp-long、mul-long/2addr、div-int/2addr等，修改寄存器的数值对结果影响很大。

如果需要对时间进行运算，比如加减乘除，需要先对v(×+1)进行赋值，然后下一行再加入计算代码。

举例：系统时间加上1秒

1.  .method public static wuaipojie()J  
    
2.      .registers 4  
    
3.    
    
4.      .prologue  
    
5.      .line 11  
    
6.      invoke-static {}, Ljava/lang/System;->currentTimeMillis()J  
    
7.    
    
8.      move-result-wide v0【v0储存系统的时间】  
    
9.    
    
10.      const-wide/16 v2, 0x3e8【对v2进行赋值，10进制为1000】  
    
11.    
    
12.      add-long/2addr v0, v2【v0=v0+v2】  
    
13.    
    
14.      return-wide v0【该方法返回v0的计算结果】  
    
15.  .end method

_复制代码_

Java代码就是：

\[Java\] _纯文本查看_ _复制代码_

1

2

3

4

`public` `static` `long` `wuaipojie()`

 `{`

 `return` `System.currentTimeMillis() + 1000L;`

 `}`

甚至可以写个又有判断又有乘除的代码：

1.  .method public static wuaipojie()J  
    
2.      .registers 4  
    
3.    
    
4.      .prologue  
    
5.      .line 11  
    
6.      invoke-static {}, Ljava/lang/System;->currentTimeMillis()J  
    
7.    
    
8.      move-result-wide v0  
    
9.    
    
10.      const-wide/32 v2, 0x60bbf54e  
    
11.    
    
12.      cmp-long v0, v0, v2  
    
13.    
    
14.      if-gez v0, :cond\_10  
    
15.    
    
16.      .line 12  
    
17.      invoke-static {}, Ljava/lang/System;->currentTimeMillis()J  
    
18.    
    
19.      move-result-wide v0  
    
20.    
    
21.      .line 14  
    
22.      :goto\_f  
    
23.      return-wide v0  
    
24.    
    
25.      :cond\_10  
    
26.      invoke-static {}, Ljava/lang/System;->currentTimeMillis()J  
    
27.    
    
28.      move-result-wide v0  
    
29.    
    
30.      const-wide/16 v2, 0x34  
    
31.    
    
32.      add-long/2addr v0, v2  
    
33.    
    
34.      const-wide/16 v2, 0x2  
    
35.    
    
36.      xor-long/2addr v0, v2  
    
37.    
    
38.      goto :goto\_f  
    
39.  .end method

_复制代码_
