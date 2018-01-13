# Oracle

* Oracle公司开发的分布式数据库
* 支持多用户
* 支持大事务量
* 具备数据安全性和完整性控制
* 支持分布式数据处理
* 具有可移植性

## Oracle的数据管理方式

* 数据库: 
    - Oracle在一个操作系统中只有一个数据库
* 实例: instance
    - 一个实例由多个后台进程(background process)和内存结构(memory structures)组成. 
    - 一个库可以有多个实例
* 数据文件: dbf, database file
    - 数据文件是数据库的物理存储单位
    - 一个表空间可以由一个或多个数据文件组成, 一个数据文件只能用于一个表空间
* 表空间:
    - 表空间是Oracle对物理数据库上数据文件的逻辑映射
    - 一个数据库在逻辑上被划分到一个或多个表空间, 每个表空间包含了在逻辑上相关联的一组结构
    - 每个数据库至少有一个表空间(即system表空间)
* 用户:
    - 用户是在表空间下建立的
    - 用户登录后只能看到和操作自己的表
    - 每建立一个应用需要创建一个用户

```
数据库 > 表空间 > 段 > 区 > 数据块
```

## 客户端

* sqlplus: 自带客户端
    - 登录: `sqlplus 用户/密码@IP:1521/库名`
    - 退出: `quit`或`exit`


## 修改字符集

```sql
# 查看当前字符集
select userenv('language') from dual;

# 默认是AMERICAN_AMERICA.ZHS16GBK
```

* 将查询到的AMERICAN_AMERICA.ZHS16GBK添加到环境变量中
    - 创建环境变量`NLS_LANG`, 值为`AMERICAN_AMERICA.ZHS16GBK`

## 数据备份和恢复

* 整库操作

```bash
# 整库导出
exp 用户名/密码 file=导出文件路径 full=y

# 整库导入
imp 用户名/密码 file=导入文件路径 full=y
```

* 按用户操作

```bash
# 按用户导出
exp 用户名/密码 owner=用户名 file=导出文件路径

# 按用户导入
imp 用户名/密码 fromuser=用户名 file=导入文件路径
```

* 按表操作

```sql
# 按表导出
exp 用户名/密码 file=导出文件路径 tables=表名1,表名2,...

# 按表导入
imp 用户名/密码 file=导入文件路径 tables=表名1, 表名2,...
```


## 语法

### 数据类型

* 字符型:
    * CHAR: 固定长度字符, 最多存储2000个字符
    * VARCHAR2: 可变长度字符, 最多存储4000个字符
    * LONG: 大文本类型, 最大存储2G
* 数值型:
    * NUMBER(位数): 数值类型
        * NUMBER(5)表示最大可以存储5位数99999
        * NUMBER(5,2)表示最大可以存储总共5位数, 其中包含2位小数, 999.99
* 日期型:
    * DATE: 日期时间类型, 精确到秒
    * TIMESTAMP: 精确到秒的小数点后9位
* 二进制型
    * CLOB: 存储字符, 最多存储4G
    * BLOB: 存储图像, 声音, 视频等二进制数据, 最多存储4G

### 创建表空间

```sql
-- 格式
CREATE TABLESPACE 表名
DATAFILE 物理文件路径
SIZE 表空间初始大小
AUTOEXTEND 是否自动扩容
NEXT 自动扩容大小;

-- 示例
CREATE TABLESPACE shop
DATAFILE '/usr/local/shop.dbf'
size 1000m
autoextend on
next 10m;
```

### 创建用户

```sql
-- 格式
CREATE USER 用户名
IDENTIFIED BY 密码
DEFAULT TABLESPACE 默认表空间名;

-- 示例
CREATE USER admin
IDENTIFIED BY 1234
DEFAULT TABLESPACE shop;
```

### 用户授权

```sql
-- 格式
GRANT 权限 TO 用户名;

-- 示例
GRANT dba TO admin;
```

### 表操作

#### 创建表

```sql
-- 格式
CREATE TABLE 表名 (
  字段名 类型(长度) 约束,
  字段名 类型(长度) 约束,
  ...
  字段名 类型(长度) 约束
);

-- 示例
CREATE TABLE t_orders (
  oid NUMBER PRIMARY KEY,
  odesc VARCHAR2(300)
);
```

#### 修改表

* 增加字段

```sql
-- 格式
ALTER TABLE 表名 ADD (
  字段名 数据类型 约束,
  ...
  字段名 数据类型 约束
);

-- 示例
ALTER TABLE t_orders ADD (
  oprice NUMBER
);
```

* 修改字段数据类型和约束

```sql
-- 格式
ALTER TABLE 表名 MODIFY (
  字段名 数据类型 约束
);

-- 示例
ALTER TABLE t_orders MODIFY (
  oprice NUMBER DEFAULT 0
);
```

* 修改字段名

