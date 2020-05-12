```
SHOW DATABASES;
USE pharmacy_ms;
SHOW TABLES;
```



# mysql 基础巩固
## 1. 查询表达式
```
SELECT 100 / 2;
```


## 2. 起别名
```sql
SELECT e_name AS name, sex AS s
FROM employee;
SELECT e_name name, sex sx
FROM employee;
SELECT e_name "emp name", sex s
FROM employee;
```


## 3. 去重
```
SELECT DISTINCT education
FROM employee;
```


## 4. '+'号的作用
```
SELECT '10' + 20; # 结果为30
SELECT NULL + 12; # 结果为null
SELECT e_name + age AS info
FROM employee; # 结果为age
SELECT concat(e_name, age) AS info
FROM employee; # 字符串拼接
SELECT concat(`e_id`, ',', e_name, ',', age) AS out_put
FROM employee;
```


## 5. 显示表的结构(字段, 类型,...)
```
DESC employee;
```


## 6.----------- 模糊查询
### like
```
SELECT *
FROM employee
WHERE e_name LIKE '刘%'; # '%'表示任意个字符
SELECT *
FROM employee
WHERE e_name LIKE '%丽';
SELECT *
FROM employee
WHERE e_name LIKE '__'; # '_'代表一个字符, 转义加'\'或者escape关键字(escape 'c'，c作为转义字符)
SELECT *
FROM employee
WHERE e_name LIKE '___';
SELECT *
FROM employee
WHERE e_name LIKE '_%';
SELECT *
FROM employee
WHERE e_name LIKE '';
```


### between ... and ...
```
SELECT *
FROM employee
WHERE age BETWEEN 31 AND 40;
```


### at [31, 40]
### in
```
SELECT *
FROM employee
WHERE education IN ('本科', '硕士');
```


### 类似 or 的效果
### is null (不能用于判断 = null)
### 安全等于 '<=>' 可以不仅可以用于null值的判断

## 7. 排序查询 order by
```
SELECT *
FROM employee
ORDER BY age DESC; # 默认升序(ascending)--从小到大，降序(descending)--从大到小
SELECT purchase_price, retail_price, trade_price
FROM medicine
ORDER BY purchase_price, retail_price DESC;
```


### 多排序
## 8. 常用函数
### 字符型
```
SELECT concat(e_name, ',', age) AS info
FROM employee;
SELECT length('你好world'); # utf-8 汉字3个字节
SELECT upper('hello');
SELECT lower('World');
SELECT substr('tom jerry', 5, 2); # sql中索引从1开始, 第二个参数是start，第二个参数是length of substring
SELECT instr('this is a word.', 'is'); # 返回子字符串出现的第一个字符的索引，找不到则返回 0
SELECT trim('aa' FROM 'aaaaa你aaa好aaaa'); # 去掉字符串前后的某些字符
SELECT lpad('你好', 5, '#'); # 在左边填充字符，直到整个字符串的长度为给定数。类似有
SELECT rpad('你好', 5, '$');
SELECT replace('this is a apple.', 'apple', 'pear');
```


### 数学
```
SELECT round(1.56, 1); # 四舍五入, 默认为0
SELECT floor(1.6); # 向下取整：1
SELECT ceil(1.2); # 向上取整：2
SELECT truncate(1.2345, 2); # 截断
SELECT mod(12, 5);
SELECT 12 % 5;
```


### 日期
```
SELECT now(); # 日期+时间
SELECT curdate(); # 日期
SELECT curtime(); # 时间
SELECT year(curdate());
SELECT year('2011-1-1');
SELECT month(curdate());
SELECT monthname(curdate());
SELECT day('2020-2-12');
SELECT str_to_date('1-1-2020', '%m-%d-%y');
SELECT date_format('2018/1/1', '%y年%m月%d日');
```


### 其他
```
SELECT version();
SELECT database();
SELECT user();
```


### 流程控制
```
SELECT if(12 > 2, 'hello', 'world');
SELECT education,
       CASE education
           WHEN '高中' THEN 0
           WHEN '本科' THEN 1
           WHEN '博士' THEN 2
           ELSE education
           END AS new_edu
FROM employee;
```


## 9. 分组函数
```
SELECT sum(age)
FROM employee;
SELECT round(avg(age), 2)
FROM employee;
SELECT min(age)
FROM employee;
SELECT max(age)
FROM employee;
SELECT count(age)
FROM employee; # 统计不为null的值
SELECT max(age), min(age), round(avg(age), 2)
FROM employee;
SELECT count(education)
FROM employee;
SELECT count(DISTINCT education)
FROM employee;
SELECT count(1)
FROM employee;
SELECT e_id, avg(age)
FROM employee;
```


