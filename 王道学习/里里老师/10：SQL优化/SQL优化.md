# 一：慢查询日志
- 在MySQL中提供了一种以供我们记录耗时SQL运行时间的日志文件，即<font color="#ff0000">慢查询日志</font>，用以帮助我们在使用MySQL的的过程中快速定位和排查耗时SQL，进而优化SQL语句提供了方便。在慢查询日志中我们可以设置指定的SQL执行上限时间，用以记录耗时SQL的执行过程。
- 比如：某个SQL语句执行的时间很长，这个慢查询日志就会进行记录。
## 1.1：慢查询日志的默认设置
- 慢查询日志的位置。
```
show variables like '%slow%';

//例如：
mysql> show variables like '%slow%';
+-----------------------------+---------------------------------+
| Variable_name               | Value
|
+-----------------------------+---------------------------------+
| log_slow_admin_statements   | OFF |
| log_slow_extra              | OFF |
| log_slow_replica_statements | OFF |
| log_slow_slave_statements   | OFF |
| slow_launch_time            | 2
| slow_query_log              | OFF |
| slow_query_log_file         | /var/lib/mysql/wangdao-slow.log |
+-----------------------------+---------------------------------+
7 rows in set (0.00 sec)
```
- 慢查询日志默认处于未开启状态。
- 慢查询SQL的时间阈值: <font color="#ff0000">默认阈值为10s</font>。
```
mysql>
show variables like '%long_query%';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
1 row in set (0.00 sec)
```
## 1.2：修改配置
1. 开启慢查询文件。
   `set global slow_query_log=on;`
2. 更改耗时SQL阈值。
   `set global long_query_time=0.1;`
3. 修改回默认设置 (测试完成之后, 改回来)。
```
set global slow_query_log=off;
set global long_query_time=10;
```
# 二：执行计划
在 MySQL中 <font color="#ff0000">explan</font> 的存在具有重要的意义。explan提供了查询执行计划的大致执行信息，包括 MySQL 如何读取表、选择索引、连接表以及过滤数据等，从而使用户能够进一步了解查询的执行过程。
![[执行计划总览图.png]]
简而言之: explain以一种可视化的形式, 帮我们近似展示SQL底层执行的状态/执行计划。
而我们使用EXPLAIN只需要在待查询语句前面加上explain关键字，就可以了.
![[explan示意图.png]]
```mysql
explain + SQL语句
```
![[执行计划.png]]
## 2.1：id列
   id标识符，我们一般通过id值的大小研究查询中的逻辑顺序。表示SQL语句中表的执行顺序；
   - id值相同，会按照从上向下执行；
   - id值不同，那么会按照从大到小的顺序执行;
   - id值既有相同也有不同，首先遵循从大到小执行，然后id相同的时候，从上向下执行;
![[执行计划id列.png]]
## 2.2：select_type列
   select_type表示select的查询类型，主要用于区分各种复杂的查询，例如普通查询、联合查询、子查询等。
![[执行计划select_type列.png]]
## 2.3：type字段
   type字段的作用, 用以显示MySQL决定使用哪种方式来访问表，对于type字段, 从性能的好坏划分, 我们可以分为。
![[执行任务type列.png]]
## 2.4：索引相关的三列
   possible_keys：可能使用到的索引；
   key:实际使用到的索引；
   key_len：使用索引的最大可能长度;
   
   计算的方式：n \*(对应的编码格式的值) + 2/0（varchar额外占用两个字节，char额外不占用字节）+ 1/0(为空会占用一个1字节,不为空就不会占用一个字节）
```
utf8mb4:一个字符占用4个字节;
utf8mb3:一个字符占用3个字节；
latin1:一个字符占用1个字节；
```
# 三：索引优化
## 3.1：索引失效的情况
1. 对索引列进行运算。
   `select * from t where a = 1;`
   `select * from t where a + 1 = 2;`
2. 对索引进行不等于判断。
   `select * from t where a != 1;`
3. 查找范围过大。
   `explain select * from t where c between 1 and 20000;`