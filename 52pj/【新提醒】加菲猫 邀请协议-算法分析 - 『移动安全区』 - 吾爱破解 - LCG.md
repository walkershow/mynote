---
created: 2021-12-13T10:50:53 (UTC +08:00)
tags: [加菲猫 邀请协议-算法分析]
source: https://www.52pojie.cn/
author: yuanyxh
---

# 【新提醒】加菲猫 邀请协议-算法分析 - 『移动安全区』 - 吾爱破解 - LCG - LSG |安卓破解|病毒分析|www.52pojie.cn

> ## Excerpt
> [md]1.加菲猫影视1.6.22.小黄鸟3.MT管理器4.IDA[/md]软件是别人发的，且被修改过，应该是服务器验证，所以没有尝试本地破解，直接进行算法分析；本人技术有限 ... 加菲猫 邀请协议-算法分析 ,吾爱破解 - LCG - LSG |安卓破解|病毒分析|www.52pojie.cn

---
_本帖最后由 yuanyxh 于 2021-12-2 22:53 编辑_

1.加菲猫影视1.6.2  
2.小黄鸟  
3.MT管理器  
4.IDA

软件是别人发的，且被修改过，应该是服务器验证，所以没有尝试本地破解，直接进行算法分析；  
本人技术有限，有说的不对的地方请大佬们指正{:1\_936:} 。

一.黄鸟抓包  
打开黄鸟选择目标为加菲猫影视，打开加菲猫影视并输入邀请码提交，黄鸟抓到包后，打开 "http://jk.b557b8.com/App/AppUserInvitation/beOne" 这一条封包信息，  
 ![[145227o0xofcn69tcoax9a.jpg]] ![[145230gwwqfxpxv94kte9w.jpg]]   
从响应信息来看就是这一条，再看提交信息，我们要分析的有 "token","token\_id"和"request\_key",其中"request\_key"明显是经过加密处理的，且每次都不一样，那我们就先分析这个。二.MT静态分析  
使用MT反编译软件，搜索字符串"request\_key",只有两个结果，打开第二个，转成java看一下。  
 ![[154024bkflolpexkm9dgxp.jpg]]  ![[154029wj1jjwtjpr1sdprq.jpg]]  ![[154026pbh0bs4e1j4vkq4h.jpg]]   
可以看到在f方法内进行了加密处理，key和iv来自c层。三.IDA静态分析  
用IDA打开 "libnative-lib.so",在导出列表里搜索Java，找到对应java层的函数，  
 ![[155427knkdbcg5cmkmg5zd.png]]   
点进去看发现都是明文  
 ![[155642u5uittdafvn11fud.png]] ![[155644az77cu6ag6a6ugfk.png]]   
找个在线解密网站试一下，加密方式为"AES/CBC/PKCS5"(ps:找的解密网站不支持，不过通过试验"AES/CBC/PKCS7"也能正常解密)，密文编码为"HEX",输出结果如下：  
 ![[161848om6r6lienxyrn2rg.png]]   
其中，"code"为邀请码，"nt"很明显是时间戳，"ns"看起来也是加密过的，那我们继续使用MT进行分析。四.ns密文分析  
继续使用MT反编译，搜索字符串"ns"并勾选区分大小写及完全匹配，搜索到5个，打开最后一个，转成java，  
 ![[164442vwclcwlu9t35jzsi.jpg]]   
nt确实是时间戳，ns则是"com.video.test.utils.EncryptUtils"类"getEncryptKey"方法的返回值，传了两个参数："Context"和当前时间戳的字符串；从前面的分析得知这个方法是一个native方法，具体实现在c层，所以继续使用IDA分析，  
 ![[171833scpca2gzarq03qap.png]]   
可以看到就是获取一系列字符串然后进行MD5加密，由于本人技术有限，能静态分析出来的只有:包名，传进来的时间戳，"&z4Y!s!2br";"GetSignatureString"函数的返回值实在是分析不出来，所以采用动态调试的方式实时观察每个寄存器的值，在关键地方下断点后运行APP，再观察寄存器的值，得到结果如下：  
 ![[191720s3d7za13qyucbzjn.png]]  ![[191722vm4zl4laa6050rpm.png]]   ![[191725npxohobbs8tzrpas.png]]   ![[191727ao66fqco13aba6a8.png]]  ![[191717ohw5tg2xxdhmhu00.png]]   