## 10. 分组查询
```
SELECT education, avg(age)
FROM employee
GROUP BY education;
SELECT education, count(*)
FROM employee
GROUP BY education;
SELECT education, count(*) AS counts
FROM employee
GROUP BY education
HAVING counts > 5;
```


### having 针对分组查询
```
SELECT education, max(age)
FROM employee
GROUP BY education;
```


### 查询每个学历中的最大年龄的编号
```
SELECT e_id, education, age
FROM employee
WHERE (education, age) IN (SELECT education, max(age)
                           FROM employee
                           GROUP BY education);
```


### 查询每个学历中的性别分布
```
SELECT education, sex, count(*) AS c
FROM employee
GROUP BY education, sex
ORDER BY c DESC;
```


## 11. 等值连接
#### 查询员工的销售数量
```
SELECT e_id, sum(sale_amount) AS amount_sum
FROM employee,
     sale_log
WHERE employee.e_id = sale_log.saler_id
GROUP BY e_id
HAVING amount_sum > 300;
```


## 12. 非等值连接
```
SELECT DISTINCT saler_id, lvl
FROM sale_log,
     sale_level
WHERE sale_amount BETWEEN min_amount AND max_amount;
```


## 13. 自连接
```
SELECT e.e_id, m.e_id
FROM employee e,
     employee m;
```


### 左边展开
## 14. 外连接
### 左,右外连接
```
SELECT e_id, saler_id, sale_amount
FROM employee AS e
         LEFT OUTER JOIN sale_log
                         ON e.e_id = sale_log.saler_id
WHERE sale_amount IS NULL;
```


### 全外连接
### 交叉连接
```
SELECT e.e_id, m.e_id
FROM employee e
         CROSS JOIN employee m;
```


## 15. 标量子查询
```
SELECT age
FROM employee
WHERE e_name = '李四';
```


### 查询比李四年龄大的员工
```
SELECT *
FROM employee
WHERE age > (
    SELECT age
    FROM employee
    WHERE e_name = '李四'
);
```


### 查询学历与李四相同，年龄比李四大的员工信息
```
SELECT *
FROM employee
WHERE education = (
    SELECT education
    FROM employee
    WHERE e_name = '李四'
)
  AND age > (
    SELECT age
    FROM employee
    WHERE e_name = '李四'
);
```


### 查询最低年龄大于高中学历员工的最低年龄的学历和最低年龄
```
SELECT education, min(age) AS min_age
FROM employee
GROUP BY education
HAVING min_age > (
    SELECT min(age)
    FROM employee
    WHERE education = '高中'
);
```


## 16. 列子查询
### 返回年龄在25或30的员工的姓名
```
SELECT e_name, age
FROM employee
WHERE age IN (25, 30);
```


## 17. select 后的子查询
```
SELECT education,
       (
           SELECT count(*)
           FROM employee e
           WHERE e.education = p.education
       )
FROM employee p
GROUP BY education;
```


## 18. from 后的子查询
```
SELECT *
FROM (
         SELECT education, avg(age) aa
         FROM employee
         GROUP BY education) AS avg_age
WHERE avg_age.aa > 30;
```


## 19. exists 后的子查询
```
SELECT exists(SELECT * FROM employee WHERE age > 70);
```


### 查询不存在销售记录的员工id
```
SELECT e_id
FROM employee
WHERE NOT exists(
        SELECT *
        FROM sale_log
        WHERE employee.e_id = sale_log.saler_id
    );
```


## 20. 分页查询 (page - 1)*size, size
```
USE world;
SELECT *
FROM city
LIMIT 2, 10;
```


### 索引从0开始
## 21. 联合查询(列数必须相同, 顺序最好不要不一致，会自动去重（除非加 ALL）)
```
USE pharmacy_ms;
```


### 查询学历为本科，或年龄大于30的员工
```
SELECT *
FROM employee
WHERE education = '本科'
UNION
SELECT *
FROM employee
WHERE age > 30;
```



