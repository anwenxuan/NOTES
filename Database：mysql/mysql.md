### 未整理





**MySQL使用**



[MySQL创建库以及表、字符集编码设置](https://www.cnblogs.com/doing-good/p/11943148.html)

docker 安装mysql 

![image-20220217073359331](image-20220217073359331.png)

**mysql**

事务：一组sql操作，要么都成功，要么都失败

连接池





创建库：

安装成功之后

1、cmd打开命令行；cd到MySQL安装目录下面的MySQL Server 5.7\bin位置；

2、mysql -u root -p;

3、enter password：登录

 

登录之后

1、show databases;　　//查看所有的库

2、create database '库名' default character set = 'utf8';　　//创建库并设置字符集编码为utf8

3、drop database '库名';　　//删除库

4、use '库名';　　//使用库

5、create table ‘表名’(id int(6),name char(10)) default character set = 'utf8'; 　//创建表

6、show table status;　　//查看库中所有的表

7、describe ‘表名’; 　//查看表结构



mysql 查看表有多少个字段

select count(*) from information_schema.COLUMNS where TABLE_SCHEMA='数据库名' and table_name='表名'



查看有多少值:

select count(*) from data_clean;



**mysql 建表时规定字符集**



**drop table if exists** `all`;



**create table** `all`

(

  **domainname char**(255) **character set** utf8 **COLLATE** utf8_general_ci **not null**

​    **primary key**,

  **timestamp datetime null**

)

  **charset** = latin1;



**create index** all_domainname_index

  **on** `all` (**domainname**);



**create index** all_timestamp_index

  **on** `all` (**timestamp**);



**INSERT INTO** DataSourceCollation.`all` (**domainname**, **timestamp**) **VALUES** (**'百度'**, **'2021-09-14 14:26:34'**);

**INSERT INTO** DataSourceCollation.`all` (**domainname**, **timestamp**) **VALUES** (**'www.google.com'**, **'2021-09-14 16:24:52'**);

**INSERT INTO** DataSourceCollation.`all` (**domainname**, **timestamp**) **VALUES** (**'www.alibaba.com'**, **'2021-09-14 16:28:21'**);

**INSERT INTO** DataSourceCollation.`all` (**domainname**, **timestamp**) **VALUES** (**'www.baidu.com'**, **'2021-09-14 16:28:21'**);

**INSERT INTO** DataSourceCollation.`all` (**domainname**, **timestamp**) **VALUES** (**'www.tencent.com'**, **'2021-09-14 16:28:21'**);