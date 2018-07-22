# MySQL笔记

## 前言
>MySQL是一种 **关系**数据库管理系统，关系数据库将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就增加了速度并提高了灵活性。MySQL所使用的 SQL 语言是用于访问数据库的最常用标准化语言。MySQL 软件采用了 **双授权政策**，分为社区版和商业版，由于其 **体积小、速度快、总体拥有成本低**，尤其是 **开放源码**这一特点，一般中小型网站的开发都选择 MySQL 作为网站数据库。

## 一、数据库的分类
数据库通常分为 **层次式数据库**、**网络式数据库**和 **关系式数据库**三种。而不同的数据库是按不同的数据结构来联系和组织的。而在当今的互联网中，最常见的数据库模型主要是两种，即 **关系型数据库**和 **非关系型数据库**。

### 关系型数据库代表
- Oracle
- SQL Server
- MySQL

### 非关系型数据库代表
- MongoDB
- Redis

## 二、MySQL的存储引擎
MySQL数据库根据应用的需要准备了不同的引擎，不同的引擎侧重点不一样

- `InnoDB`： 事务型数据库的首选引擎，支持ACID事务，支持行级锁定, `MySQL 5.5` 起成为默认数据库引擎
- `MyISAM`：`MySQL 5.0` 之前的默认数据库引擎，最为常用。拥有较高的插入，查询速度，但不支持事务
- `BDB`：源自`Berkeley DB`，事务型数据库的另一种选择，支持`Commit` 和`Rollback`等其他事务特性
- `Memory`：所有数据置于内存的存储引擎，拥有极高的插入，更新和查询效率。但是会占用和数据量成正比的内存空间。并且其内容会在 `MySQL` 重新启动时丢失
- `Archive`：非常适合存储大量的独立的，作为历史记录的数据。因为它们不经常被读取。Archive 拥有高效的插入速度，但其对查询的支持相对较差`Federated` 将不同的 `MySQL` 服务器联合起来，逻辑上组成一个完整的数据库。非常适合分布式应用`Cluster/NDB` 高冗余的存储引擎，用多台数据机器联合提供服务以提高整体性能和安全性。适合数据量大，安全和性能要求高的应用CSV 逻辑上由逗号分割数据的存储引擎。它会在数据库子目录里为每个数据表创建一个 `.csv` 文件。这是一种普通文本文件，每个数据行占用一个文本行。CSV 存储引擎不支持索引。
- `BlackHole`：黑洞引擎，写入的任何数据都会消失，一般用于记录`binlog`做复制的中继`EXAMPLE` 存储引擎是一个不做任何事情的存根引擎。它的目的是作为 `MySQL` 源代码中的一个例子，用来演示如何开始编写一个新存储引擎。同样，它的主要兴趣是对开发者。`EXAMPLE` 存储引擎不支持编索引。

各个存储引擎的比较如下图：

