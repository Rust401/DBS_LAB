# 数据库系统（2018-2019春夏学期）实验2
```shell
Project Name:   SQL数据定义和操作
Student ID  :   21714069
Email       :   zhaodonghu94@zju.edu.cn
phone       :   15700080428
Date        :   2019.03.20
```
## 建立数据库
我们先看下现有的数据库
```sql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| School             |
| TEST               |
| dude               |
| fruit              |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
8 rows in set (0.00 sec)
```
先把之前用的删掉
```sql
mysql> drop database TEST;
Query OK, 0 rows affected (0.05 sec)

mysql> drop database dude;
Query OK, 1 row affected (0.02 sec)

mysql> drop database School;
Query OK, 0 rows affected (0.00 sec)

mysql> drop database fruit;
Query OK, 1 row affected (0.01 sec)
```
再看下现有的数据库
```sql
mysql> show databases
    -> ;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
```
新建个学校的数据库并使用它
```sql
mysql> create database school
    -> ;
Query OK, 1 row affected (0.00 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| school             |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
mysql> use school
Database changed
mysql> select database()
    -> ;
+------------+
| database() |
+------------+
| school     |
+------------+
1 row in set (0.00 sec)
```
## 数据定义
### 表的建立/删除
先建一个表
```sql
mysql> create table classroom
    -> (building varchar(15),
    -> room_number varchar(7),
    -> capacity numeric(4,0),
    -> primary key (building,room_number)
    -> );
Query OK, 0 rows affected (0.04 sec)
mysql> show tables
    -> ;
+------------------+
| Tables_in_school |
+------------------+
| classroom        |
+------------------+
1 row in set (0.00 sec)
```
把这个表修改下
```sql
mysql> alter table classroom rename myClassroom;
Query OK, 0 rows affected (0.01 sec)

mysql> show tables
    -> ;
+------------------+
| Tables_in_school |
+------------------+
| myClassroom      |
+------------------+
1 row in set (0.00 sec)
```
修改包括名称、存储引擎、字段的添加/删除、字段名称和定义的修改。均以`ALTER`关键词作为开头，比较有意思的显然是当表中已经有数据之后再修改表的字段属性会出现什么情况。  
我们先插一行，把最后一个`department`的值改掉，然后再改表头。
```sql
mysql> insert into myClassroom values ('Packard', '101', '500',NULL);
Query OK, 1 row affected (0.01 sec)

mysql> select * from myClassroom
    -> ;
+----------+-------------+----------+------------+
| building | room_number | capacity | department |
+----------+-------------+----------+------------+
| Packard  | 101         |      500 | NULL       |
+----------+-------------+----------+------------+
1 row in set (0.00 sec)

mysql> update myClassroom set department="shabi" where room_number=101;
Query OK, 1 row affected (0.02 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from myClassroom;
+----------+-------------+----------+------------+
| building | room_number | capacity | department |
+----------+-------------+----------+------------+
| Packard  | 101         |      500 | shabi      |
+----------+-------------+----------+------------+
1 row in set (0.00 sec)
```
我们加了一个字段名称、删了这个字段名称（值为NULL），修改了下capacity的小数位数，然后把`department`这个字段直接删掉。
```sql
mysql> alter table myClassroom add father varchar(10);
Query OK, 0 rows affected (0.05 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> select * from myClassroom;
+----------+-------------+----------+------------+--------+
| building | room_number | capacity | department | father |
+----------+-------------+----------+------------+--------+
| Packard  | 101         |      500 | shabi      | NULL   |
+----------+-------------+----------+------------+--------+
1 row in set (0.00 sec)

mysql> alter table myClassroom drop father;
Query OK, 0 rows affected (0.06 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> select * from myClassroom;
+----------+-------------+----------+------------+
| building | room_number | capacity | department |
+----------+-------------+----------+------------+
| Packard  | 101         |      500 | shabi      |
+----------+-------------+----------+------------+
1 row in set (0.00 sec)

mysql> alter table myClassroom modify capacity numeric(4,4);
ERROR 1264 (22003): Out of range value for column 'capacity' at row 1
mysql> alter table myClassroom modify capacity numeric(4,1);
Query OK, 1 row affected (0.02 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> select * from myClassroom;
+----------+-------------+----------+------------+
| building | room_number | capacity | department |
+----------+-------------+----------+------------+
| Packard  | 101         |    500.0 | shabi      |
+----------+-------------+----------+------------+
1 row in set (0.01 sec)

mysql> alter table myClassroom modify capacity numeric(4,2);
ERROR 1264 (22003): Out of range value for column 'capacity' at row 1
mysql> alter table myClassroom modify capacity numeric(4,5);
ERROR 1427 (42000): For float(M,D), double(M,D) or decimal(M,D), M must be >= D (column 'capacity').
mysql> alter table myClassroom modify capacity numeric(4,3);
ERROR 1264 (22003): Out of range value for column 'capacity' at row 1
mysql> alter table myClassroom drop department;
Query OK, 0 rows affected (0.06 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> select * from myClassroom;
+----------+-------------+----------+
| building | room_number | capacity |
+----------+-------------+----------+
| Packard  | 101         |    500.0 |
+----------+-------------+----------+
1 row in set (0.00 sec)
```
新加的字段里的值都是`NULL`，删字段的时候不管里面数据是什么都直接drop掉了。  
接下来试一下把字段类型改的过分一点
```sql
mysql> alter table myClassroom change capacity quantity varchar(10);
Query OK, 1 row affected (0.03 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> select * from myClassroom;
+----------+-------------+----------+
| building | room_number | quantity |
+----------+-------------+----------+
| Packard  | 101         | 500.0    |
+----------+-------------+----------+
1 row in set (0.01 sec)

mysql> alter table myClassroom change building  intBuilding  numeric(10,9);
ERROR 1366 (HY000): Incorrect DECIMAL value: '0' for column '' at row -1
mysql> alter table myClassroom change building  intBuilding  numeric(10,0);
ERROR 1366 (HY000): Incorrect DECIMAL value: '0' for column '' at row -1
mysql> alter table myClassroom change building  intBuilding  varchar(20);
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> select * from myClassroom;
+-------------+-------------+----------+
| intBuilding | room_number | quantity |
+-------------+-------------+----------+
| Packard     | 101         | 500.0    |
+-------------+-------------+----------+
1 row in set (0.00 sec)
```
结果意料之中，数值可以转字符串，字符串变成数值怕是不行。
```
mysql> alter table myClassroom change room_number intRM numeric(10,0);
Query OK, 1 row affected (0.02 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> select * from myClassroom;
+-------------+-------+----------+
| intBuilding | intRM | quantity |
+-------------+-------+----------+
| Packard     |   101 | 500.0    |
+-------------+-------+----------+
1 row in set (0.00 sec)

mysql> alter table myClassroom change room_number intRM numeric(10,1);
ERROR 1054 (42S22): Unknown column 'room_number' in 'myclassroom'
mysql> alter table myClassroom change intRM  intRM numeric(10,1);
Query OK, 1 row affected (0.03 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> select * from myClassroom;
+-------------+-------+----------+
| intBuilding | intRM | quantity |
+-------------+-------+----------+
| Packard     | 101.0 | 500.0    |
+-------------+-------+----------+
1 row in set (0.00 sec)
```
长得就很数字的字符串可以转，长得不像的字符串不能转。篇幅所限，这部分的探索到此为止，先继续完成后面的实验报告。  

