# MySQL

## 简介

* SQL是用于访问和处理数据库的标准计算机语言
* `SQL`, Structured Query Language
* `RDBMS`, Relational Database Managerment System, 关系型数据库管理系统
* 数据存储在被称为`表`的数据库对象中
* `表`是相关的数据项的集合, 由列和行组成

RDBMS术语:
* `数据库`: 一些关联表的集合
* `数据表`: 数据的矩阵
* `列`: 列包含相同的数据
* `行`: 行是一组相关的数据, 也叫记录
* `主键`: 主键是唯一的. 一个数据表只能包含一个主键.
* `外键`: 用于关联两个表
* `复合键`: 将多个列作为一个索引键, 用于复合索引
* `索引`: 对数据表中一列或多列的值进行排序的一种结构, 用于快速访问表中特定信息
* `参照完整性`: 要求关系中不允许引用不存在的实体


## MySQL

MySQL数据库:
* 开源免费
* 支持5000万条记录, 32位系统的表文件最大支持4GB, 64位系统的表文件最大支持8TB

### 安装

* 官网[https://dev.mysql.com/downloads/](https://dev.mysql.com/downloads/)
* Homebrew安装: `brew install mysql`

### 服务端管理

* 查看MySQL服务端程序版本

```shell
mysqladmin --version
```

* 设置用户和密码
默认root的密码为空, 为了安全, 要设置密码. 输入密码时不会显示, 输入即可

```shell
$ mysqladmin -u root password "new_password"
```

* 程序启动停止

```shell
# 查询mysql服务端是否启动
ps -ef | grep mysqld

# 启动
mysqld start

# 重启
mysqld restart

# 停止
mysqld stop
mysqladmin -u root -p shutdown
```

* 用户管理
修编辑`mysql`数据库中的`user`表
    - 使用`PASSWORD()`对密码加密
    - `FLUSH PRIVILEGES`重新载入授权表, 如果不执行, 则只能重启mysql服务器
    - 对用户指定权限, 允许的插入`'Y'`值

```sql
mysql> use mysql;

mysql> INSERT INTO user
       (host, user, authentication_string, select_priv, insert_priv, update_priv)
       VALUES ('localhost', 'guest', PASSWORD('guest123'), 'Y', 'Y', 'Y');

mysql> FLUSH PRIVILEGES;

mysql> SELECT host, user, authentication_string FROM user WHERE user = 'guest';
```


* `/etc/my.cnf`配置文件
一般不需要改动
    - 配置错误日志存放目录

### 客户端使用

* 客户端连接

```shell
# 客户端连接数据库
$ mysql -u root -p
Enter password:
```

* 客户端断开连接

```sql
-- 断开连接
mysql> exit
```


## 数据类型

大致可分为三类: 数值, 日期/时间, 字符串

### 数值类型

|类型         |大小 |范围(有符号)                                                                   |范围(无符号)                             |用途        |
|------------|----|------------------------------------------------------------------------------|---------------------------------------|------------|
|TINYINT     |1字节|[-128, 127]                                                                   |[0,255]                                |小整数值     |
|SMALLINT    |2字节|[-32768, 32767]                                                               |[0, 65535]                             |大整数值     |
|MEDIUMINT   |3字符|[-8388608, 8388607]                                                           |[0, 16777215]                          |大整数值     |
|INT或INTEGER|4字节|[-2147483648, 2147483647]                                                     |[0, 4294967295]                        |大整数值     |
|BIGINT      |8字节|[-9233372036854775808, 9223372036854775807]                                   |[0, 18446744073709551615]              |极大整数值   |
|FLOAT       |4字节|[-3.402823466E+38, -1.175494351E-38], 0, [1.175494351E-38, 3.402823466351E+38]|[0, [1.175494351E-38, 3.402823466E+38]]|单精度浮点数值|
|DOUBLE      |8字节|[-1.7976931348623157E+308, -2.2250738585072014E-308], 0, [2.2250738585072014E-308, 1.7976931348623157E+308]|0, [2.2250738585072014E-308, 1.7976931348623157E+308]|双精度浮点数值|
|DECIMAL     |对于DECIMAL(M, D), 如果M>D, 是M+2; 否则是D+2|依赖于M和D的值|依赖于M和D的值|小数值|

### 日期和时间类型

当时间不合法时, 值为零值

|类型      |大小(字节)|范围                                    |格式               |用途                 |
|---------|---------|---------------------------------------|-------------------|--------------------|
|DATE     |3        |1000-01-01/9999-12-31                  |YYYY-MM-DD         |日期值               |
|TIME     |3        |'-838:59:59'/'838:59:59'               |HH:MM:SS           |时间值或持续时间       |
|YEAR     |1        |1901/2155                              |YYYY               |年份值               |
|DATETIME |8        |1000-01-01 00:00:00/9999-12-31 23:59:59|YYYY-MM-DD HH:MM:SS|混合日期和时间值       |
|TIMESTAMP|4        |1970-01-01 00:00:00/2037 年某时         |YYYYMMDD HHMMSS    |混合日期和时间值, 时间戳|

### 字符串类型

|类型       |大小(字节)   |用途                     |
|----------|------------|------------------------|
|CHAR      |0-255       |定长字符串                |
|VARCHAR   |0-65535     |变长字符串                |
|TINYBLOB  |0-255       |不超过255个字符的二进制字符串|
|TINYTEXT  |0-255       |短文本字符串               |
|BLOB      |0-65535     |二进制形式的长文本数据      |
|TEXT      |0-65535     |长文本数据                |
|MEDIUMBLOG|0-16777215  |二进制形式的中等长度文本数据 |
|MEDIUMTEXT|0-16777215  |中等长度文本数据           |
|LONGBLOB  |0-4294967295|二进制形式的极大文本数据    |
|LONGTEXT  |0-4294967295|极大文本数据              |

## 语法

* SQL对大小写不敏感
* 语句末端要加上`;`分号
* 文本使用`''`单引号或`""`双引号包裹
* 数字不需要包裹


### 库操作

* 列出数据库

```sql
SHOW databases;
```

* 创建数据库

```sql
CREATE DATABASE db_name;
```

* 使用数据库

```sql
USE db_name;
```

* 查看当前所使用的数据库

```sql
SELECT DATABASE();
```

* 删除数据库

```sql
DROP DATABASE db_name;
```


### 表操作

* 列出表

```sql
SHOW tables;
```

* 显示表结构

```sql
SHOW COLUMN FROM table_name;
-- 或
DESC table_name;
```

* 显示建表语句

```sql
SHOW CREATE TABLE table_name;
```

* 显示表的索引信息, 包括主键

```sql
SHOW INDEX FROM table_name;
```

* 显示性能及统计信息

```sql
-- 显示库中的所有表信息
SHOW TABLE STATUS FROM db_name;

-- 显示表名以runoob开头的表的信息
SHOW TABLE STATUS FROM db_name LIKE 'runoob%';
```

#### CREATE TABLE 创建表

创建表并定义字段

```sql
CREATE TABLE table_name (
    column_name column_type,
    column_name column_type
);
```

```sql
CREATE TABLE Book (
    id INT NOT NULL AUTO_INCREMENT,
    title VARCHAR(100) NOT NULL,
    author VARCHAR(40) NOT NULL,
    date DATE,
    PRIMARY KEY (id)
);
-- 或
CREATE TABLE Book (
    id INT PRIMARY KEY NOT NULL AUTO_INCREMENT,
    title VARCHAR(100) NOT NULL,
    author VARCHAR(40) NOT NULL,
    date DATE
);
```

#### DROP TABLE 删除表

删除表

```sql
DROP TABLE table_name;
```

```sql
DROP TABLE Book;
```

#### ALTER TABLE 修改表

```sql
-- 修改表名
ALTER TABLE table_name RENAME TO new_table_name;

-- 增加表字段
ALTER TABLE table_name ADD column_name type constraint;

-- 删除表字段
ALTER TABLE table_name DROP column_name;

-- 修改表字段: 修改原有
ALTER TABLE table_name MODIFY column_name type;

-- 修改表字段: 设置新的
ALTER TABLE table_name CHANGE column_name type constraint;

-- 增加约束
ALTER TABLE table_name
ADD constraint (column_name);

-- 删除约束
ALTER TABLE table_name
DROP constraint column_name;
```

### 约束

规定表中的数据的规则
如果数据违反约束, 则行为会被终止

#### NOT NULL 非空约束

强制列不接受NULL值, 即始终要包含值

```sql
CREATE TABLE Persons (
    id INT NOT NULL,
    name VARCHAR(25) NOT NULL,
);
```

#### UNIQUE 唯一约束

唯一标识表中的记录

```sql
UNIQUE(column_name)
```

```sql
CREATE TABLE Person (
    id INT NOT NULL,
    name VARCHAR(25) NOT NULL,
    UNIQUE (id)
);
```

#### PRIMARY KEY 主键约束

* 设置为主键
    - 主键必须包含唯一的值(UNIQUE)
    - 主键不能包含NULL值(NOT NULL)
    - 每个表有且只有一个主键

```sql
PRIMARY KEY (column_name)
```

```sql
CREATE TABLE Persons (
    id INT NOT NULL,
    name VARCHAR(25) NOT NULL,
    PRIMARY KEY (id)
);
```

#### FOREIGN KEY 外键约束

一个表的外键指向另一个表中的主键
* 作用
    - 用于预防破坏表之间的连接行为, 保持数据的完整性
    - 防止非法数据插入外键列 (因为该值必须是它指向的表中的那个值之一)
* 级联引用完整性约束选项(属于REFERENCES子句)
    - `ON DELETE NO ACTION`: 指定如果试图删除某一行, 而该行的键被其他表的现有行中的外键所引用, 则产生错误并回滚 DELETE 语句
    - `ON UPDATE NO ACTION`: 指定如果试图更新某一行中的键值, 而该行的键被其他表的现有行中的外键所引用, 则产生错误并回滚 UPDATE 语句
    - `ON DELETE CASCADE`: 指定如果试图删除某一行, 而该行的键被其他表的现有行中的外键所引用, 则也将删除所有包含那些外键的行
    - `ON UPDATE CASCADE`: 指定如果试图更新某一行中的键值, 而该行的键值被其他表的现有行中的外键所引用, 则组成外键的所有值也将更新到为该键指定的新值
    - `ON DELETE SET NULL`: 指定如果试图删除某一行, 而该行的键被其他表的现有行中的外键所引用, 则组成被引用行中的外键的所有值将被设置为 NULL. 为了执行此约束, 目标表的所有外键列必须可为空值
    - `ON UPDATE SET NULL`: 指定如果试图更新某一行, 而该行的键被其他表的现有行中的外键所引用, 则组成被引用行中的外键的所有值将被设置为 NULL. 为了执行此约束, 目标表的所有外键列必须可为空值
    - `ON DELETE SET DEFAULT`: 指定如果试图删除某一行, 而该行的键被其他表的现有行中的外键所引用, 则组成被引用行中的外键的所有值将被设置为它们的默认值. 为了执行此约束, 目标表的所有外键列必须具有默认定义. 如果某个列可为空值, 并且未设置显式的默认值, 则将使用 NULL 作为该列的隐式默认值. 因 ON DELETE SET DEFAULT 而设置的任何非空值在主表中必须有对应的值, 才能维护外键约束的有效性
    - `ON UPDATE SET DEFAULT`: 指定如果试图更新某一行, 而该行的键被其他表的现有行中的外键所引用, 则组成被引用行中的外键的所有值将被设置为它们的默认值. 为了执行此约束, 目标表的所有外键列必须具有默认定义. 如果某个列可为空值, 并且未设置显式的默认值, 则将使用 NULL 作为该列的隐式默认值. 因 ON UPDATE SET DEFAULT 而设置的任何非空值在主表中必须有对应的值, 才能维护外键约束的有效性

```sql
FOREIGN_KEY (column_name) REFERENCES table2 (column_name) [ON DELETE|UPDATE ...]
```

```sql
CREATE TABLE Orders (
    id INT NOT NULL,
    name VARCHAR(25) NOT NULL,
    FOREIGN KEY (P_id) REFERENCES Persons (id)
);
```

#### CHECK 检查约束

限制列中值的范围
* 对列约束, 则该列只允许特定的值
* 对表约束, 则此约束会基于行中其他列的值在特定的列中对值进行限制

```sql
CHECK (column_name condition)
```

```sql
CREATE TABLE Persons (
    id INT NOT NULL,
    name VARCHAR(25) NOT NULL;
    CHECK (id > 0)
);
```

#### DEFAULT 默认值约束

向列中插入默认值

```sql
DEFAULT value
```

```sql
CREATE TABLE Person (
    id INT NOT NULL,
    name VARCHAR(25) DEFAULT 'haha'
);
```

#### AUTO_INCREMENT 自增

在插入新记录时, 自动创建主键字段的值, 并且会自动增加

```sql
CREATE TABLE Persons (
    id INT NOT NULL AUTO_INCREMENT,
    PRIMARY KEY (id)
);
```

### 记录操作

#### INSERT INTO 增加语句

向表中插入记录

```sql
-- 不用指定列, 给出所有列的值
INSERT INTO table_name
VALUES (value1, value2, ...);

-- 给指定列插入指定值
INSERT INTO table_name (column_name1, column_name2, ...)
VALUES (value1, value2, ...);
```

```sql
INSERT INTO Websites
VALUES ('百度', 'https://www.baidu.com', 4, 'CN');

INSERT INTO Websites (name, url, alexa)
VALUES ('百度', 'https://www.baidu.com', 4);
```

#### DELETE FROM 删除语句

删除表中的记录

```sql
DELETE FROM table_name
WHERE condition;
```

```sql
-- 删除name为百度且country为CN的记录
DELETE FROM Websites
WHERE name='百度' AND country='CN';

-- 删除所有记录
DELETE FROM table_name;

-- 也是删除所有记录
DELETE * FROM table_name;
```

#### UPDATE 修改语句

更新表中的记录

```sql
UPDATE table_name
SET column1=value1, column2=value2, ...
[WHERE condition];
```

```sql
-- 只更新name为哈哈的记录
UPDATE Websites
SET alexa=5000, country='CN'
WHERE name='哈哈';

-- 更新所有记录
UPDATE Websites
SET alexa=5000, country='CN';
```

#### SELECT 查询语句

选取记录

```sql
-- 选择指定列
SELECT column_name, column_name ...
FROM table_name;

-- 选择所有列
SELECT * FROM table_name;
```

#### SELECT DISTINCT 去重查询

选取不重复的记录

```sql
SELECT DISTINCT column_name, column_name ...
FROM table_name;
```

#### AS 别名

为表或列指定别名. 可以增强可读性
下列情况使用别名很有用:
* 查询中涉及超过一个表
* 查询中使用了函数
* 列名称很长或可读性差
* 需要把两个或两个以上的列结合在一起

```sql
SELECT column_name AS alias_name
FROM table_name;

SELECT column_name, ...
FROM table_name AS alias_name;
```

```sql
-- 将url, alexa, country拼接在一起, 设置别名site_info
SELECT name, CONCAT(url, ', ', alexa, ', ', country) AS site_info FROM Websites;

-- 表的别名
SELECT w.name, w.url, a.count, a.date FROM Websites AS w, access_log AS a WHERE a.site_id=w.id AND w.name='haha';
```

#### WHERE 条件子句

过滤记录, 提取满足指定标准的记录

```sql
SELECT column_name, column_name ...
FROM table_name
WHERE column_name operator value;
```

运算符:
* `=`: 等于
* `<>`或`!=`: 不等于
* `>`: 大于
* `<`: 小于
* `>=`: 大于等于
* `<=`: 小于等于
* `BETWEEN`: 在某个范围内
* `LIKE`: 匹配某种模式
* `IN`: 在一个列表内
* `AND`: 两个条件都成立
* `OR`: 两个中至少一个成立

```sql
SELECT * FROM Websites WHERE country='CN';

SELECT * FROM Websites WHERE id=1;

SELECT * FROM Websites WHERE country='CN' AND alexa>50;

SELECT * FROM Websites WHERE country='USA' OR country='CN';

SELECT * FROM Websites WHERE alexa>15 AND (country='USA' OR country='CN');

UPDATE Websites SET alexa=0 WHERE country='CN';

DELETE * FROM Websites WHERE alexa=0;
```

#### LIKE 模式匹配子句

对结果集匹配指定的模式
* 通配符
默认不区分大小写
    - `%`: 匹配0个或多个字符
        - 匹配以`G`开头的所有字符: `'G%'`
        - 匹配以`Z`结尾的所有字符: `'%Z'`
        - 匹配包含`oo`的所有字符: `'%oo%'`
    - `_`: 匹配一个字符
        - 匹配`o什么o`: `'o_o'`

```sql
SELECT column_name, ...
FROM table_name
WHERE column_name, ... LIKE pattern;
```

```sql
-- 匹配以G开头的
SELECT * FROM Websites WHERE name LIKE 'G%';

-- 匹配以oogle结尾的
SELECT * FROM Websites WHERE name LIKE '_oogle';
```

#### REGEXP 和 NOT REGEXP 正则匹配

使用正则匹配过滤结果
* `REGEXP 正则表达式`: 返回匹配正则表达式的结果
* `NOT REGEXP 正则表达式`: 返回不匹配正则表达式的结果
    - `[字符列表]`: 匹配字符列表中的任何单一字符
        - 包含`aeiou`中的任意一个字符: `'[aeiou]'`
        - 包含从A到H的: `'[A-H]'`
    - `[^字符列表]`, `[!字符列表]`: 匹配不在字符列表中的任何单一字符
        - 不包含`aeiou`的任意一个字符: `[^aeiou]`, `'[!aeiou]'`
    - `^`: 以此开头
        - `^[abc]`: 以a或b或c开头的

```sql
SELECT column_name, ...
FROM table_name
WHERE column_name, ... REGEXP regex;

SELECT column_name, ...
FROM table_name
WHERE column_name, ... NOT REGEXP regex;
```

```sql
-- 匹配以G, F, 或s开头的所有结果
SELECT * FROM Websites WHERE name REGEXP '^[GFs]';

-- 匹配不以G, F, s开头的所有结果
SELECT * FROM Websites WHERE name REGEXP '^[^GFs]';
```

#### IN 列表子句

限制结果集在列表子句中

```sql
SELECT column_name, ...
FROM table_name
WHERE column_name, ... IN (value1, value2, ...);
```

```sql
SELECT * FROM Websites WHERE name IN ('Google', 'Baidu');
```

#### BETWEEN AND 范围子句

限制结果集在范围之内. 是否包含边界值, 取决于数据库的实现

```sql
SELECT column_name, ...
FROM table_name
WHERE column_name, ... BETWEEN value1 AND value2;
```

```sql
-- 介于1到20的alexa排名的网站
SELECT * FROM Websites WHERE alexa BETWEEN 1 AND 20;

-- 不是A道H的名字
SELECT * FROM Websites WHERE name NOT BETWEEN 'A' AND 'H';
```

#### OFFSET 偏移子句

指定查询偏移量, 默认为0

```sql
SELECT column_name, column_name, ...
FROM table_name
OFFSET m;
```

```sql
SELECT * FROM Websites
OFFSET 2;
```

#### LIMIT 结果数量限制子句

限制返回的记录数量

```sql
SELECT column_name, ...
FROM table_name
LIMIT n;
```

```sql
SELECT * FROM Websites
LIMIT 10;
```

#### ORDER BY 排序子句

对结果集进行排序

```sql
SELECT column_name, column_name ...
FROM table_name
ORDER BY column_name, column_name ... ASC|DESC;
```

顺序:
* `ASC`: 递增(默认)
* `DESC`: 递减

```sql
SELECT * FROM Websites ORDER BY alexa;

SELECT * FROM Websites ORDER BY alexa DESC;

SELECT * FROM Websites ORDER BY country, alexa;
```

#### GROUP BY 分组子句

结合聚合函数, 对一个或多格列对结果集进行分组

```sql
SELECT column_name, ..., 聚合函数(column_name)
FROM table_name
WHERE condition
GROUP BY column_name;
```

```sql
--
SELECT name, COUNT(*) FROM Websites GROUP BY country;
```

#### HAVING 聚合条件

通过HAVING与聚合函数一起使用, 过滤结果集

```sql
SELECT column_name, FUNCTION(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name
HAVING FUNCTION(column_name) operator value;
```

```sql
-- 总访问量大于200的网站
SELECT Websites.name, Websites.url, SUM(access_log.count) AS nums
FROM (access_log INNER JOIN Websites ON access_log.site_id=Websites.id)
GROUP BY Websites.name
HAVING SUM(access_log.count) > 200;
```


### 多表查询

#### 连接

##### 交叉连接

交叉连接查询的结果是两个表的笛卡尔积

```sql
-- 示例
SELECT * FROM a, b;
```

##### INNER JOIN 内连接子句

把来自两个或两个以上的表的记录结合起来, 基于表之间的共同字段
如果两表中都匹配, 则返回记录
* `INNER JOIN`也可写为`JOIN`

```
+---+--+---+
|   ||||   |
+---+--+---+
```

```sql
SELECT column_name, ...
FROM table_name1
[INNER] JOIN table_name2
ON table1.column_name1=table2.column_name2;
```

```sql
SELECT Websites.name, access_log.count, access_log.date FROM Websites INNER JOIN access_log ON Websites.id=access_log.site_id ORDER BY access_log.count;
```

##### LEFT JOIN 左连接子句

把来自两个或两个以上的表的记录结合起来, 基于表之间的共同字段
即使右表中没有匹配, 也从左表返回所有的记录, 右表没有匹配的记录值为NULL
* `LEFT JOIN`也可写为`LEFT OUTER JOIN`

```
+---+--+---+
||||||||   |
+---+--+---+
```

```sql
SELECT column_name, ...
FROM table_name1
LEFT [OUTER] JOIN table_name2
ON table_name1.column_name1=table_name2.column_name2;
```

```sql
SELECT Websites.name, access_log.count, access_log.date FROM Websites LEFT JOIN access_log ON Websites.id=access_log.site_id ORDER BY access_log.count DESC;
```

##### RIGHT JOIN 右连接子句

把来自两个或两个以上的表的记录结合起来, 基于表之间的共同字段
即使左表中没有匹配, 也从右表中返回所有的记录. 左表没有匹配的记录值为NULL
* `RIGHT JOIN`也可写为`RIGHT OUTER JOIN`

```
+---+--+---+
|   ||||||||
+---+--+---+
```

```sql
SELECT column_name, ...
FROM table_name1
RIGHT [OUTER] JOIN table_name2
ON table_name1.column_name1=table_name2.column_name2;
```

```sql
SELECT Websites.name, access_log.count, access_log.date FROM Websites RIGHT JOIN access_log ON Websites.id=access_log.site_id ORDER BY access_log.count DESC;
```

##### FULL JOIN 全连接子句

把来自两个或两个以上的表的记录结合起来, 基于表之间的共同字段
只要其中一个表中存在匹配, 则返回记录
* `FULL JOIN`也可写为`FULL OUTER JOIN`

```
+---+--+---+
||||||||||||
+---+--+---+
```

```sql
SELECT column_name, ...
FROM table_name1
FULL [OUTER] JOIN table_name2
ON table_name1.column_name1=table_name2.column_name2;
```

```sql
SELECT Websites.name, access_log.count, access_log.date FROM Websites FULL JOIN access_log ON Websites.id=access_log.site_id ORDER BY access_log.count DESC;
```

#### 子查询

* 一个查询的结果可以作为另一个查询的表或条件
* 子查询格式: 使用小括号`()`将子查询语句包裹起来
* 子查询注意事项:
    - 如果将子查询的结果作为FROM后的表, 则子查询结果必须起别名
    - 如果将子查询的结果作为WHERE的条件, 则可以不起别名
    - SELECT语句中可以直接使用子查询, 而UPDATE, DELETE语句中使用子查询, 必须在子查询外再套一层SELECT语句, 否则会报错

```sql
-- 子查询
SELECT * FROM (SELECT * FROM a) AS ta;

SELECT * FROM users WHERE age < (SELECT AVG(age) FROM users);

DELETE FROM users WHERE age < (SELECT * FROM (SELECT AVG(age) FROM users) AS avg_age);
```

#### UNION 合并操作符

合并两个或多个SELECT语句的结果
* 必要条件:
    - 每个SELECT语句必须拥有相同数量的列
    - 列必须拥有相似的数据类型
    - 每个SELECT语句中的列的顺序必须相同
* 默认情况下, UNION结果集会 **去重**, `DISTINCT`关键字也是去重的意思, 所以加不加都一样; 如果允许重复值, 可使用`UNION ALL`

```sql
SELECT column_name, ... FROM table_name1
[WHERE condition]
UNION [ALL|DISTINCT]
SELECT column_name, ... FROM table_name2
[WHERE condition];
```

```sql
SELECT country FROM Websites UNION SELECT country FROM apps ORDER BY country;

SELECT country FROM Websites UNION ALL SELECT country FROM apps ORDER BY country;
```

#### CASE 条件表达式

* CASE表达式类似于`if...then...else`的逻辑
* CASE分为2种
    - 简单CASE: WHEN后的表达式只能完全匹配CASE后的表达式, 相当于使用`=`. 不能匹配`NULL`
    - 搜索CASE: CASE可以作为比较条件, 使用`LIKE`, `!=`, `BETWEEN AND`等, 可以完全替代简单CASE

```sql
-- 简单CASE格式:
CASE 列名或表达式
    WHEN 值1 THEN 返回值1
    ...
    WHEN 值n THEN 返回值n
    ELSE 其他情况返回值
END

-- 简单CASE示例
CASE gender
    WHEN '1' THEN '男'
    WHEN '0' THEN '女'
    ELSE '其他'
END

-- 搜索CASE格式
CASE
    WHEN 列名或表达式 = 值1 THEN 返回值1
    ...
    WHEN 列名或表达式 = 值n THEN 返回值n
    ELSE 其他情况返回值
END

-- 搜索CASE示例
CASE
    WHEN gender = '1' THEN '男'
    WHEN gender = '0' THEN '女'
    ELSE '其他'
END

-- 一个完整的SQL示例
UPDATE table SET uid =
CASE
    WHEN id = 1 THEN 1001  -- id为1的修改为1001
    WHEN id = 2 THEN 1002  -- id为2的修改为1002
    WHEN id = 3 THEN 1003  -- id为3的修改为1003
    ELSE uid               -- id为其他的仍然使用uid的值
END
WHERE id in (1, 2, 3);

-- 输出one. CASE表达式时一个固定值, 相当于如果1=1, 则输出one
SELECT
CASE 1
    WHEN 1 THEN 'one'
    WHEN 2 THEN 'two'
    ELSE 'more'
END;

-- 输出true. CASE中WHEN表达式是一个比较
SELECT
CASE
    WHEN 1 > 0 THEN 'true'
    ELSE 'false'
END;
```


## 函数

* 聚合函数
* scalar函数

### AVG() 平均数

返回列中数值的平均值

```sql
SELECT AVG(column_name, ...) FROM table_name;
```

```sql
SELECT AVG(count) AS CountAverage FROM access_log;
```

### COUNT() 计数

返回匹配指定条件的行数

```sql
SELECT COUNT([DISTINCT] column_name) FROM table_name;
```

```sql
-- 返回记录数
SELECT COUNT(*) FROM access_log;

-- 返回指定列的不同值的数目
SELECT COUNT(DISTINCT column_name) FROM table_name;
```

### MAX() 最大值

返回指定列的最大值

```sql
SELECT MAX(column_name) FROM table_name;
```

```sql
SELECT MAX(alexa) AS max_alexa FROM Websites;
```

### MIN() 最小值

返回指定列的最小值

```sql
SELECT MIN(column_name) FROM table_name;
```

```sql
SELECT MIN(alexa) AS min_alexa FROM Websites;
```

### SUM() 求和

返回指定列数值的总和

```sql
SELECT SUM(column_name) FROM table_name;
```

```sql
SELECT SUM(count) FROM access_log;
```

### UCASE() 转换大写

将字段值转换为大写

```sql
SELECT UCASE(column_name) FROM table_name;
```

```sql
SELECT UCASE(name), url FROM Websites;
```

### LCASE() 转换小写

将字段值转换为小写

```sql
SELECT LCASE(column_name) FROM table_name;
```

```sql
SELECT LCASE(name), url FROM Websites;
```

### MID() 提取字符

从文本字段中提取字符
* `start`从1开始
* 如果不指定length, 则返回剩余文本

```sql
SELECT MID(column_name, start[, length]) FROM table_name;
```

```sql
SELECT MID(name, 1, 4) FROM Websites;
```

### LENGTH() 计算长度

返回文本字段中值的长度

```sql
SELECT LENGTH(column_name) FROM table_name;
```

```sql
SELECT LENGTH(url) FROM Websites;
```

### ROUND() 舍入小数

把数值字段四舍五入为指定小数位数
* decimals: 要返回的小数位数

```sql
SELECT ROUND(column_name[, decimals]) FROM table_name;
```

```sql
SELECT ROUND(1.23);

SELECT ROUND(1.23, 0);
```

### 日期函数

* `NOW()`: 返回当前日期和时间, `YYYY-MM-DD HH:MM:SS:sss`
* `CURDATE()`: 返回当前的日期, `YYYY-MM-DD`
* `CURTIME()`: 返回当前的时间, `HH:MM:SS`
* `DATE()`: 提取日期或日期/时间表达式的日期部分, `YYYY-MM-DD HH:MM:SS:sss`
* `EXTRACT(unit FROM date)`: 返回日期/时间的单独部分
    - `MICROSECOND`
    - `SECOND`
    - `MINUTE`
    - `HOUR`
    - `DAY`
    - `WEEK`
    - `MONTH`
    - `QUARTER`
    - `YEAR`
    - `SECOND_MICROSECOND`
    - `MINUTE_MICROSECOND`
    - `MINUTE_SECOND`
    - `HOUR_MICROSECOND`
    - `HOUR_SECOND`
    - `HOUR_MINITE`
    - `DAY_MICROSECOND`
    - `DAY_SECOND`
    - `DAY_MINITE`
    - `DAY_HOUR`
    - `YEAR_MONTH`
* `DATE_ADD(date, INTERVAL 表达式)`: 向日期添加指定的时间间隔
* `DATE_SUB()`: 从日期减去指定的时间间隔
* `DATEDIFF()`: 返回两个日期之间的天数
* `DATE_FORMAT()`: 用不同的格式显示日期/时间


## 索引

* 分类
    - 单列索引: 一个索引只包含一个列.
    - 组合索引: 一个索引包含多个列. 注意, 一个表中的多个索引不叫组合索引
* 作用
    - 在不读取整个表的情况下, 提高查询效率.
    - 但降低了增删改的效率, 因为还要额外保存一下索引文件
* 注意
    - 在经常需要查找的列上创建索引
    - 不查找的列不要创建索引, 以免影响效率
    - 索引也是一张表, 保存了主键与索引字段, 并指向实体表的记录

### 普通索引

```sql
-- 直接创建索引
CREATE INDEX index_name
ON table_name (column_name(length));

-- 修改表结构时添加索引
ALTER TABLE table_name ADD INDEX [index_name] ON (column_name(length));

-- 创建表时直接指定
CREATE TABLE table_name (
    ...
    INDEX [index_name] (column_name(length))
);
```

### 唯一索引

索引列的值必须唯一, 但允许有空值
组合索引的列值的组合必须唯一

```sql
-- 直接创建
CREATE UNIQUE INDEX index_name ON table_name (column_name(length));

-- 修改表结构
ALTER TABLE table_name ADD UNIQUE [index_name] (column_name(length));

-- 创建表直接指定
CREATE TABLE table_name (
    ...
    UNIQUE [index_name]  (column_name(length))
);
```

### 删除索引

```sql
ALTER TABLE table_name DROP INDEX index_name;
```

### 显示索引

```sql
SHOW INDEX FROM table_name;
```


## View 视图

视图是可视化的表
视图总是最新的数据, 当查询视图时, 数据库会通过视图的SQL语句重建数据

### CREATE VIEW 创建视图

```sql
CREATE VIEW view_name AS
SELECT column_name, ...
FROM table_name
WHERE condition;
```

### CREATE OR REPLACE VIEW 更新视图

```sql
CREATE OR REPLACE VIEW view_name AS
SELECT column_name, ...
FROM table_name
WHERE condition;
```

### DROP VIEW 删除视图

```sql
DROP VIEW view_name;
```


## 临时表

临时表用于存储临时数据, 只在当前连接可见, 当关闭连接时, 会自动删除临时表并释放所有空间

### CREATE TEMPORARY TABLE 创建临时表

```sql
CREATE TEMPORARY TABLE table_name (
    ...
);
```

### DROP TABLE 删除临时表

```sql
DROP TABLE table_name;
```

## 复制表

步骤:
1. 获取数据表的完整结构
2. 修改获取到的SQL语句中的表名为新表名, 并执行语句
3. 使用INSERT INTO语句将原表数据查询出来, 然后插入新表

```shell
# 1. 获取数据表完整结构
mysql> SHOW CREATE TABLE runoob_tbl \G;
*************************** 1. row ***************************
       Table: runoob_tbl
Create Table: CREATE TABLE `runoob_tbl` (
  `runoob_id` int(11) NOT NULL auto_increment,
  `runoob_title` varchar(100) NOT NULL default '',
  `runoob_author` varchar(40) NOT NULL default '',
  `submission_date` date default NULL,
  PRIMARY KEY  (`runoob_id`),
  UNIQUE KEY `AUTHOR_INDEX` (`runoob_author`)
) ENGINE=InnoDB
1 row in set (0.00 sec)

# 2. 修改表名后执行语句
mysql> CREATE TABLE `clone_tbl` (
  -> `runoob_id` int(11) NOT NULL auto_increment,
  -> `runoob_title` varchar(100) NOT NULL default '',
  -> `runoob_author` varchar(40) NOT NULL default '',
  -> `submission_date` date default NULL,
  -> PRIMARY KEY  (`runoob_id`),
  -> UNIQUE KEY `AUTHOR_INDEX` (`runoob_author`)
-> ) ENGINE=InnoDB;
Query OK, 0 rows affected (1.80 sec)

# 3. 查询原表数据并插入新表
mysql> INSERT INTO clone_tbl (runoob_id,
    ->                        runoob_title,
    ->                        runoob_author,
    ->                        submission_date)
    -> SELECT runoob_id,runoob_title,
    ->        runoob_author,submission_date
    -> FROM runoob_tbl;
Query OK, 3 rows affected (0.07 sec)
Records: 3  Duplicates: 0  Warnings: 0
```


## 元数据

元数据包括:
* 查询结果信息: 如影响的记录数
* 数据库和数据表的信息: 包含数据库和数据表的结构信息
* MySQL服务器信息: 包含数据库服务器的当前状态, 版本号等

```sql
-- 服务器版本信息
SELECT VERSION()

-- 当前数据库名
SELECT DATABASE()

-- 当前用户名
SELECT USER()

-- 服务器状态
SHOW STATUS

-- 服务器配置变量
SHOW VARIABLES
```


## 数据导入导出

### SELECT ... INTO OUTFILE 导出数据

将数据导出到文本文件

```sql
SELECT column_name
FROM table_name
INTO OUTFILE 'filepath'
[FIELDS TERMINATED BY 'symbol' [ENCLOSED BY 'symbol']]
[LINES TERMINATED BY 'symbol'];
```

```sql
SELECT a, b, a+b INTO OUTFILE '/tmp/result.text'
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n'
FROM Websites;
```

### LOAD DATA INFILE 导入数据

```sql
-- 按文件列顺序插入数据
LOAD DATA LOCAL INFILE 'filepath' INTO TABLE table_name
[FIELDS TERMINATED BY 'symbol']
[LINES TERMINATED BY 'symbol'];

-- 指定顺序插入列
LOAD DATA LOCAL INFILE 'dump.txt'
INTO TABLE mytbl (b, c, a);
```

### mysqldump 导出表为原始数据

```shell
# 导出原始数据
mysqldump -u root -p --no-create-info --tab=/tmp RUNOOB table_name
password ******

# 导出SQL格式数据
$ mysqldump -u root -p RUNOOB runoob_tbl > dump.txt
password ******
```

### mysqlimport 导入原始数据

```shell
mysqlimport -u root -p --local --fields-terminated-by=":" --lines-terminated-by="\r\n"  database_name dump.txt
password *****
```


## SQL注入

* SQL注入: 通过把SQL语句插入到Web表单提交, 最终到服务器执行恶意SQL命令
* 永远不要相信用户的输入


## 事务管理

* 事务的4个特性
    - 原子性: 事务不可分割
    - 一致性: 事务执行前后数据要保持完整性
    - 隔离性: 一个事务的执行不能受到其他事务的干扰
    - 持久性: 事务一旦结束(提交或回滚), 事务便会持久保存在数据库中
* 不考虑事务隔离性会发生的安全问题
    - 读数据问题
        - 脏读: 一个事务读到了另一个事务未提交的数据操作结果
        - 不可重复读: 一个事务对同一行数据重复读取多次, 但是却得到了不同的结果
            - 虚读: 事务T1读取某一数据后, 事务T2对其做了UPDATE修改, 当事务T1再次读该数据时得到与前一次不同的值
            - 幻读: 事务在操作过程中进行两次查询, 第二次查询的结果包含了第一次查询中未出现的数据或者缺少了第一次查询中出现的数据(这里并不要求两次查询的SQL语句相同). 这是因为在两次查询过程中有另外一个事务INSERT插入数据或DELETE删除数据造成的
    - 写数据问题
        - 更新丢失: 两个事务都同时更新一行数据, 一个事务对数据的更新把另一个事务对数据的更新覆盖了. 这是因为系统没有执行任何的锁操作, 因此并发事务并没有被隔离开来
* 事务的隔离级别可以解决隔离性引发的3种读数据的问题
    - `SELECT @@TX_ISOLATION;`: 查看当前隔离级别
    - `SET [GLOBAL|SESSION] TRANSACTION ISOLATION LEVEL 隔离级别;`: 设置事务隔离级别
        - `READ UNCOMMITTED`: 最低事务级别, 允许以上三类隔离问题的发生
        - `READ COMMITTED`: 不会发生脏读, 允许不可重复读和虚读发生
        - `REPEATABLE READ`: 不会发生脏读, 不可重复读, 允许虚读发生
        - `SERIALIZABLE`: 事务最高级别, 所有相同数据事务操作, 都会串行进行, 不存在并发, 不存在三类隔离问题
* MySQL中事务管理的2种方式
    - 执行SQL语句前手动开启事务
        - 开启事务: `START TRANSACTION;`
        - 提交事务: `COMMIT;`
        - 回滚事务: `ROLLBACK;`
    - 修改MySQL参数, 设置是否自动提交
        - 查看是否已经开启: `SHOW VARIABLES LIKE '%COMMIT%';`, 查看`autocommit`变量的值是否为`ON`
            - `ON`表示开启自动提交; `OFF`表示关闭自动提交
        - 修改自动提交: `SET AUTOCOMMIT = 'OFF';`
            - `'OFF'`: 也可使用数字`0`, 表示关闭自动提交
            - `'ON'`: 也可使用数字`1`, 表示开启自动提交
    - 注意:
        - MySQL默认开启了自动提交, 每条执行的SQL语句都会立刻提交



## 建表原则

* 一对一
    - 2种方式:
        - 唯一外键对应: 将任意一个一方假设为多方, 增加一个字段, 设置为UNIQUE, 且作为外键指向另一个一方的主键
            - UNIQUE约束用来限制该列id值唯一, 这样才能实现一对一唯一对应关系
        - 主键对应: 将两个一方的主键建立映射关系, 即两个一方的主键字段的值是相互一致的, 这样也能实现一对一唯一对应关系
* 一对多
    - 2张表
    - 在多方增加一个字段作为外键指向一方的主键
* 多对多
    - 3张表
    - 创建一个中间关系表, 其中2个字段作为外键分别指向两个多方各自的主键
