# MySQL数据库表的增删查改

* CRUD : Create(创建), Retrieve(读取)，Update(更新)，Delete（删除）

## Create

> ​	INSERT [INTO] table_name  [(column [, column] ...)]  VALUES (value_list) [, (value_list)] ...  value_list: value, [, value] ...

* insert语句主要有两种情况，一种是全行插入，就是在table_name后面不跟插入列的名称，直接在全行进行插入，还有就是指定列进行插入操作

```sql
-- 创建一张学生表
CREATE TABLE students (
   id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
   sn INT NOT NULL UNIQUE COMMENT '学号',
   name VARCHAR(20) NOT NULL,
   qq VARCHAR(20)
);
```

### 单行数据+全列插入

```sql
-- 插入两条记录，value_list 数量必须和定义表的列的数量及顺序一致
-- 注意，这里在插入的时候，也可以不用指定id(当然，那时候就需要明确插入数据到那些列了)，那么mysql会使用默认的值进行自增。
INSERT INTO students VALUES (100, 10000, '唐三藏', NULL);
Query OK, 1 row affected (0.02 sec)
)
-- 查看插入结果

SELECT * FROM students;
+-----+-------+-----------+-------+
| id | sn   | name     | qq   |
+-----+-------+-----------+-------+
| 100 | 10000 | 唐三藏     | NULL |
| 101 | 10001 | 孙悟空     | 11111 |
+-----+-------+-----------+-------+
2 rows in set (0.00 sec)

```

* 这里student后面不跟列的名称就是一种全列插入，在values后要根据顺序把每一列的值填写进去

### 多行插入 + 指定列插入

```sql
-- 插入两条记录，value_list 数量必须和指定列数量及顺序一致
INSERT INTO students (id, sn, name) VALUES 
 (102, 20001, '曹孟德'), 
 (103, 20002, '孙仲谋');
Query OK, 2 rows affected (0.02 sec)
Records: 2 Duplicates: 0  Warnings: 0

-- 查看插入结果
SELECT * FROM students;
+-----+-------+-----------+-------+
| id | sn   | name     | qq   |
+-----+-------+-----------+-------+
| 100 | 10000 | 唐三藏     | NULL |
| 101 | 10001 | 孙悟空     | 11111 |
| 102 | 20001 | 曹孟德     | NULL |
| 103 | 20002 | 孙仲谋     | NULL |
+-----+-------+-----------+-------+
4 rows in set (0.00 sec)

```

* 这里table_name后面指定了哪些列的内容就插入到哪些列当中，而且可以在插入内容后加上 **,** 继续进行插入操作 -> 多行插入

### 插入否则更新

* 这主要使用于当插入内容与主键或者unique冲突时更新内容使用

```sql
INSERT INTO students (id, sn, name) VALUES (100, 10010, '唐大师')
 ON DUPLICATE KEY UPDATE sn = 10010, name = '唐大师';
 Query OK, 2 rows affected (0.47 sec)

-- 0 row affected: 表中有冲突数据，但冲突数据的值和 update 的值相等
-- 1 row affected: 表中没有冲突数据，数据被插入
-- 2 row affected: 表中有冲突数据，并且数据已经被更新

```

### 替换

```sql
-- 主键 或者 唯一键 没有冲突，则直接插入；
-- 主键 或者 唯一键 如果冲突，则删除后再插入

REPLACE INTO students (sn, name) VALUES (20001, '曹阿瞒');
Query OK, 2 rows affected (0.00 sec)

-- 1 row affected: 表中没有冲突数据，数据被插入
-- 2 row affected: 表中有冲突数据，删除后重新插入
```

