# 一：数据库概念
数据库(Database)是按照数据结构来组织、存储和管理数据的仓库。
数据库(Database)是一种结构化信息或数据的有组织的集合。
常用数据库如下：
![[常用数据库_1.png|296]]![[常用数据库_2.png|303]]
1. <font color="#ff0000">关系型数据库(例如MySQL)</font>：关系数据库在 20 世纪 80 年代成为主流。关系数据库中的数据项被组织为一系列具有列和行的表。关系数据库为访问结构化信息提供了有效和灵活的方法.
2. <font color="#ff0000">非关系型数据库（例如redis）</font>：又被称为NoSQL数据库，它支持存储和操作非结构化及半结构化数据。而且随着 Web 应用的日益普及和复杂化，NoSQL 数据库得到了越来越广泛的应用.
- <font color="#ff0000">关系型数据库和非关系型数据库最本质的区别是：</font>
  关系型数据库以数据和数据之间存在的关系维护数据，
  而非关系型数据库是指存储数据的时候数据和数据之前没有什么特定关系。
# 二：数据库的构成
在互联网的网络交互和数据访问中,一般常见两种网络架构模式: B/S结构或者C/S结构.
B/S: Browser-Server即浏览器和服务器, 即通过浏览器和服务器发起网络交互的数据请求.
C/S: Client-Server即客户端和服务器, 即通过客户端和服务器发起网络交互的数据请求.
![[客户端和服务器.png]]
在MySQL中, 我们对数据的组织逻辑上是按照
库、表、数据
这种三级结构组织的。
数据库: 表示一份完整的数据仓库, 在这个数据仓库中分为多张不同的表。
表: 表示某种特定类型数据的的结构化清单, 里面包含多条数据。
数据: 表中数据的基本单元。
![[MySQL的内部组织方式.png]]
# 三：SQL语法（非常重要）
<font color="#ff0000">SQL不区分大小写，但是建议小写，官方建议小写。</font>
## 3.1：数据库的基本操作
### 3.1.1：登录mysql
```
mysql -u root -p [回车]
输入密码
```
![[登录数据库.png]]
注意: 注释
```
在数据库语句中如果我们需要注释某些内容, 一般有三种方式
-- 注释符(要注意的是 -- 之后要有一个空格再接着书写注释内容)
# 注释符 (之后不需要空格)
/* */ 注释符 (一般用于多行注释)
```
注意: 分号
SQL语句应该要以分号作为结束。
### 3.1.2：查看数据库
```
show databases;                     # 查看所有数据库

show databases like '%数据库名%';    # 查看和期望命名相匹配的数据库

show create database 数据库名;       # 查看数据库创建信息

show databases like '%数据库名%';   # %代表是一个通配符: 通配0-n个字符

show databases like 'test';         -- 指明就找test

show databases like '%n';           -- 一个以n字符结束的数据库

show databases like '%n%';          -- 数据库名字中, 有一个n字符

show create database test;         -- 查看之前怎么创建的test数据库(sql语句是什么)
```
![[查看数据库.png]]
- information_schema：主要存储了系统中的一些数据库对象信息，比如用户表信息、列信息、权限信息、字符集信息和分区信息等。
- mysql：MySQL 的核心数据库，主要负责存储数据库用户、用户访问权限等 MySQL 自己需要使用的控制和管理信息。常用的比如在 mysql 数据库的 user 表中修改 root 用户密码。
```
update mysql.user set authentication_string=password('123456') where user='root';

flush privileges;
```
- performance_schema：主要用于收集数据库服务器性能参数。
- sys：sys 数据库主要提供了一些视图，数据都来自于 performation_schema，主要是让开发者和使用者更方便地查看性能问题。
### 3.1.3：创建数据库
```
#创建一个wangdao的数据库, 有可能创建失败直接报错(假如数据库服务里面已经有一个wangdao的数据库了)
create database wangdao;

#创建一个叫test数据库, 并且要求编码格式是utf8, 还要求排序规则utf8_bin
create database test character set utf8 collate utf8_bin;

#如果不存在名字为wangdao的数据库, 就创建wangdao, 如果已经存在了wangdao的数据库, 就不创建(也不报错)
create database if not exists wangdao;

#创建一个指定字符编码格式的和指定排序规则的数据库
create database if not exists test character set utf8 collate utf8_bin;
```
### 3.1.4：删除数据库
- 在工作中不要删除数据库, 哪怕这个数据库已经没有任何用处了 ( 任何数据都是有价值的, 哪怕是错的数据)
```
DROP DATABASE [IF EXISTS] <数据库名>;       # 删除数据库

drop database test;                        # 删除test数据库
```
![[删除数据库.png]]
### 3.1.5：修改数据库
```
ALTER DATABASE [数据库名]
{
[ DEFAULT ] CHARACTER SET <字符集名> | [ DEFAULT ] COLLATE <校对规则名>
}

alter database test character set utf8 collate utf8_bin;
# 把test数据库的编码改成utf8, 校对规则改为utf8_bin
```
在工作中不要修改数据库, 因为没有任何意义
### 3.1.6：选择数据库
```
USE <数据库名>;         # 选择数据库
use test;              # 选择test数据库
```
一个MySQL系统中, 管理多个数据库。 我们只有进入对应的数据库中, 才能进一步操作数据库中的表和数据。
### 3.1.7：查看当前所在数据库

