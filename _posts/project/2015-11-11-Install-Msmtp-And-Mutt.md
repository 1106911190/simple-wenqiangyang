---
layout: post
title: 在树莓派中使用msmtp+mutt发送邮件
category: project
description: 在Linux使用msmtp和mutt发送邮件，包括树莓派，centos和Ubuntu。
---

这个问题的一开始只是为了解决在树莓派开机不知道其IP地址的情况下写一个脚本让它自动发送一个邮件报告自己的IP，然后好使用ssh登陆上去，但是我在这里简化一下，只用树莓派发送邮件。      
其实发送邮件的方法有很多，直接使用Python就可以发送邮件，或者是使用其他的一些邮件服务，但是在这里我们采用msmtp和mutt，因为这样最简单。    

## 安装msmtp和mutt
我是使用`pacman`来安装的，因为树莓派的`raspbian`系统是基于`debian`，而`debian`的包管理工具是`apt-get`,不过这个不太好用，这里推荐使用`pacman`，当然先用`sudo apt-get install pacman`来安装这个包管理工具，然后就可以开始我们接下来的安装了。

```bash
pacman -S msmtp
pacman -S mutt
```

## 配置msmtp和mutt
在用户根目录下创建`.msmtprc`，这是msmtp的配置文件，内容如下。

```
account default
host smtp.126.com
from xxx@126.com
auth plain
user xxx@126.com
password your_password
logfile /var/log/msmtp.log
```

在这里，也可以设定多个账户`account XXX`，然后使用`sccount default XXX`来选择默认账户。    
然后更改这个文件的权限`sudo chmod 600 .msmtprc`，因为这个文件只能有root读写权限，而默认情况下是其他用户也有读的权限。   
这里会将每一次发送邮件的日志保存在`/var/log/msmtp.log`,不过可能会有权限问题，自己创建一个并改一下权限即可   
最后跟这个一样在根目录下创建`.muttrc`文件，这是mutt的配置文件，内容如下。   

```
set sendmail="/usr/bin/msmtp"
set use_from=yes
set realname="Windard"
set editor="vim"
```

其中realname是发件人的名字，接收到的邮件中会显示出来。   

## 发送邮件
发送邮件`echo "This is test" | mutt -s "Test" me@wenqiangyang.com`.    
`This is test`是邮件的内容，`Test`是邮件的主题，`me@wenqiangyang.com`是收件人的地址。    

如果想要加上附件，在上一个语句中加上`-a 附件名称`即可。     
例如`echo "This is test" | mutt -s "Test" me@wenqiangyang.com -a test.txt`   
如果需要发送多个邮件，只需要在命令后面多加几个收件人即可。      
例如`echo "This is test" |mutt -s "Test" me@wenqiangyang.com 18607571914@163.com`    
如果想要发送读取文本里的内容，采用管道的写法即可。    
例如`cat test.txt|mutt -s "Test" me@wenqiangyang.com`     

还有一种发送邮件的方式`msmtp me@wenqiangyang.com`，然后就会进入编辑模式，按`Ctrl`+`D`退出编辑，然后邮件就会发送出去。    

>在编辑模式中使用`From:1106911190@qq.com`来表示发件人。 <br>
>使用`To:me@wenqiangyang.com`来表示收件人。 <br>
>使用`Subject:Test`来表示邮件主题 <br>
>再接下来的任意文本就是邮件内容了。 <br>
>如果你没有上面的那些的话，那么你的全部文本都会被当做邮件内容。 <br>
>![msmtp_demo.jpg](/images/msmtp_demo.jpg) <br>

## 在centos和Ubuntu在安装msmtp和mutt   

### centos   

#### 使用yum安装   

```
sudo yum install msmtp
sudo yum install mutt
```

#### 使用源码编译

配置文件   
在根目录下创建`.msmtprc`和`.muttrc`。   

`.msmtprc`      

```
defaults
logfile /var/log/msmtp.log
account windard
host smtp.qq.com
from 1106911190@qq.com
auth login
user 1106911190@qq.com
password XXXXXX
account default : windard
```

`.muttrc`     

```
set sendmail=”/usr/bin/msmtp”
set use_from=yes
set realname=”windard”
set editor=”vim”
```

在都配置好了之后，就可以使用`echo "This is test" |mutt -c "Test" me@wenqiangyang.com`  
即可发送邮件。    

### Ubuntu
使用apt-get安装

```
sudo apt-get install msmtp
sudo apt-get install mutt
```

然后配置和使用与centos基本一致，与上面的树莓派也一样。    


但是我的centos在最后的mutt却不能使用，报出来`account default not found:no configuration file available`。      
估计是因为我的msmtp是编译安装的原因，没有编译安装好，找不到配置文件。    
不过用这个才可以`sudo echo “Subject: test msmtp\r\n\r\nThis is a test.” |msmtp -d -C /etc/msmtprc -t me@wenqiangyang.com`，手动设定配置文件的位置`/etc/msmtprc`。   
但是这样的话只有主题却没有内容，去掉Subject，就只剩内容没有主题了。       


应该`msmtprc`的位置是在`/usr/local/msmtp/etc/msmtprc`，然而我放在这里了也没有用。     
还有我这样也是可以进入msmtp的编辑模式进行编辑邮件发送的`/usr/local/msmtp/bin/msmtp me@wenqiangyang.com`，然后编辑邮件就可以发送了。      
装软件还是应该直接用包管理工具的。。。。。       


*防止被当成垃圾邮件*    
在使用mutt的时候，有几次明明发送出去了，但是却没有收到邮件，在邮箱里一看，原来是被当做了垃圾邮件并没有发送成功。     
所以为了防止被当成垃圾邮件，在配置`mutt`的时候，在`.muttrc`里面加上这两句。        

```
set from="1106911190@qq.com"
set envelope_from="yes"
```


参考链接：      
[烂泥：ubuntu下配置msmtp+mutt发送邮件](http://www.ilanni.com/?p=10589)

[树莓派开机发送IP到邮箱](http://lightless.me/archives/Send-IP-To-Mail-On-Raspberry-Pi-Start-Up.html)

[[原创] 让树莓派自动上报IP地址到邮箱/Let Raspberry Pi to report its IP address via Email](http://www.codelast.com/?p=7858)

[让树莓派自动上报IP地址到邮箱/Let Raspberry Pi to report its IP address via Email](http://forum.eepw.com.cn/thread/258882/1/)

[Linux使用Mutt发送邮件/附件](http://www.isucc.me/129.html)