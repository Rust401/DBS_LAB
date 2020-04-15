# 数据库系统（2018-2019春夏学期）实验4
```shell
Project Name:   SQL安全性
Date        :   2019.04.07
```
# 建立表，考察表的生成者拥有表的哪些权限
先用root用户建表，在这之前我们check一下root用户的权限
```mysql
*************************** 1. row ***************************
mysql> select * from mysql.user\G;
                  Host: localhost
                  User: root
           Select_priv: Y
           Insert_priv: Y
           Update_priv: Y
           Delete_priv: Y
           Create_priv: Y
             Drop_priv: Y
           Reload_priv: Y
         Shutdown_priv: Y
          Process_priv: Y
             File_priv: Y
            Grant_priv: Y
       References_priv: Y
            Index_priv: Y
            Alter_priv: Y
          Show_db_priv: Y
            Super_priv: Y
 Create_tmp_table_priv: Y
      Lock_tables_priv: Y
          Execute_priv: Y
       Repl_slave_priv: Y
      Repl_client_priv: Y
      Create_view_priv: Y
        Show_view_priv: Y
   Create_routine_priv: Y
    Alter_routine_priv: Y
      Create_user_priv: Y
            Event_priv: Y
          Trigger_priv: Y
Create_tablespace_priv: Y
              ssl_type: 
            ssl_cipher: 
           x509_issuer: 
          x509_subject: 
         max_questions: 0
           max_updates: 0
       max_connections: 0
  max_user_connections: 0
                plugin: mysql_native_password
 authentication_string: *8F8C25FAC05ED2771F2771B65586040C2EAC530B
      password_expired: N
 password_last_changed: 2019-02-25 14:07:34
     password_lifetime: NULL
        account_locked: N
```
由于root用户被赋予了所有数据库所有表的权限,我们之前用root用户建立库建表删除修改等一些列操作都没有问题。用`select * from mysql.tables_priv`查看下表相关的权限
```sql
mysql> select * from mysql.tables_priv\G;
*************************** 1. row ***************************
       Host: localhost
         Db: mysql
       User: mysql.session
 Table_name: user
    Grantor: boot@connecting host
  Timestamp: 0000-00-00 00:00:00
 Table_priv: Select
Column_priv: 
*************************** 2. row ***************************
       Host: localhost
         Db: sys
       User: mysql.sys
 Table_name: sys_config
    Grantor: root@localhost
  Timestamp: 2019-02-25 13:57:23
 Table_priv: Select
Column_priv: 
*************************** 3. row ***************************
       Host: localhost
         Db: lab4
       User: rust401
 Table_name: course
    Grantor: root@localhost
  Timestamp: 0000-00-00 00:00:00
 Table_priv: Select,Insert,Update,Delete
Column_priv: 
3 rows in set (0.00 sec)
```
除了第三row之外发现找不到啥有用的信息，先前那么多DB下面那么多的表都找不到相关的记录。因为这个`table_priv`表保存的是权限被限制的用户针对某个具体表的访问权限。拿其中的第三row来说，我创建了一个localhost上的新用户rust401，起初没有赋予其任何权限。
```sql
*************************** 4. row ***************************
                  Host: localhost
                  User: rust401
           Select_priv: N
           Insert_priv: N
           Update_priv: N
           Delete_priv: N
           Create_priv: N
             Drop_priv: N
           Reload_priv: N
         Shutdown_priv: N
          Process_priv: N
             File_priv: N
            Grant_priv: N
       References_priv: N
            Index_priv: N
            Alter_priv: N
          Show_db_priv: N
            Super_priv: N
 Create_tmp_table_priv: N
      Lock_tables_priv: N
          Execute_priv: N
       Repl_slave_priv: N
      Repl_client_priv: N
      Create_view_priv: N
        Show_view_priv: N
   Create_routine_priv: N
    Alter_routine_priv: N
      Create_user_priv: N
            Event_priv: N
          Trigger_priv: N
Create_tablespace_priv: N
              ssl_type: 
            ssl_cipher: 
           x509_issuer: 
          x509_subject: 
         max_questions: 0
           max_updates: 0
       max_connections: 0
  max_user_connections: 0
                plugin: mysql_native_password
 authentication_string: *8F8C25FAC05ED2771F2771B65586040C2EAC530B
      password_expired: N
 password_last_changed: 2019-04-07 12:44:58
     password_lifetime: NULL
        account_locked: N
```
显然这个用户无法创建任何数据库
```sql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
+--------------------+
2 rows in set (0.01 sec)
mysql> create database dude;
ERROR 1044 (42000): Access denied for user 'rust401'@'localhost' to database 'dude'
```
然后我给它赋予书里给的样例中的学校数据库中的advisor和course表的权限
```sql
mysql> grant insert,select,update on lab4.advisor to rust401@'localhost';
Query OK, 0 rows affected (0.00 sec)
mysql> show tables;
+----------------+
| Tables_in_lab4 |
+----------------+
| advisor        |
| course         |
+----------------+
2 rows in set (0.00 sec)
```
现在我们可以看到这两张被赋予权限的表了。
我们看看系统中表权限的记录(用的root用户操作)
```sql
*************************** 3. row ***************************
       Host: localhost
         Db: lab4
       User: rust401
 Table_name: course
    Grantor: root@localhost
  Timestamp: 0000-00-00 00:00:00
 Table_priv: Select,Insert,Update,Delete
Column_priv: 
*************************** 4. row ***************************
       Host: localhost
         Db: lab4
       User: rust401
 Table_name: advisor
    Grantor: root@localhost
  Timestamp: 0000-00-00 00:00:00
 Table_priv: Select,Insert,Update
Column_priv: 
4 rows in set (0.00 sec)
```
可以发现特定用户对于特定表的权限已经被搞定。  
接着我们试着把整个`lab4`的权限给rust401
```sql
mysql> grant all privileges on lab4.* to rust401@'localhost';
Query OK, 0 rows affected (0.01 sec)
```
然后重新登录rust401之后查看我们lab4中的table
```sql
mysql> show tables;
+----------------+
| Tables_in_lab4 |
+----------------+
| advisor        |
| classroom      |
| course         |
| department     |
| instructor     |
| prereq         |
| section        |
| student        |
| takes          |
| teaches        |
| time_slot      |
+----------------+
11 rows in set (0.00 sec)
```
再check下系统中对权限的记录
```sql
*************************** 1. row ***************************
       Host: localhost
         Db: mysql
       User: mysql.session
 Table_name: user
    Grantor: boot@connecting host
  Timestamp: 0000-00-00 00:00:00
 Table_priv: Select
Column_priv: 
*************************** 2. row ***************************
       Host: localhost
         Db: sys
       User: mysql.sys
 Table_name: sys_config
    Grantor: root@localhost
  Timestamp: 2019-02-25 13:57:23
 Table_priv: Select
Column_priv: 
*************************** 3. row ***************************
       Host: localhost
         Db: lab4
       User: rust401
 Table_name: course
    Grantor: root@localhost
  Timestamp: 0000-00-00 00:00:00
 Table_priv: Select,Insert,Update,Delete
Column_priv: 
*************************** 4. row ***************************
       Host: localhost
         Db: lab4
       User: rust401
 Table_name: advisor
    Grantor: root@localhost
  Timestamp: 0000-00-00 00:00:00
 Table_priv: Select,Insert,Update
Column_priv: 
4 rows in set (0.00 sec)
```
系统中记录还是没有改变。所以问题来了，此刻rust401对advisor这个表究竟有没有删除功能？
```
mysql> delete from advisor where i_ID=22222;
Query OK, 2 rows affected (0.01 sec)

mysql> select * from advisor;
+-------+-------+
| s_ID  | i_ID  |
+-------+-------+
| 12345 | 10101 |
| 00128 | 45565 |
| 76543 | 45565 |
| 23121 | 76543 |
| 98988 | 76766 |
| 76653 | 98345 |
| 98765 | 98345 |
+-------+-------+
7 rows in set (0.00 sec)
```
发现还是可以删的，所以`mysql_table_priv`这个表里的记录比较没用，肯定有更高优先级的权限控制，这部分  
//TO DO