```
select database();  //或者             select schema();
                    //另外，可以使用     status;查看状态
```
## 3.2：表操作
### 3.2.1：常见数据类型
1. 整数：tinyint、int、bigint；
2. 浮点类型：float、double；
3. 字符串：char(固定长度)、varchar(变长字符串，需要额外占用一个字节或者两个字节存变长)、text、longtext;
4. 日志：year、date、time、datetime、TIMESTAMP
- <font color="#ff0000">数据类型（DATA_TYPE）是指数据库所允许的数据的类型。</font>MySQL是一个数据存储容器, 数据类型定义了可以存储什么类型数据的规则。数据库是由多张表构成。其中每个表中对应的数据列，都应该有适当的数据类型，用于限制或允许该列中存储的数据。
1. 整数：![[整数类型.png]]
2. 浮点数：
   ![[浮点数类型.png]]
    - M: 精度, 表示总数据位数。 取值范围为(1～255)。
    - D: 标度, 表示小数位的位数。 取值范围为(1～30，且不能大于 M-2)。
3. 日期：![[日期类型.png]]
4. 字符串：
   ![[字符串数据类型.png]]
    L: 存储字符的实际长度。
    M: 列的指定长度。
    CHAR (M不设置默认为1) 范围可以设置最大255 。
    VARCHAR(M不设置会报错) 最大长度为 65,535字节
    TEXT 最大长度为 65535字符
    LONGTEXT 最大长度长度为 4294967295 字符。
### 3.2.2：表的基本操作
```
#查看数据库下的表
show tables;

#查看表的创建过程
show create table 表的名字;

#删除表（表的结构与内容以及名字全部删除）
drop table 表的名字;

#修改表的结构
alter table 表的名字 change 列的名字 列的新名字 类型;
alter table person change name name_t char(10);        #举例子

#创建表的名字以及表中的结构（列）
create table person(id int, name char(10), salary float);

#向表中添加数据
insert into 表的名字 values (第一列的值, 第二列的值, 第三列的值）;
#例如：
insert into person values (4, 'xxx', 10000),(5, 'yyyy',30000), (6, 'zzzz', 80000);

#查看表中的内容
select * from person;

#更新表中的内容
update 表的名字 set 列 = 列的值 where 其他列 = 其他列的值; 
#其他列与其他列的值最好要唯一
update person set salary = 30000 where id = 2;

#删除表中的内容
delete from 表的名字 where 列 = 列的值; #列与列的值最好要唯一
delete from person where id = 6;
```
- 注意1：关键字create、drop、show、alter是对数据库或者表的进行操作的命令，然后在使用的时候，这些关键的后面需要加上database或者table。
- 注意2：insert、update、delete三个关键字只是对表进行操作的，后面就不需要加关键字table。
### 3.2.3：主键和自增
1. 主键：
   主键（PRIMARY KEY）又被称为主键约束，是 MySQL中使用非常频繁的约束。它有诸多作用(唯一标识, 查找效率...)。
   主键分类:主键分为<font color="#ff0000">单字段主键</font>和<font color="#ff0000">多字段联合主键</font>。
