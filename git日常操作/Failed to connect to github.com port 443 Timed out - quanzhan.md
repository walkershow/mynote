---
created: 2022-01-30T09:42:24 (UTC +08:00)
tags: []
source: https://www.cnblogs.com/code1992/p/11220261.html
author: quanzhan
            关注 - 30
            粉丝 - 37
        
    
    
    
    
                +加关注
---

# Failed to connect to github.com port 443: Timed out - quanzhan - 博客园

> ## Excerpt
> Git Clone下载仓库代码的时候，出现以下情况 Failed to connect to github.com port 443: Timed out 解决办法： 输入 git config --

---
Git Clone下载仓库代码的时候，出现以下情况

Failed to connect to github.com port 443: Timed out

![[429935-20190721093531050-287602511.png]]

解决办法：

输入

git config --global http.proxy http://127.0.0.1:1080
 git config \--global https.proxy http://127.0.0.1:1080

再git clone，就能正常下载代码

![[429935-20190721093753474-2139206696.png]]

 如果无效，通过下面命令删除刚才的配置

git config --global --unset http.proxy


git config \--global --unset https.proxy

参考：

[IDEA连接Github时出现：Failed to connect to github.com port 443: Connection refused的解决方法](https://blog.csdn.net/weixin_42266790/article/details/86830692)
