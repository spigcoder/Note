# MySQL表的约束



* 为什么要有表的各种约束：通过约束，让我们未来插入数据库表中的数据是符合预期的，约束的最终目标是保证数据的完整性和可预期性。

## 空属性

* 两个值：null 和 not null
* 数据库默认字段基本都是字段为空，但实际开发中，尽可能保证字段不为空，因为数据为空没法进行计算

​		案例： 创建一个班级表，包含班级名和班级所在的教室。 站在正常的业务逻辑中： 如果班级没	有名字，你不知道你在哪个班级 如果教室名字可以为空，就不知道在哪上课 所以我们在设计数据库	表的时候，一定要在表中进行限制，满足上面条件的数据就不能插入到表中。这就是“约束”。

``` sql
mysql> create table class(                                                                  │
    -> class_room varchar(20) ,                                                     │
    -> class_name varchar(20)) not null;                                                    │
Query OK, 0 rows affected (0.08 sec)                                                        │
                                                                                            │
mysql> desc classs;                                                                         
ERROR 1146 (42S02): Table 'test.classs' doesn't exist                                       │
mysql> desc class                                                                           │
    -> ;                                                                                    │
+------------+-------------+------+-----+---------+-------+                                 │
| Field      | Type        | Null | Key | Default | Extra |                                 │
+------------+-------------+------+-----+---------+-------+                                 │
| class_room | varchar(20) | YES   |     | NULL    |       |                                 │
| class_name | varchar(20) | NO   |     | NULL    |       |                                 │
+------------+-------------+------+-----+---------+-------+                                 │
2 rows in set (0.00 sec)        
```

​	当我们加入not null约束之后，想插入时就不可以插入null

```sql
mysql> insert into class(class_name) values(null);                                          
ERROR 1048 (23000): Column 'class_name' cannot be null            
```

## 默认值

* 默认值：某一种数据会经常性的出现某个具体的值，可以在一开始就指定好，在需要真实数据的时候， 用户可以选择性的使用默认值 -> 类似于c\c++的默认参数

```sql
mysql> create table people(                                                                                                                                                             
    -> name varchar(20) not null,                                                                                                                                                       
    -> age tinyint unsigned default 18,                                                                                                                                                 
    -> sex char(2) default '男'                                                                                                                                                         
    -> );
```

插入 ：

```sql
insert into people(name) values('spigcoder');
```

```sql
mysql> select * from people;                                                                                                                                                            
+-----------+------+------+                                                                                                                                                             
| name      | age  | sex  |                                                                                                                                                             
+-----------+------+------+                                                                                                                                                             
| spigcoder |   18 | 男   |                                                                                                                                                             
+-----------+------+------+                                                                                                                                                             
1 row in set (0.00 sec)  
```

这里可以明显的看到，我们直插入了名字，但是性别和年龄都是用默认值显示了出来

default和not null是可以一起使用的，它们之间是并不冲突的

> ​	如果用户像插入数据，但此时设置了not null，那么是不能插入null的，但如果用户此时忽略了这一列，那么在插入时就会使用默认值进行插入

## 列描述

```sql
mysql> create table tt12 (
   -> name varchar(20) not null comment '姓名',
   -> age tinyint unsigned default 0 comment '年龄',
   -> sex char(2) default '男' comment '性别'
   -> );
```

* 这时只是desc tt12是查询不到comment的，但是这时如果使用show create table tt12 \G就可以看到这个信息了

```sql
mysql> show create table tt12\G
*************************** 1. row ***************************
       Table: tt12
Create Table: CREATE TABLE `tt12` (
 `name` varchar(20) NOT NULL COMMENT '姓名',
 `age` tinyint(3) unsigned DEFAULT '0' COMMENT '年龄',
 `sex` char(2) DEFAULT '男' COMMENT '性别'
) ENGINE=MyISAM DEFAULT CHARSET=gbk
1 row in set (0.00 sec)

```

## zerofill

```sql
mysql> create table test1( id int zerofill);
```

```sql
mysql> insert into test2 values (21);
```

```sql
mysql> select * from test2;                                                                                                                                                      
+------------+                                                                                                                                                                        
| id         |                                                                                                                                                                        
+------------+                                                                                                                                                                          
| 0000000021 |                                                                                                                                                                          
+------------+                                                                                                                                                                          
1 row in set (0.00 sec)   
```

这里我们会看到，当加上zerofill时前面会多出8个0，我们就不难推断zerofill是一个补零的约束