![MySQL存储引擎](http://paav7duuk.bkt.clouddn.com/mysql%E5%AD%98%E5%82%A8%E5%BC%95%E6%93%8E.jpg)

## 三、MySQL数据库操作

### 创建数据库
  ```sql
  create database <数据库名>;
  ```
### 查看数据库
  ```sql
  show databases;
  ```
### 删除数据库
  ```sql
  drop database <数据库名>;
  ```
### 连接数据库
  ```sql
  use <数据库名>;
  ```
### 查看当前数据库的编码
```sql
use <数据库名>;
show variables like 'character_set_database';
```
或
```sql
show create database 数据库名;
```
### 修改当前数据库的编码
```sql
alter database <数据库名> character set utf8;
```
### 查看某个数据库的当前状态
```sql
 status; 
```
或
```sql
\s
```
## 数据表操作

### 创建数据表
  ```sql
  create table <表名> ( <字段名1> <类型1> [,..<字段名n> <类型n>]);
  ```
  例如：
  ```sql
  mysql> create table MyClass(
  > id int(4) not null primary key auto_increment,
  > name char(20) not null,
  > sex int(4) not null default '0',
  > degree double(16,2));
  ```
### 删除数据表
  ```sql
  drop table <表名>;
  ```
### 查看数据表
  ```sql
  describe <表名>;
  ```
### 向表里插入数据
  ```sql
  Insert into <表名>(字段列表) values (值列表);
  ```
### 查询表数据
  ```sql
  select <字段名> from <表名称> [查询条件];
  ```
### 删除表中的数据
  ```sql
  delete from <表名> where <表达式>
  ```
  例如：
  ```sql
  delete from students where id=10;
  ```
### 修改表中数据
  ```sql
  update <表名称> set 列名称=新值 where 更新条件;
  ```
  例如：
  ```sql
  //将id为5的手机号改为默认的
  update students set tel=default where id=5;
  //将所有人的年龄增加1
  update students set age=age+1;
  //将手机号为 13723887766 的姓名改为 "张果", 年龄改为 19
  update students set name="张果", age=19 where tel="13723887766";
  ```
### 删除表
  ```sql
  drop table 表名;
  ```
### 修改表结构
  #### 添加列
  ```sql
  alter table 表名 add 列名 列数据类型 [after 插入位置];
  ```
  例如：
  ```sql
  //在表的最后追加列 address: 
  alter table students add address char(60);
  //在名为 age 的列后插入列 birthday:
  alter table students add birthday date after age;
  ```
  #### 修改列
  ```sql
  alter table 表名 change 列名称 列新名称 新数据类型;
  ```
  例如：
  ```sql
  //将 name 列的数据类型改为 char(9): 
  alter table students change name name char(9) not null;
  ```
  #### 删除列
  ```sql
  alter table 表名 drop 列名称;
  ```
  例如：
  ```sql
  //删除 age 列: 
  alter table students drop age;
  ```
  #### 重命名表
  ```sql
  alter table 表名 rename 新表名;
  ```
  例如：
  ```sql
  //重命名 students 表为temp: 
  alter table students rename temp;
  ```
### 修改表名
```sql
rename table 原表名 to 新表名;
```
例如：
```sql
//在表MyClass名字更改为YouClass
mysql> rename table MyClass to YouClass;
```
## 其他命令
### 查看MySQL的编码
```sql
show variables like 'character%'; 
```
### 查看数据库支持的所有字符集
```sql
show character set;
```
或
```sql
show char set;
```
### 修改MySQL的编码
```sql
mysql> SET character_set_client='utf8';
mysql> SET character_set_connection='utf8'
mysql> SET character_set_results='utf8'
```
### 查看MySQL的版本
```sql
select version();
```
### 查看当前时间
```sql
select now(); 
```
### 显示年月日
  #### 显示日
  ```sql
  select DAYOFMONTH(CURRENT_DATE);
  ```
  #### 显示月
  ```sql
  select MONTH(CURRENT_DATE);
  ```
  #### 显示年
  ```sql
  select YEAR(CURRENT_DATE);
  ```

### 显示字符串
```sql
select "welecome to my blog!";
``` 

### 当计算器用
```sql
select ((23*2)/3)+25; 
```
### 计算字符串长度
```sql
select char_length('JoeWright');  //9
```
### 日期格式化
```sql
select DATE_FORMAT(now(),'%y-%m-%d');  //18-07-01
select DATE_FORMAT(now(),'%Y-%m-%d');  //2018-07-01
select DATE_FORMAT(now(),'%Y/%m/%d'); // 2018/07/01
```
### 添加和减少时间（常用于数据统计）
```sql
select DATE_ADD(date,interval expr unit)
select DATE_SUB(date,interval expr unit)
//data:日期格式，其中就包括: 如
2018-07-01，now() 等格式。
//expr：表示数量。
//unit：表示单位，支持毫秒(microsecond)，秒(second)，小时(hour)，天(day)，周(week)，年(year)等。
```
例如：
```sql
select date_add(now(),interval 1 day);  //2018-07-02 16:47:48
```
### 加密函数
```sql
select md5(data);
```
例如：
```sql
select md5("JoeWright");  //0851f9b50875d14202b94e2a6eb85a81
```
### 字符串拼接
```sql
select concat("Joe","Wright");  //JoeWright
```
### JSON函数（5.7版本以上支持）
```sql
select json_object("name","JoeWright","sex","男"); //{"sex": "男", "name": "JoeWright"}
```
```sql
select json_array("name","JoeWright","sex","男");  //["name", "JoeWright", "sex", "男"]
```
```sql
//判断json字符串是否符合规范 1:规范；0:不规范
select json_valid('{"sex": "男", "name": "JoeWright"}');  //1
```

  