# CRUD
```
CREATE DATABASE IF NOT EXISTS school;
USE school;
CREATE TABLE IF NOT EXISTS student
( -- 学生表
    s_id     char(3) NOT NULL,
    s_name   char(8) NOT NULL,
    sex      char(3) NOT NULL,
    age      tinyint,
    hometown varchar(20),
    PRIMARY KEY (s_id)
);
CREATE TABLE IF NOT EXISTS grades
(
    c_id  char(3) NOT NULL,
    s_id  char(3) NOT NULL,
    score tinyint,
    PRIMARY KEY (c_id, s_id)
);
```


## 1. 插入
```
INSERT INTO student(s_id, s_name, sex, age, hometown)
    VALUE ('001', '小红', '女', 15, 'He Bei');
INSERT INTO student(s_id, s_name, sex, age, hometown) VALUE
    ('002', '小刚', '男', 20, NULL);
INSERT INTO student (s_id, s_name, sex)
VALUES ('003', '小风', '男'); # 前后对应即可
INSERT INTO student
VALUES ('004', '李磊', '男', 22, 'Bei jing');
INSERT INTO student
SET s_id     = '005',
    s_name   = '张丽',
    sex      = '女',
    age      = 18,
    hometown = 'Tian Jing';
INSERT INTO grades (c_id, s_id, score)
VALUES ('101', '002', 67),
       ('102', '001', 77),
       ('101', '001', 88),
       ('105', '003', 66),
       ('103', '002', 54),
       ('102', '005', 92),
       ('103', '001', 83);
```


## 2. 修改
```
SELECT *
FROM student;
UPDATE student
SET age = 19
WHERE s_id = '005';
```


### 多个表修改使用连接
## 3. 删除
```
DELETE
FROM student
WHERE age IS NULL;
```


### 多表删除 delete b1, b2
```
SELECT *
FROM grades;
TRUNCATE TABLE grades; # 没有返回值，不能加where，效率高一点，不能回滚，删除后再添加，自增长列从1开始
DELETE
FROM grades;
```


### 有返回值，可以回滚，删除后再添加，自增长列从断点开始

## 1. 建库
```
CREATE DATABASE mtest;
CREATE DATABASE IF NOT EXISTS mtest;
```


## 2. 改库的字符集
```
ALTER DATABASE mtest CHARACTER SET gbk;
```


## 3. 删库
```
DROP DATABASE IF EXISTS mtest;
```


## 4. 建表
```
CREATE TABLE IF NOT EXISTS book
(
    id          int,
    bname       varchar(20),
    price       double,
    authorId    int,
    publishDate datetime
);
DROP TABLE IF EXISTS book;
DESC book;
CREATE TABLE IF NOT EXISTS author
(
    id      int,
    au_name varchar(20),
    nation  varchar(20)
);
DESC author;
```


## 5. 表的修改 (alter table)
```
DESC book;
DESC author;
```


### 修改列名
```
ALTER TABLE book
    CHANGE COLUMN publishDate pbDate datetime;
```


### 修改类型
```
ALTER TABLE book
    MODIFY COLUMN pbDate timestamp;
```


### 添加列
```
ALTER TABLE author
    ADD COLUMN annul_salary double;
```


### 删除列
```
ALTER TABLE author
    DROP COLUMN annul_salary;
```


### 修改表名
```
ALTER TABLE author RENAME TO book_author;
DESC book_author;
```


## 6. 表的删除
```
DROP TABLE IF EXISTS book_author;
SHOW TABLES;
##7. 表的复制
SELECT *
FROM student;
```


### 只复制结构
```
CREATE TABLE student_copy LIKE student;
```


### 复制部分结构
```
CREATE TABLE copy4
SELECT s_id, s_name
FROM student
WHERE FALSE;
```


### 完全复制数据
```
CREATE TABLE student_copy_full
SELECT *
FROM student;
```


### 复制部分数据（使用子查询）


# 约束
```
SHOW TABLES;
DROP TABLE IF EXISTS copy4;
USE school;
```


## 1. 列级约束
```
CREATE TABLE IF NOT EXISTS student
(                                                          -- 学生表  （从表）
    s_id    char(3) PRIMARY KEY,                           -- 主键约束
    s_name  char(8) NOT NULL UNIQUE,                       -- 非空且唯一
    gender  char(3) CHECK ( gender = '男' OR gender = '女'), -- 检查约束
    seat_id int UNIQUE,                                    -- 唯一约束
    age     int DEFAULT 18,                                -- 默认约束
    majorId int REFERENCES major (id)                      -- 外键引用约束
);
DROP TABLE IF EXISTS school.student;

CREATE TABLE IF NOT EXISTS major
( -- 专业表   （主表）
    id         int PRIMARY KEY,
    major_name varchar(20)
);
DESC school.student;
```


