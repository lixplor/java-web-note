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
