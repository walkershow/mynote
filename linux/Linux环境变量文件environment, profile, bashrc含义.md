

（1）/etc/profile： 此文件为系统的每个用户设置环境信息,当用户第一次登录时,该文件被执行. 并从/etc/profile.d目录的配置文件中搜集shell的设置。

（2）/etc/environment：是设置整个系统的环境，而/etc/profile是设置所有用户的环境，前者<font color=red>与登录用户无关</font>，后者与登录用户有关。(crontab 属于与登录用户无关)

（3）/etc/bashrc: 为每一个运行bash shell的用户执行此文件.当bash shell被打开时,该文件被读取。 

（4）~/.bash_profile: 每个用户都可使用该文件输入专用于自己使用的shell信息,当用户登录时,该文件仅仅执行一次!默认情况下,他设置一些环境变量,执行用户的.bashrc文件。 

（5）~/.bashrc: 该文件包含专用于你的bash shell的bash信息,当登录时以及每次打开新的shell时,该该文件被读取。

技巧：  
想保持多台机器的crontab一致，但变量值不完全相同，  
这个时候可以考虑将变量配置在/etc/environment中，这样crontab就可以相同了。  
  
  
如，机器1：  
A=123  
  
  
机器2：  
A=456  
  
  
两者的crontab配置：  
* * * * * echo "$A" > /x.txt  
  
  
一般不建议直接修改/etc/environment，而可采取在目录/etc/profile.d下新增一个.sh文件方式替代。  

但如果想crontab中生效，则只能修改/etc/environment，经测试/etc/profile.d方式不起作用。

  

注意：在/etc/environment设置的变量，在shell中并不生效，但crontab中有效