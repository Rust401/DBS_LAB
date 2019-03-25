# 数据库系统（2018-2019春夏学期）实验3
```shell
Project Name:   SQL数据完整性
Student Name:   胡兆冬
Student ID  :   21714069
Major       :   环境工程
Email       :   zhaodonghu94@zju.edu.cn
phone       :   15700080428
Date        :   2019.03.20
```
## 定义若干表，包括primary key, foreign key 和 check 的定义
```sql
mysql> create table classroom (building varchar(15), room_number varchar(7), capacity numeric(4,0), primary key(building,room_number) );
Query OK, 0 rows affected (0.04 sec)

mysql> create table department
    -> ( dept_name varchar(20),
    -> building varchar(15),
    -> budget numeric(12,2) check (budget>0),
    -> primary key(dept_name)
    -> );
Query OK, 0 rows affected (0.22 sec)

mysql> create table course                                                    -> (course_id varchar(8),
    -> title varchar(50),
    -> dept_name varchar(20),
    -> credits numeric(2,0) check (credits>0),
    -> primary key (course_id),
    -> foreign key (dept_name) references department(dept_name) on delete set null
    -> );
Query OK, 0 rows affected (0.02 sec)


mysql> create table instructor
    -> (
    -> ID varchar(5),
    -> name varchar(20) not nullm
    -> \c
mysql> create table instructor ( ID varchar(5), name varchar(20) not null,
    -> dept_name varchar(20),
    -> salary numeric(8,2) check (salary > 29000),
    -> primary key (ID),
    -> foreign key(dept_name) references department(dept_name) on delete set null);
Query OK, 0 rows affected (0.02 sec)


mysql> create table section
    -> (course_id varchar(8), 
    ->          sec_id varchar(8),
    ->  semester varchar(6) check (semester in ('Fall', 'Winter', 'Spring', 'Summer')), 
    ->  year numeric(4,0) check (year > 1701 and year < 2100), 
    ->  building varchar(15),
    ->  room_number varchar(7),
    ->  time_slot_id varchar(4),
    ->  primary key (course_id, sec_id, semester, year),
    ->  foreign key (course_id) references course(course_id) on delete cascade,
    ->  foreign key (building, room_number) references classroom(building, room_number) on delete set null
    -> );
Query OK, 0 rows affected (0.03 sec)


mysql> create table teaches
    -> (
    -> ID varchar(5),
    -> course_id varchar(8),
    -> sec_id varchar(8),
    -> semester varchar(6),
    -> year numeric(4,0),
    -> primary key(ID,course_id,sec_id,semester,year),
    -> foreign key(course_id,sec_id,semester,year) references section(course_id,sec_id,semester,year) on delete cascade,
    -> foreign key(ID) references instructor(ID) on delete cascade);
Query OK, 0 rows affected (0.03 sec)

mysql> create table student
    -> (ID varchar(5),
    -> name varchar(20) not null,
    -> dept_name varchar(20),
    -> tot_cred numeric(3,0) check (tot_cred>=0),
    -> primary key (ID),
    -> foreign key (dept_name) references department(dept_name) on delete set null);
Query OK, 0 rows affected (0.03 sec)


mysql> create table takes
    -> (ID varchar(5),
    -> course_id varchar(8),
    -> sec_id varchar(8),
    -> semester varchar(6),
    -> year numeric(4,0),
    -> grade varchar(2),
    -> primary key(ID,course_id,sec_id,semester,year),
    -> foreign key(course_id,sec_id,semester,year) references section(course_id,sec_id,semester,year) on delete cascade,
    -> foreign key(ID) references student(ID) on delete cascade);
Query OK, 0 rows affected (0.03 sec)


mysql> create table advisor
    -> (s_ID varchar(5),
    ->  i_ID varchar(5),
    ->  primary key (s_ID),
    ->  foreign key (i_ID) references instructor (ID) on delete set null,
    ->  foreign key (s_ID) references student (ID) on delete cascade
    -> );
Query OK, 0 rows affected (0.03 sec)


mysql> create table time_slot
    -> (
    -> time_slot_id varchar(4),
    -> day varchar(1),
    -> start_hr numeric(2) check (start_hr>=0 and start_hr <24),
    -> start_min numeric(2) check (start_min >=0 and start_min<60),
    -> end_hr numeric(2) check (end_hr<=0 and end_hr <24),
    -> end_min numeric(2) check (end_min>=0 and end_min <60),
    -> primary key (time_slot_id, day, start_hr,start_min));
Query OK, 0 rows affected (0.02 sec)


mysql> create table prereq
    -> (course_id varchar(8),
    -> prereq_id varchar(8),
    -> primary key(course_id,prereq_id),
    -> foreign key(course_id) references course(course_id) on delete cascade,
    -> foreign key(prereq_id) references course(course_id));
Query OK, 0 rows affected (0.02 sec)
```
学校的相关表格已经建好了，接下来我们来插数据。
## 让表中插入数据，考察primary key如何控制实体完整性。
先对`classroom`插一波
```sql
mysql> insert into classroom values ('Packard', '101', '500');
Query OK, 1 row affected (0.01 sec)

mysql> insert into classroom values ('Painter', '514', '10');
Query OK, 1 row affected (0.00 sec)

mysql> insert into classroom values ('Taylor', '3128', '70');
Query OK, 1 row affected (0.00 sec)

mysql> insert into classroom values ('Watson', '100', '30');
Query OK, 1 row affected (0.00 sec)

mysql> insert into classroom values ('Watson', '120', '50');
Query OK, 1 row affected (0.01 sec)
```
插个与主键重复的值进去
```sql
mysql> insert into classroom values ('Packard', '101', '600');
ERROR 1062 (23000): Duplicate entry 'Packard-101' for key 'PRIMARY'
```
一个表只能有一个主键，主键要唯一区分每个条，所以重复的主键是插不进去的，如果这个表没定义主键，我刚才那行是可以插进去的。
## 删除被引用表中的行，考察foreign key 中on delete子句如何控制参照完整性。
我们这里先还是把整个体系的数据先都插进去
```sql
mysql> source smallRelationsInsertFile.sql;
```
我们先看看`course`这个表里面有什么
```sql
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
dept_name是个外键，引用了department的主键，我们把deparment中的某个主键删掉再看看会咋样
```sql
mysql> delete from department where dept_name='Biology';
Query OK, 1 row affected (0.01 sec)