#### 查看student中的索引
```
SHOW INDEX FROM school.student;
```


## 2. 表级约束
```
CREATE TABLE IF NOT EXISTS student
(
    id      int,
    name    varchar(20),
    gender  char(3),
    seat    int,
    age     int,
    majorId int,

​    CONSTRAINT pk PRIMARY KEY (id),                                         -- 主键
​    CONSTRAINT uk UNIQUE (seat),                                            -- 唯一约束
​    CONSTRAINT ck CHECK ( gender = '男' OR gender = '女'),                    -- 检查
​    CONSTRAINT fk_student_major FOREIGN KEY (majorId) REFERENCES major (id) -- 外键
);
DESC school.student;
INSERT INTO school.student (id, name, gender, seat, age, majorId)
VALUES (1, 'bob', '男', 12, 12, 15); # 后
INSERT INTO major (id, major_name)
VALUES (15, 'math');
```


### 先
/*
  主键与唯一：
  1. 主键不能为空，唯一可以为空
  2. 都保证唯一性
  3. 主键最多一个，唯一可以有多个
  4. 都可以组合

  外键：有外键的表为从表，否则是主表
  1. 从表添加记录时的外键或者为空，或者必须是从表的某个主键值

  2. 添加数据时，先插入主表，再插入从表。删除数据时，先删除从表，再删除主表
  */

  ```
  DELETE
  FROM school.student
  WHERE id = 1; # 先
  DELETE
  FROM major
  WHERE id = 15;
  ```

  

#### 后
## 3. 修改约束
```
ALTER TABLE school.student
    MODIFY COLUMN id int CHECK ( id < 100 ); # 列级约束
ALTER TABLE school.student
    ADD CONSTRAINT pk PRIMARY KEY (id); # 表级约束
ALTER TABLE school.student
    ADD CONSTRAINT fk_student_major FOREIGN KEY (majorId) REFERENCES major (id);
DESC school.student;
SHOW INDEX FROM school.student;
```


### 删除约束，不加约束
```
ALTER TABLE school.student
    DROP CONSTRAINT fk_student_major;
```


## 4. 标识列
/*
 1. 标识列必须是唯一的

 2. 标识列最多一个

 3. 标识列只能是数值类型

 4. show variables like '%auto_increment%'
 */

```
    CREATE TABLE IF NOT EXISTS people
    (
id  int PRIMARY KEY AUTO_INCREMENT, # 自增列
nam varchar(20)
    );
INSERT INTO people(nam)
VALUE ('bob');
SELECT *
 FROM people;
 SHOW VARIABLES LIKE '%auto_increment%';
 ```
 
 
 

# 事务
```
SHOW ENGINES;
SHOW VARIABLES LIKE '%autocommit%';
```


## 1. 事务
### 隐式事务：单条语句就是事务autocommit
### 显示事务: (先禁用自动提交)  set autocommit = 0;
```
SET AUTOCOMMIT = 0; # 默认开启事务
START TRANSACTION;
```


### 可选
### ... ... 编写crud语句
```
COMMIT; # 提交
ROLLBACK; # 回滚
USE mtest;
CREATE TABLE IF NOT EXISTS account
(
    id      int PRIMARY KEY,
    name    char(12),
    balance double
);
INSERT INTO account
VALUES (1, '张三', 1000),
       (2, '李四', 1000);
SELECT *
FROM account;
```


### 事务演示
```
SET AUTOCOMMIT = 0;
START TRANSACTION;
UPDATE account
SET balance = balance + 500
WHERE name = '张三';
UPDATE account
SET balance = balance - 500
WHERE name = '李四';
```

```
COMMIT ;

ROLLBACK;
```


## 2. 隔离级别(默认为可重复读)
```
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
```


### 未提交读：read uncommitted
```
SHOW VARIABLES LIKE '%isolation%';
SELECT *
FROM account;
```


### 出现脏读，读取了未提交事务的数据
```
SET AUTOCOMMIT = 0;
START TRANSACTION;
UPDATE account
SET balance = 1111
WHERE name = '张三';
ROLLBACK;

SET AUTOCOMMIT = 0;
START TRANSACTION;
SELECT *
FROM account;
```


### 发生脏读
### 提交读：read committed
```
SET AUTOCOMMIT = 0;
START TRANSACTION;
UPDATE account
SET balance = 1111
WHERE name = '张三';
COMMIT;

SET AUTOCOMMIT = 0;
START TRANSACTION;
SELECT *
FROM account; #  commit 前读取
SELECT *
FROM account;
```