```
# 单字段主键: 既将表中的一个字段设置主键; 通过 PRIMARY KEY 关键字来指定主键.
create table table_primary1(
    id             INT(11)     primary key ,        # 主键
    name           VARCHAR(25),
    job            VARCHAR(25),
    salary         FLOAT
);                              #注意：SQL不区分大小写，关键字大写小写都一样。

# 联合主键: 复合主键(不建议)。这意味着id和name的组合在table_primary3表中表示主键。
create table table_primary3(
    id       INT(11),
    name     VARCHAR(25),
    job      VARCHAR(25),
    salary   FLOAT,
    
    PRIMARY KEY(id, name)
);
```
- 主键: 如果一个表中给某一列设置了主键, 那么这一列强制不能重复, 如果存储重复的主键会自动报错
- 主键: 不允许是null, 存储null主键, 报错每个表只能定义一个主键。
- 主键值必须唯一标识表中的每一行，且不能为 NULL，即表中不可能存在有相同主键值的两行数据。
- 如果用户没有定义主键，也没有定义索引，那么InnoDB引擎会在创建表的时候, 自动生成一个不可见的ROW_ID的列名的聚簇索引，该列是一个6字节的自增数值，随着插入而自增, 来起到主键的作用。
2. 唯一键：
   唯一键(Unique Key)是数据库中的一个约束，用于确保表中的所有记录在指定的一列或列组合上是唯一的。即:组合唯一。
```
#与主键不同的是，唯一键允许有空值(NULL)。
#表中可以有多个唯一键约束。
#在某些情况下，唯一键也可以作为主键的一部分。

create table table_primary3(
    id        INT(11),
    name      VARCHAR(25),
    job       VARCHAR(25),
    salary    FLOAT,
    
    primary key(id),
    unique key (job, salary)
);
```
3. 自增问题：
- 自增问题 AUTO_INCREMENT
```
create table table_primary1(
id        INT(11)       primary key auto_increment , -- 主键
name      VARCHAR(25),
job       VARCHAR(25),
salary    FLOAT
);
# AUTO_INCREMENT 的初始值是 1，数据增加一条，该字段值自动加 1。
# 建议: 一个表中应该只有一个字段使用 AUTO_INCREMENT 约束，且该字段一般位为作为索引/主键的id字段。
# AUTO_INCREMENT 字段应该要设置 NOT NULL 属性。
# AUTO_INCREMENT 约束的字段只能是整数类型。
# AUTO_INCREMENT 上限为所约束的类型的数值上限。

# 如果一个表表在创建的时候某些列没有自增, 设置自增
# alter table 表名 modify 列名 int auto_increment;
alter table user modify id int auto_increment;
```
## 3.3：特殊关键字
### 3.3.1：where
```
SELECT <查询内容|列等> FROM
<表名字> WHERE <查询条件|表达式>

select id, name from students where id > 10;
```
- 使用 WHERE 关键字并指定查询/删除/修改的条件, 让操作在满足条件的情况下执行数据操作。
  1. 算术运算符：可以用在条件中，也可以用在结果集中。
![[算数运算符.png]]
```
例子：
select * from students where (chinese + english + math) < 180;
select * from students where (chinese - math) > 30;
select *, (chinese*0.5 + english*0.1 + math *0.4) from students;
```
2. 比较和逻辑运算符
![[逻辑运算符.png]]
```
例子：
select * from students where chinese >= 90;
select * from students where chinese is null;
select * from students where chinese is not null;
select * from students where chinese between 60 and 90;
select * from students where chinese in (60 , 90);
select * from students where chinese not in (60 , 90);
select * from students where name = '曹操';
select * from students where name like '曹操';
select * from students where name like '曹%';
select * from students where name like '曹%'
and chinese = 90;
```
### 3.3.2：distinct
- 使用DISTINCT对数据表中一个或多个字段重复的数据进行过滤，重复的数据只返回其一条数据
```
SELECT DISTINCT <字段名> FROM <表名>;

select distinct chinese from students;
select distinct chinese, english from students;
```
1. DISTINCT 只能在SELECT语句中使用。 (对结果进行去重)
2. 当对一个或多个字段去重时，DISTINCT 要写在所有字段的最前面。
3. 如果 DISTINCT 对多个字段去重时，只有多个字段组合起来完全是一样的情况下才会被去重。
### 3.3.3：limit
```
SELECT <查询内容|列等> FROM <表名字> LIMIT 记录数目
SELECT <查询内容|列等> FROM <表名字> LIMIT 初始位置，记录数目;
SELECT <查询内容|列等> FROM <表名字> LIMIT 记录数目 OFFSET 初始位置;

select * from students limit 3;
select * from students limit 4, 3;
select * from students limit 3 offset 4;
```
    LIMIT 记录数目: 从第一条开始, 限定记录数目
    LIMIT 初始位置，记录数目: 从起始位置开始, 限定记录数目
    LIMIT 记录数目 OFFSET 初始位置: 从起始位置开始, 限定记录数目
    <font color="#ff0000">注意: 数据(默认下标从0开始)</font>
