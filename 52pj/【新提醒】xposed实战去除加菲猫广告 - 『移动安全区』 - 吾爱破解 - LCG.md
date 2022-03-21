---
created: 2021-12-13T10:51:40 (UTC +08:00)
tags: [xposed实战去除加菲猫广告]
source: https://www.52pojie.cn/
author: 正己
---

# 【新提醒】xposed实战去除加菲猫广告 - 『移动安全区』 - 吾爱破解 - LCG - LSG |安卓破解|病毒分析|www.52pojie.cn

> ## Excerpt
> [md]> 1.加菲猫影视1.6.2> 2.HttpCanary> 3.jadx-gui> 4.Android Studio> 5.MT管理器# **一、前言：**---前几天简简在研究加菲猫影视，我一起看了下。这个 ... xposed实战去除加菲猫广告 ,吾爱破解 - LCG - LSG |安卓破解|病毒分析|www.52pojie.cn

---
> 1.加菲猫影视1.6.2  
> 2.HttpCanary  
> 3.jadx-gui  
> 4.Android Studio  
> 5.MT管理器

___

前几天简简在研究加菲猫影视，我一起看了下。这个软件的前身叫南瓜影视，之前很多人研究，不过自从它换皮又上了服务器校验之后，我便很少看到有人研究了。此次教程便以去广告为主，搭配简简的  
[帖子](https://www.52pojie.cn/thread-1555004-1-1.html)  
理论上来说，可以白嫖一阵子了。  
一般来说，以前的去广告都是修改xml为主，我这次便想挑战一下，用xposed写一个去广告插件。那么，教程开始了。

___

这个软件广告是真的多，而且还有一些是少儿不宜的。  
先来看一下最终的效果图  
![[Screenshot_2021-12-04-18-32-34-611_com.jpg]]  
它总共有八处广告，我一一来述。

___

## **1、启动广告**

先看看广告长什么样子  
![[IMG_20211203_115829.jpg]]  
启动广告采用的是activity纪录分析法，从中可以看出软件的activity的过程是由SplashActivity->AdActivity->HomepageActivity，由此可见，在AdActivity中必然有一个方法会启动HomepageActivity。  
![[Screenshot_2021-12-03-12-51-36-364_bin.png]]  
打开jadx-gui，很快就发现了调用的方法  
代码如下：

 _复制代码_ _隐藏代码_`public void b2() {
        ARouter.getInstance().build("/homepage/activity").withTransition(R.anim.anim_activity_enter_alpha, R.anim.anim_activity_exit_alpha).navigation(this);
    }` 

按x查找用例  
![[20211203172232.png]]  
发现都是在本类中调用，稍微分析了下，发现一个可以方法，代码如下：

 _复制代码_ _隐藏代码_`public void p1() {
        SplashBean splashBean = this.T;
        if (splashBean == null) {
            return;
        }
        if (splashBean.getPicUrl() == null) { 
            if (u3()) {
                b2();
            }
            finish();
            return;
        }
        TextView textView = this.mTvWebUel;
        if (textView != null) {
            textView.setVisibility(8);
            w.b("AdActivity", splashBean.getPicUrl());
            AdPresenter adPresenter = (AdPresenter) this.Q;
            String picUrl = splashBean.getPicUrl();
            j.b(picUrl, "it.picUrl");
            ImageView imageView = this.mIvAd;
            if (imageView != null) {
                adPresenter.u(picUrl, imageView);
            } else {
                j.q("mIvAd");
                throw null;
            }
        } else {
            j.q("mTvWebUel");
            throw null;
        }
    }` 

跳转到getPicUrl方法看看，返回值是字符串也就是广告的url

 _复制代码_ _隐藏代码_`public String getPicUrl() {
        return this.picUrl;
    }` 

编写xpoesd模块，直接让其返回值为空即可

 _复制代码_ _隐藏代码_ `XposedHelpers.findAndHookMethod("com.video.test.javabean.SplashBean", classLoader, "getPicUrl", new XC_MethodReplacement() {
                    @Override
                    protected Object replaceHookedMethod(MethodHookParam methodHookParam) throws Throwable {
                        return null;
                    }
                });`

___

## **2.主页弹窗**

在解决启动广告后，我们进入主页。映入眼帘的就是两个弹窗广告  
长这个样子  
![[IMG_20211203_115538.jpg]]  
这个广告也同样是activity纪录分析法，这个弹窗是在HomepageActivity界面弹窗，那我们不妨搜一下dialog，因为一般这种弹窗都是属于dialog，果不其然，又发现了一个可疑的方法

 _复制代码_ _隐藏代码_`public void C0(List<HomeDialogBean> list) {
        if (!(list == null || list.isEmpty())) { 当传入的List参数不为空时，弹出主页弹窗，而这个List就是弹窗广告的图片url，但是因为晚上在写教程的软件死活不弹，我也没办法给你们验证
            ArrayList arrayList = new ArrayList();
            for (HomeDialogBean homeDialogBean : list) {
                HomeNoticeDialogFragment homeNoticeDialogFragment = null;
                homeDialogBean.getActivityType();
                if (homeDialogBean.getActivityType() == 2) {
                    homeNoticeDialogFragment = HomeNoticeDialogFragment.b0.a(homeDialogBean);
                }
                if (homeNoticeDialogFragment != null) {
                    arrayList.add(homeNoticeDialogFragment);
                    homeNoticeDialogFragment.q3(new b(arrayList));
                    homeNoticeDialogFragment.h3(new BaseHomeDialogFragment.a(arrayList) { 

                        
                        public final  List f15178b;

                        {
                            this.f15178b = r2;
                        }

                        @Override 
                        public final void a(BaseHomeDialogFragment baseHomeDialogFragment) {
                            HomepageActivity.this.u4(this.f15178b, baseHomeDialogFragment);
                        }
                    });
                }
            }
            if (!arrayList.isEmpty()) {
                ((BaseHomeDialogFragment) arrayList.get(0)).F2(I3(), "dialog0");
            }
        }
    }` 

从上面的逻辑可以看出来，如果List的值为空，那它就不会弹弹窗，我们可以把List置空,当然也可以直接把方法置空,这并不会影响软件的使用  
代码如下:

 _复制代码_ _隐藏代码_ `XposedHelpers.findAndHookMethod("com.video.test.module.homepage.HomepageActivity", classLoader, "C0", List.class, new XC_MethodReplacement() {
                    @Override
                    protected Object replaceHookedMethod(MethodHookParam methodHookParam) throws Throwable {
                        return null;
                    }
                });`

___

## **3.轮播图广告**

老样子先看看长什么样子  
![[IMG_20211203_115325.jpg]]  
这个需要我们先用小黄鸟抓个包，过滤一下ad，这几个都是关键的url请求  
这里的响应都是密文，所以我们直接用简简帖子里的来解密，可以得知轮播图广告弹窗的url就是bannerInfo  
![[IMG_20211203_213841.jpg]]  
在代码中搜索url中的Ad/bannerInfo，结果只有一个，而且是retrofit2的接口类，这就比较麻烦，没办法直接进行hook，只能找到关键的实例去修改  
![[20211203214428.png]]  
多次按x查找用例，因为层级有点多，我这里做了一个堆栈流程图，你们可以参考一下  
![[20211204153804.png]]  
最终聚焦一下这个s方法，代码如下：

 _复制代码_ _隐藏代码_`public void s(List<? extends BannerBean> list) {
        j.f(list, "list");
        if (!list.isEmpty()) {
            int size = list.size();
            ArrayList arrayList = new ArrayList(size);
            ArrayList arrayList2 = new ArrayList(size);
            for (BannerBean bannerBean : list) {
                String slidePic = bannerBean.getSlidePic();
                j.b(slidePic, "bannerBean.slidePic");
                arrayList.add(slidePic);
                String a2 = h0.a(bannerBean.getBannerContent());
                j.b(a2, "StringUtils.emptyIfNull(bannerBean.bannerContent)");
                arrayList2.add(a2);
                String str = this.f8643d;
                                                                
                w.f(str, "picUrls == " + bannerBean.getSlidePic() + " vodId == " + bannerBean.getVodId() + " bannerContent : " + bannerBean.getBannerContent());
            }
            ((e.m.a.r.j0.a) this.f8265b).f0(arrayList, arrayList2, list);
            return;
        }
        ((e.m.a.r.j0.a) this.f8265b).d2();
    }` 

最终这些数据又转为List传入f0方法中，f0传入的三个参数也就分别代表了广告图片url、广告id、广告文字，这三个数据要同时进行修改，不然会报错，这里我们就去除两个广告，然后对其中一个广告的url和文字进行修改（从轮播图中我们可以知道，第一、第二、第四个都是广告）  
注意索引！  
xposed代码如下：

 _复制代码_ _隐藏代码_ `XposedHelpers.findAndHookMethod("com.video.test.module.videotype.BaseVideoTypeListFragment", classLoader, "f0",List.class,List.class,List.class,
                new XC_MethodHook() {
                    @Override
                    protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                        super.beforeHookedMethod(param);
                        List URL = (List) param.args[0];
                        URL.set(0,"https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fc-ssl.duitang.com%2Fuploads%2Fitem%2F201810%2F22%2F20181022224306_ydqyt.thumb.1000_0.jpg&refer=http%3A%2F%2Fc-ssl.duitang.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1641197983&t=178238b2c2ce9ef7398dcc9876b54e42");
                        URL.remove(1);
                        URL.remove(3);
                        param.args[0] = URL;
                        List content = (List) param.args[1];
                        content.set(0,"正己修改");
                        content.remove(1);
                        content.remove(3);
                        param.args[1] = content;
                        List layout = (List) param.args[2];
                        layout.remove(1);
                        layout.remove(3);
                        param.args[2] = layout;
                    }
                });`

效果图：  
![[Screenshot_2021-12-04-16-23-06-009_com.jpg]]

___

## **4.公告广告**

就是在轮播图下面的公告广告，同样是通过请求的url定位，直接分析有点混乱，我还是做个流程图，不得不说接口类找实例真麻烦  
![[20211204165613.png]]  
分析一下重写的w2代码：

 _复制代码_ _隐藏代码_`public void w2(List<HomePageNoticeBean> list) {
                                
        if (list == null || list.isEmpty()) {
            this.mSwitcherNotice.d();  
            this.mSwitcherNotice.setVisibility(8);  
            return;
        }
        if (this.R == null) {
            this.R = new ViewSwitcher.ViewFactory() { 
                @Override 
                public final View makeView() {
                    return VideoRecommendFragment.this.z3();
                }
            };
        }
        if (this.S == null) {
            this.S = new a();
        }
        this.mSwitcherNotice.setFactory(this.R);
        this.mSwitcherNotice.setTextBinder(this.S);
        this.S.d(list);
        this.mSwitcherNotice.c();
    }` 

由此可知，只需将传入的参数置空即可，xposed代码如下：

 _复制代码_ _隐藏代码_ `XposedHelpers.findAndHookMethod("com.video.test.module.videorecommend.VideoRecommendFragment", classLoader, "w2", List.class, new XC_MethodHook() {
                    @Override
                    protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                        super.beforeHookedMethod(param);
                        param.args[0] = null;
                    }
                });`

___

## **5.横幅广告**

老样子，搜索url中的关键词（这里说一下，我抓包的那张截图中横幅广告的标记和公告广告的标记写反了），所以横幅广告应该搜索  
Ad/noticeInfo  
![[20211204171942.png]]  
这里主要看一下com.video.test.module.videorecommend.VideoRecommendPresenter.z的代码：

 _复制代码_ _隐藏代码_`public  void y(List list) throws Exception {
        if (list.isEmpty()) { 
            ((m) this.f8265b).q0();
            return;
        }
        ((m) this.f8265b).a3(list);
        ((m) this.f8265b).h1();
    }` 

q0的方法重写之后就是一个很明显的隐藏布局的代码

 _复制代码_ _隐藏代码_`public void q0() {
        this.mRvModule.setVisibility(8);
    }`

所以xpoesd代码如下：

 _复制代码_ _隐藏代码_ `XposedHelpers.findAndHookMethod("com.video.test.module.videorecommend.VideoRecommendPresenter", classLoader, "y", List.class, new XC_MethodHook() {
                    @Override
                    protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                        super.beforeHookedMethod(param);
                        param.args[0] = null;
                    }
                });`

___

## **6.页面中间广告**

先看一下长什么样子  
![[IMG_20211203_115608.jpg]]  
搜索字符串/Ad/barsIndexAdInfo，堆栈图  
![[20211204173634.png]]  
在j方法中注意传入的第一个参数，当他不为2时都会显示广告，分别是在加载顶栏的热门、电影、电视剧、综艺、动漫所加载的广告对应的值（可以打印一下进行验证）

 _复制代码_ _隐藏代码_`public final g.b.m<AdInfoBean> j(int i2, Context context) {
        String str;
        if (i2 == 1) {
            str = m.a.l();
        } else if (i2 == 3) {
            str = m.a.j();
        } else if (i2 == 4) {
            str = m.a.n();
        } else if (i2 == 5) {
            str = m.a.a();
        } else if (i2 != 6) {
            g.b.m<AdInfoBean> B = g.b.m.B(AdInfoBean.EMPTY_INSTANCE);
            j.b(B, "Observable.just(AdInfoBean.EMPTY_INSTANCE)");
            return B;
        } else {
            str = m.a.p();
        }
        g.b.m<AdInfoBean> F = this.a.n(i2).e0(this.a.m(context, str), a.a).F(b.a);
        j.b(F, "mModel.getAd(pid)\n      \u2026InfoBean.EMPTY_INSTANCE }");
        return F;
    }` 

所以我们只需要将第一个参数写死即可

 _复制代码_ _隐藏代码_ `XposedHelpers.findAndHookMethod("com.video.test.module.videotype.BaseVideoTypeListPresenter", classLoader, "j", int.class,Context.class, new XC_MethodHook() {
                    @Override
                    protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                        super.beforeHookedMethod(param);
                        param.args[0] = 2;
                    }
                });`

___

## **7.播放页广告**

看一下  
![[IMG_20211203_115634.jpg]]  
根据抓包搜索字符串Ad/barsPlayAdInfo，堆栈图  
![[20211204182448.png]]  
代码如下：

 _复制代码_ _隐藏代码_`public void g1(AdInfoBean adInfoBean) {
        if (adInfoBean != null) { 
            e.m.a.p.b.d(this).load(adInfoBean.getAdPic()).listener(new l(adInfoBean)).centerCrop().into(this.mIvAd);
            ImageView imageView = this.mIvAd;
            imageView.setTag(imageView.getId(), adInfoBean);
            return;
        }
        this.mIvAd.setVisibility(8);
    }` 

所以xposed代码如下：

 _复制代码_ _隐藏代码_ `Class<?> personClass = XposedHelpers.findClass("com.video.test.javabean.AdInfoBean",classLoader);
 XposedHelpers.findAndHookMethod("com.video.test.module.player.PlayerActivity", classLoader, "g1", personClass,new XC_MethodHook() {
                    @Override
                    protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                        super.beforeHookedMethod(param);
                        param.args[0] = null;
                    }
                });`

___

## **8.个人页广告**

这个就当给一些新手同学留个小问题，效果如下：  
不要直接修改xml哦，尝试去推导堆栈，然后找到关键方法进行hook  
![[Screenshot_2021-12-04-18-32-56-891_com.png]]

___

写个教程真的废时间呀，第一次尝试去hook接口类的实例，真的很麻烦，也有可能是还有其他更好的方法俺还不会，如果有大佬有其他更好的思路，请赐教