> ​	在MySQL中，`REPLACE` 语句是一种用于插入或更新数据的语句。它类似于 `INSERT` 语句，但有一个关键的区别：如果目标表中已经存在一条具有相同唯一键或主键值的记录，**`REPLACE` 会首先删除旧的记录，然后插入新的记录。**这种操作方式有时也被称为“插入或替换”。
>
> `REPLACE` 语句的基本语法如下：
>
> ```sql
> REPLACE INTO table_name (column1, column2, column3, ...)
> VALUES (value1, value2, value3, ...);
> ```
>
> 或者使用另一种形式：
>
> ```sql
> REPLACE INTO table_name
> SET column1 = value1, column2 = value2, ...;
> ```
>
> ### 工作原理
>
> 1. **查找冲突**：MySQL会检查目标表中的数据，查看是否存在与新数据具有相同唯一键或主键的记录。
> 2. **删除旧记录**：如果找到匹配的记录，MySQL会删除这条记录。此删除操作会触发任何相关的删除触发器。
> 3. **插入新记录**：然后，MySQL插入新的记录。这个插入操作会触发任何相关的插入触发器。
>
> ### 注意事项
>
> 1. **删除和插入的开销**：由于 `REPLACE` 先删除旧记录再插入新记录，这个过程可能比简单的更新操作要慢，尤其是在有外键约束或触发器时。此外，这个过程还可能导致自动增长列的值跳跃。
> 2. **唯一性约束**：`REPLACE` 语句只能应用于有唯一键（如主键或唯一索引）的表中。如果没有唯一键，`REPLACE` 将始终插入新记录。
> 3. **触发器**：如上所述，`REPLACE` 操作会触发相关的触发器，所以在设计表和数据库逻辑时需要小心处理这些触发器的影响。
>
> ### 使用场景
>
> `REPLACE` 语句常用于那些需要确保数据唯一性的场景，比如：
>
> - 同步数据：例如从其他数据源同步数据时，如果记录存在则更新，不存在则插入。
> - 数据去重：在某些情况下，可能需要根据唯一键去重，这时可以使用 `REPLACE` 代替复杂的检查和更新逻辑。

## Retrive

```sql
SELECT 
 [DISTINCT] {* | {column [, column] ...} 
 [FROM table_name] 
 [WHERE ...] 
 [ORDER BY column [ASC | DESC], ...] 
 LIMIT ... 
```

案例：

```sql
-- 创建表结构 
CREATE TABLE exam_result ( 
 id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT, 
 name VARCHAR(20) NOT NULL COMMENT '同学姓名', 
 chinese float DEFAULT 0.0 COMMENT '语文成绩', 
 math float DEFAULT 0.0 COMMENT '数学成绩', 
 english float DEFAULT 0.0 COMMENT '英语成绩' 
); 
 
-- 插入测试数据 
INSERT INTO exam_result (name, chinese, math, english) VALUES 
 ('唐三藏', 67, 98, 56), 
 ('孙悟空', 87, 78, 77), 
 ('猪悟能', 88, 98, 90), 
 ('曹孟德', 82, 84, 67), 
 ('刘玄德', 55, 85, 45), 
 ('孙权', 70, 73, 78), 
 ('宋公明', 75, 65, 30); 
Query OK, 7 rows affected (0.00 sec) 
Records: 7 Duplicates: 0 Warnings: 0 

```

### 全列查询

```sql
#通常情况下不建议使用 * 进行全列查询
#1. 查询的列越多，意味这要传输的数据量越大
#2. 可能影响到索引的使用 后面会出博客讲解
mysql> select * from exam_result;
+----+-----------+---------+------+---------+
| id | name      | chinese | math | english |
+----+-----------+---------+------+---------+
|  1 | 唐三藏    |      67 |   98 |      56 |
|  2 | 孙悟空    |      87 |   78 |      77 |
|  3 | 猪悟能    |      88 |   98 |      90 |
|  4 | 曹孟德    |      82 |   84 |      67 |
|  5 | 刘玄德    |      55 |   85 |      45 |
|  6 | 孙权      |      70 |   73 |      78 |
|  7 | 宋公明    |      75 |   65 |      30 |
+----+-----------+---------+------+---------+
7 rows in set (0.00 sec)
```