mysql> select * from course;
+-----------+----------------------------+------------+---------+
| course_id | title                      | dept_name  | credits |
+-----------+----------------------------+------------+---------+
| BIO-101   | Intro. to Biology          | NULL       |       4 |
| BIO-301   | Genetics                   | NULL       |       4 |
| BIO-399   | Computational Biology      | NULL       |       3 |
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
显然原先`Biology`的课的department已经被删掉了，所以变成了NULL。
我们再看看`on delete casecade`会咋样
```sql
mysql> select * from advisor;
+-------+-------+
| s_ID  | i_ID  |
+-------+-------+
| 12345 | 10101 |
| 44553 | 22222 |
| 45678 | 22222 |
| 00128 | 45565 |
| 76543 | 45565 |
| 23121 | 76543 |
| 98988 | 76766 |
| 76653 | 98345 |
| 98765 | 98345 |
+-------+-------+
9 rows in set (0.00 sec)

mysql> delete from student where ID=12345;
Query OK, 1 row affected (0.00 sec)

mysql> select * from advisor;
+-------+-------+
| s_ID  | i_ID  |
+-------+-------+
| 44553 | 22222 |
| 45678 | 22222 |
| 00128 | 45565 |
| 76543 | 45565 |
| 23121 | 76543 |
| 98988 | 76766 |
| 76653 | 98345 |
| 98765 | 98345 |
+-------+-------+
8 rows in set (0.00 sec)
```
我们把ID为12345的学生从studeng表中删除了，因此advisor表中s_ID为12345的条目也被删掉了。  
我们试试把某个course删掉。
```sql
mysql> delete from course where course_id='CS-101';
ERROR 1451 (23000): Cannot delete or update a parent row: a foreign key constraint fails (`lab3`.`prereq`, CONSTRAINT `prereq_ibfk_2` FOREIGN KEY (`prereq_id`) REFERENCES `course` (`course_id`))
```
显然删不掉，因为`prereq`这个表把couse_id作为foreign key（no action的那种）,所以只要引用还存在就会拒绝删除。
## 修改被引用表中的行的primary key，考察foreign key 中on update 子句如何控制参照完整性。
update course set credits=10 where course_id='BIO-101';
我们改个department的名字看看
```sql
mysql> update department set dept_name='DUDE' where dept_name='Comp. Sci.';
ERROR 1451 (23000): Cannot delete or update a parent row: a foreign key constraint fails (`lab3`.`course`, CONSTRAINT `course_ibfk_1` FOREIGN KEY (`dept_name`) REFERENCES `department` (`dept_name`) ON DELETE SET NULL)
```
由于course把dept_name作为外键，`on delete set null`，所以不能对dept_ment作update会被限制。我们试试别的模式。
```sql
mysql> update student set ID='2333' where ID='00128';
ERROR 1451 (23000): Cannot delete or update a parent row: a foreign key constraint fails (`lab3`.`advisor`, CONSTRAINT `advisor_ibfk_2` FOREIGN KEY (`s_ID`) REFERENCES `student` (`ID`) ON DELETE CASCADE)
mysql> update student set ID='2333' where name='Zhang';
ERROR 1451 (23000): Cannot delete or update a parent row: a foreign key constraint fails (`lab3`.`advisor`, CONSTRAINT `advisor_ibfk_2` FOREIGN KEY (`s_ID`) REFERENCES `student` (`ID`) ON DELETE CASCADE)
```
不论是on delete set null还是on delete cascade都无法修改成功。