```sql
mysql> alter table tt3 change a a int(5) unsigned zerofill;
mysql> show create table tt3\G
*************************** 1. row ***************************
       Table: tt3

Create Table: CREATE TABLE `tt3` (
 `a` int(5) unsigned zerofill DEFAULT NULL,  --具有了zerofill

 `b` int(10) unsigned DEFAULT NULL

) ENGINE=MyISAM DEFAULT CHARSET=gbk

1 row in set (0.00 sec)

1 row in set (0.00 sec)   
```

这里就可以看到int(5) b int (10),其中这个位是可以自行设置的，如果插入的元素没有到达这么多位，就在前面补零，如果超过了就不处理

## 主键

* 主键：primary key用来唯一的约束该字段里面的数据，不能重复，不能为空，一张表中最多只能有一个 主键；主键所在的列通常是整数类型。

这个主键简单理解就是这一行不能出现重复的元素，比如学号这种只有一份的可以设置为主键

```sql
mysql> create table tt13 (
-> id int unsigned primary key comment '学号不能为空',
-> name varchar(20) not null);
Query OK, 0 rows affected (0.00 sec)
mysql> desc tt13;
+-------+------------------+------+-----+---------+-------+
| Field | Type             | Null | Key | Default | Extra |
+-------+------------------+------+-----+---------+-------+
| id   | int(10) unsigned | NO   | PRI | NULL   |       | <= key 中 pri表示该字段是主键

| name | varchar(20)     | NO   |     | NULL   |       |
+-------+------------------+------+-----+---------+-------+
```

这时如果你向表中主键的那一行插入两个一样的元素就会报错

```sql
mysql> insert into tt13 values(1, 'aaa');
Query OK, 1 row affected (0.00 sec)

mysql> insert into tt13 values(1, 'aaa');
ERROR 1062 (23000): Duplicate entry '1' for key 'PRIMARY'
```

### 追加主键

```sql
alter table 表名 add primary key(字段列表)
```

### 删除主键

```sql
alter table 表名 drop primary key
```

### 复合主键

```sql
mysql> create table tt14(
-> id int unsigned,
-> course char(10) comment '课程代码',
-> score tinyint unsigned default 60 comment '成绩',
-> primary key(id, course) -- id和course为复合主键
-> );
Query OK, 0 rows affected (0.01 sec)

```

> 这个复合主键给大家解释一下:
>
> ​	就是说id可以相同如果你选的课程不同的化 ->一个人可以选多个课
>
> ​	课程可以相同如果id不同的化 -> 可以多个人选一个课
>
> ​	但是不能id和课程都相同 -> 不可以一个人这个课选多遍

eg:

```sql
mysql> create table score( id int, course char(20),  score tinyint, primary key(id, course));                                                                                           
Query OK, 0 rows affected (0.05 sec)                                                                                                                                                    
                                                                                                                                                                                        
mysql> desc score;                                                                                                                                                                      
+--------+----------+------+-----+---------+-------+                                                                                                                                    
| Field  | Type     | Null | Key | Default | Extra |                                                                                                                                    
+--------+----------+------+-----+---------+-------+                                                                                                                                    
| id     | int      | NO   | PRI | NULL    |       |                                                                                                                                    
| course | char(20) | NO   | PRI | NULL    |       |                                                                                                                                    
| score  | tinyint  | YES  |     | NULL    |       |                                                                                                                                    
+--------+----------+------+-----+---------+-------+                                                                                                                                    
3 rows in set (0.00 sec)                                                                                                                                                                
                                                                                                                                                                                        
mysql> insert into score values(123, '数学', 99);                                                                                                                                       
Query OK, 1 row affected (0.01 sec)                                                                                                                                                     
                                                                                                                                                                                        
mysql> insert into score values(123, '英语', 99);                                                                                                                                       
Query OK, 1 row affected (0.01 sec)                                                                                                                                                     
                                                                                                                                                                                        
mysql> insert into score values(321, '英语', 99);                                                                                                                                       
Query OK, 1 row affected (0.01 sec)                                                                                                                                                     
                                                                                                                                                                                        
mysql> select * from score;                                                                                                                                                             
+-----+--------+-------+                                                                                                                                                                
| id  | course | score |                                                                                                                                                                
+-----+--------+-------+                                                                                                                                                                
| 123 | 数学   |    99 |                                                                                                                                                                
| 123 | 英语   |    99 |                                                                                                                                                                
| 321 | 英语   |    99 |                                                                                                                                                                
+-----+--------+-------+                                                                                                                                                                
3 rows in set (0.00 sec)                                                                                                                                                                
                                                                                                                                                                                        
mysql> insert into score values(321, '英语', 88);                                                                                                                                       
ERROR 1062 (23000): Duplicate entry '321-英语' for key 'score.PRIMARY'       
```

可以看到123 可以选择两门不同的课，英语也可以同时被123和321选择，但是如果321插入两次英语但是成绩不一样是就会报错。