那么"ns"就是由：包名 + "1A060008D770327E3BC1521FAB2114C788B77435749590CBF4DA5B97512AC7FA" + " &z4Y!s!2br " + 时间戳 MD5加密得来的，功能就是验证请求是否有效。五.token及token\_id的分析  
MT反编译后搜索"token"和"token\_id",可以搜索到，但是一直分析不出来源头，不过分析代码知道每次使用时是从"SharedPreferences"获取的，那我们直接在软件的数据目录"shared\_prefs"文件夹进行搜索，  
 ![[205202pdx633553z199zzd.jpg]]   
这样就有思路了，有获取就有写入，只需要找到写入的地方就能分析出来，继续反编译软件搜索，搜索目标换成"userToken"及"userTokenid",但是通过分析依旧找不到源头，  
 ![[210626etjlloh2hjl4gjhy.jpg]]  ![[211029nf8ghzhb9bfws95k.jpg]]  ![[210621r47rkh6hblr1jwbp.jpg]]   
查找"setToken"及"setToken\_id"的调用无法找到，那就只能换一个思路了，能看到上图中有"login success"的字样，猜测是发送登录请求后返回的"token"和"token\_id",继续使用黄鸟抓包，并找到 "http://jk.256537.com/App/User/newLogin" 这一条封包，信息如下：  
 ![[213036rip1fff1vqfv0v1f.jpg]] ![[213038ebhllpjla6s6bhml.jpg]]   
其中请求信息中的"token"和"token\_id"新用户应该为"no",但是我在写教程的时候抓不到包了，所以用的是以前的抓包信息，我们继续使用解密网站解密"request\_key"和"response\_key"，第一张图片是响应的，第二张是请求，  
 ![[214028sovm0vco2ov9v92m.png]] ![[214032rc1hhsz9espe9qh9.png]]   