### 两次读取数据不一致

### 可重复读：repeatable read
```
SET AUTOCOMMIT = 0;
START TRANSACTION;
UPDATE account
SET balance = 1111
WHERE name = '张三';
COMMIT;



###--------- 可重复读
SET AUTOCOMMIT = 0;
START TRANSACTION;
UPDATE account
SET balance = 1111
WHERE name = '张三';
COMMIT;

SET AUTOCOMMIT = 0;
START TRANSACTION;
SELECT *
FROM account; #  commit 前读取
SELECT *
FROM account;
```



### 两次读取数据一致

### 可串行化：serializable
```
###---------- 幻读（多了行或少了行）
SET AUTOCOMMIT = 0;
START TRANSACTION;
UPDATE account
SET name = 'wwwww'; # 插入前执行
UPDATE account
SET name = 'qqqqq'; # 插入后执行，影响的行数不一致
COMMIT;

SET AUTOCOMMIT = 0;
START TRANSACTION;
INSERT INTO account (id, name, balance) # 若是串行，该事务会被锁住
VALUES (3, 'new', 1000);
```



## 3. 回滚点
```
SELECT *
FROM account;
SET AUTOCOMMIT = 0;
START TRANSACTION;
DELETE
FROM account
WHERE id = 1;
SAVEPOINT pt;
DELETE
FROM account
WHERE id = 2;
ROLLBACK TO pt;
ROLLBACK;
```


### delete与truncate的事务区别
### delete可以回滚，truncate不能回滚


# 视图：虚拟表, 无法修改，只有逻辑，不存储真实的数据
```
USE pharmacy_ms;
SELECT *
FROM employee;
CREATE VIEW v2
AS
(
SELECT education, avg(age)
FROM employee
GROUP BY education
    );
SELECT * FROM v2;
```


## 不可修改
```
UPDATE v2
SET `avg(age)` = 10
WHERE education = '本科';
```


## 修改视图
```
CREATE OR REPLACE VIEW v2
as
    select * FROM employee;

ALTER VIEW pharmacy_ms.v2
AS SELECT * FROM employee;
```


## 删除视图
```
DROP VIEW IF EXISTS pharmacy_ms.v2, pharmacy_ms.v1;
```


## 查看视图结构
```
DESC pharmacy_ms.v2;
SHOW CREATE VIEW pharmacy_ms.v2;
```


## 视图的更新
```
SELECT * FROM pharmacy_ms.v2;
INSERT INTO pharmacy_ms.v2 (education, `avg(age)`)
VALUES ('小学', 12);  # 报错，不支持。最简单的视图可以用，同时也修改表
```




# 变量
/*
 系统  全局变量

​          会话变量

自定义   用户变量
               局部变量
 */

## 1. 查看所有系统变量
```
SHOW GLOBAL VARIABLES ;  # 系统级别
SHOW SESSION VARIABLES ; # 会话级别
```


## 2. 查看部分变量
```
SHOW GLOBAL VARIABLES LIKE '%char%';
SHOW VARIABLES LIKE '%isolation%';
```


## 3. 查看指定变量
```
SELECT @@global.autocommit;
SELECT @@session.autocommit;
```


## 4. 设置系统变量
```
SET GLOBAL AUTOCOMMIT = 1;
SET @@global.autocommit = 1;
```



## 5. 自定义变量
### 初始化
```
SET @my_v = 1;
SET @my_v1 := 2;
SELECT @my_v2 := 3; # 不支持 =
SET @my_count = 1;
SHOW VARIABLES LIKE '%my%';
```


### 查询赋值
```
SELECT count(*) INTO @my_count
FROM employee;
SELECT @my_count;
```


## 6. 局部变量 begin ... end 中
## 7. 存储过程
/*
 语法：
 1. 参数：<参数模式 参数名 参数类型>
     例：    in     name  char(20)
    参数模式：in -- 该参数可以作为输入，可以作为传入值
             out -- 改参数可以作为输出，可以作为返回值
             inout -- 该参数既可以作为输入也可以作为输出
    
 2. 一句话的存储过程：begin ... end 可以省略

 3. 存储过程的结尾可以使用delimiter设置

 4. 中的sql语句都要加分号
 */

```
    CREATE PROCEDURE my_proc()
BEGIN
 
 存储过程体
 
 END;
 ```
 
 
 
