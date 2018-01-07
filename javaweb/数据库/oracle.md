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