我们先把rust401的权限撤销掉
```sql
mysql> revoke all privileges on lab4.* from rust401@'localhost';
Query OK, 0 rows affected (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
```
然后测试下现在的权限
```sql
mysql> show tables;
+----------------+
| Tables_in_lab4 |
+----------------+
| advisor        |
| course         |
+----------------+
2 rows in set (0.00 sec)

mysql> select * from advisor;
+-------+-------+
| s_ID  | i_ID  |
+-------+-------+
| 12345 | 10101 |
| 00128 | 45565 |
| 76543 | 45565 |
| 23121 | 76543 |
| 98988 | 76766 |
| 76653 | 98345 |
| 98765 | 98345 |
+-------+-------+
7 rows in set (0.00 sec)

mysql> delete from advisor where i_ID=10101;
ERROR 1142 (42000): DELETE command denied to user 'rust401'@'localhost' for table 'advisor'
```
这下子原先设置的权限还在，而且advisor那个表不能删除。所以这里面的优先级机制极有可能跟`ALL PRIVILEGE`这个关键词有关。
我们不用这个关键词赋予权限试一下
```sql
mysql> grant insert,select,update on lab4.* to rust401@'localhost';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```
看看现在可以访问的表
```sql
Database changed
mysql> show tables;
+----------------+
| Tables_in_lab4 |
+----------------+
| advisor        |
| classroom      |
| course         |
| department     |
| instructor     |
| prereq         |
| section        |
| student        |
| takes          |
| teaches        |
| time_slot      |
+----------------+
11 rows in set (0.00 sec)
```
再看看系统中的情况
```sql
*************************** 1. row ***************************
       Host: localhost
         Db: mysql
       User: mysql.session
 Table_name: user
    Grantor: boot@connecting host
  Timestamp: 0000-00-00 00:00:00
 Table_priv: Select
Column_priv: 
*************************** 2. row ***************************
       Host: localhost
         Db: sys
       User: mysql.sys
 Table_name: sys_config
    Grantor: root@localhost
  Timestamp: 2019-02-25 13:57:23
 Table_priv: Select
Column_priv: 
*************************** 3. row ***************************
       Host: localhost
         Db: lab4
       User: rust401
 Table_name: course
    Grantor: root@localhost
  Timestamp: 0000-00-00 00:00:00
 Table_priv: Select,Insert,Update,Delete
Column_priv: 
*************************** 4. row ***************************
       Host: localhost
         Db: lab4
       User: rust401
 Table_name: advisor
    Grantor: root@localhost
  Timestamp: 0000-00-00 00:00:00
 Table_priv: Select,Insert,Update
Column_priv: 
4 rows in set (0.00 sec)
```
令人震惊的是，系统中压根没有记录，看来跟`ALL PRIVILEGE`没啥关系，跟`lab4.*`有关。
这时候我看看能不能创建一个表
```sql
mysql> create table tt
    -> ( man varchar(10),women varchar(10));
ERROR 1142 (42000): CREATE command denied to user 'rust401'@'localhost' for table 'tt'
```
显然不行，因为我只是给了lab4中所有表的I,D,U权限，而没有给lab4的权限。
所以我把lab4的权限给下rust401
```sql
mysql> grant all privileges on lab4 to rust401@'localhost';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```
再创建表试试看
```sql
mysql> create table tt ( man varchar(10),women varchar(10));
ERROR 1142 (42000): CREATE command denied to user 'rust401'@'localhost' for table 'tt'
```
新表又被拒绝了，愁。
然后把create 语句改了一下
```sql
mysql> grant create on lab4.* to rust401@'localhost' identified by 'Hmf812754';
Query OK, 0 rows affected, 1 warning (0.01 sec)
```
这下可以插表了
```sql
Database changed
mysql> create table tt ( man varchar(10),women varchar(10));
Query OK, 0 rows affected (0.03 sec)
```
给rust401设置了lab4的create，insert，select，update权限。然后测试了一波
```sql
mysql> drop table tt;
ERROR 1142 (42000): DROP command denied to user 'rust401'@'localhost' for table 'tt'
mysql> create table papa ( man varchar(10),women varchar(10));
Query OK, 0 rows affected (0.03 sec)

mysql> select * from papa;
Empty set (0.00 sec)

mysql> insert into papa ('shabi','doubi');
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''shabi','doubi')' at line 1
mysql> insert into papa values('shabi','doubi');
Query OK, 1 row affected (0.01 sec)

mysql> select * from papa;
+-------+-------+
| man   | women |
+-------+-------+
| shabi | doubi |
+-------+-------+
1 row in set (0.00 sec)

mysql> delete from papa where man=shabi;
ERROR 1142 (42000): DELETE command denied to user 'rust401'@'localhost' for table 'papa'
```
果不其然，drop和delete操作显然是不能用的。新创建的表的权限跟当初设置的这个用户对于该数据库的权限是一样的。也就是说，我们给该用户设置了database的权限之后，这个用户create一个表（假设可以的话），那这个表的权限跟该数据库的公共权限是一样的。
# 用grant和revoke命令进行回收，看看有什么作用
这里不妨先把所所有权限拿来，然后把delete权限搞掉试一下
```sql
mysql> create user papa@'localhost' identified by 'Hmf812754';
Query OK, 0 rows affected (0.00 sec)

mysql> grant all privileges on lab4.* to papa@'localhost' identified by 'Hmf812754';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> revoke delete on lab4.* from papa@'localhost';
Query OK, 0 rows affected (0.00 sec)
```
直接看看lab4中玩意能不删
```sql
mysql> select * from papa;
+-------+-------+
| man   | women |
+-------+-------+
| shabi | doubi |
+-------+-------+
1 row in set (0.00 sec)

mysql> delete from papa where man=shabi;
ERROR 1142 (42000): DELETE command denied to user 'papa'@'localhost' for table 'papa'
```
这里revoke命令收回用户权限成功。
# 建立视图，把视图的查询权限给其它用户
```sql
mysql> create view classroom_No as (select building,room_number from classroom);
Query OK, 0 rows affected (0.03 sec)
mysql> select * from classroom_no;;
+----------+-------------+
| building | room_number |
+----------+-------------+
| Packard  | 101         |
| Painter  | 514         |
| Taylor   | 3128        |
| Watson   | 100         |
| Watson   | 120         |
+----------+-------------+
5 rows in set (0.01 sec)
```
视图建立完毕(root)，从rust401进行访问
```sql
mysql> use lab4;
Database changed
mysql> show tables;
+----------------+
| Tables_in_lab4 |
+----------------+
| advisor        |
| classroom      |
| classroom_no   |
| course         |
| department     |
| instructor     |
| papa           |
| prereq         |
| section        |
| student        |
| takes          |
| teaches        |
| time_slot      |
| tt             |
+----------------+
14 rows in set (0.00 sec)
```
表，因为只给了select和show view的权限，只能看，不能删。
```sql
mysql> select * from classroom_no;
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
可以正常看。
```sql
mysql> delete from classroom_no where room_number=101;
ERROR 1142 (42000): DELETE command denied to user 'mama'@'localhost' for table 'classroom_no'
```
delete操作显然是不行的






