# MySQL客户端命令



## 直接输入SQL语句

* 使用MySQL客户端连接到服务器之后，可以发送SQL语句到服务器执行，并且以；和\g, \G作为结束不同的结束方式显示内容有所不同**

> TIPS:
>
> 1. ;和\g结尾以表格的形式显示结果
> 1. \G以行的形式显示结果

* 在连接到服务器之后可以使用help或者\h来查看命令列表，可以根据需要自行测试，\d，\t，\T都比较常用

* help content可以查看关于MySQL数据库使用的具体帮助，包括用户管理、SQL语法、数据类型、组件等相关内容列表



## 从.sql文件执行SQL语句



### 使用source命令

1. 准备.sql文件，命名为test.sql，内容如下

   ```sql
   SET NAMES utf8mb4;
   
   SET FOREIGN_KEY_CHECKS = 0;
   
   DROP DATABASE IF EXISTS `test_db`;
   
   CREATE DATABASE `test_db` CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
   USE `test_db`;
   
   -- ----------------------------
   -- Table structure for classes
   -- ----------------------------
   
   DROP TABLE IF EXISTS `classes`;
   
   CREATE TABLE `classes` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `name` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL 
   
   DEFAULT NULL,
    `desc` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL 
   
   DEFAULT NULL,
    PRIMARY KEY (`id`) USING BTREE
   ) ENGINE = InnoDB AUTO_INCREMENT = 4 CHARACTER SET = utf8mb4 COLLATE = 
   utf8mb4_general_ci ROW_FORMAT = Dynamic;
   
   -- ----------------------------
   -- Records of classes
   -- ----------------------------
   
   INSERT INTO `classes` VALUES (1, '计算机系2019级1班', '学习了计算机原理、C和Java语
   ⾔、数据结构和算法');
   
   INSERT INTO `classes` VALUES (2, '中⽂系2019级3班', '学习了中国传统⽂学');
   
   INSERT INTO `classes` VALUES (3, '⾃动化2019级5班', '学习了机械⾃动化');
   
   -- ----------------------------
   -- Table structure for course
   -- ----------------------------
   
   DROP TABLE IF EXISTS `course`;
   
   CREATE TABLE `course` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `name` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL 
   
   DEFAULT NULL
   PRIMARY KEY (`id`) USING BTREE
   ) ENGINE = InnoDB AUTO_INCREMENT = 7 CHARACTER SET = utf8mb4 COLLATE = 
   utf8mb4_general_ci ROW_FORMAT = Dynamic;
   
   -- ----------------------------
   -- Records of course
   -- ----------------------------
   
   INSERT INTO `course` VALUES (1, 'Java');
   
   INSERT INTO `course` VALUES (2, '中国传统⽂化');
   
   INSERT INTO `course` VALUES (3, '计算机原理');
   
   INSERT INTO `course` VALUES (4, '语⽂');
   
   INSERT INTO `course` VALUES (5, '⾼阶数学');
   
   INSERT INTO `course` VALUES (6, '英⽂');
   
   -- ----------------------------
   -- Table structure for score
   -- ----------------------------
   
   DROP TABLE IF EXISTS `score`;
   
   CREATE TABLE `score` (
    `score` decimal(3, 1) NULL DEFAULT NULL,
    `student_id` int(11) NULL DEFAULT NULL,
    `course_id` int(11) NULL DEFAULT NULL
   
   ) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci 
   ROW_FORMAT = Dynamic;
   
   -- ----------------------------
   -- Records of score
   -- ----------------------------
   
   INSERT INTO `score` VALUES (70.5, 1, 1);
   
   INSERT INTO `score` VALUES (98.5, 1, 3);
   
   INSERT INTO `score` VALUES (33.0, 1, 5);
   
   INSERT INTO `score` VALUES (98.0, 1, 6);
   
   INSERT INTO `score` VALUES (60.0, 2, 1);
   
   INSERT INTO `score` VALUES (59.5, 2, 5);
   
   INSERT INTO `score` VALUES (33.0, 3, 1);
   
   INSERT INTO `score` VALUES (68.0, 3, 3);
   
   INSERT INTO `score` VALUES (99.0, 3, 5);
   
   INSERT INTO `score` VALUES (67.0, 4, 1);
   
   INSERT INTO `score` VALUES (23.0, 4, 3);
   
   INSERT INTO `score` VALUES (56.0, 4, 5);
   
   INSERT INTO `score` VALUES (72.0, 4, 6);
   
   INSERT INTO `score` VALUES (81.0, 5, 1);
   
   INSERT INTO `score` VALUES (37.0, 5, 5);
   
   -- ----------------------------
   -- Table structure for student
   -- ----------------------------
   DROP TABLE IF EXISTS `student`;
   
   CREATE TABLE `student` (
    `id` int(11) PRIMARY KEY AUTO_INCREMENT,
    `sn` int(11) NOT NULL COMMENT '学号',
    `name` varchar(20) NOT NULL COMMENT '姓名',
    `mail` varchar(20) COMMENT 'QQ邮箱'
   
   ) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci 
   ROW_FORMAT = Dynamic;
   
   -- ----------------------------
   -- Records of student
   -- ----------------------------
   
   INSERT INTO `student` VALUES (1, 50001, '张三', 'zs@bit.com');
   
   INSERT INTO `student` VALUES (2, 50002, '李四', 'ls@bit.com');
   
   INSERT INTO `student` VALUES (3, 50003, '王五', 'ww@bit.com');
   
   INSERT INTO `student` VALUES (4, 50004, '赵六', 'zl@bit.com');
   
   INSERT INTO `student` VALUES (5, 50005, '钱七', 'qq@bit.com');
   
   SET FOREIGN_KEY_CHECKS = 1;
   
   ```

   

2. 确定sql在系统中的绝对路径

3. 连接数据库查看已有数据库

4. 使用source命令执行.sql文件 -> source + sql文件的绝对路径

5. 使用show databases查看数据库是否导入成功

   **最终结果**

   ![QQ20240722-132446](C:\Users\Administrator\Desktop\QQ20240722-132446.png)

### 直接导入

* 直接使用mysql客户端程序导入.sql并执行相应的SQL语句，使用如下命令

  ```sql
  mysql db_name < text_file # 在指定的数据库下执⾏SQL,前提是数据库必须提前建⽴好 
  
  mysql < text_file # 不指定数据库.sql中必须有USE [database_name]，来指定要操作的数据库
  ```

  