## 自增长

auto_increment: 当对应的字段，不给值，会自动的被系统触发，系统会从当前字段中已经有的最大值+1操作，得到一个新的不同的值 -> 通常与主键搭配，作为逻辑主键

* 任何一个字段要做自增长，前提是本身是一个索引，（key 一栏有值）
* 自增长字段必须是整数
* 一张表最多只能有一个自增长

```sql
mysql> create table auto_increment(
    -> id int unsigned primary key auto_increment,
    -> name varchar(10) not null
    -> );
```

```sql
mysql> insert into auto_increment(name) values('a');
Query OK, 1 row affected (0.02 sec)

mysql> insert into auto_increment(name) values('n');
Query OK, 1 row affected (0.00 sec)

mysql> select * from auto_increment;
+----+------+
| id | name |
+----+------+
|  1 | a    |
|  2 | n    |
+----+------+
2 rows in set (0.00 sec)

mysql> select last_insert_id();
+------------------+
| last_insert_id() |
+------------------+
|                2 |
+------------------+
1 row in set (0.00 sec)

```

> ​	可以看到，当我们没有插入id时，因为有auto_increment 所以每次插入都会**在上一次的基础上**进行自增插入

```sql
mysql> insert into auto_increment values(122,'n');
Query OK, 1 row affected (0.01 sec)

mysql> insert into auto_increment(name) values('m');
Query OK, 1 row affected (0.01 sec)

mysql> select * from auto_increment;
+-----+------+
| id  | name |
+-----+------+
|   1 | a    |
|   2 | n    |
| 122 | n    |
| 123 | m    |
+-----+------+
4 rows in set (0.00 sec)

```

## 唯一键

* 一张表中往往有 很多字段需要唯一性，数据不能重复，但是一张表中只能有一个主键，所以需要有唯一键来保证其他不能重复的元素不出现重复的值

```sql
mysql> create table student (
   -> id char(10) unique comment '学号，不能重复，但可以为空',
   -> name varchar(10)
   -> );
Query OK, 0 rows affected (0.01 sec)
mysql> insert into student(id, name) values('01', 'aaa');
Query OK, 1 row affected (0.00 sec)
mysql> insert into student(id, name) values('01', 'bbb'); --唯一约束不能重复

ERROR 1062 (23000): Duplicate entry '01' for key 'id'

mysql> insert into student(id, name) values(null, 'bbb'); -- 但可以为空

Query OK, 1 row affected (0.00 sec)
mysql> select * from student;
+------+------+
| id   | name |
+------+------+
| 01   | aaa |
| NULL | bbb |
+------+------+

```

> ​	唯一键允许为空，而且可以多个为空，空字段不做唯一比较。

## 外键

* 外键用于定义主表和从表之间的关系：外键约束主要定义在从表上，主表则必须是有主键约束或者unique约束的。当定义外键之后，**要求外键数据必须在主表的主键列存在或为nulll**;

语法：

> foreign key (字段名) references 主表(列) 

![QQ20240729-140921](C:/Users/Administrator/Desktop/QQ20240729-140921.png)

```sql
mysql> create table class(
    -> id int primary key,
    -> name varchar(30) not null comment 'class name'
    -> );
Query OK, 0 rows affected (0.03 sec)

mysql> create table stu(
    -> id int primary key,
    -> name varchar(30) not null,
    -> class_id int,
    #建立外键，将从表中的class_id 绑定到 class中的id
    -> foreign key (class_id) references class(id)
    -> );
Query OK, 0 rows affected (0.04 sec)

mysql> desc stu;
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| id       | int         | NO   | PRI | NULL    |       |
| name     | varchar(30) | NO   |     | NULL    |       |
| class_id | int         | YES  | MUL | NULL    |       |
+----------+-------------+------+-----+---------+-------+
3 rows in set (0.00 sec)

mysql> insert into class values(1,101),(2,102);
Query OK, 2 rows affected (0.01 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> select * from class;
+----+------+
| id | name |
+----+------+
|  1 | 101  |
|  2 | 102  |
+----+------+
2 rows in set (0.00 sec)

mysql> insert into stu values(100, '张三', 1);
Query OK, 1 row affected (0.01 sec)

mysql> insert into stu values(101, '李四', 2);
Query OK, 1 row affected (0.01 sec)

#如果像插入到一个不存在的class_id是不允许的
mysql> insert into stu values(102, '李四', 3);
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`test`.`stu`, CONSTRAINT `stu_ibfk_1` FOREIGN KEY (`class_id`) REFERENCES `class` (`id`))

```

> ​	这里像删除主表中的某个元素，就要保证从表中没有元素是绑定在这个主表上的。