```sql
-- 格式
ALTER TABLE 表名 RENAME COLUMN 原字段名 TO 新字段名;

-- 示例
ALTER TABLE t_orders RENAME COLUMN odesc TO odetail;
```

* 删除字段

```sql
-- 格式
ALTER TABLE 表名 DROP COLUMN 字段名;
ALTER TABLE 表名 DROP (字段名1, 字段名2, ..., 字段名n);

-- 示例
ALTER TABLE t_orders DROP COLUMN odetail;
ALTER TABLE t_orders DROP (oname, odetail);
```

#### 删除表

```sql
-- 格式
DROP TABLE 表名;

-- 示例
DROP TABLE t_orders;
```

#### 清空表

```sql
TRUNCATE TALBE 表名;
```

### 记录操作

#### 添加记录

```sql
-- 格式
INSERT INTO 表名
(字段名1, 字段名2, ..., 字段名n)
VALUES
(值1, 值2, ..., 值n);

-- 示例
INSERT INTO t_orders
VALUES
(1, '大米');
```

#### 修改记录

```sql
-- 格式
UPDATE 表名 SET 字段名=值, 字段名=值 WHERE 条件;

-- 示例
UPDATE t_orders SET odetail = '小米' WHERE oid = 1;
```

#### 删除记录

```sql
-- 格式
DELETE FROM 表名 WHERE 条件;

-- 示例
DELETE FROM t_orders WHERE oid = 1;
```

#### 查询记录

```sql
-- 格式
SELECT 字段 FROM 表名 WHERE 条件;

-- 示例
SELECT * FROM t_orders WHERE oid > 10 AND odetail LIKE '%糖%' ORDER BY oid DESC; 
```

* 去重: `SELECT DISTINCT 字段名 FROM 表名;`
* 条件查询
    * 运算符: 等于, 不等, 大于, 大于等于, 小于, 小于等于
    * 范围: `BETWEEN ... AND ...`
    * 列表: `IN (值, 值, ..., 值)`
    * 空: `IS NULL`
    * 模糊查询: `LIKE '规则'`
    * 逻辑: `AND`, `OR`, `NOT`
* 排序:
    * `ORDER BY 字段名 ASC|DESC;`
* 分组:
    * `GROUP BY 字段 HAVING 条件`
* 聚合:
    * 求和: `SUM(字段)`
    * 平均值: `AVG(字段)`
    * 最大值: `MAX(字段)`
    * 最小值: `MIN(字段)`
    * 计数: `COUNT(字段)`
* 伪列查询:
    * 伪列是表中的附加列, 但并不真正存储在表中. 伪列只能查询, 不能增删改
    * `ROWID`: 表示每行记录在数据文件的物理地址
    * `ROWNUM`: 表示结果集中每一行的行号, 第一行为1. 
        * ROWNUM是在查询语句扫描每条记录时产生的, 不能用大于或等于比较, 只能使用小于或小于等于进行比较
        * 如果要根据ROWNUM进行分页, 需要使用子查询, 且排序时先让结果排序, 然后才能生成ROWNUM, 否则ROWNUM和记录会不对应

```sql
-- 查看每条记录的ROWID
SELECT ROWID, t.* FROM t_orders t;

-- 使用ROWID作为条件查询
SELECT ROWID, t.* FROM t_orders t WHERE ROWID = 'xxxxx';

-- 使用ROWNUM用于分页查询: 每页10条, 第2页
SELECT * FROM 
(SELECT ROWNUM, t.* FROM t_orders t WHERE ROWNUM <= 20)
WHERE ROMNUM >= 10;

-- 使用ROWNUM用于分页查询且先排序
SELECT * FROM
(SELECT ROWNUM, t.* FROM
  (SELECT * FROM t_orders ORDER BY oprice DESC) t
WHERE ROWNUM <= 20)
WHERE ROWNUM >= 10;
```

* 连接查询:
    * 内连接:
    * 左外连接
    * 右外连接

```sql

```

* 子查询
    * where子句中的子查询:
        * 单行子查询: 子查询只返回一条记录
            * `=`
	    * `>`
	    * `>=`
	    * `<`
	    * `<=`
	    * `<>`
        * 多行子查询: 子查询返回多条记录
            * `IN`: 等于列表中的任何一个
	    * `ANY`: 和子查询返回的任意一个值比较
	    * `ALL`: 和子查询返回的所有值比较
    * from子句中的子查询:
        * 多行子查询
    * select子句中的子查询:
        * 必须为单行子查询

```sql
-- WHERE单行子查询: 查询价格高于平均价格的记录
SELECT * FROM t_orders 
WHERE oprice > (SELECT AVG(oprice) FROM t_orders);

-- WHERE多行子查询: 查询订单表中订单价格高于10的用户信息
SELECT * FROM t_users 
WHERE uid IN (SELECT uid FROM t_orders WHERE oprice > 10);

-- FROM多行子查询: 查询订单表和用户表中订单金额高于10的姓张的记录 
SELECT * FROM 
(SELECT o.oid, o.oprice, u.name FROM t_orders o, t_users u WHERE o.oprice > 10)
WHERE u.name LIKE '张%';

-- SELECT单行子查询: 
SELECT 
(SELECT name FROM t_users WHERE uid = addid)
FROM t_users;
```