现在明确了"token"和"token\_id"是通过发送登录请求后返回的，而登录请求中："new\_key"是设备id（通过之前的截图能看到），"old\_key"通过多次试验是固定不变的，"ns"已经分析过，"nt"是时间戳，其他的不需要改变；到这里邀请协议所需要的信息全部分析完成。七.结语  
写了一下午，越写越糊涂，感觉写的好烂，希望看到的大佬们轻点骂{:301\_980:}  
下面是从网上搜索，拼拼补补写的一个py脚本，经过大佬[@正己](https://www.52pojie.cn/home.php?mod=space&uid=1109458) 优化的：  

 _复制代码_ _隐藏代码_ `import binascii
import re
import requests
import time
import random
import hashlib
from Cryptodome.Cipher import AES

AESKEY = '8jhM5h6dezq4QifP'  
AESIV = 'tho3aAHJyZCWAfTG'  

class AESTool:
    def __init__(self):
        self.key = AESKEY.encode('utf-8')
        self.iv = AESIV.encode('utf-8')

    def pkcs7padding(self, text):
        """
        明文使用PKCS7填充
        """
        bs = 16
        length = len(text)
        bytes_length = len(text.encode('utf-8'))
        padding_size = length if (bytes_length == length) else bytes_length
        padding = bs - padding_size % bs
        padding_text = chr(padding) * padding
        self.coding = chr(padding)
        return text + padding_text

    def aes_encrypt(self, content):
        """
        AES加密
        """
        cipher = AES.new(self.key, AES.MODE_CBC, self.iv)
        
        content_padding = self.pkcs7padding(content)
        
        encrypt_bytes = cipher.encrypt(content_padding.encode('utf-8'))
        
        result = binascii.b2a_hex(encrypt_bytes).upper()
        return result

    def aes_decrypt(self, content):
        """
        AES解密
        """
        generator = AES.new(self.key, AES.MODE_CBC, self.iv)
        content += (len(content) % 4) * '='
        
        decrpyt_bytes = binascii.a2b_hex(content)  
        meg = generator.decrypt(decrpyt_bytes)
        
        try:
            result = re.compile('[\\x00-\\x08\\x0b-\\x0c\\x0e-\\x1f\n\r\t]').sub('', meg.decode())
        except Exception:
            result = '解码失败，请重试!'
        return result

def invite():
    
    model_name = ["oppo-pedm00","oppo-peem00","oppo-peam00","oppo-x907","oppo-x909t",
                      "vivo-v2048a","vivo-v2072a","vivo-v2080a","vivo-v2031ea","vivo-v2055a",
                      "huawei-tet-an00","huawei-ana-al00","huawei-ang-an00","huawei-brq-an00","huawei-jsc-an00",
                      "xiaomi-mi 10s","xiaomi-redmi k40 pro+","xiaomi-mi 11","xiaomi-mi 6","xiaomi-redmi note 7",
                      "meizu-meizu 18","meizu-meizu 18 pro","meizu-mx2","meizu-m355","meizu-16th plus",
                      "samsung-sm-g9910","samsung-sm-g9960","samsung-sm-w2021","samsung-sm-f7070","samsung-sm-c7000",
                      "oneplus-le2120","oneplus-le2110","oneplus-kb2000","oneplus-hd1910","oneplus-oneplus a3010",
                      "sony-xq-as72","sony-f8132","sony-f5321","sony-i4293","sony-g8231",
                      "google-pixel","google-pixel xl","google-pixel 2","google-pixel 2 xl","google-pixel 3"]
    random_model_name = random.choice(model_name)

    
    aes = AESTool()

    
    t = str(int(round(time.time() * 1000)))
    
    device_id = "".join(random.choice("0123456789ABCDEF") for i in range(32))
    ns = 'com.jfm2110152DAA94115DC5C48038693654FFCC3AA095CBD093165B47BD7F15C8F83CA1BC9B&z4Y!s!2br' + t
    MD5ns = hashlib.md5(ns.encode(encoding='UTF-8')).hexdigest()
    Request_key = '{"new_key":"' + device_id + '","phone_type":"1","ns":"' + MD5ns + '","nt":"' + t + '","old_key":"YYqIkUniVrkDAACxRfIkIvsY","recommend":""}'
    MD5Request_key = aes.aes_encrypt(Request_key)
    HexMD5Request_key = MD5Request_key.decode('unicode_escape')
    body = 'token=no&token_id=no&phone_type=1&versions_code=1402&phone_model='+ random_model_name +'&request_key=' + HexMD5Request_key + '&app_id=1&ad_version=1'
    header = {
        'Cache-Control': 'no-cache',
        'Version': '210930',
        'channel_code': 'xc_tg18',
        'Referer': 'http://jk.b557b8.com',
        'Content-Type': 'application/x-www-form-urlencoded',
        'Host': 'jk.b557b8.com',
        'User-Agent': 'okhttp/3.12.0'
    }
    urls = 'http://jk.256537.com/App/User/newLogin'
    Response_body = requests.post(url=urls, data=body, headers=header).text
    Response_body_Encrypt = Response_body[49:241]
    Response_body_Decrypt = aes.aes_decrypt(Response_body_Encrypt)
    token_id = Response_body_Decrypt[13:22]
    token = Response_body_Decrypt[33:65]

    
    Request_key2 = '{"code":"2UDYUB","ns":"' + MD5ns + '","nt":"' + t + '"}'  
    Request_key_Encrypt2 = aes.aes_encrypt(Request_key2)
    HexRequest_key_Encrypt2 = Request_key_Encrypt2.decode('unicode_escape')
    body2 = 'token=' + token + '&token_id=' + token_id + '&phone_type=1&versions_code=1402&phone_model=' + random_model_name + '&request_key=' + HexRequest_key_Encrypt2 + '&app_id=1&ad_version=1'
    urls2 = 'http://jk.b557b8.com/App/AppUserInvitation/beOne'
    Response_body = requests.post(url=urls2, data=body2, headers=header) 
    print(Response_body.text)

if __name__ == '__main__':
    j = 0
    for i in range(50):  
        time.sleep(random.randint(1, 15))  
        invite()
        print("已刷{}次".format(i))
        j += 1
        if j == 50:
            break`