### 调用
```
CALL my_proc();
USE school;
SELECT * FROM school.student;
USE mtest;
```



### 空参
```
CREATE PROCEDURE myp1()
BEGIN
    INSERT INTO account(id, name, balance)
    VALUES (1, 'bob', 1112);
END ;
CALL myp1();
DELETE FROM account
WHERE id = 1;
SELECT * FROM account;
SHOW CREATE TABLE school.student;
SET AUTOCOMMIT = 1;
```


### in 模式
```
USE pharmacy_ms;
```


### 根据姓名查询销总销售量
```mysql
CREATE PROCEDURE query_amount(IN ename char(8))
BEGIN
    DECLARE res varchar(20) DEFAULT '';

​    SELECT sum(sale_amount) INTO res
​    FROM sale_log RIGHT JOIN employee
​    ON e_id = saler_id
​    WHERE e_name = ename;

​    SELECT if(res > 0, '成功', '失败');
END;
CALL query_amount('李四');
DROP PROCEDURE query_amount;
```


### out 模式
### 根据名字返回其学历
```
CREATE PROCEDURE get_edu(IN ename char(20), OUT edu char(10))
BEGIN
    SELECT education INTO edu
    FROM employee
        WHERE e_name = ename;
END;
SET @his_edu = '';  # 自定义变量
CALL get_edu('李华', @his_edu);
SELECT @his_edu;
```


### inout 模式
```
CREATE PROCEDURE cal(INOUT a int, INOUT b int)  # 返回两个值
BEGIN
    SET a = a * 2;  # 局部变量不加@
    SET b = b * 2;
END;
SET @aa = 1;
SET @bb = 12;
CALL cal(@aa, @bb);
SELECT @aa, @bb;
```


### 删除存储过程
```
DROP PROCEDURE cal;
```


### 查看存储过程
```
SHOW CREATE PROCEDURE get_edu;
```



# 函数
/*
 存储过程：增删改查，批量处理
 函数：只能返回一个值，适合做处理数据
 CREATE FUNCTION 函数名(参数列表[参数名 参数类型...]) RETURNS 返回类型
 CALL 函数名(参数列表)
 */
### 开启
```
SET GLOBAL log_bin_trust_function_creators = 1;
```


### 无参，返回员工数
```
CREATE FUNCTION func() RETURNS int
BEGIN
    DECLARE res int DEFAULT 0;
    SELECT count(*) INTO res
    FROM employee;
    RETURN res;
END;
SELECT func();
```


### 有参, 查询年龄
```
CREATE FUNCTION func2(eid char(4)) RETURNS int
BEGIN
    SET @ag = 0;
    SELECT age INTO @ag
    FROM employee
    WHERE e_id = eid;
    RETURN @ag;
END;
SELECT func2('0002');
```


### 删除函数
```
DROP FUNCTION func2;
```


### 查看函数
```
SHOW CREATE FUNCTION func;
```


## 分支结构
```
CREATE PROCEDURE case_demo(IN score int)
BEGIN
    CASE
    WHEN score < 60 THEN SELECT 'F';
    when score >= 60 THEN SELECT 'D';
    end CASE ;
END;
CALL case_demo(99);

CREATE FUNCTION if_demo(score int) RETURNS char
BEGIN
    IF score < 60 THEN RETURN 'F';
    ELSE RETURN 'D';
    end IF ;
END;
SELECT if_demo(99);
```


## 循环结构
```
/*
 while, loop, repeat
 iterate => continue
 leave => break
 */
USE mtest;
```


### 批量添加数据
```
CREATE PROCEDURE pro_while(IN ct int)
BEGIN
    DECLARE i int DEFAULT 1;
    WHILE i <= ct do
        INSERT INTO account(id, name, balance)
        VALUES (i, concat('rose', i), i * 50);
        SET i = i + 1;
        END WHILE;
END;
DROP PROCEDURE pro_while;
CALL pro_while(10);
```


## leave
```
CREATE PROCEDURE leave_loop(IN ct int)
BEGIN
    DECLARE i int DEFAULT 1;
    a: WHILE i < ct do
        INSERT INTO account(id, name, balance)
        VALUES (i, concat('rose', i), i * 50);
        IF i > 5 THEN LEAVE a;
        end IF ;
        SET i = i + 1;
        END WHILE a;
END;
TRUNCATE TABLE account;
CALL leave_loop(10);
SELECT floor(rand()*3 + 1);
```