### 指定列查询

```sql
mysql> select id, name, chinese from exam_result;
+----+-----------+---------+
| id | name      | chinese |
+----+-----------+---------+
|  1 | 唐三藏    |      67 |
|  2 | 孙悟空    |      87 |
|  3 | 猪悟能    |      88 |
|  4 | 曹孟德    |      82 |
|  5 | 刘玄德    |      55 |
|  6 | 孙权      |      70 |
|  7 | 宋公明    |      75 |
+----+-----------+---------+
7 rows in set (0.00 sec)
```

### 查询字段为表达式

* 可以直接将Chinese, English，Math进行相加并且重命名

```sql
mysql> SELECT id, name, chinese + math + english total FROM exam_result; 
+----+-----------+-------+
| id | name      | total |
+----+-----------+-------+
|  1 | 唐三藏    |   221 |
|  2 | 孙悟空    |   242 |
|  3 | 猪悟能    |   276 |
|  4 | 曹孟德    |   233 |
|  5 | 刘玄德    |   185 |
|  6 | 孙权      |   221 |
|  7 | 宋公明    |   170 |
+----+-----------+-------+
7 rows in set (0.01 sec)
```

> ​	这里可以使用 as total 也可以直接使用total进行重命名

### where 条件

* MySQL的where语句就像是c/c++的if一样，有些东西相对简单就不做过多的说明，我只挑几个我认为新手比较陌生的讲解

比较运算符

| 运算符            | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| \>, >=, <, <=     | 大于，大于等于，小于，小于等于                               |
| =                 | 等于，NULL 不安全，例如 NULL = NULL 的结果是 NULL            |
| <=>               | 等于，NULL 安全，例如 NULL <=> NULL 的结果是 TRUE(1)         |
| !=, <>            | 不等于                                                       |
| BETWEEN a0 AND a1 | 范围匹配，[a0, a1]，如果 a0 <= value <= a1，返回 TRUE(1)     |
| IN (option, ...)  | 如果是 option 中的任意一个，返回 TRUE(1)                     |
| IS NULL           | 是 NULL                                                      |
| IS NOT NULL       | 不是 NULL                                                    |
| LIKE              | 模糊匹配。% 表示任意多个（包括 0 个）任意字符；_ 表示任意一个字符 |

逻辑运算符

| 运算符 | 说明                                       |
| ------ | ------------------------------------------ |
| AND    | 多个条件必须都为 TRUE(1)，结果才是 TRUE(1) |
| OR     | 任意一个条件为 TRUE(1), 结果为 TRUE(1)     |
| NOT    | 条件为 TRUE(1)，结果为 FALSE(0)            |

#### between

```sql
语文成绩在 [80, 90] 分的同学及语文成绩\

-- 使用 AND 进行条件连接 
SELECT name, chinese FROM exam_result WHERE chinese >= 80 AND chinese <= 90; 
+-----------+-------+ 
| name | chinese | 
+-----------+-------+ 
| 孙悟空 | 87 | 
| 猪悟能 | 88 | 
| 曹孟德 | 82 | 
+-----------+-------+ 
3 rows in set (0.00 sec) 

-- 使用 BETWEEN ... AND ... 条件  
SELECT name, chinese FROM exam_result WHERE chinese BETWEEN 80 AND 90; 
+-----------+-------+ 
| name | chinese | 
+-----------+-------+ 
| 孙悟空 | 87 | 
| 猪悟能 | 88 | 
| 曹孟德 | 82 | 
+-----------+-------+ 
3 rows in set (0.00 sec)
```

#### In