### 单行函数

* 字符函数
    * `ASCII()`: 返回对应字符的十进制值
    * `CHR()`: 给出十进制返回字符
    * `CONCAT(字符串1, 字符串2)`: 拼接两个字符串, 与`||`相同
    * `INITCAT()`: 将字符串的第一个字母变为大写
    * `INSTR()`: 找出某个字符串的位置
    * `INSTRB()`: 找出某个字符串的位置和字节数
    * `LENGTH(字符串)`: 以字符给出字符串的长度
    * `LENGTHB()`: 以字节给出字符串的长度
    * `UPPER()`: 将字符串转为大写
    * `LOWER()`: 将字符串转为小写
    * `LPAD()`: 使用指定的字符在字符的左边填充
    * `RPAD()`: 使用指定的字符在字符的右边填充
    * `TRIM()`: 在左边或右边剪裁指定的字符串
    * `LTRIM()`: 在左边剪裁指定的字符
    * `RTRIM()`: 在右边剪裁指定的字符
    * `REPLACE()`: 字符串搜索替换
    * `SUBSTR(字符串, 开始索引, 个数)`: 截取子字符串
    * `SUBSTRB()`: 截取子字符串, 以字节
    * `SOUNDEX()`: 返回一个同音字符串
    * `TRANSLATE()`: 字符串搜索替换
* 数值函数
    * `ABS()`: 绝对值
    * `CEIL()`: 向上取整
    * `FLOOR()`: 向下取整
    * `ROUND(value, precision)`: 按precision精度四舍五入
    * `COS()`: 余弦
    * `COSH()`: 反余弦
    * `SIN()`: 余弦
    * `SINH()`: 反余弦
    * `TAN()`: 正切
    * `TANH()`: 反正切
    * `SIGN(value)`: value为正返回1; value为负返回-1; value为0返回0 
    * `EXP(value)`: e的value次幂
    * `LN()`: value的自然对数
    * `LOG()`: value的以10为底的对数
    * `MOD()`: 求模
    * `POWER(value, exponent)`: value的exponent次幂
    * `SQRT()`: value的平方根
    * `TRUNC(value, precision)`: 按precision截取value
    * `VSIZE(value)`: 返回value的存储空间大小
* 日期函数
    * `ADD_MONTHS()`: 在日期date上增加count个月
    * `GREATEST(date1, date2)`: 从日期列表中选出最晚的日期
    * `LAST_DAY(date)`: 返回日期date所在月的最后一天
    * `LEAST(date1, date2)`: 从日期列表中选出最早的日期
    * `MONTHS_BETWEEN(date1, date2)`: 给出date1-date2的月数
    * `NEXT_DAY(date, '星期值')`: 给出日期date之后下一天的日期
    * `NEW_TIME(date, 'this', 'other')`: 给出在this时区=other时区的日期和时间
    * `ROUND(date, '格式')`: 中午之前为`12 A.M.`
    * `TRUNC(date, '格式')`: 日期截取
    * 格式:
        * `yyyy`: 年
	* `mm`: 月
* 转换函数
    * `CHARTOROWID`: 将字符转换到ROWID类型
    * `CONVERT`: 转换字符集
    * `HEXTORAW`: 从十六进制转换为RAW
    * `RAWTOHEX`: 转换RAW为十六进制
    * `ROWIDTOCHAR`: 转换ROWID到字符
    * `TO_CHAR`: 转换日期到字符串
    * `TO_DATE`: 按照指定的格式将字符串转换到日期型
    * `TO_MULTIBYTE`: 把单字节字符转换到多字节
    * `TO_NUMBER`: 将数字字符串转换到数字
    * `TO_SINGLE_BYTE`: 转换多字节到单字节
* 其他函数
    * `NVL(检测的值, 为null时的值)`: 空值处理, 如果符合检测的值则替换为默认值
    * `NVL2(检测的值, 不为null时的值, 为null时的值)`: 空值处理, 对是否为null分别替换
    * `decode(条件, 值1, 翻译值1, 值2, 翻译值2, ...)`: 条件取值. 根据条件返回相应值


### 行列转换
  

### 事务操作

* 提交事务

```sql
commit;
```

* 回滚事务

```sql
rollback;
```


## 常见问题

* `DELETE FROM 表`和`TRUNCATE TABLE 表`区别
    * `DELETE FROM 表`是逐条删除记录, 该操作可以回滚, 该删除可能产生碎片, 且不释放空间
    * `TRUNCATE TABLE 表`是将表删除后重建