我们把这个表删了先
```
mysql> delete from myClassroom;
Query OK, 1 row affected (0.01 sec)

mysql> select * from myClassroom;
Empty set (0.00 sec)

mysql> drop table myClassroom;
Query OK, 0 rows affected (0.00 sec)

mysql> show tablse;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'tablse' at line 1
mysql> show tables;
Empty set (0.00 sec)
```
两种方法，删数据不删表以及直接删表。
### 索引的建立/删除
我们还是先来建个表吧
```sql
mysql> create table classroom (building varchar(15), room_number varchar(7), capacity numeric(4,0));
Query OK, 0 rows affected (0.02 sec)

mysql> insert into classroom values ('Packard', '101', '500');
Query OK, 1 row affected (0.00 sec)

mysql> insert into classroom values ('Painter', '514', '10');
Query OK, 1 row affected (0.00 sec)

mysql> insert into classroom values ('Taylor', '3128', '70');
Query OK, 1 row affected (0.00 sec)

mysql> insert into classroom values ('Watson', '100', '30');
Query OK, 1 row affected (0.00 sec)

mysql> insert into classroom values ('Watson', '120', '50');
Query OK, 1 row affected (0.01 sec)
```
这里先建了一个没有索引也没有主键的表`classroom`，然后插了几个数据，然后我们来建一下索引。建索引的方太多了，初始化的时候就可以建，初始化之后还可以另外加。索引的类别也很多，`PRIMARY KEY`,`INDEX`,`UNIQUE`,`FULLTEXT`,`组合索引`。
我们给它建个主键索引先。
```sql
mysql> alter table classroom add primary key (building,room_number);
Query OK, 0 rows affected (0.05 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> select * from classroom;
+----------+-------------+----------+
| building | room_number | capacity |
+----------+-------------+----------+
| Packard  | 101         |      500 |
| Painter  | 514         |       10 |
| Taylor   | 3128        |       70 |
| Watson   | 100         |       30 |
| Watson   | 120         |       50 |
+----------+-------------+----------+
5 rows in set (0.00 sec)
```
建了之后貌似看不出区别，实际上(buiding,room_number)这个pair已经唯一确定了某个row，被存在一个`BTREE`中了。我们查看下这个表的索引：
```sql
mysql> show index from classroom
    -> ;
+-----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table     | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+-----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| classroom |          0 | PRIMARY  |            1 | building    | A         |           4 |     NULL | NULL   |      | BTREE      |         |               |
| classroom |          0 | PRIMARY  |            2 | room_number | A         |           5 |     NULL | NULL   |      | BTREE      |         |               |
+-----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
2 rows in set (0.01 sec)
```
这样可以获得某个表的详细的索引信息，里面的内容可以自解释，不难理解。  
然后我们把索引删掉
```sql
mysql> alter table classroom drop primary key;
Query OK, 5 rows affected (0.05 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> show index from classroom;
Empty set (0.00 sec)
```
好了主键删掉了，其实主键就是个特殊的索引，删索引的操作其实比较类似，这里就不重复贴上来了。
### 视图的建立/删除
视图是一种虚拟存在的表，是一个逻辑表，本身并不包含数据。作为一个select语句保存在数据字典中的。通过视图，可以展现基表的部分数据，视图数据来自定义视图的查询中使用的表，使用视图动态生成。有点有，简单，安全，数据独立。
```sql
mysql> create view v_classroom as 
    -> (select * from classroom);
Query OK, 0 rows affected (0.03 sec)

mysql> select * from v_classroom;
+----------+-------------+----------+
| building | room_number | capacity |
+----------+-------------+----------+
| Packard  | 101         |      500 |
| Painter  | 514         |       10 |
| Taylor   | 3128        |       70 |
| Watson   | 100         |       30 |
| Watson   | 120         |       50 |
+----------+-------------+----------+
5 rows in set (0.01 sec)
```
```sql
mysql> create view v0 as  (select building,room_number from classroom);
Query OK, 0 rows affected (0.01 sec)
mysql> select * from v0;
+----------+-------------+
| building | room_number |
+----------+-------------+
| Packard  | 101         |
| Painter  | 514         |
| Taylor   | 3128        |
| Watson   | 100         |
| Watson   | 120         |
+----------+-------------+
5 rows in set (0.00 sec)
```
我们不想让人看到教室的capacity，所以用基表的两列创建了一个视图。换个例子，如果我们不想让别人看到员工工资我们就可以创建一个不包含salary的视图再让别人访问。  
然后我们把之前创建的视图删掉
```
mysql> drop view v0;
Query OK, 0 rows affected (0.01 sec)

mysql> drop view v_classroom;
Query OK, 0 rows affected (0.01 sec)

```
我们可以用如下语句去查看视图。
```sql
show table status where comment='view’；
```
## 表的插入/删除/更新
这里还是先把整个学校数据库的体系建立下再玩吧，这样子例子比较丰富。网站上把DDL和数据先下载下来然后用batch模式先执行一下。
```sql
mysql> source DDL-MySQL.sql;
Query OK, 0 rows affected (0.01 sec)

Query OK, 0 rows affected (0.02 sec)

Query OK, 0 rows affected (0.03 sec)

Query OK, 0 rows affected (0.02 sec)

Query OK, 0 rows affected (0.03 sec)

Query OK, 0 rows affected (0.03 sec)

Query OK, 0 rows affected (0.03 sec)

Query OK, 0 rows affected (0.05 sec)

Query OK, 0 rows affected (0.03 sec)

Query OK, 0 rows affected (0.02 sec)

Query OK, 0 rows affected (0.02 sec)

mysql> show tables;
+------------------+
| Tables_in_school |
+------------------+
| advisor          |
| classroom        |
| course           |
| department       |
| instructor       |
| prereq           |
| section          |
| student          |
| takes            |
| teaches          |
| time_slot        |
+------------------+
11 rows in set (0.00 sec)
```
表定义完了我们把数据插进去,再看表里应该已经有数据了。
```sql
mysql> source smallRelationsInsertFile.sql;
mysql> select * from course;
+-----------+----------------------------+------------+---------+
| course_id | title                      | dept_name  | credits |
+-----------+----------------------------+------------+---------+
| BIO-101   | Intro. to Biology          | Biology    |       4 |
| BIO-301   | Genetics                   | Biology    |       4 |
| BIO-399   | Computational Biology      | Biology    |       3 |
| CS-101    | Intro. to Computer Science | Comp. Sci. |       4 |
| CS-190    | Game Design                | Comp. Sci. |       4 |
| CS-315    | Robotics                   | Comp. Sci. |       3 |
| CS-319    | Image Processing           | Comp. Sci. |       3 |
| CS-347    | Database System Concepts   | Comp. Sci. |       3 |
| EE-181    | Intro. to Digital Systems  | Elec. Eng. |       3 |
| FIN-201   | Investment Banking         | Finance    |       3 |
| HIS-351   | World History              | History    |       3 |
| MU-199    | Music Video Production     | Music      |       3 |
| PHY-101   | Physical Principles        | Physics    |       4 |
+-----------+----------------------------+------------+---------+
13 rows in set (0.00 sec)
```
### 条目增加
```sql
mysql> insert into course values ('DUDE COURSE', 'DB', 'CS', '10');
ERROR 1406 (22001): Data too long for column 'course_id' at row 1
mysql> insert into course values ('DUDE', 'DB', 'CS', '10');
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`school`.`course`, CONSTRAINT `course_ibfk_1` FOREIGN KEY (`dept_name`) REFERENCES `department` (`dept_name`) ON DELETE SET NULL)
mysql> insert into course values ('BIO-101', 'Intro. to Biology', 'Biology', '5');
ERROR 1062 (23000): Duplicate entry 'BIO-101' for key 'PRIMARY'
mysql> insert into course values ('BIO-111', 'Intro. to Biology', 'Biology', '5');
Query OK, 1 row affected (0.03 sec)
mysql> select * from course;
+-----------+----------------------------+------------+---------+
| course_id | title                      | dept_name  | credits |
+-----------+----------------------------+------------+---------+
| BIO-101   | Intro. to Biology          | Biology    |       4 |
| BIO-111   | Intro. to Biology          | Biology    |       5 |
| BIO-301   | Genetics                   | Biology    |       4 |
| BIO-399   | Computational Biology      | Biology    |       3 |
| CS-101    | Intro. to Computer Science | Comp. Sci. |       4 |
| CS-190    | Game Design                | Comp. Sci. |       4 |
| CS-315    | Robotics                   | Comp. Sci. |       3 |
| CS-319    | Image Processing           | Comp. Sci. |       3 |
| CS-347    | Database System Concepts   | Comp. Sci. |       3 |
| EE-181    | Intro. to Digital Systems  | Elec. Eng. |       3 |
| FIN-201   | Investment Banking         | Finance    |       3 |
| HIS-351   | World History              | History    |       3 |
| MU-199    | Music Video Production     | Music      |       3 |
| PHY-101   | Physical Principles        | Physics    |       4 |
+-----------+----------------------------+------------+---------+
14 rows in set (0.00 sec)
```
这里面有诸多约束，其实不能随便增删，比如增加条目了的`dept_name`必须在`department`那个表里有。
### 条目修改
```sql
mysql> update course set credits=10 where course_id='BIO-101';
Query OK, 1 row affected (0.02 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from course where couse_id='BIO-101';
ERROR 1054 (42S22): Unknown column 'couse_id' in 'where clause'
mysql> select * from course where course_id='BIO-101';
+-----------+-------------------+-----------+---------+
| course_id | title             | dept_name | credits |
+-----------+-------------------+-----------+---------+
| BIO-101   | Intro. to Biology | Biology   |      10 |
+-----------+-------------------+-----------+---------+
1 row in set (0.01 sec)
```
### 条目删除
```sql
mysql> delete from course where course_id='BIO-111';
Query OK, 1 row affected (0.01 sec)

mysql> SELECT * FROM course where course_id='BIO-111';
Empty set (0.01 sec)
```
## 数据查询
实验报告里要求的作业跑通，作业的表格暂时没找到，这边以书里的学校数据库例子作为查询的样本。进行单表多表和嵌套子查询。
### 单表查询
```sql
mysql> select id,name,dept_name,salary*2 from instructor where dept_name='Comp. Sci.';
+-------+------------+------------+-----------+
| id    | name       | dept_name  | salary*2  |
+-------+------------+------------+-----------+
| 10101 | Srinivasan | Comp. Sci. | 130000.00 |
| 45565 | Katz       | Comp. Sci. | 150000.00 |
| 83821 | Brandt     | Comp. Sci. | 184000.00 |
+-------+------------+------------+-----------+
3 rows in set (0.01 sec)
```
```sql
mysql> select name
    -> from instructor
    -> where dept_name='Comp. Sci.' and salary > 7000;
+------------+
| name       |
+------------+
| Srinivasan |
| Katz       |
| Brandt     |
+------------+
3 rows in set (0.00 sec)
```
### 多表查询
```
mysql> select name,instructor.dept_name,building from instructor,department where instructor.dept_name=department.dept_name;
+------------+------------+----------+
| name       | dept_name  | building |
+------------+------------+----------+
| Crick      | Biology    | Watson   |
| Srinivasan | Comp. Sci. | Taylor   |
| Katz       | Comp. Sci. | Taylor   |
| Brandt     | Comp. Sci. | Taylor   |
| Kim        | Elec. Eng. | Taylor   |
| Wu         | Finance    | Painter  |
| Singh      | Finance    | Painter  |
| El Said    | History    | Painter  |
| Califieri  | History    | Painter  |
| Mozart     | Music      | Packard  |
| Einstein   | Physics    | Watson   |
| Gold       | Physics    | Watson   |
+------------+------------+----------+
12 rows in set (0.01 sec)
```
```sql
mysql> select name,course_id
    -> from instructor,teaches
    -> where instructor.ID=teaches.ID and instructor.dept_name='Comp. Sci.';
+------------+-----------+
| name       | course_id |
+------------+-----------+
| Srinivasan | CS-101    |
| Srinivasan | CS-315    |
| Srinivasan | CS-347    |
| Katz       | CS-101    |
| Katz       | CS-319    |
| Brandt     | CS-190    |
| Brandt     | CS-190    |
| Brandt     | CS-319    |
+------------+-----------+
8 rows in set (0.00 sec)
```
```sql
mysql> select name,title
    -> from instructor natural join teaches,course
    -> where teaches.course_id=course.course_id;
+------------+----------------------------+
| name       | title                      |
+------------+----------------------------+
| Srinivasan | Intro. to Computer Science |
| Srinivasan | Robotics                   |
| Srinivasan | Database System Concepts   |
| Wu         | Investment Banking         |
| Mozart     | Music Video Production     |
| Einstein   | Physical Principles        |
| El Said    | World History              |
| Katz       | Intro. to Computer Science |
| Katz       | Image Processing           |
| Crick      | Intro. to Biology          |
| Crick      | Genetics                   |
| Brandt     | Game Design                |
| Brandt     | Game Design                |
| Brandt     | Image Processing           |
| Kim        | Intro. to Digital Systems  |
+------------+----------------------------+
mysql> select name,title from  (instructor natural join teaches) join course using (course_id);
+------------+----------------------------+
| name       | title                      |
+------------+----------------------------+
| Crick      | Intro. to Biology          |
| Crick      | Genetics                   |
| Srinivasan | Intro. to Computer Science |
| Katz       | Intro. to Computer Science |
| Brandt     | Game Design                |
| Brandt     | Game Design                |
| Srinivasan | Robotics                   |
| Katz       | Image Processing           |
| Brandt     | Image Processing           |
| Srinivasan | Database System Concepts   |
| Kim        | Intro. to Digital Systems  |
| Wu         | Investment Banking         |
| El Said    | World History              |
| Mozart     | Music Video Production     |
| Einstein   | Physical Principles        |
+------------+----------------------------+
15 rows in set (0.01 sec)
```
### 嵌套子查询
```sql
mysql> select distinct course_id
    -> from section
    -> where semester='Fall' and year=2009 and
    -> course_id in (select course_id from section where semester='Spring' and year=2010);
+-----------+
| course_id |
+-----------+
| CS-101    |
+-----------+
1 row in set (0.01 sec)
```
```sql
mysql> select distinct course_id
    -> from section 
    -> where semester='Fall' and year=2009 and course_id not in
    -> (select course_id from section where semester='Spring' and year=2010);
+-----------+
| course_id |
+-----------+
| CS-347    |
| PHY-101   |
+-----------+
2 rows in set (0.02 sec)
```
```sql
mysql> select dept_name from instructor group by dept_name
    -> having avg(salary) >= all
    -> (select avg(salary)
    -> from instructor group by dept_name);
+-----------+
| dept_name |
+-----------+
| Physics   |
+-----------+
1 row in set (0.01 sec)
```
例子比较多，遍历完所有的组合不可能，就不一一贴上来了。