## 修改或插入表中数据，考察check子句如何控制校验完整性
```sql
mysql> select * from time_slot;
+--------------+-----+----------+-----------+--------+---------+
| time_slot_id | day | start_hr | start_min | end_hr | end_min |
+--------------+-----+----------+-----------+--------+---------+
| A            | F   |        8 |         0 |      8 |      50 |
| A            | M   |        8 |         0 |      8 |      50 |
| A            | W   |        8 |         0 |      8 |      50 |
| B            | F   |        9 |         0 |      9 |      50 |
| B            | M   |        9 |         0 |      9 |      50 |
| B            | W   |        9 |         0 |      9 |      50 |
| C            | F   |       11 |         0 |     11 |      50 |
| C            | M   |       11 |         0 |     11 |      50 |
| C            | W   |       11 |         0 |     11 |      50 |
| D            | F   |       13 |         0 |     13 |      50 |
| D            | M   |       13 |         0 |     13 |      50 |
| D            | W   |       13 |         0 |     13 |      50 |
| E            | R   |       10 |        30 |     11 |      45 |
| E            | T   |       10 |        30 |     11 |      45 |
| F            | R   |       14 |        30 |     15 |      45 |
| F            | T   |       14 |        30 |     15 |      45 |
| G            | F   |       16 |         0 |     16 |      50 |
| G            | M   |       16 |         0 |     16 |      50 |
| G            | W   |       16 |         0 |     16 |      50 |
| H            | W   |       10 |         0 |     12 |      30 |
+--------------+-----+----------+-----------+--------+---------+
mysql> insert into time_slot values ('H', 'W', '10', '0', '12', '100');
ERROR 1264 (22003): Out of range value for column 'end_min' at row 1
```
数据不能被check通过的就插不进去。
## 定义一个asseration, 并通过修改表中数据考察断言如何控制数据完整性
```sql
delimiter //
create trigger classCount before insert on classroom for each row
  begin
    declare msg varchar(200);
    declare nums int;
    set nums=(select count(*) from classroom);
    if(nums>6) then
      set msg="The count of classroom is above 6.";
      signal sqlstate 'HY000' SET message_text = msg;
    end if;
  end; //
delimiter ;

mysql> delimiter ;
mysql> insert into classroom values ('Packard', '101', '500');
ERROR 1062 (23000): Duplicate entry 'Packard-101' for key 'PRIMARY'
mysql> insert into classroom values ('Packard', '1011', '500');
Query OK, 1 row affected (0.01 sec)

mysql> insert into classroom values ('Packard', '11011', '500');
Query OK, 1 row affected (0.01 sec)

mysql> insert into classroom values ('Packard1', '11011', '500');
ERROR 1644 (HY000): The count of classroom is above 6.
```
插到一定数量之后再插入就会被这个trigger给阻止并打印出message。