```sql
数学成绩是 58 或者 59 或者 98 或者 99 分的同学及数学成绩

-- 使用 OR 进行条件连接 
 
SELECT name, math FROM exam_result 
 WHERE math = 58 
 OR math = 59 
 OR math = 98 
 OR math = 99; 
+-----------+--------+ 
| name | math | 
+-----------+--------+ 
| 唐三藏 | 98 | 
| 猪悟能 | 98 | 
+-----------+--------+ 
2 rows in set (0.01 sec) 
-- 使用 IN 条件 
 
SELECT name, math FROM exam_result WHERE math IN (58, 59, 98, 99); 
+-----------+--------+ 
| name | math | 
+-----------+--------+ 
| 唐三藏 | 98 | 
| 猪悟能 | 98 | 
+-----------+--------+ 
2 rows in set (0.00 sec) 
```

#### like

```sql
姓孙的同学 及 孙某同学 

-- % 匹配任意多个（包括 0 个）任意字符 
 
SELECT name FROM exam_result WHERE name LIKE '孙%'; 
+-----------+ 
| name | 
+-----------+ 
| 孙悟空 | 
| 孙权 | 
+-----------+ 
2 rows in set (0.00 sec) 
-- _ 匹配严格的一个任意字符 
 
SELECT name FROM exam_result WHERE name LIKE '孙_'; 
+--------+ 
| name | 
+--------+ 
| 孙权 | 
+--------+ 
1 row in set (0.00 sec) 

```

#### where不能使用别名

```sql
6.2.2.6 总分在 200 分以下的同学 
-- WHERE 条件中使用表达式 
-- 别名不能用在 WHERE 条件中 
 
SELECT name, chinese + math + english 总分 FROM exam_result 
 WHERE chinese + math + english < 200; 
+-----------+--------+ 
| name | 总分 | 
+-----------+--------+ 
| 刘玄德 | 185 | 
| 宋公明 | 170 | 
+-----------+--------+ 
2 rows in set (0.00 sec)
```

> ​	这里有一个注意到点就是哪一个先运行，这里是先选出总分<20的，然后在进行显示，所以显示的是后运行的，所以不能使用别名

### 结果排序

语法：

```sql
-- ASC 为升序（从小到大） 
-- DESC 为降序（从大到小） 
-- 默认为 ASC 
 
SELECT ... FROM table_name [WHERE ...] 
 ORDER BY column [ASC|DESC], [...]; 
```

#### 多级排序

```sql
查询同学各门成绩，依次按 数学降序，英语升序，语文升序的方式显示 

-- 多字段排序，排序优先级随书写顺序 
 
SELECT name, math, english, chinese FROM exam_result 
 ORDER BY math DESC, english, chinese; 
+-----------+--------+--------+-------+ 
| name | math | english | chinese | 
+-----------+--------+--------+-------+ 
| 唐三藏 | 98 | 56 | 67 | 
| 猪悟能 | 98 | 90 | 88 | 
| 刘玄德 | 85 | 45 | 55 | 
| 曹孟德 | 84 | 67 | 82 | 
| 孙悟空 | 78 | 77 | 87 | 
| 孙权 | 73 | 78 | 70 | 
| 宋公明 | 65 | 30 | 75 | 
+-----------+--------+--------+-------+ 
7 rows in set (0.00 sec)
```

* 这里是使用不同的优先级，就是先按照math降序排列，如果math相同就按照English升序排列（如果不写desc还是asc默认是升序），如果这两个都相同就按照Chinese排序

#### 使用别名

```sql
SELECT name, chinese + english + math total FROM exam_result 
 ORDER BY 总分 DESC; 
+-----------+--------+ 
| name | total | 
+-----------+--------+ 
| 猪悟能 | 276 | 
| 孙悟空 | 242 | 
| 曹孟德 | 233 | 
| 唐三藏 | 221 | 
| 孙权 | 221 | 
| 刘玄德 | 185 | 
| 宋公明 | 170 | 
+-----------+--------+ 
7 rows in set (0.00 sec) 
```

> ​	这里排序是可以使用别名的，因为排序是先选择在进行排序