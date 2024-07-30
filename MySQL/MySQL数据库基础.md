# MySQL数据库基础



## 数据库的操作



### 查看数据库

* 显示当前数据库

  > ​	SHOW DATABASES;	

  ```sql
  mysql> SHOW DATABASES;
  +--------------------+
  | Database           |
  +--------------------+
  | information_schema |
  | mysql              |
  | performance_schema |
  | sys                |
  | test_db            |
  +--------------------+
  5 rows in set (0.00 sec)
  ```

### 创建数据库

语法：

> ​	CREATE DATABASES [IF NOT EXISTS] db_name [create_specification]

* []中的内容是可以省略的，IF NOT EXISTS 是如果有为db_name就不创建，如果没有就创建一个
* create_specification 是一些创建的选项，如设置校验规则和指定数据库采用的字符集等
* CHARACTER SET: 指定数据库采用的字符集
* COLLATE: 指定数据库字符集的校验规则

eg：
```sql
CREATE DATABASE IF NOT EXISTS db_test CHARACTER SET utf8mb4;
```

在Linux中在MySQL中创建一个数据库就相当于在/var/lib/mysql中创建一个目录

```sql
mysql> CREATE DATABASE IF NOT EXISTS test1;
Query OK, 1 row affected (0.07 sec)
```

```sql
root@VM-24-7-ubuntu:/var/lib/mysql# ls
 auto.cnf        binlog.000007    '#ib_16384_0.dblwr'   mysql                sys
 binlog.000001   binlog.000008    '#ib_16384_1.dblwr'   mysql.ibd            test1
 binlog.000002   binlog.index      ib_buffer_pool       performance_schema   test_db
 binlog.000003   ca-key.pem        ibdata1              private_key.pem      undo_001
 binlog.000004   ca.pem            ibtmp1               public_key.pem       undo_002
 binlog.000005   client-cert.pem  '#innodb_redo'        server-cert.pem
 binlog.000006   client-key.pem   '#innodb_temp'        server-key.pem

```

可以看到这里就有一个test1出现。

### 使用数据库

> USE db_name

### 删除数据库

> DROP DATABASE [IF EXISTS] db_name;

* 数据库在删除之后，里面的表和数据全部被删除

## 表操作

需要操作数据库的表时，需要先使用该数据库

> USE db_test;

### 展示表格

> show tables;

### 查看表结构

> DESC table_name;

### 创建表

语法：

>CREATE TABLE table_name ( 
>
>​		field1 datatype, 
>
>​		field2 datatype, 
>
>​		field3 datatype 
>
>);

* datatype 是指的数据类型，将在下一小节重点讲解，这一节主要讲的是表的基本用法

eg:

```sql
create table stu_test (  

	id int,  
    
     name varchar(20) comment '姓名',  
    
	password varchar(50) comment '密码',  
    
	age int,  
    
	sex varchar(1),  
    
	birthday timestamp,  
    
	amout decimal(13,2),  

);
```



* 后面的comment是增加字段说明

### 删除表

语法:

> DROP [TEMPORARY] TABLE [IF EXISTS] **tbl_name** [, tbl_name] ...

eg:
>-- 删除 stu_test 表
>
>drop table stu_test;
>
>-- 如果存在 stu_test 表，则删除 stu_test 表
>
>drop table if exists stu_test;