### 3.3.4：as
AS关键字用来为表和字段指定别名。
```
<内容> AS <别名>

select name from students;
select * from students as s where s.chinese < 60;
select s.name from students as s ;
select s.name from students as s where s.chinese < 60;
```
### 3.3.5：order by
ORDER BY对查询数据结果集进行<font color="#ff0000">排序</font>。
```
select * from students order by chinese;
select * from students order by chinese asc;
select * from students order by chinese desc;
select * from students order by chinese, english;
```
<font color="#ff0000">不加排序模式: 升序排序.</font>
<font color="#ff0000">ASC: 升序排序.</font>
<font color="#ff0000">DESC: 降序排序.</font>
注意: 如上查询, 当我们进行多字段排序的时候, 会先满足第一个列的排序要求, 如果第一列一致的话, 再按照第二列进行排序, 以此类推.
### 3.3.6：group by
使用 GROUP BY 关键字,对数据进行<font color="#ff0000">分组。</font>
```
SELECT <查询内容|列等> FROM <表名字> GROUP BY <字段名...>

select class, group_concat(name), group_concat(chinese) from students group by class;

select class, group_concat(name) from students where chinese > 90 group by class;

select class, group_concat(name), avg(chinese) from students group by class;

select class, group_concat(name) from students group by class having count(*) > 3;
```
<font color="#ff0000">GROUP_CONCAT()函数会把每个分组的字段值都显示出来。group_concat()</font>
<font color="#ff0000">HAVING 可以让我们对分组后的各组数据过滤。(一般和分组+聚合函数配合使用)，having</font>
### 3.3.7：聚合函数
聚合函数一般用来计算列相关的指定值.。通常和分组一起使用。
<font color="#ff0000">count：用于计数；</font>
<font color="#ff0000">sum：求和；</font>
<font color="#ff0000">avg：求平均值；</font>
<font color="#ff0000">max：最大值；</font>
<font color="#ff0000">min：最小值；</font>
1. count：用于计数
```
SELECT <查询内容|列等> , COUNT <列|*> FROM <表名字> GROUP BY HAVING COUNT <表达式|条件>

select count(*) from students
select count(name) from students
select class, group_concat(name), count(*) from students group by class
```
    COUNT(\*): 表示表中总行数
    COUNT(列): 计算除了列值为NULL以外的总行数
2. sun：求和
```
SELECT <查询内容|列等> , SUM<列> FROM <表名字> GROUP BY HAVING SUM<表达式|条件>

select sum(chinese) from students;
select sum(chinese), sum(english), sum(math) from students;
select class, group_concat(name), sum(chinese) from students group by class;
```
3. avg：求平均值
```
SELECT <查询内容|列等> , AVG<列> FROM <表名字> GROUP BY HAVING AVG<表达式|条件>

select avg(chinese) from students;
select avg(chinese), avg(english), avg(math) from students;
select class, group_concat(name), avg(chinese) from students group by class;
```
4. max：最大值
```
SELECT <查询内容|列等> , MAX<列> FROM <表名字> GROUP BY HAVING MAX<表达式|条件>

eg:
select max(chinese) from students;
select max(chinese), max(english), max(math) from students;
select class, group_concat(name), max(chinese) from students group by class;
```
5. min：最小值
```
SELECT <查询内容|列等> , MIN<列> FROM <表名字> GROUP BY HAVING MIN<表达式|条件>

select min(chinese) from students;
select min(chinese), min(english), max(math) from students;
select class, group_concat(name), min(chinese) from students group by class;
```
## 3.4：SQL执行顺序（重要）
```
(5) SELECT column_name, ...
(1) FROM table_name, ...
(2) [WHERE ...]
(3) [GROUP BY ...]
(4) [HAVING ...]
(6) [ORDER BY ...];
(7) [Limit ...]
# 计算每个班级中语文成绩大于60的同学的平均数学成绩, 获得平均数学成绩大于等于40的班级中平均语文成绩最高的那个班级的平均语文和数学成绩

select class, group_concat(name), avg(chinese) as chineseA, avg(math) from students where chinese >= 60 group by class having avg(math)>=40 order by chineseA desc limit 0, 1;
```
小括号中的数字代表执行顺序。
having和select的执行顺序收到优化器的影响，可能会改变执行顺序。


















